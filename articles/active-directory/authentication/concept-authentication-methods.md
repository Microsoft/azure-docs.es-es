---
title: 'Métodos y características de autenticación: Azure Active Directory'
description: Obtenga información sobre los diferentes métodos y características de autenticación disponibles en Azure Active Directory para ayudar a mejorar y proteger los eventos de inicio de sesión.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.collection: M365-identity-device-management
ms.custom: contperfq4
ms.openlocfilehash: 40ea8c3d070d8895a6da063789279895f52189e3
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88116774"
---
# <a name="what-authentication-and-verification-methods-are-available-in-azure-active-directory"></a>¿Qué métodos de autenticación y verificación hay disponibles en Azure Active Directory?

Como parte de la experiencia de inicio de sesión de las cuentas en Azure Active Directory (Azure AD), hay distintas maneras en las que un usuario puede autenticarse. Un nombre de usuario y una contraseña son la manera más común en que los usuarios han presentado sus credenciales a lo largo de la historia. Con las características de autenticación y seguridad modernas de Azure AD, esa contraseña básica se puede complementar o reemplazar con métodos de autenticación adicionales.

Un usuario de Azure AD puede elegir autenticarse con uno de los siguientes métodos de autenticación:

* Un nombre de usuario y una contraseña tradicionales
* Inicio de sesión sin contraseña de la aplicación Microsoft Authenticator
* Token de hardware OATH o llave de seguridad FIDO2
* Inicio de sesión sin contraseña basado en SMS

Muchas cuentas de Azure AD tienen habilitado el autoservicio de restablecimiento de contraseña (SSPR) o Azure Multi-Factor Authentication. Estas características incluyen métodos de verificación adicional, como una llamada telefónica o preguntas de seguridad. Se recomienda que solicite a los usuarios que registren varios métodos de verificación. Cuando un usuario no tiene disponible un método, puede elegir autenticarse con otro método.

En la tabla siguiente se describen los métodos disponibles para la autenticación principal o secundaria:

