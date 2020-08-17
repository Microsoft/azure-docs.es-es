---
title: Uso de claves administradas por el cliente con Azure Key Vault para administrar el cifrado de cuenta
titleSuffix: Azure Storage
description: Puede usar su propia clave de cifrado para proteger los datos de la cuenta de almacenamiento. Cuando se especifica una clave administrada por el cliente, esa clave se usa para proteger y controlar el acceso a la clave que cifra los datos. Las claves administradas por el cliente ofrecen más flexibilidad para administrar controles de acceso.
services: storage
author: tamram
ms.service: storage
ms.date: 07/20/2020
ms.topic: conceptual
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: af70b1746b2ac847d964975aaf1b2186aa89be01
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87292735"
---
# <a name="use-customer-managed-keys-with-azure-key-vault-to-manage-azure-storage-encryption"></a>Uso de claves administradas por el cliente con Azure Key Vault para administrar el cifrado de Azure Storage

Puede usar su propia clave de cifrado para proteger los datos de la cuenta de almacenamiento. Cuando se especifica una clave administrada por el cliente, esa clave se usa para proteger y controlar el acceso a la clave que cifra los datos. Las claves administradas por el cliente ofrecen más flexibilidad para administrar controles de acceso.

Debe usar Azure Key Vault para almacenar las claves administradas por el cliente. Puede crear sus propias claves y almacenarlas en un almacén de claves, o puede usar las API de Azure Key Vault para generarlas. La cuenta de almacenamiento y el almacén de claves deben estar en la misma región y en el mismo inquilino de Azure Active Directory (Azure AD), pero pueden estar en distintas suscripciones. Para más información sobre Azure Key Vault, vea [¿Qué es Azure Key Vault?](../../key-vault/general/overview.md).

## <a name="about-customer-managed-keys"></a>Acerca de las claves administradas por el cliente

En el siguiente diagrama se muestra cómo Azure Storage usa Azure Active Directory y Azure Key Vault para hacer solicitudes mediante la clave administrada por el cliente:

![Diagrama que muestra cómo funcionan las claves administradas por el cliente en Azure Storage](media/encryption-customer-managed-keys/encryption-customer-managed-keys-diagram.png)

En la lista siguiente se explican los pasos numerados del diagrama:

1. Un administrador de Azure Key Vault concede permisos sobre las claves de cifrado a la identidad administrada que está asociada a la cuenta de almacenamiento.
2. Un administrador de Azure Storage configura el cifrado con una clave administrada por el cliente para la cuenta de almacenamiento.
3. Azure Storage usa la identidad administrada que está asociada a la cuenta de almacenamiento para autenticar el acceso a Azure Key Vault mediante Azure Active Directory.
4. Azure Storage encapsula la clave de cifrado de la cuenta con la clave de cliente en Azure Key Vault.
5. En operaciones de lectura y escritura, Azure Storage envía solicitudes a Azure Key Vault para desencapsular la clave de cifrado de la cuenta con el fin de realizar operaciones de cifrado y descifrado.

## <a name="create-an-account-that-supports-customer-managed-keys-for-queues-and-tables"></a>Creación de una cuenta que admita las claves administradas por el cliente para tablas y colas

Los datos almacenados en los servicios de colas y tablas no se protegen automáticamente mediante una clave administrada por el cliente cuando se habilitan las claves administradas por el cliente para la cuenta de almacenamiento. Opcionalmente, puede configurar estos servicios en el momento de crear la cuenta de almacenamiento que se va a incluir en esta protección.

Para más información sobre cómo crear una cuenta de almacenamiento que admita el uso de claves administradas por el cliente para colas y tablas, consulte [Creación de una cuenta que admita las claves administradas por el cliente para tablas y colas](account-encryption-key-create.md).

Los datos en los servicios de blobs y archivos siempre están protegidos por claves administradas por el cliente cuando el cliente configura claves administradas para la cuenta de almacenamiento.

## <a name="enable-customer-managed-keys-for-a-storage-account"></a>Habilitación de claves administradas por el cliente para una cuenta de almacenamiento

Al configurar una clave administrada por el cliente, Azure Storage encapsula la clave de cifrado de la cuenta con la clave administrada por el cliente en el almacén de claves asociado. La habilitación de claves administradas por el cliente no afecta al rendimiento y surte efecto de forma inmediata.

