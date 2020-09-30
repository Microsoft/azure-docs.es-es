---
title: Cifrado de datos en reposo del servicio Language Understanding
titleSuffix: Azure Cognitive Services
description: Microsoft ofrece claves de cifrado administradas por Microsoft y también le permite administrar las suscripciones de Cognitive Services con sus propias claves, llamadas claves administradas por el cliente (CMK). En este artículo, encontrará información sobre el cifrado de datos en Language Understanding (LUIS) y cómo habilitar y administrar CMK.
author: erindormier
manager: venkyv
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: egeaney
ms.openlocfilehash: ce6561652801d52e5600ddc63e573070281da3f2
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89078136"
---
# <a name="language-understanding-service-encryption-of-data-at-rest"></a>Cifrado de datos en reposo del servicio Language Understanding

El servicio Language Understanding cifra automáticamente los datos al guardarlos en la nube. El cifrado de Language Understanding protege los datos y le ayuda a cumplir los compromisos de cumplimiento y seguridad de la organización.

## <a name="about-cognitive-services-encryption"></a>Información sobre el cifrado de Cognitive Services

Los datos se cifran y descifran con el cifrado [AES de 256 bits](https://en.wikipedia.org/wiki/FIPS_140-2) compatible con [FIPS 140-2](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard). El cifrado y el descifrado son transparentes, lo que significa que el cifrado y el acceso se administran automáticamente. Dado que los datos están protegidos de forma predeterminada, no es necesario modificar el código ni las aplicaciones para utilizar el cifrado.

## <a name="about-encryption-key-management"></a>Información sobre la administración de claves de cifrado

De forma predeterminada, su suscripción usa claves de cifrado administradas por Microsoft. También puede administrar la suscripción con sus propias claves, que se denominan claves administradas por el cliente (CMK). Las CMK ofrecen mayor flexibilidad para crear, rotar, deshabilitar y revocar los controles de acceso. También permite auditar las claves de cifrado que se usan para proteger los datos.

## <a name="customer-managed-keys-with-azure-key-vault"></a>Claves administradas por el cliente con Azure Key Vault

También tiene una opción para administrar su suscripción con sus propias claves. Las claves administradas por el cliente (CMK), también conocidas como Bring Your Own Key (BYOK), ofrecen más flexibilidad para crear, girar, deshabilitar y revocar controles de acceso. También permite auditar las claves de cifrado que se usan para proteger los datos.

Debe usar Azure Key Vault para almacenar las claves administradas por el cliente. Puede crear sus propias claves y almacenarlas en un almacén de claves, o puede usar las API de Azure Key Vault para generarlas. El recurso de Cognitive Services y el almacén de claves deben estar en la misma región y en el mismo inquilino de Azure Active Directory (Azure AD), pero pueden estar en distintas suscripciones. Para más información sobre Azure Key Vault, vea [¿Qué es Azure Key Vault?](https://docs.microsoft.com/azure/key-vault/key-vault-overview).

### <a name="customer-managed-keys-for-language-understanding"></a>Claves administradas por el cliente para Language Understanding

Para solicitar la capacidad de usar claves administradas por el cliente, rellene y envíe el  [formulario de solicitud de claves administradas por el cliente del servicio LUIS](https://aka.ms/cogsvc-cmk). Tardará de tres a cinco días hábiles aproximadamente en recibir una respuesta sobre el estado de la solicitud. En función de la demanda, es posible que se coloque en una cola y se apruebe a medida que haya espacio disponible. Una vez aprobado el uso de CMK con LUIS, deberá crear un nuevo recurso de Language Understanding desde Azure Portal y seleccionar E0 como el plan de tarifa. La nueva SKU funcionará igual que la SKU F0 que ya está disponible, excepto CMK. Los usuarios no podrán realizar la actualización de F0 a la nueva SKU de E0.

![Imagen de la suscripción de LUIS](../media/cognitive-services-encryption/luis-subscription.png)

### <a name="limitations"></a>Limitaciones

Existen algunas limitaciones al usar el nivel E0 con aplicaciones existentes o creadas anteriormente:

* Se bloqueará la migración a un recurso E0. Los usuarios solo podrán migrar sus aplicaciones a los recursos F0. Después de migrar un recurso existente a F0, puede crear un nuevo recurso en el nivel E0. Obtenga [aquí](https://docs.microsoft.com/azure/cognitive-services/luis/luis-migration-authoring) más información sobre la migración.  
* Se bloqueará la migración de aplicaciones a un recurso E0 o desde él. Una solución alternativa a esta limitación consiste en exportar la aplicación existente e importarla como un recurso E0.
* No se admite la característica Bing Spell Check.
* El registro del tráfico de usuario final está deshabilitado si la aplicación es E0.
* La funcionalidad de preparación para la voz de Azure Bot Service no admite las aplicaciones del nivel de E0. Esta característica está disponible a través de Azure Bot Service, que no admite CMK.
* La funcionalidad de preparación para la voz del portal requiere Azure Blob Storage. Para más información, consulte [Traiga su propio almacenamiento](../Speech-Service/speech-encryption-of-data-at-rest.md#bring-your-own-storage-byos-for-customization-and-logging).

### <a name="enable-customer-managed-keys"></a>Habilitar claves administradas del cliente

Un nuevo recurso de Cognitive Services siempre se cifra mediante claves administradas por Microsoft. No es posible habilitar claves administradas por el cliente en el momento en que se crea el recurso. Las claves administradas por el cliente se almacenan en Azure Key Vault y el almacén de claves se debe aprovisionar con directivas de acceso que concedan permisos de clave a la identidad administrada que está asociada al recurso de Cognitive Services. La identidad administrada solo está disponible después de crear el recurso con el plan de tarifa para CMK.

Para aprender a usar claves administradas por el cliente con Azure Key Vault para el cifrado de Cognitive Services, consulte:

- [Configuración de claves administradas por el cliente con Key Vault para el cifrado de Cognitive Services mediante Azure Portal](../Encryption/cognitive-services-encryption-keys-portal.md)

Al habilitar las claves administradas por el cliente también se habilitará una identidad administrada asignada por el sistema, una característica de Azure AD. Una vez habilitada la identidad administrada asignada por el sistema, este recurso se registrará con Azure Active Directory. Tras su registro, se concederá a la identidad administrada acceso a la instancia de Key Vault seleccionada durante la configuración de la clave administrada por el cliente. Puede obtener más información acerca de las [identidades administradas](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

> [!IMPORTANT]
> Si deshabilita las identidades administradas asignadas por el sistema, se quitará el acceso al almacén de claves y los datos cifrados con las claves de cliente dejarán de estar disponibles. Las características dependientes de estos datos dejarán de funcionar.

> [!IMPORTANT]
> Las identidades administradas no admiten actualmente escenarios entre directorios. Al configurar las claves administradas por el cliente en Azure Portal, se asigna automáticamente una identidad administrada en segundo plano. Si posteriormente mueve la suscripción, el grupo de recursos o el recurso de un directorio de Azure AD a otro, la identidad administrada asociada al recurso no se transfiere al nuevo inquilino, por lo que es posible que las claves administradas por el cliente dejen de funcionar. Para más información, vea **Transferencia de una suscripción entre directorios de Azure AD** en [Preguntas frecuentes y problemas conocidos con identidades administradas para recursos de Azure](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/known-issues#transferring-a-subscription-between-azure-ad-directories).  

### <a name="store-customer-managed-keys-in-azure-key-vault"></a>Almacenamiento de claves administradas por el cliente en Azure Key Vault

Para habilitar las claves administradas por el cliente, debe usar una instancia de Azure Key Vault para almacenarlas. Debe habilitar las propiedades **Eliminación temporal** y **No purgar** en el almacén de claves.

Las claves RSA de tamaño 2048 son las únicas que admite el cifrado de Cognitive Services. Para más información acerca de las claves, consulte la sección **Claves en Key Vault** en [Información acerca de claves, secretos y certificados de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/about-keys-secrets-and-certificates#key-vault-keys).

### <a name="rotate-customer-managed-keys"></a>Rotación de claves administradas por el cliente

Las claves administradas por el cliente se pueden rotar en Azure Key Vault según las directivas de cumplimiento. Si se rota la clave, es preciso actualizar el recurso de Cognitive Services para que use el identificador URI de la clave nueva. Para aprender a actualizar el recurso para que use una nueva versión de la clave en Azure Portal, consulte la sección **Actualización de la versión de la clave** en [Configuración de claves administradas por el cliente para Cognitive Services mediante Azure Portal](../Encryption/cognitive-services-encryption-keys-portal.md).

La rotación de la clave no desencadena un nuevo cifrado de los datos del recurso. No es preciso que el usuario realice ninguna otra acción.

### <a name="revoke-access-to-customer-managed-keys"></a>Revocación del acceso a las claves administradas por el cliente

Para revocar el acceso a las claves administradas por el cliente, use PowerShell o la CLI de Azure. Para más información, consulte la referencia de [PowerShell para Azure Key Vault](https://docs.microsoft.com/powershell/module/az.keyvault//) o la referencia de la [CLI para Azure Key Vault](https://docs.microsoft.com/cli/azure/keyvault). La revocación del acceso bloquea de manera eficaz el acceso a todos los datos del recurso de Cognitive Services, ya que este servicio no puede acceder a la clave de cifrado.

## <a name="next-steps"></a>Pasos siguientes

* [Formulario de solicitud de claves administradas por el cliente del servicio LUIS](https://aka.ms/cogsvc-cmk)
* [Más información sobre Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview)
