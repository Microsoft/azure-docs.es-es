---
title: Inicio de sesión sin contraseña de Azure Active Directory (versión preliminar)
description: Conozca las opciones para el inicio de sesión sin contraseña en Azure Active Directory con las claves de seguridad FIDO2 o la aplicación Microsoft Authenticator.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: librown
ms.collection: M365-identity-device-management
ms.openlocfilehash: ef1148555706ff04c58733b66f4784da71849ce8
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89226682"
---
# <a name="passwordless-authentication-options-for-azure-active-directory"></a>Opciones de autenticación sin contraseña de Azure Active Directory

Características como la autenticación multifactor (MFA) son una excelente manera de proteger la organización, aunque los usuarios se frustran a menudo con la capa de seguridad adicional de tener que recordar sus contraseñas. Los métodos de autenticación sin contraseña resultan más cómodos, ya que la contraseña se quita y se reemplaza por algo que se tiene más algo que se es o se sabe.

| Authentication  | Algo que se tiene | Algo que se es o se sabe |
| --- | --- | --- |
| Inicio de sesión sin contraseña | Dispositivo, teléfono o clave de seguridad con Windows 10 | PIN o biométrica |

Cada organización tiene diferentes necesidades en cuanto a la autenticación. Microsoft ofrece las tres opciones siguientes de autenticación sin contraseña que se integran con Azure Active Directory (Azure AD):

- Windows Hello para empresas
- Aplicación Microsoft Authenticator
- Claves de seguridad FIDO2

![Autenticación: Seguridad frente a comodidad](./media/concept-authentication-passwordless/passwordless-convenience-security.png)

## <a name="windows-hello-for-business"></a>Windows Hello para empresas

Windows Hello para empresas resulta muy conveniente para los trabajadores de la información que tienen su propio equipo con Windows designado. La información biométrica y las credenciales de PIN están vinculadas directamente al equipo del usuario, lo que impide el acceso de cualquier persona que no sea el propietario. Con la integración de la infraestructura de clave pública (PKI) y la compatibilidad integrada con el inicio de sesión único (SSO), Windows Hello para empresas ofrece un método sencillo y práctico de acceder directamente a los recursos corporativos del entorno local y la nube.

![Ejemplo de inicio de sesión de un usuario con Windows Hello para empresas](./media/concept-authentication-passwordless/windows-hellow-sign-in.jpeg)

En los pasos siguientes se muestra cómo funciona el proceso de inicio de sesión con Azure AD:

![Diagrama que describe los pasos necesarios para el inicio de sesión de un usuario con Windows Hello para empresas](./media/concept-authentication-passwordless/windows-hello-flow.png)

1. Un usuario inicia sesión en Windows mediante gestos de PIN o de información biométrica. El gesto desbloquea la clave privada de Windows Hello para empresas y se envía al proveedor de compatibilidad para seguridad de la autenticación en la nube, conocido como el *proveedor de punto de acceso de nube*.
1. El proveedor de punto de acceso de nube solicita un valor nonce a Azure AD.
1. Azure AD devuelve un valor nonce que es válido durante 5 minutos.
1. El proveedor de punto de acceso de nube firma el valor nonce con la clave privada del usuario y devuelve el valor nonce firmado a Azure AD.
1. Azure AD valida el valor nonce firmado con la clave pública del usuario registrada de forma segura en la firma del valor nonce. Después de validar la firma, Azure AD valida el valor nonce firmado devuelto. Tras validar el valor nonce, Azure AD crea un token de actualización principal (PRT) con la clave de sesión que se ha cifrado con la clave de transporte del dispositivo y lo devuelve al proveedor de punto de acceso de nube.
1. El proveedor de CloudAP recibe el PRT cifrado con la clave de sesión. Mediante la clave de transporte privada del dispositivo, el proveedor de punto de acceso de nube descifra la clave de sesión y la protege con el Módulo de plataforma segura (TPM) del dispositivo.
1. El proveedor de acceso de punto de nube devuelve una respuesta de autenticación correcta a Windows. Después, el usuario puede acceder a Windows, así como a las aplicaciones locales y en la nube sin necesidad de autenticarse de nuevo (SSO).

La [guía de planeamiento](/windows/security/identity-protection/hello-for-business/hello-planning-guide) de Windows Hello para empresas se puede usar para ayudarle a tomar decisiones sobre el tipo de implementación y las opciones que es necesario tener en cuenta.

## <a name="microsoft-authenticator-app"></a>Aplicación Microsoft Authenticator

También puede permitir que el teléfono del empleado se convierta en un método de autenticación sin contraseña. Es posible que ya esté usando la aplicación Microsoft Authenticator como una opción de autenticación multifactor cómoda sumada a una contraseña. También puede usar la aplicación Authenticator como una opción sin contraseña.

