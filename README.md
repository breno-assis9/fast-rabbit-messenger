# Fast-Rabbit-Messenger

## 📜 Descrição

**Fast-Rabbit-Messenger** é uma aplicação desenvolvida em **.NET 6** para facilitar a comunicação assíncrona e eficiente entre sistemas distribuídos usando **RabbitMQ**. O projeto tem como objetivo oferecer uma solução escalável para sistemas que exigem alto desempenho no processamento de mensagens em tempo real.

Com suporte a envio (Producer) e consumo (Consumer) de mensagens, o **Fast-Rabbit-Messenger** é ideal para arquiteturas baseadas em **Microserviços**, **Event-Driven Architecture** (EDA) e **CQRS** (Command Query Responsibility Segregation), proporcionando baixa latência e alta confiabilidade.

## ⚙️ Tecnologias

- **.NET 6** (C#) 
- **RabbitMQ** 🐇
- **Docker** 🐳
- **Serilog** 📊 (Para logging e monitoramento)
- **RabbitMQ Client .NET** (RabbitMQ.Client)
- **Docker Compose** (Para facilitar o setup do RabbitMQ)

## 🚀 Funcionalidades

- **Envio de Mensagens**: Produz e envia mensagens para filas RabbitMQ de forma eficiente e com garantias de entrega.
- **Consumo Assíncrono**: Consome mensagens de forma assíncrona, processando em tempo real com suporte a múltiplos consumidores.
- **Escalabilidade Horizontal**: Suporte a múltiplos consumidores escaláveis para processar mensagens simultaneamente.
- **Resiliência**: Mecanismos de retry e **dead-letter queue (DLQ)** para garantir a entrega de mensagens mesmo em cenários de falha.
- **Monitoramento e Logging**: Monitoramento detalhado das interações do sistema através de logs utilizando o **Serilog**.
- **Facilidade de Configuração**: Configuração simplificada usando **Docker Compose** para RabbitMQ, permitindo o ambiente de desenvolvimento pronto para uso.

## 🛠️ Requisitos

- **.NET 6 SDK** ou superior.
- **RabbitMQ** em funcionamento (via Docker ou instalação local).
- **Docker** (opcional, para executar RabbitMQ em containers).

## 📥 Instalação

### 1. Clonando o Repositório

```
git clone https://github.com/usuario/fast-rabbit-messenger.git
cd fast-rabbit-messenger
```

### 2. Configuração do RabbitMQ com Docker
Caso você não tenha o RabbitMQ instalado localmente, pode facilmente configurar o ambiente com Docker.

Crie um arquivo docker-compose.yml com a seguinte configuração:

```
version: '3'

services:
  rabbitmq:
    image: rabbitmq:management
    environment:
      RABBITMQ_DEFAULT_USER: user
      RABBITMQ_DEFAULT_PASS: password
    ports:
      - "15672:15672"  # Interface de gerenciamento RabbitMQ (HTTP)
      - "5672:5672"    # Porta para comunicação com RabbitMQ
```

Para iniciar o RabbitMQ e a interface de gerenciamento, execute o comando:

```
docker-compose up -d
```

Após a execução, a interface de gerenciamento estará disponível em http://localhost:15672, com usuário user e senha password.

### 3. Configuração do Projeto

Com o RabbitMQ em funcionamento, você pode configurar a conexão em seu aplicativo Fast-Rabbit-Messenger.

No arquivo appsettings.json, configure as variáveis de ambiente para a conexão com RabbitMQ:

```
{
  "RabbitMQ": {
    "Host": "localhost",
    "Port": "5672",
    "Username": "user",
    "Password": "password",
    "QueueName": "fast-rabbit-queue",
    "RetryCount": 5,
    "DeadLetterQueue": "fast-rabbit-dlq"
  }
}
```

### 4. Execução do Projeto
Para executar o projeto, basta rodar o seguinte comando na raiz do projeto:

```
dotnet run

```
O serviço começará a produzir e consumir mensagens de forma assíncrona.

### 🏗️ Arquitetura e Funcionamento
**Produtor de Mensagens (Producer)**
O produtor de mensagens se conecta à fila RabbitMQ, envia uma mensagem para a fila configurada e espera a confirmação de que a mensagem foi recebida. A aplicação garante a entrega de mensagens, com suporte a tentativas de reenvio em caso de falha.

**Consumidor de Mensagens (Consumer)**
O consumidor de mensagens consome as mensagens da fila configurada. Ele pode ser escalado horizontalmente para processar várias mensagens simultaneamente. O Dead Letter Queue (DLQ) é configurado para lidar com mensagens que não puderam ser processadas após várias tentativas.

### 📡 Endpoints
**1. Enviar Mensagem para a Fila**
O endpoint HTTP para enviar mensagens para RabbitMQ é o seguinte:

POST /api/rabbitmq/send
Body (JSON):

{
  "message": "Mensagem importante para RabbitMQ!"
}

Este endpoint recebe uma mensagem no corpo da requisição e a coloca na fila para processamento.

**2. Consumir Mensagem**
O consumidor de mensagens começa a escutar a fila fast-rabbit-queue automaticamente. Ele processa as mensagens à medida que são consumidas.

## 🔧 Exemplo de Código
Aqui está um exemplo detalhado do código que implementa o Producer e Consumer para RabbitMQ.

**RabbitMQ Producer (Produtor de Mensagens)**

```
public class RabbitMQProducer
{
    private readonly IConnection _connection;
    private readonly IModel _channel;

    public RabbitMQProducer(IConfiguration config)
    {
        var factory = new ConnectionFactory()
        {
            HostName = config["RabbitMQ:Host"],
            Port = int.Parse(config["RabbitMQ:Port"]),
            UserName = config["RabbitMQ:Username"],
            Password = config["RabbitMQ:Password"]
        };
        
        _connection = factory.CreateConnection();
        _channel = _connection.CreateModel();
        
        // Declara a fila caso não exista
        _channel.QueueDeclare(queue: config["RabbitMQ:QueueName"],
                             durable: false,
                             exclusive: false,
                             autoDelete: false,
                             arguments: null);
    }

    public void SendMessage(string message)
    {
        var body = Encoding.UTF8.GetBytes(message);
        
        // Envia a mensagem para a fila
        _channel.BasicPublish(exchange: "",
                              routingKey: config["RabbitMQ:QueueName"],
                              basicProperties: null,
                              body: body);
        
        Console.WriteLine($"Mensagem enviada: {message}");
    }
}
```

**RabbitMQ Consumer (Consumidor de Mensagens)**

```
public class RabbitMQConsumer
{
    private readonly IConnection _connection;
    private readonly IModel _channel;

    public RabbitMQConsumer(IConfiguration config)
    {
        var factory = new ConnectionFactory()
        {
            HostName = config["RabbitMQ:Host"],
            Port = int.Parse(config["RabbitMQ:Port"]),
            UserName = config["RabbitMQ:Username"],
            Password = config["RabbitMQ:Password"]
        };

        _connection = factory.CreateConnection();
        _channel = _connection.CreateModel();
        
        // Declara a fila caso não exista
        _channel.QueueDeclare(queue: config["RabbitMQ:QueueName"],
                             durable: false,
                             exclusive: false,
                             autoDelete: false,
                             arguments: null);
    }

    public void StartConsuming()
    {
        var consumer = new EventingBasicConsumer(_channel);
        
        consumer.Received += (sender, args) =>
        {
            var message = Encoding.UTF8.GetString(args.Body.ToArray());
            Console.WriteLine($"Mensagem recebida: {message}");

            // Lógica de processamento de mensagem
        };

        // Consome mensagens da fila
        _channel.BasicConsume(queue: config["RabbitMQ:QueueName"],
                              autoAck: true,
                              consumer: consumer);
    }
}
```

**Retry e Dead Letter Queue (DLQ)**
Caso uma mensagem não seja processada corretamente, ela pode ser movida para uma Dead Letter Queue. Isso permite que você realize a análise de mensagens falhas e as reenvie para processamento posterior.

**Configuração da DLQ em appsettings.json:**

```
{
  "RabbitMQ": {
    "DeadLetterQueue": "fast-rabbit-dlq"
  }
}
```

### 📈 Escalabilidade
O Fast-Rabbit-Messenger pode ser escalado horizontalmente tanto para produção quanto para consumo de mensagens. Múltiplos consumidores podem ser iniciados para processar mensagens de forma paralela e eficiente. Isso é especialmente útil para sistemas de alta demanda.

### 🤝 Contribuição
Contribuições são bem-vindas! Para contribuir com o projeto, siga as etapas abaixo:

Faça um fork deste repositório.
Crie uma branch para sua feature (git checkout -b minha-feature).
Commit suas alterações (git commit -am 'Adiciona nova feature').
Push para a branch (git push origin minha-feature).
Abra um pull request explicando suas alterações.

### 📝 Licença
Este projeto está licenciado sob a MIT License - veja o arquivo LICENSE.

### 🛠️ Notas sobre os ícones:
- **RabbitMQ** 🐇 foi adicionado no título do projeto para representar o serviço que a aplicação utiliza.
- **Docker** 🐳 foi adicionado nas seções onde Docker é mencionado.
- **Serilog** 📊 foi adicionado para destacar a parte de logging e monitoramento.
- **Escalabilidade** foi sinalizada com o ícone **📈**.
- **Contribuição** foi sinalizada com o ícone **🤝**.


