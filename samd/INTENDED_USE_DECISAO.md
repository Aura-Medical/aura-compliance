# A escolha da indicação de uso — duas versões, para decidir com o texto na mão

**Data:** 2026-09-12 · **Para:** RT + assessoria regulatória · **Decide:** se a RDC 657/2022 se aplica

> O `INTENDED_USE.md` rev. 4.0 afirma **as duas coisas** — "CDSS" (§1) e "informações de bem-estar
> geral" (§2). O **Art. 1º, § 2º, I** exclui bem-estar do escopo da RDC, e o **Art. 2º, IX** define
> bem-estar como o que **"não é destinado a prevenção, diagnóstico, tratamento, reabilitação ou
> anticoncepção"**. Uma frase anula a outra.
>
> Abaixo, as duas redações possíveis, escritas para colar. **Não escolhi por você** — a escolha
> define o produto, não o documento.

---

## Versão A — BEM-ESTAR (fora do escopo da RDC 657)

> A Aura Medical destina-se a **encorajar e manter hábitos saudáveis e o automonitoramento de
> bem-estar** por adultos. O software agrega dados laboratoriais, biométricos e de dispositivos
> vestíveis inseridos ou autorizados pelo próprio usuário, e os apresenta de forma organizada,
> com material educativo referenciado à literatura científica.
>
> O software **não se destina a prevenção, diagnóstico, tratamento, reabilitação ou
> anticoncepção**, não produz conclusão clínica sobre o estado de saúde do usuário, não sugere
> conduta e não substitui avaliação profissional. As faixas apresentadas são **referências
> educativas de literatura**, não limiares diagnósticos, e a comparação do usuário com elas não
> constitui avaliação de risco clínico.

**O que isso exige MUDAR no produto** (não só no papel):

- **"Pontos a melhorar" por domínio** deixa de ser classificação de risco e passa a ser leitura descritiva.
- **Carga alostática e PhenoAge** ou saem, ou perdem a moldura de risco — "idade biológica" com número é afirmação sobre estado de saúde.
- **Alvo de apoB ligado a risco cardiovascular** sai da voz do produto. É a afirmação mais claramente fora do carve-out.
- O **safety gate de crise** pode ficar (encaminhar a ajuda não é diagnosticar), mas a moldura muda.

**O que ganha:** nada a notificar na ANVISA · nenhum dossiê · nenhuma AFE · nenhuma inspeção ·
responde **"No"** ao formulário da Apple com coerência · caminho FDA fica simples (general wellness).

**O que perde:** é um produto diferente do que está construído, e diferente do modelo que o RT
descreveu (médico paga para ver o paciente inteiro).

---

## Versão B — CDSS / SaMD CLASSE II (dentro do escopo)

> A Aura Medical é um **software como dispositivo médico (SaMD)** destinado a apoiar a
> **prevenção** e o automonitoramento de fatores de risco cardiometabólico, de sono, de atividade
> física e de saúde mental em adultos. O software agrega dados laboratoriais, biométricos, de
> dispositivos vestíveis e de instrumentos clínicos validados, e os classifica contra limiares
> estabelecidos na literatura primária, sinalizando desvios que merecem atenção.
>
> A saída do software é **informação para apoiar decisão**, dirigida ao usuário e ao profissional
> de saúde que o acompanha. O software **não estabelece diagnóstico, não prescreve e não
> substitui avaliação médica** — toda conduta permanece com o profissional habilitado
> (CFM 2.454/2026). A associação clínica de cada limiar é referenciada à literatura primária e
> assinada pelo Responsável Técnico.

**O que isso exige** (e a maior parte já existe):

- Notificação Classe II na ANVISA — **sem aprovação prévia**, mas sujeita a inspeção (Art. 23).
- Dossiê técnico completo do Art. 11 → **8 itens ausentes e 8 parciais**, medidos em `DOSSIE_GAP_ART11.md`.
- AFE do fabricante · RT perante a ANVISA · vigilância pós-mercado (Art. 24).
- Peticionamento a cada nova funcionalidade clínica (Art. 16).
- Na Apple: responder **"Yes"** ao formulário de dispositivo médico regulado, com documentação.
- **FDA fica mais duro**: o carve-out de CDS do 21st Century Cures exige que um profissional possa
  revisar independentemente a base da recomendação — num app voltado ao paciente ele não se aplica.

**O que ganha:** o produto que está construído pode afirmar o que já afirma · o modelo de negócio
descrito (médico paga, vê o paciente inteiro) é coerente · a Classe II é notificação, não registro.

---

## O que eu observo, sem decidir

Três fatos do produto **como ele está hoje**, que o regulatório deve pesar:

1. O app **já classifica** — "Em dia / Vencendo / Vencido", "Pontos a melhorar" por domínio,
   faixa "Ótimo / Normal / Fora da faixa". Classificar estado contra limiar é o que a Versão A
   proíbe.
2. O `RISK_ANALYSIS.md` tem **10 perigos** cujo pior caso é dano clínico
   (HAZ-01 "status falsamente saudável para paciente clinicamente comprometido"). Um produto de
   bem-estar não teria esse perigo — ter a análise é reconhecer a natureza do produto.
3. O RT **já assina** limiar clínico (`CONFIG_MGMT.md` §4.2 exige validação dele para mudança em
   limiar, prompt e safety gate). Isso é governança de dispositivo, não de app de hábitos.

Nenhum dos três decide a questão sozinho — mas os três apontam na mesma direção, e é honesto dizer
isso em vez de apresentar as versões como equivalentes.
