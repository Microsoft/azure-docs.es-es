---
title: 'Azure AD Connect: Inicio de sesión único de conexión directa: inicio rápido | Microsoft Docs'
description: En este artículo se describe cómo empezar a trabajar con el inicio de sesión único de conexión directa de Azure Active Directory.
services: active-directory
keywords: qué es Azure AD Connect, instalar Active Directory, componentes necesarios para Azure AD, SSO, inicio de sesión único
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 04/16/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: bc955fe64ae68cb1248b0e616357bccf82f5f036
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87115760"
---
# <a name="azure-active-directory-seamless-single-sign-on-quickstart"></a>Inicio de sesión único de conexión directa de Azure Active Directory: Guía de inicio rápido

## <a name="deploy-seamless-single-sign-on"></a>Implementación del inicio de sesión único de conexión directa

El inicio de sesión único de conexión directa (SSO de conexión directa) de Azure Active Directory (Azure AD) permite iniciar sesión automáticamente a los usuarios en equipos de escritorio corporativos conectados a la red de la empresa. El inicio de sesión único de conexión directa proporciona a los usuarios un acceso sencillo a las aplicaciones en la nube sin necesidad de otros componentes locales.

Para implementar SSO de conexión directa, siga estos pasos.

## <a name="step-1-check-the-prerequisites"></a>Paso 1: Comprobar los requisitos previos

Asegúrese de que se cumplen los siguientes requisitos previos:

