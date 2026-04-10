# Declaração de Indicação de Uso (Intended Use Statement)

**ID do Documento:** IU-001

**Revisão:** 3.0 (v1.0.0 Stable Release)

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

## 5. Racional de Classificação Sanitária

O software Aura Medical v1.0.0 é classificado como **SaMD Classe II**. Esta classificação justifica-se porque o sistema fornece informações que auxiliam no diagnóstico e monitorização (Suporte à Decisão), incidindo sobre situações de saúde não críticas, onde a falha do software pode induzir a uma decisão clínica subótima, mas não a um risco imediato de vida, dada a presença obrigatória do médico no ciclo de ativação de protocolos (*Doctor-in-the-loop*).

---

**Assinaturas e Responsabilidade Técnica:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Compliance e Qualidade | Frederico | | |
