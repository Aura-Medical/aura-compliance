# Análise de Risco (ISO 14971 — FMEA)

**ID do Documento:** RA-001

**Revisão:** 3.0 (v1.0.0 Stable Release)

**Data:** 2026-04-10

**Produto:** Aura Medical iOS Application & Aura+ Backend

**Norma:** ISO 14971:2019 — Aplicação da gestão de risco a dispositivos médicos

**Método:** Análise de Modos de Falha e Efeitos (FMEA)

**Classificação:** SaMD Classe II (Anvisa RDC 657/2022), IMDRF Categoria II

---

## 1. Escopo

Esta análise de risco cobre todos os perigos identificados para o sistema Aura Medical (v1.0.0), incluindo avaliações binárias determinísticas, interações com LLM (IA Generativa), integridade de dados, segurança cibernética e proteção de informações de saúde protegidas (PHI). A análise segue a metodologia FMEA com Números de Prioridade de Risco (RPN) calculados como:

**RPN = Severidade × Probabilidade × Detecção**

---

## 2. Escalas de Avaliação

### 2.1 Severidade (S)

| **Nível** | **Classificação** | **Descrição** |
|---|---|---|
| 1 | Negligível | Nenhum impacto na saúde ou segurança do utilizador. |
| 2 | Menor | Inconveniência temporária; sem impacto clínico mensurável. |
| 3 | Moderado | Decisão clínica subótima; pode requerer intervenção médica de rotina. |
| 4 | Grave | Dano significativo à saúde; potencial hospitalização. |
| 5 | Catastrófico | Risco de vida; morte ou dano permanente irreversível. |

### 2.2 Probabilidade (P)

| **Nível** | **Classificação** | **Descrição** |
|---|---|---|
| 1 | Raro | < 1 em 10.000 utilizações. |
| 2 | Improvável | 1 em 1.000 a 10.000 utilizações. |
| 3 | Ocasional | 1 em 100 a 1.000 utilizações. |
| 4 | Provável | 1 em 10 a 100 utilizações. |
| 5 | Frequente | > 1 em 10 utilizações. |

### 2.3 Detecção (D)

| **Nível** | **Classificação** | **Descrição** |
|---|---|---|
| 1 | Quase certo | Controle automatizado detecta a falha antes de qualquer impacto. |
| 2 | Alta | Controle automatizado presente, com redundância limitada. |
| 3 | Moderada | Detecção dependente de revisão manual ou queixa do utilizador. |
| 4 | Baixa | Detecção apenas por auditoria periódica. |
| 5 | Impossível | Nenhum mecanismo de detecção implementado. |

---

## 3. Tabela de Análise de Perigos

### HAZ-01: Status Falsamente Saudável para Paciente Clinicamente Comprometido

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-01 |
| **Perigo** | O avaliador binário categoriza um domínio como "Saudável" apesar de o paciente apresentar uma condição subjacente grave. |
| **Causa** | Dados laboratoriais ausentes; utilizador não importou exames recentes; dados de wearable insuficientes para acionar a flag do domínio. |
| **Efeito** | Utilizador é falsamente tranquilizado e pode atrasar a procura por cuidados médicos necessários. |
| **Severidade** | 5 |
| **Probabilidade** | 3 |
| **Detecção** | 2 |
| **RPN** | **30** |
| **Mitigação** | (1) **Verificação de Itens Mínimos:** `DomainEvaluator` exige pelo menos 3 itens respondidos por domínio para gerar um status; caso contrário, retorna um prompt conservador solicitando mais dados (Ghost Mode). (2) **Data Caps:** Biomarcadores críticos (ex: ApoB > 90) acionam imediatamente a flag do domínio, independentemente de quaisquer outros inputs positivos. (3) **Indicador de Precisão:** Exibe a percentagem de completude dos dados (REQ-29). |
| **Risco Residual** | Baixo. A natureza determinística das flags binárias impede que um biomarcador crítico seja "diluído" por respostas positivas de estilo de vida. |
| **Rastreabilidade** | REQ-01, REQ-02, REQ-12, REQ-13, REQ-15, REQ-25, REQ-29, REQ-34 |

