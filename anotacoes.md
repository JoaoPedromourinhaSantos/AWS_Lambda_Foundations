# AWS Lambda Foundations – Guia Completo 🚀

Bem-vindo ao seu guia sobre fundamentos AWS Lambda!  

## Introdução ao Serverless

A computação em nuvem revolucionou o desenvolvimento de aplicações, permitindo **abstrair a infraestrutura** e focar apenas na lógica de negócio.  
No modelo **serverless**, você não precisa se preocupar com servidores, sistemas operacionais ou provisionamento manual – basta escrever seu código e deixar a AWS cuidar do resto!

### 🏆 Por que serverless?

- **Zero gerenciamento de servidores**
- **Escalabilidade automática**
- **Redução de custos operacionais**
- **Foco total no código e na inovação**

### 📊 Tradicional vs. Serverless

| Tarefa                                      | Tradicional | Serverless |
|----------------------------------------------|:-----------:|:----------:|
| Configurar instância                         |     ✔️      |     ❌     |
| Atualizar sistema operacional                |     ✔️      |     ❌     |
| Instalar plataforma de aplicativos           |     ✔️      |     ❌     |
| Criar/implantar aplicativos                  |     ✔️      |     ✔️     |
| Configurar auto scaling/balanceamento        |     ✔️      |     ❌     |
| Proteger/monitorar instâncias                |     ✔️      |     ❌     |
| Monitorar/manter aplicativos                 |     ✔️      |     ✔️     |

---

## O que é AWS Lambda?

**AWS Lambda** é o serviço serverless da AWS para executar seu código em resposta a eventos, **sem precisar provisionar ou gerenciar servidores**.

### ✨ Destaques do Lambda

- **Alta disponibilidade**: infraestrutura redundante e resiliente
- **Escalabilidade automática**: cresce ou diminui conforme a demanda
- **Cobrança por uso**: pague apenas pelo tempo de execução e número de invocações
- **Monitoramento integrado**: via Amazon CloudWatch
- **Suporte a várias linguagens**: Python, Node.js, Java, Go, Ruby, .NET e custom runtimes

---

## Arquiteturas Orientadas a Eventos

O Lambda é a peça-chave para arquiteturas orientadas a eventos na AWS.  
**Eventos** (como uploads no S3, mensagens no SQS, atualizações em DynamoDB) **acionam funções Lambda**, que processam tudo de forma desacoplada e escalável.

![Arquitetura Orientada a Eventos](https://github.com/user-attachments/assets/4abdec71-1b7f-4d72-b0f1-82afb07125f2)

### Exemplos práticos de eventos

- Upload de arquivo no S3
- Mensagem publicada no SNS
- Registro inserido no DynamoDB
- Requisição HTTP via API Gateway

---

## Como o AWS Lambda Funciona

### Estrutura de uma função Lambda

- **Handler**: ponto de entrada do seu código
- **Runtime**: ambiente de execução da linguagem escolhida
- **Memory**: memória alocada (define a CPU proporcional)
- **Timeout**: tempo máximo de execução (até 15min)
- **Variáveis de ambiente**: configurações dinâmicas

### Modelos de Invocação

#### 1️⃣ Invocação Síncrona

- O serviço espera a resposta da função Lambda.
- Exemplos: API Gateway, Cognito, CloudFormation, Alexa, Lex, CloudFront.

![Invocação Síncrona](https://github.com/user-attachments/assets/9d92d98c-79c7-47f6-8ee2-6e1e03da0ed2)

#### 2️⃣ Invocação Assíncrona

- O serviço envia o evento e não espera resposta.
- Lambda processa e tenta novamente em caso de erro (até 2 vezes).
- Exemplos: SNS, S3, EventBridge.

![Invocação Assíncrona](https://github.com/user-attachments/assets/30289e7e-3257-49bd-b8b7-477f2b1442de)
![Exemplo de fluxo assíncrono](https://github.com/user-attachments/assets/9ad86951-a87c-4beb-b153-d47350524c44)

#### 3️⃣ Invocação por Sondagem (Polling)

- Lambda lê eventos de streams ou filas.
- Exemplos: DynamoDB Streams, Kinesis, SQS, Kafka, MQ.

---

### Comportamento de Erro

| Modelo de invocação | Comportamento do erro             |
|---------------------|-----------------------------------|
| Síncrono            | Sem novas tentativas              |
| Assíncrono          | Novas tentativas automáticas (2x) |
| Sondagem            | Depende da fonte de eventos       |

---

## Ambiente de Execução do Lambda

Lambda executa funções em ambientes isolados e seguros (_execution environments_).

![Ambiente de Execução](https://github.com/user-attachments/assets/3933bb1a-a0f2-445a-9b87-cd99ba58e3df)

### Fases do Ciclo de Vida

- **Fase INIT**: Inicialização do ambiente, carregamento de dependências e código (executada uma vez por ambiente)
- **Fase de Invocação**: Execução do handler em resposta ao evento
- **Fase de Desligamento**: Encerramento do ambiente, liberando recursos

### Inicializações a Quente e a Frio

- **Cold Start**: Novo ambiente criado → maior latência na primeira execução
- **Warm Start**: Ambiente já ativo → latência mínima nas execuções seguintes

---

## Gerenciamento de Funções Lambda

### Versionamento e Aliases

- **Versionamento**: crie versões imutáveis da função (facilita rollback e testes)
- **Aliases**: apontam para versões específicas (ex: produção, homologação)

### Limites de Simultaneidade

- **Simultaneidade**: número de execuções paralelas permitidas
- **Reserva de simultaneidade**: garante capacidade para funções críticas
- **Motivos para limitar**: gerenciar custos, proteger recursos downstream, controlar tempo de processamento de batches

---

## Permissões e Segurança

### Função de Execução do IAM

- Define o que a função Lambda pode acessar em outros serviços AWS (ex: S3, DynamoDB)
- **Princípio do menor privilégio**: conceda apenas as permissões necessárias

### Políticas Baseadas em Recursos

- Definem quem ou qual serviço pode invocar sua função Lambda
- Exemplo: permitir que S3 invoque uma função Lambda ao receber um novo objeto

---

## Monitoramento e Observabilidade

- **CloudWatch**: logs, métricas (invocações, erros, duração, etc.)
- **AWS X-Ray**: rastreamento distribuído, visualização do fluxo de chamadas e dependências
- **Lambda Insights**: métricas detalhadas de desempenho e uso de recursos

---

## Boas Práticas e Otimizações

- Divida funções por responsabilidade (microfunções)
- Evite dependências desnecessárias para reduzir cold start
- Gerencie variáveis de ambiente para configurações dinâmicas
- Implemente tratamento de erros e retries
- Monitore e alerte sobre falhas e desempenho
- Utilize layers para compartilhamento de código comum

---

## Exemplos Práticos

### Função Lambda em Python

```python
import json

def lambda_handler(event, context):
    print("Evento recebido:", event)
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }

````

## Referências
[AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
