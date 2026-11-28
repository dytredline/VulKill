# Spotify Web API TypeScript SDK Yol Parametresi Sınırları (`NEW-SDK-01`)

[English](../54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | **Türkçe** | [日本語](../ja/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | [简体中文](../zh-CN/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md) | [한국어](../ko/54-SPOTIFY-WEB-API-TS-SDK-PATH-PARAMETER-BOUNDARIES.md)

**VulKill bulgusu:** Spotify açık kaynak SDK denetiminde bulundu ve 2026-11-01 tarihinde derlenmiş üst akış SDK'sıyla izole yerel HTTP çalışma zamanı testlerinde gözlendi. Bu kayıt `NEW-SDK-01` denetim kimliğini kullanır; bir üst akış güvenlik kaydı atamaz.

## 1. Kod Kusurunun İstek Sınırı Sorununa Dönüşmesi

On bir endpoint sınıfındaki otuz iki metot, çağıranın verdiği kaynak kimliklerini tek bir yol parçası olarak kodlamadan URL şablonlarına yerleştiriyordu. Uygulama saldırgan etkisindeki kimlikleri bu metotlara verdiğinde URL normalizasyonu ulaşılan endpoint'i değiştirebilir; SDK ise uygulamanın bearer token'ını eklemeye devam eder. VulKill 32 metodun tamamını yalnız loopback üzerindeki sentetik endpoint'lere karşı çalıştırdı; Spotify'ın canlı API'sine istek gönderilmedi.

Etki, tüketen uygulamanın saldırgan denetimindeki bir kimliği kabul etmesine ve token'ın taşıdığı kapsamlara bağlıdır.

## 2. Gerçek Dünya Senaryoları

- **Senaryo A — Backend rota karışıklığı:** API sarmalayıcı istekten playlist kimliği alır ve normalize edilen SDK yolu farklı Web API rotasına ulaşır.
- **Senaryo B — Durum değiştiren istek sapması:** Değişiklik yardımcısı bearer kimlikli isteği aynı token'ın izin verdiği başka endpoint'e gönderir.
- **Senaryo C — Veri okuma sapması:** Okuma yardımcısı uygulamanın amaçladığından farklı, fakat yetkili bir rotadan veri döndürür.

## 3. Çalışma Zamanı Gözlemi

```text
kontrol: sıradan base62 benzeri kimlikler amaçlanan sentetik rotalara ulaşır
RED:     32/32 metot saldırgan etkisindeki kimliği alternatif rotaya normalize eder
GREEN:   merkezi yol-parçası denetimi/kodlaması aynı girdileri engeller
BENIGN:  sıradan kimlikler savunma değişikliğinden sonra uyumlu kalır
```

- Test edilen SDK kaynak commit'i: `c6f34491eaa17ab73698d5464ee10332d0c551cc`
- Çalışma zamanı harness SHA-256: `3e65e293ad938d9bf526027e6bcb1371e6f2340d6a94adfb7d83d7b1400d6685`
- Çalışma zamanı sonucu SHA-256: `1ac15486402c56af6c49cb5037a714db8cefd1e392c98ce9c28635ae5724aaea`
- Matris: `32` metot (`10` durum değiştiren, `22` okuma)

## 4. Savunma

Her kaynak kimliğini ortak istek oluşturucuda tek bir yol parçası olarak kodlayın; URL oluşmadan önce traversal ve kodlanmış ayırıcı biçimlerini reddedin; güvenilmeyen isteklerden alınan kimlikler için tüketici tarafında izin listeleri kullanın.
