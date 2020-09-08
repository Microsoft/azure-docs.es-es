---
title: 'Azure AD Connect: Configuración de los permisos de cuenta del conector de AD DS | Microsoft Docs'
description: En este documento se detalla la manera de configurar la cuenta del conector de AD DS con el nuevo módulo de PowerShell ADSyncConfig.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 05/18/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9a082270e2c113bcdf31ed6bd6db3d38a7117500
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279302"
---
# <a name="azure-ad-connectconfigure-ad-ds-connector-account-permissions"></a>Azure AD Connect: Configurar los permisos de cuenta del conector de AD DS 

Con la compilación 1.1.880.0 (publicada en agosto de 2018), se introdujo el módulo de PowerShell denominado [ADSyncConfig.psm1](reference-connect-adsyncconfig.md) que incluye una colección de cmdlets que le ayudarán a configurar los permisos correctos de Active Directory para su implementación de Azure AD Connect. 

## <a name="overview"></a>Información general 
Los siguientes cmdlets de PowerShell se pueden usar para configurar los permisos de Active Directory de la cuenta del conector de AD DS, para cada característica que elija habilitar en Azure AD Connect. Para evitar problemas, recuerde que debe preparar de antemano los permisos de Active Directory siempre que quiera instalar Azure AD Connect con una cuenta de dominio personalizada para conectarse a su bosque. Este módulo ADSyncConfig también se puede usar para configurar permisos después de implementar Azure AD Connect.

![información general de la cuenta de AD DS](media/how-to-connect-configure-ad-ds-connector-account/configure1.png)

Para instalar Azure AD Connect Express, se crea una cuenta generada automáticamente (MSOL_nnnnnnnnnn) en Active Directory que incluye todos los permisos necesarios. De este modo, no es necesario usar el módulo ADSyncConfig a menos que haya bloqueado la herencia de permisos en unidades organizativas o en objetos específicos de Active Directory que quiera sincronizar en Azure AD. 
 
### <a name="permissions-summary"></a>Resumen de permisos 
En la siguiente tabla se proporciona un resumen de los permisos necesarios en los objetos de AD: 

