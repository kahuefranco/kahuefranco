
---

# Documentação da API

## 1. Ambientes

- **AUTH_BASE_URL**: `https://oauth2.orionpsp.com`
- **API_BASE_URL**: `https://api.orionpsp.com/api/v1`

## 2. Autenticação

É necessário que o time comercial tenha solicitado ao time de desenvolvimento a geração das credenciais.

### cURL para autenticação:

```bash
curl --location --request POST '{{AUTH_BASE_URL}}/auth/realms/digital-banking/protocol/openid-connect/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'client_id={{your_client_id}}' \
--data-urlencode 'client_secret={{your_client_secret}}' \
--data-urlencode 'grant_type=client_credentials'
```

### Detalhes da requisição:

| Tipo  | Atributo         | Valor                              | Descrição                                                                                       |
|-------|------------------|------------------------------------|-------------------------------------------------------------------------------------------------|
| path  | AUTH_BASE_URL    | `{{your_enrironment_service}}`    | Use Ambientes de sandbox ou Produção                                                           |
| header| Content-Type     | `application/x-www-form-urlencoded`| O cabeçalho de representação Content-Type é usado para indicar o tipo de mídia original         |
| data-urlencode | client_id       | `{{your_client_id}}`                 | Seu identificador de credencial exclusivo                                                        |
| data-urlencode | client_secret   | `{{your_client_secret}}`             | Seu segredo de credencial exclusivo                                                               |
| data-urlencode | grant_type      | `client_credentials`                 | Refere-se às formas que um sistema externo possui para obter acesso a um Token de Acesso         |

### Resposta da requisição:

```json
{
  "access_token": "{{ACCESS_TOKEN}}",
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "refresh_token": "{{REFRESH_TOKEN}}",
  "token_type": "Bearer"
}
```

### Detalhes da resposta:

| Tipo   | Atributo         | Valor               | Descrição                                                                                       |
|--------|------------------|---------------------|-------------------------------------------------------------------------------------------------|
| string | access_token      | `{{ACCESS_TOKEN}}`  | JWT (JSON Web Token) este é o token gerado para ser utilizado nas chamadas para criar transações. |
| integer| expires_in        | 300                 | Tempo de expiração do token em segundos, neste exemplo o token vai durar por 5 minutos.       |
| integer| refresh_expires_in| 1800                | Tempo de expiração do refresh token em segundos, neste exemplo o refresh token vai durar por 30 minutos. |
| string | refresh_token      | `{{REFRESH_TOKEN}}` | JWT (JSON Web Token) este token pode ser utilizado para solicitar um novo access token.        |
| string | token_type        | Bearer              | Tipo do access token.                                                                           |

### Como utilizar o refresh Token

```bash
curl --location --request POST '{{AUTH_BASE_URL}}/auth/realms/digital-banking/protocol/openid-connect/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=refresh_token' \
--data-urlencode 'client_id={{your_client_id}}' \
--data-urlencode 'client_secret={{your_client_secret}}' \
--data-urlencode 'refresh_token={{REFRESH_TOKEN}}'
```

### Detalhes da requisição:

| Tipo  | Atributo         | Valor                              | Descrição                                                                                       |
|-------|------------------|------------------------------------|-------------------------------------------------------------------------------------------------|
| path  | AUTH_BASE_URL    | `{{your_enrironment_service}}`    | Use o ambiente de sandbox ou de produção                                                        |
| header| Content-Type     | `application/x-www-form-urlencoded`| O cabeçalho de representação Content-Type é usado para indicar o tipo de mídia original         |
| data-urlencode | grant_type      | `refresh_token`                     | Refere-se às formas que um sistema externo tem para obter acesso a um token de acesso          |
| data-urlencode | client_id       | `{{your_client_id}}`                | Seu identificador de credencial exclusivo                                                        |
| data-urlencode | client_secret   | `{{your_client_secret}}`            | Seu segredo de credencial exclusivo                                                               |
| data-urlencode | refresh_token   | `{{REFRESH_TOKEN}}`                  | Refresh token criado pela requisição do token de acesso                                         |

### Resposta:

```json
{
  "access_token": "{{ACCESS_TOKEN}}",
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "refresh_token": "{{REFRESH_TOKEN}}",
  "token_type": "Bearer"
}
```

### Detalhes da resposta:

