# Spotify Backstage Catalog `validate-entity` Yerel Dosya Açığa Çıkması (`GHSA-h8rc-g67j-87x3`)

[English](../51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | **Türkçe** | [日本語](../ja/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | [简体中文](../zh-CN/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md) | [한국어](../ko/51-SPOTIFY-BACKSTAGE-CATALOG-VALIDATE-ENTITY-LOCAL-FILE-READ.md)

**VulKill bulgusu:** Spotify/Backstage denetiminde bulundu ve 2026-11-01 tarihinde yerel çalışma zamanı testlerinde gözlendi; üst akış duyurusu 2026-11-18 tarihinde yayımlandı.
**VulKill denetim kimliği:** `NEW-BS-04`.
**Üst akış:** [`GHSA-h8rc-g67j-87x3`](https://github.com/backstage/backstage/security/advisories/GHSA-h8rc-g67j-87x3) · CVSS 3.1 `6.5 Medium` · `CWE-200` / `CWE-863`.

## 1. Kod Kusurunun Güvenlik Açığına Dönüşmesi

`POST /validate-entity`, sunucu kontrollü konumu atadıktan sonra çağıranın açıklamalarını bunun üzerine birleştiriyordu. Kullanıcı konum bağlamını `file:` ile değiştirebiliyor; işlemci yerel dosya okuyucusuna ulaşıyor ve YAML hata mesajları kaynak satırlarını HTTP yanıtına ekliyordu. VulKill gerçek `createRouter`, `AuthorizedValidationService`, işlemci ve `FileReaderProcessor` yolunu düşük yetkili/yönetici kontrolleriyle çalıştırdı.

Etkilenen sürümler: `@backstage/plugin-catalog-backend < 3.9.3` ve `>= 4.0.0, < 4.0.3`. Düzeltme: `3.9.3`, `4.0.3`, `4.0.4`, `4.0.5`.

## 2. Günlük Hayat Senaryoları

- **Senaryo A:** Standart oturumlu kullanıcı catalog sürecinin okuyabildiği bir yapılandırma satırını görür.
- **Senaryo B:** Ayrıştırma hatası sır içeren kaynak satırını HTTP yanıtında yansıtır.
- **Senaryo C:** Sahte kaynak açıklaması depo bağlamı sınırını aşar.

## 3. Çalışma Zamanı Gözlemi

```text
kontrol: yetkisiz misafir -> 403
RED:     catalog.entity.validate kullanıcısı + sahte konum açıklaması -> yerel işaretçi görünür
GREEN:   sunucu açıklamaları önceliklidir; yerel okuma engellenir
BENIGN:  geçerli URL tabanlı varlık doğrulaması çalışır
```

- Test edilen kaynak commit'i: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Harness SHA-256: `0bd744cadf97f031fd218d97eb187fa0394c7ec0b23d8c702fe7eec52f0403ce`
- Zincir kanıtı SHA-256: `a8f607e3adb9747d12b1995a3cbedd68ab2477378d52b92a5244fb2ada41cc82`

## 4. Savunma

Yamalı `@backstage/plugin-catalog-backend` sürümüne geçin; o zamana kadar `catalog.entity.validate` iznini güvenilir kullanıcılara sınırlayın.
