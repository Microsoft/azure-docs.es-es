---
title: Métodos de creación de certificados
description: Obtenga información sobre las diferentes opciones con las que podemos crear o importar un certificado de Key Vault en Azure Key Vault. Hay varias formas de crear un certificado de Key Vault.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: conceptual
ms.date: 01/07/2019
ms.author: mbaldwin
ms.openlocfilehash: 71d8961af372f927fab909073daa715766542f87
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88606670"
---
# <a name="certificate-creation-methods"></a>Métodos de creación de certificados

 Un certificado de Key Vault (KV) se puede crear o importar en un almacén de claves. Cuando se crea un certificado KV, la clave privada se crea en el almacén de claves y no se expone al propietario del certificado. Las siguientes son formas de crear un certificado en Key Vault:  

-   **Crear un certificado autofirmado:** así se crea un par de claves pública y privada para asociarlo a un certificado. El certificado se firmará con su propia clave.  

-    **Crear manualmente un certificado:** así se crea un par de claves pública y privada, y se genera una solicitud de firma de certificado X.509. La solicitud de firma puede estar firmada por la autoridad de registro o la entidad de certificación. Para completar el certificado KV en Key Vault, el certificado X.509 se puede combinar con el par de claves pendiente. Aunque este método requiere más pasos, proporciona mayor seguridad, ya que la clave privada se crea en Key Vault y se restringe. Esto se explica en el diagrama siguiente.  

![Creación de un certificado con la entidad emisora de certificados propia](../media/certificate-authority-1.png)  

Las siguientes descripciones se corresponden con los pasos indicados con letras verdes del diagrama anterior.

1. En el diagrama anterior, la aplicación crea un certificado que comienza con la creación interna de una clave en el almacén de claves.
2. Key Vault devuelve una solicitud de firma de certificado (CSR) a la aplicación.
3. La aplicación pasa la CSR a la entidad de certificación seleccionada.
4. La entidad de certificación elegida responde con un certificado X.509.
5. La aplicación termina de crear el certificado con una combinación del certificado X.509 de la entidad de certificación.

-   **Crear un certificado con un emisor conocido:** este método requiere una tarea de creación de un objeto de emisor solo una vez. Cuando el objeto de emisor se haya creado en el almacén de claves, se puede hacer referencia a su nombre en la directiva del certificado KV. Una solicitud para crear este tipo de certificado KV creará un par de claves en el almacén y se comunicará con el emisor con la información del objeto de emisor a la que se hace referencia para obtener un certificado X.509. El certificado X.509 se recupera del servicio emisor y se combina con el par de claves para terminar la creación del certificado KV.  

![Creación de un certificado con una entidad de certificación asociada a Key Vault](../media/certificate-authority-2.png)  

Las siguientes descripciones se corresponden con los pasos indicados con letras verdes del diagrama anterior.

1. En el diagrama anterior, la aplicación crea un certificado que comienza con la creación interna de una clave en el almacén de claves.
2. Key Vault envía una solicitud de certificado TLS/SSL a la entidad de certificación.
3. La aplicación sondea, en proceso de bucle y espera, que Key Vault termine el certificado. La creación del certificado se completa cuando Key Vault recibe la respuesta de la entidad de certificación con el certificado X.509.
4. La entidad de certificación responde a la solicitud de certificado TLS/SSL de Key Vault con un certificado TLS/SSL X.509.
5. La creación del certificado se completa con la combinación del certificado TLS/SSL X.509 de la entidad de certificación.

## <a name="asynchronous-process"></a>Proceso asincrónico
La creación de certificados KV es un proceso asincrónico. Esta operación crea una solicitud de certificado KV y devuelve un código de estado http de 202 (Aceptado). Puede realizar el seguimiento del estado de la solicitud por sondeo en el objeto pendiente que crea esta operación. En el encabezado LOCATION se devuelve el URI completo del objeto pendiente.  

Cuando se completa una solicitud para crear un certificado KV, el estado del objeto pendiente cambia de "inprogress" a "completed" y se crea una nueva versión del certificado KV, que se convierte en la versión actual.  

## <a name="first-creation"></a>Primera creación
 Cuando se crea un certificado KV por primera vez, también se crea una clave direccionable y un secreto con el mismo nombre del certificado. Si el nombre ya está en uso, se produce un error en la operación con código de estado http 409 (conflicto).
La clave direccionable y el secreto obtienen sus atributos de los del certificado KV. La clave direccionable y el secreto que se crean de esta manera se marcan como claves y secretos administrados, cuya vigencia se administra desde Key Vault. Las claves y los secretos administrados son de solo lectura. Nota: Si un certificado KV expira o está deshabilitado, la clave y el secreto correspondientes dejan de funcionar.  

 Si se trata de la primera operación de creación de certificados KV, se requiere una directiva.  También se puede suministrar una directiva con operaciones de creación sucesivas que reemplacen el recurso de la directiva. Si no se proporciona directiva, se utiliza el recurso de directiva del servicio para crear la siguiente versión del certificado KV. Tenga en cuenta que con una solicitud de creación de una versión siguiente en curso, el certificado KV actual y la clave direccionable y el secreto correspondientes permanecen sin cambios.  

## <a name="self-issued-certificate"></a>Certificado de emisión propia
 Para crear un certificado de emisión propia, establezca el nombre del emisor como "Self" en la directiva de certificado como se muestra en el siguiente fragmento de código de la directiva de certificados.  

```  
"issuer": {  
       "name": "Self"  
    }  

```  

 Si no se especifica el nombre del emisor, su nombre se establece como "Unknown". Cuando el emisor es "Unknown", el propietario del certificado tiene que obtener manualmente un certificado X.509 del emisor de su elección y combinar el certificado X.509 público con el objeto pendiente del certificado de Key Vault para terminar la creación del certificado.

```  
"issuer": {  
       "name": "Unknown"  
    }  

```  

## <a name="partnered-ca-providers"></a>Proveedores de entidades de certificación asociados
La creación de certificados puede completarse manualmente o con un emisor "Self". Key Vault también se asocia con determinados emisores para simplificar la creación de certificados. De estos emisores asociados se pueden solicitar para Key Vault los siguientes tipos de certificados.  

|Proveedor|Tipo de certificado|Configuración  
|--------------|----------------------|------------------|  
|DigiCert|Key Vault ofrece certificados SSL OV o EV con DigiCert| [Guía de integración](https://docs.microsoft.com/azure/key-vault/certificates/how-to-integrate-certificate-authority)
|GlobalSign|Key Vault ofrece certificados SSL OV o EV con GlobalSign| [Guía de integración](https://support.globalsign.com/digital-certificates/digital-certificate-installation/generating-and-importing-certificate-microsoft-azure-key-vault)

 Un emisor de certificados es una entidad que se representa en Azure Key Vault (KV) como un recurso CertificateIssuer. Se utiliza para proporcionar información sobre el origen de un certificado de KV; el nombre del emisor, el proveedor, las credenciales y otros detalles administrativos.

Tenga en cuenta que cuando se realiza un pedido al emisor, puede respetar o reemplazar las extensiones del certificado X.509 y el período de validez del certificado en función de su tipo.  

 Autorización: se requiere el permiso certificates/create.

## <a name="see-also"></a>Consulte también

 - [Supervisión y administración de la creación de certificados](create-certificate-scenarios.md)
