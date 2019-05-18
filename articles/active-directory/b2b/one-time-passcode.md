---
title: 'Autenticación de código de acceso de un solo uso para usuarios invitados de B2B: Azure Active Directory | Microsoft Docs'
description: Cómo usar el código de acceso de un solo uso de correo electrónico para autenticar a los usuarios invitados de B2B sin necesidad de una cuenta de Microsoft.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 04/08/2019
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan, seoapril2019
ms.collection: M365-identity-device-management
ms.openlocfilehash: 577f370c3be0f232be4e6628dd6754c4164ab93f
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65785256"
---
# <a name="email-one-time-passcode-authentication-preview"></a>Autenticación con código de acceso de un solo uso de correo electrónico (versión preliminar)

|     |
| --- |
| El código de acceso de un solo uso de correo electrónico es una característica en vista previa pública de Azure Active Directory. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).|
|     |

En este artículo se describe cómo habilitar la autenticación de correo electrónico única del código de acceso para los usuarios invitados de B2B. La característica de código de acceso de un solo uso de correo electrónico autentica los usuarios invitados de B2B cuando no pueden autenticarse a través de otros medios, como Azure AD, una cuenta de Microsoft (MSA) o federación de Google. Con la autenticación por código de acceso de un solo uso, no hay necesidad de crear una cuenta de Microsoft. Cuando el usuario invitado canjea una invitación o accede a un recurso compartido, puede solicitar un código temporal, que se envía a su dirección de correo electrónico. A continuación, escribe este código para continuar con el inicio de sesión.

