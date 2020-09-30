---
title: Power BI y Synapse SQL sin servidor para analizar los datos de Azure Cosmos DB con Synapse Link
description: Aprenda a crear una base de datos de Synapse SQL sin servidor y vistas de Synapse Link para Azure Cosmos DB, a consultar contenedores de Azure Cosmos y, finalmente, a crear un modelo de esas vistas con Power BI.
author: ArnoMicrosoft
ms.service: cosmos-db
ms.topic: how-to
ms.date: 09/22/2020
ms.author: acomet
ms.openlocfilehash: 03ea1b0cdfef30935b38078d0811d1408a78c41e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932012"
---
# <a name="use-power-bi-and-synapse-sql-serverless-to-analyze-azure-cosmos-db-data-with-synapse-link-preview"></a>Uso de Power BI y Synapse SQL sin servidor para analizar los datos de Azure Cosmos DB con Synapse Link (versión preliminar)

En este artículo aprenderá a crear una base de datos de Synapse SQL sin servidor (que anteriormente se conocía como **SQL a petición**) y vistas de Synapse Link para Azure Cosmos DB. Consultará los contenedores de Azure Cosmos y, después, creará un modelo con Power BI con esas vistas que refleje esa consulta.

> [!NOTE]
> El uso del almacén analítico de Azure Cosmos DB con Synapse SQL sin servidor se encuentra actualmente en versión preliminar controlada. Para solicitar acceso, póngase en contacto con el [equipo de Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com).

