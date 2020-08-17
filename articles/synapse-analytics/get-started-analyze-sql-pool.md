---
title: 'Tutorial: Introducción al análisis de datos con un grupo de SQL'
description: En este tutorial, usará los datos de ejemplo de NYC Taxi para explorar las funcionalidades de análisis del grupo de SQL.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.date: 07/20/2020
ms.openlocfilehash: e2e1d0479b8edacaae8816d74db061eeedb805a7
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325226"
---
# <a name="analyze-data-with-sql-pools"></a>Análisis de datos con grupos de SQL

Azure Synapse Analytics le ofrece la capacidad de analizar datos con el grupo de SQL. En este tutorial, usará los datos de ejemplo de NYC Taxi para explorar las funcionalidades de análisis del grupo de SQL.

## <a name="load-the-nyc-taxi-sample-data-into-the-sqldb1-database"></a>Carga de los datos del ejemplo de taxis de Nueva York en la base de datos SQLDB1

1. En Synapse Studio, en el menú azul de la parte superior, seleccione el icono del signo de interrogación ( **?** ).
1. Seleccione **Getting started** > **Getting started hub** (Introducción > Introducción al centro).
1. En la tarjeta con la etiqueta **Query sample data** (Consultar datos de ejemplo), seleccione el grupo de SQL llamado **SQLDB1**.
1. Seleccione **Query data** (Consultar datos). Aparece brevemente la notificación "Cargando datos de ejemplo". Una barra de estado de color azul claro cerca de la parte superior de Synapse Studio indica que se están cargando los datos en SQLDB1.
1. Cuando la barra de estado se vuelva verde, descártela.

## <a name="explore-the-nyc-taxi-data-in-the-sql-pool"></a>Exploración de los datos de taxis de Nueva York en el grupo de SQL

1. En Synapse Studio, vaya al centro **Data** (Datos).
1. Vaya a **SQLDB1** > **Tablas**. Verá que se han cargado varias tablas.
1. Haga clic con el botón derecho en la tabla **dbo.Trip** y seleccione **New SQL Script** > **Select TOP 100 Rows** (Nuevo script SQL > Seleccionar 100 primeras filas).
1. Espere mientras se crea un nuevo script de SQL y se ejecuta.
1. Observe que en la parte superior del script de SQL, en **Connect to** (Conectar a) está seleccionado automáticamente el grupo de SQL llamado **SQLDB1**.
1. Reemplace el texto del script de SQL por este código y ejecútelo.

    ```sql
    SELECT PassengerCount,
          SUM(TripDistanceMiles) as SumTripDistance,
          AVG(TripDistanceMiles) as AvgTripDistance
    FROM  dbo.Trip
    WHERE TripDistanceMiles > 0 AND PassengerCount > 0
    GROUP BY PassengerCount
    ORDER BY PassengerCount
    ```

    En esta consulta se muestra la relación entre las distancias recorridas totales y la distancia media recorrida, y el número de pasajeros.
1. En la ventana de resultados del script de SQL, cambie el valor de **View** (Vista) a **Chart** (Gráfico) para ver los resultados en un gráfico de líneas.



## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Análisis mediante Spark](get-started-analyze-spark.md)

