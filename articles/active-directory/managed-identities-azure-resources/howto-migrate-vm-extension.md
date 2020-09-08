---
title: 'Dejar de usar la extensión de máquina virtual de identidad administrada: Azure AD'
description: Instrucciones paso a paso para dejar de usar la extensión de máquina virtual y empezar a usar Azure Instance Metadata Service (IMDS) para la autenticación.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/25/2018
ms.author: barclayn
ms.openlocfilehash: 5b298767f9814f76dd606bab29bd0b245dad6937
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89260193"
---
# <a name="how-to-stop-using-the-virtual-machine-managed-identities-extension-and-start-using-the-azure-instance-metadata-service"></a>Dejar de usar la extensión de identidades administradas de máquina virtual y empezar a usar Azure Instance Metadata Service

## <a name="virtual-machine-extension-for-managed-identities"></a>Extensión de máquina virtual para identidades administradas

La extensión de máquina virtual para identidades administradas se usa para solicitar tokens relativos a una identidad administrada dentro de la máquina virtual. El flujo de trabajo consta de los pasos siguientes:

1. En primer lugar, la carga de trabajo en el recurso llama al punto de conexión local `http://localhost/oauth2/token` para solicitar un token de acceso.
2. Tras ello, la extensión de máquina virtual usa las credenciales de la identidad administrada para solicitar un token de acceso de Azure AD. 
3. El token de acceso se devuelve al llamador, y se puede usar para autenticarse en los servicios que admiten la autenticación de Azure AD, como Azure Key Vault o Azure Storage.

Debido a las diversas limitaciones de la extensión de máquina virtual de identidad administrada (descritas en la siguiente sección), dicha extensión está en desuso, en favor del uso del punto de conexión equivalente de Azure Instance Metadata Service (IMDS).

### <a name="provision-the-extension"></a>Aprovisionamiento de la extensión 

Al configurar una máquina virtual, o un conjunto de escalado de máquinas virtuales para tener una identidad administrada, se puede optar por aprovisionar las identidades administradas de la extensión de máquina virtual de recursos de Azure mediante el parámetro `-Type` del cmdlet [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension). Puede pasar `ManagedIdentityExtensionForWindows` o `ManagedIdentityExtensionForLinux`, según el tipo de máquina virtual, y asignar un nombre mediante el parámetro `-Name`. El parámetro `-Settings` especifica el puerto utilizado por el punto de conexión del token de OAuth para la adquisición de tokens:

```powershell
   $settings = @{ "port" = 50342 }
   Set-AzVMExtension -ResourceGroupName myResourceGroup -Location WestUS -VMName myVM -Name "ManagedIdentityExtensionForWindows" -Type "ManagedIdentityExtensionForWindows" -Publisher "Microsoft.ManagedIdentity" -TypeHandlerVersion "1.0" -Settings $settings 
```

También puede usar la plantilla de implementación de Azure Resource Manager para aprovisionar la extensión de máquina virtual, agregando para ello el siguiente JSON a la sección `resources` de la plantilla (use `ManagedIdentityExtensionForLinux` con los elementos de nombre y de tipo de la versión de Linux).

```json
{
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "[concat(variables('vmName'),'/ManagedIdentityExtensionForWindows')]",
    "apiVersion": "2018-06-01",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.ManagedIdentity",
        "type": "ManagedIdentityExtensionForWindows",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "port": 50342
        }
    }
}
```
    
    
Si trabaja con conjuntos de escalado de máquinas virtuales, puede aprovisionar las identidades administradas también para la extensión de conjunto de escalado de máquinas virtuales de recursos de Azure, mediante el cmdlet [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension). Puede pasar `ManagedIdentityExtensionForWindows` o `ManagedIdentityExtensionForLinux`, según el tipo de conjunto de escalado de máquinas virtuales, y asignar un nombre mediante el parámetro `-Name`. El parámetro `-Settings` especifica el puerto utilizado por el punto de conexión del token de OAuth para la adquisición de tokens:

   ```powershell
   $setting = @{ "port" = 50342 }
   $vmss = Get-AzVmss
   Add-AzVmssExtension -VirtualMachineScaleSet $vmss -Name "ManagedIdentityExtensionForWindows" -Type "ManagedIdentityExtensionForWindows" -Publisher "Microsoft.ManagedIdentity" -TypeHandlerVersion "1.0" -Setting $settings 
   ```
Para aprovisionar la extensión de conjunto de escalado de máquinas virtuales con la plantilla de implementación de Azure Resource Manager, agregue el siguiente JSON a la sección `extensionpProfile` de la plantilla (use `ManagedIdentityExtensionForLinux` con los elementos de nombre y de tipo de la versión de Linux).

```json
"extensionProfile": {
    "extensions": [
        {
            "name": "ManagedIdentityWindowsExtension",
            "properties": {
                "publisher": "Microsoft.ManagedIdentity",
                "type": "ManagedIdentityExtensionForWindows",
                "typeHandlerVersion": "1.0",
                "autoUpgradeMinorVersion": true,
                "settings": {
                    "port": 50342
                },
                "protectedSettings": {}
            }
        }
```

