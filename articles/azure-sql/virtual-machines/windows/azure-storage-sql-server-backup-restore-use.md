---
title: Uso de Azure Storage para la copia de seguridad y la restauración de SQL Server | Microsoft Docs
description: Aprenda a realizar una copia de seguridad de SQL Server en Azure Storage. Se explican las ventajas de realizar una copia de seguridad de bases de datos SQL en Azure Storage.
services: virtual-machines-windows
documentationcenter: ''
author: MikeRayMSFT
tags: azure-service-management
ms.assetid: 0db7667d-ef63-4e2b-bd4d-574802090f8b
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 01/31/2017
ms.author: mathoma
ms.openlocfilehash: 0b90f61bcf2698c200e16eb6f6d3fe98cc05d827
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293078"
---
# <a name="use-azure-storage-for-sql-server-backup-and-restore"></a>Uso de Azure Storage para la copia de seguridad y la restauración de SQL Server
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

A partir de SQL Server 2012 SP1 CU2, ya puede escribir copias de seguridad de SQL Server directamente en Azure Blob Storage. Puede usar esta función para realizar copias de seguridad y restauraciones desde Azure Blob Storage y una base de datos de SQL Server. Las copias de seguridad en la nube ofrecen ventajas de disponibilidad, almacenamiento externo ilimitado replicado geográficamente y facilidad de migración de datos a la nube y desde ella. Puede emitir instrucciones BACKUP o RESTORE mediante T-SQL o SMO.