| Tipo   | Atributo         | Valor               | Descrição                                                                                       |
|--------|------------------|---------------------|-------------------------------------------------------------------------------------------------|
| string | access_token      | `{{ACCESS_TOKEN}}`  | JWT (JSON Web Token) este é o token gerado para ser utilizado nas chamadas para criar transações. |
| integer| expires_in        | 300                 | Tempo de expiração do token em segundos, neste exemplo o token vai durar por 5 minutos.       |
| integer| refresh_expires_in| 1800                | Tempo de expiração do refresh token em segundos, neste exemplo o refresh token vai durar por 30 minutos. |
| string | refresh_token      | `{{REFRESH_TOKEN}}` | JWT (JSON Web Token) este token pode ser utilizado para solicitar um novo access token.        |
| string | token_type        | Bearer              | Tipo do access token.                                                                           |

## 3. Solicitando um Depósito:

```bash
curl --location --request POST '{{API_BASE_URL}}/accounts/{{account_id}}/payments' \
--header 'Authorization: Bearer {{ACCESS_TOKEN}}' \
--header 'Content-Type: application/json' \
--data-raw '{
    "type": "PIX",
    "amount": 100,
    "currency": "BRL",
    "customer": {
      "identify": {
        "type": "CPF",
        "number": "12345678900"
      },
      "name": "Test User Name",
      "email": "test@hokmapay.com",
      "phone": "75991435892",
      "address": {
        "zipCode": "38082365"
      }
    }
}'
```

### Detalhes da requisição:

| Tipo  | Atributo         | Valor                              | Opcional | Descrição                                                                                       |
|-------|------------------|------------------------------------|----------|-------------------------------------------------------------------------------------------------|
| path  | API_BASE_URL     | `{{your_enrironment_service}}`    | false    | Use as URLs de sandbox ou produção                                                               |
| path  | account_id       | `{{your_account_id}}`              | false    | Identificador da conta, esse identificador é passado através do nosso setor comercial            |
| header| Authorization     | Bearer `{{ACCESS_TOKEN}}`         | false    | ACCESS_TOKEN - Token JWT gerado durante o processo de autenticação                              |
| header| Content-Type     | `application/json`                 | false    | O cabeçalho de representação Content-Type é usado para indicar o tipo de mídia original         |
| data-raw | type           | PIX                                | true     | Identifica o tipo da operação, atualmente só suportamos PIX, logo passar esse atributo é opcional |
| data-raw | amount         | 100                                | false    | Valor monetário em centavos, por exemplo 1 é igual a 100, 0.1 é igual a 10 e 0.01 é igual a 1 |
| data-raw | currency       | BRL                                | true     | Identifica o tipo de moeda utilizada na transação, atualmente só suportamos BRL, logo esse é um atributo opcional |
| data-raw | customer       | object                             | false    | Dados de identificação do solicitante da transação                                             |
| data-raw | customer.identify.type | CPF, CNPJ                  | false    | Documento válido na receita federal (CPF 11 chars and CNPJ 14 chars)                           |
| data-raw | customer.identify.number | Exemplo 123456789000   | false    | Número do documento válido na receita federal                                                    |
| data-raw | customer.name  | string                             | false    | Nome do solicitante                                                                             |
| data-raw | customer.email | e-mail                             | false    | E-mail do solicitante                                                                           |
| data-raw | customer.phone | string                             | true     | Telefone do solicitante                                                                         |
| data-raw | customer.address.zipCode | string                 | true     | CEP do solicitante                                                                              |

### Resposta:

```json
{
  "id": "Transaction ID",
  "operationType": "CREDIT",
  "transactionMode": "DEPOSIT",
  "transactionType": "PIX",
  "amount": 100,
  "fee": 0,
  "currency": "BRL",
  "status": "WAITING_PAYMENT",
  "textCode": "Copy And Paste URL",
  "dataCode": "Qrcode Image",
  "checkoutUrl": "If Enabled Static Page, more info on Static Page Menu",
  "createdAt": "2023-11-21T12:21:28.521486Z",
  "updatedAt": "2023-11-21T12:21:29.165293Z"
}
```

### Detalhes da resposta:

| Tipo   | Atributo         | Valor               | Descrição                                                                                       |
|--------|------------------|---------------------|-------------------------------------------------------------------------------------------------|
|string	|id	|nunca	|Identificador único da transação.|
|string	|authorizationCode	|talvez	|Identificador interno da transação utilizado para transações em grupo, dificilmente necessário.|
|string	|holderName	|talvez	|Nome do dono da transação.|
|string	|operationType	|talvez	|Indica o tipo da transação, no depósito sempre será CREDIT.|
|string	|transactionMode	|talvez	|Indica o modo de operação da transação, no depósito sempre será DEPOSIT.|
|string	|transactionType	|talvez	|Tipo de transação, atualmente suportamos apenas PIX.|
|string	|description	|talvez	|Descrição relacionada à transação, atualmente com suporte apenas para pt_BR.|
|long	|amount	|nunca	|Valor monetário em centavos (1 BRL = 100, 0.1 BRL = 10, 0.01 BRL = 1).|
|long	|grossAmount	|talvez	|Deve refletir o valor de amount.|
|long	|fee	|sempre	|Este campo está depreciado, não está mais em uso.|
|string	|currency	|nunca	|Código monetário da transação no formato ISO internacional (por padrão, BRL).|
|string	|status	|nunca	|Status da transação: NEW, CREATED, PROCESSING, WAITING PAYMENT, COMPLETED, FAILED, CANCELED, entre outros.|
|string	|end2end	|talvez	|Identificador de fim-a-fim, utilizado para identificar uma transação desde que o provedor bancário o envie (pode ser nulo).|
|string	|updatedAt	|talvez	|Data da última atualização da transação no formato UTC ISO internacional.|
|string	|createdAt	|nunca	|Data em que a transação foi gerada no formato UTC ISO internacional.|


