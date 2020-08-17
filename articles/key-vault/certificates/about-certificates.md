---
title: 'Acerca de los certificados de Azure Key Vault: Azure Key Vault'
description: Información general de los certificados y la interfaz de REST de Azure Key Vault.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: overview
ms.date: 09/04/2019
ms.author: mbaldwin
ms.openlocfilehash: 74007f5d10d58cf9680d4531304098cabe9b6d8b
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87115673"
---
# <a name="about-azure-key-vault-certificates"></a>Acerca de los certificados de Azure Key Vault

La compatibilidad con certificados de Key Vault proporciona la administración de sus certificados x509 y los comportamientos siguientes:  

-   Permite que el propietario de un certificado cree un certificado a través de un proceso de creación de Key Vault o a través de la importación de un certificado existente. Incluye tanto certificados autofirmados como certificados generados por una entidad de certificación.
-   Permite que el propietario de un certificado de Key Vault implemente un almacenamiento seguro y la administración de certificados X509 sin interacción con material de clave privada.  
-   Permite que el propietario de un certificado cree una directiva que indique a Key Vault cómo administrar el ciclo de vida de un certificado.  
-   Permite que los propietarios de certificados proporcionen información de contacto para la notificación de eventos del ciclo de vida de caducidad y renovación de los certificados.  
-   Admite la renovación automática con emisores seleccionados: proveedores de certificados X509 y entidades de certificación asociados con Key Vault.

>[!Note]
>Los proveedores y entidades no asociados también pueden, pero en este caso no se admitirá la característica de renovación automática.

## <a name="composition-of-a-certificate"></a>Composición de un certificado

Cuando se crea un certificado de Key Vault, se crean también una clave direccionable y un secreto con el mismo nombre. La clave de Key Vault permite las operaciones de clave y el secreto de Key Vault permite la recuperación del valor del certificado como un secreto. Un certificado de Key Vault también contiene metadatos del certificado X.509 público.  

El identificador y la versión de los certificados es similar al de las claves y los secretos. Una versión específica de una clave direccionable y el secreto creados con la versión del certificado de Key Vault está disponible en la respuesta del certificado de Key Vault.
 
![Los certificados son objetos complejos](../media/azure-key-vault.png)

## <a name="exportable-or-non-exportable-key"></a>Clave exportable o no exportable

Cuando se crea un certificado de Key Vault, este se puede recuperar desde el secreto direccionable con la clave privada en formato PFX o PEM. La directiva utilizada para crear el certificado debe indicar que la clave es exportable. Si la directiva indica que no es exportable, la clave privada no forma parte del valor cuando se recupera como un secreto.  

La clave direccionable cobra más relevancia con los certificados de Key Vault no exportables. Las operaciones de la clave de Key Vault direccionable se asignan desde el campo *keyusage* de la directiva del certificado de Key Vault utilizada para crear el certificado de Key Vault.  

Se admiten dos tipos de clave con los certificados: *RSA* o *RSA HSM*. Los exportables solo se permiten con RSA, no se admiten en RSA HSM.  

## <a name="certificate-attributes-and-tags"></a>Etiquetas y atributos de certificado

Además de los metadatos del certificado, una clave direccionable y un secreto direccionable, un certificado de Key Vault también contiene atributos y etiquetas.  

### <a name="attributes"></a>Atributos

Los atributos del certificado se reflejan en los atributos de la clave y el secreto direccionables que se crean cuando se crea el certificado de Key Vault.  

Un certificado de Key Vault tiene los siguientes atributos:  

-   *enabled*: booleano, opcional, el valor predeterminado es **true**. Se puede especificar para indicar si se pueden recuperar los datos del certificado como secreto o es operativo como una clave. Se usa también junto con *nbf* y *exp* cuando se produce una operación entre *nbf* y *exp*, solo se permitirá si enabled se establece en true. Las operaciones fuera de la franja entre *nbf* y *exp* se deniegan automáticamente.  

Existen atributos de solo lectura adicionales que se incluyen en la respuesta:

-   *created*: tipo IntDate, indica cuándo se creó esta versión del certificado.  
-   *updated*: tipo IntDate, indica cuándo se modificó esta versión del certificado.  
-   *exp*: tipo IntDate, contiene el valor de la fecha de expiración del certificado X.509.  
-   *nbf*: tipo IntDate, contiene el valor de la fecha del certificado X.509.  

