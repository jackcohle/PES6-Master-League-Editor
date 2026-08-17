# PES6 Master League Editor v1.0 — Teknik Dokümantasyon ve Geliştirme Rehberi

> **Proje:** PES6 Master League Editor  
> **Dokümante edilen sürüm:** v1.0 Stable  
> **Yazar:** jackcohle  
> **Hedef:** Pro Evolution Soccer 6 (PC, 32-bit) + Cheat Engine  
> **Bu belgenin amacı:** kararlı CT'nin iç mimarisini açıklamak; başka Cheat Engine geliştiricilerinin sistemi inceleyebilmesini, fork oluşturabilmesini veya mevcut güvenlik mimarisini bozmadan yeni özellikler ekleyebilmesini sağlamak.

> **Reverse-engineering notu:** Bu belgede kullanılan fonksiyon ve yapı adları, reverse-engineering sırasında bu proje tarafından verilen açıklayıcı isimlerdir; PES6 executable dosyasındaki **resmî semboller değildir**. Sabit adresler, module offsetler, byte signature'ları ve yapıların görevlerine ilişkin çıkarımlar yalnızca **test edilen buildler** için belgelenmiştir ve farklı yamalarda değişebilir. Belgede anlatılan gözlemlenmiş davranışlar, Konami tarafından yayımlanmış resmî bir iç yapı tanımı değil, kararlı v1.0 uygulamasında doğrulanan davranışlardır.

---

## 1. Kapsam

Bu belge aşağıdaki işleri yapmak isteyen geliştiriciler içindir:

- yeni Player Editor alanları veya işlemleri eklemek;
- yeni oyuncu profilleri eklemek;
- yeni takım geneli presetler hazırlamak;
- yeni Master League veya maç kontrolleri eklemek;
- kadro/oyuncu resolver sistemini başka bir CT'de kullanmak;
- Score Controls ve native Game History yapısını anlamak;
- sistemleri başka bir PES6 yamasına uyarlamak;
- tüm mimariyi değiştirmeden uyumluluk sorunlarını araştırmak.

Bu belge normal kullanıcı özelliklerini anlatan bir README değildir. Normal kullanım için `README_TR.md` dosyasına bakın.

Aşağıdaki teknik ayrıntılar final kararlı v1.0 CT'nin mevcut uygulamasını açıklar. İç semboller, offsetler ve yardımcı fonksiyonlar kalıcı bir API değil, **uygulama ayrıntısıdır**.

---

## 2. Test Edilen Oyun Dosyaları

Kararlı v1.0 şu iki executable üzerinde doğrulandı:

| Ortam | Executable | Boyut | SHA-256 |
|---|---|---:|---|
| Standart / yamasız PES6 | `PES6.exe` | 21,880,832 bayt | `53263fb86b10b1bd2a9a962816c55ba23954e8f0596da80e8adebb4fead3295e` |
| PES 6 Original Season yaması | `pes6.exe` | 21,880,832 bayt | `cd30427917be6a903ea4624147ca8506c7db5462a4a4e9f50ee8dd6c9d494628` |

Kararlı CT SHA-256:

`ce9c35802e9a5aca2a29e2e273aa889eca41fcd30d75230796562fb3bfc44b1d`

İki executable byte-byte aynı değildir. Yeni bir hook, tespit edilmiş bir PES6 routine çağrısı veya sabit module offset eklendiğinde özellik uyumlu kabul edilmeden önce **iki test edilen executable üzerinde de** doğrulanmalıdır.

---

## 3. Temel Tasarım İlkeleri

Kararlı CT'de tek tek adreslerden daha önemli bazı kurallar vardır:

1. **Tek bir doğrulanmış Master League kadro haritası kullanın.** Player Selector, Player Editor, Squad Ability Presets ve Squad Fitness Overview birbirinden bağımsız farklı oyuncu listeleri tahmin etmemelidir.
2. **Sadece Player ID'ye güvenmeyin.** Aynı ID'yi görmek, kaydın hedeflenen canlı Master League oyuncusu olduğunu kanıtlamaz.
3. **Sürekli process taraması yerine doğrulanmış/önbelleğe alınmış hedefleri tercih edin.**
4. **Event veya değişiklik bazlı yazma yeterliyse sürekli yazmayın.**
5. **Maç sonu / ara sahne gibi güvenli olmayan geçişlerde canlı yazmaları durdurun.**
6. **Match History yapısını sabit boyutlu dizi olarak değil native değişken uzunluklu yapı olarak ele alın.**
7. **Karmaşık işlemlerde yazmadan önce snapshot alın ve işlem sonunda doğrulama yapın.**
8. **Doğrulama başarısızsa tahmin etmek yerine işlemi güvenli şekilde iptal edin.**
9. **`[ACTIVATE]` kapandığında bütün geçici durum temizlenmelidir.**

CT'nin bazı bölümlerinin basit bir static-address cheat table'dan daha savunmacı görünmesinin nedeni budur.

---

## 4. Genel Mimari

CT dört ana katmandan oluşur.

### 4.1 Auto Assembler hook katmanı

Ana `[ACTIVATE]` kaydı temel hook'ları kurar ve ortak editör belleğini ayırır.

Önemli hook grupları:

- mevcut oyuncu resolver yakalama;
- Master League isim/root yakalama;
- Master League ability synchronization yakalama;
- isteğe bağlı Infinite Stamina hook'u.

Hook'lar `aobscanregion` / `aobscanmodule` ile bulunur ve `assert` ile beklenen byte'lar doğrulanır.

Ana resolver hook'unun görevi bütün editörü çalıştırmak değil; Lua tarafının kullanacağı doğru runtime Player ID / pointer bilgilerini sağlamaktır.

### 4.2 Ortak symbol / mirror katmanı

CT aşağıdaki gibi semboller ayırır ve register eder:

