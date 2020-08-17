---
title: Apache Storm con componentes de Python (Azure HDInsight)
description: Aprenda a crear una topología de Apache Storm que use componentes de Python en Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017, devx-track-python
ms.date: 12/16/2019
ms.openlocfilehash: 1f062a8f358ac311b53e657fb5714583458bd9b5
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87872589"
---
# <a name="develop-apache-storm-topologies-using-python-on-hdinsight"></a>Desarrollo de topologías Apache Storm con Python en HDInsight

Aprenda a crear una topología de [Apache Storm](https://storm.apache.org/) que use componentes de Python. Apache Storm admite varios lenguajes, e incluso le permite combinar componentes de varios lenguajes en una topología. El marco de [Flux](https://storm.apache.org/releases/current/flux.html) (introducido con Storm 0.10.0) permite crear fácilmente soluciones que usan componentes de Python.

> [!IMPORTANT]  
> La información de este documento se probó con Storm en HDInsight 3.6.

## <a name="prerequisites"></a>Prerrequisitos

* Un clúster de Apache Storm en HDInsight. Consulte el artículo sobre la [Creación de clústeres de Apache Hadoop mediante Azure Portal](../hdinsight-hadoop-create-linux-clusters-portal.md) y seleccione **Storm** como **Tipo de clúster**.

* Un entorno de desarrollo de Storm local (opcional). Un entorno de Storm local solo es necesario si desea ejecutar localmente la topología. Para más información, consulte [Setting up a development environment](https://storm.apache.org/releases/current/Setting-up-development-environment.html) (Configuración de un entorno de desarrollo).

* [Python 2.7 o versiones superiores](https://www.python.org/downloads/).

* [Kit de desarrolladores de Java (JDK), versión 8](https://aka.ms/azure-jdks).

* [Apache Maven](https://maven.apache.org/download.cgi) correctamente [instalado](https://maven.apache.org/install.html) según Apache.  Maven es un sistema de compilación de proyectos de Java.

## <a name="storm-multi-language-support"></a>Compatibilidad con varios lenguajes de Storm

Apache Storm se ha diseñado para funcionar con componentes escritos con cualquier lenguaje de programación. Los componentes deben comprender cómo trabajar con la definición de Thrift para Storm. Para Python, se proporciona un módulo como parte del proyecto de Apache Storm que le permite interactuar fácilmente con Storm. Puede encontrar este módulo en [https://github.com/apache/storm/blob/master/storm-multilang/python/src/main/resources/resources/storm.py](https://github.com/apache/storm/blob/master/storm-multilang/python/src/main/resources/resources/storm.py).

Storm es un proceso de Java que se ejecuta en Máquina virtual Java (JVM). Los componentes escritos en otros lenguajes se ejecutan como subprocesos. Storm se comunica con estos subprocesos mediante mensajes JSON enviados a través de stdin y stdout. Se puede encontrar más detalles sobre la comunicación entre los componentes en la documentación de [Multi-lang protocolo](https://storm.apache.org/releases/current/Multilang-protocol.html) (Protocolo de varios lenguajes).

## <a name="python-with-the-flux-framework"></a>Python con el marco de trabajo de Flux

El marco de trabajo de Flux le permite definir las topologías de Storm de forma independiente con respecto a los componentes. El marco de trabajo de Flux usa YAML para definir la topología de Storm. El siguiente texto es un ejemplo de cómo se hace referencia a un componente de Python en el documento de YAML:

```yaml
# Spout definitions
spouts:
  - id: "sentence-spout"
    className: "org.apache.storm.flux.wrappers.spouts.FluxShellSpout"
    constructorArgs:
      # Command line
      - ["python", "sentencespout.py"]
      # Output field(s)
      - ["sentence"]
    # parallelism hint
    parallelism: 1
```

La clase `FluxShellSpout` se usa para iniciar el script `sentencespout.py` que implementa el spout.

Flux espera que los scripts de Python estén en el directorio `/resources` dentro del archivo jar que contiene la topología. Por ello, este ejemplo almacena los scripts de Python en el directorio `/multilang/resources`. El archivo `pom.xml` incluye este archivo mediante el siguiente código XML:

```xml
<!-- include the Python components -->
<resource>
    <directory>${basedir}/multilang</directory>
    <filtering>false</filtering>
</resource>
```

Como se mencionó anteriormente, hay un `storm.py`archivo que implementa la definición de Thrift para Storm. El marco de trabajo de Flux incluye `storm.py` automáticamente cuando se compila el proyecto, por lo que no tiene que preocuparse de incluirlo.

## <a name="build-the-project"></a>Compilación del proyecto

1. Descargue el proyecto de [https://github.com/Azure-Samples/hdinsight-python-storm-wordcount](https://github.com/Azure-Samples/hdinsight-python-storm-wordcount).

1. Abra un símbolo del sistema y vaya a la raíz del proyecto: `hdinsight-python-storm-wordcount-master`. Escriba el comando siguiente:

    ```cmd
    mvn clean compile package
    ```

    Este comando crea un archivo `target/WordCount-1.0-SNAPSHOT.jar` que contiene la topología compilada.

## <a name="run-the-storm-topology-on-hdinsight"></a>Ejecutar la topología de Storm en HDInsight

1. Use el [comandos SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) para copiar el `WordCount-1.0-SNAPSHOT.jar`archivo en el clúster de Storm en HDInsight. Modifique el comando siguiente: reemplace CLUSTERNAME por el nombre del clúster y, luego, escriba el comando:

    ```cmd
    scp target/WordCount-1.0-SNAPSHOT.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:
    ```

1. Una vez cargado el archivo, conéctese al clúster mediante SSH:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. En la sesión SSH, use el siguiente comando para iniciar la topología en el clúster:

    ```bash
    storm jar WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux -r -R /topology.yaml
    ```

    Cuando se ha iniciado una topología Storm, esta se ejecuta hasta que se detiene.

1. Use la interfaz de usuario de Storm para ver la topología en el clúster. La interfaz de usuario de Storm se encuentra en `https://CLUSTERNAME.azurehdinsight.net/stormui`. Reemplace `CLUSTERNAME` por el nombre del clúster.

1. Detenga la topología de Storm. Use el siguiente comando para detener la topología en el clúster:

    ```bash
    storm kill wordcount
    ```

    Como alternativa, puede usar la interfaz de usuario de Storm. En **Acciones de topología** para la topología, seleccione **Terminar**.

## <a name="run-the-topology-locally"></a>Ejecución de la topología de manera local

Para ejecutar la topología de manera local, use el siguiente comando:

```bash
storm jar WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux -l -R /topology.yaml
```

> [!NOTE]  
> Este comando requiere un entorno de desarrollo de Storm local. Para más información, consulte [Setting up a development environment](https://storm.apache.org/releases/current/Setting-up-development-environment.html) (Configuración de un entorno de desarrollo).

Una vez que se inicia la topología, esta emite información en la consola local que se parece al siguiente texto:

```output
24302 [Thread-25-sentence-spout-executor[4 4]] INFO  o.a.s.s.ShellSpout - ShellLog pid:2436, name:sentence-spout Emiting the cow jumped over the moon
24302 [Thread-30] INFO  o.a.s.t.ShellBolt - ShellLog pid:2438, name:splitter-bolt Emitting the
24302 [Thread-28] INFO  o.a.s.t.ShellBolt - ShellLog pid:2437, name:counter-bolt Emitting years:160
24302 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=the, count=599}
24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=seven, count=302}
24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=dwarfs, count=143}
24303 [Thread-25-sentence-spout-executor[4 4]] INFO  o.a.s.s.ShellSpout - ShellLog pid:2436, name:sentence-spout Emiting the cow jumped over the moon
24303 [Thread-30] INFO  o.a.s.t.ShellBolt - ShellLog pid:2438, name:splitter-bolt Emitting cow
24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=four, count=160}
```

Para detener la topología, use __Ctrl+C__.

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes documentos para ver otras maneras de usar Python con HDInsight. [Cómo usar las funciones definidas por el usuario (UDF) de Python en Apache Pig y Apache Hive](../hadoop/python-udf-hdinsight.md).
