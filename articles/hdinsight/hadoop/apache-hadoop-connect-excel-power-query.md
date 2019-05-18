---
title: Conexión de Excel a Apache Hadoop con Power Query en Azure HDInsight
description: Vea cómo aprovechar las ventajas de los componentes de inteligencia empresarial y usar Power Query para Excel a fin de acceder a los datos almacenados en Hadoop en HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/16/2018
ms.openlocfilehash: e35e86bee4dfee21d0a98867c3e0d1d7850ed0ec
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65762004"
---
# <a name="connect-excel-to-apache-hadoop-by-using-power-query"></a>Conexión de Excel a Apache Hadoop con Power Query
Una de las características clave de la solución para Big Data de Microsoft es la integración de los componentes de Microsoft Business Intelligence (BI) con los clústeres de Apache Hadoop en Azure HDInsight. Un ejemplo importante es la capacidad de conectar Excel a la cuenta de Azure Storage que contiene los datos asociados a su clúster de Hadoop mediante el complemento de Microsoft Power Query para Excel. En este artículo se describen las pautas para configurar y usar Power Query para consultar los datos asociados con un clúster de Hadoop administrado con HDInsight.

### <a name="prerequisites"></a>Requisitos previos
Antes de empezar este artículo, debe tener los siguientes elementos:

* **Un clúster de HDInsight**. Para configurar uno, vea [Get started with Azure HDInsight (Introducción a Azure HDInsight)][hdinsight-get-started].
* **Una estación de trabajo** que ejecute Windows 7, Windows Server 2008 R2 o un sistema operativo posterior.
* **Office 2016, Office Profesional Plus 2013, Office 365 ProPlus, Excel 2013 Standalone u Office Professional Plus 2010**.

## <a name="install-power-query"></a>Instalación de Power Query
Power Query puede importar datos ofrecidos o generados por un trabajo de Hadoop ejecutado en un clúster de HDInsight.

En Excel 2016, Power Query se ha integrado en la cinta de Datos en la sección Obtener y transformar. Para versiones anteriores de Excel, descargue Microsoft Power Query para Excel desde el [Centro de descarga de Microsoft][powerquery-download] e instálelo.

## <a name="import-hdinsight-data-into-excel"></a>Importación de datos de HDInsight a Excel
El complemento de Power Query para Excel facilita la importación de datos desde el clúster de HDInsight hasta Excel, donde se pueden usar herramientas de BI como PowerPivot y Power Map para la inspección, el análisis y la presentación de los datos.

**Para importar datos desde un clúster de HDInsight**

1. Abra Excel.
2. Cree un libro vacío.
3. Siga los pasos siguientes en función de la versión de Excel:

   - Excel 2016

     - Haga clic en el menú **Datos**, haga clic en **Obtener datos** desde la cinta **Obtener y transformar datos**, haga clic en **Desde Azure** y, después, en **Desde Azure HDInsight (HDFS)**.

       ![HDI.PowerQuery.SelectHdiSource](./media/apache-hadoop-connect-excel-power-query/hdi.powerquery.selecthdisource.excel2016.png)

   - Excel 2013/2010

     - Haga clic en el menú **Power Query**, en **De Azure** y, luego, en **De Microsoft Azure HDInsight**.
   
       ![HDI.PowerQuery.SelectHdiSource][image-hdi-powerquery-hdi-source]
       
       **Nota:** Si no ve el **Power Query** menú, vaya a **archivo** > **opciones** > **complementos**y seleccione  **Complementos COM** en la lista desplegable **administrar** cuadro en la parte inferior de la página. Elija el botón **Go...** y compruebe que la casilla del complemento de Power Query para Excel esté activada.
       
       **Nota:** Power Query también permite importar datos de HDFS si hace clic en **Desde otros orígenes**.
4. En **Nombre de cuenta**, escriba el nombre de la cuenta de Azure Blob Storage asociada con su clúster y, a continuación, haga clic en **Aceptar**. Esta cuenta puede ser la cuenta de almacenamiento predeterminada o una cuenta de almacenamiento vinculada.  El formato es *https://&lt;NombreDeCuentaDeAlmacenamiento>.blob.core.windows.net/*.
5. En **Clave de cuenta**, escriba la clave de cuenta para la cuenta deBlob Storage y, a continuación, haga clic en **Guardar**. (Solo tiene que escribir la información de la cuenta la primera vez que tenga acceso a este almacén).
6. En el panel del **navegador** situado a la izquierda del Editor de consultas, haga doble clic en el nombre del contenedor de almacenamiento de blobs. De forma predeterminada, el nombre del contenedor es el mismo que el del clúster.
7. Busque **HiveSampleData.txt** en la columna **Nombre** (la ruta de acceso de la carpeta es **../hive/warehouse/hivesampletable/)**, y haga clic en **Binario** a la izquierda de HiveSampleData.txt. HiveSampleData.txt incluye todo el clúster. Opcionalmente, puede utilizar su propio archivo.
   
    ![HDI.PowerQuery.ImportData][image-hdi-powerquery-importdata]
8. Si quiere, puede cambiar el nombre de las columnas. Cuando haya terminado, haga clic en **Aplicar y cerrar**.  Los datos se han cargado en el libro:
   
    ![HDI.PowerQuery.ImportedTable][image-hdi-powerquery-imported-table]

## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha aprendido a usar Power Query para recuperar datos de HDInsight en Excel. Del mismo modo, puede recuperar datos de HDInsight en Azure SQL Database. También se pueden cargar los datos en HDInsight. Para obtener más información, consulte los artículos siguientes:

* [Visualización de datos de Apache Hive con Microsoft Power BI en Azure HDInsight](apache-hadoop-connect-hive-power-bi.md).
* [Visualización de datos de Interactive Query Hive con Power BI en Azure HDInsight](../interactive-query/apache-hadoop-connect-hive-power-bi-directquery.md).
* [Uso de Apache Zeppelin para ejecutar consultas de Apache Hive en Azure HDInsight](../interactive-query/hdinsight-connect-hive-zeppelin.md).
* [Conexión de Excel a Hadoop en Azure HDInsight con Microsoft Hive ODBC Driver](apache-hadoop-connect-excel-hive-odbc-driver.md).
* [Conectarse a Azure HDInsight y ejecutar consultas de Apache Hive con Herramientas de Data Lake para Visual Studio](apache-hadoop-visual-studio-tools-get-started.md).
* [Uso de Herramientas de Azure HDInsight para Visual Studio Code](../hdinsight-for-vscode.md).
* [Carga de datos en HDInsight](./../hdinsight-upload-data.md).

[image-hdi-powerquery-hdi-source]: ./media/apache-hadoop-connect-excel-power-query/hdi.powerquery.selecthdisource.png
[image-hdi-powerquery-importdata]: ./media/apache-hadoop-connect-excel-power-query/hdi.powerquery.importdata.png
[image-hdi-powerquery-imported-table]: ./media/apache-hadoop-connect-excel-power-query/hdi.powerquery.importedtable.PNG

[powerquery-download]: https://go.microsoft.com/fwlink/?LinkID=286689