Se puede producir un error en el aprovisionamiento de la extensión de máquina virtual debido a errores de búsqueda DNS. Si esto ocurre, reinicie la máquina virtual e inténtelo de nuevo. 

### <a name="remove-the-extension"></a>Eliminación de la extensión 
Para quitar la extensión, use los modificadores `-n ManagedIdentityExtensionForWindows` o `-n ManagedIdentityExtensionForLinux` (según el tipo de máquina virtual) con [az vm extension delete](/cli/azure/vm/) o con [az vmss extension delete](/cli/azure/vmss) si es un conjunto de escalado de máquinas virtuales, usando la CLI de Azure o `Remove-AzVMExtension` de Powershell:

```azurecli-interactive
az vm identity --resource-group myResourceGroup --vm-name myVm -n ManagedIdentityExtensionForWindows
```

```azurecli-interactive
az vmss extension delete -n ManagedIdentityExtensionForWindows -g myResourceGroup -vmss-name myVMSS
```

```powershell
Remove-AzVMExtension -ResourceGroupName myResourceGroup -Name "ManagedIdentityExtensionForWindows" -VMName myVM
```

### <a name="acquire-a-token-using-the-virtual-machine-extension"></a>Adquirir un token mediante la extensión de máquina virtual

Esta es una solicitud de ejemplo en la que se usan las identidades administradas del punto de conexión de extensión de máquina virtual de recursos de Azure:

```
GET http://localhost:50342/oauth2/token?resource=https%3A%2F%2Fmanagement.azure.com%2F HTTP/1.1
Metadata: true
```

| Elemento | Descripción |
| ------- | ----------- |
| `GET` | El verbo HTTP, indicando que se van a recuperar datos desde el punto de conexión. En este caso, el token de acceso de OAuth. | 
| `http://localhost:50342/oauth2/token` | Punto de conexión de identidades administradas de recursos de Azure, donde 50342 es el puerto predeterminado y configurable. |
| `resource` | Un parámetro de cadena de consulta, que indica el URI del identificador de aplicación del recurso de destino. También aparece en la notificación `aud` (audiencia) del token emitido. En este ejemplo, se solicita un token para acceder a Azure Resource Manager, que tiene un URI de identificador de aplicación de `https://management.azure.com/`. |
| `Metadata` | Un campo de encabezado de la solicitud HTTP, requerido por las identidades administradas de recursos de Azure como mitigación frente a ataques de falsificación de solicitud de lado del servidor (SSRF). Este valor debe establecerse en "true", todo en minúsculas.|
| `object_id` | (Opcional) Un parámetro de cadena de consulta, que indica el valor de object_id de la identidad administrada para la que quiere que sea el token. Es obligatorio si la máquina virtual tiene varias identidades administradas asignadas por el usuario.|
| `client_id` | (Opcional) Un parámetro de cadena de consulta, que indica el valor de client_id de la identidad administrada para la que quiere que sea el token. Es obligatorio si la máquina virtual tiene varias identidades administradas asignadas por el usuario.|


Respuesta de ejemplo:

```
HTTP/1.1 200 OK
Content-Type: application/json
{
  "access_token": "eyJ0eXAi...",
  "refresh_token": "",
  "expires_in": "3599",
  "expires_on": "1506484173",
  "not_before": "1506480273",
  "resource": "https://management.azure.com/",
  "token_type": "Bearer"
}
```

| Elemento | Descripción |
| ------- | ----------- |
| `access_token` | El token de acceso solicitado. Al llamar a una API de REST protegida, el token se inserta en el campo `Authorization` del encabezado de la solicitud como un token de "portador", lo que permite a la API autenticar el llamador. | 
| `refresh_token` | Las identidades administradas de recursos de Azure no lo usan. |
| `expires_in` | El número de segundos que el token de acceso sigue siendo válido, antes de expirar, desde la hora de emisión. La hora de emisión puede encontrarse en la notificación `iat` del token. |
| `expires_on` | La hora a la que expira el token de acceso. La fecha se representa como el número de segundos desde "1970-01-01T0:0:0Z UTC" (se corresponde con la notificación `exp` del token). |
| `not_before` | El intervalo de tiempo cuando el token de acceso tiene efecto y se puede aceptar. La fecha se representa como el número de segundos desde "1970-01-01T0:0:0Z UTC" (se corresponde con la notificación `nbf` del token). |
| `resource` | El recurso para el que se solicitó el token de acceso, que coincide con el parámetro de la cadena de consulta `resource` de la solicitud. |
| `token_type` | El tipo de token, que es un token de acceso al "Portador", lo que significa que el recurso puede permitir el acceso al portador del token. |


### <a name="troubleshoot-the-virtual-machine-extension"></a>Solución de problemas de la extensión de máquina virtual 

#### <a name="restart-the-virtual-machine-extension-after-a-failure"></a>Reiniciar la extensión de máquina virtual tras un error

Tanto en Windows como en algunas versiones de Linux, si la extensión se detiene, se puede utilizar el siguiente cmdlet para reiniciarla manualmente:

