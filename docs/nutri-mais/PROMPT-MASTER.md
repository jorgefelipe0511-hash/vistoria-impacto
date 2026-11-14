# PROMPT MASTER — PLATAFORMA NUTRI MAIS · v2.0

> **Como usar este documento**
> Ele é a especificação-mestre e as instruções de trabalho para a IA ou equipe que vai desenvolver a Nutri Mais.
> Cole-o inteiro no início de uma sessão de desenvolvimento, ou mantenha-o no repositório e mande ler antes de cada tarefa.
> As seções **0 (Protocolo de trabalho)** e **2 (Princípios invioláveis)** valem para todo o resto e prevalecem em caso de conflito.

| Campo | Valor |
|---|---|
| Versão | 2.0 |
| Status | Pronto para iniciar a Fase 0 |
| Idioma do produto | Português (Brasil) |
| Público-alvo | Nutricionistas no Brasil e seus pacientes |
| Responsável pelas decisões de produto | Fundador(a) da Nutri Mais (chamado aqui de **Product Owner**, ou PO) |

---

## 0. PROTOCOLO DE TRABALHO (leia primeiro)

Você atua como uma equipe técnica sênior: arquitetura de software, engenharia full-stack, ciência de dados nutricionais, UX/UI, segurança da informação, LGPD e gestão de produto.

### 0.1 Regras de execução

1. **Trabalhe por fases** (seção 20). Não comece uma fase sem a aprovação do PO na anterior.
2. **Antes de implementar cada fase**, entregue um *Documento de Fase* com:
   arquitetura e componentes · modelo de dados (tabelas, relações, políticas de acesso) · regras de negócio aplicáveis (citando os IDs `RN-xx`) · fluxos de tela · riscos · plano de testes.
   **Pare e aguarde a aprovação.**
3. **Durante a implementação**, faça entregas pequenas e verificáveis (um módulo ou fluxo por vez), cada uma com testes.
4. **Ao final de cada fase**, entregue um *Relatório de Fase* com o que foi feito, como testar, os resultados dos testes, as pendências e as decisões tomadas. Depois pare.
5. **Ambiguidade:** quando a especificação não responder, **pergunte**. Não invente regra de negócio. Registre a pergunta em `docs/DECISOES.md` (seção 22).
6. **Nada fictício:** nenhum botão sem ação, nenhum dado nutricional inventado, nenhum cálculo simulado apresentado como real. Funcionalidade não implementada aparece desabilitada e com o rótulo **"Em breve"**.
7. **Registro de decisões:** toda decisão técnica relevante vira um ADR curto (Architecture Decision Record) em `docs/adr/` (contexto → decisão → alternativas → consequências).
8. **Não altere estes princípios ou regras por conta própria.** Proponha a mudança ao PO.

### 0.2 Definição de pronto (vale para qualquer funcionalidade)

Uma funcionalidade só está **pronta** quando tem:

- [ ] Regra de negócio implementada no **back-end** (o front-end nunca é a única barreira).
- [ ] Autorização verificada na API **e** no banco (RLS).
- [ ] Testes automatizados: unitários (regras e motor), de integração (API) e de isolamento entre tenants, quando acessa dados.
- [ ] Validação de entrada no servidor, com mensagens de erro sem dados sensíveis.
- [ ] Auditoria registrada, quando a ação é auditável (seção 17.6).
- [ ] Interface responsiva (testada em 360 px de largura) e acessível (seção 15.4).
- [ ] Documentação atualizada (README do módulo ou ADR).

---

## 1. VISÃO DO PRODUTO

**Nutri Mais** é uma plataforma SaaS que apoia nutricionistas na montagem de planos alimentares, no cálculo e na substituição de alimentos e no acompanhamento de pacientes. O paciente recebe uma experiência simples, clara e acolhedora.

> **Posicionamento:** "Nutri Mais — A inteligência que facilita a nutrição. Mais mobilidade para o profissional. Mais clareza para o paciente."

**Conceito:** o nutricionista define metas e parâmetros. A plataforma organiza alimentos, calcula quantidades, propõe combinações e substituições e facilita o acompanhamento. **A decisão clínica final é sempre do profissional.**

### 1.1 Objetivos mensuráveis

| # | Objetivo | Indicador de sucesso (meta inicial) |
|---|---|---|
| O1 | Reduzir o tempo de montagem de um plano | Plano completo de 6 refeições em ≤ 15 min |
| O2 | Substituições confiáveis | 100% das substituições mostram as diferenças por nutriente |
| O3 | Clareza para o paciente | Paciente encontra "o que comer agora" em ≤ 2 toques |
| O4 | Comunicação organizada | 100% das solicitações com status rastreável |
| O5 | Segurança | Zero vulnerabilidades críticas ou altas abertas em produção |
| O6 | Mobilidade | Todos os fluxos principais funcionam em celular |

### 1.2 Fora de escopo (versão 1)

Diagnóstico automático · prescrição automática sem revisão · prescrição de suplementos e fitoterápicos · videochamada própria · app nativo nas lojas (usaremos PWA) · integração com balanças e wearables · internacionalização.

---

## 2. PRINCÍPIOS INVIOLÁVEIS

