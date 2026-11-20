# Sesli Denetim Raporu

Gemide denetim yapan kişi eksikliği fotoğraflar ve sesli anlatır; uygulama ekipmanı ve yeri tanır, anlatımı
düzenli bir bulguya çevirir ve raporu Türkçe ya da İngilizce üretir (PDF, Word).

> **Özel depo.** Pruva'dan ayrı bir üründür; ileride Pruva'ya modül olarak eklenebilir ya da ayrı satılabilir.

| Ne | Nerede |
|---|---|
| Çalışan prototip (Claude Artifact) | https://claude.ai/artifact/FnkC222hwHiVPD1aRWJf25 |
| Prototip kaynak dosyası | [`uygulama/index.html`](uygulama/index.html) |
| Mobil uygulama planı (canlı belge) | https://claude.ai/artifact/53vhXB6SfCZhFEigHs94K3 |
| Planın kopyası | [`belgeler/PLAN.md`](belgeler/PLAN.md) |
| Alınan kararlar | [`belgeler/KARARLAR.md`](belgeler/KARARLAR.md) |
| Test | [`testler/`](testler/) |

## Prototipte neler var (20.11.2026)

- Denetim açma: gemi, IMO, bayrak, liman, tarih, tür (iç denetim, superintendent, PSC/vetting hazırlık, devir), denetçi, kaptan
- Bulgu: **Fotoğraf çek** (kamera) / **Galeriden seç**, sesli dikte (Claude dikte; yoksa telefon klavyesinin mikrofonu)
- **Fotoğrafı otomatik tanıma:** yükleyince Claude ekipmanı, yeri ve görünen durumu söyler ("👁 Claude gördü"), emin değilse sarı uyarı
- **Claude ile bulguya çevir:** başlık, ekipman, yer, açıklama, düzeltici faaliyet, önem (Kritik/Büyük/Küçük/Gözlem), bölüm, ilgili kural, PSC kodu — TR ve EN birlikte
- **Denetlenenler listesi:** gemi başına doğrulanan ekipman/yerler; Claude yeni fotoğraflarda önce bu adları kullanır (kendi kendine öğrenme)
- Rapor: TR/EN görünüm, Claude ile genel değerlendirme, **PDF** ve **Word** indirme, metni kopyalama
- **Pruva'ya aktar:** bulgular Pruva `defects` yapısına uygun JSON
- Arayüz TR/EN, açık/koyu tema, telefon için büyük dokunma alanları

## Teknik özet

- Tek dosyalık HTML; Claude Artifact yetenekleri: `db` (insp, find koleksiyonları), `assets` (fotoğraflar),
  `sample` (görüntülü Claude çağrıları), `dictation`, `downloads`, `user`.
- PDF: pdfmake 0.2.10, Word: docx 8.5.0 (ihtiyaç anında CDN'den yüklenir).
- Veri yapısı: `insp/{id}` (denetim), `find/{id}` (bulgu: `tr`/`en` {title, equip, loc, desc, action}, sev, area, refs, psc,
  photos[{id,w,h}], note, seen, unsure, conf, noLearn).

## Yayınlama

Artifact slug: `77c30713-df91-4d2d-9734-c1971885c04e`. `uygulama/index.html` düzenlenir, `publish_files` ile
`index.html` olarak yayınlanır (yetenekler korunur).

## Test

```
cd testler && npm install && npx playwright install chromium && node t.js
```

Sahte Claude ortamında fotoğraf yükleme, dikte, otomatik tanıma, bulgu yazma, özet, denetlenenler listesi ve
PDF/Word/Pruva çıktısını dener.
