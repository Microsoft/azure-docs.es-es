---
title: Conjuntos de notificaciones de Azure Attestation
description: Los conjuntos de notificaciones de Azure Attestation.
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: 555dccbd3c2dfe61bac5891514deface6f8a877d
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89237065"
---
# <a name="claim-sets"></a>Conjuntos de notificaciones

Las notificaciones generadas en el proceso de atestación de enclaves mediante Microsoft Azure Attestation se pueden dividir en las siguientes categorías:

- **Notificaciones entrantes**: notificaciones generadas por Microsoft Azure Attestation después de analizar la evidencia de atestación.

- **Notificaciones salientes**: notificaciones creadas como salida por Azure Attestation. Contiene todas las notificaciones que deben acabar en el token de atestación.

- **Notificaciones de propiedad**: notificaciones creadas como salida por Azure Attestation. Contiene todas las notificaciones que representan las propiedades del token de atestación, como la codificación del informe, la duración de la validez del informe, etc.

A continuación se incluyen las notificaciones definidas por la RFC de JWT que usa Azure Attestation en el objeto de respuesta:

- **Notificación "iss" (emisor)** : La notificación "iss" (emisor) identifica la entidad de seguridad que ha emitido el JWT. Por lo general, el procesamiento de esta notificación es específico de la aplicación. El valor "iss" es una cadena que distingue mayúsculas de minúsculas y que contiene un valor StringOrURI.
- **Notificación "iat" (emitido a las)** : esta notificación señala en qué momento se emitió el JWT. Se puede usar para determinar la antigüedad del JWT. Su valor debe ser un número que contenga un valor NumericDate.
- **Notificación "exp" (fecha de expiración)** : La notificación "exp" (fecha de expiración) identifica la hora de expiración en la que o después de la que el token JWT no debe ser aceptado para su procesamiento. El procesamiento de esta notificación requiere que la fecha y hora actual sea anterior a la fecha y hora de expiración indicada en ella.

  Nota: se suma un margen de 5 minutos a la hora del problema (iat) para tener en cuenta el sesgo del reloj.
- **Notificación "nbf" (no antes de)** : esta notificación identifica la hora antes de la cual no se aceptará el token JWT para su procesamiento. El procesamiento de esta notificación requiere que la fecha y hora actual sea posterior o igual a la fecha y hora indicada en ella.
  Nota: se suma un margen de 5 minutos a la hora del problema (iat) para tener en cuenta el sesgo del reloj.

## <a name="claims-issued-by-azure-attestation-in-sgx-enclaves"></a>Notificaciones emitidas por Azure Attestation en enclaves SGX

### <a name="incoming-claims"></a>Notificaciones entrantes 

- **$is-debuggable**: valor booleano que indica si el enclave tiene habilitada la depuración o no
- **$sgx-mrsigner**: valor hexadecimal codificado del campo "mrsigner" de la oferta
- **$sgx-mrenclave**: valor hexadecimal codificado del campo "mrenclave" de la oferta
- **$product-id**
- **$svn**: número de versión de seguridad codificado en la oferta 
- **$tee**: tipo de enclave 

### <a name="outgoing-claims"></a>Notificaciones salientes

- **is-debuggable**: valor booleano que indica si el enclave tiene habilitada la depuración o no
- **sgx-mrsigner**: valor hexadecimal codificado del campo "mrsigner" de la oferta
- **sgx-mrenclave**: valor hexadecimal codificado del campo "mrenclave" de la oferta
- **product-id**
- **svn**: número de versión de seguridad codificado en la oferta 
- **tee**: tipo de enclave 
- **maa-ehd**:  versión codificada en Base64Url de los datos contenidos en el enclave especificados en la solicitud de atestación 
- **aas-ehd**:  versión codificada en Base64Url de los datos contenidos en el enclave especificados en la solicitud de atestación 

## <a name="claims-issued-by-azure-attestation-in-vbs-enclaves"></a>Notificaciones emitidas por Azure Attestation en enclaves VBS

### <a name="incoming-claims-can-also-be-used-as-outgoing-claims"></a>Notificaciones entrantes (también se pueden usar como notificaciones salientes)

