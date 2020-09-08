---
title: Planeamiento y solución de problemas de los cambios de nombre principal de usuario (UPN) de Azure
description: Descripción de los problemas conocidos y sus mitigaciones relacionadas con los cambios de UPN
services: active-directory
ms.service: active-directory
ms.subservice: hybrid
ms.topic: how-to
ms.date: 03/13/2020
ms.author: baselden
author: barbaraselden
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.openlocfilehash: caf0fdf5fd167987ea0fd7111a05b04bd5bf848f
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89279795"
---
# <a name="plan-and-troubleshoot-user-principal-name-changes-in-azure-active-directory"></a>Planeamiento y solución de problemas de los cambios de nombre principal de usuario en Azure Active Directory

Un nombre principal de usuario (UPN) es un atributo que es un estándar de comunicación de Internet para las cuentas de usuario. Un UPN consta de un prefijo de UPN (el nombre de la cuenta de usuario) y un sufijo de UPN (un nombre de dominio DNS). El prefijo se une al sufijo mediante el símbolo "@". Por ejemplo, someone@example.com. Un UPN debe ser único entre todos los objetos de entidad de seguridad dentro de un bosque de directorio. 

**En este artículo se presupone que usa UPN como el identificador de usuario. Aquí se habla del planeamiento de los cambios de UPN y de la recuperación a partir de los problemas que puedan aparecer debido a los cambios de UPN.**

> [!NOTE]
> Para los desarrolladores, se recomienda usar el objectID del usuario como identificador inmutable, en lugar del UPN o direcciones de correo electrónico, ya que sus valores pueden cambiar.


## <a name="learn-about-upns-and-upn-changes"></a>Más información sobre los UPN y los cambios de UPN
Con frecuencia, las páginas de inicio de sesión solicitan a los usuarios que escriban su dirección de correo electrónico cuando el valor requerido es, en realidad, su UPN. Por lo tanto, debe asegurarse de cambiar el UPN de los usuarios cada vez que cambie su dirección de correo electrónico principal.

Las direcciones de correo electrónico principales de los usuarios pueden cambiar por muchos motivos:

* Cambio de la imagen corporativa.

* Movimiento de los empleados a distintas divisiones de la empresa. 

* Fusiones y adquisiciones.

* Cambios de nombres de los empleados.

### <a name="types-of-upn-changes"></a>Tipos de cambios de UPN

Para cambiar un UPN, puede cambiar el prefijo, el sufijo o ambos.

* **Cambio del prefijo**.

   *  Por ejemplo, si cambia el nombre de una persona, puede cambiar su nombre de cuenta:  
‎BSimon@contoso.com a BJohnson@contoso.com

   * También puede cambiar el estándar corporativo para los prefijos:  
‎Bsimon@contoso.com a Britta.Simon@contoso.com

* **Cambio del sufijo**. <br>

    Por ejemplo, si una persona se cambió de división, es posible cambiar su dominio: 

   * Britta.Simon@contoso.com a Britta.Simon@contosolabs.com <br>
     Or<br>
    * Britta.Simon@corp.contoso.com a Britta.Simon@labs.contoso.com 

Se recomienda cambiar el UPN de los usuarios cada vez que se actualice la dirección de correo electrónico principal.

Durante la sincronización inicial desde Active Directory a Azure AD, asegúrese de que los correos electrónicos de los usuarios sean idénticos a sus UPN.

### <a name="upns-in-active-directory"></a>UPN en Active Directory

En Active Directory, el sufijo UPN predeterminado es el nombre DNS del dominio en el que se creó la cuenta de usuario. En la mayoría de los casos, se trata del nombre de dominio que registra como el dominio de la empresa en Internet. Si crea la cuenta de usuario en el dominio contoso.com, el UPN predeterminado es

username@contoso.com

 Sin embargo, puede [agregar más sufijos UPN](../fundamentals/add-custom-domain.md) mediante Dominios y confianzas de Active Directory. 

