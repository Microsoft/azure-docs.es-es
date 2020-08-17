---
title: 'Inicio rápido: Conexión de una aplicación de MongoDB en Node.js a Azure Cosmos DB'
description: En este inicio rápido se muestra cómo conectar una aplicación de MongoDB existente escrita en Node.js a Azure Cosmos DB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 05/21/2019
ms.custom: seo-javascript-september2019, seo-javascript-october2019, devx-track-javascript
ms.openlocfilehash: 1603e2c0d76415b612e81f88990805b0bb1de732
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423684"
---
# <a name="quickstart-migrate-an-existing-mongodb-nodejs-web-app-to-azure-cosmos-db"></a>Inicio rápido: Migración de una aplicación web actual Node.js de MongoDB a Azure Cosmos DB 

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](create-mongodb-flask.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

En este inicio rápido, se crea y administra una cuenta de Azure Cosmos DB para MongoDB API mediante Azure Cloud Shell, y con una aplicación MEAN (MongoDB, Express, Angular y Node.js) clonada desde GitHub. Azure Cosmos DB es un servicio de base de datos multimodelo que permite crear y consultar rápidamente bases de datos de documentos, tablas, claves-valores y grafos con funcionalidades de distribución global y escala horizontal.

## <a name="prerequisites"></a>Requisitos previos
- Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). O bien, [pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sin una suscripción de Azure. También puede usar el [emulador de Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) con la cadena de conexión `.mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true`.
- [Node.js](https://nodejs.org/) y conocimientos para trabajar con Node.js.
- [Git](https://git-scm.com/downloads).
- Si no desea usar Azure Cloud Shell, use la [CLI de Azure 2.0+](/cli/azure/install-azure-cli).

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Ejecute los comandos siguientes para clonar el repositorio de ejemplo. Este repositorio de ejemplo contiene una aplicación [MEAN.js](https://meanjs.org/) predeterminada.

1. Abra un símbolo del sistema, cree una carpeta nueva denominada ejemplos de GIT y, después, cierre el símbolo del sistema.

    ```bash
    mkdir "C:\git-samples"
    ```

2. Abra una ventana de terminal de Git, como git bash y utilice el comando `cd` para cambiar a la nueva carpeta para instalar la aplicación de ejemplo.

    ```bash
    cd "C:\git-samples"
    ```

3. Ejecute el comando siguiente para clonar el repositorio de ejemplo. Este comando crea una copia de la aplicación de ejemplo en el equipo. 

    ```bash
    git clone https://github.com/prashanthmadi/mean
    ```

## <a name="run-the-application"></a>Ejecución de la aplicación

Esta aplicación de MongoDB escrita en Node.js se conecta a su base de datos de Azure Cosmos DB, que admite el cliente de MongoDB. En otras palabras, para la aplicación resulta transparente que los datos se almacenan en una base de datos de Azure Cosmos DB.

Instale los paquetes necesarios e inicie la aplicación.

```bash
cd mean
npm install
npm start
```
La aplicación intentará conectarse a un origen de MongoDB y producirá un error. Siga adelante y salga de la aplicación cuando la salida devuelva "[MongoError: connect ECONNREFUSED 127.0.0.1:27017]".

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Si decide instalar y usar la CLI localmente, para este tema es preciso que ejecute la CLI de Azure versión 2.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]. 

Si usa una CLI de Azure instalada, inicie sesión en la suscripción de Azure con el comando [az login](/cli/azure/reference-index#az-login) y siga las instrucciones de la pantalla. Puede omitir este paso si usa Azure Cloud Shell.

```azurecli
az login 
``` 
   
## <a name="add-the-azure-cosmos-db-module"></a>Agregar el módulo de Azure Cosmos DB

Si va a usar una CLI de Azure instalada, ejecute el comando `az` para comprobar si el componente `cosmosdb` ya está instalado. Si `cosmosdb` está en la lista de comandos de referencia, continúe con el comando siguiente. Puede omitir este paso si usa Azure Cloud Shell.

Si `cosmosdb` no está en la lista de comandos de referencia, vuelva a instalar la [CLI de Azure](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un [grupo de recursos](../azure-resource-manager/management/overview.md) con el comando [az group create](/cli/azure/group#az-group-create). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran recursos de Azure como aplicaciones web, bases de datos y cuentas de almacenamiento. 

En el ejemplo siguiente se crea un grupo de recursos en la región de Oeste de Europa. Elija un nombre único para el grupo de recursos.

Si usa Azure Cloud Shell, seleccione **Probarlo**, siga las indicaciones de la pantalla para iniciar sesión y, a continuación, copie el comando en el símbolo del sistema.

```azurecli-interactive
az group create --name myResourceGroup --location "West Europe"
```

## <a name="create-an-azure-cosmos-db-account"></a>Creación de una cuenta de Azure Cosmos DB

Cree una cuenta de Cosmos con el comando [az cosmosdb create](/cli/azure/cosmosdb#az-cosmosdb-create).

En el comando siguiente, sustituya su propio nombre único de la cuenta de Cosmos donde vea el marcador de posición `<cosmosdb-name>`. Este nombre único se usará como parte del punto de conexión de Cosmos DB (`https://<cosmosdb-name>.documents.azure.com/`), por lo que debe ser único entre todas las cuentas de Cosmos de Azure. 

```azurecli-interactive
az cosmosdb create --name <cosmosdb-name> --resource-group myResourceGroup --kind MongoDB
```

El parámetro `--kind MongoDB` habilita las conexiones de cliente de MongoDB.

Cuando se crea la cuenta de Azure Cosmos DB, la CLI de Azure muestra información similar a la del ejemplo siguiente. 

> [!NOTE]
> Este ejemplo usa JSON como el formato de salida de la CLI de Azure, que es el valor predeterminado. Para usar otro formato de salida, consulte [Formatos de salida para los comandos de la CLI de Azure](https://docs.microsoft.com/cli/azure/format-output-azure-cli).

```json
{
  "databaseAccountOfferType": "Standard",
  "documentEndpoint": "https://<cosmosdb-name>.documents.azure.com:443/",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Document
DB/databaseAccounts/<cosmosdb-name>",
  "kind": "MongoDB",
  "location": "West Europe",
  "name": "<cosmosdb-name>",
  "readLocations": [
    {
      "documentEndpoint": "https://<cosmosdb-name>-westeurope.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "<cosmosdb-name>-westeurope",
      "locationName": "West Europe",
      "provisioningState": "Succeeded"
    }
  ],
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.DocumentDB/databaseAccounts",
  "writeLocations": [
    {
      "documentEndpoint": "https://<cosmosdb-name>-westeurope.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "<cosmosdb-name>-westeurope",
      "locationName": "West Europe",
      "provisioningState": "Succeeded"
    }
  ]
} 
```

## <a name="connect-your-nodejs-application-to-the-database"></a>Conexión de la aplicación Node.js a la base de datos

En este paso, se conecta la aplicación de ejemplo MEAN.js a la base de datos de Azure Cosmos DB que acaba de crear. 

<a name="devconfig"></a>
## <a name="configure-the-connection-string-in-your-nodejs-application"></a>Configuración de la cadena de conexión en la aplicación Node.js

En el repositorio de MEAN.js, abra `config/env/local-development.js`.

Reemplace el contenido de este archivo por el código siguiente. No olvide reemplazar los dos marcadores de posición `<cosmosdb-name>` por el nombre de la cuenta de Cosmos.

```javascript
'use strict';

module.exports = {
  db: {
    uri: 'mongodb://<cosmosdb-name>:<primary_master_key>@<cosmosdb-name>.documents.azure.com:10255/mean-dev?ssl=true&sslverifycertificate=false'
  }
};
```

## <a name="retrieve-the-key"></a>Recuperar la clave

Para conectarse a una base de datos de Cosmos, necesita la clave de base de datos. Use el comando [az cosmosdb keys list](/cli/azure/cosmosdb/keys#az-cosmosdb-keys-list) para recuperar la clave principal.

```azurecli-interactive
az cosmosdb keys list --name <cosmosdb-name> --resource-group myResourceGroup --query "primaryMasterKey"
```

La CLI de Azure genera información similar a la del ejemplo siguiente. 

```json
"RUayjYjixJDWG5xTqIiXjC..."
```

Copie el valor de `primaryMasterKey`. Péguelo en la `<primary_master_key>` en `local-development.js`.

Guarde los cambios.

### <a name="run-the-application-again"></a>Vuelva a ejecutar la aplicación.

Vuelva a ejecutar `npm start`. 

```bash
npm start
```

Un mensaje de consola debería indicarle que el entorno de desarrollo está en funcionamiento. 

Vaya a `http://localhost:3000` en un explorador. Seleccione **Registrarse** en el menú superior e intente crear dos usuarios ficticios. 

La aplicación de ejemplo MEAN.js almacena datos de usuario en la base de datos. Si lo logra y MEAN.js inicia la sesión automáticamente del usuario creado, significa que la conexión de Azure Cosmos DB funciona. 

:::image type="content" source="./media/create-mongodb-nodejs/mongodb-connect-success.png" alt-text="MEAN.js se conecta correctamente a MongoDB":::

## <a name="view-data-in-data-explorer"></a>Ver datos en el Explorador de datos

Los datos almacenados en una base de datos de Cosmos se pueden ver y consultar en Azure Portal.

Para ver y consultar los datos de usuario creados en el paso anterior y trabajar con ellos, inicie sesión en [Azure Portal](https://portal.azure.com) en el explorador web.

En el cuadro de búsqueda superior, escriba **Azure Cosmos DB**. Cuando se abra la hoja de la cuenta de Cosmos, seleccione su cuenta de Cosmos. En el panel de navegación izquierdo, seleccione **Explorador de datos**. Expanda la colección en el panel Colecciones. Ahora puede ver los documentos de la colección, consultar los datos e incluso crear y ejecutar procedimientos almacenados, desencadenadores y UDF. 

:::image type="content" source="./media/create-mongodb-nodejs/cosmosdb-connect-mongodb-data-explorer.png" alt-text="Explorador de datos en Azure Portal":::


## <a name="deploy-the-nodejs-application-to-azure"></a>Implementación de la aplicación Node.js en Azure

En este paso, implementará la aplicación Node.js en Cosmos DB.

Probablemente habrá observado que el archivo de configuración que cambió anteriormente está pensado para el entorno de desarrollo (`/config/env/local-development.js`). Al implementar la aplicación en App Service, se ejecutará en el entorno de producción de forma predeterminada. Por esta razón, ahora debe realizar el mismo cambio en el archivo de configuración correspondiente.

En el repositorio de MEAN.js, abra `config/env/production.js`.

En el objeto `db`, reemplace el valor de `uri` como se muestra en el ejemplo siguiente. No olvide reemplazar los marcadores de posición como hizo antes.

```javascript
'mongodb://<cosmosdb-name>:<primary_master_key>@<cosmosdb-name>.documents.azure.com:10255/mean?ssl=true&sslverifycertificate=false',
```

> [!NOTE] 
> La opción `ssl=true` es importante debido a los requisitos de Cosmos DB. Para más información, consulte [Requisitos de la cadena de conexión](connect-mongodb-account.md#connection-string-requirements).
>
>

En el terminal, confirme todos los cambios en Git. Puede copiar ambos comandos para ejecutarlos juntos.

```bash
git add .
git commit -m "configured MongoDB connection string"
```
## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una cuenta de Azure Cosmos DB para MongoDB API mediante Azure Cloud Shell, y a crear y ejecutar una aplicación MEAN.js para agregar usuarios a la cuenta. Ahora puede importar datos adicionales en la cuenta de Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Importación de datos de MongoDB a Azure Cosmos DB](mongodb-migrate.md)