### HAZ-02: IA Generativa Alucina Diagnóstico Clínico ou Prescrição

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-02 |
| **Perigo** | O LLM da Aura+ fornece um diagnóstico médico, prescreve medicação por nome comercial ou dá aconselhamento de saúde perigoso. |
| **Causa** | Natureza probabilística inerente aos Large Language Models (LLMs); prompts adversariais do utilizador (jailbreak). |
| **Efeito** | Automedicação inadequada; responsabilidade legal por exercício ilegal da medicina (Ato Médico). |
| **Severidade** | 5 |
| **Probabilidade** | 3 |
| **Detecção** | 2 |
| **RPN** | **30** |
| **Mitigação** | (1) **Guardrails do System Prompt:** `system-prompt.ts` proíbe explícita e estritamente diagnosticar ou prescrever. (2) **Âncoras Estáticas:** Protocolos clínicos NÃO são gerados pela IA; são buscados de um `PROTOCOLS_CATALOG` codificado com IDs específicos. (3) **Disclaimer Obrigatório:** Disclaimer-mãe injetado automaticamente em cada sessão de chat. |
| **Risco Residual** | Baixo. A restrição da IA a emitir IDs de `protocolCard` pré-definidos elimina o risco de protocolos clínicos alucinados. |
| **Rastreabilidade** | REQ-04, REQ-33, REQ-36, REQ-37, REQ-38 |

### HAZ-03: Falha na Detecção de Crise Aguda (Falso Negativo)

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-03 |
| **Perigo** | Utilizador expressa ideação suicida ou sintomas físicos agudos no chat e o sistema responde conversacionalmente em vez de escalonar. |
| **Causa** | Utilizador emprega gíria regional não capturada pelo regex; LLM falha em interpretar a gravidade do input. |
| **Efeito** | Paciente em crise aguda não recebe instruções de emergência; potencial autolesão ou morte. |
| **Severidade** | 5 |
| **Probabilidade** | 2 |
| **Detecção** | 2 |
| **RPN** | **20** |
| **Mitigação** | (1) **Gate de Regex Determinístico:** `isCrisisInput` bypassa o LLM inteiramente ao detectar padrões, forçando um bloco de `escalation` com CVV 188 / SAMU 192. (2) **Flag de Segurança do Snapshot:** Se o item 9 do PHQ-9 (ideação) for >= 1, `has_active_safety_alert` é definido como verdadeiro por 90 dias, injetando uma restrição estrita de emergência diretamente no system prompt do LLM para TODOS os turnos subsequentes. |
| **Risco Residual** | Aceitável. A combinação de regex pré-LLM e injeção de prompt pós-LLM fornece uma rede de segurança altamente redundante. |
| **Rastreabilidade** | REQ-04, REQ-34 |

### HAZ-04: Execução de Protocolo Clínico sem Validação Médica (Bypass do Doctor-in-the-Loop)

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-04 |
| **Perigo** | Utilizador acessa e executa um protocolo de intervenção clínica completo sem validação de um médico. |
| **Causa** | Bug de UI expondo o payload completo do protocolo; LLM alucina `locked: false` no output XML. |
| **Efeito** | Utilizador realiza uma intervenção (ex: alteração dietética, cardio Zone 2 intenso) contraindicada para o seu contexto clínico oculto. |
| **Severidade** | 4 |
| **Probabilidade** | 2 |
| **Detecção** | 1 |
| **RPN** | **8** |
| **Mitigação** | (1) **Hard Gate do Backend:** `checkAuraPlusGate` bloqueia fisicamente o acesso se `user_active_protocols.unlocked_at` for nulo. (2) **Restrição de Prompt:** O LLM é estruturalmente forçado a emitir `locked: true` em todos os JSONs de `<protocolCard>` durante a Fase 1. |
| **Risco Residual** | Aceitável. Validação multicamada no backend impede que a UI renderize protocolos acionáveis sem aprovação médica. |
| **Rastreabilidade** | REQ-36, REQ-37 |

