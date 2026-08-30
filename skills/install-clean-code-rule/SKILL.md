---
name: install-clean-code-rule
description: >
  Kullanıcı bir projeye "clean code kuralı ekle", "code standard rule'unu kur",
  "clean-code-standards'ı bu projeye çevir/kur/uygula", ".claude/rules'a ekle"
  gibi bir istekte bulunduğunda kullan. Bu skill, kalıcı Clean Code Standards
  kuralını (isimlendirme, SOLID/DRY/KISS/YAGNI, hata yönetimi, format,
  proje stiline uyum) kullanıcının seçtiği/belirttiği proje dizininde
  `.claude/rules/clean-code-standards.md` dosyası olarak oluşturur ve
  bunu projenin `CLAUDE.md`'sine `@import` ederek gerçekten otomatik
  yüklenmesini sağlar; ayrıca varsa `.claude/agents/*.md` subagent
  tanımlarını da bu kuralı kontrol edecek şekilde yamalar.
---

# Install Clean Code Rule

Bu skill, bu skill klasörünün içindeki `assets/clean-code-standards.md` dosyasını, kullanıcının belirttiği proje dizinine `.claude/rules/clean-code-standards.md` olarak kopyalar.

## Adımlar

1. **Hedef proje dizinini belirle.**
   - Kullanıcı bir yol belirtmişse onu kullan.
   - Belirtmemişse ve konuşmada/iş dizininde tek bir proje bağlamı belliyse onu varsay, kısaca hangi dizini kullandığını söyle.
   - Hiçbir ipucu yoksa proje dizinini sor — bu tek gerekli netleştirme sorusudur.

2. **Hedef dizinin var olduğunu doğrula.** Yoksa kullanıcıya bildir, yanlış yazım olup olmadığını sorma (yeniden dene demek yeterli).
   - Not: bu skill için "mevcut proje mi yeni proje mi" ayrımı ve dil sorusu gerekmez — kurulan `clean-code-standards.md` dilden bağımsızdır ve her düzenlemede "önce bağlamı oku, proje varsa onun stiline uy, yoksa dilin resmi stil rehberini uygula" talimatını zaten içerir (bkz. `assets/clean-code-standards.md` §1). Yani hem mevcut hem yeni projede otomatik olarak doğru davranır; kurulum anında ekstra soru sormaya gerek yok.

3. **`.claude/rules/` klasörünü oluştur** (yoksa) hedef proje dizini altında.

4. **`assets/clean-code-standards.md` dosyasının içeriğini** hedef dizindeki `.claude/rules/clean-code-standards.md` yoluna kopyala.
   - Eğer o dosya zaten varsa, üzerine yazmadan önce kullanıcıya bildir ve onay al (mevcut özelleştirmeleri kaybetmemek için).

5. **Kuralı gerçekten otomatik hale getir — dosyayı yerleştirmek tek başına yeterli değil.** `.claude/rules/` Claude Code tarafından kendiliğinden taranan bir konum değildir; oturum başında otomatik okunan tek yer proje kök dizinindeki `CLAUDE.md`'dir. Bu yüzden:
   - Hedef proje kökünde `CLAUDE.md` var mı kontrol et.
     - Yoksa oluştur, içine tek satır ekle: `@.claude/rules/clean-code-standards.md`
     - Varsa ve bu satır (veya eşdeğeri) yoksa, dosyanın sonuna ekle. Zaten varsa hiçbir şey yapma.
   - Bu `@import` satırı Claude Code'un resmi memory-import söz dizimidir; böylece dosya gerçekten her oturumda otomatik yüklenir (yerleştirmiş olmak yetmez).

6. **Varsa mevcut subagent tanımlarını da yamala.** Hedef projede `.claude/agents/*.md` varsa (ör. `create-agent-system` skill'i ile kurulmuş bir sistem), her birine bak:
   - Dosya zaten `.claude/rules/clean-code-standards.md` referansı içeriyorsa dokunma.
   - İçermiyorsa, dosyanın başına (frontmatter'dan hemen sonra) şu talimatı ekle: `Before doing any work, check whether \`.claude/rules/clean-code-standards.md\` exists in the project root; if it does, read it and follow/check against it as the primary code-quality standard.`
   - Bu adım opsiyoneldir ama subagent'ların CLAUDE.md import'unu miras almadığı durumlarda (izole context) kuralın yine de uygulanmasını garanti eder.

7. **Sonucu kısaca özetle:** hangi dosyaların oluşturulduğu/güncellendiği (`clean-code-standards.md`, `CLAUDE.md`, varsa yamalanan agent dosyaları) ve artık kuralın hem ana oturumda hem subagent'larda otomatik uygulanacağı.

## Notlar

- Bu skill dosyayı yerleştirmekle kalmaz, `CLAUDE.md` üzerinden gerçek bir otomatik-yükleme bağlantısı kurar; sadece dosyayı bırakıp "otomatik okunur" varsaymaz.
- Kural içeriğini güncellemek istersen, önce `assets/clean-code-standards.md` dosyasını güncelle, sonra bu skill'i tekrar çalıştırarak projelere yeniden dağıt.
- Birden fazla proje için art arda çalıştırılabilir; her çağrıda tek bir hedef dizin kullanılır.
- `create-agent-system` skill'i ile kurulan agent şablonları zaten bu dosyayı kendiliğinden kontrol edecek şekilde yazılmıştır (kurulum sırası önemli değildir); adım 6, yalnızca bu repo dışında oluşturulmuş veya daha eski agent tanımları için bir güvenlik ağıdır.
