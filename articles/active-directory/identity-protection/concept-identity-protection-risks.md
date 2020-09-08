---
title: ¿Qué es el riesgo? Azure AD Identity Protection
description: Explicación del riesgo en Azure AD Identity Protection
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: conceptual
ms.date: 08/27/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 629173612f091319f6dec57b1cdfcfea41033bfc
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89047112"
---
# <a name="what-is-risk"></a>¿Qué es el riesgo?

Las detecciones de riesgo en Azure AD Identity Protection incluyen todas las acciones sospechosas identificadas, relacionadas con las cuentas de usuario en el directorio.

Identity Protection proporciona a las organizaciones el acceso a recursos eficaces, para ver y responder rápidamente a estas acciones sospechosas. 

![Información general sobre seguridad que muestra usuarios e inicios de sesión de riesgo](./media/concept-identity-protection-risks/identity-protection-security-overview.png)

## <a name="risk-types-and-detection"></a>Tipos de riesgo y detección

Hay dos tipos de riesgo, **Usuario** e **Inicio de sesión** y dos tipos de detección o cálculo **Tiempo real** y **Sin conexión**.

### <a name="user-risk"></a>Riesgo de usuario

Un riesgo de usuario representa la probabilidad de que una identidad o cuenta determinada esté en peligro. 

Estos riesgos se calculan sin conexión, usando orígenes de inteligencia sobre amenazas internas y externas de Microsoft, incluidos los investigadores de seguridad, los profesionales de la aplicación de la ley, los equipos de seguridad de Microsoft y otros orígenes de confianza.

