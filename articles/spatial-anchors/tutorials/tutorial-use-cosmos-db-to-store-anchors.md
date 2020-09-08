---
title: 'Tutorial: Uso compartido de delimitadores con Azure Cosmos DB'
description: En este tutorial, aprenderá a compartir identificadores de Azure Spatial Anchors entre los dispositivos Android o iOS de Unity con un servicio de back-end y Azure Cosmos DB.
author: ramonarguelles
manager: vriveras
services: azure-spatial-anchors
ms.author: rgarcia
ms.date: 07/31/2020
ms.topic: tutorial
ms.service: azure-spatial-anchors
ms.openlocfilehash: 8ff431c27dd53e82f9003b658c82ceb3efb5d320
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87810027"
---
# <a name="tutorial-sharing-azure-spatial-anchors-across-sessions-and-devices-with-an-azure-cosmos-db-back-end"></a>Tutorial: Uso compartido de Azure Spatial Anchors entre sesiones y dispositivos con un back-end de Azure Cosmos DB

Este tutorial es una continuación del tutorial para [compartir anclajes espaciales de Azure entre sesiones y dispositivos](../../../articles/spatial-anchors/tutorials/tutorial-share-anchors-across-devices.md). Le guiará en el proceso de agregar algunas funcionalidades más para que Azure Cosmos DB sirva como almacenamiento de back-end al mismo tiempo que se comparten los delimitadores espaciales de Azure entre las sesiones y los dispositivos.

![GIF que ilustra la persistencia de objetos](./media/persistence.gif)

Es importante destacar que, aunque en este tutorial se van a usar Unity y Azure Cosmos DB, es solo para proporcionar un ejemplo de cómo compartir los identificadores de Spatial Anchors entre dispositivos. Se pueden usar otros lenguajes y tecnologías de back-end para lograr el mismo objetivo.

## <a name="create-a-database-account"></a>Creación de una cuenta de base de datos

Agregue una base de datos de Azure Cosmos al grupo de recursos que creó anteriormente.

[!INCLUDE [cosmos-db-create-dbaccount-table](../../../includes/cosmos-db-create-dbaccount-table.md)]

Copie `Connection String`, ya que lo va a necesitar.

## <a name="make-minor-changes-to-the-sharingservice-files"></a>Realización de cambios menores en los archivos SharingService

En el **Explorador de soluciones**, abra `SharingService\Startup.cs`.

Busque `#define INMEMORY_DEMO` al principio del archivo y marque la línea como comentario. Guarde el archivo.

En el **Explorador de soluciones**, abra `SharingService\appsettings.json`.

Busque la propiedad `StorageConnectionString` y establezca que el valor sea el mismo que el de `Connection String`, que copió en el [paso de creación de la cuenta de base de datos](#create-a-database-account). Guarde el archivo.

Puede publicar el servicio de uso compartido y ejecutar la aplicación de ejemplo.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha usado Azure Cosmos DB para compartir identificadores de delimitador en todos los dispositivos. Para más información sobre cómo usar Azure Spatial Anchors en una nueva aplicación de Unity HoloLens, continúe con el siguiente tutorial.

> [!div class="nextstepaction"]
> [Inicio de una nueva aplicación de Unity HoloLens](./tutorial-new-unity-hololens-app.md)
