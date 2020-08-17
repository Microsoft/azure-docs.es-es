---
title: Escalado automático en Microsoft Azure
description: Escalado automático en Microsoft Azure
ms.subservice: autoscale
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: d9ba94d9990e494a8d3e68bbcd7c176bb30e6ce2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073479"
---
# <a name="overview-of-autoscale-in-microsoft-azure"></a>Información general sobre el escalado automático en Microsoft Azure
En este artículo se explican el concepto del escalado automático de Microsoft Azure y las ventajas que aporta, y se realiza una introducción para empezar a usarlo.  

La escalabilidad automática de Azure Monitor solo se aplica a [Virtual Machine Scale Sets](https://azure.microsoft.com/services/virtual-machine-scale-sets/), [Cloud Services](https://azure.microsoft.com/services/cloud-services/), [App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/), [Servicios API Management](../../api-management/api-management-key-concepts.md) y [clústeres de Azure Data Explorer](/azure/data-explorer/).

> [!NOTE]
> Azure tiene dos métodos de escalado automático. Una versión anterior del escalado automático se aplica a Virtual Machines (conjuntos de disponibilidad). Esta característica tiene una compatibilidad limitada, por lo que, para poder usar el escalado automático de manera más rápida y fiable, recomendamos la migración a los conjuntos de escalado de máquinas virtuales. En este artículo, se incluye un vínculo sobre cómo utilizar la tecnología antigua.  
>

## <a name="what-is-autoscale"></a>¿Qué es el escalado automático?
Gracias al escalado automático, puede ejecutar la cantidad correcta de recursos para administrar la carga de la aplicación. Permite agregar recursos para controlar el aumento de la carga y ahorrar dinero mediante la eliminación de recursos inactivos. Especifique un número mínimo y máximo de instancias para ejecutar y agregue o quite máquinas virtuales automáticamente en función de un conjunto de reglas. Tener un mínimo garantiza la ejecución de la aplicación aunque no exista carga. Tener un máximo limita el posible costo total por hora. Se escala automáticamente entre estos dos extremos en función de las reglas que se creen.

 ![Explicación del escalado automático. Agregar y quitar máquinas virtuales](./media/autoscale-overview/AutoscaleConcept.png)

Cuando se cumplen las condiciones de las reglas, se desencadenan una o varias acciones de escalado automático. Puede agregar y quitar máquinas virtuales o realizar otras acciones. En el siguiente diagrama conceptual se muestra este proceso.  

 ![Diagrama de flujo de escalado automático](./media/autoscale-overview/Autoscale_Overview_v4.png)

La explicación siguiente se aplica a las partes del diagrama anterior.   

## <a name="resource-metrics"></a>Métricas de los recursos
Los recursos generan métricas, que procesan posteriormente las reglas. Las métricas proceden de métodos diferentes.
Los conjuntos de escalado de máquinas virtuales usan datos de telemetría de los agentes de Diagnósticos de Azure, mientras que la telemetría de Web Apps y Cloud Services proviene directamente de la infraestructura de Azure. Algunas de las estadísticas que se utilizan frecuentemente son el uso de CPU, el uso de memoria, el número de subprocesos, la longitud de la cola y el uso del disco. Para ver una lista de los datos de telemetría que se pueden utilizar, consulte [Métricas comunes de escalado automático de Azure Insights](autoscale-common-metrics.md).

## <a name="custom-metrics"></a>Métricas personalizadas
También puede usar métricas personalizadas que las aplicaciones pueden generar. Si ha configurado las aplicaciones para que envíen métricas a Application Insights, puede usarlas para tomar decisiones sobre si necesita escalar o no.

## <a name="time"></a>Time
Las reglas basadas en programaciones emplean el huso horario UTC. Debe establecer la zona horaria correctamente al configurar las reglas.  

## <a name="rules"></a>Reglas
El diagrama solo muestra una única regla de escalado automático, pero puede tener muchas. Puede crear reglas complejas de superposición según su situación.  Estos son algunos de los tipos de reglas:  

* **Basadas en métricas** : por ejemplo, realizar esta acción cuando el uso de CPU sea superior al 50 %.
* **Basadas en tiempo** : por ejemplo, desencadenar un webhook todos los sábados a las 8:00 en una zona horaria determinada.

Las reglas basadas en métricas miden la carga de la aplicación y agregan o quitan máquinas virtuales en función de ella. Las reglas de programación permiten escalar al ver los modelos de tiempo de la carga, si quiere escalar antes de un posible aumento de carga o si esta disminuye.  

## <a name="actions-and-automation"></a>Acciones y escalado automático
Las reglas pueden desencadenar uno o varios tipos de acciones.

* **Escalar** : escalar o reducir horizontalmente máquinas virtuales
* **Enviar correo electrónico** : enviar correo electrónico a los administradores y coadministradores de la suscripción, así como a otras direcciones de correo electrónico que especifique
* **Automate via webhooks** (Automatizar mediante webhooks): llamar a webhooks, que pueden desencadenar varias acciones dentro o fuera de Azure. Dentro de Azure, puede iniciar runbooks de Azure Automation, Azure Function o Azure Logic Apps. La dirección URL de terceros de ejemplo fuera de Azure incluye servicios como Slack y Twilio.

## <a name="autoscale-settings"></a>Configuración de escalado automático
El escalado automático utiliza la siguiente terminología y estructura.

- El motor de escalado automático lee la **configuración de escalado automático** para determinar si se debe escalar hacia arriba o hacia abajo. Contiene uno o más perfiles, información sobre el recurso de destino y la configuración de las notificaciones.

  - Un **perfil de escalado automático** es una combinación de:

    - Una **configuración de capacidad**, que indica los valores mínimos, máximos y predeterminados para el número de instancias.
    - Un **conjunto de reglas**, cada una de las cuales incluye un desencadenador (tiempo o métrica) y una acción de escalado (escalado o reducción vertical).
    - Una **periodicidad**, que indica cuándo el escalado automático debe hacer entrar en vigor el perfil.

      Puede tener varios perfiles, lo cual le permitirá ocuparse de diferentes requisitos coincidentes. Puede tener perfiles de escalado automático diferentes, por ejemplo, para distintos momentos del día o días de la semana.

  - La **configuración de las notificaciones** define qué notificaciones deben aparecer cuando se produce un evento de escalado automático en función de los criterios de los perfiles de configuración de escalado automático que cumpla. Con el escalado automático se pueden notificar a una o más direcciones de correo electrónico o realizar llamadas a uno o más webhooks.


![Configuración, perfil y estructura de las reglas de escalado automático de Azure](./media/autoscale-overview/AzureResourceManagerRuleStructure3.png)

La lista íntegra de campos y descripciones configurables se encuentra en la [API de REST de escalado automático](/rest/api/monitor/autoscalesettings).

Para ver ejemplos de código, consulte los siguientes artículos:

* [Configuración avanzada de escalado automático con plantillas de Resource Manager para VM Scale Sets](autoscale-virtual-machine-scale-sets.md)  
* [API de REST de escalado automático](/rest/api/monitor/autoscalesettings)

## <a name="horizontal-vs-vertical-scaling"></a>Escalado horizontal frente a escalado vertical
El escalado automático solo escala horizontalmente, que es un aumento o una reducción del número de instancias de máquina virtual.  El escalado horizontal resulta más flexible en un entorno en la nube, ya que puede llegar a ejecutar miles de máquinas virtuales para administrar la carga.

En contraste, el escalado vertical es diferente. Mantiene el mismo número de máquinas virtuales, pero hace que sean más o menos potentes. La potencia se mide en memoria, velocidad de CPU, espacio en disco, etc.  El escalado vertical tiene más limitaciones, ya que depende de la disponibilidad de hardware de mayor tamaño, que supera el límite rápidamente y puede variar según la región. El escalado vertical también suele requerir que se detenga y reinicie una máquina virtual.

## <a name="methods-of-access"></a>Métodos de acceso
Puede configurar el escalado automático en los siguientes lugares:

* [Azure Portal](autoscale-get-started.md)
* [PowerShell](../samples/powershell-samples.md#create-and-manage-autoscale-settings)
* [Interfaz de línea de comandos (CLI) multiplataforma](../samples/cli-samples.md#autoscale)
* [API de REST de Azure Monitor](/rest/api/monitor/autoscalesettings)

## <a name="supported-services-for-autoscale"></a>Servicios compatibles con el escalado automático
| Servicio | Esquema y documentos |
| --- | --- |
| Web Apps |[Escalado en Web Apps](autoscale-get-started.md) |
| Cloud Services |[Escalado automático de un servicio en la nube](../../cloud-services/cloud-services-how-to-scale-portal.md) |
| Virtual Machines: Clásico |[Escalado de conjuntos de disponibilidad clásicos de máquina virtual](/archive/blogs/kaevans/autoscaling-azurevirtual-machines) |
| Virtual Machines: Conjuntos de escalado de Windows |[Escalado de conjuntos de escalado de máquinas virtuales en Windows](../../virtual-machine-scale-sets/tutorial-autoscale-powershell.md) |
| Virtual Machines: Conjuntos de escalado de Linux |[Escalado de conjuntos de escalado de máquinas virtuales en Linux](../../virtual-machine-scale-sets/tutorial-autoscale-cli.md) |
| Virtual Machines: Ejemplo de Windows |[Configuración avanzada de escalado automático con plantillas de Resource Manager para VM Scale Sets](autoscale-virtual-machine-scale-sets.md) |
| Servicio API Management|[Escalado automático de una instancia de Azure API Management](../../api-management/api-management-howto-autoscale.md)
| Clústeres de Azure Data Explorer|[Administración del escalado de clústeres de Azure Data Explorer para ajustarse a los cambios en la demanda](/azure/data-explorer/manage-cluster-horizontal-scaling)|
| Azure App Service |[Escalado de una aplicación en Azure App Service](../../app-service/manage-scale-up.md)|
| Logic Apps |[Incorporación de capacidad del Entorno del servicio de integración (ISE)](../../logic-apps/ise-manage-integration-service-environment.md#add-ise-capacity)|
## <a name="next-steps"></a>Pasos siguientes
Para más información sobre el escalado automático, consulte los tutoriales de escalado automático anteriores o los siguientes recursos:

* [Métricas comunes de escalado automático de Azure Monitor](autoscale-common-metrics.md)
* [Procedimientos recomendados de escalado automático en Azure Monitor](autoscale-best-practices.md)
* [Uso de acciones de escalado automático para enviar notificaciones de alerta por correo electrónico y Webhook en Azure Insights](autoscale-webhook-email.md)
* [API de REST de escalado automático](/rest/api/monitor/autoscalesettings)
* [Solución de problemas de escalado automático de conjuntos de escalado de máquinas virtuales](../../virtual-machine-scale-sets/virtual-machine-scale-sets-troubleshoot.md)
