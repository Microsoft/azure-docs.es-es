---
title: Requisitos del servicio Azure Import/Export | Microsoft Docs
description: Conozca los requisitos de hardware y software del servicio Azure Import/Export.
author: alkohli
services: storage
ms.service: storage
ms.topic: conceptual
ms.date: 08/12/2019
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: 7b7915e1b7779c4d0359b9ac190574c43a624cbe
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87276404"
---
# <a name="azure-importexport-system-requirements"></a>Requisitos del sistema de Azure Import/Export

En este artículo se describen los requisitos importantes del servicio Azure Import/Export. Se recoienda que revise cuidadosamente la información antes de usar el servicio Import/Export y que luego vuelva a ella si es necesario durante el funcionamiento.

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

Para preparar las unidades de disco duro con la herramienta WAImportExport, se admite el siguiente **sistema operativo de 64 bits que admite Cifrado de unidad BitLocker**.


|Plataforma |Versión |
|---------|---------|
|Windows     | Windows 7 Enterprise, Windows 7 Ultimate <br> Windows 8 Pro, Windows 8 Enterprise, Windows 8.1 Pro, Windows 8.1 Enterprise <br> Windows 10        |
|Windows Server     |Windows Server 2008 R2 <br> Windows Server 2012, Windows Server 2012 R2         |

## <a name="other-required-software-for-windows-client"></a>Otro software necesario para el cliente Windows

|Plataforma |Versión |
|---------|---------|
|.NET Framework    | 4.5.1       |
| BitLocker        |  _          |


## <a name="supported-storage-accounts"></a>Cuentas de almacenamiento admitidas

El servicio Azure Import/Export admite los siguientes tipos de cuentas de almacenamiento:

- Cuentas de almacenamiento de uso general estándar v2 (recomendadas para la mayoría de los escenarios)
- Cuentas de Blob Storage
- Cuentas de almacenamiento de uso general v1 (implementación clásica o de Azure Resource Manager),

Para más información sobre las cuentas de almacenamiento, vea [Información general acerca de las cuentas de Azure Storage](storage-account-overview.md).

Puede utilizar cada trabajo para transferir datos desde o hacia una sola cuenta de almacenamiento. Dicho de otra forma, un trabajo de importación y exportación no puede abarcar varias cuentas de almacenamiento. Para obtener información acerca de la creación de una nueva cuenta de almacenamiento, consulte [Creación de una cuenta de almacenamiento](storage-account-create.md).

> [!IMPORTANT]
> En el caso de las cuentas de almacenamiento donde se habilitó la característica [Puntos de conexión de servicio de red virtual](../../virtual-network/virtual-network-service-endpoints-overview.md) , use la opción **Permitir servicios de Microsoft de confianza…** para [habilitar el servicio Import/Export](/azure/storage/common/storage-network-security) para importar o exportar datos hacia y desde Azure.

## <a name="supported-storage-types"></a>Tipos de almacenamiento compatibles

La siguiente lista muestra los tipos de almacenamiento que se admiten con el servicio Azure Import/Export.


|Trabajo  |Servicio de almacenamiento |Compatible  |No compatible  |
|---------|---------|---------|---------|
|Importar     |  Azure Blob Storage <br><br> Almacenamiento de archivos de Azure       | Blobs en bloques y blobs en páginas admitidos <br><br> Archivos admitidos          |
|Exportación     |   Azure Blob Storage       | Blobs en bloques, blobs en páginas y blobs en anexos admitidos         | No se admite Azure Files


## <a name="supported-hardware"></a>Hardware admitido

Para el servicio Azure Import/Export, se necesitan discos admitidos para copiar los datos.

### <a name="supported-disks"></a>Discos admitidos

En la siguiente lista se muestran los discos que se pueden usar con el servicio Import/Export.


|Tipo de disco  |Size  |Compatible |
|---------|---------|---------|
|SSD    |   2,5"      |SATA III          |
|HDD     |  2,5"<br>3,5"       |SATA II, SATA III         |

No se admiten los tipos de disco siguientes:

- USB.
- Unidad de disco duro externa con adaptador USB integrado.
- Discos dentro de la carcasa de una unidad de disco duro externa.

Un único trabajo de importación o exportación puede tener:

- 10 HDD o SSD como máximo.
- Una combinación de HDD y SSD de cualquier tamaño.

Un volumen grande de unidades puede distribuirse entre varios trabajos, y no hay ningún límite en el número de trabajos que se pueden crear. Para los trabajos de importación, solo se procesa el primer volumen de datos de la unidad. El volumen de datos debe tener formato NTFS.

Al preparar las unidades de disco duro y copiar los datos mediante la herramienta WAImportExport, puede usar adaptadores de USB externos. La mayoría de los adaptadores comerciales USB 3.0 o posteriores deberían funcionar.

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a la utilidad de línea de comandos AzCopy](storage-use-azcopy.md)
