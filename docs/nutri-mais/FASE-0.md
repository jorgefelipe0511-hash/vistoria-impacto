# Nutri Mais — Documento da Fase 0 (Descoberta e Fundação Técnica)

| Campo | Valor |
|---|---|
| Versão | 1.0 |
| Data | 14/11/2026 |
| Status | **Aguardando aprovação do PO** |
| Referência | `PROMPT-MASTER.md` v2.0 |

> Conforme o protocolo (seção 0), este documento descreve a arquitetura, o modelo de dados, os riscos e o plano de testes **antes** de escrever código.
> Nenhum código de produto será escrito até a aprovação.

---

## 1. Decisões registradas

| # | Decisão | Situação | Resultado |
|---|---|---|---|
| D1 | Stack | ✅ Decidida | TypeScript + Next.js + PostgreSQL com RLS (ADR-0001) |
| D2 | Hospedagem | 🟡 Recomendação abaixo, aguarda aprovação | Supabase (banco e arquivos, em São Paulo) + Vercel (aplicação, em São Paulo) (ADR-0002) |
| D3 | Licenças TACO/TBCA | 🟠 Pesquisada, requer contato | Ver seção 2 (ADR-0003) |
| D4 | Prazo de guarda do prontuário | ⏳ Pendente | Consultar normas do CFN e assessoria jurídica antes da Fase 3 |
| D5 | E-mail transacional | 🟡 Recomendação | Resend ou Amazon SES (decidir na Fase 1) |
| D6 | IA generativa na v1 | ✅ Decidida | **Não.** Fica para depois do núcleo validado |
| D7 | Gateway de pagamento | ⏳ Fase 4 | — |
| D8 | Limites e preços dos planos | ⏳ Fase 4 | Tabela configurável já prevista no banco |
| D9 | Calculadoras de gasto energético | 🟡 Proposta | Mifflin-St Jeor e Harris-Benedict (revisada), a confirmar |
| D10 | Domínio e e-mail do admin | ⏳ Antes da homologação | Definidos por variável de ambiente |
| — | Repositório | ✅ Decidida | Repositório novo e exclusivo para a Nutri Mais |

---

## 2. Licenças das tabelas de alimentos (D3)

### O que a pesquisa encontrou

**TACO (NEPA/UNICAMP)**
- As edições publicadas trazem: *"É permitida a reprodução total ou parcial desta obra, desde que citada a fonte."*
- **Não há cláusula explícita sobre uso comercial**, nem permitindo nem proibindo.
- Conclusão: o risco é baixo, mas **convém obter confirmação por escrito** do NEPA antes de usar num produto pago.

**TBCA (USP/FoRC)**
- A consulta no site é gratuita.
- Fontes secundárias indicam que **incorporar a base em outra ferramenta exige licença paga**.
- Conclusão: **não importar a TBCA** até haver um contrato de licença.

### Decisão proposta

1. **Fase 1** usa somente a **TACO**, com citação visível da fonte em cada alimento e na tela "Sobre os dados".
2. Enviar já os dois e-mails abaixo (NEPA e TBCA).
3. Até a resposta do NEPA, a TACO é usada **apenas em desenvolvimento e homologação**. A liberação para produção depende da confirmação.

### Rascunho de e-mail — NEPA/UNICAMP (taco@unicamp.br)

> **Assunto:** Solicitação de autorização — uso da Tabela TACO em plataforma para nutricionistas
>
> Prezados,
>
> Estamos desenvolvendo a Nutri Mais, uma plataforma digital por assinatura de apoio a nutricionistas na elaboração de planos alimentares.
>
> Gostaríamos de utilizar os dados da Tabela Brasileira de Composição de Alimentos (TACO, 4ª edição) como base nutricional da plataforma, com citação da fonte em cada alimento e na documentação.
>
> Solicitamos a confirmação de que esse uso, em produto comercial, está de acordo com as condições de reprodução da obra, ou a indicação dos procedimentos necessários para obter autorização.
>
> Atenciosamente,
> [Nome] — [Contato]

### Rascunho de e-mail — TBCA/USP

> **Assunto:** Licenciamento da TBCA para uso em plataforma digital
>
> Prezados,
>
> Estamos desenvolvendo a Nutri Mais, uma plataforma digital por assinatura voltada a nutricionistas.
>
> Temos interesse em licenciar a base de dados da TBCA para integração à plataforma.
>
> Poderiam informar as condições, modalidades e valores de licenciamento para esse tipo de uso?
>
> Atenciosamente,
> [Nome] — [Contato]

---

## 3. Hospedagem recomendada (D2)

