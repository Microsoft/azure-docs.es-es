---
title: Archivado e informe con Azure Monitor en administración de derechos de Azure AD
description: Obtenga información sobre cómo archivar registros y crear informes con Azure Monitor en la administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/17/2020
ms.author: barclayn
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 89d6379f3fa41036836288ed5c75fbdaad0031da
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88783830"
---
# <a name="archive-logs-and-reporting-on-azure-ad-entitlement-management-in-azure-monitor"></a>Archivado de registros e informes sobre la administración de derechos de Azure AD en Azure Monitor

Azure AD almacena los eventos de auditoría durante un máximo de 30 días en el registro de auditoría. No obstante, puede conservar los datos de auditoría por un tiempo mayor que el período de retención predeterminado que se describe en [¿Durante cuánto tiempo Azure AD almacena los datos de informes?](../reports-monitoring/reference-reports-data-retention.md); para ello, enrútelos a una cuenta de Azure Storage o utilice Azure Monitor. A continuación, puede usar libros y consultas e informes personalizados en estos datos.


## <a name="configure-azure-ad-to-use-azure-monitor"></a>Configuración de Azure AD para usar Azure Monitor
Antes de usar los libros de Azure Monitor, debe configurar Azure AD para enviar una copia de sus registros de auditoría a Azure Monitor.

El archivado de los registros de auditoría de Azure AD requiere tener Azure Monitor en una suscripción de Azure. Puede obtener más información sobre los requisitos previos y los costos estimados de usar Azure Monitor en [Registros de actividad de Azure AD en Azure Monitor](../reports-monitoring/concept-activity-logs-azure-monitor.md).

**Rol necesario**: Administrador global

1. Inicie sesión en Azure Portal como administrador global. Asegúrese de tener acceso al grupo de recursos que contiene el área de trabajo de Azure Monitor.
 
1. Seleccione **Azure Active Directory**, a continuación, haga clic en **Configuración de diagnóstico** en Supervisión en el menú de navegación izquierdo. Compruebe si ya hay una opción para enviar los registros de auditoría a esa área de trabajo.

