---
title: Configuración de entornos de ensayo
description: Aprenda a implementar aplicaciones en una ranura que no sea de producción y realice intercambios automáticos en producción. Aumente la confiabilidad y elimine el tiempo de inactividad de las aplicaciones en las implementaciones.
ms.assetid: e224fc4f-800d-469a-8d6a-72bcde612450
ms.topic: article
ms.date: 04/30/2020
ms.custom: fasttrack-edit
ms.openlocfilehash: b12b85a2248d7709066ba3218327e0a5d52a0192
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962169"
---
# <a name="set-up-staging-environments-in-azure-app-service"></a>Configuración de entornos de ensayo en Azure App Service
<a name="Overview"></a>

Al implementar la aplicación web, la aplicación web en Linux, el back-end móvil o la aplicación API en [Azure App Service](https://go.microsoft.com/fwlink/?LinkId=529714), puede implementarlas en una ranura de implementación independiente en lugar de en la ranura de producción predeterminada si realiza la ejecución en el nivel de plan **Estándar**, **Premium** o **Aislado** de App Service. Las ranuras de implementación son aplicaciones activas con sus propios nombres de host. Los elementos de contenido y configuraciones de aplicaciones se pueden intercambiar entre dos ranuras de implementación, incluida la ranura de producción. 

La implementación de la aplicación en un espacio que no sea de producción ofrece las siguientes ventajas:

* Puede validar los cambios en la aplicación en una ranura de implementación de ensayo antes de intercambiarla con la ranura de producción.
* La implementación de una aplicación en un espacio y su posterior paso a producción garantiza que todas las instancias del espacio están preparadas antes de dicho paso a producción. Esto elimina tiempos de inactividad a la hora de implementar la aplicación. El redireccionamiento del tráfico es perfecta y no se pierde ninguna solicitud en las operaciones de intercambio. Todo este flujo de trabajo se puede automatizar mediante la configuración del [intercambio automático](#Auto-Swap) cuando no sea necesario realizar ninguna validación antes del intercambio.
* Después del intercambio, la ranura con la aplicación de ensayo anterior ahora ocupa la aplicación de producción anterior. Si las modificaciones que se han intercambiado en el espacio de producción no son los que esperaba, puede volver a realizar un intercambio inmediatamente para tener el "último sitio que sabe que funciona correctamente".

Cada nivel del plan de App Service admite un número distinto de ranuras de implementación. El uso de las ranuras de implementación no tiene costo adicional. Para averiguar el número de ranuras que admite el nivel de la aplicación, consulte [Límites de App Service](../azure-resource-manager/management/azure-subscription-service-limits.md#app-service-limits). 

Para escalar la aplicación a un nivel diferente, asegúrese de que el nivel de destino admite el número de ranuras que la aplicación ya usa. Por ejemplo, si la aplicación tiene más de cinco, no se puede reducir verticalmente al nivel **Estándar**, ya que este **nivel** solo admite cinco ranuras de implementación. 

<a name="Add"></a>

## <a name="add-a-slot"></a>Incorporación de una ranura
Para poder habilitar varias espacios de implementación, la aplicación debe ejecutarse en los niveles **Estándar**, **Premium** o **Aislado**.


1. En [Azure Portal](https://portal.azure.com/), busque y seleccione **App Services** y luego elija la aplicación. 
   
    ![Búsqueda de App Services](./media/web-sites-staged-publishing/search-for-app-services.png)
   

2. En el panel izquierdo, seleccione **Ranuras de implementación** > **Agregar ranura**.
   
    ![Agregar una nueva ranura de implementación](./media/web-sites-staged-publishing/QGAddNewDeploymentSlot.png)
   
   > [!NOTE]
   > Si la aplicación no está aún en los niveles **Estándar**, **Premium** o **Aislado**, recibirá un mensaje que indica los niveles compatibles para habilitar la publicación almacenada provisionalmente. Llegados a este punto, tiene la opción de seleccionar **Actualizar** e ir a la pestaña **Escala** de la aplicación antes de continuar.
   > 

3. En el cuadro de diálogo **Agregar ranura**, asígnele un nombre a la ranura y seleccione si desea que la configuración de la aplicación se clone de otra ranura de implementación. Haga clic en **Agregar** para continuar.
   
    ![Origen de la configuración](./media/web-sites-staged-publishing/ConfigurationSource1.png)
   
    La configuración se puede clonar de una ranura existente. La configuración que se puede clonar incluye los valores de la aplicación, las cadenas de conexión, versiones del marco del lenguaje, los sockets web, la versión de HTTP y el valor de bits de la plataforma.

4. Tras agregar la ranura, seleccione **Cerrar** para cerrar el cuadro de diálogo. La nueva ranura se muestra en la página **Ranuras de implementación**. De forma predeterminada, en la nueva ranura el valor de **Tráfico %** es 0 y todo el tráfico de los clientes se enruta a la ranura de producción.

5. Seleccione la nueva ranura de implementación para abrir su página de recursos.
   
    ![Título de la ranura de implementación](./media/web-sites-staged-publishing/StagingTitle.png)

    Como las restantes aplicaciones de App Service, el espacio de ensayo tiene una página de administración. La configuración del espacio se puede cambiar. Para recordarle que está viendo la ranura de implementación, el nombre de la aplicación se muestra como **\<app-name>/\<slot-name>** y el tipo de aplicación es **App Service (ranura)** . También puede ver la ranura como una aplicación independiente en el grupo de recursos, con las mismas designaciones.

6. Seleccione la dirección URL de la aplicación en la página de recursos de la ranura. La ranura de implementación tiene su propio nombre de host y es también una aplicación activa. Para limitar el acceso público a la ranura de implementación, consulte el artículo sobre [Restricciones de IP de Azure App Service](app-service-ip-restrictions.md).

El nuevo espacio de implementación no tiene contenido, aunque se clone la configuración de otro espacio. Por ejemplo, puede [publicar en esta ranura mediante Git](./deploy-local-git.md). La implementación en el espacio se puede realizar desde otra rama del repositorio o desde otro repositorio.

<a name="AboutConfiguration"></a>

## <a name="what-happens-during-a-swap"></a>Qué ocurre durante un intercambio

### <a name="swap-operation-steps"></a>Pasos del intercambio

Al intercambiar dos ranuras (normalmente de una ranura de ensayo a la ranura de producción), App Service hace lo siguiente para asegurarse de que la ranura de destino no experimente tiempo de inactividad:

1. Aplique las siguientes opciones de configuración de la ranura de destino (por ejemplo, la de producción) a todas las instancias de la ranura de origen: 
    - Opciones de configuración de aplicación y cadenas de conexión [específicas de la ranura](#which-settings-are-swapped), si procede.
    - Configuración de [implementación continua](deploy-continuous-deployment.md), si está habilitada.
    - Configuración de [autenticación de App Service](overview-authentication-authorization.md) configuración, si está habilitada.
    
    Cualquiera de estos casos desencadena el reinicio de todas las instancias de la ranura de origen. Durante el [intercambio con vista previa](#Multi-Phase), esta marca el final de la primera fase. La operación de intercambio se pausará y podrá validar que la ranura de origen funciona correctamente con la configuración de la ranura de destino.

1. Espere a que todas las instancias de la ranura de origen se hayan reiniciado. Si el reinicio no se puede realizar en alguna instancia, el intercambio revierte todos los cambios en la ranura de origen y detiene la operación.

1. Si la [caché local](overview-local-cache.md) está habilitada, desencadene la inicialización de la caché local mediante una solicitud HTTP a la raíz de la aplicación ("/") en cada instancia de la ranura de origen. Espere a que todas las instancias devuelvan cualquier respuesta HTTP. La inicialización de la caché local produce otro reinicio en cada instancia.

1. Si el [intercambio automático](#Auto-Swap) está habilitado con [preparación personalizada](#Warm-up), desencadene la [inicialización de la aplicación](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization) mediante una solicitud HTTP a la raíz de la aplicación ("/") en cada instancia de la ranura de origen.

    Si `applicationInitialization` no se especifica, desencadene una solicitud HTTP a la raíz de la aplicación de la ranura de origen en cada instancia. 
    
    Si una instancia devuelve cualquier respuesta HTTP, se considera que se ha preparado.

1. Si todas las instancias de la ranura de origen se han preparado correctamente, intercambie las dos ranuras; para ello, cambie las reglas de enrutamiento de las dos ranuras. Después de este paso, la ranura de destino (por ejemplo, la de producción) tendrá la aplicación que se preparó previamente en la ranura de origen.

1. Ahora que la ranura de origen tiene la aplicación que se preparó previamente antes del intercambio, realice la misma operación; aplique la configuración y reinicie las instancias.

Durante intercambio, todo el trabajo de inicialización de las aplicaciones intercambiadas se realiza en la ranura de origen. La ranura de destino permanece en línea mientras la de origen se está preparando, independientemente de en qué punto el intercambio se realice o no correctamente. Para intercambiar una ranura de ensayo con la de producción, asegúrese de que esta última es siempre la de destino. De este modo, el intercambio no afecta a la aplicación de producción.

### <a name="which-settings-are-swapped"></a>¿Qué configuración se intercambia?

[!INCLUDE [app-service-deployment-slots-settings](../../includes/app-service-deployment-slots-settings.md)]

Para configurar la aplicación o una cadena de conexión de esta para que se ajuste a una ranura específica (sin intercambio), vaya a la página **Configuración** de esa ranura. Agregue configuración o modifíquela y seleccione **configuración de ranura de implementación**. Al seleccionar esta casilla se indica a App Service que la configuración no se puede intercambiar. 

![Configuración de espacios](./media/web-sites-staged-publishing/SlotSetting.png)

<a name="Swap"></a>

## <a name="swap-two-slots"></a>Intercambio de dos espacios 
Las ranuras de implementación se pueden intercambiar en las páginas **Ranuras de implementación** e **Información general** de la aplicación. Para detalles técnicos sobre el intercambio de ranuras, consulte [Qué ocurre durante un intercambio](#AboutConfiguration).

> [!IMPORTANT]
> Antes de intercambiar una aplicación de una ranura de implementación a producción, asegúrese de la de producción es la ranura de destino y de que todas las configuraciones de la ranura de origen son exactamente como las quiere en producción.
> 
> 

Para intercambiar ranuras de implementación:

1. Vaya a la página **Ranuras de implementación** y seleccione **Intercambiar**.
   
    ![Botón Intercambiar](./media/web-sites-staged-publishing/SwapButtonBar.png)

    El cuadro de diálogo **Intercambiar** muestra la configuración de la ranura de origen y de destino seleccionadas que se van a cambiar.

2. Seleccione los espacios **Origen** y **Destino**. Por lo general, el destino suele ser el espacio de producción. Además, seleccione las pestañas **Cambios de origen** y **Cambios de destino**, y compruebe que los cambios en la configuración son los esperados. En cuanto termine, puede intercambiar los espacios. Para ello solo debe seleccionar **Intercambiar**.

    ![Intercambio completo](./media/web-sites-staged-publishing/SwapImmediately.png)

    Para ver cómo funcionaría la ranura de destino con la nueva configuración antes de que se realice el intercambio, no seleccione **Intercambiar**, siga las instrucciones de [Intercambio con vista previa](#Multi-Phase).

3. Cuando haya terminado, seleccione **Cerrar** para cerrar el cuadro de diálogo.

Si tiene problemas, consulte [Solución de problemas con los intercambios](#troubleshoot-swaps).

<a name="Multi-Phase"></a>

### <a name="swap-with-preview-multi-phase-swap"></a>Intercambio con vista previa (intercambio en varias fases)

Antes de cambiar a producción como ranura de destino, valide que la aplicación se ejecute con la configuración intercambiada. La ranura de origen también se ha preparado antes de que finalice el intercambio, algo que es conveniente para las aplicaciones críticas.

Al realizarse un intercambio con vista previa, App Service realiza el mismo [intercambio](#AboutConfiguration), pero se detiene tras el primer paso. En ese momento puede comprobar el resultado en la ranura de ensayo antes de completar el intercambio. 

Si cancela el intercambio, App Service vuelve a aplicar los elementos de configuración en la ranura de origen.

Para realizar el intercambio con vista previa:

1. Siga los pasos que se indican en [Intercambio de ranuras de implementación](#Swap), pero seleccione **Realizar intercambio con versión preliminar** .

    ![Intercambio con vista previa](./media/web-sites-staged-publishing/SwapWithPreview.png)

    El cuadro de diálogo muestra cómo cambia la configuración de la ranura de origen en la fase 1 y cómo cambian tanto la ranura de origen como la de destino en la fase 2.

2. Cuando esté listo para iniciar el intercambio, seleccione **Iniciar intercambio**.

    Cuando se complete la fase 1, se le notificará en el cuadro de diálogo. Vaya a `https://<app_name>-<source-slot-name>.azurewebsites.net` para obtener una vista previa del intercambio en la ranura de origen. 

3. Cuando esté listo para completar el intercambio pendiente, seleccione **Completar intercambio** en **Acción de intercambio** y seleccione**Completar intercambio**.

    Para cancelar un intercambio pendiente, seleccione **Cancelar intercambio** en su lugar.

4. Cuando haya terminado, seleccione **Cerrar** para cerrar el cuadro de diálogo.

Si tiene problemas, consulte [Solución de problemas con los intercambios](#troubleshoot-swaps).

Para automatizar un intercambio en varias fases, consulte [Automatización con Azure PowerShell](#automate-with-powershell).

<a name="Rollback"></a>

## <a name="roll-back-a-swap"></a>Reversión de intercambios
Si aparecen errores en el espacio de destino (por ejemplo, el espacio de producción) después de un intercambio de espacios, restaure los espacios al estado que tenían antes del intercambio. Para ello, vuelva a intercambiarlos de inmediato.

<a name="Auto-Swap"></a>

## <a name="configure-auto-swap"></a>Configuración del intercambio automático

> [!NOTE]
> El intercambio automático no se admite en las aplicaciones web en Linux.

El intercambio automático optimiza los escenarios de Azure DevOps en los que se desee implementar una aplicación continuamente sin arranques en frío ni tiempos de inactividad para los clientes de la aplicación. Cuando se habilita el intercambio automático para una ranura a producción, cada vez que se insertan los cambios de código en esa ranura, App Service [cambia automáticamente la aplicación a producción](#swap-operation-steps) después de que se haya preparado en la de origen.

   > [!NOTE]
   > Antes de configurar el intercambio automático en la ranura de producción, considere la posibilidad de probarlo en una ranura de destino distinta de la de producción.
   > 

Para configurar el intercambio automático:

1. Vaya a la página de recursos de la aplicación. Seleccione **Ranuras de implementación** >  *\<desired source slot>*  > **Configuración** > **Configuración general**.
   
2. Para **Intercambio automático habilitado**, seleccione **Activado**. A continuación, seleccione la ranura de destino deseada como **Ranura de implementación de intercambio automático** y seleccione **Guardar** en la barra de comandos. 
   
    ![Selecciones para configurar el intercambio automático](./media/web-sites-staged-publishing/AutoSwap02.png)

3. Ejecute una inserción de código en el espacio de origen. El intercambio automático se realiza al poco tiempo y la actualización se refleja en la dirección URL de la ranura de destino.

Si tiene problemas, consulte [Solución de problemas con los intercambios](#troubleshoot-swaps).

<a name="Warm-up"></a>

## <a name="specify-custom-warm-up"></a>Especificación de Preparación personalizada

Algunas aplicaciones pueden requerir acciones de preparación personalizadas para el intercambio. El elemento de configuración `applicationInitialization` de web.config permite especificar acciones de inicialización personalizadas. El [intercambio](#AboutConfiguration) espera hasta que se completa esta preparación personalizada para realizar el intercambio con la ranura de destino. He aquí un fragmento de ejemplo del archivo web.config.

```xml
<system.webServer>
    <applicationInitialization>
        <add initializationPage="/" hostName="[app hostname]" />
        <add initializationPage="/Home/About" hostName="[app hostname]" />
    </applicationInitialization>
</system.webServer>
```

Para más información sobre la personalización del elemento `applicationInitialization`, consulte [Most common deployment slot swap failures and how to fix them](https://ruslany.net/2017/11/most-common-deployment-slot-swap-failures-and-how-to-fix-them/) (Errores de intercambio de ranuras de implementación más comunes y soluciones).

También puede personalizar el comportamiento de la preparación con una o ambas de las siguientes [opciones de configuración de la aplicación](configure-common.md):

- `WEBSITE_SWAP_WARMUP_PING_PATH`: ruta de acceso para hacer ping y así preparar el sitio. Agregue esta configuración de aplicación especificando una ruta de acceso personalizada que comience con una barra diagonal como valor. Un ejemplo es `/statuscheck`. El valor predeterminado es `/`. 
- `WEBSITE_SWAP_WARMUP_PING_STATUSES`: códigos de respuesta HTTP válidos para la operación de preparación. Agregue esta configuración de aplicación con una lista de códigos HTTP separados por comas. Un ejemplo sería `200,202`. Si el código de estado devuelto no está en la lista, las operaciones de preparación e intercambio se detienen. Por defecto, todos los códigos de respuesta son válidos.

> [!NOTE]
> El elemento de configuración `<applicationInitialization>` forma parte de cada inicio de la aplicación, mientras que los dos valores de aplicación de comportamiento de preparación solo se aplican a los intercambios de ranura.

Si tiene problemas, consulte [Solución de problemas con los intercambios](#troubleshoot-swaps).

## <a name="monitor-a-swap"></a>Supervisión de un intercambio

Si el [intercambio](#AboutConfiguration) tarda mucho tiempo en completarse, puede obtener información sobre su estado en el [registro de actividad](../azure-monitor/platform/platform-logs-overview.md).

En la página de recursos de la aplicación del portal, en el panel izquierdo, seleccione **Registro de actividad**.

Una operación de intercambio aparece en la consulta de registro como `Swap Web App Slots`. Puede expandirla y seleccionar una de los suboperaciones o errores para ver los detalles.

## <a name="route-traffic"></a>Enrutamiento del tráfico

De forma predeterminada, todas las solicitudes que realiza el cliente a la dirección URL de producción de la aplicación (`http://<app_name>.azurewebsites.net`) se enrutan al espacio de producción. Una parte del tráfico se puede enrutar a otro espacio. Esta característica es útil si se necesitan los comentarios de los usuarios al respecto de una nueva actualización, pero dicha actualización aún no está lista para enviarla a producción.

### <a name="route-production-traffic-automatically"></a>Enrutamiento automático del tráfico de producción

Para enrutar automáticamente el tráfico de producción:

1. Vaya a la página de recursos de la aplicación y seleccione **Ranuras de implementación**.

2. En la columna **Tráfico %** del espacio que se desea enrutar, especifique un porcentaje (entre 0 y 100) que represente la cantidad del tráfico total que desea enrutar. Seleccione **Guardar**.

    ![Configuración de un porcentaje de tráfico](./media/web-sites-staged-publishing/RouteTraffic.png)

Una vez guardado el valor, el porcentaje especificado de clientes se enruta aleatoriamente a la ranura que no es de producción. 

Una vez enrutado un cliente automáticamente a una ranura concreta, permanece "anclado" en esa ranura todo el tiempo que dure su sesión. En el explorador del cliente, para ver a qué espacio está anclada la sesión debe examinar la cookie `x-ms-routing-name` en los encabezados HTTP. Las solicitudes que se enrutan a al espacio "de ensayo" tienen la cookie `x-ms-routing-name=staging`. Las solicitudes que se enrutan al espacio de producción tienen la cookie `x-ms-routing-name=self`.

   > [!NOTE]
   > Junto a Azure Portal, también puede usar el comando [`az webapp traffic-routing set`](/cli/azure/webapp/traffic-routing#az-webapp-traffic-routing-set) en la CLI de Azure para establecer los porcentajes de enrutamiento desde herramientas de CI/CD, como canalizaciones DevOps u otros sistemas de automatización.
   > 

### <a name="route-production-traffic-manually"></a>Enrutamiento manual del tráfico de producción

Además del enrutamiento automático del tráfico, App Service puede enrutar solicitudes a un espacio concreto. Esto es útil cuando desee que los usuarios puedan escoger el enrutamiento a la aplicación beta, o la exclusión voluntaria de esta. Para enrutar el tráfico de producción de manera manual se usa el parámetro de consulta `x-ms-routing-name`.

Para permitir que los usuarios opten por la exclusión del enrutamiento a la aplicación beta, por ejemplo, puede poner este vínculo en la página web:

```html
<a href="<webappname>.azurewebsites.net/?x-ms-routing-name=self">Go back to production app</a>
```

La cadena `x-ms-routing-name=self` especifica la ranura de producción. Cuando el explorador del cliente accede al vínculo, se le redirige a la ranura de producción. Cada solicitud posterior tiene la cookie `x-ms-routing-name=self` que ancla la sesión a la ranura de producción.

Para permitir a los usuarios participar en la aplicación de la versión beta, establezca el mismo parámetro de consulta con el nombre de la ranura que no es de producción. Este es un ejemplo:

```
<webappname>.azurewebsites.net/?x-ms-routing-name=staging
```

De forma predeterminada, las ranuras nuevas tienen una regla de enrutamiento del `0%`, en gris. Cuando se establece explícitamente este valor en `0%` (en negro), los usuarios pueden acceder a la ranura de ensayo manualmente mediante el parámetro de consulta `x-ms-routing-name`. Pero no se les enrutará automáticamente a la ranura, ya que el porcentaje se ha establecido en 0. Se trata de un escenario avanzado donde puede "ocultar" su ranura de ensayo del público y permitir que los equipos internos prueben los cambios en la ranura al mismo tiempo.

<a name="Delete"></a>

## <a name="delete-a-slot"></a>Eliminación de una ranura

Busque y seleccione la aplicación. Seleccione **Ranuras de implementación** >  *\<slot to delete>*  > **Información general**. El tipo de aplicación se muestra como **App Service (ranura)** para recordarle que está viendo una ranura de implementación. En la barra de comandos, seleccione **Eliminar**.  

![Eliminación de una ranura de implementación](./media/web-sites-staged-publishing/DeleteStagingSiteButton.png)

<!-- ======== AZURE POWERSHELL CMDLETS =========== -->

<a name="PowerShell"></a>

## <a name="automate-with-powershell"></a>Automatización con PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Azure PowerShell es un módulo que proporciona cmdlets para administrar Azure mediante Windows PowerShell, incluida la compatibilidad para administrar ranuras de implementación de Azure App Service.

Para obtener información acerca de cómo instalar y configurar Azure PowerShell y cómo autenticar Azure PowerShell con su suscripción de Azure, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/).  

---
### <a name="create-a-web-app"></a>Creación de una aplicación web
```powershell
New-AzWebApp -ResourceGroupName [resource group name] -Name [app name] -Location [location] -AppServicePlan [app service plan name]
```

---
### <a name="create-a-slot"></a>Creación de una ranura
```powershell
New-AzWebAppSlot -ResourceGroupName [resource group name] -Name [app name] -Slot [deployment slot name] -AppServicePlan [app service plan name]
```

---
### <a name="initiate-a-swap-with-a-preview-multi-phase-swap-and-apply-destination-slot-configuration-to-the-source-slot"></a>Inicio de un intercambio con vista previa (intercambio en varias fases) y aplicación de configuración de la ranura de destino a la ranura de origen
```powershell
$ParametersObject = @{targetSlot  = "[slot name – e.g. "production"]"}
Invoke-AzResourceAction -ResourceGroupName [resource group name] -ResourceType Microsoft.Web/sites/slots -ResourceName [app name]/[slot name] -Action applySlotConfig -Parameters $ParametersObject -ApiVersion 2015-07-01
```

---
### <a name="cancel-a-pending-swap-swap-with-review-and-restore-the-source-slot-configuration"></a>Cancelación de un intercambio pendiente (intercambio con vista previa) y restauración de configuración de la ranura de origen
```powershell
Invoke-AzResourceAction -ResourceGroupName [resource group name] -ResourceType Microsoft.Web/sites/slots -ResourceName [app name]/[slot name] -Action resetSlotConfig -ApiVersion 2015-07-01
```

---
### <a name="swap-deployment-slots"></a>Intercambio de ranuras de implementación
```powershell
$ParametersObject = @{targetSlot  = "[slot name – e.g. "production"]"}
Invoke-AzResourceAction -ResourceGroupName [resource group name] -ResourceType Microsoft.Web/sites/slots -ResourceName [app name]/[slot name] -Action slotsswap -Parameters $ParametersObject -ApiVersion 2015-07-01
```

### <a name="monitor-swap-events-in-the-activity-log"></a>Supervisión de los eventos de intercambio en el registro de actividad
```powershell
Get-AzLog -ResourceGroup [resource group name] -StartTime 2018-03-07 -Caller SlotSwapJobProcessor  
```

---
### <a name="delete-a-slot"></a>Eliminación de una ranura
```powershell
Remove-AzResource -ResourceGroupName [resource group name] -ResourceType Microsoft.Web/sites/slots –Name [app name]/[slot name] -ApiVersion 2015-07-01
```

## <a name="automate-with-resource-manager-templates"></a>Automatización con plantillas de Resource Manager

Las [plantillas de Resource Manager](../azure-resource-manager/templates/overview.md) son archivos JSON declarativos que se usan para automatizar la implementación y la configuración de los recursos de Azure. Para intercambiar ranuras mediante plantillas de Resource Manager, establecerá dos propiedades en los recursos *Microsoft.Web/sites/slots* and *Microsoft.Web/sites*:

- `buildVersion`: esta es una propiedad de cadena que representa la versión actual de la aplicación implementada en la ranura. Por ejemplo: "v1", "1.0.0.1" o "2019-09-20T11:53:25.2887393-07:00".
- `targetBuildVersion`: esta es una propiedad de cadena que especifica qué `buildVersion` debe tener la ranura. Si targetBuildVersion no es igual a la `buildVersion` actual, esto activará la operación de intercambio al encontrar la ranura que tiene la `buildVersion` especificada.

### <a name="example-resource-manager-template"></a>Ejemplo de plantilla de Resource Manager

La siguiente plantilla de Resource Manager actualizará el objeto `buildVersion` del espacio de ensayo y establecerá el objeto `targetBuildVersion` en el espacio de producción. Esto intercambiará las dos ranuras. La plantilla supone que ya tiene una aplicación web creada con una ranura llamada "almacenamiento provisional".

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "my_site_name": {
            "defaultValue": "SwapAPIDemo",
            "type": "String"
        },
        "sites_buildVersion": {
            "defaultValue": "v1",
            "type": "String"
        }
    },
    "resources": [
        {
            "type": "Microsoft.Web/sites/slots",
            "apiVersion": "2018-02-01",
            "name": "[concat(parameters('my_site_name'), '/staging')]",
            "location": "East US",
            "kind": "app",
            "properties": {
                "buildVersion": "[parameters('sites_buildVersion')]"
            }
        },
        {
            "type": "Microsoft.Web/sites",
            "apiVersion": "2018-02-01",
            "name": "[parameters('my_site_name')]",
            "location": "East US",
            "kind": "app",
            "dependsOn": [
                "[resourceId('Microsoft.Web/sites/slots', parameters('my_site_name'), 'staging')]"
            ],
            "properties": {
                "targetBuildVersion": "[parameters('sites_buildVersion')]"
            }
        }        
    ]
}
```

Esta plantilla de Resource Manager es idempotente, lo que significa que puede ejecutarse repetidamente y producir el mismo estado de las ranuras. Después de la primera ejecución, `targetBuildVersion` coincidirá con el `buildVersion` actual, por lo que no se activará un intercambio.

<!-- ======== Azure CLI =========== -->

<a name="CLI"></a>

## <a name="automate-with-the-cli"></a>Automatización con la CLI

Para información sobre los comandos de la [CLI de Azure](https://github.com/Azure/azure-cli) que se usan con las ranuras de implementación, consulte [az webapp deployment slot](/cli/azure/webapp/deployment/slot).

## <a name="troubleshoot-swaps"></a>Solución de problemas con los intercambios

Si se produce algún error durante un [intercambio de ranuras](#AboutConfiguration), se registra en *D:\home\LogFiles\eventlog.xml*. También se archiva en el registro de errores específicos de la aplicación.

Estos son algunos errores de intercambio habituales:

- Se superó el tiempo de una solicitud HTTP a la raíz de la aplicación. La operación de intercambio espera 90 segundos para cada solicitud HTTP y lo vuelve a intentar hasta 5 veces. Si se agota el tiempo de espera de todos los intentos, el intercambio se detiene.

- La inicialización de la caché local puede producir un error cuando el contenido de la aplicación supera la cuota de disco local especificada para la memoria caché local. Para más información, consulte la [Introducción a la caché local](overview-local-cache.md).

- Durante la [preparación personalizada](#Warm-up), las solicitudes HTTP se realizan internamente (sin pasar por la dirección URL externa). Pueden producirse errores en ciertas reglas de reescritura de dirección URL en *Web.config*. Por ejemplo, las reglas de redirección de nombres de dominio o la exigencia de HTTPS pueden impedir que las solicitudes de alcancen el código de la aplicación. Para solucionar este problema, modifique las reglas de reescritura mediante la incorporación de las dos siguientes condiciones:

    ```xml
    <conditions>
      <add input="{WARMUP_REQUEST}" pattern="1" negate="true" />
      <add input="{REMOTE_ADDR}" pattern="^100?\." negate="true" />
      ...
    </conditions>
    ```
- Las reglas de reescritura de dirección URL sin preparación personalizada pueden bloquear las solicitudes HTTP. Para solucionar este problema, modifique las reglas de reescritura mediante la incorporación de la siguiente condición:

    ```xml
    <conditions>
      <add input="{REMOTE_ADDR}" pattern="^100?\." negate="true" />
      ...
    </conditions>
    ```

- Después de los intercambios de ranura, la aplicación puede experimentar reinicios inesperados. El motivo es que después de un intercambio, la configuración de enlace del nombre de host deja de estar sincronizada, lo que de por sí no provoca reinicios. Sin embargo, algunos eventos de almacenamiento subyacentes (como las conmutaciones por error de volúmenes de almacenamiento) pueden detectar estas discrepancias y forzar el reinicio de todos los procesos de trabajo. Para reducir estos tipos de reinicios, establezca la configuración de la aplicación [`WEBSITE_ADD_SITENAME_BINDINGS_IN_APPHOST_CONFIG=1`](https://github.com/projectkudu/kudu/wiki/Configurable-settings#disable-the-generation-of-bindings-in-applicationhostconfig) en *todas las ranuras*. Sin embargo, esta configuración de la aplicación *no* funciona con aplicaciones de Windows Communication Foundation (WCF).

## <a name="next-steps"></a>Pasos siguientes
[Bloqueo del acceso a espacios que no sean de producción](app-service-ip-restrictions.md)