| Componente | Serviço | Região | Por quê |
|---|---|---|---|
| Aplicação web e API | **Vercel** (plano Pro) | Funções em São Paulo (`gru1`) | Feito para Next.js, deploy automático, HTTPS e proteção de borda incluídos. *O plano gratuito não permite uso comercial.* |
| Banco PostgreSQL | **Supabase** (plano Pro) | São Paulo (`sa-east-1`) | PostgreSQL gerenciado com RLS, backups diários e PITR (recuperação ponto a ponto) opcional |
| Arquivos | Supabase Storage | São Paulo | Privado, criptografado, com URLs assinadas |
| E-mail | Resend ou Amazon SES | — | Convites, recuperação de senha, lembretes |
| Erros e monitoramento | Sentry | — | Com filtro de dados sensíveis |

**Custo inicial estimado:** cerca de US$ 50–80 por mês na fase de homologação e piloto. *Os preços devem ser confirmados nos sites oficiais no momento da contratação.*

**Cuidados obrigatórios no Supabase**
- **Desligar a API pública automática (Data API/PostgREST)**. A aplicação acessa o banco somente pelo servidor, o que atende à exigência de não expor o banco publicamente.
- Usar o **Supabase apenas como banco e armazenamento**. A autenticação é própria (Argon2id, MFA, sessões no servidor), conforme a especificação.
- Separar projetos para homologação e produção.

**Porta de saída:** como usamos PostgreSQL padrão, a migração futura para AWS ou Google Cloud não exige reescrever a aplicação.

---

## 4. Arquitetura

```
┌───────────────────────── Navegador / PWA ─────────────────────────┐
│  Área do Nutricionista   ·   Portal do Paciente   ·   Admin        │
└───────────────────────────────┬───────────────────────────────────┘
                                │ HTTPS (TLS, HSTS, CSP)
┌───────────────────────────────▼───────────────────────────────────┐
│ apps/web — Next.js                                                 │
│  • Páginas (React)            • Rotas de servidor (API)            │
│  • Middleware: sessão, CSRF, rate limit, cabeçalhos de segurança   │
└───────────────┬───────────────────────────────┬───────────────────┘
                │                               │
┌───────────────▼──────────────┐   ┌────────────▼──────────────────┐
│ packages/core                │   │ packages/nutrition-engine     │
│  Casos de uso e regras (RN)  │──▶│  Cálculos puros, sem banco    │
│  Autorização por ação        │   │  100% testado                  │
└───────────────┬──────────────┘   └───────────────────────────────┘
                │ transação com contexto do usuário (RLS)
┌───────────────▼──────────────┐   ┌───────────────────────────────┐
│ packages/db — PostgreSQL     │   │ Armazenamento de arquivos     │
│  Schema, migrações, RLS      │   │  Privado, URLs assinadas      │
└──────────────────────────────┘   └───────────────────────────────┘
```

### Camadas de proteção dos dados (defesa em profundidade)

1. **Middleware**: exige sessão válida e verifica o perfil para a área acessada.
2. **Caso de uso (`core`)**: verifica se o usuário pode executar *aquela ação naquele recurso*.
3. **Banco (RLS)**: mesmo se as camadas 1 e 2 falharem, o banco só devolve linhas da organização e do vínculo do usuário.

---

## 5. Modelo de dados — Fase 1

> Todas as tabelas têm `id uuid`, `created_at` e `updated_at`.
> 🔒 = tabela com `organization_id` e política RLS.

### 5.1 Identidade e acesso

| Tabela | Campos principais | Observações |
|---|---|---|
| `users` | email (único), password_hash (Argon2id), mfa_secret (criptografado), mfa_enabled, status, last_login_at | Uma conta por pessoa |
| `sessions` | user_id, token_hash, expires_at, last_seen_at, ip, user_agent, revoked_at | Só o hash do token é guardado |
| `auth_tokens` | user_id, type (convite, recuperação, verificação de e-mail), token_hash, expires_at, used_at | Uso único |
| `login_attempts` | email_hash, ip, success, created_at | Limite de tentativas |
| `platform_admins` | user_id | Super Admin, sem acesso clínico |
| `mfa_recovery_codes` | user_id, code_hash, used_at | — |

### 5.2 Organizações

| Tabela | Campos principais | Observações |
|---|---|---|
| `organizations` | name, slug, status (ativa, suspensa), plan_id | O tenant |
| `memberships` 🔒 | user_id, role (owner, nutritionist, assistant), crn, crn_region, status | Liga a pessoa à organização |
| `plans` | code, name, limits (JSON), features (JSON) | Limites configuráveis (RN-80) |
| `support_access_grants` 🔒 | admin_user_id, approved_by, reason, expires_at | Acesso temporário de suporte (RN-01) |

