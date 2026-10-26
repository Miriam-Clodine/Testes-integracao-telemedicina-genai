# Cenário de Integração: Plataforma de Telemedicina

## 1. Contexto

A plataforma permite que pacientes agendem e paguem consultas remotas. O processo depende da integração REST entre módulos especializados. Para preservar a finalidade acadêmica, todos os dados utilizados são fictícios.

## 2. Serviços e responsabilidades

| Serviço | Responsabilidade | Endpoint principal |
|---|---|---|
| Agendamento | Coordenar a criação da consulta | `POST /appointments` |
| Pacientes | Validar cadastro e elegibilidade | `GET /patients/{id}` |
| Pagamentos | Autorizar cobrança | `POST /payments` |
| Prontuário | Criar vínculo da consulta | `POST /medical-records/appointments` |
| Notificações | Enviar confirmação | `POST /notifications` |

## 3. Requisição inicial

```http
POST /appointments
Content-Type: application/json
Authorization: Bearer <token>
Idempotency-Key: 7aa765d1-46bd-45bc-aac6-761da018734f
```

```json
{
  "patientId": "PAT-1001",
  "professionalId": "PRO-205",
  "scheduledAt": "2026-11-10T14:00:00-04:00",
  "specialty": "cardiology",
  "payment": {
    "method": "credit_card",
    "amount": 180.00,
    "currency": "BRL"
  }
}
```

## 4. Resposta de sucesso

```http
HTTP/1.1 201 Created
Content-Type: application/json
```

```json
{
  "appointmentId": "APT-9001",
  "status": "CONFIRMED",
  "paymentStatus": "AUTHORIZED",
  "medicalRecordStatus": "CREATED",
  "notificationStatus": "QUEUED"
}
```

## 5. Regras e invariantes

- Um horário não pode ser reservado para dois pacientes;
- Uma chave de idempotência deve produzir no máximo um agendamento e uma cobrança;
- Nenhum prontuário deve ser criado para paciente inexistente ou inativo;
- O valor cobrado deve ser igual ao valor calculado pelo Serviço de Agendamento;
- Falha de notificação não deve desfazer uma consulta já confirmada;
- Pagamento autorizado seguido de falha crítica deve acionar compensação ou revisão manual;
- Tokens inválidos ou sem escopo adequado devem ser rejeitados;
- Dados clínicos e financeiros não devem aparecer integralmente nos logs.

## 6. Pontos de falha relevantes

- Timeout ou indisponibilidade de serviço;
- Resposta HTTP inesperada;
- Payload malformado ou incompatível com o contrato;
- Requisição duplicada;
- Condição de corrida na reserva do horário;
- Pagamento autorizado sem confirmação recebida;
- Falha de compensação;
- Divergência de estado entre os serviços;
- Falha de autenticação ou autorização;
- Excesso de requisições e retorno `429 Too Many Requests`.
