---
title: ¿Qué es la supervisión de Azure Active Directory? | Microsoft Docs
description: Proporciona información general de los informes de Azure Active Directory.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: e2b3d8ce-708a-46e4-b474-123792f35526
ms.service: active-directory
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: efa4b625afb641209d3920c8663ed810ee27e1ad
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89228654"
---
# <a name="what-is-azure-active-directory-monitoring"></a>¿Qué es la supervisión de Azure Active Directory?

Con la supervisión de Azure Active Directory (Azure AD), ahora puede enrutar los registros de actividad de Azure AD a diferentes puntos de conexión. A continuación, puede conservarlo para su uso a largo plazo o integrarlo con herramientas de Administración de eventos e información de seguridad (SIEM) de terceros para obtener información sobre su entorno.

Actualmente, puede enrutar los registros a:

- Una cuenta de almacenamiento de Azure.
- Un centro de eventos de Azure, para que pueda integrarlo con sus instancias de Splunk y Sumologic.
- Un área de trabajo de Azure Log Analytics, donde puede analizar los datos, crear un panel y alertar sobre eventos específicos.

**Rol necesario**: Administrador global

> [!VIDEO https://www.youtube.com/embed/syT-9KNfug8]

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="diagnostic-settings-configuration"></a>Configuración de opciones de diagnóstico

Para configurar los valores de supervisión de los registros de actividad de Azure AD, primero inicie sesión en [Azure Portal](https://portal.azure.com) y, después, seleccione **Azure Active Directory**. Desde aquí, puede acceder a la página de configuración de los ajustes de diagnóstico de dos maneras:

* Seleccione **Configuración de diagnóstico** en la sección **Supervisión**.

    ![Configuración de diagnóstico](./media/overview-monitoring/diagnostic-settings.png)
    
* Seleccione **Registros de auditoría** o **Inicios de sesión** y, después, seleccione **Exportar configuración**. 

    ![Exportar configuración](./media/overview-monitoring/export-settings.png)


## <a name="route-logs-to-storage-account"></a>Enrutamiento de registros a la cuenta de almacenamiento

Al enrutar los registros a una cuenta de almacenamiento de Azure, puede retenerlos durante más tiempo que el período de retención predeterminado que se describe en las [directivas de retención](reference-reports-data-retention.md). Obtenga información sobre cómo [enrutar datos a la cuenta de almacenamiento](quickstart-azure-monitor-route-logs-to-storage-account.md).

## <a name="stream-logs-to-event-hub"></a>Transmisión en secuencias de registros a un centro de eventos

Enrutar los registros a un centro de eventos de Azure le permite integrarse con herramientas SIEM de terceros como Sumologic y Splunk. Esta integración le permite combinar los datos de registro de actividad de Azure AD con otros datos administrados por el SIEM, a fin de proporcionar una mejor comprensión del entorno. Aprenda cómo [transmitir registros a un centro de eventos](tutorial-azure-monitor-stream-logs-to-event-hub.md).

## <a name="send-logs-to-azure-monitor-logs"></a>Envío de registros a registros de Azure Monitor

[Registros de Azure Monitor](../../azure-monitor/log-query/log-query-overview.md) es una solución que consolida la supervisión de los datos de diferentes orígenes y proporciona un lenguaje de consulta y un motor de análisis que ofrece información sobre el funcionamiento de las aplicaciones y los recursos. Al enviar los registros de actividad de Azure AD a los registros de Azure Monitor, puede recuperar, supervisar y alertar rápidamente sobre los datos recopilados. Aprenda a [enviar datos a los registros de Azure Monitor](howto-integrate-activity-logs-with-log-analytics.md).

También puede instalar las vistas pregeneradas de los registros de actividad de Azure AD para supervisar escenarios comunes que incluyen inicios de sesión y eventos de auditoría. Aprenda a [instalar y utilizar las vistas de Log Analytics para los registros de actividad de Azure AD](howto-install-use-log-analytics-views.md).

## <a name="next-steps"></a>Pasos siguientes

* [Registros de actividad en Azure Monitor](concept-activity-logs-azure-monitor.md)
* [Transmisión en secuencias de registros a un centro de eventos](tutorial-azure-monitor-stream-logs-to-event-hub.md)
* [Envío de registros a registros de Azure Monitor](howto-integrate-activity-logs-with-log-analytics.md)