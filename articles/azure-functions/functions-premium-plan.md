---
title: Plan de funciones Premium de Azure (versión preliminar) | Microsoft Docs
description: Planean los detalles y opciones de configuración (red virtual, no en frío, la duración de ejecución ilimitado) para las funciones de Azure Premium.
services: functions
author: jeffhollan
manager: jeconnoc
ms.assetid: ''
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 4/11/2019
ms.author: jehollan
ms.openlocfilehash: 75987e7cba9f373af5a434de9f273948c3c9ae75
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544766"
---
# <a name="azure-functions-premium-plan-preview"></a>Plan de funciones Premium de Azure (versión preliminar)

El plan Premium de Azure Functions es una opción de hospedaje para aplicaciones de función. El plan Premium proporciona características como la conectividad de red virtual, no arranque en frío y hardware de primera calidad.  Se pueden implementar varias aplicaciones de función en el mismo plan Premium, y el plan le permite configurar el tamaño de la instancia de proceso, el tamaño de plan base y el tamaño máximo del plan.  Para obtener una comparación del plan Premium y otro plan y tipos de hospedaje, vea [función escalado y las opciones de hospedaje](functions-scale.md).

> [!NOTE]
> La vista previa del plan Premium admite actualmente las funciones que se ejecutan en. NET, nodo o Java a través de la infraestructura de Windows.

## <a name="create-a-premium-plan"></a>Crear un plan Premium

[!INCLUDE [functions-premium-create](../../includes/functions-premium-create.md)]

También puede crear un plan Premium de la CLI de Azure

```azurecli-interactive
az functionapp plan create -g <resource-group> -n <plan-name> -l <region> --number-of-workers 1 --sku EP1
```

## <a name="features"></a>Características

Las siguientes características están disponibles para las aplicaciones de función implementadas en un plan Premium.

### <a name="pre-warmed-instances"></a>Instancias calienten previamente

Si no hay eventos y las ejecuciones se producen hoy mismo en el plan de consumo, la aplicación puede reducir verticalmente a cero instancias. Cuando se conectan nuevos eventos, una nueva instancia debe haberse especializado con la aplicación se ejecute en él.  Especializado nuevas instancias pueden tardar algún tiempo dependiendo de la aplicación.  Esta latencia adicional en la primera llamada a menudo se denomina inicio en frío de la aplicación.

En el plan Premium, puede tener la aplicación previamente preparada en un número especificado de instancias, hasta el tamaño mínimo del plan.  Instancias previamente calienten también le permiten escalar previamente una aplicación antes de una carga elevada. Como la aplicación se escala horizontalmente, primero se escala en las instancias calienten previamente. Continuarán instancias adicionales para almacenar en búfer caliente inmediatamente en preparación para la siguiente operación de escalado y horizontal. Al tener un búfer de instancias calienten previamente, puede evitar eficazmente las latencias de arranque en frío.  Instancias previamente calienten es una característica del plan Premium y debe tener al menos una instancia en ejecución y disponible en todo momento el plan está activo.

Puede configurar el número de instancias calienten previamente en Azure portal, seleccione su **Function App**, va a la **características de la plataforma** pestaña y seleccionando el **escalabilidad horizontal**opciones. En la ventana de edición de aplicación de función, instancias previamente calienten es específico de esa aplicación, pero las instancias mínimas y máxima se aplican a todo el plan.

![Configuración de escalado elástico](./media/functions-premium-plan/scale-out.png)

También puede configurar previamente calienten instancias para una aplicación con la CLI de Azure

```azurecli-interactive
az resource update -g <resource_group> -n <function_app_name>/config/web --set properties.preWarmedInstanceCount=<desired_prewarmed_count> --resource-type Microsoft.Web/sites
```

### <a name="private-network-connectivity"></a>Conectividad de red privada