## 4. Saques:

```bash
curl --location --request POST '{{API_BASE_URL}}/accounts/{{account_id}}/withdraw' \
--header 'Authorization: Bearer {{ACCESS_TOKEN}}' \
--header 'Content-Type: application/json' \
--data-raw '{
    "type": "PIX",
    "dictKey": "DICT-EVP-TYPE-12341-1233-2323-2323131231",
    "dictType": "EVP",
    "amount": 100,
    "currency": "BRL",
    "customer": {
      "identify": {
        "type": "CPF",
        "number": "12345678900"
      },
      "name": "Test User Name",
      "email": "test@hokmapay.com",
      "phone": "112345677",
      "address": {
        "zipCode": "38082365"
      }
    }
}'
```

### Detalhes da requisição:

| Tipo  | Atributo         | Valor                              | Opcional | Descrição                                                                                       |
|-------|------------------|------------------------------------|----------|-------------------------------------------------------------------------------------------------|
| path  | API_BASE_URL     | `{{your_enrironment_service}}`    | false    | Use as URLs de sandbox ou produção                                                               |
| path  | account_id       | `{{your_account_id}}`              | false    | Identificador da conta, esse identificador é passado através do nosso setor comercial            |
| header| Authorization     | Bearer `{{ACCESS_TOKEN}}`         | false    | ACCESS_TOKEN - Token JWT gerado durante o processo de autenticação                              |
| header| Content-Type     | `application/json`                 | false    | O cabeçalho de representação Content-Type é usado para indicar o tipo de mídia original         |
| data-raw | type           | PIX                                | true     | Identifica o tipo da operação, atualmente só suportamos PIX, logo passar esse atributo é opcional |
| data-raw | dictKey       | `DICT-EVP-TYPE-12341-1233-2323-2323131231` | false | Chave do DICT que irá receber o valor correspondente à transação.                              |
| data-raw | dictType      | EVP                                | false    | Tipo que corresponde a chave do DICT.                                                          |
| data-raw | amount        | 100                                | false    | Valor monetário em centavos, por exemplo 1 é igual a 100, 0.1 é igual a 10 e 0.01 é igual a 1 |
| data-raw | currency      | BRL                                | true     | Identifica o tipo de moeda utilizada na transação, atualmente só suportamos BRL, logo esse é um atributo opcional |
| data-raw | customer      | object                             | false    | Dados de identificação do solicitante da transação                                             |
| data-raw | customer.identify.type | CPF, CNPJ                  | false    | Documento válido na receita federal (CPF 11 chars and CNPJ 14 chars)                           |
| data-raw | customer.identify.number | Exemplo 123456789000   | false    | Número do documento válido na receita federal                                                    |
| data-raw | customer.name  | string                             | false    | Nome do solicitante                                                                             |
| data-raw | customer.email | e-mail                             | false    | E-mail do solicitante                                                                           |
| data-raw | customer.phone | string                             | true     | Telefone do solicitante                                                                         |
| data-raw | customer.address.zipCode | string                 | true     | CEP do solicitante                                                                              |

### Resposta:

```json
{
  "id": "a111d11b-dcff-1111-b5f6-111ac1b11111",
  "authorizationCode": "d1bd1111-fcdb-11ec-1111-1c1111f1111f",
  "holderName": "MERCHANTNAME",
  "operationType": "DEBIT",
  "transactionMode": "WITHDRAW",
  "transactionType": "PIX",
  "description": "Transferencia enviada",
  "amount": 100,
  "grossAmount": 100,
  "fee": 0,
  "currency": "BRL",
  "status": "PROCESSING",
  "end2end": "END2ENDforthistransaction12321331",
  "updatedAt": "2022-09-27T00:08:34.758857Z",
  "createdAt": "2022-09-27T00:08:34.758845Z"
}
```

### Detalhes da resposta:

