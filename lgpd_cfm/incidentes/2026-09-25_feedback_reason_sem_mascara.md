# Registro de incidente — o motivo do feedback do chat saiu sem máscara (2026-09-25)

> Procedimento: [[BREACH_SOP]] (fases 1–5). Rastreio de engenharia: ROADMAP `DIV-642` (repo de conhecimento) e o ledger
> `.superpowers/sdd/2026-09-23-os-erros-do-teste-240/progress.md`. Registro aberto pela engenharia (agente CC) em 25/09;
> as decisões das fases 1 e 3 são do Líder de Incidente e do Encarregado.

## Fase 1 — Identificação

- **Detectado:** 25/09/2026 ~11:00 UTC, na revisão de código da DIV-634 (portão de PHI do deploy), que achou em
  `aura-backend/src/health/aura-plus/feedback.ts` o texto livre do feedback indo cru para fora do perímetro, contra o próprio
  comentário do código ("opaque IDs only").
- **O que é o dado:** o campo `reason` — texto livre de até 500 caracteres que a pessoa escreve ao avaliar uma resposta do chat
  Aura+ (`aura_chat_message_feedback.reason`). Pode conter dado de saúde. **O conteúdo não foi lido pela engenharia**
  (mínimo necessário); quem classifica o conteúdo é o RT ou o Encarregado, lendo no banco, dentro do perímetro.
- **Alcance medido (só contagens):** 14 feedbacks no total; **6 com `reason` não vazio, de 2 contas**, entre 07/05 e
  03/06/2026; nenhum depois disso. As 2 contas estão ativas e NÃO são do domínio da Aura.
- **Para onde saiu:**
  1. **Langfuse** (auto-hospedado, `langfuse.aura.med`) — como comentário de score no trace da resposta. **5** dos 6 tinham
     trace; os 5 scores existiam com o comentário.
  2. **E-mail de compliance** via Resend (operador externo) para o endereço `FEEDBACK_EMAIL_TO` — os **6** eram negativos, então
     cada um dispararia um e-mail com o texto no corpo, **se** a chave do Resend estava configurada em produção na época (o
     ambiente local não a tem; não verificável daqui).
- **Classificação inicial proposta:** **Média** — o texto foi para a infraestrutura própria (Langfuse) e para a caixa de
  compliance da própria empresa por um operador, não para terceiros não autorizados; sem evidência de acesso indevido. A
  confirmar pelo Líder de Incidente e pelo Encarregado (se o conteúdo tiver dado de saúde identificável, sobe para Alta).

## Fase 2 — Contenção

- A rota continua no ar até o conserto entrar (ver Fase 4); desde 03/06 nenhum feedback com texto foi enviado.

## Fase 3 — Notificação (DECISÃO DO ENCARREGADO)

- Avaliar se há "risco ou dano relevante" (LGPD Art. 48) para os 2 titulares. Insumos: o conteúdo dos 6 textos (ler no banco);
  se os e-mails saíram (painel do Resend, remetente `noreply@auramedical.com.br`, assunto `[Aura+] Feedback negativo`,
  maio–junho/2026); o Langfuse é da própria empresa.
- **Decisão (RT, 25/09/2026 ~21:15 UTC): NÃO notificar.** Os 2 titulares são o próprio RT e a outra pessoa que testa
  a plataforma, os dois cientes; o ambiente é de desenvolvimento, sem usuários externos. Nas palavras do RT: "estamos em dev mode e tudo que hoje possivelmente possa ser considerado vazado nao se preocupe, pois faremos um wipe no futuro antes do lancamento, todo mundo que esta na plataforma testando esta OK (eu e minha noiva apenas)".
  Os pendentes da Fase 4 (os e-mails da caixa de compliance; o armazenamento e os backups do Langfuse) entram no wipe
  pré-lançamento, que é item bloqueante do lançamento no ROADMAP (DIV-658).

## Fase 4 — Erradicação e recuperação

- **Langfuse:** os 5 scores com o texto cru foram **apagados** em 25/09 (~16:50 UTC) pela API do Langfuse; releitura: 0
  comentários restantes.
- **Código (commits no `aura-backend`, em revisão antes do deploy):** `7bb192f` — a máscara de PHI das spans sai para um módulo
  puro e o teste do portão passa a medir a função de PRODUÇÃO (antes testava uma cópia, e desligar a máscara real deixava o
  portão verde); `39f8e3f` — o `reason` vai ao Langfuse pela mesma máscara e **sai do corpo do e-mail** (o e-mail diz só se o
  motivo foi informado); testes no portão de PHI do deploy com o middleware de autenticação real provam: comentário mascarado,
  e-mail e log sem nenhum trecho do texto, e posse (mensagem de outra conta → 403, inexistente → 404, sem escrita, sem score,
  sem e-mail).
- **Pendente:** apagar da caixa de compliance os e-mails de maio–junho, se existirem e tiverem dado de saúde (Líder de Incidente).
- **Pendente (achado da revisão, 25/09):** a exclusão no Langfuse foi feita e conferida só pela API. O Langfuse é
  auto-hospedado: conferir se o armazenamento de eventos (blob/S3) e os backups dele guardam cópia dos 5 comentários, e qual é a
  retenção. Se guardarem, apagar ou deixar expirar, e registrar aqui.
- **Feito 25/09 ~17:35 UTC:** o conserto está no ar (`aura-backend` `566e115`, deploy da Render).

## Fase 5 — Pós-incidente

- **Causa-raiz:** a máscara de PHI cobria só as spans do OTel; o score do Langfuse e o e-mail eram caminhos à parte, e nenhum
  teste media a fronteira (o do portão testava uma cópia da máscara).
- **Controles novos:** o portão de PHI do deploy (DIV-634) com a máscara de produção, o `feedback` e a posse dentro dele.
- **A avaliar:** DPA com o Resend (ver [[BAA_STATUS]]); entrada no `RISK_ANALYSIS.md` se o Encarregado classificar como hazard
  novo.