- **aikValidated**:  valor booleano que indica si se ha validado o no el certificado de la clave de identidad de la atestación.
- **aikPubHash**:  cadena que contiene el hash SHA256 en base64 (clave pública de identidad de atestación en formato DER).
- **tpmVersion**:   valor entero que contiene la versión principal del Módulo de plataforma segura (TPM).
- **secureBootEnabled**: valor booleano que indica si está habilitado el arranque seguro.
- **iommuEnabled**:  valor booleano que indica si está habilitada la unidad de administración de memoria de entrada y salida (IOMMU).
- **bootDebuggingDisabled**: valor booleano que indica si está deshabilitada la depuración de arranque.
- **notSafeMode**:  valor booleano que indica si Windows no se está ejecutando en modo seguro.
- **notWinPE**:  valor booleano que indica si Windows no se está ejecutando en modo WinPE.
- **vbsEnabled**:  valor booleano que indica si VBS está habilitado.
- **vbsReportPresent**:  valor booleano que indica si está disponible el informe de enclaves VBS.
- **enclaveAuthorId**:  valor de cadena que contiene el valor codificado en Base64Url del identificador de autor del enclave; es decir, el identificador de autor del módulo principal del enclave.
- **enclaveImageId**:  valor de cadena que contiene el valor codificado en Base64Url del identificador de imagen del enclave, es decir, el identificador de imagen del módulo principal del enclave.
- **enclaveOwnerId**:  valor de cadena que contiene el valor codificado en Base64Url del identificador de propietario del enclave.
- **enclaveFamilyId**:  valor de cadena que contiene el valor codificado en Base64Url del identificador de familia del enclave; es decir, el identificador de familia del módulo principal del enclave.
- **enclaveSvn**:  valor entero que contiene el número de versión de seguridad del módulo principal del enclave.
- **enclavePlatformSvn**:  valor entero que contiene el número de versión de seguridad de la plataforma que hospeda el enclave.
- **enclaveFlags**:  valor entero que contiene las marcas que describen la directiva de tiempo de ejecución del enclave.
  
### <a name="outgoing-claims"></a>Notificaciones salientes

- **policy_hash**:  valor de cadena que contiene el hash SHA256 del texto de la directiva calculado por BASE64URL(SHA256(BASE64URL(UTF8(texto de directiva)))).
- **policy_signer**:  valor de cadena que contiene un JWK con la clave pública o la cadena de certificados presente en el encabezado de la directiva firmada.
- **ver (versión)** :  valor de cadena que contiene la versión del informe. Actualmente 1.0.
- **Notificación cnf (confirmación)** :  esta notificación se usa para identificar la clave de la prueba de posesión. Tal y como se define en RFC 7800, contiene la parte pública de la clave del enclave atestado que se representa como un objeto de clave web JSON (JWK) (RFC 7517).
- **rp_data (datos de usuario de confianza)** :  datos de usuario de confianza, si los hay, especificados en la solicitud, que utiliza este usuario como nonce para garantizar que el informe está actualizado.
- **Notificación "jti" (Id. de JWT)** : La notificación "jti" (Id. de JWT) proporciona un identificador único para el JWT. El valor del identificador se asigna de manera que se garantiza que la probabilidad de que el mismo valor se asigne accidentalmente a un objeto de datos diferente sea insignificante.

### <a name="property-claims"></a>Notificaciones de propiedad

- **report_validity_in_minutes**: notificación de entero que indica cuánto tiempo es válido el token.
  - **Valor predeterminado (tiempo)** : un día, expresado en minutos.
  - **Valor máximo (tiempo)** : un año, expresado en minutos.
- **omit_x5c**: notificación booleana que indica si Azure Attestation debe omitir el certificado usado para proporcionar la autenticidad de la prueba de servicio. Si es True, se agrega x5t al token de atestación. Si es False (valor predeterminado), se agregar x5c al token de atestación.

## <a name="next-steps"></a>Pasos siguientes
- [Creación y firma de una directiva de atestación](author-sign-policy.md)
- [Configuración de Azure Attestation con PowerShell](quickstart-powershell.md)
