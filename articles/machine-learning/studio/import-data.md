---
title: 'ML Studio (clásico): Importación de datos de entrenamiento: Azure'
description: Cómo importar los datos en Azure Machine Learning Studio (clásico) desde varios orígenes de datos. Obtenga información sobre qué tipos de datos y formatos de datos son compatibles.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: previous-author=heatherbshapiro, previous-ms.author=hshapiro
ms.date: 02/01/2019
ms.openlocfilehash: c13c394f518482853762a3a5bf9a1a2a42ca658a
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432411"
---
# <a name="import-your-training-data-into-azure-machine-learning-studio-classic-from-various-data-sources"></a>Importación de datos de entrenamiento en Azure Machine Learning Studio (clásico) desde varios orígenes de datos

**SE APLICA A:**  ![sí](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


Para usar sus propios datos en Machine Learning Studio (clásico) para desarrollar y entrenar una solución de análisis predictivo, puede usar los datos de: 

* Un **archivo local**: cargue los datos locales de antemano desde la unidad de disco duro para crear un módulo de conjunto de datos en su área de trabajo
* **Orígenes de datos en línea**: use el módulo [Importar datos][import-data] para acceder a los datos de varios orígenes en línea mientras se ejecuta su experimento
* **Experimento de Machine Learning Studio (clásico)** : use los datos que se guardaron como un conjunto de datos en Machine Learning Studio (clásico)
* [**Base de datos de SQL Server**](use-data-from-an-on-premises-sql-server.md): use los datos de una base de datos de SQL Server sin tener que copiar manualmente los datos

> [!NOTE]
> Existe una gran variedad de conjuntos de datos de ejemplo disponibles en Machine Learning Studio (clásico) que puede usar como datos de aprendizaje. Para información al respecto, consulte [Uso de los conjuntos de datos de ejemplo en Azure Machine Learning Studio (clásico)](use-sample-datasets.md).

## <a name="prepare-data"></a>Preparación de los datos

Machine Learning Studio (clásico) está diseñado para trabajar con datos rectangulares o tabulares, como datos de texto delimitados o datos estructurados de una base de datos, aunque en algunas circunstancias, es posible usar datos no rectangulares.

Es mejor si los datos están relativamente limpios antes de importarlos a Studio (clásico). Por ejemplo, conviene tener cuidado de cuestiones como cadenas sin comillas.

Sin embargo, hay módulos disponibles en Studio (clásico) que le permitirán manipular levemente los datos en el experimento después de importar los datos. Dependiendo de los algoritmos de aprendizaje automático que usará, es posible que deba decidir cómo controlar los problemas estructurales de los datos, como valores que faltan y datos esparcidos y existen módulos que pueden ayudar en esto. Observe la sección **Transformación de datos** de la paleta de módulos para los módulos que realizan estas funciones.

En cualquier momento del experimento, puede ver o descargar los datos que genera un módulo con un clic en el puerto de salida. Dependiendo del módulo, es posible que haya distintas opciones de descarga disponibles, o bien que se puedan visualizar los datos dentro del explorador web en Studio (clásico).

## <a name="supported-data-formats-and-data-types"></a>Formatos y tipos de datos admitidos

Puede importar diversos tipos de datos al experimento, dependiendo del mecanismo que usa para importar los datos y de dónde provienen estos:

* Texto sin formato (.txt)
* Valores separados por coma (CSV) con un encabezado (.csv) o sin encabezado (.nh.csv)
* Valores separados con tabulaciones (TSV) con un encabezado (.tsv) o sin encabezado (.nh.tsv)
* Archivo de Excel
* Tabla de Azure
* Tabla de Hive
* Tabla de Base de datos SQL
* Valores de OData
* Datos SVMLight (.svmlight) (consulte la [definición de SVMLight](http://svmlight.joachims.org/) para obtener más información sobre el formato)
* Datos de formato de archivo con relación de atributo (ARFF) (.arff) (consulte la [definición de ARFF](https://weka.wikispaces.com/ARFF) si desea ver información sobre el formato)
* Archivo ZIP (.zip)
* Archivo de área de trabajo u objeto de R (.RData)

Si importa datos en un formato como ARFF que incluyan metadatos, Studio (clásico) usa estos metadatos para definir el tipo de datos y encabezado de cada columna.

Si importa datos en formato TSV o CSV que no incluyan estos metadatos, Studio (clásico) infiere el tipo de datos de cada columna tomando una muestra de estos. Si los datos tampoco tienen encabezados de columna, Studio (clásico) proporciona nombres predeterminados.

Puede especificar o cambiar explícitamente los encabezados y los tipos de datos de las columnas usando el módulo [Editar metadatos][edit-metadata].

Studio (clásico) reconoce los siguientes tipos de datos:

* String
* Entero
* Double
* Boolean
* DateTime
* TimeSpan

Studio usa un tipo de datos interno llamado ***tabla de datos*** para pasar datos entre los módulos. Puede convertir explícitamente sus datos en formato de tabla de datos con el módulo [Convertir al conjunto de datos][convert-to-dataset].

Todo módulo que acepta formatos distintos de tabla de datos convertirá los datos a tabla de datos de manera silenciosa antes de pasarlos al módulo siguiente.

En caso de ser necesario, puede convertir el formato tabla de datos de vuelta al formato CSV, TSV, ARFF o SVMLight mediante otros módulos de conversión.
Consulte la sección **Conversiones de formatos de datos** de la paleta de módulos para ver los módulos que realizan estas funciones.

## <a name="data-capacities"></a>Capacidades de datos

Los módulos en Machine Learning Studio (clásico) admiten conjuntos de datos de hasta 10 GB de datos numéricos densos para casos de uso comunes. Si un módulo ocupa más de una entrada, el valor de 10 GB es el total de todos los tamaños de entrada. Puede realizar el muestreo de conjuntos de datos grandes mediante consultas de Hive o Azure SQL Database, o bien usar el procesamiento previo de aprendizaje por recuentos antes de la importación de los datos.  

Los siguientes tipos de datos se pueden expandir en conjuntos de datos grandes durante la normalización de características y están limitados a menos de 10 GB:

* Dispersos
* Categorías
* Cadenas
* Datos binarios

Los siguientes módulos están limitados a conjuntos de datos de menos de 10 GB:

* Módulos de recomendación.
* Módulo Synthetic Minority Oversampling Technique (SMOTE)
* Módulos de script: R, Python y SQL
* Módulos donde el tamaño de los datos de salida puede ser mayor que el tamaño de los datos de entrada, como en la aplicación de hash de característica o unión.
* Validación cruzada, ajuste de los hiperparámetros del modelo, regresión ordinal y multiclase uno contra todos, cuando el número de iteraciones sea muy grande.

En el caso de conjuntos de datos que tengan más de 2 GB, cargue los datos en Azure Storage o Azure SQL Database, o use Azure HDInsight, en lugar de cargarlos directamente desde el archivo local.

Puede encontrar información acerca de los datos de las imágenes en la referencia del módulo [Importar imágenes](https://docs.microsoft.com/azure/machine-learning/studio-module-reference/import-images#bkmk_Notes).

## <a name="import-from-a-local-file"></a>Importación desde un archivo local

Puede cargar un archivo de datos desde la unidad de disco duro para usar como datos de entrenamiento en Studio (clásico). Al importar un archivo de datos, crea un módulo de conjunto de datos listo para usar en los experimentos del área de trabajo.

Para importar datos desde una unidad de disco duro local, siga estos pasos:

1. Haga clic en **+NUEVO** en la parte inferior de la ventana de Studio (clásico).
2. Seleccione **CONJUNTO DE DATOS** y **DESDE ARCHIVO LOCAL**.
3. En el cuadro de diálogo **Cargar un nuevo conjunto de datos**, vaya al archivo que quiere cargar.
4. Escriba un nombre, identifique el tipo de datos y, si lo desea, escriba una descripción. Se recomienda incluir una descripción: le permite registrar cualquier característica acerca de los datos que quiera recordar cuando use los datos en el futuro.
5. La casilla **Esta es la versión nueva de un conjunto de datos existente** le permite actualizar una base de datos existente con datos nuevos. Para ello, haga clic en esta casilla y, luego, escriba el nombre de un conjunto de datos existente.

![Carga de un conjunto de datos nuevo](./media/import-data/upload-dataset-from-local-file.png)

El tiempo de la carga dependerá del tamaño de los datos y de la velocidad de conexión con el servicio. Si sabe que el archivo tardará mucho tiempo en cargarse, puede realizar otras operaciones en Studio(clásico) mientras espera. Sin embargo, si cierra el explorador antes de que se complete la carga de datos, la carga genera un error.

Una vez que los datos estén cargados, se almacenan en un módulo de conjunto de datos y se encontrarán disponibles para cualquier experimento que se realice en su área de trabajo.

Cuando edita un experimento, puede encontrar los conjuntos de datos que ha cargado en la lista **My Datasets** (Mis conjuntos de datos) que aparece en la lista **Saved Datasets** (Conjuntos de datos guardados) en la paleta de módulos. Puede arrastrar y colocar el conjunto de datos en el lienzo de experimento cuando desee usarlo para profundizar en su análisis o para Machine Learning.

## <a name="import-from-online-data-sources"></a>Importación desde orígenes de datos en línea

Con el módulo[Importar datos][import-data], el experimento puede importar datos desde varios orígenes de datos en línea mientras se está ejecutando el experimento.

> [!NOTE]
> En este artículo se proporciona información general sobre el módulo [Importar datos][import-data]. Para más información sobre los tipos de datos a los que puede acceder, los formatos, los parámetros y las respuestas a las preguntas más comunes, consulte el tema de referencia del módulo [Importar datos][import-data].

Al usar el módulo [Importar datos][import-data], puede acceder a los datos desde cualquiera de los orígenes de datos en línea mientras se está ejecutando el experimento:

* Una dirección URL web con HTTP
* Hadoop con HiveQL
* Almacenamiento de blobs de Azure
* Tabla de Azure
* Azure SQL Database. SQL Managed Instance o SQL Server
* Un proveedor de fuentes de datos, actualmente OData
* Azure Cosmos DB

Dado que se tiene acceso a estos datos de entrenamiento mientras se está ejecutando el experimento, solo están disponibles en ese experimento. En comparación, los datos que estén almacenados en módulos de conjunto de datos se encontrarán disponibles para cualquier experimento que se haga en su área de trabajo.

Para acceder a orígenes de datos en línea en el experimento de Studio (clásico), agregue el módulo [Importar datos][import-data] al experimento. Luego, seleccione **Launch Import Data Wizard** (Iniciar el asistente para importar datos) en **Propiedades** para obtener instrucciones paso a paso para seleccionar y configurar el origen de datos. Como alternativa, puede seleccionar manualmente **Origen de datos** en **Propiedades** y proporcionar los parámetros necesarios para acceder a los datos.

Los orígenes de datos en línea admitidos se muestran en la tabla siguiente. Esta tabla también resume los formatos de archivo admitidos y los parámetros usados para tener acceso a los datos.

> [!IMPORTANT]
> En estos momentos, los módulos [Importar datos][import-data] y [Exportar datos][export-data] solo pueden leer y escribir datos de la instancia de Azure Storage que se creó utilizando el modelo de implementación clásica. Es decir, todavía no se admite el nuevo tipo de cuenta de Azure Blob Storage que ofrece un nivel de acceso frecuente o esporádico al almacenamiento.
>
> Por lo general, las cuentas de almacenamiento de Azure que podría haber creado antes de que esta opción de servicio estuviera disponible no deberían verse afectadas.
> Si tiene que crear una cuenta nueva, seleccione **Clásico** en Modelo de implementación, o bien utilice Resource Manager y seleccione **Uso general** en lugar de **Blob Storage** en **Tipo de cuenta**.
>
> Para más información, consulte [Azure Blob Storage: Capas de almacenamiento de acceso frecuente y esporádico](../../storage/blobs/storage-blob-storage-tiers.md).

### <a name="supported-online-data-sources"></a>Orígenes de datos en línea admitidos
El módulo **Importar datos** de Azure Machine Learning Studio (clásico) admite los siguientes orígenes de datos:

| Origen de datos | Descripción | Parámetros |
| --- | --- | --- |
| Dirección URL web a través de HTTP |Lee datos en valores separados por comas (CSV), valores separados por tabulaciones (TSV), formato de archivo de atributo-relación (ARFF) y formatos de máquinas de vectores de soporte (SVM-light), desde cualquier dirección URL web que use HTTP |<b>URL</b>: Especifica el nombre completo del archivo, incluida la dirección URL del sitio y el nombre de archivo, con cualquier extensión. <br/><br/><b>Formato de datos</b>: Especifica uno de los formatos de datos admitidos: CSV, TSV, ARFF o SVM-light. Si los datos tienen una fila de encabezado, esta se usa para asignar nombres de columna. |
| Hadoop/HDFS |Lee datos de almacenamiento distribuido de Hadoop. Especifique los datos que desee mediante HiveQL, un lenguaje de consulta similar a SQL. HiveQL también se puede usar para agregar datos y realizar un filtrado de datos antes de agregarlos a Studio (clásico). |<b>Consulta de base de datos de Hive</b>: Especifica la consulta de Hive usada para generar los datos.<br/><br/><b>URI del servidor de HCatalog </b>: Especifica el nombre del clúster con el formato *&lt;nombre del clúster&gt;.azurehdinsight.net.*<br/><br/><b>Nombre de la cuenta de usuario de Hadoop</b>: Especifica el nombre de la cuenta de usuario de Hadoop usada para aprovisionar el clúster.<br/><br/><b>Contraseña de cuenta de usuario de Hadoop</b>: Especifica las credenciales usadas para aprovisionar el clúster. Para más información, consulte [Creación de clústeres de Hadoop en HDInsight](/azure/hdinsight/hdinsight-hadoop-provision-linux-clusters).<br/><br/><b>Ubicación de los datos de salida</b>: Especifica si los datos se almacenan en un sistema de archivos distribuido de Hadoop (HDFS) o en Azure. <br/><ul>Si almacena datos de salida en HDFS, especifique el URI del servidor HDFS. (Asegúrese de usar el nombre del clúster de HDInsight sin el prefijo HTTPS://). <br/><br/>Si almacena los datos de salida en Azure, debe especificar el nombre de cuenta de Azure Storage, la clave de acceso a Storage y el nombre del contenedor de Storage.</ul> |
| Base de datos SQL |Lee los datos almacenados en una base de datos de Azure SQL Database, SQL Managed Instance o SQL Server que se ejecuta en una máquina virtual de Azure. |<b>Nombre del servidor de bases de datos</b>: Especifica el nombre del servidor en el que se ejecuta la base de datos.<br/><ul>En el caso de Azure SQL Database, escriba el nombre del servidor que se genera. Normalmente tiene el formato *&lt;identificador_generado&gt;.database.windows.net*. <br/><br/>En el caso de un servidor SQL Server hospedado en una máquina virtual de Azure, escriba *tcp:&lt;nombre DNS de la máquina virtual&gt;, 1433*</ul><br/><b>Nombre de base de datos</b>: Especifica el nombre de la base de datos en el servidor. <br/><br/><b>Nombre de cuenta de usuario del servidor</b>: Especifica un nombre de usuario de una cuenta que tiene permisos de acceso para la base de datos. <br/><br/><b>Contraseña de cuenta de usuario del servidor</b>: Especifica la contraseña para la cuenta de usuario.<br/><br/><b>Consulta de base de datos</b>: escriba una instrucción SQL que describa los datos que desea leer. |
| SQL Database local |Lee los datos almacenados en una bases de datos SQL. |<b>Puerta de enlace de datos</b>: Especifica el nombre de la instancia de la puerta de enlace de administración de datos instalada en un equipo donde puede acceder a la base de datos de SQL Server. Para más información sobre cómo configurar la puerta de enlace, consulte [Análisis avanzados con Azure Machine Learning Studio (clásico) con datos de una base de datos de SQL Server](use-data-from-an-on-premises-sql-server.md).<br/><br/><b>Nombre del servidor de bases de datos</b>: Especifica el nombre del servidor en el que se ejecuta la base de datos.<br/><br/><b>Nombre de base de datos</b>: Especifica el nombre de la base de datos en el servidor. <br/><br/><b>Nombre de cuenta de usuario del servidor</b>: Especifica un nombre de usuario de una cuenta que tiene permisos de acceso para la base de datos. <br/><br/><b>Nombre de usuario y contraseña</b>: Haga clic en <b>Especificar valores</b> y escriba sus credenciales de la base de datos. Puede usar la autenticación integrada de Windows o la autenticación de SQL Server según la configuración de SQL Server.<br/><br/><b>Consulta de base de datos</b>: escriba una instrucción SQL que describa los datos que desea leer. |
| tabla de Azure |Lee los datos de Table service en Azure Storage.<br/><br/>Si se leen grandes cantidades de datos con poca frecuencia, use el servicio Tabla de Azure. Proporciona una solución de almacenamiento flexible, no relacional (No SQL), escalable a gran escala, económica y de alta disponibilidad. |Las opciones del módulo **Importar datos** cambian en función de si se accede a información pública o a una cuenta de almacenamiento privada que requiere credenciales de inicio de sesión. Esto viene determinado por el <b>Tipo de autenticación</b> que puede tener el valor "PublicOrSAS" o "Cuenta", cada uno de los cuales tiene su propio conjunto de parámetros. <br/><br/><b>URI de firma de acceso compartido (SAS) o público</b>: Los parámetros son los siguientes:<br/><br/><ul><b>URI de la tabla</b>: Especifica la dirección URL SAS o pública de la tabla.<br/><br/><b>Especifica las filas en las que se van a buscar nombres de propiedad</b>: Los valores son <i>TopN</i> para examinar el número especificado de filas, o <i>ScanAll</i> para buscar en todas las filas de la tabla. <br/><br/>Si los datos son homogéneos y predecibles, se recomienda que seleccione *TopN* y escriba un número para N. Para las tablas grandes, esto puede reducir los tiempos de lectura.<br/><br/>Si los datos están estructurados con conjuntos de propiedades que varían en función de la profundidad y la posición de la tabla, elija la opción *ScanAll* para examinar todas las filas. Esto garantiza la integridad de la propiedad resultante y la conversión de metadatos.<br/><br/></ul><b>Cuenta de almacenamiento privado</b>: Los parámetros son los siguientes: <br/><br/><ul><b>Nombre de cuenta</b>: Especifica el nombre de la cuenta que contiene la tabla que se va a leer.<br/><br/><b>Clave de cuenta</b>: Especifica la clave de almacenamiento asociada con la cuenta.<br/><br/><b>Nombre de tabla</b>: Especifica el nombre de la tabla que contiene los datos que se van a leer.<br/><br/><b>Filas en las que se van a buscar nombres de propiedad</b>: Los valores son <i>TopN</i> para examinar el número especificado de filas, o <i>ScanAll</i> para buscar en todas las filas de la tabla.<br/><br/>Si los datos son homogéneos y predecibles, se recomienda que seleccione *TopN* y escriba un número para N. Para las tablas grandes, esto puede reducir los tiempos de lectura.<br/><br/>Si los datos están estructurados con conjuntos de propiedades que varían en función de la profundidad y la posición de la tabla, elija la opción *ScanAll* para examinar todas las filas. Esto garantiza la integridad de la propiedad resultante y la conversión de metadatos.<br/><br/> |
| Azure Blob Storage |Lee los datos almacenados en Blob service en Azure Storage, incluidos imágenes, texto no estructurado o datos binarios.<br/><br/>Puede usar Blob service para exponer datos públicamente o para almacenar datos de aplicación de forma privada. Puede tener acceso a los datos desde cualquier lugar a través de conexiones HTTP o HTTPS. |Las opciones del módulo **Importar datos** cambian en función de si se accede a información pública o a una cuenta de almacenamiento privada que requiere credenciales de inicio de sesión. Esto viene determinado por el <b>Tipo de autenticación</b>, que puede tener un valor "PublicOrSAS" o "Cuenta".<br/><br/><b>URI de firma de acceso compartido (SAS) o público</b>: Los parámetros son los siguientes:<br/><br/><ul><b>URI</b>: Especifica la dirección URL SAS o pública del blob de almacenamiento.<br/><br/><b>Formato de archivo</b>: Especifica el formato de los datos en Blob service. Los formatos compatibles son CSV, TSV y ARFF.<br/><br/></ul><b>Cuenta de almacenamiento privado</b>: Los parámetros son los siguientes: <br/><br/><ul><b>Nombre de cuenta</b>: Especifica el nombre de la cuenta que contiene el blob que se va a leer.<br/><br/><b>Clave de cuenta</b>: Especifica la clave de almacenamiento asociada con la cuenta.<br/><br/><b>Ruta de acceso al contenedor, directorio o blob </b>: Especifica el nombre del blob que contiene los datos que se van a leer.<br/><br/><b>Formato de archivo de blob</b>: Especifica el formato de los datos en Blob service. Los formatos de datos admitidos son CSV, TSV, ARFF, CSV con una codificación especificada y Excel. <br/><br/><ul>Si el formato es CSV o TSV, asegúrese de indicar si el archivo contiene una fila de encabezado.<br/><br/>Puede usar la opción Excel para leer datos de libros de Excel. En la opción <i>Formato de datos de Excel</i>, indique si los datos están en un intervalo de hojas de cálculo de Excel o en una tabla de Excel. En la opción <i>Hoja de Excel o tabla insertada</i>, especifique el nombre de la hoja o tabla que desea leer.</ul><br/> |
| Proveedor de fuente de distribución de datos |Lee datos de un proveedor de fuente admitido. Actualmente se admite únicamente el formato Open Data Protocol (OData). |<b>Tipo de contenido de datos</b>: Especifica el formato OData.<br/><br/><b>Dirección URL de origen</b>: Especifica la dirección URL completa de la fuente de datos. <br/>Por ejemplo, la siguiente dirección URL lee de la base de datos de ejemplo de Northwind: https://services.odata.org/northwind/northwind.svc/ |

## <a name="import-from-another-experiment"></a>Importación desde otro experimento

Habrá ocasiones en las que querrá tomar un resultado intermedio de un experimento y usarlo como parte de otro experimento. Para ello, guarde el módulo como un conjunto de datos:

1. Haga clic en la salida del módulo que desea guardar como conjunto de datos.
2. Haga clic en **Guardar como conjunto de datos**.
3. Cuando se le solicite, escriba un nombre y una descripción que le permitan identificar fácilmente el conjunto de datos.
4. Haga clic en la marca de verificación **Aceptar** .

Cuando termine de guardar, el conjunto de datos estará disponible para usarlo dentro de cualquier experimento de su área de trabajo. Puede encontrarlo en la lista **Conjuntos de datos guardados** de la paleta de módulos.

## <a name="next-steps"></a>Pasos siguientes

[Implementación de servicios web de Azure Machine Learning Studio que usan módulos de importación y exportación de datos](web-services-that-use-import-export-modules.md)


<!-- Module References -->
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
[export-data]: https://msdn.microsoft.com/library/azure/7A391181-B6A7-4AD4-B82D-E419C0D6522C/


<!-- Module References -->
[convert-to-dataset]: https://msdn.microsoft.com/library/azure/72bf58e0-fc87-4bb1-9704-f1805003b975/
[edit-metadata]: https://msdn.microsoft.com/library/azure/370b6676-c11c-486f-bf73-35349f842a66/
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