```powershell
Set-AzVMExtension -Name <extension name>  -Type <extension Type>  -Location <location> -Publisher Microsoft.ManagedIdentity -VMName <vm name> -ResourceGroupName <resource group name> -ForceRerun <Any string different from any last value used>
```

Donde: 
- El tipo y el nombre de la extensión de Windows es: `ManagedIdentityExtensionForWindows`
- El tipo y el nombre de la extensión de Linux es: `ManagedIdentityExtensionForLinux`

#### <a name="automation-script-fails-when-attempting-schema-export-for-managed-identities-for-azure-resources-extension"></a>Se produce un error en el "script de automatización" al intentar exportar los esquemas de la extensión de identidades administradas para recursos de Azure.

Cuando se habilitan las identidades administradas de recursos de Azure en una máquina virtual, se muestra el siguiente error al intentar usar la característica "Script de automatización" de la máquina virtual o su grupo de recursos:

![Error de exportación del script de automatización de identidades administradas para recursos de Azure](./media/howto-migrate-vm-extension/automation-script-export-error.png)

Actualmente, las identidades administradas de la extensión de máquina virtual de recursos de Azure no admiten la posibilidad de exportar su esquema a una plantilla de grupo de recursos. Como resultado, en la plantilla generada no se muestran los parámetros de configuración para habilitar las identidades administradas para recursos de Azure en el recurso. Estas secciones se pueden agregar de forma manual siguiendo los ejemplos de [Configuración de identidades administradas de recursos de Azure en una máquina virtual de Azure mediante una plantilla](qs-configure-template-windows-vm.md).

Cuando la funcionalidad de exportación de esquema esté disponible para las identidades administradas de la extensión de máquina virtual de recursos de Azure (que dejará de utilizarse en enero de 2019), aparecerá reflejada en [Exportación de grupos de recursos que contienen extensiones de máquina virtual](../../virtual-machines/extensions/export-templates.md#supported-virtual-machine-extensions).

## <a name="limitations-of-the-virtual-machine-extension"></a>Limitaciones de la extensión de máquina virtual 

Existen varias limitaciones de primer orden en cuanto al uso de la extensión de máquina virtual. 

 * La limitación más seria es que las credenciales usadas para solicitar tokens se almacenan en la máquina virtual, con lo cual un atacante que cometa una infracción de seguridad en la máquina virtual podría filtrarlas. 
 * Además, la extensión de máquina virtual aún no es compatible con varias distribuciones Linux, lo que conlleva un enorme coste de desarrollo para modificar, compilar y probar la extensión en cada una de las distribuciones. Actualmente, solo se admiten las siguientes distribuciones Linux: 
    * CoreOS Stable
    * CentOS 7.1 
    * Red Hat 7.2 
    * Ubuntu 15.04 
    * Ubuntu 16.04
 * La implementación de máquinas virtuales con identidades administradas tiene impacto en el rendimiento, ya que la extensión de máquina virtual también tiene que estar aprovisionada. 
 * Por último, la extensión de máquina virtual solo tiene cabida para un máximo de 32 identidades administradas asignadas por usuarios en cada máquina virtual. 

## <a name="azure-instance-metadata-service"></a>Servicio de metadatos de instancia de Azure

[Azure Instance Metadata Service (IMDS)](../../virtual-machines/windows/instance-metadata-service.md) es un punto de conexión de REST que proporciona información sobre las instancias de máquina virtual en ejecución que se pueden usar para administrar y configurar las máquinas virtuales. El punto de conexión está disponible en una dirección IP no enrutable conocida (`169.254.169.254`) a la que solo se puede tener acceso desde dentro de la máquina virtual.

El uso de Azure IMDS para solicitar tokens reporta una serie de ventajas. 

1. El servicio es externo a la máquina virtual, por lo que las credenciales utilizadas por las identidades administradas ya no residen en la máquina virtual, sino que se hospedan y protegen en el equipo host de máquina virtual de Azure.   
2. Todos los sistemas operativos Windows y Linux compatibles en IaaS de Azure pueden usar identidades administradas.
3. La implementación es más rápida y más sencilla, puesto que ya no es necesario aprovisionar la extensión de máquina virtual.
4. Con el punto de conexión de IMDS, se pueden asignar hasta 1000 identidades administradas asignadas por usuario a una misma máquina virtual.
5. No hay ningún cambio significativo en cuanto a las solicitudes realizadas mediante IMDS frente a aquellas que usan la extensión de máquina virtual; por lo tanto, resulta bastante fácil adaptar las implementaciones existentes que actualmente utilizan la extensión de máquina virtual.

Por estos motivos, el servicio Azure IMDS será la forma de facto para solicitar tokens, una vez que la extensión de máquina virtual esté en desuso. 


## <a name="next-steps"></a>Pasos siguientes

* [Cómo usar identidades administradas de recursos de Azure en una máquina virtual de Azure para adquirir un token de acceso](how-to-use-vm-token.md)
* [Azure Instance Metadata Service](../../virtual-machines/windows/instance-metadata-service.md)