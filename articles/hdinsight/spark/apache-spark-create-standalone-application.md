---
title: 'Tutorial: Aplicación de Scala Maven para Spark e IntelliJ en Azure HDInsight'
description: 'Tutorial: Creación de una aplicación de Spark escrita en Scala con Apache Maven como sistema de compilación y con un arquetipo existente de Maven para Scala proporcionado por IntelliJ IDEA.'
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.custom: contperfq1
ms.date: 08/21/2020
ms.openlocfilehash: cfdb02b0a5942536a608fbc957ed69d1efd97e8e
ms.sourcegitcommit: afa1411c3fb2084cccc4262860aab4f0b5c994ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/23/2020
ms.locfileid: "88756901"
---
# <a name="tutorial-create-a-scala-maven-application-for-apache-spark-in-hdinsight-using-intellij"></a>Tutorial: Creación de una aplicación de Scala Maven para Apache Spark en HDInsight mediante IntelliJ

En este tutorial aprenderá a crear una aplicación de Apache Spark escrita en Scala mediante Apache Maven con IntelliJ IDEA. En el artículo se usa Apache Maven como sistema de compilación y comienza con un arquetipo existente de Maven para Scala proporcionado por IntelliJ IDEA.  Crear una aplicación de Scala en IntelliJ IDEA conlleva los pasos siguientes:

* Use Maven como el sistema de compilación.
* Actualice el archivo del modelo de objetos de proyectos (POM) para resolver las dependencias de módulo de Spark.
* Escriba la aplicación con Scala.
* Genere un archivo jar que se pueda enviar a los clústeres de HDInsight Spark.
* Ejecute la aplicación en un clúster Spark mediante Livy.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Instale el complemento de Scala para IntelliJ IDEA.
> * Usar IntelliJ para desarrollar una aplicación de Scala Maven
> * Creación de un proyecto de Scala independiente

## <a name="prerequisites"></a>Prerrequisitos

* Un clúster de Apache Spark en HDInsight. Para obtener instrucciones, vea [Creación de clústeres Apache Spark en HDInsight de Azure](apache-spark-jupyter-spark-sql.md).

