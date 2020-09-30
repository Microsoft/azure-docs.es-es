---
title: Cifrado de datos en reposo de Traductor
titleSuffix: Azure Cognitive Services
description: Microsoft le permite administrar las suscripciones de Cognitive Services con sus propias claves, llamadas claves administradas por el cliente (CMK). En este artículo se trata el cifrado de datos en reposo de Traductor y cómo habilitar y administrar las CMK.
author: erindormier
manager: venkyv
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: egeaney
ms.openlocfilehash: ce7ff6ae134835de23a0d2670e8b4f44783654f8
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89079207"
---
# <a name="translator-encryption-of-data-at-rest"></a>Cifrado de datos en reposo de Traductor

Traductor cifra automáticamente los datos, que se cargan para crear modelos de traducción personalizados, cuando se guardan en la nube, lo que ayuda a cumplir los objetivos de seguridad y cumplimiento de la organización.

## <a name="about-cognitive-services-encryption"></a>Información sobre el cifrado de Cognitive Services

Los datos se cifran y descifran con el cifrado [AES de 256 bits](https://en.wikipedia.org/wiki/FIPS_140-2) compatible con [FIPS 140-2](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard). El cifrado y el descifrado son transparentes, lo que significa que el cifrado y el acceso se administran automáticamente. Dado que los datos están protegidos de forma predeterminada, no es necesario modificar el código ni las aplicaciones para utilizar el cifrado.

## <a name="about-encryption-key-management"></a>Información sobre la administración de claves de cifrado

De forma predeterminada, su suscripción usa claves de cifrado administradas por Microsoft. Si usa un plan de tarifa que admite claves administradas por el cliente, puede ver la configuración de cifrado del recurso en la sección **Cifrado** de [Azure Portal](https://portal.azure.com), tal como se muestra en la siguiente imagen.

![Visualización de la configuración de cifrado](../media/cognitive-services-encryption/encryptionblade.png)

En el caso de las suscripciones que solo admiten claves de cifrado administradas por Microsoft, no tendrá una sección **Cifrado**.

## <a name="customer-managed-keys-with-azure-key-vault"></a>Claves administradas por el cliente con Azure Key Vault

De forma predeterminada, su suscripción usa claves de cifrado administradas por Microsoft. También puede administrar la suscripción con sus propias claves, que se denominan claves administradas por el cliente (CMK). Las CMK ofrecen mayor flexibilidad para crear, rotar, deshabilitar y revocar los controles de acceso. También permite auditar las claves de cifrado que se usan para proteger los datos. Si la opción CMK está configurada en su suscripción, dispondrá de un cifrado doble, lo que constituye un segundo nivel de protección, y al mismo tiempo podrá controlar la clave de cifrado mediante Azure Key Vault.

> [!IMPORTANT]
> Las claves administradas por el cliente están disponibles en todos los planes de tarifa del servicio Traductor. Para solicitar la capacidad de usar claves administradas por el cliente, rellene y envíe el [formulario de solicitud de claves administradas por el cliente de Traductor](https://aka.ms/cogsvc-cmk). Tardará entre 3 y 5 días laborables en conocer el estado de la solicitud. En función de la demanda, es posible que se coloque en una cola y se apruebe a medida que haya espacio disponible. Una vez que se apruebe el uso de CMK con el servicio Traductor, necesitará crear un nuevo recurso de Traductor. Una vez creado el recurso de Traductor, puede usar Azure Key Vault para configurar la identidad administrada.

Siga estos pasos para habilitar las claves administradas por el cliente para Traductor:

1. Cree su nuevo recurso Traductor regional o Cognitive Services regional. Esto no funcionará con un recurso global.
2. Habilite la identidad administrada en Azure Portal y agregue la información de las claves administradas por el cliente.
3. Cree un área de trabajo en Traductor personalizado y asocie esta información de suscripción.

### <a name="enable-customer-managed-keys"></a>Habilitar claves administradas del cliente

Debe usar Azure Key Vault para almacenar las claves administradas por el cliente. Puede crear sus propias claves y almacenarlas en un almacén de claves, o puede usar las API de Azure Key Vault para generarlas. El recurso de Cognitive Services y el almacén de claves deben estar en la misma región y en el mismo inquilino de Azure Active Directory (Azure AD), pero pueden estar en distintas suscripciones. Para más información sobre Azure Key Vault, vea [¿Qué es Azure Key Vault?](https://docs.microsoft.com/azure/key-vault/key-vault-overview).

Los recursos nuevos de Cognitive Services siempre se cifran mediante claves administradas por Microsoft. No es posible habilitar claves administradas por el cliente en el momento en que se crea el recurso. Las claves administradas por el cliente se almacenan en Azure Key Vault y el almacén de claves se debe aprovisionar con directivas de acceso que concedan permisos de clave a la identidad administrada que está asociada al recurso de Cognitive Services. La identidad administrada está disponible en cuanto se crea el recurso.

Para aprender a usar claves administradas por el cliente con Azure Key Vault para el cifrado de Cognitive Services, consulte:

- [Configuración de claves administradas por el cliente con Key Vault para el cifrado de Cognitive Services mediante Azure Portal](../Encryption/cognitive-services-encryption-keys-portal.md)

Al habilitar las claves administradas por el cliente también se habilitará una identidad administrada asignada por el sistema, una característica de Azure AD. Una vez habilitada la identidad administrada asignada por el sistema, este recurso se registrará con Azure Active Directory. Tras su registro, se concederá a la identidad administrada acceso a la instancia de Key Vault seleccionada durante la configuración de la clave administrada por el cliente. Puede obtener más información acerca de las [identidades administradas](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

> [!IMPORTANT]
> Si deshabilita las identidades administradas asignadas por el sistema, se quitará el acceso al almacén de claves y los datos cifrados con las claves de cliente dejarán de estar disponibles. Las características dependientes de estos datos dejarán de funcionar. Se anulará la implementación de todos los modelos que se hayan implementado. Todos los datos cargados se eliminarán de Traductor personalizado. Si se vuelven a habilitar las identidades administradas, el modelo no se reimplementará automáticamente.

> [!IMPORTANT]
> Las identidades administradas no admiten actualmente escenarios entre directorios. Al configurar las claves administradas por el cliente en Azure Portal, se asigna automáticamente una identidad administrada en segundo plano. Si posteriormente mueve la suscripción, el grupo de recursos o el recurso de un directorio de Azure AD a otro, la identidad administrada asociada al recurso no se transfiere al nuevo inquilino, por lo que es posible que las claves administradas por el cliente dejen de funcionar. Para más información, vea **Transferencia de una suscripción entre directorios de Azure AD** en [Preguntas frecuentes y problemas conocidos con identidades administradas para recursos de Azure](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/known-issues#transferring-a-subscription-between-azure-ad-directories).  

### <a name="store-customer-managed-keys-in-azure-key-vault"></a>Almacenamiento de claves administradas por el cliente en Azure Key Vault

Para habilitar las claves administradas por el cliente, debe usar una instancia de Azure Key Vault para almacenarlas. Debe habilitar las propiedades **Eliminación temporal** y **No purgar** en el almacén de claves.

Las claves RSA de tamaño 2048 son las únicas que admite el cifrado de Cognitive Services. Para más información acerca de las claves, consulte la sección **Claves en Key Vault** en [Información acerca de claves, secretos y certificados de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/about-keys-secrets-and-certificates#key-vault-keys).

> [!NOTE]
> Si se elimina todo el almacén de claves, los datos no volverán a mostrarse y se anulará la implementación de todos los modelos. Todos los datos cargados se eliminarán de Traductor personalizado. 

### <a name="revoke-access-to-customer-managed-keys"></a>Revocación del acceso a las claves administradas por el cliente

Para revocar el acceso a las claves administradas por el cliente, use PowerShell o la CLI de Azure. Para más información, consulte la referencia de [PowerShell para Azure Key Vault](https://docs.microsoft.com/powershell/module/az.keyvault//) o la referencia de la [CLI para Azure Key Vault](https://docs.microsoft.com/cli/azure/keyvault). La revocación del acceso bloquea de manera eficaz el acceso a todos los datos del recurso de Cognitive Services y se anulará la implementación de los modelos, ya que el servicio no puede acceder a la clave de cifrado. Los datos cargados también se eliminarán de Traductor personalizado.


## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview)
