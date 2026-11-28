# Spotify Backstage Scaffolder Çalışma Alanı ve Konteyner İmajı Sınırları (`NEW-BS-05`)

[English](../52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | **Türkçe** | [日本語](../ja/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | [简体中文](../zh-CN/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md) | [한국어](../ko/52-SPOTIFY-BACKSTAGE-SCAFFOLDER-WORKSPACE-AND-CONTAINER-BOUNDARIES.md)

**VulKill bulgusu:** Spotify/Backstage denetiminde bulundu ve 2026-11-01 tarihinde gerçek üst akış Scaffolder modülleriyle izole yerel çalışma zamanı testlerinde gözlendi. Bu kayıt `NEW-BS-05` denetim kimliğini kullanır; bir üst akış güvenlik kaydı atamaz.

## 1. Kod Kusurlarının Güvenlik Sınırına Dönüşmesi

`fetch:rails` eylemi çıktı yolu için metinsel bir `startsWith` karşılaştırması kullanıyordu. Bu nedenle ortak ön eke sahip kardeş bir yol veya çalışma alanı içindeki sembolik bağ görev alanının dışına çözümlenebiliyordu. Cookiecutter eylemi ise çağıranın seçtiği `imageName` değerini eylem düzeyinde imaj izin listesi olmadan konteyner çalıştırıcısına iletiyordu. VulKill gerçek eylem işleyicilerini geçici çalışma alanları ve ağı kapalı yerel bir test konteyneriyle çalıştırdı.

Gözlenen kapsam, etkilenen Scaffolder eylemini çalıştırma izni ve ilgili girdiyi denetleme imkânı gerektirir. Test, canlı Spotify servisine veya dış konteyner kayıt merkezine erişmedi.

## 2. Gerçek Dünya Senaryoları

- **Senaryo A — Kardeş çalışma alanına yazma:** Üretilen proje, adı çalışma alanıyla aynı ön eki taşıyan komşu görev dizinine dosya yazar.
- **Senaryo B — Sembolik bağ kaçışı:** Metinsel olarak içeride olan yol, sembolik bağ üzerinden görev dizininin dışına yazılır.
- **Senaryo C — Onaysız üretici imajı:** Şablon, üretilen projeye beklenmeyen iş akışı veya bağımlılık dosyaları bırakan bir imaj seçer.

## 3. Çalışma Zamanı Gözlemi

```text
kontrol: normal alt yol geçici çalışma alanında kalır
RED:     kardeş-ön-ek ve sembolik bağ yolları çalışma alanı dışına sentetik işaretçi yazar
RED:     çağıranın seçtiği test imajı yerel konteyner çalıştırıcısına ulaşır
GREEN:   güvenli-alt-yol çözümü ve imaj izin listesi aynı girdileri reddeder
BENIGN:  onaylı imaj ve sıradan alt yol çalışmayı sürdürür
```

- Test edilen Backstage kaynak commit'i: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Harness SHA-256: `fa4dab592b61c8f8fcde81743694189b5d988cd6a9c7cbf29accc50c4bc097ae`
- Çalışma zamanı zinciri SHA-256: `0e2de393ee65136719495c2824cfe5133512ce570636174ee942fc208bbdc47a`

## 4. Savunma

Her hedefi platformun güvenli-alt-yol yardımcısı ve gerçek-yol denetimiyle sınırlandırın; sembolik bağ kaçışlarını reddedin; konteyner imajlarını eylem sınırında izin listesine alın ve hassas şablonları kimlerin oluşturup çalıştırabileceğini kısıtlayın.
