# Sesli Denetim Raporu — Mobil Uygulama Planı

Kopya tarihi: 20.11.2026. Canlı ve güncel sürüm: https://claude.ai/artifact/53vhXB6SfCZhFEigHs94K3
(mimari ve yol haritası çizimleri yalnız canlı belgede).

Sesli Denetim Raporu, App Store ve Google Play'de yayınlanan, gemide internetsiz de çalışan bir telefon uygulaması
olacak. Bugünkü Claude içi sürüm, ekranların ve veri yapısının prototipi olarak kalır.

## Amaç ve kapsam

Denetçi gemide fotoğraf çeker, sesli anlatır. Uygulama ekipmanı ve yeri tanır, bulguyu yazar, raporu Türkçe ve
İngilizce üretir.

**İlk mağaza sürümünde (v1) olacaklar:**

- Denetim açma: gemi, IMO, liman, tür, denetçi
- Bulgu: fotoğraf (kamera ya da galeri), telefonun kendi sesli diktesi, elle düzeltme
- Fotoğrafı otomatik tanıma: ekipman, yer, görülen durum
- Bulgu yazma: başlık, açıklama, düzeltici faaliyet, önem, kural, PSC kodu
- Denetlenenler listesi: gemi başına öğrenme
- Rapor: TR/EN, PDF ve Word, paylaşım menüsü (e-posta, WhatsApp, Dosyalar)
- İnternetsiz çalışma ve sonradan eşitleme
- Pruva'ya aktarım dosyası

**Rapor şablonu (karar):** Müşteri kendi rapor şablonunu (antetli Word) yükleyebilir; rapor o şablona doldurulur.
Şablon yüklemeyen müşteride uygulamanın kendi şablonu kullanılır.

**v1'de olmayacaklar:** şirket yönetim paneli, şirket satışı, üçüncü dil, Pruva ile canlı bağlantı. Bunlar v2'ye kalır.

## Mimari

Uygulama üç parçadan oluşur: telefondaki uygulama, şirketin sunucusu ve dış bağlantılar (Claude API, Pruva).
Claude API anahtarı yalnız sunucuda durur, telefona hiç inmez.

- **Telefon uygulaması:** Kamera, sesli dikte, yerel veritabanı (denetim, foto, ses), gönderim kuyruğu,
  internetsiz rapor. Bugünkü ekranlar Capacitor ile yerel uygulamaya sarılır.
- **Sunucu (AB bölgesi):** API (giriş, şirket, rol), PostgreSQL veritabanı, şifreli fotoğraf deposu (S3 uyumlu),
  Claude işleri (tanıma, bulgu, özet), rapor üretimi (TR/EN, PDF ve Word).
- **Dış bağlantılar:** Claude API (görüntü + metin), Pruva (v1'de dosya ile, v2'de doğrudan), web paneli (v2).

## Sesli dikte ve kamera

Dikte, telefonun kendi konuşma tanımasıyla yapılır; mikrofon düğmesi iki sistemde de çalışır, ek ücret çıkmaz.

| | iPhone (iOS) | Android |
| --- | --- | --- |
| Konuşma tanıma | Apple Speech (SFSpeechRecognizer) | Android SpeechRecognizer (Google) |
| Türkçe ve İngilizce | Var | Var |
| İnternetsiz dikte | Cihaz üstü tanıma destekleyen modellerde | Çevrimdışı dil paketi indirilmişse |
| Kamera | Uygulama içi kamera, flaş, yakınlaştırma | Uygulama içi kamera, flaş, yakınlaştırma |
| İzinler | Mikrofon, konuşma tanıma, kamera, fotoğraflar | Mikrofon, kamera |

- İnternetsiz dikte desteği cihaza ve dile göre değişir; filodaki telefonlarda Türkçe çevrimdışı tanıma test edilmeli.
- Ses kaydı ayrıca dosya olarak saklanır; dikte eksik kalırsa sunucuda yeniden yazıya çevrilebilir.
- Fotoğraflar telefonda sıkıştırılır (en uzun kenar 1600 px).

## İnternetsiz çalışma ve eşitleme

