# Declaração de Indicação de Uso (Intended Use Statement)

**ID do Documento:** IU-001

**Revisão:** 4.0 (v1.0.0 Stable Release)

**Data:** 2026-04-10

**Produto:** Aura Medical iOS Application

**Fabricante:** Auramedical Tecnologia Ltda.

**Classificação:** SaMD Classe II (Anvisa RDC 657/2022), IMDRF Categoria II

---

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

A Aura Medical destina-se a fornecer informações de bem-estar geral e automonitorização para adultos saudáveis. O software agrega dados laboratoriais, biométricos e de wearables para identificar desvios em relação a limiares clínicos ideais estabelecidos pela doutrina **Medicine 3.0** (Attia/Huberman).

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
