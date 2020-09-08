---
title: Administración de un plan de App Service
description: Aprenda a realizar distintas tareas para administrar un plan de App Service, como crear, mover, escalar y eliminar.
keywords: servicio de aplicaciones, azure app service, escalar, plan de app service, cambiar, crear, administrar, administración
ms.assetid: 4859d0d5-3e3c-40cc-96eb-f318b2c51a3d
ms.topic: article
ms.date: 10/24/2019
ms.custom: seodec18
ms.openlocfilehash: bb0765a1d7934d60f787ed277dd3bd1f9bc1359b
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962951"
---
# <a name="manage-an-app-service-plan-in-azure"></a>Administración de un plan de App Service en Azure

Un [plan de Azure App Service](overview-hosting-plans.md) proporciona los recursos que debe ejecutar una aplicación de App Service. Esta guía muestra cómo administrar un plan de App Service.

## <a name="create-an-app-service-plan"></a>Creación de un plan de App Service

> [!TIP]
> Si no cuenta con App Service Environment, consulte [Creación de un plan de App Service en App Service Environment](environment/app-service-web-how-to-create-a-web-app-in-an-ase.md#createplan).

Puede crear un plan de App Service vacío o como parte de la creación de la aplicación.

1. En [Azure Portal](https://portal.azure.com), haga clic en **Crear un recurso**.

   ![Cree un recurso en Azure Portal.][createResource] 

1. Seleccione **Nuevo** > **Aplicación web** u otro tipo de aplicación de App Service.

   ![Cree una aplicación en Azure Portal.][createWebApp] 

2. Configure la sección **Detalles de la instancia** antes de configurar el plan de App Service. Configuraciones, como **Publicar** y **Sistemas operativos** , pueden cambiar los planes de tarifa disponibles para el plan de App Service. **Región** determina dónde se crea el plan de App Service. 
   
3. En la sección **Plan de App Service**, seleccione un plan existente o cree un plan seleccionando **Crear nuevo**.

   ![Creación de un plan de App Service.][createASP] 

4. Al crear un plan, puede seleccionar su plan de tarifa. **SKU y tamaño**: seleccione **Cambiar el tamaño** para cambiar el plan de tarifa. 

<a name="move"></a>

## <a name="move-an-app-to-another-app-service-plan"></a>Cambio de una aplicación a otro plan de App Service

Puede mover una aplicación a otro plan de App Service siempre que el plan de origen y el plan de destino se encuentren en el _mismo grupo de recursos y región geográfica_.

> [!NOTE]
> Azure implementa cada nuevo plan de App Service en una unidad de implementación, que internamente se denomina espacio web. Cada región puede tener muchos espacios web, pero la aplicación solo puede moverse entre planes que se hayan creado en el mismo espacio web. Una instancia de App Service Environment es un espacio web aislado, por lo que las aplicaciones se pueden mover entre planes de la misma instancia de App Service Environment, pero no entre planes de diferentes instancias de App Service Environment.
>
> No se puede especificar el espacio web que quiera al crear un plan, pero es posible asegurar que un plan se cree en el mismo espacio web que un plan existente. En resumen, todos los planes que se creen con la misma combinación de grupo de recursos y región se implementan en el mismo espacio de web. Por ejemplo, si crea un plan en un grupo de recursos A y en una región B, cualquier plan que cree posteriormente en el grupo de recursos A y en la región B se implementará en el mismo espacio web. Tenga en cuenta que los planes no pueden mover espacios web una vez que se crean, por lo que no podrá mover un plan al "mismo espacio web" como otro plan moviéndolo a otro grupo de recursos.
> 

1. En [Azure Portal](https://portal.azure.com), busque y seleccione **App Services** y elija la aplicación que quiere implementar.

2. En el menú de la izquierda, seleccione **Cambiar el plan de App Service**.

3. En el menú desplegable **Plan de App Service**, seleccione un plan existente al que quiere mover esta aplicación. El menú desplegable muestra únicamente los planes que se encuentran en el mismo grupo de recursos y región geográfica que el plan de App Service actual. Si no existe tal plan, le permite crear un plan de forma predeterminada. También puede crear un nuevo plan manualmente seleccionando **Crear nuevo**.

4. Si crea un plan, puede seleccionar su plan de tarifa. En **Plan de tarifa**, seleccione el existente para cambiarlo. 
   
   > [!IMPORTANT]
   > Si va a mover una aplicación de un plan de tarifa superior a un plan de tarifa inferior, como desde **D1** a **F1**, la aplicación puede perder ciertas funcionalidades en el plan de destino. Por ejemplo, si su aplicación usa certificados TLS/SSL, podría aparecer este mensaje de error:
   >
   > `Cannot update the site with hostname '<app_name>' because its current SSL configuration 'SNI based SSL enabled' is not allowed in the target compute mode. Allowed SSL configuration is 'Disabled'.`

5. Cuando termine, seleccione **Aceptar**.
   
   ![Selector de plan de App Service.][change] 

## <a name="move-an-app-to-a-different-region"></a>Cambio de una aplicación a una región diferente

La región en la que se ejecuta la aplicación es la región del plan de App Service en la que se encuentra. Sin embargo, no se puede cambiar la región de un plan de App Service. Si desea ejecutar la aplicación en una región diferente, una alternativa es clonar la aplicación. La clonación hará una copia de su aplicación en un plan de App Service nuevo o existente en cualquier región.

Puede encontrar **Clonar aplicación** en la sección **Herramientas de desarrollo** del menú.

> [!IMPORTANT]
> La clonación tiene algunas limitaciones. Puede obtener información al respecto en [Clonación de aplicaciones de Azure App Service](app-service-web-app-cloning.md).

## <a name="scale-an-app-service-plan"></a>Escalación de un plan de App Service

Para escalar verticalmente un plan de tarifa del plan de App Service, consulte [Escalado vertical de aplicaciones en Azure](manage-scale-up.md).

Para escalar horizontalmente el recuento de instancias de una aplicación, consulte [Escalado del recuento de instancias de forma manual o automática](../azure-monitor/platform/autoscale-get-started.md).

<a name="delete"></a>

## <a name="delete-an-app-service-plan"></a>Eliminación de un plan de App Service

Para evitar cargos inesperados, al eliminar la última aplicación en un plan de App Service, App Service también elimina el plan de forma predeterminada. Si, en su lugar, elige mantener el plan, debe cambiar el plan a **Gratis** para que no le cobren.

> [!IMPORTANT]
> Los planes de App Service que no tienen aplicaciones asociadas a ellos seguirán generando cargos, ya que siguen reservando las instancias de máquinas virtuales configuradas.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Escalado vertical de aplicaciones en Azure](manage-scale-up.md)

[change]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/change-appserviceplan.png
[createASP]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/create-appserviceplan.png
[createWebApp]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/create-web-app.png
[createResource]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/create-a-resource.png