### HAZ-05: PHI Exposta a LLM de Terceiros sem Consentimento

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-05 |
| **Perigo** | Informações de Saúde Protegidas (PHI) são enviadas à API da Anthropic sem consentimento explícito e juridicamente vinculante do utilizador. |
| **Causa** | Diálogo de consentimento ausente; dependência de termos de serviço genéricos em vez de opt-in explícito (LGPD Art. 11). |
| **Efeito** | Violação regulatória massiva (LGPD); perda de confiança do utilizador; penalidades legais severas. |
| **Severidade** | 5 |
| **Probabilidade** | 2 |
| **Detecção** | 1 |
| **RPN** | **10** |
| **Mitigação** | (1) **Gate de BD Explícito:** `hasAuraPlusLlmConsent` verifica a tabela `user_consents` em busca de um grant `aura_plus_llm` auditado e com timestamp (design fail-closed). (2) **Sem Log Bruto:** `ai_audit_log` faz hash tanto do prompt quanto da resposta (SHA-256) para manter uma trilha de auditoria sem armazenar PHI bruta. |
| **Risco Residual** | Aceitável. O backend impede fisicamente a chamada à API se o registro explícito de consentimento estiver ausente. |
| **Rastreabilidade** | REQ-03, REQ-19, REQ-35 |

### HAZ-06: Dados Biométricos Corrompidos em Trânsito

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-06 |
| **Perigo** | Dados de saúde interceptados ou corrompidos durante a transmissão para o Supabase. |
| **Causa** | Ataque Man-in-the-Middle (MITM); downgrade de TLS em Wi-Fi público. |
| **Efeito** | PHI exposta; dados corrompidos produzem avaliação clínica incorreta. |
| **Severidade** | 4 |
| **Probabilidade** | 1 |
| **Detecção** | 1 |
| **RPN** | **4** |
| **Mitigação** | (1) Certificate Pinning via hashes SPKI SHA-256 com política fail-closed (REQ-07). (2) Conexão do backend impõe estritamente `URLSession.pinned`. |
| **Risco Residual** | Aceitável. |
| **Rastreabilidade** | REQ-02, REQ-07, REQ-10 |

### HAZ-07: Sequestro de Sessão

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-07 |
| **Perigo** | Atacante obtém acesso à sessão autenticada do utilizador. |
| **Causa** | Token JWT roubado; dispositivo compartilhado sem bloqueio de tela. |
| **Efeito** | Acesso não autorizado aos dados de saúde e conta do utilizador. |
| **Severidade** | 4 |
| **Probabilidade** | 2 |
| **Detecção** | 2 |
| **RPN** | **16** |
| **Mitigação** | (1) Autenticação biométrica obrigatória (REQ-06). (2) Timeout de inatividade de 30 minutos (REQ-08). (3) Blur de PHI em background (REQ-05). (4) JWT armazenado em Keychain com `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` (REQ-09). |
| **Risco Residual** | Baixo. |
| **Rastreabilidade** | REQ-05, REQ-06, REQ-08, REQ-09, REQ-31 |

### HAZ-08: Violação do Banco de Dados com Exposição de PHI

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-08 |
| **Perigo** | Acesso não autorizado ao banco de dados Supabase expõe registros de saúde dos utilizadores. |
| **Causa** | Regras de BD mal configuradas; chave de serviço comprometida. |
| **Efeito** | Exposição massiva de PHI; violação regulatória. |
| **Severidade** | 5 |
| **Probabilidade** | 1 |
| **Detecção** | 2 |
| **RPN** | **10** |
| **Mitigação** | (1) Row-Level Security (RLS) rigorosamente aplicada em todas as tabelas, vinculada ao JWT `auth.uid()`. (2) Criptografia em repouso. (3) Chave de serviço nunca exposta a apps clientes. |
| **Risco Residual** | Aceitável. |
| **Rastreabilidade** | REQ-01, REQ-16, REQ-31 |

