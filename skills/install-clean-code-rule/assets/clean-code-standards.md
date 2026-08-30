# Clean Code Standards

Bu projede kod yazarken, düzenlerken, refactor ederken veya review yaparken her zaman aşağıdaki kurallara uy. Dil fark etmeksizin geçerlidir (Python, JavaScript/TypeScript, Go, Java, C#, Rust, vb.).

## 1. Önce bağlamı oku

Kod yazmadan önce:
- Projede formatter/linter config'i var mı kontrol et (`.eslintrc`, `.prettierrc`, `pyproject.toml`/`ruff.toml`, `.editorconfig`, `rustfmt.toml`, `checkstyle.xml` vb.) — varsa kurallarına harfiyen uy.
- Mevcut isimlendirme stilini tespit et (camelCase mi snake_case mi, dosya adları nasıl, sınıf/fonksiyon önekleri var mı).
- Girinti, satır uzunluğu, tırnak stili, import sıralamasını mevcut kodla tutarlı tut.
- Hata yönetimi kalıplarını (custom exception sınıfları, logger vb.) projenin geri kalanıyla aynı şekilde uygula.

Bu projede henüz bir stil belirlenmemiş alan varsa, dilin resmi/endüstri standardı stil rehberini uygula:
- Python → PEP 8 + PEP 257
- JavaScript/TypeScript → Airbnb Style Guide veya Standard, TS için `strict` mod tercih edilir
- Go → Effective Go + `gofmt`/`golint`
- Java → Google Java Style Guide
- C# → Microsoft C# Coding Conventions
- Rust → `rustfmt` + Rust API Guidelines

**Projenin tutarlılığı her zaman önceliklidir** — proje bilinçli olarak farklı bir stil kullanıyorsa buradaki genel kurallar yerine projeye uy.

## 2. İsimlendirme

- İsimler ne olduğunu/ne yaptığını açıklamalı; kısaltma ve tek harfli değişkenlerden kaçın (döngü sayaçları `i, j` gibi dar kullanımlar hariç).
- Fonksiyon/metod isimleri fiil ile başlamalı (`calculateTotal`, `is_valid`, `fetchUser`).
- Boolean değişken/fonksiyonlar `is/has/can/should` önekleriyle okunur olmalı.
- `temp`, `data`, `obj`, `foo` gibi genel isimleri sadece gerçekten anlamsız placeholder gerektiğinde kullan.

## 3. Fonksiyon ve modül tasarımı

- Tek Sorumluluk İlkesi: bir fonksiyon tek bir işi yapmalı; birden fazla soyutlama seviyesinde iş yapıyorsa böl.
- Fonksiyonlar kısa tutulmalı (genelde ~30-40 satırı geçmemeli).
- Parametre sayısı fazlaysa (4-5+) bir yapı/obje/config nesnesine topla.
- Yan etkileri belirgin ve minimum tut; saf fonksiyonları tercih et.

## 4. SOLID / DRY / KISS / YAGNI

- SOLID ilkelerini nesne yönelimli kodda uygulanabilir olduğu ölçüde gözet.
- **DRY**: tekrar eden mantığı ortak bir fonksiyon/modüle çıkar, ama erken/aşırı soyutlama yapma.
- **KISS**: en basit doğru çözümü tercih et, over-engineering'den kaçın.
- **YAGNI**: şu an ihtiyaç olmayan "belki lazım olur" esnekliği ekleme.

## 5. Hata yönetimi

- Hataları sessizce yutma (`except: pass`, boş `catch` bloğu gibi); anlamlı şekilde işle, logla veya yeniden fırlat.
- Hata mesajları neyin, neden başarısız olduğunu açıkça belirtmeli.
- Beklenen hata durumlarını (validation, not found) beklenmeyen sistem hatalarından ayır.

## 6. Yorumlar

- Kod "ne yaptığını" kendi başına anlatmalı; yorumlar "neden" böyle yapıldığını açıklamalı.
- Kodun tekrarı olan yorumlardan kaçın.
- Karmaşık mantık, public API'ler ve edge-case'ler için kısa açıklama ekle.

## 7. Format ve tutarlılık

- Girinti, boşluk, satır sonu, tırnak stili proje genelinde tutarlı olmalı.
- Import/using ifadelerini mantıklı grupla (standart kütüphane / üçüncü parti / yerel modüller).
- Ölü kod (kullanılmayan değişken, import, yorum satırına alınmış eski kod) bırakma.

## 8. Teslim etmeden önce kontrol et

- [ ] İsimler açık mı, tahmin gerektirmiyor mu?
- [ ] Her fonksiyon tek bir şey mi yapıyor?
- [ ] Tekrar eden mantık ortaklaştırılabilir mi?
- [ ] Hata durumları sessizce yutulmuş mu?
- [ ] Yorumlar gerçekten değer katıyor mu?
- [ ] Proje stiliyle tutarlı mı?
- [ ] Gereksiz karmaşıklık/soyutlama var mı?
