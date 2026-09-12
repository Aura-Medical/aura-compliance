# Exclusão de conta — como funciona de verdade

**Document ID:** AURA-POL-AD-001
**Version:** 2.0 · **Effective Date:** 2026-09-12 · **Supersedes:** 1.0 (2026-03-27)
**Last Reviewed:** 2026-09-12 · **Next Review:** 2027-03-12
**Owner:** Responsável Técnico, AURAMEDICAL SERVIÇOS MÉDICOS LTDA
**Classification:** Internal

---

## 0. ⚠️ Por que esta revisão existe, e o que a 1.0 afirmava de errado

A versão 1.0 descrevia um **desenho pretendido**, não o sistema construído. Ela foi escrita antes da
implementação e nunca foi conferida contra o banco. Isto está registrado aqui — e não apagado — porque
um documento de conformidade que descreve função inexistente é pior do que um documento ausente: ele
**produz confiança falsa** em quem audita.

Conferido no banco de produção em **2026-09-12**, item por item:

| a 1.0 afirmava | o que existe de fato |
|---|---|
| cron diário `hard_delete_expired_accounts()` apagando contas após 30 dias | **A função não existe** (`pg_proc` → 0 linhas). O `pg_cron` está instalado, com **um** job, que não é este. **Nunca houve purga.** |
| período de carência de 30 dias com recuperação por suporte | **Não existe.** Não havia o que purgar depois, então a conta "excluída" simplesmente **permanecia** |
| endpoint `DELETE /api/users/me` | **Nunca existiu.** Até 2026-09-12 o app chamava a RPC **direto** do cliente |
| `delete_my_account(target_user_id UUID)` | A função real **não tem parâmetro** — ela usa `auth.uid()` |
| reautenticação (Face ID / senha) antes de excluir | **Não acontece.** Há um diálogo de confirmação, sem segundo fator |
| apaga `daily_phenomic_scores`, `healthkit_summaries`, `push_tokens` | **Estas três tabelas não existem** no schema |
| diálogo de confirmação prometendo "mantidos por 30 dias caso você mude de ideia" | O texto real **não promete prazo** (ver §3.2) — e ainda bem, porque o prazo não existia |

O efeito prático do conjunto era a anomalia **ANO-01 / DIV-549**: a exclusão **desativava** em vez de
apagar, `auth.users` sobrevivia, e **o mesmo login voltava e recuperava o perfil**. Corrigido em
2026-09-12 (backend `eed5e99`, iOS `90c492f`). O que segue descreve o sistema **depois** da correção.

---

## 1. Objetivo e base normativa

- **Apple App Store Review Guidelines 5.1.1(v)** — app que permite criar conta deve permitir excluí-la
  no próprio app. O revisor testa **à mão**.
- **LGPD Art. 18, VI** — direito à eliminação dos dados tratados com consentimento.
- **LGPD Art. 16** — o que a lei obriga a guardar **não** é eliminado; é o limite do direito acima.
- **CFM 2.454/2026 Art. 9º** — a trilha de auditoria de IA tem de sobreviver.

---

## 2. A linha que governa o desenho

**APAGA** o que é dado do paciente. **ANONIMIZA**, apontando para o usuário sentinela
(`00000000-0000-0000-0000-000000000000`), o que é **trilha** que a norma exige preservar.

*Apagar trilha não é exclusão — é perder a prova de que o tratamento foi lícito enquanto durou.*

Por isso `user_consents` e `research_consent_records` **são anonimizados, não apagados**: destruir o
recibo de consentimento destruiria a prova da base legal do que já foi tratado. O que sobra é a versão
do texto e a data, sem ligação com pessoa.

---

## 3. O fluxo, ponta a ponta

### 3.1 Onde fica

`Ajustes` → seção **Conta** → **"Excluir minha conta"** (um nível de profundidade, ao lado de "Sair").
Botão destrutivo. Totalmente autosserviço — não requer contato com suporte.

### 3.2 Confirmação (texto real, pt-BR)

> **Excluir minha conta?**
>
> Esta ação é permanente. Seus dados não médicos serão excluídos e seus registros médicos serão
> anonimizados, conforme a legislação aplicável.
>
> [Excluir permanentemente] · [Cancelar]

