# Registro de incidente — dados pessoais reais em repositórios de código (2026-09-25)

> Procedimento: [[BREACH_SOP]]. Rastreio de engenharia: ROADMAP `DIV-644`, `DIV-645`, `DIV-646` (repo de conhecimento).
> Nenhum dado pessoal é reproduzido neste registro.

## Fase 1 — Identificação

- **Detectado:** 25/09/2026, em revisões de código e numa varredura geral que elas provocaram (CPF válido por dígito
  verificador, com algarismos de qualquer escrita, fora dos CPFs sintéticos conhecidos, nos 5 repositórios da empresa).
- **O que foi achado:**
  1. o CPF real do próprio RT em fixtures de teste (backend 1 arquivo, iOS 5 arquivos, 2 prompts arquivados) — DIV-644;
  2. CPFs reais de 4 familiares do RT usados em cadastros de teste (dogfood), com nome e às vezes idade, sexo, data de
     nascimento e identificador de conta, em documentos do repositório de conhecimento (inclusive o ROADMAP), num comentário de
     migração do backend, e em 3 comentários/testes do iOS (primeiros nomes); o histórico de migrações do banco de produção
     guardava só o nome, numa linha — DIV-645;
  3. o primeiro nome de uma testadora em ~40 arquivos, junto de detalhes das respostas e dos dados dela, e o identificador de
     produção de uma resposta dela em 4 comentários do iOS — DIV-646.
- **Exposição:** os 3 repositórios de código são PRIVADOS (organização no GitHub); o histórico de migrações só é acessível a papéis
  administrativos do banco. Sem evidência de acesso indevido.
- **Classificação inicial proposta:** **Baixa/Média** — dado pessoal retido fora da finalidade (depuração), em ambiente
  interno e restrito. A confirmar pelo Líder de Incidente e pelo Encarregado.

## Fase 4 — Erradicação (para a frente; o histórico do git NÃO foi reescrito)

- Repositório de conhecimento: CPFs, nomes, idade, sexo e datas redigidos (commit `c9ed091`); varredura = 0.
- iOS: o CPF do RT trocado por um sintético (`dfd63d6`); nomes de familiares redigidos (`67be13e`); o identificador de
  produção da testadora retirado (`e282cc4`, `60f54d8`).
- Backend: o CPF do RT trocado por sintético (`d7b6069`); o comentário da migração com os dados do familiar redigido e uma
  migração nova (`20260925_09`) que corrige a linha do histórico de migrações sem reescrever o nome (em revisão; aplicação pela
  engenharia); um teste no portão de PHI do deploy que FALHA se um CPF real voltar a entrar no repositório.
- Pacotes locais de revisão e cópias de trabalho que guardavam os números foram apagados.

## Decisões pendentes (RT / Encarregado)

1. **Histórico do git:** reescrever os 3 repositórios para purgar os dados (quebra os identificadores de commit citados no
   ROADMAP e nas ADRs como trilha de auditoria, as tags do Xcode Cloud e os clones) ou mantê-los (repositórios privados,
   conserto para a frente feito, guarda contra recorrência ativa). Recomendação da engenharia: manter.
2. **Consentimento:** os familiares cujos CPFs reais foram usados em cadastros de teste (com consulta à Receita via API Brasil)
   consentiram?
3. **O nome da testadora:** pseudonimizar ("a testadora") no código, nos testes e nos documentos daqui para a frente
   (recomendado).
