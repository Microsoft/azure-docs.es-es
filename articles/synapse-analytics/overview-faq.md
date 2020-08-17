---
title: 'Preguntas más frecuentes: Azure Synapse Analytics (versión preliminar de las áreas de trabajo)'
description: Preguntas más frecuentes sobre Azure Synapse Analytics (versión preliminar de las áreas de trabajo)
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: overview
ms.date: 04/15/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: ba6f79fffe5287be7574d422f026489d4da2795e
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287502"
---
# <a name="azure-synapse-analytics-workspaces-preview-frequently-asked-questions"></a>Preguntas frecuentes sobre Azure Synapse Analytics (versión preliminar de las áreas de trabajo)

En esta guía, encontrará las preguntas más frecuentes sobre Synapse Analytics.

[!INCLUDE [preview](includes/note-preview.md)]

## <a name="general"></a>General

### <a name="q-what-is-azure-synapse-analytics"></a>P: ¿Qué es Azure Synapse Analytics?

A. Azure Synapse es una plataforma de datos integrada para BI, IA e inteligencia continua. Conecta varios entornos de ejecución de análisis, como SQL y Spark, a través de una única plataforma que proporciona una forma unificada de:

- Proteger sus recursos de análisis, incluida la red, y administrar el acceso de inicio de sesión único a los artefactos de grupo, datos y desarrollo.
- Supervisar y optimizar rápidamente, reaccionar y depurar los eventos que ocurren en las actividades del área de trabajo en cualquier nivel.
- Administrar los metadatos entre los motores. Cree una tabla de Spark y estará disponible automáticamente en las bases de datos de Azure Synapse.
- Interactuar con los datos a través de una experiencia de usuario unificada. Synapse Studio ofrece macrodatos a los desarrolladores, ingenieros de datos, DBA, analistas de datos y científicos de datos en la misma plataforma.