![Inicio de sesión en Microsoft Edge con la aplicación Microsoft Authenticator](./media/concept-authentication-passwordless/concept-web-sign-in-microsoft-authenticator-app.png)

La aplicación Authenticator convierte cualquier teléfono Android o iOS en una credencial segura sin contraseña. Los usuarios pueden iniciar sesión en cualquier plataforma o explorador con este proceso: reciben una notificación en su teléfono, comprueban que el número mostrado en la pantalla coincide con el de su teléfono y, a continuación, usan datos biométricos (reconocimiento táctil o facial) o el PIN para confirmarlo. Consulte [Descarga e instalación de la aplicación Microsoft Authenticator](../user-help/user-help-auth-app-download-install.md) para conocer los detalles de la instalación.

La autenticación sin contraseñas mediante la aplicación Authenticator sigue el mismo patrón básico que Windows Hello para empresas. Es un poco más complicado, ya que el usuario debe identificarse para que Azure AD pueda encontrar la versión de la aplicación Microsoft Authenticator que se está usando:

![Diagrama que describe los pasos necesarios para el inicio de sesión de un usuario con la aplicación Microsoft Authenticator](./media/concept-authentication-passwordless/authenticator-app-flow.png)

1. El usuario escribe su nombre de usuario.
1. Azure AD detecta que el usuario tiene una credencial segura e inicia el flujo de credencial segura.
1. Se envía una notificación a la aplicación mediante Apple Push Notification Service (APNS) en dispositivos iOS o por medio de Firebase Cloud Messaging (FCM) en dispositivos Android.
1. El usuario recibe la notificación push y abre la aplicación.
1. La aplicación llama a Azure AD y recibe un desafío de prueba de presencia y un valor nonce.
1. Para completar el desafío, el usuario escribe su información biométrica o su PIN para desbloquear la clave privada.
1. El valor nonce se firma con la clave privada y se envía a Azure AD.
1. Azure AD realiza la validación de la clave pública-privada y devuelve un token.

Para empezar a trabajar con el inicio de sesión sin contraseña, complete el procedimiento siguiente:

> [!div class="nextstepaction"]
> [Habilitar el inicio de sesión sin contraseña con la aplicación Authenticator](howto-authentication-passwordless-phone.md)

## <a name="fido2-security-keys"></a>Claves de seguridad FIDO2

Las claves de seguridad FIDO2 son un método de autenticación sin contraseña basado en estándares que no permite la suplantación de identidad y que puede venir en cualquier factor de forma. Fast Identity Online (FIDO) es un estándar abierto para la autenticación sin contraseña. FIDO permite a los usuarios y a las organizaciones aprovechar el estándar para iniciar sesión en sus recursos sin un nombre de usuario o una contraseña mediante una clave de seguridad externa o una clave de plataforma integrada en un dispositivo.

Los empleados pueden usar claves de seguridad para iniciar sesión en sus dispositivos Windows 10 unidos a Azure AD o Azure AD híbrido y lograr el inicio de sesión único en sus recursos de nube y locales. Los usuarios también pueden iniciar sesión en exploradores compatibles. Las claves de seguridad FIDO2 son una excelente opción para las empresas que son muy conscientes de la seguridad o tienen escenarios o empleados que no quieren o no pueden usar su teléfono como un segundo factor.

Actualmente, el inicio de sesión con las claves de seguridad FIDO2 en Azure AD se encuentran en versión preliminar.

![Inicio de sesión en Microsoft Edge con una clave de seguridad](./media/concept-authentication-passwordless/concept-web-sign-in-security-key.png)

El proceso siguiente se utiliza cuando un usuario inicia sesión con una clave de seguridad FIDO2:

![Diagrama que describe los pasos necesarios para el inicio de sesión de un usuario con una clave de seguridad FIDO2](./media/concept-authentication-passwordless/fido2-security-key-flow.png)

1. El usuario conecta la clave de seguridad FIDO2 en su equipo.
2. Windows detecta la llave de seguridad FIDO2.
3. Windows envía una solicitud de autenticación.
4. Azure AD devuelve un valor nonce.
5. El usuario realiza su gesto para desbloquear la clave privada almacenada en el enclave seguro de la llave de seguridad FIDO2.
6. La llave de seguridad FIDO2 firma el valor nonce con la clave privada.
7. La solicitud del token de actualización principal (PRT) con el valor nonce firmado se envía a Azure AD.
8. Azure AD comprueba el valor nonce firmado con la clave pública FIDO2.
9. Azure AD devuelve el PRT para permitir el acceso a los recursos locales.

