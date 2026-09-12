# Informações de Fabricação, Comercialização e Versões

**ID:** FAB-001 · **Rev.:** 1.0 · **Data:** 2026-09-12
**Base:** RDC 657/2022, Art. 11 — Capítulos 1, 2 e 6 · **Produto:** Aura Medical

---

## 1. Identificação do fabricante (Cap. 6)

| Campo | Informação |
|---|---|
| Razão Social | **AURAMEDICAL SERVICOS MEDICOS LTDA** |
| CNPJ | 63.556.151/0001-40 |
| Natureza Jurídica | 206-2 — Sociedade Empresária Limitada · Porte **ME** |
| Endereço (sede e **única unidade fabril**) | R. Pais Leme, 215, Conj. 1713 — Pinheiros, São Paulo/SP, CEP 05.424-150 |
| CNAE principal | 86.30-5-03 — Atividade médica ambulatorial restrita a consultas |
| CNAE secundários | 62.02-3-00 e 62.03-1-00 (**desenvolvimento e licenciamento de software**) · 63.11-9-00 · 63.19-4-00 |
| Situação cadastral | ATIVA desde 06/11/2025 |
| Responsável Técnico | Dr. Alexandre Teixeira de Almeida |

> **Não há unidade fabril física além da sede.** A "fabricação" de SaMD é desenvolvimento,
> integração e liberação de software, executada remotamente pela equipe e versionada nos
> repositórios listados em `CONFIG_MGMT.md` §2. A infraestrutura de execução (provedores de
> nuvem) está em `SBOM.md` §5 — ela hospeda o produto, não o fabrica.
>
> ⚠️ **AFE:** a Autorização de Funcionamento de Empresa perante a ANVISA **é requisito separado**
> e não está coberta por este documento. As dispensas de alvará municipal e de licença sanitária
> que constam na documentação societária são de outra natureza e **não substituem a AFE**.

## 2. Lista de dispositivos, modelos e componentes (Cap. 1)

Um único produto, sem variantes comerciais. Componentes:

| Componente | Natureza | Papel |
|---|---|---|
| **Aura Medical (app iOS)** | binário distribuído pela App Store | interface com o usuário; classificação determinística local |
| **Aura backend** | serviço | orquestração, persistência, portões de segurança e auditoria |
| **Cérebro (retrieval clínico)** | serviço | acervo curado e grafo; **não recebe PII** |

Requisitos mínimos de hardware e sistema (Art. 17, V): iPhone com **iOS 18.0** ou superior.
Sem uso de recursos de hardware não presentes no mínimo declarado.

## 3. Descrição de firmware (Cap. 3)

**NÃO SE APLICA.** A Aura Medical é SaMD puro, conforme o Art. 2º, VII — software que atende à
definição de dispositivo médico e **não faz parte do hardware** de um dispositivo. Não há software
embarcado no sentido do Art. 2º, VIII: o produto executa em dispositivo de propósito geral
(smartphone), e o Art. 2º, VIII exclui expressamente esse caso.

Registrado como "não se aplica" e não omitido, porque ausência sem declaração é indistinguível de
esquecimento.

## 4. Histórico Global de Comercialização (Cap. 2 — exigido na Classe II)

**O produto nunca foi comercializado, em nenhum país.**

| | |
|---|---|
| Países onde está regularizado | **nenhum** |
| Países onde é comercializado | **nenhum** |
| Unidades distribuídas | **zero** |
| Distribuição até hoje | exclusivamente **TestFlight interno**, restrito a testadores nomeados |
| Eventos adversos reportados | **nenhum** |
| Recalls, ações de campo, suspensões | **nenhum** |

A ausência de histórico **é** o histórico, e está declarada como tal. Quando houver
comercialização, este item passa a exigir manutenção conforme o Art. 21.

## 5. Processo de fabricação (Cap. 6 — fluxograma)

```
  requisito clínico (RT)
        │
        ▼
  especificação  ──────────────►  TRACEABILITY.md (REQ ↔ teste ↔ risco)
        │
        ▼
  implementação em branch
        │
        ▼
  revisão  ── revisor ≠ autor ──►  obrigatória em código clínico e de segurança
        │                          (CONFIG_MGMT.md §4.2)
        ▼
  verificação automatizada  ────►  testes unitários; build verde é condição de merge
        │
        ▼
  VALIDAÇÃO CLÍNICA (RT)  ──────►  obrigatória para limiar, prompt e portão de segurança.
        │                          É o portão que distingue mudança de software de
        │                          mudança de dispositivo.
        ▼
  merge em `main`  ─────────────►  git = registro de fabricação (RDC 665/2022)
        │
        ▼
  build de release  ────────────►  número monotônico; artefato assinado
        │
        ▼
  distribuição (App Store)
        │
        ▼
  pós-mercado  ──────────────────►  RDC 657 Art. 24: monitoramento, notificação de
                                    evento adverso, queixa técnica e ação de campo
```

Detalhe de cada etapa em `CONFIG_MGMT.md` §4-§6 e `SDP.md` §3-§4.

## 6. Descritivo da versão emitida (Cap. 3)

| Campo | Valor |
|---|---|
| Versão do produto | **1.0.0** |
| Estado regulatório | **não regularizado** — nunca distribuído fora de teste interno |
| Distribuição | TestFlight interno |
| Modelo de IA em uso | `claude-sonnet-4-6` (conversa) e `claude-haiku-4-5-20251001` (processos de fundo), registrados por turno em `ai_audit_log.model` (CFM 2.454/2026 Art. 9º) |
| Componentes de terceiros | `SBOM.md` |

> A rastreabilidade **exata** de qual modelo executou cada turno é por linha de auditoria, não por
> declaração de versão — o modelo pode mudar sem mudar a versão do app, e foi por isso que a
> coluna existe. O histórico real mostra a troca: Haiku 4.5 até 2026-05-26, Sonnet 4.6 desde
> 2026-05-27.

## 7. Assinatura

| Papel | Nome | Assinatura | Data |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
