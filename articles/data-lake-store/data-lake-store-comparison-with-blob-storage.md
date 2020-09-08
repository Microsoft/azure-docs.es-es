---
title: Comparación de Azure Data Lake Storage Gen1 con Blob Storage
description: Conozca las diferencias entre Azure Data Lake Storage Gen1 y Azure Blob Storage respecto a algunos aspectos clave del procesamiento de macrodatos.
author: twooley
ms.service: data-lake-store
ms.topic: conceptual
ms.date: 03/26/2018
ms.author: twooley
ms.openlocfilehash: 6662beec47d663f92e453f52d190e41518108b0d
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88191480"
---
# <a name="comparing-azure-data-lake-storage-gen1-and-azure-blob-storage"></a>Comparación de Azure Data Lake Storage Gen1 y Azure Blob Storage

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)] 

La tabla de este artículo resume las diferencias entre Azure Data Lake Storage Gen1 y Azure Blob Storage en algunos aspectos clave del procesamiento de macrodatos. Azure Blob Storage es un almacén de objetos general escalable que está diseñado para una amplia variedad de escenarios de almacenamiento. Azure Data Lake Storage Gen1 es un repositorio a hiperescala optimizado para cargas de trabajo de análisis de macrodatos.

| Category | Azure Data Lake Storage Gen1 | Azure Blob Storage |
| -------- | ---------------------------- | ------------------ |
| Propósito |Almacenamiento optimizado para cargas de trabajo de análisis de macrodatos |Almacén de objetos de uso general para una amplia variedad de escenarios de almacenamiento, incluidos análisis de macrodatos |
| Casos de uso |Datos en lote, interactivos, de análisis de transmisiones y de aprendizaje automático, como archivos de registro, datos de IoT, transmisiones de clic y conjuntos de datos grandes |Cualquier tipo de texto o datos binarios, por ejemplo, back-end de aplicación, datos de copia de seguridad, almacenamiento multimedia para streaming y datos de uso general. Además, compatibilidad total con cargas de trabajo de análisis; datos en lote, interactivos, de análisis de streaming y de aprendizaje automático, como archivos de registro, datos de IoT, flujos de clics y conjuntos de datos grandes. |
| Conceptos clave |La cuenta de Data Lake Storage Gen1 contiene carpetas, que a su vez contienen datos almacenados como archivos |La cuenta de almacenamiento tiene contenedores, que a su vez tienen datos en forma de blobs |
| Estructura |Sistema de archivos jerárquico |Almacén de objetos con el espacio de nombres plano |
| API |API de REST a través de HTTPS |API de REST a través de HTTP/HTTPS |
| API de servidor |[WebHDFS-compatible REST API (API de REST compatible con WebHDFS)](https://msdn.microsoft.com/library/azure/mt693424.aspx) |[Azure Blob Storage REST API (API de REST de Almacenamiento de blobs de Azure)](https://msdn.microsoft.com/library/azure/dd135733.aspx) |
| Cliente de sistema de archivos de Hadoop |Sí |Sí |
| Operaciones de datos: autenticación |Basado en las [identidades de Azure Active Directory](../active-directory/develop/authentication-scenarios.md) |Basado en secretos compartidos: [teclas de acceso de cuenta](../storage/common/storage-account-keys-manage.md) y [claves de firma de acceso compartido](../storage/common/storage-dotnet-shared-access-signature-part-1.md). |
| Operaciones de datos: protocolo de autenticación |OAuth 2.0. Las llamadas deben contener un JWT válido (JSON Web Token) emitido por Azure Active Directory |Código de autenticación de mensajes basado en hash (HMAC). Las llamadas deben contener un hash SHA-256 codificado en Base64 en una parte de la solicitud HTTP. |
| Operaciones de datos: autorización |Listas de control de acceso (ACL) de POSIX.  Las ACL basadas en identidades de Azure Active Directory se pueden establecer en el nivel de archivo y de carpeta. |Para la autorización de nivel de cuenta: usar [claves de acceso de cuenta](../storage/common/storage-account-keys-manage.md)<br>Para la cuenta, el contenedor o la autorización de blob - Usar [claves de firma de acceso compartido](../storage/common/storage-dotnet-shared-access-signature-part-1.md) |
| Operaciones de datos: auditoría |Disponible. Más información [aquí](data-lake-store-diagnostic-logs.md) . |Disponible |
| Cifrado de datos en reposo |<ul><li>Transparente, en el servidor</li> <ul><li>Con claves administradas por servicios</li><li>Con claves administradas por clientes en Azure KeyVault</li></ul></ul> |<ul><li>Transparente, en el servidor</li> <ul><li>Con claves administradas por servicios</li><li>Con claves administradas por clientes en Azure KeyVault (versión preliminar)</li></ul><li>Cifrado de cliente</li></ul> |
| Operaciones de administración (por ejemplo, creación de cuentas) |[Control de acceso basado en rol de Azure (Azure RBAC)](../role-based-access-control/overview.md) para la administración de cuentas |[Control de acceso basado en rol de Azure (Azure RBAC)](../role-based-access-control/overview.md) para la administración de cuentas |
| SDK para desarrolladores |.NET, Java, Python, Node.js |.NET, Java, Python, Node.js, C++, Ruby, PHP, Go, Android, iOS |
| Rendimiento de la carga de trabajo de análisis |Rendimiento optimizado para cargas de trabajo de análisis paralelas. Gran capacidad de procesamiento e IOPS |Rendimiento optimizado para cargas de trabajo de análisis paralelas. |
| Límites de tamaño |Sin límites para el tamaño de cuenta, el tamaño de archivo o el número de archivos |Para los límites específicos, consulte [Objetivos de escalabilidad para cuentas de almacenamiento estándar](../storage/common/scalability-targets-standard-account.md) y [Objetivos de escalabilidad y rendimiento de Blob Storage](../storage/blobs/scalability-targets.md). Hay disponibles límites de cuenta más amplios; para conocerlos, póngase en contacto con el [Soporte técnico de Azure](https://azure.microsoft.com/support/faq/) |
| Redundancia geográfica |Con redundancia local (varias copias de datos en una región de Azure) |Con redundancia local (LRS), de zona (ZRS), global (GRS) y global de acceso de lectura (RA-GRS). Más información [aquí](../storage/common/storage-redundancy.md) |
| Estado de servicio |Disponibilidad general |Disponibilidad general |
| Disponibilidad regional |Consulte [aquí](https://azure.microsoft.com/regions/#services) |Disponibilidad en todas las regiones de Azure |
| Price |Consulte [Precios](https://azure.microsoft.com/pricing/details/data-lake-store/) |Consulte [Precios](https://azure.microsoft.com/pricing/details/storage/) |
