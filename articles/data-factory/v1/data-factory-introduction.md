---
title: Introducción a Data Factory, un servicio de integración de datos
description: Sepa lo que es Azure Data Factory, un servicio de integración de datos basado en la nube que organiza y automatiza el movimiento y la transformación de datos.
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: maghan
ms.assetid: cec68cb5-ca0d-473b-8ae8-35de949a009e
ms.service: data-factory
ms.workload: data-services
ms.topic: overview
ms.date: 01/22/2018
ms.openlocfilehash: 06f308b4e0b2df0487a351ae38f16f3872961cfb
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89436263"
---
# <a name="introduction-to-azure-data-factory"></a>Introducción a Azure Data Factory 
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](data-factory-introduction.md)
> * [Versión 2 (versión actual)](../introduction.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Azure Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte [Introducción a Data Factory V2](../introduction.md).


## <a name="what-is-azure-data-factory"></a>¿Qué es Azure Data Factory?
En el mundo de los macrodatos, ¿cómo aprovechan las empresas los datos existentes? ¿Es posible enriquecer los datos generados en la nube mediante el uso de datos de referencia de orígenes de datos locales u otros orígenes de datos dispares? 

Por ejemplo, una empresa de juegos recopila registros generados por juegos en la nube. Desea analizar estos registros para obtener información sobre las preferencias de los clientes, datos demográficos, comportamiento de uso, etc. La empresa también quiere identificar oportunidades de aumento de ventas y de venta cruzada y agregar nuevas características atractivas para impulsar el crecimiento del negocio y ofrecer una mejor experiencia a los clientes. 

Para analizar estos registros, la empresa debe usar datos de referencia, como información de los clientes, información de los juegos e información de las campañas de marketing, que se encuentran en un almacén de datos local. Por lo tanto, desean ingerir datos de registro del almacén de datos en la nube y datos de referencia del almacén de datos local. 

A continuación desean procesar los datos con Hadoop en la nube (Azure HDInsight). Quieren publicar los datos de resultados en un almacén de datos en la nube, como Azure Synapse Analytics (anteriormente Azure SQL Data Warehouse), o en un almacén de datos local, como SQL Server. La empresa quiere que este flujo de trabajo se ejecute una vez a la semana. 

La empresa necesita una plataforma en la que se cree un flujo de trabajo donde se puedan introducir datos de almacenes tanto locales como en la nube. La empresa también necesita transformar o procesar datos mediante servicios de proceso existentes, como Hadoop, y publicar los resultados en un almacén de datos local o en la nube para que los consuman aplicaciones de inteligencia empresarial. 

![Introducción a Data Factory](media/data-factory-introduction/what-is-azure-data-factory.png) 

Azure Data Factory es la plataforma para estos tipos de escenario. Se trata de un *servicio de integración de datos basado en la nube que permite crear flujos de trabajo basados en datos en la nube a fin de orquestar y automatizar el movimiento y la transformación de datos*. Con Azure Data Factory, puede realizar las tareas siguientes: 

- Creación y programación de flujos de trabajo basados en datos flujos (denominados "canalizaciones") que ingieren datos de almacenes de datos dispares.

- Procesamiento o transformación de los datos con servicios de proceso tipo Azure HDInsight Hadoop, Spark, Azure Data Lake Analytics y Azure Machine Learning.

-  Publicación de datos de salida en almacenes de datos como Azure Synapse Analytics para que las aplicaciones de inteligencia empresarial los consuman.  

Es más una plataforma de extracción y carga (EL) y de transformación y carga (TL) que una de extracción, transformación y carga (ETL) tradicional. En las transformaciones se procesan los datos con servicios de proceso, en lugar de al incorporar columnas derivadas, contar el número de filas, ordenar los datos, etc. 

Actualmente, en Azure Data Factory, los datos que consumen y generan los flujos de trabajo son *datos fragmentados por tiempo* (por hora, al día, a la semana, etc.). Por ejemplo, una canalización podría leer datos de entrada, procesar datos y generar datos de salida una vez al día. También puede ejecutar un flujo de trabajo una sola vez.  
  

## <a name="how-does-it-work"></a>¿Cómo funciona? 
Las canalizaciones (flujos de trabajo orientados a datos) en Azure Data Factory realizan normalmente los tres pasos siguientes:

![Tres fases de Azure Data Factory](media/data-factory-introduction/three-information-production-stages.png)

### <a name="connect-and-collect"></a>Conectar y recopilar
Las empresas tienen datos de varios tipos ubicados en orígenes dispares. El primer paso para la compilación de un sistema de producción de información es la conexión a todos los orígenes de datos y procesamientos necesarios. Estos orígenes incluyen servicios de SaaS, recursos compartidos de archivos, FTP y servicios web. A continuación, mueva los datos como sea necesario a una ubicación centralizada para el procesamiento posterior.

Sin Data Factory, las empresas deben crear componentes de movimiento de datos personalizados o escribir servicios personalizados para integrar estos orígenes de datos y procesamientos. Estos sistemas resultan costosos y difíciles de integrar y mantener. A menudo, también carecen de la supervisión de grado empresarial, las alertas y los controles que ofrece un servicio completamente administrado.

Con Data Factory, puede usar la actividad de copia en una canalización de datos para mover los datos desde almacenes de datos de orígenes locales y en la nube a un almacén de datos de centralización en la nube para su posterior análisis. 

Por ejemplo, puede recopilar datos de Azure Data Lake Store y transformarlos posteriormente mediante un servicio de proceso de Azure Data Lake Analytics. O también puede recopilar datos en Azure Blob Storage y transformarlos más adelante con un clúster de Azure HDInsight Hadoop.

### <a name="transform-and-enrich"></a>Transformar y enriquecer
Con los datos presentes en un almacén de datos centralizado en la nube, podrá procesarlos o transferirlos mediante servicios de proceso como HDInsight Hadoop, Spark, Data Lake Analytics o Machine Learning. Querrá generar de manera confiable datos transformados según una programación controlada y fácil de mantener a fin de alimentar los entornos de producción con datos de confianza. 

### <a name="publish"></a>Publicar 
Entregue datos transformados de la nube a orígenes locales como SQL Server. También puede conservarlos en los orígenes de almacenamiento en la nube para que los consuma la inteligencia empresarial y las herramientas de Analytics, así como otras aplicaciones.

## <a name="key-components"></a>Componentes claves
Una suscripción de Azure puede tener una o varias instancias de Azure Data Factory (o factorías de datos). Azure Data Factory consta de cuatro componentes principales. Estos componentes funcionan juntos para proporcionar la plataforma en la que pueda crear flujos de trabajo basados en datos con pasos para moverlos y transformarlos. 

### <a name="pipeline"></a>Canalización
Una factoría de datos puede tener una o más canalizaciones. Una canalización es un grupo de actividades. Juntas, las actividades de una canalización realizan una tarea. 

Por ejemplo, una canalización puede contener un grupo de actividades que ingiere datos de un blob de Azure y luego ejecutar una consulta de Hive en un clúster de HDInsight para particionar los datos. La ventaja de esto es que la canalización le permite administrar las actividades como un conjunto en lugar de tener que administrar cada una de ellas individualmente. Por ejemplo, puede implementar y programar la canalización completa, en lugar de programar actividades independientes. 

### <a name="activity"></a>Actividad
Una canalización puede tener una o más actividades. Las actividades definen las acciones que se van a realizar en los datos. Por ejemplo, puede utilizar una actividad de copia para copiar datos de un almacén de datos a otro. De forma similar, puede usar una actividad de Hive. Una actividad de Hive ejecuta una consulta de Hive en un clúster de Azure HDInsight para transformar o analizar los datos. Data Factory admite dos tipos de actividades: actividades de movimiento de datos y actividades de transformación de datos.

### <a name="data-movement-activities"></a>Actividades de movimiento de datos
Copiar actividad en Data Factory realiza una copia de los datos de un almacén de datos de origen a uno receptor. Se pueden escribir datos desde cualquier origen en todos los tipos de receptores. Seleccione un almacén de datos para aprender a copiar datos sus datos o a copiar datos en él. Data Factory admite los siguientes almacenes de datos:

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

Para más información, consulte [Movimiento de datos con la actividad de copia](data-factory-data-movement-activities.md).

### <a name="data-transformation-activities"></a>Actividades de transformación de datos
[!INCLUDE [data-factory-transformation-activities](../../../includes/data-factory-transformation-activities.md)]

Para más información, consulte [Movimiento de datos con la actividad de copia](data-factory-data-transformation-activities.md).

### <a name="custom-net-activities"></a>Actividades personalizadas de .NET
Cree una actividad personalizada de .NET si necesita mover datos a un almacén de datos, o desde este, que no sean compatibles con la actividad de copia, o si necesita transformar datos con su propia lógica. Consulte [Uso de actividades personalizadas en una canalización de Azure Data Factory](data-factory-use-custom-activities.md) para más información sobre la creación y el uso de una actividad personalizada.

### <a name="datasets"></a>Conjuntos de datos
Cada actividad toma cero o más conjuntos de datos como entrada y genera uno o varios conjuntos de datos como salida. Los conjuntos de datos representan estructuras de datos en los almacenes de datos. Estas estructuras apuntan o hacen referencia a los datos que desea utilizar en las actividades (por ejemplo, como entradas o salidas). 

Por ejemplo, un conjunto de datos de blob de Azure especifica el contenedor de blobs y la carpeta de Azure Blob Storage de los que la canalización debe leer los datos. O bien, un conjunto de datos de tabla de Azure SQL especifica la tabla en la que la actividad escribe los datos de salida. 

### <a name="linked-services"></a>Servicios vinculados
Los servicios vinculados son muy similares a las cadenas de conexión que definen la información de conexión necesaria para que Data Factory se conecte a recursos externos. Considérelos de esta forma: un servicio vinculado define la conexión al origen de datos y un conjunto de datos representa la estructura de los datos. 

Por ejemplo, un servicio vinculado de Azure Storage especifica la cadena de conexión para conectarse a la cuenta de Azure Storage. Un conjunto de datos de un blob de Azure especifica el contenedor de blobs y la carpeta que contiene los datos.   

Los servicios vinculados se utilizan en Data Factory por dos motivos:

* Para representar un *almacén de datos* que incluye, entre otros, una base de datos de SQL Server, una base de datos de Oracle, un recurso compartido de archivos o una cuenta de Azure Blob Storage. Para ver una lista de los almacenes de datos compatibles, consulte la sección [Actividades de movimiento de datos](#data-movement-activities) .

* Para representar un *recurso de proceso* que puede hospedar la ejecución de una actividad. Por ejemplo, la actividad HDInsightHive se ejecuta en un clúster de Hadoop para HDInsight. Consulte la sección [Actividades de transformación de datos](#data-transformation-activities) para ver una lista de los entornos de proceso admitidos.

### <a name="relationship-between-data-factory-entities"></a>Relación entre las entidades de Data Factory

![Diagrama: Data Factory, un servicio de integración de datos en la nube (conceptos clave)](./media/data-factory-introduction/data-integration-service-key-concepts.png)

## <a name="supported-regions"></a>Regiones admitidas
Actualmente, se pueden crear factorías de datos en las regiones Oeste de EE. UU., Este de EE.UU. y Norte de Europa. Sin embargo, una factoría de datos puede acceder a almacenes de datos y a servicios de proceso en otras regiones de Azure para mover datos entre almacenes de datos o para procesarlos mediante servicios de proceso.

Azure Data Factory no almacena ningún dato. Permite crear flujos de trabajo basados en datos para orquestar el movimiento de datos entre [almacenes de datos compatibles](#data-movement-activities). Por otra parte, permite procesar datos mediante [servicios de proceso](#data-transformation-activities) en otras regiones o en un entorno local. También permite [supervisar y administrar flujos de trabajo](data-factory-monitor-manage-pipelines.md) mediante mecanismos de programación y de interfaz de usuario.

Data Factory solo está disponible en las regiones Oeste de EE. UU., Este de EE. UU. y Norte de Europa. Sin embargo, el servicio que alimenta el movimiento de datos en Data Factory está disponible [globalmente](data-factory-data-movement-activities.md#global) en varias regiones. Si un almacén de datos se encuentra detrás de un firewall, será una instancia de [Data Management Gateway](data-factory-move-data-between-onprem-and-cloud.md) instalada en el entorno local la que mueva los datos en su lugar.

Por ejemplo, supongamos que sus entornos de proceso, tales como un clúster de Azure HDInsight y Azure Machine Learning, se encuentran en la región Oeste de Europa. Puede crear y utilizar una instancia de Azure Data Factory de Norte de Europa. Después, puede usarla para programar trabajos en los entornos de proceso de Oeste de Europa. Data Factory tarda unos milisegundos en desencadenar el trabajo en su entorno de proceso, pero el tiempo para ejecutar el trabajo en el entorno de proceso no cambia.

## <a name="get-started-with-creating-a-pipeline"></a>Introducción a la creación de una canalización
Puede usar una de esas herramientas o una de estas API para crear canalizaciones de datos en Azure Data Factory: 

- Visual Studio
- PowerShell
- API de .NET
- API DE REST
- Plantilla del Administrador de recursos de Azure

Para aprender a compilar factorías de datos con canalizaciones de datos, siga las instrucciones paso a paso de los siguientes tutoriales:

| Tutorial | Descripción |
| --- | --- |
| [Movimiento de datos entre dos almacenes de datos en la nube](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) |Cree una factoría de datos con una canalización que mueva datos de Blob Storage a SQL Database. |
| [Transformación de datos con el clúster de Hadoop](data-factory-build-your-first-pipeline.md) |Compile su primera instancia de Azure Data Factory con una canalización de datos que procese los datos al ejecutar un script de Hive en un clúster de Azure HDInsight (Hadoop). |
| [Movimiento de datos entre un almacén de datos local y un almacén de datos en la nube con Data Management Gateway](data-factory-move-data-between-onprem-and-cloud.md) |Compile una factoría de datos con una canalización que mueva los datos de una base de datos de SQL Server a un blob de Azure. Como parte del tutorial, instalará y configurará la puerta de enlace de administración de datos en su máquina. |