Para más información, consulte [¿Qué es Azure Synapse Analytics?](https://docs.microsoft.com/azure/synapse-analytics/overview-what-is)

### <a name="q-what-are-the-main-components-of-azure-synapse-analytics"></a>P: ¿Cuáles son los componentes principales de Azure Synapse Analytics?

A. Azure Synapse tiene las siguientes funcionalidades:

- Funcionalidades de análisis que se ofrecen mediante el grupo de SQL o SQL a petición (versión preliminar), sin servidor
- Grupo de Apache Spark (versión preliminar) con compatibilidad completa con Scala, Python, SparkSQL y C#
- Flujo de datos que ofrece una experiencia de transformación de macrodatos sin código
- Integración de datos y orquestación para integrar los datos y poner en funcionamiento todo el desarrollo de código
- Studio para acceder a todas estas funcionalidades mediante una única interfaz de usuario web

### <a name="q-how-does-azure-synapse-analytics-relate-to-azure-sql-data-warehouse"></a>P: ¿Cómo se relaciona Azure Synapse Analytics con Azure SQL Data Warehouse?

A. Azure Synapse Analytics es una evolución de Azure SQL Data Warehouse en una plataforma de análisis, que incluye el grupo de SQL como solución de almacenamiento de datos. Esta plataforma combina la exploración, ingesta, transformación y preparación de datos, así como una capa de análisis de servicios.

## <a name="use-cases"></a>Casos de uso

### <a name="q-how-do-i-rename-a-published-artifact-dataset-notebook-sql-script-and-so-on-in-azure-synapse"></a>P: ¿Cómo se cambiar el nombre de un artefacto publicado (conjunto de datos, cuaderno, script SQL, etc.) en Azure Synapse?

A. Para cambiar el nombre de un archivo de artefacto publicado, en primer lugar clone el archivo y cambie el nombre del nuevo archivo por el que prefiera. Tendrá que actualizar manualmente todas las referencias del artefacto al nuevo nombre de archivo y eliminar el anterior.

### <a name="q-what-is-a-good-use-case-for-synapse-sql-pool"></a>P: ¿Cuál es un buen caso de uso de un grupo de Synapse SQL?

A. El grupo de SQL es el núcleo de sus necesidades de almacenamiento de datos. Es la solución de almacenamiento de datos líder en [precio y rendimiento](https://azure.microsoft.com/services/sql-data-warehouse/compare/). El grupo de SQL es la solución de almacenamiento de datos en la nube líder del sector, ya que permite:

- Prestar servicio a una gran variedad de cargas de trabajo mixtas sin afectar al rendimiento, gracias a la alta simultaneidad y al aislamiento de la carga de trabajo
- Proteger los datos fácilmente a través de características avanzadas que van desde la seguridad de red hasta el acceso específico
- Beneficiarse de una amplia gama de sistemas de ecosistema

### <a name="q-what-is-a-good-use-case-for-spark-in-synapse"></a>P: ¿Cuál es un buen caso de uso para Spark en Synapse?

A. Nuestro primer objetivo es proporcionar una excelente experiencia de ingeniería de datos para transformar datos a través del lago por lotes o en flujo. Su sólida y fácil integración con nuestra orquestación de datos hace que la operación de desarrollo sea sencilla.

Otro caso de uso de Spark es para que un científico de datos:

- extraiga una característica
- explore los datos
- entrene un modelo

### <a name="q-what-is-a-good-use-case-for-sql-on-demand-in-synapse"></a>P: ¿Cuál es un buen caso de uso para SQL a petición en Synapse?

A. SQL a petición es un servicio de consulta de los datos del lago de datos. Permite el acceso a todos los datos, ya que proporciona una sintaxis T-SQL familiar para consultar los datos en su lugar, sin necesidad de copiar o cargar datos en un almacén especializado.

A continuación se muestran algunos ejemplos de casos de uso:

- Detección y exploración básicas: proporciona a los analistas de datos, a los científicos de datos y a los ingenieros de datos una sencilla ruta de acceso a los datos que residen en su lago de datos con el esquema en consultas de T-SQL de lectura.
- Almacenamiento de datos lógico: los analistas de datos pueden ejecutar la expresividad completa del lenguaje T-SQL para consultar y analizar directamente los datos que residen en Azure Storage y usar herramientas de BI conocidas (por ejemplo, servicios de análisis de Azure, Power BI Premium, etc.) para actualizar los paneles mediante volver a ejecutar consultas de Starlight.
- ETL de "consulta única": permite a los ingenieros de datos transformar los datos de Azure Storage basados en un formato en otro, filtrar o agregar en modo de procesamiento paralelo masivo, conservar los resultados de la consulta en Azure Storage y hacer que estén inmediatamente disponibles para su posterior procesamiento en la consulta de Starlight o en otros servicios de interés.

### <a name="q-what-is-a-good-use-case-for-data-flow-in-synapse"></a>P: ¿Cuál es un buen caso de uso para el flujo de datos en Synapse?

A. El flujo de datos permite a los ingenieros de datos desarrollar una lógica de transformación de datos gráfica sin necesidad de escribir código. Los flujos de datos resultantes se ejecutan como actividades dentro de la integración de datos y la orquestación. Las actividades de flujo de datos pueden ponerse en marcha mediante las funcionalidades de programación, control, flujo y supervisión existentes.

## <a name="security-and-access"></a>Seguridad y acceso

A. La experiencia de inicio de sesión único de un extremo a otro es un proceso de autenticación importante en Synapse Analytics. La administración y el paso de la identidad mediante una integración completa de AAD son imprescindibles.

### <a name="q-how-do-i-get-access-to-files-and-folders-in-the-adls-gen2"></a>P: ¿Cómo se accede a archivos y carpetas en Azure Data Lake Storage Gen2?

A. El acceso a los archivos y las carpetas se administra actualmente mediante Azure Data Lake Storage Gen2. Para más información, consulte [Control de acceso en Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-access-control.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="q-can-i-use-third-party-business-intelligence-tools-to-access-azure-synapse-analytics"></a>P: ¿Puedo usar herramientas de inteligencia empresarial de terceros para acceder a Azure Synapse Analytics?

A. Sí, puede usar aplicaciones empresariales de terceros, como Tableau y Power BI, para conectarse al grupo de SQL y a SQL a petición. Spark admite IntelliJ.

## <a name="next-steps"></a>Pasos siguientes

- [Creación de un área de trabajo](quickstart-create-workspace.md)
- [Inicio rápido: Uso de Synapse Studio (versión preliminar)](quickstart-synapse-studio.md)
- [Creación de un grupo de SQL](quickstart-create-sql-pool-portal.md)
- [Uso de SQL a petición](quickstart-sql-on-demand.md)
- [Creación de un grupo de Apache Spark](quickstart-create-apache-spark-pool-portal.md) 
