---
title: Escritorio Virtual Windows inquilinos y host de creación del grupo - Azure
description: Cómo solucionar problemas y resolver el grupo host y el inquilino problemas durante la instalación de un entorno de inquilinos de Escritorio Virtual de Windows.
services: virtual-desktop
author: ChJenk
ms.service: virtual-desktop
ms.topic: troubleshoot
ms.date: 04/08/2019
ms.author: v-chjenk
ms.openlocfilehash: 88e843c410a750387ecf58497dec79586e2a59d8
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523327"
---
# <a name="tenant-and-host-pool-creation"></a>Creación de los grupos de inquilinos y de host

En este artículo se trata problemas durante la instalación inicial de los inquilinos de Escritorio Virtual de Windows y la infraestructura del grupo de host de sesión relacionado.

## <a name="provide-feedback"></a>Proporcionar comentarios

En este momento no se aceptan casos de soporte técnico mientras Windows Virtual Desktop se encuentre en versión preliminar. Visite la [Comunidad técnica de Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop) para hablar sobre Windows Virtual Desktop con el equipo de producto y los miembros activos de la comunidad.

## <a name="acquiring-the-windows-10-enterprise-multi-session-image"></a>Adquisición de la imagen de varias sesiones de Windows 10 Enterprise

Para usar la imagen de varias sesiones de Windows 10 Enterprise, vaya a Azure Marketplace, seleccione **comenzar** > **Microsoft Windows 10** > y [Windows 10 Enterprise para Vista previa de escritorios virtuales, versión 1809](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsdesktop.windows-10?tab=PlansAndPrice).

![Captura de pantalla de selección de Windows 10 Enterprise para vista previa escritorios virtuales, versión 1809.](media/AzureMarketPlace.png)

## <a name="creating-windows-virtual-desktop-tenant"></a>Creación de inquilino de Escritorio Virtual de Windows

Esta sección trata los posibles problemas al crear al inquilino de Escritorio Virtual de Windows.

### <a name="error-the-user-isnt-authorized-to-query-the-management-service"></a>Error: El usuario no está autorizado para consultar el servicio de administración

![Ventana de captura de pantalla de PowerShell en el que un usuario no está autorizado para consultar el servicio de administración.](media/UserNotAuthorizedNewTenant.png)

Ejemplo de error bruto:

```Error
   New-RdsTenant : User isn't authorized to query the management service.
   ActivityId: ad604c3a-85c6-4b41-9b81-5138162e5559
   Powershell commands to diagnose the failure:
   Get-RdsDiagnosticActivities -ActivityId ad604c3a-85c6-4b41-9b81-5138162e5559
   At line:1 char:1
   + New-RdsTenant -Name "testDesktopTenant" -AadTenantId "01234567-89ab-c ...
   + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
       + CategoryInfo          : FromStdErr: (Microsoft.RDInf...nt.NewRdsTenant:NewRdsTenant) [New-RdsTenant], RdsPowerSh
      ellException
       + FullyQualifiedErrorId : UnauthorizedAccess,Microsoft.RDInfra.RDPowershell.Tenant.NewRdsTenant
```

**Causa:** El usuario que ha iniciado sesión no se ha asignado el rol TenantCreator en su Azure Active Directory.