| Tipo   | Atributo         | Valor               | Descrição                                                                                       |
|--------|------------------|---------------------|-------------------------------------------------------------------------------------------------|
| string | id                | `a111d11b-dcff-1111-b5f6-111ac1b11111` | Identificador único da transação.                                                               |
| string | authorizationCode | `d1bd1111-fcdb-11ec-1111-1c1111f1111f` | Identificador interno da transação utilizado para transações em grupo, difícilmente isso será necessário. |
| string | holderName       | MERCHANTNAME        | Nome do dono da transação.                                                                      |
| string | operationType    | DEBIT               | Indica o tipo da transação, no saque sempre será DEBIT.                                         |
| string | transactionMode   | WITHDRAW            | Indica o modo de operação da transação, no saque sempre será WITHDRAW.                          |
| string | transactionType   | PIX                 | Tipo de transação, neste momento só suportamos PIX.                                            |
| string | description       | Transferencia enviada| Uma descrição qualquer relacionada a transação.                                                |
| long   | amount            | 100                 | Valor monetário em centavos, por exemplo 1BRL é igual a 100, 0.1BRL é igual a 10 e 0.01BRL é igual a 1 |
| long   | grossAmount       | 100                 | Esse campo deve refletir o valor de amount.                                                    |
| long   | fee               | 0                   | Esse campo está depreciado, não estamos mais utilizando-o.                                     |
| string | currency          | BRL                 | Código monetário da transação no formato ISO internacional, por padrão esse valor é BRL.      |
| string | status            | PROCESSING          | Status da transação pode ser NEW, CREATED, PROCESSING, WAITING PAYMENT, COMPLETED, FAILED, CANCELED. |
| string | end2end           | END2ENDforthistransaction12321331 | Esse é o identificador de fim-a-fim, pode ser utilizado para identificar uma transação desde que o provedor bancário nos envie, pode ser nulo. |
| string | updatedAt         | 2022-09-27T00:08:34.758857Z | Data da última atualização da transação no formato UTC ISO internacional.                       |
| string | createdAt         | 2022-09-27T00:08:34.758845Z | Data que a transação foi gerada no formato UTC ISO internacional.                               |

## 5. Eventos

### Tipos de eventos

| Status         | Descrição                                                                                               |
|----------------|--------------------------------------------------------------------------------------------------------|
| NEW            | Pode acontecer quando uma transação é solicitada e o provedor bancário não respondeu que recebeu a requisição. |
| CREATED        | Acontece quando o provedor bancário respondeu que recebeu a requisição, mas ainda irá processá-la.        |
| PROCESSING     | É gerado na solicitação de saque quando o provedor bancário recebeu a requisição e a está processando. |
| WAITING_PAYMENT | É gerado na solicitação de depósito quando o provedor bancário recebeu a requisição e está aguardando o pagamento. |
| COMPLETED      | Acontece quando o provedor bancário informa que terminou o processamento da requisição com sucesso.     |
| FAILED         | Pode acontecer quando o provedor bancário retorna alguma

 resposta negativa em relação ao processamento.   |
| CANCELED       | O cliente pode cancelar a operação antes que o provedor bancário comece a processá-la.                 |

### Resposta:

```json
[
  {
    "id": "6d93c4e0-888b-11ec-93cd-0242ac130004",
    "account_id": "{{account_id}}",
    "transaction_id": "Transaction ID",
    "event_type": "NEW",
    "created_at": "2022-09-27T00:08:34.758845Z"
  },
  {
    "id": "6d93c4e0-888b-11ec-93cd-0242ac130004",
    "account_id": "{{account_id}}",
    "transaction_id": "Transaction ID",
    "event_type": "COMPLETED",
    "created_at": "2022-09-27T00:08:34.758845Z"
  }
]
```

### Detalhes da resposta:

| Tipo   | Atributo         | Valor               | Descrição                                                                                       |
|--------|------------------|---------------------|-------------------------------------------------------------------------------------------------|
| string | id                | `6d93c4e0-888b-11ec-93cd-0242ac130004` | Identificador único do evento.                                                                    |
| string | account_id       | `{{account_id}}`     | Identificador da conta que gerou o evento.                                                      |
| string | transaction_id   | `Transaction ID`     | Identificador da transação relacionada ao evento.                                               |
| string | event_type       | NEW/CREATED/PROCESSING/WAITING_PAYMENT/COMPLETED/FAILED/CANCELED | Tipo de evento gerado.                                                                          |
| string | created_at       | `2022-09-27T00:08:34.758845Z` | Data que o evento foi gerado no formato UTC ISO internacional.                                  |

---

Se precisar de mais alguma coisa ou de alguma parte específica dessa documentação, é só avisar!
