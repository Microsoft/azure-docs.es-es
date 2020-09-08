---
title: Configuración del nivel PremiumV2
description: Obtenga información sobre cómo mejorar el rendimiento para su aplicación web, móvil y de API en Azure App Service mediante el escalado al nuevo plan de tarifa de PremiumV2.
keywords: servicio de aplicaciones, servicio de aplicaciones de azure, escala, escalable, plan del servicio de aplicaciones, costo del servicio de aplicaciones
ms.assetid: ff00902b-9858-4bee-ab95-d3406018c688
ms.topic: article
ms.date: 07/25/2018
ms.custom: seodec18
ms.openlocfilehash: 4db7c6bf29d0874b5441a8a0eb90f7d1ada33d9c
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962441"
---
# <a name="configure-premiumv2-tier-for-azure-app-service"></a>Configuración del nivel PremiumV2 para Azure App Service

Con el nuevo plan de tarifa **PremiumV2**, se obtienen procesadores más rápidos, almacenamiento SSD y el doble de relación memoria-núcleo que con los planes de tarifa actuales. Con la ventaja de rendimiento, puede ahorrar dinero mediante la ejecución de las aplicaciones en menos instancias. En este artículo, aprenderá a crear una aplicación en el nivel **PremiumV2** o escalar verticalmente una aplicación al nivel **PremiumV2**.

## <a name="prerequisites"></a>Requisitos previos

Para escalar verticalmente una aplicación en **PremiumV2**, debe tener una aplicación de Azure App Service que se ejecute en un plan de tarifa inferior a **PremiumV2** y esta aplicación debe ejecutarse en una implementación de App Service que admita PremiumV2.

<a name="availability"></a>

## <a name="premiumv2-availability"></a>Disponibilidad del nivel PremiumV2

El nivel **PremiumV2** está disponible para App Service tanto en _Windows_ como en _Linux_.

**PremiumV2** está disponible en la mayoría de las regiones de Azure. Para ver si está disponible en su región, ejecute el siguiente comando de la CLI de Azure en [Azure Cloud Shell](../cloud-shell/overview.md):

```azurecli-interactive
az appservice list-locations --sku P1V2
```

<a name="create"></a>

## <a name="create-an-app-in-premiumv2-tier"></a>Creación de una aplicación en el nivel PremiumV2

El plan de tarifa de una aplicación de App Service se define en el [plan de App Service](overview-hosting-plans.md) en el que se ejecuta. Puede crear un plan de App Service de forma independiente o durante el proceso de creación de la aplicación.

Cuando configure el plan de App Service en <a href="https://portal.azure.com" target="_blank">Azure Portal</a>, seleccione **Plan de tarifa**. 

Seleccione **Producción**, luego **P1V2**, **P2V2** o **P3V2** y, a continuación, haga clic en **Aplicar**.

![Captura de pantalla que muestra los planes de tarifa recomendados para la aplicación.](media/app-service-configure-premium-tier/scale-up-tier-select.png)