### HAZ-09: Entrada de Sexo Biológico Incorreto pelo Utilizador

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-09 |
| **Perigo** | Utilizador insere sexo biológico incorreto, afetando faixas de referência laboratorial. |
| **Causa** | Erro de digitação ou entrada intencional incorreta. |
| **Efeito** | Faixas de referência específicas por sexo (ex: Ácido Úrico, HDL) aplicadas incorretamente; avaliação binária distorcida. |
| **Severidade** | 3 |
| **Probabilidade** | 2 |
| **Detecção** | 3 |
| **RPN** | **18** |
| **Mitigação** | (1) `DomainEvaluator` condicionaliza estritamente as verificações de limiares com base em `answers["biological_sex"]`. (2) Tela de perfil permite revisão e correção a qualquer momento. |
| **Risco Residual** | Baixo. |
| **Rastreabilidade** | REQ-24 |

### HAZ-10: Acesso Físico Não Autorizado ao Dispositivo

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-10 |
| **Perigo** | Pessoa não autorizada acessa fisicamente o dispositivo iOS e visualiza dados de saúde protegidos. |
| **Causa** | Dispositivo perdido ou roubado; dispositivo compartilhado em ambiente doméstico sem bloqueio de tela. |
| **Efeito** | Exposição de dados de saúde sensíveis; violação de privacidade. |
| **Severidade** | 4 |
| **Probabilidade** | 2 |
| **Detecção** | 2 |
| **RPN** | **16** |
| **Mitigação** | (1) Autenticação biométrica (Face ID / Touch ID) obrigatória para abrir o app (REQ-06). (2) App Attest garante integridade do dispositivo (REQ-11). (3) Keychain com acesso restrito a `WhenUnlockedThisDeviceOnly` (REQ-09). (4) PHI blur ativado em multitarefa (REQ-05). (5) Timeout de inatividade de 30 minutos (REQ-08). |
| **Risco Residual** | Baixo. Defesa em profundidade com 5 controles independentes. |
| **Rastreabilidade** | REQ-05, REQ-06, REQ-08, REQ-09, REQ-11, REQ-31 |

### HAZ-11: Avaliação Clínica Baseada em Dados Insuficientes

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-11 |
| **Perigo** | O sistema emite uma avaliação de domínio com base em dados incompletos, levando a uma leitura imprecisa do estado de saúde. |
| **Causa** | Utilizador não conectou wearable; poucos itens preenchidos no questionário; ausência de dados laboratoriais. |
| **Efeito** | Avaliação de risco imprecisa; falsa confiança em um status parcialmente avaliado. |
| **Severidade** | 3 |
| **Probabilidade** | 3 |
| **Detecção** | 2 |
| **RPN** | **18** |
| **Mitigação** | (1) **Ghost Mode:** Domínios com menos de 3 itens retornam `level: nil` sem gerar um status (REQ-13). (2) **Indicador de Confiança:** Badge visual mostra disponibilidade de wearables, instrumentos validados e labs (REQ-29). (3) Instrumentos clínicos validados (PHQ-9, GAD-7, PSQI, FINDRISC) substituem dados de baixa confiança (REQ-14). |
| **Risco Residual** | Baixo. O Ghost Mode impede a emissão de um status com base em dados insuficientes. |
| **Rastreabilidade** | REQ-10, REQ-13, REQ-14, REQ-28, REQ-29 |

### HAZ-12: Trilha de Auditoria de IA Incompleta ou Ausente

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-12 |
| **Perigo** | Uma interação de IA não é registrada no log de auditoria, comprometendo a rastreabilidade regulatória (CFM 2.314 Art. 5). |
| **Causa** | Falha no Supabase durante a gravação do log; timeout de rede; erro silencioso no pipeline de logging. |
| **Efeito** | Impossibilidade de auditar uma decisão de suporte clínico; não-conformidade regulatória. |
| **Severidade** | 4 |
| **Probabilidade** | 2 |
| **Detecção** | 2 |
| **RPN** | **16** |
| **Mitigação** | (1) **Log Fail-Soft:** `logAiInteraction` não bloqueia a resposta ao utilizador em caso de falha, mas registra o erro internamente para retry. (2) Hash SHA-256 de prompt e resposta garante integridade sem armazenar PHI bruta (REQ-03). (3) Versão do modelo fixada em snapshot datado (REQ-21). |
| **Risco Residual** | Baixo. Design fail-soft maximiza a taxa de sucesso de logging sem impactar a experiência do utilizador. |
| **Rastreabilidade** | REQ-03, REQ-21 |

