# Resumo Geral de Evidência Clínica e Literatura Clínica Relevante

**ID:** EC-001 · **Rev.:** 1.0 · **Data:** 2026-09-12
**Produto:** Aura Medical · **Fabricante:** AURAMEDICAL SERVICOS MEDICOS LTDA (CNPJ 63.556.151/0001-40)
**Base:** RDC 657/2022, Art. 11, Capítulo 4 (Classe II)

> **Todos os números desta seção foram medidos no banco de produção em 2026-09-12** e são
> reproduzíveis pelas consultas indicadas. Nenhum é estimativa.

---

## 1. Natureza da evidência deste SaMD

A Aura Medical **não gera** conhecimento clínico novo: ela **aplica** conhecimento publicado.
A evidência que sustenta o produto é, portanto, de duas naturezas:

1. **Evidência da literatura** que embasa cada limiar e cada associação clínica afirmada.
2. **Evidência de que o software aplica essa literatura fielmente** — que é a rastreabilidade
   entre a afirmação exibida ao usuário e a fonte primária que a sustenta.

O item 2 é o que distingue este dossiê: a rastreabilidade **não é documental, é estrutural** —
está no dado, por linha, e é consultável a qualquer momento.

## 2. O acervo, medido

| | medido em 2026-09-12 |
|---|---|
| Documentos-fonte ingeridos | **102** |
| Trechos curados (chunks) | **6.475**, dos quais **6.411 servindo** e 64 depreciados |
| Conceitos clínicos (nós do grafo) | **2.161** |
| Associações clínicas (arestas) | **10.904** |
| **Associações ASSINADAS pelo RT** | **7.985** |
| — com trecho verbatim da fonte (`source_span`) | **6.798** (85,1%) |
| — com qualificador de escopo clínico (`condition`) | **4.929** (61,7%) |
| — com PMID da primária (`anchor_source_pmid`) | **499** |
| — com âncora quantitativa (HR/OR/LR, `anchor_value`) | **134** |

**Distribuição por grau de evidência** (`evidence_grade`) das 7.985 assinadas:

| grau | n |
|---|---|
| `meta_sr` (meta-análise / revisão sistemática) | 149 |
| `rct` (ensaio clínico randomizado) | 129 |
| `guideline` (diretriz de sociedade) | 533 |
| `cohort_obs` (coorte / observacional) | 423 |
| `mechanistic_expert` (mecanístico / opinião de especialista) | 1.884 |
| **sem grau atribuído** | **4.867** |

> ⚠️ **Declarado como lacuna, não omitido:** 4.867 das 7.985 associações assinadas (60,9%) ainda
> não têm `evidence_grade` preenchido. Elas **estão assinadas pelo RT e têm trecho verbatim**, mas
> o grau da primária não foi atribuído. Isto é anomalia de completude documental, não de segurança
> — está registrada em `RESIDUAL_ANOMALIES.md` (ANO-03) com análise de risco.

## 3. Os controles que garantem fidelidade à fonte

Estes controles são **estruturais** e verificáveis por consulta:

- **Trecho verbatim obrigatório.** Toda associação carrega o `source_span` — o texto literal da
  fonte que a sustenta. Associação sem trecho é rejeitada: há **39 linhas** com status
  `rejeitado_sem_span` no banco, que são a prova de que o portão funciona.
- **Refutação adversarial antes da assinatura.** Associações passam por refutadores independentes
  com instrução de derrubar; o default na dúvida é rejeitar. Há **17 linhas**
  `rejeitado_span_nao_sustenta` e **21** `rejeitado_rt_parecer` — o registro do que não passou.
- **Qualificador de escopo obrigatório** quando a afirmação é restrita a subgrupo. Doutrina escrita
  em `CLAUDE.md` §"Qualificador clínico ausente = afirmação para o PACIENTE ERRADO": aresta cujo
  escopo se perdeu não é imprecisa, é **falsa** para quem está fora do escopo.
- **Força de associação derivada da âncora**, nunca preenchida à mão — é coluna gerada no banco, não
  julgamento. Doutrina em `CLAUDE.md` §"Força de aresta = DERIVADA da âncora".
- **Veículo ≠ grau.** O grau de evidência é o da **primária** que sustenta a afirmação, nunca o do
  veículo que a trouxe (diretriz, livro, terciário).
- **O grau NÃO ranqueia o retrieval.** Ele é portão de ingestão e informa o tom da geração; a
  ordenação é por relevância. Doutrina em `CLAUDE.md` §"Grade NÃO ranqueia".

## 4. Literatura clínica relevante

As 102 fontes ingeridas cobrem os cinco domínios da indicação de uso. A lista nominal com
título, ano, veículo e domínio é gerada por:

```sql
SELECT doc_slug, source_year, domain, count(*) AS chunks
  FROM corpus_chunks c JOIN corpus_documents d USING (doc_id)
 WHERE c.status = 'approved' GROUP BY 1,2,3 ORDER BY 3,2 DESC;
```

> **A lista nominal NÃO é reproduzida aqui, e isso é decisão de doutrina, não descuido.**
> Martelo do RT de 2026-09-05: *"o acervo é servido em totalidade; a proveniência é do RT; ao
> consumidor só citação/autores"*. A lista de títulos é proveniência — pertence ao dossiê interno
> e à inspeção, não a documento que circula. Ela é **gerada sob demanda** pela consulta acima, e
> é essa geração que deve ser anexada numa inspeção, com a data da extração.

## 5. O que este documento NÃO afirma

- **Não há estudo clínico próprio.** Nenhum ensaio, nenhuma coorte, nenhuma validação clínica
  prospectiva do produto foi conduzida. O Art. 11 nota 2 diz que o Resumo de Evidência Clínica é
  *"aplicável apenas quando evidência clínica for exigida em decorrência de demonstração de
  segurança e desempenho"* — o caso aqui é de **evidência bibliográfica aplicada**, não gerada.
- **Não há validação clínica (utilidade clínica) medida** no sentido do Art. 2º, XII — medir se a
  saída produz impacto positivo mensurável na saúde. Isso exigiria estudo, e não foi feito.
- **A acurácia da classificação não foi medida contra padrão-ouro externo.** Os limiares vêm da
  literatura; a aplicação deles é determinística e testada, mas a concordância com julgamento
  clínico humano não foi quantificada.

Estas três ausências estão declaradas porque **declarar lacuna é diferente de ter lacuna
escondida** — e é a diferença que uma inspeção procura.

## 6. Assinatura

| Papel | Nome | Assinatura | Data |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