- `ml3_team_ids`
- `ml3_team_base_record_ptrs`
- `ml3_team_current_ability_ptrs`
- `ml3_selected_id`
- `ml3_selected_ptr`
- `ml3_contract_record_ptr`
- `ml3_ability_values`
- `ml3_position_flags`
- `ml3_special_flags`
- `pes6_ml_match_time_selector`
- `pes6_ml_home_score_status`
- `pes6_ml_away_score_status`

Player Editor satırlarının çoğu değişken bir oyun pointer'ına doğrudan bağlanmak yerine editörün kendi sembollerine bağlanır.

Bu ayrım önemlidir: kullanıcı sabit editör buffer'ını düzenler, runtime ise değişikliğin gerçek oyuna **ne zaman ve hangi doğrulanmış kayda** yazılacağına karar verir.

### 4.3 Lua yönetim katmanı

Ana activation kaydı resolver, cache, kadro, preset, recovery, score/clock ve güvenlik mantığını tanımlar.

Ana runtime timer **60 ms** aralıkla çalışır.

Başlıca görevleri:

- display-only skor mirror değerlerini güncellemek;
- güvenli olmayan maç sonu durumlarını tespit etmek;
- Custom Match Time controller'ı yürütmek;
- doğrulanmış Master League kadrosunu hazırlamak/kurtarmak;
- Player Selector / Overview durumunu yenilemek;
- seçili oyuncuyu çözümlemek;
- fitness/contract kayıtlarını takip etmek;
- Player Editor buffer'ı gerçekten değiştiğinde write-back yapmak;
- squad preset devamlılığını sürekli yazma yapmadan korumak.

### 4.4 Tek seferlik action katmanı

Görünür `[ACTION]`, `[PROFILE]` ve `[SELECT]` kayıtlarının çoğu bilinçli olarak küçük wrapper'lardır.

Ortak Lua fonksiyonlarını çağırırlar ve ardından kendilerini otomatik olarak kapatırlar.

Örnekler:

- `PES6MLApplyEditorAction(...)`
- `PES6MLApplyPlayerPreset(...)`
- `PES6MLSetSingleTeamSlot(...)`
- `PES6MLApplySelectedRecovery(...)`
- `PES6MLApplyTeamRecovery(...)`
- `PES6MLRequestTeamPreset(...)`

Yeni özellik eklerken büyük kod bloklarını her görünür CheatEntry içine kopyalamak yerine ortak implementasyonu genişletmek tercih edilmelidir.

---

## 5. Activation ve Kapanış Yaşam Döngüsü

### 5.1 Enable

`[ACTIVATE]` etkinleştirildiğinde CT:

1. gerekli kod imzalarını tarar;
2. `assert` ile beklenen byte'ları doğrular;
3. ortak buffer ve pointer dizilerini ayırır;
4. görünür memory record'ların kullandığı sembolleri register eder;
5. resolver/name/ability hook'larını kurar;
6. Lua helper fonksiyonlarını ve cache'leri oluşturur;
7. önceki Match Clock runtime durumunu temizler;
8. 60 ms döngüsünde kullanılacak sabit skor/status adreslerini cache'ler;
9. ana runtime timer'ı başlatır;
10. ilk Overview / Player Selector durumunu oluşturur veya yeniler.

### 5.2 Disable

`[ACTIVATE]` kapatılması gerçek bir session sınırıdır.

Kapanış kodu:

- Raw clock freeze durumunu bırakır;
- gerektiğinde aktif squad preset'i restore eder;
- runtime timer'larını yok eder;
- dinamik Overview kayıtlarını siler;
- player/preset/name/scan cache'lerini temizler;
- eski resolver pointer'larını temizler;
- post-match durumunu temizler;
- sembolleri unregister eder;
- hook'lanan orijinal byte'ları geri yükler;
- ayrılmış belleği serbest bırakır.

Bu nedenle kullanıcı dokümantasyonunda Master League'den tamamen çıktıktan sonra `[ACTIVATE]` kapatılıp tekrar açılması istenir.

### Geliştirme kuralı

Yeni bir global tablo, timer, dinamik CheatEntry, cache, allocated buffer veya registered symbol ekliyorsanız aynı değişiklik içinde `[DISABLE]` temizliğini de ekleyin.

---

## 6. Ortak 32-Slot Master League Kadrosu

Editör maksimum 32 slotluk kadro haritası kullanır.

Ana sembol:

`ml3_team_ids`

Bu, 32 adet WORD Player ID dizisidir.

İlişkili pointer dizileri:

- `ml3_team_base_record_ptrs`
- `ml3_team_current_ability_ptrs`
- `ml3_team_source_ability_ptrs`

Her slotun dolu olduğu varsayılmaz. Boş/geçersiz ID'ler yok sayılır.

### 6.1 Kadronun hazırlanması

`PES6MLPrepareTeamFromFitness()` kararlı kadroyu Squad Fitness Overview ile aynı kaynaktan oluşturur.

Kadro kullanılabilir kabul edilmeden önce yeterli sayıda geçerli Master League oyuncusu bulunmalıdır. Mevcut kod en az 11 geçerli oyuncu ister.

Kadro signature değişirse önceki takıma bağlı cache'ler temizlenir.

### 6.2 Neden kadro "frozen"

Geçerli takım hazırlandıktan sonra Player Selector ve takım geneli işlemler aynı slot haritasını kullanır.

Böylece resolver değişirken "5. oyuncu" gibi bir slotun anlamının sessizce başka bir oyuncuya dönmesi önlenir.

### 6.3 Tek oyuncu seçimi

Player Selector:

`PES6MLSetSingleTeamSlot(slot, true)`

çağrısını yapar ve bu:

`PES6MLLockTeamSlot(slot)`

üzerinden çözümlenir.

Seçimde şunlar birlikte kullanılır:

- roster slot;
- Player ID;
- yakalanmış base/current pointer'ları;
- isim doğrulaması;
- squad-preset target map ile ortak recovery mimarisi.

