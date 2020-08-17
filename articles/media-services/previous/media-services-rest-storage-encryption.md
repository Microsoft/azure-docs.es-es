---
title: Cifrado del contenido con cifrado de almacenamiento mediante la API de REST de AMS
description: Aprenda a cifrar el contenido con cifrado de almacenamiento mediante las API de REST de AMS.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: a0a79f3d-76a1-4994-9202-59b91a2230e0
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 761a508543af79f3a242bfa2133e22a00b0ca689
ms.sourcegitcommit: 14bf4129a73de2b51a575c3a0a7a3b9c86387b2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87439605"
---
# <a name="encrypting-your-content-with-storage-encryption"></a>Cifrado del contenido con cifrado de almacenamiento 

> [!NOTE]
> Para completar este tutorial, deberá tener una cuenta de Azure. Para obtener más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/).   > No se van a agregar características o funcionalidades nuevas a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).
>

Este artículo proporciona información general del cifrado de almacenamiento de AMS y muestra cómo cargar el contenido cifrado del almacenamiento:

* Cree una clave de contenido.
* Cree un recurso. Establezca AssetCreationOption en StorageEncryption cuando cree el recurso.
  
     Los recursos cifrados se asocian con claves de contenido.
* Vincule la clave de contenido con el recurso.  
* Establezca los parámetros relacionados con el cifrado en las entidades AssetFile.

## <a name="considerations"></a>Consideraciones 

Si desea entregar un recurso de almacenamiento cifrado, debe configurar la directiva de entrega de recursos. Antes de poder transmitir el recurso, el servidor de streaming quita el cifrado de almacenamiento y transmite el contenido usando la directiva de entrega especificada. Para obtener más información, consulte [Configuración de directivas de entrega de recursos](media-services-rest-configure-asset-delivery-policy.md).

Al obtener acceso a las entidades de Media Services, debe establecer los campos de encabezado específicos y los valores en las solicitudes HTTP. Para obtener más información, consulte [Configuración del desarrollo de la API de REST de Media Services](media-services-rest-how-to-use.md). 

### <a name="storage-side-encryption"></a>Cifrado del lado de almacenamiento

|Opción de cifrado|Descripción|Media Services v2|Media Services v3|
|---|---|---|---|
|Cifrado de almacenamiento en Media Services|Cifrado de AES-256, clave administrada por Media Services|Admitido<sup>(1)</sup>|No admitido<sup>(2)</sup>|
|[Storage Service Encryption para datos en reposo](../../storage/common/storage-service-encryption.md)|Cifrado en el lado del servidor que ofrece Azure Storage, clave administrada por Azure o por el cliente|Compatible|Compatible|
|[Cifrado en el lado de cliente de almacenamiento](../../storage/common/storage-client-side-encryption.md)|Cifrado en el lado de cliente que ofrece Azure Storage, clave administrada por el cliente en Key Vault|No compatible|No compatible|

<sup>1</sup> Aunque Media Services admite el control del contenido de una forma clara o sin ninguna forma de cifrado, hacerlo así no es recomendable.

<sup>2</sup> En Media Services v3, el cifrado de almacenamiento (cifrado con AES-256) solo es compatible con versiones anteriores si los recursos se crearon con Media Services v2. Esto significa que la versión v3 funciona con los recursos cifrados de almacenamiento ya existentes pero no permitirá la creación de otros nuevos.

## <a name="connect-to-media-services"></a>Conexión con Media Services