### HAZ-13: Retenção Indevida de Dados Pessoais Após Revogação

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-13 |
| **Perigo** | Dados pessoais de saúde permanecem armazenados localmente ou no servidor após o utilizador revogar o consentimento, solicitar exclusão ou efetuar logout. |
| **Causa** | Falha no processo de exclusão em cascata; dados órfãos em cache SwiftData local; consentimento revogado mas dados persistentes. |
| **Efeito** | Violação da LGPD (direito ao esquecimento); exposição de PHI a utilizadores subsequentes do dispositivo. |
| **Severidade** | 4 |
| **Probabilidade** | 1 |
| **Detecção** | 2 |
| **RPN** | **8** |
| **Mitigação** | (1) **Exclusão de Conta:** RPC `delete_my_account()` realiza soft-delete em cascata em todas as tabelas com retenção de 30 dias (REQ-17). (2) **Higiene de Logout:** `signOut()` apaga modelos SwiftData, entradas de Keychain e estado em memória (REQ-18). (3) **Consent Gate fail-closed:** `hasAuraPlusLlmConsent` impede processamento se o consentimento for revogado (REQ-35). |
| **Risco Residual** | Aceitável. Limpeza em 3 camadas (servidor, persistência local, memória) garante eliminação completa. |
| **Rastreabilidade** | REQ-17, REQ-18, REQ-35 |

### HAZ-14: Aplicação Incorreta de Limiares de Referência por Sexo no Código

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-14 |
| **Perigo** | O código do avaliador aplica o limiar de referência do sexo errado para um biomarcador sex-specific, mesmo quando o input do utilizador está correto. |
| **Causa** | Bug na lógica de condicionalização; novo biomarcador adicionado sem branch de sexo; regressão após refatoração. |
| **Efeito** | Limiar incorreto aplicado (ex: Ácido Úrico masculino aplicado a paciente feminino), gerando avaliação distorcida. |
| **Severidade** | 3 |
| **Probabilidade** | 2 |
| **Detecção** | 2 |
| **RPN** | **12** |
| **Mitigação** | (1) Testes unitários obrigatórios para ambos os sexos em cada biomarcador sex-specific (EV-04 em VERIFICATION.md). (2) Revisão clínica pelo RT obrigatória para adição de novos biomarcadores (CONFIG_MGMT.md Seção 4.2). |
| **Risco Residual** | Aceitável. Cobertura de teste automatizada previne regressões. |
| **Rastreabilidade** | REQ-24 |

### HAZ-15: Comprometimento de Chaves Criptográficas do Dispositivo

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-15 |
| **Perigo** | Chaves criptográficas armazenadas no dispositivo (tokens, attestation keys) são extraídas ou comprometidas. |
| **Causa** | Backup não protegido do Keychain; migração de dispositivo com transferência indevida; jailbreak do iOS. |
| **Efeito** | Acesso não autorizado a APIs protegidas; falsificação de identidade do dispositivo. |
| **Severidade** | 4 |
| **Probabilidade** | 1 |
| **Detecção** | 2 |
| **RPN** | **8** |
| **Mitigação** | (1) Keychain configurado com `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` — dados não migram para backups nem novos dispositivos (REQ-09). (2) App Attest gera chave vinculada ao hardware (REQ-11). (3) SwiftData protegido com `NSFileProtectionComplete` (REQ-16). |
| **Risco Residual** | Aceitável. Vínculo ao hardware impede extração das chaves. |
| **Rastreabilidade** | REQ-09, REQ-11, REQ-16 |

