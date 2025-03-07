# Cenários do Azure Functions

**Artigo** | Data: 07/03/2025 | Colaboradores: 10

Este tutorial apresenta diversos cenários de uso do Azure Functions. Se você deseja criar APIs web, processar eventos em tempo real, manipular arquivos ou agendar tarefas, entre outros, o Azure Functions pode ser a solução sem servidor ideal para o seu projeto.

> Dica: Escolha sua linguagem de programação preferida (C#, Java, JavaScript, PowerShell, Python, TypeScript) conforme o cenário.

---

## 1. Processar Uploads de Arquivo

Utilize funções para processar arquivos enviados para contêineres de blobs – ideal para cenários em que, por exemplo, parceiros enviam catálogos de produtos.

- **Como funciona:**  
  Ao fazer upload de um arquivo, uma função disparada (Blob Trigger) valida, transforma e processa cada registro do arquivo.

- **Exemplo em C# (Trigger de Blob):**

  ```csharp
  [FunctionName("ProcessCatalogData")]
  public static async Task Run(
      [BlobTrigger("catalog-uploads/{name}", Source = BlobTriggerSource.EventGrid, Connection = "<NAMED_STORAGE_CONNECTION>")] Stream myCatalogData,
      string name,
      ILogger log)
  {
      log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myCatalogData.Length} Bytes");

      using (var reader = new StreamReader(myCatalogData))
      {
          var catalogEntry = await reader.ReadLineAsync();
          while (catalogEntry != null)
          {
              // Processa a entrada do catálogo.
              // ...
              catalogEntry = await reader.ReadLineAsync();
          }
      }
  }


Links úteis:

1. **Carregar e analisar um arquivo com o Azure Functions e o Armazenamento de Blobs**  
   Aqui, você pode aprender a usar o Azure Functions com o Armazenamento de Blobs para carregar e processar arquivos.  
   [Saiba mais](https://learn.microsoft.com/pt-br/azure/storage/blobs/blob-upload-function-trigger?tabs=azure-portal)

2. **Disparar o Azure Functions usando uma assinatura de evento em contêineres de blob**
   Neste caso, você descobrirá como disparar o Azure Functions quando um arquivo é carregado em um contêiner de blob.  
   [Saiba mais](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-event-grid-blob-trigger?pivots=programming-language-csharp)


![image](https://github.com/user-attachments/assets/5c1cba40-d527-4ec7-8ef0-776a15df22f8)

Diagrama de Upload de Arquivo

## 2.Processamento de Eventos e Fluxo em Tempo Real
Ideal para manipular grandes volumes de dados gerados por aplicativos, dispositivos IoT e outras fontes, processando informações quase em tempo real.


Como funciona:  

Funções usando gatilhos de Event Hub ou Grade de Eventos recebem e transformam dados em tempo real, enviando-os para bancos de dados como o Azure Cosmos DB.


Exemplo em C# (Gatilho de Event Hub):


[FunctionName("ProcessorFunction")]
public static async Task Run(
    [EventHubTrigger("%Input_EH_Name%", Connection = "InputEventHubConnectionString", ConsumerGroup = "%Input_EH_ConsumerGroup%")] EventData[] inputMessages,
    [EventHub("%Output_EH_Name%", Connection = "OutputEventHubConnectionString")] IAsyncCollector<SensorDataRecord> outputMessages,
    PartitionContext partitionContext,
    ILogger log)
{
    var debatcher = new Debatcher(log);
    var debatchedMessages = await debatcher.Debatch(inputMessages, partitionContext.PartitionId);

    var xformer = new Transformer(log);
    await xformer.Transform(debatchedMessages, partitionContext.PartitionId, outputMessages);
}


Links úteis:



Streaming em escala com Hubs de Eventos, Functions e SQL do Azure

Gatilho de Hubs de Eventos do Azure para Azure Functions

![image](https://github.com/user-attachments/assets/c22cfa46-1122-4cfa-bd4a-bc526777a4e8)

Diagrama de Fluxo em Tempo Real

## 3. Machine Learning e IA
As funções podem ser usadas para integrar modelos de aprendizado de máquina e inteligência artificial – conectando com serviços como o Azure OpenAI ou TensorFlow para análises e inferências.


Como funciona:  

Você pode invocar modelos de IA em resposta a eventos ou dados processados pela função, como para preenchimento de texto, sumarização ou classificação de imagens.


Links úteis:



Preenchimento de texto usando o Azure OpenAI

Exemplo de sumarização de texto com IA

Amostras do Azure Functions OpenAI Extension

![image](https://github.com/user-attachments/assets/80d2ad9c-33d4-49f0-a798-3ea69308e6ec)

Diagrama de Machine Learning e IA

## 4. Executar Tarefas Agendadas
Utilize funções para executar código em intervalos programados – ideal para tarefas recorrentes como limpeza de dados, sincronizações ou auditorias.


Como funciona:  

Com um agendamento cron, a função é disparada periodicamente para realizar a ação desejada.


Exemplo em C# (Timer Trigger):


[FunctionName("TimerTriggerCSharp")]
public static void Run([TimerTrigger("0 */15 * * * *")] TimerInfo myTimer, ILogger log)
{
    if (myTimer.IsPastDue)
    {
        log.LogInformation("Timer is running late!");
    }
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
    
    // Realiza a deduplicação no banco de dados ou outra tarefa agendada.
}


Link útil:



Gatilho de temporizador para o Azure Functions

![image](https://github.com/user-attachments/assets/e7586503-985f-4b18-998a-9687e5906612)


Diagrama de Tarefa Agendada

## 5. Criar uma API Web Escalonável
Funções disparadas por HTTP permitem a construção de APIs sem servidor, facilitando a criação de endpoints para webhooks, integrações e comunicação com outros serviços.


Como funciona:  

Uma função HTTP pode receber requisições, processar dados e interagir com bancos de dados ou outros sistemas.


Exemplo em C# (HTTP Trigger com Cosmos DB):


[FunctionName("InsertName")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req,
    [CosmosDB(
        databaseName: "my-database",
        collectionName: "my-container",
        ConnectionStringSetting = "CosmosDbConnectionString")] IAsyncCollector<dynamic> documentsOut,
    ILogger log)
{
    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    string name = data?.name;

    if (name == null)
    {
        return new BadRequestObjectResult("Please pass a name in the request body json");
    }

    await documentsOut.AddAsync(new
    {
        id = System.Guid.NewGuid().ToString(), // Cria um ID aleatório
        name = name
    });

    return new OkResult();
}


Links úteis:

![image](https://github.com/user-attachments/assets/0f9b74bd-f3d9-4116-a8a9-d86fbfdb910e)


Gatilho HTTP do Azure Functions

Exemplo: API com Azure Functions, Aplicativos Web Estáticos e BD SQL do Azure


Diagrama da API Web Escalonável

## 6. Criar um Fluxo de Trabalho Sem Servidor
O Azure Functions pode ser parte integrante de fluxos de trabalho sem servidor – integrando com Aplicativos Lógicos ou usando a extensão Durable Functions para orquestração de processos.


Como funciona:  

Combine várias funções para comandar um fluxo de trabalho mais complexo ou de longa duração.


Links úteis:



Visão geral do Durable Functions

Criar uma função integrada aos Aplicativos Lógicos do Azure
![image](https://github.com/user-attachments/assets/50de0bf8-f514-43e3-a8e3-f1156dc1e35d)


Diagrama de Fluxo de Trabalho Sem Servidor

## 7. Responder a Alterações no Banco de Dados
Crie funções que sejam acionadas quando houver alterações em seus dados, permitindo registrar, auditar ou processar essas modificações.


Como funciona:  

Gatilhos específicos (como os do Cosmos DB ou Banco de Dados SQL) permitem reagir a inserções, atualizações ou deleções.


Links úteis:



Azure Functions com Azure Cosmos DB no Visual Studio Code

Limpeza de banco de dados SQL do Azure com Azure Functions

![image](https://github.com/user-attachments/assets/53d3358c-9bb1-4f45-8218-ed5433fb4d62)

Diagrama de Alterações no Banco de Dados

## 8. Criar Sistemas de Mensagens Confiáveis
Utilize funções para orquestrar fluxos de mensagens com serviços como filas ou o Barramento de Serviço do Azure, garantindo uma comunicação robusta e escalonável entre componentes do sistema.


Como funciona:  

As funções podem enviar e consumir mensagens de filas (como as do Armazenamento do Microsoft Azure) ou tópicos do Barramento de Serviço.


Links úteis:



Azure Functions e Armazenamento de Filas com Visual Studio Code

Gatilho do Barramento de Serviço do Azure para Functions

![image](https://github.com/user-attachments/assets/27e03c6d-5e1d-4fc4-8a1e-6e052f5aee43)


Diagrama de Sistemas de Mensagens Confiáveis

Próximas Etapas
Para começar a explorar e implementar o Azure Functions em seus projetos, confira a Introdução ao Azure Functions.

Este tutorial oferece uma visão geral estruturada dos cenários mais comuns com o Azure Functions. Experimente os exemplos, adapte-os ao seu contexto e potencialize suas aplicações sem servidor!
## Was
