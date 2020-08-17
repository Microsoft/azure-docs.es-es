---
title: 'Azure AD Connect: Solución de problemas de autenticación de paso a través | Microsoft Docs'
description: En este artículo se describe cómo solucionar problemas de autenticación de paso a través de Azure Active Directory (Azure AD).
services: active-directory
keywords: Solucionar problemas de autenticación de paso a través de Azure AD Connect, instalar Active Directory, componentes necesarios para Azure AD, SSO, inicio de sesión único
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 07/27/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: f480118aaabf24bd7c5ca472bf04b12ee1405010
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87446996"
---
# <a name="troubleshoot-azure-active-directory-pass-through-authentication"></a>Solución de problemas de autenticación de paso a través de Azure Active Directory

Este artículo sirve de ayuda para encontrar información acerca de cómo solucionar los problemas comunes relativos a la autenticación de paso a través de Azure AD.

>[!IMPORTANT]
>Si se enfrenta a problemas de inicio de sesión de los usuarios con la autenticación de paso a través, no deshabilite la característica ni desinstale los agentes de autenticación de paso a través sin tener una cuenta de administrador global solo en la nube a la que retroceder. Información acerca de la [incorporación de una cuenta de administrador global que está solo en la nube](../active-directory-users-create-azure-portal.md). Este paso es esencial y se asegura de no quedar bloqueado fuera de su inquilino.

## <a name="general-issues"></a>Problemas generales

### <a name="check-status-of-the-feature-and-authentication-agents"></a>Comprobación de estado de la característica y de los agentes de autenticación

