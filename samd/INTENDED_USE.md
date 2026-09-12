# Declaração de Indicação de Uso (Intended Use Statement)

**ID do Documento:** IU-001

**Revisão:** 5.1 — notificação ADIADA por decisão do RT (ver o bloco ⛔ abaixo)

**Data:** 2026-09-12 (rev. 4.0 em 2026-04-10)

**Produto:** Aura Medical iOS Application

**Fabricante:** AURAMEDICAL SERVICOS MEDICOS LTDA — CNPJ 63.556.151/0001-40

**Classificação:** SaMD Classe II (Anvisa RDC 657/2022), IMDRF Categoria II

---

## ⛔ EMENDA 5.1 — 2026-09-12, fim do dia: a notificação foi ADIADA, e o produto NÃO mudou

**Leia isto antes de qualquer coisa abaixo.** A rev. 5.0, escrita algumas horas antes, resolveu a
contradição do documento escolhendo a **Versão B (CDSS/SaMD Classe II)** e foi escrita para
sustentar uma notificação que seria protocolada no mesmo dia. **Ela não foi.**

**O que aconteceu:** o portal da ANVISA recusou o peticionamento por falta de **AFE**, que exige
licença sanitária do estabelecimento, que exige endereço físico — e a empresa opera em escritório
virtual. O formulário pede o número da AFE em **§2.10** e em **§3.1**.

**E havia razão independente para não insistir:** o §7 do formulário é assinado pelo RL **e** pelo
RT sob a Lei 6.437 e o **art. 273 do Código Penal**, declarando conformidade com a RDC 546/2021 e a
RDC 665/2022. O relatório de usabilidade (IEC 62366-1), os testes de compatibilidade e o relatório
de V&V **não foram executados** (`DOSSIE_GAP_ART11.md`). Notificar hoje seria declarar verdadeiro o
que ainda não é.

**A decisão do RT, em três partes:**

1. **Notificação adiada, sem data.** Gatilho nomeado para retomar: endereço próprio → licença
   sanitária → AFE → os três relatórios executados.
2. **As funções que classificam contra limiar clínico FICAM — todas.** Carga alostática, PREVENT
   (AHA 2024), selos Ótimo/Normal/Fora da faixa, PHQ-9/GAD-7/PSQI/FINDRISC pontuados e
   classificados. Palavras do RT: *"ficam — assumo o risco"*.
3. **As DECLARAÇÕES de CDSS/SaMD saíram** do app (`AvisoLegalView`) e dos termos publicados
   (`aura.med/pt/termos` e `/en`).

**⚠️ O estado resultante não é a Versão A nem a Versão B — é comportamento da Versão B com
declarações da Versão A.** Está escrito aqui, sem maquiagem, porque um dossiê que esconde isso é
pior do que um dossiê ausente. A exposição é administrativa (Lei 6.437: advertência, multa,
suspensão), recai sobre a empresa e sobre o RT, e a decisão é dele como responsável técnico e
fabricante.

**Medida que fundamentou a decisão** (levantamento adversarial no código real, 45 agentes, refutador
independente por achado, padrão *na dúvida refuta*): **40 achados · 10 refutados · 28 sobreviveram**
como funções fora do carve-out de bem-estar — **27 visíveis ao usuário, 29 no MVP gratuito**. Ou
seja: a Versão A **não estava disponível** sem desmontar o produto. Não era questão de redação.

