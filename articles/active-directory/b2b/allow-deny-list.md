---
title: Permitir o bloquear invita a organizaciones específicas - Azure Active Directory | Microsoft Docs
description: Se muestra cómo un administrador puede usar Azure Portal o PowerShell para definir una lista de acceso o denegación a fin de permitir o bloquear a los usuarios B2B de determinados dominios.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 04/19/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: sasubram
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: b302ec0265473e09b3960660b10661faa1960442
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65812964"
---
# <a name="allow-or-block-invitations-to-b2b-users-from-specific-organizations"></a>Permitir o bloquear invitaciones a usuarios B2B procedentes de determinadas organizaciones

Puede usar una lista de permitidos o de denegación para permitir o bloquear las invitaciones a usuarios B2B procedentes de determinadas organizaciones. Por ejemplo, si quiere bloquear dominios de direcciones de correo electrónico personales, puede configurar una lista de denegación que contenga dominios como Gmail.com y Outlook.com. O bien, si su empresa tiene una asociación con otras empresas como Contoso.com, Fabrikam.com y Litware.com, y quiere restringir las invitaciones a solo estas organizaciones, puede agregar Contoso.com, Fabrikam.com y Litware.com a la lista de permitidos.
  
## <a name="important-considerations"></a>Consideraciones importantes

- Puede crear una lista de permitidos o una lista de denegación. No se pueden configurar ambos tipos de listas. De forma predeterminada, los dominios que no están en la lista de permitidos están en la de denegación y viceversa. 
- Solo puede crear una directiva por organización. Puede actualizar la directiva para incluir más dominios, o puede eliminar la directiva para crear una nueva. 
- Esta lista funciona con independencia de las listas de permitidos o bloqueados de OneDrive para la Empresa y SharePoint Online. Si quiere restringir el uso compartido individual de archivos en SharePoint Online, debe configurar una lista de permitidos o bloqueados para OneDrive para la Empresa y SharePoint Online. Para más información, consulte [Uso compartido de dominios restringidos en SharePoint Online y OneDrive para la Empresa](https://support.office.com/article/restricted-domains-sharing-in-sharepoint-online-and-onedrive-for-business-5d7589cd-0997-4a00-a2ba-2320ec49c4e9).
- Esta lista no se aplica a usuarios externos que ya han canjeado la invitación. La lista se aplicará después de configurarla. Si una invitación de usuario está en estado pendiente y se define una directiva que bloquea su dominio, el intento del usuario para canjear la invitación producirá error.

## <a name="set-the-allow-or-deny-list-policy-in-the-portal"></a>Definición de la directiva de lista de permitidos o de denegación en el portal

De forma predeterminada, la opción **Allow invitations to be sent to any domain (most inclusive)** (Permitir que se envíen invitaciones a cualquier dominio [más inclusivo]) está habilitada. En este caso, puede invitar a usuarios B2B de cualquier organización.

### <a name="add-a-deny-list"></a>Adición de una lista de denegación

Este es el escenario más típico, donde su organización quiere trabajar con casi cualquier organización, pero desea impedir que los usuarios de dominios específicos sean invitados como usuarios B2B.

Para agregar una lista de denegación:

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).
2. Seleccione **Azure Active Directory** > **Usuarios** > **Configuración de usuario**.
3. En **Usuarios externos**, seleccione **Administrar la configuración de colaboración externa**.
4. En **Restricciones de colaboración**, seleccione **Deny invitations to the specified domains** (Denegar invitaciones a los dominios especificados).
5. En **DOMINIOS DE DESTINO**, escriba el nombre de uno de los dominios que quiere bloquear. Si hay varios dominios, especifique cada dominio en una nueva línea. Por ejemplo:

   ![Muestra la opción de denegación con los dominios agregados](./media/allow-deny-list/DenyListSettings.png)
 
6. Cuando haya terminado, haga clic en **Guardar**.

Después de establecer la directiva, si intenta invitar a un usuario de un dominio bloqueado, recibirá un mensaje que indica que la directiva actual de invitación bloquea el dominio del usuario.
 
### <a name="add-an-allow-list"></a>Adición de una lista de permitidos

Esta es una configuración más restrictiva, donde puede establecer dominios específicos en la lista de permitidos y restringir las invitaciones a las otras organizaciones o dominios que no se mencionan. 

Si desea usar una lista de permitidos, asegúrese de dedicar tiempo a evaluar exhaustivamente las necesidades de su empresa. Si hace esta directiva demasiado restrictiva, los usuarios pueden elegir enviar documentos por correo electrónico o buscar otras formas de colaboración no sancionadas por TI.


Para agregar una lista de permitidos:

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).
2. Seleccione **Azure Active Directory** > **Usuarios** > **Configuración de usuario**.
3. En **Usuarios externos**, seleccione **Administrar la configuración de colaboración externa**.
4. En **Restricciones de colaboración**, seleccione **Allow invitations only to the specified domains (most restrictive)** (Permitir invitaciones solo a los dominios especificados [más restrictivo]).
5. En **DOMINIOS DE DESTINO**, escriba el nombre de uno de los dominios que quiere permitir. Si hay varios dominios, especifique cada dominio en una nueva línea. Por ejemplo:

   ![Muestra la opción de permitir con los dominios agregados](./media/allow-deny-list/AllowListSettings.png)
 