Aunque hay muchas claves certificadas como FIDO2 por FIDO Alliance, Microsoft necesita que el proveedor implemente algunas extensiones opcionales de la especificación Client-to-Authenticator Protocol (CTAP) FIDO2 para garantizar la máxima seguridad y la mejor experiencia.

Una clave de seguridad **DEBE** implementar las siguientes extensiones y características del protocolo FIDO2 CTAP para ser compatible con Microsoft:

| # | Confianza de característica o extensión | ¿Por qué se requiere esta característica o extensión? |
| --- | --- | --- |
| 1 | Clave residente | Esta característica permite que la clave de seguridad sea portátil, ya que la credencial se almacena en la clave de seguridad. |
| 2 | PIN de cliente | Esta característica permite proteger las credenciales con un segundo factor y se aplica a las claves de seguridad que no tienen una interfaz de usuario. |
| 3 | hmac-secret | Esta extensión garantiza que pueda iniciar sesión en el dispositivo cuando está sin conexión o en modo avión. |
| 4 | Varias cuentas por RP | Esta característica garantiza que pueda usar la misma clave de seguridad en varios servicios, como Microsoft Account y Azure Active Directory. |

### <a name="fido2-security-key-providers"></a>Proveedores de claves de seguridad FIDO2

Los siguientes proveedores ofrecen claves de seguridad FIDO2 o diferentes factores de forma que permiten iniciar sesión sin contraseña. Animamos a los clientes a evaluar las propiedades de seguridad de estas claves poniéndose en contacto con el proveedor y con FIDO Alliance.