| Detección de riesgos | Descripción |
| --- | --- |
| Credenciales con fugas | Este tipo de detección de riesgo indica que se han filtrado las credenciales válidas del usuario. Cuando los cibercriminales llegan a poner en peligro las contraseñas válidas de usuarios legítimos, es frecuente que las compartan. Normalmente lo hacen publicándolas en la Web oscura, los sitios de pegado, o bien mediante el intercambio o la venta de esas credenciales en el mercado negro. Cuando el servicio de credenciales filtradas de Microsoft adquiere las credenciales de usuario de la Web oscura, los sitios de pegado u otros orígenes, se comparan con las credenciales válidas actuales de los usuarios de Azure AD para encontrar coincidencias válidas. Para obtener más información sobre las credenciales filtradas, consulte [Preguntas frecuentes](#common-questions). |
| Inteligencia de Azure AD sobre amenazas | Este tipo de detección de riesgo indica una actividad de usuario poco común para el usuario en cuestión o coherente con patrones de ataque conocidos basados en orígenes de inteligencia sobre amenazas internas y externas de Microsoft. |

### <a name="sign-in-risk"></a>Riesgo de inicio de sesión

Un riesgo de inicio de sesión representa la probabilidad de que el propietario de la identidad no haya autorizado una solicitud de autenticación determinada. 

Estos riesgos se pueden calcular en tiempo real o sin conexión, usando orígenes de inteligencia sobre amenazas internas y externas de Microsoft, incluidos los investigadores de seguridad, los profesionales de la aplicación de la ley, los equipos de seguridad de Microsoft y otros orígenes de confianza.

| Detección de riesgos | Tipo de detección | Descripción |
| --- | --- | --- |
| Dirección IP anónima | Tiempo real | Este tipo de detección de riesgos indica inicios de sesión desde una dirección IP anónima (por ejemplo, el explorador Tor o redes VPN anónimas). Estas direcciones IP normalmente las usan actores que quieren ocultar su telemetría de inicio de sesión (dirección IP, ubicación, dispositivo, etc.) con fines potencialmente malintencionados. |
| Viaje atípico | Sin conexión | Este tipo de detección de riesgos identifica dos inicios de sesión procedentes de ubicaciones geográficamente distantes, donde al menos una de las ubicaciones puede también ser inusual para el usuario, según su comportamiento anterior. Entre otros factores, este algoritmo de aprendizaje automático tiene en cuenta el tiempo entre los dos inicios de sesión y el tiempo que habría necesitado el usuario para viajar de la primera ubicación a la segunda, lo que indica que otro usuario utiliza las mismas credenciales. <br><br> Este algoritmo omite "falsos positivos" obvios que contribuyen a una condición de viaje imposible, como las VPN y las ubicaciones que usan con regularidad otros usuarios de la organización. El sistema tiene un período de aprendizaje inicial de 14 días o 10 inicios de sesión, lo que ocurra primero, durante el cual aprende el comportamiento de inicio de sesión del nuevo usuario. |
| Dirección IP vinculada al malware | Sin conexión | Este tipo de detección de riesgos indica inicios de sesión desde direcciones IP infectadas con malware, que se sabe que se comunican activamente con un servidor bot. Esta detección se determina mediante la correlación de direcciones IP del dispositivo del usuario con direcciones IP que han estado en contacto con un servidor bot mientras este último estaba activo. |
| Propiedades de inicio de sesión desconocidas | Tiempo real | Este tipo de detección de riesgos tiene en cuenta el historial de inicio de sesión anterior (dirección IP, latitud/longitud y ASN) para determinar inicios de sesión anómalos. El sistema almacena información acerca de las ubicaciones anteriores que ha utilizado un usuario y considera estas ubicaciones "conocidas". La detección de riesgos se desencadena cuando el inicio de sesión se produce desde una ubicación que no está en la lista de ubicaciones conocidas. Los usuarios recién creados estarán en "modo de aprendizaje" durante un período de tiempo, en el que las detecciones de riesgo de las propiedades de inicio de sesión desconocidas estarán desactivadas mientras nuestros algoritmos aprenden el comportamiento del usuario. La duración del modo de aprendizaje es dinámica y depende de cuánto tiempo tarde el algoritmo en recopilar información suficiente sobre los patrones de inicio de sesión del usuario. La duración mínima es de cinco días. Un usuario puede volver al modo de aprendizaje tras un largo período de inactividad. El sistema también ignora los inicios de sesión desde dispositivos conocidos y ubicaciones geográficamente cercanas a una ubicación conocida. <br><br> También se ejecuta esta detección para una autenticación básica o para protocolos heredados. Dado que estos protocolos no tienen propiedades modernas como el identificador de cliente, hay una telemetría limitada para reducir los falsos positivos. Se recomienda que los clientes realicen la migración a la autenticación moderna. |
| Vulneración de identidad de usuario confirmada por el administrador | Sin conexión | Esta detección indica que un administrador ha seleccionado "Confirmar vulneración de la identidad del usuario" en la interfaz de usuario de Usuarios de riesgo o mediante riskyUsers API. Para ver qué administrador ha confirmado este usuario comprometido, compruebe el historial de riesgos del usuario (a través de la interfaz de usuario o la API). |
| Dirección IP malintencionada | Sin conexión | Esta detección indica el inicio de sesión desde una dirección IP malintencionada. Una dirección IP se considera malintencionada si se recibe una alta tasa de errores debidos a credenciales no válidas desde la dirección IP u otros orígenes de reputación de IP. |
| Reglas de manipulación sospechosa de la bandeja de entrada | Sin conexión | Esta detección se debe a [Microsoft Cloud App Security (MCAS)](/cloud-app-security/anomaly-detection-policy#suspicious-inbox-manipulation-rules). Esta detección genera un perfil del entorno y activa alertas cuando se establecen reglas sospechosas que eliminan o mueven mensajes o carpetas en la bandeja de entrada de un usuario. Esta detección puede indicar que la cuenta del usuario está en peligro, que los mensajes se están ocultando intencionadamente y que el buzón se está usando para distribuir correo no deseado o malware en su organización. |
| Difusión de contraseñas | Sin conexión | Un ataque de difusión de contraseñas es aquel por el que se ataca a varios nombres de usuario mediante contraseñas comunes, en un único ataque por la fuerza bruta, para obtener acceso no autorizado. Esta detección de riesgo se desencadena cuando se realiza un ataque de difusión de contraseñas. |
| Viaje imposible | Sin conexión | Esta detección se debe a [Microsoft Cloud App Security (MCAS)](/cloud-app-security/anomaly-detection-policy#impossible-travel). Esta detección identifica dos actividades de usuario (en una o varias sesiones) que se originan desde ubicaciones geográficamente distantes dentro de un período de tiempo menor que el tiempo que habría tardado el usuario para viajar de la primera ubicación a la segunda, lo que indica que otro usuario está usando las mismas credenciales. |

### <a name="other-risk-detections"></a>Otras detecciones de riesgo

| Detección de riesgos | Tipo de detección | Descripción |
| --- | --- | --- |
| Riesgo adicional detectado | En tiempo real o sin conexión | Esta detección indica que se descubrió una de las detecciones prémium anteriores. Dado que las detecciones premium solo son visibles para los clientes de Azure AD Premium P2, se denominan "Riesgo adicional detectado" para los clientes sin licencias de Azure AD Premium P2. |

## <a name="common-questions"></a>Preguntas frecuentes

### <a name="risk-levels"></a>Niveles de riesgo

Identity Protection clasifica el riesgo en tres niveles: bajo, medio y alto. 

Aunque Microsoft no proporciona detalles específicos sobre cómo se calcula el riesgo, diremos que cada nivel aporta una mayor seguridad de que el usuario o el inicio de sesión están en peligro. Por ejemplo, cosas como un caso de propiedades de inicio de sesión desconocidas para un usuario podría no ser tan amenazante como la filtración de credenciales para otro usuario.

### <a name="leaked-credentials"></a>Credenciales con fugas

#### <a name="where-does-microsoft-find-leaked-credentials"></a>¿Dónde busca Microsoft las credenciales filtradas?

Microsoft busca las credenciales filtradas en una gran variedad de lugares, entre otros:

- Los sitios de pegado públicos, como pastebin.com y paste.ca, en los que los infiltrados normalmente publican este material. Esta ubicación es la parada obligada para la mayoría de los infiltrados en su búsqueda de credenciales robadas.
- Organismos de autoridad judicial.
- Otros grupos de Microsoft que investigan la web oscura.

#### <a name="why-arent-i-seeing-any-leaked-credentials"></a>¿Por qué no veo ninguna credencial filtrada?

Las credenciales filtradas se procesan siempre que Microsoft encuentra un nuevo lote disponible públicamente. Debido a la naturaleza confidencial, las credenciales filtradas se eliminan poco después de su procesamiento. Solo se procesarán en el inquilino las nuevas credenciales filtradas que se encuentren después de habilitar la sincronización de hash de contraseñas (PHS). No se comprueban los pares de credenciales detectados anteriormente. 

#### <a name="i-havent-seen-any-leaked-credential-risk-events-for-quite-some-time"></a>No he encontrado ningún evento de riesgo de credenciales filtradas durante bastante tiempo.

Si no ha detectado ningún evento de riesgo de credenciales filtradas, se debe a los siguientes motivos:

- No tiene PHS habilitada para el inquilino.
- Microsoft no ha encontrado ningún par de credenciales filtradas que coincida con sus usuarios.

#### <a name="how-often-does-microsoft-process-new-credentials"></a>¿Con qué frecuencia Microsoft procesa nuevas credenciales?

Las credenciales se procesan inmediatamente después de que se han encontrado, normalmente varios lotes al día.

## <a name="next-steps"></a>Pasos siguientes

- [Directivas disponibles para mitigar los riesgos](concept-identity-protection-policies.md)
- [Información general sobre seguridad](concept-identity-protection-security-overview.md)
