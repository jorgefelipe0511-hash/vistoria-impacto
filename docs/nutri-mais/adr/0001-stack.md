# ADR-0001 — Stack tecnológica

- **Status:** Aceito (14/11/2026)
- **Contexto:** É preciso uma stack que mantenha as regras no servidor, isole os dados por organização no próprio banco e seja produtiva para uma equipe pequena apoiada por IA.
- **Decisão:** TypeScript em todas as camadas. Monorepo (pnpm + Turborepo). Next.js (App Router). PostgreSQL com Row Level Security. Drizzle ORM. Motor nutricional em pacote puro (`packages/nutrition-engine`). Autenticação própria com Argon2id, TOTP e sessões no servidor.
- **Alternativas consideradas:**
  - Laravel/PHP e Django/Python: maduros, mas exigem duas linguagens se o front-end for React.
  - Firebase: banco não relacional, com regras de acesso menos adequadas a dados clínicos relacionais.
- **Consequências:** Tipos compartilhados entre o motor, a API e a interface. Exige disciplina no RLS (testes obrigatórios no CI).
