---
title: 'Ejemplos de PowerShell V2 para la administración de grupos: Azure AD | Microsoft Docs'
description: En esta página se proporcionan ejemplos de PowerShell para ayudarlo a administrar grupos de Azure Active Directory.
keywords: Azure AD, Azure Active Directory, PowerShell, grupos, administración de grupos
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: how-to
ms.date: 11/08/2019
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: e2da67b9996156f136aa2eddb8aa3d2b1777a554
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88794550"
---
# <a name="azure-active-directory-version-2-cmdlets-for-group-management"></a>Cmdlets de la versión 2 de Azure Active Directory para la administración de grupos

> [!div class="op_single_selector"]
> - [Azure Portal](../fundamentals/active-directory-groups-create-azure-portal.md?context=azure/active-directory/users-groups-roles/context/ugr-context)
> - [PowerShell](groups-settings-v2-cmdlets.md)
>
>

En este artículo se incluyen ejemplos de cómo usar PowerShell para administrar grupos en Azure Active Directory (Azure AD).  También se indica cómo configurar el módulo de Azure AD PowerShell. En primer lugar, debe [descargar el módulo de Azure AD PowerShell](https://www.powershellgallery.com/packages/AzureAD/).

## <a name="install-the-azure-ad-powershell-module"></a>Instalación del módulo de PowerShell de Azure AD

Para instalar el módulo de Azure AD PowerShell, use los siguientes comandos:

```powershell
    PS C:\Windows\system32> install-module azuread
    PS C:\Windows\system32> import-module azuread
```

Para comprobar que el módulo esté listo para usar, ejecute el siguiente comando:

```powershell
    PS C:\Windows\system32> get-module azuread

    ModuleType Version      Name                                ExportedCommands
    ---------- ---------    ----                                ----------------
    Binary     2.0.0.115    azuread                      {Add-AzureADAdministrati...}
```

Ahora puede empezar a usar los cmdlets del módulo. Para ver una descripción completa de los cmdlets del módulo de Azure AD, consulte la documentación de referencia en línea para obtener [Azure Active Directory PowerShell versión 2](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0).

> [!NOTE]
> Los cmdlets de Azure AD PowerShell no funcionan con el nuevo PowerShell 7, ya que se basa en .NET Core. Somos conscientes de que esto está en proceso de actualización. En este momento, se recomienda usar el módulo de Windows PowerShell 5.x para las operaciones de Azure AD PowerShell. 


## <a name="connect-to-the-directory"></a>Conexión al directorio

Antes de empezar a administrar los grupos mediante los cmdlets de Azure AD PowerShell, debe conectar la sesión de PowerShell al directorio que quiera administrar. Use el comando siguiente:

```powershell
    PS C:\Windows\system32> Connect-AzureAD
```

El cmdlet le pide las credenciales que quiere utilizar para tener acceso al directorio. En este ejemplo, vamos a utilizar karen@drumkit.onmicrosoft.com para tener acceso al directorio de demostración. El cmdlet devuelve una confirmación para mostrar que la sesión se ha conectado correctamente al directorio:

```powershell
    Account                       Environment Tenant ID
    -------                       ----------- ---------
    Karen@drumkit.onmicrosoft.com AzureCloud  85b5ff1e-0402-400c-9e3c-0f…
```

Ahora puede empezar a usar los cmdlets de Azure AD para administrar grupos en el directorio.

## <a name="retrieve-groups"></a>Recuperación de los grupos

Para recuperar grupos que ya se encuentren en el directorio, utilice el cmdlet Get-AzureADGroups. 

Para recuperar todos los grupos del directorio, ejecute el cmdlet sin parámetros:

```powershell
    PS C:\Windows\system32> get-azureadgroup
```

El cmdlet devuelve todos los grupos del directorio conectado.

Puede usar el parámetro -objectID si quiere recuperar un grupo específico para el que especifica el id. de objeto del grupo:

```powershell
    PS C:\Windows\system32> get-azureadgroup -ObjectId e29bae11-4ac0-450c-bc37-6dae8f3da61b
```

El cmdlet devuelve ahora el grupo cuyo id. de objeto coincide con el valor del parámetro especificado:

```powershell
    DeletionTimeStamp            :
    ObjectId                     : e29bae11-4ac0-450c-bc37-6dae8f3da61b
    ObjectType                   : Group
    Description                  :
    DirSyncEnabled               :
    DisplayName                  : Pacific NW Support
    LastDirSyncTime              :
    Mail                         :
    MailEnabled                  : False
    MailNickName                 : 9bb4139b-60a1-434a-8c0d-7c1f8eee2df9
    OnPremisesSecurityIdentifier :
    ProvisioningErrors           : {}
    ProxyAddresses               : {}
    SecurityEnabled              : True
```

Puede buscar un grupo específico mediante el parámetro - filter. Este parámetro toma una cláusula de filtro ODATA y devuelve todos los grupos que coinciden con él, como en el ejemplo siguiente:

```powershell
    PS C:\Windows\system32> Get-AzureADGroup -Filter "DisplayName eq 'Intune Administrators'"


    DeletionTimeStamp            :
    ObjectId                     : 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df
    ObjectType                   : Group
    Description                  : Intune Administrators
    DirSyncEnabled               :
    DisplayName                  : Intune Administrators
    LastDirSyncTime              :
    Mail                         :
    MailEnabled                  : False
    MailNickName                 : 4dd067a0-6515-4f23-968a-cc2ffc2eff5c
    OnPremisesSecurityIdentifier :
    ProvisioningErrors           : {}
    ProxyAddresses               : {}
    SecurityEnabled              : True
```

> [!NOTE]
> Los cmdlets de Azure AD PowerShell implementan el estándar de consulta de OData. Para obtener más información, consulte **$filter** en [Opciones de la consulta del sistema OData mediante el punto de conexión de OData](/previous-versions/dynamicscrm-2015/developers-guide/gg309461(v=crm.7)#BKMK_filter).

## <a name="create-groups"></a>Creación de grupos

Para crear un nuevo grupo en el directorio, use el cmdlet New-AzureADGroup. Este cmdlet crea un nuevo grupo de seguridad llamado "Marketing":

```powershell
    PS C:\Windows\system32> New-AzureADGroup -Description "Marketing" -DisplayName "Marketing" -MailEnabled $false -SecurityEnabled $true -MailNickName "Marketing"
```

## <a name="update-groups"></a>Actualización de grupos

Para actualizar un grupo que ya exista, utilice el cmdlet Set-AzureADGroup. En este ejemplo, vamos a cambiar la propiedad DisplayName del grupo "Administradores de Intune". En primer lugar, buscaremos el grupo mediante el cmdlet Get-AzureADGroup y aplicaremos un filtro usando el atributo DisplayName:

```powershell
    PS C:\Windows\system32> Get-AzureADGroup -Filter "DisplayName eq 'Intune Administrators'"


    DeletionTimeStamp            :
    ObjectId                     : 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df
    ObjectType                   : Group
    Description                  : Intune Administrators
    DirSyncEnabled               :
    DisplayName                  : Intune Administrators
    LastDirSyncTime              :
    Mail                         :
    MailEnabled                  : False
    MailNickName                 : 4dd067a0-6515-4f23-968a-cc2ffc2eff5c
    OnPremisesSecurityIdentifier :
    ProvisioningErrors           : {}
    ProxyAddresses               : {}
    SecurityEnabled              : True
```

Después, vamos a cambiar la propiedad Description del nuevo valor de "Administradores de dispositivos de Intune":

```powershell
    PS C:\Windows\system32> Set-AzureADGroup -ObjectId 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df -Description "Intune Device Administrators"
```

Ahora, si encontramos el grupo nuevo, veremos que la propiedad Description se actualiza para reflejar el nuevo valor:

```powershell
    PS C:\Windows\system32> Get-AzureADGroup -Filter "DisplayName eq 'Intune Administrators'"

    DeletionTimeStamp            :
    ObjectId                     : 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df
    ObjectType                   : Group
    Description                  : Intune Device Administrators
    DirSyncEnabled               :
    DisplayName                  : Intune Administrators
    LastDirSyncTime              :
    Mail                         :
    MailEnabled                  : False
    MailNickName                 : 4dd067a0-6515-4f23-968a-cc2ffc2eff5c
    OnPremisesSecurityIdentifier :
    ProvisioningErrors           : {}
    ProxyAddresses               : {}
    SecurityEnabled              : True
```

## <a name="delete-groups"></a>Eliminación de grupos

Para eliminar grupos del directorio, use el cmdlet Remove-AzureADGroup de la siguiente manera:

```powershell
    PS C:\Windows\system32> Remove-AzureADGroup -ObjectId b11ca53e-07cc-455d-9a89-1fe3ab24566b
```

## <a name="manage-group-membership"></a>Administración de pertenencia al grupo

### <a name="add-members"></a>Adición de miembros

Para agregar nuevos miembros a un grupo, utilice el cmdlet Add-AzureADGroupMember. Este comando agrega un miembro al grupo de administradores de Intune que utilizamos en el ejemplo anterior:

```powershell
    PS C:\Windows\system32> Add-AzureADGroupMember -ObjectId 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df -RefObjectId 72cd4bbd-2594-40a2-935c-016f3cfeeeea
```

El parámetro -ObjectId es el id. de objeto del grupo al que desea agregar un miembro y RefObjectId es el del usuario que desea agregar como miembro al grupo.

### <a name="get-members"></a>Obtención de miembros

Para obtener los miembros existentes de un grupo, use el cmdlet Get-AzureADGroupMember, como en este ejemplo:

```powershell
    PS C:\Windows\system32> Get-AzureADGroupMember -ObjectId 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df

    DeletionTimeStamp ObjectId                             ObjectType
    ----------------- --------                             ----------
                          72cd4bbd-2594-40a2-935c-016f3cfeeeea User
                          8120cc36-64b4-4080-a9e8-23aa98e8b34f User
```

### <a name="remove-members"></a>Eliminación de miembros

Para quitar al miembro que agregamos anteriormente al grupo, use el cmdlet Remove-AzureADGroupMember, como se muestra aquí:

```powershell
    PS C:\Windows\system32> Remove-AzureADGroupMember -ObjectId 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df -MemberId 72cd4bbd-2594-40a2-935c-016f3cfeeeea
```

### <a name="verify-members"></a>Comprobación de miembros

Para comprobar si un usuario pertenece a un grupo, use el cmdlet Select-AzureADGroupIdsUserIsMemberOf. Este cmdlet toma como parámetros el id. de objeto del usuario que se comprueba si pertenece a uno o varios grupos, así como una lista de grupos para realizar dicha comprobación. La lista de grupos debe proporcionarse como una variable compleja del tipo Microsoft.Open.AzureAD.Model.GroupIdsForMembershipCheck, así que, antes, debemos creamos una variable con ese tipo:

```powershell
    PS C:\Windows\system32> $g = new-object Microsoft.Open.AzureAD.Model.GroupIdsForMembershipCheck
```

Después, proporcionamos los valores de los id. de grupo para comprobar el atributo GroupIds de esta variable compleja:

```powershell
    PS C:\Windows\system32> $g.GroupIds = "b11ca53e-07cc-455d-9a89-1fe3ab24566b", "31f1ff6c-d48c-4f8a-b2e1-abca7fd399df"
```

Ahora, si queremos comprobar si un usuario con el id. de objeto 72cd4bbd-2594-40a2-935c-016f3cfeeeea pertenece a uno de los grupos de $g, debemos usar lo siguiente:

```powershell
    PS C:\Windows\system32> Select-AzureADGroupIdsUserIsMemberOf -ObjectId 72cd4bbd-2594-40a2-935c-016f3cfeeeea -GroupIdsForMembershipCheck $g

    OdataMetadata                                                                                                 Value
    -------------                                                                                                  -----
    https://graph.windows.net/85b5ff1e-0402-400c-9e3c-0f9e965325d1/$metadata#Collection(Edm.String)             {31f1ff6c-d48c-4f8a-b2e1-abca7fd399df}
```

El valor devuelto es una lista con los grupos de los que es miembro este usuario. También puede aplicar este método para comprobar si determinados contactos, grupos o entidades de seguridad pertenecen a los grupos de una lista concreta. Para ello, utilice Select-AzureADGroupIdsContactIsMemberOf, Select-AzureADGroupIdsGroupIsMemberOf o Select-AzureADGroupIdsServicePrincipalIsMemberOf.

## <a name="disable-group-creation-by-your-users"></a>Deshabilitación de la creación de grupos por los usuarios

Puede impedir que los usuarios no administradores creen grupos de seguridad. El comportamiento predeterminado en Microsoft Online Directory Services (MSODS) es permitir que los usuarios no administrativos creen grupos, esté o no habilitada la administración de grupos en autoservicio (SSGM). La configuración de SSGM controla el comportamiento solo en el panel de acceso Mis aplicaciones.

Para deshabilitar la creación de grupos por usuarios no administrativos:

1. Compruebe que los usuarios no administradores pueden crear grupos:
   
   ```powershell
   PS C:\> Get-MsolCompanyInformation | fl UsersPermissionToCreateGroupsEnabled
   ```
  
2. Si devuelve `UsersPermissionToCreateGroupsEnabled : True`, los usuarios no administrativos pueden crear grupos. Para deshabilitar esta característica:
  
   ```powershell 
   Set-MsolCompanySettings -UsersPermissionToCreateGroupsEnabled $False
   ```
  
## <a name="manage-owners-of-groups"></a>Administración de propietarios de grupos

Para agregar propietarios a un grupo, utilice el cmdlet Add-AzureADGroupOwner:

```powershell
    PS C:\Windows\system32> Add-AzureADGroupOwner -ObjectId 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df -RefObjectId 72cd4bbd-2594-40a2-935c-016f3cfeeeea
```

El parámetro -ObjectId es el identificador de objeto del grupo al que queremos agregar un propietario y -RefObjectId es el identificador del usuario o la entidad de servicio que queremos agregar como propietario del grupo.

Para recuperar los propietarios de un grupo, utilice el cmdlet Get-AzureADGroupOwner:

```powershell
    PS C:\Windows\system32> Get-AzureADGroupOwner -ObjectId 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df
```

El cmdlet devuelve la lista de propietarios (usuarios y entidades de servicio) del grupo especificado:

```powershell
    DeletionTimeStamp ObjectId                             ObjectType
    ----------------- --------                             ----------
                          e831b3fd-77c9-49c7-9fca-de43e109ef67 User
```

Si desea quitar un propietario de un grupo, utilice el cmdlet Remove-AzureADGroupOwner:

```powershell
    PS C:\Windows\system32> remove-AzureADGroupOwner -ObjectId 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df -OwnerId e831b3fd-77c9-49c7-9fca-de43e109ef67
```

## <a name="reserved-aliases"></a>Alias reservados

Cuando se crea un grupo, algunos de los puntos de conexión permiten al usuario final especificar un mailNickname o alias que se usarán como parte de la dirección de correo electrónico del grupo. Los grupos con los siguientes alias de correo electrónico con privilegios elevados solo puede crearlos un administrador global de Azure AD. 
  
* abuse
* admin
* administrator
* hostmaster
* majordomo
* postmaster
* root
* secure
* security
* ssl-admin
* webmaster

## <a name="group-writeback-to-on-premises-preview"></a>Escritura diferida de grupos en entornos locales (versión preliminar)

Hoy en día, se siguen administrando muchos grupos en Active Directory local. Para responder a las solicitudes para sincronizar grupos en la nube de nuevo con entornos locales, la característica de escritura diferida de grupos de Office 365 para Azure AD está ahora disponible en versión preliminar.

Los grupos de Office 365 se crean y administran en la nube. La capacidad de escritura diferida le permite volver a escribir los grupos de Office 365 como grupos de distribución en un bosque de Active Directory con Exchange instalado. Después, los usuarios con buzones de Exchange locales pueden enviar y recibir correos electrónicos de estos grupos. La característica de reescritura de grupos no admite grupos de distribución ni grupos de seguridad de Azure AD.

Para obtener más información, consulte la documentación para el [servicio de sincronización de Azure AD Connect](../hybrid/how-to-connect-syncservice-features.md).

La escritura diferida de grupos de Office 365 es una característica en versión preliminar pública de Azure Active Directory (Azure AD) y está disponible con cualquier plan de licencia de Azure AD de pago. Para obtener alguna información legal sobre las versiones preliminares, vea [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="next-steps"></a>Pasos siguientes

Puede encontrar más documentación de Azure Active Directory PowerShell en el artículo sobre los [cmdlets de Azure Active Directory](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0).

* [Administración del acceso a los recursos con grupos de Azure Active Directory](../fundamentals/active-directory-manage-groups.md?context=azure/active-directory/users-groups-roles/context/ugr-context)
* [Integración de las identidades locales con Azure Active Directory](../hybrid/whatis-hybrid-identity.md?context=azure/active-directory/users-groups-roles/context/ugr-context)