Asegúrese de que la característica de autenticación de paso a través sigue **habilitada** en su inquilino y de que el estado de los agentes de autenticación es **activo** y no **inactivo**. Puede comprobar el estado; para ello, vaya a la hoja **Azure AD Connect** en el [Centro de administración de Azure Active Directory](https://aad.portal.azure.com/).

![Centro de administración de Azure Active Directory: hoja de Azure AD Connect](./media/tshoot-connect-pass-through-authentication/pta7.png)

![Centro de administración de Azure Active Directory: hoja de Autenticación de paso a través](./media/tshoot-connect-pass-through-authentication/pta11.png)

### <a name="user-facing-sign-in-error-messages"></a>Mensajes de error de inicio de sesión para el usuario

Si el usuario no ha podido iniciar sesión con la autenticación de paso a través, puede ver uno de los siguientes errores para el usuario en la pantalla de inicio de sesión de Azure AD: 

|Error|Descripción|Resolución
| --- | --- | ---
|AADSTS80001|No es posible conectarse a Active Directory.|Asegúrese de que los servidores del agente sean miembros del mismo bosque de AD que los usuarios cuyas contraseñas haya que validar y que pueden conectarse a Active Directory.  
|AADSTS8002|Se ha agotado el tiempo de espera al conectarse a Active Directory.|Asegúrese de que Active Directory está disponible y responde a las solicitudes de los agentes.
|AADSTS80004|El nombre de usuario transferido al agente no era válido.|Asegúrese de que el usuario esté intentando iniciar sesión con el nombre de usuario correcto.
|AADSTS80005|La validación encontró una excepción WebException impredecible|Se trata de un error transitorio. Vuelva a intentarlo. Si el error no desaparece, póngase en contacto con el soporte técnico de Microsoft.
|AADSTS80007|Error al establecer comunicación con Active Directory.|Compruebe los registros del agente para más información y verifique que Active Directory está funcionando según lo previsto.

### <a name="users-get-invalid-usernamepassword-error"></a>Los usuarios obtienen un error de nombre de usuario y contraseña no válidos 

Esto puede ocurrir cuando el UserPrincipalName (UPN) local de un usuario es diferente del UPN de nube del usuario.

Para confirmar que este es el problema, primero compruebe que el agente de autenticación de paso a través funciona correctamente:


1. Cree una cuenta de prueba.  
2. Importe el módulo de PowerShell en el equipo del agente:
 
 ```powershell
 Import-Module "C:\Program Files\Microsoft Azure AD Connect Authentication Agent\Modules\PassthroughAuthPSModule\PassthroughAuthPSModule.psd1"
 ```
3. Ejecute el comando Invoke de PowerShell: 

 ```powershell
 Invoke-PassthroughAuthOnPremLogonTroubleshooter 
 ``` 
4. Cuando se le pida que escriba las credenciales, escriba el mismo nombre de usuario y contraseña que usa para iniciar sesión (https://login.microsoftonline.com).

Si recibe el mismo error de nombre de usuario y contraseña, significa que el agente de autenticación de paso a través funciona correctamente y el problema puede ser que el UPN local no sea enrutable. Para saber más, consulte [Configuración del identificador de inicio de sesión alternativo]( https://docs.microsoft.com/windows-server/identity/ad-fs/operations/configuring-alternate-login-id#:~:text=%20Configuring%20Alternate%20Login%20ID,See%20Also.%20%20More).

> [!IMPORTANT]
> Si el servidor de Azure AD Connect no está unido a un dominio, lo cual es un requisito mencionado en [Azure AD Connect: requisitos previos](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-install-prerequisites#installation-prerequisites), se produce el problema de nombre de usuario o contraseña no válidos.

### <a name="sign-in-failure-reasons-on-the-azure-active-directory-admin-center-needs-premium-license"></a>Motivos del error de inicio de sesión en el centro de administración de Azure Active Directory (necesita una licencia Premium)

Si el inquilino tiene una licencia de Azure AD Premium asociada, también puede buscar el [informe actividad de inicio de sesión](../reports-monitoring/concept-sign-ins.md) en el [centro de administración de Azure Active Directory](https://aad.portal.azure.com/).

![Centro de administración de Azure Active Directory: informe de inicios de sesión](./media/tshoot-connect-pass-through-authentication/pta4.png)

Vaya a **Azure Active Directory** -> **Inicios de sesión** en el [centro de administración de Azure Active Directory](https://aad.portal.azure.com/) y haga clic en la actividad de inicio de sesión de un usuario específico. Busque el campo **CÓDIGO DE ERROR DE INICIO DE SESIÓN**. Busque la correspondencia entre el valor de ese campo y un motivo de error y la resolución en la siguiente tabla:

|Código de error de inicio de sesión|Motivo del error de inicio de sesión|Resolución
| --- | --- | ---
| 50144 | Ha expirado la contraseña de Active Directory del usuario. | Restablezca la contraseña del usuario en Active Directory local.
| 80001 | No hay ningún agente de autenticación disponible. | Instale y registre un agente de autenticación.
| 80002 | El tiempo de espera se agotó para la solicitud de validación de contraseña del agente de autenticación. | Compruebe si Active Directory es accesible desde el agente de autenticación.
| 80003 | El agente de autenticación recibió una respuesta no válida. | Si el problema puede reproducirse habitualmente a través de varios usuarios, compruebe la configuración de Active Directory.
| 80004 | Se usó un nombre principal de usuario (UPN) incorrecto en una solicitud de inicio de sesión. | Pida al usuario que inicie sesión con el nombre de usuario correcto.
| 80005 | Agente de autenticación: Se ha producido un error. | Se trata de un error transitorio. Vuelva a intentarlo más tarde.
| 80007 | El agente de autenticación no puede conectarse a Active Directory. | Compruebe si Active Directory es accesible desde el agente de autenticación.
| 80010 | El agente de autenticación no puede descifrar la contraseña. | Si el problema se puede reproducir habitualmente, instale y registre un nuevo agente de autenticación. Después, desinstale el actual. 
| 80011 | El agente de autenticación no puede recuperar la clave de descifrado. | Si el problema se puede reproducir habitualmente, instale y registre un nuevo agente de autenticación. Después, desinstale el actual.

>[!IMPORTANT]
>Para autenticar a los usuarios de Azure AD, los agentes de autenticación de paso a través validan sus nombres de usuario y contraseñas en Active Directory mediante la llamada a la [API LogonUser de Win32](https://msdn.microsoft.com/library/windows/desktop/aa378184.aspx). Como resultado, si ha establecido la configuración de inicio de sesión en Active Directory para limitar el acceso de inicio de sesión de la estación de trabajo, tendrá que agregar también los servidores que hospedan los agentes de autenticación de paso a través a la lista de servidores de inicio de sesión. Si no lo hace, los usuarios no podrán iniciar sesión en Azure AD.

## <a name="authentication-agent-installation-issues"></a>Problemas de instalación del agente de autenticación

### <a name="an-unexpected-error-occurred"></a>Se ha producido un error inesperado

[Recopile los registros del agente](#collecting-pass-through-authentication-agent-logs) en el servidor y póngase en contacto con el Soporte técnico de Microsoft para que le solucionen el problema.

## <a name="authentication-agent-registration-issues"></a>Problemas de registro del agente de autenticación

### <a name="registration-of-the-authentication-agent-failed-due-to-blocked-ports"></a>No se pudo realizar el registro del agente de autenticación porque había puertos bloqueados

Asegúrese de que el servidor en el que se ha instalado el agente de autenticación puede comunicarse tanto con nuestras direcciones URL de servicio como con los puertos que se enumeran [aquí](how-to-connect-pta-quick-start.md#step-1-check-the-prerequisites).

### <a name="registration-of-the-authentication-agent-failed-due-to-token-or-account-authorization-errors"></a>No se puede registrar el agente de autenticación debido a errores de autorización de token o de cuenta

Asegúrese de que usa una cuenta de administrador global solo en la nube para todas las operaciones de instalación y registro del agente de autenticación independiente o de Azure AD Connect. Hay un problema conocido con las cuentas de administrador global habilitadas para MFA; desactive temporalmente MFA (solo para completar las operaciones) para proporcionar una solución alternativa.

### <a name="an-unexpected-error-occurred"></a>Se ha producido un error inesperado

[Recopile los registros del agente](#collecting-pass-through-authentication-agent-logs) en el servidor y póngase en contacto con el Soporte técnico de Microsoft para que le solucionen el problema.

## <a name="authentication-agent-uninstallation-issues"></a>Problemas de desinstalación del agente de autenticación

### <a name="warning-message-when-uninstalling-azure-ad-connect"></a>Mensaje de advertencia al desinstalar Azure AD Connect

Si tiene habilitada la autenticación de paso a través en el inquilino y trata de desinstalar Azure AD Connect, se muestra el siguiente mensaje de advertencia: "Los usuarios no podrán iniciar sesión en Azure AD a menos que tenga otros agentes de autenticación de paso a través instalados en otros servidores".

Para no interrumpir el inicio de sesión del usuario, es preciso tener una instalación de [alta disponibilidad](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability) en vigor antes de desinstalar Azure AD Connect.

## <a name="issues-with-enabling-the-feature"></a>Problemas con la habilitación de la característica

### <a name="enabling-the-feature-failed-because-there-were-no-authentication-agents-available"></a>La característica no se pudo habilitar porque no había agentes de autenticación disponibles

Necesita tener al menos un agente de autenticación activo para habilitar la autenticación de paso a través en el inquilino. Puede instalar un agente de autenticación instalando Azure AD Connect o un agente de autenticación independiente.

### <a name="enabling-the-feature-failed-due-to-blocked-ports"></a>La característica no se habilitó porque había puertos bloqueados

Asegúrese de que el servidor en el que se ha instalado Azure AD Connect puede comunicarse tanto con nuestras direcciones URL de servicio como con los puertos que se enumeran [aquí](how-to-connect-pta-quick-start.md#step-1-check-the-prerequisites).

### <a name="enabling-the-feature-failed-due-to-token-or-account-authorization-errors"></a>No se puede realizar la habilitación de la característica debido a errores de autorización de cuenta o de token.

Asegúrese de que usa una cuenta de administrador global solo en la nube cuando habilite la característica. Hay un problema conocido con las cuentas de administrador global habilitadas para Multi-Factor Authentication (MFA); desactive temporalmente MFA (solo para completar la operación) para proporcionar una solución alternativa.

## <a name="collecting-pass-through-authentication-agent-logs"></a>Recopilación de registros del agente de autenticación de autenticación de paso a través

En función del tipo de problema, es posible que tenga que buscar estos registros en distintos lugares.

### <a name="azure-ad-connect-logs"></a>Registros de Azure AD Connect

Para errores relacionados con la instalación, compruebe los registros de Azure AD Connect en **%ProgramData%\AADConnect\trace-\*.log**.

### <a name="authentication-agent-event-logs"></a>Registros de eventos del agente de autenticación

Para ver los errores relacionados con el agente de autenticación , abra la aplicación Visor de eventos en el servidor y consulte **Application and Service Logs\Microsoft\AzureAdConnect\AuthenticationAgent\Admin**.

Para obtener un análisis detallado, habilite el registro "Sesión" (haga clic con el botón derecho en la aplicación Visor de eventos para encontrar esta opción). No ejecute el agente de autenticación con este registro habilitado durante las operaciones normales; úselo solo para solucionar problemas. Tenga en cuenta que el contenido del registro solo se ve cuando el registro se vuelve a deshabilitar.

### <a name="detailed-trace-logs"></a>Registros de seguimiento detallados

Para solucionar errores de inicio de sesión de losusuarios, busque los registros de seguimiento en **%ProgramData%\Microsoft\Azure AD Connect Authentication Agent\Trace\\** . Estos registros incluyen los motivos por los que un usuario concreto no pudo iniciar sesión mediante la característica Autenticación de paso a través. Estos errores también pueden hacerse corresponder con los motivos de errores de inicio de sesión mostrados en la tabla anterior. La siguiente es una entrada del registro de ejemplo:

```
    AzureADConnectAuthenticationAgentService.exe Error: 0 : Passthrough Authentication request failed. RequestId: 'df63f4a4-68b9-44ae-8d81-6ad2d844d84e'. Reason: '1328'.
        ThreadId=5
        DateTime=xxxx-xx-xxTxx:xx:xx.xxxxxxZ
```

Para obtener detalles descriptivos del error ("1328" en el ejemplo anterior), abra el símbolo del sistema y ejecute el siguiente comando (Nota: Reemplace '1328' por el número de error real que ve en los registros):

`Net helpmsg 1328`

![Autenticación de paso a través](./media/tshoot-connect-pass-through-authentication/pta3.png)

### <a name="domain-controller-logs"></a>Registros de controlador de dominio

Si están habilitados los registros de auditoría, se puede encontrar información adicional en los registros de seguridad de los controladores de dominio. A continuación puede ver una manera sencilla de consultar las solicitudes de inicio de sesión enviadas por los agentes de autenticación de paso a través:

```
    <QueryList>
    <Query Id="0" Path="Security">
    <Select Path="Security">*[EventData[Data[@Name='ProcessName'] and (Data='C:\Program Files\Microsoft Azure AD Connect Authentication Agent\AzureADConnectAuthenticationAgentService.exe')]]</Select>
    </Query>
    </QueryList>
```

## <a name="performance-monitor-counters"></a>Contadores de Performance Monitor

Otra forma de supervisar a los agentes de autenticación consiste en realizar un seguimiento de los contadores específicos de Performance Monitor en cada servidor en que está instalado el agente de autenticación. Use los siguientes contadores globales ( **# PTA authentications**, **#PTA failed authentications** y **#PTA successful authentications**) y los contadores de errores ( **# PTA authentication errors**):

![Contadores de Performance Monitor de la Autenticación de paso a través](./media/tshoot-connect-pass-through-authentication/pta12.png)

>[!IMPORTANT]
>La Autenticación de paso a través ofrece alta disponibilidad mediante la utilización de varios agentes de autenticación, pero _no_ usa el equilibrio de carga. Según la configuración, _no_ todos los agentes de autenticación reciben aproximadamente el _mismo_ número de solicitudes. Es posible que un agente de autenticación específico no reciba ningún tráfico.
