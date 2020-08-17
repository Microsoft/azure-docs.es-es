---
title: Habilitación de Azure AD Domain Services mediante PowerShell | Microsoft Docs
description: Aprenda a configurar y habilitar Azure Active Directory Domain Services con Azure AD PowerShell y Azure PowerShell.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.assetid: d4bc5583-6537-4cd9-bc4b-7712fdd9272a
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: sample
ms.date: 07/09/2020
ms.author: iainfou
ms.openlocfilehash: 9c2345c93a163464ea735400c9269e2e3fc27ecf
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87488183"
---
# <a name="enable-azure-active-directory-domain-services-using-powershell"></a>Habilitación de Azure Active Directory Domain Services mediante PowerShell

Azure Active Directory Domain Services (Azure AD DS) proporciona servicios de dominio administrados como, por ejemplo, unión a un dominio, directiva de grupo, LDAP o autenticación Kerberos/NTLM, que son totalmente compatibles con Windows Server Active Directory. Estos servicios de dominio se usan sin necesidad de implementar, administrar ni aplicar revisiones a los controladores de dominio. Azure AD DS se integra con el inquilino de Azure AD existente. Esta integración permite a los usuarios iniciar sesión con sus credenciales corporativas y, además, le permite usar grupos y cuentas de usuario existentes para proteger el acceso a los recursos.

En este artículo se muestra cómo habilitar Azure AD DS con PowerShell.

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este artículo, necesita los siguientes recursos:

* Instale y configure Azure PowerShell.
    * Si es necesario, siga las instrucciones para [instalar el módulo de Azure PowerShell y conectarse a la suscripción de Azure](/powershell/azure/install-az-ps).
    * Asegúrese de que inicia sesión en su suscripción de Azure con el cmdlet [Connect-AzAccount][Connect-AzAccount].
* Instale y configure Azure AD PowerShell.
    * Si es necesario, siga las instrucciones para [instalar el módulo de Azure AD PowerShell y conectarse a Azure AD](/powershell/azure/active-directory/install-adv2).
    * Asegúrese de iniciar sesión en el inquilino de Azure AD mediante el cmdlet [Connect-AzureAD][Connect-AzureAD].
* Necesita privilegios de *administrador global* en el inquilino de Azure AD para habilitar Azure AD DS.
* Necesita privilegios de *colaborador* en la suscripción de Azure para crear los recursos de Azure AD DS necesarios.

## <a name="create-required-azure-ad-resources"></a>Creación de los recursos de Azure AD necesarios

Azure AD DS requiere una entidad de servicio y un grupo de Azure AD. Estos recursos permiten al dominio administrado de Azure AD DS sincronizar los datos y definir qué usuarios tienen permisos administrativos en él.

En primer lugar, cree una entidad de servicio de Azure AD para Azure AD DS para comunicarse y autenticarse. Se usa un identificador de aplicación específico denominado *Servicios de controlador de dominio* con un identificador de *2565bd9d-da50-47d4-8b85-4c97f669dc36*. No cambie este identificador de aplicación.

Cree una entidad de servicio de Azure AD con el cmdlet [New-AzureADServicePrincipal][New-AzureADServicePrincipal]:

```powershell
New-AzureADServicePrincipal -AppId "2565bd9d-da50-47d4-8b85-4c97f669dc36"
```

Ahora, cree un grupo de Azure AD denominado *Administradores de controlador de dominio de AAD*. A los usuarios agregados a este grupo se les conceden permisos para realizar tareas de administración en el dominio administrado.

Cree el grupo *Administradores de controlador de dominio de AAD* mediante el cmdlet [New-AzureADGroup][New-AzureADGroup]:

```powershell
New-AzureADGroup -DisplayName "AAD DC Administrators" `
  -Description "Delegated group to administer Azure AD Domain Services" `
  -SecurityEnabled $true -MailEnabled $false `
  -MailNickName "AADDCAdministrators"
