# Executando Tarefas Automatizadas com AWS Lambda e S3

Repositório desenvolvido como parte do Desafio de Projeto da [DIO](https://dio.me) sobre automação de tarefas na AWS utilizando **Amazon S3**, **AWS Lambda** e **DynamoDB**, simulados localmente com **LocalStack**.

---

## 🎯 Objetivo do Projeto
Consolidar na prática como configurar e executar tarefas serverless automatizadas disparadas por eventos de armazenamento:
- Criar e gerenciar buckets no Amazon S3.
- Configurar funções serverless no AWS Lambda.
- Acionar a função Lambda automaticamente a partir do upload de um arquivo no S3 (S3 Event Notifications).
- Processar os metadados do arquivo e persistir os registros no Amazon DynamoDB.
- Testar e validar todo o fluxo em ambiente de desenvolvimento local usando o LocalStack.

---

## 🛠️ Tecnologias e Ferramentas
- **AWS S3**: Armazenamento de objetos e fonte de eventos (triggers).
- **AWS Lambda**: Computação serverless orientada a eventos.
- **Amazon DynamoDB**: Banco de dados NoSQL para registro das execuções.
- **LocalStack**: Emulador local dos serviços da nuvem AWS.
- **AWS CLI / awslocal**: Interface de linha de comando para interagir com os serviços locais.
- **Python**: Linguagem utilizada para a função Lambda.

---

## ⚙️ Arquitetura e Fluxo de Execução

1. **Upload no S3**: Um arquivo é enviado para o bucket (ex: `meu-bucket-local`).
2. **Disparo do Evento**: O S3 gera um evento `s3:ObjectCreated:*` e aciona a função Lambda configurada.
3. **Processamento Serverless**: A função Lambda extrai o nome do bucket e a chave do arquivo recebido.
4. **Persistência**: As informações (nome do arquivo, tamanho, timestamp) são salvas em uma tabela no DynamoDB.

---

## 🚀 Passo a Passo da Implementação

### 1. Iniciar o LocalStack
Subir o container do LocalStack com Docker:
```bash
docker run -d --name localstack -p 4566:4566 -p 4510-4559:4510-4559 localstack/localstack
```

### 2. Criar a Tabela no DynamoDB
```bash
awslocal dynamodb create-table \
    --table-name RegistroUploads \
    --attribute-definitions AttributeName=id,AttributeType=S \
    --key-schema AttributeName=id,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST
```

### 3. Criar o Bucket no S3
```bash
awslocal s3 mb s3://meu-bucket-processamento
```

### 4. Criar e Publicar a Função Lambda
Compactar o código da função (`lambda_function.py`):
```bash
zip function.zip lambda_function.py
```

Criar a função no LocalStack:
```bash
awslocal lambda create-function \
    --function-name ProcessadorS3 \
    --runtime python3.9 \
    --role arn:aws:iam::000000000000:role/lambda-role \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip
```

### 5. Configurar a Notificação do S3 para a Lambda
Configurar o bucket para notificar a Lambda sempre que um novo arquivo for inserido:
```bash
awslocal s3api put-bucket-notification-configuration \
    --bucket meu-bucket-processamento \
    --notification-configuration '{
        "LambdaFunctionConfigurations": [
            {
                "LambdaFunctionArn": "arn:aws:lambda:us-east-1:000000000000:function:ProcessadorS3",
                "Events": ["s3:ObjectCreated:*"]
            }
        ]
    }'
```

### 6. Testar o Fluxo
Enviar um arquivo de teste para o bucket S3:
```bash
awslocal s3 cp teste.txt s3://meu-bucket-processamento/
```

Consultar a tabela do DynamoDB para validar o registro:
```bash
awslocal dynamodb scan --table-name RegistroUploads
```

---

## 🧠 Insights e Aprendizados
- **Arquitetura Orientada a Eventos:** O desacoplamento proporcionado pelo S3 disparando a Lambda reduz complexidade e custos operacionais.
- **LocalStack como Acelerador:** Simular serviços da AWS localmente permite iterar com agilidade sem gerar custos em conta real de nuvem.
- **Infraestrutura Serverless:** Foco exclusivo nas regras de negócio sem necessidade de gerenciar servidores.

---
Feito com dedicação para a formação DIO!
