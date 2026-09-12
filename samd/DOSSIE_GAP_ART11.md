# Levantamento do dossiê contra o Art. 11 da RDC 657/2022 — Classe II

**Data:** 2026-09-12 · **Produto:** Aura Medical · **Fabricante:** AURAMEDICAL SERVICOS MEDICOS LTDA (CNPJ 63.556.151/0001-40)

> Método: cada linha da tabela do **Art. 11**, coluna **Classe II**, conferida contra os
> 10 documentos de `samd/` por leitura e busca textual. Onde não achei, está escrito
> **não encontrado** — não inferido a partir de documento vizinho.
>
> O dossiê **permanece em posse da empresa** (Art. 11, caput). Ele não é enviado na
> notificação, mas é o que a ANVISA pede numa inspeção — e o Art. 23 é explícito:
> produto regularizado fica sujeito a auditoria e inspeção.

---

## ⛔ O bloqueio que precede a tabela

**A indicação de uso se contradiz, e isso decide se a tabela inteira se aplica.**

O `INTENDED_USE.md` afirma as duas coisas:

- §1: *"funciona como um sistema de suporte à decisão clínica (CDSS)"*
- §2: *"destina-se a fornecer informações de bem-estar geral e automonitorização para adultos saudáveis"*

O **Art. 1º, § 2º, I** exclui do escopo da RDC os *"softwares para bem-estar"*, e o **Art. 2º, IX**
os define como os destinados a manter saúde e estilo de vida saudável **"que não são destinados a
prevenção, diagnóstico, tratamento, reabilitação ou anticoncepção"**.

Ou seja: **se for bem-estar, a RDC 657 não se aplica e não existe Classe II a notificar.** Se for
CDSS com finalidade de prevenção, aplica-se e a Classe II faz sentido. Uma frase anula a outra, e
notificar com essa ambiguidade é fragilidade na primeira pergunta que a fiscalização fizer.

**Nada abaixo deve ser executado antes desta escolha** — ela muda o que cada item precisa dizer.

---

## Capítulo 1 — Administrativo

| Item (Art. 11) | Estado | Onde / o que falta |
|---|---|---|
| Informações Administrativas e Técnicas | ⚠️ **a refazer** | Formulário do portal da ANVISA, com o CNPJ **novo**. Depende de AFE e da definição do RT perante a ANVISA. |
| Lista dos dispositivos (Modelos/Componentes/Variantes) | 🟡 **parcial** | `RMP.md` §2 identifica o produto; falta a lista formal de variantes (app iOS · backend · cérebro) como componentes. |

## Capítulo 2 — Descrição e finalidade

| Item | Estado | Onde / o que falta |
|---|---|---|
| Descrição Detalhada do Software e Fundamentos de Funcionamento | ✅ | `INTENDED_USE.md` §1 + `SDP.md` §2 |
| Finalidade Pretendida (uso · propósito · usuário · indicação) | ⛔ **contraditória** | `INTENDED_USE.md` §2 — ver o bloqueio acima |
| Ambiente / Contexto de Uso Pretendido | ✅ | `IFU.md` §5 + `INTENDED_USE.md` §3 |
| Contraindicações de Uso | ✅ | `IFU.md` §6 + `INTENDED_USE.md` §2.2 |
| **Histórico Global de Comercialização** *(só Classe II)* | ❌ **não encontrado** | Produto nunca comercializado — mas isso **é** o histórico, e precisa estar escrito. ~meia página. |

## Capítulo 3 — Engenharia, risco e segurança

