# Instruções de Uso (IFU)

**ID do Documento:** IF-001

**Revisão:** 1.0

**Data:** 2026-04-10

**Produto:** Aura Medical — Aplicativo iOS para Saúde Preventiva

**Fabricante:** Auramedical Tecnologia Ltda.

**Classificação:** SaMD Classe II (Anvisa RDC 657/2022)

**Norma:** RDC 657/2022 Art. 12; RDC 751/2022 (Rotulagem de Dispositivos Médicos)

---

## 1. Identificação do Produto

| Campo | Informação |
|---|---|
| **Nome comercial** | Aura Medical |
| **Módulo de IA** | Aura+ |
| **Versão** | 1.0.0 |
| **Plataforma** | iOS 17 ou superior |
| **Fabricante** | Auramedical Tecnologia Ltda. |
| **Responsável Técnico** | Dr. Alexandre Teixeira de Almeida |
| **Classificação de Risco** | SaMD Classe II (Médio Risco) |

## 2. Indicação de Uso

A Aura Medical é um aplicativo de suporte à decisão clínica (CDSS) que auxilia adultos saudáveis na monitorização de indicadores de saúde preventiva e longevidade. O aplicativo:

- Avalia 5 domínios de saúde (metabólico, cardiovascular, sono, emocional/cognitivo e movimento) utilizando limiares binários baseados em evidências.
- Processa instrumentos clínicos validados (PHQ-9, GAD-7, PSQI, FINDRISC).
- Integra dados de wearables (Apple Watch, Garmin, Oura) e resultados laboratoriais.
- Oferece conversas educativas sobre saúde preventiva via inteligência artificial (Aura+).
- Exibe protocolos de intervenção em modo de visualização prévia (preview).

## 3. Indicação de Uso — O Que Este Software FAZ

- Apresenta alertas binários (Saudável/Alerta) para cada domínio de saúde com base nos seus dados.
- Calcula e interpreta scores de instrumentos clínicos padronizados.
- Fornece insights educacionais sobre saúde preventiva baseados na doutrina Medicine 3.0.
- Detecta automaticamente expressões de crise emocional e direciona para emergência (CVV 188 / SAMU 192).

## 4. Limitações — O Que Este Software NÃO FAZ

**ATENÇÃO — Leia com cuidado:**

- **NÃO fornece diagnósticos médicos.** O software identifica alertas de risco, mas não nomeia doenças nem substitui avaliação clínica presencial.
- **NÃO prescreve medicamentos ou suplementos.** Nenhuma recomendação de produto por nome comercial é emitida.
- **NÃO substitui consulta médica.** A Aura Medical é uma ferramenta de apoio, não um profissional de saúde.
- **NÃO executa tratamentos.** Protocolos de intervenção são exibidos em modo de visualização prévia e permanecem bloqueados até validação por um médico.
- **NÃO controla dispositivos médicos.** O software não regula bombas de insulina, pacemakers ou qualquer equipamento médico.
- **NÃO deve ser utilizado em emergências médicas.** Em caso de emergência, ligue 192 (SAMU). Em crise emocional, ligue 188 (CVV) — 24h, gratuito e sigiloso.

## 5. Público-Alvo

- **Utilizadores:** Adultos de 18 a 80 anos com literacia digital básica.
- **Profissionais:** Médicos que utilizam o sistema para acompanhar pacientes e desbloquear protocolos.

## 6. Contraindicações

O software **NÃO é adequado** para:

- Indivíduos com patologias crônicas descompensadas (ex: diabetes descontrolada, insuficiência cardíaca NYHA III-IV).
- Populações pediátricas (< 18 anos).
- Situações de emergência médica ou psiquiátrica aguda.
- Uso em unidades de cuidados intensivos, blocos operatórios ou ambientes clínicos de alta complexidade.
- Gestantes (limiares de biomarcadores não ajustados para gravidez).

## 7. Advertências e Precauções

### 7.1 Advertências

- Os resultados do software são informativos e educacionais. Não tome decisões de saúde baseado exclusivamente neste aplicativo.
- Valores laboratoriais inseridos manualmente estão sujeitos a erro de digitação. Verifique os valores antes de confirmar.
- A inteligência artificial (Aura+) pode gerar informações imprecisas. Sempre confirme com seu médico.
- O sistema de detecção de crise utiliza padrões de texto e pode não capturar todas as formas de expressão de sofrimento.

### 7.2 Precauções

- Mantenha o aplicativo atualizado para garantir os limiares clínicos mais recentes.
- Não compartilhe seu dispositivo com terceiros — o aplicativo contém dados de saúde sensíveis.
- A biometria (Face ID / Touch ID) é recomendada para proteção dos seus dados.
- Dados de wearables dependem da calibração do dispositivo. O software não verifica a precisão do hardware de origem.

## 8. Como Usar

### 8.1 Primeiro Acesso

1. Baixe o aplicativo Aura Medical na App Store.
2. Crie uma conta com email e senha.
3. Preencha o questionário inicial de saúde (onboarding).
4. Conecte seus wearables (Apple Watch, Garmin, Oura) via HealthKit.
5. Opcionalmente, insira resultados laboratoriais via digitação manual ou fotografia do exame.

### 8.2 Uso do Módulo Aura+ (IA)

1. Ao abrir o Aura+, uma tela de consentimento será apresentada explicando que seus dados de saúde serão processados por inteligência artificial.
2. Após consentir, você pode conversar com a Aura+ sobre seus indicadores de saúde.
3. A Aura+ pode sugerir protocolos de intervenção em modo de visualização prévia (preview).
4. Para acessar protocolos completos, uma consulta com médico da rede Aura é necessária.

### 8.3 Detecção de Crise

Se você expressar sofrimento emocional grave durante uma conversa com a Aura+, o sistema **interromperá automaticamente** a conversa e exibirá:

- **CVV 188** — Centro de Valorização da Vida (24h, gratuito, sigiloso)
- **SAMU 192** — Serviço de Atendimento Móvel de Urgência

Esta detecção funciona independentemente da inteligência artificial.

## 9. Dados e Privacidade

- Seus dados de saúde são classificados como dados pessoais sensíveis (LGPD Art. 5°, II).
- O processamento por IA requer consentimento explícito, que pode ser revogado a qualquer momento em Configurações > Privacidade.
- As conversas com a Aura+ são registradas com hash criptográfico (SHA-256) para fins de auditoria, sem armazenamento do conteúdo bruto.
- A exclusão de conta pode ser solicitada em Configurações > Conta > Excluir Conta.

Para detalhes completos, consulte a Política de Privacidade disponível no aplicativo.

## 10. Suporte

- **Email:** suporte@auramedical.com.br
- **Dentro do app:** Configurações > Ajuda

## 11. Informações Regulatórias

- **Classificação:** SaMD Classe II (Anvisa RDC 657/2022)
- **Classificação de IA:** Médio Risco (CFM 2.454/2026)
- **Normas aplicáveis:** IEC 62304, ISO 14971, LGPD, CFM 2.454/2026
- **Responsável Técnico:** Dr. Alexandre Teixeira de Almeida

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Compliance e Qualidade | Frederico | | |
