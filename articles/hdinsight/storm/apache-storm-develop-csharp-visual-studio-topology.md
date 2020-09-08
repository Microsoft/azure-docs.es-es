---
title: 'Topologías de Apache Storm con Visual Studio y C#: Azure HDInsight'
description: Aprenda a crear topologías de Storm en C#. Cree una topología de recuento de palabras en Visual Studio con las herramientas de Hadoop para Visual Studio.
ROBOTS: NOINDEX
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 12/31/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 271f62625433a6651ba0e3230a62be51e5147f3e
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89000199"
---
# <a name="develop-c-topologies-for-apache-storm-by-using-the-data-lake-tools-for-visual-studio"></a>Desarrollo de topologías de C# para Apache Storm con Herramientas de Azure Data Lake para Visual Studio

Aprenda a crear una topología de Apache Storm en C# con herramientas de Azure Data Lake (Apache Hadoop) para Visual Studio. Este documento le guía a través del proceso para crear un proyecto de Storm en Visual Studio, probarlo localmente e implementarlo en un clúster de Apache Storm en Azure HDInsight.

También aprenderá a crear topologías híbridas que usan componentes de C# y Java.

Las topologías de C# usan .NET 4.5 y emplean Mono para su ejecución en el clúster de HDInsight. Para más información sobre las posibles incompatibilidades, consulte [Mono compatibility](https://www.mono-project.com/docs/about-mono/compatibility/). Para usar una topología de C#, debe actualizar el paquete de NuGet `Microsoft.SCP.Net.SDK` que se usa en el proyecto a la versión 0.10.0.6 o posterior. La versión del paquete también debe coincidir con la versión principal de Storm instalada en HDInsight.

| Versión de HDInsight | Versión de Apache Storm | Versión de SCP.NET | Versión de Mono predeterminada |
|:-----------------:|:-------------:|:---------------:|:--------------------:|
| 3.4 | 0.10.0.x | 0.10.0.x | 3.2.8 |
| 3,5 | 1.0.2.x | 1.0.0.x | 4.2.1 |
| 3.6 | 1.1.0.x | 1.0.0.x | 4.2.8 |

## <a name="prerequisite"></a>Requisito previo

Un clúster de Apache Storm en HDInsight. Consulte el artículo sobre la [Creación de clústeres de Apache Hadoop mediante Azure Portal](../hdinsight-hadoop-create-linux-clusters-portal.md) y seleccione **Storm** como **Tipo de clúster**.

## <a name="install-visual-studio"></a>Instalación de Visual Studio

Puede desarrollar topologías C# con SCP.NET mediante [Visual Studio](https://visualstudio.microsoft.com/downloads/). Las instrucciones que se incluyen aquí usan Visual Studio 2019, pero también puede usar versiones anteriores de Visual Studio.

## <a name="install-data-lake-tools-for-visual-studio"></a>Instalación de Herramientas de Data Lake para Visual Studio