> [!IMPORTANT] 
> Si no ve **P1V2**, **P2V2** y **P3V2** como opciones, o si las opciones están atenuadas, es probable que **PremiumV2** no esté disponible en la implementación subyacente de App Service que contiene el plan de App Service. Vea [Escalado vertical desde una combinación de región y grupos de recursos no admitidos](#unsupported) para obtener más información.

## <a name="scale-up-an-existing-app-to-premiumv2-tier"></a>Escalado vertical de una aplicación existente a nivel PremiumV2

Antes de escalar una aplicación existente al nivel **PremiumV2**, asegúrese de que **PremiumV2** está disponible. Para información, consulte [Disponibilidad del nivel PremiumV2](#availability). Si no está disponible en su región, ve [Escalado vertical desde una combinación de región y grupos de recursos no admitidos](#unsupported).

Dependiendo del entorno de hospedaje, el escalado vertical puede requerir pasos adicionales. 

En <a href="https://portal.azure.com" target="_blank">Azure Portal</a>, abra la página de la aplicación de App Service.

En el panel izquierdo de navegación de la página de la aplicación de App Service, seleccione **Escalar verticalmente (plan de App Service)** .

![Captura de pantalla que muestra cómo escalar verticalmente el plan de App Service.](media/app-service-configure-premium-tier/scale-up-tier-portal.png)

Seleccione **Producción**, luego **P1V2**, **P2V2** o **P3V2** y, a continuación, haga clic en **Aplicar**.

![Captura de pantalla que muestra los planes de tarifa recomendados para la aplicación.](media/app-service-configure-premium-tier/scale-up-tier-select.png)

Si la operación finaliza correctamente, la página de información general de la aplicación muestra que ahora está en un plan **PremiumV2**.

![Captura de pantalla que muestra el plan de tarifa de PremiumV2 en la página de información general de la aplicación.](media/app-service-configure-premium-tier/finished.png)

### <a name="if-you-get-an-error"></a>Si obtiene un mensaje de error

Algunos planes de App Service no se pueden escalar verticalmente al nivel PremiumV2 si la implementación de App Service subyacente no admite PremiumV2.  Vea [Escalado vertical desde una combinación de región y grupos de recursos no admitidos](#unsupported) para obtener más información.

<a name="unsupported"></a>

## <a name="scale-up-from-an-unsupported-resource-group-and-region-combination"></a>Escalado vertical desde una combinación de región y grupos de recursos no admitidos

Si la aplicación se ejecuta en una implementación de App Service en la que **PremiumV2** no está disponible o se ejecuta en una región que actualmente no admite **PremiumV2**, tendrá que volver a implementar la aplicación para aprovechar **PremiumV2**.  Tiene dos opciones:

- Cree **otro** grupo de recursos. A continuación, cree **otra** aplicación y un plan de App Service en el **nuevo** grupo de recursos y elija la región de Azure que prefiera durante el proceso de creación.  **Debe** seleccionar el plan **PremiumV2** en el momento en que se crea el nuevo plan de App Service.  Esto garantiza que la combinación del grupo de recursos, el plan de App Service y la región de Azure tendrá como resultado que el plan de App Service se cree en una implementación de App Service que admita **PremiumV2**.  Luego, vuelva a implementar el código de aplicación en la aplicación recién creada y el plan de App Service. Si quiere, posteriormente puede reducir verticalmente el plan de App Service en **PremiumV2** para ahorrar costos y aún podrá volver a escalarlo verticalmente en el futuro mediante **PremiumV2**.
- Si la aplicación ya se ejecuta en un nivel **Premium** existente, puede clonar la aplicación con todos los valores de aplicación, las cadenas de conexión y la configuración de implementación en un nuevo plan de App Service que utilice **PremiumV2**.

    ![Captura de pantalla en que se muestra cómo clonar la aplicación.](media/app-service-configure-premium-tier/clone-app.png)

    En la página **Clonar aplicación**, puede crear un plan de App Service con **PremiumV2** en la región que prefiera y especificar la configuración que quiera clonar.

## <a name="automate-with-scripts"></a>Automatizar con scripts

Puede automatizar la creación de la aplicación en el nivel **PremiumV2** con scripts mediante la [CLI de Azure](/cli/azure/install-azure-cli) o [Azure PowerShell](/powershell/azure/).

### <a name="azure-cli"></a>Azure CLI

El siguiente comando crea un plan de App Service en _P1V2_. También puede ejecutarlo en Cloud Shell. Las opciones para `--sku` son P1V2, _P2V2_ y _P3V2_.

```azurecli-interactive
az appservice plan create \
    --resource-group <resource_group_name> \
    --name <app_service_plan_name> \
    --sku P1V2
```

### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

El siguiente comando crea un plan de App Service en _P1V2_. Las opciones para `-WorkerSize` son _Pequeño_, _Medio_ y _Grande_.

```powershell
New-AzAppServicePlan -ResourceGroupName <resource_group_name> `
    -Name <app_service_plan_name> `
    -Location <region_name> `
    -Tier "PremiumV2" `
    -WorkerSize "Small"
```
## <a name="more-resources"></a>Más recursos

[Escalado vertical de aplicaciones en Azure](manage-scale-up.md)  
[Escalado del recuento de instancias de forma manual o automática](../azure-monitor/platform/autoscale-get-started.md)