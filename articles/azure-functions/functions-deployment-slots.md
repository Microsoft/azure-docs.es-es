---
title: Ranuras de implementación de Azure Functions
description: Aprenda a crear y a usar ranuras de implementación con Azure Functions.
author: craigshoemaker
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: cshoe
ms.openlocfilehash: f84dc17c6c074fc4dbda8a13fad3586a397fdf10
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87055429"
---
# <a name="azure-functions-deployment-slots"></a>Ranuras de implementación de Azure Functions

Las ranuras de implementación de Azure Functions permiten que la aplicación de funciones ejecute instancias diferentes conocidas como "ranuras". Las ranuras son entornos diferentes que se exponen a través de un punto de conexión disponible públicamente. Una instancia de la aplicación siempre se asigna a la ranura de producción, y se pueden intercambiar instancias asignadas a una ranura a petición. Las aplicaciones de funciones que se ejecutan en el plan de App Service pueden tener varias ranuras, mientras que en el plan de consumo solo se permite una ranura.

Aquí se indica cómo se ven afectadas las funciones cuando se intercambian ranuras:

- El redireccionamiento del tráfico es perfecto y no se pierde ninguna solicitud debido al intercambio.
- Si una función se está ejecutando durante un intercambio, la ejecución continúa y los desencadenadores posteriores se enrutan a la instancia de la aplicación intercambiada.

> [!NOTE]
> Actualmente no hay ranuras disponibles para el plan Consumo para Linux.

## <a name="why-use-slots"></a>¿Por qué usar ranuras?

El uso de ranuras de implementación reporta algunas ventajas. En los siguientes escenarios se describen los usos comunes de las ranuras:

- **Entornos diferentes con distintos fines**: El uso de ranuras distintas ofrece la posibilidad de diferenciar las instancias de la aplicación antes de cambiar a producción o a un espacio de ensayo.
- **Preparación**: implementar en una ranura en lugar de implementar directamente en producción permite que la aplicación se prepare para empezar. Además, el uso de ranuras reduce la latencia de las cargas de trabajo desencadenadas mediante HTTP. Las instancias se preparan antes de la implementación, lo que reduce el arranque en frío de las funciones recién implementadas.
- **Reservas sencillas**: después del intercambio con producción, la ranura con la aplicación de ensayo anterior ocupa ahora la aplicación de producción anterior. Si las modificaciones que se han intercambiado en el espacio de producción no son las previstas, puede revertir el intercambio inmediatamente para volver a tener la "última instancia que sabe que funciona correctamente".

## <a name="swap-operations"></a>Operaciones de intercambio

Durante un intercambio, una ranura se considera el origen y la otra, el destino. La ranura de origen tiene la instancia de la aplicación que se aplica a la ranura de destino. Con los siguientes pasos se evita que la ranura de destino experimente tiempos de inactividad durante un intercambio:

