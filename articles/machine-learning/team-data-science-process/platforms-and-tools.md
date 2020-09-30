---
title: 'Plataformas y herramientas para proyectos de ciencia de datos: proceso de ciencia de datos en equipo'
description: Detalla y describe los recursos de datos y análisis disponibles para las empresas y que están normalizados en el proceso de ciencia de datos en equipo.
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 37faf5d8e6292844245f475d9fcf0f832a0cca10
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89440004"
---
# <a name="platforms-and-tools-for-data-science-projects"></a>Plataformas y herramientas para proyectos de ciencia de datos

Microsoft proporciona un amplio espectro de recursos de análisis para plataformas locales y en la nube. Se pueden implementar para que la ejecución de los proyectos de ciencia de datos sean eficaces y escalables. El [proceso de ciencia de datos en equipo](overview.md) (TDSP) sirve de guía a los equipos que implementan proyectos de ciencia de datos con seguimiento, con control de versiones y en colaboración.  En [Roles y tareas del proceso de ciencia de datos en equipo](roles-tasks.md) dispone de un esquema de los roles profesionales y las tareas asociadas de las que se ocupa un equipo de ciencia de datos que sigue este proceso como estándar.

Los recursos de análisis a disposición de los equipos de ciencia de datos que siguen el proceso de ciencia de datos en equipo incluyen:

- Data Science Virtual Machine (tanto Windows como Linux CentOS)
- Clústeres de HDInsight Spark
- Synapse Analytics
- Azure Data Lake
- Clústeres de HDInsight Hive
- Azure File Storage
- Servicios R y Python para SQL Server 2019
- Azure Databricks

En este documento, se describen brevemente los recursos y se proporcionan vínculos a tutoriales que los equipos del proceso de ciencia de datos en equipo han publicado. Aprenderá a usarlos paso a paso y podrá comenzar a usarlos para compilar las aplicaciones inteligentes. Podrá encontrar más información sobre estos recursos en las páginas de producto. 

## <a name="data-science-virtual-machine-dsvm"></a>Data Science Virtual Machine (DSVM)

La máquina virtual de ciencia de datos que Microsoft ofrece tanto con Windows como con Linux contiene herramientas conocidas para actividades de desarrollo y modelado de ciencia de datos. Incluye herramientas tales como:

- Microsoft R Server Developer Edition 
- Anaconda Python Distribution
- Jupyter Notebook para Python y R 
- Visual Studio Community Edition con Python y Herramientas de R en Windows/Eclipse en Linux
- Power BI Desktop para Windows
- SQL Server 2016 Developer Edition en Windows/Postgres en Linux

También incluye **herramientas de ML e IA**, como xgboost, mxnet y Vowpal Wabbit.

Actualmente, la máquina virtual de ciencia de datos está disponible con los sistemas operativos **Windows** y **CentOS Linux**. Elija el tamaño de la máquina virtual de ciencia de datos (número de núcleos de CPU y cantidad de memoria) en función de los proyectos de ciencia de datos que se vayan a ejecutar en ella. 

Para más información sobre la edición de Windows de DSVM, consulte [Microsoft Data Science Virtual Machine](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.dsvm-win-2019) en Azure Marketplace. Para la edición Linux de la máquina virtual de ciencia de datos, consulte [Linux Data Science Virtual Machine](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.ubuntu-1804).

Para aprender a ejecutar de forma eficaz algunas de las tareas comunes de ciencia de datos en DSVM, consulte [Diez cosas que puede hacer en Windows Data Science Virtual Machine](../data-science-virtual-machine/vm-do-ten-things.md).


## <a name="azure-hdinsight-spark-clusters"></a>Clústeres de Azure HDInsight Spark

Apache Spark es una plataforma de procesamiento paralelo de código abierto que admite el procesamiento en memoria para mejorar el rendimiento de las aplicaciones analíticas de Big Data. El motor de procesamiento Spark se ha creado para ofrecer velocidad, facilidad de uso y análisis sofisticados. Las funcionalidades de cálculo en memoria de Spark lo convierten en una buena opción para algoritmos iterativos en los cálculos de grafos y aprendizaje automático. Spark también es compatible con Azure Storage Blob (WASB), por lo que se pueden procesar los datos existentes almacenados en Azure fácilmente mediante Spark.

