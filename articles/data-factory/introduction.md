---
title: Introducción a Azure Data Factory
description: Sepa lo que es Azure Data Factory, un servicio de integración de datos basado en la nube que organiza y automatiza el movimiento y la transformación de datos.
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
ms.service: data-factory
ms.workload: data-services
ms.topic: overview
ms.date: 09/30/2019
ms.openlocfilehash: 1840bf93cbca73e593465c999b416e7cbd7af201
ms.sourcegitcommit: c52e50ea04dfb8d4da0e18735477b80cafccc2cf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/08/2020
ms.locfileid: "89536319"
---
# <a name="what-is-azure-data-factory"></a>¿Qué es Azure Data Factory?

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En el mundo de los macrodatos, los datos sin procesar y desorganizados suelen almacenarse en sistemas de almacenamiento relacionales, no relacionales y de otros tipos. Sin embargo, en sí mismos, los datos sin procesar no tienen el contexto o el significado adecuados para proporcionar información significativa a los analistas, científicos de datos y responsables de decisiones empresariales. 

Los macrodatos requieren un servicio que pueda orquestar y operacionalizar los procesos para convertir estos enormes almacenes de datos sin procesar en información empresarial en función de la cual se puedan emprender acciones. Azure Data Factory es un servicio en la nube administrado creado para estos complejos proyectos híbridos de extracción, transformación y carga (ETL), extracción, carga y transformación (ELT) e integración de datos.

Por ejemplo, imagine una empresa de juegos que recopila petabytes de registros de juegos generados por juegos en la nube. La empresa quiere analizar estos registros para obtener información sobre las preferencias de los clientes, los datos demográficos y el comportamiento de uso. También quiere identificar oportunidades de venta cruzada y vertical, desarrollar nuevas y atractivas características, impulsar el crecimiento empresarial y ofrecer una mejor experiencia a sus clientes.

Para analizar estos registros, la empresa debe usar datos de referencia, como información de clientes, información de juegos e información de campañas de marketing, que se encuentran en un almacén de datos local. La empresa quiere usar estos datos del almacén de datos local y combinarlos con datos de registro adicionales que tiene en un almacén de datos en la nube. 

Para extraer información, espera procesar los datos combinados mediante un clúster de Spark en la nube (Azure HDInsight) y publicar los datos transformados en un almacenamiento de datos en la nube como Azure Synapse Analytics (anteriormente SQL Data Warehouse) para generar fácilmente un informe sobre él. Quieren automatizar este flujo de trabajo y supervisarlo y administrarlo según una programación diaria. También quieren ejecutarlo cuando los archivos lleguen a un contenedor de almacén de blobs.

Azure Data Factory es la plataforma que resuelve estos escenarios de datos. Se trata de un *servicio de integración de datos y ETL basado en la nube que le permite crear flujos de trabajo orientados a datos a fin de coordinar el movimiento y la transformación de datos a escala*. Con Azure Data Factory, puede crear y programar flujos de trabajo basados en datos (llamados canalizaciones) que pueden ingerir datos de distintos almacenes de datos. Puede crear procesos ETL complejos que transformen datos visualmente con flujos de datos o mediante servicios de proceso como Azure HDInsight Hadoop, Azure Databricks y Azure SQL Database. 

Además, puede publicar los datos transformados en almacenes de datos, como Azure Synapse Analytics, para que los consuman aplicaciones de inteligencia empresarial (BI). En última instancia, mediante Azure Data Factory, los datos sin procesar se pueden organizar en almacenes de datos y Data Lakes significativos para tomar mejores decisiones empresariales.

![Vista de nivel superior de Data Factory](media/data-flow/overview.png)

## <a name="how-does-it-work"></a>¿Cómo funciona?

Data Factory contiene una serie de sistemas interconectados que proporcionan una plataforma completa de un extremo a otro para los ingenieros de datos.

### <a name="connect-and-collect"></a>Conectar y recopilar

Las empresas tienen datos de varios tipos que se encuentran en orígenes locales dispares, en la nube, estructurados, no estructurados y semiestructurados, que llegan todos según distintos intervalos y velocidades. 

El primer paso en la creación de un sistema de producción de información es conectarse a todos los orígenes de datos y procesamiento necesarios, como servicios de software como servicio (SaaS), bases de datos, recursos compartidos de archivos y servicios web FTP. El siguiente paso es mover los datos según sea necesario a una ubicación centralizada para su posterior procesamiento.

