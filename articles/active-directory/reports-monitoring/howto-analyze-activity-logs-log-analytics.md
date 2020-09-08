---
title: Análisis de registros de actividad mediante registros de Azure Monitor | Microsoft Docs
description: Aprenda a analizar los registros de actividad de Azure Active Directory mediante registros de Azure Monitor.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 4535ae65-8591-41ba-9a7d-b7f00c574426
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4ab60cb83e68a0c325ec1e2ac102870493016b57
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89231238"
---
# <a name="analyze-azure-ad-activity-logs-with-azure-monitor-logs"></a>Análisis de registros de actividad de Azure AD con registros de Azure Monitor

Después de [integrar los registros de actividad de Azure AD con los registros de Azure Monitor](howto-integrate-activity-logs-with-log-analytics.md), puede usar la eficacia de estos últimos para obtener información sobre el entorno. También puede instalar las [vistas de Log Analytics para los registros de actividad de Azure AD](howto-install-use-log-analytics-views.md) para acceder a informes pregenerados sobre eventos de auditoría y de inicio de sesión en el entorno.

En este artículo, aprenderá a analizar registros de actividad de Azure AD en el área de trabajo de Log Analytics. 

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Prerrequisitos 

Para continuar, necesita:

* Un área de trabajo de Log Analytics en la suscripción a Azure. Aprenda a [crear un área de trabajo de Log Analytics](../../azure-monitor/learn/quick-create-workspace.md).
* Primero, complete los pasos para [enrutar los registros de actividad de Azure AD al área de trabajo de Log Analytics](howto-integrate-activity-logs-with-log-analytics.md).
*  [Acceso](../../azure-monitor/platform/manage-access.md#manage-access-using-workspace-permissions) al área de trabajo de Log Analytics
* Los siguientes roles en Azure Active Directory (si obtiene acceso a Log Analytics a través del portal de Azure Active Directory)
    - Administrador de seguridad
    - Lector de seguridad
    - Lector de informes
    - Administrador global
    
## <a name="navigate-to-the-log-analytics-workspace"></a>Acceso al área de trabajo de Log Analytics

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 

2. Seleccione **Azure Active Directory** y, después, **Registros** en la sección **Supervisión** para abrir el área de trabajo de Log Analytics. Se abre el área de trabajo con una consulta predeterminada.

    ![Consulta predeterminada](./media/howto-analyze-activity-logs-log-analytics/defaultquery.png)


## <a name="view-the-schema-for-azure-ad-activity-logs"></a>Visualización del esquema para los registros de actividad de Azure AD

Los registros se insertan en las tablas **AuditLogs** y **SigninLogs** del área de trabajo. Para ver el esquema de estas tablas:

1. En la vista de consulta predeterminada de la sección anterior, seleccione **Esquema** y expanda el área de trabajo. 

2. Expanda la sección **Administración de registros** y, a continuación, expanda **AuditLogs** o **SignInLogs** para ver el esquema de registro.
    ![Registros de auditoría](./media/howto-analyze-activity-logs-log-analytics/auditlogschema.png) ![Registros de inicio de sesión](./media/howto-analyze-activity-logs-log-analytics/signinlogschema.png)

## <a name="query-the-azure-ad-activity-logs"></a>Consulta de registros de actividad de Azure AD

Ahora que tiene los registros en el área de trabajo, puede ejecutar consultas en ellas. Por ejemplo, para obtener las mejores aplicaciones usadas en la última semana, reemplace la consulta predeterminada por la siguiente y seleccione **Ejecutar**.

```
SigninLogs 
| where CreatedDateTime >= ago(7d)
| summarize signInCount = count() by AppDisplayName 
| sort by signInCount desc 
```

Para obtener unos mejores eventos de auditoría de la última semana, utilice la siguiente consulta:

```
AuditLogs 
| where TimeGenerated >= ago(7d)
| summarize auditCount = count() by OperationName 
| sort by auditCount desc 
```
## <a name="alert-on-azure-ad-activity-log-data"></a>Alerta sobre datos del registro de actividad de Azure AD

También puede configurar alertas en la consulta. Por ejemplo, para configurar una alerta cuando se han utilizado más de 10 aplicaciones en la última semana:

1. En el área de trabajo, seleccione **Establecer alerta** para abrir la página **Crear regla**.

    ![Establecer alerta](./media/howto-analyze-activity-logs-log-analytics/setalert.png)

2. Seleccione los **criterios de alerta** predeterminados creados en la alerta y actualice el **umbral** en la métrica predeterminada a 10.

    ![Criterios de alerta](./media/howto-analyze-activity-logs-log-analytics/alertcriteria.png)

3. Escriba un nombre y descripción para la alerta y elija el nivel de gravedad. En nuestro ejemplo, podríamos establecerlo en **Información**.

4. Seleccione el **grupo de acciones** al que se avisará cuando se produzca la señal. Puede elegir notificar al equipo por correo electrónico o mensaje de texto, o puede automatizar la acción mediante webhooks, funciones de Azure o aplicaciones lógicas. Obtenga más información sobre cómo [crear y administrar grupos de alerta en Azure Portal](../../azure-monitor/platform/action-groups.md).

5. Cuando haya configurado la alerta, seleccione **Crear alerta** para habilitarla. 

## <a name="install-and-use-pre-built-views-for-azure-ad-activity-logs"></a>Instalación y uso de vistas pregeneradas para los registros de actividad de Azure AD

También puede descargar las vistas pregeneradas de Log Analytics para los registros de actividad de Azure AD. Las vistas proporcionan varios informes relacionados con escenarios comunes que implican eventos de auditoría y de inicio de sesión. También se puede alertar sobre cualquiera de los datos proporcionados en los informes, siguiendo los pasos descritos en la sección anterior.

* **Azure AD Account Provisioning Events** (Eventos de aprovisionamiento de la cuenta de Azure AD): Esta vista muestra informes relacionados con la auditoría de la actividad de aprovisionamiento, como el número de nuevos usuarios aprovisionados y errores de aprovisionamiento, el número de usuarios actualizados y errores de actualización y el número de usuarios desaprovisionados y los errores correspondientes.    
* **Sign-ins Events** (Eventos de inicio de sesión): Esta vista muestra los informes más importantes relacionados con la supervisión de la actividad de inicio de sesión, como los inicios de sesión por aplicación, usuario y dispositivo, así como una vista resumida que muestra el número de inicios de sesión a lo largo del tiempo.
* **Users Performing Consent** (Usuarios que dan su consentimiento): Esta vista muestra los informes relacionados con el consentimiento del usuario, como las concesiones de consentimiento por parte del usuario, los inicios de sesión de los usuarios que han concedido el consentimiento, así como los inicios de sesión por aplicación para todas las aplicaciones basadas en el consentimiento. 

Aprenda a [instalar y utilizar las vistas de Log Analytics para los registros de actividad de Azure AD](howto-install-use-log-analytics-views.md). 


## <a name="next-steps"></a>Pasos siguientes

* [Introducción a las consultas de registros de Azure Monitor](../../azure-monitor/log-query/get-started-queries.md)
* [Creación y administración de grupos de alertas en Azure Portal](../../azure-monitor/platform/action-groups.md)
* [Instalación y uso de las vistas de Log Analytics para Azure Active Directory](howto-install-use-log-analytics-views.md)