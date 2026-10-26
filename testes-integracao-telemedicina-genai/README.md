# Testes de Integração em Serviços REST com Apoio de IA Generativa

## Autora

**Miriam Felix**

## Contexto acadêmico

Atividade desenvolvida no módulo de Testes da Pós-Graduação em Engenharia de Software Apoiada por IA Generativa.

## 1. Objetivo

Este repositório documenta um experimento de uso de modelos de linguagem de grande porte (LLMs) para apoiar o planejamento de testes de integração. Foram aplicadas duas técnicas de engenharia de prompts:

1. **Few-Shot Prompting**, para orientar a LLM por meio de exemplos representativos e de um formato de saída previamente definido;
2. **Chain-of-Thought Prompting orientado**, para induzir uma análise estruturada das dependências, riscos e transições de estado antes da geração dos casos de teste.

O objetivo é avaliar como essas estratégias influenciam a qualidade, a diversidade, a rastreabilidade e a utilidade dos casos de teste gerados.

> A LLM atua como ferramenta de apoio. Os casos produzidos devem ser revisados por uma pessoa responsável pelos testes antes da automação ou execução em ambiente real.

## 2. Cenário de integração

O cenário escolhido é uma **plataforma de telemedicina baseada em serviços REST**, contexto atual que envolve comunicação entre módulos, dados sensíveis, operações financeiras e dependências distribuídas.

### Serviços envolvidos

- **Front-end Web/Mobile:** coleta os dados do paciente e inicia o agendamento;
- **Serviço de Agendamento:** valida disponibilidade e coordena o fluxo da consulta;
- **Serviço de Pacientes:** valida a existência e a situação cadastral do paciente;
- **Serviço de Pagamentos:** autoriza e registra a cobrança;
- **Serviço de Prontuário:** cria o registro da consulta no prontuário eletrônico;
- **Serviço de Notificações:** envia a confirmação por e-mail, SMS ou push.

### Fluxo principal

1. O paciente seleciona profissional, data e horário.
2. O Front-end envia `POST /appointments` ao Serviço de Agendamento.
3. O Serviço de Agendamento consulta `GET /patients/{patientId}`.
4. O Serviço de Agendamento confirma a disponibilidade do horário.
5. O Serviço de Pagamentos recebe `POST /payments`.
6. Após a autorização, o Serviço de Prontuário recebe `POST /medical-records/appointments`.
7. O Serviço de Notificações recebe `POST /notifications`.
8. O agendamento retorna como `CONFIRMED`.

### Estados principais do agendamento

`PENDING` → `PAYMENT_AUTHORIZED` → `RECORD_CREATED` → `CONFIRMED`

Em caso de falha, o fluxo pode assumir estados como `PAYMENT_FAILED`, `COMPENSATION_PENDING`, `CANCELLED` ou `MANUAL_REVIEW`.

Mais detalhes estão em [`cenario-integracao.md`](cenario-integracao.md).

## 3. Estrutura do repositório

```text
testes-integracao-telemedicina-genai/
├── README.md
├── cenario-integracao.md
├── .gitignore
├── prompts/
│   ├── few-shot-prompt.md
│   └── chain-of-thought-prompt.md
├── resultados/
│   ├── few-shot-resultados-esperados.md
│   └── chain-of-thought-resultados-esperados.md
└── evidencias/
    └── orientacoes-registro-execucao.md
```

## 4. Metodologia

Os dois prompts utilizam o mesmo cenário e exigem que a LLM produza casos com:

- identificador único;
- classificação positiva ou negativa;
- objetivo;
- serviços envolvidos;
- pré-condições;
- dados de teste;
- requisições e respostas REST relevantes;
- passos de execução;
- resultado esperado;
- critérios de aceitação verificáveis;
- evidências a coletar;
- prioridade e tipo de risco coberto.

Os resultados devem ser avaliados segundo cinco critérios:

1. **Correção:** coerência entre estímulo, comportamento esperado e regras do fluxo;
2. **Cobertura:** presença de fluxo principal, erros, limites, segurança, contrato e resiliência;
3. **Diversidade:** variação real entre os riscos cobertos, evitando casos repetidos com pequenas mudanças;
4. **Testabilidade:** critérios de aceitação objetivos e verificáveis;
5. **Rastreabilidade:** identificação dos serviços, endpoints, estados e evidências relacionados.

## 5. Como o Few-Shot influenciou os resultados

O prompt Few-Shot apresenta exemplos completos antes de solicitar novos casos. Essa estratégia tende a melhorar principalmente a **padronização**, a **clareza** e a **testabilidade** das respostas, pois a LLM recebe uma referência concreta de estrutura, nível de detalhe e linguagem esperada.

### Influências observadas e esperadas

