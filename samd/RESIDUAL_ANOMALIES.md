# Lista de Anomalias Residuais

**ID:** AR-001 · **Rev.:** 1.0 · **Data:** 2026-09-12
**Base:** RDC 657/2022 Art. 11 (Cap. 3) — *"Lista de anomalias residuais (incluindo os erros e
defeitos conhecidos) não resolvidos com análise de risco"*

> **A fonte é um registro vivo, não uma lista feita para este documento.** O produto mantém
> `docs/dividas/REGISTRO.md` (540 itens: 394 abertos · 143 fechados · 3 com risco aceito), com ID
> único nunca reusado, severidade, histórico datado e prova de fechamento. Este documento é o
> **recorte clínico e de segurança** dele: os itens cujo pior caso alcança o usuário.
>
> Os outros 380+ são dívidas de engenharia, conteúdo e processo — reais, rastreadas, e **fora do
> escopo do Art. 11**, que pede anomalia do dispositivo, não backlog.

---

## Critério do recorte

Entra aqui o item que satisfaz as duas condições:
1. está **aberto** (não fechado, não risco-aceito), e
2. seu pior caso alcança **o usuário** — dado errado exibido, dado exposto, ou função de segurança que não cumpre o prometido.

Severidade e probabilidade seguem as escalas de `RISK_ANALYSIS.md` §2.

---

## Anomalias

### ANO-01 — "Excluir minha conta" desativa, não apaga
**Origem:** levantamento 2026-09-11 · **Severidade:** Moderada · **Status:** ABERTA
`delete_my_account()` marca `deleted_at` e encerra sessões, mas **não apaga `auth.users`**, e três
tabelas com dado de paciente não são tocadas (`client_events`, `health_insights_cache`,
`lab_intake_sessions`). O mesmo login retorna e recupera o perfil.
**Impacto:** direito de eliminação (LGPD Art. 18, VI) não cumprido; o texto publicado afirma o
contrário. **Não é risco clínico** — é risco de conformidade e de confiança.
**Controle atual:** nenhum. **Mitigação:** reescrita da função + rota autenticada com
`auth.admin.deleteUser` + teste de erasure contando linhas por tabela + guarda de drift que falha
quando aparecer tabela nova com `user_id` não coberta.
**Detectabilidade:** alta (o caminho nunca foi exercido: só existe o sentinela de 1970).

### ANO-02 — App Attest é gerado e nunca verificado no servidor
**Severidade:** Moderada · **Status:** ABERTA
O app gera a chave de atestação; **não há verificação no backend** (busca por `appAttest` fora de
teste = 0). Comentários no código justificam o CORS permissivo *por causa* do App Attest.
**Impacto:** a guarda que o código declara ter não existe; requisição forjada fora do app não é
distinguida. **Mitigação:** verificar no servidor, ou remover a justificativa e endurecer o CORS —
o que não pode permanecer é a declaração sem o controle.

### ANO-03 — 60,9% das associações assinadas sem grau de evidência
**Severidade:** Menor · **Status:** ABERTA
4.867 de 7.985 associações `rt_approved` não têm `evidence_grade`. Elas **têm** trecho verbatim e
assinatura do RT — o que falta é o grau da primária.
**Impacto:** o grau **não ordena o retrieval** (doutrina `CLAUDE.md` §"Grade NÃO ranqueia"), então
a ausência **não muda o que o usuário recebe**. Ela afeta o tom da geração e a completude do
dossiê. **Mitigação:** atribuição em lote com assinatura do RT, por fonte.

### ANO-04 — Webhook do Garmin autenticado por identificador público
**Severidade:** Menor · **Status:** ABERTA
Dos quatro webhooks, três (Stripe, Apple, Didit) verificam assinatura; o do Garmin usa `client_id`,
que é público. **Impacto:** injeção de dado de wearable falso, que alimenta classificação exibida
ao usuário. **Probabilidade baixa** (exige conhecer o endpoint e o id). **Mitigação:** validar
assinatura, ou tratar o dado como não-confiável até reconciliação com a API do Garmin.

### ANO-05 — Texto do paciente alcança a observabilidade sem máscara
**Severidade:** Moderada · **Status:** ABERTA
A instrumentação captura entrada e saída do modelo em 5 caminhos; a mensagem do paciente chega
verbatim ao servidor de observabilidade, que resolve para o mesmo IP do cérebro e **não está
declarado como operador** na política publicada.
**Impacto:** tratamento de dado sensível em local não declarado (LGPD Art. 11 e Art. 37).
O endpoint exige autenticação (medido: 401), então **não é exposição aberta**.
**Mitigação imediata (15 min):** `OPENINFERENCE_HIDE_INPUTS/OUTPUTS` no ambiente — mantém métrica
e latência, remove o texto. **Mitigação durável:** declarar o operador na política.

### ANO-06 — Zero fabricado servido como medida no eixo movimento
**Severidade:** Moderada · **Status:** ABERTA
`snapshot.ts` soma colunas de intensidade que **só o Garmin preenche**; para usuário de Apple
Watch o resultado é `0` — indistinguível de "mediu e deu zero".
**Impacto:** é o modo de falha mais grave de um SaMD de classificação: **dado ausente apresentado
como dado medido**, que pode gerar sinalização de sedentarismo em quem se exercita. É o mesmo
padrão do HAZ-01 (`RISK_ANALYSIS.md`).
**Mitigação:** distinguir `null` de `0` no caminho inteiro, e a interface dizer "sem medida".
**Prioridade: a mais alta desta lista** — é a única que produz afirmação falsa sobre o usuário.

---

## Anomalias FECHADAS em 2026-09-11 (registro, não pendência)

Quatro vulnerabilidades de acesso a dado, todas na porta PostgREST, corrigidas com prova —
detalhe em `SECURITY_ARCH.md` §5. Listadas aqui porque o Art. 11 pede o histórico de mudanças e
porque uma inspeção que encontre as migrações sem contexto merece a explicação.

## Assinatura

| Papel | Nome | Assinatura | Data |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
