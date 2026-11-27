# Spotify Backstage GitHub URL Okuyucusunda Kodlanmış Gezinme (`GHSA-fp7g-2pw8-726f`)

[English](../50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | **Türkçe** | [日本語](../ja/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | [简体中文](../zh-CN/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md) | [한국어](../ko/50-SPOTIFY-BACKSTAGE-GITHUB-URL-READER-ENCODED-TRAVERSAL.md)

**VulKill bulgusu:** Spotify/Backstage denetiminde bulundu ve 2026-11-01 tarihinde yerel çalışma zamanı testlerinde gözlendi; üst akış duyurusu 2026-11-27 tarihinde yayımlandı.
**Üst akış:** [`GHSA-fp7g-2pw8-726f`](https://github.com/backstage/backstage/security/advisories/GHSA-fp7g-2pw8-726f) · CVSS 3.1 `7.7 High` · `CWE-22` / `CWE-200`.

## 1. Kod Kusurunun Güvenlik Açığına Dönüşmesi

GitHub URL okuyucusu çözülen dosya yolunu denetliyor, ancak depo sahibinin adında aynı çoklu çözme ve ayraç kontrollerini uygulamıyordu. Kodlanmış gezinme doğrulamayı geçtikten sonra GitHub API URL'si oluşturulurken kimlik bilgili okuma başka bir API yoluna yönlenebiliyordu. VulKill gerçek `GithubUrlReader`, `getGithubFileFetchUrl` ve `parseGitUrlSafe` yolunda isteğin kontrollü canary uç noktasına ulaştığını gözledi.

Etkilenen sürüm: `@backstage/integration >= 0.1.3, < 2.3.0`. Düzeltme: `2.3.0`.

## 2. Günlük Hayat Senaryoları

- **Senaryo A:** Şablon kullanıcısı ortak GitHub App belirtecinin okuyabildiği özel kaynak koduna ulaşır.
- **Senaryo B:** Özel iş akışı loglarında bulunan dağıtım anahtarları okunur.
- **Senaryo C:** Organizasyon çapındaki entegrasyon belirteci, saldırganın kendi GitHub izinlerinden daha geniş bir etki alanı oluşturur.

## 3. Çalışma Zamanı Gözlemi

```text
kontrol: normal dosya URL'si -> amaçlanan depo içerik uç noktası
RED:     kodlanmış owner bileşeni -> entegrasyon kimliğiyle farklı GitHub API yolu
GREEN:   çoklu çözme ve ayraç/gezinme kontrolü URL'yi reddeder
BENIGN:  normal owner/repository/ref/file URL'leri çalışır
```

- Test edilen kaynak commit'i: `ac2de0fcbea3edcb38da36116268aae3cce189c1`
- Harness SHA-256: `dcf9f78d66b543e438dfa311509a481d78955070dd882a15bb6445552d0fd2c3`
- Zincir kanıtı SHA-256: `0e2de393ee65136719495c2824cfe5133512ce570636174ee942fc208bbdc47a`

## 4. Savunma

`@backstage/integration 2.3.0` veya daha yeni sürüme geçin; ortak entegrasyon belirteçlerinin yetkilerini en aza indirin.
