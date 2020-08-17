---
title: 'Creación de grupos de inquilinos y de hosts de Windows Virtual Desktop (clásico): Azure'
description: Solución de problemas de grupos de inquilinos y de hosts durante la instalación de un entorno de inquilinos de Windows Virtual Desktop (clásico).
author: Heidilohr
ms.topic: troubleshooting
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: eed1b0e1b01d5d13330b927429eca9a28ff80658
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009263"
---
# <a name="tenant-and-host-pool-creation-in-windows-virtual-desktop-classic"></a>Creación de grupos de inquilinos y de hosts en Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager. Si está tratando de administrar objetos de Windows Virtual Desktop para Azure Resource Manager, consulte [este artículo](../troubleshoot-set-up-issues.md).

En este artículo se tratan problemas que se producen durante la instalación inicial del inquilino de Windows Virtual Desktop y la infraestructura del grupo de host de la sesión relacionada.

## <a name="provide-feedback"></a>Envío de comentarios

Visite la [Comunidad técnica de Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop) para hablar sobre Windows Virtual Desktop con el equipo de producto y los miembros activos de la comunidad.

## <a name="acquiring-the-windows-10-enterprise-multi-session-image"></a>Adquisición de la imagen de sesión múltiple de Windows 10 Enterprise

Para usar la imagen de sesión múltiple de Windows 10 Enterprise, vaya a Azure Marketplace, seleccione **Comenzar** > **Microsoft Windows 10** > y [Windows 10 Enterprise para escritorios virtuales, versión 1809](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsdesktop.windows-10?tab=PlansAndPrice).

> [!div class="mx-imgBorder"]
> ![Una captura de pantalla de selección de Windows 10 Enterprise para escritorios virtuales, versión 1809.](../media/AzureMarketPlace.png)

## <a name="creating-windows-virtual-desktop-tenant"></a>Crear un inquilino de Windows Virtual Desktop

En esta sección se tratan los posibles problemas al crear el inquilino de Windows Virtual Desktop.

### <a name="error-aadsts650052-the-app-needs-access-to-a-service"></a>Error: AADSTS650052 La aplicación necesita acceso a un servicio.

Ejemplo de error no procesado:

```Error
AADSTS650052 Message The app needs access to a service(\"{name}\") that your organization
\"{organization}\" has not subscribed to or enabled. Contact your IT Admin to review the
configuration of your service subscriptions.650052 Message The app needs access to a service
(\"{name}\") that your organization \"{organization}\" has not subscribed to or enabled.
Contact your IT Admin to review the configuration of your service subscriptions.
```

**Causa:** Consentimiento no concedido Windows Virtual Desktop en la instancia de Azure Active Directory.

