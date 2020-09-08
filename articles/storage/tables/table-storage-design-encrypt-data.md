---
title: Cifrado de datos de tabla de Azure Storage | Microsoft Docs
description: Aprenda sobre el cifrado de datos de tabla en Azure Storage. La biblioteca cliente de Azure Storage para .NET permite cifrar entidades de cadena para operaciones de inserción y reemplazo.
services: storage
author: tamram
ms.author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/11/2018
ms.subservice: tables
ms.openlocfilehash: b921be718bfeb5eb95d4a802fb4d2a8cdd0946c1
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88236784"
---
# <a name="encrypt-table-data"></a>Cifrado de datos de tabla
La biblioteca de clientes de Azure Storage para .NET admite el cifrado de propiedades de entidades de cadena en operaciones de insertar y reemplazar. Las cadenas cifradas se almacenan en el servicio como propiedades binarias y se convierten de nuevo en cadenas después del descifrado.    

Para las tablas, además de la directiva de cifrado, los usuarios deben especificar las propiedades que se van a cifrar. Para ello, pueden especificar un atributo EncryptProperty (para las entidades POCO que se derivan de TableEntity) o una resolución de cifrado en las opciones de solicitud. Una resolución de cifrado es un delegado que toma una clave de partición, una clave de fila y un nombre de propiedad y devuelve un valor booleano que indica si se debe cifrar dicha propiedad. Durante el cifrado, la biblioteca de cliente usa esta información para decidir si se debe cifrar una propiedad mientras se escribe en la conexión. El delegado también proporciona la posibilidad de lógica con respecto a la forma de cifrar las propiedades. (Por ejemplo, si el valor es X, hay que cifrar la propiedad A; en caso contrario, hay que cifrar las propiedades A y B). No es necesario proporcionar esta información para leer o consultar entidades.

## <a name="merge-support"></a>Compatibilidad con combinación

Las operaciones de combinación no se admiten actualmente. Puesto que un subconjunto de propiedades puede haberse cifrado previamente con una clave distinta, si simplemente se combinan las nuevas propiedades y se actualizan los metadatos, se produce una pérdida de datos. Para realizar una combinación es necesario realizar llamadas de servicio adicionales para leer la entidad existente desde el servicio. También puede usar una nueva clave por propiedad. Ninguno de estos procedimientos es adecuado por motivos de rendimiento.     

Para información acerca del cifrado de datos de tablas, consulte [Cifrado del lado cliente y Azure Key Vault para Microsoft Azure Storage](../common/storage-client-side-encryption.md).  

## <a name="next-steps"></a>Pasos siguientes

- [Patrones de diseño de tablas](table-storage-design-patterns.md)
- [Modelado de relaciones](table-storage-design-modeling.md)
- [Modelado de relaciones](table-storage-design-modeling.md)
- [Diseño para la modificación de datos](table-storage-design-for-modification.md)
