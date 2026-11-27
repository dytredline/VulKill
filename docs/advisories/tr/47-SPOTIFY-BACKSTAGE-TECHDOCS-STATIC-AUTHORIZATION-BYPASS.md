# Spotify Backstage TechDocs Statik İçerik Yetkilendirme Atlaması (`GHSA-rg9r-hr7g-5gc2` / `CVE-2026-106489`)

[English](../47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | **Türkçe** | [日本語](../ja/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | [简体中文](../zh-CN/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md) | [한국어](../ko/47-SPOTIFY-BACKSTAGE-TECHDOCS-STATIC-AUTHORIZATION-BYPASS.md)

**VulKill bulgusu:** Spotify/Backstage denetiminde bulundu ve 2026-11-01 tarihinde gerçek üst akış modülleriyle yerel çalışma zamanı testlerinde gözlendi. Üst akış duyurusu 2026-08-28 tarihinde yayımlanmıştı.
**Üst akış:** [`GHSA-rg9r-hr7g-5gc2`](https://github.com/backstage/backstage/security/advisories/GHSA-rg9r-hr7g-5gc2) · `CVE-2026-106489` · CVSS 3.1 `6.5 Medium` · `CWE-862`.

## 1. Kod Kusurunun Güvenlik Açığına Dönüşmesi

TechDocs statik içerik rotası, çift eğik çizgi içeren hazırlanmış bir yolu yetkilendirme ve depolama çözümlemesi sırasında farklı normalleştiriyordu. İzin çerçevesinin açık olduğu kurulumlarda, yalnız bir dokümantasyon varlığını okuyabilen kullanıcı harici depolamadaki başka bir varlığın içeriğine ulaşabiliyordu. VulKill; gerçek `createRouter`, `LocalPublish` ve `AzureBlobStoragePublish` modüllerini sentetik işaretçilerle çalıştırdı.

Etkilenen sürüm: `@backstage/plugin-techdocs-backend < 2.2.4`. Düzeltme: `2.2.4`.

## 2. Günlük Hayat Senaryoları

- **Senaryo A:** Bir geliştirici başka ekibin kısıtlı çalışma kılavuzunu okur.
- **Senaryo B:** Dahili adresler veya dokümana yazılmış örnek kimlik bilgileri açığa çıkar.
- **Senaryo C:** Paylaşımlı Backstage kurulumunda arayüzdeki varlık izni backend statik içerik rotasında aşılır.

## 3. Çalışma Zamanı Gözlemi

```text
kontrol: düşük yetkili kullanıcı + normal yol -> 404
RED:     aynı kullanıcı + hazırlanmış statik yol -> 200, korunan işaretçi döndü
GREEN:   düzeltme sonrası -> 404
BENIGN:  yetkili kullanıcı + normal yol -> 200
```

- Test edilen kaynak commit'i: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Harness SHA-256: `30bdbb5fa044c7568985a06b2e27c498296d1883344b728b99919f6736b8a665`
- Ortak runtime log SHA-256: `86cec273f9c154713aa8b9afb4283ad4b65422466adc12fff4710484512af0de`

## 4. Savunma

`@backstage/plugin-techdocs-backend 2.2.4` veya daha yeni bir sürüme geçin.