Cuando crea un clúster Spark en HDInsight, aprovisiona recursos de proceso de Azure con Spark instalado y configurado. Se tardan unos diez minutos en crear un clúster Spark en HDInsight. Almacene los datos que se van procesar en Azure Blob Storage. Para más información sobre cómo usar Azure Blob Storage con HDInsight, consulte [Uso de Azure Storage con clústeres de Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-blob-storage.md).

El equipo de proceso de ciencia de datos en equipo de Microsoft ha publicado dos tutoriales completos sobre el uso de clústeres de Azure HDInsight Spark para crear soluciones de ciencia de datos, en Python y en Scala. Para más información sobre **clústeres de Azure HDInsight Spark**, consulte [Introducción a Apache Spark en HDInsight para Linux](../../hdinsight/spark/apache-spark-overview.md). Para obtener información sobre cómo crear una solución de ciencia de datos mediante **Python** en un clúster de Azure HDInsight Spark, consulte [Información general sobre la ciencia de datos con Azure HDInsight Spark](spark-overview.md). Para obtener información sobre cómo crear una solución de ciencia de datos mediante **Scala** en un clúster de Azure HDInsight Spark, consulte [Ciencia de datos mediante Scala y Spark en Azure](scala-walkthrough.md). 


##  <a name="azure-synapse-analytics"></a>Azure Synapse Analytics

Azure Synapse Analytics le permite escalar los recursos de proceso fácilmente y en segundos, sin tener que realizar un aprovisionamiento excesivo o pagar en exceso. Además, ofrece la exclusiva ventaja de pausar el uso de los recursos de proceso, lo que da total libertad para administrar mejor los costos de la nube. La posibilidad de implementar recursos de proceso escalables permite almacenar todos los datos en Azure Synapse Analytics. Los costos de almacenamiento son mínimos y puede ejecutar el proceso solo en aquellas partes de los conjuntos de datos que desea analizar. 

