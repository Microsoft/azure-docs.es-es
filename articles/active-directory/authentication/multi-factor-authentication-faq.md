---
title: Preguntas más frecuentes sobre Azure Multi-Factor Authentication - Azure Active Directory
description: Preguntas y respuestas más frecuentes relacionadas con Azure Multi-Factor Authentication.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/14/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: d6f5cabace81d53edf36ac6be0a2eb8830e6cc5f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87035084"
---
# <a name="frequently-asked-questions-about-azure-multi-factor-authentication"></a>Preguntas más frecuentes relacionadas con Azure Multi-Factor Authentication

Este artículo de P+F da respuesta a las preguntas más frecuentes sobre Azure Multi-Factor Authentication y el uso del servicio Multi-Factor Authentication. Se divide en preguntas sobre el servicio en general, los modelos de facturación, las experiencias del usuario y la solución de problemas.

> [!IMPORTANT]
> A partir del 1 de julio de 2019, Microsoft ya no ofrecerá el Servidor MFA para implementaciones nuevas. Los clientes nuevos que quieran exigir la autenticación multifactor a sus usuarios deberán usar Azure Multi-Factor Authentication basado en la nube. Los clientes existentes que hayan activado el Servidor MFA antes del 1 de julio podrán descargar la versión más reciente y las actualizaciones futuras, así como generar credenciales de activación como de costumbre.
>
> La información que se comparte a continuación sobre el Servidor Microsoft Azure Multi-Factor Authentication solo es aplicable para los usuarios que ya tienen el Servidor MFA en ejecución.
>
> Las licencias por consumo ya no están disponible para nuevos clientes desde el 1 de septiembre de 2018.
> A partir del 1 de septiembre de 2018, no se pueden crear nuevos clientes de autenticación. Los proveedores de autenticación existentes se pueden seguir usando y actualizando. La autenticación multifactor seguirá siendo una característica disponible en las licencias de Azure AD Premium.

## <a name="general"></a>General