Başka oyuncu seçilene kadar hedef kilitli kalır.

### 6.4 Duplicate ID ve transfer/özel oyuncular

**Sadece ID ile oyuncu çözmeyin.**

CT'de ek isim/root/pointer doğrulamaları olmasının nedeni, farklı kayıtların aynı ID'yi taşıyabilmesi veya PES6'nın geçici/canlı kopyalar oluşturabilmesidir.

Özel oyuncularda Player Editor bir **shadow mode** kullanabilir:

- lokal 0x7C boyutunda player biçimli buffer hazırlanır;
- kurtarılan 72-byte live block `shadow + 0x34` adresine kopyalanır;
- Player Editor shadow üzerinde çalışır;
- doğrulanmış write-back gerçek live copy'yi günceller.

Yeni player özelliği iki durumda da test edilmelidir:

1. normal full-record modu;
2. recovered/shadow live-block modu.

---

## 7. Player Record Modeli

### 7.1 Boyutlar

Normal player record için kullanılan yapı:

- full record size / stride: `0x7C`
- edit edilen canlı ability block başlangıcı: `+0x34`
- editörün kullandığı current ability block: **72 byte**

Squad/preset fonksiyonlarının çoğu bu nedenle `origin = 0x34` olan 72-byte block üzerinde çalışır.

### 7.2 26 temel ability

26 değer `ml3_ability_values[0..25]` olarak tutulur.

Full player record'a göre offsetler:

| Ability index | Offset |
|---:|---:|
| 0–19 | `0x36` – `0x49` |
| 20 | `0x4A` |
| 21 | `0x4C` |
| 22 | `0x4D` |
| 23 | `0x4E` |
| 24 | `0x4F` |
| 25 | `0x4B` |

Ability değeri byte'ın alt 7 bitindedir.

Bazı byte'ların bit 7'sinde pozisyon veya special flag bulunduğu için tüm byte'ı doğrudan overwrite etmeyin.

### 7.3 Playable Positions

12 adet `ml3_position_flags` vardır.

Position flag'leri full-record:

`0x36` – `0x41`

offsetlerinin **bit 7** kısmında tutulur.

Alt 7 bit ability olduğundan read-modify-write yapılmalıdır.

### 7.4 Special Abilities

23 adet `ml3_special_flags` vardır.

v1.0 mapping:

| Grup | Depolama |
|---|---|
| İlk 14 special | `0x44,0x45,0x46,0x47,0x48,0x49,0x4A,0x4B,0x4C,0x4D,0x4E,0x4F,0x43,0x42` offsetlerinde `0x80` biti |
| Sonraki 8 special | `0x52` offsetinde `01,02,04,08,10,20,40,80` maskeleri |
| Long Throw | `0x54` offsetinde `0x80` biti |

Bu alanlarda mutlaka read-modify-write mantığı kullanılmalıdır.

### 7.5 Packed performans/stil alanları

#### Full record `+0x34`

- bit 0: Preferred Foot
- bit 1–4: Free Kick Style
- bit 5–7: Penalty Kick Style - 1

#### Full record `+0x35`

- bit 0–1: Dribbling Style - 1
- bit 2–3: Drop Kick Style - 1
- bit 4–7: Registered Position

#### Full record `+0x50`

- bit 0–2: Consistency - 1
- bit 3–5: Weak Foot Frequency - 1
- bit 6–7: Injury Tolerance

#### Full record `+0x51`

- bit 0–2: Condition - 1
- bit 3–5: Weak Foot Accuracy - 1
- bit 6–7: Favoured Side

### 7.6 Identity / fiziksel encoding

#### Full record `+0x58`

- alt 6 bit: Height encoded value
- gösterilen boy = low6 + `0x94` (148)
- üst 2 bit: Skin Colour

#### Full record `+0x59`

- alt 7 bit: Weight raw

#### Full record `+0x70`

- alt 7 bit: Nationality

#### Full record `+0x71`

- bit 1–5: encoded age
- gösterilen yaş = encoded + 15

Bu encoding'ler mevcut Player Editor kodunda hazırdır. Yeni kod yazarken mevcut bit helper'larını kullanmak daha güvenlidir.

---

## 8. Player Editor Buffer ve Write-Back

Görünür Player Editor alanları çoğunlukla oyun kaydına doğrudan değil CT'nin kendi sembollerine yazılır.

Runtime editör durumundan bir signature oluşturur.

`Instant Live Write-Back` açıkken:

1. 60 ms döngüsü mevcut editor signature ile önceki signature'ı karşılaştırır;
2. değişiklik varsa `PES6ML40WriteCurrent()` çalışır;
3. seçili doğrulanmış hedef güncellenir;
4. gerektiğinde bilinen stale/live copy'ler reconcile edilir;
5. live-write counter artırılır.

### Yeni Player Editor alanı için kural

Yeni değer editable player block içindeyse:

1. editor symbol ayır/register et;
2. oyuncu yüklenirken decode et;
3. pack/signature içine ekle;
4. write-back sırasında encode et;
5. yasal aralığı clamp/validate et;
6. aynı byte'taki ilgisiz bitleri koru;
7. shadow mode'u test et;
8. görünür memory record'u doğrudan game pointer'a değil editor symbol'e bağla.

`ml3_selected_ptr` üzerinden doğrudan volatile pointer editleri normalde tercih edilmemelidir.

---

## 9. Contract, Fitness ve Shirt Number

Bu değerler:

`ml3_contract_record_ptr`

üzerinden çözülür.

Mevcut record offsetleri:

| Değer | Offset / bit |
|---|---|
| Shirt Number | `+0x03`, Byte |
| Match Condition | `+0x0C`, bit 0–2 |
| Pre-Match Stamina | `+0x0D`, Byte |
| Accumulated Fatigue | `+0x0E`, Byte |
| Contract Years Remaining | `+0x11`, bit 2–7 |
| Yearly Salary | `+0x12`, 2 Bytes |