| ID | Princípio |
|---|---|
| P1 | **Decisão clínica é humana.** Nenhuma sugestão vira prescrição sem a aprovação explícita do nutricionista. |
| P2 | **Nada altera o plano sozinho.** Solicitações, diário e mensagens do paciente nunca modificam a prescrição. |
| P3 | **Dados nutricionais têm fonte.** Todo valor tem origem rastreável. O que falta aparece como "não informado", nunca como zero. |
| P4 | **Cálculo é determinístico.** O mesmo pedido com os mesmos dados gera o mesmo resultado. A IA generativa **nunca** calcula nutrientes ou quantidades. |
| P5 | **Equivalência calórica ≠ equivalência nutricional.** Toda substituição exibe as diferenças. |
| P6 | **Duas linguagens.** Técnica para o profissional, simples e acolhedora para o paciente. |
| P7 | **Segurança e privacidade desde o projeto.** Dados de saúde são dados pessoais sensíveis (LGPD, art. 5º, II). |
| P8 | **Isolamento total entre tenants**, garantido na aplicação **e** no banco de dados. |
| P9 | **Sem julgamento.** Nenhuma mensagem automática avalia, culpa ou compara o corpo ou o comportamento do paciente. |

---

## 3. GLOSSÁRIO

| Termo | Definição |
|---|---|
| **Organização (tenant)** | Unidade de isolamento de dados. Pode ser um consultório individual ou uma clínica com vários profissionais. |
| **Nutricionista** | Profissional com CRN, membro de uma organização. |
| **Paciente** | Pessoa atendida. Vinculada a uma organização e a um ou mais nutricionistas dela. |
| **Plano alimentar** | Conjunto de refeições prescritas a um paciente, com versões. |
| **Versão do plano** | Fotografia imutável de um plano em determinado estado (seção 9). |
| **Refeição** | Bloco do plano (café da manhã, almoço…) com horário, itens e metas. |
| **Opção de refeição** | Alternativa aprovada para a mesma refeição (Opção 1, 2, 3). |
| **Item** | Alimento ou receita com quantidade, dentro de uma refeição. |
| **Substituição aprovada** | Troca autorizada pelo nutricionista para um item, já com a quantidade calculada. |
| **Medida caseira** | Unidade prática (colher, unidade, xícara) com peso em gramas **validado por alimento**. |
| **Meta** | Valor-alvo de energia ou nutriente (diário ou por refeição), com tolerância. |
| **Motor nutricional** | Módulo isolado e testado que faz todos os cálculos (seção 8). |

---

## 4. PERFIS E PERMISSÕES

### 4.1 Perfis

| Perfil | Escopo | MFA |
|---|---|---|
| **Super Admin (Nutri Mais)** | Toda a plataforma, **sem acesso a dados clínicos por padrão** | Obrigatório |
| **Dono da Organização** | Sua organização: membros, marca, plano comercial | Obrigatório |
| **Nutricionista** | Pacientes vinculados a ele dentro da organização | Opcional (recomendado) |
| **Assistente / Secretária** *(Fase 3+)* | Agenda e cadastro básico. **Sem dados clínicos** | Opcional |
| **Paciente** | Apenas os próprios dados publicados | Opcional |
| **Responsável legal** | Dados do paciente menor de idade sob sua responsabilidade | Opcional |

> Na versão 1, um nutricionista individual é **Dono da Organização** e **Nutricionista** ao mesmo tempo. A estrutura de organização existe desde o início para permitir clínicas sem refazer o banco.

### 4.2 Matriz de permissões (resumo)

| Recurso | Super Admin | Dono Org. | Nutricionista | Assistente | Paciente |
|---|---|---|---|---|---|
| Organizações e assinaturas | CRUD | Ler a própria | — | — | — |
| Banco oficial de alimentos | CRUD | Ler | Ler | — | — |
| Alimentos privados da organização | — | CRUD | CRUD | — | — |
| Pacientes (cadastro) | — | Ler | CRUD (vinculados) | Criar/Ler básico | Ler/editar contato próprio |
| Anamnese e avaliações | — | — | CRUD (vinculados) | — | Preencher pré-consulta |
| Planos alimentares | — | — | CRUD + publicar | — | Ler versão **publicada** |
| Diário alimentar | — | — | Ler | — | CRUD próprio |
| Solicitações | — | — | Ler/responder | — | Criar/ler próprias |
| Agenda | — | Ler | CRUD | CRUD | Ler as próprias |
| Marca (white label) | — | CRUD | Ler | — | — (vê aplicada) |
| Auditoria | Ler (plataforma) | Ler (organização) | Ler (seus atos) | — | — |
| Indicadores | Agregados e anonimizados | Da organização | Próprios | — | — |

### 4.3 Regras de acesso

- **RN-01** O Super Admin só acessa dados clínicos via **acesso de suporte temporário**: solicitado, aprovado pelo Dono da Organização, limitado a no máximo 24 h e totalmente auditado.
- **RN-02** O e-mail do Super Admin inicial é definido por variável de ambiente na implantação. O primeiro acesso exige criar senha **e** configurar MFA. Não existem senhas padrão.
- **RN-03** O paciente é cadastrado pelo nutricionista e recebe um **convite por e-mail** (link de uso único, válido por 72 h) para criar a senha.
- **RN-04** O paciente menor de 18 anos exige um responsável legal cadastrado, com consentimento específico (LGPD, art. 14). O acesso direto do menor é configurável pelo nutricionista.
- **RN-05** Ao encerrar o vínculo com a organização, o paciente perde o acesso ao portal, mas pode solicitar a exportação dos seus dados (portabilidade).

---

## 5. CADASTRO DE PACIENTES E ANAMNESE

### 5.1 Cadastro

