# Prontidão para a notificação — o que está pronto, o que falta, e o que eu não escrevi

**Data:** 2026-09-12 · **Para:** RT · **Decisão aplicada:** Versão B — CDSS / SaMD Classe II

---

## ⚠️ A distinção que muda o plano

**O dossiê técnico NÃO é enviado à ANVISA.** O Art. 11 é literal: ele *"permanece em posse da
empresa detentora da notificação"*. O que se envia é o **formulário de petição de notificação**
(Art. 10), e a notificação **não depende de aprovação**.

Portanto há duas listas diferentes, e confundi-las atrasa por meses ou expõe por anos:

| | |
|---|---|
| **O que bloqueia NOTIFICAR** | AFE · formulário · dados da empresa |
| **O que bloqueia SOBREVIVER À INSPEÇÃO** | o dossiê completo (Art. 23: produto regularizado fica sujeito a auditoria e inspeção) |

Você pode notificar antes do dossiê estar completo. **O risco não é a recusa — é a inspeção.**

---

## 1. O que bloqueia a notificação (e nada disso sou eu)

1. **AFE na ANVISA** para o CNPJ 63.556.151/0001-40. As dispensas de alvará municipal e licença sanitária que estão na pasta societária **são outra coisa**.
2. **RT perante a ANVISA.** RT de fabricante de dispositivo não precisa ser médico; confirmar se o CRM atende ou se convém um RT técnico ao lado, com o senhor assinando a validação clínica.
3. **Formulário de petição** no portal, preenchido.
4. **Enquadramento** confirmado com a assessoria: Classe II pela RDC 751/2022 (o Art. 4º da 657 remete às regras gerais).

## 2. O que ficou PRONTO hoje

| documento | o que cobre |
|---|---|
| `INTENDED_USE.md` **rev. 5.0** | ⛔→✅ a contradição resolvida: CDSS/SaMD, com o que a indicação **não** autoriza |
| `CLINICAL_EVIDENCE.md` | Cap. 4 inteiro — evidência e literatura, com os números medidos no banco |
| `SECURITY_ARCH.md` | arquitetura de cibersegurança, incluindo o que foi achado aberto e corrigido |
| `RESIDUAL_ANOMALIES.md` | 6 anomalias com análise de risco, recortadas de 394 dívidas |
| `MANUFACTURING.md` | fabricante · unidade fabril · lista de componentes · firmware N/A · histórico de comercialização · fluxograma · descritivo da versão |
| `DOSSIE_GAP_ART11.md` | o levantamento que gerou tudo isto |

**Placar do Art. 11:** de 8 ausentes e 8 parciais, restam **3 ausentes** e **5 parciais**.

## 3. O que eu NÃO escrevi, e não vou escrever

Estes três exigem **execução**, não redação. Escrevê-los sem a execução seria inventar evidência
num dossiê regulatório — o que é pior do que a lacuna, porque a lacuna é honesta e a invenção é
falsificação.

| item | por que não | o que é preciso |
|---|---|---|
| **Relatório de usabilidade (IEC 62366-1)** | exige avaliação com usuários reais | protocolo de fatores humanos, sessões com participantes, achados e mitigações. Sem isso, o Art. 15 permite a alternativa: justificativa técnica + descritivo de ciclo de vida + relatório de risco — **mas isso é declarar que a norma não foi plenamente atendida** |
| **Testes de compatibilidade e interoperabilidade** | exige rodar os testes | matriz HealthKit · Garmin · Apple Watch · Supabase · Anthropic · Google, com resultado por combinação. Posso escrever a matriz e o método; os resultados têm de ser medidos |
| **Relatório de V&V** (já em `RMP.md` §4) | exige executar os 24 casos | execução real e registro |

**Também não posso**: coletar as assinaturas, negociar o DPA com a Anthropic, obter a AFE.

## 4. Duas coisas do produto que a decisão de hoje torna urgentes

**ANO-06 sobe de prioridade.** O eixo movimento serve **zero fabricado como medida** para usuário
de Apple Watch. Num produto de bem-estar isso é bug; num **SaMD que classifica**, é dado ausente
apresentado como dado medido — o mesmo padrão do HAZ-01. **É a anomalia que eu consertaria
primeiro**, antes de qualquer outra da lista.

**O formulário da Apple muda de resposta.** Com SaMD Classe II declarado, responder "No" ao
formulário de dispositivo médico regulado passa a ser inconsistente com o que a empresa declara à
ANVISA. Isso não impede o lançamento no Brasil — mas amarra a resposta do formulário, e amarra a
decisão sobre EUA/UE.

## 5. Assinatura

| Papel | Nome | Assinatura | Data |
|---|---|---|---|
| Responsável Técnico (RT) | Dr. Alexandre Teixeira de Almeida | | |
