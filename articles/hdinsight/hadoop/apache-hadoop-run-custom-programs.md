---
title: Ejecución de programas MapReduce personalizados en Azure HDInsight
description: Cuándo y cómo ejecutar programas de Apache MapReduce personalizados en clústeres de Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/01/2020
ms.openlocfilehash: 1dcc2a944fc527e4cbc8c7c1072503377ecb5798
ms.sourcegitcommit: 59ea8436d7f23bee75e04a84ee6ec24702fb2e61
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/07/2020
ms.locfileid: "89505274"
---
# <a name="run-custom-mapreduce-programs"></a>Ejecutar programas MapReduce personalizados

Los sistemas de macrodatos basados en Apache Hadoop, como HDInsight, permiten el procesamiento de datos con una amplia gama de herramientas y tecnologías. En la siguiente tabla se describen las principales ventajas y consideraciones de cada uno.

| Mecanismo de consulta | Ventajas | Consideraciones |
| --- | --- | --- |
| **Apache Hive con HiveQL** | <ul><li>Una excelente solución para el procesamiento por lotes y el análisis de grandes cantidades de datos inmutables, para el resumen de datos y para las consultas a petición. Usa una sintaxis familiar similar a SQL.</li><li>Se puede usar para generar tablas persistentes de datos que se pueden particionar e indexar fácilmente.</li><li>Pueden crearse varias vistas y tablas externas con los mismos datos.</li><li>Admite una implementación de almacenamiento de datos simple que proporciona muchas funciones de escalabilidad horizontal y tolerancia a errores para el procesamiento y almacenamiento de datos.</li></ul> | <ul><li>Se necesita que los datos de origen tengan al menos alguna estructura identificable.</li><li>No es adecuado para consultas en tiempo real y actualizaciones de nivel de fila. Se usa preferiblemente para trabajos por lotes en grandes conjuntos de datos.</li><li>Es posible que no pueda realizar algunos tipos de tareas de procesamiento complejas.</li></ul> |
| **Apache Pig con Pig Latin** | <ul><li>Una solución excelente para manipular datos como conjuntos, combinar y filtrar conjuntos de datos, aplicar funciones a registros o grupos de registros y reestructurar datos mediante la definición de columnas, la agrupación de valores o la conversión de columnas a filas.</li><li>Puede usar un enfoque basado en flujos de trabajo como una secuencia de operaciones en datos.</li></ul> | <ul><li>Es posible que Pig Latin resulte menos familiar y más difícil de usar que HiveQL para los usuarios de SQL.</li><li>La salida predeterminada suele ser un archivo de texto y puede ser más difícil de usar con herramientas de visualización como Excel. Normalmente, se sobrepone una tabla de Hive en la salida.</li></ul> |
| **Asignación/reducción personalizada** | <ul><li>Proporciona un control total sobre las fases y la ejecución.</li><li>Permite optimizar consultas para lograr un rendimiento máximo del clúster o para reducir la carga en los servidores y la red.</li><li>Se pueden escribir los componentes en varios lenguajes conocidos.</li></ul> | <ul><li>Es más difícil de usar que Pig o Hive porque debe crear su propia asignación y reducir los componentes.</li><li>Los procesos que requieren combinar conjuntos de datos son más difíciles de implementar.</li><li>Aunque hay marcos de pruebas disponibles, la depuración de código es más compleja que una aplicación normal porque el código se ejecuta como un trabajo por lotes bajo control del programador de trabajos de Hadoop.</li></ul> |
| **Apache HCatalog** | <ul><li>Resume los detalles de ruta de acceso del almacenamiento, lo que facilita la administración y elimina la necesidad de que los usuarios sepan dónde se almacenan los datos.</li><li>Habilita la notificación de eventos, como la disponibilidad de datos, lo que permite que otras herramientas, como Oozie, detecten cuándo se realizaron las operaciones.</li><li>Expone una vista relacional de los datos, que incluye la creación de particiones por clave, y facilita el acceso a los datos.</li></ul> | <ul><li>Admite los formatos de archivo RCFile, texto CSV, texto JSON, SequenceFile y ORC de manera predeterminada, pero es posible que tenga que escribir un SerDe personalizado para otros formatos.</li><li>HCatalog no es seguro para subprocesos.</li><li>Hay algunas restricciones en los tipos de datos de las columnas cuando se usa el cargador de HCatalog en scripts de Pig. Para obtener más información, consulte [HCatLoader Data Types](https://cwiki.apache.org/confluence/display/Hive/HCatalog%20LoadStore#HCatalogLoadStore-HCatLoaderDataTypes) (Tipos de datos de HCatLoader) en la documentación de Apache HCatalog.</li></ul> |

Normalmente, se usa el enfoque más simple que pueda proporcionar los resultados que necesita. Por ejemplo, es posible que pueda lograr estos resultados solo con Hive, pero puede que tenga que usar Pig para escenarios más complejos o, incluso, escribir sus propios componentes de asignación y reducción. También puede decidir, después de experimentar con Hive o Pig, que los componentes personalizados de asignación y reducción pueden proporcionar un mejor rendimiento, ya que permiten ajustar y optimizar el procesamiento.

## <a name="custom-mapreduce-components"></a>Componentes personalizados de asignación/reducción

El código de asignación/reducción está compuesto de dos funciones independientes que se implementan como componentes de **asignación** y **reducción**. El componente de **asignación** se ejecuta en paralelo en varios nodos de clúster y cada nodo aplica la asignación al propio subconjunto de datos del nodo. El componente de **reducción** intercala y resume los resultados de todas las funciones de asignación. Para obtener más información sobre estos dos componentes, consulte [Uso de MapReduce en Hadoop en HDInsight](hdinsight-use-mapreduce.md).

En la mayoría de los escenarios de procesamiento de HDInsight, es más sencillo y eficaz usar una abstracción de nivel superior, como Pig o Hive. También puede crear componentes de asignación y reducción personalizados para usarlos en scripts de Hive para realizar procesamientos más sofisticados.

Los componentes personalizados de asignación/reducción se suelen escribir en Java. Hadoop proporciona una interfaz de transmisión por secuencias que también permite usar componentes que se desarrollan en otros lenguajes, como C#, F #, Visual Basic, Python y JavaScript.

* Para ver un tutorial sobre el desarrollo de programas personalizados MapReduce de Java, consulte [Desarrollo de programas MapReduce de Java para Hadoop en HDInsight](apache-hadoop-develop-deploy-java-mapreduce-linux.md).

Considere la posibilidad de crear sus propios componentes de asignación y reducción para las condiciones siguientes:

* Tiene que procesar datos que no están estructurados de ningún modo mediante el análisis de los datos y el uso de lógica personalizada para obtener información estructurada.
* Quiere realizar tareas complejas que son difíciles (o imposibles) de expresar en Pig o Hive sin tener que recurrir a la creación de una UDF. Por ejemplo, es posible que tenga que usar un servicio de geocodificación externo para convertir coordenadas de latitud y longitud o direcciones IP de los datos de origen a nombres de ubicación geográfica.
* Quiere volver a usar código existente de .NET, Python o JavaScript en componentes de asignación/reducción mediante el uso de la interfaz de transmisión por secuencias de Hadoop.

## <a name="upload-and-run-your-custom-mapreduce-program"></a>Cargar y ejecutar un programa MapReduce personalizado

Los programas MapReduce más comunes se escriben en Java y se compilan en un archivo jar.

1. Después de haber desarrollado, compilado y probado el programa MapReduce, use el comando `scp` para cargar el archivo jar en el nodo principal.

    ```cmd
    scp mycustomprogram.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

    Reemplace CLUSTERNAME por el nombre del clúster. Si usa una contraseña para proteger la cuenta SSH, se le solicitará que la escriba. Si utilizó un certificado, tal vez tenga que usar el parámetro `-i` para especificar el archivo de claves privadas.

1. Use el [comando SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) para conectarse al clúster. Modifique el comando siguiente: reemplace CLUSTERNAME por el nombre del clúster y, luego, escriba el comando:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. En la sesión de SSH, ejecute el programa MapReduce mediante YARN.

    ```bash
    yarn jar mycustomprogram.jar mynamespace.myclass /example/data/sample.log /example/data/logoutput
    ```

    Este comando envía el trabajo MapReduce a YARN. El archivo de entrada es `/example/data/sample.log`, y el directorio de salida, `/example/data/logoutput`. El archivo de entrada y los archivos de salida se almacenan en el almacenamiento predeterminado del clúster.

## <a name="next-steps"></a>Pasos siguientes

* [Uso de C# con el streaming de MapReduce en Apache Hadoop en HDInsight](apache-hadoop-dotnet-csharp-mapreduce-streaming.md)
* [Desarrollo de programas MapReduce de Java para Apache Hadoop en HDInsight](apache-hadoop-develop-deploy-java-mapreduce-linux.md)
* [Uso de Azure Toolkit for Eclipse para crear aplicaciones de Apache Spark para un clúster de HDInsight](../spark/apache-spark-eclipse-tool-plugin.md)
* [Uso de funciones definidas por el usuario (UDF) de Python con Apache Hive y Apache Pig en HDInsight](python-udf-hdinsight.md)