1. Si no hay una opción de configuración, haga clic en **Agregar configuración de diagnóstico**. Siga las instrucciones del artículo [Integración de registros de Azure AD con registros de Azure Monitor](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md#send-logs-to-azure-monitor) para enviar el registro de auditoría de Azure AD al área de trabajo de Azure Monitor.

    ![Panel Configuración de diagnóstico](./media/entitlement-management-logs-and-reporting/audit-log-diagnostics-settings.png)


1. Después de enviar el registro a Azure Monitor, seleccione **Áreas de trabajo de Log Analytics** y seleccione el área de trabajo que contiene los registros de auditoría de Azure AD.

1. Seleccione **Uso y costos estimados** y haga clic en **Retención de datos**. Cambie el control deslizante al número de días que desea conservar los datos para cumplir los requisitos de auditoría.

    ![Panel Áreas de trabajo de Log Analytics](./media/entitlement-management-logs-and-reporting/log-analytics-workspaces.png)

1. Más adelante, para ver el intervalo de fechas que se mantiene en su área de trabajo, puede usar el libro *Intervalo de fechas de registros archivados*:  
    
    1. Seleccione **Azure Active Directory** y después haga clic en **Libros**. 
    
    1. Expanda la sección **Solución de problemas de Azure Active Directory**y haga clic en **Intervalo de fechas de registros archivados**. 


## <a name="view-events-for-an-access-package"></a>Ver los eventos de un paquete de acceso  

Para ver los eventos de un paquete de acceso, debe tener acceso al área de trabajo subyacente de Azure Monitor (consulte [Administración del acceso a los datos de registro y las áreas de trabajo en Azure Monitor](../../azure-monitor/platform/manage-access.md#manage-access-using-azure-permissions) para obtener información) y en uno de los roles siguientes: 

- Administrador global  
- Administrador de seguridad  
- Lector de seguridad  
- Lector de informes  
- Administrador de aplicaciones  

Use el procedimiento siguiente para ver los eventos: 

1. En Azure Portal, seleccione **Azure Active Directory** y, a continuación, haga clic en **Libros**. Si solo tiene una suscripción, vaya al paso 3. 

1. Si tiene varias suscripciones, seleccione la suscripción que contenga el área de trabajo.  

1. Seleccione el libro llamado *Actividad de acceso a paquetes*. 

1. En ese libro, seleccione un intervalo de tiempo (cambie a **Todo** si no está seguro) y seleccione un identificador de paquete de acceso en la lista desplegable de todos los paquetes de acceso que tenían actividad durante ese intervalo de tiempo. Se mostrarán los eventos relacionados con el paquete de acceso que se produjeron durante el intervalo de tiempo seleccionado.  

    ![Ver eventos de paquete de acceso](./media/entitlement-management-logs-and-reporting/view-events-access-package.png) 

    Cada fila incluye la hora, el identificador de paquete de acceso, el nombre de la operación, el identificador de objeto, el UPN y el nombre para mostrar del usuario que inició la operación.  Se incluyen detalles adicionales en JSON.   


## <a name="create-custom-azure-monitor-queries-using-the-azure-portal"></a>Creación de consultas personalizadas de Azure Monitor mediante Azure Portal
Puede crear sus propias consultas en eventos de auditoría de Azure AD, incluidos los eventos de administración de derechos.  

1. En Azure Active Directory de Azure Portal, haga clic en **Registros** en la sección Supervisión del menú de navegación de la izquierda para crear una nueva página de consulta.

1. El área de trabajo se debe mostrar en la parte superior izquierda de la página de consulta. Si tiene varias áreas de trabajo de Azure Monitor, y no se muestra el área de trabajo que está usando para almacenar eventos de auditoría de Azure AD, haga clic en **Seleccionar ámbito**. A continuación, seleccione la suscripción y el área de trabajo correctas.

1. A continuación, en el área de texto de la consulta, elimine la cadena "search *" y reemplácela por la siguiente consulta:

    ```
    AuditLogs | where Category == "EntitlementManagement"
    ```

1. A continuación, haga clic en **Ejecutar**. 

    ![Haga clic en Ejecutar para iniciar la consulta.](./media/entitlement-management-logs-and-reporting/run-query.png)

De forma predeterminada, la tabla mostrará los eventos del registro de auditoría para la administración de derechos de la última hora. Puede cambiar el valor de "Intervalo de tiempo" para ver los eventos anteriores. Sin embargo, si se cambia esta configuración, solo se mostrarán los eventos que se produjeron después de que se haya configurado Azure AD para enviar eventos a Azure Monitor.

Si desea conocer los eventos de auditoría más antiguos y más recientes contenidos en Azure Monitor, use la siguiente consulta:

```
AuditLogs | where TimeGenerated > ago(3653d) | summarize OldestAuditEvent=min(TimeGenerated), NewestAuditEvent=max(TimeGenerated) by Type
```

Para obtener más información sobre las columnas que se almacenan para los eventos de auditoría en Azure Monitor, consulte [Interpretación del esquema de registros de auditoría de Azure AD en Azure Monitor](../reports-monitoring/reference-azure-monitor-audit-log-schema.md).

## <a name="create-custom-azure-monitor-queries-using-azure-powershell"></a>Creación de consultas personalizadas de Azure Monitor mediante Azure PowerShell

Puede acceder a los registros a través de PowerShell después de haber configurado Azure AD para enviar registros a Azure Monitor. A continuación, envíe consultas desde scripts o desde la línea de comandos de PowerShell, sin necesidad de ser administrador global en el inquilino. 

### <a name="ensure-the-user-or-service-principal-has-the-correct-role-assignment"></a>Asegurarse de que el usuario o la entidad de servicio tengan la asignación de roles correcta

Asegúrese de que tener, como usuario o entidad de servicio que se va a autenticar en Azure AD, el rol de Azure adecuado en el área de trabajo de Log Analytics. Las opciones de rol son Lector de Log Analytics o Colaborador de Log Analytics. Si ya tiene uno de esos roles, vaya directamente a [Recuperación del id. de Log Analytics con una suscripción a Azure](#retrieve-log-analytics-id-with-one-azure-subscription).

Para establecer la asignación de roles y crear una consulta, siga estos pasos:

1. En Azure Portal, seleccione el [área de trabajo de Log Analytics](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.OperationalInsights%2Fworkspaces
).

1. Seleccione **Access Control (IAM)** .

1. A continuación, haga clic en **Agregar** para agregar una asignación de roles.

    ![Adición de una asignación de roles](./media/entitlement-management-logs-and-reporting/workspace-set-role-assignment.png)

### <a name="install-azure-powershell-module"></a>Instalación del módulo de Azure PowerShell

Una vez que tenga la asignación de roles adecuada, inicie PowerShell e [Instale el módulo de Azure PowerShell](/powershell/azure/install-az-ps?view=azps-3.3.0) (si aún no lo ha hecho). Para hacerlo, escriba:

```azurepowershell
install-module -Name az -allowClobber -Scope CurrentUser
```
    
Ahora está listo para autenticarse en Azure AD y recuperar el id. del área de trabajo de Log Analytics que está consultando.

### <a name="retrieve-log-analytics-id-with-one-azure-subscription"></a>Recuperación del id. de Log Analytics con un suscripción a Azure
Si solo tiene una suscripción a Azure y una sola área de trabajo de Log Analytics, escriba lo siguiente para autenticarse en Azure AD, conectarse a esa suscripción y recuperar esa área de trabajo:
 
```azurepowershell
Connect-AzAccount
$wks = Get-AzOperationalInsightsWorkspace
```
 
### <a name="retrieve-log-analytics-id-with-multiple-azure-subscriptions"></a>Recuperación del id. de Log Analytics con varias suscripciones a Azure

 [Get-AzOperationalInsightsWorkspace](/powershell/module/Az.OperationalInsights/Get-AzOperationalInsightsWorkspace) funciona en una suscripción a la vez. Por lo tanto, si tiene varias suscripciones a Azure, querrá asegurarse de conectarse a la que tiene el área de trabajo de Log Analytics con los registros de Azure AD. 
 
 En los siguientes cmdlets se muestra una lista de suscripciones y se busca el id. de la suscripción que tiene el área de trabajo de Log Analytics:
 
```azurepowershell
Connect-AzAccount
$subs = Get-AzSubscription
$subs | ft
```
 
Puede volver a autenticarse y asociar la sesión de PowerShell a esa suscripción mediante un comando, como `Connect-AzAccount –Subscription $subs[0].id`. Para obtener más información sobre cómo autenticarse en Azure desde PowerShell, incluida la forma no interactiva, consulte [Inicio de sesión con Azure PowerShell](/powershell/azure/authenticate-azureps?view=azps-3.3.0&viewFallbackFrom=azps-2.5.0
).

Si tiene varias áreas de trabajo de Log Analytics en esa suscripción, el cmdlet [Get-AzOperationalInsightsWorkspace](/powershell/module/Az.OperationalInsights/Get-AzOperationalInsightsWorkspace) devuelve la lista de áreas de trabajo. A continuación, puede buscar la que tiene los registros de Azure AD. El campo `CustomerId` devuelto por este cmdlet es el mismo que el valor del "Id. de área de trabajo" que se muestra en la información general del área de trabajo de Log Analytics en Azure Portal.
 
```powershell
$wks = Get-AzOperationalInsightsWorkspace
$wks | ft CustomerId, Name
```

### <a name="send-the-query-to-the-log-analytics-workspace"></a>Envío de la consulta al área de trabajo de Log Analytics
Por último, una vez que tenga identificada un área de trabajo, puede usar [Invoke-AzOperationalInsightsQuery](/powershell/module/az.operationalinsights/Invoke-AzOperationalInsightsQuery?view=azps-3.3.0
) para enviar una consulta de Kusto a esa área de trabajo. Estas consultas se escriben en [lenguaje de consulta Kusto](/azure/kusto/query/).
 
Por ejemplo, puede recuperar el intervalo de fechas de los registros de eventos de auditoría en el área de trabajo de Log Analytics, con cmdlets de PowerShell para enviar una consulta como la siguiente:
 
```powershell
$aQuery = "AuditLogs | where TimeGenerated > ago(3653d) | summarize OldestAuditEvent=min(TimeGenerated), NewestAuditEvent=max(TimeGenerated) by Type"
$aResponse = Invoke-AzOperationalInsightsQuery -WorkspaceId $wks[0].CustomerId -Query $aQuery
$aResponse.Results |ft
```

También puede recuperar los eventos de administración de derechos mediante una consulta como la siguiente:

```azurepowershell
$bQuery = 'AuditLogs | where Category == "EntitlementManagement"'
$bResponse = Invoke-AzOperationalInsightsQuery -WorkspaceId $wks[0].CustomerId -Query $Query
$bResponse.Results |ft 
```

## <a name="next-steps"></a>Pasos siguientes:
- [Crear informes interactivos con libros de Azure Monitor](../../azure-monitor/platform/workbooks-overview.md)