### 5.3 Pacientes

| Tabela | Campos principais | Observações |
|---|---|---|
| `patients` 🔒 | full_name, social_name, birth_date, sex, gender, phone, email, user_id (quando ativa o portal), status | Dados de cadastro |
| `patient_care_team` 🔒 | patient_id, membership_id, role (responsável, colaborador) | Quais nutricionistas atendem o paciente |
| `guardians` 🔒 | patient_id, user_id, name, relationship, consent_at | Responsável legal de menores (RN-04) |
| `patient_clinical_profiles` 🔒 | patient_id, goals, routine, preferences, aversions, restrictions, conditions, medications, food_history | **Criptografia adicional por campo** |
| `patient_allergies` 🔒 | patient_id, allergen_code, severity, notes | Estruturado, alimenta o filtro RN-20 |
| `professional_notes` 🔒 | patient_id, author_membership_id, content (criptografado) | Nunca visível ao paciente |
| `intake_forms` 🔒 | organization_id, name, schema (JSON) | Modelos de pré-consulta |
| `intake_responses` 🔒 | patient_id, form_id, answers (criptografado), submitted_at | — |
| `consents` 🔒 | subject_user_id, type, version, accepted_at, revoked_at | Registro LGPD |

### 5.4 Alimentos

| Tabela | Campos principais | Observações |
|---|---|---|
| `food_sources` | code (TACO), name, version, license_note, citation | Origem dos dados |
| `food_categories` | name, parent_id | — |
| `foods` | organization_id (nulo = oficial), source_id, source_code, name, synonyms, category_id, preparation_state, validation_status | 🔒 só para alimentos privados |
| `food_nutrients` | food_id, nutrient_code, amount_per_100g (`numeric`, pode ser nulo = "não informado") | RN-10 |
| `nutrients` | code, name, unit | Catálogo (energia, proteína, …) |
| `household_measures` | food_id, name, grams, source | Medida caseira por alimento |
| `food_factors` | food_id, correction_factor, cooking_factor, source | Fatores de correção e cocção |
| `food_allergens` | food_id, allergen_code | Ligação com RN-20 |

### 5.5 Auditoria

| Tabela | Campos principais | Observações |
|---|---|---|
| `audit_events` | actor_user_id, organization_id, action, resource_type, resource_id, ip, user_agent, result, created_at | **Apenas inclusão**: sem permissão de UPDATE ou DELETE para a aplicação. Sem conteúdo clínico (RN-60). |

*As tabelas de planos alimentares, agenda, diário e solicitações entram nos Documentos das Fases 2 e 3.*

### 5.6 Estratégia de RLS

- A aplicação conecta com um papel de banco **sem privilégios de superusuário e sem `BYPASSRLS`**.
- No início de cada transação, a aplicação define:
  `app.user_id`, `app.organization_id` e `app.role`.
- Exemplo de política (pacientes):
  - A linha precisa pertencer à organização atual **e**
  - o usuário precisa ser o dono da organização **ou** fazer parte da equipe de cuidado do paciente.
- O paciente acessa apenas linhas em que `patients.user_id = app.user_id`.
- O Super Admin **não** tem política de leitura clínica. O acesso de suporte só funciona com um registro válido em `support_access_grants`.
- Migrações e importação de alimentos usam um papel separado, que nunca é usado pela aplicação.

---

## 6. Design system (proposta)

### 6.1 Cores (contraste verificado com WCAG)

| Token | Claro | Uso | Contraste sobre branco |
|---|---|---|---|
| `--primary` | `#1F4D3A` verde-escuro | Botões, cabeçalho, links | 9,6 : 1 ✅ |
| `--primary-soft` | `#E8F3EC` verde suave | Fundos de cards e destaques | Texto primário sobre ele: 8,5 : 1 ✅ |
| `--ink` | `#1F2A24` | Texto principal | 14,8 : 1 ✅ |
| `--muted` | `#5B6B62` | Texto secundário | 5,6 : 1 ✅ |
| `--gold` | `#8A6A1F` dourado | Detalhes discretos, selos | 5,1 : 1 ✅ |
| `--danger` | `#B3261E` | Alertas e erros | 6,5 : 1 ✅ |
| `--bg` | `#FFFFFF` | Fundo | — |

**Modo escuro:** fundo `#0F1A15`, verde `#7FC8A0` (9,1 : 1) e dourado `#D9B45A` (9,0 : 1).

### 6.2 Tipografia e componentes

- **Fonte:** Inter (texto) e, opcionalmente, uma serifada elegante só nos títulos de marketing.
- Corpo de texto com no mínimo 16 px no portal do paciente.
- Componentes acessíveis (Radix), botões com área de toque de 44 px ou mais, foco visível.

