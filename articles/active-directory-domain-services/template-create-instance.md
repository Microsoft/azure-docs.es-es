---
title: Habilitación de Azure AD Domain Services mediante una plantilla | Microsoft Docs
description: Aprenda a configurar y habilitar Azure Active Directory Domain Services con una plantilla de Azure Resource Manager.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: sample
ms.date: 07/09/2020
ms.author: iainfou
ms.openlocfilehash: 880ccf9a69d5898da98aeabcfd89d05ff94e3b43
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87489816"
---
# <a name="create-an-azure-active-directory-domain-services-managed-domain-using-an-azure-resource-manager-template"></a>Creación de un dominio administrado de Azure Active Directory Domain Services mediante una plantilla de Resource Manager

Azure Active Directory Domain Services (Azure AD DS) proporciona servicios de dominio administrados como, por ejemplo, unión a un dominio, directiva de grupo, LDAP o autenticación Kerberos/NTLM, que son totalmente compatibles con Windows Server Active Directory. Estos servicios de dominio se usan sin necesidad de implementar, administrar ni aplicar revisiones a los controladores de dominio. Azure AD DS se integra con el inquilino de Azure AD existente. Esta integración permite a los usuarios iniciar sesión con sus credenciales corporativas y, además, le permite usar grupos y cuentas de usuario existentes para proteger el acceso a los recursos.

En este artículo se muestra cómo crear un dominio administrado mediante una plantilla de Azure Resource Manager. Los recursos auxiliares se crean mediante Azure PowerShell.

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

## <a name="dns-naming-requirements"></a>Requisitos de nomenclatura de DNS

Cuando se crea un dominio administrado de Azure AD DS, se especifica un nombre DNS. Hay algunas consideraciones que se deben tener en cuenta al elegir este nombre DNS:

* **Nombre de dominio integrado:** de forma predeterminada, se usa el nombre de dominio integrado del directorio (un sufijo *.onmicrosoft.com*). Si se quiere habilitar el acceso LDAP seguro al dominio administrado a través de Internet, no se puede crear un certificado digital para proteger la conexión con este dominio predeterminado. Microsoft es el propietario del dominio *.onmicrosoft.com*, por lo que una entidad de certificación no emitirá un certificado.
* **Nombres de dominio personalizados:** el enfoque más común consiste en especificar un nombre de dominio personalizado, normalmente uno que ya se posee y es enrutable. Cuando se usa un dominio personalizado enrutable, el tráfico puede fluir correctamente según sea necesario para admitir las aplicaciones.
* **Sufijos de dominio no enrutables:** por lo general, se recomienda evitar un sufijo de nombre de dominio no enrutable, como *contoso.local*. El sufijo *.local* no es enrutable y puede provocar problemas con la resolución de DNS.

> [!TIP]
> Si crea un nombre de dominio personalizado, tenga cuidado con los espacios de nombres DNS existentes. Se recomienda usar un nombre de dominio independiente de cualquier espacio de nombres de DNS local o de Azure existentes.
>
> Por ejemplo, si tiene un espacio de nombres de DNS existente para *contoso.com*, cree un dominio administrado con el nombre de dominio personalizado *aaddscontoso.com*. Si necesita usar LDAP seguro, debe registrar y poseer este nombre de dominio personalizado para generar los certificados necesarios.
>
> Es posible que tenga que crear algunos registros DNS adicionales para otros servicios del entorno o reenviadores DNS condicionales entre los espacios de nombres de DNS existentes en el entorno. Por ejemplo, si ejecuta un servidor web que hospeda un sitio mediante el nombre DNS raíz, puede haber conflictos de nomenclatura que requieran más entradas DNS.
>
> En este ejemplo y artículos de procedimientos, el dominio personalizado *aaddscontoso.com* se usa como ejemplo breve. En todos los comandos, especifique su propio nombre de dominio.

También se aplican las siguientes restricciones de nombre DNS:

* **Restricciones de prefijo de dominio:** no se puede crear un dominio administrado con un prefijo de más de 15 caracteres. El prefijo del nombre de dominio especificado (por ejemplo, *aaddscontoso* en el nombre de dominio *aaddscontoso.com*) debe contener 15 caracteres o menos.
* **Conflictos de nombres de red:** el nombre de dominio DNS del dominio administrado no puede existir ya en la red virtual. Busque, en concreto, los siguientes escenarios que provocarían un conflicto de nombres:
    * Ya tiene un dominio de Active Directory con el mismo nombre de dominio DNS en la red virtual de Azure.
    * La red virtual en la que planea habilitar el dominio administrado tiene una conexión VPN con la red local. En este escenario, asegúrese de que no tiene un dominio con el mismo nombre de dominio DNS de la red local.
    * Ya dispone de un servicio en la nube de Azure con ese nombre en la red virtual de Azure.

