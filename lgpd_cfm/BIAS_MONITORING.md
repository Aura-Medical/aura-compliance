# Procedimento de Monitoramento de Viés Discriminatório

**ID do Documento:** BM-001

**Revisão:** 1.0

**Data:** 2026-04-10

**Produto:** Aura Medical — Módulo Aura+ (IA Conversacional)

**Norma:** Resolução CFM 2.454/2026 Anexo III, item II; Arts. 9°–10°

---

## 1. Propósito

Este documento define o procedimento de monitoramento contínuo dos outputs da IA para identificação e mitigação de vieses discriminatórios ilegais ou antiéticos, conforme exigido pelo Anexo III, item II, da Resolução CFM 2.454/2026.

## 2. Definições

| **Termo** | **Definição (Anexo I, XIV)** |
|---|---|
| **Viés discriminatório ilegal ou abusivo** | Resultado indevidamente discriminatório produzido por um sistema de IA que cria, agrava ou reforça preconceitos ou exclusões, derivado ou não dos dados utilizados ou de falhas de modelagem, e que resulte em tratamento injustificado diferenciado entre indivíduos ou grupos em contexto médico. |

## 3. Escopo de Monitoramento

### 3.1 Categorias Protegidas

O monitoramento avalia viés em relação às seguintes características (Art. 9°; Exposição de Motivos, I):

- Origem e etnia
- Raça e cor
- Sexo e gênero
- Orientação sexual
- Idade
- Religião
- Condição socioeconômica
- Condição de saúde (incluindo saúde mental)

### 3.2 Dimensões de Análise

| **Dimensão** | **O que é monitorado** | **Risco** |
|---|---|---|
| Qualidade de resposta | Diferenças na profundidade/utilidade das respostas entre grupos demográficos. | IA fornecer respostas menos detalhadas para determinados perfis. |
| Safety Gate | Taxa de acionamento do Safety Gate estratificada por grupo. | Disparidade injustificada no encaminhamento de crise. |
| Linguagem e tom | Variações de tom, empatia ou assertividade entre grupos. | Tratamento linguístico discriminatório. |
| Recomendações de bem-estar | Diferenças nas sugestões de estilo de vida entre grupos. | Estereótipos culturais ou de gênero nas recomendações. |

## 4. Metodologia

### 4.1 Coleta de Dados

A análise de viés utiliza dados **agregados e anonimizados** do `ai_audit_log` e metadados demográficos voluntariamente fornecidos pelo utilizador (quando disponíveis). Em nenhum momento dados individuais identificáveis são expostos na análise de viés.

### 4.2 Análise Trimestral

| **Etapa** | **Ação** | **Responsável** |
|---|---|---|
| 1. Extração | Extrair métricas agregadas do `ai_audit_log` (volume, Safety Gate, tokens). | Engenharia |
| 2. Estratificação | Estratificar métricas por grupo demográfico disponível (idade, sexo). | Engenharia |
| 3. Teste Estatístico | Aplicar testes de diferença significativa entre grupos (chi-quadrado, Mann-Whitney). | Engenharia |
| 4. Revisão Qualitativa | Amostragem aleatória de interações (reconstruídas via hash) para revisão de tom e conteúdo. | Coordenador Médico |
| 5. Relatório | Documentar achados, comparar com trimestre anterior, recomendar ações. | Comissão de IA |
| 6. Deliberação | Comissão delibera sobre ações corretivas, se necessário. | Comissão de IA |

### 4.3 Testes com Prompts Sintéticos

Complementarmente à análise de dados reais, serão realizados testes trimestrais com **prompts sintéticos** (sem dados reais de pacientes) que simulam perfis demográficos distintos, para avaliar se o LLM produz respostas qualitativamente diferentes para perfis equivalentes variando apenas a categoria protegida.

**Exemplo:**
- Prompt A: Homem, 35 anos, mesmo perfil de saúde → Resposta A
- Prompt B: Mulher, 35 anos, mesmo perfil de saúde → Resposta B
- Análise: Diferenças significativas em profundidade, tom ou recomendações?

## 5. Critérios de Detecção e Escala de Ação

### 5.1 Níveis de Severidade

| **Nível** | **Critério** | **Ação** | **Prazo** |
|---|---|---|---|
| **Informacional** | Diferença estatisticamente não significativa entre grupos. | Documentar, manter monitoramento. | Próximo ciclo trimestral. |
| **Atenção** | Diferença estatisticamente significativa, mas sem impacto clínico aparente. | Investigar causa raiz. Ajustar system prompt se atribuível. | 30 dias. |
| **Crítico** | Viés com potencial impacto clínico ou discriminação evidente. | Medida corretiva imediata: ajuste de prompt, retreinamento, restrição de funcionalidade. | 7 dias. |
| **Inaceitável** | Viés não eliminável após medidas corretivas. | Descontinuação da funcionalidade afetada conforme Art. 10, §2°. | Imediato. |

### 5.2 Medidas Corretivas (Anexo III, II)

Conforme a resolução, em caso de detecção de viés indevido:

1. **Ajuste do modelo** — Modificação do system prompt para incluir instruções explícitas de equidade.
2. **Retreinamento com dados balanceados** — Não aplicável diretamente (LLM de terceiro — Anthropic), mas aplicável via ajuste de contexto e few-shot examples.
3. **Restrição de uso** — Limitar funcionalidade afetada até correção.
4. **Descontinuação** — Em casos graves e não corrigíveis, descontinuar o sistema conforme Art. 10, §2°.

## 6. Relatório de Viés

Cada ciclo trimestral gera um **Relatório de Monitoramento de Viés** contendo:

1. Período analisado.
2. Volume de interações analisadas (agregado).
3. Métricas estratificadas por grupo (quando disponível).
4. Resultados de testes estatísticos.
5. Resultados de testes com prompts sintéticos.
6. Comparação com ciclo anterior.
7. Ações corretivas tomadas ou recomendadas.
8. Classificação de severidade.
9. Aprovação da Comissão de IA.

Os relatórios são armazenados como evidência de compliance e disponibilizados aos órgãos de controle conforme AI_GOVERNANCE.md §4.6.

## 7. Rastreabilidade

| **Requisito** | **Vínculo** |
|---|---|
| Prevenção de viés (Anexo III, II) | Este documento |
| Viés discriminatório (Anexo I, XIV) | Seção 2 |
| Não discriminação (Art. 9°, Exposição de Motivos I) | Seção 3.1 |
| Monitoramento regular — médio risco (Anexo II, §2°) | RISK_CLASSIFICATION.md §4 |
| Deliberação da Comissão (Art. 14) | AI_GOVERNANCE.md §4.2 |

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Encarregado (DPO) | Frederico | | |
