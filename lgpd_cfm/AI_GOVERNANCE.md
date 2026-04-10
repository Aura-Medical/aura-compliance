# Carta de Governança — Comissão de IA e Telemedicina

**ID do Documento:** GV-001

**Revisão:** 1.0

**Data:** 2026-04-10

**Produto:** Aura Medical — Módulo Aura+ (IA Conversacional)

**Norma:** Resolução CFM 2.454/2026 Art. 14 e Anexo III

---

## 1. Propósito

Este documento institui a Comissão de Inteligência Artificial e Telemedicina da Aura Medical, conforme exigido pelo Art. 14, parágrafo único, da Resolução CFM 2.454/2026, que determina a criação de comissão sob coordenação médica, subordinada à diretoria técnica, para assegurar o uso ético de sistemas de IA.

## 2. Base Legal

| **Dispositivo** | **Obrigação** |
|---|---|
| Art. 14 | Estabelecer processos internos de governança para segurança, qualidade e ética. |
| Art. 14, parágrafo único | Criar Comissão de IA e Telemedicina sob coordenação médica. |
| Art. 15, parágrafo único | As soluções de IA não são soberanas; supervisão humana é obrigatória. |
| Anexo III | Medidas de governança (itens I a VIII). |

## 3. Composição da Comissão

| **Função na Comissão** | **Nome** | **Cargo** | **Responsabilidade** |
|---|---|---|---|
| **Coordenador Médico** | Dr. Alexandre Teixeira de Almeida | Responsável Técnico (RT) | Coordenação geral, decisões sobre segurança clínica e ética. |
| **Diretor Técnico** | Arthur Teixeira de Almeida | Gerência Executiva | Supervisão administrativa e subordinação hierárquica da comissão. |
| **Encarregado de Dados (DPO)** | Frederico | DPO | Conformidade com LGPD, proteção de dados, interface com ANPD. |
| **Representante de Engenharia** | A definir | Líder Técnico | Implementação técnica, monitoramento e auditoria de sistemas. |

### 3.1 Subordinação Hierárquica

Conforme Art. 14, parágrafo único, a Comissão é **subordinada à diretoria técnica** (Arthur Teixeira de Almeida) e opera sob **coordenação médica** (Dr. Alexandre Teixeira de Almeida).

## 4. Competências da Comissão

### 4.1 Governança e Transparência (Anexo III, I)

- Aprovar e revisar relatórios de governança da IA que descrevam desempenho, limitações, vieses identificados e mitigações.
- Garantir que informações sobre funcionamento, finalidades e mecanismos de supervisão da IA sejam divulgadas de forma acessível.
- Manter documentação atualizada e acessível sobre todos os sistemas de IA em uso.

### 4.2 Prevenção de Viés Discriminatório (Anexo III, II)

- Supervisionar o monitoramento contínuo de outputs da IA para identificação de vieses.
- Deliberar sobre medidas corretivas em caso de detecção de viés (ajuste, retreinamento, restrição de uso).
- Autorizar descontinuação de sistema em caso de viés insolúvel conforme procedimento em BIAS_MONITORING.md.

### 4.3 Mecanismos de Governança Interna (Anexo III, III)

- O Diretor Técnico é o responsável pela fiscalização das diretrizes de segurança, ética e transparência.
- A Comissão reporta diretamente ao Diretor Técnico.

### 4.4 Gestão do Ciclo de Vida (Anexo III, VI)

- Aprovar atualizações de modelos de IA (mudança de versão do LLM).
- Revisar e aprovar novas funcionalidades da IA antes da implantação.
- Supervisionar o processo de validação/testes de cada release.
- Detalhes operacionais em LIFECYCLE_MGMT.md.

### 4.5 Classificação e Reclassificação de Risco (Arts. 12-13)

- Realizar a avaliação preliminar de risco para novos sistemas ou funcionalidades de IA.
- Deliberar sobre reclassificação de risco conforme gatilhos definidos em RISK_CLASSIFICATION.md §5.
- Documentar e comunicar mudanças de classificação.

### 4.6 Acesso de Órgãos de Controle (Anexo III, VIII)

- Garantir acesso a relatórios de auditoria e monitoramento quando solicitado por:
  - Conselho Regional de Medicina (CRM)
  - Conselho Federal de Medicina (CFM)
  - Autoridade Nacional de Proteção de Dados (ANPD)
  - Agência Nacional de Vigilância Sanitária (Anvisa)
  - Ministério Público
  - Comissão Nacional de Ética em Pesquisa (CONEP), quando aplicável
- Designar ponto de contato para cada órgão solicitante.

### 4.7 Comunicação de Falhas (Art. 7°, §2)

- Receber e avaliar relatórios internos de falhas, riscos relevantes ou usos inadequados da IA.
- Deliberar sobre comunicação às instâncias competentes (CRM, CFM, Anvisa).

## 5. Funcionamento

### 5.1 Reuniões Ordinárias

| **Tipo** | **Frequência** | **Pauta Mínima** |
|---|---|---|
| Ordinária | Trimestral | Revisão de métricas de auditoria, relatório de viés, incidentes, reclassificação de risco. |
| Extraordinária | Sob demanda | Incidentes de segurança, falhas de IA, solicitação de órgão regulatório, mudança significativa. |

### 5.2 Quórum e Deliberação

- Quórum mínimo: Coordenador Médico + 1 membro.
- Decisões sobre reclassificação de risco ou descontinuação de sistema exigem aprovação do Coordenador Médico.
- Atas de reunião são documentadas e arquivadas como evidência de compliance.

### 5.3 Registros

Todas as deliberações da Comissão são registradas em ata contendo:
- Data, participantes e pauta.
- Decisões tomadas e justificativa.
- Ações atribuídas, responsáveis e prazos.
- Próxima revisão programada.

## 6. Checklist de Conformidade com Anexo III

| **Medida (Anexo III)** | **Status** | **Documento** |
|---|---|---|
| I — Transparência da governança | Implementado | AI_AUDIT.md, este documento |
| II — Prevenção de viés | Implementado | BIAS_MONITORING.md |
| III — Governança interna (Diretor Técnico) | Implementado | Este documento §4.3 |
| IV — Interoperabilidade | Em avaliação | Roadmap técnico |
| V — Flexibilidade e adaptabilidade | Implementado | Modelo configurável via CONFIG_MGMT.md |
| VI — Gestão do ciclo de vida | Implementado | LIFECYCLE_MGMT.md |
| VII — APIs de interoperabilidade | Em avaliação | Roadmap técnico |
| VIII — Acesso de órgãos de controle | Implementado | Este documento §4.6 |

## 7. Rastreabilidade

| **Requisito** | **Vínculo** |
|---|---|
| Governança de IA (Art. 14) | Este documento |
| Comissão de IA e Telemedicina (Art. 14, par. único) | Seção 3 |
| Supervisão humana obrigatória (Art. 15, par. único) | Seção 4.3 |
| Medidas de governança (Anexo III, I-VIII) | Seção 6 |

---

**Assinaturas de Aprovação:**

| **Função** | **Nome** | **Data** | **Assinatura** |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
| Gerência Executiva | Arthur Teixeira de Almeida | | |
| Encarregado (DPO) | Frederico | | |
