---
title: Uso de los comandos y las características de cuadernos integrados en cuadernos de Python de Azure Cosmos DB (versión preliminar)
description: Aprenda a usar los comandos y las características integrados para realizar operaciones comunes con los cuadernos de Python integrados de Azure Cosmos DB.
author: deborahc
ms.service: cosmos-db
ms.topic: how-to
ms.date: 05/19/2020
ms.author: dech
ms.openlocfilehash: f4236b099915e737f9d489d89e5bf18b84736d75
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87761202"
---
# <a name="use-built-in-notebook-commands-and-features-in-azure-cosmos-db-python-notebooks-preview"></a>Uso de los comandos y las características de cuadernos integrados en cuadernos de Python de Azure Cosmos DB (versión preliminar)

Los cuadernos de Jupyter integrados en Azure Cosmos DB permiten analizar y visualizar los datos desde Azure Portal. En este artículo se describe cómo usar los comandos y las características de cuadernos integrados para realizar operaciones comunes en cuadernos de Python.

## <a name="install-a-new-package"></a>Instalar un nuevo paquete
Después de habilitar la compatibilidad con el cuaderno en sus cuentas de Azure Cosmos, puede abrir un nuevo cuaderno e instalar un paquete.

En una nueva celda de código, inserte y ejecute el código siguiente, reemplazando ``PackageToBeInstalled`` con el paquete de Python que quiera usar.
```python
import sys
!{sys.executable} -m pip install PackageToBeInstalled --user
```
Este paquete estará disponible en cualquier cuaderno del área de trabajo de la cuenta de Azure Cosmos. 