Nome · nome social · data de nascimento · sexo biológico (para cálculos, quando aplicável) e gênero (opcional) · contatos · responsável legal (se menor) · objetivos declarados · rotina e horários · preferências · aversões · **alergias e intolerâncias (estruturadas, ligadas a componentes alimentares)** · restrições (vegetariano, vegano, religiosas etc.) · condições de saúde informadas · medicamentos informados · histórico alimentar · observações profissionais (**privadas**, nunca visíveis ao paciente).

**Campos mínimos**: colete apenas o necessário (LGPD, princípio da necessidade). Todo campo sensível tem finalidade documentada no mapeamento de dados.

### 5.2 Pré-consulta digital

- O nutricionista envia um formulário (link autenticado) que o paciente preenche antes da consulta.
- O nutricionista pode usar o modelo padrão ou criar modelos próprios.
- As respostas ficam organizadas em um **resumo estruturado** por blocos (rotina, preferências, restrições, sintomas relatados).
- **RN-06** O sistema **não** gera diagnóstico, classificação clínica ou alerta de risco automático a partir das respostas. Alergias declaradas, porém, alimentam automaticamente os filtros de segurança do motor (RN-20).

### 5.3 Avaliações

Registro datado de medidas antropométricas e outras avaliações que o nutricionista escolher (peso, estatura, circunferências, dobras, bioimpedância, exames informados), com **visibilidade por campo** para o paciente (seção 13).

---

## 6. BANCO DE ALIMENTOS

### 6.1 Estrutura de cada alimento

Nome · sinônimos · categoria e subcategoria · estado de preparo (cru, cozido, grelhado…) · **valores por 100 g**: energia (kcal e kJ), proteínas, carboidratos totais, fibras, lipídios totais, saturados, sódio, e outros micronutrientes disponíveis · medidas caseiras (cada uma com peso em gramas e fonte) · **fator de correção** (peso bruto ÷ peso líquido) · **fator de cocção** (peso cozido ÷ peso cru), quando disponíveis · alérgenos e componentes (glúten, lactose, leite, ovo, amendoim, castanhas, soja, frutos do mar…) · fonte · versão da fonte · data de atualização · **status de validação**.

### 6.2 Status de validação

`Validado` (fonte oficial, sem pendências) · `Parcial` (algum nutriente ausente) · `Pendente` (aguardando revisão) · `Privado` (cadastrado pela organização, responsabilidade dela).

### 6.3 Regras

- **RN-10** Nunca inventar ou estimar valores nutricionais. Nutriente ausente fica `null` e aparece como **"não informado"**. Ele **nunca** é somado como zero sem aviso: os totais mostram "valor parcial: X itens sem dado de fibra".
- **RN-11** Fontes iniciais: **TACO** (NEPA/UNICAMP) e **TBCA** (USP/FoRC). **Antes de importar, verificar e documentar em ADR a licença de cada fonte para uso comercial em SaaS.** Se uma fonte exigir autorização, não importar até obtê-la.
- **RN-12** O banco oficial é mantido pelo Super Admin, com importação por script versionado e reprodutível.
- **RN-13** Organizações podem cadastrar **alimentos privados** (ex.: produtos industrializados pelo rótulo), visíveis só para elas e marcados como responsabilidade do profissional.
- **RN-14** Alterações no banco oficial **não alteram planos já publicados**. Cada versão publicada guarda os valores usados no momento (snapshot).

---

## 7. ARQUITETURA E TECNOLOGIA

### 7.1 Stack de referência

> Esta é a proposta padrão. Mudanças exigem um ADR aprovado pelo PO.

| Camada | Tecnologia | Justificativa |
|---|---|---|
| Linguagem | **TypeScript** (front e back) | Tipagem compartilhada entre o motor, a API e a interface |
| Monorepo | pnpm workspaces + Turborepo | Separa o motor da interface com reaproveitamento |
| Web | **Next.js** (App Router) + React | SSR, PWA, ecossistema maduro |
| UI | Tailwind CSS + componentes acessíveis (Radix/shadcn) | Rapidez com acessibilidade |
| API | Rotas de servidor do Next.js **ou** NestJS, com camada de serviços isolada | Regras no servidor, testáveis sem HTTP |
| Banco | **PostgreSQL** com **Row Level Security** | Relacional, com isolamento garantido no próprio banco |
| ORM | Drizzle ORM + migrações versionadas | SQL explícito e compatível com RLS |
| Motor nutricional | Pacote puro `packages/nutrition-engine` | Sem I/O, 100% testável |
| Otimização | Solver de programação linear (ex.: HiGHS via WASM) | Cálculo de quantidades determinístico |
| Autenticação | Senhas com **Argon2id** · sessões no servidor com cookie `HttpOnly`/`Secure`/`SameSite` · TOTP para MFA | Padrões OWASP |
| Arquivos | Armazenamento S3-compatível, privado, criptografado, com URLs assinadas e de curta duração | Fotos, logotipos, PDFs |
| PDF | Renderização no servidor (HTML → PDF) | Fidelidade e marca do profissional |
| Filas e tarefas | Fila baseada em Postgres (ex.: pg-boss) | E-mails, PDFs, lembretes |
| E-mail | Provedor transacional (definir na Fase 0) | Convites, recuperação de senha, lembretes |
| Testes | Vitest (unidade) · Playwright (ponta a ponta) · testes de RLS no banco | Cobertura em todas as camadas |
| Observabilidade | Logs estruturados + rastreamento de erros + métricas | Sem dados sensíveis nos logs |
| Hospedagem | Provedor em nuvem com **região no Brasil**, preferencialmente | Latência e conformidade com a LGPD |

### 7.2 Estrutura sugerida