1. **Aplicar la configuración:** la configuración de la ranura de destino se aplica a todas las instancias de la ranura de origen. Por ejemplo, la configuración de producción se aplica a la instancia de ensayo. La configuración aplicada incluye las siguientes categorías:
    - Opciones de configuración de aplicación y cadenas de conexión [específicas de la ranura](#manage-settings) (si procede)
    - Configuración de [implementación continua](../app-service/deploy-continuous-deployment.md) (si está habilitada)
    - Configuración de [autenticación de App Service](../app-service/overview-authentication-authorization.md) configuración (si está habilitada)

1. **Esperar reinicios y disponibilidad:** el intercambio espera a que todas las instancias de la ranura de origen terminen de reiniciarse y estén disponibles para las solicitudes. Si el reinicio no se puede realizar en alguna instancia, el intercambio revierte todos los cambios en la ranura de origen y detiene la operación.

1. **Actualizar el enrutamiento:** si todas las instancias de la ranura de origen se han preparado correctamente, las dos ranuras completan el intercambio cambiando sus reglas de enrutamiento correspondientes. Después de este paso, la ranura de destino (por ejemplo, la de producción) tendrá la aplicación que se preparó previamente en la ranura de origen.

1. **Repetir la operación:** Ahora que la ranura de origen tiene la aplicación que se preparó previamente antes del intercambio, complete la misma operación, es decir, aplique la configuración y reinicie las instancias de la ranura de origen.

Tenga en cuenta los siguientes puntos:

- Durante intercambio, la inicialización de las aplicaciones intercambiadas tiene lugar en la ranura de origen. La ranura de destino permanece en línea mientras la de origen se prepara, independientemente de si el intercambio se realiza o no correctamente.

- Para intercambiar una ranura de ensayo con la de producción, asegúrese de que esta última es *siempre* la de destino. De este modo, el intercambio no afecta a la aplicación de producción.

- La configuración relativa a los enlaces y orígenes de eventos se debe definir como [configuración de la ranura de implementación](#manage-settings) *antes de iniciar el intercambio*. Al marcarlos como "permanentes", se garantiza que los eventos y las salidas se dirigen a la instancia adecuada.

## <a name="manage-settings"></a>Administración de la configuración

[!INCLUDE [app-service-deployment-slots-settings](../../includes/app-service-deployment-slots-settings.md)]

### <a name="create-a-deployment-setting"></a>Creación de una configuración de implementación

Puede marcar una configuración como configuración de la implementación, lo que la convierte en "permanente". Una configuración permanente no se intercambia con la instancia de la aplicación.

Si crea una configuración de implementación en una ranura, asegúrese de crear la misma configuración con un valor único en cualquier otra ranura que participe en un intercambio. De este modo, mientras el valor de una configuración no cambie, los nombres de las configuraciones seguirán siendo los mismos en todas las ranuras. Esta coherencia de nombre evita que el código intente tener acceso a una configuración que está definida en una ranura, pero no en otra.

Use los siguientes pasos para crear una configuración de implementación:

1. Vaya a **Ranuras de implementación** en la aplicación de funciones y, luego, seleccione el nombre de la ranura.

    :::image type="content" source="./media/functions-deployment-slots/functions-navigate-slots.png" alt-text="Busque ranuras en Azure Portal." border="true":::

1. Seleccione **Configuración** y luego seleccione el nombre de la configuración que quiera hacer permanente en la ranura actual.

    :::image type="content" source="./media/functions-deployment-slots/functions-configure-deployment-slot.png" alt-text="Configure la configuración de la aplicación para una ranura en Azure Portal." border="true":::

1. Seleccione **Configuración de ranura de implementación** y luego seleccione **Aceptar**.

    :::image type="content" source="./media/functions-deployment-slots/functions-deployment-slot-setting.png" alt-text="Configure la configuración de ranura de implementación." border="true":::

1. Cuando la sección de configuración desaparezca, seleccione **Guardar** para conservar los cambios

    :::image type="content" source="./media/functions-deployment-slots/functions-save-deployment-slot-setting.png" alt-text="Guarde la configuración de ranura de implementación." border="true":::

## <a name="deployment"></a>Implementación

Cuando se crean ranuras, estas están vacías. Puede utilizar cualquiera de las [tecnologías de implementación admitidas](./functions-deployment-technologies.md) para implementar la aplicación en una ranura.

## <a name="scaling"></a>Ampliación

Todas las ranuras escalan al mismo número de trabajos que la ranura de producción.

- En el caso de los planes de consumo, la ranura escala a medida que lo hace la aplicación de funciones.
- En el caso de los planes de App Service, la aplicación escala a un número fijo de trabajos. Las ranuras se ejecutan en el mismo número de trabajos que el plan de la aplicación.

## <a name="add-a-slot"></a>Incorporación de una ranura

Una ranura se puede agregar a través de la [CLI](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-create) o mediante el portal. En los siguientes pasos se describe cómo crear una ranura en el portal:

1. Vaya a la aplicación de funciones.

1. Seleccione **Ranuras de implementación**y, luego, seleccione **+ Agregar ranura**.

    :::image type="content" source="./media/functions-deployment-slots/functions-deployment-slots-add.png" alt-text="Agregue una ranura de implementación de Azure Functions." border="true":::

1. Escriba el nombre de la ranura y seleccione **Agregar**.

    :::image type="content" source="./media/functions-deployment-slots/functions-deployment-slots-add-name.png" alt-text="Asigne un nombre a la ranura de implementación de Azure Functions." border="true":::

## <a name="swap-slots"></a>Intercambio de ranuras

Las ranuras se pueden intercambiar a través de la [CLI](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-swap) o mediante el portal. En los siguientes pasos se describe cómo intercambiar ranuras en el portal:

1. Vaya a la aplicación de función.
1. Seleccione **Ranuras de implementación**y, luego, seleccione **Intercambiar**.

    :::image type="content" source="./media/functions-deployment-slots/functions-swap-deployment-slot.png" alt-text="Intercambie la ranura de implementación." border="true":::

1. Compruebe los valores de configuración del intercambio y seleccione **Intercambiar**
    
    :::image type="content" source="./media/functions-deployment-slots/azure-functions-deployment-slots-swap-config.png" alt-text="Intercambie la ranura de implementación." border="true":::

La operación puede tardar un rato mientras la operación de intercambio se lleva a cabo.

## <a name="roll-back-a-swap"></a>Reversión de intercambios

Si un intercambio genera errores o simplemente quiere "deshacer" un intercambio, puede revertir al estado inicial. Para volver al estado previo al intercambio, realice otro intercambio para invertirlo.

## <a name="remove-a-slot"></a>Eliminación de una ranura

Una ranura se puede quitar a través de la [CLI](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-delete) o mediante el portal. En los siguientes pasos se describe cómo quitar una ranura en el portal:

1. Vaya a **Ranuras de implementación** en la aplicación de funciones y, luego, seleccione el nombre de la ranura.

    :::image type="content" source="./media/functions-deployment-slots/functions-navigate-slots.png" alt-text="Busque ranuras en Azure Portal." border="true":::

1. Seleccione **Eliminar**.

    :::image type="content" source="./media/functions-deployment-slots/functions-delete-deployment-slot.png" alt-text="Elimine la ranura de implementación en Azure Portal." border="true":::

1. Escriba el nombre de la ranura de implementación que desea eliminar y, luego, seleccione **Eliminar**.

    :::image type="content" source="./media/functions-deployment-slots/functions-delete-deployment-slot-details.png" alt-text="Elimine la ranura de implementación en Azure Portal." border="true":::

1. Cierre el panel de confirmación de eliminación.

    :::image type="content" source="./media/functions-deployment-slots/functions-deployment-slot-deleted.png" alt-text="Confirmación de eliminación de la ranura de implementación." border="true":::

## <a name="automate-slot-management"></a>Automatización de la administración de ranuras

Con la [CLI de Azure](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest) se pueden automatizar las siguientes acciones en una ranura:

- [crear](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-create)
- [delete](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-delete)
- [list](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-list)
- [swap](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-swap)
- [auto-swap](/cli/azure/functionapp/deployment/slot?view=azure-cli-latest#az-functionapp-deployment-slot-auto-swap)

## <a name="change-app-service-plan"></a>Cambio del plan de App Service

En una aplicación de funciones que se ejecuta en un plan de App Service, puede cambiar el plan de App Service subyacente de una ranura.

> [!NOTE]
> No se puede cambiar el plan de App Service de una ranura que está en el plan de consumo.

Haga lo siguiente para cambiar el plan de App Service de una ranura:

1. Vaya a **Ranuras de implementación** en la aplicación de funciones y, luego, seleccione el nombre de la ranura.

    :::image type="content" source="./media/functions-deployment-slots/functions-navigate-slots.png" alt-text="Busque ranuras en Azure Portal." border="true":::

1. En **Plan de App Service**, seleccione **Cambiar el plan de App Service**.

1. Seleccione el plan al que desea cambiarse o cree un nuevo plan.

    :::image type="content" source="./media/functions-deployment-slots/azure-functions-deployment-slots-change-app-service-apply.png" alt-text="Cambie el plan de App Service en Azure Portal." border="true":::

1. Seleccione **Aceptar**.

## <a name="limitations"></a>Limitaciones

Las ranuras de implementación de Azure Functions presentan las siguientes limitaciones:

- El número de ranuras disponibles para una aplicación depende del plan. El plan de consumo solo se permite en una ranura de implementación. Hay más ranuras disponibles para las aplicaciones que se ejecutan en el plan de App Service.
- Al intercambiar una ranura, se restablecen las claves de las aplicaciones que tienen una configuración de aplicación `AzureWebJobsSecretStorageType` igual a `files`.
- No hay ranuras disponibles para el plan Consumo para Linux.

## <a name="support-levels"></a>Niveles de compatibilidad

Hay dos niveles de compatibilidad para las ranuras de implementación:

- **Disponibilidad general (GA)** : significa que es totalmente compatible y está aprobada para su uso en producción.
- **Versión preliminar**: aún no cuenta con soporte pero se espera que llegue al estado de disponibilidad general en el futuro.

| Sistema operativo o plan de hospedaje           | Nivel de compatibilidad     |
| ------------------------- | -------------------- |
| Consumo de Windows       | Disponibilidad general |
| Windows Premium           | Disponibilidad general  |
| Dedicado de Windows         | Disponibilidad general |
| Consumo de Linux         | No compatible          |
| Linux Premium             | Disponibilidad general  |
| Dedicado de Linux           | Disponibilidad general |

## <a name="next-steps"></a>Pasos siguientes

- [Tecnologías de implementación en Azure Functions](./functions-deployment-technologies.md)
