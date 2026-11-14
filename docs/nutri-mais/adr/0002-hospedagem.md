# ADR-0002 — Hospedagem

- **Status:** Proposto (aguarda aprovação do PO)
- **Contexto:** Os dados de saúde são sensíveis. Queremos baixa latência no Brasil, baixo custo inicial e pouca operação.
- **Decisão proposta:**
  - Vercel Pro, com funções em São Paulo (`gru1`), para a aplicação.
  - Supabase Pro, em São Paulo (`sa-east-1`), usado **somente** como PostgreSQL e Storage, com a Data API/PostgREST desligada.
  - Projetos separados para homologação e produção.
- **Alternativas consideradas:** AWS ou Google Cloud gerenciados diretamente (mais controle, porém mais custo e operação no início). Neon como banco.
- **Consequências:** Início rápido e PostgreSQL padrão, o que permite migrar no futuro. O plano gratuito da Vercel não serve, porque não permite uso comercial.
