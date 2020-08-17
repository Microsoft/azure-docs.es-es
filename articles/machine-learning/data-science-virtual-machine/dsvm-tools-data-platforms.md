---
title: Plataformas de datos compatibles
titleSuffix: Azure Data Science Virtual Machine
description: Obtenga información sobre las herramientas y las plataformas de datos admitidas en Azure Data Science Virtual Machine.
keywords: herramientas de ciencia de datos, máquina virtual de ciencia de datos, herramientas para la ciencia de datos, ciencia de datos de linux
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 12/12/2019
ms.openlocfilehash: 3e1cfa13a3fd568ab2f1253c015880c57f99ce7e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87012492"
---
# <a name="data-platforms-supported-on-the-data-science-virtual-machine"></a>Plataformas de datos admitidas en Data Science Virtual Machine

Con una máquina virtual Data Science Virtual Machine (DSVM), puede crear los análisis en una amplia variedad de plataformas de datos. Además de interfaces para plataformas de datos remotos, DSVM proporciona una instancia local para el desarrollo rápido y la creación de prototipos.

Las siguientes son las herramientas de la plataforma de datos compatibles con DSVM.

## <a name="sql-server-developer-edition"></a>SQL Server Developer Edition

| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   | Una instancia de base de datos relacional local      |
| Ediciones de DSVM admitidas      | Windows 2016: SQL Server 2017, Windows 2019: SQL Server 2019      |
| Usos típicos      | <ul><li>Desarrollo rápido de forma local con un conjunto de datos más pequeño</li><li>Ejecución de R en base de datos</li></ul> |
| Vínculos a ejemplos      | <ul><li>Una pequeña muestra del conjunto de datos de la ciudad de Nueva York se carga en la instancia de SQL Database:<br/>  `nyctaxi`</li><li>Puede encontrar un ejemplo de Jupyter que muestra Microsoft Machine Learning Server y el análisis en bases de datos en:<br/> `~notebooks/SQL_R_Services_End_to_End_Tutorial.ipynb`</li></ul> |
| Herramientas relacionadas en DSVM       | <ul><li>SQL Server Management Studio</li><li>Controladores ODBC/JDBC</li><li>pyodbc, RODBC</li><li>Apache Drill</li></ul> |

> [!NOTE]
> SQL Server Developer Edition solo puede utilizarse para fines de desarrollo y prueba. Se necesita una licencia o una de las máquinas virtuales de SQL Server para ejecutarlo en producción.


### <a name="setup"></a>Configurar

El servidor de bases de datos ya está preconfigurado y los servicios de Windows relacionados con SQL Server (como `SQL Server (MSSQLSERVER)`) están configurados para ejecutarse automáticamente. El único paso manual implica habilitar el análisis en la base de datos mediante Microsoft Machine Learning Server. Puede habilitar el análisis ejecutando el siguiente comando como una acción única en SQL Server Management Studio (SSMS). Ejecute este comando después de iniciar sesión como administrador del equipo, abra una nueva consulta en SSMS y asegúrese de que la base de datos seleccionada sea `master`:

```sql
CREATE LOGIN [%COMPUTERNAME%\SQLRUserGroup] FROM WINDOWS 
```

(Reemplace %COMPUTERNAME% por el nombre de la máquina virtual).

Para ejecutar SQL Server Management Studio, puede buscar "SQL Server Management Studio" en la lista de programas o usar la búsqueda de Windows para buscarlo y ejecutarlo. Cuando se le piden credenciales, seleccione **Autenticación de Windows** y use el nombre del equipo o ```localhost``` en el campo de **nombre de SQL Server**.

### <a name="how-to-use-and-run-it"></a>Cómo usarla y ejecutarla

De manera predeterminada, el servidor de bases de datos con la instancia de base de datos predeterminada se ejecuta automáticamente. Puede usar herramientas como SQL Server Management Studio en la máquina virtual para tener acceso a la base de datos de SQL Server de forma local. Las cuentas locales de administradores tienen acceso de administrador en la base de datos.

Además, DSVM incluye controladores ODBC y JDBC para comunicarse con SQL Server, bases de datos de Azure SQL y Azure SQL Data Warehouse desde aplicaciones escritas en varios lenguajes, como Python y Machine Learning Server.

### <a name="how-is-it-configured-and-installed-on-the-dsvm"></a>¿Cómo se configura e instala en DSVM? 

 El servidor SQL Server está instalado de manera estándar. Se puede encontrar en `C:\Program Files\Microsoft SQL Server`. La instancia de Machine Learning Server en la base de datos se encuentra en `C:\Program Files\Microsoft SQL Server\MSSQL13.MSSQLSERVER\R_SERVICES`. La máquina DSVM también tiene una instancia independiente de Machine Learning Server, que se instala en `C:\Program Files\Microsoft\R Server\R_SERVER`. Estas dos instancias de Machine Learning Server no comparten bibliotecas.


## <a name="apache-spark-2x-standalone"></a>Apache Spark 2.x (independiente)

