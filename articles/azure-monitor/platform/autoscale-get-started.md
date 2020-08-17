---
title: Introducción al escalado automático en Azure
description: Obtenga información sobre cómo escalar Web Apps, Cloud Services, Virtual Machines o Virtual Machine Scale Set del recurso en Azure.
ms.topic: conceptual
ms.date: 07/07/2017
ms.subservice: autoscale
ms.openlocfilehash: 67b041476ecc5b5da389ab1377025a94675fc42a
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88078893"
---
# <a name="get-started-with-autoscale-in-azure"></a>Introducción al escalado automático en Azure
Este artículo describe cómo configurar el escalado automático de recursos en Microsoft Azure Portal.

La escalabilidad automática de Azure Monitor solo se aplica a [Virtual Machine Scale Sets](https://azure.microsoft.com/services/virtual-machine-scale-sets/), [Cloud Services](https://azure.microsoft.com/services/cloud-services/), [App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/) y los [servicios de API Management](../../api-management/api-management-key-concepts.md).

## <a name="discover-the-autoscale-settings-in-your-subscription"></a>Detección de la configuración de escalado automático en la suscripción

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4u7ts]

Puede detectar todos los recursos a los que se aplica el escalado automático en Azure Monitor. Siga los pasos siguientes para obtener un tutorial paso a paso:

1. Abra [Azure Portal][1].
1. Haga clic en el icono de Azure Monitor en el panel izquierdo.
  ![Abrir Azure Monitor][2]
1. Haga clic en **Escalado automático** para ver todos los recursos para los que se puede aplicar el escalado automático, junto con su estado de escalado automático actual.
  ![Detectar el escalado automático en Azure Monitor][3]

Puede utilizar el panel de filtro de la parte superior para limitar la lista a los recursos seleccionados de un grupo de recursos específico, tipos de recursos concretos o un recurso determinado.

Para cada recurso, encontrará el recuento de instancias actual y el estado de escalado automático. El estado de escalado automático puede ser:

- **No configurado**: aún no se ha habilitado el escalado automático para este recurso.
- **Habilitado**: se ha habilitado el escalado automático para este recurso.
- **Disabled**: se ha deshabilitado el escalado automático para este recurso.

## <a name="create-your-first-autoscale-setting"></a>Creación de la primera configuración de escalado automático

Se va a ofrecer ahora un tutorial paso a paso sencillo sobre cómo crear la primera configuración de escalado automático.

1. Abra la hoja **Escalado automático** en Azure Monitor y seleccione un recurso que desee escalar (en los pasos siguientes se usa un plan de App Service asociado con una aplicación web. Puede [crear su primera aplicación web de ASP.NET en Azure en cinco minutos][4]).
1. Tenga en cuenta que el número actual de instancias es 1. Haga clic en **Enable autoscale** (Habilitar escalado automático).
  ![Configuración de escalado para la nueva aplicación web][5]
1. Proporcione un nombre para la configuración de escalado y haga clic en **Agregar una regla**. Tenga en cuenta que las opciones de la regla de escalado se abren como un panel Contexto en el lado derecho. De forma predeterminada, establece la opción para escalar el recuento de instancias en 1 si el porcentaje de CPU del recurso supera el 70 %. Deje los valores predeterminados y haga clic en **Agregar**.
  ![Crear configuración de escalado para una aplicación web][6]
1. Ahora ha creado su primera regla de escalado. Tenga en cuenta que UX ofrece procedimientos recomendados e indica que "se recomienda tener al menos una regla de reducción horizontal". Para ello:

    a. Haga clic en **Agregar una regla**.

    b. Establezca **Operador** en **Menos que**.

    c. Establezca **Umbral** en **20**.

    d. Establezca **Operación** en **Reducir recuento en**.

   Ahora debería tener una configuración de escalado que escale o reduzca horizontalmente en función del uso de CPU.
   ![Escala en función de la CPU][8]
1. Haga clic en **Save**(Guardar).

Felicidades. Ya ha establecido correctamente la primera configuración del escalado para escalar automáticamente la aplicación web en función del uso de CPU.

> [!NOTE]
> Los mismos pasos son aplicables para empezar a trabajar con un rol de servicio en la nube o un conjunto de escalado de máquinas virtuales.