Bunlar geçici 72-byte ability/preset block'tan ayrıdır.

Bazıları kullanıcı Master League'i oyun içinden kaydettiğinde kalıcı olabilir. Yeni değer eklenirse save davranışı ayrıca dokümante edilmelidir.

---

## 10. Quick Actions ve Player Profiles

### 10.1 Quick Actions

Görünür işlemler:

`PES6MLApplyEditorAction(kind)`

çağrısını kullanır.

Yeni quick action mümkün olduğunca **Player Editor buffer'ını** değiştirmeli ve gerçek write işlemini mevcut write-back sistemine bırakmalıdır.

### 10.2 Player Profiles

Profiller:

`PES6MLApplyPlayerPreset(key)`

üzerinden uygulanır.

Profil tanımları merkezde tutulur; her görünür memory record içine tekrar tekrar büyük write kodu konmaz.

### Önerilen wrapper

```text
[ENABLE]
{$lua}
if syntaxcheck then return end
if not PES6MLApplyPlayerPreset then
  error('Activate the main editor first.')
end

PES6MLApplyPlayerPreset('my_new_profile')

local r=memrec
local t=createTimer(nil,false)
t.Interval=100
t.OnTimer=function(x)
  x.destroy()
  if r and r.Active then r.Active=false end
end
{$asm}

[DISABLE]
```

Asıl profil verisini ortak profile table / preset fonksiyonuna ekleyin.

---

## 11. Squad Recovery

Squad Recovery de aynı doğrulanmış kadro haritasını kullanır.

Public giriş noktaları:

- `PES6MLApplySelectedRecovery(mode)`
- `PES6MLApplyTeamRecovery(mode)`

Yeni takım recovery özelliği için ikinci bir bağımsız "takımımı bul" sistemi yazmayın.

---

## 12. Squad Ability Presets

### 12.1 Public giriş noktası

Görünür işlemler:

`PES6MLRequestTeamPreset(mode)`

çağırır.

Bu fonksiyon:

`PES6MLSetTeamPreset(mode)`

üzerinden doğrulanmış işlemi yapar.

### 12.2 Mevcut mode değerleri

| Mode | İşlem |
|---:|---|
| 0 | Restore original squad ability values |
| 1 | Complete Squad Boost |
| 2 | 26 ability = 99 |
| 3 | Tüm Special Abilities |
| 4 | Max Performance Settings |
| 5 | +5 abilities |
| 6 | +10 abilities |
| 7 | +15 abilities |
| 8 | +20 abilities |
| 9 | Ultimate Squad by Position |

Byte dönüşümü merkezde:

`PES6MLApplyModeToBytes(bytes, origin, mode)`

ile yapılır.

### 12.3 Yazmadan önce backup

Preset bir hedefe yazmadan önce orijinal 72-byte hedefi yedekler.

Target çözümleme veya write başarısızsa eski durum geri yüklenmeye çalışılır.

Bu hem kullanıcıdaki Restore Original özelliği hem de yarım kalan işlemlerin güvenliği için gereklidir.

### 12.4 Session persistence

Preset session-based'dir; fakat PES6 maç sonrasında player ability copy'lerini yeniden oluşturabilir.

CT bütün oyunculara her 60 ms'de tekrar yazmaz.

Bunun yerine:

- yeni ortaya çıkan doğrulanmış resolver record aktif overlay'i bir kez alabilir;
- lightweight mass-reload detector bilinen preset hedeflerini karşılaştırır;
- anlamlı sayıda oyuncu aynı anda resetlenmişse preset yeniden uygulanır.

Mevcut eşik:

- en az 6 geçerli hedef kontrol edilmeli;
- `max(6, ceil(checked * 0.25))` veya daha fazla hedef mismatch olmalı.

Kısa cooldown menü geçişlerinde tekrar tekrar burst write yapılmasını önler.

### 12.5 Yeni squad preset eklemek

1. kullanılmayan yeni mode numarası seç;
2. `PES6MLApplyModeToBytes` içine dönüşümü ekle;
3. mevki bazlıysa mevcut Registered Position role mantığını kullan;
4. `PES6MLRequestTeamPreset(newMode)` çağıran one-shot action ekle;
5. Active Squad Preset dropdown açıklamasını güncelle;
6. Restore Original test et;
7. normal maç sonrası persistence test et;
8. şampiyonluk/kutlama geçişini test et;
9. bireysel Player Editor override'larının preset'in üstünde kalmasını test et.

---

## 13. Maç Sonu / Kutlama Güvenlik Sistemi

Bu sistem korunması gereken en kritik yapılardan biridir.

`PES6MLUpdatePostMatchSafety()` Raw match clock ile resolver yaşam döngüsünü takip eder.

### 13.1 Guard'ın aktif olması

Raw=0 görülmesi tek başına guard'ı devreye sokmaz.

Önce gerçekten çalışan bir maç belirli sayıda tick boyunca görülmelidir.

Çalışan period bittikten sonra Raw 0'a düşerse korumalı moda girilir.

### 13.2 Korumalı durumda

- Player Editor live write durur;
- roster preparation write yapmaz;
- squad preset resolver-follow write durur;
- eski resolver/scan cache'leri atılır;
- eski resolver generation'ın unload olması beklenir.

### 13.3 İkinci yarı

Devre arası da Raw değeri 0 yapar.

Aynı maç devam edip Raw yeniden pozitif olduğunda guard kaldırılır; bu durum tam maç sonu teardown olarak değerlendirilmez.

### 13.4 Gerçek maç sonundan sonra resume

CT:

- resolver unload kanıtı;
- kısa güvenlik gecikmesi;
- yeni doğrulanmış Master League roster

bekler.

Daha sonra yeni pointer'lar kabul edilir ve aktif squad preset yeni hedeflerde yeniden kurulur.

### Geliştirme kuralı