**Solución:** [Siga esta guía](https://docs.microsoft.com/azure/virtual-desktop/virtual-desktop-fall-2019/tenant-setup-azure-active-directory#grant-permissions-to-windows-virtual-desktop) para conceder consentimiento.

### <a name="error-the-user-isnt-authorized-to-query-the-management-service"></a>Error: El usuario no está autorizado para consultar el servicio de administración

> [!div class="mx-imgBorder"]
> ![Captura de pantalla de una ventana de PowerShell en que un usuario no está autorizado para consultar el servicio de administración.](../media/UserNotAuthorizedNewTenant.png)

Ejemplo de error no procesado:

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

**Causa:** No han asignado al usuario que ha iniciado sesión el rol TenantCreator en su Azure Active Directory.

**Solución:** Siga las instrucciones de [Asignar el rol de aplicación TenantCreator a un usuario en el inquilino de Azure Active Directory](tenant-setup-azure-active-directory.md#assign-the-tenantcreator-application-role). Después de seguir las instrucciones, tendrá un usuario asignado al rol TenantCreator.

> [!div class="mx-imgBorder"]
> ![Captura de pantalla del rol TenantCreator asignado.](../media/TenantCreatorRoleAssigned.png)

## <a name="creating-windows-virtual-desktop-session-host-vms"></a>Creación de máquinas virtuales de host de sesión de Windows Virtual Desktop

Las máquinas virtuales de host de sesión se pueden crear de varias maneras, pero el equipo de Windows Virtual Desktop solo admite problemas de aprovisionamiento de máquinas virtuales relacionados con la oferta de [Azure Marketplace](https://azuremarketplace.microsoft.com/). Para más información, vea [Problemas con Windows Virtual Desktop: aprovisionar una oferta de Azure Marketplace de grupo de host](#issues-using-windows-virtual-desktop--provision-a-host-pool-azure-marketplace-offering).

## <a name="issues-using-windows-virtual-desktop--provision-a-host-pool-azure-marketplace-offering"></a>Problemas con Windows Virtual Desktop: aprovisionar una oferta de Azure Marketplace de grupo de host

Windows Virtual Desktop: en Azure Marketplace está disponible una plantilla de grupo de host.

### <a name="error-when-using-the-link-from-github-the-message-create-a-free-account-appears"></a>Error: El mensaje "Crear una cuenta gratuita" se muestra al utilizar el vínculo de GitHub

> [!div class="mx-imgBorder"]
> ![Captura de pantalla para crear una cuenta gratuita.](../media/be615904ace9832754f0669de28abd94.png)

**Causa 1:** No existen suscripciones activas en la cuenta utilizada para iniciar sesión en Azure, o bien la cuenta utilizada no tiene permisos para ver las suscripciones.

**Corrección 1:** Inicie sesión con una cuenta que tenga acceso de colaborador (como mínimo) en la suscripción en que se van a implementar las máquinas virtuales del host de sesión.

**Causa 2:** La suscripción que se utiliza es parte de un inquilino del proveedor de servicios en la nube (CSP) de Microsoft.

**Corrección 2:** Vaya a la ubicación de GitHub, consulte **Creación y aprovisionamiento de un grupo de host nuevo de Windows Virtual Desktop** y siga estas instrucciones:

1. Haga clic con el botón derecho en **Implementar en Azure** y seleccione **Copiar dirección de vínculo**.
2. Abra el **Bloc de notas** y pegue el vínculo.
3. Antes del carácter #, inserte el nombre del inquilino del cliente final del CSP.
4. Abra el vínculo nuevo en un explorador y la plantilla se cargará en el portal de Azure.

    ```Example
    Example: https://portal.azure.com/<CSP end customer tenant name>
    #create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%
    2FRDS-Templates%2Fmaster%2Fwvd-templates%2FCreate%20and%20provision%20WVD%20host%20pool%2FmainTemplate.json
    ```

### <a name="error-you-receive-template-deployment-is-not-valid-error"></a>Error: recibe el error "la implementación de la plantilla no es válida"

> [!div class="mx-imgBorder"]
> ![Captura de pantalla del error "la implementación de la plantilla no es válida"](../media/troubleshooting-marketplace-validation-error-generic.png)

Antes de realizar una acción específica, deberá comprobar el registro de actividad para ver el error detallado de la validación de la implementación con errores.

Para ver el error en el registro de actividad:

1. Salga de la oferta de implementación de Azure Marketplace actual.
2. En la barra de búsqueda superior, busque y seleccione **Registro de actividad**.
3. Busque una actividad denominada **Validar implementación** que tenga el estado **Erróneo** y seleccione la actividad.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de la actividad individual **Validar implementación** con un estado *Erróneo*](../media/troubleshooting-marketplace-validation-error-activity-summary.png)

4. Seleccione JSON y, a continuación, desplácese hacia abajo hasta la parte inferior de la pantalla hasta que vea el campo "statusMessage".

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de la actividad con errores, con un cuadro rojo alrededor de la propiedad statusMessage del texto JSON.](../media/troubleshooting-marketplace-validation-error-json-boxed.png)

Si la plantilla de la operación supera el límite de cuota, puede realizar una de las siguientes acciones para solucionarlo:

 - Ejecute Azure Marketplace con los parámetros que usó la primera vez, pero esta vez use menos máquinas virtuales y núcleos de máquina virtual.
 - Abra el vínculo que aparece en el campo **statusMessage** en un explorador para enviar una solicitud a fin de aumentar la cuota de su suscripción a Azure para la SKU de la máquina virtual especificada.

## <a name="azure-resource-manager-template-and-powershell-desired-state-configuration-dsc-errors"></a>Errores de la plantilla de Azure Resource Manager y de la Configuración de estado deseado (DSC) de PowerShell

Siga estas instrucciones para solucionar problemas de implementaciones incorrectas de DSC de PowerShell y plantillas de Azure Resource Manager.

1. Revise los errores en la implementación mediante [Ver operaciones de implementación con Azure Resource Manager](../../azure-resource-manager/resource-manager-deployment-operations.md).
2. Si no hay ningún error en la implementación, revise los errores en el registro de actividad mediante [Ver registros de actividad para auditar las acciones sobre los recursos](../../azure-resource-manager/resource-group-audit.md).
3. Una vez identificado el error, utilice el mensaje de error y los recursos de [Solucionar errores comunes de implementación de Azure con Azure Resource Manager](../../azure-resource-manager/resource-manager-common-deployment-errors.md) para solucionar el problema.
4. Elimine los recursos creados durante la implementación anterior y vuelva a intentar implementar la plantilla.

### <a name="error-your-deployment-failedhostnamejoindomain"></a>Error: La implementación no se pudo…\<hostname>/joindomain

> [!div class="mx-imgBorder"]
> ![Captura de pantalla de error de implementación.](../media/e72df4d5c05d390620e07f0d7328d50f.png)

Ejemplo de error no procesado:

```Error
 {"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details.
 Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"Conflict","message":"{\r\n \"status\": \"Failed\",\r\n \"error\":
 {\r\n \"code\": \"ResourceDeploymentFailure\",\r\n \"message\": \"The resource operation completed with terminal provisioning state 'Failed'.
 \",\r\n \"details\": [\r\n {\r\n \"code\": \"VMExtensionProvisioningError\",\r\n \"message\": \"VM has reported a failure when processing
 extension 'joindomain'. Error message: \\\"Exception(s) occurred while joining Domain 'diamondsg.onmicrosoft.com'\\\".\"\r\n }\r\n ]\r\n }\r\n}"}]}
```

**Causa 1:** Las credenciales proporcionadas para unir las máquinas virtuales al dominio son incorrectas.

**Corrección 1:** Vea el error "Credenciales incorrectas" cuando las máquinas virtuales no están unidas al dominio en [Configuración de máquina virtual del host de sesión](troubleshoot-vm-configuration-2019.md).

**Causa 2:** No se resuelve el nombre de dominio.

**Corrección 2:** Consulte [Error: El nombre de dominio no se resuelve](troubleshoot-vm-configuration-2019.md#error-domain-name-doesnt-resolve) en [Configuración de máquina virtual del host de sesión](troubleshoot-vm-configuration-2019.md).

**Causa 3:** La configuración de DNS de la red virtual (VNET) está establecida en **Predeterminada**.

Para corregirlo, siga estos pasos:

1. Abra Azure Portal y vaya a la pestaña **Redes virtuales**.
2. Busque la red virtual y, después, seleccione **Servidores DNS**.
3. El menú Servidores DNS debe aparecer en el lado derecho de la pantalla. En ese menú, seleccione **Personalizado**.
4. Asegúrese de que los servidores DNS que aparecen en Personalizado coinciden con el controlador de dominio o el dominio de Active Directory. Si no ve el servidor DNS, escriba su valor en el campo **Agregar servidor DNS** para agregarlo.

### <a name="error-your-deployment-failedunauthorized"></a>Error: Error al realizar la implementación...\Unauthorized

```Error
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"Unauthorized","message":"{\r\n \"Code\": \"Unauthorized\",\r\n \"Message\": \"The scale operation is not allowed for this subscription in this region. Try selecting different region or scale option.\",\r\n \"Target\": null,\r\n \"Details\": [\r\n {\r\n \"Message\": \"The scale operation is not allowed for this subscription in this region. Try selecting different region or scale option.\"\r\n },\r\n {\r\n \"Code\": \"Unauthorized\"\r\n },\r\n {\r\n \"ErrorEntity\": {\r\n \"ExtendedCode\": \"52020\",\r\n \"MessageTemplate\": \"The scale operation is not allowed for this subscription in this region. Try selecting different region or scale option.\",\r\n \"Parameters\": [\r\n \"default\"\r\n ],\r\n \"Code\": \"Unauthorized\",\r\n \"Message\": \"The scale operation is not allowed for this subscription in this region. Try selecting different region or scale option.\"\r\n }\r\n }\r\n ],\r\n \"Innererror\": null\r\n}"}]}
```

**Causa:** La suscripción que está usando es de un tipo que no puede acceder a las características necesarias en la región donde el cliente está intentando realizar la implementación. Por ejemplo, las suscripciones a MSDN, Free o Education pueden mostrar este error.

**Solución:** Cambie el tipo de suscripción o la región a uno que pueda acceder a las características necesarias.

### <a name="error-vmextensionprovisioningerror"></a>Error: VMExtensionProvisioningError

> [!div class="mx-imgBorder"]
> ![Captura de pantalla de error de implementación con error de estado de aprovisionamiento terminal.](../media/7aaf15615309c18a984673be73ac969a.png)

**Causa 1:** Error transitorio del entorno de Windows Virtual Desktop.

**Causa 2:** Error transitorio de conexión.

**Solución:** Inicie sesión con PowerShell para confirmar que el entorno de Windows Virtual Desktop es correcto. Finalice manualmente el registro de la máquina virtual en [Creación de un grupo de host con PowerShell](create-host-pools-powershell-2019.md).

### <a name="error-the-admin-username-specified-isnt-allowed"></a>Error: El nombre de usuario de administrador especificado no está permitido

> [!div class="mx-imgBorder"]
> ![Captura de pantalla de error de implementación en que un administrador especificado no está permitido.](../media/f2b3d3700e9517463ef88fa41875bac9.png)

Ejemplo de error no procesado:

```Error
 { "id": "/subscriptions/EXAMPLE/resourceGroups/demoHostDesktop/providers/Microsoft.
  Resources/deployments/vmCreation-linkedTemplate/operations/EXAMPLE", "operationId": "EXAMPLE", "properties": { "provisioningOperation":
 "Create", "provisioningState": "Failed", "timestamp": "2019-01-29T20:53:18.904917Z", "duration": "PT3.0574505S", "trackingId":
 "1f460af8-34dd-4c03-9359-9ab249a1a005", "statusCode": "BadRequest", "statusMessage": { "error": { "code": "InvalidParameter", "message":
 "The Admin Username specified is not allowed.", "target": "adminUsername" } }, "targetResource": { "id": "/subscriptions/EXAMPLE
 /resourceGroups/demoHostDesktop/providers/Microsoft.Compute/virtualMachines/demo", "resourceType": "Microsoft.Compute/virtualMachines", "resourceName": "demo" } }}
```

**Causa:** La contraseña proporcionada contiene subcadenas prohibidas (admin, administrador, raíz).

**Solución:** Actualice el nombre de usuario o utilice distintos usuarios.

### <a name="error-vm-has-reported-a-failure-when-processing-extension"></a>Error: La máquina virtual ha indicado un error al procesar la extensión

> [!div class="mx-imgBorder"]
> ![Captura de pantalla de la operación de recursos completa con un estado de aprovisionamiento terminal de Error de implementación.](../media/49c4a1836a55d91cd65125cf227f411f.png)

Ejemplo de error no procesado:

```Error
{ "id": "/subscriptions/EXAMPLE/resourceGroups/demoHostD/providers/Microsoft.Resources/deployments/
 rds.wvd-provision-host-pool-20190129132410/operations/5A0757AC9E7205D2", "operationId": "5A0757AC9E7205D2", "properties":
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

**Causa:** La extensión DSC de PowerShell no ha podido obtener acceso de administrador a la máquina virtual.

**Solución:** Confirme que el nombre de usuario y la contraseña tienen acceso administrativo a la máquina virtual y vuelva a ejecutar la plantilla de Azure Resource Manager.

### <a name="error-deploymentfailed--powershell-dsc-configuration-firstsessionhost-completed-with-errors"></a>Error: DeploymentFailed: la configuración de DSC de PowerShell "FirstSessionHost" se ha completado con errores

> [!div class="mx-imgBorder"]
> ![Captura de pantalla de error de implementación con la configuración de DSC de PowerShell "FirstSessionHost" completada con errores.](../media/64870370bcbe1286906f34cf0a8646ab.png)

Ejemplo de error no procesado:

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

**Causa:** La extensión DSC de PowerShell no ha podido obtener acceso de administrador a la máquina virtual.

**Solución:** Confirme que el nombre de usuario y la contraseña proporcionados tienen acceso administrativo a la máquina virtual y vuelva a ejecutar la plantilla de Azure Resource Manager.

### <a name="error-deploymentfailed--invalidresourcereference"></a>Error: DeploymentFailed: InvalidResourceReference

Ejemplo de error no procesado:

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

**Causa:** Parte del nombre del grupo de recursos se usa para determinados recursos que se crean mediante la plantilla. Debido a los recursos existentes con coincidencia de nombres, la plantilla puede seleccionar un recurso existente de un grupo diferente.

**Solución:** Al ejecutar la plantilla de Azure Resource Manager para implementar las máquinas virtuales del host de sesión, asegúrese de los dos primeros caracteres sean únicos para el nombre del grupo de recursos de suscripción.

### <a name="error-deploymentfailed--invalidresourcereference"></a>Error: DeploymentFailed: InvalidResourceReference

Ejemplo de error no procesado:

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

**Causa:** Este error se debe a que la NIC creada con la plantilla de Azure Resource Manager tiene el mismo nombre que otra NIC de la red virtual.

**Solución:** Utilice un prefijo de host diferente.

### <a name="error-deploymentfailed--error-downloading"></a>Error: DeploymentFailed: error al descargar

Ejemplo de error no procesado:

```Error
\\\"The DSC Extension failed to execute: Error downloading
https://catalogartifact.azureedge.net/publicartifacts/rds.wvd-provision-host-pool-
2dec7a4d-006c-4cc0-965a-02bbe438d6ff-prod
/Artifacts/DSC/Configuration.zip after 29 attempts: The remote name could not be
resolved: 'catalogartifact.azureedge.net'.\\nMore information about the failure can
be found in the logs located under
'C:\\\\WindowsAzure\\\\Logs\\\\Plugins\\\\Microsoft.Powershell.DSC\\\\2.77.0.0' on
the VM.\\\"
```

**Causa:** Este error se debe a que una ruta estática, una regla de firewall o un NSG bloquean la descarga del archivo zip vinculado a la plantilla de Azure Resource Manager.

**Solución:** Suprima la ruta estática de bloqueo, la regla de firewall o el NSG. De manera opcional, abra el archivo json de plantilla de Azure Resource Manager en un editor de texto, comprima el vínculo en un archivo zip y descargue el recurso en una ubicación permitida.

### <a name="error-the-user-isnt-authorized-to-query-the-management-service"></a>Error: El usuario no está autorizado para consultar el servicio de administración

Ejemplo de error no procesado:

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

**Causa:** El administrador de inquilinos de Windows Virtual Desktop especificado no tiene una asignación de roles válida.

**Solución:** El usuario que ha creado el inquilino de Windows Virtual Desktop debe iniciar sesión en PowerShell para Windows Virtual Desktop y asignar un rol al usuario que lo ha intentado. Si está ejecutando los parámetros de la plantilla de GitHub Azure Resource Manager, siga estas instrucciones mediante los comandos de PowerShell:

```PowerShell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
New-RdsRoleAssignment -TenantName <Windows Virtual Desktop tenant name> -RoleDefinitionName "RDS Contributor" -SignInName <UPN>
```

### <a name="error-user-requires-azure-multi-factor-authentication-mfa"></a>Error: El usuario requiere autenticación multifactor (MFA) de Azure

> [!div class="mx-imgBorder"]
> ![Captura de pantalla de error de implementación debido a la falta de Autenticación multifactor (MFA)](../media/MFARequiredError.png)

Ejemplo de error no procesado:

```Error
"message": "{\r\n  \"status\": \"Failed\",\r\n  \"error\": {\r\n    \"code\": \"ResourceDeploymentFailure\",\r\n    \"message\": \"The resource operation completed with terminal provisioning state 'Failed'.\",\r\n    \"details\": [\r\n      {\r\n        \"code\": \"VMExtensionProvisioningError\",\r\n        \"message\": \"VM has reported a failure when processing extension 'dscextension'. Error message: \\\"DSC Configuration 'FirstSessionHost' completed with error(s). Following are the first few: PowerShell DSC resource MSFT_ScriptResource  failed to execute Set-TargetResource functionality with error message: One or more errors occurred.  The SendConfigurationApply function did not succeed.\\\".\"\r\n      }\r\n    ]\r\n  }\r\n}"
```

**Causa:** El administrador de inquilinos de Windows Virtual Desktop especificado requiere autenticación multifactor (MFA) de Azure para iniciar sesión.

**Solución:** Cree una entidad de servicio y asígnele un rol para el inquilino de Windows Virtual Desktop mediante los pasos descritos en [Tutorial: Creación de entidades de servicio y asignaciones de roles con PowerShell](create-service-principal-role-powershell.md). Después de comprobar que puede iniciar sesión en Windows Virtual Desktop con la entidad de servicio, vuelva a ejecutar la oferta de Azure Marketplace o la plantilla de GitHub Azure Resource Manager, según el método que utilice. Siga las instrucciones siguientes para introducir los parámetros correctos para el método.

Si está ejecutando la oferta de Azure Marketplace, proporcione valores para los parámetros siguientes a fin de autenticarse correctamente en Windows Virtual Desktop:

- Propietario de RDS del inquilino de Windows Virtual Desktop: Entidad de servicio
- Identificador de aplicación: La identificación de la aplicación de la nueva entidad de servicio que ha creado
- Contraseña/Confirmar contraseña: El secreto de contraseña generado para la entidad de servicio
- Identificador de inquilino de Azure AD: El identificador de inquilino de Azure AD de la entidad de servicio que ha creado

Si está ejecutando la plantilla de GitHub Azure Resource Manager, proporcione valores para los parámetros siguientes a fin de autenticarse correctamente en Windows Virtual Desktop:

- Nombre principal de usuario de administrador de inquilinos (UPN) o identificador de aplicación: La identificación de la aplicación de la nueva entidad de servicio que ha creado
- Contraseña del administrador de inquilinos: El secreto de contraseña generado para la entidad de servicio
- IsServicePrincipal: **true**
- AadTenantId: El identificador de inquilino de Azure AD de la entidad de servicio que ha creado

### <a name="error-vmsubnet-not-available-when-configuring-virtual-networks"></a>Error: vmSubnet no disponible al configurar redes virtuales

**Causa:** En la plantilla WVD Marketplace, la interfaz de usuario solo muestra las subredes que tienen al menos tantas direcciones IP disponibles como el número total de máquinas virtuales especificadas en la plantilla. El número real de direcciones IP disponibles en la subred solo debe ser igual al número de nuevas máquinas virtuales que se implementan, pero no puede calcularse con la interfaz de usuario actual.

**Solución:** Puede especificar una subred con al menos tantas direcciones IP disponibles como la cantidad de máquinas virtuales que se agregan al no usar la interfaz de usuario de Marketplace, esto se puede hacer especificando el nombre de subred en el parámetro "**existingSubnetName**" cuando [volver a implementar una implementación existente](expand-existing-host-pool-2019.md#redeploy-from-azure) o [implementar usando la plantilla ARM subyacente de GitHub](create-host-pools-arm-template.md#run-the-azure-resource-manager-template-for-provisioning-a-new-host-pool).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general sobre cómo solucionar problemas de Windows Virtual Desktop y las pistas de escalación, consulte [Introducción, comentarios y soporte técnico para solucionar problemas](troubleshoot-set-up-overview-2019.md).
- Para solucionar problemas al configurar una máquina virtual (VM) en Windows Virtual Desktop, consulte [Configuración de la máquina virtual del host de sesión](troubleshoot-vm-configuration-2019.md).
- Para solucionar problemas con conexiones de cliente de Windows Virtual Desktop, consulte [Conexiones de servicios de Windows Virtual Desktop](troubleshoot-service-connection-2019.md).
- Para solucionar problemas con los clientes de Escritorio remoto, consulte [Solución de problemas del cliente de Escritorio remoto](../troubleshoot-client.md).
- Para solucionar problemas al usar PowerShell con Windows Virtual Desktop, consulte [PowerShell para Windows Virtual Desktop](troubleshoot-powershell-2019.md).
- Para más información sobre el servicio, consulte [Entorno de Windows Virtual Desktop](environment-setup-2019.md).
- Para realizar un tutorial de solución de problemas, consulte [Tutorial: Solución de problemas de las implementaciones de plantillas de Resource Manager](../../azure-resource-manager/templates/template-tutorial-troubleshoot.md).
- Para más información sobre las acciones de auditoría, consulte [Operaciones de auditoría con Resource Manager](../../azure-resource-manager/management/view-activity-logs.md).
- Si desea conocer más detalles sobre las acciones que permiten determinar los errores durante la implementación, consulte [Visualización de operaciones de implementación con el Portal de Azure](../../azure-resource-manager/templates/deployment-history.md).
