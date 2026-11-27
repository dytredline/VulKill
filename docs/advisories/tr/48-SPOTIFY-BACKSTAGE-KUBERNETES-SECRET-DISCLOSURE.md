# Spotify Backstage Kubernetes Kaynaklarında Gizli Veri Açığa Çıkması (`GHSA-p795-mqf2-36mf` / `CVE-2026-106561`)

[English](../48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | **Türkçe** | [日本語](../ja/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | [简体中文](../zh-CN/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md) | [한국어](../ko/48-SPOTIFY-BACKSTAGE-KUBERNETES-SECRET-DISCLOSURE.md)

**VulKill bulgusu:** Spotify/Backstage denetiminde bulundu ve 2026-11-01 tarihinde gerçek Kubernetes router/fetcher akışında yerel çalışma zamanı testlerinde gözlendi. Üst akış duyurusu 2026-08-28 tarihinde yayımlanmıştı.
**Üst akış:** [`GHSA-p795-mqf2-36mf`](https://github.com/backstage/backstage/security/advisories/GHSA-p795-mqf2-36mf) · `CVE-2026-106561` · CVSS 3.1 `5.0 Medium` · `CWE-200`.

## 1. Kod Kusurunun Güvenlik Açığına Dönüşmesi

Backstage normal `Secret.data` alanını maskelerken Kubernetes kaynağı açıklamalarındaki eşdeğer gizli veriyi her sorgu yolunda kaldırmıyordu. Standart kaynak okuma iznine sahip kullanıcı, backend'in gizlemesi beklenen düz metin değerleri alabiliyordu. VulKill bu davranışı gerçek `KubernetesRouter`, `KubernetesProxy` ve `KubernetesClientBasedFetcher` yolunda sentetik bir sır işaretçisiyle gözledi.

Etkilenen sürüm: `@backstage/plugin-kubernetes-backend < 0.21.9`. Düzeltme: `0.21.9`.

## 2. Günlük Hayat Senaryoları

- **Senaryo A:** Servis kaynaklarını inceleyen geliştirici açıklamada kalan veritabanı parolasını görür.
- **Senaryo B:** Üretilmiş manifestte kalan CI/CD anahtarı maskeyi aşar.
- **Senaryo C:** Paylaşımlı kümede başka bir dahili servise ait kimlik bilgisi açığa çıkar.

## 3. Çalışma Zamanı Gözlemi

```text
kontrol: doğrudan secret proxy isteği -> 403
RED:     izinli servis-kaynağı sorgusu -> 200; Secret.data maskeli, açıklama işaretçisi görünür
GREEN:   düzeltme sonrası gizli açıklama değeri maskelenir
BENIGN:  hassas olmayan kaynak metadatası kullanılabilir kalır
```

- Test edilen kaynak commit'i: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Harness SHA-256: `d6acd23d33238a8dda450ca77e744f008094e415f80f4efdc39cb9a71071a9a8`
- Ortak runtime log SHA-256: `86cec273f9c154713aa8b9afb4283ad4b65422466adc12fff4710484512af0de`

## 4. Savunma

`@backstage/plugin-kubernetes-backend 0.21.9` veya daha yeni sürüme geçin; `kubernetes.resources.read` iznini ve servis hesabı RBAC kapsamını daraltın.
