---
title: 'Configuración de Blockchain Data Manager con la CLI de Azure: Azure Blockchain Service'
description: Crear y administrar una instancia de Blockchain Data Manager para Azure Blockchain Service mediante la CLI de Azure
ms.date: 03/30/2020
ms.topic: how-to
ms.reviewer: ravastra
ms.openlocfilehash: f067f4413f6ad8541cd36a7581f9243bed4e195f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87023745"
---
# <a name="configure-blockchain-data-manager-using-azure-cli"></a>Configuración de la cadena de bloques Data Manager con la CLI de Azure

Configure la cadena de bloques Data Manager para que la cadena de bloques de Azure Blockchain Service capture los datos de la cadena de bloques y los envíe a un tema Azure Event Grid.

Para configurar una instancia de cadena de bloques Data Manager, puede:

* Crear una instancia del Administrador de la cadena de bloques
* Crear una entrada a un nodo de transacción de Azure Blockchain Service
* Crear una salida para un tema Azure Event Grid
* Agregar una aplicación de cadena de bloques
* Iniciar una instancia

## <a name="prerequisites"></a>Requisitos previos

* Instale el [CLI de Azure](/cli/azure/install-azure-cli) más reciente y con la sesión iniciada`az login`.
* Realizar el tutorial [Quickstart: Uso de Visual Studio Code para conectarse a una red del consorcio de Azure Blockchain Service](connect-vscode.md). Se recomienda el nivel *Estándar* de Azure Blockchain Service cuando se usa Blockchain Data Manager.
* Crear un [tema de Event Grid](../../event-grid/custom-event-quickstart-portal.md#create-a-custom-topic)
* Más información sobre [Controladores de eventos en Azure Event Grid](../../event-grid/event-handlers.md)

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta.

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/bash](https://shell.azure.com/bash) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

Si prefiere instalar y usar la CLI de forma local, en este inicio rápido se requiere la versión 2.0.51 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive
az group create --name myRG --location eastus
```

## <a name="create-instance"></a>Crear instancia

Una instancia de la cadena de bloques Data Manager supervisa un nodo de transacción de servicio de Azure Blockchain. Una instancia captura todos los datos de transacciones y bloques sin procesar del nodo de transacción. Blockchain Data Manager publica un mensaje **RawBlockAndTransactionMsg**, que es un superconjunto de la información devuelta por las consultas web3.eth [getBlock](https://web3js.readthedocs.io/en/v1.2.0/web3-eth.html#getblock) y [getTransaction](https://web3js.readthedocs.io/en/v1.2.0/web3-eth.html#gettransaction).

``` azurecli
az resource create \
                   --resource-group <Resource group> \
                   --name <Blockchain Data Manager instance name> \
                   --resource-type Microsoft.blockchain/watchers \
                   --is-full-object \
                   --properties <watcher resource properties>
```

| Parámetro | Descripción |
|-----------|-------------|
| resource-group | Nombre del grupo de recursos donde se creará la instancia de la cadena de bloques Data Manager. |
| name | Nombre de la instancia de la cadena de bloques Data Manager. |
| tipo de recurso | El tipo de recurso de una instancia de la cadena de bloques Data Manager es **Microsoft.blockchain/watchers**. |
| is-full-object | Indica que las propiedades contienen opciones para el recurso de monitor. |
| properties | Cadena con formato JSON que contiene las propiedades del recurso de monitor. Se puede pasar como una cadena o un archivo.  |

### <a name="create-instance-examples"></a>Ejemplos de creación de instancia

Ejemplo de configuración de JSON para crear una instancia del administrador de la cadena de bloques en la región **este de EE. UU.** .

``` json
{
    "location": "eastus",
    "properties": {
    }
}
```

| Elemento | Descripción |
|---------|-------------|
| ubicación | Región en la que se creará el recurso de monitor |
| properties | Propiedades que se establecen al crear el recurso de monitor |

Cree una instancia de la cadena de bloques de Data Manager denominada *mywatcher* con una cadena JSON para la configuración.

``` azurecli-interactive
az resource create \
                    --resource-group myRG \
                    --name mywatcher \
                    --resource-type Microsoft.blockchain/watchers \
                     --is-full-object \
                     --properties '{"location":"eastus"}'
```

Cree una instancia de la cadena de bloques de Data Manager denominada *mywatcher* con el archivo de configuración JSON.

``` azurecli
az resource create \
                    --resource-group myRG \
                    --name mywatcher \
                    --resource-type Microsoft.blockchain/watchers \
                    --is-full-object \
                    --properties @watcher.json
```

## <a name="create-input"></a>Creación de una entrada

Una entrada conecta a la cadena de bloques Data Manager a un nodo de transacción de servicio Azure Blockchain. Solo los usuarios con acceso al nodo de transacción pueden crear una conexión.

``` azurecli
az resource create \
                   --resource-group <Resource group> \
                   --name <Input name> \
                   --namespace Microsoft.Blockchain \
                   --resource-type inputs \
                   --parent watchers/<Watcher name> \
                   --is-full-object \
                   --properties <input resource properties>
```

| Parámetro | Descripción |
|-----------|-------------|
| resource-group | Nombre del grupo de recursos donde se creará el recurso de entrada. |
| name | Nombre de la entrada. |
| espacio de nombres | Use el espacio de nombres del proveedor **Microsoft.Blockchain**. |
| tipo de recurso | El tipo de recurso para una entrada de la cadena de bloques Data Manager es **entradas**. |
| primario | Ruta de acceso al monitor al que está asociada la entrada. Por ejemplo, **monitor/mi monitor**. |
| is-full-object | Indica que las propiedades contienen opciones para el recurso de monitor. |
| properties | Cadena con formato JSON que contiene las propiedades del recurso de monitor. Se puede pasar como una cadena o un archivo. |

### <a name="input-examples"></a>Ejemplos de entrada

Ejemplo de JSON de configuración para crear un recurso de entrada en la región *Este de EE. UU.* que está conectada a \<Blockchain member\>.

``` json
{
    "location": "eastus",
    "properties": {
        "inputType": "Ethereum",
        "dataSource": {
            "resourceId": "/subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/blockchainMembers/<Blockchain member>/transactionNodes/transaction-node"
        }
    }
}
```

| Elemento | Descripción |
|---------|-------------|
| ubicación | Región en la que se va a crear el recurso de entrada. |
| inputType | Tipo de libro de contabilidad del miembro Azure Blockchain Service. Actualmente, se admite **Ethereum**. |
| resourceId | Nodo de transacción al que está conectada la entrada. Reemplace \<Subscription ID\>, \<Resource group\> y \<Blockchain member\> con los valores del recurso del nodo de transacción. La entrada se conecta al nodo de transacción predeterminado para el miembro Azure Blockchain Service. |

Cree una entrada denominada *myinput* para *mywatcher* con la cadena JSON para la configuración.

``` azurecli-interactive
az resource create \
                   --resource-group myRG \
                   --name myInput \
                   --namespace Microsoft.Blockchain \
                   --resource-type inputs \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties '{"location":"eastus", "properties":{"inputType":"Ethereum","dataSource":{"resourceId":"/subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/BlockchainMembers/<Blockchain member>/transactionNodes/transaction-node"}}}'
```

Cree una entrada denominada *myinput* para *mywatcher* con un archivo de configuración JSON.

``` azurecli
az resource create \
                   --resource-group myRG \
                   --name input \
                   --namespace Microsoft.Blockchain \ --resource-type inputs \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties @input.json
```

## <a name="create-output"></a>Crear salida

Una conexión saliente envía datos de la cadena de bloques a Azure Event Grid. Puede enviar datos de la cadena de bloques a un único destino o enviar datos de la cadena de bloques a varios destinos. La cadena de bloques Data Manager admite varias conexiones salientes de tema Event Grid para cualquier instancia de cadena de bloques Data Manager determinada.

``` azurecli
az resource create \
                   --resource-group <Resource group> \
                   --name <Output name> \
                   --namespace Microsoft.Blockchain \
                   --resource-type outputs \
                   --parent watchers/<Watcher name> \
                   --is-full-object \
                   --properties <output resource properties>
```

| Parámetro | Descripción |
|-----------|-------------|
| resource-group | Nombre del grupo de recursos donde se creará el recurso de entrada. |
| name | Nombre de la salida. |
| espacio de nombres | Use el espacio de nombres del proveedor **Microsoft.Blockchain**. |
| tipo de recurso | El tipo de recurso para una salida de la cadena de bloques Data Manager es **salidas**. |
| primario | Ruta de acceso al monitor al que está asociada la salida. Por ejemplo, **monitor/mi monitor**. |
| is-full-object | Indica que las propiedades contienen opciones para el recurso de salida. |
| properties | Cadena con formato JSON que contiene las propiedades del recurso de monitor. Se puede pasar como una cadena o un archivo. |

### <a name="output-examples"></a>Ejemplos de salida

Ejemplo de JSON de configuración para crear un recurso de salida en la región *Este de EE. UU.* que está conectada a un tema de Event Grid denominado \<event grid topic\>.

``` json
{
    "location": "eastus",
    "properties": {
        "outputType": "EventGrid",
        "dataSource": {
            "resourceId": "/subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.EventGrid/topics/<event grid topic>"
        }
    }
}
```

| Elemento | Descripción |
|---------|-------------|
| ubicación | Región en la que se va a crear el recurso de salida. |
| outputType | Tipo de salida. Actualmente, se admite **EventGrid**. |
| resourceId | Recurso al que está conectada la salida. Reemplace \<Subscription ID\>, \<Resource group\> y \<Blockchain member\> con los valores del recurso de Event Grid. |

Cree una salida con el nombre de*myoutput* para *mywatcher* que se conecta a un tema de Event Grid con una cadena de configuración de JSON.

``` azurecli-interactive
az resource create \
                   --resource-group myRG \
                   --name myoutput \
                   --namespace Microsoft.Blockchain \
                   --resource-type outputs \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties '{"location":"eastus","properties":{"outputType":"EventGrid","dataSource":{"resourceId":"/subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.EventGrid/topics/<event grid topic>"}}}'
```

Cree una salida con el nombre de *myoutput* para *mywatcher* que se conecta a un tema de Event Grid con un archivo de configuración JSON.

``` azurecli
az resource create \
                   --resource-group myRG \
                   --name myoutput \
                   --namespace Microsoft.Blockchain \
                   --resource-type outputs \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties @output.json
```

## <a name="add-blockchain-application"></a>Agregar aplicación de cadena de bloques

Si agrega una aplicación de cadena de bloques, la cadena de bloques Data Manager descodifica el estado de los eventos y las propiedades de la aplicación. De lo contrario, solo se envían datos de transacciones y bloques sin procesar. La cadena de bloques Data Manager también detecta direcciones de contrato cuando se implementa el contrato. Puede agregar varias aplicaciones de la cadena de bloques a una instancia de cadena de bloques Data Manager.


> [!IMPORTANT]
> Actualmente, las aplicaciones de cadena de bloques que declaran [tipos de matriz ](https://solidity.readthedocs.io/en/v0.5.12/types.html#arrays) o [tipos de asignación](https://solidity.readthedocs.io/en/v0.5.12/types.html#mapping-types) de Solidity no son totalmente compatibles. Las propiedades declaradas como matrices o tipos de asignación no se descodificarán en mensajes *ContractPropertiesMsg* ni *DecodedContractEventsMsg*.

``` azurecli
az resource create \
                   --resource-group <Resource group> \
                   --name <Application name> \
                   --namespace Microsoft.Blockchain \
                   --resource-type artifacts \
                   --parent watchers/<Watcher name> \
                   --is-full-object \
                   --properties <Application resource properties>
```

| Parámetro | Descripción |
|-----------|-------------|
| resource-group | Nombre del grupo de recursos donde se creará el recurso de entrada. |
| name | Nombre de la aplicación. |
| espacio de nombres | Use el espacio de nombres del proveedor **Microsoft.Blockchain**. |
| tipo de recurso | El tipo de recurso de una aplicación de cadena de bloques Data Manager es **artefactos**. |
| primario | Ruta de acceso al monitor al que está asociada la aplicación. Por ejemplo, **monitor/mi monitor**. |
| is-full-object | Indica que las propiedades contienen opciones para el recurso de la aplicación. |
| properties | Cadena con formato JSON que contiene las propiedades del recurso de la aplicación. Se puede pasar como una cadena o un archivo. |

### <a name="blockchain-application-examples"></a>Ejemplos de aplicaciones de Blockchain

Ejemplo de JSON de configuración para crear un recurso de aplicación en la región del *este de EE. UU.* que supervisa un contrato inteligente definido por la ABI del contrato y el código de bytes.

``` json
{
    "location": "eastus",
    "properties": {
        "artifactType": "EthereumSmartContract",
        "content": {
            "abiFileUrl": "<ABI URL>",
            "bytecodeFileUrl": "<Bytecode URL>",
            "queryTargetTypes": [
                "ContractProperties",
                "ContractEvents"
            ]
        }
    }
}
```

| Elemento | Descripción |
|---------|-------------|
| ubicación | Región en la que se va a crear el recurso de aplicación. |
| artifactType | Tipo de aplicación. Actualmente, se admite **EthereumSmartContract**. |
| abiFileUrl | Dirección URL para el archivo JSON de la ABI del contrato inteligente. Para más información sobre cómo obtener la ABI de contrato y crear una dirección URL, vea [obtener contrato ABI y código de bytes](data-manager-portal.md#get-contract-abi-and-bytecode) y [crear contrato de la dirección URL de la ABI y el código de bytes](data-manager-portal.md#create-contract-abi-and-bytecode-url). |
| bytecodeFileUrl | Dirección URL del archivo JSON del código de bytes implementado del contrato inteligente. Para más información sobre cómo obtener el código de bytes implementado del contrato inteligente y cómo crear una dirección URL, consulte [Obtener la ABI del contrato y el código de bytes](data-manager-portal.md#get-contract-abi-and-bytecode) y [Obtener la ABI del contrato y una dirección URL del código de bytes](data-manager-portal.md#create-contract-abi-and-bytecode-url). Nota: Blockchain Data Manager requiere el **código de bytes implementado**. |
| queryTargetTypes | Tipos de mensajes publicados. La especificación de **ContractProperties** publica el tipo de mensaje  *ContractPropertiesMsg*. La especificación de **ContractEvents** publica el tipo de mensaje  *DecodedContractEventsMsg*. Nota: Siempre se publican los tipo de mensaje *RawBlockAndTransactionMsg* y *RawTransactionContractCreationMsg*. |

Cree una aplicación denominada *myApplication* para *mywatcher* de los monitores de un contrato inteligente definido por una cadena JSON.

``` azurecli-interactive
az resource create \
                   --resource-group myRG \
                   --name myApplication \
                   --namespace Microsoft.Blockchain \
                   --resource-type artifacts \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties '{"location":"eastus","properties":{"artifactType":"EthereumSmartContract","content":{"abiFileUrl":"<ABI URL>","bytecodeFileUrl":"<Bytecode URL>","queryTargetTypes":["ContractProperties","ContractEvents"]}}}'
```

Cree una aplicación denominada *myApplication* para *mywatcher* de los monitores de un contrato inteligente definido por archivo de configuración JSON.

``` azurecli
az resource create \
                   --resource-group myRG \
                   --name myApplication \
                   --namespace Microsoft.Blockchain \
                   --resource-type artifacts \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties @artifact.json
```

## <a name="start-instance"></a>Iniciar instancia

Al ejecutar, una instancia del administrador de Blockchain supervisa los eventos de la cadena de bloques de las entradas definidas y envía los datos a las salidas definidas.

``` azurecli
az resource invoke-action \
                          --action start \
                          --ids /subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/watchers/<Watcher name>
```

| Parámetro | Descripción |
|-----------|-------------|
| action | Use **iniciar** para ejecutar el monitor. |
| ids | Identificador de recurso de monitor. Reemplace \<Subscription ID\>, \<Resource group\> y \<Watcher name\> con los valores del recurso de monitor.|

### <a name="start-instance-example"></a>Ejemplo de instancia de inicio

Inicie una instancia de cadena de bloques Data Manager denominada *mywatcher*.

``` azurecli-interactive
az resource invoke-action \
                          --action start \
                          --ids /subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/watchers/mywatcher
```

## <a name="stop-instance"></a>Detener instancia

Detenga una instancia de cadena de bloques Data Manager.

``` azurecli
az resource invoke-action \
                          --action stop \
                          --ids /subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/watchers/<Watcher name>
```

| Parámetro | Descripción |
|-----------|-------------|
| action | Use **detener** para detener el monitor. |
| ids | Nombre del monitor. Reemplace \<Subscription ID\>, \<Resource group\> y \<Watcher name\> con los valores del recurso de monitor. |

### <a name="stop-watcher-example"></a>Ejemplo de monitor de detención

Detenga una instancia denominada *mywatcher*.

``` azurecli-interactive
az resource invoke-action \
                          --action stop \
                          --ids /subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/watchers/mywatcher
```

## <a name="delete-instance"></a>Eliminar instancia

Elimine una instancia de cadena de bloques Data Manager.

``` azurecli
az resource delete \
                   --resource-group <Resource group> \
                   --name <Watcher name> \
                   --resource-type Microsoft.Blockchain/watchers
```

| Parámetro | Descripción |
|-----------|-------------|
| resource-group | Nombre del grupo de recursos del monitor que se va a eliminar. |
| name | Nombre del monitor que se va a eliminar. |
| tipo de recurso | El tipo de recurso de un monitor de cadena de bloques Data Manager es **Microsoft.blockchain/watchers**. |

### <a name="delete-instance-example"></a>Ejemplo de eliminación de instancia

Elimine una instancia denominada *mywatcher* en el grupo de recursos *myRG*.

``` azurecli-interactive
az resource delete \
                   --resource-group myRG \
                   --name mywatcher \
                   --resource-type Microsoft.blockchain/watchers
```

## <a name="next-steps"></a>Pasos siguientes

Intente crear con el siguiente tutorial un explorador de mensajes de transacción de cadena de bloques con Blockchain Data Manager y Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Uso de Blockchain Data Manager para enviar datos a Azure Cosmos DB](data-manager-cosmosdb.md)