En este escenario usará datos ficticios sobre las ventas de productos de Surface en un comercio asociado. Analizará los ingresos por tienda en función de la proximidad a núcleos familiares grandes y el impacto de la publicidad durante una semana específica. En este artículo creará dos vistas denominadas **RetailSales** y **StoreDemographics** y una consulta entre ellas. Puede obtener los datos del producto de ejemplo en este repositorio de [GitHub](https://github.com/Azure-Samples/Synapse/tree/master/Notebooks/PySpark/Synapse%20Link%20for%20Cosmos%20DB%20samples/Retail/RetailData).

## <a name="prerequisites"></a>Requisitos previos

Asegúrese de crear los siguientes recursos antes de empezar:

* [Cree una cuenta de Azure Cosmos DB de tipo SQL (Core) o MongoDB.](create-cosmosdb-resources-portal.md)

* Habilitación de Azure Synapse Link para la [cuenta de Azure Cosmos](configure-synapse-link.md#enable-synapse-link)

* Cree una base de datos en la cuenta de Azure Cosmos y dos contenedores que tengan el [almacén analítico habilitado.](configure-synapse-link.md#create-analytical-ttl)

* Cargue los datos de los productos en los contenedores de Azure Cosmos como se indica en este cuaderno sobre [ingesta de datos por lotes](https://github.com/Azure-Samples/Synapse/blob/master/Notebooks/PySpark/Synapse%20Link%20for%20Cosmos%20DB%20samples/Retail/spark-notebooks/pyspark/1CosmoDBSynapseSparkBatchIngestion.ipynb).

* [Cree un área de trabajo de Synapse](../synapse-analytics/quickstart-create-workspace.md) denominada **SynapseLinkBI**.

* [Conecte la base de datos de Azure Cosmos al área de trabajo de Synapse](../synapse-analytics/synapse-link/how-to-connect-synapse-link-cosmos-db.md?toc=/azure/cosmos-db/toc.json&bc=/azure/cosmos-db/breadcrumb/toc.json).

## <a name="create-a-database-and-views"></a>Creación de una base de datos y vistas

En el área de trabajo de Synapse, vaya a la pestaña **Desarrollar**, y seleccione el icono **+** y **Script SQL**.

:::image type="content" source="./media/synapse-link-power-bi/add-sql-script.png" alt-text="Incorporación de un script SQL al área de trabajo de Synapse Analytics":::

Todas las áreas de trabajo incluyen un punto de conexión de Synapse SQL sin servidor. Después de crear un script SQL, en la barra de herramientas de la parte superior, conéctese a **SQL a petición**.

:::image type="content" source="./media/synapse-link-power-bi/enable-sql-on-demand-endpoint.png" alt-text="Habilitación del script SQL para usar el punto de conexión de Synapse SQL sin servidor en el área de trabajo":::

Cree una base de datos, denominada **RetailCosmosDB**, y una vista SQL con los contenedores habilitados para Synapse Link. El siguiente comando muestra cómo crear una base de datos:

```sql
-- Create database
Create database RetailCosmosDB
```

A continuación, cree varias vistas en distintos contenedores de Azure Cosmos habilitados para Synapse Link. Esto le permitirá usar T-SQL para combinar y consultar datos de Azure Cosmos DB de distintos contenedores.  Asegúrese de seleccionar la base de datos **RetailCosmosDB** al crear las vistas.

Los scripts siguientes muestran cómo crear vistas en cada contenedor. Para simplificar, vamos a usar la característica de [inferencia automática de esquemas](analytical-store-introduction.md#analytical-schema) de Synapse SQL sin servidor con contenedores habilitados para Synapse Link:


### <a name="retailsales-view"></a>Vista RetailSales:

```sql
-- Create view for RetailSales container
CREATE VIEW  RetailSales
AS  
SELECT  *
FROM OPENROWSET (
    'CosmosDB', N'account=<Your Azure Cosmos account name>;database=<Your Azure Cosmos database name>;region=<Your Azure Cosmos DB Region>;key=<Your Azure Cosmos DB key here>',RetailSales)
AS q1
```

Asegúrese de insertar la región de Azure Cosmos DB y la clave principal en el script SQL anterior. Todos los caracteres del nombre de la región deben estar en minúsculas, sin espacios. A diferencia de los demás parámetros del comando `OPENROWSET`, el parámetro del nombre de contenedor debe especificarse sin comillas.

### <a name="storedemographics-view"></a>Vista StoreDemographics:

```sql
-- Create view for StoreDemographics container
CREATE VIEW StoreDemographics
AS  
SELECT  *
FROM OPENROWSET (
    'CosmosDB', N'account=<Your Azure Cosmos account name>;database=<Your Azure Cosmos database name>;region=<Your Azure Cosmos DB Region>;key=<Your Azure Cosmos DB key here>', StoreDemographics)
AS q1
```

Ahora ejecute el script SQL; para ello, seleccione el comando **Run**.

## <a name="query-the-views"></a>Consulta de las vistas

Ahora que se han creado las dos vistas, vamos a definir la consulta para combinar esas dos vistas de la siguiente manera:

```sql
SELECT 
sum(p.[revenue]) as revenue
,p.[advertising]
,p.[storeId]
,p.[weekStarting]
,q.[largeHH]
 FROM [dbo].[RetailSales] as p
INNER JOIN [dbo].[StoreDemographics] as q ON q.[storeId] = p.[storeId]
GROUP BY p.[advertising], p.[storeId], p.[weekStarting], q.[largeHH]
```

Seleccione **Run** para obtener la siguiente tabla como resultado:

:::image type="content" source="./media/synapse-link-power-bi/join-views-query-results.png" alt-text="Resultados de la consulta después de combinar las vistas StoreDemographics y RetailSales":::

## <a name="model-views-over-containers-with-power-bi"></a>Vistas del modelo en contenedores con Power BI

A continuación, abra Power BI Desktop y conéctese al punto de conexión de Synapse SQL sin servidor; para ello, siga estos pasos:

1. Abra la aplicación Power BI Desktop. Seleccione **Obtener datos** y **Más**.

1. Elija **Azure Synapse Analytics (SQL DW)** de la lista de opciones de conexión.

1. Escriba el nombre del punto de conexión de SQL donde se encuentra la base de datos. Escriba `SynapseLinkBI-ondemand.sql.azuresynapse.net` en el campo **Servidor**. En este ejemplo, **SynapseLinkBI** es el nombre del área de trabajo. Reemplácelo si le ha dado un nombre diferente al área de trabajo. Seleccione **DirectQuery** como modo de conectividad y **Aceptar**.

1. Seleccione el método de autenticación preferido, por ejemplo, Azure AD.

1. Seleccione la base de datos **RetailCosmosDB** y las vistas **RetailSales** y **StoreDemographics**.

1. Seleccione **Cargar** para cargar las dos vistas en el modo de consulta directa.

1. Seleccione **Modelo** para crear una relación entre las dos vistas mediante la columna **storeId**.

1. Arrastre la columna **StoreId** de la vista **RetailSales** a la columna **StoreId** de la vista **StoreDemographics**.

1. Seleccione la relación Varios a uno (*:1), ya que hay varias filas con el mismo identificador de tienda en la vista **RetailSales**, pero solo una fila de identificador de tienda en **StoreDemographics** (es una tabla de dimensiones)

Ahora vaya a la ventana **report** y cree un informe para comparar la importancia relativa del tamaño de los núcleos familiares con el promedio de ingresos por tienda en función de la representación dispersa de ingresos y el índice LargeHH:

1. Seleccione **Gráfico de dispersión**.

1. Arrastre y coloque **LargeHH** desde la vista **StoreDemographics** hasta el eje X.

1. Arrastre y coloque **Revenue** (Ingresos) desde la vista **RetailSales** hasta el eje Y. Seleccione **Media** para obtener el promedio de ventas por producto por tienda y por semana.

1. Arrastre y coloque **productCode** de la vista **RetailSales** en la leyenda para seleccionar una línea de productos específica.
Después de elegir estas opciones, debería ver un gráfico como el de la siguiente captura de pantalla:

:::image type="content" source="./media/synapse-link-power-bi/household-size-average-revenue-report.png" alt-text="Informe de comparación de la importancia relativa del tamaño del núcleo familiar con el promedio de ingresos por tienda":::

## <a name="next-steps"></a>Pasos siguientes

Uso de Synapse SQL sin servidor para [analizar Azure Open Datasets y visualizar los resultados en Azure Synapse Studio](../synapse-analytics/sql/tutorial-data-analyst.md)
