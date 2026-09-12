# Lista de Anomalias Residuais

**ID:** AR-001 · **Rev.:** 1.0 · **Data:** 2026-09-12
**Base:** RDC 657/2022 Art. 11 (Cap. 3) — *"Lista de anomalias residuais (incluindo os erros e
defeitos conhecidos) não resolvidos com análise de risco"*

> **A fonte é um registro vivo, não uma lista feita para este documento.** O produto mantém
> `docs/dividas/REGISTRO.md` (555 itens: 399 abertos · 153 fechados · 3 com risco aceito), com ID
> único nunca reusado, severidade, histórico datado e prova de fechamento. Este documento é o
> **recorte clínico e de segurança** dele: os itens cujo pior caso alcança o usuário.
>
> Os outros 390+ são dívidas de engenharia, conteúdo e processo — reais, rastreadas, e **fora do
> escopo do Art. 11**, que pede anomalia do dispositivo, não backlog.
>
> ⚠️ **Correção de rastreabilidade, 2026-09-12.** Quando este documento foi escrito, a frase acima
> ainda não era verdade: as seis anomalias tinham nascido no levantamento de conformidade e **nunca
> haviam entrado no registro**. Uma inspeção que procurasse ANO-02 no ledger não o encontraria.
> As seis foram inscritas com ID próprio e o `ANO-NN` virou alias — a correspondência agora é
> verificável nos dois sentidos: **ANO-01 → DIV-549 · ANO-02 → DIV-550 · ANO-03 → DIV-551 ·
> ANO-04 → DIV-552 · ANO-05 → DIV-553 · ANO-06 → DIV-554**.

---

## Critério do recorte

Entra aqui o item que satisfaz as duas condições:
1. está **aberto** (não fechado, não risco-aceito), e
2. seu pior caso alcança **o usuário** — dado errado exibido, dado exposto, ou função de segurança que não cumpre o prometido.

Severidade e probabilidade seguem as escalas de `RISK_ANALYSIS.md` §2.

---

## Anomalias

### ANO-01 (DIV-549) — "Excluir minha conta" desativa, não apaga — **FECHADA 2026-09-12**
**Origem:** levantamento 2026-09-11 · **Severidade:** Moderada · **Status:** ✅ FECHADA 2026-09-12
`delete_my_account()` marcava `deleted_at` e encerrava sessões, mas **não apagava `auth.users`**, e
três tabelas com dado de paciente não eram tocadas (`client_events`, `health_insights_cache`,
`lab_intake_sessions`). O mesmo login retornava e recuperava o perfil.
**Impacto:** direito de eliminação (LGPD Art. 18, VI) não cumprido; o texto publicado afirmava o
contrário. **Não era risco clínico** — era risco de conformidade e de confiança.

**Correção aplicada** (backend `eed5e99` · iOS `90c492f`, no TestFlight como 2026091201):
- migração `20260912_01` (registrada `20260912000100`, com rollback irmão que **declara** que
  reintroduz a não-conformidade): apaga dado de paciente — inclusive as três tabelas que faltavam,
  mais `doctor_profiles`, `budget_debito`, `budget_ledger` — e **anonimiza** para o sentinela a
  trilha que a norma exige preservar (`ai_audit_log` CFM Art. 9º; `user_consents` e
  `research_consent_records` LGPD Art. 16; registro clínico; assinatura de curadoria).
  `user_consents` **mudou de apagar para anonimizar**: destruir o recibo de consentimento apagaria
  a prova da base legal do tratamento que já ocorreu.
- rota `DELETE /api/aura-plus/me` (service role): chama a RPC e **depois** apaga `auth.users`. A
  ordem é obrigatória — `ai_audit_log` tem FK CASCADE para `auth.users`, então inverter apagaria a
  trilha de auditoria em silêncio. O caso 4 de `me-delete-account.test.ts` reprova a inversão.
- meia exclusão é **declarada**, não silenciosa: se `auth.users` sobreviver, a rota responde
  `delete_partial` 500 e o app mostra texto próprio (não oferece "tente novamente", porque o dado
  já foi e repetir não desfaz).

**Três defeitos que só apareceram ao exercer o caminho de verdade** (o teste roda contra o banco
real dentro de transação com ROLLBACK): (a) 29 FKs `NO ACTION` apontam para `auth.users` e sete
tabelas não cobertas bloqueariam o DELETE; (b) `auth.refresh_tokens.user_id` é `varchar`, não
`uuid` — sem `::text` a função lança; (c) a "lápide" de `user_profiles` era invenção da primeira
versão da correção e era **ela** quem bloqueava a exclusão real (conferido em `pg_constraint`:
zero tabelas referenciam `user_profiles`).

**Prova:** sequência completa em produção, `BEGIN`/`ROLLBACK`, resíduo 0 — `auth.users` 0 ·
`user_profiles` 0 · `user_memory` 0 · `client_events` 0 · `health_insights_cache` 0 ·
`lab_intake_sessions` 0 · consentimento anonimizado 1. Rota provada pela internet:
`DELETE /api/aura-plus/me` sem token → **401**; rota inexistente de controle → **404**.

