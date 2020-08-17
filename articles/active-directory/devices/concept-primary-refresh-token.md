---
title: 'Token de actualización principal (PRT) y Azure AD: Azure Active Directory'
description: ¿Cuál es la función del token de actualización principal (PRT) y cómo se administra en Azure Active Directory?
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: conceptual
ms.date: 07/20/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: ravenn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9971eb554825a968f8cfa72d6a0cf78d7c0bcb76
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87025887"
---
# <a name="what-is-a-primary-refresh-token"></a>¿Qué es un token de actualización principal?

Un token de actualización principal (PRT) es un artefacto de claves de autenticación de Azure AD en dispositivos Windows 10, iOS y Android. Se trata de un token JWT (JSON Web Token) emitido especialmente para agentes de token propios de Microsoft con el fin de habilitar el inicio de sesión único (SSO) en las aplicaciones usadas en esos dispositivos. En este artículo, se proporcionan detalles sobre cómo se emite, se usa y se protege un PRT en dispositivos Windows 10.

Se da por hecho que ya conoce los distintos estados de dispositivo disponibles en Azure AD y cómo funciona el inicio de sesión único en Windows 10. Para más información sobre los dispositivos de Azure AD, consulte el artículo [¿Qué es la administración de dispositivos en Azure Active Directory?](overview.md)

## <a name="key-terminology-and-components"></a>Componentes y terminología más importantes

Los siguientes componentes de Windows desempeñan un papel fundamental a la hora de solicitar y usar un PRT:

* **Proveedor de autenticación en la nube** (CloudAP): CloudAP es el proveedor de autenticación moderno para el inicio de sesión de Windows, que comprueba los usuarios conectados a un dispositivo Windows 10. CloudAP proporciona un marco de complementos en el que pueden basarse los proveedores de identidades para habilitar la autenticación en Windows mediante las credenciales del proveedor de identidades en cuestión.
* **Administrador de cuentas web** (WAM): WAM es el agente de token predeterminado en dispositivos Windows 10. WAM también proporciona un marco de complementos en el que pueden basarse los proveedores de identidades para habilitar el inicio de sesión único en sus aplicaciones que dependen del proveedor de identidades en cuestión.
* **Complemento CloudAP de Azure AD**: un complemento específico de Azure AD basado en el marco CloudAP, que comprueba las credenciales de usuario con Azure AD durante el inicio de sesión de Windows.
* **Complemento WAM de Azure AD**: un complemento específico de Azure AD basado en el marco WAM, que permite el inicio de sesión único en las aplicaciones que dependen de Azure AD para la autenticación.
* **Dsreg**: un componente específico de Azure AD en Windows 10, que controla el proceso de registro de dispositivos con todos los estados.
* **Módulo de plataforma segura (TPM)** : un TPM es un componente de hardware integrado en un dispositivo, que proporciona funciones de seguridad basadas en hardware para los secretos de los usuarios y de los dispositivos. Se puede encontrar más información en el artículo [Información general sobre la tecnología del Módulo de plataforma segura](/windows/security/information-protection/tpm/trusted-platform-module-overview).

## <a name="what-does-the-prt-contain"></a>¿Qué contiene el PRT?

Un PRT contiene notificaciones normalmente incluidas en cualquier token de actualización de Azure AD. Además, se incluyen algunas notificaciones específicas del dispositivo. Los pasos son los siguientes:

* **Identidad del dispositivo**: un PRT se emite para un usuario de un dispositivo específico. La notificación de la identidad del dispositivo `deviceID` determina el dispositivo en el que se emitió el PRT para el usuario. Esta notificación se emite más adelante para los tokens obtenidos mediante el PRT. La notificación de la identidad del dispositivo se usa para determinar la autorización del acceso condicional en función del estado o cumplimiento del dispositivo.
* **Clave de sesión**: la clave de sesión es una clave simétrica cifrada, generada por el servicio de autenticación de Azure AD, que se emite como parte del PRT. La clave de sesión actúa como prueba de posesión cuando se usa un PRT para obtener tokens para otras aplicaciones.