## <a name="create-required-azure-ad-resources"></a>Creación de los recursos de Azure AD necesarios

Azure AD DS requiere una entidad de servicio y un grupo de Azure AD. Estos recursos permiten al dominio administrado sincronizar los datos y definir qué usuarios tienen permisos administrativos en él.

En primer lugar, registre el proveedor de recursos de Azure AD Domain Services mediante el cmdlet [Register-AzResourceProvider][Register-AzResourceProvider]:

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.AAD
```

Cree una entidad de servicio de Azure AD con el cmdlet [New-AzureADServicePrincipal][New-AzureADServicePrincipal] para Azure AD DS para comunicarse y autenticarse. Se usa un identificador de aplicación específico denominado *Servicios de controlador de dominio* con un identificador de *2565bd9d-da50-47d4-8b85-4c97f669dc36*. No cambie este identificador de aplicación.

```powershell
New-AzureADServicePrincipal -AppId "2565bd9d-da50-47d4-8b85-4c97f669dc36"
```

Ahora, cree un grupo de Azure AD denominado *Administradores de controlador de dominio de AAD* con el cmdlet [New-AzureADGroup][New-AzureADGroup]. A los usuarios agregados a este grupo se les conceden permisos para realizar tareas de administración en el dominio administrado.

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

Por último, cree un grupo de recursos con el cmdlet [New-AzResourceGroup][New-AzResourceGroup]. En el ejemplo siguiente, el grupo de recursos se denomina *myResourceGroup* y se crea en la región *westus*. Use su propio nombre y la región deseada:

```powershell
New-AzResourceGroup `
  -Name "myResourceGroup" `
  -Location "WestUS"
