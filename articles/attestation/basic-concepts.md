---
title: Conceptos básicos de Azure Attestation
description: Conceptos básicos de Azure Attestation.
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: a4ab8372e23e3621f7d73f8dbc38957c809acc9c
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89237049"
---
# <a name="basic-concepts"></a>Conceptos básicos

A continuación se muestran algunos conceptos básicos relacionados con Microsoft Azure Attestation.

## <a name="json-web-token-jwt"></a>JSON Web Token (JWT)

[JSON Web Token](https://jwt.io/) (JWT) es un método de [RFC7519](https://tools.ietf.org/html/rfc7519) estándar abierto para transmitir información de forma segura entre entidades como un objeto de notación de objetos JavaScript (JSON). Esta información se puede comprobar y es de confianza porque está firmada digitalmente. Los JWT se pueden firmar mediante un secreto o un par de claves pública y privada.

## <a name="json-web-key-jwk"></a>Clave de web JSON (JWK)

La [clave web de JSON](https://tools.ietf.org/html/rfc7517) (JWK) es una estructura de datos JSON que representa una clave criptográfica. Esta especificación también define una estructura de datos Set JSON de JWK que representa un conjunto de JWK.

## <a name="attestation-provider"></a>Proveedor de atestación

El proveedor de atestación pertenece al proveedor de recursos de Azure denominado Microsoft.Attestation. El proveedor de recursos es un punto de conexión de servicio que proporciona el contrato REST de Azure Attestation y se implementa mediante [Azure Resource Manager](../azure-resource-manager/management/overview.md). Cada proveedor de atestación respeta una directiva específica y reconocible. 

Los proveedores de atestación se crean con una directiva predeterminada para cada tipo de TEE (tenga en cuenta que el enclave de VBS no tiene ninguna directiva predeterminada). Consulte los [ejemplos de directivas de atestación](policy-examples.md) para más información sobre la directiva predeterminada de SGX.

### <a name="regional-default-provider"></a>Proveedor predeterminado regional

Azure Attestation proporciona un proveedor predeterminado en cada región. Los clientes pueden elegir entre usar el proveedor predeterminado para la atestación o crear sus propios proveedores con directivas personalizadas. Los proveedores predeterminados son accesibles para cualquier usuario de Azure AD y la directiva asociada a un proveedor predeterminado no se puede modificar.

| Region | URI de atestación | 
|--|--|
| Sur de Reino Unido | https://shareduks.uks.attest.azure.net | 
| Este de EE. UU. 2 | https://sharedeus2.eus2.attest.azure.net | 
| Centro de EE. UU. | https://sharedcus.cus.attest.azure.net | 
| Este de EE. UU.| https://sharedeus.eus.attest.azure.net | 
| Centro de Canadá | https://sharedcac.cac.attest.azure.net | 

## <a name="attestation-request"></a>Solicitud de atestación

La solicitud de atestación es un objeto JSON serializado que la aplicación cliente envía al proveedor de atestación. El objeto de solicitud para el enclave de SGX tiene dos propiedades: 
- "Quote": el valor de la propiedad "Quote" es una cadena que contiene una representación codificada en Base64URL de la oferta de atestación.
- "EnclaveHeldData": el valor de la propiedad "EnclaveHeldData" es una cadena que contiene una representación codificada en Base64URL de los datos contenidos en el enclave.

Azure Attestation validará el valor de "Quote" proporcionado en TEE y, a continuación, garantizará que el hash SHA256 de los datos contenidos en el enclave proporcionado se expresa en los primeros 32 bytes del campo reportData de la oferta. 

## <a name="attestation-policy"></a>Directiva de atestación

La directiva de atestación se usa para procesar la evidencia de atestación. Los clientes pueden configurarla. En el núcleo de Azure Attestation se encuentra un motor de directivas que procesa las notificaciones que constituyen la evidencia. Las directivas se usan para determinar si Azure Attestation emitirá o no un token de atestación en función de la evidencia y, por tanto, aprobará o no el atestador. En consecuencia, si no se superan todas las directivas, no se emitirá ningún token JWT.

Si la directiva de TEE predeterminada en el proveedor de atestación no satisface las necesidades, los clientes podrán crear directivas personalizadas en cualquiera de las regiones que Azure Attestation admita. La administración de directivas es una característica clave que se proporciona a los clientes mediante Azure Attestation. Las directivas serán específicas del TEE y se pueden usar para identificar enclaves, agregar notificaciones al token de salida o para modificar notificaciones en un token de salida. 

Consulte [ejemplos de una directiva de atestación](policy-examples.md) para obtener ejemplos y contenido predeterminados para las directivas.

## <a name="benefits-of-policy-signing"></a>Ventajas de la firma de directivas

En última instancia, una directiva de atestación es lo que determina si Azure Attestation emitirá el token de atestación. La directiva también determina las notificaciones que se van a generar en el token de atestación. Por lo tanto, es de la máxima importancia que la directiva que el servicio evalúe sea de hecho la que haya escrito el administrador y ninguna entidad externa la haya manipulado ni modificado. 

El modelo de confianza define el modelo de autorización del proveedor de atestación para definir y actualizar la directiva.  Se admiten dos modelos: uno basado en la autorización de Azure AD y otro basado en la posesión de claves criptográficas administradas por el cliente (conocidas como "modelo aislado").  El modelo aislado habilitará Azure Attestation para garantizar que la directiva que el cliente envíe no se manipule.

En el modelo aislado, el administrador crea un proveedor de atestación que especifica un conjunto de certificados X.509 de firma de confianza en un archivo. A continuación, el administrador puede agregar una directiva firmada al proveedor de atestación. Al procesar la solicitud de atestación, Azure Attestation validará la firma de la directiva mediante la clave pública representada por el parámetro "jwk" o "x5c" en el encabezado.  Azure Attestation también comprobará si la clave pública del encabezado de la solicitud está en la lista de certificados de firma de confianza asociados al proveedor de atestación. De esta manera, el usuario de confianza (Azure Attestation) puede confiar en una directiva firmada con los certificados X.509 que conozca. 

Consulte los [ejemplos de certificado de firma de directivas](policy-signer-examples.md).

## <a name="attestation-token"></a>Token de atestación

La respuesta de Azure Attestation será una cadena JSON cuyo valor contenga JWT. Azure Attestation empaquetará las notificaciones y generará un JWT firmado. La operación de firma se realiza mediante un certificado autofirmado con un nombre de sujeto que coincide con el elemento AttestUri del proveedor de atestación.

La API de obtención de metadatos de OpenID devuelve una respuesta de configuración de OpenID según lo especificado por el protocolo de detección de [OpenID Connect](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig). La API recupera los metadatos sobre los certificados de firma que Azure Attestation use.

Ejemplo de JWT generado para un enclave de SGX:

```
{
  “alg”: “RS256”,
  “jku”: “https://tradewinds.us.attest.azure.net/certs”,
  “kid”: “f1lIjBlb6jUHEUp1/Nh6BNUHc6vwiUyMKKhReZeEpGc=”,
  “typ”: “JWT”
}.{
  “maa-ehd”: <input enclave held data>,
  “exp”: 1568187398,
  “iat”: 1568158598,
  “is-debuggable”: false,
  “iss”: “https://tradewinds.us.attest.azure.net”,
  “nbf”: 1568158598,
  “product-id”: 4639,
  “sgx-mrenclave”: “”,
  “sgx-mrsigner”: “”,
  “svn”: 0,
  “tee”: “sgx”
}.[Signature]
```
Las notificaciones como "exp", "iat", "iss" o "nbf" se definen mediante el [RFC de JWT](https://tools.ietf.org/html/rfc7517) y el resto las genera Azure Attestation. Para más información, consulte las [notificaciones emitidas por Azure Attestation](claim-sets.md).

## <a name="next-steps"></a>Pasos siguientes

- [Creación y firma de una directiva de atestación](author-sign-policy.md)
- [Configuración de Azure Attestation con PowerShell](quickstart-powershell.md)
