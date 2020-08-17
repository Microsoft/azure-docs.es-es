---
title: 'Problemas de diagnóstico de Windows Virtual Desktop (clásico): Azure'
description: Procedimientos para usar la característica de diagnóstico de Windows Virtual Desktop (clásico) para diagnosticar problemas.
author: Heidilohr
ms.topic: conceptual
ms.date: 05/13/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 7e652f04b42b132e7c1307503b1764dda7b2036b
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009348"
---
# <a name="identify-and-diagnose-issues-in-windows-virtual-desktop-classic"></a>Identificación y diagnóstico de problemas de Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager. Si está tratando de administrar objetos de Windows Virtual Desktop para Azure Resource Manager, consulte [este artículo](../diagnostics-role-service.md).

Windows Virtual Desktop ofrece una característica de diagnóstico que permite al administrador detectar problemas a través de una única interfaz. Los roles de Windows Virtual Desktop registran una actividad de diagnóstico cada vez que un usuario interactúa con el sistema. Cada registro contiene información relevante, como los roles de Windows Virtual Desktop que han participado en la transacción, los mensajes de error, la información del inquilino y la información del usuario. Las actividades de diagnóstico se crean mediante acciones tanto del administrador como del usuario final, y se pueden clasificar en tres categorías principales:

* Actividades de suscripción a fuente: el usuario final desencadena estas actividades cada vez que intenta conectarse a su fuente mediante aplicaciones de Microsoft Remote Desktop.
* Actividades de conexión: el usuario final desencadena estas actividades cada vez que intenta conectarse a un escritorio o a RemoteApp mediante aplicaciones de Microsoft Remote Desktop.
* Actividades de administración: el administrador desencadena estas actividades cada vez que realiza operaciones de administración en el sistema, como crear grupos de hosts, asignar usuarios a grupos de aplicaciones y crear asignaciones de roles.

Las conexiones que no lleguen a Windows Virtual Desktop no aparecerán en los resultados de diagnóstico, ya que el propio servicio de rol de diagnóstico forma parte de Windows Virtual Desktop. Se pueden producir problemas de conexión a Windows Virtual Desktop si el usuario final está experimentando problemas de conectividad de red.

Para empezar, [descargue e importe el módulo de PowerShell para Windows Virtual Desktop](/powershell/windows-virtual-desktop/overview/) que se usará en la sesión de PowerShell, si aún no lo ha hecho. Después, ejecute el siguiente cmdlet para iniciar sesión en su cuenta:

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

## <a name="diagnose-issues-with-powershell"></a>Diagnóstico de problemas con PowerShell

Los diagnósticos de Windows Virtual Desktop usan un solo cmdlet de PowerShell, pero contienen muchos parámetros opcionales que sirven para restringir y aislar problemas. En las siguientes secciones se enumeran los cmdlets que se pueden ejecutar para diagnosticar problemas. La mayoría de los filtros se pueden aplicar de manera conjunta. Los valores que se muestran entre corchetes angulares, como `<tenantName>`, se deben reemplazar por los valores oportunos según su situación.

>[!IMPORTANT]
>La característica de diagnóstico es para la solución de problemas de un solo usuario. Todas las consultas que usan PowerShell deben incluir los parámetros *-UserName* o *-ActivityID*. Para las funcionalidades de supervisión, use Log Analytics. Consulte [Uso de Log Analytics para la característica de diagnóstico](diagnostics-log-analytics-2019.md) para obtener más información sobre cómo enviar datos de diagnóstico al área de trabajo.

### <a name="filter-diagnostic-activities-by-user"></a>Filtrado de actividades de diagnóstico por usuario

El parámetro **-UserName** devuelve una lista de las actividades de diagnóstico que el usuario especificado ha iniciado, como se muestra en el siguiente cmdlet de ejemplo.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -UserName <UserUPN>
```

El parámetro **-UserName** se puede combinar también con otros parámetros de filtrado opcionales.

### <a name="filter-diagnostic-activities-by-time"></a>Filtrado de actividades de diagnóstico por hora

La lista de actividades de diagnóstico devuelta se puede filtrar con los parámetros **-StartTime** y **-EndTime**. El parámetro **-StartTime** devolverá una lista de actividades de diagnóstico a partir de una fecha concreta, como se muestra en el siguiente ejemplo.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -UserName <UserUPN> -StartTime "08/01/2018"
```

Por su parte, el parámetro **-EndTime** se puede agregar a un cmdlet donde se haya especificado el parámetro **-StartTime** para indicar un período de tiempo específico del que quiere recibir resultados. El siguiente cmdlet de ejemplo devolverá una lista de las actividades de diagnóstico que han tenido lugar entre el 1 de agosto y el 10 de agosto.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -UserName <UserUPN> -StartTime "08/01/2018" -EndTime "08/10/2018"
```

Los parámetros **-StartTime** y **-EndTime** se pueden combinar también con otros parámetros de filtrado opcionales.

### <a name="filter-diagnostic-activities-by-activity-type"></a>Filtrado de actividades de diagnóstico por tipo de actividad

Las actividades de diagnóstico también se pueden filtrar por tipo de actividad con el parámetro **-ActivityType**. El siguiente cmdlet devolverá una lista de conexiones del usuario final:

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -UserName <UserUPN> -ActivityType Connection
```