1. Denetim, bulgular, fotoğraflar ve ses kayıtları önce telefondaki yerel veritabanına (SQLite) yazılır.
2. Her işlem bir "gönderilecekler" kuyruğuna girer: fotoğraf yükleme, fotoğrafı tanıma, bulgu yazma, rapor.
3. Bağlantı gelince kuyruk sırayla boşalır; her bulguda "bekliyor / gönderildi / Claude yazdı" durumu görünür.
4. İnternet yokken de elle yazılan bulgulardan PDF rapor alınabilir.
5. Aynı denetimi iki kişi düzenlerse alan bazında son yazan kazanır; fotoğraflar birleşir.

Ayarlarda "yalnız Wi-Fi'da fotoğraf gönder" seçeneği olur (uydu internetinde veri pahalı).

## Kullanıcılar, güvenlik ve veri

| Rol | Neler yapar |
| --- | --- |
| Denetçi | Denetim açar, bulgu ve rapor hazırlar |
| Teknik müdür (v2) | Tüm denetimleri görür, raporu onaylar |
| Şirket yöneticisi (v2) | Kullanıcı ve gemi ekler, abonelik yönetir |
| Kaptan (isteğe bağlı) | Kendi gemisinin raporunu görür, düzeltme kanıtı ekler |

- Giriş: e-posta ile tek kullanımlık kod; kurumsal müşteride Microsoft/Google ile giriş.
- Telefon kaybolursa uzaktan oturum kapatma; telefondaki veriler şifreli.
- Claude'a giden veriler: fotoğraf, anlatım ve geminin denetlenenler listesi. API verileri model eğitiminde kullanılmaz.
- Veri yeri AB (KVKK ve GDPR); fotoğraflarda mürettebat yüzü olabileceği için KVKK aydınlatma metni.
- Günlük yedek; silinen denetim 30 gün geri alınabilir.

## Mağaza yayını ve maliyet

| Kalem | Nasıl ödenir | Not |
| --- | --- | --- |
| Apple Developer Program | Yıllık, yaklaşık 99 USD (doğrulanmalı) | Şirket hesabı için D-U-N-S numarası |
| Google Play geliştirici hesabı | Tek sefer, yaklaşık 25 USD (doğrulanmalı) | Yeni hesaplarda kapalı test zorunluluğu olabilir |
| Claude API | Kullanım kadar | Fotoğraf başına 2 çağrı: tanıma + bulgu yazma |
| Sunucu ve veritabanı | Aylık | Kullanıcı sayısıyla artar |
| Fotoğraf depolama | GB başına aylık | Denetim başına 30-60 fotoğraf varsayımı |

Bireysel abonelik mağaza içi satın alma ile satılır; mağaza komisyonu uygulanır.

## Aşamalar ve takvim (tahmini 14-15 hafta)

| Aşama | Süre | İçerik |
| --- | --- | --- |
| 1. Hazırlık | 1-2 hafta | Mağaza hesapları, ad ve pilot gemi, sunucu bölgesi |
| 2. Temel | 4 hafta | Sunucu ve giriş, kamera, dikte, Capacitor kabuğu |
| 3. İnternetsiz | 3 hafta | Yerel veritabanı, kuyruk, eşitleme, telefonda rapor |
| — Pilot kapısı | | İnternetsiz uçtan uca bir denetim |
| 4. Pilot | 4 hafta | Gerçek gemide denetim, düzeltme, maliyet ölçümü |
| — Yayın kapısı | | Pilot gemide 10 denetim sorunsuz |
| 5. Yayın (v1) | 2 hafta | App Store, Google Play, mağaza incelemesi, kullanıcı eğitimi |

## Rakipler ve fiyat önerisi (araştırma: 20.11.2026)

Mağazalarda denizciliğe özel denetim uygulamaları var; ancak fotoğraftan ekipman tanıyıp sesli anlatımı iki dilde
kurallı bulguya çeviren ve gemi başına öğrenen bir uygulama görülmedi. Büyük oyuncular fiyatı teklif usulü veriyor.