6. Cuando haya terminado, haga clic en **Guardar**.

Después de establecer la directiva, si intenta invitar a un usuario de un dominio que no está en la lista de permitidos, recibirá un mensaje que indica que la directiva de invitación actual bloquea el dominio del usuario.

### <a name="switch-from-allow-to-deny-list-and-vice-versa"></a>Cambio de la lista de permitidos a la lista de denegación y viceversa 

Si cambia de una directiva a la otra, se descarta la configuración de directiva existente. Asegúrese de realizar una copia de seguridad de los detalles de la configuración antes de ejecutar el cambio. 

## <a name="set-the-allow-or-deny-list-policy-using-powershell"></a>Definición de la directiva de lista de permitidos o lista de denegación con PowerShell

### <a name="prerequisite"></a>Requisito previo

Para establecer la lista de permitidos o de denegación mediante PowerShell, debe instalar la versión preliminar del Módulo Azure Active Directory para Windows PowerShell. En concreto, instale la versión 2.0.0.98 o superior del módulo AzureADPreview.

Para comprobar la versión del módulo (y ver si está instalado):
 
1. Abra Windows PowerShell como usuario con privilegios elevados (Ejecutar como administrador). 
2. Ejecute el siguiente comando para ver si tiene alguna versión del Módulo Azure Active Directory para Windows PowerShell instalada en el equipo:

   ```powershell  
   Get-Module -ListAvailable AzureAD*
   ```

Si el módulo no está instalado o no tiene la versión adecuada, realice lo siguiente:

- Si no se devuelve ningún resultado, ejecute el siguiente comando para instalar la versión más reciente del módulo AzureADPreview:
  
   ```powershell  
   Install-Module AzureADPreview
   ```
- Si solo se muestra el módulo AzureAD en los resultados, ejecute los comandos siguientes para instalar el módulo AzureADPreview: 

   ```powershell 
   Uninstall-Module AzureAD 
   Install-Module AzureADPreview 
   ```
- Si solo se muestra el módulo AzureADPreview en los resultados, pero la versión es inferior a 2.0.0.98, ejecute los siguientes comandos para actualizarla: 

   ```powershell 
   Uninstall-Module AzureADPreview 
   Install-Module AzureADPreview 
   ```

- Si ambos módulos, AzureAD y AzureADPreview, se muestran en los resultados, pero la versión del módulo AzureADPreview es inferior a la 2.0.0.98, ejecute los siguientes comandos para actualizarla: 

   ```powershell 
   Uninstall-Module AzureAD 
   Uninstall-Module AzureADPreview 
   Install-Module AzureADPreview 
    ```

### <a name="use-the-azureadpolicy-cmdlets-to-configure-the-policy"></a>Uso de los cmdlets de AzureADPolicy para configurar la directiva

Para crear una lista de permitidos o de denegación, use el cmdlet [New-AzureADPolicy](https://docs.microsoft.com/powershell/module/azuread/new-azureadpolicy?view=azureadps-2.0-preview). En el ejemplo siguiente se muestra cómo establecer una lista de denegación que bloquea el dominio "live.com".

```powershell 
$policyValue = @("{`"B2BManagementPolicy`":{`"InvitationsAllowedAndBlockedDomainsPolicy`":{`"AllowedDomains`": [],`"BlockedDomains`": [`"live.com`"]}}}")

New-AzureADPolicy -Definition $policyValue -DisplayName B2BManagementPolicy -Type B2BManagementPolicy -IsOrganizationDefault $true 
```

A continuación se muestra el mismo ejemplo, pero con la definición de directiva insertada.

```powershell  
New-AzureADPolicy -Definition @("{`"B2BManagementPolicy`":{`"InvitationsAllowedAndBlockedDomainsPolicy`":{`"AllowedDomains`": [],`"BlockedDomains`": [`"live.com`"]}}}") -DisplayName B2BManagementPolicy -Type B2BManagementPolicy -IsOrganizationDefault $true 
```

Para establecer la directiva de lista de permitidos o de denegación, use el cmdlet [Set-AzureADPolicy](https://docs.microsoft.com/powershell/module/azuread/set-azureadpolicy?view=azureadps-2.0-preview). Por ejemplo:

```powershell   
Set-AzureADPolicy -Definition $policyValue -Id $currentpolicy.Id 
```

Para obtener la directiva, use el cmdlet [Get-AzureADPolicy](https://docs.microsoft.com/powershell/module/azuread/get-azureadpolicy?view=azureadps-2.0-preview). Por ejemplo:

```powershell
$currentpolicy = Get-AzureADPolicy | ?{$_.Type -eq 'B2BManagementPolicy'} | select -First 1 
```

Para quitar la directiva, use el cmdlet [Remove-AzureADPolicy](https://docs.microsoft.com/powershell/module/azuread/remove-azureadpolicy?view=azureadps-2.0-preview). Por ejemplo:

```powershell
Remove-AzureADPolicy -Id $currentpolicy.Id 
```

## <a name="next-steps"></a>Pasos siguientes

- Para información general sobre B2B de Azure AD, consulte [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
- Para información sobre el acceso condicional y la colaboración B2B, consulte [Acceso condicional para usuarios de colaboración B2B](conditional-access.md).



