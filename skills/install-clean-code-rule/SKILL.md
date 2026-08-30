---
name: install-clean-code-rule
description: >
  Kullanıcı bir projeye "clean code kuralı ekle", "code standard rule'unu kur",
  "clean-code-standards'ı bu projeye çevir/kur/uygula", ".claude/rules'a ekle"
  gibi bir istekte bulunduğunda kullan. Bu skill, kalıcı Clean Code Standards
  kuralını (isimlendirme, SOLID/DRY/KISS/YAGNI, hata yönetimi, format,
  proje stiline uyum) kullanıcının seçtiği/belirttiği proje dizininde
  otomatik olarak `.claude/rules/clean-code-standards.md` dosyası olarak
  oluşturur, böylece Claude Code o projede her oturumda bu kuralı otomatik
  okur — ayrıca bir import eklemeye gerek kalmaz.
---

# Install Clean Code Rule

Bu skill, bu skill klasörünün içindeki `assets/clean-code-standards.md` dosyasını, kullanıcının belirttiği proje dizinine `.claude/rules/clean-code-standards.md` olarak kopyalar.

## Adımlar

1. **Hedef proje dizinini belirle.**
   - Kullanıcı bir yol belirtmişse onu kullan.
   - Belirtmemişse ve konuşmada/iş dizininde tek bir proje bağlamı belliyse onu varsay, kısaca hangi dizini kullandığını söyle.
   - Hiçbir ipucu yoksa proje dizinini sor — bu tek gerekli netleştirme sorusudur.

2. **Hedef dizinin var olduğunu doğrula.** Yoksa kullanıcıya bildir, yanlış yazım olup olmadığını sorma (yeniden dene demek yeterli).

3. **`.claude/rules/` klasörünü oluştur** (yoksa) hedef proje dizini altında.

4. **`assets/clean-code-standards.md` dosyasının içeriğini** hedef dizindeki `.claude/rules/clean-code-standards.md` yoluna kopyala.
   - Eğer o dosya zaten varsa, üzerine yazmadan önce kullanıcıya bildir ve onay al (mevcut özelleştirmeleri kaybetmemek için).

5. **Sonucu kısaca özetle:** dosyanın tam yolu ve bunun ne anlama geldiği (Claude Code'un bu projede her oturumda kuralı otomatik okuyacağı — ekstra `@import` gerekmediği).

## Notlar

- Bu skill sadece dosyayı ilgili konuma yerleştirir; proje içeriğini veya kodunu değiştirmez.
- Kural içeriğini güncellemek istersen, önce `assets/clean-code-standards.md` dosyasını güncelle, sonra bu skill'i tekrar çalıştırarak projelere yeniden dağıt.
- Birden fazla proje için art arda çalıştırılabilir; her çağrıda tek bir hedef dizin kullanılır.
