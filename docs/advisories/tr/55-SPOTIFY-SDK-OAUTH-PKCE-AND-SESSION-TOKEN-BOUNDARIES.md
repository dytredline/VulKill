# Spotify SDK OAuth, PKCE ve Oturum Token'ı Sınırları (`NEW-SDK-02`)

[English](../55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | **Türkçe** | [日本語](../ja/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | [简体中文](../zh-CN/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md) | [한국어](../ko/55-SPOTIFY-SDK-OAUTH-PKCE-AND-SESSION-TOKEN-BOUNDARIES.md)

**VulKill bulgusu:** Spotify'ın açık kaynak TypeScript, iOS ve Android SDK/örnek yüzeylerinde bulundu; izole yerel, emülatör ve fiziksel cihaz test ortamlarında gözlendi. Bu kayıt `NEW-SDK-02` denetim kimliğini kullanır; bir üst akış güvenlik kaydı atamaz.

## 1. Kod Kusurlarının OAuth ve Oturum Sınırına Dönüşmesi

Test edilen TypeScript SDK, PKCE verifier malzemesini `Math.random()` ile üretiyor ve verifier'ı ilk okumada siliyordu. Web Playback örneği en son OAuth token'ını süreç genelinde tek değişkende tutuyor ve ortak token rotasından sunuyordu. İncelenen mobil örnek yolları da callback veya deep-link token durumunu, başlatan oturuma tutarlı biçimde bağlamadan saklanan oturum verisini değiştirebiliyordu.

Bu gözlemler test edilen SDK/örnek commit'leri ve entegrasyon biçimleriyle ilgilidir. Uygulama etkisi, benimseyen tarafın callback rotalarını nasıl açtığına, tarayıcı oturumlarını nasıl ayırdığına ve örnek kodu nasıl kullandığına bağlıdır.

## 2. Gerçek Dünya Senaryoları

- **Senaryo A — Erken verifier silinmesi:** Callback biçimli istek, kullanıcının gerçek callback'i tamamlanmadan önbellekteki PKCE verifier'ı tüketir.
- **Senaryo B — Oturumlar arası token karışması:** İki kullanıcı aynı örnek sunucu sürecini paylaşır; sonraki giriş önceki oturuma dönen token'ı değiştirir.
- **Senaryo C — Mobil callback karışıklığı:** Deep link veya OAuth yanıtı, başlatan mobil oturuma bağlı `state` değeri olmadan kabul edilir.

## 3. Çalışma Zamanı Gözlemi

```text
RED:     callback biçimli istek başarılı değişimden önce önbellekteki verifier'ı tüketir
RED:     iki sentetik kullanıcı örnek sunucuda süreç genelinde token değişimini gözler
RED:     test edilen mobil callback yolları tam oturum bağı kurulmadan state/token verisini kabul eder
GREEN:   kriptografik rastgelelik, başarı sonrası silme, oturuma özel token saklama ve state denetimi durumları engeller
BENIGN:  doğru bağlanmış OAuth callback'leri tamamlanmayı sürdürür
```

- TypeScript SDK kaynak commit'i: `c6f34491eaa17ab73698d5464ee10332d0c551cc`
- iOS auth kaynak commit'i: `1e92eb3ddd632a3ab3b2ed852057948f8ef05067`
- Android auth kaynak commit'i: `eef9af9ffed5ed4e5ed63e2c162131c8da711bba`
- Web Playback örneği kaynak commit'i: `d22700174073dec831635aac452acbf5b4186853`
- Çalışma zamanı sonucu SHA-256: `1ac15486402c56af6c49cb5037a714db8cefd1e392c98ce9c28635ae5724aaea`
- Mobil bağlantı harness SHA-256: `0035b900685a91318a76117f59b476512b7e6b7f85e5b46098b08244415e60a0`
- Fiziksel cihaz sonucu SHA-256: `fe2a5ab870c226dc8f06ad9793b2bf774868092c36f35716ae8f36fd830a7a80`

## 4. Savunma

PKCE için kriptografik rastgele kaynak kullanın; verifier'ı başarılı ve `state` ile bağlı değişime kadar saklayın; token'ları her oturum için ayrı tutun; OAuth `state` değerini birebir denetleyin ve örnek uygulamaların üretim oturum kontrollerine yine ihtiyaç duyduğunu kabul edin.