Aprovecha las funciones de Azure implementadas en un plan Premium de [nueva integración con red virtual para las aplicaciones web](../app-service/web-sites-integrate-with-vnet.md#new-vnet-integration).  Cuando se configura, la aplicación puede comunicarse con recursos de su red virtual o protegido a través de extremos de servicio.  Las restricciones de IP también están disponibles en la aplicación para restringir el tráfico entrante.

Al asignar una subred a la aplicación de función en un plan Premium, necesitará una subred con suficientes direcciones IP para cada instancia posibles. Aunque el número máximo de instancias puede variar durante la versión preliminar, es necesario un bloque de IP con al menos 100 direcciones disponibles.

Para obtener más información, vea [integrar la aplicación de función con una red virtual](functions-create-vnet.md).

### <a name="rapid-elastic-scale"></a>Escala elástica rápida

Las instancias de proceso adicionales se agregan automáticamente para su aplicación mediante la misma lógica de escalado rápida como el plan de consumo.  Para obtener más información acerca de cómo escala funciona, vea [función escalado y hospedaje](./functions-scale.md#how-the-consumption-and-premium-plans-work).

### <a name="unbounded-run-duration"></a>Duración de la ejecución de unbounded

Azure Functions en un plan de consumo se limitan a 10 minutos para una sola ejecución.  En el plan Premium, la duración de ejecución predeterminado es 30 minutos para evitar que las ejecuciones descontroladas. Sin embargo, puede [modificar la configuración de host.json](./functions-host-json.md#functiontimeout) para que sea ilimitado para las aplicaciones de plan Premium.

En versión preliminar, la duración no se garantiza que más allá de 12 minutos y tendrá la mejor oportunidad de ejecutar más de 30 minutos si la aplicación no se escala más allá de su recuento mínimo de trabajo.

## <a name="plan-and-sku-settings"></a>Configuración de plan y SKU

Cuando se crea el plan, configure dos opciones de configuración: el número mínimo de instancias (o tamaño de plan) y el límite máximo de ráfaga.  Las instancias de un plan Premium mínimas es 1, y la máxima durante la versión preliminar es 20.  Número mínimo de instancias está reservados y siempre en ejecución.

> [!IMPORTANT]
> Se le cobrará por cada instancia asignada en el número mínimo de instancias independientemente de si se están ejecutando las funciones o no.

Si su aplicación requiere instancias más allá de su tamaño de plan, igualmente puede seguir escalar horizontalmente hasta que el número de instancias alcanza el límite máximo de ráfaga.  Se le facturará para instancias más allá de su tamaño de plan solo mientras se estén ejecutando y alquiladas para usted.  Haremos todo lo posible al escalar la aplicación horizontalmente a su límite máximo definido, mientras que se garantiza que las instancias de plan mínimo para la aplicación.

Puede configurar el tamaño de plan y seleccionar valores máximos en el portal de Azure mediante el **escalabilidad horizontal** opciones en el plan o una aplicación de función implementadas en ese plan (bajo **características de la plataforma**).

También puede aumentar el límite máximo de ráfaga de la CLI de Azure:

```azurecli-interactive
az resource update -g <resource_group> -n <premium_plan_name> --set properties.maximumElasticWorkerCount=<desired_max_burst> --resource-type Microsoft.Web/serverfarms 
```

### <a name="available-instance-skus"></a>SKU de instancia disponible

Al crear nuestra su plan de capacidad de escalado, puede elegir entre tres tamaños de instancia.  Se le facturará el número total de núcleos y memoria consumida por segundo.  La aplicación automáticamente puede escalar horizontalmente a varias instancias según sea necesario.  

|SKU|Núcleos|Memoria|Almacenamiento|
|--|--|--|--|
|EP1|1|3.5GB|250 GB|
|EP2|2|7 GB|250 GB|
|EP3|4|14 GB|250 GB|

## <a name="regions"></a>Regiones

A continuación se muestran las regiones admitidas actualmente para la versión preliminar pública.

|Área|
|--|
|Este de Australia|
|Sudeste de Australia|
|Centro de Canadá|
|Centro de EE. UU.|
|Asia oriental|
|Este de EE. UU. 2|
|Centro de Francia|
|Oeste de Japón|
|Corea Central|
|Centro-Norte de EE. UU|
|Norte de Europa|
|Centro y sur de EE. UU.|
|Sur de la India|
|Sudeste asiático|
|Oeste de Reino Unido|
|Europa occidental|
|Oeste de la India|
|Oeste de EE. UU.|

## <a name="known-issues"></a>Problemas conocidos

Puede realizar un seguimiento del estado de los problemas conocidos de la [versión preliminar pública en GitHub](https://github.com/Azure/Azure-Functions/wiki/Premium-plan-known-issues).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Comprender la escala de Azure Functions y las opciones de hospedaje](functions-scale.md)