| Método | Autenticación principal | Autenticación secundaria |
| --- | --- | --- |
| [Contraseña](#password) | Sí | |
| [Aplicación Microsoft Authenticator](#microsoft-authenticator-app) | Sí (versión preliminar) | MFA y SSPR |
| [Llaves de seguridad FIDO2 (versión preliminar)](#fido2-security-keys) | Sí | Solo MFA |
| [Tokens de software OATH](#oath-software-tokens) | No | MFA |
| [Tokens de hardware OATH (versión preliminar)](#oath-hardware-tokens-preview) | No | MFA |
| [SMS](#phone-options) | Sí (versión preliminar) | MFA y SSPR |
| [Llamada de voz](#phone-options) | No | MFA y SSPR |
| [Preguntas de seguridad](#security-questions) | No | Solo autoservicio de restablecimiento de contraseña |
| [Dirección de correo electrónico](#email-address) | No | Solo autoservicio de restablecimiento de contraseña |
| [Contraseñas de aplicación](#app-passwords) | No | MFA solo en determinados casos |

Estos métodos de autenticación se pueden configurar en Azure Portal y, cada vez más, con la [Versión beta de la API REST Microsoft Graph API](/graph/api/resources/authenticationmethods-overview?view=graph-rest-beta).

En este artículo se describen los distintos métodos de autenticación y verificación disponibles en Azure AD y las limitaciones o restricciones específicas.

![Métodos de autenticación en uso en la pantalla de inicio de sesión](media/concept-authentication-methods/overview-login.png)

## <a name="password"></a>Contraseña

Una contraseña de Azure AD suele ser uno de los métodos de autenticación principales. El método de autenticación de contraseña no se puede deshabilitar.

Incluso si usa un método de autenticación como el [inicio de sesión basado en SMS](howto-authentication-sms-signin.md) cuando el usuario no use la contraseña para iniciar sesión, la contraseña sigue siendo un método de autenticación disponible.

## <a name="microsoft-authenticator-app"></a>Aplicación Microsoft Authenticator

La aplicación Authenticator proporciona un nivel de seguridad adicional para su cuenta profesional o educativa de Azure AD o su cuenta de Microsoft, y está disponible para [Android](https://go.microsoft.com/fwlink/?linkid=866594), [iOS](https://go.microsoft.com/fwlink/?linkid=866594) y [Windows Phone](https://www.microsoft.com/p/microsoft-authenticator/9nblgggzmcj6). Con la aplicación Microsoft Authenticator, los usuarios pueden autenticarse sin contraseñas durante el inicio de sesión, o como una opción de verificación adicional durante los eventos de autoservicio de restablecimiento de contraseña o de Azure Multi-Factor Authentication.

Los usuarios pueden recibir una notificación a través de la aplicación móvil para que aprueben o rechacen, o usar la aplicación Authenticator para generar un código de verificación de OATH que se pueda escribir en una interfaz de inicio de sesión. Si habilita tanto una notificación como un código de verificación, los usuarios que registran la aplicación Authenticator podrán utilizar cualquiera de los métodos para verificar su identidad.

Para usar la aplicación Authenticator en una solicitud de inicio de sesión en lugar de una combinación de nombre de usuario y contraseña, consulte [Habilitar el inicio de sesión sin contraseña en Azure AD con la aplicación Microsoft Authenticator (versión preliminar)](howto-authentication-passwordless-phone.md).

> [!NOTE]
> Los usuarios no tienen la opción de registrar su aplicación móvil cuando habilitan el autoservicio de restablecimiento de contraseña. En su lugar, los usuarios pueden registrar su aplicación móvil en [https://aka.ms/mfasetup](https://aka.ms/mfasetup) o como parte del registro de información de seguridad combinado en [https://aka.ms/setupsecurityinfo](https://aka.ms/setupsecurityinfo).

### <a name="notification-through-mobile-app"></a>Notificación a través de aplicación móvil

La aplicación Authenticator puede ayudar a impedir el acceso no autorizado a las cuentas y detener las transacciones fraudulentas mediante el envío de una notificación al smartphone o a la tableta. Los usuarios ven la notificación y, si es legítima, seleccionan **Comprobar**. De lo contrario, pueden seleccionar **Denegar**.

![Captura de pantalla del aviso de ejemplo en el explorador web para la notificación de la aplicación Authenticator para completar el proceso de inicio de sesión](media/tutorial-enable-azure-mfa/azure-multi-factor-authentication-browser-prompt.png)

> [!NOTE]
> Si su organización tiene personal que trabaja en China o que va a viajar allí, el método *Notificación a través de aplicación móvil* en dispositivos Android no funciona en ese país o región, ya que los servicios de Google Play están bloqueados en la región. Sin embargo, las notificaciones de iOS funcionan. En el caso de los dispositivos Android, se deben poner a disposición de los usuarios métodos de autenticación alternativos.

### <a name="verification-code-from-mobile-app"></a>Código de verificación desde aplicación móvil

La aplicación Authenticator puede utilizarse como un token de software para generar un código de verificación de OATH. Después de escribir el nombre de usuario y la contraseña, especifique el código que facilita la aplicación Authenticator en la interfaz de inicio de sesión. El código de verificación es una forma adicional de autenticación.

Los usuarios pueden tener una combinación de hasta cinco tokens de hardware OATH o aplicaciones de autenticación, como la aplicación Microsoft Authenticator, configurada para utilizarse en cualquier momento.

> [!WARNING]
> Para garantizar el máximo nivel de seguridad para el autoservicio de restablecimiento de contraseña cuando se requiere solo un método para el restablecimiento, un código de verificación es la única opción disponible para los usuarios.
>
> Si se requieren dos métodos, los usuarios pueden usar una notificación o el código de verificación, además de con cualquier otro método habilitado.

## <a name="fido2-security-keys"></a>Claves de seguridad FIDO2

FIDO (Fast IDentity Online) Alliance ayuda a promover los estándares de autenticación abiertos y a reducir el uso de contraseñas como forma de autenticación. FIDO2 es el estándar más reciente que incorpora el estándar de autenticación web (WebAuthn).

Para usar las llaves de seguridad de FIDO2 en una solicitud de inicio de sesión en lugar de una combinación de nombre de usuario y contraseña, consulte [Habilitar el inicio de sesión con clave de seguridad sin contraseña (versión preliminar)](howto-authentication-passwordless-security-key.md).

Los usuarios pueden registrarse y luego seleccionar una llave de seguridad de FIDO2 en la interfaz de inicio de sesión como medio principal de autenticación. Estas llaves de seguridad de FIDO2 suelen ser dispositivos USB, pero también pueden usar Bluetooth o NFC. Con un dispositivo de hardware que controla la autenticación, se aumenta la seguridad de una cuenta, ya que no hay ninguna contraseña que pueda quedar expuesta ni adivinarse.

Actualmente, las llaves de seguridad de FIDO2 en Azure AD se encuentran en versión preliminar. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="oath-tokens"></a>Tokens OATH

TOTP (contraseñas de un solo uso y duración definida) de OATH es un estándar abierto que especifica cómo se general los códigos de contraseña de un solo uso (OTP). TOTP de OATH se puede implementar mediante software o hardware para generar los códigos. Azure AD no es compatible con HOTP de OATH, otro estándar de generación de códigos.

### <a name="oath-software-tokens"></a>Tokens de software OATH

Los tokens de software OATH suelen ser aplicaciones, como la aplicación Microsoft Authenticator y otras aplicaciones de autenticador. Azure AD genera la clave secreta, o valor de inicialización, que se introduce en la aplicación y se usa para generar cada OTP.

La aplicación Authenticator genera códigos automáticamente cuando se configura para realizar notificaciones de inserción, de modo que un usuario tenga una copia de seguridad incluso si el dispositivo no tiene conectividad. También se pueden usar aplicaciones de terceros que usen TOTP de OATH para generar códigos.

Algunos tokens de hardware TOTP de OATH son programables, es decir, no incluyen una clave secreta ni un valor de inicialización programados previamente. Estos tokens de hardware programables se pueden configurar con la clave secreta o el valor de inicialización obtenidos del flujo de configuración del token de software. Los clientes pueden adquirir estos tokens del proveedor de su elección y usar la clave secreta o el valor de inicialización en el proceso de configuración de su proveedor.

### <a name="oath-hardware-tokens-preview"></a>Tokens de hardware OATH (versión preliminar)

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

## <a name="phone-options"></a>Opciones de teléfono

En el caso de la autenticación directa mediante mensajes de texto, puede consultar [Configuración y habilitación de los usuarios para la autenticación basada en SMS mediante Azure Active Directory (versión preliminar)](howto-authentication-sms-signin.md). El inicio de sesión basado en SMS es ideal para los trabajadores de primera línea. Con el inicio de sesión basado en SMS, los usuarios no necesitan conocer un nombre de usuario y una contraseña para acceder a las aplicaciones y servicios. En su lugar, el usuario escribe su número de teléfono móvil registrado, recibe un mensaje de texto con un código de verificación, y escribe el código en la interfaz de inicio de sesión.

Los usuarios también pueden verificarse mediante su teléfono móvil o teléfono de la oficina como forma secundaria de autenticación con Azure Multi-Factor Authentication o el autoservicio de restablecimiento de contraseña (SSPR).

Para funcionar correctamente, los números de teléfono deben tener el formato *+códigoPaís númeroTeléfono*, *por ejemplo: +1 4251234567*.

> [!NOTE]
> Debe haber un espacio entre el código de país/región y el número de teléfono.
>
> El restablecimiento de contraseña no admite extensiones telefónicas. Incluso con el formato *+1 4251234567X12345*, las extensiones se quitan antes de hacer la llamada.

### <a name="mobile-phone-verification"></a>Verificación por teléfono móvil

En el caso de Azure Multi-Factor Authentication o autoservicio de restablecimiento de contraseña, los usuarios pueden elegir recibir un mensaje de texto con un código de verificación para acceder a la interfaz de inicio de sesión, o recibir una llamada de teléfono con una notificación para escribir su código PIN definido.

Si los usuarios no quieren que su número de teléfono móvil sea visible en el directorio, pero quieren usarlo para restablecer la contraseña, los administradores no deben rellenar el número de teléfono en el directorio. En su lugar, los usuarios deben rellenar el atributo **Teléfono de autenticación** mediante el registro de información de seguridad combinado en [https://aka.ms/setupsecurityinfo](https://aka.ms/setupsecurityinfo). Los administradores pueden ver esta información en el perfil del usuario, pero no se publica en ningún otro lugar.

![Captura de pantalla de Azure Portal, que muestra los métodos de autenticación con un número de teléfono rellenado](media/concept-authentication-methods/user-authentication-methods.png)

Microsoft no garantiza la entrega rápida y coherente de Azure Multi-Factor Authentication por SMS o llamada de voz en el mismo número. Por el interés de los usuarios, podemos agregar o quitar códigos cortos cuando lo estimemos oportuno en tanto realicemos ajustes de enrutamiento para mejorar la capacidad de entrega de SMS. Microsoft no admite códigos cortos para países o regiones que no sean Estados Unidos y Canadá.

#### <a name="text-message-verification"></a>Verificación por mensaje de texto

Con la verificación por mensajes de texto durante el autoservicio de restablecimiento de contraseña o Azure Multi-Factor Authentication, se envía un SMS con un código de verificación al número de teléfono móvil. Para completar el proceso de inicio de sesión, el código de verificación entregado debe introducirse en la interfaz de inicio de sesión.

#### <a name="phone-call-verification"></a>Verificación por llamada telefónica

Con la verificación por llamada telefónica durante el autoservicio de restablecimiento de contraseña o Azure Multi-Factor Authentication, se hace una llamada de voz automatizada al número de teléfono registrado por el usuario. Para completar el proceso de inicio de sesión, se pide al usuario que escriba el número PIN, seguido de # en el teclado.

### <a name="office-phone-verification"></a>Verificación por teléfono de la oficina

El administrador de Azure AD administra el atributo de teléfono de la oficina, que no puede ser registrado por el usuario mismo.

Con la verificación por llamada telefónica durante el autoservicio de restablecimiento de contraseña o Azure Multi-Factor Authentication, se hace una llamada de voz automatizada al número de teléfono registrado por el usuario. Para completar el proceso de inicio de sesión, se pide al usuario que escriba el número PIN, seguido de # en el teclado.

### <a name="troubleshooting-phone-options"></a>Solución de problemas de las opciones de teléfono

Si tiene problemas con la autenticación telefónica en Azure AD, revise los siguientes pasos de solución de problemas:

* Identificador de llamada bloqueado en un solo dispositivo.
   * Revise los números bloqueados configurados en el dispositivo.
* Número de teléfono incorrecto o código de país o región incorrecto, o confusión entre el número telefónico personal y el número telefónico del trabajo.
   * Solución de problemas de objeto de usuario y métodos de autenticación configurados. Asegúrese de haber registrado los números de teléfono correctos.
* El PIN especificado es incorrecto.
   * Confirme que el usuario ha usado el PIN correcto, según está registrado para su cuenta.
* La llamada se desvió al correo de voz.
   * Asegúrese de que el usuario tiene encendido el teléfono y que el servicio está disponible en su área, o use un método alternativo.
* El usuario está bloqueado
   * Solicite al administrador de Azure AD que desbloquee el usuario en Azure Portal.
* No se han suscrito los SMS en el dispositivo.
   * Solicite al usuario que cambie los métodos o active los SMS en el dispositivo.
* Problemas con los proveedores de telecomunicaciones, por ejemplo: no se detecta entrada de teléfono, incidencias sobre la falta de tonos DTMF, identificador de llamada bloqueado en varios dispositivos o SMS bloqueados en varios dispositivos.
   * Microsoft usa varios proveedores de telecomunicaciones con el fin de enrutar las llamadas telefónicas y los mensajes SMS para la autenticación. Si observa alguno de los problemas anteriores, solicite al usuario que intente usar el método al menos cinco veces en 5 minutos, y tenga la información del usuario disponible cuando se ponga en contacto con el soporte técnico de Microsoft.

## <a name="security-questions"></a>Preguntas de seguridad

Las preguntas de seguridad no se usan como método de autenticación durante un evento de inicio de sesión. En su lugar, las preguntas de seguridad se pueden usar durante el proceso de autoservicio de restablecimiento de contraseña (SSPR) para confirmar quién es. Las cuentas de administrador no pueden usar preguntas de seguridad como método de verificación con SSPR.

Cuando los usuarios se registren en SSPR, se les pedirá que elijan los métodos de autenticación que usarán. Si optan por usar preguntas de seguridad, pueden elegir entre un conjunto de preguntas para que se muestren y, a continuación, proporcionar sus propias respuestas.

![Captura de pantalla de Azure Portal, que muestra los métodos de autenticación y las opciones para las preguntas de seguridad](media/concept-authentication-methods/security-questions-authentication-method.png)

> [!NOTE]
> Las preguntas de seguridad se almacenan de manera privada y segura en un objeto de usuario del directorio y solo las pueden responder los usuarios durante el registro. No hay forma de que un administrador lea o modifique las preguntas o respuestas de un usuario.

Las preguntas de seguridad pueden ser menos seguras que otros métodos porque es posible que algunas personas sepan las respuestas de las preguntas de otro usuario. Si usa preguntas de seguridad con SSPR, se recomienda usarlas junto con otro método. Se puede solicitar al usuario usar la aplicación Microsoft Authenticator o la autenticación por teléfono para verificar su identidad durante el proceso de autoservicio de restablecimiento de contraseña, y elegir preguntas de seguridad solo si no tiene su teléfono o dispositivo registrado consigo.

### <a name="predefined-questions"></a>Preguntas predefinidas

Las siguientes preguntas de seguridad predefinidas están disponibles para su uso como método de verificación con SSPR. Todas estas preguntas de seguridad están traducidas y localizadas en el conjunto completo de idiomas de Office 365 en función de la configuración regional del explorador del usuario:

* ¿En qué ciudad conoció a su cónyuge o pareja?
* ¿En qué ciudad se conocieron sus padres?
* ¿En qué ciudad vive su hermano más próximo?
* ¿En qué ciudad nació su padre?
* ¿En qué ciudad tuvo su primer trabajo?
* ¿En qué ciudad nació su madre?
* ¿En qué ciudad estaba en la Nochevieja del año 2000?
* ¿Cuál es el apellido de su profesor favorito del instituto?
* ¿En qué universidad solicitó plaza pero no asistió?
* ¿Cómo se llama el lugar en el que celebró su primer matrimonio?
* ¿Cuál es el segundo apellido de su padre?
* ¿Cuál es su comida favorita?
* ¿Cuál es el nombre y apellido de su abuela materna?
* ¿Cuál es el segundo apellido de su madre?
* ¿Cuáles son el año y mes de nacimiento del mayor de sus hermanos? (por ejemplo, noviembre de 1985)
* ¿Cuál es el segundo apellido del mayor de sus hermanos?
* ¿Cuál es el nombre y apellido de su abuelo paterno?
* ¿Cuántos años se lleva con el menor de sus hermanos?
* ¿En qué escuela cursó el sexto curso?
* ¿Cuál era el nombre y apellido de su mejor amigo de la infancia?
* ¿Cuál era el nombre y apellido de su primera pareja?
* ¿Cuál era el nombre de su maestro de primaria favorito?
* ¿Cuál era la marca y modelo de su primer coche o moto?
* ¿Cómo se llamaba la primera escuela a la que asistió?
* ¿Cómo se llamaba el hospital en el que nació?
* ¿Cuál era la calle de su primera casa de la infancia?
* ¿Cuál era el nombre de su héroe de la infancia?
* ¿Cuál era el nombre de su peluche favorito?
* ¿Cuál era el nombre de su primera mascota?
* ¿Cuál era su apodo en la infancia?
* ¿Cuál era su deporte favorito en el instituto?
* ¿Cuál fue su primer trabajo?
* ¿Cuáles eran los cuatro últimos números de su teléfono de la infancia?
* Cuando era joven, ¿qué quería ser de mayor?
* ¿Cuál es la persona más famosa que ha conocido?

### <a name="custom-security-questions"></a>Preguntas de seguridad personalizadas

Para mayor flexibilidad, puede definir sus propias preguntas de seguridad personalizadas. La longitud máxima de una pregunta de seguridad personalizada es 200 caracteres.

Las preguntas de seguridad personalizadas no se localizan automáticamente como sí lo están las preguntas de seguridad predeterminadas. Todas las preguntas personalizadas se muestran en el mismo idioma en el que se han escrito en la interfaz de usuario administrativa, aunque la configuración regional del explorador del usuario sea otra. Si necesita preguntas localizadas, debería usar las preguntas predefinidas.

### <a name="security-question-requirements"></a>Requisitos de las preguntas de seguridad

En el caso tanto de las preguntas de seguridad predeterminadas como de las personalizadas, se aplican los siguientes requisitos y limitaciones:

* El límite mínimo de caracteres de las respuestas es de tres.
* El límite máximo de caracteres de las respuestas es de 40.
* Los usuarios no pueden responder a la misma pregunta más de una vez.
* Los usuarios no pueden proporcionar la misma respuesta a más de una pregunta.
* Se puede usar cualquier juego de caracteres para definir las preguntas y respuestas, incluidos los caracteres Unicode.
* El número de preguntas que se definen debe ser mayor o igual que el número de preguntas que fueron necesarias para registrarse.

## <a name="email-address"></a>Dirección de correo electrónico

No puede usarse una dirección de correo electrónico como método de autenticación directo. La dirección de correo electrónico solo está disponible como opción de verificación para el autoservicio de restablecimiento de contraseña (SSPR). Cuando se selecciona una dirección de correo electrónico durante el SSPR, se envía un correo electrónico al usuario para completar el proceso de autenticación y verificación.

Durante el registro en el SSPR, un usuario proporciona la dirección de correo electrónico que se va a usar. Se recomienda usar una cuenta de correo electrónico diferente a la de la cuenta corporativa para asegurar que se pueda acceder a ella durante el SSPR.

## <a name="app-passwords"></a>Contraseñas de aplicación

Algunas aplicaciones antiguas que no se basan en explorador no entienden las pausas ni interrupciones en el proceso de autenticación. Si un usuario tiene habilitada la autenticación multifactor e intenta usar una de estas aplicaciones antiguas sin explorador, normalmente no podrá autenticarse correctamente. Una contraseña de aplicación permite a los usuarios continuar con la autenticación correcta en aplicaciones antiguas sin explorador sin interrupciones.

De forma predeterminada, los usuarios no pueden crear contraseñas de aplicación. Si quiere permitir que los usuarios creen contraseñas de aplicación, seleccione **Permitir a los usuarios crear contraseñas de aplicación para iniciar sesión en aplicaciones que no son de explorador** en *Configuración del servicio* de las propiedades de Azure Multi-Factor Authentication del usuario.

![Captura de pantalla de Azure Portal que muestra la configuración del servicio para Multi-Factor Authentication para permitir el uso de contraseñas de aplicación](media/concept-authentication-methods/app-password-authentication-method.png)

Si exige Azure Multi-Factor Authentication a través de directivas de acceso condicional y no a través de MFA por usuario, no podrá crear contraseñas de aplicación. Las aplicaciones modernas que utilizan directivas de acceso condicional para controlar el acceso no necesitan contraseñas de aplicación.

Si su organización está federada con Azure AD para el inicio de sesión único (SSO) y usa Azure Multi-Factor Authentication, se aplican las siguientes consideraciones:

* La contraseña de aplicación se verifica mediante Azure AD y, por tanto, se omite la federación. La federación solo se usa activamente al configurar contraseñas de aplicación. Para los usuarios federados (SSO), las contraseñas se almacenan en el identificador de organización. Si el usuario abandona la empresa, esta información tiene que fluir al identificador de la organización con DirSync. Los eventos de deshabilitación o eliminación de una cuenta pueden tardar hasta tres horas en sincronizarse, lo que retrasa la deshabilitación o eliminación de las contraseñas de aplicación en Azure AD.
* Las contraseñas de aplicación no cumplen con la configuración del control de acceso de cliente local.
* No hay disponible ninguna funcionalidad de registro o auditoría de autenticación local para las contraseñas de aplicación.
* Algunos diseños de arquitectura avanzada pueden requerir el uso de una combinación de nombre de usuario y contraseñas de la organización y de contraseñas de aplicación cuando se usa la autenticación multifactor, en función de dónde se autentiquen.
    * Para los clientes que se autentican en una infraestructura local, usaría un nombre de usuario y una contraseña de la organización.
    * Para los clientes que se autentican en Azure AD, usaría la contraseña de aplicación.

## <a name="next-steps"></a>Pasos siguientes

Para comenzar, consulte el [tutorial del autoservicio de restablecimiento de contraseña][tutorial-sspr] y [Azure Multi-Factor Authentication][tutorial-azure-mfa].

Para obtener más información sobre los conceptos del SSPR, consulte [Funcionamiento: Autoservicio de restablecimiento de contraseña de Azure AD][concept-sspr].

Para obtener más información sobre los conceptos de MFA, consulte [Funcionamiento: Azure Multi-Factor Authentication][concept-mfa].

Más información sobre la configuración de métodos de autenticación con la [Versión beta de la API REST Microsoft Graph API](/graph/api/resources/authenticationmethods-overview?view=graph-rest-beta).

<!-- INTERNAL LINKS -->
[tutorial-sspr]: tutorial-enable-sspr.md
[tutorial-azure-mfa]: tutorial-enable-azure-mfa.md
[concept-sspr]: concept-sspr-howitworks.md
[concept-mfa]: concept-mfa-howitworks.md