```

Si elige una región que admite zonas de disponibilidad, los recursos de Azure AD DS se distribuyen entre las zonas para conseguir redundancia adicional. Las zonas de disponibilidad son ubicaciones físicas exclusivas dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Para garantizar la resistencia, hay un mínimo de tres zonas independientes en todas las regiones habilitadas.

No es necesario realizar ninguna configuración para que Azure AD DS se distribuya entre zonas. La plataforma Azure controla automáticamente la distribución en zonas de los recursos. Para más información y ver la disponibilidad regional, consulte [¿Qué son las zonas de disponibilidad en Azure?][availability-zones]

## <a name="resource-definition-for-azure-ad-ds"></a>Definición de recursos para Azure AD DS

Como parte de la definición de recursos de Resource Manager, son necesarios los siguientes parámetros de configuración:

| Parámetro               | Value |
|-------------------------|---------|
| domainName              | Nombre de dominio DNS para el dominio administrado, teniendo en cuenta los puntos anteriores sobre los conflictos y los prefijos de nomenclatura. |
| filteredSync            | Azure AD DS permite sincronizar *todos* los usuarios y grupos disponibles en Azure AD, o bien realizar una sincronización *con ámbito* solo de grupos específicos.<br /><br /> Para más información sobre la sincronización con ámbito, consulte [Sincronización con ámbito de Azure AD Domain Services][scoped-sync].|
| notificationSettings    | Si se generan alertas en el dominio administrado, se pueden enviar notificaciones por correo electrónico. <br /><br />Los *administradores globales* del inquilino de Azure y los miembros del grupo de *administradores de AAD DC* pueden estar *habilitados* para estas notificaciones.<br /><br /> Si lo prefiere, puede agregar destinatarios adicionales de las notificaciones cuando haya alertas que requieran atención.|
| domainConfigurationType | De forma predeterminada, un dominio administrado se crea como un bosque de *usuarios*. Este tipo de bosque sincroniza todos los objetos de Azure AD, incluidas las cuentas de usuario creadas en un entorno de AD DS local. No es necesario especificar un valor de *domainConfiguration* para crear un bosque de usuarios.<br /><br /> Un bosque de *Recursos* solo sincroniza los usuarios y grupos creados directamente en Azure AD. Establezca el valor en *ResourceTrusting* para crear un bosque de recursos.<br /><br />Para más información sobre los bosques de *Recursos*, incluido el motivo por el que puede usar uno y cómo crear confianzas de bosque con dominios de AD DS locales, consulte [Introducción a los bosques de recursos de Azure AD DS][resource-forests].|

La siguiente definición de parámetros comprimidos muestra cómo se declaran estos valores. Se crea un bosque de usuarios denominado *aaddscontoso.com* con todos los usuarios de Azure AD sincronizados con el dominio administrado:

```json
"parameters": {
    "domainName": {
        "value": "aaddscontoso.com"
    },
    "filteredSync": {
        "value": "Disabled"
    },
    "notificationSettings": {
        "value": {
            "notifyGlobalAdmins": "Enabled",
            "notifyDcAdmins": "Enabled",
            "additionalRecipients": []
        }
    },
    [...]
}
```

El siguiente tipo de recurso comprimido de plantilla de Resource Manager se utiliza para definir y crear el dominio administrado. Ya debe existir una subred y una red virtual de Azure, o bien crearse como parte de la plantilla de Resource Manager. El dominio administrado está conectado a esta subred.

```json
"resources": [
    {
        "apiVersion": "2017-06-01",
        "type": "Microsoft.AAD/DomainServices",
        "name": "[parameters('domainName')]",
        "location": "[parameters('location')]",
        "dependsOn": [
            "[concat('Microsoft.Network/virtualNetworks/', parameters('vnetName'))]"
        ],
        "properties": {
            "domainName": "[parameters('domainName')]",
            "subnetId": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/virtualNetworks/', parameters('vnetName'), '/subnets/', parameters('subnetName'))]",
            "filteredSync": "[parameters('filteredSync')]",
            "notificationSettings": "[parameters('notificationSettings')]"
        }
    },
    [...]
]
```

Estos parámetros y el tipo de recurso se pueden usar como parte de una plantilla de Resource Manager más ancha para implementar un dominio administrado, tal y como se muestra en la sección siguiente.

## <a name="create-a-managed-domain-using-sample-template"></a>Creación de un dominio administrado mediante una plantilla de ejemplo

La siguiente plantilla de ejemplo completa de Resource Manager crea un dominio administrado y las reglas de compatibilidad de red virtual, subred y grupo de seguridad de red. Las reglas del grupo de seguridad de red son necesarias para proteger el dominio administrado y asegurarse de que el tráfico puede fluir correctamente. Se crea un bosque de usuarios con el nombre DNS de *aaddscontoso.com*, con todos los usuarios sincronizados desde Azure AD:

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "apiVersion": {
            "value": "2017-06-01"
        },
        "domainConfigurationType": {
            "value": "FullySynced"
        },
        "domainName": {
            "value": "aaddscontoso.com"
        },
        "filteredSync": {
            "value": "Disabled"
        },
        "location": {
            "value": "westus"
        },
        "notificationSettings": {
            "value": {
                "notifyGlobalAdmins": "Enabled",
                "notifyDcAdmins": "Enabled",
                "additionalRecipients": []
            }
        },
        "subnetName": {
            "value": "aadds-subnet"
        },
        "vnetName": {
            "value": "aadds-vnet"
        },
        "vnetAddressPrefixes": {
            "value": [
                "10.1.0.0/24"
            ]
        },
        "subnetAddressPrefix": {
            "value": "10.1.0.0/24"
        },
        "nsgName": {
            "value": "aadds-nsg"
        }
    },
    "resources": [
        {
            "apiVersion": "2017-06-01",
            "type": "Microsoft.AAD/DomainServices",
            "name": "[parameters('domainName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Network/virtualNetworks/', parameters('vnetName'))]"
            ],
            "properties": {
                "domainName": "[parameters('domainName')]",
                "subnetId": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/virtualNetworks/', parameters('vnetName'), '/subnets/', parameters('subnetName'))]",
                "filteredSync": "[parameters('filteredSync')]",
                "domainConfigurationType": "[parameters('domainConfigurationType')]",
                "notificationSettings": "[parameters('notificationSettings')]"
            }
        },
        {
            "type": "Microsoft.Network/NetworkSecurityGroups",
            "name": "[parameters('nsgName')]",
            "location": "[parameters('location')]",
            "properties": {
                "securityRules": [
                    {
                        "name": "AllowSyncWithAzureAD",
                        "properties": {
                            "access": "Allow",
                            "priority": 101,
                            "direction": "Inbound",
                            "protocol": "Tcp",
                            "sourceAddressPrefix": "AzureActiveDirectoryDomainServices",
                            "sourcePortRange": "*",
                            "destinationAddressPrefix": "*",
                            "destinationPortRange": "443"
                        }
                    },
                    {
                        "name": "AllowPSRemoting",
                        "properties": {
                            "access": "Allow",
                            "priority": 301,
                            "direction": "Inbound",
                            "protocol": "Tcp",
                            "sourceAddressPrefix": "AzureActiveDirectoryDomainServices",
                            "sourcePortRange": "*",
                            "destinationAddressPrefix": "*",
                            "destinationPortRange": "5986"
                        }
                    },
                    {
                        "name": "AllowRD",
                        "properties": {
                            "access": "Allow",
                            "priority": 201,
                            "direction": "Inbound",
                            "protocol": "Tcp",
                            "sourceAddressPrefix": "CorpNetSaw",
                            "sourcePortRange": "*",
                            "destinationAddressPrefix": "*",
                            "destinationPortRange": "3389"
                        }
                    }
                ]
            },
            "apiVersion": "2018-04-01"
        },
        {
            "type": "Microsoft.Network/virtualNetworks",
            "name": "[parameters('vnetName')]",
            "location": "[parameters('location')]",
            "apiVersion": "2018-04-01",
            "dependsOn": [
                "[concat('Microsoft.Network/NetworkSecurityGroups/', parameters('nsgName'))]"
            ],
            "properties": {
                "addressSpace": {
                    "addressPrefixes": "[parameters('vnetAddressPrefixes')]"
                },
                "subnets": [
                    {
                        "name": "[parameters('subnetName')]",
                        "properties": {
                            "addressPrefix": "[parameters('subnetAddressPrefix')]",
                            "networkSecurityGroup": {
                                "id": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/NetworkSecurityGroups/', parameters('nsgName'))]"
                            }
                        }
                    }
                ]
            }
        }
    ],
    "outputs": {}
}
```