**Não há reautenticação** (nem Face ID nem senha). É uma diferença consciente em relação à 1.0 e está
registrada como tal; a mitigação atual é o diálogo destrutivo explícito.

### 3.3 O app chama a rota

`AuthViewModel.deleteAccount()` → `AuraBackendClient.deleteMyAccount()` →
**`DELETE /api/aura-plus/me`** com JWT do Supabase.

**Por que a rota, e não a RPC direto:** `delete_my_account()` roda como o próprio usuário e **não tem
permissão sobre `auth.users`** — só a *service role* tem. Enquanto o app chamava a RPC direto, o dado ia
embora e a linha de login ficava.

### 3.4 O servidor, em duas metades, nesta ordem

1. **`supabase.rpc('delete_my_account')`** — apaga dado de paciente, anonimiza a trilha.
2. **`supabase.auth.admin.deleteUser(user.id)`** — apaga a linha de login.

**A ordem é obrigatória.** `ai_audit_log` tem FK **CASCADE** para `auth.users`: apagar o login antes de
anonimizar **apagaria a trilha do CFM junto, sem erro nenhum**. O caso 4 de
`me-delete-account.test.ts` reprova a inversão.

### 3.5 Respostas

| situação | resposta | o que o app faz |
|---|---|---|
| as duas metades passaram | **204**, corpo vazio | limpa estado local, encerra sessão, volta ao login |
| a RPC falhou | **500** `delete_failed` | **nada foi apagado**; alerta com "tente novamente" |
| a RPC passou e `auth.users` sobreviveu | **500** `delete_partial` | o dado **já foi**; alerta manda falar com o suporte — **não** oferece repetir |

Dizer 204 na meia exclusão seria o app afirmar uma exclusão que ficou pela metade. E até 2026-09-12 a
falha era **muda**: a tela descartava o resultado e nunca lia a mensagem de erro que já existia.

### 3.6 Limpeza local (após o 204)

`signOut()` apaga os modelos SwiftData (`LocalScore`, `LocalProfile`, `LocalBiometrics`), zera o
Keychain (`AuraKeychain.deleteAll()`), tranca o `BiometricAuthManager`, limpa o `HealthInsightsStore`
e **cancela as notificações locais agendadas** (senão a janela de exame desta conta dispararia no
aparelho depois que outra pessoa entrasse). Além disso, a exclusão limpa as chaves de `AppStorage` que
um logout comum **preserva de propósito** (`aura_plus_view_mode`, `wearableRealConnection` e outras).

**HealthKit não é apagado** — o dado é do app Saúde da Apple, não da Aura. O usuário o remove por lá.

---

## 4. O que é apagado e o que é anonimizado

### 4.1 Apagado (dado do paciente)

`aura_chat_message_feedback` · `aura_chat_messages` · `aura_chat_sessions` · `candidate_memory` ·
`forgotten_memories_log` · `memory_extraction_errors` · `user_memory` · `wearable_daily_summaries` ·
`wearable_sync_metadata` · `garmin_connections` · `health_snapshot_cache` · `device_tokens` ·
`scheduled_notifications` · `notification_preferences` · `aura_plus_free_quota` · `subscriptions` ·
`user_active_protocols` · `questionnaire_responses` (com `questionnaire_answers` em CASCADE) ·
`biomarker_results` · `user_biometrics` · `kyc_session_log` · `patient_doctor_relationships` ·
**`client_events`** · **`health_insights_cache`** · **`lab_intake_sessions`** · `doctor_profiles` ·
`budget_debito` · `budget_ledger` · **`user_profiles`** · `auth.sessions` · `auth.refresh_tokens` ·
e por fim **`auth.users`**, pela rota.

As três em negrito no meio ficavam para trás — a função estava **4 meses atrás do schema**.

### 4.2 Anonimizado (trilha, aponta para o sentinela)

| tabela | base legal |
|---|---|
| `ai_audit_log` | CFM 2.454/2026 Art. 9º — auditoria de IA |
| `user_consents`, `research_consent_records` | LGPD Art. 16 — prova da base legal do tratamento já ocorrido |
| `consultations`, `clinical_notes`, `prescriptions`, `exam_orders`, `patient_anamnesis`, `chat_sessions`, `kyc_verifications`, `clinical_assessments` | registro do ato médico prestado |
| `doctrine_changes_log`, `instrument_changes_log`, `clinical_flag_rules` | assinatura de curadoria clínica — quem assinou o quê precisa sobreviver a quem assinou |
| `data_access_log`, `data_export_requests`, `chat_messages` | LGPD Art. 37 — registro das operações |