- **Maior uniformidade:** os casos seguem um formato semelhante, facilitando comparação e importação posterior para ferramentas de gestão de testes;
- **Critérios mais objetivos:** os exemplos mostram como transformar resultados vagos em condições verificáveis, como código HTTP, estado persistido e ausência de duplicidade;
- **Dados mais consistentes:** os exemplos relacionam payloads, identificadores e estados do domínio;
- **Menor ambiguidade:** a LLM tende a repetir o padrão de pré-condições, passos e evidências;
- **Risco de ancoragem:** a diversidade pode ficar limitada aos tipos de falha apresentados nos exemplos.

Para reduzir a ancoragem, o prompt exige explicitamente categorias diferentes de teste: fluxo positivo, validação, autenticação, autorização, contrato, timeout, concorrência, idempotência, indisponibilidade, limite e consistência transacional.

## 6. Como o Chain-of-Thought influenciou os resultados

O segundo prompt solicita uma análise estruturada das integrações antes dos casos. Em vez de pedir a exposição de raciocínio interno irrestrito, ele orienta a LLM a entregar uma **síntese verificável da análise**, composta por dependências, invariantes, transições de estado, pontos de falha e mecanismos de recuperação.

### Influências observadas e esperadas

- **Maior profundidade:** os casos consideram falhas parciais que ocorrem depois de uma operação bem-sucedida, como pagamento autorizado e falha na criação do prontuário;
- **Maior diversidade:** surgem cenários de compensação, reconciliação, idempotência, retry, incompatibilidade de contrato e timeout em cascata;
- **Melhor análise de consistência:** o prompt exige validação do estado final em todos os serviços participantes;
- **Maior foco arquitetural:** os testes exploram dependências e efeitos em cadeia, não apenas validação de entrada;
- **Menor padronização espontânea:** sem um esquema obrigatório, a resposta poderia ficar extensa ou irregular. Por isso, o prompt também define um formato de saída rígido.

## 7. Comparação entre as técnicas

| Dimensão | Few-Shot | Chain-of-Thought orientado |
|---|---|---|
| Padronização | Alta | Alta, devido ao esquema obrigatório |
| Facilidade de revisão | Alta | Média/Alta |
| Cobertura de falhas simples | Alta | Alta |
| Cobertura de falhas distribuídas | Média/Alta | Alta |
| Diversidade de riscos | Média/Alta | Alta |
| Detecção de inconsistências entre serviços | Média | Alta |
| Risco principal | Repetir excessivamente os exemplos | Produzir análise longa ou cenários difíceis de executar |

## 8. Síntese dos resultados esperados

A execução dos prompts deve produzir um conjunto complementar de testes:

- O **Few-Shot** deve gerar casos mais uniformes e imediatamente utilizáveis;
- O **Chain-of-Thought orientado** deve ampliar a cobertura de falhas sistêmicas e estados intermediários;
- O uso conjunto deve produzir um portfólio mais equilibrado, combinando consistência documental e diversidade técnica.

Os resultados esperados estão documentados separadamente em:

- [`resultados/few-shot-resultados-esperados.md`](resultados/few-shot-resultados-esperados.md)
- [`resultados/chain-of-thought-resultados-esperados.md`](resultados/chain-of-thought-resultados-esperados.md)

## 9. Como executar o experimento

1. Copiar integralmente um dos arquivos da pasta `prompts/`;
2. Executar o prompt na LLM escolhida;
3. Salvar a resposta original, sem ajustes, na pasta `evidencias/`;
4. Comparar a resposta com os resultados esperados;
5. Registrar data, modelo utilizado e parâmetros disponíveis;
6. Revisar tecnicamente os casos antes de considerá-los válidos;
7. Repetir o processo com o segundo prompt;
8. Comparar cobertura, diversidade, clareza e testabilidade.

## 10. Limitações

- A qualidade dos resultados depende do modelo, da versão e da configuração utilizada;
- A LLM pode gerar códigos HTTP, contratos ou comportamentos não definidos no sistema real;
- Casos plausíveis não são automaticamente casos corretos;
- Dados pessoais e clínicos reais não devem ser inseridos nos prompts;
- Os resultados precisam de revisão humana e validação contra requisitos e contratos oficiais.

## 11. Conclusão

As duas técnicas são complementares. O Few-Shot favorece respostas consistentes, bem formatadas e mais fáceis de transformar em artefatos de teste. O Chain-of-Thought orientado amplia a exploração de riscos distribuídos, estados intermediários e mecanismos de recuperação. Em conjunto, elas demonstram como a IA Generativa pode apoiar a elaboração inicial de testes de integração, desde que os resultados sejam avaliados criticamente e contextualizados por profissionais da área.
