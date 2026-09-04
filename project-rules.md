# Project Rules

Auto-maintained by the `project-rules` skill. Do not remove entries silently — if a rule seems outdated, ask the user before deleting it.

Bu repo, yeni projelere Claude Code kurulumu kazandıran **skill tanımlarını** barındırır (bkz. `skills/*/SKILL.md`); aşağıdaki seed rules bu skill dosyalarının mevcut yazım/yapı konvansiyonlarından çıkarılmıştır.

## Dil & Ton
- Kullanıcıya yönelik içerik (SKILL.md gövdesi, açıklamalar, adımlar) Türkçe yazılır; YAML frontmatter anahtarları (`name`, `description`), dosya/dizin yolları, kod ve teknik terimler İngilizce kalır. (2026-09-04)
- `description` alanı tetikleme örnekleri (kullanıcının söyleyebileceği ifadeler, tırnak içinde) ve skill'in ne zaman kullanılmaması gerektiğini içerir — sadece ne yaptığını değil, ne zaman/ne zaman değil tetikleneceğini de anlatır. (2026-09-04)

## Skill Dosya Yapısı
- Her skill kendi klasöründe `SKILL.md` olarak tutulur; YAML frontmatter'da en az `name` ve `description` bulunur. (2026-09-04)
- Gövde yapısı sabit bir şablon izler: başlık (`# Skill Adı`) → kısa özet paragrafı → numaralı `## Adımlar`/`## Setup steps` bölümü → kapanışta `## Notlar` bölümü. (2026-09-04)
- Şablon/asset dosyaları (kopyalanacak agent tanımları, kural dosyaları vb.) skill klasörü altında `assets/` içine konur, SKILL.md içine gömülmez. (2026-09-04)
- Skill'ler birbirinden bağımsız çalışacak şekilde yazılır: bir skill başka bir skill'in kurulum sırasına (önce/sonra) bağımlı olmamalı — "kurulum sırası ne olursa olsun çalışır" davranışı hedeflenir (bkz. `create-agent-system`, `install-clean-code-rule`). (2026-09-04)

## Dosya/İçerik Değişikliklerinde Davranış
- Var olan bir dosyanın (`CLAUDE.md`, `.claude/agents/*.md`, proje kuralları vb.) üzerine sessizce yazılmaz; zaten var olan referans/satır varsa dokunulmaz, yoksa sona eklenir, çakışma riski varsa kullanıcıya sorulur. (2026-09-04)
- Var olan `.claude/agents/*.md` gibi subagent dosyaları bulunursa, bunlar körü körüne değiştirilmeden önce okunur; ilgili referans zaten varsa atlanır, yoksa dosyanın stiline uyacak şekilde küçük, kendi içinde tutarlı bir blok eklenir. (2026-09-04)
- Yeni bir skill/kural kurulumunda önce hedef ortam taranır (kod örnekleri, mevcut config dosyaları), bulgular kullanıcıya kısaca özetlenir; tahmini/varsayımsal kurallar sert kural gibi sunulmaz. (2026-09-04)

## README / Doküman Güncellemeleri
- Kök `README.md`'deki skill tablosu ve önerilen kurulum sırası, yeni bir skill eklendiğinde veya davranışı değiştiğinde güncel tutulur. (2026-09-04)

---

## Karpathy Guidelines

<!-- Kaynak: Andrej Karpathy'nin LLM coding pitfalls üzerine gözlemlerinden türetilmiştir, Forrest Chang tarafından derlenmiştir. -->
<!-- SABİT KURALLAR — bu bölüm silinemez, üzerine yazılamaz veya diğer proje kurallarıyla gevşetilemez. Yeni seed/reviewer kuralları bu bölümün ÜSTÜNE değil, YUKARIDAKİ bölümlere eklenir. -->

- **Think Before Coding**: Varsayımları örtük bırakma, açıkça belirt. Birden fazla yorum mümkünse hepsini sun, sessizce birini seçip ilerleme. Belirsizlik varsa sor, tahmin etme. Daha basit bir yaklaşım varsa söyle.

- **Simplicity First**: Problemi çözecek minimum kodu yaz. İstenmeyen özellik ekleme. Tek kullanımlık kod için soyutlama üretme. 200 satır 50 satıra inebiliyorsa yeniden yaz.

- **Surgical Changes**: Sadece görevin gerektirdiği yere dokun. Komşu kodu veya formatlamayı "iyileştirme". Mevcut stile uy. İlgisiz ölü kod görürsen bahset, ama silme.

- **Goal-Driven Execution**: Başarı kriterini net tanımla, doğrulanana kadar döngüde kal. "Validation ekle" yerine "geçersiz girdiler için test yaz, sonra onları geçir" gibi test-önce, doğrulanabilir hedeflere dönüştür.