| Característica | Permisos |
| --- | --- |
| característica ms-DS-ConsistencyGuid |Permisos de lectura y escritura para el atributo ms-DS-ConsistencyGuid documentado en [Conceptos de diseño: Uso de ms-DS-ConsistencyGuid como sourceAnchor](plan-connect-design-concepts.md#using-ms-ds-consistencyguid-as-sourceanchor). | 
| Sincronización de hash de contraseñas |<li>Replicación de cambios de directorio</li>  <li>Replicación de todos los cambios de directorio |
| Implementación híbrida de Exchange |Permisos de lectura y escritura en los atributos que se documentan en [Escritura diferida híbrida de Exchange](reference-connect-sync-attributes-synchronized.md#exchange-hybrid-writeback) para usuarios, grupos y contactos. |
| Carpeta pública de correo de Exchange |Permisos de lectura para los atributos que se documentan en [carpetas públicas de correo electrónico de Exchange](reference-connect-sync-attributes-synchronized.md#exchange-mail-public-folder) para las carpetas públicas. | 
| escritura diferida de contraseñas |Permisos de lectura y escritura en los atributos que se documentan en [Introducción a la administración de contraseñas](../authentication/tutorial-enable-sspr-writeback.md) para los usuarios. |
| Escritura diferida de dispositivos |Permisos de lectura y escritura a objetos de dispositivo y contenedores que se documentan en la [escritura diferida de dispositivo](how-to-connect-device-writeback.md). |
| Escritura diferida de grupos |Objetos de grupo Leer, Crear, Actualizar y Eliminar para **grupos de Office 365** sincronizados.|

## <a name="using-the-adsyncconfig-powershell-module"></a>Usar el módulo de PowerShell ADSyncConfig 
El módulo ADSyncConfig requiere las [Herramientas de administración remota del servidor (RSAT) para AD DS](/windows-server/remote/remote-server-administration-tools), ya que depende del módulo y las herramientas de PowerShell para AD DS. Para instalar RSAT para AD DS, abra una ventana de Windows PowerShell con la opción "Ejecutar como administrador" y ejecute: 

``` powershell
Install-WindowsFeature RSAT-AD-Tools 
```
![Configuración](media/how-to-connect-configure-ad-ds-connector-account/configure2.png)

>[!NOTE]
>También puede copiar el archivo **C:\Archivos de programa\Microsoft Azure Active Directory Connect\AdSyncConfig\ADSyncConfig.psm1** a un controlador de dominio que ya tenga RSAT para AD DS instalado y usar este módulo de PowerShell desde ahí.

Para comenzar a usar ADSyncConfig, debe cargar el módulo en una ventana de Windows PowerShell: 

``` powershell
Import-Module "C:\Program Files\Microsoft Azure Active Directory Connect\AdSyncConfig\AdSyncConfig.psm1" 
```

Para comprobar todos los cmdlets incluidos en este módulo, puede escribir lo siguiente:  

``` powershell
Get-Command -Module AdSyncConfig  
```

![Comprobar](media/how-to-connect-configure-ad-ds-connector-account/configure3.png)

Cada cmdlet tiene los mismos parámetros para introducir la cuenta del conector de AD DS y un modificador de AdminSDHolder. Para especificar su cuenta de conector de AD DS, puede proporcionar el nombre de la cuenta y el dominio, o simplemente el nombre distintivo (DN) de la cuenta.

Por ejemplo:

```powershell
Set-ADSyncPasswordHashSyncPermissions -ADConnectorAccountName <ADAccountName> -ADConnectorAccountDomain <ADDomainName>
```

O bien:

```powershell
Set-ADSyncPasswordHashSyncPermissions -ADConnectorAccountDN <ADAccountDN>
```

No olvide reemplazar `<ADAccountName>`, `<ADDomainName>` y `<ADAccountDN>` con los valores adecuados para su entorno.

En caso de que no quiera modificar los permisos en el contenedor AdminSDHolder, use el modificador `-SkipAdminSdHolders`. 

De forma predeterminada, todos los cmdlets de permisos establecidos intentarán establecer los permisos de AD DS en la raíz de cada dominio en el bosque, lo que significa que el usuario que ejecuta la sesión de PowerShell necesita tener derechos de administrador de dominio en cada dominio del bosque.  Debido a este requisito, es recomendable usar un administrador de empresa desde la raíz del bosque. Si su implementación de Azure AD Connect tiene varios conectores de AD DS, será necesario que ejecute el mismo cmdlet en cada bosque que tenga un conector de AD DS. 

Asimismo, también puede establecer permisos en un objeto específico de la unidad organizativa o de AD DS mediante el parámetro `-ADobjectDN`, seguido del nombre distintivo del objeto de destino en el que quiere establecer esos permisos. Cuando se usa un parámetro ADobjectDN de destino, el cmdlet establecerá los permisos solo en este objeto y no en la raíz del dominio o el contenedor AdminSDHolder. Este parámetro puede ser útil cuando tiene ciertas unidades organizativas u objetos de AD DS que tienen la herencia de permisos deshabilitada (consulte la sección Buscar objetos de AD DS con la herencia de permisos deshabilitada). 

Las excepciones a estos parámetros comunes son el cmdlet `Set-ADSyncRestrictedPermissions`, que se usa para establecer los permisos en la propia cuenta del conector de AD DS, y el cmdlet `Set-ADSyncPasswordHashSyncPermissions`, puesto que los permisos necesarios para la sincronización del hash de contraseñas solo se establecen en la raíz del dominio y, por lo tanto, el cmdlet no incluye los parámetros `-ObjectDN` o `-SkipAdminSdHolders`.

### <a name="determine-your-ad-ds-connector-account"></a>Determine su cuenta de conector de AD DS 
En caso de que Azure AD Connect ya esté instalado y quiera comprobar cuál es la cuenta del conector de AD DS que usa actualmente Azure AD Connect, puede ejecutar el siguiente cmdlet: 

``` powershell
Get-ADSyncADConnectorAccount 
```
### <a name="locate-ad-ds-objects-with-permission-inheritance-disabled"></a>Buscar objetos de AD DS con la herencia de permisos deshabilitada 
En caso de que quiera comprobar si hay algún objeto de AD DS con la herencia de permisos deshabilitada, puede ejecutar lo siguiente: 

``` powershell
Get-ADSyncObjectsWithInheritanceDisabled -SearchBase '<DistinguishedName>' 
```
De forma predeterminada, este cmdlet solo buscará unidades organizativas con la herencia deshabilitada, pero puede especificar otras clases de objetos de AD DS en el parámetro `-ObjectClass` o usar "*" para todas las clases de objetos, de la siguiente manera: 

``` powershell
Get-ADSyncObjectsWithInheritanceDisabled -SearchBase '<DistinguishedName>' -ObjectClass * 
```
 
### <a name="view-ad-ds-permissions-of-an-object"></a>Ver los permisos de AD DS de un objeto 
Puede usar el cmdlet que hay a continuación para ver la lista de permisos establecidos actualmente en un objeto de Active Directory mediante el elemento DistinguishedName: 

``` powershell
Show-ADSyncADObjectPermissions -ADobjectDN '<DistinguishedName>' 
```

## <a name="configure-ad-ds-connector-account-permissions"></a>Configurar los permisos de cuenta del conector de AD DS 
 
### <a name="configure-basic-read-only-permissions"></a>Configurar los permisos básicos de solo lectura 
Para configurar los permisos básicos de solo lectura para la cuenta del conector de AD DS cuando no use ninguna característica de Azure AD Connect, ejecute lo siguiente: 

``` powershell
Set-ADSyncBasicReadPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```


O bien: 

``` powershell
Set-ADSyncBasicReadPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```


Este cmdlet establecerá los siguientes permisos: 
 

|Tipo |Nombre |Acceso |Se aplica a| 
|-----|-----|-----|-----|
|Allow |Cuenta del conector de AD DS |Lectura de todas las propiedades |Objetos del dispositivo descendientes| 
|Allow |Cuenta del conector de AD DS|Lectura de todas las propiedades |Objetos InetOrgPerson descendientes| 
|Allow |Cuenta del conector de AD DS |Lectura de todas las propiedades |Objetos del equipo descendientes| 
|Allow |Cuenta del conector de AD DS |Lectura de todas las propiedades |Objetos foreignSecurityPrincipal descendientes| 
|Allow |Cuenta del conector de AD DS |Lectura de todas las propiedades |Objetos del grupo descendientes| 
|Allow |Cuenta del conector de AD DS |Lectura de todas las propiedades |Objetos del usuario descendientes| 
|Allow |Cuenta del conector de AD DS |Lectura de todas las propiedades |Objetos del contacto descendiente| 

 
### <a name="configure-ms-ds-consistency-guid-permissions"></a>Configurar los permisos de MS-DS-Consistency-Guid 
Para establecer los permisos de la cuenta del conector de AD DS cuando se usa el atributo ms-Ds-Consistency-Guid como el delimitador de origen (también conocido como la opción "Let Azure manage the source anchor for me" [Permitir que Azure administre automáticamente el delimitador de origen]), ejecute lo siguiente: 

``` powershell
Set-ADSyncMsDsConsistencyGuidPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```

O bien: 

``` powershell
Set-ADSyncMsDsConsistencyGuidPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```

Este cmdlet establecerá los siguientes permisos: 

|Tipo |Nombre |Acceso |Se aplica a|
|-----|-----|-----|-----| 
|Allow|Cuenta del conector de AD DS|Propiedad de lectura/escritura|Objetos del usuario descendientes|

### <a name="permissions-for-password-hash-synchronization"></a>Permisos para la sincronización del hash de contraseñas 
Para establecer los permisos de la cuenta del conector de AD DS cuando se usa la sincronización del hash de contraseñas, ejecute lo siguiente: 

``` powershell
Set-ADSyncPasswordHashSyncPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [<CommonParameters>] 
```


O bien: 

``` powershell
Set-ADSyncPasswordHashSyncPermissions -ADConnectorAccountDN <String> [<CommonParameters>] 
```

Este cmdlet establecerá los siguientes permisos: 

|Tipo |Nombre |Acceso |Se aplica a|
|-----|-----|-----|-----| 
|Allow |Cuenta del conector de AD DS |Replicación de los cambios de directorio |Solo este objeto (raíz del dominio)| 
|Allow |Cuenta del conector de AD DS |Replicación de todos los cambios de directorio |Solo este objeto (raíz del dominio)| 
  
### <a name="permissions-for-password-writeback"></a>Permisos de escritura diferida de contraseñas 
Para establecer los permisos de la cuenta del conector de AD DS cuando se usa la escritura diferida de contraseñas, ejecute lo siguiente: 

``` powershell
Set-ADSyncPasswordWritebackPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```


O bien:

``` powershell
Set-ADSyncPasswordWritebackPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```
Este cmdlet establecerá los siguientes permisos: 

|Tipo |Nombre |Acceso |Se aplica a|
|-----|-----|-----|-----| 
|Allow |Cuenta del conector de AD DS |Restablecimiento de contraseña |Objetos del usuario descendientes| 
|Allow |Cuenta del conector de AD DS |Escritura del elemento lockoutTime de la propiedad |Objetos del usuario descendientes| 
|Allow |Cuenta del conector de AD DS |Escritura del elemento pwdLastSet de la propiedad |Objetos del usuario descendientes| 

### <a name="permissions-for-group-writeback"></a>Permisos de escritura diferida de grupos 
Para establecer los permisos de la cuenta del conector de AD DS cuando se usa la escritura diferida de grupos, ejecute lo siguiente: 

``` powershell
Set-ADSyncUnifiedGroupWritebackPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```
O bien: 

``` powershell
Set-ADSyncUnifiedGroupWritebackPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>]
```
 
Este cmdlet establecerá los siguientes permisos: 

|Tipo |Nombre |Acceso |Se aplica a|
|-----|-----|-----|-----| 
|Allow |Cuenta del conector de AD DS |Lectura/escritura genérica |Todos los atributos del grupo de tipo de objeto y subobjetos| 
|Allow |Cuenta del conector de AD DS |Creación o eliminación de los objetos secundarios |Todos los atributos del grupo de tipo de objeto y subobjetos| 
|Allow |Cuenta del conector de AD DS |Eliminación/eliminación de objetos de árbol|Todos los atributos del grupo de tipo de objeto y subobjetos|

### <a name="permissions-for-exchange-hybrid-deployment"></a>Permisos para la implementación híbrida de Exchange 
Para establecer los permisos de la cuenta del conector de AD DS cuando se usa la implementación híbrida de Exchange, ejecute lo siguiente: 

``` powershell
Set-ADSyncExchangeHybridPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```


O bien: 

``` powershell
Set-ADSyncExchangeHybridPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```

Este cmdlet establecerá los siguientes permisos:  
 

|Tipo |Nombre |Acceso |Se aplica a|
|-----|-----|-----|-----| 
|Allow |Cuenta del conector de AD DS |Lectura y escritura de todas las propiedades |Objetos del usuario descendientes| 
|Allow |Cuenta del conector de AD DS |Lectura y escritura de todas las propiedades |Objetos InetOrgPerson descendientes| 
|Allow |Cuenta del conector de AD DS |Lectura y escritura de todas las propiedades |Objetos del grupo descendientes| 
|Allow |Cuenta del conector de AD DS |Lectura y escritura de todas las propiedades |Objetos del contacto descendiente| 

### <a name="permissions-for-exchange-mail-public-folders-preview"></a>Permisos para carpetas públicas de correo de Exchange (versión preliminar) 
Para establecer los permisos de la cuenta del conector de AD DS cuando se usa la característica de las carpetas públicas de correo de Exchange, ejecute lo siguiente: 

``` powershell
Set-ADSyncExchangeMailPublicFolderPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```


O bien: 

``` powershell
Set-ADSyncExchangeMailPublicFolderPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```
Este cmdlet establecerá los siguientes permisos: 

|Tipo |Nombre |Acceso |Se aplica a|
|-----|-----|-----|-----| 
|Allow |Cuenta del conector de AD DS |Lectura de todas las propiedades |Objetos PublicFolder descendientes| 

### <a name="restrict-permissions-on-the-ad-ds-connector-account"></a>Restringir los permisos en la cuenta del conector de AD DS 
Este script de PowerShell reforzará los permisos para la cuenta del conector de AD que se proporcionó como parámetro. Para limitar los permisos, es necesario realizar los pasos siguientes: 

- Deshabilite la herencia en el objeto especificado. 
- Elimine todas las ACE en el objeto específico, excepto las ACE específicas de SELF, ya que queremos mantener intactos los permisos predeterminados cuando usemos SELF. 
 
  El parámetro -ADConnectorAccountDN es la cuenta de AD cuyos permisos deben limitarse. Esta suele ser la cuenta de dominio MSOL_nnnnnnnnnnnn que está configurada en el conector de AD DS (consulte la sección Determinar su cuenta de conector de AD DS). El parámetro -Credential, por su parte, es necesario para especificar la cuenta de administrador que tiene los privilegios necesarios para restringir los permisos de Active Directory en el objeto de AD de destino. Por lo general, suele ser el administrador de organización o de dominio.  

``` powershell
Set-ADSyncRestrictedPermissions [-ADConnectorAccountDN] <String> [-Credential] <PSCredential> [-DisableCredentialValidation] [-WhatIf] [-Confirm] [<CommonParameters>] 
```
 
Por ejemplo: 

``` powershell
$credential = Get-Credential 
Set-ADSyncRestrictedPermissions -ADConnectorAccountDN'CN=ADConnectorAccount,CN=Users,DC=Contoso,DC=com' -Credential $credential  
```

Este cmdlet establecerá los siguientes permisos: 

|Tipo |Nombre |Acceso |Se aplica a|
|-----|-----|-----|-----| 
|Allow |SYSTEM |Control total |Este objeto 
|Allow |Administradores de empresas |Control total |Este objeto 
|Allow |Administradores de dominio |Control total |Este objeto 
|Allow |Administradores |Control total |Este objeto 
|Allow |Enterprise Domain Controllers |Contenido de la lista |Este objeto 
|Allow |Enterprise Domain Controllers |Lectura de todas las propiedades |Este objeto 
|Allow |Enterprise Domain Controllers |Permisos de lectura |Este objeto 
|Allow |Usuarios autenticados |Contenido de la lista |Este objeto 
|Allow |Usuarios autenticados |Lectura de todas las propiedades |Este objeto 
|Allow |Usuarios autenticados |Permisos de lectura |Este objeto 

## <a name="next-steps"></a>Pasos siguientes
- [Azure AD Connect: cuentas y permisos](reference-connect-accounts-permissions.md)
- [Instalación rápida](how-to-connect-install-express.md)
- [Instalación personalizada](how-to-connect-install-custom.md)
- [Referencia de ADSyncConfig](reference-connect-adsyncconfig.md)