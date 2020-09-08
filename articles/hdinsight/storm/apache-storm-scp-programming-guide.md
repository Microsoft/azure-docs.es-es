---
title: Guía de programación de SCP.NET para Storm en Azure HDInsight
description: Aprenda a usar SCP.NET para crear topologías de Storm basadas en .NET para usarlas con Storm ejecutándose en HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 01/13/2020
ms.openlocfilehash: d54a06c457451fc5323ae37b34b53411cdd6abda
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89000148"
---
# <a name="scp-programming-guide-for-apache-storm-in-azure-hdinsight"></a>Guía de programación de SCP para Apache Storm en Azure HDInsight

SCP es una plataforma para compilar aplicaciones de procesamiento de datos confiables, coherentes y de alto rendimiento en tiempo real. Se basa en [Apache Storm](https://storm.incubator.apache.org/), un sistema de procesamiento de secuencias diseñado por comunidades de software de código abierto. Nathan Marz creó Storm. Twitter lo publicó como código abierto. Storm usa [Apache ZooKeeper](https://zookeeper.apache.org/), otro proyecto de Apache que permite una coordinación distribuida muy confiable y la administración de estados.

El proyecto SCP no solo ha trasladado Storm a Windows, sino también extensiones agregadas al proyecto y personalización del ecosistema Windows. Entre las extensiones se incluyen la experiencia para desarrolladores de .NET y las bibliotecas de .NET. La personalización incluye la implementación basada en Windows.

Con las extensiones y la personalización, no es necesario bifurcar los proyectos de software de código abierto. Puede usar entornos derivados basados en Storm.

## <a name="processing-model"></a>Modelo de procesamiento

Los datos de SCP se han modelado como secuencias continuas de tuplas. Normalmente, las tuplas:

1. Se introducen en una cola.
1. Son seleccionadas por una lógica de negocios hospedada en una topología de Storm, que las transforma.
1. Canalizan el resultado como tuplas a otro sistema SCP o se confirman en almacenes como sistemas de archivos distribuidos y bases de datos, como SQL Server.

![Diagrama de una cola que alimenta de datos a un procesamiento, lo que a su vez alimenta un almacén de datos](./media/apache-storm-scp-programming-guide/queue-feeding-data-to-processing-to-data-store.png)

En Storm, una topología de aplicación define un gráfico de cálculo. Cada nodo de una topología contiene lógica de procesamiento. Los vínculos entre los nodos indican el flujo de datos.

Los nodos que insertan datos de entrada en la topología se denominan _spouts_. Puede utilizarlos para secuenciar los datos. El origen de los datos de entrada puede ser unos registros de archivo, bases de datos transaccionales o contadores de rendimiento del sistema.

Los nodos que tienen flujos de datos de entrada y salida se denominan _bolts_. Se encargan del filtrado, las selecciones y la agregación de datos.

SCP admite procesamiento de datos de tipo best-effort, at-least-once y exactly-once.

En una aplicación de procesamiento de secuencias distribuida, pueden producirse errores durante el procesamiento de datos. Entre estos errores se incluyen una interrupción de la red, un error de la máquina o un error en el código. El procesamiento de tipo at-least-once garantiza que todos los datos se van a procesar al menos una vez, para lo cual se reproducen automáticamente los mismos datos en caso de error.

El procesamiento at-least-once es sencillo, confiable y adecuado para muchas aplicaciones. Sin embargo, cuando una aplicación requiere un recuento exacto, el procesamiento at-least-once es insuficiente ya que existe la posibilidad de que los mismos datos se reproduzcan en la topología de la aplicación. En este caso, el procesamiento exactly-once garantiza que el resultado sea correcto incluso cuando los datos se reproducen y procesan varias veces.

SCP permite a los desarrolladores de .NET crear aplicaciones de procesamiento de datos en tiempo real a la vez que usan una Máquina virtual Java (JVM) con Storm. La Máquina virtual Java y .NET se comunican mediante sockets locales de TCP. Cada spout/bolt es un par de procesos de .NET/Java, donde la lógica del usuario se ejecuta en un proceso de .NET como un complemento.

Para compilar una aplicación de procesamiento de datos encima de SCP, siga estos pasos:

1. Diseñe e implemente los spouts para extraer datos de colas.
1. Diseñe e implemente bolts para procesar los datos de entrada y guarde los datos en almacenes externos como, por ejemplo, una base de datos.
1. Diseñe la topología y, a continuación, envíela y ejecútela.

La topología define vértices y los datos que fluyen entre ellos. SCP toma una especificación de topología y la implementa en un clúster de Storm, en el que cada vértice ejecuta un nodo lógico. El programador de tareas de Storm se encarga de la conmutación por error y el escalado.

En este artículo se usan algunos ejemplos sencillos para examinar cómo se compilan las aplicaciones de procesamiento de datos con SCP.

## <a name="scp-plug-in-interface"></a>Interfaz de complemento de SCP

Los complementos de SCP son aplicaciones independientes. Pueden ejecutarse dentro de Visual Studio durante el desarrollo y estar conectados a la canalización de Storm después de la implementación de producción.

La escritura de un complemento de SCP es igual que la escritura de las demás aplicaciones de consola de Windows. La plataforma SCP.NET declara algunas interfaces para spout/bolt. El código del complemento implementa estas interfaces. El propósito principal de este diseño es permitirle centrarse en la lógica de negocios mientras la plataforma SCP.NET controla otras cosas.

El código de complemento implementa una de las interfaces siguientes. La interfaz depende de si la topología es transaccional o no transaccional y si el componente es un spout o un bolt.

* **ISCPSpout**
* **ISCPBolt**
* **ISCPTxSpout**
* **ISCPBatchBolt**

### <a name="iscpplugin"></a>ISCPPlugin

**ISCPPlugin** es la interfaz común para muchos complementos. Actualmente es una interfaz ficticia.

```csharp
public interface ISCPPlugin
{
}
```

### <a name="iscpspout"></a>ISCPSpout

**ISCPSpout** es la interfaz de un spout no transaccional.

```csharp
public interface ISCPSpout : ISCPPlugin
{
    void NextTuple(Dictionary<string, Object> parms);
    void Ack(long seqId, Dictionary<string, Object> parms); 
    void Fail(long seqId, Dictionary<string, Object> parms);
}
```

Cuando se llama a **NextTuple**, el código de C# puede emitir una o más tuplas. Si no hay nada que emitir, este método se debe devolver sin emitir nada.

Se llama a los métodos **NextTuple**, **Ack** y **Fail** en un bucle cerrado en un único subproceso de un proceso de C#. Cuando no haya ninguna tupla que emitir, mantenga **NextTuple** inactivo durante una breve cantidad de tiempo (como 10 milisegundos). Esto ayudará a evitar que se desperdicie la disponibilidad de la CPU.

Los métodos**Ack** y **Fail** solo se llaman cuando un archivo de especificación habilita el mecanismo de confirmación. El parámetro *seqId* identifica la tupla que se ha confirmado o en la que se ha producido un error. Si la confirmación se habilita en una topología no transaccional, se debe usar la siguiente función **Emit** en un spout:

```csharp
public abstract void Emit(string streamId, List<object> values, long seqId);
```

Si una topología no transaccional no admite la confirmación, **Ack** y **Fail** pueden dejarse como funciones vacías.

El parámetro de entrada *parms* de estas funciones especifica un diccionario vacío y se reserva para su uso en el futuro.

### <a name="iscpbolt"></a>ISCPBolt

**ISCPBolt** es la interfaz de un bolt no transaccional.

```csharp
public interface ISCPBolt : ISCPPlugin
{
void Execute(SCPTuple tuple);
}
```

Cuando hay una nueva tupla disponible, se llama a la función **Execute** para procesarla.

### <a name="iscptxspout"></a>ISCPTxSpout

**ISCPTxSpout** es la interfaz de un spout transaccional.

```csharp
public interface ISCPTxSpout : ISCPPlugin
{
    void NextTx(out long seqId, Dictionary<string, Object> parms);  
    void Ack(long seqId, Dictionary<string, Object> parms);         
    void Fail(long seqId, Dictionary<string, Object> parms);        
}
```

Del mismo modo que su homólogo no transaccional, **NextTx**, **Ack**, y **Fail** se llaman en un bucle cerrado en un solo subproceso de un proceso de C#. Cuando no haya ninguna tupla que emitir, mantenga **NextTx** inactivo durante una breve cantidad de tiempo (como 10 milisegundos). Esto ayudará a evitar que se desperdicie la disponibilidad de la CPU.

Cuando se llama a **NextTx** para iniciar una nueva transacción, el parámetro de salida *seqId* identifica la transacción. La transacción también se usa en **Ack** y **Fail**. El método **NextTx** puede emitir datos a la parte Java. Los datos se almacenan en ZooKeeper para admitir la reproducción. Dado que ZooKeeper tiene una capacidad limitada, el código solo debe emitir metadatos y no datos masivos en un spout transaccional.

Dado que Storm reproduce automáticamente una transacción con errores, normalmente no se llamará a **Fail**. Sin embargo, si SCP puede comprobar los metadatos emitidos por un spout transaccional, puede llamar a **Fail** cuando los metadatos no sean válidos.

El parámetro de entrada *parms* de estas funciones especifica un diccionario vacío y se reserva para su uso en el futuro.

### <a name="iscpbatchbolt"></a>ISCPBatchBolt

**ISCPBatchBolt** es la interfaz de un bolt transaccional.

```csharp
public interface ISCPBatchBolt : ISCPPlugin
{
    void Execute(SCPTuple tuple);
    void FinishBatch(Dictionary<string, Object> parms);  
}
```

Se llama al método **Execute** cuando llega una nueva tupla al bolt. Se llama al método **FinishBatch** cuando finaliza esta transacción. El parámetro de entrada *parms* se reserva para su uso en el futuro.

En el caso de una topología transaccional, **StormTxAttempt** es una clase importante. Tiene dos miembros: **TxId** y **AttemptId**. El miembro **TxId** identifica una transacción concreta. Es posible que una transacción se intente varias veces si se produce un error y se reproduce.

SCP.NET crea un nuevo objeto **ISCPBatchBolt** para procesar cada objeto **StormTxAttempt**, del mismo modo que hace Storm en Java. La finalidad de este diseño es admitir el procesamiento en paralelo de transacciones. Cuando un intento de transacción se completa, el objeto **ISCPBatchBolt** correspondiente se destruye y se recopila como no utilizado.

## <a name="object-model"></a>Modelo de objetos

SCP.NET también proporciona un conjunto sencillo de objetos clave para que los desarrolladores programen con ellos. Los objetos son **Context**, **StateStore** y **SCPRuntime**. Se describen en esta sección.

### <a name="context"></a>Context

El objeto **Context** proporciona un entorno de ejecución para una aplicación. Cada instancia de **ISCPPlugin** de **ISCPSpout**, **ISCPBolt**, **ISCPTxSpout** o **ISCPBatchBolt** tiene su correspondiente instancia de **Context**. La funcionalidad suministrada por **Context** se divide en dos partes:

* La parte estática, que está disponible en todo el proceso de C#.
* La parte dinámica, que solo está disponible para la instancia de **Context** determinada.

### <a name="static-part"></a>Parte estática

```csharp
public static ILogger Logger = null;
public static SCPPluginType pluginType;
public static Config Config { get; set; }
public static TopologyContext TopologyContext { get; set; }  
```

El objeto **Logger** se proporciona con fines de registro.

El objeto **pluginType** indica el tipo de complemento del proceso de C#. Si el proceso se ejecuta en modo de prueba local sin Java, el tipo de complemento es **SCP_NET_LOCAL**.

```csharp
public enum SCPPluginType 
{
    SCP_NET_LOCAL = 0,
    SCP_NET_SPOUT = 1,
    SCP_NET_BOLT = 2,
    SCP_NET_TX_SPOUT = 3,
    SCP_NET_BATCH_BOLT = 4  
    }
```

La propiedad **Config** obtiene parámetros de configuración de la parte Java, que los pasa cuando se inicializa un complemento de C#. Los parámetros de **Config** se dividen en dos partes: **stormConf** y **pluginConf**.

```csharp
public Dictionary<string, Object> stormConf { get; set; }  
public Dictionary<string, Object> pluginConf { get; set; }  
```

La parte **stormConf** son parámetros definidos por Storm y la parte **pluginConf** son parámetros definido por SCP. Este es un ejemplo:

```csharp
public class Constants
{
    … …

    // constant string for pluginConf
    public static readonly String NONTRANSACTIONAL_ENABLE_ACK = "nontransactional.ack.enabled";  

    // constant string for stormConf
    public static readonly String STORM_ZOOKEEPER_SERVERS = "storm.zookeeper.servers";
    public static readonly String STORM_ZOOKEEPER_PORT = "storm.zookeeper.port";
}
```

El tipo **TopologyContext** obtiene el contexto de la topología. Resulta especialmente útil para varios componentes paralelos. Este es un ejemplo:

```csharp
//demo how to get TopologyContext info
if (Context.pluginType != SCPPluginType.SCP_NET_LOCAL)
{
    Context.Logger.Info("TopologyContext info:");
    TopologyContext topologyContext = Context.TopologyContext;
    Context.Logger.Info("taskId: {0}", topologyContext.GetThisTaskId());
    taskIndex = topologyContext.GetThisTaskIndex();
    Context.Logger.Info("taskIndex: {0}", taskIndex);
    string componentId = topologyContext.GetThisComponentId();
    Context.Logger.Info("componentId: {0}", componentId);
    List<int> componentTasks = topologyContext.GetComponentTasks(componentId);  
    Context.Logger.Info("taskNum: {0}", componentTasks.Count);
}
```

### <a name="dynamic-part"></a>Parte dinámica

Las siguientes interfaces son pertinentes para una determinada instancia de **Context**, que se crea mediante la plataforma SCP.NET y se pasa al código:

```csharp
// Declare the Output and Input Stream Schemas

public void DeclareComponentSchema(ComponentStreamSchema schema);

// Emit tuple to default stream.
public abstract void Emit(List<object> values);

// Emit tuple to the specific stream.
public abstract void Emit(string streamId, List<object> values);  
```

Para un spout no transaccional que admite confirmación, se proporciona el método siguiente:

```csharp
// for nontransactional spout that supports ack
public abstract void Emit(string streamId, List<object> values, long seqId);  
```

Un bolt no transaccional que admite confirmación debe llamar explícitamente a **Ack** o **Fail** con la tupla que ha recibido. Al emitir una nueva tupla, el bolt también debe especificar los delimitadores de la tupla. Se proporcionan los siguientes métodos:

```csharp
public abstract void Emit(string streamId, IEnumerable<SCPTuple> anchors, List<object> values);
public abstract void Ack(SCPTuple tuple);
public abstract void Fail(SCPTuple tuple);
```

### <a name="statestore"></a>StateStore

El objeto **StateStore** proporciona servicios de metadatos, generación de secuencias monotónicas y coordinación sin espera. Puede compilar abstracciones de simultaneidad distribuidas de nivel más alto en **StateStore**. Estas abstracciones incluyen bloqueos distribuidos, colas distribuidas, barreras y servicios de transacción.

Las aplicaciones de SCP pueden usar el objeto **State** para serializar información en [Apache ZooKeeper](https://zookeeper.apache.org/). Esta capacidad es especialmente valiosa para una topología transaccional. Si un spout transaccional deja de responder y se reinicia, **State** puede recuperar la información necesaria de ZooKeeper y reiniciar la canalización.

El objeto **StateStore** tiene estos métodos principales:

```csharp
/// <summary>
/// Static method to retrieve a state store of the given path and connStr 
/// </summary>
/// <param name="storePath">StateStore path</param>
/// <param name="connStr">StateStore address</param>
/// <returns>Instance of StateStore</returns>
public static StateStore Get(string storePath, string connStr);

/// <summary>
/// Create a new state object in this state store instance
/// </summary>
/// <returns>State from StateStore</returns>
public State Create();

/// <summary>
/// Retrieve all states that were previously uncommitted, excluding all exited states
/// </summary>
/// <returns>Uncommitted states</returns>
public IEnumerable<State> GetUnCommitted();

/// <summary>
/// Get all the states in the StateStore
/// </summary>
/// <returns>All the states</returns>
public IEnumerable<State> States();

/// <summary>
/// Get state or registry object
/// </summary>
/// <param name="info">Registry name (registry only)</param>
/// <typeparam name="T">Type, registry or state</typeparam>
/// <returns>Return registry or state</returns>
public T Get<T>(string info = null);

/// <summary>
/// List all the committed states
/// </summary>
/// <returns>Registries containing the committed state </returns>
public IEnumerable<Registry> Committed();

/// <summary>
/// List all the exited states in the StateStore
/// </summary>
/// <returns>Registries containing the exited states</returns>
public IEnumerable<Registry> Aborted();

/// <summary>
/// Retrieve an existing state object from this state store instance 
/// </summary>
/// <returns>State from StateStore</returns>
/// <typeparam name="T">stateId, id of the State</typeparam>
public State GetState(long stateId)
```

El objeto **State** tiene estos métodos principales:

```csharp
/// <summary>
/// Set the status of the state object to commit
/// </summary>
public void Commit(bool simpleMode = true);

/// <summary>
/// Set the status of the state object to exit
/// </summary>
public void Abort();

/// <summary>
/// Put an attribute value under the given key
/// </summary>
/// <param name="key">Key</param>
/// <param name="attribute">State attribute</param>
    public void PutAttribute<T>(string key, T attribute);

/// <summary>
/// Get the attribute value associated with the given key
/// </summary>
/// <param name="key">Key</param>
/// <returns>State attribute</returns>
    public T GetAttribute<T>(string key);
```

Cuando **simpleMode** se establece en **true**, el método **Commit** eliminará el ZNode correspondiente de ZooKeeper. De lo contrario, el método elimina el ZNode actual y agrega un nuevo nodo en COMMITTED\_PATH.

### <a name="scpruntime"></a>SCPRuntime

La clase **SCPRuntime** proporciona los dos métodos siguientes:

```csharp
public static void Initialize();

public static void LaunchPlugin(newSCPPlugin createDelegate);  
```

El método **Initialize** inicializa el entorno de tiempo de ejecución de SCP. En este método, el proceso de C# se conecta a la parte Java para obtener los parámetros de configuración y el contexto de la topología.

El método **LaunchPlugin** inicia el bucle de procesamiento de mensajes. En este bucle, el complemento de C# recibe mensajes de la parte Java. Estos mensajes incluyen tuplas y señales de control. A continuación, el complemento procesa los mensajes, posiblemente llamando al método de interfaz proporcionado por el código.

El parámetro de entrada de **LaunchPlugin** es un delegado. El método puede devolver un objeto que implementa la interfaz **ISCPSpout**, **ISCPBolt**, **ISCPTxSpout** o **ISCPBatchBolt**.

```csharp
public delegate ISCPPlugin newSCPPlugin(Context ctx, Dictionary<string, Object> parms);
```

En el caso de **ISCPBatchBolt**, puede obtener un objeto **StormTxAttempt** del parámetro *parms* y usarlo para determinar si el intento es un intento reproducido. La comprobación de un intento de reproducción suele realizarse en el bolt de confirmación. En el ejemplo HelloWorldTx que aparece más adelante en este artículo se muestra esta comprobación.

Los complementos de SCP normalmente se pueden ejecutar en dos modos: modo de prueba local y modo normal.

#### <a name="local-test-mode"></a>Modo de prueba local

En este modo, los complementos de SCP del código de C# se ejecutan en Visual Studio durante la fase de desarrollo. Puede usar la interfaz **ILocalContext** en este modo. La interfaz proporciona métodos para serializar las tuplas emitidas a archivos locales y leerlas de nuevo en la memoria RAM.

```csharp
public interface ILocalContext
{
    List<SCPTuple> RecvFromMsgQueue();
    void WriteMsgQueueToFile(string filepath, bool append = false);  
    void ReadFromFileToMsgQueue(string filepath);
}
```

#### <a name="regular-mode"></a>Modo normal

En este modo, el proceso Java de Storm ejecuta los complementos de SCP. Este es un ejemplo:

```csharp
namespace Scp.App.HelloWorld
{
public class Generator : ISCPSpout
{
    … …
    public static Generator Get(Context ctx, Dictionary<string, Object> parms)
    {
    return new Generator(ctx);
    }
}

class HelloWorld
{
    static void Main(string[] args)
    {
    /* Setting the environment variable here can change the log file name */
    System.Environment.SetEnvironmentVariable("microsoft.scp.logPrefix", "HelloWorld");

    SCPRuntime.Initialize();
    SCPRuntime.LaunchPlugin(new newSCPPlugin(Generator.Get));
    }
}
}
```

## <a name="topology-specification-language"></a>Lenguaje de especificación de topologías

La especificación de topologías de SCP es un lenguaje específico de dominios (DSL) para la descripción y configuración de topologías de SCP. Se basa en [Clojure DSL de Storm](https://storm.incubator.apache.org/documentation/Clojure-DSL.html) y se extiende mediante SCP.

Las especificaciones de topologías se pueden enviar directamente a un clúster de Storm para su ejecución mediante el comando **runSpec**.

SCP.NET ha agregado las siguientes funciones para definir topologías transaccionales:

| Nueva función | Parámetros | Descripción |
| --- | --- | --- |
| **tx-topolopy** |*topology-name*<br />*spout-map*<br />*bolt-map* |Permite definir una topología transaccional con el nombre de la topología, el mapa de definición de spouts y el mapa de definición de bolts. |
| **scp-tx-spout** |*exec-name*<br />*args*<br />*fields* |Permite definir un spout transaccional. La función ejecuta la aplicación especificada por *exec-name* y usa *args*.<br /><br />El parámetro *fields* especifica los campos de salida del spout. |
| **scp-tx-batch-bolt** |*exec-name*<br />*args*<br />*fields* |Permite definir un bolt de lote transaccional. La función ejecuta la aplicación especificada por *exec-name* y usa *args*.<br /><br />El parámetro *fields* especifica los campos de salida del bolt. |
| **scp-tx-commit-bolt** |*exec-name*<br />*args*<br />*fields* |Permite definir un bolt de confirmación transaccional. La función ejecuta la aplicación especificada por *exec-name* y usa *args*.<br /><br />El parámetro *fields* especifica los campos de salida del bolt. |
| **nontx-topology** |*topology-name*<br />*spout-map*<br />*bolt-map* |Permite definir una topología no transaccional con el nombre de la topología, el mapa de definición de spouts y el mapa de definición de bolts. |
| **scp-spout** |*exec-name*<br />*args*<br />*fields*<br />*parameters* |Permite definir un spout no transaccional. La función ejecuta la aplicación especificada por *exec-name* y usa *args*.<br /><br />El parámetro *fields* especifica los campos de salida del spout.<br /><br />El parámetro *parameters* es opcional. Úselo para especificar parámetros como "nontransactional.ack.enabled". |
| **scp-bolt** |*exec-name*<br />*args*<br />*fields*<br />*parameters* |Permite definir un bolt no transaccional. La función ejecuta la aplicación especificada por *exec-name* y usa *args*.<br /><br />El parámetro *fields* especifica los campos de salida del bolt.<br /><br />El parámetro *parameters* es opcional. Úselo para especificar parámetros como "nontransactional.ack.enabled". |

SCP.NET define las siguientes palabras clave:

| Palabra clave | Descripción |
| --- | --- |
| **:name** |Nombre de la topología |
| **:topology** |Topología que usa las funciones de la tabla anterior y funciones integradas |
| **:p** |Sugerencia de paralelismo para cada spout o bolt |
| **:config** |Permite definir si se configuran los parámetros o se actualizan los existentes |
| **:schema** |Esquema de la secuencia |

SCP.NET también define estos parámetros usados con frecuencia:

| Parámetro | Descripción |
| --- | --- |
| "plugin.name" |Nombre del archivo .exe del complemento de C# |
| "plugin.args" |Argumentos del complemento |
| "output.schema" |Esquema de salida |
| "nontransactional.ack.enabled" |Indica si se ha habilitado confirmación para una topología no transaccional |

El comando **runSpec** se implementa junto con los bits. Este es el uso del comando:

```csharp
.\bin\runSpec.cmd
usage: runSpec [spec-file target-dir [resource-dir] [-cp classpath]]
ex: runSpec examples\HelloWorld\HelloWorld.spec specs examples\HelloWorld\Target
```

El parámetro *resource-dir* es opcional. Especifíquelo cuando desee conectar una aplicación de C#. El directorio especificado contiene la aplicación, las dependencias y las configuraciones.

El parámetro *classpath* también es opcional. Especifica el parámetro classpath de Java si el archivo de especificación contiene un spout o un bolt de Java.

## <a name="miscellaneous-features"></a>Características varias

### <a name="input-and-output-schema-declarations"></a>Declaración de esquema de entrada y salida

Los procesos de C# pueden emitir tuplas. Para ello, la plataforma serializa las tuplas en objetos **byte[]** y transfiere los objetos a la parte Java. Después, Storm transfiere estas tuplas a los destinos.

En los componentes de nivel inferior, los procesos de C# reciben tuplas de vuelta de la parte Java y las convierten a los tipos originales de la plataforma. La plataforma oculta todas estas operaciones.

Para que admita la serialización y la deserialización, el código debe declarar el esquema de las entradas y salidas. El esquema se define como un diccionario. El identificador de secuencia es la clave del diccionario. El valor de la clave es el parámetro types de las columnas. Un componente puede declarar varias secuencias.

```csharp
public class ComponentStreamSchema
{
    public Dictionary<string, List<Type>> InputStreamSchema { get; set; }
    public Dictionary<string, List<Type>> OutputStreamSchema { get; set; }
    public ComponentStreamSchema(Dictionary<string, List<Type>> input, Dictionary<string, List<Type>> output)
    {
        InputStreamSchema = input;
        OutputStreamSchema = output;
    }
}
```

La siguiente función se agrega a un objeto **Context**:

```csharp
public void DeclareComponentSchema(ComponentStreamSchema schema)
```

Los desarrolladores deben asegurarse de que las tuplas emitidas obedecen el esquema definido para una secuencia. De lo contrario, el sistema producirá una excepción en tiempo de ejecución.

### <a name="multistream-support"></a>Compatibilidad con varias secuencias

SCP permite que el código emita o reciba desde varias secuencias distintas a la vez. El objeto **Context** refleja esta compatibilidad como el parámetro de identificador de secuencia opcional del método **Emit**.

Se han agregado dos métodos al objeto **Context** de SCP.NET. Emiten una o más tuplas para secuencias concretas. El parámetro *streamId* es una cadena. Su valor debe ser el mismo en el código de C# y en la especificación de definición de la topología.

```csharp
/* Emit tuple to the specific stream. */
public abstract void Emit(string streamId, List<object> values);

/* for nontransactional spout only */
public abstract void Emit(string streamId, List<object> values, long seqId);
```

La emisión a una secuencia inexistente provoca excepciones en tiempo de ejecución.

### <a name="fields-grouping"></a>Agrupación de campos

La agrupación de campos integrada de Storm no funciona correctamente en SCP.NET. En la parte de proxy Java, el tipo de datos de todos los campos es realmente **byte[]** . La agrupación de campos usa el código hash del objeto **byte[]** para realizar la agrupación. El código hash es la dirección de este objeto en la memoria RAM. Por lo tanto, la agrupación será incorrecta para objetos multibyte que compartan el mismo contenido pero no la misma dirección.

SCP.NET agrega un método de agrupación personalizado y usa el contenido del objeto **byte[]** para realizar la agrupación. En un archivo de especificación, la sintaxis es similar a la de este ejemplo:

```csharp
(bolt-spec
    {
        "spout_test" (scp-field-group :non-tx [0,1])
    }
    …
)
```

En el archivo de especificación anterior:

* `scp-field-group` especifica que la agrupación es una agrupación de campos personalizada implementada por SCP.
* `:tx` o `:non-tx` especifica si la topología es transaccional. Necesita esta información porque el índice inicial es diferente entre las topologías transaccionales y las no transaccionales.
* `[0,1]` especifica un conjunto hash de identificadores de campo que empiezan por cero.

### <a name="hybrid-topology"></a>Topologías híbridas

El código de Storm nativo se escribe en Java. SCP.NET ha mejorado Storm para permitirle escribir código de C# para controlar la lógica de negocios. Sin embargo, SCP.NET también admite topologías híbridas que no solo contienen spouts y bolts de C#, sino también spouts y bolts de Java.

### <a name="specify-java-spoutbolt-in-a-specification-file"></a>Especificación de spouts y bolts de Java en un archivo de especificación

Puede usar **scp-spout** y **scp-bolt** en un archivo de especificación para especificar spouts y bolts de Java. Este es un ejemplo:

```csharp
(spout-spec 
  (microsoft.scp.example.HybridTopology.Generator.)
  :p 1)
```

Aquí, `microsoft.scp.example.HybridTopology.Generator` es el nombre de la clase spout de Java.

### <a name="specify-the-java-classpath-in-a-runspec-command"></a>Especificación de classpath de Java en un comando runSpec

Si desea enviar una topología que contiene spouts o bolts de Java, compílelos primero para generar archivos JAR. A continuación, especifique el classpath de Java que contiene los archivos JAR al enviar la topología. Este es un ejemplo:

```csharp
bin\runSpec.cmd examples\HybridTopology\HybridTopology.spec specs examples\HybridTopology\net\Target -cp examples\HybridTopology\java\target\*
```

Aquí, `examples\HybridTopology\java\target\` es la carpeta que contiene el archivo JAR de spouts y bolts de Java.

### <a name="serialization-and-deserialization-between-java-and-c"></a>Serialización y deserialización entre Java y C#

El componente de SCP incluye la parte Java y la parte C#. Para interactuar con spouts o bolts nativos de Java, se debe realizar la serialización y la deserialización entre la parte Java y la parte C#, como se muestra en el siguiente gráfico:

![Diagrama de un componente de Java que envía a un componente de SCP que, a su vez, envía a otro componente de Java](./media/apache-storm-scp-programming-guide/java-compent-sending-to-scp-component-sending-to-java-component.png)

#### <a name="serialization-in-the-java-side-and-deserialization-in-the-c-side"></a>Serialización en la parte Java y deserialización en la parte C#

En primer lugar, proporcione la implementación predeterminada de serialización en la parte Java y de deserialización en la parte C#.

Especifique el método de serialización de la parte Java en un archivo de especificación.

```csharp
(scp-bolt
    {
        "plugin.name" "HybridTopology.exe"
        "plugin.args" ["displayer"]
        "output.schema" {}
        "customized.java.serializer" ["microsoft.scp.storm.multilang.CustomizedInteropJSONSerializer"]
    })
```

Especifique el método de deserialización de la parte C# en el código de C#.

```csharp
Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
inputSchema.Add("default", new List<Type>() { typeof(Person) });
this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, null));
this.ctx.DeclareCustomizedDeserializer(new CustomizedInteropJSONDeserializer());
```  

Si el tipo de datos no es demasiado complejo, esta implementación predeterminada debe servir para la mayoría de los casos. Los siguientes son los casos en los que puede conectar su propia implementación:

* El tipo de datos es demasiado complejo para la implementación predeterminada.
* El rendimiento de la implementación predeterminada no cumple sus requisitos.

La interfaz de serialización en la parte Java se define de la manera siguiente:

```csharp
public interface ICustomizedInteropJavaSerializer {
    public void prepare(String[] args);
    public List<ByteBuffer> serialize(List<Object> objectList);
}
```

La interfaz de deserialización en la parte C# se define de la manera siguiente:

```csharp
public interface ICustomizedInteropCSharpDeserializer
{
    List<Object> Deserialize(List<byte[]> dataList, List<Type> targetTypes);
}
```

#### <a name="serialization-in-the-c-side-and-deserialization-in-the-java-side"></a>Serialización en la parte C# y deserialización en la parte Java

Especifique el método de serialización de la parte C# en el código de C#.

```csharp
this.ctx.DeclareCustomizedSerializer(new CustomizedInteropJSONSerializer()); 
```

Especifique el método de deserialización de la parte Java en un archivo de especificación.

```csharp
(scp-spout
   {
     "plugin.name" "HybridTopology.exe"
     "plugin.args" ["generator"]
     "output.schema" {"default" ["person"]}
     "customized.java.deserializer" ["microsoft.scp.storm.multilang.CustomizedInteropJSONDeserializer" "microsoft.scp.example.HybridTopology.Person"]
   }
)
```

Aquí, `"microsoft.scp.storm.multilang.CustomizedInteropJSONDeserializer"` es el nombre del deserializador y `"microsoft.scp.example.HybridTopology.Person"` es la clase de destino en la que se deserializan los datos.

También puede conectar su propia implementación de serializador de C# y deserializador de Java.

Este código es la interfaz del serializador de C#:

```csharp
public interface ICustomizedInteropCSharpSerializer
{
    List<byte[]> Serialize(List<object> dataList);
}
```

Este código es la interfaz del deserializador de Java:

```csharp
public interface ICustomizedInteropJavaDeserializer {
    public void prepare(String[] targetClassNames);
    public List<Object> Deserialize(List<ByteBuffer> dataList);
}
```

## <a name="scp-host-mode"></a>Modo host de SCP

En este modo, puede compilar su código como DLL y usar el SCPHost.exe proporcionado por SCP para enviar una topología. Un archivo de especificación es similar a este código:

```csharp
(scp-spout
  {
    "plugin.name" "SCPHost.exe"
    "plugin.args" ["HelloWorld.dll" "Scp.App.HelloWorld.Generator" "Get"]
    "output.schema" {"default" ["sentence"]}
  })
```

En este caso, `"plugin.name"` se especifica como `"SCPHost.exe"`, proporcionado por el SDK de SCP. SCPHost.exe acepta tres parámetros en el siguiente orden:

1. El nombre del archivo DLL que, en este ejemplo, es `"HelloWorld.dll"`.
1. El nombre de la clase que, en este ejemplo, es `"Scp.App.HelloWorld.Generator"`.
1. El nombre de un método estático público, que se puede invocar para obtener una instancia de **ISCPPlugin**.

En el modo host, compile el código como un archivo DLL para que lo invoque la plataforma SCP. Dado que la plataforma puede obtener el control completo de toda la lógica de procesamiento, se recomienda que envíe la topología en el modo host de SCP. De este modo, se simplifica la experiencia de desarrollo. También ofrece más flexibilidad y una mejor compatibilidad con versiones anteriores en las versiones posteriores.

## <a name="scp-programming-examples"></a>Ejemplos de programación de SCP

### <a name="helloworld"></a>HelloWorld

En el siguiente ejemplo sencillo de HelloWorld, se muestra una idea de SCP.NET. Se usa una topología no transaccional, con un spout llamado **generator** y dos bolts llamados **splitter** y **counter**. El spout **generator** genera aleatoriamente algunas oraciones y emite estas oraciones a **splitter**. El bolt **splitter** divide las oraciones en palabras y emite estas palabras al bolt **counter**. El bolt **counter** usa un diccionario para registrar las repeticiones de cada palabra.

Este ejemplo tiene dos archivos de especificación: HelloWorld.spec y HelloWorld\_EnableAck.spec. En el código de C#, se puede descubrir si se ha habilitado confirmación obteniendo el objeto `pluginConf` de la parte Java.

```csharp
/* demo how to get pluginConf info */
if (Context.Config.pluginConf.ContainsKey(Constants.NONTRANSACTIONAL_ENABLE_ACK))
{
    enableAck = (bool)(Context.Config.pluginConf[Constants.NONTRANSACTIONAL_ENABLE_ACK]);
}
Context.Logger.Info("enableAck: {0}", enableAck);
```

Si se ha habilitado la confirmación en el spout, un diccionario almacena en caché las tuplas que no se han confirmado. Si llama a `Fail`, la tupla con errores se reproduce.

```csharp
public void Fail(long seqId, Dictionary<string, Object> parms)
{
    Context.Logger.Info("Fail, seqId: {0}", seqId);
    if (cachedTuples.ContainsKey(seqId))
    {
        /* get the cached tuple */
        string sentence = cachedTuples[seqId];

        /* replay the failed tuple */
        Context.Logger.Info("Re-Emit: {0}, seqId: {1}", sentence, seqId);
        this.ctx.Emit(Constants.DEFAULT_STREAM_ID, new Values(sentence), seqId);
    }
    else
    {
        Context.Logger.Warn("Fail(), can't find cached tuple for seqId {0}!", seqId);
    }
}
```

### <a name="helloworldtx"></a>HelloWorldTx

El ejemplo HelloWorldTx siguiente muestra cómo implementar la topología transaccional. El ejemplo incluye un spout denominado **generator**, un bolt de lote denominado **partial-count** y un bolt de confirmación denominado **count-sum**. El ejemplo también tiene tres archivos de texto existentes: DataSource0.txt, DataSource1.txt y DataSource2.txt.

En cada transacción, el spout **generator** elige aleatoriamente dos de los tres archivos existentes y emite los nombres de estos dos archivos al bolt **partial-count**. El bolt **partial-count**:

1. Obtiene un nombre de archivo de la tupla recibida.
1. Abre el archivo correspondiente.
1. Cuenta el número de palabras del archivo.
1. Emite el recuento de palabras al bolt **count-sum**.

El bolt **count-sum** resume el recuento total.

Para conseguir la semántica exactly once, es necesario que el bolt de confirmación **count-sum** decida si se trata de una transacción reproducida. En este ejemplo, tiene la siguiente variable de miembro estática:

```csharp
public static long lastCommittedTxId = -1; 
```

Una vez creada una instancia de **ISCPBatchBolt**, obtiene el valor del objeto `txAttempt` de los parámetros de entrada.

```csharp
public static CountSum Get(Context ctx, Dictionary<string, Object> parms)
{
    /* for transactional topology, we can get txAttempt from the input parms */
    if (parms.ContainsKey(Constants.STORM_TX_ATTEMPT))
    {
        StormTxAttempt txAttempt = (StormTxAttempt)parms[Constants.STORM_TX_ATTEMPT];
        return new CountSum(ctx, txAttempt);
    }
    else
    {
        throw new Exception("null txAttempt");
    }
}
```

Cuando se llama a `FinishBatch`, `lastCommittedTxId` se actualiza si no se trata de una transacción reproducida.

```csharp
public void FinishBatch(Dictionary<string, Object> parms)
{
    /* judge whether it is a replayed transaction */
    bool replay = (this.txAttempt.TxId <= lastCommittedTxId);

    if (!replay)
    {
        /* If it is not replayed, update the totalCount and lastCommittedTxId value */
        totalCount = totalCount + this.count;
        lastCommittedTxId = this.txAttempt.TxId;
    }
    … …
}
```

### <a name="hybridtopology"></a>HybridTopology

Esta topología contiene un spout de Java y un bolt de C#. Usa la implementación predeterminada de serialización y deserialización que proporciona la plataforma SCP. Consulte el archivo HybridTopology.spec en la carpeta examples\\HybridTopology para ver los detalles del archivo de especificación. Consulte también SubmitTopology.bat para saber cómo especificar el classpath de Java.

### <a name="scphostdemo"></a>SCPHostDemo

Este ejemplo es, básicamente, igual a HelloWorld. La única diferencia es que el código se compila como DLL y la topología se envía mediante SCPHost.exe. Consulte la sección "Modo host de SCP" para ver una explicación más detallada.

## <a name="next-steps"></a>Pasos siguientes

Para ver ejemplos de topologías de Apache Storm creadas con SCP, consulte los siguientes artículos:

* [Desarrollo de topologías de C# para Apache Storm en HDInsight con Visual Studio](apache-storm-develop-csharp-visual-studio-topology.md)
* [Procesamiento de eventos desde Azure Event Hubs con Apache Storm en HDInsight](apache-storm-develop-csharp-event-hub-topology.md)
* [Procesamiento de datos de sensor de vehículos desde Event Hubs mediante Apache Storm en HDInsight](https://github.com/hdinsight/hdinsight-storm-examples/tree/master/IotExample)
* [Extracción, transformación y carga (ETL) desde Azure Event Hubs hasta HBase](https://github.com/hdinsight/hdinsight-storm-examples/blob/master/RealTimeETLExample)
