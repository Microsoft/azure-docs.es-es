---
title: 'Módulo de PowerShell para Windows Virtual Desktop: Azure'
description: Procedimientos de instalación y configuración del módulo de PowerShell para Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 04/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: c99aa1bdb246d242c4c1cbd33672367b66f923ea
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89229980"
---
# <a name="set-up-the-powershell-module-for-windows-virtual-desktop"></a>Configuración del módulo de PowerShell para Windows Virtual Desktop

>[!IMPORTANT]
>Este contenido se aplica a la integración de Windows Virtual Desktop con Azure Resource Manager.

El módulo de PowerShell para Windows Virtual Desktop está integrado en el módulo de Azure PowerShell. En este artículo se explica cómo configurar el módulo de PowerShell con el fin de ejecutar cmdlets para Windows Virtual Desktop.

## <a name="set-up-your-powershell-environment"></a>Configuración del entorno de PowerShell

Para empezar a usar el módulo, instale primero la [versión más reciente de PowerShell Core](/powershell/scripting/install/installing-powershell#powershell-core). Los cmdlets de Windows Virtual Desktop solo funcionan actualmente con PowerShell Core.

A continuación, deberá instalar el módulo DesktopVirtualization para usarlo en su sesión de PowerShell.

Ejecute el siguiente cmdlet de PowerShell con permisos elevados para instalar el módulo:

```powershell
Install-Module -Name Az.DesktopVirtualization
```

>[!NOTE]
> Si este cmdlet no funciona, intente ejecutarlo de nuevo con permisos elevados.

Use el siguiente cmdlet de PowerShell para conectarse a Azure:

```powershell
Connect-AzAccount
```

>[IMPORTANTE] Si se va a conectar al portal de US Gov, ejecute este cmdlet en su lugar:
> 
> ```powershell
> Connect-AzAccount -EnvironmentName AzureUSGovernment
> ```

Si va a iniciar sesión en su cuenta de Azure, necesitará un código que se genera al ejecutar el cmdlet Connect. Para iniciar sesión, vaya a <https://microsoft.com/devicelogin>, escriba el código y, a continuación, utilice sus credenciales de administrador de Azure.

```powershell
Account SubscriptionName TenantId Environment

------- ---------------- -------- -----------

Youradminupn subscriptionname AzureADTenantID AzureCloud
```

De este modo, iniciará sesión directamente en la suscripción predeterminada de sus credenciales de administrador.

## <a name="change-the-default-subscription"></a>Cambio de la suscripción predeterminada

Si desea cambiar la suscripción predeterminada después de iniciar sesión, ejecute el siguiente cmdlet:

```powershell
Select-AzSubscription -Subscription <preferredsubscriptionname>
```

También puede seleccionar una de una lista mediante el cmdlet Out-GridView:

```powershell
Get-AzSubscription | Out-GridView -PassThru | Select-AzSubscription
```

Si selecciona una nueva suscripción, no es necesario que especifique el identificador de la suscripción en los cmdlets que ejecute posteriormente. Por ejemplo, el siguiente cmdlet recupera un host de sesión concreto, sin necesidad de especificar el identificador de suscripción:

```powershell
Get-AzWvdSessionHost -HostPoolName <hostpoolname> -Name <sessionhostname> -ResourceGroupName <resourcegroupname>
```

También puede cambiar las suscripciones para cada cmdlet. Para ello, agregue el nombre de suscripción que desee como parámetro. El siguiente cmdlet es el mismo que en el ejemplo anterior, excepto que se ha agregado el identificador de suscripción como parámetro para cambiar la suscripción que el cmdlet utiliza.

```powershell
Get-AzWvdSessionHost -HostPoolName <hostpoolname> -Name <sessionhostname> -ResourceGroupName <resourcegroupname> -SubscriptionId <subscriptionGUID>
```

## <a name="get-locations"></a>Obtención de ubicaciones

El parámetro de ubicación es obligatorio para todos los cmdlets **New-AzWVD** que crean nuevos** objetos.

Ejecute el siguiente cmdlet para obtener una lista de las ubicaciones admitidas para su suscripción:

```powershell
Get-AzLocation
```

El resultado de **Get-AzLocation** será similar al siguiente:

```powershell
Location : eastasia

DisplayName : East Asia

Providers : {Microsoft.RecoveryServices, Microsoft.ManagedIdentity,
Microsoft.SqlVirtualMachine, microsoft.insightsΓÇª}

Location : southeastasia

DisplayName : Southeast Asia

Providers : {Microsoft.RecoveryServices, Microsoft.ManagedIdentity,
Microsoft.SqlVirtualMachine, microsoft.insightsΓÇª}

Location : centralus

DisplayName : Central US

Providers : {Microsoft.RecoveryServices, Microsoft.DesktopVirtualization,
Microsoft.ManagedIdentity, Microsoft.SqlVirtualMachineΓÇª}

Location : eastus

DisplayName : East US

Providers : {Microsoft.RecoveryServices, Microsoft.DesktopVirtualization,
Microsoft.ManagedIdentity, Microsoft.SqlVirtualMachineΓÇª}
```

Una vez que conozca la ubicación de su cuenta, podrá usarla en un cmdlet. Por ejemplo, a continuación se incluye un cmdlet que crea un grupo de hosts en la ubicación "southeastasia":

```powershell
New-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -WorkspaceName <workspacename> -Location “southeastasia”
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha configurado el módulo de PowerShell, puede ejecutar cmdlets para realizar todo tipo de tareas en Windows Virtual Desktop. Estos son algunos de los lugares en los que puede usar su módulo:

- Realice nuestros [tutoriales de Windows Virtual Desktop]() para configurar su propio entorno de Windows Virtual Desktop.
- [Creación de un grupo host con PowerShell](create-host-pools-powershell.md)
- [Configuración del método de equilibrio de carga de Windows Virtual Desktop](configure-host-pool-load-balancing.md)
- [Configuración del tipo de asignación de grupo de host de escritorio personal](configure-host-pool-personal-desktop-assignment-type.md)
- Y mucho más.

Si tiene algún problema, consulte nuestro [artículo de solución de problemas de PowerShell](troubleshoot-powershell.md) para obtener ayuda.