```

Con este *grupo* creado, agregue un usuario al grupo con el cmdlet [Add-AzureADGroupMember][Add-AzureADGroupMember]. Primero obtendrá el identificador de objeto del grupo *Administradores de controlador de dominio de AAD* con el cmdlet [Get-AzureADGroup][Get-AzureADGroup] y, luego, el identificador de objeto del usuario deseado con el cmdlet [Get-AzureADUser][Get-AzureADUser].

En el ejemplo siguiente, el identificador de objeto de usuario de la cuenta con un UPN de `admin@contoso.onmicrosoft.com`. Reemplace esta cuenta de usuario por el UPN del usuario que quiere agregar al grupo *Administradores de controlador de dominio de AAD*:

```powershell
# First, retrieve the object ID of the newly created 'AAD DC Administrators' group.
$GroupObjectId = Get-AzureADGroup `
  -Filter "DisplayName eq 'AAD DC Administrators'" | `
  Select-Object ObjectId

# Now, retrieve the object ID of the user you'd like to add to the group.
$UserObjectId = Get-AzureADUser `
  -Filter "UserPrincipalName eq 'admin@contoso.onmicrosoft.com'" | `
  Select-Object ObjectId

# Add the user to the 'AAD DC Administrators' group.
Add-AzureADGroupMember -ObjectId $GroupObjectId.ObjectId -RefObjectId $UserObjectId.ObjectId
```

## <a name="create-supporting-azure-resources"></a>Creación de recursos de Azure de apoyo

En primer lugar, registre el proveedor de recursos de Azure AD Domain Services mediante el cmdlet [Register-AzResourceProvider][Register-AzResourceProvider]:

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.AAD
```

A continuación, cree un grupo de recursos con el cmdlet [New-AzResourceGroup][New-AzResourceGroup]. En el ejemplo siguiente, el grupo de recursos se denomina *myResourceGroup* y se crea en la región *westus*. Use su propio nombre y la región deseada:

```powershell
$ResourceGroupName = "myResourceGroup"
$AzureLocation = "westus"

# Create the resource group.
New-AzResourceGroup `
  -Name $ResourceGroupName `
  -Location $AzureLocation
```

Cree la red virtual y las subredes para Azure AD Domain Services. Se crean dos subredes: una para *DomainServices* y otra para *Cargas de trabajo*. Azure AD DS se implementa en la subred dedicada *DomainServices*. No implemente otras aplicaciones o cargas de trabajo en esta subred. Use la subred *Cargas de trabajo* u otras subredes con el resto de las máquinas virtuales.

Cree las subredes mediante el cmdlet [New-AzVirtualNetworkSubnetConfig][New-AzVirtualNetworkSubnetConfig] y, luego, cree la red virtual mediante el cmdlet [New-AzVirtualNetwork][New-AzVirtualNetwork].

```powershell
$VnetName = "myVnet"

# Create the dedicated subnet for AAD Domain Services.
$AaddsSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name DomainServices `
  -AddressPrefix 10.0.0.0/24

$WorkloadSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name Workloads `
  -AddressPrefix 10.0.1.0/24

# Create the virtual network in which you will enable Azure AD Domain Services.
$Vnet= New-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location westus `
  -Name $VnetName `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $AaddsSubnet,$WorkloadSubnet
```

## <a name="create-a-managed-domain"></a>Creación de un dominio administrado

Ahora vamos a crear un dominio administrado. Establezca el identificador de la suscripción de Azure y luego proporcione un nombre para el dominio administrado, como *aaddscontoso.com*. Puede obtener el identificador de la suscripción con el cmdlet [Get-AzSubscription][Get-AzSubscription].

Si elige una región que admite zonas de disponibilidad, los recursos de Azure AD DS se distribuyen entre las zonas para conseguir redundancia adicional.

Las zonas de disponibilidad son ubicaciones físicas exclusivas dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Para garantizar la resistencia, hay un mínimo de tres zonas independientes en todas las regiones habilitadas.

No es necesario realizar ninguna configuración para que Azure AD DS se distribuya entre zonas. La plataforma Azure controla automáticamente la distribución en zonas de los recursos. Para más información y ver la disponibilidad regional, consulte [¿Qué son las zonas de disponibilidad en Azure?][availability-zones]

```powershell
$AzureSubscriptionId = "YOUR_AZURE_SUBSCRIPTION_ID"
$ManagedDomainName = "aaddscontoso.com"

# Enable Azure AD Domain Services for the directory.
New-AzResource -ResourceId "/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.AAD/DomainServices/$ManagedDomainName" `
  -Location $AzureLocation `
  -Properties @{"DomainName"=$ManagedDomainName; `
    "SubnetId"="/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.Network/virtualNetworks/$VnetName/subnets/DomainServices"} `
  -Force -Verbose
```

