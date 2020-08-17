---
title: Información sobre claves, secretos y certificados de Azure Key Vault
description: Información general de los detalles para desarrolladores y la interfaz de REST de Azure Key Vault para claves, secretos y certificados.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.topic: overview
ms.date: 04/17/2020
ms.author: mbaldwin
ms.openlocfilehash: cb8a29c5d2eff46eecb2cf977bfb492f28731e68
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87043637"
---
# <a name="about-keys-secrets-and-certificates"></a>Información acerca de claves, secretos y certificados

Azure Key Vault permite a las aplicaciones y los usuarios de Microsoft Azure almacenar y usar varios tipos de datos de secretos y claves:

- Claves de cifrado: admite varios tipos de claves y algoritmos y habilita el uso de módulos de seguridad de hardware (HSM) para claves de alto valor. Para más información, consulte [Acerca de las claves](../keys/about-keys.md).
- Secretos: proporciona un almacenamiento seguro de secretos, como contraseñas y cadenas de conexión de base de datos. Para más información, consulte [Acerca de los secretos](../secrets/about-secrets.md).
- Certificates: admite certificados, que se basan en claves y secretos, y agrega una característica de renovación automática. Para más información, consulte [Acerca de los certificados](../certificates/about-certificates.md).
- Azure Storage: puede administrar automáticamente las claves de una cuenta de Azure Storage. Internamente, Key Vault puede enumerar (sincronizar) las claves con una cuenta de almacenamiento de Azure y volver a generar (rotar) las claves periódicamente. Para más información, consulte [Administración de claves de cuenta de almacenamiento con Key Vault](../secrets/overview-storage-keys.md).

Para más información sobre Key Vault, consulte [Acerca de Azure Key Vault](overview.md).

## <a name="data-types"></a>Tipos de datos

Consulte las especificaciones JOSE para tipos de datos relevantes para claves, cifrado y firma.  

-   **algorithm**: un algoritmo admitido para una operación de clave, por ejemplo, RSA1_5  
-   **ciphertext-value**: octetos de texto cifrado codificados con Base64URL  
-   **digest-value**: la salida de un algoritmo de hash codificada con Base64URL  
-   **key-type**: uno de los tipos de clave admitidos; por ejemplo, RSA (Rivest-Shamir-Adleman).  
-   **plaintext-value**: octetos de texto no cifrado codificados con Base64URL  
-   **signature-value**: la salida de un algoritmo de firma codificada con Base64URL  
-   **base64URL**: un valor binario codificado con Base64URL [RFC4648]  
-   **boolean**: true o false  
-   **Identity**: una identidad de Azure Active Directory (AAD).  
-   **IntDate**: un valor decimal JSON que representa el número de segundos desde 1970-01-01T0:0:0Z UTC hasta la fecha y hora UTC especificada. Consulte RFC3339 para más información acerca de la fecha y hora en general, y la hora UTC en particular.  

## <a name="objects-identifiers-and-versioning"></a>Objetos, identificadores y control de versiones

Se aplica el control de versiones de los objetos almacenados en Key Vault cuando se crea una instancia de un objeto. Cada versión tiene asignados un identificador único y una dirección URL. Cuando se crea un objeto por primera vez, se le asigna un identificador de versión único y se marca como la versión actual del objeto. La creación de una instancia con el mismo nombre de objeto proporciona al nuevo objeto un identificador de versión único, que hace que se convierta en la versión actual.  

Los objetos de Key Vault se pueden direccionar especificando una versión o omitiendo la versión de las operaciones en la versión actual del objeto. Por ejemplo, dada una clave con el nombre `MasterKey`, realizar operaciones sin especificar una versión hace que el sistema use la versión más reciente disponible. Realizar operaciones con el identificador específico de la versión hace que el sistema use esa versión específica del objeto.  

Los objetos se identifican de forma única en Key Vault mediante una dirección URL. No hay dos objetos en el sistema que tengan la misma dirección URL, independientemente de la ubicación geográfica. La dirección URL completa a un objeto se denomina identificador de objeto. La dirección URL consta de un prefijo que identifica la instancia de Key Vault, el tipo de objeto, el nombre de objeto proporcionado por el usuario y la versión de un objeto. El nombre del objeto es inmutable y no distingue entre mayúsculas y minúsculas. Los identificadores que no incluyen la versión del objeto se conocen como identificadores base.  

Para más información, consulte [Autenticación, solicitudes y respuestas](authentication-requests-and-responses.md)

Un identificador de objeto tiene el formato general siguiente:  

`https://{keyvault-name}.vault.azure.net/{object-type}/{object-name}/{object-version}`  

Donde:  

| Elemento | Descripción |  
|-|-|  
|`keyvault-name`|Nombre de un almacén de claves en el servicio Microsoft Azure Key Vault.<br /><br /> Los nombres de los almacenes de claves los selecciona el usuario y son globalmente únicos.<br /><br /> El nombre de una instancia de Key Vault debe ser una cadena con una longitud de 3 a 24 caracteres que solo contenga 0-9, a-z, A-Z y -.|  
|`object-type`|El tipo del objeto, "keys", "secrets" o "certificates".|  
|`object-name`|`object-name` es un nombre proporcionado por el usuario y debe ser único dentro de un almacén de claves. El nombre debe ser una cadena de entre 1 y 127 caracteres que solo contenga 0-9, a-z, A-Z y -.|  
|`object-version`|`object-version` es un identificador de cadena de 32 caracteres generada por el sistema que, opcionalmente, se utiliza para referirse a una versión única de un objeto.|  

## <a name="next-steps"></a>Pasos siguientes

- [Información acerca de las claves](../keys/about-keys.md)
- [Información acerca de los secretos](../secrets/about-secrets.md)
- [Información acerca de los certificados](../certificates/about-certificates.md)
- [Autenticación, solicitudes y respuestas](../general/authentication-requests-and-responses.md)
- [Guía del desarrollador de Key Vault](../general/developers-guide.md)