* [¿Cómo controla Servidor Microsoft Azure Multi-Factor Authentication los datos de usuario?](#how-does-azure-multi-factor-authentication-server-handle-user-data)
* [¿Qué códigos cortos de SMS se usan para enviar mensajes SMS a mis usuarios?](#what-sms-short-codes-are-used-for-sending-sms-messages-to-my-users)

### <a name="how-does-azure-multi-factor-authentication-server-handle-user-data"></a>¿Cómo controla Servidor Microsoft Azure Multi-Factor Authentication los datos de usuario?

Con Servidor Multi-Factor Authentication, los datos del usuario se almacenan solo en los servidores locales. Los datos de usuario persistentes no se almacenan en la nube. Cuando el usuario realiza la comprobación en dos pasos, Servidor Multi-Factor Authentication envía datos al servicio en la nube Azure Multi-Factor Authentication para llevar a cabo la autenticación. Para establecer la comunicación entre Servidor Multi-Factor Authentication y el servicio en la nube Multi-Factor Authentication se utilizan los protocolos Capa de sockets seguros (SSL) o Seguridad de la capa de transporte (TLS) en el puerto 443 de salida.

Cuando se envían solicitudes de autenticación al servicio en la nube, se recopilan datos para los informes de uso y autenticación. Los campos de datos siguientes se incluyen en los registros de comprobación en dos pasos:

* **Id. único** (cualquier nombre de usuario o id. de Servidor Multi-Factor Authentication local)
* **Nombre y apellidos** (opcional)
* **Dirección de correo electrónico** (opcional)
* **Número de teléfono** (al realizar la autenticación de una llamada de voz o SMS)
* **Token de dispositivo** (al realizar la autenticación de una aplicación móvil)
* **Modo de autenticación**
* **Resultado de la autenticación**
* **Nombre de Servidor Multi-Factor Authentication**
* **IP de Servidor Multi-Factor Authentication**
* **IP de cliente** (si está disponible)

Los campos opcionales pueden configurarse en Servidor Multi-Factor Authentication.

El resultado de la comprobación (aceptación o denegación) y el motivo de la denegación se almacenan con los datos de autenticación. Los datos están disponibles en los informes de autenticación y uso.

Para obtener más información, consulte [Residencia de datos y datos de cliente en Azure Multi-Factor Authentication](concept-mfa-data-residency.md).

### <a name="what-sms-short-codes-are-used-for-sending-sms-messages-to-my-users"></a>¿Qué códigos cortos de SMS se usan para enviar mensajes SMS a mis usuarios?

En Estados Unidos, Microsoft utiliza los siguientes códigos cortos de SMS:

* *97671*
* *69829*
* *51789*
* *99399*

En Canadá, usamos los siguientes códigos cortos de SMS:

* *759731*
* *673801*

No se garantiza la entrega rápida y coherente de Multi-Factor Authentication por SMS o llamada de voz en el mismo número. Por el interés de los usuarios, podemos agregar o quitar códigos cortos cuando lo estimemos oportuno en tanto realicemos ajustes de enrutamiento para mejorar la capacidad de entrega de SMS.

No admitimos códigos cortos para países o regiones que no sean Estados Unidos y Canadá.

## <a name="billing"></a>Facturación

La mayoría de las preguntas sobre facturación se pueden responder si se consulta la [página sobre precios de Multi-Factor Authentication](https://azure.microsoft.com/pricing/details/multi-factor-authentication/) o la documentación sobre [versiones y planes de consumo de Azure Multi-Factor Authentication](concept-mfa-licensing.md).

* [¿Se le aplican cargos a mi organización por el envío de llamadas telefónicas y mensajes de texto que se usan para la autenticación?](#is-my-organization-charged-for-sending-the-phone-calls-and-text-messages-that-are-used-for-authentication)
* [En el modelo de facturación por usuario, ¿los cargos se aplican por todos los usuarios habilitados o solo por los que realizaron la verificación en dos pasos?](#does-the-per-user-billing-model-charge-me-for-all-enabled-users-or-just-the-ones-that-performed-two-step-verification)
* [¿Cómo funciona la facturación de Multi-Factor Authentication?](#how-does-multi-factor-authentication-billing-work)
* [¿Existe una versión gratuita de Azure Multi-Factor Authentication?](#is-there-a-free-version-of-azure-multi-factor-authentication)
* [¿Puede mi organización cambiar entre los modelos de facturación de consumo por usuario y por autenticación en cualquier momento?](#can-my-organization-switch-between-per-user-and-per-authentication-consumption-billing-models-at-any-time)
* [¿Mi organización puede cambiar entre la facturación basada en el consumo y las suscripciones (un modelo basado en licencias) en cualquier momento?](#can-my-organization-switch-between-consumption-based-billing-and-subscriptions-a-license-based-model-at-any-time)
* [¿Mi organización tiene que usar y sincronizar las identidades para usar Azure Multi-Factor Authentication?](#does-my-organization-have-to-use-and-synchronize-identities-to-use-azure-multi-factor-authentication)

### <a name="is-my-organization-charged-for-sending-the-phone-calls-and-text-messages-that-are-used-for-authentication"></a>¿Se le aplican cargos a mi organización por el envío de llamadas telefónicas y mensajes de texto que se usan para la autenticación?

No, no se le cobrará nada por las llamadas de teléfono individuales que se realicen o los mensajes de texto que se envíen a los usuarios cuando se use Azure Multi-Factor Authentication. Si usa un proveedor de MFA por autenticación, se le facturará por cada autenticación pero no por el método que se use.

Podrían cobrarse a los usuarios las llamadas de teléfono o los mensajes de texto que reciban, dependiendo de su servicio telefónico personal.

### <a name="does-the-per-user-billing-model-charge-me-for-all-enabled-users-or-just-the-ones-that-performed-two-step-verification"></a>En el modelo de facturación por usuario, ¿los cargos se aplican por todos los usuarios habilitados o solo por los que realizaron la verificación en dos pasos?

La facturación se basa en la cantidad de usuarios que están configurados para usar Multi-Factor Authentication, sin tener en cuenta si realizaron o no una verificación en dos pasos el mes en cuestión.

### <a name="how-does-multi-factor-authentication-billing-work"></a>¿Cómo funciona la facturación de Multi-Factor Authentication?

Cuando se crea un proveedor de MFA por usuario o por autenticación, se factura mensualmente a la suscripción de Azure de su organización según el uso. Este modelo de facturación es similar a la forma en que Azure factura el uso de máquinas virtuales y Web Apps.

Al adquirir una suscripción para Azure Multi-Factor Authentication, su organización solo paga la cuota de licencia anual para cada usuario. Las licencias MFA así como los conjuntos de productos de Office 365, Azure AD Premium o Enterprise Mobility + Security se facturan de esta manera.

Para más información, consulte [Cómo obtener Azure Multi-Factor Authentication](concept-mfa-licensing.md).

### <a name="is-there-a-free-version-of-azure-multi-factor-authentication"></a>¿Existe una versión gratuita de Azure Multi-Factor Authentication?

Los valores predeterminados de seguridad se pueden habilitar en el nivel Azure AD Free. Con los valores predeterminados de seguridad, todos los usuarios están habilitados para autenticación multifactor mediante la aplicación Microsoft Authenticator. No se puede usar el mensaje de texto ni la comprobación del teléfono con los valores predeterminados de seguridad, solo la aplicación Microsoft Authenticator.

Para obtener más información, vea [¿Cuáles son los valores de seguridad predeterminados?](../fundamentals/concept-fundamentals-security-defaults.md)

### <a name="can-my-organization-switch-between-per-user-and-per-authentication-consumption-billing-models-at-any-time"></a>¿Puede mi organización cambiar entre los modelos de facturación de consumo por usuario y por autenticación en cualquier momento?

Si la organización adquiere MFA como servicio independiente con facturación basada en el consumo, se elige un modelo de facturación cuando se crea un proveedor de MFA. No se puede cambiar el modelo de facturación después de que se crea un proveedor de MFA. 

Si el proveedor de MFA *no* está vinculado a un inquilino de Azure AD o si vincula el nuevo proveedor de MFA a un inquilino de Azure AD diferente, las opciones de configuración y parámetros de usuario no se transferirán. Además, los servidores Azure MFA se tendrán que reactivar mediante las credenciales de activación generadas a través del nuevo proveedor de MFA. Reactivar los servidores MFA para vincularlos al nuevo proveedor de MFA no afecta a la autenticación por llamada telefónica y mensaje de texto, sino que las notificaciones de aplicación móvil dejarán de funcionar para todos los usuarios hasta que se reactive la aplicación móvil.

Encuentre más información sobre los proveedores de MFA en [Introducción al Proveedor de Azure Multi-Factor Authentication](concept-mfa-authprovider.md).

### <a name="can-my-organization-switch-between-consumption-based-billing-and-subscriptions-a-license-based-model-at-any-time"></a>¿Mi organización puede cambiar entre la facturación basada en el consumo y las suscripciones (un modelo basado en licencias) en cualquier momento?

En algunos casos, sí.

Si el directorio tiene un proveedor de Azure Multi-Factor Authentication *por usuario*, puede agregar licencias de MFA. Los usuarios con licencia no se cuentan en la facturación basada en consumo por usuario. Los usuarios sin licencias podrán seguir habilitados para MFA a través del proveedor de MFA. Si compra y asigna licencias para todos los usuarios que están configurados para usar Multi-Factor Authentication, puede eliminar el proveedor de Azure Multi-Factor Authentication. Siempre puede crear otro proveedor de MFA por usuario si en el futuro tiene más usuarios que licencias.

Si el directorio tiene un proveedor de Azure Multi-Factor Authentication *por autenticación*, siempre se le facturará por cada autenticación mientras que el proveedor de MFA esté vinculado a la suscripción. Puede asignar licencias de MFA a los usuarios, pero se le seguirá facturando por cada solicitud de verificación en dos pasos, independientemente de si viene de alguien que tiene asignada una licencia de MFA o no.

### <a name="does-my-organization-have-to-use-and-synchronize-identities-to-use-azure-multi-factor-authentication"></a>¿Mi organización tiene que usar y sincronizar las identidades para usar Azure Multi-Factor Authentication?

Si la organización usa un modelo de facturación basado en el consumo, Azure Active Directory es opcional, no obligatorio. Si el proveedor de MFA no está vinculado a un inquilino de Azure AD, solo puede implementar Servidor Microsoft Azure Multi-Factor Authentication de manera local.

El modelo de licencia requiere Azure Active Directory porque las licencias se agregan al inquilino de Azure AD cuando las compra y asigna a los usuarios en el directorio.

## <a name="manage-and-support-user-accounts"></a>Administración y soporte técnico de las cuentas de usuario

* [¿Qué debo decir a los usuarios que hagan si no reciben respuesta en el teléfono?](#what-should-i-tell-my-users-to-do-if-they-dont-receive-a-response-on-their-phone)
* [¿Qué debo hacer si uno de los usuarios no puede ingresar a su cuenta?](#what-should-i-do-if-one-of-my-users-cant-get-in-to-their-account)
* [¿Qué debo hacer si a uno de los usuarios se le pierde el teléfono en el que usa las contraseñas de aplicación?](#what-should-i-do-if-one-of-my-users-loses-a-phone-that-is-using-app-passwords)
* [¿Qué puede hacer un usuario si no puede iniciar sesión en aplicaciones sin explorador?](#what-if-a-user-cant-sign-in-to-non-browser-apps)
* [Mis usuarios dicen que hay ocasiones en que no reciben el mensaje de texto, pero se agota el tiempo de espera de la comprobación.](#my-users-say-that-sometimes-they-dont-receive-the-text-message-or-the-verification-times-out)
* [¿Puedo cambiar la cantidad de tiempo que los usuarios tienen para escribir el código de verificación de un mensaje de texto antes de que se agote el tiempo de espera del sistema?](#can-i-change-the-amount-of-time-my-users-have-to-enter-the-verification-code-from-a-text-message-before-the-system-times-out)
* [¿Puedo usar tokens de hardware con Servidor Microsoft Azure Multi-Factor Authentication?](#can-i-use-hardware-tokens-with-azure-multi-factor-authentication-server)
* [¿Puedo usar Servidor Microsoft Azure Multi-Factor Authentication para proteger Terminal Services?](#can-i-use-azure-multi-factor-authentication-server-to-secure-terminal-services)
* [Configuré el identificador de llamada en Servidor MFA, pero mis usuarios siguen recibiendo llamadas de Multi-Factor Authentication provenientes de un autor de llamada anónimo.](#i-configured-caller-id-in-mfa-server-but-my-users-still-receive-multi-factor-authentication-calls-from-an-anonymous-caller)
* [¿Por qué se les pide a mis usuarios que registren su información de seguridad?](#why-are-my-users-being-prompted-to-register-their-security-information)

### <a name="what-should-i-tell-my-users-to-do-if-they-dont-receive-a-response-on-their-phone"></a>¿Qué debo decir a los usuarios que hagan si no reciben respuesta en el teléfono?

Pida a los usuarios que intenten hasta 5 veces en 5 minutos obtener una llamada de teléfono o un SMS para la autenticación. Microsoft usa varios proveedores para realizar las llamadas y el envío de mensajes SMS. Si este enfoque no funciona, abra una incidencia de soporte técnico para solucionar el problema.

Las aplicaciones de seguridad de terceros también pueden bloquear el mensaje de texto del código de verificación o la llamada telefónica. Si usa una aplicación de seguridad de terceros, pruebe a deshabilitar la protección y, a continuación, solicitar que se envíe otro código de verificación de MFA.

Si los pasos anteriores no funcionan, compruebe si los usuarios están configurados para más de un método de comprobación. Intente volver a iniciar sesión, pero seleccione un método de comprobación distinto en la página de inicio de sesión.

Para obtener más información, consulte la [guía de solución de problemas del usuario final](../user-help/multi-factor-authentication-end-user-troubleshoot.md).

### <a name="what-should-i-do-if-one-of-my-users-cant-get-in-to-their-account"></a>¿Qué debo hacer si uno de los usuarios no puede acceder a su cuenta?

Puede restablecer la cuenta del usuario; para ello, pídale que vuelva a realizar el proceso de registro. Puede obtener más información en [Administración de la configuración de usuario con Azure Multi-Factor Authentication en la nube](howto-mfa-userdevicesettings.md).

### <a name="what-should-i-do-if-one-of-my-users-loses-a-phone-that-is-using-app-passwords"></a>¿Qué debo hacer si a uno de los usuarios se le pierde el teléfono en el que usa las contraseñas de aplicación?

Para evitar el acceso no autorizado, elimine todas las contraseñas de aplicación del usuario. Una vez que el usuario tenga un dispositivo de reemplazo, pueden volver a crearlas. Puede obtener más información en [Administración de la configuración de usuario con Azure Multi-Factor Authentication en la nube](howto-mfa-userdevicesettings.md).

### <a name="what-if-a-user-cant-sign-in-to-non-browser-apps"></a>¿Qué puede hacer un usuario si no puede iniciar sesión en aplicaciones sin explorador?

Si la organización todavía usa clientes heredados y se [permite el uso de contraseñas de aplicación](howto-mfa-app-passwords.md), los usuarios no podrán iniciar sesión en estos clientes heredados con su nombre de usuario y contraseña. En lugar de eso, deberán [configurar contraseñas de aplicación](../user-help/multi-factor-authentication-end-user-app-passwords.md). Los usuarios deben borrar (eliminar) su información de inicio de sesión, reiniciar la aplicación y, luego, iniciar sesión con su nombre de usuario y la *contraseña de aplicación* en lugar de la contraseña habitual.

Si la organización no tiene clientes heredados, no debe permitir que los usuarios creen contraseñas de aplicación.

> [!NOTE]
> **Autenticación moderna para clientes de Office 2013**
>
> Las contraseñas de aplicación solo son necesarias para las aplicaciones que no admiten la autenticación moderna. Los clientes de Office 2013 admiten protocolos de autenticación moderna, pero se deben configurar. La autenticación moderna está disponible para cualquier cliente con la actualización de marzo de 2015 o posterior de Office 2013. Para obtener más información, vea la entrada de blog [Autenticación moderna actualizada de Office 365](https://www.microsoft.com/microsoft-365/blog/2015/11/19/updated-office-365-modern-authentication-public-preview/).

### <a name="my-users-say-that-sometimes-they-dont-receive-the-text-message-or-the-verification-times-out"></a>Mis usuarios dicen que hay ocasiones en que no reciben el mensaje de texto, pero se agota el tiempo de espera de la comprobación.

La entrega de mensajes SMS no está garantizada, pues hay factores que no se pueden controlar y que podrían afectar a la confiabilidad del servicio. Estos factores incluyen el país o la región de destino, el operador del teléfono móvil y la intensidad de la señal.

Las aplicaciones de seguridad de terceros también pueden bloquear el mensaje de texto del código de verificación o la llamada telefónica. Si usa una aplicación de seguridad de terceros, pruebe a deshabilitar la protección y, a continuación, solicitar que se envíe otro código de verificación de MFA.

Si sus usuarios tienen problemas con frecuencia para recibir mensajes de texto de manera confiable, indíqueles que usen en su lugar la aplicación Microsoft Authenticator o el método de llamada de teléfono. Microsoft Authenticator puede recibir notificaciones con conexiones de telefonía móvil y Wi-Fi. Además, la aplicación móvil puede generar códigos de comprobación aunque el dispositivo no tenga señal. La aplicación Microsoft Authenticator está disponible para [Android](https://go.microsoft.com/fwlink/?Linkid=825072), [iOS](https://go.microsoft.com/fwlink/?Linkid=825073) y [Windows Phone](https://www.microsoft.com/p/microsoft-authenticator/9nblgggzmcj6).

### <a name="can-i-change-the-amount-of-time-my-users-have-to-enter-the-verification-code-from-a-text-message-before-the-system-times-out"></a>¿Puedo cambiar la cantidad de tiempo que los usuarios tienen para escribir el código de verificación de un mensaje de texto antes de que se agote el tiempo de espera del sistema?

En algunos casos, sí es posible.

Para SMS unidireccionales con el servidor Azure MFA v7.0 o superior, puede configurar el ajuste de tiempo de espera estableciendo una clave del registro. Una vez que el servicio de nube MFA envía el mensaje de texto, se devuelve el código de verificación (o código de acceso de un solo uso) al servidor MFA. El servidor MFA almacena el código en la memoria durante 300 segundos de forma predeterminada. Si el usuario no escribe el código antes de que transcurran los 300 segundos, se denegará la autenticación. Siga estos pasos para cambiar la configuración de tiempo de espera predeterminada:

1. Ir a `HKLM\Software\Wow6432Node\Positive Networks\PhoneFactor`.
2. Cree una clave del Registro **DWORD** llamada *pfsvc_pendingSmsTimeoutSeconds* y establezca el tiempo (en segundos) durante el cual desea que el Servidor Azure MFA almacene los códigos de acceso de un solo uso.

>[!TIP]
>
> Si tiene varios servidores MFA, solo el que procesó la solicitud de autenticación original conoce el código de verificación que se envió al usuario. Cuando el usuario escribe el código, la solicitud de autenticación para validarlo tiene que enviarse al mismo servidor. Si la validación del código se envía a un servidor diferente, se deniega la autenticación.

Si los usuarios no responden al SMS dentro del período de tiempo de espera definido, se deniega la autenticación.

Para SMS unidireccionales con Azure MFA en la nube (incluido el adaptador de AD FS o la extensión del Servidor de directivas de redes), no se puede configurar el ajuste de tiempo de espera. Azure AD almacena el código de verificación durante 180 segundos.

### <a name="can-i-use-hardware-tokens-with-azure-multi-factor-authentication-server"></a>¿Puedo usar tokens de hardware con Servidor Microsoft Azure Multi-Factor Authentication?

Si usa Servidor Microsoft Azure Multi-Factor Authentication, puede importar los tokens de contraseña de un solo uso de duración definida (TOTP) y los de autenticación abierta (OATH) de terceros y, después, utilizarlos para realizar la comprobación en dos pasos.

Puede usar tokens de ActiveIdentity del tipo OATH TOTP si coloca la clave secreta en un archivo CSV y lo importa a Servidor Azure Multi-Factor Authentication. Puede usar tokens de OATH con Active Directory Federation Services (ADFS), autenticación basada en formularios de Internet Information Services (IIS) y Servicio de autenticación remota telefónica de usuario (RADIUS) siempre que el sistema cliente pueda aceptar las entradas de usuario.

Puede importar tokens de OATH TOTP de terceros con los siguientes formatos:  

- Contenedor de claves simétricas portátil (PSKC)
- CSV si el archivo contiene un número de serie, una clave secreta en formato Base 32 y un intervalo de tiempo

### <a name="can-i-use-azure-multi-factor-authentication-server-to-secure-terminal-services"></a>¿Puedo usar Servidor Microsoft Azure Multi-Factor Authentication para proteger Terminal Services?

Sí, pero si usa Windows Server 2012 R2 o versiones posteriores, solo puede proteger Terminal Services con Puerta de enlace de Escritorio remoto (Puerta de enlace de RD).

Gracias a los cambios de seguridad realizados en Windows Server 2012 R2, se ha modificado la forma en que Servidor Microsoft Azure Multi-Factor Authentication se conecta al paquete de seguridad de autoridad de seguridad local (LSA) en Windows Server 2012 y versiones anteriores. Para las versiones de Terminal Services en Windows 2012 o versiones anteriores, puede [proteger una aplicación con la autenticación de Windows](howto-mfaserver-windows.md#to-secure-an-application-with-windows-authentication-use-the-following-procedure). Sin embargo, si va a utilizar Windows Server 2012 R2, necesita el servicio Puerta de enlace de Escritorio remoto.

### <a name="i-configured-caller-id-in-mfa-server-but-my-users-still-receive-multi-factor-authentication-calls-from-an-anonymous-caller"></a>Configuré el identificador de llamada en Servidor MFA, pero mis usuarios siguen recibiendo llamadas de Multi-Factor Authentication provenientes de un autor de llamada anónimo.

A veces, cuando las llamadas de Multi-Factor Authentication se realizan a través de la red telefónica pública, se enrutan a través de un operador que no admite la característica de identificación de llamadas. Debido a este comportamiento del operador, los identificadores de llamada no están garantizados, aunque el sistema de Multi-Factor Authentication los envíe siempre.

### <a name="why-are-my-users-being-prompted-to-register-their-security-information"></a>¿Por qué se les pide a mis usuarios que registren su información de seguridad?

Hay varios motivos por los cuales se les pueden pedir a los usuarios que registren su información de seguridad:

- El administrador habilitó al usuario para MFA en Azure AD, pero todavía no tiene registrada la información de seguridad de su cuenta.
- Se habilitó al usuario para el restablecimiento de contraseña de autoservicio en Azure AD. La información de seguridad le ayudará a restablecer su contraseña en el futuro si llegara a olvidarla.
- El usuario tuvo acceso a una aplicación con una directiva de acceso condicional para requerir MFA y no se registró anteriormente para MFA.
- El usuario registra un dispositivo con Azure AD (incluido Azure AD Join) y la organización requiere MFA para el registro de dispositivos, pero el usuario no se registró anteriormente para MFA.
- El usuario genera Windows Hello para empresas en Windows 10 (que requiere MFA) y no se registró anteriormente para MFA.
- La organización creó y habilitó una directiva de registro de MFA que se aplicó al usuario.
- El usuario se registró anteriormente para MFA, pero eligió un método de comprobación que un administrador deshabilitó desde entonces. Por lo tanto, el usuario debe volver a registrarse para MFA para poder seleccionar un método de comprobación predeterminado nuevo.

## <a name="errors"></a>Errors

* [¿Qué deben hacer los usuarios si ven un mensaje de error que indica que la solicitud de autenticación no es para una cuenta activada al usar notificaciones de aplicación móvil?](#what-should-users-do-if-they-see-an-authentication-request-is-not-for-an-activated-account-error-message-when-using-mobile-app-notifications)
* [¿Qué deben hacer los usuarios si ven un mensaje de error 0x800434D4L al iniciar sesión en una aplicación que no es de explorador?](#what-should-users-do-if-they-see-a-0x800434d4l-error-message-when-signing-in-to-a-non-browser-application)

### <a name="what-should-users-do-if-they-see-an-authentication-request-is-not-for-an-activated-account-error-message-when-using-mobile-app-notifications"></a>¿Qué deben hacer los usuarios si ven un mensaje de error que indica que la solicitud de autenticación no es para una cuenta activada al usar notificaciones de aplicación móvil?

Pida al usuario que complete el siguiente procedimiento para quitar su cuenta de Microsoft Authenticator y, a continuación, agréguela de nuevo:

1. Vaya a [su perfil de Azure Portal](https://account.activedirectory.windowsazure.com/profile/) e inicie sesión con la cuenta de la organización.
2. Seleccione **Comprobación de seguridad adicional**.
3. Quite la cuenta existente de la aplicación Microsoft Authenticator.
4. Haga clic en **Configurar** y siga las instrucciones para volver a configurar Microsoft Authenticator.

### <a name="what-should-users-do-if-they-see-a-0x800434d4l-error-message-when-signing-in-to-a-non-browser-application"></a>¿Qué deben hacer los usuarios si ven un mensaje de error 0x800434D4L al iniciar sesión en una aplicación que no es de explorador?

El error *0x800434D4L* se genera cuando intenta iniciar sesión en una aplicación sin explorador, instalada en un equipo local, que no funciona con las cuentas que requieren la verificación en dos pasos.

Una forma de solucionar este error es tener una cuenta de usuario independiente para las operaciones relacionadas con la administración y otra para las no administrativas. Más adelante, puede vincular los buzones entre la cuenta de administrador y una sin derechos administrativos; de este modo, podrá iniciar sesión en Outlook con la cuenta sin derechos administrativos. Para más información, consulte [Dar a un administrador la capacidad de abrir y ver el contenido del buzón de correo de un usuario](https://help.outlook.com/141/gg709759.aspx?sl=1).

## <a name="next-steps"></a>Pasos siguientes

Si su pregunta no tiene aquí una respuesta, están disponibles las siguientes opciones de soporte técnico:

* Busque soluciones a problemas técnicos comunes en [Microsoft Support Knowledge Base](https://support.microsoft.com).
* Busque y examine cuestiones técnicas y sus respuestas en la comunidad, o bien realice su propia pregunta en las [preguntas y respuestas de Azure Active Directory](https://docs.microsoft.com/answers/topics/azure-active-directory.html).
* Póngase en contacto con un profesional de Microsoft a través del [soporte técnico de Servidor Azure Multi-Factor Authentication](https://support.microsoft.com/oas/default.aspx?prid=14947). Al ponerse en contacto con nosotros, nos resultará de gran utilidad que incluya tanta información sobre su problema como sea posible. Entre la información que puede aportar se incluyen la página donde vio el error, el código de error específico, el identificador de sesión específico y el identificador del usuario que vio el error.
* Si es un cliente antiguo de PhoneFactor y tiene alguna pregunta o necesita ayuda para restablecer una contraseña, use la [phonefactorsupport@microsoft.com](mailto:phonefactorsupport@microsoft.com) dirección de correo electrónico para abrir una incidencia de soporte técnico.
