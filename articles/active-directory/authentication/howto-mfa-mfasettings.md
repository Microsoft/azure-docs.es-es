---
title: Configurar Azure Multi-Factor Authentication - Azure Active Directory
description: Aprenda cómo configurar Azure Multi-Factor Authentication en Azure Portal
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 06/05/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.custom: contperfq4
ms.openlocfilehash: 8b695bad791388dc51123a118344b8fda0f54ca8
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87027706"
---
# <a name="configure-azure-multi-factor-authentication-settings"></a>Configuración de Azure Multi-Factor Authentication

Para personalizar la experiencia del usuario final de Azure Multi-Factor Authentication, puede definir opciones de configuración como los umbrales de bloqueo de la cuenta o las alertas y notificaciones de fraude. Algunas configuraciones están directamente en Azure Portal para Azure Active Directory (Azure AD) y otras en un portal de Azure Multi-Factor Authentication independiente.

Las siguientes configuraciones de Azure Multi-Factor Authentication están disponibles en Azure Portal:

| Característica | Descripción |
| ------- | ----------- |
| [Bloqueo de cuenta](#account-lockout) | Impida temporalmente que las cuentas usen Azure Multi-Factor Authentication si hay demasiados intentos de autenticación denegados en una fila. Esta característica solo se aplica a los usuarios que escriben un PIN para autenticarse. (Servidor MFA) |
| [Bloqueo y desbloqueo de usuarios](#block-and-unblock-users) | Impida que usuarios específicos puedan recibir solicitudes de Azure Multi-Factor Authentication. Todos los intentos de autenticación para los usuarios bloqueados se denegarán automáticamente. Los usuarios permanecen bloqueados durante noventa días a partir del momento en que se bloqueen o se desbloquean manualmente. |
| [Alerta de fraude](#fraud-alert) | Defina configuraciones que permitan a los usuarios notificar solicitudes de comprobación fraudulentas. |
| [Notificaciones](#notifications) | Permite notificaciones de eventos desde el Servidor MFA. |
| [Tokens OATH](concept-authentication-methods.md#oath-tokens) | Se usa en entornos de Azure MFA basados en la nube para administrar tokens de OATH para los usuarios. |
| [Configuración de las llamadas telefónicas](#phone-call-settings) | Configure valores relacionados con llamadas de teléfono y saludos para entornos locales y en la nube. |
| Proveedores | Se mostrarán los proveedores de autenticación existentes que pueden haberse asociado con su cuenta. A partir del 1 de septiembre de 2018 no se pueden crear nuevos proveedores de autenticación. |

![Azure Portal: Configuración de Multi-Factor Authentication de Azure AD](./media/howto-mfa-mfasettings/multi-factor-authentication-settings-portal.png)

## <a name="account-lockout"></a>Bloqueo de cuenta

Para evitar intentos repetidos de MFA como parte de un ataque, la configuración de bloqueo de cuenta le permite especificar el número de intentos erróneos que se permiten antes de que la cuenta quede bloqueada durante un período de tiempo. La configuración de bloqueo de la cuenta solo se aplica cuando se especifica un código PIN para el aviso de MFA.

Las siguientes configuraciones están disponibles:

* Número de denegaciones de MFA para desencadenar el bloqueo de cuenta
* Minutos hasta que se restablezca el contador de bloqueos de cuenta
* Minutos hasta que la cuenta se desbloquee automáticamente

Para definir las configuraciones de bloqueo de cuenta, complete las siguientes configuraciones:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador.
1. Vaya a **Azure Active Directory** > **Seguridad** > **MFA** > **Bloqueo de cuenta**.
1. Escriba los valores necesarios para el entorno y seleccione **Guardar**.

    ![Captura de pantalla de la configuración de bloqueo de la cuenta en Azure Portal](./media/howto-mfa-mfasettings/account-lockout-settings.png)

## <a name="block-and-unblock-users"></a>Bloqueo y desbloqueo de usuarios

Si el dispositivo de un usuario se ha perdido o ha sido robado, puede bloquear los intentos de autenticación para la cuenta asociada. Todos los intentos de autenticación para los usuarios bloqueados se denegarán automáticamente. Los usuarios permanecen bloqueados durante 90 días a partir del momento en que se bloqueen.

### <a name="block-a-user"></a>Bloquear a un usuario

Para bloquear a un usuario, complete los siguientes pasos:

1. Vaya a **Azure Active Directory** > **Seguridad** > **MFA** > **bloqueo/desbloqueo de usuarios**.
1. Seleccione **Agregar** para bloquear a un usuario.
1. Seleccione el **Grupo de replicación** y elija el *valor predeterminado de Azure*.

    Escriba el nombre de usuario del usuario bloqueado como `username\@domain.com` y proporcione un comentario en el campo *Motivo*.
1. Cuando esté listo, seleccione **Aceptar** para bloquear al usuario.

### <a name="unblock-a-user"></a>Desbloquear a un usuario

Para desbloquear a un usuario, complete los siguientes pasos:

1. Vaya a **Azure Active Directory** > **Seguridad** > **MFA** > **bloqueo/desbloqueo de usuarios**.
1. Seleccione la columna *Acción* situada junto al usuario que la interesa y seleccione **Desbloquear**.
1. Escriba un comentario en el campo *Motivo para desbloquear*.
1. Cuando esté listo, seleccione **Aceptar** para desbloquear al usuario.

## <a name="fraud-alert"></a>Alerta de fraude

La característica de alerta de fraude permite a los usuarios informar sobre intentos fraudulentos de acceso a sus recursos. Cuando se recibe un mensaje de MFA desconocido y sospechoso, los usuarios pueden informar del intento de fraude mediante la aplicación Microsoft Authenticator o a través de su teléfono.

Están disponibles las siguientes opciones de configuración de alertas de fraude:

* **Bloquear automáticamente a los usuarios que notifican fraudes**: si se informa de que un usuario ha cometido fraude, su cuenta se bloquea durante 90 días o hasta que un administrador la desbloquee. Un administrador puede revisar los inicios de sesión usando el informe de inicio de sesión y puede tomar las medidas adecuadas para prevenir el fraude en el futuro. El administrador puede, a continuación, [desbloquear](#unblock-a-user) la cuenta de usuario.
* **Código para notificar fraudes durante el saludo inicial**: cuando los usuarios reciben una llamada telefónica para realizar la autenticación multifactor, normalmente presionan **#** para confirmar el inicio de sesión. Si desea notificar un fraude, el usuario introduce un código antes de presionar **#** . De manera predeterminada, dicho código es **0**, pero se puede personalizar.

   > [!NOTE]
   > El saludo predeterminado de Microsoft solicita a los usuarios que presionen **0#** para enviar una alerta de fraude. Si quiere usar un código distinto a **0**, grabe y cargue su propio saludo personalizado con las instrucciones adecuadas para sus usuarios.

Para habilitar y configurar las alertas de fraude, complete los pasos siguientes:

1. Vaya a **Azure Active Directory** > **Seguridad** > **MFA** > **Alertas de fraude**.
1. Establezca la configuración *Permitir a los usuarios enviar alertas de fraude* en **Activado**.
1. Configure las opciones *Bloquear automáticamente a los usuarios que notifican fraudes* o *Código para notificar fraudes durante el saludo inicial* según sea necesario.
1. Cuando esté preparado, seleccione **Guardar**.

### <a name="view-fraud-reports"></a>Visualización de notificaciones de fraude

Seleccione **Azure Active Directory** > **Inicios de sesión** > **Detalles de autenticación**. El informe de fraude ahora forma parte del informe de inicios de sesión de Azure AD estándar y se mostrará en el **"Detalle del resultado"** con la denegación de MFA y el código de fraude especificado.
 
## <a name="notifications"></a>Notificaciones

Las notificaciones por correo electrónico se pueden configurar cuando los usuarios notifican alertas de fraude. Normalmente, estas notificaciones se envían a los administradores de identidad, ya que es probable que las credenciales de la cuenta del usuario estén en peligro. En el ejemplo siguiente se muestra el aspecto de un correo electrónico de notificación de alerta de fraude:

![Correo electrónico de notificación de alerta de fraude de ejemplo](./media/howto-mfa-mfasettings/multi-factor-authentication-fraud-alert-email.png)

Para configurar las notificaciones de alerta de fraude, complete las configuraciones siguientes:

1. Vaya a **Azure Active Directory** > **Seguridad** > **Autenticación multifactor** > **Notificaciones**.
1. Escriba la dirección de correo electrónico que desea agregar en el cuadro siguiente.
1. Para quitar una dirección de correo electrónico existente, seleccione la opción **...** situada junto a la dirección de correo electrónico deseada y luego seleccione **Eliminar**.
1. Cuando esté preparado, seleccione **Guardar**.

## <a name="oath-tokens"></a>Tokens OATH

Azure AD admite el uso de tokens TOTP SHA-1 de OATH, que actualizan los códigos cada 30 o 60 segundos. Los clientes pueden adquirir estos tokens a través del proveedor de su elección.

Los tokens de hardware TOTP de OATH suelen incluir una clave secreta, o valor de inicialización, programada previamente en el token. Estas claves se deben introducir en Azure AD según se describe en los pasos siguientes. Las claves secretas se limitan a 128 caracteres lo que puede no ser compatible con todos los tokens. La clave secreta solo puede contener los caracteres *a-z* o *A-Z* y los dígitos *1-7*, y debe estar codificada en *base 32*.

Los tokens de hardware TOTP de OATH programables que se pueden reinicializar también se pueden configurar con Azure AD en el flujo de configuración de los tokens de software.

Los tokens de hardware OATH se admiten como parte de una versión preliminar pública. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)

![Carga de tokens OATH en la hoja de tokens OATH de MFA](media/concept-authentication-methods/mfa-server-oath-tokens-azure-ad.png)

Una vez que se adquieren los tokens, se deben cargar en un formato de archivo de valores separados por comas (CSV), incluidos los valores de UPN, número de serie, clave secreta, intervalo de tiempo, fabricante y modelo, como se muestra en el ejemplo siguiente.

```csv
upn,serial number,secret key,time interval,manufacturer,model
Helga@contoso.com,1234567,1234567abcdef1234567abcdef,60,Contoso,HardwareKey
```

> [!NOTE]
> Asegúrese de incluir la fila de encabezado en el archivo CSV.

Una vez formateado correctamente como archivo CSV, un administrador puede iniciar sesión en Azure Portal e ir a **Azure Active Directory > Seguridad > MFA > Tokens OATH** y cargar el archivo CSV resultante.

En función del tamaño del archivo CSV, puede tardar unos minutos en procesarse. Seleccione el botón **Actualizar** para ver el estado actual. Si hay algún error en el archivo, puede descargar un archivo CSV que enumere los errores para poder resolverlos. Los nombres de campo del archivo CSV descargado son diferentes de los de la versión cargada.

Una vez solucionados los errores, para activar cada clave, el administrador puede seleccionar **Activar** para el token y escribir la OTP que se muestra en el token.

Los usuarios pueden tener una combinación de hasta cinco tokens de hardware OATH o aplicaciones de autenticación, como la aplicación Microsoft Authenticator, configurada para utilizarse en cualquier momento.

## <a name="phone-call-settings"></a>Configuración de la llamada telefónica

Si los usuarios reciben llamadas telefónicas de solicitudes de MFA, puede configurar su experiencia, como el identificador del autor de la llamada o el saludo de voz que escuchan.

En Estados Unidos, si no ha configurado el identificador del autor de la llamada de MFA, las llamadas de voz de Microsoft proceden de los números siguientes. Si usa filtros de correo no deseado, asegúrese de excluir estos números:

* *+1 (866) 539 4191*
* *+1 (855) 330 8653*
* *+1 (877) 668 6536*

> [!NOTE]
> A veces, cuando las llamadas de Azure Multi-Factor Authentication se realizan a través de la red telefónica pública, se enrutan a través de un operador que no admite la característica de identificación de llamadas. Por este motivo, los identificadores de llamada no están garantizados, aunque Azure Multi-Factor Authentication los envíe siempre.

Para configurar su propio número de identificador del autor de la llamada, complete los pasos siguientes:

1. Vaya a **Azure Active Directory** > **Seguridad** > **MFA** > **Configuración de llamada telefónica**.
1. Establezca **Número de id. del autor de llamada de MFA** en el número que desea que los usuarios en sus teléfonos. Solo se permiten números de Estados Unidos.
1. Cuando esté preparado, seleccione **Guardar**.

### <a name="custom-voice-messages"></a>Mensajes de voz personalizados

Puede usar sus propias grabaciones o saludos para Azure Multi-Factor Authentication con la característica de mensajes de voz personalizados. Se pueden utilizar estos mensajes además de las grabaciones de Microsoft predeterminadas; asimismo aquellos pueden reemplazar a estas.

Antes de comenzar, tenga en cuenta las restricciones siguientes:

* Los formatos de archivo compatibles son *.wav* y *.mp3*.
* El límite de tamaño de archivo es de 1 MB.
* Los mensajes de autenticación deben durar menos de 20 segundos. Los mensajes que duren más de 20 segundos pueden hacer que la verificación cause error. El usuario podría no responder antes de que finalice el mensaje y se agote el tiempo de espera de la verificación.

### <a name="custom-message-language-behavior"></a>Comportamiento de idioma de mensaje personalizado

Cuando se reproduce un mensaje de voz personalizado para el usuario, el idioma del mensaje depende de los siguientes factores:

* El idioma del usuario actual.
  * El idioma detectado en el explorador del usuario.
  * Otros escenarios de autenticación pueden comportarse de manera diferente.
* El idioma de otros mensajes personalizados disponibles.
  * Este idioma lo elige el administrador, cuando se agrega un mensaje personalizado.

Por ejemplo, si hay solo un mensaje personalizado en alemán:

* Un usuario que se autentique en alemán oirá el mensaje personalizado en ese idioma.
* Un usuario que se autentique en inglés oirá el mensaje personalizado en ese idioma.

### <a name="custom-voice-message-defaults"></a>Valores predeterminados de los mensajes de voz personalizados

Los siguientes scripts de ejemplo se pueden usar para crear sus propios mensajes personalizados. Estas frases son las predeterminadas si no configura sus propios mensajes personalizados:

| Nombre del mensaje | Script |
| --- | --- |
| Autenticación correcta | Su inicio de sesión se comprobó correctamente. Adiós. |
| Solicitud de extensión | Gracias por usar el sistema de comprobación de inicio de sesión de Microsoft. Presione la tecla almohadilla para continuar. |
| Confirmación de fraude | Se ha enviado una alerta de fraude. Para desbloquear su cuenta, póngase en contacto con el departamento de soporte técnico de TI de su empresa. |
| Saludo de fraude (Estándar) | Gracias por usar el sistema de comprobación de inicio de sesión de Microsoft. Presione la tecla almohadilla para finalizar la comprobación. Si no inició esta comprobación, es posible que otra persona esté intentando acceder a su cuenta. Presione cero para enviar una alerta de fraude. Se enviará una notificación al equipo de TI de su empresa y se bloqueará cualquier otro intento de comprobación. |
| Fraude notificado: se ha enviado una alerta de fraude. | Para desbloquear su cuenta, póngase en contacto con el departamento de soporte técnico de TI de su empresa. |
| Activación | Gracias por usar el sistema de comprobación de inicio de sesión de Microsoft. Presione la tecla almohadilla para finalizar la comprobación. |
| Reintento tras denegación de autenticación | Comprobación denegada. |
| Reintento (Estándar) | Gracias por usar el sistema de comprobación de inicio de sesión de Microsoft. Presione la tecla almohadilla para finalizar la comprobación. |
| Saludo (Estándar) | Gracias por usar el sistema de comprobación de inicio de sesión de Microsoft. Presione la tecla almohadilla para finalizar la comprobación. |
| Saludo (PIN) | Gracias por usar el sistema de comprobación de inicio de sesión de Microsoft. Escriba el PIN seguido de la tecla almohadilla para finalizar la comprobación. |
| Saludo de fraude (PIN) | Gracias por usar el sistema de comprobación de inicio de sesión de Microsoft.  Escriba el PIN seguido de la tecla almohadilla para finalizar la comprobación. Si no inició esta comprobación, es posible que otra persona esté intentando acceder a su cuenta. Presione cero para enviar una alerta de fraude. Se enviará una notificación al equipo de TI de su empresa y se bloqueará cualquier otro intento de comprobación. |
| Reintento (PIN) | Gracias por usar el sistema de comprobación de inicio de sesión de Microsoft. Escriba el PIN seguido de la tecla almohadilla para finalizar la comprobación. |
| Solicitud de extensión tras dígitos | Si ya se encuentra conectado a esta extensión, presione la tecla almohadilla para continuar. |
| Autenticación denegada | Lo sentimos, no puede iniciar sesión en este momento. Inténtelo de nuevo más tarde. |
| Saludo de activación (Estándar) | Gracias por usar el sistema de comprobación de inicio de sesión de Microsoft. Presione la tecla almohadilla para finalizar la comprobación. |
| Reintento de activación (Estándar) | Gracias por usar el sistema de comprobación de inicio de sesión de Microsoft. Presione la tecla almohadilla para finalizar la comprobación. |
| Saludo de activación (PIN) | Gracias por usar el sistema de comprobación de inicio de sesión de Microsoft. Escriba el PIN seguido de la tecla almohadilla para finalizar la comprobación. |
| Solicitud de extensión antes de dígitos | Gracias por usar el sistema de comprobación de inicio de sesión de Microsoft. Transfiera esta llamada a la extensión… |

### <a name="set-up-a-custom-message"></a>Configuración de un mensaje personalizado

Para usar sus propios mensajes personalizados, complete los pasos siguientes:

1. Vaya a **Azure Active Directory** > **Seguridad** > **MFA** > **Configuración de llamada telefónica**.
1. Seleccione **Agregar saludo**.
1. Elija un tipo de saludo en **Tipo**, como *Saludo (Estándar)* o *Autenticación correcta*.
1. En **Idioma** seleccione un idioma basándose en la sección anterior [Comportamiento de idioma de mensaje personalizado](#custom-message-language-behavior).
1. Busque y seleccione un archivo de sonido *.mp3* o *.wav* para cargarlo.
1. Cuando esté listo, seleccione **Agregar** y después **Guardar**.

## <a name="mfa-service-settings"></a>Configuración del servicio MFA

La configuración para contraseñas de aplicación, IP de confianza, opciones de comprobación y recordar Multi-factor Authentication de Azure Multi-Factor Authentication se pueden encontrar en la configuración del servicio. Este es más de un portal heredado y no forma parte del portal convencional de Azure AD.

Se puede tener acceso a la configuración del servicio desde el Azure Portal, para ello, vaya a **Azure Active Directory** > **Seguridad** > **MFA** > **Introducción** > **Configurar** > **Configuración de MFA basada en la nube**. Se abre una nueva ventana o pestaña con opciones adicionales de *configuración del servicio*.

## <a name="trusted-ips"></a>IP de confianza

La característica _IP de confianza_ de Azure Multi-Factor Authentication omite las solicitudes de autenticación multifactor para los usuarios que inician sesión desde un intervalo de direcciones IP definido. Puede establecer intervalos de direcciones IP de confianza para los entornos locales, de manera que, cuando los usuarios se encuentran en una de estas ubicaciones, Azure Multi-Factor Authentication no emite ninguna solicitud.

> [!NOTE]
> Las direcciones IP de confianza pueden incluir intervalos de direcciones IP privadas solo cuando se usa un servidor MFA. En el caso de Azure Multi-Factor Authentication basado en la nube, solo puede usar intervalos de direcciones IP públicas.
>
> Los intervalos de IPv6 solo se admiten en la interfaz de [Ubicaciones con nombre (versión preliminar)](../conditional-access/location-condition.md#preview-features).

Si su organización implementa la extensión NPS para proporcionar Multi-Factor Authentication en aplicaciones locales, tenga en cuenta que la dirección IP de origen siempre parecerá ser el servidor NPS a través del que fluye el intento de autenticación.

| Tipo de inquilino de Azure AD | Opciones de características de direcciones IP de confianza |
|:--- |:--- |
| Administrado |**Intervalos específicos de direcciones IP**: los administradores pueden especificar un intervalo de direcciones IP que puede omitir la verificación en dos pasos para los usuarios que inician sesión desde la intranet de la empresa. Se puede configurar un máximo de cincuenta intervalos de direcciones IP de confianza.|
| Federado |**Todos los usuarios federados**: todos los usuarios federados que inician sesión desde dentro de la organización pueden omitir la verificación en dos pasos. Los usuarios omiten la verificación mediante el uso de una notificación que emiten los servicios de federación de Active Directory (AD FS).<br/>**Intervalos específicos de direcciones IP**: los administradores pueden especificar un intervalo de direcciones IP que puede omitir la verificación en dos pasos para los usuarios que inician sesión desde la intranet de la empresa. |

La omisión de las direcciones IP de confianza solo funciona desde dentro de la intranet de la empresa. Si selecciona la opción **Todos los usuarios federados** y un usuario inicia sesión desde fuera de la intranet de la empresa, el usuario tendrá que autenticarse mediante la verificación en dos pasos. El proceso es el mismo, incluso si el usuario presenta una notificación de AD FS.

### <a name="end-user-experience-inside-of-corpnet"></a>Experiencia del usuario final dentro de la red corporativa

Cuando se deshabilita la característica de direcciones IP de confianza, la autenticación multifactor es necesaria para los flujos del explorador. Se necesitan contraseñas de aplicación para anteriores aplicaciones de cliente enriquecidas.

Cuando se usan direcciones IP de confianza, no se requiere la autenticación multifactor para los flujos del explorador. Las contraseñas de aplicación no son obligatorias para las anteriores aplicaciones de cliente enriquecidas, siempre que el usuario no haya creado una contraseña de aplicación. Después de que la contraseña de aplicación esté en uso, la contraseña sigue siendo necesaria.

### <a name="end-user-experience-outside-corpnet"></a>Experiencia del usuario final fuera de la red corporativa

Independientemente de si se han definido direcciones IP de confianza, la autenticación multifactor es necesaria para los flujos del explorador. Se necesitan contraseñas de aplicación para anteriores aplicaciones de cliente enriquecidas.

### <a name="enable-named-locations-by-using-conditional-access"></a>Habilitación de las ubicaciones con nombre mediante el acceso condicional

Puede usar las reglas de acceso condicional para definir ubicaciones con nombre mediante los pasos siguientes:

1. En Azure Portal, busque y seleccione **Azure Active Directory** y luego vaya a **Seguridad** > **Acceso condicional** > **Ubicaciones con nombre**.
1. Seleccione **Nueva ubicación**.
1. Escriba un nombre para la ubicación.
1. Seleccione **Marcar como ubicación de confianza**.
1. Escriba el intervalo de direcciones IP en la notación CIDR para su entorno, como *40.77.182.32/27*.
1. Seleccione **Crear**.

### <a name="enable-the-trusted-ips-feature-by-using-conditional-access"></a>Habilite la característica de direcciones IP de confianza mediante el acceso condicional

Para habilitar direcciones IP de confianza mediante directivas de acceso condicional, realice los pasos siguientes:

1. En Azure Portal, busque y seleccione **Azure Active Directory** y luego vaya a **Seguridad** >  **Acceso condicional** > **Ubicaciones con nombre**.
1. Seleccione **Configurar IP de confianza de MFA**.
1. En la página **Configuración del servicio**, en **IP de confianza**, seleccione una de las dos opciones siguientes:

   * **Para solicitudes de usuarios federados cuyo origen esté en mi intranet**: Para elegir esta opción, seleccione la casilla. Todos los usuarios federados que inicien sesión desde la red corporativa omitirán la verificación en dos pasos mediante una notificación emitida por AD FS. Asegúrese de que AD FS tiene una regla para agregar la notificación de intranet al tráfico adecuado. Si la regla no existe, cree la siguiente regla en AD FS:

      `c:[Type== "http://schemas.microsoft.com/ws/2012/01/insidecorporatenetwork"] => issue(claim = c);`

   * **Para solicitudes de un intervalo de IP públicas específico**: para elegir esta opción, escriba las direcciones IP en el cuadro de texto mediante la notación CIDR.
      * Para las direcciones IP que se encuentran en el intervalo xxx.xxx.xxx.1 mediante xxx.xxx.xxx.254, use una notación como **xxx.xxx.xxx.0/24**.
      * Para una única dirección IP, use esta notación: **xxx.xxx.xxx.xxx/32**.
      * Especifique un máximo de 50 intervalos de direcciones IP. Los usuarios que inician sesión desde estas direcciones IP omiten la comprobación en dos pasos.

1. Seleccione **Guardar**.

### <a name="enable-the-trusted-ips-feature-by-using-service-settings"></a>Habilite la característica de direcciones IP de confianza mediante la configuración del servicio

Si no desea usar las directivas de acceso condicional para habilitar las direcciones IP de confianza, puede definir la *configuración del servicio* para Azure Multi-Factor Authentication mediante los pasos siguientes:

1. En Azure Portal, busque y seleccione **Azure Active Directory** y luego elija **Usuarios**.
1. Seleccione **Multi-Factor Authentication**.
1. En Multi-Factor Authentication, seleccione **Configuración del servicio**.
1. En la página **Configuración del servicio**, en **IP de confianza**, seleccione una de las opciones siguientes (o ambas):

   * **Para solicitudes de usuarios federados en mi intranet**: Para elegir esta opción, seleccione la casilla. Todos los usuarios federados que inicien sesión desde la red corporativa omitirán la verificación en dos pasos mediante una notificación emitida por AD FS. Asegúrese de que AD FS tiene una regla para agregar la notificación de intranet al tráfico adecuado. Si la regla no existe, cree la siguiente regla en AD FS:

      `c:[Type== "http://schemas.microsoft.com/ws/2012/01/insidecorporatenetwork"] => issue(claim = c);`

   * **Para solicitudes de un intervalo específico de subredes de direcciones IP**: para elegir esta opción, escriba las direcciones IP en el cuadro de texto mediante la notación CIDR.
      * Para las direcciones IP que se encuentran en el intervalo xxx.xxx.xxx.1 mediante xxx.xxx.xxx.254, use una notación como **xxx.xxx.xxx.0/24**.
      * Para una única dirección IP, use esta notación: **xxx.xxx.xxx.xxx/32**.
      * Especifique un máximo de 50 intervalos de direcciones IP. Los usuarios que inician sesión desde estas direcciones IP omiten la comprobación en dos pasos.

1. Seleccione **Guardar**.

## <a name="verification-methods"></a>Métodos de comprobación

Puede elegir los métodos de verificación que estén disponibles para los usuarios en el portal de configuración del servicio. Cuando los usuarios inscriben sus cuentas en Azure Multi-Factor Authentication, deciden su método de verificación preferido de las opciones que ha habilitado. Las instrucciones para el proceso de inscripción de los usuarios se proporcionan en [Configuración de mi cuenta para la verificación en dos pasos](../user-help/multi-factor-authentication-end-user-first-time.md).

Están disponibles los siguientes métodos de verificación:

| Método | Descripción |
|:--- |:--- |
| Llamada al teléfono |Hace una llamada de voz automática. El usuario responde a la llamada y pulsa la # del teclado del teléfono para autenticarse. El número de teléfono no se sincroniza con Active Directory local. |
| Mensaje de texto al teléfono |Envía un mensaje de texto que contiene un código de verificación. Se pide al usuario que escriba el código de verificación en la interfaz de inicio de sesión. Este proceso se llama "SMS unidireccional". SMS bidireccional significa que el usuario debe devolver un mensaje de texto con un código determinado. SMS bidireccional se encuentra en desuso y no se admitirá después del 14 de noviembre de 2018. Los administradores deben habilitar otro método para los usuarios que antes usaban SMS bidireccional.|
| Notificación a través de aplicación móvil |Envía una notificación push a su teléfono o dispositivo registrado. El usuario ve la notificación y selecciona **Comprobar** para completar la comprobación. La aplicación Microsoft Authenticator está disponible para [Windows Phone](https://www.microsoft.com/p/microsoft-authenticator/9nblgggzmcj6), [Android](https://go.microsoft.com/fwlink/?Linkid=825072) e [IOS](https://go.microsoft.com/fwlink/?Linkid=825073). |
| Código de verificación de aplicación móvil o token de hardware |La aplicación Microsoft Authenticator genera un nuevo código de verificación de OATH cada 30 segundos. El usuario escribe el código de verificación en la interfaz de inicio de sesión. La aplicación Microsoft Authenticator está disponible para [Windows Phone](https://www.microsoft.com/p/microsoft-authenticator/9nblgggzmcj6), [Android](https://go.microsoft.com/fwlink/?Linkid=825072) e [IOS](https://go.microsoft.com/fwlink/?Linkid=825073). |

Para más información, consulte [¿Qué métodos de autenticación y verificación hay disponibles en Azure Active Directory?](concept-authentication-methods.md)

### <a name="enable-and-disable-verification-methods"></a>Habilitar y deshabilitar los métodos de verificación

Para habilitar o deshabilitar los métodos de verificación, complete los pasos siguientes:

1. En Azure Portal, busque y seleccione **Azure Active Directory** y luego elija **Usuarios**.
1. Seleccione **Multi-Factor Authentication**.
1. En Multi-Factor Authentication, seleccione **Configuración del servicio**.
1. En la página **Configuración del servicio**, en **Opciones de verificación**, active o desactive los métodos para proporcionar a los usuarios.
1. Haga clic en **Save**(Guardar).

## <a name="remember-multi-factor-authentication"></a>Recordar Multi-Factor Authentication

La característica _Recordar Multi-Factor Authentication_ permite a los usuarios omitir las verificaciones posteriores durante un número especificado de días, una vez hayan iniciado sesión correctamente en un dispositivo mediante el uso de Multi-Factor Authentication. Esta característica permite mejorar la facilidad de uso, ya que minimiza el número de veces que un usuario puede realizar MFA en el mismo dispositivo.

> [!IMPORTANT]
> Si una cuenta o un dispositivo corren peligro, el hecho de recordar Multi-Factor Authentication para los dispositivos de confianza puede afectar a la seguridad. Si una cuenta corporativa se pone en peligro o un dispositivo de confianza es objeto de pérdida o robo, debe [revocar las sesiones de MFA](howto-mfa-userdevicesettings.md).
>
> La acción de restauración revoca el estado de confianza de todos los dispositivos y el usuario debe volver a realizar la verificación en dos pasos. También puede pedir a los usuarios que restauren Multi-Factor Authentication en sus propios dispositivos tal y como se indica en [Administración de la configuración de la verificación en dos pasos](../user-help/multi-factor-authentication-end-user-manage-settings.md#turn-on-two-factor-verification-prompts-on-a-trusted-device).

### <a name="how-the-feature-works"></a>Cómo funciona la característica

La característica Recordar Multi-Factor Authentication establece una cookie persistente en el explorador cuando un usuario selecciona la opción **No preguntar de nuevo durante X días** en el momento del inicio de sesión. Al usuario no se le volverá a solicitar Multi-Factor Authentication desde ese mismo explorador hasta que expire la cookie. Si el usuario abre un explorador diferente en el mismo dispositivo o borra sus cookies, se le pedirá de nuevo la verificación.

La opción **No preguntar de nuevo durante X días** no está disponible en las aplicaciones sin explorador, independientemente de si la aplicación admite la autenticación moderna. Estas aplicaciones usan _tokens de actualización_ que proporcionan nuevos tokens de acceso cada hora. Cuando se valida un token de actualización, Azure AD comprueba que la última verificación en dos pasos estaba dentro del número de días especificado.

La característica reduce el número de autenticaciones en las aplicaciones web, que normalmente se solicitan siempre. La característica aumenta el número de autenticaciones para los clientes de autenticación moderna, que normalmente se solicita cada 90 días. También se puede aumentar el número de autenticaciones cuando se combina con las directivas de acceso condicional.

> [!IMPORTANT]
> La característica **Recordar Multi-Factor Authentication** no es compatible con la característica **Mantener la sesión iniciada** de AD FS cuando los usuarios realizan la verificación en dos pasos para AD FS mediante el Servidor Azure Multi-Factor Authentication o una solución de terceros para la autenticación multifactor.
>
> Si los usuarios seleccionan **Mantener la sesión iniciada** en AD FS y también marcan su dispositivo como de confianza para Multi-Factor Authentication, no se comprobará el usuario automáticamente después de que expire el número de días de la característica **Recordar Multi-Factor Authentication**. Azure AD solicita una nueva verificación en dos pasos, pero AD FS devuelve un token con la fecha y la notificación originales de Multi-Factor Authentication en lugar de volver a realizar la verificación en dos pasos. **Esta reacción crea un bucle de comprobación entre Azure AD y AD FS.**
>
> La característica **Recordar Multi-Factor Authentication** no es compatible con los usuarios de B2B y no será visible para ellos al iniciar sesión en los inquilinos invitados.
>

### <a name="enable-remember-multi-factor-authentication"></a>Habilitación de Recordar Multi-Factor Authentication

Para habilitar y configurar la opción para que los usuarios recuerden su estado de MFA y omitan las solicitudes, complete los pasos siguientes:

1. En Azure Portal, busque y seleccione **Azure Active Directory** y luego elija **Usuarios**.
1. Seleccione **Multi-Factor Authentication**.
1. En Multi-Factor Authentication, seleccione **Configuración del servicio**.
1. En la página **Configuración del servicio**, **administre Recordar Multi-Factor Authentication** y seleccione la opción **Permitir que los usuarios recuerden la autenticación multifactor en dispositivos de confianza**.
1. Establezca el número de días para permitir que los dispositivos de confianza omitan la verificación en dos pasos. El valor predeterminado es 14 días.
1. Seleccione **Guardar**.

### <a name="mark-a-device-as-trusted"></a>Marca de un dispositivo como de confianza

Después de habilitar la característica Recordar Multi-Factor Authentication, al iniciar sesión los usuarios pueden marcar un dispositivo como de confianza si seleccionan la opción **No volver a preguntar**.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre los métodos disponibles para su uso en Azure Multi-Factor Authentication, consulte [¿Qué métodos de autenticación y verificación hay disponibles en Azure Active Directory?](concept-authentication-methods.md)
