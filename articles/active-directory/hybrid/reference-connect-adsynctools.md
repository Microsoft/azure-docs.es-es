---
title: 'Azure AD Connect: Referencia de PowerShell ADSyncTools | Microsoft Docs'
description: En este documento se proporciona información de referencia para el módulo de PowerShell ADSyncTools.psm1.
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.date: 04/23/2020
ms.subservice: hybrid
ms.author: billmath
ms.topic: reference
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f81af557242503c6380d0ff7bc1dfaed852cd908
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89070690"
---
# <a name="azure-ad-connect--adsynctools-powershell-reference"></a>Azure AD Connect:  Referencia de PowerShell ADSyncTools
En la siguiente documentación se proporciona información de referencia para el módulo de PowerShell ADSyncTools.psm1 que se incluye con Azure AD Connect.

## <a name="install-the-adsynctools-powershell-module"></a>Instalación del módulo de PowerShell ADSyncTools
Para instalar el módulo de PowerShell ADSyncTools, haga lo siguiente:

1.  Abra Windows PowerShell con privilegios de administrador.
2.  Escriba o copie y pegue lo siguiente: 
    ``` powershell
    Import-module -Name "C:\Program Files\Microsoft Azure Active Directory Connect\Tools\AdSyncTools"
    ```
3.  Presione Entrar.
4.  Para comprobar que se ha instalado el módulo, escriba o copie y pegue lo siguiente:
    ```powershell
    Get-module AdSyncTools
    ```
5.  Ahora debería ver información sobre el módulo.


## <a name="clear-adsynctoolsconsistencyguid"></a>Clear-ADSyncToolsConsistencyGuid

### <a name="synopsis"></a>SINOPSIS
Borrar mS-Ds-ConsistencyGuid del usuario de AD

### <a name="syntax"></a>SINTAXIS

```
Clear-ADSyncToolsConsistencyGuid [-User] <Object> [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
Borrar el valor de mS-Ds-ConsistencyGuid para el usuario de AD de destino

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
Example of how to use this cmdlet
```

#### <a name="example-2"></a>EJEMPLO 2
```
Another example of how to use this cmdlet
```

### <a name="parameters"></a>PARAMETERS

#### <a name="-user"></a>-User
Usuario de destino de AD que se va a establecer

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: True
Position: 1
Default value: None
Accept pipeline input: True (ByPropertyName, ByValue)
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

## <a name="confirm-adsynctoolsadmoduleloaded"></a>Confirm-ADSyncToolsADModuleLoaded

### <a name="synopsis"></a>SINOPSIS
{{Indicar sinopsis}}

### <a name="syntax"></a>SINTAXIS

```
Confirm-ADSyncToolsADModuleLoaded
```

### <a name="description"></a>DESCRIPTION
{{Indicar descripción}}

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>Ejemplo 1
```powershell
PS C:\> {{ Add example code here }}
```

{{Agregar descripción de ejemplo aquí}}

## <a name="connect-adsyncdatabase"></a>Connect-AdSyncDatabase

### <a name="synopsis"></a>SINOPSIS
{{Indicar sinopsis}}

### <a name="syntax"></a>SINTAXIS

