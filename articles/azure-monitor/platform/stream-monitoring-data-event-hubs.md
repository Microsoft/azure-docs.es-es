---
title: Transmisión de datos de supervisión de Azure a un centro de eventos y asociados externos
description: Obtenga información sobre cómo hacer streaming de los datos de supervisión de Azure a un centro de eventos para que una herramienta de un asociado de Administración de eventos e información de seguridad o de análisis puedan disponer de ellos.
services: azure-monitor
author: bwren
ms.author: bwren
ms.topic: conceptual
ms.date: 07/15/2020
ms.subservice: ''
ms.openlocfilehash: f6272e3d976c7c3b04d5b1332e2d7b3410c3045c
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318885"
---
# <a name="stream-azure-monitoring-data-to-an-event-hub-or-external-partner"></a>Transmisión de datos de supervisión de Azure a un centro de eventos o asociado externo

Azure Monitor proporciona una solución completa de supervisión de pila para aplicaciones y servicios en Azure, en otras nubes y locales. Asimismo, se puede usar Azure Monitor para analizar esos datos y aprovecharlos en diferentes escenarios de supervisión, aunque es posible que deba enviarlos a otras herramientas de supervisión de su entorno. En la mayoría de los casos, el método más efectivo para transmitir datos de supervisión a herramientas externas, es usar [Azure Event Hubs](../../event-hubs/index.yml). En este artículo se proporciona una breve descripción de cómo hacerlo y luego se enumeran algunos de los asociados en los que puede enviar datos. Algunos tienen una integración especial con Azure Monitor y se pueden hospedar en Azure.  

## <a name="create-an-event-hubs-namespace"></a>Creación de un espacio de nombres de Event Hubs

Antes de configurar la transmisión de cualquier origen de datos, debe [crear un espacio de nombres de Event Hubs y un centro de eventos](../../event-hubs/event-hubs-create.md). Estos centro de eventos y espacio de nombres son el destino de todos los datos de supervisión. Un espacio de nombres de Event Hubs es una agrupación lógica de centros de eventos que comparten la misma directiva de acceso, al igual que una cuenta de almacenamiento tiene blobs individuales dentro de esa cuenta de almacenamiento. Tenga en cuenta los siguientes detalles sobre el espacio de nombres de los centros de eventos y aquellos centros de eventos que use para transmitir los datos de supervisión:

* El número de unidades de rendimiento permite aumentar la escala de rendimiento para los centros de eventos. Solo se necesita una unidad de procesamiento normalmente. Si necesita escalar verticalmente a medida que el uso del registro aumenta, siempre puede aumentar manualmente el número de unidades de procesamiento del espacio de nombres o habilitar la inflación automática.
* El número de particiones permite paralelizar el consumo entre muchos consumidores. Una sola partición puede admitir hasta 20 MBps o, aproximadamente, 20 000 mensajes por segundo. Dependiendo de la herramienta que consume los datos, puede o no puede admitir el consumo de varias particiones. Es razonable comenzar con cuatro particiones si no está seguro acerca de la cantidad de particiones que debe configurar.
* Igualmente, debe establecer la retención de mensajes en el centro de eventos durante al menos 7 días. Si la herramienta de consumo deja de funcionar durante más de un día, esta opción garantiza que dicha herramienta pueda continuar desde donde se quedó (en cuanto a los eventos de hasta 7 días de antigüedad).
* Debe usar el grupo de consumidores predeterminado en el centro de eventos. No es necesario para crear otros grupos de consumidores o usar un grupo de consumidores independientes a menos que piense disponer de dos herramientas diferentes que consuman los mismos datos del mismo centro de eventos.
* En cuanto al registro de actividad de Azure, se elige un espacio de nombres de Event Hubs, y Azure Monitor crea un centro de eventos dentro de ese espacio de nombres llamado _insights-logs-operational-logs_. Para otros tipos de registro, puede elegir un centro de eventos existente o hacer que Azure Monitor cree un centro de eventos en función de la categoría de registro.
* Los puertos de salida 5671 y 5672 deben estar abiertos en el equipo o red virtual que consume datos del centro de eventos.

## <a name="monitoring-data-available"></a>Datos de supervisión disponibles
En [Orígenes de datos de supervisión de Azure Monitor](data-sources.md) se describen los diferentes niveles de datos de las aplicaciones de Azure y los tipos de datos de supervisión disponibles para cada uno. En la siguiente tabla se enumera cada uno de esos niveles y se proporciona una descripción de cómo esos datos pueden transmitirse a un centro de eventos. Siga los vínculos proporcionados para obtener más detalles.