Para obtener más información sobre cómo conectarse a la API de Azure Media Services, consulte [Acceso a la API de Azure Media Services con la autenticación de Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

## <a name="storage-encryption-overview"></a>Información general del cifrado de almacenamiento
El cifrado de almacenamiento de AMS aplica el cifrado de modo **AES-CTR** a todo el archivo.  El modo AES-CTR es un cifrado de bloques que puede cifrar los datos de longitud arbitraria sin necesidad de relleno. Funciona mediante el cifrado de un bloque de contador con el algoritmo de AES y, a continuación, aplicando XOR al resultado de AES con los datos para el cifrado o el descifrado.  El bloque de contador utilizado se crea copiando el valor de InitializationVector en bytes de 0 a 7 del valor de contador y los bytes de 8 y 15 del valor de contador se establecen en cero. Del bloque de contador de 16 bytes, los bytes de 8 a 15 (es decir, los bytes menos significativos) se utilizan como un entero sin firmar de 64 bits simple que se incrementa en uno por cada bloque siguiente de datos procesados y se mantiene en el orden de bytes de la red. Si este entero alcanza el valor máximo (0xFFFFFFFFFFFFFFFF), al incrementarlo se restablece el contador de bloque a cero (bytes de 8 a 15) sin que afecte a los otros 64 bits del contador (es decir, bytes de 0 a 7).   Para mantener la seguridad del cifrado del modo AES-CTR, el valor de InitializationVector para un identificador de clave determinado para cada clave de contenido debe ser único para cada archivo y los archivos deben tener una longitud inferior a 2^64 bloques.  Este valor único es para asegurarse de que un valor de contador nunca se reutiliza con una clave determinada. Para obtener más información sobre el modo CTR, consulte [esta página de la wiki](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#CTR) (el artículo de la wiki utiliza el término "Nonce" en lugar de "InitializationVector").

Utilice **cifrado de almacenamiento** para cifrar el contenido no cifrado de manera local mediante el cifrado AES de 256 bits y luego cargarlo a Azure Storage donde se almacena cifrado en reposo. Los recursos protegidos con el cifrado de almacenamiento se descifran automáticamente y se colocan en un sistema de archivos cifrados antes de la codificación y, opcionalmente, se vuelven a cifrar antes de volver a cargarlos como un nuevo recurso de salida. El caso de uso principal para el cifrado de almacenamiento es cuando desea proteger los archivos multimedia de entrada de alta calidad con un sólido cifrado en reposo en disco.

Para entregar a un recurso cifrado de almacenamiento, debe configurar la directiva de entrega del recurso para que Media Services sepa cómo desea entregar el contenido. Antes de poder transmitir el activo, el servidor de streaming quita el cifrado de almacenamiento y transmite el contenido usando la directiva de entrega especificada (por ejemplo, AES, cifrado común o sin cifrado).

## <a name="create-contentkeys-used-for-encryption"></a>Creación de ContentKeys para el cifrado
Los recursos cifrados están asociados con claves de cifrado de almacenamiento. Cree la clave de contenido que se usará para el cifrado antes de crear los archivos de recursos. En esta sección se describe cómo crear una clave de contenido.

A continuación, se muestran los pasos generales para generar claves de contenido que asocia a los recursos que desee cifrar. 

1. Para el cifrado de almacenamiento, genere de forma aleatoria una clave AES de 32 bytes. 
   
    Esta es la clave de contenido de su recurso, lo que significa que todos los archivos asociados a dicho recurso deben usar la misma clave de contenido durante el descifrado. 
2. Llame a los métodos [GetProtectionKeyId](/rest/api/media/operations/rest-api-functions#getprotectionkeyid) y [GetProtectionKey](/rest/api/media/operations/rest-api-functions#getprotectionkey) para obtener el certificado X.509 correcto que debe usarse para cifrar la clave de contenido.
3. Cifre la clave de contenido con la clave pública del certificado X.509. 
   
   El SDK de Media Services para .NET SDK usa RSA con OAEP al realizar el cifrado.  Puede ver un ejemplo de .NET en la [función EncryptSymmetricKeyData](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.FileEncryption/EncryptionUtils.cs).
4. Cree un valor de suma de comprobación calculado con el identificador de clave y la clave de contenido. El siguiente ejemplo de .NET calcula la suma de comprobación con la parte del GUID del identificador de clave y la clave de contenido sin cifrar.

    ```csharp
    public static string CalculateChecksum(byte[] contentKey, Guid keyId)
    {
        const int ChecksumLength = 8;
        const int KeyIdLength = 16;

        byte[] encryptedKeyId = null;

        // Checksum is computed by AES-ECB encrypting the KID
        // with the content key.
        using (AesCryptoServiceProvider rijndael = new AesCryptoServiceProvider())
        {
            rijndael.Mode = CipherMode.ECB;
            rijndael.Key = contentKey;
            rijndael.Padding = PaddingMode.None;

            ICryptoTransform encryptor = rijndael.CreateEncryptor();
            encryptedKeyId = new byte[KeyIdLength];
            encryptor.TransformBlock(keyId.ToByteArray(), 0, KeyIdLength, encryptedKeyId, 0);
        }

        byte[] retVal = new byte[ChecksumLength];
        Array.Copy(encryptedKeyId, retVal, ChecksumLength);

        return Convert.ToBase64String(retVal);
    }
    ```

5. Cree la clave de contenido con los valores **EncryptedContentKey** (convertida en cadena codificada en base 64), **ProtectionKeyId**, **ProtectionKeyType**, **ContentKeyType** y **Checksum** que recibió en los pasos anteriores.

    Para el cifrado de almacenamiento, se deben incluir las siguientes propiedades en el cuerpo de la solicitud.

    Propiedad del cuerpo de la solicitud    | Descripción
    ---|---
    Identificador | El identificador de ContentKey se genera mediante el siguiente formato: "nb:kid:UUID:\<NEW GUID>".
    ContentKeyType | El tipo de clave de contenido es un entero que define la clave. En el formato de cifrado de almacenamiento, el valor es 1.
    EncryptedContentKey | Creamos un valor de clave de contenido que es un valor de 256 bits (32 bytes). La clave se cifra con el certificado X.509 de cifrado de almacenamiento que recuperamos de Microsoft Azure Media Services mediante la ejecución de una solicitud HTTP GET para los métodos de GetProtectionKeyId y GetProtectionKey. A modo de ejemplo, vea el siguiente código. NET: el método **EncryptSymmetricKeyData** definido [aquí](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.FileEncryption/EncryptionUtils.cs).
    ProtectionKeyId | Es el identificador de la clave de protección para el certificado X.509 de cifrado de almacenamiento que se usó para cifrar la clave de contenido.
    ProtectionKeyType | Es el tipo de cifrado para la clave de protección que se usó para cifrar la clave de contenido. Este valor es StorageEncryption(1) en nuestro ejemplo.
    Suma de comprobación |Suma de comprobación calculada de MD5 para la clave de contenido. Se calcula mediante el cifrado del identificador de contenido con la clave de contenido. El código de ejemplo muestra cómo calcular la suma de comprobación.


### <a name="retrieve-the-protectionkeyid"></a>Recuperación de ProtectionKeyId
En el ejemplo siguiente se muestra cómo recuperar ProtectionKeyId, una huella digital de certificado, para el certificado que debe usar al cifrar la clave de contenido. Realice este paso para asegurarse de que ya tiene el certificado apropiado en el equipo.

Solicitud:

```console
GET https://media.windows.net/api/GetProtectionKeyId?contentKeyType=0 HTTP/1.1
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: media.windows.net
```

Respuesta:

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 139
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Server: Microsoft-IIS/8.5
request-id: 2b6aa7a4-3a09-4b08-b581-26b55667f817
x-ms-request-id: 2b6aa7a4-3a09-4b08-b581-26b55667f817
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 04 Feb 2015 02:42:52 GMT

{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Edm.String","value":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C"}
```

### <a name="retrieve-the-protectionkey-for-the-protectionkeyid"></a>Recuperación de ProtectionKey para ProtectionKeyId
En el ejemplo siguiente se muestra cómo recuperar el certificado X.509 mediante la entidad ProtectionKeyId que recibió en el paso anterior.

Solicitud:

```console
GET https://media.windows.net/api/GetProtectionKey?ProtectionKeyId='7D9BB04D9D0A4A24800CADBFEF232689E048F69C' HTTP/1.1
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 78d1247a-58d7-40e5-96cc-70ff0dfa7382
Host: media.windows.net
```

Respuesta:

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 1227
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Server: Microsoft-IIS/8.5
x-ms-client-request-id: 78d1247a-58d7-40e5-96cc-70ff0dfa7382
request-id: 1523e8f3-8ed2-40fe-8a9a-5d81eb572cc8
x-ms-request-id: 1523e8f3-8ed2-40fe-8a9a-5d81eb572cc8
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Thu, 05 Feb 2015 07:52:30 GMT

{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Edm.String",
"value":"MIIDSTCCAjGgAwIBAgIQqf92wku/HLJGCbMAU8GEnDANBgkqhkiG9w0BAQQFADAuMSwwKgYDVQQDEyN3YW1zYmx1cmVnMDAxZW5jcnlwdGFsbHNlY3JldHMtY2VydDAeFw0xMjA1MjkwNzAwMDBaFw0zMjA1MjkwNzAwMDBaMC4xLDAqBgNVBAMTI3dhbXNibHVyZWcwMDFlbmNyeXB0YWxsc2VjcmV0cy1jZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAzR0SEbXefvUjb9wCUfkEiKtGQ5Gc328qFPrhMjSo+YHe0AVviZ9YaxPPb0m1AaaRV4dqWpST2+JtDhLOmGpWmmA60tbATJDdmRzKi2eYAyhhE76MgJgL3myCQLP42jDusWXWSMabui3/tMDQs+zfi1sJ4Ch/lm5EvksYsu6o8sCv29VRwxfDLJPBy2NlbV4GbWz5Qxp2tAmHoROnfaRhwp6WIbquk69tEtu2U50CpPN2goLAqx2PpXAqA+prxCZYGTHqfmFJEKtZHhizVBTFPGS3ncfnQC9QIEwFbPw6E5PO5yNaB68radWsp5uvDg33G1i8IT39GstMW6zaaG7cNQIDAQABo2MwYTBfBgNVHQEEWDBWgBCOGT2hPhsvQioZimw8M+jOoTAwLjEsMCoGA1UEAxMjd2Ftc2JsdXJlZzAwMWVuY3J5cHRhbGxzZWNyZXRzLWNlcnSCEKn/dsJLvxyyRgmzAFPBhJwwDQYJKoZIhvcNAQEEBQADggEBABcrQPma2ekNS3Wc5wGXL/aHyQaQRwFGymnUJ+VR8jVUZaC/U/f6lR98eTlwycjVwRL7D15BfClGEHw66QdHejaViJCjbEIJJ3p2c9fzBKhjLhzB3VVNiLIaH6RSI1bMPd2eddSCqhDIn3VBN605GcYXMzhYp+YA6g9+YMNeS1b+LxX3fqixMQIxSHOLFZ1G/H2xfNawv0VikH3djNui3EKT1w/8aRkUv/AAV0b3rYkP/jA1I0CPn0XFk7STYoiJ3gJoKq9EMXhit+Iwfz0sMkfhWG12/XO+TAWqsK1ZxEjuC9OzrY7pFnNxs4Mu4S8iinehduSpY+9mDd3dHynNwT4="}
```

### <a name="create-the-content-key"></a>Creación de la clave de contenido
Después de recuperar el certificado X.509 y usar su clave pública para cifrar la clave de contenido, cree una entidad **ContentKey** y establezca sus valores de propiedad según corresponda.

Uno de los valores que debe establecer al crear la clave de contenido es el tipo. Cuando se usa el cifrado de almacenamiento, el valor debe establecerse en "1". 

En el ejemplo siguiente se muestra cómo crear una entidad **ContentKey** con una entidad **ContentKeyType** establecida para el cifrado de almacenamiento ("1") y la entidad **ProtectionKeyType** establecida en "0" para indicar que el identificador de clave de protección sea la huella digital del certificado X.509.  

Solicitud

```console
POST https://media.windows.net/api/ContentKeys HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: media.windows.net
{
"Name":"ContentKey",
"ProtectionKeyId":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C", 
"ContentKeyType":"1", 
"ProtectionKeyType":"0",
"EncryptedContentKey":"your encrypted content key",
"Checksum":"calculated checksum"
}
```

Respuesta:

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 777
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://media.windows.net/api/ContentKeys('nb%3Akid%3AUUID%3A9c8ea9c6-52bd-4232-8a43-8e43d8564a99')
Server: Microsoft-IIS/8.5
request-id: 76e85e0f-5cf1-44cb-b689-b3455888682c
x-ms-request-id: 76e85e0f-5cf1-44cb-b689-b3455888682c
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 04 Feb 2015 02:37:46 GMT

{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#ContentKeys/@Element",
"Id":"nb:kid:UUID:9c8ea9c6-52bd-4232-8a43-8e43d8564a99","Created":"2015-02-04T02:37:46.9684379Z",
"LastModified":"2015-02-04T02:37:46.9684379Z",
"ContentKeyType":1,
"EncryptedContentKey":"your encrypted content key",
"Name":"ContentKey",
"ProtectionKeyId":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C",
"ProtectionKeyType":0,
"Checksum":"calculated checksum"}
```

## <a name="create-an-asset"></a>Creación de un recurso
En el ejemplo siguiente se muestra cómo crear un recurso.

**Solicitud HTTP**

```console
POST https://media.windows.net/api/Assets HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: media.windows.net

{"Name":"BigBuckBunny" "Options":1}
```

**Respuesta HTTP**

Si se realiza correctamente, se devuelve la respuesta siguiente:

```console
HTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 452
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Assets('nb%3Acid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')
Server: Microsoft-IIS/8.5
x-ms-client-request-id: c59de965-bc89-4295-9a57-75d897e5221e
request-id: e98be122-ae09-473a-8072-0ccd234a0657
x-ms-request-id: e98be122-ae09-473a-8072-0ccd234a0657
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Sun, 18 Jan 2015 22:06:40 GMT
{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets/@Element",
   "Id":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "State":0,
   "Created":"2015-01-18T22:06:40.6010903Z",
   "LastModified":"2015-01-18T22:06:40.6010903Z",
   "AlternateId":null,
   "Name":"BigBuckBunny.mp4",
   "Options":1,
   "Uri":"https://storagetestaccount001.blob.core.windows.net/asset-9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "StorageAccountName":"storagetestaccount001"
}
```

## <a name="associate-the-contentkey-with-an-asset"></a>Asociación de la entidad ContentKey a un recurso
Después de crear la entidad ContentKey, asóciela a su recurso mediante la operación $links, tal como se muestra en el ejemplo siguiente:

Solicitud:

```console
POST https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3Afbd7ce05-1087-401b-aaae-29f16383c801')/$links/ContentKeys HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Content-Type: application/json
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: media.windows.net

{"uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeys('nb%3Akid%3AUUID%3A01e6ea36-2285-4562-91f1-82c45736047c')"}
```

Respuesta:

```console
HTTP/1.1 204 No Content 
```

## <a name="create-an-assetfile"></a>Creación de AssetFile
La entidad [AssetFile](/rest/api/media/operations/assetfile) representa un archivo de audio o vídeo que se almacena en un contenedor de blobs. Un archivo de recursos siempre está asociado a un recurso y un recurso puede contener uno o varios archivos de recursos. La tarea de Media Services produce un error si un objeto de archivo de recursos no está asociado a un archivo digital de un contenedor de blobs.

La instancia de **AssetFile** y el archivo multimedia real son dos objetos distintos. La instancia de AssetFile contiene metadatos sobre el archivo multimedia, mientras que el archivo multimedia contiene el contenido multimedia real.

Después de cargar el archivo multimedia digital en un contenedor de blobs, usará la solicitud HTTP **MERGE** para actualizar la entidad AssetFile con información sobre el archivo multimedia (no se muestra en este artículo). 

**Solicitud HTTP**

```console
POST https://media.windows.net/api/Files HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: media.windows.net
Content-Length: 164

{  
   "IsEncrypted":"true",
   "EncryptionScheme" : "StorageEncryption", 
   "EncryptionVersion" : "1.0",       
   "EncryptionKeyId" : "nb:kid:UUID:32e6efaf-5fba-4538-b115-9d1cefe43510",
   "InitializationVector" : "397304628502661816</d:InitializationVector",
   "Options":0,
   "IsPrimary":"false",
   "MimeType":"video/mp4",
   "Name":"BigBuckBunny.mp4",
   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
}
```

**Respuesta HTTP**

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 535
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5')
Server: Microsoft-IIS/8.5
request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
x-ms-request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 00:34:07 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Files/@Element",
   "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
   "Name":"BigBuckBunny.mp4",
   "ContentFileSize":"0",
   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "EncryptionVersion": "1.0",
   "EncryptionScheme": "StorageEncryption",
   "IsEncrypted":true,
   "EncryptionKeyId":"nb:kid:UUID:32e6efaf-5fba-4538-b115-9d1cefe43510",
   "InitializationVector":"397304628502661816</d:InitializationVector",
   "IsPrimary":false,
   "LastModified":"2015-01-19T00:34:08.1934137Z",
   "Created":"2015-01-19T00:34:08.1934137Z",
   "MimeType":"video/mp4",
   "ContentChecksum":null
}
```
