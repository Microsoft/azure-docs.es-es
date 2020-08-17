---
title: 'Acceder a los datos en cuadernos de Jupyter: versión preliminar de Azure Notebooks'
description: Aprenda a acceder a los archivos, las API REST, las bases de datos y los distintos recursos de Azure Storage desde un cuaderno de Jupyter.
ms.topic: how-to
ms.date: 12/04/2018
ms.custom: devx-track-python
ms.openlocfilehash: b2254e6d966ca3281cd9c8b0771cb77fb6dede33
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87846622"
---
# <a name="access-cloud-data-in-a-notebook"></a>Acceso a los datos en la nube en un cuaderno

[!INCLUDE [notebooks-status](../../includes/notebooks-status.md)]

Un trabajo interesante en un cuaderno de Jupyter requiere datos. Los datos, de hecho, son fundamentales en los cuadernos.

Puede [importar archivos de datos en un proyecto](work-with-project-data-files.md) e incluso utilizar comandos como `curl` desde un cuaderno para descargar un archivo directamente. Sin embargo, es probable que necesite trabajar con datos mucho más amplios que están disponibles desde orígenes no basados en archivos, como las API REST, las bases de datos relacionales y el almacenamiento en la nube, como las tablas de Azure.

En este artículo se describen brevemente estas diferentes opciones. Dado que el acceso a datos se ve mejor en acción, puede encontrar código ejecutable en [Ejemplos de Azure Notebooks: acceso a los datos](https://github.com/Microsoft/AzureNotebooks/blob/master/Samples/Access%20your%20data%20in%20Azure%20Notebooks.ipynb).

## <a name="rest-apis"></a>API de REST

Por lo general, se tiene acceso a la gran cantidad de datos disponibles en Internet no mediante archivos, sino con las API REST. Afortunadamente, dado que una celda del cuaderno puede contener el código que desee, puede usar este código para enviar solicitudes y recibir datos en formato JSON. A continuación, puede convertir dicho formato JSON en cualquier formato que desee usar, como una trama de datos de pandas.

Para acceder a los datos mediante una API REST, utilice el mismo código en las celdas de código de un cuaderno que el que utiliza en cualquier otra aplicación. La estructura general con la biblioteca de solicitudes es la siguiente:

```python
import pandas
import requests

# New York City taxi data for 2014
data_url = 'https://data.cityofnewyork.us/resource/gkne-dk5s.json'

# General data request; include other API keys and credentials as needed in the data argument
response = requests.get(data_url, data={"limit": "20"})

if response.status_code == 200:
    dataframe_rest2 = pandas.DataFrame.from_records(response.json())
    print(dataframe_rest2)
```

## <a name="azure-sql-database-and-sql-managed-instance"></a>Azure SQL Database y SQL Managed Instance

Puede acceder a las bases de datos en SQL Database o SQL Managed Instance con la ayuda de las bibliotecas yodbc o pymssql.

[Uso de Python para consultar una base de datos de Azure SQL](https://docs.microsoft.com/azure/sql-database/sql-database-connect-query-python) le ofrece instrucciones sobre cómo crear una base de datos en SQL Database que contiene los datos de AdventureWorks y le muestra cómo consultar dichos datos. El mismo código se muestra en el cuaderno de ejemplo de este artículo.

## <a name="azure-storage"></a>Azure Storage

Azure Storage proporciona varios tipos diferentes de almacenamiento no relacional, en función del tipo de datos que tenga y cómo necesite acceder a ellos:

- Table Storage: proporciona almacenamiento de bajo costo y gran volumen para datos tabulares, como los registros recopilados de sensores, registros de diagnóstico, etc.
- Blob Storage: proporciona almacenamiento de tipo archivo para cualquier tipo de datos.

El cuaderno de ejemplo muestra el trabajo con tablas y blobs, incluida la forma de usar una firma de acceso compartido para permitir el acceso de solo lectura a los blobs.

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

Azure Cosmos DB proporciona un almacén noSQL completamente indexado para documentos JSON. Los siguientes artículos ofrecen una serie de formas diferentes de trabajar con Cosmos DB desde Python:

- [Creación de una aplicación de SQL API con Python](https://docs.microsoft.com/azure/cosmos-db/create-sql-api-python)
- [Compilación de una aplicación de Flask con la API de Azure Cosmos DB para MongoDB](https://docs.microsoft.com/azure/cosmos-db/create-mongodb-flask)
- [Creación una base de datos de grafos mediante Python y Gremlin API](https://docs.microsoft.com/azure/cosmos-db/create-graph-python)
- [Creación de una aplicación de Cassandra con Python y Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/create-cassandra-python)
- [Creación de una aplicación de Table API con Python y Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/create-table-python)

Cuando trabaje con Cosmos DB, puede usar la biblioteca [azure-cosmosdb-table](https://pypi.org/project/azure-cosmosdb-table/).

## <a name="other-azure-databases"></a>Otras bases de datos de Azure

Azure proporciona otros tipos de base de datos que puede usar. Los artículos siguientes proporcionan instrucciones acceder a esas bases de datos desde Python:

- [Azure Database for PostgreSQL: uso de Python para conectarse a los datos y consultarlos](https://docs.microsoft.com/azure/postgresql/connect-python)
- [Inicio rápido: Uso de Azure Redis Cache con Python](https://docs.microsoft.com/azure/redis-cache/cache-python-get-started)
- [Azure Database for MySQL: uso de Python para conectarse a los datos y consultarlos](https://docs.microsoft.com/azure/mysql/connect-python)
- [Azure Data Factory](https://azure.microsoft.com/services/data-factory/)
  - [Asistente para copia de Azure Data Factory](https://azure.microsoft.com/updates/code-free-copy-wizard-for-azure-data-factory/)

## <a name="next-steps"></a>Pasos siguientes

- [Cómo: trabajo con archivos de datos de proyecto](work-with-project-data-files.md)