```
apps/web                 → interface (profissional, paciente, admin)
packages/nutrition-engine → cálculos puros (sem banco, sem HTTP)
packages/db              → schema, migrações, políticas RLS, seeds
packages/core            → serviços e regras de negócio (casos de uso)
packages/ui              → design system
docs/                    → especificação, ADRs, decisões, LGPD
```

### 7.3 Regras de arquitetura

- **RN-15** Regras nutricionais e de negócio ficam em `nutrition-engine` e `core`, **nunca só no front-end**.
- **RN-16** Toda tabela com dados de tenant tem a coluna `organization_id` e uma política RLS. A aplicação define o contexto do usuário por transação.
- **RN-17** Segredos ficam apenas em variáveis de ambiente ou num cofre de segredos. Nenhuma credencial no código ou no repositório (varredura automática no CI).
- **RN-18** Ambientes separados: desenvolvimento, homologação e produção, com bancos e chaves distintos. Dados reais **nunca** vão para desenvolvimento.
- **RN-19** IDs públicos não sequenciais (UUID). Nenhum dado pessoal em URLs.

---

## 8. MOTOR DE CÁLCULO NUTRICIONAL

Módulo independente da interface, determinístico, documentado e coberto por testes.

### 8.1 Funções obrigatórias

1. **Nutrientes de um item** = valores por 100 g × (quantidade em g ÷ 100), respeitando o estado de preparo.
2. **Totais** por refeição, por opção e por dia, com indicação de **dados parciais**.
3. **Metas**: diárias e por refeição (em valor absoluto, % do VET ou g/kg de peso), cada uma com **tolerância configurável** (padrão ±5% para energia e ±10% para macronutrientes).
4. **Comparação planejado × calculado**, com a diferença absoluta e percentual e um semáforo: dentro da tolerância / próximo / fora.
5. **Conversão de medidas**: gramas ↔ medida caseira, apenas com a medida validada daquele alimento.
6. **Conversão cru ↔ cozido** pelo fator de cocção, quando disponível. Sem fator, avisar.
7. **Arredondamento prático** (RN-23).
8. **Calculadoras de gasto energético (opcionais)**: Mifflin-St Jeor, Harris-Benedict e outras que o PO aprovar. A fórmula e as variáveis ficam visíveis e o resultado é **sugestão**, que o nutricionista aceita ou edita.

### 8.2 Regras

- **RN-20** **Filtro de segurança:** alimentos com alérgenos declarados pelo paciente **nunca** são sugeridos. Se o nutricionista adicionar um manualmente, aparece um alerta que exige confirmação explícita, registrada na auditoria.
- **RN-21** **Energia:** usar a kcal da fonte de dados. Mostrar ao lado, apenas como referência técnica, o valor por fatores de Atwater (4/4/9). Se divergirem mais que 10%, sinalizar.
- **RN-22** Precisão interna em decimal com pelo menos 4 casas. Arredondamento apenas na exibição.
- **RN-23** Quantidades sugeridas são arredondadas para valores práticos: gramas em múltiplos de 5 g (ou 1 g abaixo de 20 g), unidades em ½ e colheres em ½. **Os totais exibidos refletem a quantidade arredondada.**
- **RN-24** Toda função do motor tem testes unitários com casos de referência calculados à mão e documentados, incluindo casos-limite: quantidade zero, nutriente ausente, alimento sem medida caseira, divisão por zero.

---

## 9. CICLO DE VIDA DO PLANO ALIMENTAR

```
Rascunho ──► Publicado ──► Substituído (quando outra versão é publicada)
    ▲            │
    └── nova versão (cópia editável) ◄──┘                Arquivado (encerrado)
```

- **RN-30** O paciente vê **somente a versão publicada vigente**.
- **RN-31** Publicar exige ação explícita do nutricionista ("Revisar e publicar"), com um resumo das alterações em relação à versão anterior.
- **RN-32** Versões publicadas são **imutáveis**. Editar cria um novo rascunho.
- **RN-33** Cada publicação registra autor, data e hora, o diff e o snapshot dos valores nutricionais (RN-14).
- **RN-34** Rascunhos são salvos automaticamente.

---

## 10. NUTRI MAIS INTELIGÊNCIA (MONTAGEM ASSISTIDA)

### 10.1 Como funciona

1. O nutricionista escolhe uma refeição e define as metas (ou usa a distribuição do dia).
2. Opcionalmente, escolhe alimentos-base ("quero que tenha ovo e pão").
3. O motor **filtra** os candidatos (alergias, restrições, aversões, disponibilidade informada) e **calcula as quantidades** por otimização: minimiza o desvio das metas, respeitando os limites de porção, a diversidade e as preferências.
4. A plataforma apresenta as combinações com a quantidade de cada item, os totais, o desvio de cada meta e os nutrientes não informados.
5. **Nada entra no plano sem confirmação.**

### 10.2 Ações (todas funcionais)

| Botão | Ação |
|---|---|
| **Sugerir alimentos** | Gera combinações para a meta da refeição |
| **Substituir alimento** | Abre o módulo de substituição (seção 11) |
| **Adicionar alimento** | Busca no banco e calcula o impacto imediato |
| **Gerar outras opções** | Novas combinações, excluindo as já vistas |
| **Recalcular refeição** | Reajusta as quantidades dos itens atuais para a meta |
| **Confirmar refeição** | Leva para o rascunho do plano (não publica) |

### 10.3 Uso de IA generativa (opcional, a partir da Fase 2)

