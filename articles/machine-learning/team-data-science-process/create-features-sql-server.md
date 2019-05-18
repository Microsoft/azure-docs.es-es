---
title: 'Creación de características de SQL Server con SQL y Python: proceso de ciencia de datos en equipos'
description: Generar características para los datos almacenados en una máquina virtual de SQL Server en Azure con SQL y Python; parte del proceso de ciencia de datos en equipos.
services: machine-learning
author: marktab
manager: cgronlun
editor: cgronlun
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 11/21/2017
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 7bc44d8e755af3d212d616425c6a1fd925172298
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65602940"
---
# <a name="create-features-for-data-in-sql-server-using-sql-and-python"></a>Creación de características para datos de SQL Server con SQL y Python
En este documento se muestra cómo generar características para los datos almacenados en una VM de SQL Server en Azure que ayudan a los algoritmos a aprender de forma eficaz de los datos. Puede utilizar SQL o un lenguaje de programación como Python para realizar esta tarea. Ambos enfoques se muestran aquí.

Esta tarea constituye un paso del [proceso de ciencia de datos en equipos (TDSP)](https://docs.microsoft.com/azure/machine-learning/team-data-science-process/).

> [!NOTE]
> Para obtener un ejemplo práctico, puede usar el [conjunto de datos de los taxis de la Ciudad de Nueva York](https://www.andresmh.com/nyctaxitrips/) y consultar el IPNB llamado [NYC Data wrangling using IPython Notebook and SQL Server](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/iPythonNotebooks/machine-Learning-data-science-process-sql-walkthrough.ipynb) (Tratamiento de datos de la Ciudad de Nueva York mediante un Bloc de notas de IPython y SQL Server), un tutorial completo.
> 
> 

## <a name="prerequisites"></a>Requisitos previos
En este artículo se supone que ha:

* Creado una cuenta de almacenamiento de Azure. Si necesita instrucciones, consulte [Creación de una cuenta de Azure Storage](../../storage/common/storage-quickstart-create-account.md)
* Almacenado los datos en SQL Server. Si no es así, consulte [Traslado de datos a Azure SQL Database para Azure Machine Learning](move-sql-azure.md) para obtener instrucciones sobre cómo trasladar los datos.

## <a name="sql-featuregen"></a>Generación de características con SQL
En esta sección, se describen formas de generar características mediante SQL:  

1. [Generación de características basadas en recuentos](#sql-countfeature)
2. [Generación de características de discretización](#sql-binningfeature)
3. [Implementación de las características de una sola columna](#sql-featurerollout)

> [!NOTE]
> Cuando genere características adicionales, puede agregarlas como columnas a la tabla existente o crear una nueva tabla con las características adicionales y la clave principal, que se pueden combinar con la tabla original.
> 
> 

### <a name="sql-countfeature"></a>Generación de características basadas en recuentos
En este documento se muestran dos maneras de generar características de recuento. El primer método usa la suma condicional y el segundo utiliza la cláusula 'where'. Estos pueden entonces combinarse con la tabla original (con columnas de clave principal) para disponer de características de recuento junto con los datos originales.

    select <column_name1>,<column_name2>,<column_name3>, COUNT(*) as Count_Features from <tablename> group by <column_name1>,<column_name2>,<column_name3>

    select <column_name1>,<column_name2> , sum(1) as Count_Features from <tablename>
    where <column_name3> = '<some_value>' group by <column_name1>,<column_name2>

### <a name="sql-binningfeature"></a>Generación de características de discretización
En el ejemplo siguiente se muestra cómo generar características discretizadas mediante la discretización (con 5 discretizaciones) de una columna numérica que puede usarse en su lugar como una característica:

    `SELECT <column_name>, NTILE(5) OVER (ORDER BY <column_name>) AS BinNumber from <tablename>`


### <a name="sql-featurerollout"></a>Implementación de las características de una sola columna
En esta sección, se muestra cómo se implementa una sola columna de una tabla para generar características adicionales. En el ejemplo se supone que hay una columna de latitud o longitud en la tabla a partir de la cual está intentando generar características.

Aquí se incluye un breve manual sobre los datos de ubicación de latitud y longitud (extraído de stackoverflow `https://gis.stackexchange.com/questions/8650/how-to-measure-the-accuracy-of-latitude-and-longitude`). Estas son algunas cuestiones útiles que hay que comprender acerca de los datos de ubicación antes de crear características desde el campo:

* La señal indica si estamos en el norte o sur, y este u oeste del mundo.
* Un dígito de centena distinto de cero indica que se usa la longitud, y no la latitud.
* El dígito de las decenas ofrece una posición a aproximadamente 1.000 kilómetros. Nos brinda información útil sobre el continente u océano en el que nos encontramos.
* El dígito de las unidades (un grado decimal) indica una posición de hasta 111 kilómetros (60 millas náuticas, aproximadamente 69 millas). Indica aproximadamente, el estado grande o país o región se encuentran en.
* La primera posición decimal tiene un valor de hasta 11,1 km: puede distinguir la posición de una ciudad grande de otra ciudad grande vecina.
* La segundo posición decimal tiene un valor de hasta 1,1 km: puede separar un pueblo del siguiente.
* La tercera posición decimal tiene un valor de hasta 110 m: puede identificar un campo agrícola extenso o campus universitario.
* La cuarta posición decimal tiene un valor de hasta 11 m: puede identificar una parcela de tierra. Es comparable a la precisión típica de una unidad GPS sin corregir y sin interferencias.
* La quinta posición decimal tiene un valor de hasta 1,1 m: puede distinguir entre distintos árboles. Solo es posible conseguir una precisión de este nivel con unidades GPS comerciales con corrección diferencial.
* La sexta posición decimal tiene un valor de hasta 0,11 m: puede usarse para diseñar estructuras en detalle, para el diseño de paisajes o la construcción de carreteras. Debería ser más que suficiente para realizar el seguimiento de los movimientos de glaciares y ríos. Esto se consigue al tomar medidas meticulosas con GPS, como GPS corregido de forma diferencial.

La información de ubicación se puede caracterizar diferenciando entre la información de región, ubicación y ciudad. Tenga en cuenta que también es posible llamar a un extremo de REST, como la API de mapas de Bing disponible en `https://msdn.microsoft.com/library/ff701710.aspx` para obtener la información de la región o el distrito.

    select
        <location_columnname>
        ,round(<location_columnname>,0) as l1        
        ,l2=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 1 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),1,1) else '0' end     
        ,l3=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 2 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),2,1) else '0' end     
        ,l4=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 3 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),3,1) else '0' end     
        ,l5=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 4 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),4,1) else '0' end     
        ,l6=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 5 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),5,1) else '0' end     
        ,l7=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 6 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),6,1) else '0' end     
    from <tablename>

