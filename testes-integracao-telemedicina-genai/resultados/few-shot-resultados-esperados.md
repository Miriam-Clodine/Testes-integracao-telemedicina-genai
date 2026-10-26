# Resultados Esperados da Execução do Prompt Few-Shot

## Finalidade

Este documento representa o tipo de resultado útil esperado. A resposta real da LLM pode variar, mas deve manter o mesmo nível de precisão, diversidade e testabilidade.

## FS-01 — Payload sem `patientId`

- **Classificação:** negativo
- **Categoria:** validação de payload
- **Prioridade:** alta
- **Objetivo:** impedir processamento de consulta sem identificação do paciente.
- **Serviços:** Agendamento
- **Dados:** payload válido, exceto pela ausência de `patientId`.
- **Falha configurada:** nenhuma.
- **Passos:** enviar `POST /appointments`; capturar resposta; verificar persistência.
- **Resultado esperado:** HTTP 400; nenhuma chamada aos demais serviços; nenhum agendamento criado.
- **Critérios de aceitação:** resposta identifica o campo obrigatório; zero registros novos; logs sem dados sensíveis.
- **Evidências:** resposta HTTP, contagem de registros e trace da requisição.

## FS-02 — Token expirado

- **Classificação:** negativo
- **Categoria:** autenticação
- **Prioridade:** alta
- **Resultado esperado:** HTTP 401 antes de qualquer operação de negócio.
- **Critérios de aceitação:** nenhum agendamento, pagamento ou prontuário; tentativa registrada com correlationId; token não exposto no log.

## FS-03 — Usuário autenticado sem permissão de agendamento

- **Classificação:** negativo
- **Categoria:** autorização
- **Prioridade:** alta
- **Resultado esperado:** HTTP 403.
- **Critérios de aceitação:** requisição autenticada, porém bloqueada por escopo; nenhum efeito colateral.

## FS-04 — Campo obrigatório removido do contrato de Pacientes

- **Classificação:** negativo
- **Categoria:** contrato de API
- **Prioridade:** alta
- **Falha configurada:** resposta de `GET /patients/{id}` sem o campo que informa a situação do paciente.
- **Resultado esperado:** fluxo interrompido de modo controlado; agendamento não confirmado.
- **Critérios de aceitação:** incompatibilidade detectada; nenhuma cobrança; alerta técnico emitido.

## FS-05 — Timeout ao validar paciente

- **Classificação:** negativo
- **Categoria:** timeout
- **Prioridade:** alta
- **Resultado esperado:** serviço não avança para pagamento.
- **Critérios de aceitação:** estado não ultrapassa `PENDING`; timeout observável; nenhuma cobrança ou criação de prontuário.

## FS-06 — Pagamentos retorna HTTP 503

- **Classificação:** negativo
- **Categoria:** indisponibilidade
- **Prioridade:** alta
- **Resultado esperado:** consulta não confirmada e horário liberado ou marcado para compensação conforme a regra definida.
- **Critérios de aceitação:** nenhuma cobrança; estado final consistente; evento de falha registrado.

## FS-07 — Dois pacientes tentam reservar o mesmo horário

- **Classificação:** negativo
- **Categoria:** concorrência
- **Prioridade:** alta
- **Passos:** enviar duas requisições simultâneas para o mesmo profissional e horário.
- **Resultado esperado:** somente uma consulta confirmada.
- **Critérios de aceitação:** um único vencedor; a outra solicitação recebe conflito compatível com o contrato; nenhuma cobrança do pedido rejeitado.

## FS-08 — Valor do pagamento diverge do valor do agendamento

- **Classificação:** negativo
- **Categoria:** integridade financeira
- **Prioridade:** alta
- **Dados:** agendamento em BRL 180,00 e solicitação de pagamento em BRL 150,00.
- **Resultado esperado:** transação bloqueada.
- **Critérios de aceitação:** nenhuma cobrança; agendamento não confirmado; divergência registrada sem expor dados financeiros completos.

## FS-09 — Resposta de autorização do pagamento é perdida

- **Classificação:** negativo
- **Categoria:** idempotência e reconciliação
- **Prioridade:** alta
- **Resultado esperado:** retry não gera segunda cobrança.
- **Critérios de aceitação:** uma autorização financeira; um agendamento; estado reconciliado; tentativa duplicada rastreável.

## FS-10 — Rate limit no Serviço de Notificações

- **Classificação:** negativo
- **Categoria:** rate limit
- **Prioridade:** média
- **Falha configurada:** HTTP 429 com orientação de espera definida pelo contrato.
- **Resultado esperado:** consulta continua confirmada; notificação fica pendente de retry.
- **Critérios de aceitação:** política de espera respeitada; sem envio duplicado; evento observável.

## FS-11 — Falha na compensação após pagamento autorizado

- **Classificação:** negativo
- **Categoria:** compensação
- **Prioridade:** crítica
- **Cenário:** criação do prontuário falha e a reversão financeira também falha.
- **Resultado esperado:** estado `MANUAL_REVIEW` ou equivalente definido; alerta crítico.
- **Critérios de aceitação:** caso não termina como sucesso; ocorrência rastreável; nenhuma repetição automática ilimitada.

## FS-12 — Agendamento confirmado sem registro no Prontuário

- **Classificação:** negativo
- **Categoria:** consistência distribuída
- **Prioridade:** crítica
- **Resultado esperado:** divergência detectada e processo de reconciliação acionado.
- **Critérios de aceitação:** alerta gerado; estado global não apresentado como totalmente concluído antes da correção; vínculo criado uma única vez após recuperação.

## Cobertura esperada

O conjunto deve cobrir validação, autenticação, autorização, contrato, timeout, indisponibilidade, concorrência, integridade financeira, idempotência, rate limit, compensação e reconciliação.
