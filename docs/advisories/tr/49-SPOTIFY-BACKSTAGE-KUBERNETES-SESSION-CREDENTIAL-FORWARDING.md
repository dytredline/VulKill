# Spotify Backstage Kubernetes Proxy Oturum Kimliği İletimi (`GHSA-27gw-fw83-xfpp`)

[English](../49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | **Türkçe** | [日本語](../ja/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | [简体中文](../zh-CN/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md) | [한국어](../ko/49-SPOTIFY-BACKSTAGE-KUBERNETES-SESSION-CREDENTIAL-FORWARDING.md)

**VulKill bulgusu:** Spotify/Backstage denetiminde bulundu ve 2026-11-01 tarihinde yerel çalışma zamanı testlerinde gözlendi; üst akış duyurusu 2026-11-18 tarihinde yayımlandı.
**VulKill denetim kimliği:** `NEW-BS-01`.
**Üst akış:** [`GHSA-27gw-fw83-xfpp`](https://github.com/backstage/backstage/security/advisories/GHSA-27gw-fw83-xfpp) · CVSS 3.1 `6.4 Medium` · `CWE-200` / `CWE-441`.

## 1. Kod Kusurunun Güvenlik Açığına Dönüşmesi

Kubernetes proxy, seçilen küme kimlik doğrulama yöntemi bunları değiştirmediğinde çağıranın `Authorization` ve `Cookie` başlıklarını üst hedefe iletiyordu. Katalog katkıcısı küme API adresini etkileyebiliyorsa, bir kullanıcının Kubernetes görünümünü açması Backstage oturum bilgisini katkıcının kontrol ettiği adrese gönderebiliyordu. VulKill gerçek `CatalogClusterLocator -> KubernetesRouter` akışını çalıştırdı.

Etkilenen sürüm: `@backstage/plugin-kubernetes-backend >= 0.9.0, < 0.21.13`. Düzeltme: `0.21.13`; desteklenen backport'lar `0.21.14` ve `0.21.15`.

## 2. Günlük Hayat Senaryoları

- **Senaryo A:** Düşük yetkili katkıcı kontrollü bir küme varlığı ekler; yönetici görünümü açtığında oturum belirteci bu adrese gider.
- **Senaryo B:** Alınan kısa ömürlü kimlik belirteci başka Backstage eklentilerinde kurban adına kullanılır.
- **Senaryo C:** Cookie tabanlı kurulumda kurbanın oturum çerezi üst kümeye iletilir.

## 3. Çalışma Zamanı Gözlemi

```text
kontrol: operatör kontrollü küme yalnız yapılandırılmış küme kimliğini alır
RED:     katkıcı etkili küme çağıranın Authorization/Cookie bilgisini alır
GREEN:   proxy çağıran oturum bilgisini üst istekten çıkarır
BENIGN:  açık Backstage-Kubernetes-Authorization akışı çalışmaya devam eder
```

- Test edilen kaynak commit'i: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Runtime zinciri SHA-256: `a516e6c2e814083b8d697d9adf4b98c2ccc86ddb63b033de23b07ca0a089e3b7`
- Ortak runtime log SHA-256: `86cec273f9c154713aa8b9afb4283ad4b65422466adc12fff4710484512af0de`

## 4. Savunma

Yamalı sürüme geçin, katalog kümesi oluşturma yetkisini daraltın ve güvenilmeyen kullanıcılara `kubernetes.proxy` izni vermeyin.