Para instalar Herramientas de Data Lake para Visual Studio, siga los pasos de [Get started using Data Lake tools for Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md#install-data-lake-tools-for-visual-studio) (Introducción al uso de Herramientas de Data Lake para Visual Studio).

## <a name="install-java"></a>Instalar Java

Cuando se envía una topología de Storm desde Visual Studio, SCP.NET genera un archivo zip que contiene la topología y las dependencias. Java se usa para crear estos archivos zip, ya que emplea un formato más compatible con los clústeres basados en Linux.

1. Instale Java Development Kit (JDK) 7 o posterior en el entorno de desarrollo. Puede obtener el JDK de Oracle en [Oracle](https://openjdk.java.net/). También puede usar [otras distribuciones de Java](/java/azure/jdk/).

2. Establezca la variable de entorno `JAVA_HOME` en el directorio que contiene Java.

3. Establezca la variable de entorno `PATH` para incluir el directorio `%JAVA_HOME%\bin`.

Puede compilar y ejecutar la siguiente aplicación de consola de C# para comprobar que Java y el JDK se han instalado correctamente:

```csharp
using System;
using System.IO;
namespace ConsoleApplication2
{
   class Program
   {
       static void Main(string[] args)
       {
           string javaHome = Environment.GetEnvironmentVariable("JAVA_HOME");
           if (!string.IsNullOrEmpty(javaHome))
           {
               string jarExe = Path.Combine(javaHome + @"\bin", "jar.exe");
               if (File.Exists(jarExe))
               {
                   Console.WriteLine("JAVA Is Installed properly");
                    return;
               }
               else
               {
                   Console.WriteLine("A valid JAVA JDK is not found. Looks like JRE is installed instead of JDK.");
               }
           }
           else
           {
             Console.WriteLine("A valid JAVA JDK is not found. JAVA_HOME environment variable is not set.");
           }
       }  
   }
}
```

## <a name="apache-storm-templates"></a>Plantillas de Apache Storm

Herramientas de Data Lake para Visual Studio proporciona las siguientes plantillas:

| Tipo de proyecto | Muestra |
| --- | --- |
| Storm Application |Un proyecto vacío de topología de Storm. |
| Storm Azure SQL Writer Sample |Cómo escribir en Azure SQL Database. |
| Storm Azure Cosmos DB Reader Sample |Cómo leer desde Azure Cosmos DB. |
| Storm Azure Cosmos DB Writer Sample |Cómo escribir en Azure Cosmos DB. |
| Storm EventHub Reader Sample |Cómo leer desde Azure Event Hubs. |
| Storm EventHub Writer Sample |Cómo escribir en Azure Event Hubs. |
| Storm HBase Reader Sample |Cómo leer desde HBase en clústeres de HDInsight. |
| Storm HBase Writer Sample |Cómo escribir en HBase en clústeres de HDInsight. |
| Storm Hybrid Sample |Cómo usar un componente de Java. |
| Storm Sample |Una topología de recuento de palabras básica. |

> [!WARNING]  
> No todas las plantillas funcionan con HDInsight basado en Linux. Es posible que los paquetes NuGet que usan las plantillas no sean compatibles con Mono. Para identificar posibles problemas, consulte [Mono compatibility](https://www.mono-project.com/docs/about-mono/compatibility/) y use el [Analizador de portabilidad de .NET](../hdinsight-hadoop-migrate-dotnet-to-linux.md#automated-portability-analysis).

En los pasos de este documento, usará el tipo de proyecto Storm Application básico para crear una topología.

### <a name="apache-hbase-templates"></a>Plantillas de Apache HBase

Las plantillas de lector y escritor de HBase usan la API de REST de HBase, no la API de Java de HBase, para comunicarse con un HBase en un clúster de HDInsight.

### <a name="eventhub-templates"></a>Plantillas de EventHub

> [!IMPORTANT]  
> Es posible que el componente de spout de EventHub basado en Java incluido en la plantilla de lector de EventHub no funcione con Storm en HDInsight versión 3.5 o posterior. Hay una versión actualizada de este componente disponible en [GitHub](https://github.com/hdinsight/hdinsight-storm-examples/tree/master/HDI3.5/lib).

Para obtener una topología de ejemplo que usa este componente y funciona con Storm en HDInsight 3.5, vea [GitHub](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub).

## <a name="create-a-c-topology"></a>Creación de una topología de C#

Para crear un proyecto de topología C# en Visual Studio:

1. Abra Visual Studio.

1. En la ventana **Inicio**, seleccione **Crear un proyecto**.

1. En la ventana **Crear un nuevo proyecto**, desplácese a la opción **Aplicación Storm** y selecciónela, y luego seleccione **Siguiente**.

1. En la ventana **Configure su nuevo proyecto**, en el campo **Nombre de proyecto**, escriba el nombre *WordCount*, vaya a o cree una ruta de acceso al directorio en **Ubicación** y luego seleccione **Crear**.

    ![Aplicación Storm, cuadro de diálogo Configure su nuevo proyecto, Visual Studio](./media/apache-storm-develop-csharp-visual-studio-topology/apache-storm-new-project.png)

Después de crear el proyecto, debe tener los siguientes archivos:

* *Program.cs*: La definición de la topología para el proyecto. Una topología predeterminada que consta de un spout y un bolt se crea de manera predeterminada.

* *Spout.cs*: un spout de ejemplo que emite números aleatorios.

* *Bolt.cs*: un bolt de ejemplo que mantiene un recuento de los números emitidos por el spout.

Cuando se crea el proyecto, NuGet descarga el [paquete SCP.NET](https://www.nuget.org/packages/Microsoft.SCP.Net.SDK/) más reciente.

### <a name="implement-the-spout"></a>Implementación del spout

Luego, agregue el código para spout, que se usa para leer los datos de una topología de un origen externo. Este spout emite aleatoriamente una frase en la topología.

1. Abra *Spout.cs*. Los componentes principales de un spout son:

   * `NextTuple`: llamado por Storm cuando se permite que el spout emita nuevas tuplas.

   * `Ack` (solo topología transaccional): controla las confirmaciones que iniciaron otros componentes de la topología para las tuplas enviadas desde el spout. La confirmación de una tupla permite que el spout conozca que se ha procesado correctamente por componentes de bajada.

   * `Fail` (solo topología transaccional): controla las tuplas que producen un error al procesar otros componentes de la topología. Implementar un método `Fail` le permite volver a emitir la tupla para que se pueda procesar de nuevo.

2. Reemplace el contenido de la clase `Spout` por el texto siguiente:

    ```csharp
    private Context ctx;
    private Random r = new Random();
    string[] sentences = new string[] {
        "the cow jumped over the moon",
        "an apple a day keeps the doctor away",
        "four score and seven years ago",
        "snow white and the seven dwarfs",
        "i am at two with nature"
    };

    public Spout(Context ctx)
    {
        // Set the instance context
        this.ctx = ctx;

        Context.Logger.Info("Generator constructor called");

        // Declare Output schema
        Dictionary<string, List<Type>> outputSchema = new Dictionary<string, List<Type>>();
        // The schema for the default output stream is
        // a tuple that contains a string field
        outputSchema.Add("default", new List<Type>() { typeof(string) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(null, outputSchema));
    }

    // Get an instance of the spout
    public static Spout Get(Context ctx, Dictionary<string, Object> parms)
    {
        return new Spout(ctx);
    }

    public void NextTuple(Dictionary<string, Object> parms)
    {
        Context.Logger.Info("NextTuple enter");
        // The sentence to be emitted
        string sentence;

        // Get a random sentence
        sentence = sentences[r.Next(0, sentences.Length - 1)];
        Context.Logger.Info("Emit: {0}", sentence);
        // Emit it
        this.ctx.Emit(new Values(sentence));

        Context.Logger.Info("NextTuple exit");
    }

    public void Ack(long seqId, Dictionary<string, Object> parms)
    {
        // Only used for transactional topologies
    }

    public void Fail(long seqId, Dictionary<string, Object> parms)
    {
        // Only used for transactional topologies
    }
    ```

### <a name="implement-the-bolts"></a>Implementación de los bolts

Ahora cree dos bolts Storm en este ejemplo:

1. Elimine el archivo *Bolt.cs* existente del proyecto.

2. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Agregar** > **Nuevo elemento**. En la lista, seleccione **Bolt de Storm** y escriba *Splitter.cs* como nombre. En el código del nuevo archivo, cambie el nombre del espacio de nombres a `WordCount`. Repita este proceso para crear un segundo bolt denominado *Counter.cs*.

   * *Splitter.cs*: implementa un bolt que divide la frase en palabras individuales y emite una nueva secuencia de palabras.

   * *Counter.cs*: implementa un bolt que cuenta cada palabra y emite una nueva secuencia de palabras y el recuento de cada palabra.

     > [!NOTE]  
     > Estos bolts leen y escriben en las secuencias, pero también se puede usar un bolt para comunicarse con orígenes como una base de datos o un servicio.

3. Abra *Splitter.cs*. Solo tiene un método de manera predeterminada: `Execute`. Se llama al método `Execute` cuando el bolt recibe una tupla para el procesamiento. En este caso, puede leer y procesar las tuplas entrantes y emitir tuplas salientes.

4. Reemplace el contenido de la clase `Splitter` por el código siguiente:

    ```csharp
    private Context ctx;

    // Constructor
    public Splitter(Context ctx)
    {
        Context.Logger.Info("Splitter constructor called");
        this.ctx = ctx;

        // Declare Input and Output schemas
        Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
        // Input contains a tuple with a string field (the sentence)
        inputSchema.Add("default", new List<Type>() { typeof(string) });
        Dictionary<string, List<Type>> outputSchema = new Dictionary<string, List<Type>>();
        // Outbound contains a tuple with a string field (the word)
        outputSchema.Add("default", new List<Type>() { typeof(string) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, outputSchema));
    }

    // Get a new instance of the bolt
    public static Splitter Get(Context ctx, Dictionary<string, Object> parms)
    {
        return new Splitter(ctx);
    }

    // Called when a new tuple is available
    public void Execute(SCPTuple tuple)
    {
        Context.Logger.Info("Execute enter");

        // Get the sentence from the tuple
        string sentence = tuple.GetString(0);
        // Split at space characters
        foreach (string word in sentence.Split(' '))
        {
            Context.Logger.Info("Emit: {0}", word);
            //Emit each word
            this.ctx.Emit(new Values(word));
        }

        Context.Logger.Info("Execute exit");
    }
    ```

5. Abra *Counter.cs* y sustituya el contenido de la clase por el código siguiente:

    ```csharp
    private Context ctx;

    // Dictionary for holding words and counts
    private Dictionary<string, int> counts = new Dictionary<string, int>();

    // Constructor
    public Counter(Context ctx)
    {
        Context.Logger.Info("Counter constructor called");
        // Set instance context
        this.ctx = ctx;

        // Declare Input and Output schemas
        Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
        // A tuple containing a string field - the word
        inputSchema.Add("default", new List<Type>() { typeof(string) });

        Dictionary<string, List<Type>> outputSchema = new Dictionary<string, List<Type>>();
        // A tuple containing a string and integer field - the word and the word count
        outputSchema.Add("default", new List<Type>() { typeof(string), typeof(int) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, outputSchema));
    }

    // Get a new instance
    public static Counter Get(Context ctx, Dictionary<string, Object> parms)
    {
        return new Counter(ctx);
    }

    // Called when a new tuple is available
    public void Execute(SCPTuple tuple)
    {
        Context.Logger.Info("Execute enter");

        // Get the word from the tuple
        string word = tuple.GetString(0);
        // Do we already have an entry for the word in the dictionary?
        // If no, create one with a count of 0
        int count = counts.ContainsKey(word) ? counts[word] : 0;
        // Increment the count
        count++;
        // Update the count in the dictionary
        counts[word] = count;

        Context.Logger.Info("Emit: {0}, count: {1}", word, count);
        // Emit the word and count information
        this.ctx.Emit(Constants.DEFAULT_STREAM_ID, new List<SCPTuple> { tuple }, new Values(word, count));
        Context.Logger.Info("Execute exit");
    }
    ```

### <a name="define-the-topology"></a>Definición de la topología

Los spouts y bolts se organizan en un gráfico, que define cómo fluyen los datos entre componentes. Para esta topología, el gráfico es como sigue:

![Diagrama de organización de componentes de spout y bolt, topología de Storm](./media/apache-storm-develop-csharp-visual-studio-topology/word-count-topology1.png)

Spout emite oraciones que se distribuyen a las instancias del bolt Splitter. El bolt divisor divide las frases en palabras, que se distribuyen en el bolt contador.

Dado que la instancia de Counter guarda localmente el recuento de palabras, debería asegurarse de que determinadas palabras fluyan a la misma instancia del bolt Counter. Cada instancia realiza el seguimiento de palabras específicas. Dado que el bolt divisor no mantiene ningún estado, realmente no importa qué instancia del divisor recibe cada frase.

Abra *Program.cs*. El método importante aquí es `GetTopologyBuilder`, que se usa para definir la topología que se envía a Storm. Reemplace el contenido de `GetTopologyBuilder` por el código siguiente para implementar la topología que se describe anteriormente:

```csharp
// Create a new topology named 'WordCount'
TopologyBuilder topologyBuilder = new TopologyBuilder(
    "WordCount" + DateTime.Now.ToString("yyyyMMddHHmmss"));

// Add the spout to the topology.
// Name the component 'sentences'
// Name the field that is emitted as 'sentence'
topologyBuilder.SetSpout(
    "sentences",
    Spout.Get,
    new Dictionary<string, List<string>>()
    {
        {Constants.DEFAULT_STREAM_ID, new List<string>(){"sentence"}}
    },
    1);
// Add the splitter bolt to the topology.
// Name the component 'splitter'
// Name the field that is emitted 'word'
// Use suffleGrouping to distribute incoming tuples
//   from the 'sentences' spout across instances
//   of the splitter
topologyBuilder.SetBolt(
    "splitter",
    Splitter.Get,
    new Dictionary<string, List<string>>()
    {
        {Constants.DEFAULT_STREAM_ID, new List<string>(){"word"}}
    },
    1).shuffleGrouping("sentences");

// Add the counter bolt to the topology.
// Name the component 'counter'
// Name the fields that are emitted 'word' and 'count'
// Use fieldsGrouping to ensure that tuples are routed
//   to counter instances based on the contents of field
//   position 0 (the word). This could also have been
//   List<string>(){"word"}.
//   This ensures that the word 'jumped', for example, will always
//   go to the same instance
topologyBuilder.SetBolt(
    "counter",
    Counter.Get,
    new Dictionary<string, List<string>>()
    {
        {Constants.DEFAULT_STREAM_ID, new List<string>(){"word", "count"}}
    },
    1).fieldsGrouping("splitter", new List<int>() { 0 });

// Add topology config
topologyBuilder.SetTopologyConfig(new Dictionary<string, string>()
{
    {"topology.kryo.register","[\"[B\"]"}
});

return topologyBuilder;
```

## <a name="submit-the-topology"></a>Envío de la topología

Ahora está listo para enviar la topología al clúster de HDInsight.

1. Vaya a **Ver** > **Explorador de servidores**.

1. Haga clic con el botón derecho en **Azure**, seleccione **Conectar a la suscripción de Microsoft Azure...** y complete el proceso de inicio de sesión.

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Enviar a Storm en HDInsight**.

1. En el cuadro de diálogo **Enviar topología**, en la lista desplegable **Clúster de Storm**, elija el clúster de Storm en HDInsight y seleccione **Enviar**. Puede comprobar si el envío es correcto al consultar el panel **Salida**.

    Cuando la topología se envió correctamente, debe aparecer la ventana **Vista Topologías de Storm** del clúster. Seleccione la topología **WordCount** de la lista para consultar la información sobre la topología en ejecución.

    ![Ventana Vista Topologías de Storm, clúster de HDInsight, Visual Studio](./media/apache-storm-develop-csharp-visual-studio-topology/storm-topology-view.png)

    > [!NOTE]  
    > También puede ver **topologías de Storm** desde el **Explorador de servidores**. Expanda **Azure** > **HDInsight**, haga clic con el botón derecho en un clúster de Storm en HDInsight y luego seleccione **Ver topologías de Storm**.

    Para ver información sobre los componentes de la topología, seleccione un componente en el diagrama.

1. En la sección **Resumen de topología**, seleccione **Terminar** para detener la topología.

    > [!NOTE]  
    > Las topologías de Storm continúan ejecutándose hasta que se desactiven o se elimine el clúster.

## <a name="transactional-topology"></a>Topología transaccional

La topología anterior no es transaccional. Los componentes de la topología no implementan la funcionalidad para reproducir los mensajes. Para ver un ejemplo de una topología transaccional, cree un proyecto y seleccione **Muestra de Storm** como tipo de proyecto.

Las topologías transaccionales implementan lo siguiente para que admitan la reproducción de datos:

* **Almacenamiento en caché de metadatos**: el spout debe almacenar metadatos sobre los datos emitidos para que los datos puedan recuperarse y volver a emitirse si se produce un error. Dado que los datos que emite la muestra son pequeños, los datos sin procesar de cada tupla se almacenan en un diccionario para la reproducción.

* **Ack**: cada bolt de la topología puede llamar a `this.ctx.Ack(tuple)` para confirmar que ha procesado una tupla correctamente. Una vez que todos los bolts han confirmado la tupla, se llama al método `Ack` del spout. El método `Ack` permite al spout quitar los datos que se almacenaron en caché para la reproducción.

* **Fail**: cada bolt puede llamar a `this.ctx.Fail(tuple)` para indicar que se produjo un error al procesar una tupla. El error se propaga al método `Fail` del spout, donde la tupla puede reproducirse mediante metadatos almacenados en caché.

* **Id. de secuencia**: al emitir una tupla, se puede especificar un identificador de secuencia único. Este valor identifica la tupla para el procesamiento de reproducción (`Ack` y `Fail`). Por ejemplo, el spout del proyecto **Ejemplo de Storm** usa la siguiente llamada de método al emitir datos:

  `this.ctx.Emit(Constants.DEFAULT_STREAM_ID, new Values(sentence), lastSeqId);`

  Este código emite una tupla que contiene una oración a la secuencia predeterminada, con el valor de id. de secuencia incluido en `lastSeqId`. En este ejemplo, `lastSeqId` se incrementa con cada tupla emitida.

Como se muestra en el proyecto **Muestra de Storm**, se puede establecer si un componente es transaccional en runtime, según la configuración.

## <a name="hybrid-topology-with-c-and-java"></a>Topología híbrida con C# y Java

También puede usar Herramientas de Data Lake para Visual Studio para crear topologías híbridas, donde algunos componentes son de C# y otros de Java.

Para ver un ejemplo de una topología híbrida, cree un proyecto y seleccione **Muestra híbrida de Storm**. Este tipo de ejemplo ilustra los conceptos siguientes:

* **Spout de Java** y **bolt de C#** : Definido en la clase `HybridTopology_javaSpout_csharpBolt`.

  Una versión transaccional se define en la clase `HybridTopologyTx_javaSpout_csharpBolt`.

* **Spout de C#** y **bolt de Java**: Definido en la clase `HybridTopology_csharpSpout_javaBolt`.

  Una versión transaccional se define en la clase `HybridTopologyTx_csharpSpout_javaBolt`.

  > [!NOTE]  
  > Esta versión también muestra cómo usar código de Clojure desde un archivo de texto como un componente de Java.

Para cambiar la topología que se usa cuando se envía el proyecto, mueva la instrucción `[Active(true)]` a la topología que quiere usar antes de enviarla al clúster.

> [!NOTE]  
> Todos los archivos de Java necesarios se ofrecen como parte de este proyecto en la carpeta *JavaDependency* .

Tenga en cuenta lo siguiente al crear y enviar una topología híbrida:

* Use `JavaComponentConstructor` para crear una instancia de la clase de Java para un spout o bolt.

* Use `microsoft.scp.storm.multilang.CustomizedInteropJSONSerializer` para serializar los datos dentro o fuera de los componentes de Java a partir de objetos de Java a JSON.

* Al enviar la topología al servidor, debe usar la opción **Configuraciones adicionales** para especificar las **rutas de acceso del archivo Java**. La ruta de acceso especificada debe ser el directorio en el que se incluyen los archivos JAR que contienen las clases de Java.

### <a name="azure-event-hubs"></a>Azure Event Hubs

La versión 0.9.4.203 de SCP.NET presenta una nueva clase y un nuevo método específicos para trabajar con el spout del centro de eventos (un spout de Java que lee desde Event Hubs). Al crear una topología que usa un spout Event Hub (por ejemplo, mediante la plantilla **Ejemplo del lector de EventHub de Storm**), use las siguientes API:

* Clase `EventHubSpoutConfig`: crea un objeto que contiene la configuración para el componente spout.

* Método `TopologyBuilder.SetEventHubSpout`: Agrega el componente de spout de Event Hub a la topología.

> [!NOTE]  
> Aún debe usar el objeto `CustomizedInteropJSONSerializer` para serializar los datos que genera el spout.

## <a name="use-configurationmanager"></a>Uso de ConfigurationManager

No use **ConfigurationManager** para recuperar valores de configuración de los componentes de bolt y spout. Si lo hace, puede generar una excepción de puntero nulo. En su lugar, pase la configuración del proyecto a la topología de Storm como un par de clave y valor en el contexto de la topología. Cada componente que se basa en los valores de configuración debe recuperarlos del contexto durante la inicialización.

El código siguiente muestra cómo recuperar estos valores:

```csharp
public class MyComponent : ISCPBolt
{
    // To hold configuration information loaded from context
    Configuration configuration;
    ...
    public MyComponent(Context ctx, Dictionary<string, Object> parms)
    {
        // Save a copy of the context for this component instance
        this.ctx = ctx;
        // If it exists, load the configuration for the component
        if(parms.ContainsKey(Constants.USER_CONFIG))
        {
            this.configuration = parms[Constants.USER_CONFIG] 
                as System.Configuration.Configuration;
        }
        // Retrieve the value of "Foo" from configuration
        var foo = this.configuration.AppSettings.Settings["Foo"].Value;
    }
    ...
}
```

Si usa un método `Get` para devolver una instancia de su componente, debe asegurarse de que pasa los parámetros `Context` y `Dictionary<string, Object>` al constructor. El ejemplo siguiente es un método `Get` básico que pasa estos valores correctamente:

```csharp
public static MyComponent Get(Context ctx, Dictionary<string, Object> parms)
{
    return new MyComponent(ctx, parms);
}
```

## <a name="how-to-update-scpnet"></a>Cómo actualizar SCP.NET

Las versiones recientes de SCP.NET admiten la actualización de paquetes a través de NuGet. Cuando está disponible una nueva actualización, recibirá una notificación de actualización. Para comprobar manualmente si hay una actualización, siga estos pasos:

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet**.

2. En el administrador de paquetes, seleccione **Actualizaciones**. Si hay disponible una actualización para el paquete de registros de SCP.NET, se muestra en la lista. Seleccione **Actualizar** para el paquete y luego, en el cuadro de diálogo **Vista previa de los cambios** y seleccione **Aceptar** para instalarlo.

> [!IMPORTANT]  
> Si el proyecto se creó con una versión anterior de SCP.NET que no usó NuGet, debe realizar los pasos siguientes para actualizar a una versión más reciente:
>
> 1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet**.
> 2. Mediante el campo **Buscar**, busque el objeto `Microsoft.SCP.Net.SDK` y agréguelo al proyecto.

## <a name="troubleshoot-common-issues-with-topologies"></a>Solución de problemas comunes de las topologías

### <a name="null-pointer-exceptions"></a>Excepciones de puntero nulo

Al usar una topología de C# con un clúster de HDInsight basado en Linux, los componentes de bolt y spout que usan **ConfigurationManager** para leer los valores de configuración en runtime pueden devolver excepciones de puntero nulo.

La configuración del proyecto se pasa a la topología de Storm como un par de clave y valor en el contexto de la topología. Se puede recuperar del objeto de diccionario que se pasó a los componentes cuando se inicializaron.

Para más información, consulte la sección [Usar ConfigurationManager](#use-configurationmanager) de este documento.

### <a name="systemtypeloadexception"></a>System.TypeLoadException

Al usar una topología de C# con un clúster de HDInsight basado en Linux, puede aparecer el siguiente error:

`System.TypeLoadException: Failure has occurred while loading a type.`

Este error se produce cuando se usa un archivo binario que no es compatible con la versión de .NET que admite Mono.

En el caso de los clústeres de HDInsight basados en Linux, debe asegurarse de que el proyecto use archivos binarios compilados para .NET 4.5.

### <a name="test-a-topology-locally"></a>Prueba de una topología localmente

Aunque es fácil implementar una topología en un clúster, en algunos casos, puede que sea necesario probar una topología localmente. Siga los pasos que se muestran a continuación para ejecutar y probar localmente la topología de ejemplo de este artículo en su entorno de desarrollo.

> [!WARNING]  
> Las pruebas locales solo funcionan para topologías básicas de C#. No se pueden usar pruebas locales para las topologías híbridas o para las topologías que usan varias secuencias.

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Propiedades**. En las propiedades del proyecto. Luego, cambie el valor de **Tipo de salida** a **Aplicación de consola**.

   ![Aplicación de Storm de HDInsight, propiedades del proyecto, tipo de salida](./media/apache-storm-develop-csharp-visual-studio-topology/hdi-output-type-window.png)

   > [!NOTE]
   > No olvide cambiar el **tipo de salida** de nuevo a **Biblioteca de clases** antes de implementar la topología en un clúster.

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Agregar** > **Nuevo elemento**. Seleccione **Clase** y escriba *LocalTest.cs* como nombre de clase. Por último, seleccione **Agregar**.

1. Abra *LocalTest.cs* y agregue la siguiente instrucción `using` en la parte superior:

    ```csharp
    using Microsoft.SCP;
    ```

1. Use el siguiente código como contenido de la clase `LocalTest`:

    ```csharp
    // Drives the topology components
    public void RunTestCase()
    {
        // An empty dictionary for use when creating components
        Dictionary<string, Object> emptyDictionary = new Dictionary<string, object>();

        #region Test the spout
        {
            Console.WriteLine("Starting spout");
            // LocalContext is a local-mode context that can be used to initialize
            // components in the development environment.
            LocalContext spoutCtx = LocalContext.Get();
            // Get a new instance of the spout, using the local context
            Spout sentences = Spout.Get(spoutCtx, emptyDictionary);

            // Emit 10 tuples
            for (int i = 0; i < 10; i++)
            {
                sentences.NextTuple(emptyDictionary);
            }
            // Use LocalContext to persist the data stream to file
            spoutCtx.WriteMsgQueueToFile("sentences.txt");
            Console.WriteLine("Spout finished");
        }
        #endregion

        #region Test the splitter bolt
        {
            Console.WriteLine("Starting splitter bolt");
            // LocalContext is a local-mode context that can be used to initialize
            // components in the development environment.
            LocalContext splitterCtx = LocalContext.Get();
            // Get a new instance of the bolt
            Splitter splitter = Splitter.Get(splitterCtx, emptyDictionary);

            // Set the data stream to the data created by the spout
            splitterCtx.ReadFromFileToMsgQueue("sentences.txt");
            // Get a batch of tuples from the stream
            List<SCPTuple> batch = splitterCtx.RecvFromMsgQueue();
            // Process each tuple in the batch
            foreach (SCPTuple tuple in batch)
            {
                splitter.Execute(tuple);
            }
            // Use LocalContext to persist the data stream to file
            splitterCtx.WriteMsgQueueToFile("splitter.txt");
            Console.WriteLine("Splitter bolt finished");
        }
        #endregion

        #region Test the counter bolt
        {
            Console.WriteLine("Starting counter bolt");
            // LocalContext is a local-mode context that can be used to initialize
            // components in the development environment.
            LocalContext counterCtx = LocalContext.Get();
            // Get a new instance of the bolt
            Counter counter = Counter.Get(counterCtx, emptyDictionary);

            // Set the data stream to the data created by splitter bolt
            counterCtx.ReadFromFileToMsgQueue("splitter.txt");
            // Get a batch of tuples from the stream
            List<SCPTuple> batch = counterCtx.RecvFromMsgQueue();
            // Process each tuple in the batch
            foreach (SCPTuple tuple in batch)
            {
                counter.Execute(tuple);
            }
            // Use LocalContext to persist the data stream to file
            counterCtx.WriteMsgQueueToFile("counter.txt");
            Console.WriteLine("Counter bolt finished");
        }
        #endregion
    }
    ```

    Dedique un momento para leer los comentarios del código. Este código usa `LocalContext` para ejecutar los componentes en el entorno de desarrollo. Conserva el flujo de datos entre los componentes en los archivos de texto de la unidad local.

1. Abra *Program.cs* y agregue el siguiente código al método `Main`:

    ```csharp
    Console.WriteLine("Starting tests");
    System.Environment.SetEnvironmentVariable("microsoft.scp.logPrefix", "WordCount-LocalTest");
    // Initialize the runtime
    SCPRuntime.Initialize();

    //If we are not running under the local context, throw an error
    if (Context.pluginType != SCPPluginType.SCP_NET_LOCAL)
    {
        throw new Exception(string.Format("unexpected pluginType: {0}", Context.pluginType));
    }
    // Create test instance
    LocalTest tests = new LocalTest();
    // Run tests
    tests.RunTestCase();
    Console.WriteLine("Tests finished");
    Console.ReadKey();
    ```

1. Guarde los cambios y luego presione **F5** o seleccione **Depurar** > **Iniciar depuración** para iniciar el proyecto. Debe aparecer una ventana de consola y el estado del registro a medida que progresen las pruebas. Cuando aparezca `Tests finished`, seleccione cualquier tecla para cerrar la ventana.

1. Use el **Explorador de Windows** para buscar el directorio que contiene el proyecto. (Por ejemplo: *C:\\Users\\\<your_user_name>\\source\\repos\\WordCount\\WordCount*). A continuación, en este directorio, abra *Bin* y seleccione *Depurar*. Debería ver los archivos de texto que se generaron cuando se ejecutaron las pruebas: *sentences.txt*, *counter.txt* y *splitter.txt*. Abra cada archivo de texto e inspeccione los datos.

   > [!NOTE]  
   > Los datos de cadena se guardan como persistentes como una matriz de valores decimales en estos archivos. Por ejemplo, `[[97,103,111]]` en el archivo **splitter.txt** representa la palabra *ago*.

> [!NOTE]  
> Asegúrese de volver a establecer la opción **Tipo de proyecto** en **Biblioteca de clases** en las propiedades del proyecto antes de implementarlo en un clúster de Storm en HDInsight.

### <a name="log-information"></a>Información de registro

Puede registrar fácilmente información de los componentes de la topología mediante `Context.Logger`. Por ejemplo, el siguiente comando crea una entrada de registro informativo:

`Context.Logger.Info("Component started");`

Se puede ver la información registrada desde el **registro del servicio Hadoop**, que se encuentra en el **Explorador de servidores**. Expanda la entrada del clúster de Storm en HDInsight y luego expanda **Registro del servicio Hadoop**. Por último, seleccione el archivo de registro que desea consultar.

> [!NOTE]  
> Los registros se almacenan en la cuenta de Azure Storage que usa el clúster. Para ver los registros en Visual Studio, debe iniciar sesión en la suscripción de Azure a la que pertenece la cuenta de almacenamiento.

### <a name="view-error-information"></a>Visualización de la información del error

Para ver los errores que se han producido en una topología en ejecución, siga estos pasos:

1. En el **Explorador de servidores**, haga clic con el botón derecho en el clúster de Storm en HDInsight y seleccione **Ver topologías de Storm**.

   En los componentes de **spout** y **bolt**, la columna **Último error** contiene información sobre el último error.

2. Seleccione el **id. de spout** o el **id. de bolt** del componente del que se muestra un error. En la página de detalles de muestra información adicional sobre el error, en la sección **Errores** al final de la página.

3. Para más información, seleccione un **puerto** en la sección **Executors** (Ejecutores) de la página para ver el registro de trabajo de Storm de los últimos minutos.

### <a name="errors-submitting-topologies"></a>Errores en el envío de topologías

Si se producen errores al enviar una topología a HDInsight, puede encontrar registros para los componentes de servidor que controlan el envío de la topología en su clúster de HDInsight. Para descargar estos registros, use el siguiente comando desde una línea de comandos:

```cmd
scp sshuser@clustername-ssh.azurehdinsight.net:/var/log/hdinsight-scpwebapi/hdinsight-scpwebapi.out .
```

Reemplace *sshuser* por la cuenta de usuario SSH del clúster. Reemplace *clustername* por el nombre del clúster de HDInsight. Para más información sobre cómo usar `scp` y `ssh` con HDInsight, vea [Uso de SSH con HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

Se pueden producir errores en los envíos por varios motivos:

* El JDK no está instalado o no está en la ruta de acceso.
* Las dependencias de Java necesarias no se incluyen en el envío.
* Las dependencias son incompatibles.
* Los nombres de las topologías están duplicados.

Si el archivo de registro *hdinsight-scpwebapi.out* contiene un objeto `FileNotFoundException`, la excepción podría deberse a las siguientes condiciones:

* El JDK no está en la ruta de acceso en el entorno de desarrollo. Compruebe que el JDK está instalado en el entorno de desarrollo y que `%JAVA_HOME%/bin` está en la ruta de acceso.
* Falta una dependencia de Java. Asegúrese de incluir todos los archivos .jar necesarios como parte del envío.

## <a name="next-steps"></a>Pasos siguientes

Para obtener un ejemplo de procesamiento de datos desde Event Hubs, vea [Procesamiento de eventos desde Centros de eventos de Azure con Storm en HDInsight](apache-storm-develop-csharp-event-hub-topology.md).

Para ver un ejemplo de una topología de C# que divide los datos de una secuencia en varias secuencias, consulte [Ejemplo de Storm en C#](https://github.com/Blackmist/csharp-storm-example).

Para más información sobre cómo crear topologías de C#, vea [GitHub](https://github.com/hdinsight/hdinsight-storm-examples/blob/master/SCPNet-GettingStarted.md).

Para conocer más formas de trabajar con HDInsight y obtener más ejemplos de Storm en HDInsight, vea los siguientes documentos:

**Microsoft SCP.NET**

* [Guía de programación de SCP para Apache Storm en Azure HDInsight](apache-storm-scp-programming-guide.md)

**Apache Storm en HDInsight**

* [Implementación y administración de topologías de Apache Storm en Azure HDInsight](apache-storm-deploy-monitor-topology-linux.md)
* [Topologías de Apache Storm de ejemplo en Azure HDInsight](apache-storm-example-topology.md)

**Apache Hadoop en HDInsight**

* [¿Qué son Apache Hive y HiveQL en Azure HDInsight?](../hadoop/hdinsight-use-hive.md)
* [Uso de MapReduce con Apache Hadoop en HDInsight](../hadoop/hdinsight-use-mapreduce.md)

**Apache HBase en HDInsight**

* [Uso de Apache HBase en Azure HDInsight](../hbase/apache-hbase-tutorial-get-started-linux.md)
