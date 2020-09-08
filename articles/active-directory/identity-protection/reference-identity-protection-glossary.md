---
title: Glosario de Azure Active Directory Identity Protection
description: Glosario de Azure Active Directory Identity Protection
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: reference
ms.date: 10/18/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1a17f52e519a84c58c9ed5b75259cc0b96d0a175
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88949181"
---
# <a name="azure-active-directory-identity-protection-glossary"></a>Glosario de Azure Active Directory Identity Protection

### <a name="at-risk-user"></a>En riesgo (usuario)
Usuario con una o más detecciones de riesgo activas. 

### <a name="atypical-sign-in-location"></a>Ubicación de inicio de sesión inusual
Inicio de sesión desde una ubicación geográfica que no es la usual para un usuario específico, usuarios similares o el inquilino.

### <a name="azure-ad-identity-protection"></a>Azure AD Identity Protection
Módulo de seguridad de Azure Active Directory que proporciona una vista consolidada de las detecciones de riesgo y posibles puntos vulnerables que afectan a las identidades de una organización.

### <a name="conditional-access"></a>Acceso condicional
Directiva para proteger el acceso a los recursos. Las reglas de acceso condicional se almacenan en Azure Active Directory, que las evalúa antes de conceder acceso al recurso.  Algunas reglas de ejemplo son la restricción de acceso según la ubicación del usuario, el estado del dispositivo o el método de autenticación de usuario.

### <a name="credentials"></a>Credenciales
Información que incluye la identificación y prueba de identificación que se usa para obtener acceso a recursos locales y de red. Ejemplos de credenciales son nombres de usuario, contraseñas, tarjetas inteligentes y certificados.

### <a name="event"></a>Evento
Registro de una actividad en Azure Active Directory.

### <a name="false-positive-risk-detection"></a>Falso positivo (detección de riesgo)
Estado de detección de riesgo establecido manualmente por un usuario de Identity Protection, que indica que la detección de riesgo se investigó y marcó incorrectamente como detección de riesgo.

### <a name="identity"></a>Identidad
Persona o entidad que se debe comprobar mediante autenticación, basándose en criterios como contraseñas o certificados.

### <a name="identity-risk-detection"></a>Detección de riesgo de identidad
Evento de Azure AD marcado como anómalo por Identity Protection, que puede indicar que una identidad está en peligro.

### <a name="ignored-risk-detection"></a>Ignorada (detección de riesgo)
Estado de detección de riesgo establecido manualmente por un usuario de Identity Protection, que indica que la detección de riesgo se cierra sin aplicar una acción de corrección.

### <a name="impossible-travel-from-atypical-locations"></a>Viaje imposible desde ubicaciones inusuales
Detección de riesgo que se desencadena cuando se detectan dos inicios de sesión para el mismo usuario, en el que al menos uno de ellos es desde una ubicación de inicio de sesión inusual y en el que el tiempo entre inicios de sesión es menor que el tiempo mínimo que se tardaría en viajar físicamente entre estas ubicaciones.  

### <a name="investigation"></a>Investigación
Proceso de revisión de las actividades, los registros y otra información pertinente relacionada con una detección de riesgo para decidir si son necesarios pasos de mitigación o de corrección, y entender si se ha puesto en peligro la identidad y de qué manera, así como comprender cómo se utilizaba la identidad en peligro.

### <a name="leaked-credentials"></a>Credenciales con fugas
Detección de riesgo que se desencadena cuando se encuentran credenciales de usuario actuales (nombre de usuario y contraseña) publicadas en la web oscura por nuestros investigadores.

### <a name="mitigation"></a>Mitigación
Acción para limitar o eliminar la capacidad de un atacante de aprovechar una identidad o un dispositivo en peligro sin restaurar la identidad o el dispositivo a un estado seguro. Una mitigación no resuelve las anteriores detecciones de riesgo asociadas a la identidad o el dispositivo.