El siguiente cmdlet devolverá una lista de tareas de administración del administrador:

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -ActivityType Management
```

Actualmente, el cmdlet **Get-RdsDiagnosticActivities** no admite que se especifique "Feed" dentro de la sección ActivityType.

### <a name="filter-diagnostic-activities-by-outcome"></a>Filtrado de actividades de diagnóstico por resultado

La lista de actividades de diagnóstico devuelta se puede filtrar por resultado con el parámetro **-Outcome**. El siguiente cmdlet de ejemplo devolverá una lista de las actividades de diagnóstico realizadas correctamente.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -UserName <UserUPN> -Outcome Success
```

El siguiente cmdlet de ejemplo devolverá una lista de las actividades de diagnóstico con errores.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -Outcome Failure
```

El parámetro **-Outcome** se puede combinar también con otros parámetros de filtrado opcionales.

### <a name="retrieve-a-specific-diagnostic-activity-by-activity-id"></a>Recuperación de una actividad de diagnóstico específica por su identificador de actividad

El parámetro **-ActivityId** devuelve una actividad de diagnóstico concreta (si existe), como se muestra en el siguiente cmdlet de ejemplo.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -ActivityId <ActivityIdGuid>
```

### <a name="view-error-messages-for-a-failed-activity-by-activity-id"></a>Visualización de los mensajes de error de una actividad errónea por identificador de actividad

Para ver los mensajes de error de una actividad errónea, debe ejecutar el cmdlet con el parámetro **-Detailed**. Puede ver la lista de errores ejecutando el cmdlet **Select-Object**.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantname> -ActivityId <ActivityGuid> -Detailed | Select-Object -ExpandProperty Errors
```

### <a name="retrieve-detailed-diagnostic-activities"></a>Recuperación de actividades de diagnóstico detalladas

El parámetro **-Detailed** proporciona más detalles sobre cada actividad de diagnóstico que se devuelve. El formato de cada actividad varía según el tipo de actividad. El parámetro **-Detailed** se puede incluir en cualquier consulta **Get-RdsDiagnosticActivities**, como se ilustra en el siguiente ejemplo.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -ActivityId <ActivityGuid> -Detailed
```

## <a name="common-error-scenarios"></a>Escenarios de error habituales

Los escenarios de error se clasifican como internos en el servicio y como externos en Windows Virtual Desktop.

* Problema interno: especifica escenarios que el administrador de inquilinos no puede mitigar y que deben resolverse como un problema de soporte técnico. Cuando aporte sus comentarios a través de la [comunidad de técnicos de Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop), incluya el identificador de la actividad y el período de tiempo aproximado en que se produjo el problema.
* Problema externo: hace referencia a escenarios que el administrador del sistema puede mitigar. Son externos en Windows Virtual Desktop.

En la siguiente tabla se enumeran los errores comunes que los administradores pueden encontrarse.

>[!NOTE]
>Esta lista incluye los errores más comunes y se actualiza de manera periódica. Para estar seguro de que dispone de la información más actualizada, procure consultar en este artículo al menos una vez al mes.

### <a name="external-management-error-codes"></a>Códigos de errores de administración externos