### 4.3 Não há carência, e isso é intencional

**A exclusão é imediata e definitiva.** Não há período de 30 dias, não há recuperação, não há
restauração por suporte. A 1.0 prometia as três; nenhuma existia. Prometer carência exigiria construir
a purga diferida **e** a autenticação de identidade para restaurar — e a alternativa honesta é a que
está implementada: **apagar quando o usuário pede**.

Consequência que o usuário precisa saber, e que o texto de confirmação já diz: **a ação é permanente.**

---

## 5. Prova

Executada em **produção**, dentro de transação com `ROLLBACK`, sobre um usuário de teste criado para
isto, com contagem por tabela antes e depois. Resultado:

```
auth.users 0 · user_profiles 0 · user_memory 0 · client_events 0 ·
health_insights_cache 0 · lab_intake_sessions 0 · consentimento anonimizado 1
resíduo após o ROLLBACK: 0
```

A rota foi provada **pela internet**, no ambiente publicado:
`DELETE /api/aura-plus/me` sem token → **401**; rota inexistente de controle → **404**.

**Três defeitos só apareceram porque o caminho foi exercido de verdade** — nenhum deles seria visto por
leitura de código:

1. **29 FKs `NO ACTION`** apontam para `auth.users`; sete tabelas não cobertas bloqueariam o `DELETE`.
2. **`auth.refresh_tokens.user_id` é `varchar`, não `uuid`** (o GoTrue guarda assim) — sem `::text` a
   função **lança**.
3. A **"lápide"** de `user_profiles` (nulificar o PII e manter a linha) foi uma invenção da primeira
   tentativa de correção, e era **ela** quem fazia `DELETE FROM auth.users` falhar. Conferido em
   `pg_constraint`: **zero tabelas referenciam `user_profiles`**. A linha é apagada.

---

## 6. Controle contra a próxima defasagem

`src/health/aura-plus/__tests__/delete-account-drift.test.ts` cruza
`information_schema.columns` com o `prosrc` da função e **reprova** quando aparece tabela nova com
coluna de usuário que a função não menciona. Tabelas legitimamente fora do escopo ficam num mapa
`ISENTAS` **explícito**, com motivo — isenção sem motivo escrito é defasagem disfarçada.

Um segundo teste afirma o contrário do óbvio: que `ai_audit_log` e `user_consents` **continuam
existindo** depois da exclusão, anonimizados. Se alguém "melhorar" a função apagando-os, este teste cai.

Era a ausência deste controle que deixou a função envelhecer 4 meses sem ninguém notar.

---

## 7. Conformidade

### 7.1 Apple 5.1.1(v)

| requisito | implementação | estado |
|---|---|---|
| exclusão disponível no app | Ajustes → Conta → "Excluir minha conta" | ✅ |
| fácil de achar | um nível de profundidade | ✅ |
| apaga os dados associados | §4.1, incluindo `auth.users` | ✅ |
| funciona sem contatar o suporte | autosserviço completo | ✅ |
| cancelamento de assinatura | a linha de `subscriptions` é apagada; **a assinatura em si só o usuário cancela**, na App Store | ⚠️ ver nota |
| falha visível ao usuário | alerta com o motivo (corrigido em 2026-09-12) | ✅ |

> **Nota sobre assinatura.** A 1.0 registrava "Planned — StoreKit integration", como se fosse uma
> pendência de engenharia. Não é: **a Apple não expõe API que permita ao app cancelar a assinatura do
> usuário** — `AppStore.showManageSubscriptions` apenas *abre a tela dela*. O app tem esse caminho
> ("Ajustes → Gerenciar Assinatura"), mas **o diálogo de exclusão não avisa que a cobrança continua**
> se o usuário não cancelar por lá. Conferido em 2026-09-12: nenhuma copy do app diz isso.
> **Lacuna aberta — DIV-555**, e é a que mais provavelmente vira reclamação de cobrança.

### 7.2 LGPD

