---
title: 'Azure Active Directory Connect: solución de problemas de inicio de sesión único de conexión directa | Microsoft Docs'
description: En este tema se describe cómo solucionar los problemas del inicio de sesión único de conexión directa de Azure Active Directory.
services: active-directory
author: billmath
ms.reviewer: swkrish
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.topic: troubleshooting
ms.date: 10/07/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6afba520db399cc4eb99c8716540886cc88f6ae7
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89275885"
---
# <a name="troubleshoot-azure-active-directory-seamless-single-sign-on"></a>Solución de problemas de inicio de sesión único de conexión directa de Azure Active Directory

Este artículo sirve de ayuda para encontrar información sobre cómo solucionar los problemas comunes relativos al inicio de sesión único de conexión directa (SSO de conexión directa) de Azure Active Directory (Azure AD).

## <a name="known-issues"></a>Problemas conocidos

- En algunos casos, el proceso para habilitar el inicio de sesión único de conexión directa puede tardar hasta 30 minutos.
- Si deshabilita y vuelve a habilitar Inicio de sesión único de conexión directa en el inquilino, los usuarios no podrán tener la experiencia de inicio de sesión único hasta que sus vales de Kerberos en caché, que normalmente son válidos durante diez horas, hayan expirado.
- Si el inicio de sesión único de conexión directa se realiza correctamente, el usuario no tiene la oportunidad de seleccionar **Mantener la sesión iniciada**. Debido a este comportamiento, los [escenarios de asignación de SharePoint y OneDrive](https://support.microsoft.com/help/2616712/how-to-configure-and-to-troubleshoot-mapped-network-drives-that-connec) no funcionan.
- Se admiten los clientes Win32 de Office 365 (Outlook, Word, Excel, etc.) con las versiones 16.0.8730 y posteriores mediante un flujo no interactivo. No se admiten otras versiones; en estas, para iniciar sesión, los usuarios escribirán sus nombres de usuario, pero no las contraseñas. En OneDrive, tendrá que activar la [función de configuración silenciosa de OneDrive](https://techcommunity.microsoft.com/t5/Microsoft-OneDrive-Blog/Previews-for-Silent-Sync-Account-Configuration-and-Bandwidth/ba-p/120894) para disfrutar de una experiencia de inicio de sesión silenciosa.
- SSO de conexión directa no funciona en modo de exploración privada en Firefox.
- El inicio de sesión único de conexión directa no funciona en Internet Explorer cuando está activado el modo de protección mejorada.
- El inicio de sesión único de conexión directa no funciona en exploradores móviles en iOS y Android.
- Si un usuario forma parte de demasiados grupos de Active Directory, es probable que el valor de Kerberos del usuario sea demasiado largo para procesarse, lo que hará que se produzca un error en el inicio de sesión único de conexión directa. Las solicitudes HTTPS de Azure AD pueden tener encabezados con un tamaño máximo de 50 KB; los vales de Kerberos deben ser menores que ese número para albergar otros artefactos de Azure AD (generalmente, de 2 a 5 KB), como las cookies. Nuestra recomendación es reducir la pertenencia a grupos del usuario y volver a intentarlo.
- Si va a sincronizar treinta bosques de Active Directory o más, no se puede habilitar el inicio de sesión único de conexión directa mediante Azure AD Connect. Como alternativa, también puede [habilitar manualmente](#manual-reset-of-the-feature) la característica en su inquilino.
- Agregar la dirección URL del servicio de Azure AD (`https://autologon.microsoftazuread-sso.com`) a la zona de sitios de confianza en lugar de a la zona de intranet local *impide que los usuarios inicien sesión*.
- El inicio de sesión único de conexión directa admite los tipos de cifrado AES256_HMAC_SHA1, AES128_HMAC_SHA1 y RC4_HMAC_MD5 para Kerberos. Se recomienda que el tipo de cifrado de la cuenta AzureADSSOAcc$ se establezca en AES256_HMAC_SHA1 o uno de los tipos AES en lugar de RC4, para mayor seguridad. El tipo de cifrado se almacena en el atributo msDS-SupportedEncryptionTypes de la cuenta en Active Directory.  Si el tipo de cifrado de la cuenta AzureADSSOAcc$ está establecido en RC4_HMAC_MD5 y desea cambiarlo a uno de los tipos de cifrado AES, asegúrese de revertir primero la clave de descifrado de Kerberos de dicha cuenta, tal como se explica en la pregunta correspondiente del [documento de preguntas más frecuentes](how-to-connect-sso-faq.md); de lo contrario, no se producirá el inicio de sesión único de conexión directa.

## <a name="check-status-of-feature"></a>Comprobación del estado de la característica

Asegúrese de que la característica de SSO de conexión directa esté aún **habilitada** en el inquilino. Puede comprobar el estado; para ello, vaya al panel **Azure AD Connect** en el [Centro de administración de Azure Active Directory](https://aad.portal.azure.com/).

![Centro de administración de Azure Active Directory: Panel de Azure AD Connect](./media/tshoot-connect-sso/sso10.png)

Haga clic para ver todos los bosques de AD que están habilitados para SSO de conexión directa.

![Centro de administración de Azure Active Directory: Panel de SSO de conexión directa](./media/tshoot-connect-sso/sso13.png)

## <a name="sign-in-failure-reasons-in-the-azure-active-directory-admin-center-needs-a-premium-license"></a>Motivos del error de inicio de sesión en el centro de administración de Azure Active Directory (se necesita una licencia Premium)

Si el inquilino tiene una licencia de Azure AD Premium asociada, también puede buscar el [informe actividad de inicio de sesión](../reports-monitoring/concept-sign-ins.md) en el [centro de administración de Azure Active Directory](https://aad.portal.azure.com/).

![Centro de administración de Azure Active Directory: Informe de inicios de sesión](./media/tshoot-connect-sso/sso9.png)

Examine **Azure Active Directory** > **Inicios de sesión** en el [centro de administración de Azure Active Directory](https://aad.portal.azure.com/) y después seleccione la actividad de inicio de sesión de un usuario específico. Busque el campo **CÓDIGO DE ERROR DE INICIO DE SESIÓN**. Busque la correspondencia entre el valor de ese campo y un motivo de error y la resolución en la siguiente tabla:

|Código de error de inicio de sesión|Motivo del error de inicio de sesión|Solución
| --- | --- | ---
| 81001 | El vale de Kerberos del usuario es demasiado grande. | Reduzca la pertenencia a grupos del usuario e inténtelo de nuevo.
| 81002 | No se puede validar el vale Kerberos del usuario. | Vea la [Lista de comprobación de solución de problemas](#troubleshooting-checklist).
| 81003 | No se puede validar el vale Kerberos del usuario. | Vea la [Lista de comprobación de solución de problemas](#troubleshooting-checklist).
| 81004 | Error al intentar la autenticación Kerberos. | Vea la [Lista de comprobación de solución de problemas](#troubleshooting-checklist).
| 81008 | No se puede validar el vale Kerberos del usuario. | Vea la [Lista de comprobación de solución de problemas](#troubleshooting-checklist).
| 81009 | No se puede validar el vale Kerberos del usuario. | Vea la [Lista de comprobación de solución de problemas](#troubleshooting-checklist).
| 81010 | Error de SSO de conexión directa porque el vale de Kerberos del usuario ha expirado o no es válido. | El usuario debe iniciar sesión desde un dispositivo unido al dominio dentro de la red corporativa.
| 81011 | No se puede encontrar el objeto de usuario con la información del vale de Kerberos del usuario. | Use Azure AD Connect para sincronizar la información del usuario en Azure AD.
| 81012 | El usuario que intenta iniciar sesión en Azure AD es distinto del que inició sesión en el dispositivo. | El usuario debe iniciar sesión desde un dispositivo diferente.
| 81013 | No se puede encontrar el objeto de usuario con la información del vale de Kerberos del usuario. |Use Azure AD Connect para sincronizar la información del usuario en Azure AD. 

## <a name="troubleshooting-checklist"></a>Lista de comprobación de solución de problemas

Use la siguiente lista de comprobación para solucionar problemas de SSO de conexión directa:

- Asegúrese de que la característica SSO de conexión directa está habilitada en Azure AD Connect. Si no puede habilitarla (por ejemplo, debido a que hay un puerto bloqueado), asegúrese de que cumple todos los [requisitos previos](how-to-connect-sso-quick-start.md#step-1-check-the-prerequisites).
- Si ha habilitado tanto [Azure AD Join](../devices/overview.md) como SSO de conexión directa en el inquilino, asegúrese de que el problema no está relacionado con Azure AD Join. SSO de Azure AD Join tiene prioridad sobre SSO de conexión directa, si el dispositivo está tanto registrado con Azure AD como unido a un dominio. Con SSO de Azure AD Join, el usuario ve un icono de inicio de sesión que dice "Conectado a Windows".
- Asegúrese de que la dirección URL de Azure AD (`https://autologon.microsoftazuread-sso.com`) forma parte de la configuración de zonas de intranet del usuario.
- Asegúrese de que el dispositivo corporativo se ha unido al dominio de Active Directory. El dispositivo _no_ tiene que [unirse a Azure AD](../devices/overview.md) para que SSO de conexión directa funcione.
- Asegúrese de que el usuario ha iniciado sesión en el dispositivo a través de una cuenta de dominio de Active Directory.
- Asegúrese de que la cuenta del usuario provenga de un bosque de Active Directory donde esté configurado SSO de conexión directa.
- Asegúrese de que el dispositivo está conectado a la red corporativa.
- Asegúrese de que la hora del dispositivo esté sincronizada con la de Active Directory y la de los controladores de dominio, y de que difieren entre sí un máximo de cinco minutos.
- Asegúrese de que la cuenta de equipo `AZUREADSSOACC` esté presente y habilitada en cada bosque de AD que desee habilitar SSO de conexión directa. Si la cuenta de equipo se ha eliminado o no está, puede usar [cmdlets de PowerShell](#manual-reset-of-the-feature) para volver a crearla.
- Enumere los vales de Kerberos existentes en el dispositivo mediante el comando `klist` desde un símbolo del sistema. Asegúrese de que los vales emitidos para la cuenta de equipo `AZUREADSSOACC` están presentes. Los vales de Kerberos de los usuarios suelen ser válidos durante diez horas. Puede que haya configuraciones distintas en Active Directory.
- Si deshabilita y vuelve a habilitar Inicio de sesión único de conexión directa en el inquilino, los usuarios no podrán tener la experiencia de inicio de sesión único hasta que sus vales de Kerberos en caché hayan expirado.
- Purgue los vales de Kerberos existentes desde el dispositivo mediante el comando `klist purge` e inténtelo de nuevo.
- Para determinar si hay problemas relacionados con JavaScript, revise los registros de la consola del explorador (en **Herramientas de desarrollo**).
- Revise los [registros de controlador de dominio](#domain-controller-logs).

### <a name="domain-controller-logs"></a>Registros del controlador de dominio

Si habilita la auditoría de eventos correctos en el controlador de dominio, cada vez que un usuario inicie sesión mediante SSO de conexión directa, se graba una entrada de seguridad en el registro de eventos. Para encontrar estos eventos de seguridad, utilice la consulta siguiente. (Busque el evento **4769** asociado a la cuenta de equipo **AzureADSSOAcc$** ).

```
    <QueryList>
      <Query Id="0" Path="Security">
    <Select Path="Security">*[EventData[Data[@Name='ServiceName'] and (Data='AZUREADSSOACC$')]]</Select>
      </Query>
    </QueryList>
```

## <a name="manual-reset-of-the-feature"></a>Restablecimiento manual de la característica

Si el procedimiento de solución de problemas no sirve de ayuda, restablezca manualmente la característica en su inquilino. Siga estos pasos en el servidor local donde se ejecuta Azure AD Connect.

### <a name="step-1-import-the-seamless-sso-powershell-module"></a>Paso 1: Importación del módulo de PowerShell de SSO de conexión directa

1. Primero, descargue e instale [Azure AD PowerShell](/powershell/azure/active-directory/overview).
2. Examine la carpeta `%programfiles%\Microsoft Azure Active Directory Connect`.
3. Importe el módulo de PowerShell de SSO de conexión directa mediante este comando: `Import-Module .\AzureADSSO.psd1`.

### <a name="step-2-get-the-list-of-active-directory-forests-on-which-seamless-sso-has-been-enabled"></a>Paso 2: Obtención de la lista de bosques de Active Directory en la que se habilitó el SSO de conexión directa

1. Ejecute PowerShell como administrador. En PowerShell, llame a `New-AzureADSSOAuthenticationContext`. Cuando se le solicite, escriba las credenciales de administrador global de su inquilino.
2. Llame a `Get-AzureADSSOStatus`. Este comando proporciona la lista de bosques de Active Directory (examine la lista "Dominios") en la que se ha habilitado esta característica.

### <a name="step-3-disable-seamless-sso-for-each-active-directory-forest-where-youve-set-up-the-feature"></a>Paso 3: Deshabilitación de SSO de conexión directa para cada bosque de Active Directory en el que se configuró la característica

1. Llame a `$creds = Get-Credential`. Cuando se le pida, escriba las credenciales del administrador de dominio para el bosque de Active Directory deseado.

   > [!NOTE]
   >El nombre de usuario de las credenciales de administrador de dominio debe especificarse con el formato de nombre de cuenta SAM (contoso\johndoe o contoso.com\johndoe). La parte de dominio del nombre de usuario se usa para localizar el controlador de dominio del administrador de dominio con DNS.

   >[!NOTE]
   >La cuenta de administrador de dominio usada no puede ser miembro del grupo Usuarios protegidos. De lo contrario, la operación presentará un error.

2. Llame a `Disable-AzureADSSOForest -OnPremCredentials $creds`. Este comando quita la cuenta de equipo `AZUREADSSOACC` del controlador de dominio local para este bosque de Active Directory específico.
3. Repita los pasos anteriores para cada bosque de Active Directory en el que se configuró la característica.

### <a name="step-4-enable-seamless-sso-for-each-active-directory-forest"></a>Paso 4: Habilitación de SSO de conexión directa para cada bosque de Active Directory

1. Llame a `Enable-AzureADSSOForest`. Cuando se le pida, escriba las credenciales del administrador de dominio para el bosque de Active Directory deseado.

   > [!NOTE]
   >El nombre de usuario de las credenciales de administrador de dominio debe especificarse con el formato de nombre de cuenta SAM (contoso\johndoe o contoso.com\johndoe). La parte de dominio del nombre de usuario se usa para localizar el controlador de dominio del administrador de dominio con DNS.

   >[!NOTE]
   >La cuenta de administrador de dominio usada no puede ser miembro del grupo Usuarios protegidos. De lo contrario, la operación presentará un error.

2. Repita los pasos anteriores para cada bosque de Active Directory en el que desea configurar la característica.

### <a name="step-5-enable-the-feature-on-your-tenant"></a>Paso 5. Habilite la característica en su inquilino

Para activar la característica en su inquilino, llame a `Enable-AzureADSSO -Enable $true`.