Para obtener más información sobre Azure Synapse Analytics, consulte el sitio web de [Azure Synapse Analytics](https://azure.microsoft.com/services/sql-data-warehouse). Para obtener información sobre cómo crear soluciones completas de análisis avanzado con Azure Synapse Analytics, consulte [Proceso de ciencia de datos en equipo en acción: uso de Azure Synapse Analytics](sqldw-walkthrough.md).


## <a name="azure-data-lake"></a>Azure Data Lake

Azure Data Lake es un repositorio empresarial de todos los tipos de datos recopilados en una única ubicación antes de la aplicación de requisitos o esquemas formales. Esta flexibilidad permite conservar cualquier tipo de datos en Data Lake, con independencia del tamaño, la estructura o la velocidad de ingestión. Las organizaciones pueden usar después Hadoop o análisis avanzado para buscar patrones en estos Data Lake. Data Lake también sirve de repositorio para una preparación de los datos más económica antes de su mantenimiento y transferencia a un almacenamiento de datos.

Para más información sobre Azure Data Lake, consulte la entrada de blob [Introducing Azure Data Lake](https://azure.microsoft.com/blog/introducing-azure-data-lake/) (Introducción a Azure Data Lake). Para obtener información sobre cómo crear una solución de ciencia de datos completa y escalable con Azure Data Lake, consulte [Ciencia de datos escalables con Azure Data Lake: tutorial completo](data-lake-walkthrough.md)


## <a name="azure-hdinsight-hive-hadoop-clusters"></a>Clústeres de Azure HDInsight Hive (Hadoop)

Apache Hive es un sistema de almacén de datos para Hadoop, que permite realizar resúmenes de datos, consultas y análisis de datos mediante HiveQL (una lenguaje de consultas similar a SQL). Hive se puede usar para explorar los datos de forma interactiva o para crear trabajos de procesamiento por lotes reutilizables.

Hive le permite proyectar la estructura del proyecto en datos que en gran medida no están estructurados. Después de definir la estructura, puede usar Hive para consultar esos datos en un clúster Hadoop sin tener que usar, incluso sin conocer, Java o MapReduce. HiveQL (el lenguaje de consultas de Hive) permite escribir consultas con instrucciones similares a T-SQL.

Para los científicos de datos, Hive puede ejecutar funciones de Python definidas por el usuario en consultas de Hive para procesar registros. Esta capacidad amplía considerablemente la funcionalidad de consultas de Hive en el análisis de datos. Específicamente, permite a los científicos de datos realizar ingeniería de características escalables en los lenguajes con los que están más familiarizados, como Python y HiveQA de tipo SQL. 

Para más información sobre los clústeres de Azure HDInsight Hive, consulte [Uso de Hive y HiveQL con Hadoop en HDInsight](../../hdinsight/hadoop/hdinsight-use-hive.md). Para aprender a crear una solución de ciencia de datos completa y escalable en clústeres de Hive de Azure HDInsight, consulte el artículo sobre [el proceso de ciencia de datos en equipo en acción: uso de clústeres de Hadoop de HDInsight](hive-walkthrough.md).


## <a name="azure-file-storage"></a>Azure File Storage 

Azure File Storage es un servicio que ofrece recursos compartidos de archivos en la nube mediante el protocolo Bloque de mensajes del servidor (SMB) estándar. Se admiten SMB 2.1 y SMB 3.0. Con Almacenamiento de archivos de Azure puede migrar aplicaciones heredadas basadas en recursos compartidos de archivos a Azure con rapidez y sin necesidad de costosas reescrituras. Las aplicaciones que se ejecutan en máquinas virtuales de Azure o en servicios en la nube o desde clientes locales pueden montar un recurso compartido de archivos en la nube, igual que una aplicación de escritorio monta un recurso compartido SMB típico. Cualquier número de componentes de aplicación puede montar y acceder simultáneamente al recurso compartido de Almacenamiento de archivos.

Especialmente útil para los proyectos de ciencia de datos es la capacidad para crear un almacén de archivos de Azure como el lugar para compartir datos del proyecto con los miembros del equipo de proyecto. Cada uno de ellos tiene acceso entonces a la misma copia de los datos en el almacenamiento de archivos de Azure. También puede utilizar este almacenamiento de archivos para compartir conjuntos de características generados durante la ejecución del proyecto. Si el proyecto es una interacción con los clientes, estos pueden crear un almacenamiento de archivos de Azure en su propia suscripción de Azure para compartir los datos de proyecto y las características con el usuario. De esta manera, el cliente tiene control total sobre los recursos de datos del proyecto. Para más información sobre Azure File Storage, consulte [Introducción a Azure File Storage en Windows](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-files) o [Uso de Azure File Storage con Linux](../../storage/files/storage-how-to-use-files-linux.md).


## <a name="sql-server-2019-r-and-python-services"></a>Servicios R y Python para SQL Server 2019

R Services (En base de datos) proporciona una plataforma para desarrollar e implementar aplicaciones inteligentes que pueden destapar nueva información. Puede usar el completo y eficaz lenguaje R, incluidos los numerosos paquetes proporcionados por la comunidad R, para crear modelos y generar predicciones con sus datos de SQL Server. Dado que R Services (En base de datos) integra el lenguaje R con SQL Server, los análisis se mantienen cerca de los datos, lo que elimina los costos y los riesgos de seguridad asociados al movimiento de datos.

R Services (En base de datos) admite el lenguaje R de código abierto con un completo conjunto de tecnologías y herramientas de SQL Server. Ofrece un rendimiento superior, seguridad, confiabilidad y facilidad de uso. Puede implementar soluciones en R con herramientas conocidas y convenientes. Las aplicaciones de producción pueden llamar al entorno en tiempo de ejecución de R y recuperar las predicciones y los objetos visuales mediante Transact-SQL. También puede usar las bibliotecas de ScaleR para mejorar la escalabilidad y el rendimiento de las soluciones de R. Para más información, consulte [SQL Server R Services](https://docs.microsoft.com/sql/advanced-analytics/r/sql-server-r-services).

El equipo del proceso de ciencia de datos en equipo de Microsoft ha publicado dos tutoriales completos que muestran cómo crear soluciones de ciencia de datos en SQL Server 2016 R Services, uno para los programadores de R y otro para desarrolladores de SQL. Para los **programadores de R**, consulte el [tutorial completo de ciencia de datos](https://docs.microsoft.com/sql/advanced-analytics/tutorials/walkthrough-data-science-end-to-end-walkthrough). Para los **desarrolladores de SQL**, consulte [Análisis de R en bases de datos para desarrolladores de SQL (tutorial)](https://docs.microsoft.com/sql/advanced-analytics/tutorials/sqldev-in-database-r-for-sql-developers).


## <a name="appendix-tools-to-set-up-data-science-projects"></a><a name="appendix"></a>Apéndice: Herramientas para configurar los proyectos de ciencia de datos

### <a name="install-git-credential-manager-on-windows"></a>Instalación del administrador de credenciales de Git en Windows

Si está siguiendo el proceso de ciencia de datos en equipo en **Windows**, debe instalar la **el Administrador de credenciales (GCM), Git** para comunicarse con los repositorios de Git. Para instalar GCM, primero debe instalar **Chocolaty**. Para instalar Chocolaty y GCM, ejecute los siguientes comandos en Windows PowerShell como **administrador**:  

```powershell
iwr https://chocolatey.org/install.ps1 -UseBasicParsing | iex
choco install git-credential-manager-for-windows -y
```  

### <a name="install-git-on-linux-centos-machines"></a>Instalación de Git en máquinas Linux (CentOS)

Ejecute el siguiente comando de bash para instalar Git en máquinas Linux (CentOS):

```powershell
sudo yum install git
```

### <a name="generate-public-ssh-key-on-linux-centos-machines"></a>Generación de clave SSH pública en máquinas Linux (CentOS)

Si se usan máquinas Linux (CentOS) para ejecutar los comandos de GIT, tendrá que agregar la clave SSH pública de su máquina a Azure DevOps Services, a fin de que Azure DevOps Services reconozca esta máquina. En primer lugar, debe generar una clave SSH pública y agregar la clave a las claves públicas de SSH en la página de configuración de seguridad de Azure DevOps Services. 

1. Para generar la clave SSH, ejecute los dos comandos siguientes: 

   ```
   ssh-keygen
   cat .ssh/id_rsa.pub
   ```
   
   ![Comandos para generar la clave SSH](./media/platforms-and-tools/resources-1-generate_ssh.png)

1. Copie la clave ssh completa, incluido *ssh-rsa*. 
1. Inicie sesión en Azure DevOps Services. 
1. Haga clic en **<Su nombre\>** en la esquina superior derecha de la página y luego en **Seguridad**. 
    
   ![Haga clic en el nombre y luego haga clic en Seguridad.](./media/platforms-and-tools/resources-2-user-setting.png)

1. Haga clic en **Claves públicas de SSH** y haga clic en **+Agregar**. 

   ![Haga clic en Claves públicas de SSH y, luego, haga clic en +Agregar.](./media/platforms-and-tools/resources-3-add-ssh.png)

1. Pegue la clave SSH copiada en el cuadro de texto y haga clic en Guardar.


## <a name="next-steps"></a>Pasos siguientes

También se proporcionan tutoriales completos que muestran todos los pasos del proceso en **escenarios concretos** . Se enumeran y enlazan con descripciones en miniatura en los [tutoriales de ejemplos](walkthroughs.md). En ellos se ilustra cómo combinar herramientas y servicios locales y en la nube en un flujo de trabajo o canalización para crear una aplicación inteligente. 

Para ver ejemplos que muestran cómo ejecutar los pasos en el proceso de ciencia de datos en equipo mediante Azure Machine Learning Studio (clásico), consulte la ruta de aprendizaje [con Azure ML](https://docs.microsoft.com/azure/machine-learning/team-data-science-process/).