### <a name="can-i-see-whats-in-a-prt"></a>¿Se puede ver lo que hay en un PRT?

Un PRT es un blob opaco que se envía desde Azure AD y cuyo contenido es desconocido para los componentes cliente. No se puede ver lo que hay dentro de un PRT.

## <a name="how-is-a-prt-issued"></a>¿Cómo se emite un PRT?

Un requisito previo para la autenticación basada en el dispositivo en Azure AD es el registro de los dispositivos. Los PRT solo se emiten para los usuarios de dispositivos registrados. Para más información sobre el registro de dispositivos, consulte el artículo [Windows Hello para empresas y registro de dispositivos](/windows/security/identity-protection/hello-for-business/hello-how-it-works-device-registration). Durante el registro de dispositivos, el componente dsreg genera dos conjuntos de pares de claves criptográficas:

* Clave de dispositivo (dkpub/dkpriv)
* Clave de transporte (tkpub/tkpriv)

Las claves privadas se enlazan al TPM del dispositivo si este tiene un TPM válido que funcione, mientras que las claves públicas se envían a Azure AD durante el proceso de registro del dispositivo. Estas claves se usan para validar el estado del dispositivo durante las solicitudes PRT.

El PRT se emite durante la autenticación del usuario en un dispositivo Windows 10 en dos escenarios:

* **Unido a Azure AD** o **Unido a Azure AD híbrido**: cuando un usuario inicia sesión con sus credenciales de la organización, se emite un PRT durante el inicio de sesión de Windows. El PRT se emite con todas las credenciales admitidas por Windows 10, por ejemplo, contraseña y Windows Hello para empresas. En este escenario, el complemento CloudAP de Azure AD es la autoridad principal del PRT.
* **Dispositivos registrados de Azure AD**: un PRT se emite cuando un usuario agrega una cuenta profesional secundaria a su dispositivo Windows 10. Los usuarios pueden agregar una cuenta a Windows 10 de dos maneras diferentes:  
   * Mediante el aviso **Usar esta cuenta en cualquier lugar en el dispositivo** después de iniciar sesión en una aplicación (por ejemplo, Outlook)
   * Desde **Configuración** > **Cuentas** > **Obtener acceso a trabajo o escuela** > **Conectar**.

En escenarios de dispositivos registrados de Azure AD, el complemento WAM de Azure AD es la autoridad principal del PRT puesto que no va a tener lugar un inicio de sesión de Windows con esta cuenta de Azure AD.

> [!NOTE]
> Los proveedores de identidades de terceros necesitan admitir el protocolo WS-Trust para permitir la emisión de PRT en dispositivos Windows 10. Sin WS-Trust, no se puede emitir un PRT para los usuarios de dispositivos unidos a Azure AD híbrido o unidos a Azure AD. En ADFS solo se requieren puntos de conexión usernamemixed. Tanto adfs/services/trust/2005/windowstransport como adfs/services/trust/13/windowstransport se deben habilitar como puntos de conexión accesibles desde la intranet y **NO deben exponerse** como accesible desde la extranet mediante el Proxy de aplicación web.

## <a name="what-is-the-lifetime-of-a-prt"></a>¿Cuál es la duración de un PRT?

Una vez emitido, un PRT es válido durante 14 días y se renueva continuamente siempre y cuando el usuario use activamente el dispositivo.  

## <a name="how-is-a-prt-used"></a>¿Cómo se usa un PRT?

Dos componentes principales de Windows usan un PRT:

