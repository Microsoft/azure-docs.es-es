---
title: Escalado de una instancia de Azure SignalR Service
description: Aprenda a escalar una instancia de Azure SignalR Service para agregar o reducir la capacidad, a través de Azure Portal o la CLI de Azure.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.date: 12/11/2019
ms.author: zhshang
ms.custom: devx-track-azurecli
ms.openlocfilehash: bd112526c7b36d5636d5d85c66b25b6310d35d1e
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500494"
---
# <a name="how-to-scale-an-azure-signalr-service-instance"></a>Cómo escalar una instancia de Azure SignalR Service?
En este artículo se muestra cómo escalar la instancia de Azure SignalR Service. Existen dos escenarios de escalado: escalado vertical y escalado horizontal.

* [Escalado vertical](https://en.wikipedia.org/wiki/Scalability#Horizontal_and_vertical_scaling): Obtenga más unidades, conexiones, mensajes y otras funciones. Puede escalar verticalmente cambiando el plan de tarifa de Gratis a Estándar.
* [Escalado horizontal](https://en.wikipedia.org/wiki/Scalability#Horizontal_and_vertical_scaling): Aumento del número de unidades de SignalR. Puede escalar horizontalmente hasta un máximo de 100 unidades.

La configuración de escala tarda unos minutos en aplicarse. En raras ocasiones, puede tardar unos 30 minutos en aplicarse. No hay que modificar el código ni volver a implementar la aplicación de servidor.

Para obtener información sobre los precios y la funcionalidad de instancias individuales de SignalR Service, consulte los [precios de Azure SignalR Service](https://azure.microsoft.com/pricing/details/signalr-service/).  

> [!NOTE]
> Al cambiar SignalR Service del nivel **Gratis** al nivel **Estándar**, o viceversa, se cambiará la dirección IP del servicio público y normalmente se tarda de 30 a 60 minutos en propagar el cambio a los servidores DNS en todo Internet. Es posible que no se pueda acceder al servicio antes de que se actualice el DNS. Por lo general, no se recomienda cambiar el plan de tarifa con demasiada frecuencia.


## <a name="scale-on-azure-portal"></a>Escalado en Azure Portal

1. En el explorador, abra [Azure Portal](https://portal.azure.com).

2. En la página de SignalR Service, en el menú de la izquierda, seleccione **Escalar**.
   
3. Elija el plan de tarifa y luego haga clic en **Seleccionar**. Establezca el número de unidades del nivel **Estándar**.
   
    ![Escalado en el portal](./media/signalr-howto-scale/signalr-howto-scale.png)

4. Haga clic en **Save**(Guardar).

## <a name="scale-using-azure-cli"></a>Escalado con la CLI de Azure

Este script crea un recurso de SignalR Service del nivel **Gratis** y un grupo de recursos, que escala hasta el nivel **Estándar**. 

```azurecli-interactive
#!/bin/bash

# Generate a unique suffix for the service name
let randomNum=$RANDOM*$RANDOM

# Generate a unique service and group name with the suffix
SignalRName=SignalRTestSvc$randomNum
#resource name must be lowercase
mySignalRSvcName=${SignalRName,,}
myResourceGroupName=$SignalRName"Group"

# Create resource group 
az group create --name $myResourceGroupName --location eastus

# Create the Azure SignalR Service resource
az signalr create \
  --name $mySignalRSvcName \
  --resource-group $myResourceGroupName \
  --sku Free_F1 \
  --service-mode Default

# Scale up to Standard Tier, and scale out to 50 units
az signalr update \
  --name $mySignalRSvcName \
  --resource-group $myResourceGroupName \
  --sku Standard_S1 \
  --unit-count 50
```

Anote el nombre real generado para el nuevo grupo de recursos. Usará ese nombre de grupo de recursos cuando quiera eliminar todos los recursos del grupo.

[!INCLUDE [cli-script-clean-up](../../includes/cli-script-clean-up.md)]

## <a name="compare-pricing-tiers"></a>Comparar planes de tarifa

Si necesita información detallada, como los tamaños de máquina virtual para cada plan de tarifa, consulte los [precios de SignalR Service](https://azure.microsoft.com/pricing/details/signalr-service/).

Para ver una tabla de límites, cuotas y restricciones del servicio en cada nivel, consulte los [límites de SignalR Service](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-signalr-service-limits).

## <a name="next-steps"></a>Pasos siguientes

En esta guía aprendió a escalar una única instancia de SignalR Service.

También se admiten varios puntos de conexión para escenarios de escalado, particionamiento y entre regiones.

> [!div class="nextstepaction"]
> [Escalado de SignalR Service con varias instancias](./signalr-howto-scale-multi-instances.md)