## <a name="overview"></a>Información general
SQL Server 2016 presenta nuevas funcionalidades; puede usar [copias de seguridad de instantánea de archivos](https://msdn.microsoft.com/library/mt169363.aspx) para realizar copias de seguridad prácticamente instantáneas y restauraciones increíblemente rápidas.

En este tema se explica por qué podría decidir utilizar Azure Storage para realizar copias de seguridad de SQL Server y además se describen los componentes implicados. Puede usar los recursos que se incluyen al final del artículo para acceder a tutoriales e información adicional que le permitan comenzar a usar este servicio para las copias de seguridad de SQL Server.

## <a name="benefits-of-using-azure-blob-storage-for-sql-server-backups"></a>Ventajas del uso de Azure Blob Storage para las copias de seguridad de SQL Server
A la hora de realizar una copia de seguridad de SQL Server, se enfrenta a varias dificultades. Entre estas cabe mencionar la administración del almacenamiento, el riesgo de error de almacenamiento, el acceso a almacenamiento sin conexión y la configuración del hardware. Muchas de estas dificultades se solucionan mediante el uso de Azure Blob Storage para realizar las copias de seguridad de SQL Server. Considere las siguientes ventajas:

* **Facilidad de uso**: el almacenamiento de sus copias de seguridad en blobs de Azure puede ser una manera cómoda, flexible y fácil de acceder a opciones fuera del sitio. Crear almacenamiento externo para sus copias de seguridad de SQL Server puede ser tan fácil como modificar sus scripts o trabajos existentes para usar la sintaxis **BACKUP TO URL** . Normalmente, el almacenamiento externo debe estar suficientemente alejado de la ubicación de la base de datos de producción para impedir que un desastre afecte a las ubicaciones de las bases de datos externa y de producción. Al elegir la [replicación geográfica de los blobs de Azure](../../../storage/common/storage-redundancy.md), tiene una capa adicional de protección en caso de que se produzca un desastre que pudiera afectar a toda la región.
* **Archivo de copias de seguridad**: Azure Blob Storage ofrece una mejor alternativa a las opciones de cinta que se usan normalmente para archivar las copias de seguridad. El almacenamiento en cinta puede requerir el transporte físico a una instalación externa y medidas para proteger los medios. El almacenamiento de las copias de seguridad en Azure Blob Storage ofrece una opción de archivado instantánea, altamente disponible y resistente.
* **Hardware administrado**: con los servicios de Azure no hay sobrecarga por la administración del hardware. Los servicios de Azure administran el hardware y ofrecen replicación geográfica para conseguir redundancia y protección contra los errores del hardware.
* **Almacenamiento ilimitado**: al habilitar una copia de seguridad directa en blobs de Azure, tendrá acceso a un almacenamiento prácticamente ilimitado. Como alternativa, la copia de seguridad en un disco de máquina virtual de Azure tiene límites basados en el tamaño de la máquina. Existe un límite en el número de discos que se pueden conectar a una máquina virtual de Azure para copias de seguridad. Este límite es de 16 discos para una instancia muy grande e inferior para instancias menores.
* **Disponibilidad de copias de seguridad**: las copias de seguridad almacenadas en blobs de Azure están disponibles desde cualquier lugar y en cualquier momento y es fácil tener acceso a ellas para restauraciones en una instancia de SQL Server, sin necesidad de conectar o desconectar la base de datos o de descargar y conectar el VHD.
* **Costo**: Solo se paga por el servicio que se usa. Puede ser rentable como opción de archivado externo y de copia de seguridad. Consulte la [Calculadora de precios de Azure](https://go.microsoft.com/fwlink/?LinkId=277060 "Calculadora de precios"), y el [artículo de precios de Azure](https://go.microsoft.com/fwlink/?LinkId=277059 "Artículo sobre precios") para más información.
* **Instantáneas de almacenamiento**: cuando los archivos de base de datos se almacenan en un blob de Azure y está utilizando SQL Server 2016, puede usar [copias de seguridad de instantánea de archivo](https://msdn.microsoft.com/library/mt169363.aspx) para realizar copias de seguridad casi instantáneas y restauraciones increíblemente rápidas.

Para obtener más información, consulte [Copia de seguridad y restauración de SQL Server con el servicio Microsoft Azure Blob Storage](https://go.microsoft.com/fwlink/?LinkId=271617).

En las dos secciones siguientes se presenta Azure Blob Storage, junto con los componentes necesarios de SQL Server. Es importante comprender los componentes y su interacción para usar correctamente la característica de copia de seguridad y restauración de Azure Blob Storage.

## <a name="azure-blob-storage-components"></a>Componentes de Azure Blob Storage
Cuando se realiza una copia de seguridad en Azure Blob Storage, se utilizan los siguientes componentes de Azure.

| Componente | Descripción |
| --- | --- |
| **Cuenta de almacenamiento** |La cuenta de almacenamiento es el punto de partida para todos los servicios de almacenamiento. Para acceder a Azure Blob Storage, primero debe crear una cuenta de Azure Storage. Para obtener más información acerca de Azure Blob Storage, consulte [Uso de Azure Blob Storage](https://azure.microsoft.com/develop/net/how-to-guides/blob-storage/). |
| **Contenedor** |Un contenedor ofrece la agrupación de un conjunto de blobs y puede almacenar un número ilimitado de ellos. Para realizar una copia de seguridad de SQL Server en Azure Blob Storage, debe haber creado un contenedor raíz como mínimo. |
| **Blob** |Un archivo de cualquier tipo y tamaño. Los blobs son direccionables mediante el formato de dirección URL siguiente: **https://[cuenta de almacenamiento].blob.core.windows.net/[contenedor]/[blob]** . Para más información sobre los blobs en páginas, consulte la [descripción de los blobs en bloques y en páginas](https://msdn.microsoft.com/library/azure/ee691964.aspx) |

## <a name="sql-server-components"></a>Componentes de SQL Server
Cuando se realiza una copia de seguridad en Azure Blob Storage, se utilizan los siguientes componentes de SQL Server.

| Componente | Descripción |
| --- | --- |
| **URL** |Una dirección URL especifica un identificador uniforme de recursos (URI) para un archivo de copia de seguridad único. La URL se usa para proporcionar la ubicación y el nombre del archivo de la copia de seguridad de SQL Server. La dirección URL debe apuntar a un blob real, no solo a un contenedor. Si el blob no existe, se crea. Si se especifica un blob que ya existe, BACKUP provoca un error, a menos que se especifique la opción > WITH FORMAT. El siguiente es un ejemplo de la dirección URL que especificaría en el comando BACKUP: **http[s]://[storageaccount].blob.core.windows.net/[container]/[FILENAME.bak]** . Se recomienda HTTPS, pero no es obligatorio. |
| **Credential:** |La información necesaria para conectarse y autenticarse en Azure Blob Storage se almacena como credencial. Para que SQL Server escriba copias de seguridad en un blob de Azure o realice la restauración desde él es preciso crear una credencial de SQL Server. Para obtener más información, vea [Credencial de SQL Server](https://msdn.microsoft.com/library/ms189522.aspx). |

> [!NOTE]
> SQL Server 2016 se actualizó para admitir los blobs en bloques. Consulte el [Tutorial: Usar el servicio Azure Blob Storage con SQL Server 2016](https://msdn.microsoft.com/library/dn466438.aspx) para obtener más información.
> 
> 

## <a name="next-steps"></a>Pasos siguientes
1. Cree una cuenta de Azure si aún no tiene una. Si va a evaluar Azure, podría considerar la [evaluación gratuita](https://azure.microsoft.com/free/).
2. A continuación, siga uno de los siguientes tutoriales para crear una cuenta de almacenamiento y realizar una restauración.
   
   * **SQL Server 2014**: [Tutorial: Copia de seguridad y restauración de SQL Server en el servicio Azure Blob Storage](https://msdn.microsoft.com/library/jj720558\(v=sql.120\).aspx).
   * **SQL Server 2016**: [Tutorial: Usar el servicio Azure Blob Storage con SQL Server 2016](https://msdn.microsoft.com/library/dn466438.aspx).
3. Revise la documentación adicional, empezando por [Copia de seguridad y restauración de SQL Server con el servicio Microsoft Azure Blob Storage](https://msdn.microsoft.com/library/jj919148.aspx).

Si tiene algún problema, revise el tema [Prácticas recomendadas y solución de problemas de Copia de seguridad en URL de SQL Server](https://msdn.microsoft.com/library/jj919149.aspx).

Para ver otras opciones de copia de seguridad y restauración de SQL Server, consulte [Copia de seguridad y restauración de SQL Server en VM de Azure](backup-restore.md).

