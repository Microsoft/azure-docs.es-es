---
title: 'Tutorial: Introducción al análisis de datos en cuentas de almacenamiento'
description: En este tutorial, aprenderá a analizar los datos ubicados en una cuenta de almacenamiento.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.date: 07/20/2020
ms.openlocfilehash: 836e56884659c60c129eba0bb5505eddd9981283
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87093904"
---
# <a name="analyze-data-in-a-storage-account"></a>Análisis de datos en una cuenta de almacenamiento

En este tutorial, aprenderá a analizar los datos ubicados en una cuenta de almacenamiento.

## <a name="overview"></a>Información general

Hasta ahora, hemos descrito escenarios en los que los datos residían en las bases de datos del área de trabajo. Ahora le mostraremos cómo trabajar con archivos que residen en cuentas de almacenamiento. En este escenario, se usará la cuenta de almacenamiento principal del área de trabajo y el contenedor que se especificó al crear el área de trabajo.

* El nombre de la cuenta de almacenamiento: **contosolake**
* El nombre del contenedor en la cuenta de almacenamiento: **users**

### <a name="create-csv-and-parquet-files-in-your-storage-account"></a>Creación de archivos CSV y Parquet en la cuenta de almacenamiento

Ejecute el siguiente código en un cuaderno. Crea un archivo .csv y un archivo Parquet en la cuenta de almacenamiento.

```py
%%pyspark
df = spark.sql("SELECT * FROM nyctaxi.passengercountstats")
df = df.repartition(1) # This ensure we'll get a single file during write()
df.write.mode("overwrite").csv("/NYCTaxi/PassengerCountStats.csv")
df.write.mode("overwrite").parquet("/NYCTaxi/PassengerCountStats.parquet")
```

### <a name="analyze-data-in-a-storage-account"></a>Análisis de datos en una cuenta de almacenamiento

1. En Synapse Studio, vaya al centro **Data** (Datos) y, a continuación, seleccione **Vinculado**.
1. Vaya a **Cuentas de almacenamiento** > **myworkspace (Principal: contosolake)** .
1. Seleccione **users (Primary)** (usuarios [Principal]). Debería ver la carpeta **NYCTaxi**. Dentro debería ver dos carpetas llamadas **PassengerCountStats.csv** y **PassengerCountStats.parquet**.
1. Abra la carpeta **PassengerCountStats.parquet**. Dentro, verá un archivo Parquet con un nombre como `part-00000-2638e00c-0790-496b-a523-578da9a15019-c000.snappy.parquet`.
1. Haga clic con el botón secundario en el archivo **.parquet** y, a continuación, seleccione **Nuevo cuaderno**. Se crea un cuaderno que tiene una celda similar a la siguiente:

    ```py
    %%pyspark
    data_path = spark.read.load('abfss://users@contosolake.dfs.core.windows.net/NYCTaxi/PassengerCountStats.parquet/part-00000-1f251a58-d8ac-4972-9215-8d528d490690-c000.snappy.parquet', format='parquet')
    data_path.show(100)
    ```

1. Ejecute la celda.
1. Haga clic con el botón derecho en el archivo Parquet que está dentro y seleccione **New SQL Script** > **Select TOP 100 Rows** (Nuevo script SQL > Seleccionar 100 primeras filas). Se crea un script SQL similar al siguiente:

    ```sql
    SELECT TOP 100 *
    FROM OPENROWSET(
        BULK 'https://contosolake.dfs.core.windows.net/users/NYCTaxi/PassengerCountStats.parquet/part-00000-1f251a58-d8ac-4972-9215-8d528d490690-c000.snappy.parquet',
        FORMAT='PARQUET'
    ) AS [r];
    ```

     En el script, el campo **Adjuntar a** se establecerá en **SQL On-Demand**.

1. Ejecute el script.



## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Organización de actividades con canalizaciones](get-started-pipelines.md)
