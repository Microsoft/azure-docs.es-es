---
title: Creación y firma de una directiva de Azure Attestation
description: Explicación de cómo crear y firmar una directiva de atestación.
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: a3afb12ac831d87b03d0bb16d1b7ef553f1bb906
ms.sourcegitcommit: 3c66bfd9c36cd204c299ed43b67de0ec08a7b968
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "90006826"
---
# <a name="how-to-author-and-sign-an-attestation-policy"></a>Procedimiento para crear y firmar una directiva de atestación

La directiva de atestación es un archivo cargado en Microsoft Azure Attestation. Azure Attestation ofrece la flexibilidad para cargar una directiva en un formato específico de la atestación. Como alternativa, también se puede cargar una versión codificada de la directiva, en firma web JSON. El administrador de directivas es el responsable de escribir la directiva de atestación. En la mayoría de los escenarios de atestación, el usuario de confianza actúa como administrador de directivas. El cliente que realiza la llamada de atestación envía la evidencia de la misma, que el servicio analiza y convierte en notificaciones entrantes (conjunto de propiedades, valor). Después, el servicio procesa las notificaciones, en función de lo que se defina en la directiva, y devuelve el resultado calculado.

La directiva contiene reglas que determinan los criterios de autorización, las propiedades y el contenido del token de atestación. Un archivo de directiva de ejemplo tiene la siguiente apariencia:

```
version=1.0;
authorizationrules
{
   c:[type="secureBootEnables", issuer=="AttestationService"]=> permit()
};

issuancerules
{
  c:[type="secureBootEnables", issuer=="AttestationService"]=> issue(claim=c)
  c:[type="notSafeMode", issuer=="AttestationService"]=> issue(claim=c)
};
```
 
Un archivo de directiva tiene tres segmentos, como se ha indicado anteriormente:

- **version**:  el número de versión de la gramática que se sigue. 

    ```
    version=MajorVersion.MinorVersion   
    ```

    Actualmente, la única versión admitida es "1.0".

- **authorizationrules**: una colección de las reglas de notificación que se comprobarán en primer lugar, para determinar si Azure Attestation debe continuar con **issuancerules**. Las reglas de notificación se aplican en el orden en el que se definan.

- **issuancerules**: colección de las reglas de notificación que se evaluarán para agregar información adicional al resultado de la atestación tal y como se defina en la directiva. Las reglas de notificación se aplican en el orden en el que se definan y también son opcionales.

Consulte las [notificaciones y las reglas de notificación](claim-rule-grammar.md) para más información.
   
## <a name="drafting-the-policy-file"></a>Borrador del archivo de directiva

1. Cree un archivo.
1. Agregue la versión al archivo.
1. Agregue secciones para **authorizationrules** e **issuancerules**.

  ```
  version=1.0;
  authorizationrules
  {
  =>deny();
  };
  
  issuancerules
  {
  };
  ```

  Las reglas de autorización contienen la acción deny() sin ninguna condición, para asegurarse de que no se procese ninguna regla de emisión. Como alternativa, la regla de autorización también puede contener la acción permit() para permitir el procesamiento de las reglas de emisión.
  
4. Incorporación de reglas de notificación a las reglas de autorización

  ```
  version=1.0;
  authorizationrules
  {
  [type=="secureBootEnabled", value==true, issuer=="AttestationService"]=>permit();
  };
  
  issuancerules
  {
  };
  ```

  Si el conjunto de notificaciones entrantes contiene una notificación que coincida con el tipo, el valor y el emisor, la acción permit() le indicará al motor de directivas que procese las **issuancerules**.
  
5. Agregue reglas de notificación a **issuancerules**.

  ```
  version=1.0;
  authorizationrules
  {
  [type=="secureBootEnabled", value==true, issuer=="AttestationService"]=>permit();
  };
  
  issuancerules
  {
  => issue(type="SecurityLevelValue", value=100);
  };
  ```
  
  El conjunto de notificaciones salientes contendrá una notificación con:

  ```
  [type="SecurityLevelValue", value=100, valueType="Integer", issuer="AttestationPolicy"]
  ```

  Las directivas complejas se pueden diseñar de manera similar. Para más información, consulte los [ejemplos de directivas de atestación](policy-examples.md).
  
6. Guarde el archivo.

## <a name="creating-the-policy-file-in-json-web-signature-format"></a>Creación del archivo de directivas en formato de firma Web de JSON

Después de crear un archivo de directiva, para cargar una directiva en formato JWS, siga estos pasos.

1. Genere el JWS, RFC 7515 con la directiva (codificado en UTF-8) como la carga.
     - El identificador de carga de la directiva codificada en Base64Url debe ser "AttestationPolicy".
     
     JWT de ejemplo:
     ```
     Header: {"alg":"none"}
     Payload: {"AttestationPolicy":" Base64Url (policy)"}
     Signature: {}

     JWS format: eyJhbGciOiJub25lIn0.XXXXXXXXX.
     ```

2. (Opcional) Firme la directiva. Azure Attestation admite los siguientes algoritmos:
     - **Ninguna**: no firme la carga de la directiva.
     - **RS256**: algoritmo admitido para firmar la carga de la directiva.

3. Cargue el JWS y valide la directiva.
     - Si el archivo de directiva no tiene errores de sintaxis, el servicio lo acepta.
     - Si contiene errores de sintaxis, el servicio lo rechaza.

## <a name="signing-the-policy"></a>Firma de la directiva

A continuación se muestra un script de Python de ejemplo sobre cómo realizar la operación de firma de la directiva.

```python
from OpenSSL import crypto
import jwt
import getpass
       
def cert_to_b64(cert):
              cert_pem = crypto.dump_certificate(crypto.FILETYPE_PEM, cert)
              cert_pem_str = cert_pem.decode('utf-8')
              return ''.join(cert_pem_str.split('\n')[1:-2])
       
print("Provide the path to the PKCS12 file:")
pkcs12_path = str(input())
pkcs12_password = getpass.getpass("\nProvide the password for the PKCS12 file:\n")
pkcs12_bin = open(pkcs12_path, "rb").read()
pkcs12 = crypto.load_pkcs12(pkcs12_bin, pkcs12_password.encode('utf8'))
ca_chain = pkcs12.get_ca_certificates()
ca_chain_b64 = []
for chain_cert in ca_chain:
   ca_chain_b64.append(cert_to_b64(chain_cert))
   signing_cert_pkey = crypto.dump_privatekey(crypto.FILETYPE_PEM, pkcs12.get_privatekey())
signing_cert_b64 = cert_to_b64(pkcs12.get_certificate())
ca_chain_b64.insert(0, signing_cert_b64)

print("Provide the path to the policy text file:")
policy_path = str(input())
policy_text = open(policy_path, "r").read()
encoded = jwt.encode({'text': policy_text }, signing_cert_pkey, algorithm='RS256', headers={'x5c' : ca_chain_b64})
print("\nAttestation Policy JWS:")
print(encoded.decode('utf-8'))
```

## <a name="next-steps"></a>Pasos siguientes
- [Configuración de Azure Attestation con PowerShell](quickstart-powershell.md)
- [Atestación de un enclave SGX con ejemplos de código](https://docs.microsoft.com/samples/browse/?expanded=azure&terms=attestation)