> [!Note] 
> Si un certificado de Key Vault expira, la clave y el secreto direccionables dejan de funcionar.  

### <a name="tags"></a>Etiquetas

 Diccionario especificado por el cliente de pares de clave y valor, similar a las etiquetas de las claves y los secretos.  

 > [!Note]
> Un llamador puede leer las etiquetas si tiene el permiso *list* o *get* para ese tipo de objeto (claves, secretos o certificados).

## <a name="certificate-policy"></a>Directiva de certificados

Una directiva de certificados contiene información sobre cómo crear y administrar el ciclo de vida de un certificado de Key Vault. Cuando se importa un certificado con clave privada en Key Vault, se crea una directiva predeterminada mediante la lectura del certificado X.509.  

Cuando se crea un certificado de Key Vault desde el principio, debe proporcionarse una directiva. La directiva especifica cómo crear esta versión de certificado de Key Vault o la próxima versión de certificado de Key Vault. Una vez que se ha establecido una directiva, no se requiere en sucesivas operaciones create para futuras versiones. Solo hay una instancia de una directiva para todas las versiones de un certificado de Key Vault.  

En un nivel general, una directiva de certificado contiene la siguiente información (sus decisiones se pueden encontrar [aquí](https://docs.microsoft.com/powershell/module/az.keyvault/set-azkeyvaultcertificatepolicy?view=azps-4.4.0):  

-   Propiedades del certificado X509: contiene el nombre de asunto, nombres alternativos de asunto y otras propiedades que se usan para crear una solicitud de certificado X509.  
-   Propiedades de clave: contiene el tipo de clave, la longitud y los campos de ReuseKeyOnRenewal. Estos campos indican a Key Vault cómo generar una clave.  
-   Propiedades del secreto: contiene las propiedades del secreto, como el tipo de contenido del secreto direccionable para generar el valor secreto, para recuperar el certificado como un secreto.  
-   Acciones de vigencia: contiene las acciones de vigencia del certificado de Key Vault. Cada acción de vigencia contiene:  

     - Desencadenador: se especifica como días antes de la expiración o como un porcentaje del intervalo de vigencia  

     - Acción: se especifica el tipo de acción: *emailContacts* o *autoRenew*  

-   Emisor: parámetros sobre el emisor del certificado que se utiliza para emitir certificados X.509.  
-   Atributos de directiva: contiene atributos asociados a la directiva  

### <a name="x509-to-key-vault-usage-mapping"></a>Correspondencia entre el uso de X.509 y Key Vault

La tabla siguiente representa la correspondencia entre la directiva de uso de claves X.509 y las operaciones de clave efectivas de una clave creada como parte de la creación de certificados de Key Vault.

|**Marcas de uso de claves X.509**|**Operaciones de clave de Key Vault**|**Comportamiento predeterminado**|
|----------|--------|--------|
|DataEncipherment|cifrar, descifrar| N/D |
|DecipherOnly|Descifrado| N/D  |
|DigitalSignature|firmar, verificar| Valor predeterminado de Key Vault sin una especificación de uso en el momento de creación del certificado | 
|EncipherOnly|encrypt| N/D |
|KeyCertSign|firmar, verificar|N/D|
|KeyEncipherment|encapsular clave, desencapsular clave| Valor predeterminado de Key Vault sin una especificación de uso en el momento de creación del certificado | 
|NonRepudiation|firmar, verificar| N/D |
|crlsign|firmar, verificar| N/D |

## <a name="certificate-issuer"></a>Emisor de certificados

Un objeto de certificado de Key Vault contiene una configuración que se usa para comunicarse con un proveedor de emisor de certificados seleccionado para solicitar certificados X.509.  

-   Key Vault está asociado con los proveedores de emisión de certificado siguientes para certificados TLS/SSL

|**Nombre del proveedor**|**Ubicaciones**|
|----------|--------|
|DigiCert|Compatible con todas las ubicaciones del servicio de Key Vault en la nube pública y Azure Government|
|GlobalSign|Compatible con todas las ubicaciones del servicio de Key Vault en la nube pública y Azure Government|

Para poder crear un emisor de certificados en un almacén de claves, deben realizarse correctamente los pasos de requisitos previos 1 y 2 siguientes.  

1. Incorporación de proveedores de entidad emisora (CA) de certificados  

    -   Un administrador de la organización debe incorporar su empresa (por ejemplo, Contoso) con al menos un proveedor de CA.  

2. El administrador crea las credenciales del solicitante para que Key Vault inscriba (y renueve) certificados TLS/SSL  

    -   Proporciona la configuración que se usará para crear un objeto de emisor del proveedor en el almacén de claves  

Para más información acerca de cómo crear objetos de emisor desde el portal de certificados, consulte el [blog de certificados de Key Vault](https://aka.ms/kvcertsblog)  

Key Vault permite la creación de varios objetos de emisor con diferente configuración del proveedor de emisor. Una vez que se crea un objeto de emisor, se puede hacer referencia a su nombre en una o varias directivas de certificado. Al hacer referencia al objeto de emisor se indica a Key Vault que utilice la configuración de acuerdo con lo especificado en el objeto de emisor al solicitar el certificado X.509 al proveedor de la entidad emisora de certificados durante la creación y la renovación de certificados.  

Los objetos de emisor se crean en el almacén y solo se pueden usar con certificados de Key Vault del mismo almacén.  

## <a name="certificate-contacts"></a>Contactos de certificados

Los contactos de certificados contienen información de contacto para enviar notificaciones desencadenadas por los eventos de vigencia del certificado. La información de los contactos es compartida por todos los certificados del almacén de claves. Se envía una notificación a todos los contactos especificados para un evento de cualquier certificado del almacén de claves.  

Si se establece la directiva de un certificado para la renovación automática, se envía una notificación en los siguientes eventos.  

- Antes de la renovación del certificado
- Tras la renovación del certificado, indicando si el certificado se renovó correctamente o si se produjo un error que exige la renovación manual del certificado.  

  Si se establece una directiva de certificado en renovación manual (solo correo electrónico), cuando llega el momento de renovar el certificado se envía una notificación.  

## <a name="certificate-access-control"></a>Control de acceso al certificado

 El control de acceso para los certificados lo administra Key Vault y lo proporciona la instancia de Key Vault que contiene dichos certificados. La directiva de control de acceso para los certificados es distinta de la directiva de control de acceso para las claves y los secretos en la misma instancia de Key Vault. Los usuarios pueden crear uno o varios almacenes para almacenar los certificados, para mantener la segmentación adecuada del escenario y la administración de los certificados.  

 Los siguientes permisos se pueden utilizar, en una base por entidad, en la entrada de control de acceso de secretos en un almacén de claves y reflejan fielmente las operaciones permitidas en un objeto de secreto:  

- Permisos para operaciones de administración de certificados
  - *get*: obtener la versión actual del certificado o cualquier versión de un certificado 
  - *list*: enumerar los certificados actuales o las versiones de un certificado  
  - *update*: crear un certificado
  - *create*: crear un certificado de Key Vault
  - *import*: importar material del certificado en un certificado de Key Vault
  - *delete*: eliminar un certificado, su directiva y todas sus versiones  
  - *recover*: recuperar un certificado eliminado
  - *backup*: copia de seguridad de un certificado de un almacén de claves
  - *restore*: restaurar una copia de seguridad de un certificado a un almacén de claves
  - *managecontacts*: administrar los contactos del certificado de Key Vault  
  - *manageissuers*: administrar emisores y entidades de certificados de Key Vault
  - *getissuers*: obtener emisores y entidades de un certificado
  - *listissuers*: enumerar emisores y entidades de un certificado  
  - *setissuers*: crear o actualizar emisores y entidades de un certificado de Key Vault  
  - *deleteissuers*: eliminar emisores y entidades de un certificado de Key Vault  
 
- Permisos para las operaciones con privilegios
  - *purge*: purgar (eliminar permanentemente) un certificado eliminado

Para más información, consulte las [operaciones con certificados en la referencia de la API REST de Key Vault](/rest/api/keyvault). Para obtener información sobre cómo establecer permisos, vea [Almacenes: crear o actualizar](/rest/api/keyvault/vaults/createorupdate) y [Almacenes: actualizar directiva de acceso](/rest/api/keyvault/vaults/updateaccesspolicy).

## <a name="next-steps"></a>Pasos siguientes

- [Acerca de Key Vault](../general/overview.md)
- [Información acerca de claves, secretos y certificados](../general/about-keys-secrets-certificates.md)
- [Información acerca de las claves](../keys/about-keys.md)
- [Información acerca de los secretos](../secrets/about-secrets.md)
- [Autenticación, solicitudes y respuestas](../general/authentication-requests-and-responses.md)
- [Guía del desarrollador de Key Vault](../general/developers-guide.md)