Esta plantilla se puede implementar con el método de implementación que prefiera, como [Azure Portal][portal-deploy], [Azure PowerShell][powershell-deploy] o una canalización de CI/CD. En el ejemplo siguiente se usa el cmdlet [New-AzResourceGroupDeployment][New-AzResourceGroupDeployment]. Especifique su propio nombre del grupo de recursos y nombre de archivo de la plantilla:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "myResourceGroup" -TemplateFile <path-to-template>
```

Tarda unos minutos hasta que se crea el recurso y se devuelve el control al símbolo del sistema de PowerShell. El dominio administrado se sigue aprovisionando en segundo plano y puede tardar hasta una hora en finalizar la implementación. En Azure Portal, la página **Información general** del dominio administrado muestra el estado actual durante esta fase de implementación.

Cuando en Azure Portal se muestra que el dominio administrado ha terminado de aprovisionarse, es necesario realizar las siguientes tareas:

* Actualice la configuración de DNS para la red virtual de manera que las máquinas virtuales puedan encontrar el dominio administrado para la unión o autenticación de dominios.
    * Para configurar DNS, seleccione el dominio administrado en el portal. En la ventana **Información general**, se le pedirá que configure automáticamente estos valores de DNS.
* [Habilite la sincronización de contraseñas en Azure AD DS](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds) de manera que los usuarios finales puedan iniciar sesión en el dominio administrado mediante sus credenciales corporativas.

## <a name="next-steps"></a>Pasos siguientes

Para ver el dominio administrado en acción, puede [unir a un dominio una máquina virtual Windows][windows-join], [configurar LDAP seguro][tutorial-ldaps] y [configurar la sincronización de hash de contraseñas][tutorial-phs].

<!-- INTERNAL LINKS -->
[windows-join]: join-windows-vm.md
[tutorial-ldaps]: tutorial-configure-ldaps.md
[tutorial-phs]: tutorial-configure-password-hash-sync.md
[availability-zones]: ../availability-zones/az-overview.md
[portal-deploy]: ../azure-resource-manager/templates/deploy-portal.md
[powershell-deploy]: ../azure-resource-manager/templates/deploy-powershell.md
[scoped-sync]: scoped-synchronization.md
[resource-forests]: concepts-resource-forest.md

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
[Get-AzSubscription]: /powershell/module/Az.Accounts/Get-AzSubscription
[cloud-shell]: /azure/cloud-shell/cloud-shell-windows-users
[naming-prefix]: /windows-server/identity/ad-ds/plan/selecting-the-forest-root-domain
[New-AzResourceGroupDeployment]: /powershell/module/Az.Resources/New-AzResourceGroupDeployment
