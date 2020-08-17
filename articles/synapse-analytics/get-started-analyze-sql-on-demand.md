---
title: 'Tutorial: Introducción a los análisis de datos con SQL a petición'
description: En este tutorial, aprenderá a analizar datos con SQL a petición mediante los datos ubicados en bases de datos de Spark.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.date: 07/20/2020
ms.openlocfilehash: 4011cd93879d9203d8231f24bbf531d14e6e815a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87093823"
---
# <a name="analyze-data-with-sql-on-demand"></a>Análisis de datos con SQL a petición

En este tutorial, aprenderá a analizar datos con SQL a petición mediante los datos ubicados en bases de datos de Spark. 

## <a name="analyze-nyc-taxi-data-in-spark-databases-using-sql-on-demand"></a>Análisis de los datos de taxis de Nueva York en las bases de datos de Spark mediante SQL On-Demand

SQL On-Demand ve y puede consultar automáticamente las tablas de las bases de datos de Spark.

1. En Synapse Studio, vaya al centro **Develop** (Desarrollo) y cree un script de SQL.
1. En **Conectarse a** seleccione **SQL On-Demand**.
1. Pegue el siguiente texto en el script y ejecute el script.

    ```sql
    SELECT *
    FROM nyctaxi.dbo.passengercountstats
    ```

    > [!NOTE]
    > La primera vez que se ejecuta una consulta que usa SQL On-Demand, se tardarán unos 10 segundos en recopilar los recursos de SQL necesarios para ejecutar las consultas. Las consultas posteriores serán mucho más rápidas.
  


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Análisis mediante Spark](get-started-analyze-spark.md)
