# AWS Lambda Foundations – Guia Completo

## Índice

1. [Introdução ao Serverless](#introdução-ao-serverless)
2. [O que é AWS Lambda?](#o-que-é-aws-lambda)
3. [Arquiteturas Orientadas a Eventos](#arquiteturas-orientadas-a-eventos)
4. [Como o AWS Lambda Funciona](#como-o-aws-lambda-funciona)
    - [Modelos de Invocação](#modelos-de-invocação)
    - [Comportamento de Erro](#comportamento-de-erro)
5. [Ambiente de Execução do Lambda](#ambiente-de-execução-do-lambda)
    - [Fases do Ciclo de Vida](#fases-do-ciclo-de-vida)
    - [Inicializações a Quente e a Frio](#inicializações-a-quente-e-a-frio)
6. [Gerenciamento de Funções Lambda](#gerenciamento-de-funções-lambda)
    - [Versionamento e Aliases](#versionamento-e-aliases)
    - [Limites de Simultaneidade](#limites-de-simultaneidade)
7. [Permissões e Segurança](#permissões-e-segurança)
    - [Função de Execução do IAM](#função-de-execução-do-iam)
    - [Políticas Baseadas em Recursos](#políticas-baseadas-em-recursos)
8. [Monitoramento e Observabilidade](#monitoramento-e-observabilidade)
9. [Boas Práticas e Otimizações](#boas-práticas-e-otimizações)
10. [Exemplos Práticos](#exemplos-práticos)
11. [Referências](#referências)

---

## Introdução ao Serverless

A computação em nuvem revolucionou a forma como aplicativos são desenvolvidos, permitindo **abstrair a camada de infraestrutura**. O modelo **Serverless** (sem servidor) elimina a necessidade de gerenciar hardware físico, sistemas operacionais ou instâncias de servidores. O foco passa a ser **apenas no código** e na lógica de negócio.

### Comparativo: Tradicional vs. Serverless

| Implantação e tarefas operacionais           | Ambiente tradicional | Sem servidor |
|----------------------------------------------|:-------------------:|:------------:|
| Configurar uma instância                     | SIM                 | -            |
| Atualizar Sistema Operacional (OS)           | SIM                 | -            |
| Instalar plataforma de aplicativos           | SIM                 | -            |
| Criar e implantar aplicativos                | SIM                 | SIM          |
| Configurar auto scaling e balanceamento de carga | SIM             | -            |
| Proteger e monitorar continuamente instâncias| SIM                 | -            |
| Monitorar e manter aplicativos               | SIM                 | SIM          |

---

## O que é AWS Lambda?

**AWS Lambda** é um serviço de computação serverless que executa código em resposta a eventos, **sem necessidade de provisionar ou gerenciar servidores**. Ele oferece:

- **Alta disponibilidade**: O Lambda executa seu código em uma infraestrutura redundante.
- **Escalabilidade automática**: Aumenta ou reduz o número de execuções conforme a demanda.
- **Gerenciamento de recursos**: A AWS cuida da manutenção dos servidores, do sistema operacional, do provisionamento de capacidade, do auto scaling, do monitoramento e dos logs.
- **Cobrança por uso**: Você paga apenas pelo tempo de execução do código (em milissegundos) e pela quantidade de solicitações.

### Benefícios do Lambda

- **Sem provisionamento de servidores**
- **Resposta a eventos de diversos serviços AWS**
- **Escalabilidade automática**
- **Monitoramento integrado via CloudWatch**
- **Suporte a múltiplas linguagens** (Python, Node.js, Java, Go, Ruby, .NET, custom runtimes)

---

## Arquiteturas Orientadas a Eventos

Lambda é fundamental para **arquiteturas orientadas a eventos**.  
Nessa abordagem, **eventos** (como uploads no S3, mensagens no SQS, atualizações em DynamoDB, notificações via SNS) **acionam funções Lambda**, que processam esses eventos de forma desacoplada e escalável.

<img width="1473" height="599" alt="image" src="https://github.com/user-attachments/assets/4abdec71-1b7f-4d72-b0f1-82afb07125f2" />

### Exemplos de Eventos

- Upload de arquivo em um bucket S3
- Mensagem publicada em um tópico SNS
- Registro inserido em uma tabela DynamoDB
- Requisição HTTP via API Gateway

---

## Como o AWS Lambda Funciona

### Estrutura da Função Lambda

- **Handler**: Função principal que processa o evento recebido.
- **Runtime**: Ambiente de execução da linguagem escolhida.
- **Memory**: Quantidade de memória alocada (define também a CPU proporcional).
- **Timeout**: Tempo máximo de execução (até 15 minutos).
- **Variáveis de ambiente**: Configurações dinâmicas para a função.

### Modelos de Invocação

#### 1. Invocação Síncrona

- O serviço que invoca o Lambda espera a resposta.
- Exemplos: API Gateway, Cognito, CloudFormation, Alexa, Lex, CloudFront.

<img width="508" height="281" alt="image" src="https://github.com/user-attachments/assets/9d92d98c-79c7-47f6-8ee2-6e1e03da0ed2" />

#### 2. Invocação Assíncrona

- O serviço invoca o Lambda e não espera resposta.
- O Lambda processa o evento e, em caso de erro, tenta novamente duas vezes.
- Exemplos: SNS, S3, EventBridge.

<img width="964" height="330" alt="image" src="https://github.com/user-attachments/assets/30289e7e-3257-49bd-b8b7-477f2b1442de" />
<img width="869" height="520" alt="image" src="https://github.com/user-attachments/assets/9ad86951-a87c-4beb-b153-d47350524c44" />

#### 3. Invocação por Sondagem (Poll-Based)

- Lambda lê eventos de streams ou filas.
- Exemplos: DynamoDB Streams, Kinesis, SQS, Kafka, MQ.

---

### Comportamento de Erro

| Modelo de invocação | Comportamento do erro                 |
|---------------------|---------------------------------------|
| Síncrono            | Sem novas tentativas                  |
| Assíncrono          | Novas tentativas automáticas (2x)     |
| Sondagem            | Depende da fonte de eventos           |

---

## Ambiente de Execução do Lambda

Lambda executa funções em **ambientes isolados e seguros** chamados de _execution environments_.

<img width="962" height="601" alt="image" src="https://github.com/user-attachments/assets/3933bb1a-a0f2-445a-9b87-cd99ba58e3df" />

### Fases do Ciclo de Vida

- **Fase INIT**:  
  Ocorre a inicialização do ambiente, carregamento das dependências e do código da função. Executada apenas uma vez por ambiente.

- **Fase de Invocação**:  
  O Lambda executa o handler em resposta a cada evento.

- **Fase de Desligamento**:  
  O ambiente é destruído, liberando recursos. Pode ocorrer por inatividade ou atualização.

---

### Inicializações a Quente e a Frio

- **Inicialização a Frio (Cold Start)**:  
  Ocorre quando o Lambda precisa criar um novo ambiente de execução, resultando em maior latência na primeira execução.

- **Inicialização a Quente (Warm Start)**:  
  Ocorre quando o ambiente já está ativo, permitindo execuções subsequentes com latência mínima.

---

## Gerenciamento de Funções Lambda

### Versionamento e Aliases

- **Versionamento**: Permite criar versões imutáveis da função, facilitando rollback e testes.
- **Aliases**: Apontam para versões específicas, permitindo deploys controlados (ex: produção, homologação).

### Limites de Simultaneidade

- **Simultaneidade**: Número de execuções paralelas permitidas.
- **Reserva de Simultaneidade**: Garante capacidade para funções críticas.
- **Limites globais**: Cada conta tem limites (soft/hard) de simultaneidade.

**Motivos para limitar simultaneidade:**
- Gerenciar custos
- Proteger recursos downstream
- Controlar tempo de processamento de batches

---

## Permissões e Segurança

### Função de Execução do IAM

- **Execution Role**: Define o que a função Lambda pode fazer em outros serviços da AWS (ex: ler S3, gravar DynamoDB).
- **Princípio do menor privilégio**: Conceda apenas as permissões necessárias.

### Políticas Baseadas em Recursos

- Permitem definir quem/qual serviço pode invocar a função Lambda.
- Exemplo: Permitir que S3 invoque uma função Lambda ao receber um novo objeto.

---

## Monitoramento e Observabilidade

- **Amazon CloudWatch**: Logs, métricas (invocações, erros, duração, etc.).
- **AWS X-Ray**: Rastreamento distribuído, visualização do fluxo de chamadas e dependências.
- **Lambda Insights**: Métricas detalhadas de desempenho e uso de recursos.

---

## Boas Práticas e Otimizações

- **Divida funções por responsabilidade** (microfunções)
- **Evite dependências desnecessárias** para reduzir cold start
- **Gerencie variáveis de ambiente** para configurações dinâmicas
- **Implemente tratamento de erros e retries**
- **Monitore e alerte sobre falhas e desempenho**
- **Utilize layers** para compartilhamento de código comum

---

## Exemplos Práticos

### Exemplo: Função Lambda em Python

```python
import json

def lambda_handler(event, context):
    print("Evento recebido:", event)
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