### HAZ-16: Decisão Clínica Baseada em Dados Desatualizados (Cache Stale)

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-16 |
| **Perigo** | O LLM recebe um snapshot de dados clínicos desatualizado e gera recomendações baseadas em um estado de saúde que não é mais o atual. |
| **Causa** | TTL do cache expirado sem renovação; falha na re-query ao Supabase; utilizador atualizou dados durante uma sessão ativa. |
| **Efeito** | Recomendação de IA desalinhada com o estado atual de saúde do utilizador. |
| **Severidade** | 3 |
| **Probabilidade** | 2 |
| **Detecção** | 2 |
| **RPN** | **12** |
| **Mitigação** | (1) TTL de 6 horas no `health_snapshot_cache` com rebuild automático de 5 queries paralelas ao Supabase em caso de miss (REQ-32). (2) Flag `has_active_safety_alert` propagada do snapshot para o system prompt, garantindo que alertas de segurança ativos nunca expirem do contexto da IA prematuramente. |
| **Risco Residual** | Aceitável. TTL de 6 horas é conservador para dados que mudam na escala de dias/semanas. |
| **Rastreabilidade** | REQ-32 |

### HAZ-17: Falha na Ingestão de Dados de Wearables Externos

| **Campo** | **Valor** |
|---|---|
| **ID do Perigo** | HAZ-17 |
| **Perigo** | Dados de um wearable externo (Garmin, Oura, Whoop) falham ao serem ingeridos ou são ingeridos com formato corrompido. |
| **Causa** | API do fabricante indisponível; formato de dado incompatível; bundle identifier desconhecido. |
| **Efeito** | Avaliação incompleta dos domínios de Sono e Movimento; utilizador não recebe feedback sobre domínios dependentes de wearable. |
| **Severidade** | 3 |
| **Probabilidade** | 2 |
| **Detecção** | 2 |
| **RPN** | **12** |
| **Mitigação** | (1) `HealthSource` enum valida bundle identifiers conhecidos antes da ingestão (REQ-15). (2) `HealthKitSyncService` realiza sync em background com tratamento gracioso de erros (REQ-28). (3) Ghost Mode previne avaliação de domínios sem dados suficientes (REQ-13 via HAZ-11). |
| **Risco Residual** | Aceitável. Falha na ingestão resulta em domínio sem status (Ghost Mode), não em avaliação incorreta. |
| **Rastreabilidade** | REQ-15, REQ-28 |

---

## 4. Matriz Resumo de Risco

| **Faixa de RPN** | **Classificação** | **Quantidade** | **IDs dos Perigos** |
|---|---|---|---|
| **1–15 (Aceitável)** | Risco Aceitável | 9 | HAZ-04, HAZ-05, HAZ-06, HAZ-08, HAZ-13, HAZ-14, HAZ-15, HAZ-16, HAZ-17 |
| **16–39 (Baixo)** | Risco Baixo | 8 | HAZ-01, HAZ-02, HAZ-03, HAZ-07, HAZ-09, HAZ-10, HAZ-11, HAZ-12 |
| **40–74 (Médio)** | Risco Médio | 0 | Nenhum |
| **75–125 (Alto)** | Risco Alto | 0 | Nenhum |

**Avaliação Geral de Risco:** Ao migrar da antiga engine estatística (regressão linear em TypeScript) para o **Modelo de Limiares Binários de Seeman** (avaliador determinístico em Swift), o sistema eliminou todos os riscos de divergência matemática (erros de ponto flutuante, pesos dinâmicos). A introdução de LLMs criou novos riscos (Alucinação, Negligência de Crise), que foram mitigados com sucesso para os níveis "Baixo" ou "Aceitável" através de restrições arquiteturais estritas (bloqueios Doctor-in-the-Loop, gates de regex pré-LLM e trilhas de auditoria com hash criptográfico).

---

## 5. Análise Benefício-Risco

A aplicação Aura Medical (v1.0.0) fornece capacidades significativas de monitorização preventiva de saúde e suporte à decisão clínica (CDSS). A implementação de sobreposições de segurança determinísticas (`isCrisisInput`) e bloqueios rígidos de backend (`checkAuraPlusGate`) garante que os benefícios da IA conversacional na educação em saúde superam significativamente os riscos residuais.

O produto **não diagnostica** e **não executa tratamentos de forma autônoma**. O perfil de risco residual é totalmente consistente com as classificações IEC 62304 Classe B e IMDRF Categoria II.

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Gestor de Risco (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gestor de Qualidade | Frederico | | |
| Líder de Desenvolvimento | Arthur Teixeira de Almeida | | |
