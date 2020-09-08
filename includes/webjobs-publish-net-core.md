---
title: archivo de inclusión
description: archivo de inclusión
services: app-service
author: ggailey777
ms.service: app-service
ms.topic: include
ms.date: 06/26/2020
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: e208b52c67f173bd0d289715b63562df656b1ec9
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009810"
---
1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Publicar**.

1. En el cuadro de diálogo **Publicar**, seleccione **Azure** en **Destino** y, luego, **Siguiente**. 

1. Seleccione **Azure WebJobs** en **Specific target** (Destino específico) y, luego, elija **Siguiente**.

1. Seleccione **Create a new Azure WebJob** (Crear Azure WebJob).

   ![Selección del destino de publicación](./media/webjobs-publish-netcore/pick-publish-target.png)

1. En el cuadro de diálogo **App Service (Windows)** , use la configuración de hospedaje de la tabla siguiente.

    | Configuración      | Valor sugerido  | Descripción                                |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Nombre** | Nombre único globalmente | Nombre que identifica de forma única la nueva aplicación de función. |
    | **Suscripción** | Elija una suscripción | La suscripción de Azure que se va a usar. |
    | **[Grupo de recursos](../articles/azure-resource-manager/management/overview.md)** | myResourceGroup |  Nombre del grupo de recursos en el que se va a crear la instancia de Function App. Elija **Nuevo** para crear un grupo de recursos nuevo.|
    | **[Plan de hospedaje](../articles/app-service/overview-hosting-plans.md)** | Plan de App Service | Un [plan de App Service](../articles/app-service/overview-hosting-plans.md) especifica la ubicación, el tamaño y las características de la granja de servidores web que hospeda la aplicación. Para ahorrar dinero cuando hospede varias aplicaciones, configure las aplicaciones web para que compartan un único plan de App Service. Los planes de App Service definen la región, el tamaño de la instancia, el recuento de escala y la SKU (gratis, compartida, básica, estándar o premium). Seleccione **Nuevo** para crear un plan de App Service. |

    ![Cuadro de diálogo Crear servicio de aplicaciones](./media/webjobs-publish-netcore/app-service-dialog.png)

1. Seleccione **Crear** para crear un WebJobs y los recursos relacionados en Azure con esta configuración e implementar el código del proyecto.