> [!TIP]
> Si el cuaderno requiere un paquete personalizado, es recomendable que agregue una celda en el cuaderno para instalar el paquete, ya que los paquetes se eliminan si [restablece el área de trabajo](#reset-notebooks-workspace).  

## <a name="run-a-sql-query"></a>Ejecutar una consulta SQL

Puede usar el comando mágico ``%%sql`` para ejecutar una [consulta de SQL](sql-query-getting-started.md) en cualquier contenedor de su cuenta. La sintaxis es la siguiente:

```python
%%sql --database {database_id} --container {container_id}
{Query text}
```

- Reemplace ``{database_id}`` y ``{container_id}`` por el nombre de la base de datos y el contenedor en la cuenta de Cosmos. Si no se proporcionan los argumentos ``--database`` y ``--container``, la consulta se ejecutará en la [base de datos y en el contenedor predeterminados](#set-default-database-for-queries).
- Puede ejecutar cualquier consulta SQL que sea válida en Azure Cosmos DB. El texto de la consulta debe estar en una línea nueva.

Por ejemplo: 
```python
%%sql --database RetailDemo --container WebsiteData
SELECT c.Action, c.Price as ItemRevenue, c.Country, c.Item FROM c
```
Ejecute ```%%sql?``` en una celda para ver la documentación de ayuda del comando mágico de SQL en el cuaderno.

## <a name="run-a-sql-query-and-output-to-a-pandas-dataframe"></a>Ejecutar una consulta SQL y devolver resultados en un DataFrame de Pandas

Puede acceder a los resultados de una consulta ``%%sql`` en [Pandas DataFrame](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html#pandas.DataFrame). La sintaxis es la siguiente: 

```python
%%sql --database {database_id} --container {container_id} --output {outputDataFrameVar}
{Query text}
```
- Reemplace ``{database_id}`` y ``{container_id}`` por el nombre de la base de datos y el contenedor en la cuenta de Cosmos. Si no se proporcionan los argumentos ``--database`` y ``--container``, la consulta se ejecutará en la [base de datos y en el contenedor predeterminados](#set-default-database-for-queries).
- Reemplace ``{outputDataFrameVar}`` por el nombre de la variable de DataFrame que contendrá los resultados.
- Puede ejecutar cualquier consulta SQL que sea válida en Azure Cosmos DB. El texto de la consulta debe estar en una línea nueva. 

Por ejemplo:

```python
%%sql --database RetailDemo --container WebsiteData --output df_cosmos
SELECT c.Action, c.Price as ItemRevenue, c.Country, c.Item FROM c
```
```python
df_cosmos.head(10)

    Action    ItemRevenue    Country/Region    Item
0    Viewed    9.00    Tunisia    Black Tee
1    Viewed    19.99    Antigua and Barbuda    Flannel Shirt
2    Added    3.75    Guinea-Bissau    Socks
3    Viewed    3.75    Guinea-Bissau    Socks
4    Viewed    55.00    Czech Republic    Rainjacket
5    Viewed    350.00    Iceland    Cosmos T-shirt
6    Added    19.99    Syrian Arab Republic    Button-Up Shirt
7    Viewed    19.99    Syrian Arab Republic    Button-Up Shirt
8    Viewed    33.00    Tuvalu    Red Top
9    Viewed    14.00    Cabo Verde    Flip Flop Shoes
```
## <a name="set-default-database-for-queries"></a>Establecer la base de datos predeterminada para las consultas
Puede establecer los comandos ```%%sql``` de la base de datos predeterminada que se usarán para el cuaderno. Reemplace ```{database_id}``` por el nombre de la base de datos.

```python
%database {database_id}
```
Ejecute ```%database?``` en una celda para ver la documentación en el cuaderno.

## <a name="set-default-container-for-queries"></a>Establecer el contenedor predeterminado para las consultas
Puede establecer los comandos ```%%sql``` del contenedor predeterminado que se usarán para el cuaderno. Reemplace ```{container_id}``` por el nombre del contenedor.

```python
%container {container_id}
```
Ejecute ```%container?``` en una celda para ver la documentación en el cuaderno.

## <a name="upload-json-items-to-a-container"></a>Carga de elementos JSON en un contenedor
Puede usar el comando mágico ``%%upload`` para cargar datos de un archivo JSON en un contenedor específico de Azure Cosmos. Ejecute el siguiente comando para cargar los elementos:

```python
%%upload --databaseName {database_id} --containerName {container_id} --url {url_location_of_file}
```

- Reemplace ``{database_id}`` y ``{container_id}`` por el nombre de la base de datos y el contenedor en la cuenta de Azure Cosmos. Si no se proporcionan los argumentos ``--database`` y ``--container``, la consulta se ejecutará en la [base de datos y en el contenedor predeterminados](#set-default-database-for-queries).
- Reemplace ``{url_location_of_file}`` por la ubicación del archivo JSON. El archivo debe ser una matriz de objetos JSON válidos y accesible a través de la red pública de Internet.

Por ejemplo:

```python
%%upload --database databaseName --container containerName --url 
https://contoso.com/path/to/data.json
```
```
Documents successfully uploaded to ContainerName
Total number of documents imported : 2654
Total time taken : 00:00:38.1228087 hours
Total RUs consumed : 25022.58
```
Con las estadísticas de salida, puede calcular las RU/s reales que se usan para cargar los elementos. Por ejemplo, si 25 000 RU se consumieron más de 38 segundos, las RU/s reales son 25 000 RU/38 segundos = 658 RU/s.

## <a name="run-another-notebook-in-current-notebook"></a>Ejecución de otro cuaderno en el actual 
Puede usar el comando mágico ``%%run`` para ejecutar otro cuaderno en el área de trabajo desde el cuaderno actual. La sintaxis es la siguiente:

```python
%%run ./path/to/{notebookName}.ipynb
```
Reemplace ``{notebookName}`` por el nombre del cuaderno que quiere ejecutar. El cuaderno debe estar en el área de trabajo "My Notebooks" (Mis cuadernos) actual. 

## <a name="use-built-in-nteract-data-explorer"></a>Uso del explorador de datos nteract integrado
Puede usar el explorador de datos integrado [nteract](https://blog.nteract.io/designing-the-nteract-data-explorer-f4476d53f897) para filtrar y visualizar una trama de datos. Para habilitar esta característica, establezca la opción ``pd.options.display.html.table_schema`` en ``True`` y ``pd.options.display.max_rows`` en el valor deseado (puede establecer ``pd.options.display.max_rows`` en ``None`` para mostrar todos los resultados).

```python
import pandas as pd
pd.options.display.html.table_schema = True
pd.options.display.max_rows = None

df_cosmos.groupby("Item").size()
```
:::image type="content" source="media/use-notebook-features-and-commands/nteract-built-in-chart.png" alt-text="Explorador de datos nteract":::

## <a name="use-the-built-in-python-sdk"></a>Uso del SDK de Python integrado
La versión 4 del [SDK de Python de Azure Cosmos DB para la API de SQL](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cosmos/azure-cosmos) está instalada e incluida en el entorno del cuaderno para la cuenta de Azure Cosmos.

Use la instancia integrada ``cosmos_client`` para ejecutar cualquier operación de SDK. 

Por ejemplo:

```python
## Import modules as needed
from azure.cosmos.partition_key import PartitionKey

## Create a new database if it doesn't exist
database = cosmos_client.create_database_if_not_exists('RetailDemo')

## Create a new container if it doesn't exist
container = database.create_container_if_not_exists(id='WebsiteData', partition_key=PartitionKey(path='/CartID'))
```
consulte los [ejemplos de SDK de Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cosmos/azure-cosmos/samples). 

> [!IMPORTANT]
> El SDK de Python integrado solo se admite para las cuentas de la API de SQL (Core). Para otras API, deberá [instalar el controlador de Python correspondiente](#install-a-new-package) a la API. 

## <a name="create-a-custom-instance-of-cosmos_client"></a>Creación de una instancia personalizada de ``cosmos_client``
Para lograr una mayor flexibilidad, puede crear una instancia personalizada de ``cosmos_client`` con el fin de:

- Personalizar la [directiva de conexión](https://docs.microsoft.com/python/api/azure-cosmos/azure.cosmos.documents.connectionpolicy?view=azure-python-preview).
- Ejecución de operaciones en una cuenta de Azure Cosmos diferente de la que se está usando.

Puede obtener acceso a la cadena de conexión y a la clave principal de la cuenta actual a través de las [variables de entorno](#access-the-account-endpoint-and-primary-key-env-variables). 

```python
import azure.cosmos.cosmos_client as cosmos
import azure.cosmos.documents as documents

# These should be set to a region you've added for Azure Cosmos DB
region_1 = "Central US" 
region_2 = "East US 2"

custom_connection_policy = documents.ConnectionPolicy()
custom_connection_policy.PreferredLocations = [region_1, region_2] # Set the order of regions the SDK will route requests to. The regions should be regions you've added for Cosmos, otherwise this will error.

# Create a new instance of CosmosClient, getting the endpoint and key from the environment variables
custom_client = cosmos.CosmosClient(url=COSMOS.ENDPOINT, credential=COSMOS.KEY, connection_policy=custom_connection_policy)

```
## <a name="access-the-account-endpoint-and-primary-key-env-variables"></a>Acceder al punto de conexión de la cuenta y a las variables de tipo "env" de la clave principal
Puede acceder al punto de conexión integrado y a la clave de la cuenta en que se encuentra el cuaderno.

```python
endpoint = COSMOS.ENDPOINT
primary_key = COSMOS.KEY
```
> [!IMPORTANT]
> Las variables ``COSMOS.ENDPOINT`` y ``COSMOS.KEY`` solo se pueden aplicar a la API de SQL. En cuanto a otras API, busque el punto de conexión y la clave en la hoja de las **cadenas de conexión** o las **claves** en su cuenta de Azure Cosmos.  

## <a name="reset-notebooks-workspace"></a>Restablecer el área de trabajo de los cuadernos
Para restablecer la configuración predeterminada del área de trabajo de los cuadernos, seleccione **Reset Workspace** (Restablecer el área de trabajo) en la barra de comandos. Se quitarán todos los paquetes instalados personalizados y se reiniciará el servidor de Jupyter. Los cuadernos, los archivos y los recursos de Azure Cosmos no se verán afectados.  

:::image type="content" source="media/use-notebook-features-and-commands/reset-workspace.png" alt-text="Restablecer el área de trabajo de los cuadernos":::

## <a name="next-steps"></a>Pasos siguientes

- Conozca las ventajas de los [cuadernos de Jupyter para Azure Cosmos DB](cosmosdb-jupyter-notebooks.md)
- Obtenga información acerca del [SDK de Python para Azure Cosmos DB para la API de SQL](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cosmos/azure-cosmos)