Sin Data Factory, las empresas deben crear componentes de movimiento de datos personalizados o escribir servicios personalizados para integrar estos orígenes de datos y procesamientos. Estos sistemas son costosos y difíciles de mantener e integrar. Además, con frecuencia carecen de la supervisión de grado empresarial, las alertas y los controles que puede ofrecer un servicio totalmente administrado.

Con Data Factory, puede usar la [actividad de copia](copy-activity-overview.md) en una canalización de datos para mover los datos desde almacenes de datos de origen locales y en la nube a un almacén de datos de centralización en la nube para su posterior análisis. Por ejemplo, puede recopilar datos de Azure Data Lake Storage y transformarlos posteriormente mediante un servicio de proceso de Azure Data Lake Analytics. También puede recopilar datos de Azure Blob Storage y transformarlos más adelante mediante un clúster de Azure HDInsight Hadoop.

### <a name="transform-and-enrich"></a>Transformar y enriquecer
Cuando los datos están presentes en un almacén de datos centralizado en la nube, procese o transforme los datos recopilados mediante flujos de datos de asignación de ADF. Los flujos de datos permiten a los ingenieros de datos crear y mantener gráficos de transformación de datos que se ejecutan en Spark sin necesidad de comprender los clústeres de Spark o la programación de Spark.

Si prefiere codificar las transformaciones a mano, ADF admite actividades externas para ejecutar las transformaciones en servicios de proceso como HDInsight Hadoop, Spark, Data Lake Analytics y Machine Learning.

### <a name="cicd-and-publish"></a>CI/CD y publicación
Data Factory ofrece compatibilidad total con CI/CD de sus canalizaciones de datos mediante Azure DevOps y GitHub. Esto le permite desarrollar y distribuir incrementalmente los procesos ETL antes de publicar el producto terminado. Después de que se han procesado los datos sin procesar en un formato compatible listo para la empresa, cargue los datos en Azure Data Warehouse, Azure SQL Database, Azure CosmosDB o un motor de análisis al que puedan apuntar los usuarios con sus herramientas de inteligencia empresarial.

### <a name="monitor"></a>Supervisión
Una vez creada e implementada correctamente la canalización de integración de datos, que proporciona un valor empresarial a partir de datos procesados, supervise las canalizaciones y las actividades programadas para ver las tasas de éxito y error. Azure Data Factory tiene compatibilidad integrada para la supervisión de canalizaciones mediante Azure Monitor, API, PowerShell, los registros de Azure Monitor y los paneles de mantenimiento de Azure Portal.

## <a name="top-level-concepts"></a>Conceptos de nivel superior
Una suscripción de Azure puede tener una o varias instancias de Azure Data Factory (o factorías de datos). Azure Data Factory consta de los siguientes componentes principales.
- Pipelines
- Actividades
- Conjuntos de datos
- Servicios vinculados
- Flujos de datos
- Integration Runtime

Estos componentes funcionan juntos para proporcionar la plataforma en la que pueda crear flujos de trabajo basados en datos con pasos para moverlos y transformarlos.

### <a name="pipeline"></a>Canalización
Una factoría de datos puede tener una o más canalizaciones. La canalización es una agrupación lógica de actividades para realizar una unidad de trabajo. Juntas, las actividades de una canalización realizan una tarea. Por ejemplo, una canalización puede contener un grupo de actividades que ingiere datos de un blob de Azure y luego ejecutar una consulta de Hive en un clúster de HDInsight para particionar los datos. 

La ventaja de esto es que la canalización le permite administrar las actividades como un conjunto en lugar de tener que administrar cada una de ellas individualmente. Las actividades de una canalización se pueden encadenar juntas para operar de forma secuencial o pueden funcionar de forma independiente en paralelo.

### <a name="mapping-data-flows"></a>Asignación de flujos de datos
Cree y administre gráficos de lógica de transformación de datos que puede usar para transformar datos de cualquier tamaño. Puede crear una biblioteca reutilizable de rutinas de transformación de datos y ejecutar esos procesos con escalabilidad horizontal desde las canalizaciones de ADF. Data Factory ejecutará la lógica en un clúster de Spark que se pone en marcha y se detiene cuando lo necesita. Nunca tendrá que administrar o mantener clústeres.

### <a name="activity"></a>Actividad
Las actividades representan un paso del procesamiento en una canalización. Por ejemplo, puede usar una actividad de copia para copiar datos de un almacén de datos a otro. De igual forma, puede usar una actividad de Hive, que ejecuta una consulta de Hive en un clúster de Azure HDInsight para transformar o analizar los datos. Data Factory admite tres tipos de actividades: actividades de movimiento de datos, actividades de transformación de datos y actividades de control.

