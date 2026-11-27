# Spotify Backstage Oturum Açma Alan Adı ve Azure EasyAuth JWT Sınırları (`NEW-BS-06`)

[English](../53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | **Türkçe** | [日本語](../ja/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | [简体中文](../zh-CN/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md) | [한국어](../ko/53-SPOTIFY-BACKSTAGE-AUTH-DOMAIN-AND-EASYAUTH-JWT-BOUNDARIES.md)

**VulKill bulgusu:** Spotify/Backstage denetiminde bulundu ve 2026-11-01 tarihinde gerçek üst akış kimlik doğrulama ve ağ geçidi modüllerinde izole yerel çalışma zamanı testlerinde gözlendi. Bu kayıt `NEW-BS-06` denetim kimliğini kullanır; bir üst akış güvenlik kaydı atamaz.

## 1. Kod Kusurlarının Kimlik Doğrulama Sınırına Dönüşmesi

`emailMatchingUserEntityProfileEmail` çözücüsü seçenek şemasında `allowedDomains` kabul ediyor, fakat listeyi oturum açarken uygulamıyordu. Test edilen Azure EasyAuth yolunda doğrulayıcı, iletilen ID-token başlığını bu sınırda kriptografik imza, issuer ve audience denetimi yapmadan çözüyordu. VulKill gerçek çözücüyü, ağ geçidi yönlendirmesini, token fabrikasını, anahtar deposunu ve kimlik istemcisi yolunu sentetik kimlikler ile yerelde üretilmiş anahtarlarla çalıştırdı.

Alan adı davranışı etkilenen çözücüyü ve eşleşen bir katalog kullanıcısını gerektirir. EasyAuth davranışı, istemcinin sağladığı kimlik başlığının güvenilir proxy tarafından değiştirilmeden sağlayıcıya ulaşabildiği bir dağıtım yoluna bağlıdır.

## 2. Gerçek Dünya Senaryoları

- **Senaryo A — Alan adı politikası boşluğu:** Yapılandırma yalnız iç alan adlarını listelese de dış alan adlı katalog kimliği oturum açar.
- **Senaryo B — Güvenilir başlık sınırının kaybı:** Dağıtım, istemcinin sağladığı kimlik-token başlığını EasyAuth sağlayıcısına iletir.
- **Senaryo C — Mevcut kimliğe bürünme:** Kabul edilen başlıktaki iddialar ayrıcalıklı katalog kimliğine çözülür ve o kimlik için imzalı Backstage oturumu üretilir.

## 3. Çalışma Zamanı Gözlemi

```text
kontrol: kardeş çözücü dış alan adlı sentetik kimliği reddeder
RED:     etkilenen çözücü aynı kimliği kabul eder ve ES256 Backstage token'ı üretir
RED:     yerel ağ geçidi sentetik imzasız ID token'ını EasyAuth sağlayıcısına iletir
GREEN:   alan adı uygulaması ile imza/issuer/audience denetimleri iki durumu da reddeder
BENIGN:  iç alan adlı kimlik ve güvenilir proxy token'ı çalışmayı sürdürür
```

- Test edilen Backstage kaynak commit'i: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Alan adı çözücü harness SHA-256: `753692625e86b7007c02b5cf59e6d73c5ddc244cc6d1aeabe824480bafb2f2c8`
- EasyAuth/ağ geçidi harness SHA-256: `d3590c092c37f719926b4aec5de779ae6a612c8a45f9c057eb78bddc1aececca`
- Alan adı çalışma zamanı zinciri SHA-256: `a516e6c2e814083b8d697d9adf4b98c2ccc86ddb63b033de23b07ca0a089e3b7`
- EasyAuth çalışma zamanı zinciri SHA-256: `6778c3e716d8a3e1646dd98bbf94e5d55f16be37815058792f8873d8ea55228f`

## 4. Savunma

`allowedDomains` listesini çalışma zamanında uygulayın; EasyAuth JWT imzalarını sabit issuer ve audience ile denetleyin; herkese açık ağ geçidinde kimlik başlıklarını kaldırın ve yalnız belgelenmiş güvenilir ters proxy'den gelen başlıkları kabul edin.