| Uygulama | Öne çıkan | Fiyat |
| --- | --- | --- |
| [Kaiko Systems](https://kaikosystems.com) | Mobil denetim, bulut panel, KAI yapay zekâ asistanı; 250+ gemi | Teklif ile |
| [DNV ShipManager Mobile Inspection](https://dnv.com/services/shipmanager-qhse-mobile-inspection-app-177001) | QHSE denetimi; metin, ses, foto, video; internetsiz | ShipManager ile, teklif |
| [Hanseaticsoft CFM Inspection Report](https://cloudfleetmanager.com) | Foto, video, yorum, sesli not | Cloud Fleet Manager ile, teklif |
| [UniSea Inspections](https://apps.apple.com/ca/app/id6484271813) | İnternetsiz denetim, web'de rapor | Yayınlanmamış |
| [SSH Audit](https://apps.apple.com/us/app/ssh-audit/id6473077793) | SIRE/OCIMF, internetsiz imzalı PDF, sesli not | Ücretsiz indirme |
| [Marine Vessel Inspection](https://apps.apple.com/us/app/id1440993124) | PDF rapor, foto, video, sesli not | Ücretsiz sürüm 2 gemi; uygulama içi satın alma |
| [Shipsanitor](https://apps.apple.com/app/id6738053220) | Sağlık-hijyen denetimi; AI ile yeniden yazma, sesli yazı | Ücretsiz + uygulama içi |
| [MIA Maritime AI Inspector](https://apps.apple.com/app/id6769673083) | PSC, SIRE 2.0 için AI soru-cevap | Ücretsiz + uygulama içi |
| [SafetyCulture](https://safetyculture.com/monthly-billing-faqs) (genel) | Hazır denetim formları | 24 USD/kullanıcı/ay yıllık, 29 USD aylık |
| [Navatom](https://navatom.com/pricing/details) (tam gemi yönetimi) | PMS, personel, satınalma, HSEQ | 250–600 USD/gemi/ay |
| [Sealogical](https://sealogical.com/pricing) (tam gemi yönetimi) | Kullanıcı başına ücret yok | 360 USD/gemi/ay |

Sektör yazısına göre orta ölçekli denizcilik denetim yazılımları gemi başına yıllık 10.000–50.000 USD
([kaynak](https://marineinspection.app/article/marine-inspection-software-pricing-what-fleets-pay-2026)).

**Fiyat önerisi (taslak):**

| Paket | Kime | Taslak fiyat |
| --- | --- | --- |
| Deneme | Herkes | 7 gün tam sürüm (mağaza hesabına bağlı, raporlarda filigran) |
| Denetçi | Bireysel: sörveyör, superintendent | Aylık 29 USD ya da yıllık 290 USD |
| Filo | Gemi işletmecisi | v2'de; taslak 40–80 USD/gemi/ay |
| Şablon kurulumu | Müşteri şablonu isteyen | Tek seferlik ücret |

**Deneme kötüye kullanımına karşı:** (1) deneme mağaza aboneliğine bağlı, Apple ID / Google hesabı başına bir kez,
ödeme yöntemiyle; (2) cihaz işareti (iOS DeviceCheck, Android cihaz kimliği); (3) deneme raporlarında
"DENEME SÜRÜMÜ" filigranı.

**Türkiye fiyatı:** Türkiye App Store'da abonelik medyanı ABD'nin yaklaşık %49 altında
([kaynak](https://opentherank.com/tr/app-price-index/turkey/)). [Site Audit Pro](https://apps.apple.com/tr/app/site-audit-pro/id430234732?l=tr)
Türkiye'de aylık 46,99 TL, yıllık 399,99 TL; [RiskAI](https://apps.apple.com/tr/app/id6760133422) (Türk İSG, AI)
kredi satıyor (10 kredi 10,99 TL – 1000 kredi 1.000 TL). Türkiye'de bireysel denetçiye yönelik denizcilik
uygulaması bulunamadı. Claude maliyeti dolar bazlı; TL planında aylık rapor üst sınırı önerilir.

## Açık sorular

- [ ] Uygulamanın ve şirketin ticari adı (Kerteriz Marine Yazılım altında mı?)
- [ ] Mobil kabuk: Capacitor mı, React Native mi?
- [ ] Pilot müşteri ve pilot gemi; kaç denetçi?
- [ ] Şablon kurulumu ücretli mi?
- [ ] Pruva ile canlı bağlantı v1'de mi, v2'de mi?
- [ ] Türkiye TL fiyatı ve aylık rapor sınırı