- **Permitido:** reescrever orientações em linguagem simples, sugerir nomes para refeições e receitas, resumir respostas da pré-consulta **para o profissional**.
- **Proibido:** calcular nutrientes ou quantidades, escolher alimentos sem passar pelo motor, gerar diagnósticos, enviar texto ao paciente sem revisão do profissional.
- Dados enviados a provedores de IA são **minimizados e pseudonimizados**. O provedor precisa ter contrato compatível com a LGPD e não pode treinar modelos com esses dados. Requer ADR e aprovação do PO.

---

## 11. SUBSTITUIÇÃO INTELIGENTE

### 11.1 Critérios disponíveis

Equivalência **energética** · **proteica** · **de carboidratos** · **de lipídios** · **combinada** (otimiza vários nutrientes com pesos ajustáveis).

Substituições entre categorias diferentes são permitidas (ex.: trocar carne por leguminosa).

### 11.2 Fluxo

1. Selecionar o item e o critério.
2. O motor calcula a quantidade do substituto (RN-23).
3. Tabela comparativa por nutriente: **original × substituto × diferença (g e %)**.
4. Recalcular a refeição e o dia inteiro, mostrando o impacto nas metas.
5. O nutricionista ajusta a quantidade, se quiser.
6. Aprovar como **substituição do item** (visível ao paciente) ou **troca definitiva** no rascunho.

### 11.3 Regras

- **RN-40** **Diferença relevante** = qualquer nutriente-chave (energia, proteína, carboidrato, lipídio, fibra) fora da tolerância configurada (padrão ±10%). Nesse caso a interface mostra **"Não equivalente em: …"** e **nunca** usa o rótulo "equivalente".
- **RN-41** Quantidade calculada fora de limites razoáveis (padrão > 3× a porção de referência) gera alerta.
- **RN-42** O paciente só vê substituições **aprovadas**, já com a quantidade em medida compreensível.

---

## 12. EDITOR DE PLANOS E GERADOR DE ALTERNATIVAS

### 12.1 Editor

Adicionar, excluir, duplicar e reordenar refeições (arrastar e soltar, com alternativa acessível por teclado) · alterar horários · refeições personalizadas · inserir alimentos e receitas · alterar quantidades (em g ou medida caseira) · substituir · orientações por refeição e gerais · **resumo nutricional fixo** (refeição e dia, planejado × meta) · salvamento automático · histórico de versões (seção 9).

### 12.2 Alternativas

- O nutricionista pede **N opções** (padrão 3) para uma refeição.
- Cada opção mostra os alimentos, as quantidades, a energia, as proteínas, os carboidratos, os lipídios, as fibras e o desvio das metas.
- É possível comparar lado a lado, editar, excluir e aprovar.
- As opções aprovadas aparecem ao paciente como **"Opção 1, Opção 2…"** da mesma refeição.

---

## 13. EXPERIÊNCIA DO PACIENTE

### 13.1 Portal (PWA instalável, com prioridade para celular)

Menu: **Hoje** (o que comer agora) · Minhas refeições · Minhas substituições · Minhas receitas · Lista de compras · Meu diário · Minha evolução · Minha experiência com a dieta · Falar com meu nutricionista.

### 13.2 Linguagem e apresentação

- Mostrar **alimento + quantidade em medida compreensível** ("2 colheres de sopa de arroz", com "≈ 50 g" em letra menor).
- **RN-50** Exibir **calorias e macronutrientes ao paciente é configurável por paciente** e vem **desligado por padrão**. Isso protege pacientes para quem números fazem mal (ex.: histórico de transtornos alimentares).
- **RN-51** Nenhuma mensagem automática de julgamento ("você comeu demais", "fora da dieta"). Use linguagem neutra e acolhedora.

### 13.3 Receitas

Nome · imagem · ingredientes (ligados ao banco de alimentos) · quantidades · modo de preparo · rendimento · porções · tempo · categoria. **A composição é calculada a partir dos ingredientes**, considerando o fator de cocção e sinalizando ingredientes sem dados. O nutricionista mantém uma biblioteca e compartilha receitas com pacientes.

### 13.4 Lista de compras

- Gerada a partir do plano publicado para um período escolhido (ex.: 7 dias).
- **RN-52** As quantidades são convertidas para **peso de compra** (cru e bruto, usando os fatores de cocção e de correção). Sem fator disponível, a lista avisa.
- Itens somados e agrupados por categoria, com opção de marcar como comprado (o estado é salvo por paciente).
- Opções de refeição: o paciente escolhe quais opções pretende seguir, ou a lista usa a Opção 1 por padrão.

### 13.5 Diário alimentar

- O paciente escolhe a refeição, registra os alimentos (busca no banco ou texto livre), a quantidade, observações, uma **foto opcional** e dificuldades.
- O nutricionista consulta o diário em uma linha do tempo.
- **RN-53** Sem pontuação, julgamento ou diagnóstico automático.
- **RN-54** Fotos: remoção de metadados (EXIF e localização), validação do tipo real do arquivo, limite de tamanho, verificação antimalware e armazenamento privado.

### 13.6 Minha experiência com a dieta (central de solicitações)

- **Motivos:** dificuldade com a refeição · não gosto deste alimento · não encontro este alimento · minha rotina mudou · dificuldade com as quantidades · sugerir substituição · preciso conversar · outro.
- **Campos:** motivo, refeição ou alimento relacionado (opcional) e mensagem.
- **Status:** `Recebida → Em análise → Aprovada | Não aprovada → Concluída`.
- O nutricionista recebe uma notificação, responde e, se quiser, abre um novo rascunho do plano a partir da solicitação.
- **RN-55** Nenhuma solicitação altera o plano automaticamente (P2).