|Código numérico|Código de error|Solución propuesta|
|---|---|---|
|1322|ConnectionFailedNoMappingOfSIDinAD|El usuario no es miembro de Azure Active Directory. Siga las instrucciones del [Centro de administración de Active Directory](/windows-server/identity/ad-ds/get-started/adac/active-directory-administrative-center) para agregarlas.|
|3|UnauthorizedAccess|El usuario que intentó ejecutar el cmdlet de PowerShell administrativo no tiene permisos para hacerlo o ha escrito incorrectamente el nombre de usuario.|
|1000|TenantNotFound|El nombre de inquilino especificado no coincide con ningún inquilino existente. Revise el nombre de inquilino para ver si hay errores tipográficos e inténtelo de nuevo.|
|1006|TenantCannotBeRemovedHasSessionHostPools|Un inquilino no se puede eliminar si contiene objetos. Elimine primero los grupos de hosts de la sesión e inténtelo de nuevo.|
|2000|HostPoolNotFound|El nombre de grupo de hosts especificado no coincide con ningún grupo de hosts existente. Revise el nombre de grupo de hosts para ver si hay errores tipográficos e inténtelo de nuevo.|
|2005|HostPoolCannotBeRemovedHasApplicationGroups|Un grupo de hosts no se puede eliminar si contiene objetos. Quite primero todos los grupos de aplicaciones del grupo de hosts.|
|2004|HostPoolCannotBeRemovedHasSessionHosts|Quite todos los hosts de sesión antes de eliminar el grupo de hosts de la sesión.|
|5001|SessionHostNotFound|El host de sesión de la consulta puede estar sin conexión. Compruebe el estado del grupo de hosts.|
|5008|SessionHostUserSessionsExist |Antes de ejecutar la actividad de administración que pretende, debe cerrar la sesión de todos los usuarios en el host de sesión.|
|6000|AppGroupNotFound|El grupo de aplicaciones especificado no coincide con ningún grupo de aplicaciones. Revise el nombre del grupo de aplicaciones para ver si hay errores tipográficos e inténtelo de nuevo.|
|6022|RemoteAppNotFound|El nombre de la instancia de RemoteApp especificado no coincide con ninguna instancia de RemoteApp existente. Revise el nombre de la instancia de RemoteApp para ver si hay errores tipográficos e inténtelo de nuevo.|
|6010|PublishedItemsExist|El nombre del recurso que está intentando publicar es igual que el de un recurso que ya existe. Cambie el nombre del recurso e inténtelo de nuevo.|
|7002|NameNotValidWhiteSpace|No se pueden usar espacios en blanco en el nombre.|
|8000|InvalidAuthorizationRoleScope|El nombre de rol especificado no coincide con ningún nombre de rol existente. Revise el nombre de rol para ver si hay errores tipográficos e inténtelo de nuevo. |
|8001|UserNotFound |El nombre de usuario especificado no coincide con ningún nombre de usuario existente. Revise el nombre para ver si hay errores tipográficos e inténtelo de nuevo.|
|8005|UserNotFoundInAAD |El nombre de usuario especificado no coincide con ningún nombre de usuario existente. Revise el nombre para ver si hay errores tipográficos e inténtelo de nuevo.|
|8008|TenantConsentRequired|Siga las instrucciones que se indican [aquí](tenant-setup-azure-active-directory.md#grant-permissions-to-windows-virtual-desktop) para proporcionar consentimiento para el inquilino.|

### <a name="external-connection-error-codes"></a>Códigos de error de conexión externa

|Código numérico|Código de error|Solución propuesta|
|---|---|---|
|-2147467259|ConnectionFailedAdErrorNoSuchMember|El usuario no es miembro de Active Directory. Siga las instrucciones del [Centro de administración de Active Directory](/windows-server/identity/ad-ds/get-started/adac/active-directory-administrative-center) para agregarlas.|
|-2147467259|ConnectionFailedAdTrustedRelationshipFailure|El host de sesión no está unido correctamente a Active Directory.|
|-2146233088|ConnectionFailedUserHasValidSessionButRdshIsUnhealthy|Las conexiones no se pudieron establecer porque el host de sesión no está disponible. Compruebe el estado del host de sesión.|
|-2146233088|ConnectionFailedClientDisconnect|Si ve este error con frecuencia, asegúrese de que el equipo del usuario está conectado a la red.|
|-2146233088|ConnectionFailedNoHealthyRdshAvailable|La sesión a la que el usuario de host ha intentado conectarse no está en un estado adecuado. Depure la máquina virtual.|
|-2146233088|ConnectionFailedUserNotAuthorized|El usuario no tiene permiso para acceder al escritorio o la aplicación publicada. Este error puede aparecer después de que el administrador quite recursos publicados. Pida al usuario que actualice la fuente en la aplicación de Escritorio remoto.|
|2|FileNotFound|La aplicación a la que el usuario intentó tener acceso incorrectamente está instalada o establecida en una ruta de acceso incorrecta.|
|3|InvalidCredentials|El nombre de usuario o contraseña especificados por el usuario no coincide con el nombre de usuario o contraseña existentes. Revise las credenciales para ver si hay errores tipográficos e inténtelo de nuevo.|
|8|ConnectionBroken|La conexión entre el cliente y la puerta de enlace o servidor se ha interrumpido. No es necesario hacer nada a menos que esto ocurra de forma inesperada.|
|14|UnexpectedNetworkDisconnect|La conexión a la red se ha interrumpido. Pida al usuario que vuelva a conectarse.|
|24|ReverseConnectFailed|La máquina virtual host no tiene línea de visión directa a la puerta de enlace de Escritorio remoto. Asegúrese de que la dirección IP de la puerta de enlace se puede resolver.|
|1322|ConnectionFailedNoMappingOfSIDinAD|El usuario no es miembro de Active Directory. Siga las instrucciones del [Centro de administración de Active Directory](/windows-server/identity/ad-ds/get-started/adac/active-directory-administrative-center) para agregarlas.|

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los roles de Windows Virtual Desktop, consulte [Entorno de Windows Virtual Desktop](environment-setup-2019.md).

Para ver una lista de cmdlets de PowerShell disponibles para Windows Virtual Desktop, vea la [referencia de PowerShell](/powershell/windows-virtual-desktop/overview).