### <a name="multi-factor-authentication"></a>Multi-Factor Authentication
Método de autenticación que requiere dos o más métodos de autenticación, que pueden incluir algo que el usuario tiene, como un certificado; algo que el usuario sabe, como nombres de usuario, contraseñas o frases de contraseña; atributos físicos, como una huella digital; y atributos personales, como una firma personal.

### <a name="offline-detection"></a>Detección sin conexión
Detección de anomalías y evaluación del riesgo de un evento como el intento de inicio de sesión después del hecho, para un evento que ya ha ocurrido.

### <a name="policy-condition"></a>Condición de directiva
Parte de una directiva de seguridad que define las entidades (grupos, usuarios, aplicaciones, plataformas de dispositivos, estados de dispositivos, intervalos IP, tipos de cliente) incluidas en la directiva o excluidas de ella.

### <a name="policy-rule"></a>Regla de directiva
Parte de una directiva de seguridad que describe las circunstancias que pueden desencadenar la directiva y las acciones aplicadas cuando se desencadena la directiva.

### <a name="prevention"></a>Prevención
Acción para evitar daños en la organización debidos al uso inapropiado de una identidad o un dispositivo sospechoso o que se sabe que está en peligro. Una acción de prevención no protege la identidad o el dispositivo y no resuelve detecciones de riesgo anteriores.

### <a name="privileged-user"></a>Con privilegios (usuario)
Usuario que, en el momento de una detección de riesgo, tenía permisos de administrador permanentes o temporales para uno o más recursos de Azure Active Directory, como un administrador global, administrador de facturación, administrador de servicios, administrador de usuarios y administrador de contraseñas. 

### <a name="real-time"></a>Tiempo real
Ver Detección en tiempo real.

### <a name="real-time-detection"></a>Detección en tiempo real
Detección de anomalías y evaluación del riesgo de un evento como el intento de inicio de sesión antes de que el evento pueda continuar.

### <a name="remediated-risk-detection"></a>Corregida (detección de riesgo)
Estado de detección de riesgo establecido automáticamente por Identity Protection, que indica que la detección de riesgo se corrigió mediante la acción de corrección estándar para este tipo de detecciones de riesgo. Por ejemplo, cuando se restablece la contraseña de usuario, se corrigen automáticamente muchas detecciones de riesgo que indican que se había puesto en peligro la contraseña anterior.

### <a name="remediation"></a>Corrección
Acción para proteger una identidad o un dispositivo que antes fue sospechoso o que se sabe que estuvo en peligro. Una acción de corrección restaura la identidad o el dispositivo a un estado seguro y resuelve las anteriores detecciones de riesgo asociadas a la identidad o el dispositivo.

### <a name="resolved-risk-detection"></a>Resuelta (detección de riesgo)
Estado de detección de riesgo establecido manualmente por un usuario de Identity Protection, que indica que el usuario aplicó una acción de corrección adecuada fuera de Identity Protection y que se debe considerar la detección de riesgo cerrada.

### <a name="risk-detection-status"></a>Estado de detección de riesgo
Propiedad de una detección de riesgo, que indica si el evento está activo y, si está cerrado, el motivo para haberlo cerrarlo.

### <a name="risk-detection-type"></a>Tipo de detección de riesgo
Categoría de la detección de riesgo, que indica el tipo de anomalía que provocó el evento para que se considere de riesgo.

### <a name="risk-level-risk-detection"></a>Nivel de riesgo (detección de riesgo)
Indicación (Alto, Medio o Bajo) de la gravedad de la detección de riesgo para ayudar a los usuarios de Identity Protection a dar prioridad a las acciones que aplican a fin de reducir el riesgo en su organización. 

### <a name="risk-level-sign-in"></a>Nivel de riesgo (inicio de sesión)
Indicación (Alto, Medio o Bajo) de la probabilidad de que, para un inicio de sesión específico, otra persona esté intentando utilizar la identidad del usuario.