### 13.7 Mensagens

- Mensagens **assíncronas** entre paciente e nutricionista, organizadas por conversa.
- O nutricionista configura um texto de horário de atendimento.
- **RN-56** O aviso fixo **"Este canal não é para emergências. Em caso de urgência, procure atendimento médico ou ligue 192."** aparece sempre na tela de mensagens.

### 13.8 Minha evolução

- Mostra **apenas os dados liberados pelo nutricionista, campo a campo**.
- Gráficos de evolução, metas compartilhadas, observações escritas para o paciente e histórico de consultas.
- **RN-57** Sem comparação com outras pessoas, sem rankings e sem imagens corporais comparativas automáticas. Textos automáticos neutros.

---

## 14. AGENDA, NOTIFICAÇÕES E DASHBOARD

### 14.1 Agenda

Consultas e retornos (presencial ou online, com link externo) · visualização por dia, semana e mês · registro do atendimento ligado ao prontuário · histórico · lembretes automáticos ao paciente (configuráveis) · lista de **"pacientes sem retorno agendado há mais de X dias"**.

### 14.2 Notificações

- **Versão 1:** e-mail transacional e notificações dentro da plataforma (e push via PWA, quando suportado).
- **Futuro:** WhatsApp, por provedor oficial e com opt-in.
- **RN-58** Notificações **não incluem dados clínicos** no conteúdo. Exemplo: "Você tem uma nova mensagem da sua nutricionista. Acesse a plataforma."

### 14.3 Dashboard do nutricionista

Total de pacientes e ativos · planos em rascunho · próximas consultas · solicitações pendentes · retornos programados · pacientes sem acompanhamento recente · atalhos ("Novo paciente", "Montar plano").

---

## 15. IDENTIDADE VISUAL, UX E ACESSIBILIDADE

### 15.1 Identidade Nutri Mais

Verde-escuro principal · verde suave secundário · base branca · detalhes dourados discretos · tipografia sem serifa moderna e legível · ícones simples de linha · cards organizados · gráficos objetivos. A paleta final (com códigos de cor e contraste validado) é entregue na Fase 0.

### 15.2 Duas linguagens

- **Profissional:** densidade técnica controlada (tabelas, metas, desvios, fontes de dados).
- **Paciente:** uma ação principal por tela, textos curtos, sem jargão, tom acolhedor.

### 15.3 Responsividade

Projeto **mobile-first**, testado em 360 px, 768 px e 1280 px ou mais. O editor de planos precisa ser **usável** no tablet e no celular, não só visível.

### 15.4 Acessibilidade

**WCAG 2.1 nível AA**: contraste mínimo de 4,5:1, navegação completa por teclado, rótulos para leitores de tela, foco visível, área de toque de pelo menos 44 px, nenhuma informação transmitida só por cor.

---

## 16. WHITE LABEL ("MINHA MARCA") E DOCUMENTOS

### 16.1 Personalização (por organização, isolada)

Nome profissional · nome do consultório · logotipo · foto · cor primária e secundária (com **verificação automática de contraste**) · CRN · contatos · frase de apresentação · rodapé dos documentos.

Exemplo: **"Michelle Souza Nutrição"**. O portal do paciente e os PDFs usam essa marca. A frase **"Tecnologia Nutri Mais"** aparece de forma discreta.

*Planos superiores (futuro): domínio próprio e e-mails com a marca do profissional.*

### 16.2 PDFs

- **Conteúdo:** identificação do profissional (com CRN) · identificação do paciente · plano (versão e data) · refeições e opções · quantidades em medida caseira e em gramas · orientações · substituições aprovadas · receitas selecionadas · lista de compras (opcional).
- Layout para impressão A4 e para leitura no celular.
- Calorias e macronutrientes aparecem no PDF do paciente conforme a configuração RN-50.
- *(Opcional, futuro)* Assinatura digital do profissional.
- **RN-59** O PDF é gerado no servidor a partir da versão publicada. O link de download é assinado e expira em pouco tempo.

---

## 17. SEGURANÇA DA INFORMAÇÃO (NUTRI MAIS SECURITY)

Referência: **OWASP ASVS 4.x, nível 2**, e **OWASP Top 10**.

### 17.1 Autenticação e sessão

- Argon2id para senhas. Política de senha forte com verificação contra senhas vazadas (k-anonymity).
- MFA (TOTP) obrigatório para Super Admin e Dono da Organização, e disponível para os demais. Códigos de recuperação de uso único.
- Recuperação de senha por link de uso único, válido por **30 min**, com resposta genérica (não revela se o e-mail existe).
- Limite de tentativas por conta e por IP, com bloqueio progressivo e CAPTCHA após falhas.
- Sessões no servidor: o identificador muda ao fazer login, há expiração por inatividade (profissional 30 min, paciente 7 dias com reautenticação para ações sensíveis), lista de sessões ativas e opção "encerrar todas".

### 17.2 Autorização e isolamento

- Verificação de permissão em **toda** rota e caso de uso (negar por padrão).
- RLS no PostgreSQL em todas as tabelas de tenant (RN-16).
- **Testes automatizados de isolamento** no CI: para cada recurso, o usuário da organização A tenta ler, editar e excluir dados da B e deve receber 404 ou 403. O mesmo vale entre pacientes da mesma organização e entre nutricionistas sem vínculo com o paciente.

### 17.3 Proteção de aplicação

