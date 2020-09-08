---
title: Obtención de eventos de recursos en Azure App Service
description: Aprenda a obtener eventos de recursos mediante registros de actividad y Event Grid en la aplicación de App Service.
ms.topic: article
ms.date: 04/24/2020
ms.author: msangapu
ms.openlocfilehash: c5c84891187c540c0b24162cf5c8c7f96e9e731a
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962305"
---
# <a name="get-resource-events-in-azure-app-service"></a>Obtención de eventos de recursos en Azure App Service

Azure App Service proporciona herramientas integradas para supervisar el estado y el mantenimiento de los recursos. Los eventos de recursos le ayudan a entender los cambios que se realizaron en los recursos subyacentes de la aplicación web y a tomar las medidas necesarias. Algunos ejemplos de eventos son el escalado de instancias, las actualizaciones de la configuración de la aplicación, el reinicio de la aplicación web y muchas más. En este artículo, aprenderá a ver los [registros de actividad de Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log) y a habilitar [Event Grid](../event-grid/index.yml) para supervisar los eventos de recursos relacionados con la aplicación web de App Service.

> [!NOTE]
> La integración de App Service con Event Grid se encuentra en **versión preliminar**. [Para más información, consulte el anuncio.](https://aka.ms/app-service-event-grid-announcement)
>

## <a name="view-azure-activity-logs"></a>Visualización de los registros de actividad de Azure
Los registros de actividad de Azure contienen eventos de recursos emitidos por las operaciones realizadas en los recursos de la suscripción. Tanto las acciones del usuario en Azure Portal como en las plantillas de Azure Resource Manager contribuyen a los eventos capturados por el registro de actividad. 

Los registros de actividad de Azure para App Service contienen detalles como:
- qué operaciones se efectuaron en los recursos (por ejemplo, planes de App Service)
- quién inició la operación
- cuándo tuvo lugar la operación
- el estado de la operación
- valores de propiedad para ayudarle a investigar la operación

### <a name="what-can-you-do-with-azure-activity-logs"></a>¿Qué puede hacer con los registros de actividad de Azure?

Los registros de actividad de Azure se pueden consultar mediante Azure Portal, PowerShell, la API REST o la CLI. Puede enviar los registros a una cuenta de almacenamiento, a un centro de eventos y a Log Analytics. También puede analizarlos en Power BI o crear alertas para que permanezcan actualizados en los eventos de recursos.

[Visualización y recuperación de eventos del registro de actividad de Azure.](../azure-monitor/platform/activity-log.md#view-the-activity-log)

## <a name="ship-activity-logs-to-event-grid"></a>Envío de registros de actividad a Event Grid

Aunque los registros de actividad están basados en el usuario, hay una nueva integración de [Event Grid](../event-grid/index.yml) con App Service (versión preliminar) que registra las acciones del usuario y los eventos automatizados. Con Event Grid, puede configurar un controlador para que reaccione a los mencionados eventos. Por ejemplo, use Event Grid para desencadenar al instante una función sin servidor que ejecute un análisis de la imagen cada vez que se agrega una fotografía nueva a un contenedor de Blob Storage.

También puede usar Event Grid con Logic Apps para procesar datos en cualquier parte, sin necesidad de escribir código. Event Grid conecta orígenes de datos y controladores de eventos. Por ejemplo, use Event Grid para desencadenar al instante una función sin servidor que ejecute un análisis de la imagen cada vez que se agrega una fotografía nueva a un contenedor de Blob Storage.

[Visualización de las propiedades y el esquema de los eventos de Azure App Service.](../event-grid/event-schema-app-service.md)

## <a name="next-steps"></a><a name="nextsteps"></a> Pasos siguientes
* [Consulta de registros con Azure Monitor](../azure-monitor/log-query/log-query-overview.md)
* [Control de Supervisión de aplicaciones en Azure App Service](web-sites-monitor.md)
* [Troubleshooting Azure App Service in Visual Studio](troubleshoot-dotnet-visual-studio.md) (Solución de problemas de Azure App Service en Visual Studio)
* [Analyze app Logs in HDInsight](https://gallery.technet.microsoft.com/scriptcenter/Analyses-Windows-Azure-web-0b27d413) (Análisis de registros de aplicación en HDInsight)