## <a name="other-considerations"></a>Otras consideraciones
### <a name="scale-based-on-a-schedule"></a>Escalación según una programación
Además de escalar en función de la CPU, puede establecer la escala de forma diferente para días específicos de la semana.

1. Haga clic en **Add a scale condition** (Agregar una condición de escalado).
1. Establezca el modo y las reglas de escalado de la misma forma que la condición predeterminada.
1. Seleccione **Repeat specific days** (Repetir días específicos) para la programación.
1. Seleccione los días y la hora de inicio y fin en que debe aplicarse la condición de escalado.

![Condición de escalado basada en programación][9]
### <a name="scale-differently-on-specific-dates"></a>Escalado distinto en fechas concretas
Además de escalar en función de la CPU, puede establecer la escala de forma diferente para fechas específicas.

1. Haga clic en **Add a scale condition** (Agregar una condición de escalado).
1. Establezca el modo y las reglas de escalado de la misma forma que la condición predeterminada.
1. Seleccione **Specify start/end dates** (Especificar fechas de inicio y fin) para la programación.
1. Seleccione las fechas y la hora de inicio y fin en que debe aplicarse la condición de escalado.

![Condición de escalado basada en fechas][10]

### <a name="view-the-scale-history-of-your-resource"></a>Consulta del historial de escalado de un recurso
Cada vez que un recurso se escala o reduce verticalmente, se registra un evento en el registro de actividad. Puede consultar el historial de escalado del recurso de las últimas 24 horas si cambia a la pestaña **Historial de ejecución**.

![Historial de ejecuciones][11]

Si desea consultar el historial de escalado completo (de hasta 90 días), seleccione **Haga clic aquí para ver más detalles**. Se abre el registro de actividad, con el escalado automático previamente seleccionado para el recurso y la categoría.

### <a name="view-the-scale-definition-of-your-resource"></a>Consulta de la definición de escalado del recurso
El escalado automático es un recurso de Azure Resource Manager. Puede ver la definición de escala en JSON al cambiar a la pestaña **JSON**.

![Definición de escala][12]

Puede realizar cambios en JSON directamente, si es necesario. Estos cambios se reflejarán después de guardarlos.

### <a name="disable-autoscale-and-manually-scale-your-instances"></a>Deshabilitar el escalado automático y escalar manualmente las instancias
Pueden darse ocasiones en que desee deshabilitar la configuración actual de escalado y escalar los recursos manualmente.

Haga clic en el botón **Deshabilitar escalado automático** en la parte superior.
![Deshabilitar escalado automático][13]

> [!NOTE]
> Esta opción deshabilita su configuración. Sin embargo, puede volver a ella después de habilitar de nuevo el escalado automático.

Ahora puede establecer el número de instancias a las que desea escalar manualmente.

![Definición manual de escalado][14]

Siempre se puede volver al escalado automático; para ello, haga clic en **Enable autoscale** (Habilitar escalado automático) y luego haga clic en **Save** (Guardar).

## <a name="next-steps"></a>Pasos siguientes
- [Creación de una alerta de registro de actividades para supervisar todas las operaciones del motor de escalado automático en su suscripción](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-alert)
- [Creación de una alerta de registro de actividades para supervisar todas las operaciones erróneas de escalado automático y reducción horizontal en su suscripción](https://github.com/Azure/azure-quickstart-templates/tree/master/monitor-autoscale-failed-alert)

<!--Reference-->
[1]:https://portal.azure.com
[2]: ./media/autoscale-get-started/azure-monitor-launch.png
[3]: ./media/autoscale-get-started/discover-autoscale-azure-monitor.png
[4]: ../../app-service/quickstart-dotnetcore.md
[5]: ./media/autoscale-get-started/scale-setting-new-web-app.png
[6]: ./media/autoscale-get-started/create-scale-setting-web-app.png
[7]: ./media/autoscale-get-started/scale-in-recommendation.png
[8]: ./media/autoscale-get-started/scale-based-on-cpu.png
[9]: ./media/autoscale-get-started/scale-condition-schedule.png
[10]: ./media/autoscale-get-started/scale-condition-dates.png
[11]: ./media/autoscale-get-started/scale-history.png
[12]: ./media/autoscale-get-started/scale-definition-json.png
[13]: ./media/autoscale-get-started/disable-autoscale.png
[14]: ./media/autoscale-get-started/set-manualscale.png