Tarda unos minutos hasta que se crea el recurso y se devuelve el control al símbolo del sistema de PowerShell. El dominio administrado se sigue aprovisionando en segundo plano y puede tardar hasta una hora en finalizar la implementación. En Azure Portal, la página **Información general** del dominio administrado muestra el estado actual durante esta fase de implementación.

Cuando en Azure Portal se muestra que el dominio administrado ha terminado de aprovisionarse, es necesario realizar las siguientes tareas:

* Actualice la configuración de DNS para la red virtual de manera que las máquinas virtuales puedan encontrar el dominio administrado para la unión o autenticación de dominios.
    * Para configurar DNS, seleccione el dominio administrado en el portal. En la ventana **Información general**, se le pedirá que configure automáticamente estos valores de DNS.
* Cree un grupo de seguridad de red para restringir el tráfico de la red virtual en el dominio administrado. Se crea un equilibrador de carga estándar de Azure que exige la presencia de estas reglas. Este grupo de seguridad de red protege Azure AD DS y es necesario para que el dominio administrado funcione correctamente.
    * Para crear el grupo de seguridad de red y las reglas necesarias, instale primero el script `New-AzureAddsNetworkSecurityGroup` mediante el comando `Install-Script -Name New-AaddsNetworkSecurityGroup` y, después, ejecute `New-AaddsNetworkSecurityGroup`. Las reglas necesarias para el dominio administrado se crean automáticamente.
* [Habilite la sincronización de contraseñas en Azure AD DS](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds) de manera que los usuarios finales puedan iniciar sesión en el dominio administrado mediante sus credenciales corporativas.

## <a name="complete-powershell-script"></a>Script completo de PowerShell

El siguiente script completo de PowerShell combina todas las tareas que se muestran en este artículo. Copie el script y guárdelo en un archivo con la extensión `.ps1`. Ejecute el script en una consola local de PowerShell o en [Azure Cloud Shell][cloud-shell].

> [!NOTE]
> Para habilitar Azure AD DS, debe ser administrador global del inquilino de Azure AD. También necesita al menos privilegios de *colaborador* en la suscripción de Azure.

```powershell
# Change the following values to match your deployment.
$AaddsAdminUserUpn = "admin@contoso.onmicrosoft.com"
$ResourceGroupName = "myResourceGroup"
$VnetName = "myVnet"
$AzureLocation = "westus"
$AzureSubscriptionId = "YOUR_AZURE_SUBSCRIPTION_ID"
$ManagedDomainName = "aaddscontoso.com"

# Connect to your Azure AD directory.
Connect-AzureAD

# Login to your Azure subscription.
Connect-AzAccount

# Create the service principal for Azure AD Domain Services.
New-AzureADServicePrincipal -AppId "2565bd9d-da50-47d4-8b85-4c97f669dc36"

# Create the delegated administration group for AAD Domain Services.
New-AzureADGroup -DisplayName "AAD DC Administrators" `
  -Description "Delegated group to administer Azure AD Domain Services" `
  -SecurityEnabled $true -MailEnabled $false `
  -MailNickName "AADDCAdministrators"

# First, retrieve the object ID of the newly created 'AAD DC Administrators' group.
$GroupObjectId = Get-AzureADGroup `
  -Filter "DisplayName eq 'AAD DC Administrators'" | `
  Select-Object ObjectId

# Now, retrieve the object ID of the user you'd like to add to the group.
$UserObjectId = Get-AzureADUser `
  -Filter "UserPrincipalName eq '$AaddsAdminUserUpn'" | `
  Select-Object ObjectId

# Add the user to the 'AAD DC Administrators' group.
Add-AzureADGroupMember -ObjectId $GroupObjectId.ObjectId -RefObjectId $UserObjectId.ObjectId

# Register the resource provider for Azure AD Domain Services with Resource Manager.
Register-AzResourceProvider -ProviderNamespace Microsoft.AAD

# Create the resource group.
New-AzResourceGroup `
  -Name $ResourceGroupName `
  -Location $AzureLocation