Maç sırasında canlı player/team write yapan yeni bir özellik eklenirse kendi full-time detector'ını yazmak yerine mevcut post-match safety state kullanılmalıdır.

---

## 14. Score Controls Mimarisi

Final v1.0 Score Controls bilinçli olarak konservatif tasarlanmıştır.

### 14.1 Display-only skor satırları

`[STATUS] Home Score` ve `[STATUS] Away Score` gerçek skor adresine bağlı değildir.

Şu mirror sembollere bağlıdır:

- `pes6_ml_home_score_status`
- `pes6_ml_away_score_status`

60 ms timer yalnızca mirror değeri gerçek skordan farklıysa mirror'a tekrar gerçek değeri yazar.

Dolayısıyla kullanıcı CE Value alanına başka sayı yazsa da oyun skoru değişmez; display tekrar gerçek skora döner.

### 14.2 Test edilen buildlere ait uygulama adresleri

Final uygulama iki test edilen build'de aşağıdaki adresleri kullanır. Tablodaki isimler **proje tarafından verilen açıklayıcı adlardır**; adresler evrensel bir PES6 memory map olarak değil, bu buildlere ait uygulama referansları olarak değerlendirilmelidir.

| Proje etiketi / amaç | Test edilen build adresi |
|---|---|
| Remaining Match Time (Raw) | `pes6.exe+37E098C` |
| Home gameplay score | `pes6.exe+C17B30` |
| Home display/result score | `pes6.exe+C17B38` |
| Away gameplay score | `pes6.exe+C17E24` |
| Away display/result score | `pes6.exe+C17E2C` |
| Home goal counter base | `pes6.exe+C17B5C` |
| Home assist counter base | `pes6.exe+C17B60` |
| Away goal counter base | `pes6.exe+C17E50` |
| Away assist counter base | `pes6.exe+C17E54` |
| Played-player map | `pes6.exe+C18DE8` |
| Home roster IDs | `pes6.exe+CD4536` |
| Away roster IDs | `pes6.exe+CD775E` |
| Raw Game History stream | `pes6.exe+C1DD70` |
| Game History metadata/index | `pes6.exe+C18F50` |
| Game History raw-byte offset | `pes6.exe+C24170` |
| Game History record count | `pes6.exe+C24172` |
| Native event-size table | `pes6.exe+C24174` |
| Native match-stat dispatcher | `pes6.exe+1FFDE0` |
| Native goal-history builder | `pes6.exe+200350` |
| Native history appender path | `pes6.exe+203DA0` |
| Native history metadata rebuild | `pes6.exe+203E80` |

Bu adresler başka patchler için bir uyumluluk garantisi değildir. Aynı implementasyon başka bir buildde kullanılmadan önce hem adres hem de çevresindeki kod/veri semantiği yeniden doğrulanmalıdır.

### 14.3 Add Home/Away Goal

Final stabil Add Goal **yalnızca golcü** oluşturur.

Sentetik asist bilinçli olarak oluşturulmaz.

İşlem sırası:

1. Raw remaining time > 0 şartı;
2. played-player map ve roster çözümlemesi;
3. gerçek oynayan saha oyuncularından bir golcü seçimi;
4. skor, goal/assist ve hidden per-period goal counter snapshot;
5. assistant identity `0xFF` ("no assistant") olan native event oluşturma;
6. event kind `6` ile doğrulanmış native stat dispatcher çağrısı;
7. şu kontroller:
   - total goal tam +1;
   - dört hidden per-period counter'dan yalnızca biri +1;
   - assist counter'ları değişmemiş;
8. gameplay + display skor byte'larını artırma;
9. PES6 native goal-history builder çağrısı;
10. native history count ve byte offsetin tam beklenen miktarda arttığını doğrulama;
11. herhangi bir failure'da değiştirilen counter/skorları rollback.

Rastgele golcü seçiminde kaleci hariç tutulur.

### 14.4 Hidden per-period goal counter'ları

PES6 total goal counter'ın hemen önünde dört WORD counter tutar:

`goalCounterAddress - 8 + period*2`, `period=0..3`

Master League maç sonu gol istatistiği için bunlar önemlidir.

Sadece görünür total goal counter'ı değiştirmek yeterli değildir.

### 14.5 Native Game History değişken uzunlukludur

Kritik kural:

> **Bütün Game History kayıtlarını 8 byte kabul etmeyin.**

Stabil Remove/Reset parser'ı:

- metadata offsetlerini;
- metadata event type bilgisini;
- native event-size table'ı

kullanır.

Raw stream değişken uzunluklu native event stream olarak ele alınır.

### 14.6 Remove Home/Away Goal

Remove:

1. Raw > 0 kontrolü yapar;
2. mevcut skor ile native history'nin senkron olduğunu doğrular;
3. bütün native event stream'i parse eder;
4. istenen taraftaki son golü seçer;
5. goal event içinden scorer'ı çözer;
6. yalnızca native record gerçekten assistant içeriyorsa assistant'ı çözer;
7. raw history, metadata, score ve player counter snapshot alır;
8. yalnızca hedef goal event byte'larını çıkarır;
9. şunları azaltır:
   - scorer total;
   - bir hidden per-period scorer counter;
   - silinen doğal golde asist varsa assistant match counter;
10. ilgili tarafın skorunu 1 azaltır;
11. PES6 native history metadata rebuild fonksiyonunu çağırır;
12. yeniden parse edip sonucu doğrular;
13. herhangi bir failure'da snapshot'ın tamamını geri yükler.

Bu nedenle Remove hem CT ile eklenen scorer-only golleri hem doğal asistli golleri güvenli şekilde silebilir.

### 14.7 Reset Score to 0–0

Reset bütün Game History'yi silmez.

Şunları yapar:

