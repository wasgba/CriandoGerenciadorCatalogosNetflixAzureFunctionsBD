# Cenários do Azure Functions

- Artigo
- 19/12/2024
- 9 colaboradores

Comentários

Escolha uma linguagem de programação

C#JavaJavaScriptPowerShellPythonTypeScript

Neste artigo[Processar uploads de arquivo](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-scenarios?pivots=programming-language-csharp#process-file-uploads)[Processamento de eventos e fluxo em tempo real](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-scenarios?pivots=programming-language-csharp#real-time-stream-and-event-processing)[Machine learning e IA](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-scenarios?pivots=programming-language-csharp#machine-learning-and-ai)[Executar tarefas agendadas](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-scenarios?pivots=programming-language-csharp#run-scheduled-tasks)Mostrar mais 5

Muitas vezes, criamos sistemas para reagir a uma série de eventos críticos. Se você estiver criando uma API Web, respondendo a alterações de banco de dados, processando fluxos de eventos ou mensagens, o Azure Functions pode ser usado para implementá-las.

Em muitos casos, uma função [integra-se a uma matriz de serviços de nuvem](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-triggers-bindings) para fornecer implementações ricas em recursos. Veja a seguir um conjunto comum (mas não exaustivo) de cenários para o Azure Functions.

Selecione sua linguagem de desenvolvimento na parte superior do artigo.



## Processar uploads de arquivo

Há várias maneiras de usar funções para processar arquivos dentro ou fora de um contêiner de armazenamento de blobs. Para saber mais sobre as opções para disparar em um contêiner de blob, confira [Trabalhando com blobs](https://learn.microsoft.com/pt-br/azure/azure-functions/storage-considerations#working-with-blobs) na documentação de melhores práticas.

Por exemplo, em uma solução de varejo, um sistema de parceiros pode enviar informações do catálogo de produtos como arquivos para o armazenamento de blobs. Você pode usar uma função disparada por blob para validar, transformar e processar os arquivos no sistema main conforme eles são carregados.

[![Diagrama de um processo de upload de arquivo usando o Azure Functions.](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/process-file-uploads.png)](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/process-file-uploads-expanded.png#lightbox)

Os tutoriais a seguir usam um gatilho de Blob (baseado em Grade de Eventos) para processar arquivos em um contêiner de blob:

Por exemplo, usando o gatilho de blob com uma assinatura de evento em contêineres de blob:

C#

```csharp
[FunctionName("ProcessCatalogData")]
public static async Task Run([BlobTrigger("catalog-uploads/{name}", Source = BlobTriggerSource.EventGrid, Connection = "<NAMED_STORAGE_CONNECTION>")]Stream myCatalogData, string name, ILogger log)
{
    log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myCatalogData.Length} Bytes");

    using (var reader = new StreamReader(myCatalogData))
    {
        var catalogEntry = await reader.ReadLineAsync();
        while(catalogEntry !=null)
        {
            // Process the catalog entry
            // ...

            catalogEntry = await reader.ReadLineAsync();
        }
    }
}
```

- [Carregar e analisar um arquivo com o Azure Functions e o Armazenamento de Blobs](https://learn.microsoft.com/pt-br/azure/storage/blobs/blob-upload-function-trigger)
- [Disparar o Azure Functions em contêineres de blob usando uma assinatura de evento](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-event-grid-blob-trigger?pivots=programming-language-csharp)



## Processamento de eventos e fluxo em tempo real

Muita telemetria é gerada e coletada de aplicativos em nuvem, dispositivos IoT e dispositivos de rede. O Azure Functions pode processar esses dados quase em tempo real como o caminho crítico e, em seguida, armazená-los no [Azure Cosmos DB](https://learn.microsoft.com/pt-br/azure/cosmos-db/introduction) para uso em um painel de análise.

Suas funções também podem usar gatilhos de evento de baixa latência, como a Grade de Eventos, e saídas em tempo real, como o SignalR, para processar dados quase em tempo real.

[![Diagrama de um processo de fluxo em tempo real usando o Azure Functions.](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/real-time-stream-processing.png)](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/real-time-stream-processing-expanded.png#lightbox)

Por exemplo, usando o gatilho de hubs de eventos para ler de um hub de eventos e a associação de saída para gravar em um hub de eventos depois de depurar e transformar os eventos:

C#

```csharp
[FunctionName("ProcessorFunction")]
public static async Task Run(
    [EventHubTrigger(
        "%Input_EH_Name%",
        Connection = "InputEventHubConnectionString",
        ConsumerGroup = "%Input_EH_ConsumerGroup%")] EventData[] inputMessages,
    [EventHub(
        "%Output_EH_Name%",
        Connection = "OutputEventHubConnectionString")] IAsyncCollector<SensorDataRecord> outputMessages,
    PartitionContext partitionContext,
    ILogger log)
{
    var debatcher = new Debatcher(log);
    var debatchedMessages = await debatcher.Debatch(inputMessages, partitionContext.PartitionId);

    var xformer = new Transformer(log);
    await xformer.Transform(debatchedMessages, partitionContext.PartitionId, outputMessages);
}
```

- [Streaming em escala com o Hubs de Eventos do Azure, Functions e SQL do Azure](https://github.com/Azure-Samples/streaming-at-scale/tree/main/eventhubs-functions-azuresql)
- [Streaming em escala com Hubs de Eventos do Azure, Functions e Cosmos DB](https://github.com/Azure-Samples/streaming-at-scale/tree/main/eventhubs-functions-cosmosdb)
- [Streaming em escala com o Hubs de Eventos do Azure com o produtor do Kafka, Functions com gatilho do Kafka e Cosmos DB](https://github.com/Azure-Samples/streaming-at-scale/tree/main/eventhubskafka-functions-cosmosdb)
- [Streaming em escala com o Hub IoT do Azure, Functions e SQL do Azure](https://github.com/Azure-Samples/streaming-at-scale/tree/main/iothub-functions-azuresql)
- [Gatilho de Hubs de Eventos do Azure para o Azure Functions](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-bindings-event-hubs-trigger?pivots=programming-language-csharp)
- [Gatilho do Apache Kafka para Azure Functions](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-bindings-kafka-trigger?pivots=programming-language-csharp)



## Machine learning e IA

Além do processamento de dados, o Azure Functions pode ser usados para inferir em modelos. A [Aextensão de associação do Azure OpenAI](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-bindings-openai) permite integrar facilmente recursos e comportamentos do [serviço Azure OpenAI](https://learn.microsoft.com/pt-br/azure/ai-services/openai/overview) em suas execuções de código de função.

As funções podem se conectar a recursos OpenAI para habilitar conclusões de texto e chat, usar assistentes e aproveitar inserções e pesquisa semântica.

Uma função também pode chamar um modelo TensorFlow ou serviços de IA do Azure para processar e classificar um fluxo de imagens.

[![Diagrama de um processo de aprendizado de máquina e IA usando o Azure Functions.](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/machine-learning-and-ai.png)](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/machine-learning-and-ai-expanded.png#lightbox)

- Tutorial: [preenchimento de texto usando o Azure OpenAI](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-add-openai-text-completion?pivots=programming-language-csharp)
- Exemplo: [Carregar arquivos de texto e acessar dados usando vários recursos do OpenAI](https://github.com/azure-samples/azure-functions-openai-demo).
- Exemplo: [sumarização de texto usando o Serviço de Linguagem Cognitiva de IA](https://github.com/Azure-Samples/function-csharp-ai-textsummarize)
- Amostra: [preenchimento de texto usando o Azure OpenAI](https://github.com/Azure/azure-functions-openai-extension/tree/main/samples/textcompletion/csharp-ooproc)
- Amostra: [fornecer habilidades de assistente ao seu modelo](https://github.com/Azure/azure-functions-openai-extension/tree/main/samples/assistant/csharp-ooproc)
- Amostra: [gerar inserções](https://github.com/Azure/azure-functions-openai-extension/tree/main/samples/embeddings/csharp-ooproc/Embeddings)
- Amostra: [aproveitar a pesquisa semântica](https://github.com/Azure/azure-functions-openai-extension/tree/main/samples/rag-aisearch/csharp-ooproc)



## Executar tarefas agendadas

As funções permitem que você execute seu código com base em um [agendamento cron](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-bindings-timer#usage) definido.

Saiba como [Criar uma função no portal do Azure executada segundo uma agenda](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-create-scheduled-function).

Um banco de dados do cliente de serviços financeiros, por exemplo, pode ser analisado para entradas duplicadas a cada 15 minutos para evitar que várias comunicações saiam para o mesmo cliente.

[![Diagrama de uma tarefa agendada em que uma função limpa um banco de dados a cada 15 minutos, desduplicando as entradas com base na lógica de negócios.](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/scheduled-task.png)](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/scheduled-task-expanded.png#lightbox)

C#

```csharp
[FunctionName("TimerTriggerCSharp")]
public static void Run([TimerTrigger("0 */15 * * * *")]TimerInfo myTimer, ILogger log)
{
    if (myTimer.IsPastDue)
    {
        log.LogInformation("Timer is running late!");
    }
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");

    // Perform the database deduplication
}
```

- [Gatilho de temporizador para o Azure Functions](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-bindings-timer?pivots=programming-language-csharp)



## Criar uma API Web escalonável

Uma função disparada por HTTP define um ponto de extremidade HTTP. Esses pontos de extremidade executam código de função que pode se conectar a outros serviços diretamente ou usando extensões de associação. Você pode compor os pontos de extremidade em uma API baseada na Web.

Você também pode usar um ponto de extremidade de função disparada por HTTP como uma integração de webhook, como os webhooks do GitHub. Dessa forma, você pode criar funções que processam dados de eventos do GitHub. Para saber mais, confira [Monitorar eventos do GitHub usando um webhook com o Azure Functions](https://learn.microsoft.com/pt-br/training/modules/monitor-github-events-with-a-function-triggered-by-a-webhook/).

[![Diagrama de processamento de uma solicitação HTTP usando o Azure Functions.](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/scalable-web-api.png)](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/scalable-web-api-expanded.png#lightbox)

Para obter exemplos, confira o seguinte:

C#

```csharp
[FunctionName("InsertName")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req,
    [CosmosDB(
        databaseName: "my-database",
        collectionName: "my-container",
        ConnectionStringSetting = "CosmosDbConnectionString")]IAsyncCollector<dynamic> documentsOut,
    ILogger log)
{
    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    string name = data?.name;

    if (name == null)
    {
        return new BadRequestObjectResult("Please pass a name in the request body json");
    }

    // Add a JSON document to the output container.
    await documentsOut.AddAsync(new
    {
        // create a random ID
        id = System.Guid.NewGuid().ToString(), 
        name = name
    });

    return new OkResult();
}
```

- Artigo: [criar APIs sem servidor no Visual Studio usando o Azure Functions e a integração do Gerenciamento de API](https://learn.microsoft.com/pt-br/azure/azure-functions/openapi-apim-integrate-visual-studio)
- Treinamento: [expor vários aplicativos de funções como uma API consistente, usando o Gerenciamento de API do Azure](https://learn.microsoft.com/pt-br/training/modules/build-serverless-api-with-functions-api-management/)
- Exemplo: [Implemente o padrão de nó geográfico implantando a API em nós geográficos em regiões distribuídas do Azure.](https://github.com/mspnp/geode-pattern-accelerator)
- Artigo: [Gatilho HTTP do Azure Functions](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-bindings-http-webhook?pivots=programming-language-csharp)
- Exemplo: [aplicativo Web com uma API C# e BD SQL do Azure em Aplicativos Web Estáticos e Functions](https://learn.microsoft.com/pt-br/samples/azure-samples/todo-csharp-sql-swa-func/todo-csharp-sql-swa-func/)



## Criar um fluxo de trabalho sem servidor

As funções geralmente são o componente de computação em uma topologia de fluxo de trabalho sem servidor, como um fluxo de trabalho dos Aplicativos Lógicos. Você também pode criar orquestrações de execução longa usando a extensão Durable Functions. Para obter mais informações, confira [Visão geral do Durable Functions](https://learn.microsoft.com/pt-br/azure/azure-functions/durable/durable-functions-overview).

[![Um diagrama de combinação de uma série de fluxos de trabalho sem servidor específicos usando o Azure Functions.](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/build-a-serverless-workflow.png)](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/build-a-serverless-workflow-expanded.png#lightbox)

- Tutorial: [criar uma função que se integra aos Aplicativos Lógicos do Azure](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-twitter-email)
- Início rápido: [criar sua primeira função durável no Azure usando C#](https://learn.microsoft.com/pt-br/azure/azure-functions/durable/durable-functions-isolated-create-first-csharp)
- Treinamento: [implantar as APIs sem servidor com o Azure Functions, Aplicativos Lógicos e Banco de Dados SQL do Azure](https://learn.microsoft.com/pt-br/training/modules/deploy-backend-apis/)



## Responder a alterações no banco de dados

Há processos em que talvez seja necessário registrar, auditar ou executar alguma outra operação quando os dados armazenados forem alterados. Os gatilhos de funções fornecem uma boa maneira de ser notificado sobre alterações de dados para inicializar essa operação.

[![Diagrama de uma função usada para responder a alterações no banco de dados.](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/respond-to-database-changes.png)](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/respond-to-database-changes-expanded.png#lightbox)

Considere estes exemplos:

- Artigo: [conectar o Azure Functions ao Azure Cosmos DB usando o Visual Studio Code](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-add-output-binding-cosmos-db-vs-code?pivots=programming-language-csharp&tabs=isolated-process)
- Artigo: [conectar o Azure Functions ao Banco de Dados SQL do Azure usando o Visual Studio Code](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-add-output-binding-azure-sql-vs-code?pivots=programming-language-csharp&tabs=isolated-process)
- Artigo: [usar o Azure Functions para limpar um Banco de Dados SQL do Azure](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-scenario-database-table-cleanup)



## Criar sistemas de mensagens confiáveis

Você pode usar o Functions com os serviços de mensagens do Azure para criar soluções avançadas de mensagens controladas por eventos.

Por exemplo, você pode usar gatilhos em filas do Armazenamento do Microsoft Azure como uma maneira de encadear uma série de execuções de função. Ou use filas e gatilhos do barramento de serviço para um sistema de pedidos online.

[![Diagrama do Azure Functions em um sistema de mensagens confiável.](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/create-reliable-message-systems.png)](https://learn.microsoft.com/pt-br/azure/azure-functions/media/functions-scenarios/create-reliable-message-systems-expanded.png#lightbox)

Estes artigos mostram como gravar a saída em uma fila de armazenamento:

- Artigo: [conectar o Azure Functions ao Armazenamento do Microsoft Azure usando o Visual Studio Code](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-add-output-binding-storage-queue-vs-code?pivots=programming-language-csharp&tabs=isolated-process)
- Artigo: [criar uma função disparada pelo Armazenamento de Filas do Azure (portal do Azure)](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-create-storage-queue-triggered-function)

E esses artigos mostram como disparar de uma fila ou tópico do Barramento de Serviço do Azure.

- [Gatilho do Barramento de Serviço do Azure para Azure Functions](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-bindings-service-bus-trigger?pivots=programming-language-csharp)



## Próximas etapas

[Introdução ao Azure Functions](https://learn.microsoft.com/pt-br/azure/azure-functions/functions-get-started)