* [Kit de desarrollo de Oracle Java](https://www.azul.com/downloads/azure-only/zulu/).  En este tutorial se usa la versión 8.0.202 de Java.

* Un IDE de Java. En este artículo se usa [IntelliJ IDEA Community  2018.3.4](https://www.jetbrains.com/idea/download/).

* Azure Toolkit for IntelliJ.  Consulte [Instalación de Azure Toolkit for IntelliJ](https://docs.microsoft.com/java/azure/intellij/azure-toolkit-for-intellij-create-hello-world-web-app#installation-and-sign-in).

## <a name="install-scala-plugin-for-intellij-idea"></a>Instale el complemento de Scala para IntelliJ IDEA.

Para instalar el complemento Scala, siga estos pasos:

1. Abra IntelliJ IDEA.

2. En la pantalla de bienvenida, vaya a **Configure** > **Plugins** (Configurar > Complementos) para abrir la ventana **Plugins** (Complementos).

    ![IntelliJ IDEA: habilitar complemento Scala](./media/apache-spark-create-standalone-application/enable-scala-plugin1.png)

3. Seleccione **Install** (Instalar) para el complemento de Scala que se presenta en la nueva ventana.  

    ![IntelliJ IDEA: instalar complemento Scala](./media/apache-spark-create-standalone-application/install-scala-plugin.png)

4. Después de que el complemento se instale correctamente, debe reiniciar el IDE.

## <a name="use-intellij-to-create-application"></a>Uso de IntelliJ para crear la aplicación

1. Inicie IntelliJ IDEA y seleccione **Create New Project** (Crear proyecto) para abrir la ventana **New Project** (Nuevo proyecto).

2. Seleccione **Azure Spark/HDInsight** en el panel izquierdo.

3. Seleccione **Spark Project (Scala)** (Proyecto de Spark [Scala]) en la ventana principal.

4. En la lista desplegable **Build tool** (Herramienta de compilación), seleccione uno de los valores siguientes:
      * **Maven**: para agregar compatibilidad con el asistente para la creación de proyectos de Scala.
      * **SBT** para administrar las dependencias y compilar el proyecto de Scala.

   ![IntelliJ: cuadro de diálogo New Project (Nuevo proyecto)](./media/apache-spark-create-standalone-application/intellij-project-apache-spark.png)

5. Seleccione **Next** (Siguiente).

6. En la ventana **New Project** (Nuevo proyecto), proporcione la siguiente información:  

  	|  Propiedad   | Descripción   |  
  	| ----- | ----- |  
  	|Nombre de proyecto| Escriba un nombre.|  
  	|Project&nbsp;location (Ubicación del proyecto)| Escriba la ubicación para guardar el proyecto.|
  	|Project SDK (SDK del proyecto)| Este campo estará en blanco la primera vez que se use IDEA.  Seleccione **New...** (Nuevo...) y vaya a su JDK.|
  	|Versión de Spark|El asistente de creación integra la versión adecuada de los SDK de Spark y Scala. Si la versión del clúster de Spark es anterior a 2.0, seleccione **Spark 1.x**. De lo contrario, seleccione **Spark2.x**. En este ejemplo se usa **Spark 2.3.0 (Scala 2.11.8)** .|

    ![IDEA de IntelliJ: selección del SDK de Spark](./media/apache-spark-create-standalone-application/hdi-scala-new-project.png)

7. Seleccione **Finalizar**.

## <a name="create-a-standalone-scala-project"></a>Creación de un proyecto de Scala independiente

1. Inicie IntelliJ IDEA y seleccione **Create New Project** (Crear proyecto) para abrir la ventana **New Project** (Nuevo proyecto).

2. Seleccione **Maven** en el panel izquierdo.

3. Especifique un **SDK de proyecto**. Si está en blanco, seleccione **New...** (Nuevo...) y vaya al directorio de instalación de Java.

4. Active la casilla **Create from archetype** (Crear desde arquetipo).  

5. En la lista de arquetipos, seleccione **`org.scala-tools.archetypes:scala-archetype-simple`** . Este arquetipo crea la estructura de directorios adecuada y descargará las dependencias predeterminadas necesarias para escribir el programa con Scala.

    ![IntelliJ IDEA: crear proyecto Maven](./media/apache-spark-create-standalone-application/intellij-project-create-maven.png)

6. Seleccione **Next** (Siguiente).

7. Expanda **Coordenadas de artefacto**. Proporcione los valores correspondientes para **GroupId** (Id. de grupo) y **ArtifactId** (Id. de artefacto). **Name** (Nombre) y **Location** (Ubicación) se rellenarán automáticamente. Los siguientes valores se utilizan en este tutorial:

    - **GroupId:** com.microsoft.spark.example
    - **ArtifactId:** SparkSimpleApp

    ![IntelliJ IDEA: crear proyecto Maven](./media/apache-spark-create-standalone-application/intellij-artifact-coordinates.png)

8. Seleccione **Next** (Siguiente).

9. Compruebe la configuración y seleccione **Next** (Siguiente).

10. Compruebe el nombre y la ubicación del proyecto, y seleccione **Finish**(Finalizar).  El proyecto tardará unos minutos en importarse.

11. Una vez importado el proyecto, en el panel izquierdo, vaya a **SparkSimpleApp** > **src** > **test** > **scala** > **com** > **microsoft** > **spark** > **example**.  Haga clic con el botón derecho en **MySpec** y, luego, seleccione **Delete...** (Eliminar...). Este archivo no es necesario para la aplicación.  Seleccione **OK** (Aceptar) en el cuadro de diálogo.
  
12. En los pasos siguientes, actualizará el archivo **pom.xml** para definir las dependencias de la aplicación de Scala para Spark. Para que tales dependencias se descarguen y resuelvan automáticamente, debe configurar Maven.

13. En el menú **File** (Archivo), seleccione **Settings** (Configuración) para abrir la ventana **Settings** (Configuración).

14. En la ventana **Settings** (Configuración), vaya a **Build, Execution, Deployment** > **Build Tools** > **Maven** > **Importing** (Compilación, ejecución o implementación > Herramientas de compilación > Maven > Importación).

15. Active la casilla **Import Maven projects automatically** (Importar proyectos de Maven automáticamente).

16. Seleccione **Aplicar** y luego **Aceptar**.  Volverá a la ventana del proyecto.

    ![Configurar Maven para descargas automáticas](./media/apache-spark-create-standalone-application/configure-maven-download.png)

17. En el panel izquierdo, vaya a **src** > **main** > **scala** > **com.microsoft.spark.example**y haga doble clic en **App** para abrir App.scala.

18. Reemplace el código de ejemplo existente por el código siguiente y guarde los cambios. Este código lee los datos de HVAC.csv (disponible en todos los clústeres de HDInsight Spark). Recupera las filas que solo tienen un dígito en la sexta columna. Luego escribe el resultado en **/HVACOut** en el contenedor de almacenamiento predeterminado para el clúster.

    ```scala
    package com.microsoft.spark.example

    import org.apache.spark.SparkConf
    import org.apache.spark.SparkContext

    /**
      * Test IO to wasb
      */
    object WasbIOTest {
        def main (arg: Array[String]): Unit = {
            val conf = new SparkConf().setAppName("WASBIOTest")
            val sc = new SparkContext(conf)
    
            val rdd = sc.textFile("wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")
    
            //find the rows which have only one digit in the 7th column in the CSV
            val rdd1 = rdd.filter(s => s.split(",")(6).length() == 1)
    
            rdd1.saveAsTextFile("wasb:///HVACout")
        }
    }
    ```

19. En el panel izquierdo, haga doble clic en **pom.xml**.  

20. En `<project>\<properties>`, agregue los siguientes segmentos:

    ```xml
    <scala.version>2.11.8</scala.version>
    <scala.compat.version>2.11.8</scala.compat.version>
    <scala.binary.version>2.11</scala.binary.version>
    ```

21. En `<project>\<dependencies>`, agregue los siguientes segmentos:

    ```xml
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-core_${scala.binary.version}</artifactId>
        <version>2.3.0</version>
    </dependency>
    ```

    Guarde los cambios en pom.xml.

22. Cree el archivo .jar. IntelliJ IDEA permite crear JAR como un artefacto de un proyecto. Siga estos pasos.

    1. En el menú **File** (Archivo), seleccione **Project Structure...** (Estructura de proyecto).

    2. En la ventana **Project Structure** (Estructura de proyecto), vaya a **Artifacts** > **the plus symbol +**  > **JAR** > **From modules with dependencies...** (Artefactos > el símbolo + > JAR > Desde módulos con dependencias).

        ![Estructura del proyecto IntelliJ IDEA: agregar jar](./media/apache-spark-create-standalone-application/hdinsight-create-jar1.png)

    3. En la ventana **Create JAR from Modules** (Crear JAR a partir de módulos), seleccione el icono de carpeta en el cuadro de texto **Main Class** (Clase principal).

    4. En el cuadro de diálogo **Select Main Class** (Seleccionar clase principal), seleccione la clase que aparece de forma predeterminada y, luego, seleccione **OK** (Aceptar).

        ![Estructura del proyecto IntelliJ IDEA: seleccionar clase](./media/apache-spark-create-standalone-application/hdinsight-create-jar2.png)

    5. En la ventana **Create JAR from Modules** (Crear JAR desde módulos), asegúrese de que está seleccionada la opción **Extract to the target JAR** (Extraer al archivo JAR de destino) y, después, seleccione **OK** (Aceptar).  Este valor crea un archivo JAR único con todas las dependencias.

        ![Estructura del proyecto IDEA de IntelliJ: jar desde módulo](./media/apache-spark-create-standalone-application/hdinsight-create-jar3.png)

    6. La pestaña **Output Layout** (Diseño de salida) enumera todos los archivos JAR que forman parte del proyecto Maven. Puede seleccionar y eliminar aquellos de los que la aplicación de Scala no tenga ninguna dependencia directa. Para la aplicación que va a crear aquí, puede quitar todos, salvo el último (**SparkSimpleApp compile output**). Seleccione los archivos JAR que va a eliminar y, después, seleccione símbolo negativo **-** .

        ![Estructura del proyecto IntelliJ IDEA: eliminar salida](./media/apache-spark-create-standalone-application/hdi-delete-output-jars.png)

        Asegúrese de que la casilla **Include in project build** (Incluir en compilación del proyecto) está activada, lo que garantiza que el archivo jar se crea cada vez que el proyecto se compila o actualiza. Seleccione **Apply** (Aplicar) y, después, **OK** (Aceptar).

    7. Para crear el archivo JAR, vaya a **Build** > **Build Artifacts** > **Build** (Compilar > Artefactos de compilación > Compilar). El proyecto se compilará al cabo de 30 segundos aproximadamente.  El archivo JAR de salida se crea en **\out\artifacts**.

        ![Proyecto IDEA de IntelliJ: salida de artefacto](./media/apache-spark-create-standalone-application/hdi-artifact-output-jar.png)

## <a name="run-the-application-on-the-apache-spark-cluster"></a>Ejecución de la aplicación en el clúster de Apache Spark

Para ejecutar la aplicación en el clúster, puede usar los siguientes enfoques:

* **Copie el archivo jar de la aplicación en el blob de Azure Storage** asociado con el clúster. Puede usar **AzCopy**, una utilidad de línea de comandos, para hacerlo. Hay muchos otros clientes que se pueden utilizar también para cargar datos. Puede encontrar más información al respecto en [Carga de datos para trabajos de Apache Hadoop en HDInsight](../hdinsight-upload-data.md).

* **Use Apache Livy para enviar un trabajo de la aplicación de manera remota** al clúster Spark. Los clústeres Spark en HDInsight incluye Livy, que expone los puntos de conexión REST para enviar trabajos de Spark de forma remota. Para más información, consulte [Envío remoto de trabajos de Apache Spark mediante la utilización de Apache Livy con clústeres Spark en HDInsight](apache-spark-livy-rest-interface.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimine el clúster que creó mediante los siguientes pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. En el cuadro **Búsqueda** en la parte superior, escriba **HDInsight**.

1. Seleccione **Clústeres de HDInsight** en **Servicios**.

1. En la lista de clústeres de HDInsight que aparece, seleccione el signo **...** situado junto al clúster que ha creado para este tutorial.

1. Seleccione **Eliminar**. Seleccione **Sí**.

![Eliminación de clúster de HDInsight en Azure Portal](./media/apache-spark-create-standalone-application/hdinsight-azure-portal-delete-cluster.png "Eliminación de un clúster de HDInsight")

## <a name="next-step"></a>Paso siguiente

En este artículo ha aprendido a crear una aplicación en Scala para Apache Spark. Vaya al siguiente artículo para aprender a ejecutar esta aplicación en un clúster de HDInsight Spark con Livy.

> [!div class="nextstepaction"]
>[Ejecución de trabajos de forma remota en un clúster de Apache Spark mediante Apache Livy](./apache-spark-livy-rest-interface.md)