| Proveedor | Contacto |
| --- | --- |
| Yubico | [https://www.yubico.com/support/contact/](https://www.yubico.com/support/contact/) |
| Feitian | [https://shop.ftsafe.us/pages/microsoft](https://shop.ftsafe.us/pages/microsoft) |
| HID | [https://www.hidglobal.com/contact-us](https://www.hidglobal.com/contact-us) |
| Ensurity | [https://www.ensurity.com/contact](https://www.ensurity.com/contact) |
| TrustKey Solutions | [https://www.trustkeysolutions.com/security-keys/](https://www.trustkeysolutions.com/security-keys/) |
| AuthenTrend | [https://authentrend.com/about-us/#pg-35-3](https://authentrend.com/about-us/#pg-35-3) |
| Gemalto (Grupo Thales) | [https://safenet.gemalto.com/multi-factor-authentication/authenticators/passwordless-authentication/](https://safenet.gemalto.com/multi-factor-authentication/authenticators/passwordless-authentication/) |
| OneSpan Inc. | [https://www.onespan.com/products/fido](https://www.onespan.com/products/fido) |
| IDmelon Technologies Inc. | [https://www.idmelon.com/#idmelon](https://www.idmelon.com/#idmelon) | 

> [!NOTE]
> Si adquiere y planea usar claves de seguridad basadas en NFC, necesita un lector NFC compatible para la clave de seguridad. El lector NFC no es un requisito o limitación de Azure. Consulte al proveedor de la clave de seguridad basada en NFC para obtener una lista de lectores de NFC admitidos.

Si es proveedor y quiere que su dispositivo aparezca en esta lista de dispositivos compatibles, póngase en contacto con [Fido2Request@Microsoft.com](mailto:Fido2Request@Microsoft.com).

Para empezar a usar las claves de seguridad FIDO2, realice el procedimiento siguiente:

> [!div class="nextstepaction"]
> [Habilitar el inicio de sesión con claves de seguridad FIDO2 sin contraseña](howto-authentication-passwordless-security-key.md)


## <a name="what-scenarios-work-with-the-preview"></a>¿Qué escenarios funcionan con la versión preliminar?

Las características de inicio de sesión sin contraseña de Azure AD se encuentran actualmente en versión preliminar. Se aplican las siguientes consideraciones:

- Los administradores pueden habilitar métodos de autenticación sin contraseña para su inquilino
- Los administradores pueden establecer como destino a todos los usuarios o seleccionar usuarios o grupos dentro de su inquilino para cada método
- Los usuarios finales pueden registrar y administrar estos métodos de autenticación sin contraseña en el portal de la cuenta
- Los usuarios finales pueden iniciar sesión con estos métodos de autenticación sin contraseña
   - Aplicación Microsoft Authenticator: Funciona en los escenarios donde se usa la autenticación de Azure AD, lo que incluye todos los exploradores, durante la configuración rápida (OOBE) de Windows 10 y con aplicaciones móviles integradas en cualquier sistema operativo.
   - Claves de seguridad: funcionan en la pantalla de bloqueo de Windows 10 e Internet en exploradores compatibles como Microsoft Edge (tanto versiones heredadas como la nueva Edge).

## <a name="choose-a-passwordless-method"></a>Elección de un método sin contraseña

La elección entre estas tres opciones sin contraseña depende de los requisitos de seguridad, plataforma y aplicación de su empresa.

Estos son algunos de los factores que se deben tener en cuenta al elegir la tecnología sin contraseña de Microsoft:

||**Windows Hello para empresas**|**Inicio de sesión sin contraseña con la aplicación Microsoft Authenticator**|**Llaves de seguridad FIDO2**|
|:-|:-|:-|:-|
|**Requisito previo**| Windows 10, versión 1809 o posterior<br>Azure Active Directory| Aplicación Microsoft Authenticator<br>Teléfono (dispositivos iOS y Android que ejecutan Android 6.0 o posterior)|Windows 10, versión 1809 o posterior<br>Azure Active Directory|
|**Modo**|Plataforma|Software|Hardware|
|**Sistemas y dispositivos**|PC con un módulo de plataforma segura (TPM) integrado<br>Reconocimiento de PIN e información biométrica |PIN y reconocimiento biométrico en el teléfono|Dispositivos de seguridad FIDO2 que son compatibles con Microsoft|
|**Experiencia del usuario**|Inicie sesión con un PIN o mediante reconocimiento biométrico (facial, iris o huella digital) con dispositivos Windows.<br>La autenticación de Windows Hello está vinculada al dispositivo; el usuario necesita el dispositivo y un componente de inicio de sesión, como un PIN o un factor biométrico, para acceder a los recursos corporativos.|Inicio de sesión con un teléfono móvil con la huella digital, el reconocimiento facial o del iris, o bien con un PIN.<br>Los usuarios inician sesión en su cuenta profesional o personal desde su PC o teléfono móvil.|Inicio de sesión con el dispositivo de seguridad FIDO2 (información biométrica, PIN y NFC)<br>El usuario puede acceder al dispositivo según los controles de la organización y autenticarse con un PIN, información biométrica mediante dispositivos como llaves de seguridad USB, y por medio de tarjetas inteligentes, llaves o dispositivos ponibles habilitados para NFC.|
|**Escenarios habilitados**| Experiencia sin contraseña con dispositivos Windows.<br>Se aplica a PC de trabajo dedicados con posibilidad de inicio de sesión único en aplicaciones y dispositivos.|Solución en cualquier parte sin contraseña con el teléfono móvil.<br>Se aplica para el acceso a aplicaciones de trabajo o personales en la web desde cualquier dispositivo.|Experiencia sin contraseña para los trabajadores con NFC, información biométrica y PIN.<br>Aplicable a PC compartidos y cuando un teléfono móvil no es una opción viable (por ejemplo, personal del departamento de soporte técnico, quiosco multimedia público o equipo de hospital).|

Use la tabla siguiente para elegir qué método será más adecuado para sus requisitos y usuarios.

|Persona|Escenario|Entorno|Tecnología sin contraseña|
|:-|:-|:-|:-|
|**Administrador**|Acceso seguro a un dispositivo para tareas de administración|Dispositivo Windows 10 asignado|Windows Hello para empresas, llave de seguridad FIDO2 o ambos|
|**Administrador**|Tareas de administración en dispositivos que no son Windows| Dispositivo móvil o que no sean Windows|Inicio de sesión sin contraseña con la aplicación Microsoft Authenticator|
|**Trabajador de la información**|Trabajo de productividad|Dispositivo Windows 10 asignado|Windows Hello para empresas, llave de seguridad FIDO2 o ambos|
|**Trabajador de la información**|Trabajo de productividad| Dispositivo móvil o que no sean Windows|Inicio de sesión sin contraseña con la aplicación Microsoft Authenticator|
|**Trabajador de primera línea**|Quioscos multimedia de una fábrica, planta, comercio o entrada de datos|Dispositivos Windows 10 compartidos|Llaves de seguridad FIDO2|

## <a name="next-steps"></a>Pasos siguientes

Para empezar a trabajar sin contraseñas en Azure AD, siga uno de los artículos de procedimientos:

* [Habilitar el inicio de sesión con clave de seguridad FIDO2 sin contraseña](howto-authentication-passwordless-security-key.md)
* [Habilitación del inicio de sesión sin contraseña mediante el teléfono con la aplicación Authenticator](howto-authentication-passwordless-phone.md)

### <a name="external-links"></a>Vínculos externos

* [FIDO Alliance](https://fidoalliance.org/)
* [Especificación FIDO2 CTAP](https://fidoalliance.org/specs/fido-v2.0-id-20180227/fido-client-to-authenticator-protocol-v2.0-id-20180227.html)