- native **goal event** kayıtlarını çıkarır;
- goal dışındaki eventleri korur;
- iki skoru 0 yapar;
- goal ve assist match counter'larını 0 yapar;
- hidden per-period goal counter'larını 0 yapar;
- native Game History metadata rebuild yapar;
- final yapıyı doğrular.

### 14.8 Raw = 0 koruması

Add / Remove / Reset `Remaining Match Time (Raw) == 0` iken çalışmaz.

Raw 0 şu pencereleri kapsayabilir:

- stoppage-time / period transition;
- devre arası;
- maç sonu;
- kutlama / teardown.

Butonları her an çalıştırmak için bu korumayı kaldırmak önerilmez.

---

## 15. Native Routine Signature Doğrulaması

Score Controls match state üzerinde değişiklik yapmadan önce önemli PES6 routine'lerini doğrular.

Aşağıdaki routine adları **bu projenin kullandığı açıklayıcı iç etiketlerdir**. Bunlar executable içindeki resmî symbol adları değildir ve Konami'nin orijinal fonksiyon isimlendirmesine ilişkin bir iddia taşımaz.

Final signature'lar:

| Proje etiketi | Module offset | Beklenen başlangıç |
|---|---:|---|
| Match-history appender path | `+203DA0` | `0F B7 05 70 41 02 01` |
| Native match-stat dispatcher | `+1FFDE0` | `56 8B 74 24 0C 85 F6` |
| Native goal-history builder | `+200350` | `83 EC 18 53 56 57 8B 7C 24 28` |
| Native metadata rebuild | `+203E80` | `51 55 0F B7 2D 72 41 02 01` |

Bir patch bu byte'ları değiştirdiyse güvenli davranış işlemi iptal etmek ve o build için uyumluluğu ayrıca geliştirmektir.

Unsupported EXE'yi çalıştırmak için signature check'leri silmeyin.

---

## 16. Match Clock Controls

### 16.1 Native match-time

Native match-time mode:

`pes6.exe+388F1B0`, alt 3 bit.

### 16.2 Custom selector

Editör sembolü:

`pes6_ml_match_time_selector`

İzin verilen değerler:

- `0` = PES6 Native
- `3` = 3 Minutes
- `7` = 7 Minutes
- `12` = 12 Minutes

Mapping:

| Custom hedef | PES6 Native base | Native mode |
|---:|---:|---:|
| 3 dk | 5 dk | 0 |
| 7 dk | 10 dk | 1 |
| 12 dk | 15 dk | 2 |

Bu iki maç süresi ayarı **Master League menü ekranında, maç başlamadan önce** değiştirilmelidir.

Maç başladıktan sonra pre-match selector/native duration kilitlenir. Devam eden maçın doğrudan süre kontrolü yalnızca `Remaining Match Time (Raw)` üzerinden yapılır.

### 16.3 Raw time

Raw clock:

`pes6.exe+37E098C`

Custom-time controller normal raw azalmayı takip eder ve kickoff sonrasında native base'i değiştirmeden hedef custom süreyi elde etmek için kontrollü ek azalma uygular.

Custom seçimde güvenli Raw limiti:

`safeMax = baseMinutes * 1800`

Aralık dışı değer reddedilir ve son güvenli değer geri yüklenir.

### 16.4 End Current Period

`End Current Period` onay isteyen one-shot action'dır ve Raw time değerini 0 yapar.

---

## 17. Infinite Stamina

Infinite Stamina 60 ms write loop yerine ayrı AOB hook kullanır.

Runtime'da kullanıcının Master League takımının Home/Away tarafını belirler ve stamina depletion yalnızca o tarafa uygulanmaz.

Genişletirken korunması gereken kurallar:

- side detection runtime bazlı kalmalı;
- normal kullanımda sabit Home veya Away zorlanmamalı;
- değiştirdiğiniz register/flag'ler korunmalı;
- disable'da orijinal instruction byte'ları geri yüklenmeli;
- allocated/registered her kaynak temizlenmeli.

---

## 18. Master League Settings ve Finances

Test edilen buildlerdeki mevcut uygulama referansları:

| Değer | Adres | Taşınabilirlik notu |
|---|---|---|
| Match Time (PES6 Native) | `pes6.exe+388F1B0`, alt 3 bit | Test edilen build module offseti |
| Game Difficulty | `pes6.exe+388F1B1`, alt 3 bit | Test edilen build module offseti |
| Transfer Frequency | `pes6.exe+388F1B5`, bit 0–1 | Test edilen build module offseti |
| Master League Difficulty | `pes6.exe+388F1B5`, bit 3–5 | Test edilen build module offseti |
| Current Funds | `03CE229C` | **Patch-sensitive absolute address** |

`Current Funds` özellikle **patch-sensitive** olarak işaretlenmiştir; çünkü kararlı CT bu değeri runtime'da yeniden çözümlenen module-relative bir hedef yerine doğrudan absolute address üzerinden kullanır. Başka bir executable veya patch için uyumluluk belirtilmeden önce bu adres bağımsız olarak yeniden doğrulanmalıdır.

---

## 19. Performans / CPU Kuralları

Kararlı v1.0 gereksiz sürekli işlemleri azaltacak şekilde temizlendi.

### Yapın

- sabit adresleri mümkün olduğunda bir kez cache'leyin;
- yazmadan önce mevcut değeri karşılaştırın;
- player/team cache'leri kullanın;
- takım signature veya generation değişince cache'i invalidate edin;
- pahalı recovery scan'lerini yalnız normal resolution başarısız olursa kullanın;
- repair logic için cooldown kullanın;
- görünür action'ları one-shot yapın;
- gerçekten event-driven işlemlerde AOB hook tercih edin.

### Yapmayın

- her 60 ms'de tüm process üzerinde `AOBScan`;
- hot loop içinde aynı sabit sembolü tekrar tekrar çözümlemek;
- preset aktif kalsın diye 26 oyuncuya sürekli yazmak;
- zaten doğru olan değeri her tick yeniden yazmak;
- normal Player Editor değişikliğinde tüm memory'yi taramak;
- one-shot action record'unu sürekli Active bırakmak.