# Create the dedicated subnet for AAD Domain Services.
$AaddsSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name DomainServices `
  -AddressPrefix 10.0.0.0/24

$WorkloadSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name Workloads `
  -AddressPrefix 10.0.1.0/24

# Create the virtual network in which you will enable Azure AD Domain Services.
$Vnet=New-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location $AzureLocation `
  -Name $VnetName `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $AaddsSubnet,$WorkloadSubnet

# Enable Azure AD Domain Services for the directory.
New-AzResource -ResourceId "/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.AAD/DomainServices/$ManagedDomainName" `
  -Location $AzureLocation `
  -Properties @{"DomainName"=$ManagedDomainName; `
    "SubnetId"="/subscriptions/$AzureSubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.Network/virtualNetworks/$VnetName/subnets/DomainServices"} `
  -Force -Verbose
```

Tarda unos minutos hasta que se crea el recurso y se devuelve el control al símbolo del sistema de PowerShell. El dominio administrado se sigue aprovisionando en segundo plano y puede tardar hasta una hora en finalizar la implementación. En Azure Portal, la página **Información general** del dominio administrado muestra el estado actual durante esta fase de implementación.

Cuando en Azure Portal se muestra que el dominio administrado ha terminado de aprovisionarse, es necesario realizar las siguientes tareas:

* Actualice la configuración de DNS para la red virtual de manera que las máquinas virtuales puedan encontrar el dominio administrado para la unión o autenticación de dominios.
    * Para configurar DNS, seleccione el dominio administrado en el portal. En la ventana **Información general**, se le pedirá que configure automáticamente estos valores de DNS.
* Cree un grupo de seguridad de red para restringir el tráfico de la red virtual en el dominio administrado. Se crea un equilibrador de carga estándar de Azure que exige la presencia de estas reglas. Este grupo de seguridad de red protege Azure AD DS y es necesario para que el dominio administrado funcione correctamente.
    * Para crear el grupo de seguridad de red y las reglas necesarias, instale primero el script `New-AzureAddsNetworkSecurityGroup` mediante el comando `Install-Script -Name New-AaddsNetworkSecurityGroup` y, después, ejecute `New-AaddsNetworkSecurityGroup`. Las reglas necesarias para el dominio administrado se crean automáticamente.
* [Habilite la sincronización de contraseñas en Azure AD DS](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds) de manera que los usuarios finales puedan iniciar sesión en el dominio administrado mediante sus credenciales corporativas.

## <a name="next-steps"></a>Pasos siguientes

Para ver el dominio administrado en acción, puede [unir a un dominio una máquina virtual Windows][windows-join], [configurar LDAP seguro][tutorial-ldaps] y [configurar la sincronización de hash de contraseñas][tutorial-phs].

<!-- INTERNAL LINKS -->
[windows-join]: join-windows-vm.md
[tutorial-ldaps]: tutorial-configure-ldaps.md
[tutorial-phs]: tutorial-configure-password-hash-sync.md

<!-- EXTERNAL LINKS -->
[Connect-AzAccount]: /powershell/module/Az.Accounts/Connect-AzAccount
[Connect-AzureAD]: /powershell/module/AzureAD/Connect-AzureAD
[New-AzureADServicePrincipal]: /powershell/module/AzureAD/New-AzureADServicePrincipal
[New-AzureADGroup]: /powershell/module/AzureAD/New-AzureADGroup
[Add-AzureADGroupMember]: /powershell/module/AzureAD/Add-AzureADGroupMember
[Get-AzureADGroup]: /powershell/module/AzureAD/Get-AzureADGroup
[Get-AzureADUser]: /powershell/module/AzureAD/Get-AzureADUser
[Register-AzResourceProvider]: /powershell/module/Az.Resources/Register-AzResourceProvider
[New-AzResourceGroup]: /powershell/module/Az.Resources/New-AzResourceGroup
[New-AzVirtualNetworkSubnetConfig]: /powershell/module/Az.Network/New-AzVirtualNetworkSubnetConfig
[New-AzVirtualNetwork]: /powershell/module/Az.Network/New-AzVirtualNetwork
[Get-AzSubscription]: /powershell/module/Az.Accounts/Get-AzSubscription
[cloud-shell]: /azure/cloud-shell/cloud-shell-windows-users
[availability-zones]: ../availability-zones/az-overview.md