| Nivel | data | Método |
|:---|:---|:---|
| [Inquilino de Azure](data-sources.md#azure-tenant) | Registros de auditoría de Azure Active Directory | Configure un valor de diagnóstico de inquilino en el inquilino de AAD. Consulte [Tutorial: Streaming de los registros de Azure Active Directory a Azure Event Hubs](../../active-directory/reports-monitoring/tutorial-azure-monitor-stream-logs-to-event-hub.md) para obtener más detalles. |
| [Suscripción de Azure](data-sources.md#azure-subscription) | Azure Activity Log | Cree un perfil de registro para exportar eventos del registro de actividades a Event Hubs.  Consulte [Transmisión de registros de plataforma de Azure a Azure Event Hubs](./resource-logs.md#send-to-azure-event-hubs) para más información. |
| [Recursos de Azure](data-sources.md#azure-resources) | Métricas de la plataforma<br> Registros del recurso |Ambos tipos de datos se envían a un centro de eventos mediante la configuración de diagnóstico de recursos. Consulte [Transmisión de registros de recursos de Azure a un centro de eventos](./resource-logs.md#send-to-azure-event-hubs) para más detalles. |
| [Sistema operativo (invitado)](data-sources.md#operating-system-guest) | Azure Virtual Machines | Instale la [extensión de Azure Diagnostics](diagnostics-extension-overview.md) en máquinas virtuales de Windows y Linux en Azure. Consulte [Streaming de datos de Azure Diagnostics en la ruta de acceso activa mediante Event Hubs](diagnostics-extension-stream-event-hubs.md) para obtener detalles sobre las VM de Windows y [Usar la extensión de diagnósticos de Linux para supervisar métricas y registros](../../virtual-machines/extensions/diagnostics-linux.md#protected-settings) para obtener detalles sobre las VM de Linux. |
| [Código de aplicación](data-sources.md#application-code) | Application Insights | Application Insights no proporciona un método directo para transmitir datos a LOS centros de eventos. Puede [configurar la exportación continua](../app/export-telemetry.md) de los datos de Application Insights a una cuenta de almacenamiento y luego usar una aplicación lógica para enviar esos datos a un centro de eventos, tal como se describe en [Streaming manual con la aplicación lógica](#manual-streaming-with-logic-app). |

## <a name="manual-streaming-with-logic-app"></a>Streaming manual con la aplicación lógica
Para los datos que no puede transmitir directamente a un centro de eventos, puede escribir en Azure Storage y luego usar una aplicación lógica que se desencadene según tiempo y que [extraiga datos de Blob Storage](../../connectors/connectors-create-api-azureblobstorage.md#add-action) y [ los envíe como mensaje al centro de eventos](../../connectors/connectors-create-api-azure-event-hubs.md#add-action). 


## <a name="partner-tools-with-azure-monitor-integration"></a>Herramientas de asociados con la integración de Azure Monitor

El enrutamiento de los datos de supervisión a un centro de eventos con Azure Monitor facilita la integración con las herramientas externas de SIEM y de supervisión. Se incluyen los siguientes ejemplos de herramientas con la integración de Azure Monitor :

| Herramienta | Hospedado en Azure | Descripción |
|:---|:---| :---|
|  IBM QRadar | No | Los protocolos Microsoft Azure DSM y Microsoft Azure Even Hub se pueden descargar en el sitio web de [el sitio web de soporte técnico de IBM](https://www.ibm.com/support). Encontrará más información acerca de la integración con Azure en [Configuración de QRadar DSM](https://www.ibm.com/support/knowledgecenter/SS42VS_DSM/c_dsm_guide_microsoft_azure_overview.html?cp=SS42VS_7.3.0). |
| Splunk | No | [El complemento Azure Monitor para Splunk](https://splunkbase.splunk.com/app/3534/) está disponible en Splunkbase y es un proyecto de código abierto. La documentación está disponible en [Complemento de Azure Monitor para Splunk](https://github.com/Microsoft/AzureMonitorAddonForSplunk/wiki/Azure-Monitor-Addon-For-Splunk).<br><br> Si no puede instalar un complemento en la instancia de Splunk (por ejemplo, si usa un proxy o si se ejecuta en Splunk Cloud), puede reenviar estos eventos al recopilador de eventos de HTTP de Splunk mediante [esta función de Azure para Splunk](https://github.com/Microsoft/AzureFunctionforSplunkVS) que desencadenan los nuevos mensajes del centro de eventos. |
| sumologic | No | Las instrucciones para configurar SumoLogic para consumir datos de un centro de eventos están disponibles en [Recopilar registros para la aplicación Azure Audit desde el centro de eventos](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure-Audit/02Collect-Logs-for-Azure-Audit-from-Event-Hub). |
| ArcSight | No | El conector inteligente ArcSight de Azure Event Hubs está disponible como parte de [esta colección de conectores inteligentes de ArcSight](https://community.softwaregrp.com/t5/Discussions/Announcing-General-Availability-of-ArcSight-Smart-Connectors-7/m-p/1671852). |
| Servidor de Syslog | No | Si quiere transmitir datos de Azure Monitor directamente a un servidor syslog, puede usar una solución [basada en una función de Azure](https://github.com/miguelangelopereira/azuremonitor2syslog/).
| LogRhythm | No| Las instrucciones necesarias para configurar LogRhythm con el fin de recopilar registros de un centro de eventos están disponibles [aquí](https://logrhythm.com/six-tips-for-securing-your-azure-cloud-environment/). 
|Logz.io | Sí | Para más información, consulte [Introducción a la supervisión y el registro con Logz.io para aplicaciones Java que se ejecutan en Azure](/azure/developer/java/fundamentals/java-get-started-with-logzio).

También pueden estar disponibles otros asociados. Para obtener una lista completa de todos los asociados de Azure Monitor y sus funcionalidades, consulte [Integraciones de asociados de Azure Monitor](partners.md).

## <a name="next-steps"></a>Pasos siguientes
* [Archivar el registro de actividad en una cuenta de almacenamiento](./activity-log.md#legacy-collection-methods)
* [Leer la introducción sobre el registro de actividad de Azure](./platform-logs-overview.md)
* [Configurar una alerta basada en un evento del registro de actividad](./alerts-log-webhook.md)