### Mevcut örnekler

60 ms runtime gerçek/mirror score adreslerini başta cache'ler ve yalnız mirror ile gerçek değer farklıysa write yapar.

Squad preset persistence da her tick tüm kadroyu rewrite etmek yerine event/değişiklik mantığı kullanır.

---

## 20. Güvenli AOB Hook Şablonu

Yeni code hook için konsept olarak:

```text
[ENABLE]

aobscanmodule(MY_HOOK,pes6.exe,<specific signature>)
assert(MY_HOOK,<expected original bytes>)
alloc(MY_MEM,1024,MY_HOOK)

label(return)

MY_MEM:
  // değiştirdiğiniz register/flag'leri koruyun
  // minimum gerekli işi yapın
  // overwrite edilen orijinal instruction'ları yeniden çalıştırın
  jmp return

MY_HOOK:
  jmp MY_MEM
  nop
return:

registersymbol(MY_HOOK)

[DISABLE]

MY_HOOK:
  db <exact original bytes>

unregistersymbol(MY_HOOK)
dealloc(MY_MEM)
```

Kurallar:

- signature yeterince spesifik ve mümkünse unique olsun;
- iki desteklenen EXE'de doğrulayın;
- orijinal semantiği koruyun;
- kendi kodunuz register değiştiriyorsa oyunun bunu beklemediğini varsaymayın;
- `assert` kaldırmayın;
- overwrite edilen byte'ları disable'da birebir geri yükleyin.

---

## 21. Güvenli Lua Action Şablonu

Görünür action normalde one-shot olmalıdır:

```text
[ENABLE]
{$lua}
if syntaxcheck then return end

local ok,err=pcall(function()
  -- state validation
  -- tek transaction
  -- sonuç doğrulaması
end)

if not ok then
  showMessage('Action failed:\n\n'..tostring(err))
end

local r=memrec
local t=createTimer(nil,false)
t.Interval=100
t.OnTimer=function(x)
  x.destroy()
  if r and r.Active then r.Active=false end
end
{$asm}

[DISABLE]
```

Karmaşık memory editlerinde transaction içine snapshot/rollback aşaması ekleyin.

---

## 22. Yeni Player Editor Alanı Ekleme Checklist

1. değerin hangi yapıda olduğunu belirleyin:
   - 72-byte live player block;
   - contract/fitness record;
   - başka Master League structure;
2. iki desteklenen EXE'de doğrulayın;
3. Player Editor block içindeyse:
   - stabil editor symbol ayırın;
   - shared player-load içinde decode edin;
   - pack/signature state'e ekleyin;
   - shared write-back içinde encode edin;
   - aynı byte'taki ilgisiz bitleri koruyun;
4. validation/clamp ekleyin;
5. editor symbol'e bağlı görünür CheatEntry ekleyin;
6. normal oyuncu test edin;
7. kaleci test edin;
8. transferred/unusual shadow-mode oyuncu test edin;
9. maç sonrası reload test edin;
10. Master League çıkış/giriş test edin.

---

## 23. Yeni Match Feature Ekleme Checklist

Match özellikleri Player Editor alanlarından daha sıkı ele alınmalıdır.

1. değer türünü belirleyin:
   - yalnız UI;
   - gameplay state;
   - maç sonu kalıcı state;
   - native event/history state;
2. PES6'nın senkron kalmasını beklediği bütün duplicate/derived copy'leri belirleyin;
3. period transition sırasında tehlikeliyse Raw/match guard ekleyin;
4. `executeCodeEx` ile çağrılan her native fonksiyona signature doğrulaması ekleyin;
5. dokunulacak bütün memory alanlarını snapshot alın;
6. mümkün olan en küçük değişikliği yapın;
7. game-owned metadata varsa native rebuild/update fonksiyonunu kullanın;
8. final counter/yapıyı doğrulayın;
9. failure'da rollback yapın;
10. şunları test edin:
    - ilk yarı;
    - stoppage/Raw 0;
    - devre arası;
    - ikinci yarı;
    - maç sonu;
    - result screen;
    - maç sonrası Master League menüsü.

Bir byte'ı değiştirince görüntü doğru oldu diye o değerin yalnızca "visual" olduğunu varsaymayın.

---

## 24. Score / History Geliştirmesinden Çıkan Dersler

Bu bölüm gelecekte aynı hataların tekrar edilmemesi için repository'de kalmalıdır.

### 24.1 Ekranda doğru görünmek internal state'in doğru olduğu anlamına gelmez

Score/history değişikliği doğru görünüp devre/maç sonunda PES6 structure'ı işlerken crash olabilir.

Transition boundary mutlaka test edilmelidir.

### 24.2 Game History metadata içerir

Raw event byte'ları ile metadata/index table uyumlu olmalıdır.

Record silinirse native rebuild routine kullanın.

### 24.3 Event boyları değişkendir

Game'in event-size table'ını kullanın. Evrensel 8-byte event size hard-code etmeyin.

### 24.4 Gol istatistiğinin hidden state'i vardır

Total goal counter tek başına yeterli değildir. Önündeki dört per-period WORD Master League post-match goal stat zincirinde önemlidir.

### 24.5 Sentetik asist bilinçli olarak kararlı v1.0 kapsamının dışındadır

Kararlı v1.0 bilinçli olarak **scorer-only sentetik gol** oluşturur; çünkü Master League'deki kalıcı asist commit davranışı bu projenin kararlı sürüm standardını karşılayacak düzeyde doğrulanmamıştır.

Bu bir kararlılık sınırıdır; live assist alanının önemsiz olduğu varsayımı değildir. Yalnızca live assist counter'ını artırmak veya goal-history kaydına assistant identity yazmak, asistin kalıcı Master League istatistiklerine doğru şekilde commit edildiğini kanıtlamaya tek başına yeterli değildir.

