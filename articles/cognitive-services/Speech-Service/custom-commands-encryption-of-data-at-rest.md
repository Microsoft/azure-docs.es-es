---
title: Cifrado del servicio Custom Commands para datos en reposo
titleSuffix: Azure Cognitive Services
description: Cifrado de Custom Commands para datos en reposo.
services: cognitive-services
author: singhsaumya
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/05/2020
ms.author: sausin
ms.openlocfilehash: 83b6e6be8764a86c41bd9156cc96f8a594dbe1e9
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293869"
---
# <a name="custom-commands-encryption-of-data-at-rest"></a>Cifrado de Comandos personalizados de datos en reposo

Custom Commands cifra automáticamente los datos al guardarlos en la nube. El cifrado del servicio Custom Commands protege los datos y ayuda a cumplir los requisitos de cumplimiento y de seguridad de la organización.

> [!NOTE]
> El servicio Custom Commands no habilita automáticamente el cifrado de los recursos de LUIS asociados a la aplicación. Si lo necesita, debe habilitar el cifrado para el recurso de LUIS [aquí](./../LUIS/luis-encryption-of-data-at-rest.md).

## <a name="about-cognitive-services-encryption"></a>Información sobre el cifrado de Cognitive Services
Los datos se cifran y descifran con el cifrado [AES de 256 bits](https://en.wikipedia.org/wiki/FIPS_140-2) compatible con [FIPS 140-2](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard). El cifrado y el descifrado son transparentes, lo que significa que el cifrado y el acceso se administran automáticamente. Dado que los datos están protegidos de forma predeterminada, no es necesario modificar el código ni las aplicaciones para utilizar el cifrado.

## <a name="about-encryption-key-management"></a>Información sobre la administración de claves de cifrado

Al usar Custom Commands, el servicio de Voz almacenará los siguientes datos en la nube:
* JSON de configuración de la aplicación de Custom Commands
* Claves de creación y predicción de LUIS

De forma predeterminada, su suscripción usa claves de cifrado administradas por Microsoft. Sin embargo, también puede administrar la suscripción con sus propias claves de cifrado. Las claves administradas por el cliente (CMK), también conocidas como Bring Your Own Key (BYOK), ofrecen más flexibilidad para crear, girar, deshabilitar y revocar controles de acceso. También permite auditar las claves de cifrado que se usan para proteger los datos.


> [!IMPORTANT]
> Las claves administradas por el cliente solo serán aquellos recursos disponibles que se hayan creado después del 27 de junio de 2020. Para usar la clave administrada por el cliente con los servicios de Voz, deberá crear un recurso de Voz. Una vez creado el recurso, puede usar Azure Key Vault para configurar la identidad administrada.

Para solicitar la capacidad de usar claves administradas por el cliente, rellene y envíe el formulario de solicitud de claves administradas por el cliente. Tardará de tres a cinco días hábiles aproximadamente en recibir una respuesta sobre el estado de la solicitud. En función de la demanda, es posible que se coloque en una cola y se apruebe a medida que haya espacio disponible. Una vez aprobado el uso de la clave administrada por el cliente con los servicios de Voz, deberá crear un recurso de Voz a partir de Azure Portal.
   > [!NOTE]
   > **Las claves administradas por el cliente (CMK) solo se admiten para Custom Commands.**
   >
   >  **Habla personalizada y Voz personalizada solo admiten el almacenamiento propio (BYOS).**  [Más información](speech-encryption-of-data-at-rest.md)
   >
   > Si utiliza el recurso de Voz determinado para acceder a este servicio, configure el almacenamiento propio explícitamente para hacerlo conforme a las necesidades de cumplimiento.


## <a name="customer-managed-keys-with-azure-key-vault"></a>Claves administradas por el cliente con Azure Key Vault

Debe usar Azure Key Vault para almacenar las claves administradas por el cliente. Puede crear sus propias claves y almacenarlas en un almacén de claves, o puede usar las API de Azure Key Vault para generarlas. El recurso de Voz y el almacén de claves deben estar en la misma región y en el mismo inquilino de Azure Active Directory (Azure AD), pero pueden estar en suscripciones distintas. Para más información sobre Azure Key Vault, vea [¿Qué es Azure Key Vault?](https://docs.microsoft.com/azure/key-vault/key-vault-overview).

Al crear un recurso de Voz y usarlo para aprovisionar la aplicación de Custom Commands, los datos siempre se cifran con claves que administra Microsoft. No es posible habilitar claves administradas por el cliente en el momento en que se crea el recurso. Las claves administradas por el cliente se almacenan en Azure Key Vault y el almacén de claves se debe aprovisionar con directivas de acceso que concedan permisos de clave a la identidad administrada que está asociada al recurso de Cognitive Services. La identidad administrada solo está disponible después de crear el recurso con el plan de tarifa requerido para CMK.

Al habilitar las claves administradas por el cliente, también se habilitará una [identidad administrada](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) asignada por el sistema, una característica de Azure AD. Una vez habilitada la identidad administrada asignada por el sistema, este recurso se registrará con Azure Active Directory. Tras su registro, se concederá a la identidad administrada acceso a la instancia de Key Vault seleccionada durante la configuración de la clave administrada por el cliente. 

> [!IMPORTANT]
> Si deshabilita las identidades administradas asignadas por el sistema, se quitará el acceso al almacén de claves y los datos cifrados con las claves de cliente dejarán de estar disponibles. Las características dependientes de estos datos dejarán de funcionar.

> [!IMPORTANT]
> Las identidades administradas no admiten actualmente escenarios entre directorios. Al configurar las claves administradas por el cliente en Azure Portal, se asigna automáticamente una identidad administrada en segundo plano. Si posteriormente mueve la suscripción, el grupo de recursos o el recurso de un directorio de Azure AD a otro, la identidad administrada asociada al recurso no se transfiere al nuevo inquilino, por lo que es posible que las claves administradas por el cliente dejen de funcionar. Para más información, vea **Transferencia de una suscripción entre directorios de Azure AD** en [Preguntas frecuentes y problemas conocidos con identidades administradas para recursos de Azure](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/known-issues#transferring-a-subscription-between-azure-ad-directories).  

## <a name="configure-azure-key-vault"></a>Configuración de Azure Key Vault

Para usar las claves administradas por el cliente, es necesario establecer dos propiedades en el almacén de claves, **Eliminación temporal** y **No purgar**. Estas propiedades no están habilitadas de forma predeterminada, pero se pueden habilitar mediante PowerShell o la CLI de Azure tanto en un almacén de claves nuevo como en uno existente.

> [!IMPORTANT]
> Si no tiene habilitadas las propiedades **Eliminación temporal** y **No purgar** y elimina la clave, no podrá recuperar los datos en el recurso de Cognitive Services.

Para aprender a habilitar estas propiedades en un almacén de claves existente, consulte las secciones **Habilitación de la eliminación temporal** y **Habilitación de la protección de purgas** en cualquiera de los siguientes artículos:

- [Uso de la eliminación temporal con PowerShell](https://docs.microsoft.com/azure/key-vault/key-vault-soft-delete-powershell).
- [Uso de la eliminación temporal con la CLI](https://docs.microsoft.com/azure/key-vault/key-vault-soft-delete-cli).

Las claves RSA de tamaño 2048 son las únicas que admite el cifrado de Azure Storage. Para más información acerca de las claves, consulte la sección **Claves en Key Vault** en [Información acerca de claves, secretos y certificados de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/about-keys-secrets-and-certificates#key-vault-keys).

## <a name="enable-customer-managed-keys-for-your-speech-resource"></a>Habilitación de claves administradas por el cliente para el recurso de Voz

Para habilitar claves administradas del cliente en Azure Portal, siga estos pasos:

1. Vaya al recurso de Voz.
1. En la hoja **Configuración** del recurso de Voz, haga clic en **Cifrado**. Seleccione la opción **Claves administradas de cliente**, como se muestra en la ilustración siguiente.

 ![Captura de pantalla que muestra cómo seleccionar Claves administradas por el cliente](media/custom-commands/select-cmk.png)

## <a name="specify-a-key"></a>Especificar una clave

Después de habilitar las claves administradas por el cliente, tendrá la oportunidad de especificar una clave para asociarla con el recurso de Cognitive Services.

### <a name="specify-a-key-as-a-uri"></a>Especificación de una clave como URI

Para especificar una clave como URI, siga estos pasos:

1. Para buscar el URI de la clave en Azure Portal, vaya al almacén de claves y seleccione la opción de configuración **Claves**. Seleccione la clave que desee y luego haga clic en ella para ver sus versiones. Seleccione cualquiera de las versiones de clave para ver su configuración.
1. Copie el valor del campo **Identificador de clave**, que proporciona el URI.

    ![Captura de pantalla en que se muestra el URI de la clave del almacén de claves](../media/cognitive-services-encryption/key-uri-portal.png)

1. En las opciones de configuración de **Cifrado** de la cuenta de Voz, elija la opción **Escriba el URI de la clave**.
1. Pegue el identificador URI que ha copiado en el campo **URI de clave**.

1. Especifique el identificador de la suscripción que contiene el almacén de claves.
1. Guarde los cambios.

### <a name="specify-a-key-from-a-key-vault"></a>Especificación de una clave a partir de un almacén de claves

Para especificar una clave a partir de un almacén de claves, asegúrese de tener un almacén de claves que contenga una clave. Para especificar una clave a partir de un almacén de claves, siga estos pasos:

1. Elija la opción **Select from Key Vault** (Seleccionar desde almacén de claves).
1. Seleccione el almacén de claves que contiene la clave que desea usar.
1. Seleccione la clave en el almacén de claves.

   ![Captura de pantalla en que se muestra la opción de clave administrada del cliente](media/custom-commands/configure-cmk-fromKV.png)

1. Guarde los cambios.

## <a name="update-the-key-version"></a>Actualización de la versión de la clave

Al crear una nueva versión de una clave, actualice la cuenta del recurso de Voz para que utilice la versión nueva. Siga estos pasos:

1. Vaya a la cuenta del recurso de Voz y muestre las opciones de configuración de **Cifrado**.
1. Escriba el identificador URI de la nueva versión de la clave. Como alternativa, puede volver a seleccionar el almacén de claves y la clave para actualizar la versión.
1. Guarde los cambios.

## <a name="use-a-different-key"></a>Uso de una clave distinta

Para cambiar la clave que se usa para el cifrado, siga estos pasos:

1. Vaya a la cuenta del recurso de Voz y muestre las opciones de configuración de **Cifrado**.
1. Escriba el identificador URI de la nueva clave. Otra opción es seleccionar el almacén de claves y elegir una clave nueva.
1. Guarde los cambios.

## <a name="rotate-customer-managed-keys"></a>Rotación de claves administradas por el cliente

Las claves administradas por el cliente se pueden rotar en Azure Key Vault según las directivas de cumplimiento. Cuando la clave rota, hay que actualizar la cuenta del recurso de Voz para usar el nuevo identificador URI de la clave. Para obtener información sobre cómo actualizar el recurso para usar una nueva versión de la clave en Azure Portal, consulte la sección [Actualización de la versión de la clave](#update-the-key-version).

La rotación de la clave no desencadena un nuevo cifrado de los datos del recurso. No es preciso que el usuario realice ninguna otra acción.

## <a name="revoke-access-to-customer-managed-keys"></a>Revocación del acceso a las claves administradas por el cliente

Para revocar el acceso a las claves administradas por el cliente, use PowerShell o la CLI de Azure. Para más información, consulte la referencia de [PowerShell para Azure Key Vault](https://docs.microsoft.com/powershell/module/az.keyvault//) o la referencia de la [CLI para Azure Key Vault](https://docs.microsoft.com/cli/azure/keyvault). La revocación del acceso bloquea de manera eficaz el acceso a todos los datos del recurso de Cognitive Services, ya que el servicio no puede acceder a la clave de cifrado.

## <a name="disable-customer-managed-keys"></a>Deshabilitación de claves administradas por el cliente

Cuando se deshabilitan las claves administradas por el cliente, la cuenta del recurso de Voz se cifra con claves que administra Microsoft. Para deshabilitar las claves administradas por el cliente, siga estos pasos:

1. Vaya a la cuenta del recurso de Voz y muestre las opciones de configuración de **Cifrado**.
1. Anule la selección de la casilla que se encuentra junto al valor **Usar su propia clave**.

## <a name="next-steps"></a>Pasos siguientes

* [Formulario de solicitud de claves administradas por el cliente de Voz](https://aka.ms/cogsvc-cmk)
* [Más información sobre Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview)
* [¿Qué son las identidades administradas?](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview)



