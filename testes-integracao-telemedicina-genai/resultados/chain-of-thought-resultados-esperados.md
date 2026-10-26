# Resultados Esperados da Execução do Prompt Chain-of-Thought Orientado

## 1. Síntese de análise esperada

A LLM deve identificar que o Serviço de Agendamento funciona como coordenador do fluxo e que falhas antes do pagamento têm efeitos diferentes das falhas posteriores. Também deve reconhecer como invariantes: unicidade do horário, unicidade da cobrança, compatibilidade do valor, existência do paciente e convergência entre Agendamento e Prontuário.

## 2. Casos esperados

## CT-01 — Pagamento autorizado e falha na criação do prontuário

- **Hipótese de risco:** o sistema pode cobrar o paciente sem terminar a preparação da consulta.
- **Prioridade:** crítica
- **Falha:** `POST /medical-records/appointments` retorna HTTP 503.
- **Resultado por serviço:** Pagamentos mantém autorização identificável; Prontuário não cria registro; Agendamento inicia compensação ou revisão.
- **Estado final global:** não deve ser `CONFIRMED` enquanto a inconsistência persistir.
- **Critérios:** uma cobrança no máximo; alerta crítico; compensação ou revisão registrada; correlationId em todos os eventos.

## CT-02 — Pagamento autorizado, resposta perdida e retry

- **Hipótese de risco:** cobrança duplicada após falha de rede.
- **Prioridade:** crítica
- **Resultado final:** somente uma cobrança e um agendamento.
- **Critérios:** mesma chave idempotente; contagem financeira igual a um; reconciliação do status.

## CT-03 — Duas requisições concorrentes para o mesmo horário

- **Hipótese de risco:** condição de corrida.
- **Prioridade:** crítica
- **Resultado final:** somente uma consulta confirmada.
- **Critérios:** uma reserva; uma cobrança; segunda requisição rejeitada sem efeito colateral.

## CT-04 — Mensagem repetida após recuperação do cliente

- **Hipótese de risco:** duplicidade em retry do Front-end.
- **Prioridade:** alta
- **Resultado final:** recurso original retornado ou comportamento equivalente definido pelo contrato.
- **Critérios:** identificadores estáveis; nenhum registro duplicado.

## CT-05 — Cliente envia versão antiga do contrato de Agendamento

- **Hipótese de risco:** campo renomeado ou estrutura incompatível.
- **Prioridade:** alta
- **Resultado final:** requisição rejeitada sem chamadas a dependências.
- **Critérios:** código de erro contratual definido; mensagem útil; zero efeitos colaterais.

## CT-06 — Serviço de Pacientes devolve resposta incompatível

- **Hipótese de risco:** consumidor interpreta incorretamente paciente inativo como ativo.
- **Prioridade:** crítica
- **Resultado final:** fluxo interrompido com segurança.
- **Critérios:** nenhuma cobrança; nenhum prontuário; alerta de contrato.

## CT-07 — Timeout repetido no Serviço de Pagamentos

- **Hipótese de risco:** retries não controlados aumentam a indisponibilidade.
- **Prioridade:** alta
- **Resultado final:** política de retry/circuit breaker aplicada conforme configuração.
- **Critérios:** ausência de loop infinito; estado consistente; métricas e alerta disponíveis.

## CT-08 — Timeout em cascata originado no Serviço de Pacientes

- **Hipótese de risco:** uma dependência lenta consome todo o orçamento de resposta.
- **Prioridade:** alta
- **Resultado final:** pagamento e prontuário não são chamados.
- **Critérios:** falha controlada; timeout respeitado; rastreabilidade ponta a ponta.

## CT-09 — Token válido sem escopo para criação de consulta

- **Hipótese de risco:** violação de autorização entre módulos.
- **Prioridade:** crítica
- **Resultado final:** HTTP 403 e nenhum efeito colateral.
- **Critérios:** tentativa auditável; token e dados sensíveis mascarados.

## CT-10 — Dados clínicos ou financeiros expostos em logs

- **Hipótese de risco:** vazamento de informação durante falha de integração.
- **Prioridade:** crítica
- **Falha:** provocar erro de validação contendo payload sensível.
- **Resultado final:** logs registram contexto técnico sem conteúdo integral sensível.
- **Critérios:** ausência de número completo de cartão, token e conteúdo clínico; correlationId preservado.

## CT-11 — Rate limit durante notificação

- **Hipótese de risco:** retry agressivo gera duplicidade ou prolonga o bloqueio.
- **Prioridade:** média
- **Resultado final:** consulta confirmada e notificação pendente.
- **Critérios:** indicação de espera respeitada; retry controlado; uma notificação final.

## CT-12 — Reconciliação encontra pagamento autorizado e agendamento pendente

- **Hipótese de risco:** estados divergentes permanecem indefinidamente.
- **Prioridade:** crítica
- **Resultado final:** processo corrige o estado ou direciona para revisão manual.
- **Critérios:** divergência detectada; decisão registrada; nenhuma nova cobrança; resultado auditável.

## 3. Autoavaliação esperada

A resposta deve apresentar uma matriz de cobertura sem duplicidades relevantes, indicar premissas não especificadas, separar candidatos a smoke, regressão e resiliência, e identificar quais casos precisam de mocks, fault injection ou controle de concorrência para automação.