**Caminho aprovado que reduz a distância sem custar produto** (agendado para depois da submissão,
DIV-559): a **remoldagem de voz** — o app deixa de *julgar* (*"sua Lp(a) está fora da faixa, risco
alto"*) e passa a *atribuir* (o número do paciente ao lado de uma referência nomeada e citada).
Muda o autor da afirmação. Cobre os exames e, com um argumento melhor ainda, os instrumentos — que
apenas **reportam o corte que o autor do instrumento publicou**. **Não cobre** a carga alostática
nem o PREVENT: os dois produzem um número **novo** sobre a pessoa.

**Decisão completa:** `~/Documents/Claude/Projects/Aura/docs/decision-enquadramento-mvp-2026-09-12.md`

> O texto abaixo (§1 em diante) descreve o produto e permanece **tecnicamente correto** — ele
> descreve o que o software faz, e o software não mudou. O que deixou de valer é a **conclusão
> regulatória** da rev. 5.0 de que isso seria notificado agora.

---

> **Migração de titularidade — 2026-09-12.** O fabricante/controlador passa de
> *Auramedical Tecnologia Ltda.* para **AURAMEDICAL SERVICOS MEDICOS LTDA**
> (CNPJ 63.556.151/0001-40), por decisão do RT. A empresa médica já carrega os CNAEs de
> software no objeto social — 62.02-3-00 e 62.03-1-00 (desenvolvimento e
> licenciamento de programas de computador) — além do CNAE médico principal
> 86.30-5-03, então ela pode ser fabricante de SaMD **e** prestadora do ato
> médico. Isso elimina a inconsistência futura entre o titular da conta da App
> Store e o prestador do serviço (App Review 5.1.1(ix)).

## 1. Descrição do Produto

A Aura Medical é uma aplicação de saúde móvel para iOS que funciona como um sistema de suporte à decisão clínica (CDSS), focado no cálculo e monitorização da **carga alostática** e otimização da longevidade. O software utiliza o **Modelo de Seeman de Limiares Binários** para processar dados de múltiplas fontes e fornecer uma avaliação do estado fisiológico do utilizador.

### 1.1 Domínios de Saúde Avaliados

O software analisa cinco domínios fisiológicos distintos através de uma lógica de "flags" de risco:

- **Metabólico:** Avalia biomarcadores como HOMA-IR, Glicose, HbA1c, Insulina e Relação Cintura/Altura.
- **Cardiovascular:** Monitoriza Pressão Arterial, ApoB, LDL, Lp(a), Homocisteína e inflamação sistémica (PCR-us).
- **Sono:** Processa métricas de arquitetura do sono (eficiência, latência, duração) via integração com wearables e o instrumento PSQI.
- **Nervoso / Emocional:** Analisa a Variabilidade da Frequência Cardíaca (HRV), Cortisol matinal e escalas psiquiátricas validadas (PHQ-9/GAD-7).
- **Movimento:** Avalia a capacidade aeróbica (VO2 Max) e níveis de atividade física.

## 2. Indicação de Uso

> **Revisão 5.0 — 2026-09-12.** Esta seção foi REESCRITA. A redação anterior dizia
> *"informações de bem-estar geral"* ao mesmo tempo em que o §1 deste documento
> descrevia o produto como CDSS. As duas afirmações se anulam sob a RDC 657/2022:
> o **Art. 1º, §2º, I** exclui do escopo os "softwares para bem-estar", e o
> **Art. 2º, IX** os define como os que **não** se destinam a prevenção,
> diagnóstico, tratamento, reabilitação ou anticoncepção. Ou o produto é
> bem-estar e a RDC não se aplica, ou é dispositivo e a Classe II vale.
> O RT decidiu pela segunda (2026-09-12). A escolha e as duas redações avaliadas
> estão em `INTENDED_USE_DECISAO.md`.

A Aura Medical é um **software como dispositivo médico (SaMD)** destinado a apoiar a
**prevenção** e o automonitoramento de fatores de risco cardiometabólico, do sono, da
atividade física e da saúde mental em **adultos**. O software agrega dados
laboratoriais, biométricos, de dispositivos vestíveis e de instrumentos clínicos
validados, e os classifica contra limiares estabelecidos na literatura primária,
sinalizando desvios que merecem atenção.

A saída do software é **informação para apoiar decisão**, dirigida ao usuário e ao
profissional de saúde que o acompanha. O software **não estabelece diagnóstico, não
prescreve e não substitui avaliação médica** — toda conduta permanece com o
profissional habilitado (CFM 2.454/2026, Art. 1º e Art. 9º). A associação clínica de
cada limiar é referenciada à **literatura primária** e assinada pelo Responsável
Técnico antes de entrar em produção (ver `CONFIG_MGMT.md` §4.2).

**O que esta indicação NÃO autoriza**, e está escrito para não ser lido a mais:
o software não se destina a **tratamento**, **reabilitação** ou **anticoncepção**;
não é destinado a uso pediátrico; não é destinado a situação de emergência; e não
substitui monitorização clínica de paciente instável.

### 2.1 Funções Clínicas do Software

- **Avaliação Binária de Risco:** Classifica cada input de forma determinística como Saudável (0) ou Alerta (1), impedindo que dados positivos ocultem riscos críticos (Data Caps).
- **Triagem de Segurança (Safety Gate):** Intercepta de forma automática padrões de texto que indiquem crise aguda (ideação suicida) e interrompe a conversa para fornecer instruções de emergência.
- **Processamento de Instrumentos:** Calcula e interpreta scores de escalas clínicas padronizadas (ex: PHQ-9), gerando flags de risco psicossocial auditáveis.
- **Preview de Protocolos:** Sugere intervenções baseadas em evidências (ex: Alvo Proteico, Zone 2), permitindo a visualização educativa de estratégias de longevidade.

### 2.2 Limitações e Contraindicações (O que NÃO faz)

- **Não fornece diagnósticos médicos:** O software identifica alertas de risco, mas não nomeia patologias nem substitui a avaliação clínica presencial.
- **Não prescreve terapias:** A recomendação de suplementos ou fármacos por nome comercial é estritamente proibida; o software atua apenas na camada de educação sobre classes terapêuticas.
- **Não controla dispositivos médicos:** O software não regula bombas de insulina, pacemakers ou qualquer sistema de suporte à vida.
- **Acesso bloqueado a protocolos ativos:** Protocolos de intervenção detalhados permanecem inacessíveis (locked) até que um médico realize a consulta e valide a segurança clínica daquela estratégia para o utilizador.

## 3. Utilizadores-Alvo e Perfil

- **Público-Alvo:** Adultos (18-80 anos) com literacia digital básica.
- **Contraindicação de Uso:** O software não é adequado para indivíduos com patologias crónicas descompensadas, populações pediátricas ou em situações de emergência médica.
- **Ambiente:** Utilização pessoal e doméstica; não destinado ao uso em unidades de cuidados intensivos ou blocos operatórios.

## 4. Fontes de Dados e Integração

O software é agnóstico em relação ao hardware e processa dados provenientes de:

- **Apple HealthKit:** HRV, Passos, Sono e Frequência Cardíaca.
- **Wearables Externos:** Ingestão de dados de Garmin, Oura, Whoop e balanças de bioimpedância via APIs.
- **Laboratórios:** Resultados de sangue processados via entrada manual ou leitura OCR.

## 5. Base Regulatória

### 5.1 Classificação Anvisa (RDC 657/2022)

O software é classificado como SaMD Classe II nos termos da RDC 657/2022:

- **Art. 6°:** Define Software como Dispositivo Médico (SaMD) como software destinado a ser utilizado para uma ou mais finalidades médicas, executando essas finalidades sem ser parte de um dispositivo médico de hardware.
- **Art. 7°, II:** Classe II — software cujo uso incorreto pode contribuir para decisão clínica subótima, sem risco imediato de vida.
- **Art. 12:** Exige descrição da finalidade pretendida e perfil do utilizador-alvo (Seções 2 e 3 deste documento).

### 5.2 Classificação IMDRF (SaMD N12)

Conforme a matriz de categorização do IMDRF *Software as a Medical Device: Possible Framework for Risk Categorization and Corresponding Considerations* (IMDRF/SaMD WG/N12FINAL:2014):

| Significância da Informação | Não-Crítico | Sério | Crítico |
|---|---|---|---|
| Informar decisão clínica | I | II | II |
| **Conduzir gestão clínica** | **II** | III | III |
| Tratar ou diagnosticar | III | III | IV |

**Enquadramento da Aura Medical:** "Conduzir Gestão Clínica" × "Estado de Saúde Não-Crítico" = **Categoria II**.

**Justificativa:**
- O software **conduz** (*drive*) a gestão clínica ao classificar domínios de saúde em limiares binários e sugerir protocolos preventivos baseados em evidências, indo além de meramente informar.
- O estado de saúde abordado é **não-crítico** (longevidade preventiva e bem-estar — Medicine 3.0; não UTI, emergência ou doenças descompensadas).
- A presença de instrumentos clínicos validados (PHQ-9, GAD-7, PSQI, FINDRISC) confere significância clínica ao output, distinguindo o software de apps de "wellness" genéricos.

### 5.3 Argumento CDSS — Clinical Decision Support System

A Aura Medical opera como CDSS conforme definição do IMDRF N12 §6.3: fornece informações derivadas de dados de saúde que auxiliam profissionais e pacientes na tomada de decisão clínica, sem substituir o julgamento clínico humano.

A isenção de "bem-estar/wellness" da RDC 657/2022 **não se aplica** porque o software:
- Processa dados fisiológicos vitais (ApoB, HbA1c, HOMA-IR) com limiares clinicamente definidos.
- Utiliza instrumentos clínicos de triagem validados internacionalmente (PHQ-9, FINDRISC).
- Gera alertas de risco que informam a gestão clínica do utilizador.

### 5.4 Argumento Doctor-in-the-Loop — Não-Elevação a Classe III

O sistema implementa **supervisão médica obrigatória** (Doctor-in-the-Loop) como controle arquitetural que impede autonomia clínica:

- Protocolos de intervenção permanecem bloqueados (`locked: true`) até validação médica presencial ou por telemedicina (REQ-36, `gate.ts`).
- O Safety Gate intercepta crises agudas antes do LLM, encaminhando para emergência humana (REQ-04, `safety.ts`).
- A IA não diagnostica, não prescreve e não comunica prognósticos sem mediação humana.

Este controle é o fator determinante para a classificação como Classe II e não Classe III: a falha do software não gera morte ou dano irreversível imediato porque a ação clínica final depende sempre do médico.

**Referência:** CFM 2.454/2026 Art. 15, parágrafo único — *"As soluções de IA não são soberanas, sendo obrigatória a supervisão humana."*

### 5.5 Comunicação da Classificação de Risco ao Utilizador (CFM 2.454/2026 Art. 13)

Conforme Art. 13 da Resolução CFM 2.454/2026, o nível de risco é comunicado ao utilizador na tela de consentimento (`lgpd_cfm/CONSENT.md` §4.1) e na política de privacidade. Para a classificação de risco específica do módulo de IA, vide `lgpd_cfm/RISK_CLASSIFICATION.md`.

## 6. Racional de Classificação Sanitária

O software Aura Medical v1.0.0 é classificado como **SaMD Classe II** (RDC 657/2022 Art. 7°, II) / **IMDRF Categoria II** (N12). Esta classificação justifica-se porque:

1. O sistema fornece informações que **conduzem a gestão clínica** (CDSS), incidindo sobre situações de saúde **não-críticas** (prevenção e longevidade).
2. A falha do software pode induzir a uma decisão clínica subótima, mas **não a risco imediato de vida**, dada a presença obrigatória do médico no ciclo de ativação de protocolos (*Doctor-in-the-Loop* — §5.4).
3. A classificação como Categoria II e não III fundamenta-se na presença do controle arquitetural Doctor-in-the-Loop (REQ-36) e na incidência exclusiva sobre estados de saúde não-críticos (§5.2).
4. O perfil de risco é consistente com IEC 62304 Classe B.

Para a análise detalhada de perigos e mitigações, vide `RISK_ANALYSIS.md` (RA-001).

## 7. Referências Cruzadas

| **Documento** | **Relação** |
|---|---|
| `RISK_ANALYSIS.md` (RA-001) | Análise FMEA com 17 perigos vinculados ao perfil SaMD Classe II |
| `TRACEABILITY.md` (TM-001) | 33 requisitos que implementam os controles de risco |
| `lgpd_cfm/RISK_CLASSIFICATION.md` (RC-001) | Classificação de risco de IA (CFM 2.454/2026 Arts. 12–13) |
| `lgpd_cfm/AI_GOVERNANCE.md` (GV-001) | Comissão de IA e Telemedicina (CFM 2.454/2026 Art. 14) |
| `lgpd_cfm/CONSENT.md` (CN-001) | Comunicação da classificação de risco ao utilizador (Art. 13) |

---

**Assinaturas e Responsabilidade Técnica:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Compliance e Qualidade | Frederico | | |