Al habilitar o deshabilitar las claves administradas por el cliente, o al modificar la clave o la versión de la clave, la protección de la clave de cifrado raíz cambia, pero no es necesario volver a cifrar los datos de la cuenta de Azure Storage.

Las claves administradas por el cliente solo se pueden habilitar en cuentas de almacenamiento existentes. El almacén de claves se debe configurar con directivas de acceso que concedan permisos a la identidad administrada que está asociada a la cuenta de almacenamiento. La identidad administrada solo está disponible después de crear la cuenta de almacenamiento.

Puede cambiar entre las claves administradas por el cliente y las claves administradas por Microsoft en cualquier momento. Para obtener más información sobre las claves administradas por Microsoft, vea [Información sobre la administración de claves de cifrado](storage-service-encryption.md#about-encryption-key-management).

Para aprender a usar claves administradas por el cliente con Azure Key Vault para el cifrado de Azure Storage, consulte uno de estos artículos:

- [Configuración de claves administradas por el cliente con Key Vault para el cifrado de Azure Storage mediante Azure Portal](storage-encryption-keys-portal.md)
- [Configuración de claves administradas por el cliente con Key Vault para el cifrado de Azure Storage mediante PowerShell](storage-encryption-keys-powershell.md)
- [Configuración de claves administradas por el cliente con Key Vault para el cifrado de Azure Storage mediante la CLI de Azure](storage-encryption-keys-cli.md)

> [!IMPORTANT]
> Las claves administradas por el cliente dependen de identidades administradas para los recursos de Azure, una característica de Azure AD. Las identidades administradas no admiten actualmente escenarios entre directorios. Al configurar las claves administradas por el cliente en Azure Portal, se asigna automáticamente una identidad administrada a la cuenta de almacenamiento en segundo plano. Si posteriormente mueve la suscripción, el grupo de recursos o la cuenta de almacenamiento de un directorio de Azure AD a otro, la identidad administrada asociada a la cuenta de almacenamiento no se transfiere al nuevo inquilino, por lo que es posible que las claves administradas por el cliente dejen de funcionar. Para más información, vea **Transferencia de una suscripción entre directorios de Azure AD** en [Preguntas frecuentes y problemas conocidos con identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).  

## <a name="store-customer-managed-keys-in-azure-key-vault"></a>Almacenamiento de claves administradas por el cliente en Azure Key Vault

Para habilitar las claves administradas por el cliente en una cuenta de almacenamiento, debe usar un almacén de Azure Key Vault para almacenar las claves. Debe habilitar las propiedades **Eliminación temporal** y **No purgar** en el almacén de claves.

El cifrado de almacenamiento de Azure admite claves RSA y RSA-HSM de los tamaños 2048, 3072 y 4096. Para más información acerca de las claves, consulte la sección **Claves en Key Vault** en [Información acerca de claves, secretos y certificados de Azure Key Vault](../../key-vault/about-keys-secrets-and-certificates.md#key-vault-keys).

El uso de Azure Key Vault tiene costos asociados. Para más información, vea [Precios de Key Vault](https://azure.microsoft.com/pricing/details/key-vault/).

## <a name="rotate-customer-managed-keys"></a>Rotación de claves administradas por el cliente

Las claves administradas por el cliente se pueden rotar en Azure Key Vault según las directivas de cumplimiento. Tiene dos opciones para rotar una clave administrada por el cliente:

- **Rotación automática**: para configurar la rotación automática de claves administradas por el cliente, omita la versión de la clave al habilitar el cifrado con las claves administradas por el cliente para la cuenta de almacenamiento. Si se omite la versión de la clave, Azure Storage comprueba Azure Key Vault diariamente para obtener una nueva versión de una clave administrada por el cliente. Si hay disponible una nueva versión de la clave, Azure Storage usa automáticamente la versión más reciente.
- **Rotación manual**: para usar una versión de clave determinada para el cifrado de Azure Storage, especifique la versión de la clave al habilitar el cifrado con las claves administradas por el cliente para la cuenta de almacenamiento. Si especifica la versión de la clave, Azure Storage usará esa versión para el cifrado hasta que actualice manualmente la versión de la clave.

    Cuando la clave se rota manualmente, hay que actualizar la cuenta de almacenamiento para usar el nuevo identificador URI de la versión de la clave. Para obtener información sobre cómo actualizar la cuenta de almacenamiento para usar una nueva versión de la clave en Azure Portal, consulte la sección [Actualización manual de la versión de la clave](storage-encryption-keys-portal.md#manually-update-the-key-version).

La rotación de la clave administrada por el cliente no desencadena un nuevo cifrado de los datos en la cuenta de almacenamiento. No es preciso que el usuario realice ninguna otra acción.

## <a name="revoke-access-to-customer-managed-keys"></a>Revocación del acceso a las claves administradas por el cliente

Puede revocar el acceso de la cuenta de almacenamiento a la clave administrada por el cliente en cualquier momento. Después de revocar el acceso a las claves administradas por el cliente, o después de que se haya deshabilitado o eliminado la clave, los clientes no pueden llamar a las operaciones que leen o escriben en un blob o en sus metadatos. Los intentos de llamar a cualquiera de las siguientes operaciones producirán un error con el código de error 403 (prohibido) para todos los usuarios:

- [List Blobs](/rest/api/storageservices/list-blobs), cuando se llama con el parámetro `include=metadata` en el URI de solicitud
- [Get Blob](/rest/api/storageservices/get-blob)
- [Get Blob Properties](/rest/api/storageservices/get-blob-properties)
- [Get Blob Metadata](/rest/api/storageservices/get-blob-metadata)
- [Set Blob Metadata](/rest/api/storageservices/set-blob-metadata)
- [Snapshot Blob](/rest/api/storageservices/snapshot-blob), cuando se llama con el encabezado de solicitud `x-ms-meta-name`
- [Copy Blob](/rest/api/storageservices/copy-blob)
- [Copy Blob From URL](/rest/api/storageservices/copy-blob-from-url)
- [Set Blob Tier](/rest/api/storageservices/set-blob-tier)
- [Put Block](/rest/api/storageservices/put-block)
- [Put Block From URL](/rest/api/storageservices/put-block-from-url)
- [Append Block](/rest/api/storageservices/append-block)
- [Append Block From URL](/rest/api/storageservices/append-block-from-url)
- [Put Blob](/rest/api/storageservices/put-blob)
- [Put Page](/rest/api/storageservices/put-page)
- [Put Page From URL](/rest/api/storageservices/put-page-from-url)
- [Incremental Copy Blob](/rest/api/storageservices/incremental-copy-blob) (Copia incremental del blob)

Para volver a llamar a estas operaciones, restaure el acceso a la clave administrada por el cliente.

Todas las operaciones de datos que no aparecen en esta sección pueden continuar después de que se hayan revocado las claves administradas por el cliente o se haya deshabilitado o eliminado una clave.

Para revocar el acceso a las claves administradas por el cliente, use [PowerShell](storage-encryption-keys-powershell.md#revoke-customer-managed-keys) o la [CLI de Azure](storage-encryption-keys-cli.md#revoke-customer-managed-keys).

## <a name="customer-managed-keys-for-azure-managed-disks"></a>Claves administradas por el cliente para discos administrados por Azure

Las claves administradas por el cliente también están disponibles para administrar el cifrado de discos administrados por Azure. Las claves administradas por el cliente se comportan de forma diferente en los discos administrados que en los recursos de Azure Storage. Para más información, consulte [Cifrado del lado servidor de Azure Managed Disks](../../virtual-machines/windows/disk-encryption.md) para Windows o [Cifrado del lado servidor de Azure Managed Disks](../../virtual-machines/linux/disk-encryption.md) para Linux.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de claves administradas por el cliente con Key Vault para el cifrado de Azure Storage mediante Azure Portal](storage-encryption-keys-portal.md)
- [Configuración de claves administradas por el cliente con Key Vault para el cifrado de Azure Storage mediante PowerShell](storage-encryption-keys-powershell.md)
- [Configuración de claves administradas por el cliente con Key Vault para el cifrado de Azure Storage mediante la CLI de Azure](storage-encryption-keys-cli.md)
- [Cifrado de Azure Storage para datos en reposo](storage-service-encryption.md)