**Controle permanente:** `delete-account-drift.test.ts` cruza `information_schema.columns` com o
`prosrc` da função e reprova tabela nova com `user_id` não coberta (com mapa `ISENTAS` explícito).
Era a ausência dessa guarda que deixou a função envelhecer 4 meses atrás do schema.

**Pendência residual (não bloqueia o fechamento):** prova por cabo no aparelho. Ela é destrutiva por
natureza — roda em conta descartável, nunca na do RT nem na da 2ª paciente.

### ANO-02 (DIV-550) — App Attest é gerado e nunca verificado no servidor
**Severidade:** Moderada · **Status:** ABERTA
O app gera a chave de atestação; **não há verificação no backend** (busca por `appAttest` fora de
teste = 0). Comentários no código justificam o CORS permissivo *por causa* do App Attest.
**Impacto:** a guarda que o código declara ter não existe; requisição forjada fora do app não é
distinguida. **Mitigação:** verificar no servidor, ou remover a justificativa e endurecer o CORS —
o que não pode permanecer é a declaração sem o controle.

### ANO-03 (DIV-551) — 60,9% das associações assinadas sem grau de evidência
**Severidade:** Menor · **Status:** ABERTA
4.867 de 7.985 associações `rt_approved` não têm `evidence_grade`. Elas **têm** trecho verbatim e
assinatura do RT — o que falta é o grau da primária.
**Impacto:** o grau **não ordena o retrieval** (doutrina `CLAUDE.md` §"Grade NÃO ranqueia"), então
a ausência **não muda o que o usuário recebe**. Ela afeta o tom da geração e a completude do
dossiê. **Mitigação:** atribuição em lote com assinatura do RT, por fonte.

### ANO-04 (DIV-552) — Webhook do Garmin autenticado por identificador público
**Severidade:** Menor · **Status:** ABERTA
Dos quatro webhooks, três (Stripe, Apple, Didit) verificam assinatura; o do Garmin usa `client_id`,
que é público. **Impacto:** injeção de dado de wearable falso, que alimenta classificação exibida
ao usuário. **Probabilidade baixa** (exige conhecer o endpoint e o id). **Mitigação:** validar
assinatura, ou tratar o dado como não-confiável até reconciliação com a API do Garmin.

### ANO-05 (DIV-553) — Texto do paciente alcança a observabilidade sem máscara
**Severidade:** Moderada · **Status:** ABERTA
A instrumentação captura entrada e saída do modelo em 5 caminhos; a mensagem do paciente chega
verbatim ao servidor de observabilidade, que resolve para o mesmo IP do cérebro e **não está
declarado como operador** na política publicada.
**Impacto:** tratamento de dado sensível em local não declarado (LGPD Art. 11 e Art. 37).
O endpoint exige autenticação (medido: 401), então **não é exposição aberta**.
**Mitigação imediata (15 min):** `OPENINFERENCE_HIDE_INPUTS/OUTPUTS` no ambiente — mantém métrica
e latência, remove o texto. **Mitigação durável:** declarar o operador na política.

### ANO-06 (DIV-554) — Zero fabricado servido como medida no eixo movimento — **FECHADA 2026-09-12**
**Severidade:** Moderada · **Status:** ✅ FECHADA 2026-09-12 (backend `86110dc`)
`snapshot.ts` somava colunas de intensidade que **só o Garmin preenche** (`?? 0` em cada uma); para
usuário de Apple Watch **toda** linha virava amostra de valor `0` — indistinguível de "mediu e deu
zero".
**Impacto:** era o modo de falha mais grave de um SaMD de classificação — **dado ausente
apresentado como dado medido**. O consumidor era a regra de sedentarismo do `opener.ts`, que marca
"sedentário sustentado" com mediana de 30 dias abaixo de 10 minutos: **quem se exercitava todo dia
era sinalizado como sedentário**. Mesmo padrão do HAZ-01 (`RISK_ANALYSIS.md`).

**Correção aplicada:** três pontos de fabricação em `snapshot.ts` — as amostras, o `latest_value` e
o markdown (que imprimia `"0mod"` para medida inexistente). Devolver `null` faz
`computeTimeTierStats` filtrar a amostra; abaixo de 3 amostras ela já devolve `median: null` +
`signal_strength: 'insuficiente'`, que é exatamente a condição que a regra do opener exige para
**não** disparar. **A regra se desarma sozinha** — o conserto não precisou tocá-la.

**A distinção que o conserto preserva:** medida PARCIAL continua contando (quem tem só `moderate`
tem uma medida) e **zero MEDIDO continua valendo zero**. O conserto separa ausência de zero; não
suprime sedentarismo real. Os dois casos têm teste. 4 testes novos; suíte verde.

---

## Anomalias FECHADAS em 2026-09-11 (registro, não pendência)

Quatro vulnerabilidades de acesso a dado, todas na porta PostgREST, corrigidas com prova —
detalhe em `SECURITY_ARCH.md` §5. Listadas aqui porque o Art. 11 pede o histórico de mudanças e
porque uma inspeção que encontre as migrações sem contexto merece a explicação.

## Assinatura

| Papel | Nome | Assinatura | Data |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
