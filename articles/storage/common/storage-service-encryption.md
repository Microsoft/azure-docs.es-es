---
title: Cifrado de Azure Storage para datos en reposo
description: Azure Storage protege los datos cifrándolos automáticamente antes de guardarlos en la nube. Puede confiar en las claves administradas por Microsoft para el cifrado de los datos de la cuenta de almacenamiento, o puede administrar el cifrado con sus propias claves.
services: storage
author: tamram
ms.service: storage
ms.date: 08/24/2020
ms.topic: conceptual
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: e949c3db6d8c0cafab8556dbfde367e6e49273e9
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89078204"
---
# <a name="azure-storage-encryption-for-data-at-rest"></a>Cifrado de Azure Storage para datos en reposo

Azure Storage cifra automáticamente los datos al guardarlos en la nube. El cifrado de Azure Storage protege los datos y le ayuda a satisfacer los requisitos de cumplimiento normativo y seguridad de la organización.

## <a name="about-azure-storage-encryption"></a>Acerca del cifrado de Azure Storage

Los datos de Azure Storage se cifran y descifran de forma transparente mediante el [cifrado AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) de 256 bits, uno de los cifrados de bloques más sólidos que hay disponibles, y son compatibles con FIPS 140-2. El cifrado de Azure Storage es similar al cifrado de BitLocker en Windows.

El cifrado de Azure Storage está habilitado para todas las cuentas de almacenamiento, incluidas las cuentas de almacenamiento clásicas y las de Resource Manager. El cifrado de Azure Storage no se puede deshabilitar. Como los datos están protegidos de forma predeterminada, no es necesario modificar el código o las aplicaciones para aprovechar el cifrado de Azure Storage.

Los datos de una cuenta de almacenamiento se cifran independientemente de su nivel de rendimiento (Estándar o Premium), del nivel de acceso (frecuente o esporádico) o del modelo de implementación (Azure Resource Manager o clásico). También se cifran todos los blobs del nivel de archivo. Todas las opciones de redundancia de Azure Storage admiten el cifrado, y todos los datos de las regiones primaria y secundaria se cifran cuando la replicación geográfica está habilitada. Se cifran todos los recursos de Azure Storage, incluidos los blobs, los discos, los archivos, las colas y las tablas. También se cifran todos los metadatos de objetos. No hay ningún costo adicional para el cifrado de Azure Storage.

Todos los blobs en bloques, blobs en anexos o blobs en páginas que se escribieron en Azure Storage después del 20 de octubre de 2017 está cifrados. Los blobs creados antes de esta fecha se siguen cifrando mediante un proceso en segundo plano. Para forzar el cifrado de un blob creado antes del 20 de octubre de 2017, puede volver a escribir el blob. Para más información sobre cómo comprobar el estado de cifrado de un blob, consulte [Comprobación del estado de cifrado de un blob](../blobs/storage-blob-encryption-status.md).

Para más información sobre de los módulos criptográficos subyacentes al cifrado de Azure Storage, vea [API de criptografía: última generación](https://docs.microsoft.com/windows/desktop/seccng/cng-portal).

Para más información sobre el cifrado y la administración de claves para Azure Managed Disks, consulte [Cifrado del lado servidor de Azure Managed Disks](../../virtual-machines/windows/disk-encryption.md) para máquinas virtuales Windows o [Cifrado del lado servidor de Azure Managed Disks](../../virtual-machines/linux/disk-encryption.md) para máquinas virtuales Linux.

## <a name="about-encryption-key-management"></a>Información sobre la administración de claves de cifrado

Los datos de una cuenta de almacenamiento nueva se cifran con claves administradas por Microsoft de manera predeterminada. Puede seguir confiando en las claves administradas por Microsoft para el cifrado de los datos, o puede administrar el cifrado con sus propias claves. Si opta por administrar el cifrado con sus propias claves, tiene dos opciones. Puede usar cualquiera de los tipos de administración de claves, o ambos:

- Puede especificar una *clave administrada por el cliente* con Azure Key Vault que se usará para cifrar y descifrar datos en el almacenamiento de blobs y en Azure Files.<sup>1,2</sup> Para más información sobre las claves administradas por el cliente, consulte [Uso de claves administradas por el cliente con Azure Key Vault para administrar el cifrado de Azure Storage](encryption-customer-managed-keys.md).
- En las operaciones de almacenamiento de blobs, puede especificar una *clave proporcionada por el cliente*. Un cliente que realiza una solicitud de lectura o escritura en el almacenamiento de blobs puede incluir una clave de cifrado en la solicitud para tener un control detallado sobre el cifrado y el descifrado de los datos de blob. Para más información sobre las claves proporcionadas por el cliente, consulte [Especificación de una clave de cifrado en una solicitud a Blob Storage](encryption-customer-provided-keys.md).

En la tabla siguiente se comparan las opciones de administración de claves para el cifrado de Azure Storage.

| Parámetro de administración de claves | Claves administradas por Microsoft | Claves administradas por el cliente | Claves proporcionadas por el cliente |
|--|--|--|--|
| Operaciones de cifrado y descifrado | Azure | Azure | Azure |
| Servicios de Azure Storage admitidos | All | Blob Storage, Azure Files<sup>1,2</sup> | Blob Storage |
| Almacenamiento de claves | Almacén de claves de Microsoft | Azure Key Vault | Propio almacén de claves del cliente |
| Responsabilidad de la rotación de claves | Microsoft | Customer | Customer |
| Control de claves | Microsoft | Customer | Customer |

<sup>1</sup> Para obtener información sobre cómo crear una cuenta que admita el uso de claves administradas por el cliente con Queue Storage, consulte [Creación de una cuenta que admita las claves administradas por el cliente para colas](account-encryption-key-create.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json).<br />
<sup>2</sup> Para obtener información sobre cómo crear una cuenta que admita el uso de claves administradas por el cliente con Table Storage, consulte [Creación de una cuenta que admita las claves administradas por el cliente para tablas](account-encryption-key-create.md?toc=%2fazure%2fstorage%2ftables%2ftoc.json).

> [!NOTE]
> Las claves administradas por Microsoft se rotan correctamente según los requisitos de cumplimiento. Si tiene requisitos específicos de rotación de claves, Microsoft recomienda que se cambie a las claves administradas por el cliente para que pueda administrar y auditar la rotación usted mismo.

## <a name="encryption-scopes-for-blob-storage-preview"></a>Ámbitos de cifrado para Blob Storage (versión preliminar)

De forma predeterminada, una cuenta de almacenamiento está cifrada con una clave cuyo ámbito es la cuenta de almacenamiento. Puede optar por usar claves administradas por Microsoft o claves administradas por el cliente almacenadas en Azure Key Vault para proteger y controlar el acceso a la clave que cifra sus datos.

Los ámbitos de cifrado permiten administrar opcionalmente el cifrado a nivel del contenedor o de un blob individual. Se pueden usar ámbitos de cifrado para crear límites seguros entre los datos que residen en la misma cuenta de almacenamiento, pero que pertenecen a clientes distintos.

Puede crear uno o varios ámbitos de cifrado para una cuenta de almacenamiento mediante el proveedor de recursos de Azure Storage. Cuando se crea un ámbito de cifrado, se especifica si el ámbito está protegido con una clave administrada por Microsoft o con una clave administrada por el cliente almacenada en Azure Key Vault. Distintos ámbitos de cifrado de una misma cuenta de almacenamiento pueden usar claves administradas por Microsoft o por el cliente.

Después de crear un ámbito de cifrado, puede especificar ese ámbito de cifrado en una solicitud para crear un contenedor o un blob. Para obtener más información acerca de cómo crear un ámbito de cifrado, consulte [Creación y administración de ámbitos de cifrado (versión preliminar)](../blobs/encryption-scope-manage.md).

> [!NOTE]
> Los ámbitos de cifrado no se admiten con las cuentas de almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) ni con las cuentas de almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS) durante la versión preliminar.

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

