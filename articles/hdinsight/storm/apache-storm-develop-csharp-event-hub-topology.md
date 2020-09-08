---
title: Procesamiento de eventos desde Event Hubs con Storm - Azure HDInsight
description: Aprenda a procesar datos desde Azure Event Hubs con una topología de C# Storm creada en Visual Studio mediante las herramientas de HDInsight para Visual Studio.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 01/14/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: cb1c2d8daa74d1224ad07ef7a2fb5a74f4773338
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89000318"
---
# <a name="process-events-from-azure-event-hubs-with-apache-storm-on-hdinsight-c"></a>Procesamiento de eventos desde Azure Event Hubs con Apache Storm en HDInsight (C#)

Aprenda a trabajar con Azure Event Hubs desde [Apache Storm](https://storm.apache.org/) en HDInsight. En este documento se usa una topología de Storm en C# para leer y escribir datos desde Event Hubs

> [!NOTE]  
> Para ver una versión en Java de este proyecto, consulte [Procesamiento de eventos desde Azure Event Hubs con Apache Storm en HDInsight (Java)](https://github.com/Azure-Samples/hdinsight-java-storm-eventhub).

## <a name="scpnet"></a>SCP.NET

Los pasos descritos en este documento usan SCP.NET, un paquete NuGet que facilita la creación de topologías y componentes de C# para su uso con Storm en HDInsight.

HDInsight 3.4 y versiones superiores usan Mono para ejecutar topologías de C#. El ejemplo usado en este documento funciona con HDInsight 3.6. Si tiene previsto crear sus propias soluciones de .NET para HDInsight, consulte el documento [compatibilidad de Mono](https://www.mono-project.com/docs/about-mono/compatibility/) para detectar posibles incompatibilidades.

### <a name="cluster-versioning"></a>Control de versiones de clúster

El paquete de NuGet Microsoft.SCP.Net.SDK que se usa en el proyecto debe coincidir con la versión principal de Storm instalada en HDInsight. Las versiones de HDInsight 3.5 y 3.6 usan Storm 1.x, por lo que debe usar la versión 1.0.x.x de SCP.NET con estos clústeres.

Las topologías de C# también deben tener como destino .NET 4.5.

## <a name="how-to-work-with-event-hubs"></a>Trabajo con Event Hubs

Microsoft proporciona un conjunto de componentes de Java que se pueden usar para la comunicación con Event Hubs desde una topología de Storm. En [https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar](https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar) encontrará el archivo de Java (JAR) que contiene una versión de estos componentes compatible con HDInsight 3.6.

> [!IMPORTANT]  
> Si bien los componentes están escritos en Java, puede usarlos fácilmente desde una topología de C#.

En este ejemplo se usan los componentes siguientes:

* __EventHubSpout__: lee datos de Event Hubs.
* __EventHubBolt__: escribe datos en Event Hubs.
* __EventHubSpoutConfig__: se usa para configurar EventHubSpout.
* __EventHubBoltConfig__: se usa para configurar EventHubBolt.

### <a name="example-spout-usage"></a>Ejemplo de uso de spout

SCP.NET proporciona métodos para agregar un objeto EventHubSpout a la topología. Estos métodos hacen que sea más fácil agregar un spout que usar los métodos genéricos para agregar un componente Java. En el ejemplo siguiente se muestra cómo crear un spout mediante los métodos __SetEventHubSpout__ y **EventHubSpoutConfig** proporcionados por SCP.NET:

```csharp
 topologyBuilder.SetEventHubSpout(
    "EventHubSpout",
    new EventHubSpoutConfig(
        ConfigurationManager.AppSettings["EventHubSharedAccessKeyName"],
        ConfigurationManager.AppSettings["EventHubSharedAccessKey"],
        ConfigurationManager.AppSettings["EventHubNamespace"],
        ConfigurationManager.AppSettings["EventHubEntityPath"],
        eventHubPartitions),
    eventHubPartitions);
```

En el ejemplo anterior se crea un nuevo componente de spout denominado __EventHubSpout__ y se configura para la comunicación con un centro de eventos. La indicación de paralelismo del componente está establecida en el número de particiones del centro de eventos. Esta configuración permite a Storm crear una instancia del componente para cada partición.

### <a name="example-bolt-usage"></a>Ejemplo de uso de bolt

Use el método **JavaComponmentConstructor** para crear una instancia del bolt. En el ejemplo siguiente se muestra cómo crear y configurar una nueva instancia **EventHubBolt**:

```csharp
// Java construcvtor for the Event Hub Bolt
JavaComponentConstructor constructor = JavaComponentConstructor.CreateFromClojureExpr(
    String.Format(@"(org.apache.storm.eventhubs.bolt.EventHubBolt. (org.apache.storm.eventhubs.bolt.EventHubBoltConfig. " +
        @"""{0}"" ""{1}"" ""{2}"" ""{3}"" ""{4}"" {5}))",
        ConfigurationManager.AppSettings["EventHubPolicyName"],
        ConfigurationManager.AppSettings["EventHubPolicyKey"],
        ConfigurationManager.AppSettings["EventHubNamespace"],
        "servicebus.windows.net",
        ConfigurationManager.AppSettings["EventHubName"],
        "true"));

// Set the bolt to subscribe to data from the spout
topologyBuilder.SetJavaBolt(
    "eventhubbolt",
    constructor,
    partitionCount)
        .shuffleGrouping("Spout");
```

> [!NOTE]  
> En este ejemplo se usa una expresión Clojure pasada como una cadena en lugar de usar **JavaComponentConstructor** para crear un objeto **EventHubBoltConfig**, como se hizo en el ejemplo con el spout. Cualquiera de estos métodos funciona. Use el método con el que se sienta mejor.

## <a name="download-the-completed-project"></a>Descarga del proyecto finalizado

Puede descargar una versión completa del proyecto creado en este artículo desde [GitHub](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub). Sin embargo, deberá proporcionar ajustes de configuración siguiendo los pasos de este artículo.

### <a name="prerequisites"></a>Prerequisites

* Un clúster de Apache Storm en HDInsight. Consulte el artículo sobre la [Creación de clústeres de Apache Hadoop mediante Azure Portal](../hdinsight-hadoop-create-linux-clusters-portal.md) y seleccione **Storm** como **Tipo de clúster**.

* Un [centro de eventos de Azure](../../event-hubs/event-hubs-create.md).

* El [SDK de .NET para Azure](https://azure.microsoft.com/downloads/).

* Las [herramientas de HDInsight para Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md).

* Java JDK 1.8 o posterior en el entorno de desarrollo. Las descargas de JDK están disponibles en [Oracle](https://aka.ms/azure-jdks).

  * La variable de entorno **JAVA_HOME** debe señalar al directorio que contiene Java.
  * El directorio **%JAVA_HOME%/bin** debe estar en la ruta de acceso.

## <a name="download-the-event-hubs-components"></a>Descarga de los componentes de Event Hubs

Descarga del spout de Event Hubs y el componente bolt de [https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar](https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar).

Cree un directorio denominado `eventhubspout` y guarde el archivo en él.

## <a name="configure-event-hubs"></a>Configuración de Event Hubs

Event Hubs es el origen de datos para este ejemplo. Use la información de la sección "Creación de un espacio de nombres de Event Hubs y un centro de eventos" de [Introducción al envío de mensajes a Azure Event Hubs en .NET Standard](../../event-hubs/event-hubs-create.md).

1. Después de crear el centro de eventos, vaya a la configuración de **EventHub** de Azure Portal y seleccione **Directivas de acceso compartido**. Haga clic en **+ Agregar** para agregar las siguientes directivas:

   | Nombre | Permisos |
   | --- | --- |
   | escritor |Envío |
   | lector |Escuchar |

    ![Captura de pantalla de la ventana Directivas de acceso compartido](./media/apache-storm-develop-csharp-event-hub-topology/share-access-policies.png)

2. Seleccione las directivas de **lectura** y **escritura**. Copie y guarde el valor de clave principal de ambas directivas, ya que estos valores se usan más adelante.

## <a name="configure-the-eventhubwriter"></a>Configuración de EventHubWriter

1. Si aún no tiene instalada la versión más reciente de las herramientas de HDInsight para Visual Studio, vea [Introducción al uso de las herramientas de HDInsight para Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md).

2. Descargue la solución de [eventhub-storm-hybrid](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub).

3. Abra **EventHubExample.sln**. En el proyecto **EventHubWriter**, abra el archivo **App.config**. Use la información del centro de eventos que configuró anteriormente para rellenar el valor de las claves siguientes:

   | Clave | Value |
   | --- | --- |
   | EventHubPolicyName |escritura (si ha usado un nombre distinto para la directiva con el permiso *Envío*, úselo en su lugar). |
   | EventHubPolicyKey |Clave de la directiva de escritura. |
   | EventHubNamespace |Espacio de nombres que contiene el centro de eventos. |
   | EventHubName |Nombre del centro de eventos. |
   | EventHubPartitionCount |Número de particiones del centro de eventos. |

4. Guarde y cierre el archivo **App.config**.

## <a name="configure-the-eventhubreader"></a>Configuración de EventHubReader

1. Abra el proyecto **EventHubReader**.

2. Abra el archivo **App.config** para **EventHubReader**. Use la información del centro de eventos que configuró anteriormente para rellenar el valor de las claves siguientes:

   | Clave | Value |
   | --- | --- |
   | EventHubPolicyName |lectura (si ha usado un nombre distinto para la directiva con el permiso *escucha*, úselo en su lugar). |
   | EventHubPolicyKey |Clave de la directiva de lectura. |
   | EventHubNamespace |Espacio de nombres que contiene el centro de eventos. |
   | EventHubName |Nombre del centro de eventos. |
   | EventHubPartitionCount |Número de particiones del centro de eventos. |

3. Guarde y cierre el archivo **App.config**.

## <a name="deploy-the-topologies"></a>Implementación de las topologías

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **EventHubReader** y seleccione **Enviar a Storm en HDInsight**.

    ![Captura de pantalla del Explorador de soluciones con Enviar a Storm en HDInsight resaltado](./media/apache-storm-develop-csharp-event-hub-topology/submit-to-apache-storm.png)

2. En el cuadro de diálogo **Enviar topología**, seleccione el **clúster de Storm**. Expanda **Configuraciones adicionales**, seleccione **Rutas de acceso del archivo Java**, elija **...** y luego seleccione el directorio que contiene el archivo JAR que descargó anteriormente. Finalmente, haga clic en **Enviar**.

    ![Captura de pantalla del cuadro de diálogo Enviar topología](./media/apache-storm-develop-csharp-event-hub-topology/submit-storm-topology.png)

3. Cuando se haya enviado la topología, aparecerá el **Visor de topologías de Storm**. Para ver información sobre la topología, seleccione la topología **EventHubReader** en el panel izquierdo.

    ![Captura de pantalla del Visor de topologías de Storm](./media/apache-storm-develop-csharp-event-hub-topology/storm-topology-viewer.png)

4. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **EventHubWriter** y seleccione **Enviar a Storm en HDInsight**.

5. En el cuadro de diálogo **Enviar topología**, seleccione el **clúster de Storm**. Expanda **Configuraciones adicionales**, seleccione **Rutas de acceso del archivo Java**, elija **...** y seleccione el directorio que contiene el archivo JAR que descargó anteriormente. Finalmente, haga clic en **Enviar**.

6. Cuando se haya enviado la topología, actualice la lista de topologías en el **Visor de topologías de Storm** para comprobar que las dos se estén ejecutando en el clúster.

7. En el **Visor de topologías de Storm**, seleccione la topología **EventHubReader**.

8. Para abrir el resumen de componentes del bolt, haga doble clic en el componente **LogBolt** en el diagrama.

9. En la sección **Executors** (Ejecutores), seleccione uno de los vínculos de la columna **Port** (Puerto). Esto muestra información registrada por el componente. La información registrada es similar al texto siguiente:

    ```output
    2017-03-02 14:51:29.255 m.s.p.TaskHost [INFO] Received C# STDOUT: 2017-03-02 14:51:29,255 [1] INFO  EventHubReader_LogBolt [(null)] - Received data: {"deviceValue":1830978598,"deviceId":"8566ccbc-034d-45db-883d-d8a31f34068e"}
    2017-03-02 14:51:29.283 m.s.p.TaskHost [INFO] Received C# STDOUT: 2017-03-02 14:51:29,283 [1] INFO  EventHubReader_LogBolt [(null)] - Received data: {"deviceValue":1756413275,"deviceId":"647a5eff-823d-482f-a8b4-b95b35ae570b"}
    2017-03-02 14:51:29.313 m.s.p.TaskHost [INFO] Received C# STDOUT: 2017-03-02 14:51:29,312 [1] INFO  EventHubReader_LogBolt [(null)] - Received data: {"deviceValue":1108478910,"deviceId":"206a68fa-8264-4d61-9100-bfdb68ee8f0a"}
    ```

## <a name="stop-the-topologies"></a>Detención de las topologías

Para detener las topologías, seleccione cada una en el **Visor de topologías de Storm** y haga clic en **Eliminar**.

![Captura de pantalla del Visor de topologías de Storm con el botón Eliminar resaltado](./media/apache-storm-develop-csharp-event-hub-topology/kill-storm-topology1.png)

## <a name="delete-your-cluster"></a>Eliminación del clúster

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido a usar el spout y bolt de Event Hubs de Java en una topología de C# para trabajar con datos en Azure Event Hubs. Para más información sobre cómo crear topologías de C#, vea lo siguiente:

* [Desarrollo de topologías de C# para Apache Storm en HDInsight con Visual Studio](apache-storm-develop-csharp-visual-studio-topology.md)
* [Guía de programación de SCP](apache-storm-scp-programming-guide.md)
* [Topologías de ejemplo para Apache Storm en HDInsight](apache-storm-example-topology.md)
