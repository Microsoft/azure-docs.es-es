---
title: Creación de una aplicación Angular con la API de Azure Cosmos DB para MongoDB (primera parte)
description: Parte 4 de la serie de tutoriales en la que se explica cómo crear una aplicación de MongoDB con Angular y Node en Azure Cosmos DB utilizando las mismas API que para MongoDB
author: johnpapa
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 12/06/2018
ms.author: jopapa
ms.custom: seodec18, devx-track-javascript
ms.reviewer: sngun
ms.openlocfilehash: e4127024404489d0aa8478a9cbd073773f8b512a
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423939"
---
# <a name="create-an-angular-app-with-azure-cosmos-dbs-api-for-mongodb---create-a-cosmos-account"></a>Creación de una aplicación de Angular con la API de Azure Cosmos DB para MongoDB: Creación de una cuenta de Cosmos

En este tutorial de varias partes se muestra cómo crear una nueva aplicación escrita en Node.js con Express y Angular y cómo conectarla a la [cuenta de Cosmos configurada con la API de Cosmos DB para MongoDB](mongodb-introduction.md).

La parte 4 del tutorial se basa en la [parte 3](tutorial-develop-mongodb-nodejs-part3.md) y aborda las tareas siguientes:

> [!div class="checklist"]
> * Creación de un grupo de recursos de Azure con la CLI de Azure
> * Creación de una cuenta de Cosmos con la CLI de Azure

## <a name="video-walkthrough"></a>Tutorial en vídeo

> [!VIDEO https://www.youtube.com/embed/hfUM-AbOh94]

## <a name="prerequisites"></a>Requisitos previos

Antes de iniciar esta parte del tutorial, asegúrese de que ha completado los pasos descritos en la [parte 3](tutorial-develop-mongodb-nodejs-part3.md). 

En esta sección del tutorial, puede usar Azure Cloud Shell (en el explorador de Internet) o la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli) instalada localmente.

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

[!INCLUDE [Log in to Azure](../../includes/login-to-azure.md)]

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group.md)]

> [!TIP]
> Este tutorial le guía por los pasos necesarios para crear la aplicación paso a paso. Si desea descargar el proyecto finalizado, puede obtener la aplicación completa en el [repositorio de angular-cosmosdb](https://github.com/Azure-Samples/angular-cosmosdb) en GitHub.

## <a name="create-an-azure-cosmos-db-account"></a>Creación de una cuenta de Azure Cosmos DB

Creación de una cuenta de Azure Cosmos DB con el comando [`az cosmosdb create`](/cli/azure/cosmosdb#az-cosmosdb-create).

```azurecli-interactive
az cosmosdb create --name <cosmosdb-name> --resource-group myResourceGroup --kind MongoDB
```

* Para que `<cosmosdb-name>` use su propio nombre único de cuenta de Azure Cosmos DB, el nombre debe ser único en todos los nombres de cuenta de Azure Cosmos DB en Azure.
* La opción `--kind MongoDB` habilita la configuración de Azure Cosmos DB para tener conexiones de cliente de MongoDB.

Puede tardar un minuto o dos en completarse el comando. Cuando termina, la ventana de terminal muestra información sobre la nueva base de datos. 

Una vez creada la cuenta de Azure Cosmos DB:
1. Abra una nueva ventana del explorador y vaya a [https://portal.azure.com](https://portal.azure.com)
1. Haga clic en el logotipo :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part4/azure-cosmos-db-icon.png"::: de Azure Cosmos DB de la barra de la izquierda y se mostrarán todas las instancias que tenga.
1. Haga clic en la cuenta de Azure Cosmos DB recién creada, seleccione la pestaña **Introducción** y desplácese hacia abajo para ver el mapa donde se encuentra la base de datos. 

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part4/azure-cosmos-db-angular-portal.png" alt-text="Nueva cuenta de Azure Cosmos DB en Azure Portal":::

4. Desplácese hacia abajo en el panel de navegación izquierdo y haga clic en la pestaña **Replicar datos globalmente**. Se mostrará un mapa donde puede ver las distintas áreas en las que se puede replicar. Por ejemplo, puede hacer clic en Sudeste de Australia o en Este de Australia y replicar los datos en Australia. Puede aprender más acerca de la replicación global en [Cómo se distribuyen datos globalmente con Azure Cosmos DB](distribute-data-globally.md). Por ahora, solo vamos a mantener la otra instancia y, cuando deseemos replicarla, ya sabemos cómo.

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part4/azure-cosmos-db-replicate-portal.png" alt-text="Nueva cuenta de Azure Cosmos DB en Azure Portal":::

## <a name="next-steps"></a>Pasos siguientes

En esta parte del tutorial, ha hecho lo siguiente:

> [!div class="checklist"]
> * Creó un grupo de recursos de Azure con la CLI de Azure
> * Creó una cuenta de Azure Cosmos DB mediante la CLI de Azure

Puede continuar con la siguiente parte del tutorial para conectar Azure Cosmos DB a su aplicación mediante Mongoose.

> [!div class="nextstepaction"]
> [Uso de Mongoose para conectarse a Azure Cosmos DB](tutorial-develop-mongodb-nodejs-part5.md)
