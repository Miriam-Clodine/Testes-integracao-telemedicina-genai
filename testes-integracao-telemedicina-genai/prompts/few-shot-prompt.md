# Prompt Few-Shot para Geração de Testes de Integração

## Finalidade

Orientar a LLM por meio de exemplos completos, mantendo padronização sem limitar a diversidade dos riscos.

## Prompt pronto para execução

```text
Atue como engenheiro de testes sênior especializado em APIs REST e sistemas distribuídos.

CONTEXTO
Uma plataforma de telemedicina possui os serviços REST de Agendamento, Pacientes, Pagamentos, Prontuário e Notificações. O fluxo principal é:
1. POST /appointments recebe a solicitação.
2. GET /patients/{patientId} valida o paciente.
3. O horário é reservado.
4. POST /payments autoriza o pagamento.
5. POST /medical-records/appointments cria o registro da consulta.
6. POST /notifications agenda a confirmação.
7. O agendamento termina como CONFIRMED.

REGRAS
- Uma Idempotency-Key deve gerar no máximo um agendamento e uma cobrança.
- Um mesmo horário não pode ser confirmado para dois pacientes.
- O valor do pagamento deve coincidir com o valor calculado no agendamento.
- Falha de notificação não cancela uma consulta confirmada; deve gerar retry.
- Pagamento autorizado seguido de falha crítica deve gerar compensação ou revisão manual.
- Tokens inválidos ou sem escopo devem ser rejeitados.
- Não use dados pessoais reais.

FORMATO OBRIGATÓRIO DE CADA CASO
- ID
- Título
- Classificação: positivo ou negativo
- Categoria de risco
- Prioridade: alta, média ou baixa
- Objetivo
- Serviços envolvidos
- Pré-condições
- Dados de teste fictícios
- Requisição REST relevante
- Mock/falha a configurar
- Passos
- Resultado esperado
- Critérios de aceitação mensuráveis
- Evidências a coletar

EXEMPLO 1
ID: FS-EX-01
Título: Agendamento confirmado no fluxo principal
Classificação: positivo
Categoria de risco: fluxo funcional
Prioridade: alta
Objetivo: verificar a integração completa quando todas as dependências respondem com sucesso.
Serviços envolvidos: Agendamento, Pacientes, Pagamentos, Prontuário e Notificações.
Pré-condições: paciente ativo; profissional disponível; horário livre; serviços acessíveis.
Dados de teste: patientId=PAT-1001, professionalId=PRO-205, amount=180.00 BRL.
Requisição REST relevante: POST /appointments com Idempotency-Key única.
Mock/falha a configurar: nenhuma.
Passos: enviar a solicitação; validar chamadas aos serviços; consultar o estado final.
Resultado esperado: HTTP 201 e status CONFIRMED.
Critérios de aceitação: uma consulta criada; uma cobrança autorizada; um registro de prontuário criado; uma notificação enfileirada; correlationId presente em todos os logs.
Evidências a coletar: respostas HTTP, registros persistidos e logs correlacionados.

EXEMPLO 2
ID: FS-EX-02
Título: Reenvio da mesma solicitação com chave de idempotência
Classificação: negativo
Categoria de risco: duplicidade e idempotência
Prioridade: alta
Objetivo: garantir que retry do cliente não gere consulta nem cobrança duplicada.
Serviços envolvidos: Agendamento e Pagamentos.
Pré-condições: primeira requisição processada com sucesso.
Dados de teste: mesmo payload e mesma Idempotency-Key nas duas chamadas.
Requisição REST relevante: duas chamadas POST /appointments.
Mock/falha a configurar: simular perda da primeira resposta no cliente.
Passos: enviar a primeira chamada; descartar a resposta; reenviar a mesma chamada; consultar os registros.
Resultado esperado: a segunda chamada retorna o recurso já criado, sem novo processamento financeiro.
Critérios de aceitação: apenas um appointmentId; apenas uma cobrança; mesma resposta lógica para a chave; tentativa duplicada registrada sem dados sensíveis.
Evidências a coletar: contagem de registros, identificadores e logs.

EXEMPLO 3
ID: FS-EX-03
Título: Falha no Serviço de Notificações após confirmação
Classificação: negativo
Categoria de risco: falha parcial e retry
Prioridade: média
Objetivo: verificar que uma falha não crítica não desfaz a consulta confirmada.
Serviços envolvidos: Agendamento e Notificações.
Pré-condições: pagamento autorizado e prontuário criado.
Dados de teste: appointmentId=APT-9001.
Requisição REST relevante: POST /notifications.
Mock/falha a configurar: retornar HTTP 503.
Passos: concluir o fluxo até a notificação; aplicar a falha; consultar o agendamento e a fila de retry.
Resultado esperado: consulta CONFIRMED e notificação RETRY_PENDING.
Critérios de aceitação: consulta não cancelada; retry registrado; nenhuma notificação duplicada após recuperação.
Evidências a coletar: estado do agendamento, evento de retry e logs.

TAREFA
Gere 12 novos casos, sem repetir os exemplos. Distribua-os obrigatoriamente entre estas categorias:
1. validação de payload;
2. autenticação;
3. autorização;
4. contrato de API;
5. timeout;
6. indisponibilidade HTTP 5xx;
7. concorrência na reserva do horário;
8. divergência de valor;
9. resposta perdida após pagamento;
10. rate limit HTTP 429;
11. falha de compensação;
12. consistência entre Agendamento e Prontuário.

Use IDs FS-01 a FS-12. Não invente endpoints além dos informados; quando precisar de uma consulta de estado, descreva-a como verificação em banco, log ou ferramenta de observabilidade. Produza casos executáveis, com critérios objetivos. Ao final, apresente uma matriz resumida relacionando ID, categoria de risco, serviços e prioridade.
```

## Por que este prompt é mais robusto

- Define regras de negócio e invariantes antes da geração;
- Fornece três exemplos com níveis de complexidade diferentes;
- Exige diversidade por categoria, reduzindo repetição;
- Obriga evidências e critérios mensuráveis;
- Evita a invenção de endpoints;
- Solicita matriz final para facilitar revisão de cobertura.