| Item | Estado | Onde / o que falta |
|---|---|---|
| Gerenciamento de Risco | ✅ **forte** | `RISK_ANALYSIS.md` (FMEA, 10 perigos) + `RISK_REPORT.md` (ISO 14971 §7) |
| **Lista dos Requisitos Essenciais de Segurança e Desempenho** *(só II)* | 🟡 **citada, não listada** | `DECLARATION.md` menciona; falta a **lista** item a item contra a RDC 546/2021 (o Art. 17 remete a ela). |
| Lista de Normas Técnicas | ✅ | `DECLARATION.md` — IEC 62304, IEC 62366-1, ISO 14971 (as três do Art. 13) |
| Descrição do Firmware | ➖ **não se aplica** | SaMD puro, sem dispositivo embarcado (Art. 2º, VIII). **Precisa estar escrito como N/A**, não ausente. |
| **Plano de desenvolvimento e de manutenção** *(só II)* | 🟡 **metade** | `SDP.md` cobre desenvolvimento; o plano de **manutenção** (pós-release) não existe. |
| Arquitetura de Software | 🟡 | `SDP.md` + `CONFIG_MGMT.md` §2 descrevem repositórios; falta diagrama de arquitetura como artefato. |
| Testes de compatibilidade e interoperabilidade | ❌ **não encontrado** | HealthKit, Garmin, Apple Watch, Supabase, Anthropic, Google — nada documentado como teste. |
| Lista de anomalias residuais (erros e defeitos conhecidos) | ❌ **não encontrado** | ⚠️ E isto **existe de fato**: `docs/dividas/REGISTRO.md` tem 394 dívidas abertas. É a fonte; falta o recorte das que são anomalia residual **com análise de risco**. |
| **Documento de Rastreabilidade** *(só II)* | ✅ **forte** | `TRACEABILITY.md`, 576 linhas |
| Histórico de revisão com descrição das mudanças | 🟡 | `CONFIG_MGMT.md` §8 declara o git como registro de fabricação; falta o histórico **consolidado** por versão. |
| Descritivos das versões (incluindo componentes) | 🟡 | `CONFIG_MGMT.md` §3 define o esquema; falta o descritivo da versão 1.0.0 emitida. |
| Arquitetura de cibersegurança | ❌ **não encontrado** | Só citada em `TRACEABILITY.md`. E há material medido para escrevê-la: JWT, RLS, AES-GCM nos tokens Garmin, App Attest, scrub de PII antes do cérebro, e as correções de 11/09. |
| **Declaração de conformidade com normas (Art. 13-15)** *(só II)* | ✅ | `DECLARATION.md` |
| Usabilidade / Fatores Humanos | ❌ **não encontrado** | IEC 62366-1 é citada na declaração, mas **não há relatório**. O Art. 15 exige relatório de usabilidade se a norma não for plenamente atendida. |

## Capítulo 4 — Evidência clínica

| Item | Estado | Onde / o que falta |
|---|---|---|
| Resumo Geral de Evidência Clínica | ❌ **não encontrado** | ⚠️ Paradoxo: o acervo clínico é o **ativo central** do produto (corpus curado, grafo, primárias com PMID, `evidence_grade`, assinatura do RT) e **nada disso está no dossiê**. É o item com maior material disponível e zero documento. |
| **Literatura Clínica Relevante** *(só II)* | ❌ **não encontrado** | Idem. O `_shared/` e os dossiês clínicos do repo de conhecimento são a matéria-prima. |

## Capítulo 5 — Rotulagem

| Item | Estado | Onde / o que falta |
|---|---|---|
| Rotulagem do Produto | 🟡 | `IFU.md` §11 tem informações regulatórias; falta o rótulo formal conforme Art. 7º/8º (o Art. 8º §1º dispensa rótulo **físico** para distribuição virtual, mas exige as informações acessíveis no software). |
| Instruções de Uso / Manual do Usuário | ✅ **forte** | `IFU.md`, 156 linhas |

## Capítulo 6 — Fabricação

| Item | Estado | Onde / o que falta |
|---|---|---|
| Informações Gerais de Fabricação (endereços das unidades fabris) | ❌ **não encontrado** | Endereço agora existe (Pinheiros/SP), mas não está no dossiê como unidade fabril. |
| Processo de Fabricação (fluxograma) | 🟡 | `CONFIG_MGMT.md` §5-§6 descreve build e release em texto; falta o **fluxograma**. |
| Informações de Projeto e Desenvolvimento | ✅ | `SDP.md` |

---

## Placar

| | itens |
|---|---|
| ✅ pronto | **7** |
| 🟡 parcial (existe, incompleto) | **8** |
| ❌ não encontrado | **8** |
| ➖ não se aplica (a declarar) | **1** |
| ⛔ bloqueado por decisão | **1** — a indicação de uso |

## O que falta, em ordem de esforço

1. **Decidir CDSS vs bem-estar** — trava tudo. Sem custo de escrita, com custo de decisão.
2. **Itens de meia página cada** (4): histórico global de comercialização · firmware como N/A · informações de fabricação · descritivo da versão 1.0.0.
3. **Itens que já têm a matéria-prima e só precisam ser escritos** (3): arquitetura de cibersegurança · lista de anomalias residuais (recorte do `REGISTRO.md` com análise de risco) · fluxograma de fabricação.
4. **Itens que exigem trabalho novo** (4): relatório de usabilidade (IEC 62366-1) · testes de compatibilidade e interoperabilidade · resumo de evidência clínica · literatura clínica relevante.
5. **Itens que dependem de terceiros** (3, já no `RMP.md` §4): V&V com execução real dos 24 casos · assinaturas · DPA com a Anthropic.

## Fora do dossiê, mas na mesma decisão

- **AFE da ANVISA** para o fabricante. As dispensas na pasta `CREMESP/` são de alvará municipal e licença sanitária — outra coisa.
- **RT perante a ANVISA**: o RT de fabricante de dispositivo não precisa ser médico; confirmar se o CRM atende ou se convém um RT técnico ao lado.
- **Conta da App Store** ainda no CNPJ da empresa de tecnologia (`CONFIG_MGMT.md` §7).
- **Consentimento v1.2**: controlador novo + os dois erros de fato (modelo e armazenamento).