### 6.3 A marca do profissional (white label)

As cores do nutricionista substituem `--primary` e `--primary-soft`. O sistema **calcula o contraste automaticamente** e ajusta a cor do texto ou bloqueia combinações ilegíveis.

---

## 7. Fluxos principais da Fase 1 (wireframes descritos)

1. **Primeiro acesso do Super Admin:** e-mail definido na implantação → link de ativação → criar senha → configurar MFA (QR code) → guardar códigos de recuperação.
2. **Criar organização (nutricionista):** o admin cria a organização e convida o dono → o nutricionista ativa a conta, cria senha e MFA → informa o CRN.
3. **Login:** e-mail e senha → código MFA (se ativo) → dashboard. Após 5 falhas: espera progressiva e CAPTCHA.
4. **Recuperar senha:** e-mail → mensagem genérica → link de 30 min, uso único → nova senha → **todas as sessões são encerradas**.
5. **Cadastrar paciente:** dados mínimos → (opcional) dados clínicos e alergias → "Convidar para o portal" envia um link de 72 h.
6. **Pré-consulta:** o nutricionista envia o formulário → o paciente responde pelo celular → o resumo aparece organizado no prontuário.
7. **Consultar alimentos:** busca com sinônimos e sem acentos → ficha com nutrientes por 100 g, medidas caseiras, fonte e status de validação, e "não informado" onde faltar dado.

---

## 8. Esteira de qualidade (CI)

A cada envio de código, executar automaticamente:

| Etapa | Ferramenta | Bloqueia o merge? |
|---|---|---|
| Formatação e lint | ESLint + Prettier | Sim |
| Tipos | `tsc` | Sim |
| Testes unitários (motor e regras) | Vitest | Sim |
| Testes de RLS e isolamento | Vitest + PostgreSQL em contêiner | Sim |
| Testes ponta a ponta | Playwright | Sim (fluxos críticos) |
| Dependências vulneráveis | `pnpm audit` / Dependabot | Sim, para críticas e altas |
| Análise estática de segurança | CodeQL | Sim, para críticas e altas |
| Segredos no código | Gitleaks | Sim |
| Acessibilidade | axe (no Playwright) | Sim, para violações sérias |

---

## 9. Riscos e mitigações

| Risco | Impacto | Mitigação |
|---|---|---|
| NEPA não autorizar o uso comercial da TACO | Alto | Pedir autorização já. Plano B: alimentos cadastrados pelo próprio nutricionista e licenciamento da TBCA |
| Vazamento entre organizações | Crítico | RLS + autorização em código + suíte de isolamento obrigatória no CI |
| Escopo grande para o MVP | Alto | Fases com saída clara. Nada fora do escopo da fase |
| Dependência de fornecedores (Vercel, Supabase) | Médio | PostgreSQL padrão e infraestrutura documentada, facilitando migrar |
| Prazo de guarda do prontuário indefinido | Médio | Exclusão lógica com bloqueio. Política definida antes da Fase 3 |
| Exposição acidental da API do Supabase | Alto | Data API desligada e verificação no checklist de implantação |

---

## 10. Plano de testes da Fase 1

- **Autenticação:** login correto e incorreto, bloqueio progressivo, MFA, recuperação (link expirado, reutilizado ou adulterado), troca de senha encerrando sessões.
- **Isolamento:** para cada rota com dados, a organização A tenta ler, editar e excluir dados da B (esperado: 404). Nutricionista fora da equipe de cuidado tenta acessar o paciente (esperado: 404). Paciente tenta acessar outro paciente (esperado: 404).
- **RLS direto no banco:** consultas sem contexto retornam zero linhas. O papel da aplicação não consegue alterar `audit_events`.
- **Alimentos:** a importação da TACO é reprodutível (mesmo arquivo → mesmo resultado). Nutriente ausente fica nulo, nunca zero.
- **Segurança:** cabeçalhos presentes, cookies com `HttpOnly`/`Secure`/`SameSite`, nenhum dado pessoal em URLs ou logs.

---

## 11. Próximos passos

### Para você (PO)

1. **Criar o repositório novo** no GitHub (ex.: `nutri-mais`, **privado**) e instalar nele o app do Claude.
2. **Aprovar ou ajustar** a hospedagem (seção 3) e o design system (seção 6).
3. **Enviar os e-mails** ao NEPA e à TBCA (seção 2), com seu nome e contato.
4. **Confirmar as calculadoras** de gasto energético (D9).

### Para a equipe técnica (após a aprovação)

1. Criar o esqueleto do monorepo, o CI e os ambientes.
2. Iniciar a Fase 1 pelo módulo de autenticação e pelo RLS.
