# Prompt Chain-of-Thought Orientado para Testes de Integração

## Finalidade

Induzir uma análise sistemática das dependências e estados do fluxo antes da geração dos casos, sem depender de uma resposta vaga ou de raciocínio interno irrestrito.

## Prompt pronto para execução

```text
Atue como arquiteto de testes especializado em APIs REST, consistência distribuída e resiliência.

CONTEXTO
Uma plataforma de telemedicina integra os serviços de Agendamento, Pacientes, Pagamentos, Prontuário e Notificações.
Fluxo:
1. POST /appointments recebe a solicitação.
2. GET /patients/{patientId} valida o paciente.
3. O horário é reservado.
4. POST /payments autoriza o pagamento.
5. POST /medical-records/appointments cria o registro.
6. POST /notifications agenda a confirmação.
7. O agendamento termina como CONFIRMED.

INVARIANTES
- Um horário só pode ter uma consulta confirmada.
- Uma Idempotency-Key gera no máximo um agendamento e uma cobrança.
- O valor cobrado deve coincidir com o valor do agendamento.
- Paciente inexistente ou inativo não pode originar prontuário.
- Falha de notificação não cancela consulta confirmada.
- Falha posterior ao pagamento precisa de compensação ou revisão manual.
- Tokens inválidos ou sem escopo adequado devem ser rejeitados.
- Logs não devem expor dados clínicos ou financeiros completos.

ETAPA A — ANÁLISE ESTRUTURADA
Antes dos casos de teste, apresente somente uma síntese objetiva com:
1. mapa de dependências entre serviços;
2. estados e transições críticas do agendamento;
3. falhas possíveis antes, durante e depois de cada chamada REST;
4. inconsistências possíveis entre estados dos serviços;
5. mecanismos esperados de proteção: timeout, retry com backoff, idempotência, compensação, reconciliação e circuit breaker;
6. contratos e códigos HTTP que precisam ser validados.

Não exponha raciocínio interno detalhado. Entregue apenas conclusões, riscos e justificativas verificáveis.

ETAPA B — GERAÇÃO DOS CASOS
Com base na análise, gere 12 casos de integração, usando IDs CT-01 a CT-12. Inclua pelo menos:
- 2 falhas parciais após pagamento autorizado;
- 2 cenários de concorrência ou duplicidade;
- 2 cenários de quebra/evolução de contrato;
- 2 cenários de timeout, retry ou circuit breaker;
- 1 cenário de autenticação/autorização;
- 1 cenário de privacidade em logs;
- 1 cenário de rate limit;
- 1 cenário de reconciliação de estados.

FORMATO OBRIGATÓRIO
- ID
- Título
- Hipótese de risco
- Classificação: positivo ou negativo
- Prioridade
- Serviços e estados envolvidos
- Pré-condições
- Dados fictícios
- Falha/mocks a configurar
- Sequência de chamadas REST
- Resultado esperado por serviço
- Estado final global
- Critérios de aceitação mensuráveis
- Evidências a coletar
- Estratégia de recuperação esperada

RESTRIÇÕES
- Não invente endpoints além dos informados.
- Não utilize dados pessoais reais.
- Não aceite “erro tratado corretamente” como critério; indique código, estado, quantidade de registros, alerta ou evento verificável.
- Evite casos equivalentes com nomes diferentes.
- Quando houver retry, informe que a política exata deve seguir a configuração do sistema, sem inventar números.
- Se uma regra estiver indefinida, registre-a como premissa a validar.

ETAPA C — AUTOAVALIAÇÃO
Ao final, produza:
1. matriz de cobertura por categoria de risco;
2. lista de lacunas e premissas;
3. indicação de casos candidatos à automação;
4. verificação de duplicidade entre os casos;
5. classificação dos casos em smoke, regressão ou resiliência.
```

## Por que este prompt é mais robusto

- Transforma o raciocínio em artefatos auditáveis, como mapa de dependências e matriz de cobertura;
- Explora estados intermediários e falhas parciais;
- Exige resultado por serviço e estado global;
- Inclui mecanismos de recuperação e observabilidade;
- Solicita autoavaliação para identificar lacunas e redundâncias;
- Impede critérios vagos ou endpoints inventados.