Consultas parametrizadas (contra SQL injection) · escape de saída e CSP restritiva (contra XSS) · tokens anti-CSRF e cookies `SameSite` · rate limiting por rota · validação de schema em toda entrada · cabeçalhos de segurança (HSTS, CSP, X-Content-Type-Options, Referrer-Policy, Permissions-Policy) · uploads com validação de tipo real, limite de tamanho, verificação antimalware e armazenamento fora do webroot · WAF na borda · mensagens de erro genéricas ao usuário, com detalhes só no log interno.

### 17.4 Criptografia e segredos

TLS 1.2 ou superior em todas as conexões (HSTS) · criptografia em repouso no banco, nos arquivos e nos backups · **criptografia adicional por campo** para observações clínicas e anamnese · chaves em serviço gerenciado (KMS) com rotação · nenhum segredo no código (varredura no CI).

### 17.5 Infraestrutura

Banco sem acesso público, apenas por rede privada · princípio do menor privilégio nas credenciais de serviço · ambientes isolados (RN-18) · dependências atualizadas, com varredura automática (SCA) e análise estática (SAST) no CI · infraestrutura como código · monitoramento e alertas de disponibilidade, erros e eventos suspeitos.

### 17.6 Auditoria

- **Eventos registrados:** login (sucesso e falha) · logout · MFA · alteração de senha e de e-mail · alteração de permissões · acesso de suporte (RN-01) · criação, edição e publicação de planos · acesso a prontuário · exportações e downloads de PDF · operações administrativas · consentimentos.
- **Campos:** quem, quando, o quê, recurso (ID), IP e user-agent, resultado.
- **RN-60** Os logs **nunca** contêm senhas, tokens, conteúdo clínico ou dados pessoais desnecessários. Os registros de auditoria são apenas de inclusão (append-only) e ficam retidos pelo prazo definido na política de retenção.

### 17.7 Backup e recuperação

Backups automáticos diários, mais recuperação ponto a ponto (PITR) · criptografados · cópia em conta ou região separada · retenção definida (ex.: 35 dias) · **teste de restauração mensal, documentado** · metas iniciais de **RPO ≤ 24 h** e **RTO ≤ 8 h** · plano de continuidade documentado.

### 17.8 Testes de segurança

Testes automatizados de autorização e isolamento · SAST, SCA e varredura de segredos no CI · DAST em homologação · revisão de código focada em segurança · **pentest independente antes do lançamento público** · nenhuma vulnerabilidade crítica ou alta aberta ao entrar em produção.

---

## 18. LGPD E CONFORMIDADE PROFISSIONAL

### 18.1 Papéis

- **Organização (nutricionista ou clínica):** controladora dos dados dos pacientes.
- **Nutri Mais:** operadora desses dados, e controladora dos dados de cadastro e cobrança dos profissionais.
- Os papéis são formalizados nos Termos de Uso e no Contrato de Tratamento de Dados (DPA).

### 18.2 Entregáveis obrigatórios

Política de Privacidade · Termos de Uso (profissional e paciente) · DPA · mapeamento de dados e registro das operações (art. 37) · bases legais por finalidade (dados de saúde: art. 11, ex.: tutela da saúde por profissional da área ou consentimento específico) · gestão de consentimentos, quando aplicável · canal e fluxo para os direitos do titular (acesso, correção, portabilidade, eliminação) · política de retenção e eliminação · plano de resposta a incidentes (com comunicação à ANPD e aos titulares) · indicação do encarregado (DPO) · relatório de impacto (RIPD) sobre dados sensíveis.

### 18.3 Regras

- **RN-70** **Retenção × exclusão:** o pedido de exclusão é atendido respeitando o prazo legal ou regulamentar de guarda do prontuário. Durante esse prazo, os dados ficam bloqueados (sem uso) e depois são eliminados. **O prazo deve ser confirmado nas normas vigentes do CFN e por assessoria jurídica antes da Fase 3.**
- **RN-71** Exportação dos dados do paciente em formato legível (PDF) e estruturado (JSON).
- **RN-72** As funcionalidades de teleatendimento e prescrição devem estar alinhadas às resoluções vigentes do **Conselho Federal de Nutricionistas (CFN)**. Validar com especialista antes do lançamento.

---

## 19. MODELO COMERCIAL (SaaS)

- **Planos:** Inicial · Profissional · Premium. Os limites (pacientes ativos, membros da organização, armazenamento, white label avançado, IA generativa) ficam em **tabela configurável**, nunca fixos no código.
- **RN-80** "Paciente ativo" = paciente com acesso ao portal habilitado. Pacientes arquivados não contam no limite.
- **RN-81** Ao atingir o limite, novas criações são bloqueadas com um aviso claro. Os dados existentes **nunca** são bloqueados ou apagados por inadimplência sem aviso prévio e prazo.
- Assinatura, renovação, cancelamento, histórico financeiro e período de teste.
- **Pagamentos (Fase 4):** gateway brasileiro com Pix, cartão e boleto (escolha em ADR). A plataforma **nunca armazena dados de cartão**.
- A arquitetura prevê *feature flags* por plano desde a Fase 1.

---

## 20. FASES E ENTREGÁVEIS

### Fase 0 — Descoberta e fundação técnica

- Responder às decisões abertas (seção 22).
- ADRs da stack, de hospedagem e das licenças das tabelas de alimentos.
- Modelo de dados inicial e políticas RLS.
- Design system (paleta com contraste validado, tipografia, componentes base).
- Wireframes dos fluxos principais.
- Repositório, CI (lint, testes, SAST, SCA, varredura de segredos) e ambientes de desenvolvimento e homologação.