### <a name="datasets"></a>Conjuntos de datos
Los conjuntos de datos representan las estructuras de datos de los almacenes de datos que simplemente apuntan o hacen referencia a los datos que desea utilizar en sus actividades como entradas o salidas. 

### <a name="linked-services"></a>Servicios vinculados
Los servicios vinculados son muy similares a las cadenas de conexión que definen la información de conexión necesaria para que Data Factory se conecte a recursos externos. Considérelos de esta forma: un servicio vinculado define la conexión al origen de datos y un conjunto de datos representa la estructura de los datos. Por ejemplo, un servicio vinculado de Azure Storage especifica la cadena de conexión para conectarse a la cuenta de Azure Storage. Además, un conjunto de datos de Azure Blob especifica el contenedor de blobs y la carpeta que contiene los datos.

Los servicios vinculados se utilizan con dos fines en Data Factory:

- Para representar un **almacén de datos** que incluye, entre otros, una base de datos de SQL Server, una base de datos de Oracle, un recurso compartido de archivos o una cuenta de Azure Blob Storage. Para obtener una lista de almacenes de datos compatibles, consulte el artículo [actividad de copia](copy-activity-overview.md).

- Para representar un **recurso de proceso** que puede hospedar la ejecución de una actividad. Por ejemplo, la actividad HDInsightHive se ejecuta en un clúster de Hadoop para HDInsight. Consulte el artículo sobre [transformación de datos](transform-data.md) para ver una lista de los entornos de proceso y las actividades de transformación admitidos.

### <a name="triggers"></a>Desencadenadores
Los desencadenadores representan una unidad de procesamiento que determina cuándo es necesario poner en marcha una ejecución de canalización. Existen diferentes tipos de desencadenadores para diferentes tipos de eventos.

### <a name="pipeline-runs"></a>Ejecuciones de la canalización
Una ejecución de la canalización es una instancia de la ejecución de la canalización. Normalmente, las instancias de ejecuciones de canalización se crean al pasar argumentos a los parámetros definidos en las canalizaciones. Los argumentos se pueden pasar manualmente o dentro de la definición del desencadenador.

### <a name="parameters"></a>Parámetros
Los parámetros son pares clave-valor de configuración de solo lectura.  Los parámetros se definen en la canalización. Los argumentos de los parámetros definidos se pasan durante la ejecución desde el contexto de ejecución creado por un desencadenador o una canalización que se ejecuta manualmente. Las actividades dentro de la canalización consumen los valores de parámetro.

Un conjunto de datos es un parámetro fuertemente tipado y una entidad reutilizable o a la que se puede hacer referencia. Una actividad puede hacer referencia a conjuntos de datos y puede consumir las propiedades definidas en la definición del conjunto de datos.

Un servicio vinculado también es un parámetro fuertemente tipado que contiene la información de conexión a un almacén de datos o a un entorno de proceso. También es una entidad reutilizable o a la que se puede hacer referencia.

### <a name="control-flow"></a>Flujo de control
El flujo de control es una orquestación de actividades de canalización que incluye el encadenamiento de actividades en una secuencia, la bifurcación, la definición de parámetros en el nivel de canalización y el paso de argumentos mientras se invoca la canalización a petición o desde un desencadenador. También incluye el paso a un estado personalizado y contenedores de bucle, es decir, los iteradores Para cada.

### <a name="variables"></a>variables
Las variables se pueden usar dentro de las canalizaciones para almacenar valores temporales y también se pueden usar junto con parámetros para habilitar el paso de valores entre canalizaciones, flujos de datos y otras actividades.

## <a name="next-steps"></a>Pasos siguientes
Estos son los documentos importantes del paso siguiente que se van a explorar:

- [Dataset and linked services](concepts-datasets-linked-services.md) (Conjuntos de datos y servicios vinculados)
- [Canalizaciones y actividades](concepts-pipelines-activities.md)
- [Integration runtime](concepts-integration-runtime.md) (Tiempo de ejecución de integración)
- [Asignación de flujos de datos](concepts-data-flow-overview.md)
- [Interfaz de usuario de Data Factory en Azure Portal](quickstart-create-data-factory-portal.md)
- [Herramienta Copiar datos de Azure Portal](quickstart-create-data-factory-copy-data-tool.md)
- [PowerShell](quickstart-create-data-factory-powershell.md)
- [.NET](quickstart-create-data-factory-dot-net.md)
- [Python](quickstart-create-data-factory-python.md)
- [REST](quickstart-create-data-factory-rest-api.md)
- [Plantilla de Azure Resource Manager](quickstart-create-data-factory-resource-manager-template.md)
 