**Fix:** Siga las instrucciones de [asignar el rol de aplicación TenantCreator a un usuario en el inquilino de Azure Active Directory](https://docs.microsoft.com/azure/virtual-desktop/tenant-setup-azure-active-directory#assign-the-tenantcreator-application-role-to-a-user-in-your-azure-active-directory-tenant). Después de seguir las instrucciones, tendrá un usuario asignado a la función TenantCreator.

![Captura de pantalla de TenantCreator rol asignado.](media/TenantCreatorRoleAssigned.png)

## <a name="creating-windows-virtual-desktop-session-host-vms"></a>Creación de máquinas virtuales de host de sesión de Escritorio Virtual de Windows

Host de sesión de las máquinas virtuales se puede crear de varias maneras, pero los equipos de Escritorio Virtual de Windows/servicios de escritorio remoto admiten sólo problemas relacionados con la plantilla de Azure Resource Manager de aprovisionamiento de máquinas virtuales. La plantilla de Azure Resource Manager está disponible en [Azure Marketplace](https://azuremarketplace.microsoft.com/) y [GitHub](https://github.com/).

## <a name="issues-using-windows-virtual-desktop--provision-a-host-pool-azure-marketplace-offering"></a>Problemas con el escritorio Virtual de Windows: aprovisionar una oferta de Azure Marketplace del grupo host

Aplicaciones de Escritorio Virtual Windows: aprovisionar una plantilla de grupo host está disponible en Azure Marketplace.

### <a name="error-when-using-the-link-from-github-the-message-create-a-free-account-appears"></a>Error: Aparece cuando se usa el vínculo de GitHub, el mensaje "crear una cuenta gratuita"

![Captura de pantalla para crear una cuenta gratuita.](media/be615904ace9832754f0669de28abd94.png)

**Causa 1:** No existen suscripciones activas en la cuenta utilizada para iniciar sesión en Azure o la cuenta utilizada no tiene permisos para ver las suscripciones.

**Corrección 1:** Inicie sesión con una cuenta que tenga acceso de colaborador (como mínimo) a la suscripción donde las máquinas virtuales del host de sesión se van a implementar.

**Causa 2:** La suscripción que se va a usar es parte de un inquilino del proveedor de servicios de Microsoft en la nube (CSP).

**Corrección 2:** Vaya a la ubicación de GitHub para **creación y aprovisionamiento de grupo de host de Escritorio Virtual de Windows nueva** y siga estas instrucciones:

1. Haga doble clic en **implementar en Azure** y seleccione **Copiar dirección de vínculo**.
2. Abra **el Bloc de notas** y pegue el vínculo.
3. Antes del carácter #, inserte el nombre del inquilino de cliente CSP final.
4. Abra el nuevo vínculo en un explorador y el portal de Azure cargará la plantilla.

    ```Example
    Example: https://portal.azure.com/<CSP end customer tenant name>
    #create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%
    2FRDS-Templates%2Fmaster%2Fwvd-templates%2FCreate%20and%20provision%20WVD%20host%20pool%2FmainTemplate.json
    ```

## <a name="azure-resource-manager-template-and-powershell-desired-state-configuration-dsc-errors"></a>Plantilla de Azure Resource Manager y los errores de PowerShell Desired State Configuration (DSC)

Siga estas instrucciones para solucionar problemas de las implementaciones incorrectas de DSC de PowerShell y plantillas de Azure Resource Manager.

1. Revise los errores en la implementación mediante [ver operaciones de implementación con Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-deployment-operations).
2. Si no hay ningún error en la implementación, revise los errores en el registro de actividad usando [ver registros de actividad para auditar las acciones sobre los recursos](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-audit).
3. Una vez que se identifica el error, utilice el mensaje de error y los recursos de [solucionar errores comunes de implementación de Azure con Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-common-deployment-errors) para solucionar el problema.
4. Elimine los recursos creados durante la implementación anterior y reintentar la implementación de la plantilla de nuevo.

### <a name="error-your-deployment-failedhostnamejoindomain"></a>Error: La implementación no se pudo...<hostname>/joindomain

![La captura de pantalla de error de implementación.](media/e72df4d5c05d390620e07f0d7328d50f.png)

Ejemplo de error bruto:

```Error
 {"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. 
 Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"Conflict","message":"{\r\n \"status\": \"Failed\",\r\n \"error\":
 {\r\n \"code\": \"ResourceDeploymentFailure\",\r\n \"message\": \"The resource operation completed with terminal provisioning state 'Failed'.
 \",\r\n \"details\": [\r\n {\r\n \"code\": \"VMExtensionProvisioningError\",\r\n \"message\": \"VM has reported a failure when processing
 extension 'joindomain'. Error message: \\\"Exception(s) occurred while joining Domain 'diamondsg.onmicrosoft.com'\\\".\"\r\n }\r\n ]\r\n }\r\n}"}]}
```

**Causa 1:** Las credenciales proporcionadas para unir las máquinas virtuales al dominio son incorrectas.

**Corrección 1:** Vea el error "Credenciales incorrectas" para las máquinas virtuales no están unidas al dominio en [configuración de máquina virtual del host de sesión](troubleshoot-vm-configuration.md).

**Causa 2:** No se resuelve el nombre de dominio.

**Corrección 2:** Aparece el error "no se resuelve el nombre de dominio" para las máquinas virtuales no están unidas al dominio en [configuración de máquina virtual del host de sesión](troubleshoot-vm-configuration.md).

### <a name="error-vmextensionprovisioningerror"></a>Error: VMExtensionProvisioningError

![Captura de pantalla de la implementación con estado de aprovisionamiento no se pudo.](media/7aaf15615309c18a984673be73ac969a.png)

**Causa 1:** Error transitorio con el entorno de Escritorio Virtual de Windows.

**Causa 2:** Error transitorio con conexión.

**Fix:** Confirme que el entorno de Escritorio Virtual de Windows está en buen estado Iniciando sesión con PowerShell. Finalizar el registro de la máquina virtual manualmente en [creación de un grupo host con PowerShell](https://docs.microsoft.com/azure/virtual-desktop/create-host-pools-powershell).

### <a name="error-the-admin-username-specified-isnt-allowed"></a>Error: No se permite el nombre de usuario de administrador especificado

![Captura de pantalla de la implementación de error en el que un administrador especificado no está permitido.](media/f2b3d3700e9517463ef88fa41875bac9.png)

Ejemplo de error bruto:

```Error
 { "id": "/subscriptions/EXAMPLE/resourceGroups/demoHostDesktop/providers/Microsoft.
  Resources/deployments/vmCreation-linkedTemplate/operations/EXAMPLE", "operationId": "EXAMPLE", "properties": { "provisioningOperation":
 "Create", "provisioningState": "Failed", "timestamp": "2019-01-29T20:53:18.904917Z", "duration": "PT3.0574505S", "trackingId":
 "1f460af8-34dd-4c03-9359-9ab249a1a005", "statusCode": "BadRequest", "statusMessage": { "error": { "code": "InvalidParameter", "message":
 "The Admin Username specified is not allowed.", "target": "adminUsername" } }, "targetResource": { "id": "/subscriptions/EXAMPLE
 /resourceGroups/demoHostDesktop/providers/Microsoft.Compute/virtualMachines/demo", "resourceType": "Microsoft.Compute/virtualMachines", "resourceName": "demo" } }}
```

**Causa:** Contraseña proporcionada contiene subcadenas prohibidas (admin, administrador, raíz).

**Fix:** Actualizar nombre de usuario o usar distintos usuarios.

### <a name="error-vm-has-reported-a-failure-when-processing-extension"></a>Error: Máquina virtual indicó un error al procesar la extensión

![Captura de pantalla de la operación de recursos que se ha completado con el estado de aprovisionamiento en el error de implementación.](media/49c4a1836a55d91cd65125cf227f411f.png)

Ejemplo de error bruto:

```Error
{ "id": "/subscriptions/EXAMPLE/resourceGroups/demoHostD/providers/Microsoft.Resources/deployments/
 rds.wvd-hostpool4-preview-20190129132410/operations/5A0757AC9E7205D2", "operationId": "5A0757AC9E7205D2", "properties":
 { "provisioningOperation": "Create", "provisioningState": "Failed", "timestamp": "2019-01-29T21:43:05.1416423Z",
 "duration": "PT7M56.8150879S", "trackingId": "43c4f71f-557c-4abd-80c3-01f545375455", "statusCode": "Conflict",
 "statusMessage": { "status": "Failed", "error": { "code": "ResourceDeploymentFailure", "message":
 "The resource operation completed with terminal provisioning state 'Failed'.", "details": [ { "code":
 "VMExtensionProvisioningError", "message": "VM has reported a failure when processing extension 'dscextension'. 
 Error message: \"DSC Configuration 'SessionHost' completed with error(s). Following are the first few: 
 PowerShell DSC resource MSFT_ScriptResource failed to execute Set-TargetResource functionality with error message: 
 One or more errors occurred. The SendConfigurationApply function did not succeed.\"." } ] } }, "targetResource": 
 { "id": "/subscriptions/EXAMPLE/resourceGroups/demoHostD/providers/Microsoft. 
 Compute/virtualMachines/desktop-1/extensions/dscextension",
 "resourceType": "Microsoft.Compute/virtualMachines/extensions", "resourceName": "desktop-1/dscextension" } }}
```

**Causa:** Extensión DSC de PowerShell no pudo obtener acceso de administrador en la máquina virtual.

**Fix:** Confirmar nombre de usuario y la contraseña tienen acceso administrativo en la máquina virtual y vuelva a ejecutan la plantilla de Azure Resource Manager.

### <a name="error-deploymentfailed--powershell-dsc-configuration-firstsessionhost-completed-with-errors"></a>Error: DeploymentFailed: configuración de DSC de PowerShell 'FirstSessionHost' se completó con errores

![Captura de pantalla de la implementación producirá un error de configuración de DSC de PowerShell 'FirstSessionHost' se completó con errores.](media/64870370bcbe1286906f34cf0a8646ab.png)

Ejemplo de error bruto:

```Error
{
    "code": "DeploymentFailed",
   "message": "At least one resource deployment operation failed. Please list 
 deployment operations for details. 4 Please see https://aka.ms/arm-debug for usage details.",
 "details": [
         { "code": "Conflict",  
         "message": "{\r\n \"status\": \"Failed\",\r\n \"error\": {\r\n \"code\":
         \"ResourceDeploymentFailure\",\r\n \"message\": \"The resource
         operation completed with terminal provisioning state 'Failed'.\",\r\n
         \"details\": [\r\n {\r\n \"code\":
        \"VMExtensionProvisioningError\",\r\n \"message\": \"VM has
              reported a failure when processing extension 'dscextension'.
              Error message: \\\"DSC Configuration 'FirstSessionHost'
              completed with error(s). Following are the first few:
              PowerShell DSC resource MSFT ScriptResource failed to
              execute Set-TargetResource functionality with error message:
              One or more errors occurred. The SendConfigurationApply
              function did not succeed.\\\".\"\r\n }\r\n ]\r\n }\r\n}"  }

```

**Causa:** Extensión DSC de PowerShell no pudo obtener acceso de administrador en la máquina virtual.

**Fix:** Confirmar nombre de usuario y la contraseña proporcionada tienen acceso administrativo en la máquina virtual y vuelva a ejecutar la plantilla de Azure Resource Manager.

### <a name="error-deploymentfailed--invalidresourcereference"></a>Error: DeploymentFailed – InvalidResourceReference

Ejemplo de error bruto:

```Error
{"code":"DeploymentFailed","message":"At least one resource deployment operation
failed. Please list deployment operations for details. Please see https://aka.ms/arm-
debug for usage details.","details":[{"code":"Conflict","message":"{\r\n \"status\":
\"Failed\",\r\n \"error\": {\r\n \"code\": \"ResourceDeploymentFailure\",\r\n
\"message\": \"The resource operation completed with terminal provisioning state
'Failed'.\",\r\n \"details\": [\r\n {\r\n \"code\": \"DeploymentFailed\",\r\n
\"message\": \"At least one resource deployment operation failed. Please list
deployment operations for details. Please see https://aka.ms/arm-debug for usage
details.\",\r\n \"details\": [\r\n {\r\n \"code\": \"BadRequest\",\r\n \"message\":
\"{\\r\\n \\\"error\\\": {\\r\\n \\\"code\\\": \\\"InvalidResourceReference\\\",\\r\\n
\\\"message\\\": \\\"Resource /subscriptions/EXAMPLE/resourceGroups/ernani-wvd-
demo/providers/Microsoft.Network/virtualNetworks/wvd-vnet/subnets/default
referenced by resource /subscriptions/EXAMPLE/resourceGroups/ernani-wvd-
demo/providers/Microsoft.Network/networkInterfaces/erd. Please make sure that
the referenced resource exists, and that both resources are in the same
region.\\\",\\r\\n\\\"details\\\": []\\r\\n }\\r\\n}\"\r\n }\r\n ]\r\n }\r\n ]\r\n }\r\n}"}]}
```

**Causa:** Parte del nombre del grupo de recursos se usa para determinados recursos que se crean mediante la plantilla. Debido a los recursos existentes de coincidencia de nombres, la plantilla puede seleccionar un recurso existente desde un grupo diferente.

**Fix:** Cuando se ejecuta la plantilla de Azure Resource Manager para implementar las máquinas virtuales del host de sesión, asegúrese de los dos primeros caracteres únicos para el nombre del grupo de recursos de suscripción.

### <a name="error-deploymentfailed--invalidresourcereference"></a>Error: DeploymentFailed – InvalidResourceReference

Ejemplo de error bruto:

```Error
{"code":"DeploymentFailed","message":"At least one resource deployment operation
failed. Please list deployment operations for details. Please see https://aka.ms/arm-
debug for usage details.","details":[{"code":"Conflict","message":"{\r\n \"status\":
\"Failed\",\r\n \"error\": {\r\n \"code\": \"ResourceDeploymentFailure\",\r\n
\"message\": \"The resource operation completed with terminal provisioning state
'Failed'.\",\r\n \"details\": [\r\n {\r\n \"code\": \"DeploymentFailed\",\r\n
\"message\": \"At least one resource deployment operation failed. Please list
deployment operations for details. Please see https://aka.ms/arm-debug for usage
details.\",\r\n \"details\": [\r\n {\r\n \"code\": \"BadRequest\",\r\n \"message\":
\"{\\r\\n \\\"error\\\": {\\r\\n \\\"code\\\": \\\"InvalidResourceReference\\\",\\r\\n
\\\"message\\\": \\\"Resource /subscriptions/EXAMPLE/resourceGroups/ernani-wvd-
demo/providers/Microsoft.Network/virtualNetworks/wvd-vnet/subnets/default
referenced by resource /subscriptions/EXAMPLE/resourceGroups/DEMO/providers/Microsoft.Network/networkInterfaces
/EXAMPLE was not found. Please make sure that the referenced resource exists, and that both
resources are in the same region.\\\",\\r\\n \\\"details\\\": []\\r\\n }\\r\\n}\"\r\n
}\r\n ]\r\n }\r\n ]\r\n }\r\n\
```

**Causa:** Este error es porque la NIC creadas con la plantilla de Azure Resource Manager tiene el mismo nombre que otra NIC de la red virtual.

**Fix:** Utilizar un prefijo de host diferente.

### <a name="error-deploymentfailed--error-downloading"></a>Error: DeploymentFailed: Error al descargar

Ejemplo de error bruto:

```Error
\\\"The DSC Extension failed to execute: Error downloading
https://catalogartifact.azureedge.net/publicartifacts/rds.wvd-hostpool-3-preview-
2dec7a4d-006c-4cc0-965a-02bbe438d6ff-private-preview-
1/Artifacts/DSC/Configuration.zip after 29 attempts: The remote name could not be
resolved: 'catalogartifact.azureedge.net'.\\nMore information about the failure can
be found in the logs located under
'C:\\\\WindowsAzure\\\\Logs\\\\Plugins\\\\Microsoft.Powershell.DSC\\\\2.77.0.0' on
the VM.\\\"
```

**Causa:** Este error es debido a una ruta estática, regla de firewall o NSG bloquea la descarga del archivo zip que se relacionan con la plantilla de Azure Resource Manager.

**Fix:** Quitar bloqueo de ruta estática, regla de firewall o NSG. Si lo desea, abra el archivo json de plantilla de Azure Resource Manager en un editor de texto, tomar el vínculo al archivo zip y descargar el recurso a una ubicación permitida.

### <a name="error-the-user-isnt-authorized-to-query-the-management-service"></a>Error: El usuario no está autorizado para consultar el servicio de administración

Ejemplo de error bruto:

```Error
"response": { "content": { "startTime": "2019-04-01T17:45:33.3454563+00:00", "endTime": "2019-04-01T17:48:52.4392099+00:00", 
"status": "Failed", "error": { "code": "VMExtensionProvisioningError", "message": "VM has reported a failure when processing 
extension 'dscextension'. Error message: \"DSC Configuration 'FirstSessionHost' completed with error(s). 
Following are the first few: PowerShell DSC resource MSFT_ScriptResource failed to execute Set-TargetResource
 functionality with error message: User is not authorized to query the management service.
\nActivityId: 1b4f2b37-59e9-411e-9d95-4f7ccd481233\nPowershell commands to diagnose the failure:
\nGet-RdsDiagnosticActivities -ActivityId 1b4f2b37-59e9-411e-9d95-4f7ccd481233\n 
The SendConfigurationApply function did not succeed.\"." }, "name": "2c3272ec-d25b-47e5-8d70-a7493e9dc473" } } }}
```

**Causa:** El Administrador de inquilinos de Escritorio Virtual de Windows especificado no tiene una asignación de roles válidos.

**Fix:** El usuario que creó al inquilino de Escritorio Virtual de Windows necesita iniciar sesión Windows PowerShell de Escritorio Virtual y asigne al usuario ha intentado efectuar una asignación de roles. Si está ejecutando los parámetros de plantilla de GitHub de Azure Resource Manager, siga estas instrucciones mediante comandos de PowerShell:

```PowerShell
Add-RdsAccount -DeploymentUrl “https://rdbroker.wvd.microsoft.com”
New-RdsRoleAssignment -TenantName <Windows Virtual Desktop tenant name> -RoleDefinitionName “RDS Contributor” -SignInName <UPN>
```

### <a name="error-user-requires-azure-multi-factor-authentication-mfa"></a>Error: Usuario que requiere Azure Multi-factor Authentication (MFA)

![Captura de pantalla de la implementación de error debido a la falta de Multi-factor Authentication (MFA)](media/MFARequiredError.png)

Ejemplo de error bruto:

```Error
"message": "{\r\n  \"status\": \"Failed\",\r\n  \"error\": {\r\n    \"code\": \"ResourceDeploymentFailure\",\r\n    \"message\": \"The resource operation completed with terminal provisioning state 'Failed'.\",\r\n    \"details\": [\r\n      {\r\n        \"code\": \"VMExtensionProvisioningError\",\r\n        \"message\": \"VM has reported a failure when processing extension 'dscextension'. Error message: \\\"DSC Configuration 'FirstSessionHost' completed with error(s). Following are the first few: PowerShell DSC resource MSFT_ScriptResource  failed to execute Set-TargetResource functionality with error message: One or more errors occurred.  The SendConfigurationApply function did not succeed.\\\".\"\r\n      }\r\n    ]\r\n  }\r\n}"
```

**Causa:** El Administrador de inquilinos de Escritorio Virtual de Windows especificado requiere Azure Multi-factor Authentication (MFA) para iniciar sesión.

**Fix:** Crear una entidad de servicio y asígnele un rol para el inquilino de Escritorio Virtual de Windows siguiendo los pasos descritos en [Tutorial: Creación de entidades de servicio y las asignaciones de roles con PowerShell](https://docs.microsoft.com/azure/virtual-desktop/create-service-principal-role-powershell). Después de comprobar que puede iniciar sesión Escritorio Virtual de Windows con la entidad de servicio, vuelva a ejecutar la oferta de Azure Marketplace o en la plantilla de GitHub de Azure Resource Manager, dependiendo de qué método usa. Siga las instrucciones siguientes para especificar los parámetros correctos para el método.

Si está ejecutando la oferta de Azure Marketplace, proporcione valores para los parámetros siguientes autenticar correctamente al escritorio Virtual de Windows:

- Propietario de RDS del inquilino de Escritorio Virtual de Windows: Entidad de servicio
- Identificador de aplicación: La identificación de la aplicación de la nueva entidad de servicio que creó.
- Contraseña/Confirmar contraseña: El secreto de contraseña generado para la entidad de servicio
- Id. de inquilino de Azure AD: El identificador de inquilino de Azure AD de la entidad de servicio que creó.

Si ejecuta la plantilla de GitHub de Azure Resource Manager, proporcione valores para los parámetros siguientes autenticar correctamente al escritorio Virtual de Windows:

- Nombre principal de usuario de administrador de inquilinos (UPN) o Id. de aplicación: La identificación de la aplicación de la nueva entidad de servicio que creó.
- Contraseña de administrador de inquilinos: El secreto de contraseña generado para la entidad de servicio
- IsServicePrincipal: **true**
- AadTenantId: El identificador de inquilino de Azure AD de la entidad de servicio que creó.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general sobre cómo solucionar problemas de Escritorio Virtual de Windows y las pistas de escalado, consulte [solución de problemas de introducción, comentarios y soporte técnico](troubleshoot-set-up-overview.md).
- Para solucionar problemas al configurar una máquina virtual (VM) en el escritorio Virtual de Windows, consulte [configuración de máquina virtual del host de sesión](troubleshoot-vm-configuration.md).
- Para solucionar problemas con conexiones de cliente de Escritorio Virtual de Windows, consulte [las conexiones de cliente de escritorio remoto](troubleshoot-client-connection.md).
- Para solucionar problemas al usar PowerShell con el escritorio Virtual de Windows, consulte [Windows PowerShell de Escritorio Virtual](troubleshoot-powershell.md).
- Para obtener más información sobre el servicio en versión preliminar, consulte [entorno de versión preliminar de Windows Desktop](https://docs.microsoft.com/azure/virtual-desktop/environment-setup).
- Para realizar un tutorial de solución de problemas, consulte [Tutorial: Solución de problemas de implementaciones de plantilla de Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-tutorial-troubleshoot).
- Para más información sobre las acciones de auditoría, consulte [Operaciones de auditoría con Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-audit).
- Si desea conocer más detalles sobre las acciones que permiten determinar los errores durante la implementación, consulte [Visualización de operaciones de implementación con el Portal de Azure](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-deployment-operations).