Por ejemplo, puede que quiera agregar labs.contoso.com y que los UPN y los correos electrónicos de los usuarios lo reflejen. De ese modo, se convertirían en

username@labs.contoso.com.

>[!IMPORTANT]
> Si [cambia el sufijo en Active Directory](../fundamentals/add-custom-domain.md), debe asegurarse de que se haya [agregado y comprobado en Azure AD](../fundamentals/add-custom-domain.md) un nombre de dominio personalizado que coincida. 

![Captura de pantalla de los dominios comprobados](./media/howto-troubleshoot-upn-changes/custom-domains.png)

### <a name="upns-in-azure-active-directory"></a>UPN en Azure Active Directory

Los usuarios inician sesión en Azure AD con el valor en el atributo userPrincipalName. 

Cuando se usa Azure AD junto con Active Directory local, las cuentas de usuario se sincronizan mediante el servicio de Azure AD Connect. De manera predeterminada, el asistente de Azure AD Connect usa el atributo userPrincipalName de Active Directory local como el UPN en Azure AD. Puede cambiarlo a un atributo distinto en una instalación personalizada.

Es importante que tenga un proceso definido al actualizar un nombre principal de usuario (UPN) de un usuario único o para toda la organización. 

Consulte los problemas conocidos y sus soluciones que aparecen en este documento.

Cuando sincronice las cuentas de usuario de Active Directory a Azure AD, asegúrese de que los UPN de Active Directory se asignen a los dominios comprobados de Azure AD.

![Captura de pantalla de los dominios comprobados](./media/howto-troubleshoot-upn-changes/verified-domains.png)

Si el valor del atributo userPrincipalName no corresponde a un dominio comprobado de Azure AD, el proceso de sincronización reemplaza el sufijo con un valor predeterminado .onmicrosoft.com.


### <a name="roll-out-bulk-upn-changes"></a>Deshacer cambios masivos de UPN

Siga los [procedimientos recomendados para un piloto](../fundamentals/active-directory-deployment-plans.md) para realizar cambios masivos de UPN. También debe tener un plan de reversión comprobado para revertir los UPN si se encuentra con problemas que no se puede resolver de manera rápida. Una vez que se ejecuta el piloto, puede empezar a dirigirse a pequeños conjuntos de usuarios con varios roles de organización y sus conjuntos específicos de aplicaciones o dispositivos.

Pasar por este primer subconjunto de usuarios le proporcionará una buena idea de lo que los usuarios deben esperar como parte del cambio. Incluya esta información en las comunicaciones de los usuarios.

Cree un procedimiento definido para cambiar los UPN en usuarios individuales como parte de las operaciones normales. Se recomienda tener un procedimiento comprobado que incluya documentación sobre problemas conocidos y soluciones alternativas.

En las secciones siguientes se detallan posibles problemas conocidos y soluciones alternativas para cuando se cambian los UPN.

## <a name="apps-known-issues-and-workarounds"></a>Problemas conocidos de aplicaciones y soluciones alternativas