Estas características basadas en la ubicación se pueden desarrollar más para generar características de recuento adicionales, tal y como se describió anteriormente.

> [!TIP]
> Puede insertar mediante programación los registros con el lenguaje que prefiera. Debe insertar los datos en fragmentos para mejorar la eficacia de la escritura. [Este es un ejemplo de cómo realizar esa tarea con los pyodbc](https://code.google.com/p/pypyodbc/wiki/A_HelloWorld_sample_to_access_mssql_with_python).
> Otra alternativa consiste en insertar datos en la base de datos mediante la [utilidad BCP](https://msdn.microsoft.com/library/ms162802.aspx).
> 
> 

### <a name="sql-aml"></a>Conexión con Azure Machine Learning
La característica recién generada se puede agregar como una columna a una tabla existente o se puede almacenar en una tabla nueva y combinar con la tabla original para el aprendizaje automático. Es posible generar o acceder a las características si ya se han creado, mediante el módulo [Importar datos](https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/) en Aprendizaje automático de Azure, como se muestra a continuación:

![Lectores de Azure Machine Learning](./media/sql-server-virtual-machine/reader_db_featurizedinput.png)

## <a name="python"></a>Uso de un lenguaje de programación como Python
Usar Python para generar características cuando los datos están en SQL Server es parecido a procesar los datos en un blob de Azure mediante Python. Para la comparación, vea [Proceso de datos de Azure Blob en el entorno de ciencia de datos](data-blob.md). Cargue los datos de la base de datos en una trama de datos Pandas para seguir procesándolos. El proceso de conexión a la base de datos y de carga de los datos en la trama de datos se documenta en esta sección.

El formato de cadena de conexión siguiente puede usarse para conectarse a una base de datos de SQL Server desde Python mediante pyodbc (reemplace servername, dbname, username y password con sus valores específicos):

    #Set up the SQL Azure connection
    import pyodbc
    conn = pyodbc.connect('DRIVER={SQL Server};SERVER=<servername>;DATABASE=<dbname>;UID=<username>;PWD=<password>')

La [biblioteca Pandas](https://pandas.pydata.org/) en Python ofrece un amplio conjunto de herramientas de análisis de datos y estructuras de datos para la manipulación de datos para la programación en Python. El código siguiente lee los resultados que se devuelven desde una base de datos de SQL Server en una trama de datos de Pandas:

    # Query database and load the returned results in pandas data frame
    data_frame = pd.read_sql('''select <columnname1>, <columnname2>... from <tablename>''', conn)

Ya puede trabajar con la trama de datos de Pandas como se explica en los temas [Creación de características para los datos de Azure Blob Storage mediante Panda](create-features-blob.md).