Esta característica está disponible actualmente en versión preliminar (consulte [Participación en la versión preliminar](#opting-in-to-the-preview) a continuación). Después de la versión preliminar, esta característica se activará de forma predeterminada para todos los inquilinos.

> [!NOTE]
> Los usuarios con código de acceso de un solo uso deben iniciar sesión con un vínculo que incluya el contexto del inquilino (por ejemplo, `https://myapps.microsoft.com/?tenantid=<tenant id>` o `https://portal.azure.com/<tenant id>`, o en el caso de un dominio verificado, `https://myapps.microsoft.com/<verified domain>.onmicrosoft.com`). Los vínculos directos a aplicaciones y los recursos también funcionan siempre que incluyan el contexto del inquilino. Actualmente, los usuarios invitados no pueden iniciar sesión con puntos de conexión sin contexto de inquilino. Por ejemplo, el uso de `https://myapps.microsoft.com`, `https://portal.azure.com` o el punto de conexión común de los equipos producirá un error. 

## <a name="user-experience-for-one-time-passcode-guest-users"></a>Experiencia de usuario para los usuarios invitados de código de acceso de un solo uso
Con la autenticación de código de acceso de un solo uso, el usuario invitado puede canjear su invitación haciendo clic en un vínculo directo o mediante el correo electrónico de invitación. En cualquier caso, un mensaje en el explorador indica que se enviará un código a la dirección de correo electrónico del usuario invitado. El usuario invitado selecciona **Enviar código**:
 
   ![Captura de pantalla que muestra el botón de código de envío](media/one-time-passcode/otp-send-code.png)
 
Se envía un código de acceso a la dirección de correo electrónico del usuario. El usuario recupera el código de acceso del correo electrónico y lo escribe en la ventana del explorador:
 
   ![Captura de pantalla muestra la página de códigos ENTRAR](media/one-time-passcode/otp-enter-code.png)
 
Ahora el usuario invitado se ha autenticado y puede ver el recurso compartido o continuar con el inicio de sesión. 

> [!NOTE]
> Los códigos de acceso de un solo uso son válidos durante 30 minutos. Después de 30 minutos, ese código de acceso de un solo uso específico ya no será válido y el usuario deberá solicitar uno nuevo. Las sesiones del usuario expiran después de 24 horas. Después de ese tiempo, el usuario invitado recibe un nuevo código de acceso cuando accede al recurso. La expiración de la sesión proporciona mayor seguridad, en especial cuando un usuario invitado deja su empresa o ya no necesita tener acceso.

## <a name="when-does-a-guest-user-get-a-one-time-passcode"></a>¿Cuándo un usuario invitado obtiene un código de acceso de un solo uso?

Cuando un usuario invitado canjea una invitación o usa un vínculo a un recurso que se ha compartido con él, recibe un código de acceso de un solo uso si:
- No tiene una cuenta de Azure AD 
- No tiene una cuenta de Microsoft 
- El inquilino que invita no ha configurado la federación de Google para los usuarios @gmail.com y @googlemail.com 

En el momento de la invitación, no hay ninguna indicación de que el usuario al que está invitando usará la autenticación de código de acceso de un solo uso. Pero cuando el usuario invitado inicia sesión, la autenticación de código de acceso de un solo uso será el método de reserva si no se puede usar ningún otro método de autenticación. 

Puede ver los usuarios invitados que se autentican con códigos de acceso de un solo uso en Azure Portal, en **Azure Active Directory** > **Relaciones organizativas** > **Usuarios de otras organizaciones**.

![Captura de pantalla con un usuario de código de acceso de un solo uso con el valor de origen de OTP](media/one-time-passcode/otp-users.png)

> [!NOTE]
> Cuando un usuario canjea un código de acceso de un solo uso y más adelante obtiene una MSA, una cuenta de Azure AD u otra cuenta federada, seguirá autenticándose con un código de acceso de un solo uso. Si quiere actualizar el método de autenticación, puede eliminar la cuenta de usuario invitado y volver a invitarlo.

### <a name="example"></a>Ejemplo
Se invita al usuario invitado alexdoe@gmail.com a Fabrikam, que no tiene configurada la federación de Google. Alex no tiene una cuenta de Microsoft. Recibe un código de acceso de un solo uso para la autenticación.

## <a name="opting-in-to-the-preview"></a>Participación en la versión preliminar 
Es posible que tarde unos minutos para que la acción de participación surta efecto. Después de eso, solo los usuarios recién invitados que cumplan las condiciones anteriores usarán la autenticación de código de acceso de un solo uso. Los usuarios invitados que anteriormente canjearon una invitación seguirán usando el mismo método de autenticación.

### <a name="to-opt-in-using-the-azure-ad-portal"></a>Para participar con el portal de Azure AD
1.  Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador global de Azure AD.
2.  En el panel de navegación, seleccione **Azure Active Directory**.
3.  En **Administrar**, seleccione **Relaciones organizativas**.
4.  Seleccione **Configuración**.
5.  En **Habilitar el código de acceso de un solo uso de correo electrónico para invitados (versión preliminar)**, seleccione **Sí**.
 
### <a name="to-opt-in-using-powershell"></a>Para participar con PowerShell

En primer lugar, tiene que instalar la versión más reciente de Azure AD PowerShell para el módulo Graph (AzureADPreview). A continuación, decidirá si las directivas de B2B ya existen y ejecutan los comandos adecuados.

#### <a name="prerequisite-install-the-latest-azureadpreview-module"></a>Requisito previo: Instalación del último módulo de AzureADPreview
En primer lugar, compruebe qué módulos ha instalado. Abra Windows PowerShell como un usuario con privilegios elevados (Ejecutar como administrador) y ejecute el comando siguiente:
 
```powershell  
Get-Module -ListAvailable AzureAD*
```

Si se muestra el módulo AzureADPreview sin ningún mensaje que indica que hay una versión posterior, significa que está listo. De lo contrario, en función de la salida, realice una de las siguientes acciones:

- Si no se devuelve ningún resultado, ejecute el siguiente comando para instalar el módulo AzureADPreview:
  
   ```powershell  
   Install-Module AzureADPreview
   ```
- Si solo se muestra el módulo AzureAD en los resultados, ejecute los comandos siguientes para instalar el módulo AzureADPreview: 

   ```powershell 
   Uninstall-Module AzureAD 
   Install-Module AzureADPreview 
   ```
- Si solo se muestra el módulo AzureADPreview en los resultados, pero recibe un mensaje que indica que hay una versión posterior, ejecute los comandos siguientes para actualizar el módulo: 

   ```powershell 
   Uninstall-Module AzureADPreview 
   Install-Module AzureADPreview 
  ```

Puede que reciba un mensaje que indica que está instalando el módulo desde un repositorio de confianza. Esto se produce si el repositorio de PSGallery no se ha establecido previamente como un repositorio de confianza. Presione **Y** para instalar el módulo.

#### <a name="check-for-existing-policies-and-opt-in"></a>Comprobación de las directivas existentes y participación

A continuación, ejecute lo siguiente para comprobar si actualmente existe una directiva B2BManagementPolicy:

```powershell 
$currentpolicy =  Get-AzureADPolicy | ?{$_.Type -eq 'B2BManagementPolicy' -and $_.IsOrganizationDefault -eq $true} | select -First 1
$currentpolicy -ne $null
```
- Si el resultado es False, la directiva no existe actualmente. Cree una directiva B2BManagementPolicy y participe en la versión preliminar ejecutando lo siguiente:

   ```powershell 
   $policyValue=@("{`"B2BManagementPolicy`":{`"PreviewPolicy`":{`"Features`":[`"OneTimePasscode`"]}}}")
   New-AzureADPolicy -Definition $policyValue -DisplayName B2BManagementPolicy -Type B2BManagementPolicy -IsOrganizationDefault $true
   ```

- Si el resultado es True, la directiva B2BManagementPolicy existe actualmente. Para actualizar la directiva y participar en la versión preliminar, ejecute lo siguiente:
  
   ```powershell 
   $policy = $currentpolicy.Definition | ConvertFrom-Json
   $features=[PSCustomObject]@{'Features'=@('OneTimePasscode')}; $policy.B2BManagementPolicy | Add-Member 'PreviewPolicy' $features -Force; $policy.B2BManagementPolicy
   $updatedPolicy = $policy | ConvertTo-Json -Depth 3
   Set-AzureADPolicy -Definition $updatedPolicy -Id $currentpolicy.Id
   ```

## <a name="opting-out-of-the-preview-after-opting-in"></a>Dejar de participar en la versión preliminar tras la participación
Es posible que tarde unos minutos para que la acción de dejar de participar surta efecto. Si desactiva la versión preliminar, los usuarios invitados que han canjeado un código de acceso de un solo uso no podrán iniciar sesión. Puede eliminar el usuario invitado y volver a invitar al usuario para permitir que vuelva a iniciarla con otro método de autenticación.

### <a name="to-turn-off-the-preview-using-the-azure-ad-portal"></a>Para desactivar la versión preliminar desde el portal de Azure AD
1.  Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador global de Azure AD.
2.  En el panel de navegación, seleccione **Azure Active Directory**.
3.  En **Administrar**, seleccione **Relaciones organizativas**.
4.  Seleccione **Configuración**.
5.  En **Habilitar el código de acceso de un solo uso de correo electrónico para invitados (versión preliminar)**, seleccione **No**.

### <a name="to-turn-off-the-preview-using-powershell"></a>Para desactivar la versión preliminar con PowerShell
Instale el módulo más reciente de AzureADPreview si aún no lo ha hecho (consulte [Requisitos previos: Instalación del último módulo de AzureADPreview](#prerequisite-install-the-latest-azureadpreview-module) más arriba). A continuación, para verificar que la directiva de versión preliminar del código de acceso de un solo uso exista actualmente, ejecute lo siguiente:

```powershell 
$currentpolicy = Get-AzureADPolicy | ?{$_.Type -eq 'B2BManagementPolicy' -and $_.IsOrganizationDefault -eq $true} | select -First 1
($currentPolicy -ne $null) -and ($currentPolicy.Definition -like "*OneTimePasscode*")
```

Si el resultado es True, deje de participar de la versión preliminar ejecutando lo siguiente:

```powershell 
$policy = $currentpolicy.Definition | ConvertFrom-Json
$policy.B2BManagementPolicy.PreviewPolicy.Features = $policy.B2BManagementPolicy.PreviewPolicy.Features.Where({$_ -ne "OneTimePasscode"})
$updatedPolicy = $policy | ConvertTo-Json -Depth 3
Set-AzureADPolicy -Definition $updatedPolicy -Id $currentpolicy.Id
```