### <a name="risk-level-user-compromise"></a>Nivel de riesgo (compromiso de usuario)
Indicación (Alto, Medio o Bajo) de la probabilidad de que se haya puesto en peligro una identidad.

### <a name="risk-level-vulnerability"></a>Nivel de riesgo (vulnerabilidad)
Indicación (Alto, Medio o Bajo) de la gravedad del punto vulnerable para ayudar a los usuarios de Identity Protection a dar prioridad a las acciones que aplican a fin de reducir el riesgo en su organización.

### <a name="secure-identity"></a>Proteger (identidad)
Aplicar acciones de corrección, como un cambio de contraseña o un restablecimiento de la imagen inicial de una máquina para restaurar una identidad potencialmente en peligro a un estado seguro.

### <a name="security-policy"></a>Directiva de seguridad
Recopilación de reglas y condiciones de una directiva. Una directiva se puede aplicar a entidades como usuarios, grupos, aplicaciones, dispositivos, plataformas de dispositivos, estados de dispositivos, intervalos IP y tipos de clientes Auth2.0. Cuando una directiva está habilitada, se evalúa cada vez que una entidad incluida en la directiva emite un token para un recurso.

### <a name="sign-in-v"></a>Iniciar sesión
Autenticar una identidad en Azure Active Directory.

### <a name="sign-in-n"></a>Inicio de sesión
Proceso o acción de autenticar una identidad en Azure Active Directory y evento que captura esta operación.

### <a name="sign-in-from-anonymous-ip-address"></a>Inicio de sesión desde dirección IP anónima
Detección de riesgo que se desencadena después de un inicio de sesión correcto desde una dirección IP identificada como una dirección IP de proxy anónima.

### <a name="sign-in-from-infected-device"></a>Inicio de sesión desde dispositivo infectado
Detección de riesgo que se desencadena cuando un inicio de sesión se origina desde una dirección IP que se sabe que usan uno o más dispositivos en peligro que están intentando comunicarse de forma activa con un servidor bot.

### <a name="sign-in-from-ip-address-with-suspicious-activity"></a>Inicio de sesión desde dirección IP con actividad sospechosa
Detección de riesgo que se desencadena después de un inicio de sesión correcto desde una dirección IP con un gran número de intentos de inicios de sesión erróneos entre varias cuentas de usuario durante un corto período de tiempo.

### <a name="sign-in-from-unfamiliar-location"></a>Inicio de sesión desde ubicación desconocida
Detección de riesgo que se desencadena cuando un usuario inicia sesión correctamente desde una nueva ubicación (IP, latitud/longitud y aviso de envío por adelantado).

### <a name="sign-in-risk"></a>Riesgo de inicio de sesión
Ver Nivel de riesgo (inicio de sesión)

### <a name="sign-in-risk-policy"></a>Directiva de riesgo de inicio de sesión
Directiva de acceso condicional que evalúa el riesgo en un inicio de sesión específico y aplica mitigaciones de acuerdo con las reglas y condiciones predefinidas.

### <a name="user-compromise-risk"></a>Riesgo de compromiso de usuario
Ver Nivel de riesgo (compromiso de usuario).

### <a name="user-risk"></a>Riesgo de usuario
Ver Nivel de riesgo (compromiso de usuario).

### <a name="user-risk-policy"></a>Directiva de riesgo de usuario
Directiva de acceso condicional que tiene en cuenta el inicio de sesión y aplica mitigaciones de acuerdo con las reglas y condiciones predefinidas.

### <a name="users-flagged-for-risk"></a>Usuarios marcados con riesgo
Usuarios con detecciones de riesgo activas o corregidas

### <a name="vulnerability"></a>Punto vulnerable
Configuración o condición en Azure Active Directory, que hace que el directorio sea susceptible de recibir ataques o amenazas.

## <a name="see-also"></a>Consulte también

- [Azure Active Directory Identity Protection](./overview-identity-protection.md)