| requisito | artigo | implementação |
|---|---|---|
| direito à eliminação | Art. 18, VI | exclusão imediata e definitiva (§4) |
| prazo de resposta | Art. 18, §5º | imediato — supera o exigido |
| retenção só com base legal | Art. 16 | §4.2, tabela por tabela |
| informar a consequência | Art. 18, §1º | o diálogo diz que é permanente |
| revogação de consentimento | Art. 18, IX | a exclusão revoga; o **recibo** do consentimento passado é preservado anonimizado |

### 7.3 CFM / SaMD

| requisito | implementação |
|---|---|
| trilha de auditoria de IA preservada (2.454/2026 Art. 9º) | `ai_audit_log` anonimizado, **nunca apagado**; a ordem das duas metades existe para garantir isto |
| registro do ato médico | anonimizado, preservado (§4.2) |

---

## 8. Casos de teste

| caso | esperado | onde |
|---|---|---|
| sem `Authorization` | 401 | `me-delete-account.test.ts` #1 |
| as duas metades passam | 204, corpo vazio | #2 |
| `auth.users` sobrevive | 500 `delete_partial`, nunca 204 | #3 |
| a RPC vem **antes** de `deleteUser` | ordem exata | #4 |
| RPC falha → `auth.users` **não é tocado** | 500 `delete_failed`, zero chamadas | #5 |
| tabela nova com `user_id` não coberta | suíte reprova | `delete-account-drift.test.ts` |
| `ai_audit_log` / `user_consents` sobrevivem anonimizados | presentes após a exclusão | idem |
| exclusão de ponta a ponta no banco real | resíduo 0 (§5) | prova manual com `ROLLBACK` |
| **prova por cabo no aparelho** | conta some, app volta ao login | ⏳ **pendente** — destrutiva por natureza: roda em **conta descartável**, nunca em conta real |

---

## 9. Erros e monitoração

| erro | resposta | o que o usuário vê |
|---|---|---|
| rede indisponível | — | "Não foi possível excluir sua conta. Tente novamente." |
| RPC falhou | 500 `delete_failed` | idem (nada foi apagado — repetir é seguro) |
| `auth.users` sobreviveu | 500 `delete_partial` | "Seus dados foram excluídos, mas o acesso à conta ainda não foi encerrado. Fale com o suporte antes de criar uma conta nova." |

O log do servidor registra `userId` e o erro, **sem PHI**. `delete_partial` é a única condição que
exige intervenção humana e deve ser tratada como incidente de dado: a conta ficou sem dado e com login.

---

## 10. Documentos relacionados

- [`../samd/RESIDUAL_ANOMALIES.md`](../samd/RESIDUAL_ANOMALIES.md) — ANO-01 (DIV-549), com a prova
- [`../samd/SECURITY_ARCH.md`](../samd/SECURITY_ARCH.md) §3
- [`DATA_RETENTION.md`](DATA_RETENTION.md) — períodos por tipo de dado
- [`BREACH_RESPONSE.md`](BREACH_RESPONSE.md)
- `supabase/migrations/20260912_01_delete_my_account_v3.sql` (+ rollback irmão, que **declara** que
  reintroduz a não-conformidade)
- `src/health/aura-plus/me-routes.ts` — a rota
- `AuraMedical/ViewModels/AuthViewModel.swift` · `AuraMedical/Views/Settings/SettingsView.swift`
- `Packages/AuraMedicalCore/Sources/AuraMedicalCore/Services/AuraBackendClient.swift`

---

## Histórico de revisões

| Versão | Data | Autor | Mudanças |
|---|---|---|---|
| 1.0 | 2026-03-27 | Engineering Lead | Versão inicial — **descrevia o desenho pretendido, não o construído** (§0) |
| 2.0 | 2026-09-12 | Responsável Técnico | Reescrita contra o sistema real, conferido no banco de produção. Registra o que a 1.0 afirmava de errado em vez de apagar. Documenta a correção da ANO-01 (DIV-549): a exclusão passa a apagar `auth.users`, as três tabelas que faltavam entram, `user_consents` muda de apagar para anonimizar, e a falha deixa de ser silenciosa no app. Remove a carência de 30 dias, a recuperação e o cron — **nenhum dos três existia** |