* **Configuración del servidor de Azure AD Connect**: si usa la [autenticación de paso a través](how-to-connect-pta.md) como método de inicio de sesión, no es necesaria ninguna otra comprobación de requisitos previos. Si va a usar la [sincronización de hash de contraseña](how-to-connect-password-hash-synchronization.md) como método de inicio de sesión y hay un firewall entre Azure AD Connect y Azure AD, asegúrese de que:
   - Usa Azure AD Connect 1.1.644.0 o cualquier versión posterior. 
   - Si el firewall o el proxy lo permiten, agregue las conexiones a la lista de permitidos para las direcciones URL **\*.msappproxy.net** en el puerto 443. En caso contrario, permita el acceso a los [intervalos de direcciones IP del centro de datos de Azure](https://www.microsoft.com/download/details.aspx?id=41653), que se actualizan cada semana. Este requisito solo es aplicable cuando se habilita la característica. No es necesario para los inicios de sesión de usuarios reales.

    >[!NOTE]
    >Las versiones de Azure AD Connect 1.1.557.0, 1.1.558.0, 1.1.561.0 y 1.1.614.0 tienen un problema relacionado con la sincronización de hash de contraseña. Si _no_ tiene pensado utilizar la sincronización de hash de contraseña junto con la autenticación de paso a través, consulte las [notas del historial de versiones de Azure AD Connect](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-version-history#116470) para más información.

* **Uso de una topología de Azure AD Connect**: asegúrese de que usa una de las topologías compatibles con Azure AD Connect que se describen [aquí](plan-connect-topologies.md).

    >[!NOTE]
    >El inicio de sesión único de conexión directa es compatible con varios bosques de AD, así haya relaciones de confianza de AD entre ellos o no.

* **Configuración de credenciales del administrador de dominio**: debe tener credenciales de administrador de dominio para cada bosque de Active Directory que:
    * Sincronice en Azure AD mediante Azure AD Connect.
    * Contenga usuarios para los que desea habilitar el SSO de conexión directa.
    
* **Habilitación de la autenticación moderna**: para que esta característica funcione, es preciso que habilite la [autenticación moderna](https://docs.microsoft.com/office365/enterprise/modern-auth-for-office-2013-and-2016) en el inquilino.

* **Uso de las versiones más recientes de los clientes de Office 365**: para obtener una experiencia de inicio de sesión silenciosa con clientes de Office 365 (Outlook, Word, Excel, etc.), los usuarios deben usar las versiones 16.0.8730.xxxx o superiores.

## <a name="step-2-enable-the-feature"></a>Paso 2: Habilitar la característica

Habilite el SSO de conexión directa mediante [Azure AD Connect](whatis-hybrid-identity.md).

>[!NOTE]
> También puede [habilitar el SSO de conexión directa con PowerShell](tshoot-connect-sso.md#manual-reset-of-the-feature) si Azure AD Connect no cumple sus requisitos. Utilice esta opción si tiene más de un dominio por cada bosque de Active Directory y desea ser más específico sobre el dominio para el que quiere habilitar SSO de conexión directa.

Si va a realizar una instalación nueva de Azure AD Connect, elija la [ruta de acceso de instalación personalizada](how-to-connect-install-custom.md). En la página **Inicio de sesión de usuario**, active la opción **Habilitar el inicio de sesión único**.

>[!NOTE]
> La opción solo estará disponible para seleccionarse si el método de inicio de sesión es **Sincronización de hash de contraseñas** o **Autenticación de paso a través**.

![Azure AD Connect: Inicio de sesión de usuario](./media/how-to-connect-sso-quick-start/sso8.png)

Si ya tiene una instalación de Azure AD Connect, seleccione la página **Cambiar inicio de sesión de usuario** en Azure AD Connect y después seleccione **Siguiente**. Si usa Azure AD Connect 1.1.880.0 o versiones posteriores, la opción **Habilitar inicio de sesión único** estará seleccionada de forma predeterminada. Si usa versiones anteriores de Azure AD Connect, seleccione la opción **Habilitar inicio de sesión único**.

![Azure AD Connect: Cambio del inicio de sesión de usuario](./media/how-to-connect-sso-quick-start/changeusersignin.png)

Continúe con el asistente hasta llegar a la página **Habilitar el inicio de sesión único**. Proporcione las credenciales de administrador de dominio para cada bosque de Active Directory que:

* Sincronice en Azure AD mediante Azure AD Connect.
* Contenga usuarios para los que desea habilitar el SSO de conexión directa.

Cuando haya finalizado con el asistente, el SSO de conexión directa estará habilitado en su inquilino.

>[!NOTE]
> Las credenciales de administrador de dominio no se almacenan en Azure AD Connect ni en Azure AD. Solo se usan para habilitar la característica.

Siga estas instrucciones para verificar que ha habilitado SSO de conexión directa correctamente:

1. Inicie sesión en el [Centro de administración de Azure Active Directory](https://aad.portal.azure.com) con las credenciales de administrador global del inquilino.
2. Seleccione **Azure Active Directory** en el panel izquierdo.
3. Seleccione **Azure AD Connect**.
4. Verifique que la característica **Inicio de sesión único de conexión directa** aparece como **Habilitado**.

![Azure Portal: Panel de Azure AD Connect](./media/how-to-connect-sso-quick-start/sso10.png)

>[!IMPORTANT]
> SSO de conexión directa crea una cuenta de equipo llamada `AZUREADSSOACC` en la instancia local de Active Directory (AD) de cada bosque de AD. La cuenta de equipo `AZUREADSSOACC` precisa de una gran protección por motivos de seguridad. Solo los administradores de dominio deben poder administrar la cuenta de equipo. Asegúrese de que la delegación de Kerberos de la cuenta de equipo está deshabilitada y de que ninguna otra cuenta de Active Directory tiene permisos de delegación en la cuenta de equipo `AZUREADSSOACC`. Almacene la cuenta de equipo en una unidad organizativa (OU) donde esté a salvo de eliminaciones accidentales y donde solo los administradores de dominio tengan acceso.

>[!NOTE]
> Si utiliza las arquitecturas Pass-the-Hash y Credential Theft Mitigation en su entorno local, realice los cambios adecuados para asegurarse de que la cuenta de equipo `AZUREADSSOACC` no termine en el contenedor de cuarentena. 

## <a name="step-3-roll-out-the-feature"></a>Paso 3: Implementación de la característica

Puede implementar el inicio de sesión único de conexión directa gradualmente a los usuarios con las instrucciones que se proporcionan a continuación. Para empezar, agregue la siguiente dirección URL de Azure AD a la configuración de la zona de la intranet de todos los usuarios, o de los seleccionados, mediante la directiva de grupo de Active Directory:

- `https://autologon.microsoftazuread-sso.com`

Además, tiene que habilitar a una configuración de directiva de zona de intranet denominada **Permitir actualizaciones en la barra de estado a través de script** mediante la directiva de grupo. 

>[!NOTE]
> Las siguientes instrucciones solo funcionan en Internet Explorer y Google Chrome en Windows (si comparte el mismo conjunto de direcciones URL de sitios de confianza que Internet Explorer). Lea la sección siguiente para obtener instrucciones acerca de cómo configurar Mozilla Firefox y Google Chrome en macOS.

### <a name="why-do-you-need-to-modify-users-intranet-zone-settings"></a>¿Por qué es necesario modificar la configuración de zona de Intranet de los usuarios?

De forma predeterminada, el explorador calcula automáticamente la zona correcta (Internet o intranet) de una dirección URL específica. Por ejemplo, `http://contoso/` se asigna a la zona de intranet, mientras que `http://intranet.contoso.com/` se asigna a la zona de Internet (porque la dirección URL contiene un punto). Los exploradores no enviarán vales de Kerberos a un punto de conexión en la nube, como la dirección URL de Azure AD, a menos que agregue explícitamente la dirección URL a la zona de intranet del explorador.

Hay dos formas de modificar la configuración de zona de intranet de los usuarios:

| Opción | Consideración de administrador | Experiencia del usuario |
| --- | --- | --- |
| Directiva de grupo | El administrador bloquea la edición de la configuración de la zona de intranet | Los usuarios no pueden modificar su propia configuración |
| Preferencia de directiva de grupo |  El administrador permite la edición de la configuración de la zona de intranet | Los usuarios pueden modificar su propia configuración |

### <a name="group-policy-option---detailed-steps"></a>Opción "Directiva de grupo": pasos detallados

1. Abra la herramienta Editor de administración de directivas de grupo.
2. Edite la directiva de grupo que se aplica a algunos o todos los usuarios. En este ejemplo se usa la **directiva de dominio predeterminada**.
3. Vaya a **Configuración de usuario** > **Directiva** > **Plantillas administrativas** > **Componentes de Windows** > **Internet Explorer** > **Panel de control de Internet** > **Página de seguridad**. A continuación, seleccione **Lista de asignación de sitio a zona**.
    ![Inicio de sesión único](./media/how-to-connect-sso-quick-start/sso6.png)
4. Habilite la directiva y escriba los valores siguientes en el cuadro de diálogo:
   - **Nombre de valor**: dirección URL de Azure AD a la que se reenvían los vales de Kerberos.
   - **Valor** (datos): **1** indica la zona de intranet.

     El resultado tiene el aspecto siguiente:

     Nombre de valor: `https://autologon.microsoftazuread-sso.com`
  
     Valor (datos): 1

   >[!NOTE]
   > Si no desea permitir que algunos usuarios utilicen SSO de conexión directa (por ejemplo, si estos usuarios inician sesión en quioscos compartidos) establezca los valores anteriores en **4**. De este modo, agrega la dirección URL de Azure AD a la zona Sitios restringidos, y el SSO de conexión directa no puede conectar a esos usuarios.
   >

5. Seleccione **Aceptar** y, después, otra vez **Aceptar**.

    ![Inicio de sesión único](./media/how-to-connect-sso-quick-start/sso7.png)

6. Vaya a **Configuración de usuario** > **Directiva** > **Plantillas administrativas** > **Componentes de Windows** > **Internet Explorer** > **Panel de control de Internet** > **Página de seguridad** > **Zona Intranet**. Después, seleccione **Permitir actualizaciones en la barra de estado a través de script**.

    ![Inicio de sesión único](./media/how-to-connect-sso-quick-start/sso11.png)

7. Habilite la configuración de directiva y después seleccione **Aceptar**.

    ![Inicio de sesión único](./media/how-to-connect-sso-quick-start/sso12.png)

### <a name="group-policy-preference-option---detailed-steps"></a>Opción "Preferencia de directiva de grupo": pasos detallados

1. Abra la herramienta Editor de administración de directivas de grupo.
2. Edite la directiva de grupo que se aplica a algunos o todos los usuarios. En este ejemplo se usa la **directiva de dominio predeterminada**.
3. Vaya a **Configuración de usuario** > **Preferencias** > **Configuración de Windows** > **Registro** > **Nuevo** > **Elemento del Registro**.

    ![Inicio de sesión único](./media/how-to-connect-sso-quick-start/sso15.png)

4. Escriba los siguientes valores en los campos apropiados y haga clic en **Aceptar**.
   - **Ruta de acceso de la clave**: ***Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Domains\microsoftazuread-sso.com\autologon***
   - **Nombre de valor**: ***https***.
   - **Tipo de valor**: ***REG_DWORD***
   - **Información del valor**: ***00000001***
 
     ![Inicio de sesión único](./media/how-to-connect-sso-quick-start/sso16.png)
 
     ![Inicio de sesión único](./media/how-to-connect-sso-quick-start/sso17.png)

### <a name="browser-considerations"></a>Consideraciones sobre el explorador

#### <a name="mozilla-firefox-all-platforms"></a>Mozilla Firefox (todas las plataformas)

Mozilla Firefox no realiza automáticamente la autenticación Kerberos. Cada usuario debe agregar manualmente la dirección URL de Azure AD a su configuración de Firefox mediante estos pasos:
1. Ejecute Firefox y escriba `about:config` en la barra de direcciones. Descarte las notificaciones que vea.
2. Busque la preferencia **network.negotiate-auth.trusted-URI**. Esta preferencia enumera los sitios de confianza de Firefox para la autenticación Kerberos.
3. Haga clic con el botón derecho y seleccione **Modificar**.
4. Escriba `https://autologon.microsoftazuread-sso.com` en el campo.
5. Seleccione **Aceptar** y después vuelva a abrir el explorador.

#### <a name="safari-macos"></a>Safari (macOS)

Asegúrese de que la máquina que ejecuta macOS se ha unido a AD. Las instrucciones para unir su dispositivo macOS a AD están fuera del ámbito de este artículo.

#### <a name="microsoft-edge-based-on-chromium-all-platforms"></a>Microsoft Edge basado en Chromium (todas las plataformas)

Si ha reemplazado la configuración de las directivas [AuthNegotiateDelegateAllowlist](https://docs.microsoft.com/DeployEdge/microsoft-edge-policies#authnegotiatedelegateallowlist) o [AuthServerAllowlist](https://docs.microsoft.com/DeployEdge/microsoft-edge-policies#authserverallowlist) en el entorno, asegúrese de agregar también la dirección URL de Azure AD (`https://autologon.microsoftazuread-sso.com`).

#### <a name="microsoft-edge-based-on-chromium-macos-and-other-non-windows-platforms"></a>Microsoft Edge basado en Chromium (macOS y otras plataformas que no son de Windows)

En Microsoft Edge basado en Chromium en macOS y otras plataformas que no son de Windows, consulte la [lista de directivas de Microsoft Edge basado en Chromium](https://docs.microsoft.com/DeployEdge/microsoft-edge-policies#authserverallowlist) para obtener información sobre cómo agregar la dirección URL de Azure AD para la autenticación integrada a la lista de permitidos.

#### <a name="google-chrome-all-platforms"></a>Google Chrome (todas las plataformas)

Si ha reemplazado la configuración de las directivas [AuthNegotiateDelegateWhitelist](https://www.chromium.org/administrators/policy-list-3#AuthNegotiateDelegateWhitelist) o [AuthServerWhitelist](https://www.chromium.org/administrators/policy-list-3#AuthServerWhitelist) en su entorno, asegúrese de agregar también la dirección URL de Azure AD (`https://autologon.microsoftazuread-sso.com`).

#### <a name="google-chrome-macos-and-other-non-windows-platforms"></a>Google Chrome (macOS y otras plataformas que no son Windows)

En el caso de Google Chrome para macOS y otras plataformas que no son de Windows, consulte la [lista de directivas del proyecto Chromium](https://dev.chromium.org/administrators/policy-list-3#AuthServerWhitelist) para obtener información sobre cómo controlar la lista de permitidos para la dirección URL de Azure AD para la autenticación integrada.

El uso de las extensiones de directiva de grupo de Active Directory de terceros para implementar la dirección URL de AD Azure de Firefox y Google Chrome para los usuarios de Mac está fuera del ámbito de este artículo.

#### <a name="known-browser-limitations"></a>Limitaciones de exploradores conocidos

La opción SSO de conexión directa no funciona en modo de exploración privada en los navegadores Firefox y Microsoft Edge. Tampoco funciona en Internet Explorer si el navegador se ejecuta en modo de protección mejorada. En el caso de la versión siguiente de Microsoft Edge basado en Chromium, no funciona en modo InPrivate e Invitado por diseño.

## <a name="step-4-test-the-feature"></a>Paso 4: Prueba de la característica

Para probar la característica con un usuario específico, asegúrese de que se cumplen todas las condiciones siguientes:
  - El usuario inicia sesión en un dispositivo corporativo.
  - El dispositivo se une a su dominio de Active Directory. El dispositivo _no_ debe estar [unido a Azure AD](../active-directory-azureadjoin-overview.md).
  - El dispositivo tiene una conexión directa a su controlador de dominio (DC), en la red cableada o inalámbrica de la empresa o mediante una conexión de acceso remoto, como una conexión VPN.
  - Ha [implantado la característica](#step-3-roll-out-the-feature) para este usuario mediante la directiva de grupo.

Para probar el escenario en el que el usuario escribe solo su nombre de usuario, pero no la contraseña:
   - Inicie sesión en `https://myapps.microsoft.com/` es una sesión de explorador privada nueva.

Para probar el escenario en el que el usuario no tiene que escribir ni su nombre de usuario ni su contraseña, realice alguno de estos pasos: 
   - Inicie sesión en `https://myapps.microsoft.com/contoso.onmicrosoft.com` es una sesión de explorador privada nueva. Reemplace *contoso* con el nombre de su inquilino.
   - Inicie sesión en `https://myapps.microsoft.com/contoso.com` es una sesión de explorador privada nueva. Reemplace *contoso.com* con un dominio comprobado (no un dominio federado) en el inquilino.

## <a name="step-5-roll-over-keys"></a>Paso 5: Sustitución de claves

En el paso 2, Azure AD Connect crea cuentas de equipo (que representan a Azure AD) en todos los bosques de Active Directory en que se ha habilitado SSO de conexión directa. Para más información, consulte [Azure Active Directory Seamless Single Sign-On: Technical deep dive](how-to-connect-sso-how-it-works.md) (Inicio de sesión único de conexión directa de Azure Active Directory: información técnica detallada).

>[!IMPORTANT]
>Si se pierde la clave de descifrado de Kerberos de la cuenta de un equipo, se puede usar para generar vales de Kerberos para todos los usuarios de su bosque de AD. En ese caso, actores malintencionados pueden suplantar los inicios de sesión de Azure AD de los usuarios en peligro. Se recomienda encarecidamente cambiar las claves de descifrado de Kerberos de manera periódica (al menos cada 30 días).

Para obtener instrucciones sobre cómo sustituir las claves, consulte [Azure Active Directory Seamless Single Sign-On: Frequently asked questions](how-to-connect-sso-faq.md) (inicio de sesión único de conexión directa de Azure Active Directory: preguntas más frecuentes). Estamos trabajando en una funcionalidad para introducir el cambio automático de claves.

>[!IMPORTANT]
>No es necesario realizar este paso _inmediatamente_ después de haber habilitado la característica. Sustituya las claves de descifrado de Kerberos al menos cada treinta días.

## <a name="next-steps"></a>Pasos siguientes

- [Profundización técnica](how-to-connect-sso-how-it-works.md): comprenda cómo funciona la característica de inicio de sesión único de conexión directa.
- [Preguntas más frecuentes](how-to-connect-sso-faq.md): obtenga respuestas a las preguntas más frecuentes sobre el inicio de sesión único de conexión directa.
- [Solución de problemas](tshoot-connect-sso.md): aprenda a resolver problemas comunes con la característica de inicio de sesión único de conexión directa.
- [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect): Use el foro de Azure Active Directory para solicitar nuevas características.
