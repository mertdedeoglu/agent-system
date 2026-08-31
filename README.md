# Agent System

Bu repo, yeni bir yazılım projesine baştan sağlam bir Claude Code kurulumu (subagent mimarisi, kod standardı, proje kuralları ve bilgi tabanı) kazandırmak için kullanılan Claude Code **skill**'lerini bir arada tutar.

Amaç: yeni bir proje başlarken burayı referans alıp içindeki skill'leri o projede çağırmak, böylece her seferinde sıfırdan mimari kurmak yerine denenmiş bir kurulumu tekrar kullanmak.

## İçindekiler

| Skill | Ne yapar |
|---|---|
| [`skills/create-agent-system`](skills/create-agent-system) | Projeye 8 parçalı bir Claude Code subagent sistemi kurar: **Planner → {Backend/Frontend Implementer} → {Backend/Frontend Reviewer} → Tester**, ayrıca duruma göre devreye giren **Debugger**, **Security Auditor** ve **Docs Writer**. Backend'de varsayılan olarak C# konvansiyonlarını, frontend'de tespit ettiği framework'ü (React/Angular/Vue) kullanır; ayrı bir frontend yoksa tek implementer/reviewer çiftine düşer. |
| [`skills/install-clean-code-rule`](skills/install-clean-code-rule) | `.claude/rules/clean-code-standards.md` dosyasını projeye kurar (isimlendirme, SOLID/DRY/KISS/YAGNI, hata yönetimi, format kuralları). Claude Code bu kuralı her oturumda otomatik okur. |
| [`skills/project-rules`](skills/project-rules) | Projeye özel, kendini geliştiren bir `project-rules.md` kurar/işletir: kullanıcıdan veya reviewer agent'tan gelen düzeltmeleri kalıcı kurala çevirip sonraki her kod görevinde uygular. |
| [`skills/llm-wiki-setup`](skills/llm-wiki-setup) | Andrej Karpathy'nin "LLM Wiki" desenini projeye kurar: `raw/` (ham kaynak), `wiki/` (Claude'un derlediği canlı proje bilgisi), `code/` üçlüsüyle, proje hakkındaki anlayışın sıfırdan tekrar üretilmek yerine kalıcı ve versiyonlanmış şekilde biriktirilmesini sağlar. Yalnızca açıkça istendiğinde kurulur.

## Nasıl kullanılır

1. Yeni projede Claude Code'u aç.
2. Bu repoyu klonla veya skill klasörlerini kopyala:
   ```bash
   git clone https://github.com/mertdedeoglu/agent-system.git
   ```
3. İstediğin skill klasörünü Claude Code'un skill dizinine ekle (ör. proje-yerel kullanım için `<proje>/.claude/skills/<skill-adı>` altına kopyala), ya da doğrudan bu repoyu işaret ederek Claude'a "şu skill'i kullan" de.
4. Claude Code'da ilgili skill'i tetikle — örneğin:
   - "Bu projeye agent sistemi kur" → `create-agent-system`
   - "Clean code kuralını bu projeye ekle" → `install-clean-code-rule`
   - "Proje kurallarını başlat" → `project-rules`
   - "Bu proje için LLM wiki kur" → `llm-wiki-setup`

Her skill kendi `SKILL.md` dosyasında ne zaman tetikleneceğini ve ne ürettiğini tanımlar; Claude Code bu açıklamaya göre otomatik olarak doğru skill'i seçer.

## Önerilen sıra

Skill'ler birbirinden bağımsız çalışır (bkz. Notlar), ama hepsini yeni bir projede kurmak istiyorsan şu sırayla gitmek en temiz sonucu verir — her skill kendinden öncekini otomatik tespit edip ona bağlanacak şekilde yazıldığı için, sıradaki her adım bir öncekini boşa düşürmez:

1. **`llm-wiki-setup`** (yeni proje ise) — `raw/`/`wiki/` temelini en başta kurar, böylece sonraki adımlar üstüne inşa edebilir.
2. **`install-clean-code-rule`** — global kod standardını `.claude/rules/` altına koyar.
3. **`project-rules`** — projeye özel, kendini geliştiren kural dosyasını başlatır.
4. **`create-agent-system`** — agent takımını kurar; implementer/reviewer agent'lar `clean-code-standards.md` ve `project-rules.md`'yi, docs-writer ise `wiki/`'yi varsa otomatik tespit edip kullanır — kurulum sırası ne olursa olsun çalışır, ama bu sıra ilk kurulumda hiçbir şeyin "sonradan bağlanmasını" beklemene gerek bırakmaz.

## Notlar

- Bu repo tek seferlik bir kurulum kaynağıdır — projeler değiştikçe skill'ler burada güncellenip diğer projelere tekrar taşınabilir.
- Skill'ler birbirinden bağımsızdır; bir projede hepsini birden kurmak zorunlu değildir.
