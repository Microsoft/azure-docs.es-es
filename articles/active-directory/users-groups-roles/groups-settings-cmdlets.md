---
title: Configurar opciones de grupo mediante PowerShell Azure AD | Microsoft Docs
description: Administración de la configuración de grupos mediante cmdlets de Azure Active Directory
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: how-to
ms.date: 08/13/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 63f0c55823899be8eb4146860787aede2cd2d6b5
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88797911"
---
# <a name="azure-active-directory-cmdlets-for-configuring-group-settings"></a>Cmdlets de Azure Active Directory para configurar las opciones de grupo

Este artículo contiene instrucciones para usar cmdlets de PowerShell de Azure Active Directory (Azure AD) para crear y actualizar grupos. Este contenido se aplica únicamente a grupos de Microsoft 365, también conocidos como grupos unificados.

> [!IMPORTANT]
> Algunas configuraciones requieren una licencia de Azure Active Directory Premium P1. Para más información, consulte la tabla [Configuración de plantillas](#template-settings).

Para más información sobre cómo evitar que los usuarios no administradores creen grupos de seguridad, establezca  `Set-MsolCompanySettings -UsersPermissionToCreateGroupsEnabled $False` como se describe en [Set-MSOLCompanySettings](/powershell/module/msonline/set-msolcompanysettings?view=azureadps-1.0).

Los grupos de Microsoft 365 se configuran mediante un objeto Settings y un objeto SettingsTemplate. Al principio no ve ningún objeto de configuración en el directorio porque este se ha configurado de forma predeterminada. Para cambiar la configuración predeterminada, debe crear un nuevo objeto Settings utilizando una plantilla SettingsTemplate. Las plantillas de configuración las define Microsoft. Hay varias plantillas de configuración diferentes. Para configurar los valores del grupo de Microsoft 365 para su directorio, se utiliza la plantilla denominada "Group.Unified". Para configurar los valores del grupo de Microsoft 365 en un único grupo, utilice la plantilla denominada "Group.Unified.Guest". Esta plantilla se usa para administrar el acceso de invitado a un grupo de Microsoft 365. 

Los cmdlets forman parte del módulo Azure Active Directory PowerShell V2. Para obtener instrucciones sobre cómo descargar e instalar el módulo en el equipo, consulte el artículo [Azure Active Directory PowerShell versión 2](/powershell/azure/active-directory/overview). Puede instalar la versión 2 del módulo desde [la Galería de PowerShell](https://www.powershellgallery.com/packages/AzureAD/).

## <a name="install-powershell-cmdlets"></a>Instalación de los cmdlets de PowerShell

Asegúrese de desinstalar cualquier versión anterior del módulo Azure Active Directory PowerShell for Graph para Windows PowerShell y de instalar [Azure Active Directory PowerShell for Graph - Public Preview Release 2.0.0.137](https://www.powershellgallery.com/packages/AzureADPreview) antes de ejecutar los comandos de PowerShell.

1. Abra la aplicación Windows PowerShell como administrador.
2. Desinstale cualquier versión anterior de AzureADPreview.
  
   ``` PowerShell
   Uninstall-Module AzureADPreview
   Uninstall-Module azuread
   ```

3. Instale la versión más reciente de AzureADPreview.
  
   ``` PowerShell
   Install-Module AzureADPreview
   ```
   
## <a name="create-settings-at-the-directory-level"></a>Creación de una configuración en el nivel de directorio
Con estos pasos se crean configuraciones en el nivel de directorio que se aplican a todos los grupos de Microsoft 365 del directorio. El cmdlet Get-AzureADDirectorySettingTemplate solo está disponible en el [módulo de versión preliminar de Azure AD PowerShell for Graph](https://www.powershellgallery.com/packages/AzureADPreview).

1. En los cmdlets DirectorySettings, debe especificar el identificador de la plantilla SettingsTemplate que desea usar. Si no conoce este identificador, este cmdlet devuelve la lista de plantillas de configuración:
  
   ```powershell
   Get-AzureADDirectorySettingTemplate

   ```
   Esta llamada al cmdlet devuelve todas las plantillas que están disponibles:
  
   ``` PowerShell
   Id                                   DisplayName         Description
   --                                   -----------         -----------
   62375ab9-6b52-47ed-826b-58e47e0e304b Group.Unified       ...
   08d542b9-071f-4e16-94b0-74abb372e3d9 Group.Unified.Guest Settings for a specific Microsoft 365 group
   16933506-8a8d-4f0d-ad58-e1db05a5b929 Company.BuiltIn     Setting templates define the different settings that can be used for the associ...
   4bc7f740-180e-4586-adb6-38b2e9024e6b Application...
   898f1161-d651-43d1-805c-3b0b388a9fc2 Custom Policy       Settings ...
   5cf42378-d67d-4f36-ba46-e8b86229381d Password Rule       Settings ...
   ```
2. Para agregar una dirección URL de instrucciones de uso, primero debe obtener el objeto SettingsTemplate que define el valor pertinente, es decir, la plantilla Group.Unified:
  
   ```powershell
   $TemplateId = (Get-AzureADDirectorySettingTemplate | where { $_.DisplayName -eq "Group.Unified" }).Id
   $Template = Get-AzureADDirectorySettingTemplate | where -Property Id -Value $TemplateId -EQ
   ```
3. A continuación, cree un nuevo objeto de configuración basado en esa plantilla:
  
   ```powershell
   $Setting = $Template.CreateDirectorySetting()
   ```  
4. Luego, actualice el valor de instrucciones de uso:
  
   ```powershell
   $Setting["UsageGuidelinesUrl"] = "https://guideline.example.com"
   ```  
5. A continuación, aplique la configuración:
  
   ```powershell
   New-AzureADDirectorySetting -DirectorySetting $Setting
   ```
6. Puede leer los valores mediante:

   ```powershell
   $Setting.Values
   ```
   
## <a name="update-settings-at-the-directory-level"></a>Actualización de la configuración en el nivel de directorio
Para actualizar el valor de UsageGuideLinesUrl en la plantilla de configuración, lea la configuración actual de Azure AD; de lo contrario, podríamos acabar sobrescribiendo una configuración existente distinta de UsageGuideLinesUrl.

1. Obtenga la configuración actual del objeto SettingsTemplate de Group.Unified:
   
   ```powershell
   $Setting = Get-AzureADDirectorySetting | ? { $_.DisplayName -eq "Group.Unified"}
   ```  
2. Comprobación de la configuración actual:
   
   ```powershell
   $Setting.Values
   ```
   
   Salida:
   ```powershell
    Name                          Value
    ----                          -----
    EnableMIPLabels               false
    CustomBlockedWordsList
    EnableMSStandardBlockedWords  False
    ClassificationDescriptions
    DefaultClassification
    PrefixSuffixNamingRequirement
    AllowGuestsToBeGroupOwner     False
    AllowGuestsToAccessGroups     True
    GuestUsageGuidelinesUrl
    GroupCreationAllowedGroupId
    AllowToAddGuests              True
    UsageGuidelinesUrl            https://guideline.example.com
    ClassificationList
    EnableGroupCreation           True
    ```
3. Para quitar el valor de UsageGuideLinesUrl, edite la dirección URL de modo que sea una cadena vacía:
   
   ```powershell
   $Setting["UsageGuidelinesUrl"] = ""
   ```  
4. Guarde la actualización en el directorio:
   
   ```powershell
   Set-AzureADDirectorySetting -Id $Setting.Id -DirectorySetting $Setting
   ```  

## <a name="template-settings"></a>Configuración de plantillas
Esta es la configuración definida en el objeto SettingsTemplate Group.Unified. Salvo que se indique lo contrario, estas características requieren una licencia Azure Active Directory Premium P1. 

| **Configuración** | **Descripción** |
| --- | --- |
|  <ul><li>EnableGroupCreation<li>Escriba:  Boolean<li>Valor predeterminado: True |Marca que indica si se permite la creación de grupos de Microsoft 365 en el directorio por parte de usuarios que no sean administradores. Este valor no requiere una licencia Azure Active Directory Premium P1.|
|  <ul><li>GroupCreationAllowedGroupId<li>Escriba:  String<li>Default: "" |El GUID del grupo de seguridad para el que se permite a los miembros crear grupos de Microsoft 365 incluso cuando EnableGroupCreation == false. |
|  <ul><li>UsageGuidelinesUrl<li>Escriba:  String<li>Default: "" |Un vínculo a las instrucciones de uso de grupos. |
|  <ul><li>ClassificationDescriptions<li>Escriba:  String<li>Default: "" | Una lista delimitada por comas de las descripciones de clasificación. El valor de ClassificationDescriptions solo es válido en este formato:<br>$setting["ClassificationDescriptions"] ="Classification:Description,Classification:Description"<br>donde el valor de Classification coincide con una entrada en ClassificationList.<br>Esta configuración no se aplica cuando EnableMIPLabels = = True.|
|  <ul><li>DefaultClassification<li>Escriba:  String<li>Default: "" | La clasificación que se va a utilizar como la clasificación predeterminada para un grupo si no se ha especificado ninguno.<br>Esta configuración no se aplica cuando EnableMIPLabels = = True.|
|  <ul><li>PrefixSuffixNamingRequirement<li>Escriba:  String<li>Default: "" | Cadena de una longitud máxima de 64 caracteres que define la convención de nomenclatura configurada para los grupos de Microsoft 365. Para obtener más información, vea [Aplicación de una directiva de nomenclatura en los grupos de Microsoft 365](groups-naming-policy.md). |
| <ul><li>CustomBlockedWordsList<li>Escriba:  String<li>Default: "" | Cadena de frases separadas por comas que los usuarios no tendrán permiso para usar en los nombres de grupos o alias. Para obtener más información, vea [Aplicación de una directiva de nomenclatura en los grupos de Microsoft 365](groups-naming-policy.md). |
| <ul><li>EnableMSStandardBlockedWords<li>Escriba:  Boolean<li>Valor predeterminado: "False" | No debe usarse
|  <ul><li>AllowGuestsToBeGroupOwner<li>Escriba:  Boolean<li>Valor predeterminado: False | Valor booleano que indica si un usuario invitado puede ser o no un propietario de grupos. |
|  <ul><li>AllowGuestsToAccessGroups<li>Escriba:  Boolean<li>Valor predeterminado: True | Valor booleano que indica si un usuario invitado puede tener o no acceso al contenido de grupos de Microsoft 365.  Este valor no requiere una licencia Azure Active Directory Premium P1.|
|  <ul><li>GuestUsageGuidelinesUrl<li>Escriba:  String<li>Default: "" | La dirección URL de un vínculo a las instrucciones de uso del invitado. |
|  <ul><li>AllowToAddGuests<li>Escriba:  Boolean<li>Valor predeterminado: True | Un valor booleano que indica si está permitido o no agregar invitados a este directorio. <br>Esta configuración puede invalidarse y convertirse en solo lectura si *EnableMIPLabels* está establecida en *True* y una directiva de invitado está asociada a la etiqueta de confidencialidad asignada al grupo.<br>Si el valor de AllowToAddGuests se establece en False en el nivel de inquilino, se omite cualquier valor de AllowToAddGuests en el nivel de grupo. Si quiere habilitar el acceso de invitado solo para algunos grupos, debe establecer AllowToAddGuests en true en el nivel de inquilino y, luego, deshabilitarlo de forma selectiva para grupos específicos. |
|  <ul><li>ClassificationList<li>Escriba:  String<li>Default: "" | Lista de valores de clasificación válidos delimitados por comas que se puede aplicar a grupos de Microsoft 365. <br>Esta configuración no se aplica cuando EnableMIPLabels = = True.|
|  <ul><li>EnableMIPLabels<li>Escriba:  Boolean<li>Valor predeterminado: "False" |Marca que indica si las etiquetas de confidencialidad publicadas en el Centro de cumplimiento de Microsoft 365 se pueden aplicar a los grupos de Microsoft 365. Para más información, vea [Asignación de etiquetas de confidencialidad para los grupos de Microsoft 365](groups-assign-sensitivity-labels.md). |

## <a name="example-configure-guest-policy-for-groups-at-the-directory-level"></a>Ejemplo: Configuración de la directiva de invitado para grupos en el nivel de directorio
1. Obtenga todas las plantillas de configuración:
   ```powershell
   Get-AzureADDirectorySettingTemplate
   ```
2. Para establecer la directiva de invitado para los grupos en el nivel de directorio, necesita la plantilla Group.Unified.
   ```powershell
   $Template = Get-AzureADDirectorySettingTemplate | where -Property Id -Value "62375ab9-6b52-47ed-826b-58e47e0e304b" -EQ
   ```
3. A continuación, cree un nuevo objeto de configuración basado en esa plantilla:
  
   ```powershell
   $Setting = $template.CreateDirectorySetting()
   ```  
4. Después, actualice la configuración de AllowToAddGuests.
   ```powershell
   $Setting["AllowToAddGuests"] = $False
   ```  
5. A continuación, aplique la configuración:
  
   ```powershell
   Set-AzureADDirectorySetting -Id (Get-AzureADDirectorySetting | where -Property DisplayName -Value "Group.Unified" -EQ).id -DirectorySetting $Setting
   ```
6. Puede leer los valores mediante:

   ```powershell
   $Setting.Values
   ```   

## <a name="read-settings-at-the-directory-level"></a>Lectura de la configuración en el nivel de directorio

Si conoce el nombre de la configuración que desea recuperar, puede usar el siguiente cmdlet para recuperar el valor de configuración actual. En este ejemplo, se recuperará el valor de una configuración denominada "UsageGuidelinesUrl". 

   ```powershell
   (Get-AzureADDirectorySetting).Values | Where-Object -Property Name -Value UsageGuidelinesUrl -EQ
   ```
Con estos pasos se lee la configuración en el nivel de directorio, la cual se aplica a todos los grupos de Office del directorio.

1. Lea toda la configuración de directorio existente:
   ```powershell
   Get-AzureADDirectorySetting -All $True
   ```
   Este cmdlet devuelve la lista de las opciones de configuración del directorio:
   ```powershell
   Id                                   DisplayName   TemplateId                           Values
   --                                   -----------   ----------                           ------
   c391b57d-5783-4c53-9236-cefb5c6ef323 Group.Unified 62375ab9-6b52-47ed-826b-58e47e0e304b {class SettingValue {...
   ```

2. Lea toda la configuración de un grupo específico:
   ```powershell
   Get-AzureADObjectSetting -TargetObjectId ab6a3887-776a-4db7-9da4-ea2b0d63c504 -TargetType Groups
   ```

3. Lea todos los valores de configuración de directorio de un objeto de configuración de directorio específico, con el GUID de identificador de configuración:
   ```powershell
   (Get-AzureADDirectorySetting -Id c391b57d-5783-4c53-9236-cefb5c6ef323).values
   ```
   Este cmdlet devuelve los nombres y valores de este objeto de configuración para este grupo en particular:
   ```powershell
   Name                          Value
   ----                          -----
   ClassificationDescriptions
   DefaultClassification
   PrefixSuffixNamingRequirement
   CustomBlockedWordsList        
   AllowGuestsToBeGroupOwner     False 
   AllowGuestsToAccessGroups     True
   GuestUsageGuidelinesUrl
   GroupCreationAllowedGroupId
   AllowToAddGuests              True
   UsageGuidelinesUrl            https://guideline.example.com
   ClassificationList
   EnableGroupCreation           True
   ```

## <a name="remove-settings-at-the-directory-level"></a>Eliminación de la configuración en el nivel de directorio
Con estos pasos se elimina la configuración en el nivel de directorio, lo cual se aplica a todos los grupos de Office del directorio.
   ```powershell
   Remove-AzureADDirectorySetting –Id c391b57d-5783-4c53-9236-cefb5c6ef323c
   ```

## <a name="create-settings-for-a-specific-group"></a>Creación de la configuración de un grupo específico

1. Busque la plantilla de configuración denominada "Groups.Unified.Guest"
   ```powershell
   Get-AzureADDirectorySettingTemplate
  
   Id                                   DisplayName            Description
   --                                   -----------            -----------
   62375ab9-6b52-47ed-826b-58e47e0e304b Group.Unified          ...
   08d542b9-071f-4e16-94b0-74abb372e3d9 Group.Unified.Guest    Settings for a specific Microsoft 365 group
   4bc7f740-180e-4586-adb6-38b2e9024e6b Application            ...
   898f1161-d651-43d1-805c-3b0b388a9fc2 Custom Policy Settings ...
   5cf42378-d67d-4f36-ba46-e8b86229381d Password Rule Settings ...
   ```
2. Recupere el objeto de plantilla para la plantilla Groups.Unified.Guest:
   ```powershell
   $Template1 = Get-AzureADDirectorySettingTemplate | where -Property Id -Value "08d542b9-071f-4e16-94b0-74abb372e3d9" -EQ
   ```
3. Cree un nuevo objeto de configuración a partir de la plantilla:
   ```powershell
   $SettingCopy = $Template1.CreateDirectorySetting()
   ```

4. Establezca la configuración al valor requerido:
   ```powershell
   $SettingCopy["AllowToAddGuests"]=$False
   ```
5. Obtenga el identificador del grupo al que desea aplicar esta configuración:
   ```powershell
   $groupID= (Get-AzureADGroup -SearchString "YourGroupName").ObjectId
   ```
6. Cree la nueva configuración para el grupo necesario en el directorio:
   ```powershell
   New-AzureADObjectSetting -TargetType Groups -TargetObjectId $groupID -DirectorySetting $SettingCopy
   ```
7. Para comprobar la configuración, ejecute este comando:
   ```powershell
   Get-AzureADObjectSetting -TargetObjectId $groupID -TargetType Groups | fl Values
   ```

## <a name="update-settings-for-a-specific-group"></a>Actualización de la configuración de un grupo específico
1. Obtenga el identificador del grupo cuya configuración desea actualizar:
   ```powershell
   $groupID= (Get-AzureADGroup -SearchString "YourGroupName").ObjectId
   ```
2. Recupere la configuración del grupo:
   ```powershell
   $Setting = Get-AzureADObjectSetting -TargetObjectId $groupID -TargetType Groups
   ```
3. Actualice la configuración del grupo como sea necesario, por ejemplo:
   ```powershell
   $Setting["AllowToAddGuests"] = $True
   ```
4. A continuación, obtenga el identificador de la configuración para este grupo específico:
   ```powershell
   Get-AzureADObjectSetting -TargetObjectId $groupID -TargetType Groups
   ```
   Obtendrá una respuesta similar a esta:
   ```powershell
   Id                                   DisplayName            TemplateId                             Values
   --                                   -----------            -----------                            ----------
   2dbee4ca-c3b6-4f0d-9610-d15569639e1a Group.Unified.Guest    08d542b9-071f-4e16-94b0-74abb372e3d9   {class SettingValue {...
   ```
5. A continuación, puede establecer el nuevo valor para esta configuración:
   ```powershell
   Set-AzureADObjectSetting -TargetType Groups -TargetObjectId $groupID -Id 2dbee4ca-c3b6-4f0d-9610-d15569639e1a -DirectorySetting $Setting
   ```
6. Puede leer el valor de la configuración para asegurarse de que se ha actualizado correctamente:
   ```powershell
   Get-AzureADObjectSetting -TargetObjectId $groupID -TargetType Groups | fl Values
   ```

## <a name="cmdlet-syntax-reference"></a>Referencia de sintaxis de cmdlet
Puede encontrar más documentación de Azure Active Directory PowerShell en el artículo sobre los [cmdlets de Azure Active Directory](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0).

## <a name="additional-reading"></a>Lecturas adicionales

* [Administración del acceso a los recursos con grupos de Azure Active Directory](../fundamentals/active-directory-manage-groups.md)
* [Integración de las identidades locales con Azure Active Directory](../hybrid/whatis-hybrid-identity.md)