```
Connect-AdSyncDatabase [-Server] <String> [[-Instance] <String>] [[-Database] <String>] [[-UserName] <String>]
 [[-Password] <String>] [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
{{Indicar descripción}}

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>Ejemplo 1
```powershell
PS C:\> {{ Add example code here }}
```

{{Agregar descripción de ejemplo aquí}}

### <a name="parameters"></a>PARAMETERS

#### <a name="-database"></a>-Database
{{Indicar descripción de la base de datos}}

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 2
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-instance"></a>-Instance
{{Indicar descripción de la instancia}}

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 1
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-password"></a>-Password
{{Indicar descripción de la contraseña}}

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 4
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-server"></a>-Server
{{Indicar descripción del servidor}}

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: True
Position: 0
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-username"></a>-UserName
{{Indicar descripción del nombre de usuario}}

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 3
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

## <a name="export-adsynctoolsconsistencyguidmigration"></a>Export-ADSyncToolsConsistencyGuidMigration

### <a name="synopsis"></a>SINOPSIS
Exportar informe de ConsistencyGuid

### <a name="syntax"></a>SINTAXIS

```
Export-ADSyncToolsConsistencyGuidMigration [-AlternativeLoginId] [-UserPrincipalName] <String>
 [-ImmutableIdGUID] <String> [-Output] <String> [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
Genera un informe de ConsistencyGuid basado en un archivo CSV de importación desde Import-ADSyncToolsImmutableIdMigration

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
Import-Csv .\AllSyncUsers.csv | Export-ADSyncToolsConsistencyGuidMigration -Output ".\AllSyncUsers-Report"
```

#### <a name="example-2"></a>EJEMPLO 2
```
Another example of how to use this cmdlet
```

### <a name="parameters"></a>PARAMETERS

#### <a name="-alternativeloginid"></a>-AlternativeLoginId
Usar un id. de inicio de sesión alternativo (correo)

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: False
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-userprincipalname"></a>-UserPrincipalName
UserPrincipalName

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: True
Position: 1
Default value: None
Accept pipeline input: True (ByPropertyName, ByValue)
Accept wildcard characters: False
```

#### <a name="-immutableidguid"></a>-ImmutableIdGUID
ImmutableIdGUID

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: True
Position: 2
Default value: None
Accept pipeline input: True (ByPropertyName, ByValue)
Accept wildcard characters: False
```

#### <a name="-output"></a>-Output
Nombre de archivo de salida para archivos CSV y LOG

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: True
Position: 3
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

## <a name="get-adsyncsqlbrowserinstances"></a>Get-ADSyncSQLBrowserInstances

### <a name="synopsis"></a>SINOPSIS
{{Indicar sinopsis}}

### <a name="syntax"></a>SINTAXIS

```
Get-ADSyncSQLBrowserInstances [[-hostName] <String>]
```

### <a name="description"></a>DESCRIPTION
{{Indicar descripción}}

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>Ejemplo 1
```powershell
PS C:\> {{ Add example code here }}
```

{{Agregar descripción de ejemplo aquí}}

### <a name="parameters"></a>PARAMETERS

#### <a name="-hostname"></a>-hostName
{{Indicar descripción del nombre de host}}

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 0
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

## <a name="get-adsynctoolsaduser"></a>Get-ADSyncToolsADuser

### <a name="synopsis"></a>SINOPSIS
Obtener usuario de AD

### <a name="syntax"></a>SINTAXIS

```
Get-ADSyncToolsADuser [-User] <Object> [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
Devuelve un objeto de AD ACCIÓN: Compatibilidad con varios bosques

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
Example of how to use this cmdlet
```

#### <a name="example-2"></a>EJEMPLO 2
```
Another example of how to use this cmdlet
```

### <a name="parameters"></a>PARAMETERS

#### <a name="-user"></a>-User
Usuario de destino de AD para establecer ConsistencyGuid

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: True
Position: 1
Default value: None
Accept pipeline input: True (ByPropertyName, ByValue)
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

## <a name="get-adsynctoolsconsistencyguid"></a>Get-ADSyncToolsConsistencyGuid

### <a name="synopsis"></a>SINOPSIS
Obtener mS-Ds-ConsistencyGuid del usuario de AD

### <a name="syntax"></a>SINTAXIS

```
Get-ADSyncToolsConsistencyGuid [-User] <Object> [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
Devuelve el valor del atributo mS-Ds-ConsistencyGuid del usuario de AD de destino en formato de GUID

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
Example of how to use this cmdlet
```

#### <a name="example-2"></a>EJEMPLO 2
```
Another example of how to use this cmdlet
```

### <a name="parameters"></a>PARAMETERS

#### <a name="-user"></a>-User
Usuario de destino de AD que se va a establecer

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: True
Position: 1
Default value: None
Accept pipeline input: True (ByPropertyName, ByValue)
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

## <a name="get-adsynctoolsobjectguid"></a>Get-ADSyncToolsObjectGuid

### <a name="synopsis"></a>SINOPSIS
Obtener ObjectGuid del usuario de AD

### <a name="syntax"></a>SINTAXIS

```
Get-ADSyncToolsObjectGuid [-User] <Object> [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
Devuelve el valor del atributo ObjectGUID del usuario de AD de destino en formato de GUID

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
Example of how to use this cmdlet
```

#### <a name="example-2"></a>EJEMPLO 2
```
Another example of how to use this cmdlet
```

### <a name="parameters"></a>PARAMETERS

#### <a name="-user"></a>-User
Usuario de destino de AD que se va a establecer

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: True
Position: 1
Default value: None
Accept pipeline input: True (ByPropertyName, ByValue)
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

## <a name="get-adsynctoolsrunhistory"></a>Get-ADSyncToolsRunHistory

### <a name="synopsis"></a>SINOPSIS
Obtener historial de ejecución de AAD Connect

### <a name="syntax"></a>SINTAXIS

```
Get-ADSyncToolsRunHistory [[-Days] <Int32>] [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
Función que devuelve el historial de ejecución de AAD Connect en formato XML

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
Get-ADSyncToolsRunHistory
```

#### <a name="example-2"></a>EJEMPLO 2
```
Get-ADSyncToolsRunHistory -Days 1
```

### <a name="parameters"></a>PARAMETERS

#### <a name="-days"></a>-Days
{{Indicar descripción de días}}

```yaml
Type: Int32
Parameter Sets: (All)
Aliases:

Required: False
Position: 1
Default value: 3
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

## <a name="get-adsynctoolssourceanchorchanged"></a>Get-ADSyncToolsSourceAnchorChanged

### <a name="synopsis"></a>SINOPSIS
Obtener usuarios con errores de SourceAnchor cambiado

### <a name="syntax"></a>SINTAXIS

```
Get-ADSyncToolsSourceAnchorChanged [-sourcePath] <Object> [-outputPath] <Object> [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
Función que consulta el historial de ejecución de AAD Connect y exporta todos los usuarios que notifican un error que indica que el atributo SourceAnchor ha cambiado.

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
#Required Parameters
```

$sourcePath = Read-Host -Prompt "Escriba la ruta de acceso del archivo de registro con el nombre de archivo" #"\<Source_Path\>" $outputPath = Read-Host -Prompt "Escriba la ruta de acceso del archivo de salida con el nombre de archivo" #"\<Out_Path\>"
 
 Get-ADSyncToolsUsersSourceAnchorChanged -sourcePath $sourcePath -outputPath $outputPath

#### <a name="example-2"></a>EJEMPLO 2
```
Another example of how to use this cmdlet
```

### <a name="parameters"></a>PARAMETERS

#### <a name="-sourcepath"></a>-sourcePath
{{Indicar descripción de sourcePath}}

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: True
Position: 1
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-outputpath"></a>-outputPath
{{Indicar la descripción de outputPath}}

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: True
Position: 2
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

## <a name="import-adsynctoolsimmutableidmigration"></a>Import-ADSyncToolsImmutableIdMigration

### <a name="synopsis"></a>SINOPSIS
Importar ImmutableID de AAD

### <a name="syntax"></a>SINTAXIS

```
Import-ADSyncToolsImmutableIdMigration [-Output] <String> [-IncludeSyncUsersFromRecycleBin]
 [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
Genera un archivo con todos los usuarios sincronizados de Azure AD que contiene el valor de ImmutableID en los requisitos de formato GUID: módulo de PowerShell MSOnline

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
Import-ADSyncToolsImmutableIdMigration -OutputFile '.\AllSyncUsers.csv'
```

#### <a name="example-2"></a>EJEMPLO 2
```
Another example of how to use this cmdlet
```

### <a name="parameters"></a>PARAMETERS

#### <a name="-output"></a>-Output
Archivo CSV de salida

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: True
Position: 1
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-includesyncusersfromrecyclebin"></a>-IncludeSyncUsersFromRecycleBin
Obtener usuarios sincronizados de la papelera de reciclaje de Azure AD

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: False
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).


## <a name="invoke-adsyncdatabasequery"></a>Invoke-AdSyncDatabaseQuery

### <a name="synopsis"></a>SINOPSIS
{{Indicar sinopsis}}

### <a name="syntax"></a>SINTAXIS

```
Invoke-AdSyncDatabaseQuery [-SqlConnection] <SqlConnection> [[-Query] <String>] [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
{{Indicar descripción}}

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>Ejemplo 1
```powershell
PS C:\> {{ Add example code here }}
```

{{Agregar descripción de ejemplo aquí}}

### <a name="parameters"></a>PARAMETERS

#### <a name="-query"></a>-Query
{{Indicar descripción de la consulta}}

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 1
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-sqlconnection"></a>-SqlConnection
{{Indicar descripción de SqlConnection}}

```yaml
Type: SqlConnection
Parameter Sets: (All)
Aliases:

Required: True
Position: 0
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

## <a name="remove-adsynctoolsexpiredcertificates"></a>Remove-ADSyncToolsExpiredCertificates

### <a name="synopsis"></a>SINOPSIS
Script para quitar los certificados caducados del atributo UserCertificate

### <a name="syntax"></a>SINTAXIS

```
Remove-ADSyncToolsExpiredCertificates [-TargetOU] <String> [[-BackupOnly] <Boolean>] [-ObjectClass] <String>
 [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
Este script toma todos los objetos de un unidad organizativa de destino en su dominio de Active Directory, filtrado por clase de objeto (usuario o equipo), y elimina todos los certificados caducados presentes en el atributo UserCertificate.
De forma predeterminada (modo BackupOnly) solo realizará la copia de seguridad de los certificados caducados en un archivo y no realizará ningún cambio en AD.
Si usa -BackupOnly $false, cualquier certificado caducado presente en el atributo UserCertificate para estos objetos se quitará de AD después de copiarse en el archivo.
La copia de seguridad de cada certificado se realizará con un nombre de archivo diferente: ObjectClass_ObjectGUID_CertThumprint.cer. El script también creará un archivo de registro en formato CSV que mostrará todos los usuarios con certificados válidos o caducados, incluida la acción real realizada (Omitido/Exportado/Eliminado).

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
Check all users in target OU - Expired Certificates will be copied to separated files and no certificates will be removed
```

Remove-ADSyncToolsExpiredCertificates -TargetOU "OU=Users,OU=Corp,DC=Contoso,DC=com" -ObjectClass user

#### <a name="example-2"></a>EJEMPLO 2
```
Delete Expired Certs from all Computer objects in target OU - Expired Certificates will be copied to files and removed from AD
```

Remove-ADSyncToolsExpiredCertificates -TargetOU "OU=Computers,OU=Corp,DC=Contoso,DC=com" -ObjectClass computer -BackupOnly $false

### <a name="parameters"></a>PARAMETERS

#### <a name="-targetou"></a>-TargetOU
Unidad organizativa de destino para objetos de AD

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: True
Position: 1
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-backuponly"></a>-BackupOnly
BackupOnly no eliminará ningún certificado de AD

```yaml
Type: Boolean
Parameter Sets: (All)
Aliases:

Required: False
Position: 2
Default value: True
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-objectclass"></a>-ObjectClass
Filtro de clases de objetos

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: True
Position: 3
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

## <a name="repair-adsynctoolsautoupgradestate"></a>Repair-ADSyncToolsAutoUpgradeState

### <a name="synopsis"></a>SINOPSIS
Descripción breve

### <a name="syntax"></a>SINTAXIS

```
Repair-ADSyncToolsAutoUpgradeState
```

### <a name="description"></a>DESCRIPTION
Descripción larga

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
Example of how to use this cmdlet
```

#### <a name="example-2"></a>EJEMPLO 2
```
Another example of how to use this cmdlet
```

## <a name="resolve-adsynchostaddress"></a>Resolve-ADSyncHostAddress

### <a name="synopsis"></a>SINOPSIS
{{Indicar sinopsis}}

### <a name="syntax"></a>SINTAXIS

```
Resolve-ADSyncHostAddress [[-hostName] <String>]
```

### <a name="description"></a>DESCRIPTION
{{Indicar descripción}}

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>Ejemplo 1
```powershell
PS C:\> {{ Add example code here }}
```

{{Agregar descripción de ejemplo aquí}}

### <a name="parameters"></a>PARAMETERS

#### <a name="-hostname"></a>-hostName
{{Indicar descripción del nombre de host}}

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 0
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

## <a name="restore-adsynctoolsexpiredcertificates"></a>Restore-ADSyncToolsExpiredCertificates

### <a name="synopsis"></a>SINOPSIS
(TAREA PENDIENTE) Restaura el atributo UserCertificate de AD de un archivo de certificado

### <a name="syntax"></a>SINTAXIS

```
Restore-ADSyncToolsExpiredCertificates
```

### <a name="description"></a>DESCRIPTION
Descripción larga

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
Example of how to use this cmdlet
```

#### <a name="example-2"></a>EJEMPLO 2
```
Another example of how to use this cmdlet
```

## <a name="set-adsynctoolsconsistencyguid"></a>Set-ADSyncToolsConsistencyGuid

### <a name="synopsis"></a>SINOPSIS
Establecer mS-Ds-ConsistencyGuid en el usuario de AD

### <a name="syntax"></a>SINTAXIS

```
Set-ADSyncToolsConsistencyGuid [-User] <Object> [-Value] <Object> [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
Establecer un valor en el atributo mS-Ds-ConsistencyGuid para el usuario de AD de destino

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
Example of how to use this cmdlet
```

#### <a name="example-2"></a>EJEMPLO 2
```
Another example of how to use this cmdlet
```

### <a name="parameters"></a>PARAMETERS

#### <a name="-user"></a>-User
Usuario de destino de AD para establecer ConsistencyGuid

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: True
Position: 1
Default value: None
Accept pipeline input: True (ByPropertyName, ByValue)
Accept wildcard characters: False
```

#### <a name="-value"></a>-Value
ImmutableId (matriz de bytes, GUID, cadena GUID o cadena en Base64)

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: True
Position: 2
Default value: None
Accept pipeline input: True (ByPropertyName, ByValue)
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

## <a name="test-adsyncnetworkport"></a>Test-ADSyncNetworkPort

### <a name="synopsis"></a>SINOPSIS
{{Indicar sinopsis}}

### <a name="syntax"></a>SINTAXIS

```
Test-ADSyncNetworkPort [[-hostName] <String>] [[-port] <String>]
```

### <a name="description"></a>DESCRIPTION
{{Indicar descripción}}

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>Ejemplo 1
```powershell
PS C:\> {{ Add example code here }}
```

{{Agregar descripción de ejemplo aquí}}

### <a name="parameters"></a>PARAMETERS

#### <a name="-hostname"></a>-hostName
{{Indicar descripción del nombre de host}}

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 0
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-port"></a>-port
{{Indicar descripción del puerto}}

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 1
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

## <a name="trace-adsynctoolsadimport"></a>Trace-ADSyncToolsADImport

### <a name="synopsis"></a>SINOPSIS
Crea un archivo de seguimiento desde un paso de importación de AD

### <a name="syntax"></a>SINTAXIS

```
Trace-ADSyncToolsADImport [[-ADConnectorXML] <String>] [[-dc] <String>] [[-rootDN] <String>]
 [[-filter] <String>] [-SkipCredentials] [[-ADwatermark] <String>] [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
Realiza un seguimiento de todas las consultas LDAP de una importación de AD de AAD Connect ejecutada desde un punto de control de marca de agua de AD determinado (cookie de partición). Crea un archivo de seguimiento “.\ADimportTrace_aaaaMMddHHmmss.log” en la carpeta actual.

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
Example of how to use this cmdlet
```

#### <a name="example-2"></a>EJEMPLO 2
```
Another example of how to use this cmdlet
```

### <a name="parameters"></a>PARAMETERS

#### <a name="-adconnectorxml"></a>-ADConnectorXML
{{Indicar descripción de ADConnectorXML}}

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 1
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-dc"></a>-dc
Archivo XML de exportación de conector AD

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 2
Default value: DC1.domain.contoso.com
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-rootdn"></a>-rootDN
Controlador de dominio de destino

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 3
Default value: DC=Domain,DC=Contoso,DC=com
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-filter"></a>-filter
DN raíz del bosque

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 4
Default value: (&(objectClass=*))
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-skipcredentials"></a>-SkipCredentials
Tipos de objetos de AD de los que se realizará un seguimiento \> * = todos los tipos de objeto

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: False
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-adwatermark"></a>-ADwatermark
Si ya opera como administrador de dominio, no es necesario proporcionar las credenciales de AD.
La entrada manual de la marca de agua, en lugar del archivo XML como por ejemplo $ADwatermark = "TVNEUwMAAAAXyK9ir1zSAQAAAAAAAAAA(...)"

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 5
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

## <a name="trace-adsynctoolsldapquery"></a>Trace-ADSyncToolsLdapQuery

### <a name="synopsis"></a>SINOPSIS
Descripción breve

### <a name="syntax"></a>SINTAXIS

```
Trace-ADSyncToolsLdapQuery [-Context] <String> [-Server] <String> [-Port] <Int32> [-Filter] <String>
 [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
Descripción larga

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
Example of how to use this cmdlet
```

#### <a name="example-2"></a>EJEMPLO 2
```
Another example of how to use this cmdlet
```

### <a name="parameters"></a>PARAMETERS

#### <a name="-context"></a>-Context
Descripción de ayuda de param1

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: True
Position: 1
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-server"></a>-Server
Descripción de ayuda de param2

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: True
Position: 2
Default value: Localhost
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-port"></a>-Port
Descripción de ayuda de param2

```yaml
Type: Int32
Parameter Sets: (All)
Aliases:

Required: True
Position: 3
Default value: 389
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-filter"></a>-Filter
Descripción de ayuda de param2

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: True
Position: 4
Default value: (objectClass=*)
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

## <a name="update-adsynctoolsconsistencyguidmigration"></a>Update-ADSyncToolsConsistencyGuidMigration

### <a name="synopsis"></a>SINOPSIS
Actualiza los usuarios con el nuevo ConsistencyGuid (ImmutableId)

### <a name="syntax"></a>SINTAXIS

```
Update-ADSyncToolsConsistencyGuidMigration [[-DistinguishedName] <String>] [-ImmutableIdGUID] <String>
 [-Action] <String> [-Output] <String> [-WhatIf] [-Confirm] [<CommonParameters>]
```

### <a name="description"></a>DESCRIPTION
Actualiza los usuarios con el nuevo valor de ConsistencyGuid (ImmutableId) tomado del informe ConsistencyGuid. Esta función admite el modificador WhatIf. Nota: el informe ConsistencyGuid debe importarse con el delimitador de tabulador

### <a name="examples"></a>EJEMPLOS

#### <a name="example-1"></a>EJEMPLO 1
```
Import-Csv .\AllSyncUsersTEST-Report.csv -Delimiter "`t"| Update-ADSyncToolsConsistencyGuidMigration -Output .\AllSyncUsersTEST-Result2 -WhatIf
```

#### <a name="example-2"></a>EJEMPLO 2
```
Import-Csv .\AllSyncUsersTEST-Report.csv -Delimiter "`t"| Update-ADSyncToolsConsistencyGuidMigration -Output .\AllSyncUsersTEST-Result2
```

### <a name="parameters"></a>PARAMETERS

#### <a name="-distinguishedname"></a>-DistinguishedName
DistinguishedName

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: False
Position: 1
Default value: False
Accept pipeline input: True (ByPropertyName, ByValue)
Accept wildcard characters: False
```

#### <a name="-immutableidguid"></a>-ImmutableIdGUID
ImmutableIdGUID

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: True
Position: 2
Default value: None
Accept pipeline input: True (ByPropertyName, ByValue)
Accept wildcard characters: False
```

#### <a name="-action"></a>-Action
Acción

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: True
Position: 3
Default value: None
Accept pipeline input: True (ByPropertyName, ByValue)
Accept wildcard characters: False
```

#### <a name="-output"></a>-Output
Nombre de archivo de salida para archivos LOG

```yaml
Type: String
Parameter Sets: (All)
Aliases:

Required: True
Position: 4
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-whatif"></a>-WhatIf
Muestra lo que sucedería si se ejecutara el cmdlet.
El cmdlet no se ejecuta.

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases: wi

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="-confirm"></a>-Confirm
Le solicita su confirmación antes de ejecutar el cmdlet.

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases: cf

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

#### <a name="commonparameters"></a>CommonParameters
Este cmdlet admite los parámetros comunes siguientes: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction y -WarningVariable.
Para más información, consulte about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).