**Saída:** Documento de Fase 0 aprovado e esqueleto rodando com CI verde.

### Fase 1 — Fundação (MVP interno)

- Autenticação completa (login, MFA, recuperação de senha, sessões, limite de tentativas).
- Organizações, perfis e permissões, com RLS e testes de isolamento.
- **Admin mínimo:** criar e suspender organizações, gerenciar o banco oficial.
- Importação do banco de alimentos (fonte com licença confirmada).
- Cadastro de pacientes, convite ao paciente e anamnese.
- Auditoria básica.
- Base de LGPD (termos, aceite, registro de consentimento).

**Saída:** um nutricionista consegue entrar, cadastrar um paciente e consultar alimentos.

### Fase 2 — Inteligência nutricional

- Motor nutricional completo, com testes.
- Metas diárias e por refeição.
- Editor de planos e ciclo de vida (rascunho → publicado).
- Sugestões, substituições, alternativas e comparação.
- Alimentos privados.

**Saída:** plano completo montado, calculado e publicado, com testes de referência aprovados.

### Fase 3 — Experiência do paciente

- Portal PWA, receitas, lista de compras, diário, central de solicitações, mensagens, evolução.
- Agenda e lembretes.
- PDF com a marca do profissional e "Minha Marca" básica.
- Notificações por e-mail e na plataforma.
- Perfil Assistente.

**Saída:** um paciente real usa o portal no celular do início ao fim.

### Fase 4 — Gestão e comercialização

- Administração completa e indicadores agregados.
- Planos, limites, assinaturas, pagamentos e histórico financeiro.
- White label avançado e clínicas com vários profissionais.
- Relatórios.

**Saída:** cobrança funcionando em ambiente de testes do gateway.

### Fase 5 — Validação de segurança e lançamento

> Segurança é implementada **desde a Fase 1**. Esta fase é de **validação independente**.

- Auditoria técnica, DAST e revisão final.
- Pentest independente e correções.
- Teste de restauração de backup e simulação de incidente.
- Homologação com nutricionistas piloto.
- Liberação controlada (beta fechado → abertura gradual).

**Saída:** relatório de pentest sem itens críticos ou altos, restauração comprovada e aceite dos pilotos.

---

## 21. CRITÉRIOS DE ACEITAÇÃO

A plataforma **não** está concluída por ter telas bonitas. Cada item exige evidência (teste automatizado, relatório ou gravação).

| # | Critério | Evidência mínima |
|---|---|---|
| 1 | Login, MFA e recuperação de senha | Testes ponta a ponta dos fluxos de sucesso e de falha |
| 2 | Perfis separados | Testes da matriz de permissões (seção 4.2) |
| 3 | Isolamento entre tenants | Suíte de isolamento no CI, cobrindo 100% das rotas com dados |
| 4 | Pacientes e anamnese | Testes de integração e ponta a ponta |
| 5 | Banco de alimentos | Importação reprodutível, busca em menos de 300 ms (p95) e tratamento de dado ausente |
| 6 | Cálculos corretos | 100% das funções do motor testadas, tolerância de 0,01 em relação aos casos de referência |
| 7 | Substituições | Casos de referência por critério, com alerta de não equivalência testado |
| 8 | Editor e versões | Teste ponta a ponta: criar → editar → publicar → nova versão |
| 9 | Aprovação profissional | Teste comprovando que o paciente não vê rascunhos e que nenhuma solicitação altera o plano |
| 10 | Portal do paciente | Teste ponta a ponta em viewport de celular |
| 11 | Solicitações | Fluxo completo de status testado |
| 12 | Marca personalizada | Isolamento da marca entre organizações e contraste validado |
| 13 | PDFs | Snapshot visual e conteúdo conferido com o plano publicado |
| 14 | Segurança | SAST, SCA e DAST sem itens críticos ou altos e pentest aprovado |
| 15 | Backup | Relatório de restauração com tempo medido |
| 16 | Mobile | Lighthouse mobile ≥ 90 (desempenho e acessibilidade) nas telas principais |
| 17 | Acessibilidade | Verificação automática (axe) sem violações sérias e teste manual por teclado |

---

## 22. DECISÕES EM ABERTO (responder na Fase 0)

| # | Decisão | Opções ou observações |
|---|---|---|
| D1 | Confirmar a stack (seção 7.1) | Manter a proposta ou ajustar |
| D2 | Provedor de nuvem e região | Preferência por região no Brasil |
| D3 | Licenças TACO e TBCA para uso comercial | Consultar os termos e, se preciso, pedir autorização |
| D4 | Prazo de guarda do prontuário | Confirmar nas normas do CFN e com assessoria jurídica |
| D5 | Provedor de e-mail transacional | — |
| D6 | Uso de IA generativa na v1? | Sim (com as restrições da seção 10.3) ou deixar para depois |
| D7 | Gateway de pagamento | Decidir na Fase 4 |
| D8 | Limites e preços dos planos | Definir com o PO |
| D9 | Calculadoras de gasto energético incluídas | Lista de fórmulas aprovadas |
| D10 | Nome do domínio e e-mail do Super Admin inicial | Definido na implantação (RN-02) |

---

## 23. PRINCÍPIO CENTRAL

> **A plataforma deve facilitar a vida do nutricionista, simplificar a compreensão do paciente e proteger as informações de ambos.**
> A tecnologia trabalha para o profissional, e não cria mais trabalho para ele.

**NUTRI MAIS — A inteligência que facilita a nutrição.**
