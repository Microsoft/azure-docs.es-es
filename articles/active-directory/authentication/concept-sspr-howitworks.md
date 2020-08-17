---
title: 'Profundización en el autoservicio de restablecimiento de contraseña: Azure Active Directory'
description: ¿Cómo funciona el autoservicio de restablecimiento de contraseña?
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 89d9d06433e2b915b8a96375bb39157adbce6ef2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87027689"
---
# <a name="how-it-works-azure-ad-self-service-password-reset"></a>Funcionamiento: Autoservicio de restablecimiento de contraseña de Azure AD

El autoservicio de restablecimiento de contraseña (SSPR) de Azure Active Directory (Azure AD) ofrece a los usuarios la posibilidad de cambiar o restablecer su contraseña sin necesidad de que intervenga el administrador o el departamento de soporte técnico. Si la cuenta de un usuario está bloqueada o se ha olvidado su contraseña, puede seguir las indicaciones para desbloquearse y volver al trabajo. Esta capacidad reduce las llamadas al departamento de soporte técnico y la pérdida de productividad cuando un usuario no puede iniciar sesión en su dispositivo o en una aplicación.

> [!IMPORTANT]
> Este artículo teórico explica al administrador cómo funciona el autoservicio de restablecimiento de contraseña. Si es un usuario final registrado para el autoservicio de restablecimiento de contraseña y necesita volver a su cuenta, visite [https://aka.ms/sspr](https://aka.ms/sspr).
>
> Si el equipo de TI no ha habilitado la capacidad para restablecer su propia contraseña, póngase en contacto con el departamento de soporte técnico para obtener ayuda adicional.

## <a name="how-does-the-password-reset-portal-work"></a>¿Cómo funciona el portal de restablecimiento de contraseñas?

Un usuario puede restablecer o cambiar su contraseña desde el [portal de SSPR](https://aka.ms/sspr). Primero es necesario que haya registrado los métodos de autenticación que desee utilizar. Cuando un usuario accede al portal de SSPR, la plataforma Azure tiene en cuenta las siguientes cuestiones:

* ¿Cómo se debe localizar la página?
* ¿Es válida la cuenta de usuario?
* ¿A qué organización pertenece el usuario?
* ¿Dónde se administra la contraseña del usuario?
* ¿Tiene el usuario licencia para usar la característica?

Cuando un usuario selecciona el vínculo **No se puede tener acceso a la cuenta** desde una aplicación o página, o bien accede directamente a [https://aka.ms/sspr](https://passwordreset.microsoftonline.com), el idioma utilizado en el portal de SSPR se basa en las siguientes opciones:

* De forma predeterminada, se utiliza la configuración regional del explorador para mostrar el portal de SSPR en el idioma correspondiente. La experiencia de restablecimiento de contraseña está traducida a los mismos idiomas que [admite Office 365](https://support.microsoft.com/office/what-languages-is-office-available-in-26d30382-9fba-45dd-bf55-02ab03e2a7ec).
* Si desea crear un vínculo al portal de SSPR en un idioma traducido específico, anexe `?mkt=` al final de la dirección URL de restablecimiento de contraseña, junto con la configuración regional necesaria.
    * Por ejemplo, para especificar la configuración regional de español *es-us*, utilice `?mkt=es-us` - [https://passwordreset.microsoftonline.com/?mkt=es-us](https://passwordreset.microsoftonline.com/?mkt=es-us).

Una vez que el portal de SSPR se muestra en el idioma necesario, se solicita al usuario que especifique un Id. de usuario y verifique un captcha. Azure AD verifica que el usuario pueda utilizar SSPR. Para ello, lleva a cabo las siguientes comprobaciones:

* Comprueba que el usuario tenga habilitado SSPR y asignada una licencia de Azure AD.
  * Si el usuario no tiene habilitado SSPR o una licencia asignada, se le solicitará que se ponga en contacto con el administrador para restablecer la contraseña.
* Comprueba que el usuario tiene los métodos de comprobación correctos definidos en la cuenta según la directiva del administrador.
  * Si la directiva requiere un único método de autenticación, comprueba que el usuario tenga definidos los datos adecuados para al menos uno de los métodos de autenticación habilitados por la directiva del administrador.
    * Si los métodos de autenticación no están configurados, es aconsejable que el usuario se ponga en contacto con el administrador para restablecer la contraseña.
  * Si la directiva requiere dos métodos, comprueba que el usuario tiene definidos los datos adecuados para al menos dos de los métodos de autenticación habilitados por la directiva del administrador.
    * Si los métodos de autenticación no están configurados, es aconsejable que el usuario se ponga en contacto con el administrador para restablecer la contraseña.
  * Si se asigna un rol de administrador de Azure al usuario, se aplica la directiva de contraseñas seguras de dos puertas. Para más información, consulte [Diferencias entre directivas de restablecimiento de administrador](concept-sspr-policy.md#administrator-reset-policy-differences).
* Comprueba si la contraseña del usuario se administra localmente (por ejemplo, como cuando un inquilino de Azure AD utiliza la autenticación federada de paso a través, o bien la sincronización de hash de contraseña):
  * Si la escritura diferida de SSPR se ha configurado y la contraseña del usuario se administra localmente, el usuario puede continuar con la autenticación y restablecer la contraseña.
  * Si la escritura diferida de SSPR no se ha implementado y la contraseña del usuario se administra localmente, se solicitará al usuario que se ponga en contacto con el administrador para restablecer la contraseña.

Si todas las comprobaciones anteriores se han completado correctamente, se guiará al usuario a través del proceso de restablecimiento o cambio de contraseña.

Para empezar a trabajar con SSPR, complete el siguiente tutorial:

> [!div class="nextstepaction"]
> [Tutorial: Habilitación del autoservicio de restablecimiento de contraseña (SSPR)](tutorial-enable-sspr.md)

## <a name="registration-options"></a>Opciones de registro

Antes de que los usuarios puedan restablecer o cambiar la contraseña mediante SSPR, es necesario que se registren, junto con los métodos de autenticación que se van a utilizar. Como se mencionó en la sección anterior, el usuario debe estar registrado para SSPR y tener aplicada la licencia adecuada.

### <a name="require-users-to-register-when-they-sign-in"></a>Exigir a los usuarios que se registren al iniciar sesión

Esta opción se puede habilitar para exigir que un usuario complete el registro SSPR al iniciar sesión en cualquier aplicación mediante Azure AD. Este flujo de trabajo incluye las siguientes aplicaciones:

* Office 365
* Portal de Azure
* Panel de acceso
* Aplicaciones federadas
* Aplicaciones personalizadas mediante Azure AD

Cuando el registro no sea un requisito, no se instará a los usuarios a completar el registro durante el inicio de sesión, pero podrán registrarse manualmente. Los usuarios pueden visitar [https://aka.ms/ssprsetup](https://aka.ms/ssprsetup), o bien seleccionar el vínculo **Registrarme para restablecer la contraseña** en la pestaña **Perfil** del Panel de acceso.

![Opciones de registro para SSPR en Azure Portal][Registration]

> [!NOTE]
> Para descartar el portal de registro SSPR, los usuarios tienen que seleccionar **Cancelar** o cerrar la ventana. Sin embargo, se les solicitará que se registren cada vez que inician sesión hasta que completen el registro.
>
> Esta interrupción para solicitar el registro no afecta a la conexión del usuario si ya ha iniciado sesión.

### <a name="set-the-number-of-days-before-users-are-asked-to-reconfirm-their-authentication-information"></a>Establecer el número de días que pasan hasta que se pide a los usuarios que vuelvan a confirmar su información de autenticación

Para asegurarse de que los métodos de autenticación sean correctos cuando es necesario restablecer o cambiar la contraseña de los usuarios, puede exigir que estos confirmen la información que hayan registrado transcurrido un periodo de tiempo determinado. Esta opción solo está disponible si habilita la opción **Exigir a los usuarios que se registren al iniciar sesión**.

Los valores válidos para solicitar a un usuario que confirme los métodos que ha registrado abarcan de *0* a *730* días. Si el valor se establece en *0*, nunca se solicitará a los usuarios que confirmen la información de autenticación.

## <a name="authentication-methods"></a>Métodos de autenticación

Cuando un usuario está habilitado para SSPR, tiene que registrar al menos un método de autenticación. Es muy recomendable elegir dos o más métodos de autenticación. De este modo, los usuarios tendrán más flexibilidad en el caso de que no puedan acceder a uno de los métodos cuando lo necesiten. Para más información, consulte [¿Qué son los métodos de autenticación?](concept-authentication-methods.md).

Están disponibles los siguientes métodos de autenticación:

* Notificación en aplicación móvil
* Código de aplicación móvil
* Email
* Teléfono móvil
* Teléfono del trabajo
* Preguntas de seguridad

Los usuarios solo pueden restablecer su contraseña si tienen registrado un método de autenticación que el administrador haya habilitado.

> [!WARNING]
> Las cuentas que tienen asignados roles de *administrador* de Azure deben utilizar los métodos definidos en la sección [Diferencias entre directivas de restablecimiento de administrador](concept-sspr-policy.md#administrator-reset-policy-differences).

![Selección de métodos de autenticación en Azure Portal][Authentication]

### <a name="number-of-authentication-methods-required"></a>Número de métodos de autenticación requeridos

El número de métodos de autenticación disponibles que un usuario debe proporcionar para restablecer o desbloquear su contraseña se puede configurar. El valor se puede establecer en *uno* o *dos*.

Los usuarios pueden y deben registrar varios métodos de autenticación. Nuevamente, es muy recomendable que los usuarios registren dos o más métodos de autenticación para tener más flexibilidad en el caso de que no puedan acceder a uno de los métodos cuando lo necesiten.

Si un usuario no ha registrado el número mínimo de métodos necesarios, cuando intente utilizar SSPR, verá una página de error que le redirigirá para solicitar que un administrador restablezca su contraseña. Tenga cuidado si cambia de uno a dos el número de métodos necesarios cuando haya usuarios registrados para SSPR, puesto que podrían tener dificultades para utilizar la característica. Para más información, consulte la sección siguiente para [cambiar los métodos de autenticación](#change-authentication-methods).

#### <a name="mobile-app-and-sspr"></a>Aplicación móvil y SSPR

Si se utiliza una aplicación móvil (por ejemplo, Microsoft Authenticator) como método para restablecer la contraseña, deben tenerse en cuenta las siguientes consideraciones:

* Cuando los administradores requieren que se utilice un método para restablecer una contraseña, el código de verificación es la única opción disponible.
* Cuando los administradores requieren que se utilicen dos métodos para restablecer una contraseña, los usuarios podrán utilizar una notificación, **o bien** un código de verificación, además de cualquier otro método habilitado.

| Número de métodos requeridos para el restablecimiento | Uno | Dos |
| :---: | :---: | :---: |
| Características de las aplicaciones móviles disponibles | Código | Código o notificación |

Los usuarios no tendrán la opción de registrar su aplicación móvil cuando se registren en el autoservicio de restablecimiento de contraseña desde [https://aka.ms/ssprsetup](https://aka.ms/ssprsetup). Pueden registrar su aplicación móvil en [https://aka.ms/mfasetup](https://aka.ms/mfasetup) o en la página de registro de información de seguridad combinada en [https://aka.ms/setupsecurityinfo](https://aka.ms/setupsecurityinfo).

> [!IMPORTANT]
> La aplicación autenticadora no se puede seleccionar como único método de autenticación cuando se requiere un único método. Tampoco es posible seleccionar la aplicación autenticadora y un único método adicional cuando se requieren dos métodos.
>
> Al configurar directivas de SSPR que incluyan la aplicación autenticadora como método, es necesario seleccionar al menos un método adicional cuando se requiere un método, y al menos dos métodos adicionales cuando se requieren dos métodos.
>
> Este requisito se debe a que la experiencia de registro SSPR actual no ofrece la posibilidad de registrar la aplicación autenticadora. La opción para registrar la aplicación autenticadora se ofrece con la nueva [experiencia de registro combinada](concept-registration-mfa-sspr-converged.md).
>
> Si se admiten directivas que solo usan la aplicación autenticadora (cuando se requiere un método), o bien la aplicación autenticadora y un único método adicional (cuando se requieren dos métodos), es posible que los usuarios no puedan registrarse para SSPR hasta que se modifique la configuración para que utilicen la nueva experiencia de registro combinada.

### <a name="change-authentication-methods"></a>Cambio de métodos de autenticación

¿Qué sucede si empieza con una directiva que solo tiene registrado un método de autenticación requerido para el restablecimiento o el desbloqueo y cambia a dos métodos?

| Número de métodos registrados | Número de métodos requeridos | Resultado |
| :---: | :---: | :---: |
| 1 o más | 1 | **Permite** restablecer o desbloquear |
| 1 | 2 | **No permite** restablecer o desbloquear |
| 2 o más | 2 | **Permite** restablecer o desbloquear |

Cambiar los métodos de autenticación disponibles también puede plantear problemas a los usuarios. Si cambia los tipos de métodos de autenticación que puede usar un usuario, es posible que impida sin darse cuenta que los usuarios puedan usar SSPR si no tienen la cantidad mínima de datos disponibles.

Considere el escenario de ejemplo siguiente:

1. La directiva original se configura con dos métodos de autenticación necesarios. Solo usa el número de teléfono de la oficina y las preguntas de seguridad.
1. El administrador cambia la directiva para dejar de usar las preguntas de seguridad, pero permite el uso de un teléfono móvil y de un correo electrónico alternativo.
1. Los usuarios que tengan vacíos los campos de teléfono móvil o correo electrónico alternativo en este momento no podrán restablecer sus contraseñas.

## <a name="notifications"></a>Notificaciones

Para mejorar el reconocimiento de los eventos de contraseña, SSPR permite configurar notificaciones para los usuarios y los administradores de identidades.

### <a name="notify-users-on-password-resets"></a>¿Quiere notificar a los usuarios los restablecimientos de contraseña?

Si esta opción se ha establecida en **Sí**, los usuarios que restablezcan la contraseña recibirán un correo electrónico para informarles de que la contraseña se ha cambiado. El correo electrónico se envía a través del portal de SSPR a las direcciones de correo electrónico principal y alternativa que se hayan almacenado en Azure AD. A ningún otro usuario se le informa del evento de restablecimiento.

### <a name="notify-all-admins-when-other-admins-reset-their-passwords"></a>Notificación a todos los administradores cuando otros administradores restablecen sus contraseñas

Si esta opción se ha establecido en **Sí**, todos los administradores de Azure recibirán un correo electrónico en la dirección de correo electrónico principal que tienen almacenada en Azure AD. En el correo electrónico se les notifica que otro administrador ha cambiado su contraseña mediante SSPR.

Considere el escenario de ejemplo siguiente:

* hay cuatro administradores en un entorno.
* El administrador *A* restablece la contraseña de los usuarios mediante SSPR.
* Los administradores *B*, *C* y *D* recibirán un correo electrónico para informarles del restablecimiento de contraseña.

## <a name="on-premises-integration"></a>Integración local

Si utiliza un entorno híbrido, puede configurar Azure AD Connect para volver a escribir eventos de cambio de contraseña desde Azure AD a un directorio local.

![Validación de escritura diferida de contraseñas habilitada en funcionamiento][Writeback]

Azure AD comprueba la conectividad híbrida actual y muestra uno de los siguientes mensajes en Azure Portal:

* El cliente de escritura diferida local está en funcionamiento.
* Azure AD está en línea y conectado al cliente de escritura diferida local. Sin embargo, parece que la versión instalada de Azure AD Connect no está actualizada. Considere la posibilidad de [actualizar Azure AD Connect](../hybrid/how-to-upgrade-previous-version.md) para asegurarse de que tiene las características de conectividad más recientes y las correcciones de errores importantes.
* Lamentablemente, no podemos comprobar el estado del cliente de escritura diferida local porque la versión instalada de Azure AD Connect está obsoleta. [Actualice Azure AD Connect](../hybrid/how-to-upgrade-previous-version.md) para poder comprobar su estado de conexión.
* Desafortunadamente, parece que no podemos conectarnos a su cliente de escritura diferida local ahora mismo. [Solucione problemas de Azure AD Connect](active-directory-passwords-troubleshoot.md#troubleshoot-password-writeback-connectivity) para restaurar la conexión.
* Desafortunadamente, no podemos conectarnos a su cliente de escritura diferida local porque la escritura diferida de contraseñas no se ha configurado correctamente. [Configure la escritura diferida de contraseñas](howto-sspr-writeback.md) para restaurar la conexión.
* Desafortunadamente, parece que no podemos conectarnos a su cliente de escritura diferida local ahora mismo. Esto puede deberse a problemas temporales por nuestra parte. Si el problema persiste, [solucione los problemas de Azure AD Connect](active-directory-passwords-troubleshoot.md#troubleshoot-password-writeback-connectivity) para restaurar la conexión.

Para empezar a trabajar con la escritura diferida de SSPR, realice el siguiente tutorial:

> [!div class="nextstepaction"]
> [Tutorial: Habilitación de la escritura diferida del autoservicio de restablecimiento de contraseña (SSPR)](tutorial-enable-writeback.md).

### <a name="write-back-passwords-to-your-on-premises-directory"></a>Escritura diferida de contraseñas en un directorio local

Puede habilitar la escritura diferida de contraseñas desde Azure Portal. También puede deshabilitar temporalmente la escritura diferida de contraseñas sin necesidad de volver a configurar Azure AD Connect.

* Si la opción se ha establecido en **Sí**, la escritura diferida estará habilitada. Los usuarios que utilicen la autenticación federada de paso a través o la sincronización de hash de contraseña podrán restablecer sus contraseñas.
* Si la opción se ha establecido en **No**, la escritura diferida estará deshabilitada. Los usuarios que utilicen la autenticación federada de paso a través o la sincronización de hash de contraseña no podrán restablecer sus contraseñas.

### <a name="allow-users-to-unlock-accounts-without-resetting-their-password"></a>Permitir a los usuarios desbloquear las cuentas sin restablecer la contraseña

De forma predeterminada, Azure AD desbloquea las cuentas cuando se realiza un restablecimiento de contraseña. Para proporcionar flexibilidad, puede permitir que los usuarios desbloqueen sus cuentas locales sin tener que restablecer la contraseña. Utilice esta opción para separar esas dos operaciones.

* Si se establece en **Sí**, los usuarios tienen la opción de restablecer su contraseña y desbloquear la cuenta, o bien de desbloquear la cuenta sin tener que restablecer la contraseña.
* Si se establece en **No**, los usuarios solo pueden realizar una operación combinada de restablecimiento de contraseña y desbloqueo de cuenta.

### <a name="on-premises-active-directory-password-filters"></a>Filtros de la contraseña de Active Directory local

SSPR efectúa una operación equivalente a un restablecimiento de contraseña iniciado por el administrador en Active Directory. Si utiliza un filtro de contraseñas de terceros para aplicar reglas de contraseñas personalizadas y requiere que este filtro de contraseñas se compruebe durante el autoservicio de restablecimiento de contraseña de Azure AD, asegúrese de que el filtro de contraseña de terceros se haya configurado para utilizarse en el escenario de restablecimiento de contraseña del administrador. De forma predeterminada, se proporciona compatibilidad con la [protección con contraseña de Azure AD para Windows Server Active Directory](concept-password-ban-bad-on-premises.md).

## <a name="password-reset-for-b2b-users"></a>Restablecimiento de contraseña para usuarios B2B

El restablecimiento y el cambio de contraseña son totalmente compatibles con todas las configuraciones negocio a negocio (B2B). Se admiten los tres casos siguientes para el restablecimiento de contraseña de usuario B2B:

* **Usuarios de una organización asociada con un inquilino de Azure AD existente**: Si la organización con la que colabora tiene un inquilino de Azure AD, se respetarán todas las directivas de restablecimiento de contraseña que estén habilitadas en dicho inquilino. Para que el restablecimiento de contraseña funcione, la organización asociada solo tiene que asegurarse de que SSPR de Azure AD está habilitado. No tiene costo adicional para los clientes de Office 365.
* **Usuarios que se registran mediante el** registro de autoservicio: si la organización con la que colabora utilizó la característica de [registro de autoservicio](../users-groups-roles/directory-self-service-signup.md) para acceder a un inquilino, se permitirá que restablezca la contraseña con el correo electrónico que hubiera registrado.
* **Usuarios B2B**: cualquier nuevo usuario B2B creado con la nueva [funcionalidad B2B de Azure AD](../b2b/what-is-b2b.md) podrá restablecer su contraseña con el correo electrónico que haya registrado durante el proceso de invitación.

Para probar este escenario, vaya a https://passwordreset.microsoftonline.com con uno de estos usuarios asociados. Si tienen un correo electrónico alternativo o de autenticación definido, el restablecimiento de contraseña funcionará según lo esperado.

> [!NOTE]
> Las cuentas de Microsoft que tengan acceso de invitado para su inquilino de Azure AD, como las de Hotmail.com, Outlook.com u otras direcciones de correo electrónico personales, no pueden utilizar SSPR de Azure AD. Para restablecer su contraseña, se debe usar la información que se encuentra en el artículo [Cuando no puedes iniciar sesión en tu cuenta Microsoft](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant).

## <a name="next-steps"></a>Pasos siguientes

Para empezar a trabajar con SSPR, complete el siguiente tutorial:

> [!div class="nextstepaction"]
> [Tutorial: Habilitación del autoservicio de restablecimiento de contraseña (SSPR)](tutorial-enable-sspr.md)

En los siguientes artículos se proporciona información adicional sobre el restablecimiento de contraseña con Azure AD:

[Authentication]: ./media/concept-sspr-howitworks/manage-authentication-methods-for-password-reset.png "Métodos de autenticación de Azure AD disponibles y cantidad requerida"
[Registration]: ./media/concept-sspr-howitworks/configure-registration-options.png "Configuración de opciones de registro SSPR en Azure Portal"
[Writeback]: ./media/concept-sspr-howitworks/on-premises-integration.png "Integración local para SSPR en Azure Portal"
