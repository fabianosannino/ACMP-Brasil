# CLAUDE.md — regras para o Claude Code neste repositório

Arquivo inicial, escrito a partir da estrutura do repositório. Cobre o que é o
projeto, os riscos de segurança da arquitetura escolhida e o fluxo de PR.
Convenções de código ainda não estão documentadas — ao aprender uma, registre-a
aqui.

## O que é

Site da ACMP Brasil. **Não é aplicação Node**: é site estático em HTML, CSS e
JavaScript puro, servido pela Vercel, com PWA (`sw.js`, `manifest.json`).
Não há `package.json`, bundler nem framework — o que está no repositório é o
que vai para o navegador.

## Estrutura

- `index.html`, `404.html` — páginas na raiz
- `pages/`, `blog/`, `jogos/` — seções do site
- `admin/`, `membros/` — áreas com acesso restrito
- `css/`, `js/`, `img/` — assets
- `supabase/` — configuração
- `*.sql` na raiz — 13 scripts (RLS, leaderboard, quiz, game-invites, storage)

## Dados e segurança — leia antes de mexer em qualquer coisa com Supabase

O Supabase é chamado **direto do navegador**. Três consequências que não são
negociáveis:

1. **A chave anon é pública por natureza.** Ela está no JS servido ao cliente, e
   isso é o desenho correto — não é vazamento e não precisa ser "corrigido".
2. **A `service_role` nunca pode entrar em arquivo servido ao browser.** Aqui não
   existe servidor onde escondê-la: se ela aparecer no repositório, está vazada.
   Não há meio-termo, e nenhum uso legítimo dela cabe neste projeto.
3. **A proteção real dos dados são as policies RLS**, não o código do cliente.
   Qualquer pessoa pode chamar a API do Supabase com a chave anon; o que ela
   alcança é exatamente o que a policy permitir. Validação em JavaScript é
   conveniência de UX, nunca barreira. Antes de criar tabela, escreva a policy.

Os `.sql` soltos na raiz (`supabase-rls-final.sql`, `supabase-fix-rls.sql` e os
demais) são o histórico dessas policies. Eles nunca foram auditados.

## CI

`.github/workflows/validate.yml` e `.github/workflows/scrape-jobs.yml`.
O repositório é público, então o Actions tem cota e o CI roda de verdade.

## Pull Requests

A `main` não tem branch protection e o auto-merge nativo não está armado — não
existe check obrigatório bloqueando merge.

- Fluxo padrão: abrir o PR, marcar como ready e **dar merge direto** (SQUASH).
  Não deixar PR parado esperando aprovação manual.
- Como o CI aqui roda de verdade, **esperar ficar verde antes do merge**.
- Nunca mergear com o `validate.yml` vermelho, mesmo sem gate no GitHub para
  impedir.