Sentetik asist yeniden ele alınırsa ayrı bir reverse-engineering özelliği olarak geliştirilmelidir: tam post-match assist commit yolu bulunup doğrulanmalı, maç geçişleri boyunca test edilmeli ve ancak bundan sonra kararlı sürüme alınmalıdır.

---

## 25. Diagnostics

Stabil CT şu read-only tanı satırlarını içerir:

- Last Resolver Player ID
- Selected Player Record Address
- Live Write Counter
- Selected Contract Record Address
- Requested Player ID
- Selected Roster Slot
- Selected Player ID Occurrences

Yeni intrusive hook eklemeden önce bunları kullanın.

Development build'de geçici diagnostic eklemek normaldir; fakat kullanılmayan capture counter/hook'larını public stable CT'den çıkarın.

---

## 26. Sık Görülen Hata Türleri

### Yanlış oyuncu editleniyor

Genellikle shared roster/slot resolver bypass edilip yalnız live pointer veya Player ID'ye güvenildiğinde oluşur.

### Preset maç sonuna kadar çalışıyor, sonra crash

Çoğunlukla stale pointer veya geçici cutscene/post-match record'a write işaretidir.

### Preset maç sonrası kayboluyor

PES6 ability copy'leri rebuild etmiş olabilir. Sürekli yüksek frekanslı write loop yerine mevcut mass-reload repair mimarisini kullanın.

### Add/Remove doğru görünüyor ama sonra crash

Score, player counter, raw history, metadata ve hidden stat state'in tamamı doğrulanana kadar transaction tamamlanmış kabul edilmemelidir.

### CT boşta yüksek CPU kullanıyor

Kontrol edin:

- timer içinde full-process scan;
- tekrar tekrar symbol çözme;
- unconditional write;
- her tick tüm oyuncularda loop;
- cooldown/generation kontrolü olmayan repair code.

---

## 27. Yeni Patch İçin Uyumluluk Stratejisi

Yeni PES6 patch desteği eklerken:

1. executable boyutu ve SHA-256 kaydedin;
2. main resolver AOB test edin;
3. My Team name-build AOB test edin;
4. ML ability-sync AOB test edin;
5. Infinite Stamina AOB test edin;
6. ekleyeceğiniz özelliğin kullandığı fixed match/module offsetleri doğrulayın;
7. native function signature'larını doğrulayın;
8. en az 11 oyunculu roster detection test edin;
9. transferred/unusual oyuncu test edin;
10. Player Selector + Player Editor test edin;
11. squad preset'i maç geçişinde test edin;
12. mümkünse championship/celebration test edin;
13. score Add/Remove/Reset'i devre ve maç sonundan geçirerek test edin;
14. Native ve Custom Match Time test edin;
15. idle CPU kontrol edin.

Sadece `[ACTIVATE]` açıldı diye bir patch'i supported ilan etmeyin.

---

## 28. Fork / Contribution İçin Önerilen Kurallar

- CheatEntry ID'leri unique tutun;
- görünür açıklamaları kullanıcı odaklı ve kısa tutun;
- experimental diagnostics'i açıkça işaretleyin;
- eklenen her fixed address/native function'ı dokümante edin;
- özelliğin temporary mi save-persistent mı olduğunu yazın;
- enable sırasında oluşturulan her kaynak için disable cleanup ekleyin;
- başka EXE çalışsın diye safety check'leri sessizce kaldırmayın;
- shared core code değiştiğinde iki orijinal desteklenen build'i tekrar test edin;
- README davranış açıklamalarını CT ile senkron tutun;
- release yaparken yeni stable CT SHA-256 üretin.

Proje **GPL-3.0** altında dağıtılmaktadır. Fork veya değiştirilmiş sürüm dağıtanlar lisans şartlarına uymalı ve lisansın gerektirdiği bildirim/source erişimini korumalıdır.

---

## 29. Faydalı İç Extension Point'ler

| Fonksiyon / sembol | Amaç |
|---|---|
| `PES6MLSetSingleTeamSlot(slot, enabled)` | doğrulanmış squad slot seçmek |
| `PES6MLLockTeamSlot(slot)` | slot'u Player Editor'a resolve/lock etmek |
| `PES6MLApplyEditorAction(kind)` | selected-player quick action |
| `PES6MLApplyPlayerPreset(key)` | selected-player profile |
| `PES6MLApplySelectedRecovery(mode)` | selected-player recovery |
| `PES6MLApplyTeamRecovery(mode)` | full-squad recovery |
| `PES6MLRequestTeamPreset(mode)` | public squad preset request |
| `PES6MLSetTeamPreset(mode)` | validated squad preset transaction |
| `PES6MLRecoverOverviewTeamPointers()` | verified squad pointer recovery |
| `PES6MLUpdateFitnessOverview()` | Overview oluşturma/güncelleme |
| `PES6MLUpdatePostMatchSafety()` | unsafe transition guard |
| `PES6MLResetMatchClockControls(resetRaw)` | Raw controller reset/release |
| `ml3_team_ids` | stabil 32-slot roster ID map |
| `ml3_selected_id` / `ml3_selected_ptr` | Player Editor mevcut identity/record |
| `ml3_contract_record_ptr` | selected ML contract/fitness record |
| `ml3_ability_values` | editör tarafı 26 ability buffer |
| `ml3_position_flags` | editör tarafı 12 position flag |
| `ml3_special_flags` | editör tarafı 23 special flag |

Bu fonksiyonlar yalnızca ana editör aktifken mevcuttur.

---

## 30. Son Kural

Yeni özellik geliştirirken:

> **verified roster → validated target → minimum transaction → gerekiyorsa native synchronization → verification → failure durumunda rollback**

yaklaşımını,

> **bir kere adres buldum → sürekli yaz**

yaklaşımına tercih edin.

Kararlı v1.0 mimarisinin temel mantığı budur.