* **Complemento CloudAP de Azure AD**: durante el inicio de sesión de Windows, el complemento CloudAP de Azure AD solicita un PRT a Azure AD mediante las credenciales proporcionadas por el usuario. También almacena en caché el PRT para permitir el inicio de sesión almacenado en caché cuando el usuario no tiene acceso a una conexión de Internet.
* **Complemento WAM de Azure AD**: cuando los usuarios intentan acceder a las aplicaciones, el complemento WAM de Azure AD usa el PRT para permitir el inicio de sesión único en Windows 10. El complemento WAM de Azure AD usa el PRT para solicitar tokens de acceso y actualización para las aplicaciones en las que las solicitudes de tokens dependen de WAM. También permite el inicio de sesión único en los exploradores mediante la inserción del PRT en las solicitudes del explorador. El inicio de sesión único en el explorador en Windows 10 se admite en Microsoft Edge (de forma nativa) y en Chrome (mediante la extensión [Windows 10 Accounts](https://chrome.google.com/webstore/detail/windows-10-accounts/ppnbnpeolgkicgegkbkbjmhlideopiji?hl=en) u [Office Online](https://chrome.google.com/webstore/detail/office/ndjpnladcallmjemlbaebfadecfhkepb?hl=en)).

## <a name="how-is-a-prt-renewed"></a>¿Cómo se renueva un PRT?

Un PRT se renueva de dos maneras diferentes:

* **Complemento CloudAP de Azure AD cada cuatro horas**: el complemento CloudAP renueva el PRT cada cuatro horas durante el inicio de sesión de Windows. Si el usuario no tiene conexión a Internet durante ese tiempo, el complemento CloudAP renueva el PRT una vez que el dispositivo se conecta a Internet.
* **Complemento WAM de Azure AD durante las solicitudes de tokens de aplicación**: el complemento WAM permite el inicio de sesión único en dispositivos Windows 10 mediante la habilitación de solicitudes de token en modo silencioso en las aplicaciones. El complemento WAM puede renovar el PRT durante estas solicitudes de token de dos maneras diferentes:
   * Una aplicación solicita a WAM un token de acceso en modo silencioso, pero no hay ningún token de actualización disponible para esa aplicación. En este caso, WAM usa el PRT para solicitar un token para la aplicación y recibe un nuevo PRT en la respuesta.
   * Una aplicación solicita a WAM un token de acceso, pero el PRT no es válido o Azure AD necesita autorización adicional (por ejemplo, Azure Multi-factor Authentication). En este escenario, WAM realiza un inicio de sesión interactivo que pide al usuario que se vuelva a autenticar o que proporcione verificación adicional y, si la autenticación es correcta, se emite un nuevo PRT.

### <a name="key-considerations"></a>Consideraciones clave

* Un PRT solo se emite y se renueva durante la autenticación de aplicaciones nativas. Un PRT no se renueva ni emite durante una sesión del explorador.
* En dispositivos unidos a Azure AD y unidos a Azure AD híbrido, el complemento CloudAP es la autoridad principal para un PRT. Si se renueva un PRT durante una solicitud de token basada en WAM, el PRT se devuelve al complemento CloudAP, que comprueba su validez con Azure AD antes de aceptarlo.

## <a name="how-is-the-prt-protected"></a>¿Cómo se protege el PRT?

Para proteger un PRT, se enlaza con el dispositivo en el que el usuario ha iniciado la sesión. Azure AD y Windows 10 permiten la protección del PRT mediante los métodos siguientes:

* **Durante el primer inicio de sesión**: en el transcurso del primer inicio de sesión, se emite un PRT mediante la firma de las solicitudes que usan la clave del dispositivo generada criptográficamente durante el registro de dicho dispositivo. En un dispositivo con un TPM válido que funciona, la clave del dispositivo queda protegida por el TPM, que impide cualquier acceso malintencionado. Un PRT no se emite si no se puede validar la firma de la clave de dispositivo correspondiente.
* **Durante las solicitudes y renovaciones de tokens**: cuando se emite un PRT, Azure AD también emite una clave de sesión cifrada para el dispositivo. Esta clave se cifra con la clave de transporte pública (tkpub) generada y se envía a Azure AD como parte del registro del dispositivo. Esta clave de sesión solo se puede descifrar con la clave de transporte privada (tkpriv) protegida por el TPM. La clave de sesión es la clave de prueba de posesión (POP) de las solicitudes enviadas a Azure AD.  La clave de sesión también está protegida por el TPM y ningún otro componente del sistema operativo puede acceder a ella. Las solicitudes de token o las solicitudes de renovación del PRT están firmadas de forma segura con esta clave de sesión mediante el TPM, de ahí que, no se pueda alterar. Azure AD invalidará las solicitudes procedentes de dispositivos que no estén firmadas con la clave de sesión correspondiente.

Al proteger estas claves con el TPM, los actores malintencionados no pueden robarlas ni reproducir el PRT en otra parte dado que el TPM es inaccesible, incluso si un atacante está en posesión del dispositivo.  Por lo tanto, el uso de un TPM mejora enormemente la seguridad de los dispositivos unidos a Azure AD, unidos a Azure AD híbrido y registrados en Azure AD frente al robo de credenciales. En cuanto al rendimiento y la confiabilidad, TPM 2.0 es la versión recomendada para todos los escenarios de registro de dispositivos de Azure AD en Windows 10.

### <a name="how-are-app-tokens-and-browser-cookies-protected"></a>¿Cómo se protegen los tokens de la aplicación y las cookies del explorador?

**Tokens de la aplicación**: cuando una aplicación solicita un token mediante WAM, Azure AD emite un token de actualización y un token de acceso. Sin embargo, WAM solo devuelve el token de acceso a la aplicación y protege el token de actualización en su caché mediante su cifrado con la clave de la interfaz de programación de aplicaciones de protección de datos (DPAPI) del usuario. Para usar de forma segura el token de actualización, WAM firma las solicitudes con la clave de sesión para emitir tokens de acceso adicionales. La clave DPAPI está protegida con una clave simétrica basada en Azure AD en el propio Azure AD. Cuando el dispositivo tiene que descifrar el perfil de usuario con la clave DPAPI, Azure AD proporciona la clave DPAPI cifrada con la clave de sesión, que el complemento CloudAP solicita descifrar con TPM. Esta funcionalidad garantiza la coherencia en la protección de los tokens de actualización y evita que las aplicaciones implementen sus propios mecanismos de protección.  

**Cookies del explorador**: en Windows 10, Azure AD admite el inicio de sesión único del explorador en Internet Explorer y Microsoft Edge de forma nativa o en Google Chrome mediante la extensión Windows 10 Accounts. La seguridad se ha creado no solo para proteger las cookies, sino también los puntos de conexión a los que se envían. Las cookies del explorador están protegidas de la misma manera que un PRT, mediante la clave de sesión para firmar y proteger las cookies.

Cuando un usuario inicia una interacción con el explorador, el explorador (o la extensión) invoca un host de cliente nativo COM. El host de cliente nativo garantiza que la página procede de uno de los dominios permitidos. El explorador podría enviar otros parámetros al host de cliente nativo, como un nonce; sin embargo, el host de cliente nativo garantiza la validación del nombre de host. El host de cliente nativo solicita una cookie con PRT al complemento CloudAP, que la crea y la firma con la clave de sesión protegida por TPM. Como la cookie con PRT está firmada con la clave de sesión, no se puede alterar. Esta cookie con PRT se incluye en el encabezado de la solicitud para que Azure AD valide el dispositivo desde el que se origina. Si usa el explorador Chrome, solo se puede invocar con la extensión definida explícitamente en el manifiesto del host de cliente nativo, lo que impide que extensiones arbitrarias creen estas solicitudes. Una vez que Azure AD valida la cookie con PRT, emite una cookie de sesión para el explorador. Esta cookie de sesión también contiene la misma clave de sesión emitida con un PRT. Durante las solicitudes posteriores, para validar la clave de sesión de manera efectiva se enlaza la cookie con el dispositivo y se impide que se reproduzca desde cualquier parte.

## <a name="when-does-a-prt-get-an-mfa-claim"></a>¿Cuándo recibe un PRT una notificación de MFA?

Un PRT puede recibir una notificación de autenticación multifactor (MFA) en escenarios concretos. Cuando se usa un PRT basado en MFA para solicitar tokens para las aplicaciones, la notificación de MFA se transfiere a esos tokens. Esta funcionalidad proporciona a los usuarios una experiencia perfecta ya que evita el desafío de MFA en cada aplicación que lo requiera. Un PRT puede recibir una notificación de MFA de las maneras siguientes:

* **Inicio de sesión con Windows Hello para empresas**: Windows Hello para empresas reemplaza las contraseñas y usa claves criptográficas para proporcionar autenticación segura en dos fases. Windows Hello para empresas es específico de un usuario de un dispositivo y requiere MFA para el aprovisionamiento. Cuando un usuario inicia sesión con Windows Hello para empresas, el PRT del usuario recibe una notificación de MFA. Este escenario también se aplica a los usuarios que inician sesión con tarjetas inteligentes si la autenticación de la tarjeta inteligente genera una notificación de MFA desde ADFS.
   * Como Windows Hello para empresas se considera autenticación multifactor, la notificación de MFA se actualiza cuando se actualiza el propio PRT, por lo que la duración de MFA se extenderá continuamente cuando los usuarios inicien sesión con WIndows Hello para empresas
* **MFA durante el inicio de sesión interactivo de WAM**: durante una solicitud de token a través de WAM, si a un usuario se le exige MFA para acceder a la aplicación, el PRT que se renueva durante esta interacción se graba con una notificación de MFA.
   * En este caso, la notificación de MFA no se actualiza continuamente, por lo que la duración de MFA se basa en la duración establecida en el directorio.
   * Cuando se usan un PRT y un RT anteriores existentes para acceder a una aplicación, estos se consideran como la primera prueba de autenticación. Se requiere un nuevo AT con una segunda prueba y una notificación de MFA impresa. Este además emite un nuevo PRT y RT.
* **MFA durante el registro del dispositivo**: si un administrador ha configurado su dispositivo en Azure AD para [exigir MFA para registrar los dispositivos](device-management-azure-portal.md#configure-device-settings), el usuario necesita realizar MFA para completar el registro. Durante este proceso, el PRT emitido para el usuario tiene la notificación de MFA obtenida durante el registro. Esta funcionalidad solo se aplica al usuario que realizó la operación de unión, no a otros usuarios que inicien sesión en ese dispositivo.
   * Al igual que el inicio de sesión interactivo de WAM, la notificación de MFA no se actualiza continuamente, por lo que la duración de MFA se basa en la duración establecida en el directorio.

Windows 10 mantiene una lista separada de los PRT de cada credencial. Por lo tanto, hay un PRT para cada una de las credenciales de Windows Hello para empresas, contraseñas o tarjetas inteligentes. Esta separación garantiza que las notificaciones de MFA están aisladas en función de la credencial usada, y no se mezclan durante las solicitudes de token.

## <a name="how-is-a-prt-invalidated"></a>¿Cómo se invalida un PRT?

Un PRT se invalida en los escenarios siguientes:

* **Usuario no válido**: si un usuario se elimina o deshabilita en Azure AD, su PRT se invalida y no se puede usar para obtener tokens para las aplicaciones. Si un usuario eliminado o deshabilitado ya ha iniciado sesión en un dispositivo antes, el inicio de sesión almacenado en caché le dejaría conectarse, hasta que CloudAP se dé cuenta de que su estado no es válido. Una vez que CloudAP determina que el usuario no es válido, bloquea los posteriores inicios de sesión. A un usuario no válido se le impide automáticamente iniciar sesión en nuevos dispositivos que no tengan las credenciales almacenadas en caché.
* **Dispositivo no válido**: si un dispositivo se elimina o deshabilita en Azure AD, el PRT obtenido en ese dispositivo se invalida y no se puede usar para obtener tokens para otras aplicaciones. Si un usuario ya ha iniciado sesión en un dispositivo no válido, podrá seguir haciéndolo, pero todos los tokens del dispositivo se invalidan y el usuario no tiene inicio de sesión único en ningún recurso de ese dispositivo.
* **Cambio de contraseña**: después de que un usuario cambie su contraseña, Azure AD invalida el PRT obtenido con la contraseña anterior. El cambio de contraseña tiene como resultado la obtención de un nuevo PRT por el usuario. Esta invalidación puede suceder de dos maneras diferentes:
   * Si el usuario inicia sesión Windows con su nueva contraseña, CloudAP descarta el PRT antiguo y solicita a Azure AD que emita un nuevo PRT con su nueva contraseña. Si el usuario no tiene una conexión a Internet, la nueva contraseña no se puede validar, y Windows puede pedir al usuario que use su contraseña anterior.
   * Si un usuario ha iniciado sesión con su contraseña anterior o ha cambiado su contraseña después de iniciar sesión en Windows, se usa el antiguo PRT en las solicitudes de token basadas en WAM. En este escenario, se pide al usuario que se autentique de nuevo durante la solicitud del token de WAM y se emite un nuevo PRT.
* **Problemas del TPM**: en ocasiones, el TPM de un dispositivo puede producir un error y dar lugar a la inaccesibilidad de las claves protegidas por él. En este caso, el dispositivo no puede obtener un PRT ni solicitar tokens mediante un PRT existente ya que no puede demostrar la posesión de las claves criptográficas. Como resultado, Azure AD invalida los PRT existentes. Cuando Windows 10 detecta un error, inicia un flujo de recuperación para volver a registrar el dispositivo con nuevas claves criptográficas. Con la unión a Azure AD híbrido, al igual que con el registro inicial, la recuperación se produce en modo silencioso sin intervención del usuario. En los dispositivos registrados en Azure AD o unidos a Azure AD, la recuperación debe realizarla un usuario que tenga privilegios de administrador en el dispositivo en cuestión. En este escenario, un aviso de Windows inicia el flujo de recuperación que guía al usuario en la recuperación satisfactoria del dispositivo.

## <a name="detailed-flows"></a>Flujos detallados

Los siguientes diagramas ilustran los detalles subyacentes en la emisión, la renovación y el uso de un PRT para solicitar un token de acceso para una aplicación. Además, estos pasos también describen cómo se aplican los mecanismos de seguridad mencionados anteriormente durante estas interacciones.

### <a name="prt-issuance-during-first-sign-in"></a>Emisión del PRT durante el primer inicio de sesión

![Flujo detallado de la emisión del PRT durante el primer inicio de sesión](./media/concept-primary-refresh-token/prt-initial-sign-in.png)

> [!NOTE]
> En los dispositivos unidos a Azure AD, este intercambio se produce sincrónicamente para emitir un PRT y que el usuario pueda iniciar sesión en Windows. En dispositivos unidos a Azure AD híbrido, el directorio local de Active Directory es la autoridad principal. Por lo tanto, el usuario solo espera a obtener un TGT para iniciar la sesión, mientras que la emisión del PRT tiene lugar de manera asincrónica. Este escenario no se aplica a los dispositivos registrados en Azure AD dado que el inicio de sesión no usa credenciales de Azure AD.

| Paso | Descripción |
| :---: | --- |
| Un | El usuario escribe su contraseña en la interfaz de usuario de inicio de sesión. LogonUI pasa las credenciales contenidas en un búfer de autenticación a LSA que, a su vez, las pasa internamente a CloudAP. CloudAP reenvía esta solicitud al complemento CloudAP. |
| B | El complemento CloudAP inicia una solicitud de detección del dominio para identificar el proveedor de identidades del usuario. Si el inquilino del usuario tiene un programa de instalación del proveedor de federación, Azure AD devuelve el punto de conexión de intercambio de metadatos (MEX) del proveedor de federación. Si no, Azure AD devuelve que el usuario está administrado, lo que indica que el usuario puede autenticarse con Azure AD. |
| C | Si el usuario está administrado, CloudAP obtendrá el nonce de Azure AD. Si el usuario está federado, el complemento CloudAP solicita un token SAML al proveedor de federación con las credenciales del usuario. Una vez recibido el token SAML, solicita un nonce a Azure AD. |
| D | El complemento CloudAP construye la solicitud de autenticación con las credenciales del usuario, el nonce o un ámbito de agente, firma la solicitud con la clave del dispositivo (dkpriv) y la envía a Azure AD. En un entorno federado, el complemento CloudAP usa el token SAML devuelto por el proveedor de federación en lugar de las credenciales del usuario. |
| E | Azure AD valida las credenciales del usuario, el nonce y la firma del dispositivo, comprueba que el dispositivo es válido en el inquilino y emite el PRT cifrado. Junto con el PRT, Azure AD también emite una clave simétrica llamada clave de sesión cifrada por Azure AD mediante la clave de transporte (tkpub). Además, la clave de sesión también se inserta en el PRT. Esta clave de sesión actúa como la clave de prueba de posesión (PoP) para las solicitudes posteriores con el PRT. |
| F | El complemento CloudAP pasa el PRT y la clave de sesión cifrados a CloudAP. CloudAP solicita al TPM que descifre la clave de sesión mediante la clave de transporte (tkpriv) y la vuelve a cifrar con la propia clave del TPM. CloudAP almacena la clave de sesión cifrada en su caché junto con el PRT. |

### <a name="prt-renewal-in-subsequent-logons"></a>Renovación del PRT en los sucesivos inicios de sesión

![Renovación del PRT en los sucesivos inicios de sesión](./media/concept-primary-refresh-token/prt-renewal-subsequent-logons.png)

| Paso | Descripción |
| :---: | --- |
| Un | El usuario escribe su contraseña en la interfaz de usuario de inicio de sesión. LogonUI pasa las credenciales contenidas en un búfer de autenticación a LSA que, a su vez, las pasa internamente a CloudAP. CloudAP reenvía esta solicitud al complemento CloudAP. |
| B | Si anteriormente el usuario ha iniciado sesión, Windows realiza el inicio de sesión almacenado en caché y valida las credenciales de inicio de sesión del usuario. Cada cuatro horas, el complemento CloudAP inicia la renovación del PRT de manera asincrónica. |
| C | El complemento CloudAP inicia una solicitud de detección del dominio para identificar el proveedor de identidades del usuario. Si el inquilino del usuario tiene un programa de instalación del proveedor de federación, Azure AD devuelve el punto de conexión de intercambio de metadatos (MEX) del proveedor de federación. Si no, Azure AD devuelve que el usuario está administrado, lo que indica que el usuario puede autenticarse con Azure AD. |
| D | Si el usuario está federado, el complemento CloudAP solicita un token SAML al proveedor de federación con las credenciales del usuario. Una vez recibido el token SAML, solicita un nonce a Azure AD. Si el usuario está administrado, CloudAP obtendrá directamente el nonce de Azure AD. |
| E | El complemento CloudAP construye la solicitud de autenticación con las credenciales del usuario, el nonce y el PRT existente, firma la solicitud con la clave de sesión y la envía a Azure AD. En un entorno federado, el complemento CloudAP usa el token SAML devuelto por el proveedor de federación en lugar de las credenciales del usuario. |
| F | Azure AD valida la firma de la clave de sesión, para lo cual la compara con la clave de sesión insertada en el PRT, valida el nonce y comprueba que el dispositivo sea válido en el inquilino y emite un nuevo PRT. Como se vio antes, el PRT viene acompañado de nuevo por la clave de sesión cifrada mediante la clave de transporte (tkpub). |
| G | El complemento CloudAP pasa el PRT y la clave de sesión cifrados a CloudAP. CloudAP solicita al TPM que descifre la clave de sesión mediante la clave de transporte (tkpriv) y la vuelve a cifrar con la propia clave del TPM. CloudAP almacena la clave de sesión cifrada en su caché junto con el PRT. |

> [!NOTE]
> Un PRT se puede renovar externamente sin necesidad de una conexión VPN cuando los puntos de conexión usernamemixed están habilitados externamente.

### <a name="prt-usage-during-app-token-requests"></a>Uso del PRT durante las solicitudes de tokens de aplicación

![Uso del PRT durante las solicitudes de tokens de aplicación](./media/concept-primary-refresh-token/prt-usage-app-token-requests.png)

| Paso | Descripción |
| :---: | --- |
| Un | Una aplicación (por ejemplo, Outlook, OneNote, etc.) inicia una solicitud de token a WAM. WAM, a su vez, pide al complemento WAM de Azure AD que atienda la solicitud de token. |
| B | Si un token de actualización de la aplicación ya está disponible, el complemento WAM de Azure AD lo usa para solicitar un token de acceso. Para proporcionar una prueba de enlace de dispositivo, el complemento WAM firma la solicitud con la clave de sesión. Azure AD valida la clave de sesión y emite un token de acceso y un nuevo token de actualización para la aplicación, cifrados con la clave de sesión. El complemento WAM solicita al complemento CloudAP que descifre los tokens, el cual, a su vez, solicita al TPM que los descifre con la clave de sesión, lo que da lugar a que el complemento WAM obtenga ambos tokens. A continuación, el complemento WAM proporciona solo el token de acceso a la aplicación, mientras vuelve a cifrar el token de actualización con DPAPI y lo almacena en su propia caché  |
| C |  Si un token de actualización de la aplicación no está disponible, el complemento WAM de Azure AD usa el PRT para solicitar un token de acceso. Para proporcionar la prueba de posesión, el complemento WAM firma la solicitud que contiene el PRT con la clave de sesión. Azure AD valida la firma de la clave de sesión, para lo cual la compara con la clave de sesión insertada en el PRT, comprueba que el dispositivo sea válido y emite un token de acceso y un token de actualización para la aplicación. Además, Azure AD puede emitir un nuevo PRT (según el ciclo de actualización), todos ellos cifrados con la clave de sesión. |
| D | El complemento WAM solicita al complemento CloudAP que descifre los tokens, el cual, a su vez, solicita al TPM que los descifre con la clave de sesión, lo que da lugar a que el complemento WAM obtenga ambos tokens. A continuación, el complemento WAM proporciona solo el token de acceso a la aplicación, mientras vuelve a cifrar el token de actualización con DPAPI y lo almacena en su propia caché. El complemento WAM usará el token de actualización de aquí en adelante para esta aplicación. El complemento WAM también devuelve el nuevo PRT al complemento CloudAP, que lo valida con Azure AD antes de actualizarlo en su propia caché. El complemento CloudAP usará el nuevo PRT de aquí en adelante. |
| E | WAM proporciona el token de acceso recién emitido a WAM, que, a su vez, lo proporciona a la aplicación que realiza la llamada.|

### <a name="browser-sso-using-prt"></a>Inicio de sesión único del explorador mediante PRT

![Inicio de sesión único del explorador mediante PRT](./media/concept-primary-refresh-token/browser-sso-using-prt.png)

| Paso | Descripción |
| :---: | --- |
| Un | El usuario inicia sesión en Windows con sus credenciales para obtener un PRT. Una vez que el usuario abre el explorador, el explorador (o la extensión) carga las direcciones URL del Registro. |
| B | Cuando un usuario abre una dirección URL de inicio de sesión de Azure AD, el explorador o la extensión validan la dirección URL con las obtenidas del Registro. Si coinciden, el explorador invoca el host de cliente nativo para obtener un token. |
| C | El host de cliente nativo valida que las direcciones URL pertenezcan a los proveedores de identidades de Microsoft (cuenta Microsoft o Azure AD), extrae un nonce enviado desde la dirección URL y realiza una llamada al complemento CloudAP para obtener una cookie con PRT. |
| D | El complemento CloudAP crea la cookie con PRT, inicia la sesión con la clave de sesión enlazada al TPM y la devuelve al host de cliente nativo. Como la cookie está firmada con la clave de sesión, no se pueda alterar. |
| E | El host de cliente nativo devolverá esta cookie con PRT al explorador, que la incluirá como parte del encabezado de solicitud llamado x-ms-RefreshTokenCredential y solicitará tokens a Azure AD. |
| F | Azure AD valida la firma de la clave de sesión de la cookie con PRT, valida el nonce, comprueba que el dispositivo sea válido en el inquilino y emite un token de identidad para la página web y una cookie de sesión cifrada para el explorador. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo solucionar problemas relacionados con el PRT, consulte el artículo [Solución de problemas de dispositivos Windows 10 y Windows Server 2016 unidos a Azure Active Directory híbrido](troubleshoot-hybrid-join-windows-current.md).
