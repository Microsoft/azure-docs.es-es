---
title: 'Tutorial: Uso compartido de delimitadores entre sesiones y dispositivos'
description: En este tutorial, obtendrá información sobre cómo compartir los identificadores espaciales de Azure Spatial Anchors entre dispositivos Android/iOS de Unity con un servicio de back-end.
author: ramonarguelles
manager: vriveras
services: azure-spatial-anchors
ms.author: rgarcia
ms.date: 07/31/2020
ms.topic: tutorial
ms.service: azure-spatial-anchors
ms.openlocfilehash: 12a15a2d0cd45207b4adefc7315e48c6d642c0f7
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87810192"
---
# <a name="tutorial-share-azure-spatial-anchors-across-sessions-and-devices"></a>Tutorial: Uso compartido de Azure Spatial Anchors entre sesiones y dispositivos

En este tutorial, obtendrá información sobre cómo usar [Azure Spatial Anchors](../overview.md) para crear delimitadores durante una sesión y, a continuación, buscarlos, en el mismo dispositivo o en uno diferente. Estos mismos delimitadores podrían también ser encontrados por varios dispositivos en el mismo lugar y al mismo tiempo.

![Persistencia](./media/persistence.gif)

Azure Spatial Anchors es un servicio multiplataforma para desarrolladores que le permite crear experiencias de realidad mixta mediante objetos cuya ubicación persiste en todos los dispositivos a lo largo del tiempo. Cuando haya terminado, tendrá una aplicación que se puede implementar en dos o más dispositivos. Los delimitadores espaciales creados por una instancia de Azure Spatial Anchors se pueden compartir con las demás.

Aprenderá a:

> [!div class="checklist"]
> * Implementar una aplicación web de ASP.NET Core en Azure que se pueda usar para compartir los delimitadores y almacenarlos en memoria durante un tiempo.
> * Configurar la escena AzureSpatialAnchorsLocalSharedDemo dentro del ejemplo de Unity de nuestros inicios rápidos para aprovechar las ventajas del uso compartido de la aplicación web de delimitadores.
> * Realizar la implementación y ejecución en uno o varios dispositivos.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [Share Anchors Sample Prerequisites](../../../includes/spatial-anchors-share-sample-prereqs.md)]

Es importante destacar que, aunque se va a usar Unity y una aplicación web de ASP.NET Core en este tutorial, es solo para mostrar un ejemplo sobre cómo compartir los identificadores espaciales de Azure Spatial Anchors con otros dispositivos. Se pueden usar otros lenguajes y tecnologías de back-end para lograr el mismo objetivo.

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="download-the-sample-project"></a>Descarga del proyecto de ejemplo

[!INCLUDE [Clone Sample Repo](../../../includes/spatial-anchors-clone-sample-repository.md)]

## <a name="deploy-your-sharing-anchors-service"></a>Implementación del uso compartido del servicio de delimitadores

## <a name="visual-studio"></a>[Visual Studio](#tab/VS)

Abra Visual Studio y abra el proyecto en la carpeta `Sharing\SharingServiceSample`.

[!INCLUDE [Publish Azure](../../../includes/spatial-anchors-publish-azure.md)]

## <a name="visual-studio-code"></a>[Visual Studio Code](#tab/VSC)

Tendrá que crear un grupo de recursos y un plan de App Service antes de implementar el servicio en VS Code.

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Vaya a <a href="https://portal.azure.com/" target="_blank">Azure Portal</a> e inicie sesión en la suscripción de Azure.

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

[!INCLUDE [resource group intro text](../../../includes/resource-group.md)]

Junto a **Grupo de recursos**, seleccione **Nuevo**.

Asigne el nombre **myResourceGroup** al grupo de recursos y seleccione **Aceptar**.

### <a name="create-an-app-service-plan"></a>Creación de un plan de App Service

[!INCLUDE [app-service-plan](../../../includes/app-service-plan.md)]

Junto a **Plan de hospedaje**, seleccione **Nuevo**.

En el cuadro de diálogo **Configurar un plan de hospedaje**, use estos valores:

| Configuración | Valor sugerido | Descripción |
|-|-|-|
|Plan de App Service| MySharingServicePlan | Nombre del plan de App Service. |
| Location | Oeste de EE. UU. | El centro de datos donde se hospeda la aplicación web. |
| Size | Gratuito | El [plan de tarifa](https://azure.microsoft.com/pricing/details/app-service/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) que determina las características de hospedaje. |

Seleccione **Aceptar**.

Abra Visual Studio Code y abra el proyecto en la carpeta `Sharing\SharingServiceSample`. Siga <a href="https://docs.microsoft.com/aspnet/core/tutorials/publish-to-azure-webapp-using-vscode?view=aspnetcore-2.2#open-it-with-visual-studio-code" target="_blank">este tutorial</a> para implementar el servicio de uso compartido mediante Visual Studio Code. Puede seguir los pasos a partir de la sección "Abrirlo con Visual Studio Code". No cree otro proyecto de ASP.NET, tal como se explicó en el paso anterior, porque ya tiene el proyecto que debe implementarse y publicarse: SharingServiceSample.

---

## <a name="deploy-the-sample-app"></a>Implementación de la aplicación de ejemplo

[!INCLUDE [Run Share Anchors Sample](../../../includes/spatial-anchors-run-share-sample.md)]

[!INCLUDE [Clean-up section](../../../includes/clean-up-section-portal.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha implementado una aplicación web de ASP.NET Core en Azure y ha configurado e implementado una aplicación de Unity. Ha creado delimitadores espaciales con la aplicación y los ha compartido con otros dispositivos mediante la aplicación web de ASP.NET Core.

Puede mejorar una aplicación web de ASP.NET Core para que use Azure Cosmos DB a fin de hacer persistir el almacenamiento de los delimitadores de anclaje espacial compartidos. Al agregar compatibilidad con Azure Cosmos DB, permitirá que la aplicación web de ASP.NET Core cree un delimitador hoy y, cuando regrese días más tarde, pueda encontrarlo de nuevo mediante el identificador de delimitador almacenado en la aplicación web.

> [!div class="nextstepaction"]
> [Uso de Azure Cosmos DB para almacenar delimitadores](./tutorial-use-cosmos-db-to-store-anchors.md)

