# Arquitetura de Cibersegurança

**ID:** SEC-001 · **Rev.:** 1.0 · **Data:** 2026-09-12
**Base:** RDC 657/2022 Art. 11 (Cap. 3) e Art. 17, V · **Produto:** Aura Medical

> Cada controle abaixo foi **verificado no código ou no banco em 2026-09-12**, com arquivo:linha
> ou consulta. O §5 lista o que foi encontrado ABERTO e corrigido no mesmo dia — está aqui porque
> dossiê que só mostra o que deu certo não serve a uma inspeção.

---

## 1. Superfície e perímetro

| camada | onde | autenticação |
|---|---|---|
| App iOS | dispositivo do usuário | sessão Supabase (JWT) no Keychain, `WhenUnlockedThisDeviceOnly` |
| API (Hono/Node) | Render | JWT verificado contra o Supabase a cada requisição |
| Banco (Postgres) | Supabase | RLS por usuário + PostgREST com `anon`/`authenticated` |
| Cérebro (retrieval clínico) | VPS própria | HTTPS + autenticação básica; **sem PII** |

**Dois caminhos atendem na mesma origem** — a API e o PostgREST — e essa é a lição estrutural do
§5: proteger só a API deixa a outra porta aberta, e testes de aplicação não a alcançam.

## 2. Identidade e autorização

- **JWT verificado de verdade** contra o Supabase (`shared/auth.ts:20-26`), não decodificado.
- **Identidade sempre do token.** Varredura sistemática em 2026-09-11: **zero rotas** aceitam
  `user_id` vindo do cliente. É o controle que impede IDOR por parâmetro.
- **404 em vez de 403** para não-dono — não confirma existência de recurso alheio.
- **RLS** nas tabelas de paciente, por `auth.uid()`.
- **Grant por COLUNA** em `user_profiles`: `role`, `is_admin`, `kyc_status`, `deleted_at`,
  `research_id`, `is_active` e `stripe_customer_id` **não são graváveis** pelo cliente
  (migração `20260911_02`). A policy restringe a LINHA; o grant restringe a COLUNA.

## 3. Dados

- **PHI removida antes de sair do perímetro**: `scrubPhi` nos 4 caminhos até o cérebro.
- **Tokens de terceiros em repouso**: Garmin em **AES-GCM**.
- **Sem bucket de Storage** — nenhum PDF de exame em repouso.
- **Nenhum segredo versionado** em três repositórios (verificado).
- **Trilha imutável**: `ai_audit_log` protegido por trigger contra UPDATE (CFM 2.454/2026 Art. 9º).
- **Exclusão de conta**: ver `hipaa/ACCOUNT_DELETION.md`. ⚠️ Anomalia aberta — ANO-01.

## 4. Integridade da cadeia de fornecimento

- SBOM em `SBOM.md`; avaliação SOUP conforme IEC 62304 §5.3.3.
- Verificação de assinatura nos webhooks: **Stripe, Apple e Didit verificam de verdade**.
  ⚠️ O do **Garmin** autentica por `client_id` público — anomalia ANO-04.
- Notificações da App Store validadas por JWS contra o **Apple Root CA G3**
  (`services/apple/client.ts`), com o root parseado na construção do verificador — testado.

## 5. O que foi encontrado ABERTO e corrigido em 2026-09-11

Levantamento com 31 agentes e refutação adversarial por achado. **Quatro bloqueadores no
PostgREST**, nenhum alcançável pela API — e por isso invisíveis a 1.629 testes de aplicação:

| achado | correção | prova |
|---|---|---|
| `match_user_memories` era `SECURITY DEFINER` com o UUID-alvo vindo do chamador e `EXECUTE` para `authenticated` — **qualquer conta lia a memória clínica de qualquer paciente** (552 linhas reais) | `REVOKE EXECUTE` | migração `20260911_01`; `has_function_privilege('authenticated', …) = false` |
| 6 tabelas com RLS **desligada** e INSERT/UPDATE/DELETE para `anon` — incluindo a **trilha de assinatura do RT** | `ENABLE RLS` + `REVOKE` | idem; `relrowsecurity = true` nas 6 |
| Paciente gravava o próprio `kyc_status`/`role`/`is_admin` | `REVOKE UPDATE` da tabela + `GRANT` por coluna | migração `20260911_02` |
| `rpc_upsert_subscription` e `rpc_mark_consult_paid` chamáveis por `authenticated` — autoconcessão de assinatura e de consulta paga | `REVOKE EXECUTE` | migração `20260911_01` |

**Lição registrada:** a camada HTTP estava correta; o buraco inteiro estava na porta que corre em
**paralelo** a ela. Controle de aplicação não substitui controle de banco.

## 6. Controles de abuso e custo

- **Rate limit por rota e por usuário** — o `/chat` recebeu teto em 2026-09-11 (DIV-543); era a
  única rota cara sem nenhum.
- **Cota por saldo** com falha **FECHADA** (DIV-544/547): a contagem indisponível **recusa** o
  turno em vez de liberar. Falhar aberta liberaria gasto ilimitado no minuto do incidente.
- **Disjuntor agregado** de 30 dias (DIV-548), em três degraus, protegendo contra criação de
  contas em massa. Assinante nunca é afetado.

## 7. Anomalias de segurança em aberto

Rastreadas em `RESIDUAL_ANOMALIES.md`: ANO-01 (exclusão de conta), ANO-02 (App Attest gerado e
não verificado no servidor), ANO-04 (webhook Garmin sem assinatura), ANO-05 (texto do paciente
alcança observabilidade sem máscara).

## 8. Assinatura

| Papel | Nome | Assinatura | Data |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