| Category | Value |
| ------------- | ------------- |
| ¿Qué es?   | Una instancia independiente (nodo único In-Process) de la popular plataforma Apache Spark, un sistema de procesamiento de datos y aprendizaje automático rápido y a gran escala.     |
| Ediciones de DSVM admitidas      | Linux     |
| Usos típicos      | <ul><li>Desarrollo rápido de aplicaciones de Spark/PySpark localmente con un conjunto de datos más pequeño y más adelante implementación en clústeres Spark grandes, como Azure HDInsight</li><li>Prueba del contexto de Spark en Microsoft Machine Learning Server</li><li>Uso de la biblioteca de código abierto [MMLSpark](https://github.com/Azure/mmlspark) de Microsoft o SparkML para crear aplicaciones de aprendizaje automático</li></ul> |
| Vínculos a ejemplos      |    Ejemplo de Jupyter:<ul><li>~/notebooks/SparkML/pySpark</li><li>~/notebooks/MMLSpark</li></ul><p>Microsoft Machine Learning Server (contexto de Spark): /dsvm/samples/MRS/MRSSparkContextSample.R</p> |
| Herramientas relacionadas en DSVM       | <ul><li>PySpark, Scala</li><li>Jupyter (Kernels Spark/PySpark)</li><li>Microsoft Machine Learning Server, SparkR, Sparklyr</li><li>Apache Drill</li></ul> |

### <a name="how-to-use-it"></a>Cómo usarlo
Puede enviar trabajos de Spark en la línea de comandos ejecutando el comando `spark-submit` o `pyspark`. También puede crear una instancia de Jupyter Notebook creando un nuevo bloc de notas con el kernel de Spark.

Puede usar Spark desde R mediante el uso de bibliotecas como SparkR, Sparklyr y Microsoft Machine Learning Server, que están disponibles en DSVM. Vea punteros a los ejemplos en la tabla anterior.

### <a name="setup"></a>Configurar
Antes de ejecutar en el contexto de Spark en Microsoft Machine Learning Server en la edición de Ubuntu Linux DSVM, tiene que llevar a cabo un paso de configuración una sola vez con el fin de habilitar un HDFS de Hadoop de un solo nodo y una instancia de Yarn. De manera predeterminada, los servicios de Hadoop están instalados pero deshabilitados en la DSVM. Para habilitarlos, ejecute los comandos siguientes como raíz la primera vez:

```bash
echo -e 'y\n' | ssh-keygen -t rsa -P '' -f ~hadoop/.ssh/id_rsa
cat ~hadoop/.ssh/id_rsa.pub >> ~hadoop/.ssh/authorized_keys
chmod 0600 ~hadoop/.ssh/authorized_keys
chown hadoop:hadoop ~hadoop/.ssh/id_rsa
chown hadoop:hadoop ~hadoop/.ssh/id_rsa.pub
chown hadoop:hadoop ~hadoop/.ssh/authorized_keys
systemctl start hadoop-namenode hadoop-datanode hadoop-yarn
```

Puede detener los servicios relacionados con Hadoop cuando no los necesite si ejecuta ```systemctl stop hadoop-namenode hadoop-datanode hadoop-yarn```.

En el directorio `/dsvm/samples/MRS` se proporciona un ejemplo en el que se muestra cómo desarrollar y probar MRS en el contexto de Spark remoto (que es la instancia independiente de Spark en DSVM).


### <a name="how-is-it-configured-and-installed-on-the-dsvm"></a>¿Cómo se configura e instala en DSVM? 
|Plataforma|Ubicación de instalación ($SPARK_HOME)|
|:--------|:--------|
|Linux   | /dsvm/tools/spark-X.X.X-bin-hadoopX.X|


Las bibliotecas para acceder a datos de Azure Blob Storage o Azure Data Lake Storage, utilizando las bibliotecas de aprendizaje automático de MMLSpark de Microsoft, están preinstaladas en $SPARK_HOME/jars. Estos archivos JAR se carga automáticamente cuando se inicia Spark. De forma predeterminada, Spark utiliza datos en el disco local. 

Para que la instancia de Spark en DSVM acceda a los datos almacenados en Blob Storage o Azure Data Lake Storage, debe crear y configurar el archivo `core-site.xml` basado en la plantilla que se encuentra en $SPARK_HOME/conf/core-site.xml.template. También debe tener las credenciales adecuadas para tener acceso a Blob Storage y Azure Data Lake Storage. (Tenga en cuenta que los archivos de plantilla usan marcadores de posición para las configuraciones de Blob Storage y Azure Data Lake Storage).

Para obtener información más detallada sobre cómo crear credenciales para el servicio Azure Data Lake Storage, consulte el artículo sobre [la autenticación entre servicios con Azure Data Lake Storage Gen1](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-authenticate-using-active-directory). Una vez que las credenciales para Blob Storage o Azure Data Lake Storage se escriban en el archivo core-site.xml, puede hacer referencia a los datos almacenados en esos orígenes con el prefijo URI de wasb:// o adl://.