> [!IMPORTANT]
> La versión preliminar de los ámbitos de cifrado está pensada para usos distintos del de producción. En este momento no hay contratos de nivel de servicio de producción disponibles.
>
> Para evitar costos inesperados, asegúrese de deshabilitar los ámbitos de cifrado que no necesite actualmente.

### <a name="create-a-container-or-blob-with-an-encryption-scope"></a>Creación de un contenedor o blob con un ámbito de cifrado

Los blobs que se crean en un ámbito de cifrado se cifran con la clave especificada para ese ámbito. Puede especificar un ámbito de cifrado para un blob individual al crear el blob, o bien puede especificar un ámbito de cifrado predeterminado al crear un contenedor. Al especificar un ámbito de cifrado predeterminado en el nivel de un contenedor, todos los blobs de dicho contenedor se cifran con la clave asociada al ámbito predeterminado.

Al crear un blob en un contenedor que tiene un ámbito de cifrado predeterminado, puede especificar un ámbito de cifrado que invalide el ámbito de cifrado predeterminado si el contenedor está configurado para permitir invalidaciones del ámbito de cifrado predeterminado. Para evitar invalidaciones del ámbito de cifrado predeterminado, configure el contenedor para denegar las invalidaciones para un blob individual.

Las operaciones de lectura en un blob que pertenece a un ámbito de cifrado se producen de forma transparente, siempre y cuando el ámbito de cifrado no esté deshabilitado.

### <a name="disable-an-encryption-scope"></a>Deshabilitación de un ámbito de cifrado

Al deshabilitar un ámbito de cifrado, las operaciones de lectura o escritura posteriores realizadas con el ámbito de cifrado producirán un error con el código de error HTTP 403 (prohibido). Si vuelve a habilitar el ámbito de cifrado, las operaciones de lectura y escritura continuarán con normalidad.

Cuando se deshabilita un ámbito de cifrado, ya no se le facturará. Deshabilite los ámbitos de cifrado que no sean necesarios para evitar cargos innecesarios.

Si el ámbito de cifrado está protegido con claves administradas por el cliente para Azure Key Vault, también puede eliminar la clave asociada en el almacén de claves para deshabilitar el ámbito de cifrado. Tenga en cuenta que las claves administradas por el cliente en Azure Key Vault están protegidas por la protección de eliminación y purga temporal, y una clave eliminada está sujeta al comportamiento definido por esas propiedades. Para más información, consulte uno de los siguientes temas en la documentación de Azure Key Vault:

- [Uso de la eliminación temporal con PowerShell](../../key-vault/general/soft-delete-powershell.md)
- [Uso de la eliminación temporal con la CLI](../../key-vault/general/soft-delete-cli.md).

> [!NOTE]
> No es posible eliminar un ámbito de cifrado.

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es Azure Key Vault?](../../key-vault/general/overview.md)
- [Configurar las claves administradas por el cliente para el cifrado de Azure Storage desde Azure Portal](storage-encryption-keys-portal.md)
- [Configurar las claves administradas por el cliente para el cifrado de Azure Storage desde PowerShell](storage-encryption-keys-powershell.md)
- [Configuración de las claves administradas por el cliente para el cifrado de Azure Storage desde la CLI de Azure](storage-encryption-keys-cli.md)