Las aplicaciones de [software como servicio (SaaS)](https://azure.microsoft.com/overview/what-is-saas/) y de línea de negocio (LoB) se suelen basar en los UPN para buscar usuarios y almacenan la información del perfil de usuario, incluidos los roles. Los cambios de UPN pueden afectar las aplicaciones que usan el [aprovisionamiento Just in Time](../app-provisioning/user-provisioning.md) para crear un perfil de usuario cuando los usuarios inician sesión por primera vez.

**Problema conocido**<br>
Cambiar el UPN de un usuario podría interrumpir la relación entre el usuario de Azure AD y el perfil de usuario creado en la aplicación. Si la aplicación usa el [aprovisionamiento Just-in-Time](../app-provisioning/user-provisioning.md), podría crear un perfil de usuario completamente nuevo. Esto requerirá que el administrador de la aplicación haga cambios manuales para corregir esta relación.

**Solución alternativa**<br>
El [aprovisionamiento automático de usuarios de Azure AD](../app-provisioning/user-provisioning.md) le permite crear, mantener y quitar de manera automática las identidades de los usuarios en aplicaciones en la nube compatibles. La configuración del aprovisionamiento automático de usuarios en las aplicaciones actualiza automáticamente los UPN en las aplicaciones. Pruebe las aplicaciones como parte de la implementación progresiva para validar que no se vean afectadas por los cambios de UPN.
Si es un desarrollador, considere la posibilidad de [agregar compatibilidad con SCIM a la aplicación](../app-provisioning/use-scim-to-provision-users-and-groups.md) para habilitar el aprovisionamiento automático de usuarios desde Azure Active Directory. 

## <a name="managed-devices-known-issues-and-workarounds"></a>Problemas conocidos y soluciones alternativas para los dispositivos administrados

Al [traer sus dispositivos a Azure AD](../devices/overview.md), está maximizando la productividad de los usuarios mediante un inicio de sesión único (SSO) en los recursos de nube y del entorno local.

### <a name="azure-ad-joined-devices"></a>Dispositivos unidos a Azure AD

Los dispositivos [unidos a Azure AD](../devices/concept-azure-ad-join.md) se unen directamente a Azure AD y permiten que los usuarios inicien sesión en el dispositivo con la identidad de su organización.

**Problemas conocidos** <br>
Los usuarios pueden experimentar problemas con el inicio de sesión único en aplicaciones que dependen de Azure AD para la autenticación.

**Resolución** <br>
Los problemas que se mencionan en esta sección se han corregido en la actualización 2020 de Windows 10 (2004).

**Solución alternativa** <br>
Deje tiempo suficiente para que el cambio de UPN se sincronice con Azure AD. Una vez que compruebe que el UPN nuevo se refleja en el portal de Azure AD, pídale al usuario que seleccione el icono "Otro usuario" para iniciar sesión con el UPN nuevo. También puede realizar la comprobación mediante [PowerShell](/powershell/module/azuread/get-azureaduser?view=azureadps-2.0). Después de iniciar sesión con el UPN nuevo, es posible que las referencias al UPN anterior sigan apareciendo en la opción de Windows "Obtener acceso a trabajo o escuela".

![Captura de pantalla de los dominios comprobados](./media/howto-troubleshoot-upn-changes/other-user.png)


### <a name="hybrid-azure-ad-joined-devices"></a>Dispositivos híbridos unidos a Azure AD

Los dispositivos híbridos [unidos a Azure AD](../devices/concept-azure-ad-join-hybrid.md) se unen a Active Directory y Azure AD. Puede implementar Azure AD híbrido si el entorno tiene una superficie de Active Directory local y también quiere aprovechar las funcionalidades que proporciona Azure AD.

**Problemas conocidos** 

Es probable que los dispositivos híbridos unidos a Azure AD en Windows 10 experimenten problemas con el acceso y reinicios inesperados.

Si los usuarios inician sesión en Windows antes de que el UPN nuevo se haya sincronizado con Azure AD o siguen usando una sesión de Windows existente, pueden experimentar problemas con el inicio de sesión único en las aplicaciones que usan Azure AD para la autenticación si se configuró el acceso condicional para exigir el uso de dispositivos unidos híbridos para tener acceso a los recursos. 

Además, aparecerá el mensaje siguiente, que forzará un reinicio después de un minuto. 

"El equipo se reiniciará automáticamente en un minuto. Se produjo un problema con Windows y se debe reiniciar. Cierre ahora este mensaje y guarde su trabajo".

**Resolución** <br>
Los problemas que se mencionan en esta sección se han corregido en la actualización 2020 de Windows 10 (2004).

**Solución alternativa** 

Es necesario separar el dispositivo de Azure AD y reiniciarlo. Después del reinicio, el dispositivo se unirá automáticamente a Azure AD de nuevo y el usuario deberá iniciar sesión con el UPN nuevo seleccionando el icono "Otro usuario". Para separar un dispositivo de Azure AD, ejecute el comando siguiente en un símbolo del sistema:

**dsregcmd /leave**

El usuario deberá [volver a inscribirse](/windows/security/identity-protection/hello-for-business/hello-hybrid-cert-whfb-provision) en Windows Hello para empresas si se está usando. Los dispositivos Windows 7 y 8.1 no se ven afectados por este problema después de que los UPN cambian.


## <a name="microsoft-authenticator-known-issues-and-workarounds"></a>Problemas conocidos y soluciones alternativas de Microsoft Authenticator

Es posible que la organización requiera el uso de la [aplicación Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) para iniciar sesión y acceder a los datos y las aplicaciones de la organización. Aunque puede aparecer un nombre de usuario en la aplicación, la cuenta no está configurada para funcionar como método de comprobación hasta que el usuario completa el proceso de registro.

La [aplicación Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) tiene cuatro funciones principales:

* Autenticación multifactor a través de una notificación push u un código de verificación.

* Actuar como agente de autenticación en dispositivos iOS y Android para proporcionar el inicio de sesión único en aplicaciones que usan la [autenticación asincrónica](../develop/brokered-auth.md).

* Registro de dispositivos (también conocido como Workplace Join) en Azure AD, que es requisito para otras características como Intune App Protection e inscripción o administración de dispositivos.

* Inicio de sesión en el teléfono, que requiere MFA y registro de dispositivos.

### <a name="multi-factor-authentication-with-android-devices"></a>Multi-Factor Authentication con dispositivos Android

La aplicación Microsoft Authenticator ofrece una opción de comprobación fuera de banda. En lugar de realizar una llamada de teléfono automatizada o enviar SMS al usuario durante el inicio de sesión, [Multi-Factor Authentication (MFA)](../authentication/concept-mfa-howitworks.md) inserta una notificación en la aplicación Microsoft Authenticator en el smartphone o en la tableta del usuario. El usuario simplemente pulsa Aprobar (o escribe un PIN o datos biométricos y pulsa "Autenticar") en la aplicación para completar el inicio de sesión.

**Problemas conocidos** 

Al cambiar el UPN de un usuario, el UPN anterior se sigue mostrando en la cuenta de usuario y es posible que no se reciba una notificación. Los [códigos de verificación](../user-help/user-help-auth-app-faq.md) siguen funcionando.

**Solución alternativa**

Si se recibe una notificación, indique al usuario que la descarte, abra la aplicación Authenticator, pulse la opción "Comprobar notificaciones" y apruebe el aviso de MFA. Después de esto, se actualizará el UPN que aparece en la cuenta. Tenga en cuenta que es posible que el UPN actualizado se muestre como una cuenta nueva, debido a que se está usando otra funcionalidad de Authenticator. Para más información, consulte los problemas conocidos adicionales que aparecen en este artículo.

### <a name="brokered-authentication"></a>Autenticación asincrónica

En los agentes de Android e iOS, como Microsoft Authenticator, habilite lo siguiente:

* Inicio de sesión único: los usuarios no tendrán que iniciar sesión en cada aplicación.

* Identificación de dispositivos: el agente accede al certificado del dispositivo que se creó en este al unirse al área de trabajo.

* Comprobación de identificación de la aplicación: cuando una aplicación llama al agente, pasa su dirección URL de redireccionamiento y el agente la comprueba.

Además, permite que las aplicaciones participen en características más avanzadas, como el [Acceso condicional](../conditional-access/index.yml), y admite [escenarios de Microsoft Intune](../develop/msal-net-use-brokers-with-xamarin-apps.md).

**Problemas conocidos**<br>
El usuario ve más peticiones de autenticación interactivas en las aplicaciones nuevas que usan el inicio de sesión asistido por agente debido a un error de coincidencia entre el valor de login_hint que pasa la aplicación y el UPN que está almacenado en el agente.

**Solución alternativa** <br> El usuario debe quitar manualmente la cuenta de Microsoft Authenticator y establecer un inicio de sesión nuevo desde una aplicación asistida por agente. La cuenta se agregará automáticamente después de la autenticación inicial.

### <a name="device-registration"></a>Registro de dispositivos

La aplicación Microsoft Authenticator es responsable de registrar el dispositivo en Azure AD. El registro de dispositivos permite que el dispositivo se autentique en Azure AD y es un requisito para los escenarios siguientes:

* Intune App Protection

* Inscripción de dispositivos de Intune

* Inicio de sesión en el teléfono

**Problemas conocidos**<br>
Al cambiar el UPN, aparece una cuenta nueva con el UPN nuevo en la aplicación Microsoft Authenticator, mientras que la cuenta con el UPN anterior todavía aparece en la lista. Además, el UPN anterior aparece en la sección Registro de dispositivos en la configuración de la aplicación. No hay ningún cambio en la funcionalidad normal del Registro de dispositivos ni de los escenarios dependientes.

**Solución alternativa** <br> Para quitar todas las referencias al UPN anterior en la aplicación Microsoft Authenticator, indíquele al usuario que quite manualmente las cuentas antiguas y nuevas de Microsoft Authenticator, que vuelva a registrarse en MFA y que vuelva a unir al dispositivo.

### <a name="phone-sign-in"></a>Inicio de sesión en el teléfono

El inicio de sesión en el teléfono le permite a los usuarios inicia sesión en Azure AD sin contraseña. Para habilitar el inicio de sesión en el teléfono, el usuario debe registrarse en MFA mediante la aplicación Authenticator y, a continuación, habilitar el inicio de sesión en el teléfono directamente en Authenticator. Como parte de la configuración, el dispositivo se registra con Azure AD.

**Problemas conocidos** <br>
Los usuarios no pueden usar el inicio de sesión en el teléfono porque no reciben ninguna notificación. Si el usuario pulsa Comprobar notificaciones, obtiene un error.

**Solución alternativa**<br>
El usuario debe seleccionar el menú desplegable de la cuenta habilitada para el inicio de sesión en el teléfono y seleccionar Deshabilitar inicio de sesión en el teléfono. Si lo desea, puede volver a habilitar el inicio de sesión en el teléfono.

## <a name="security-key-fido2-known-issues-and-workarounds"></a>Problemas conocidos y soluciones alternativas para la clave de seguridad (FIDO2)

**Problemas conocidos** <br>
Cuando varios usuarios se registran en la misma clave, la pantalla de inicio de sesión muestra una página de selección de cuenta en la que aparece el UPN antiguo. Los cambios de UPN no afectan los inicios de sesión mediante claves de seguridad.  

**Solución alternativa**<br>
Para quitar las referencias a los UPN anteriores, los usuarios deben [restablecer la clave de seguridad y volver a registrarse](../authentication/howto-authentication-passwordless-security-key.md#known-issues).

## <a name="onedrive-known-issues-and-workarounds"></a>Problemas conocidos y soluciones alternativas para OneDrive

Se sabe que los usuarios de OneDrive experimentan problemas después de los cambios de UPN. Para más información, consulte el artículo sobre [cómo afectan los cambios de UPN la dirección URL y las características de OneDrive](/onedrive/upn-changes).

## <a name="next-steps"></a>Pasos siguientes

Consulte estos recursos:
* [Azure AD Connect: conceptos de diseño](./plan-connect-design-concepts.md).

* [Rellenado de userPrincipalName de Azure AD](./plan-connect-userprincipalname.md)

* [Tokens de id. de la plataforma de identidad de Microsoft](../develop/id-tokens.md)