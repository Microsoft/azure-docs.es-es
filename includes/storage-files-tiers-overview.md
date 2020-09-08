---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 12/27/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 7fd91e898c12a13e35ae8b9055ebb5a57de2a051
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89272179"
---
Azure Files ofrece cuatro niveles diferentes de almacenamiento: Premium, Optimizado para transacciones, Frecuente y Esporádico, con el fin de que pueda adaptar sus recursos compartidos a los requisitos de rendimiento y precio de su escenario:

- **Premium**: Los recursos compartidos de archivos Premium están respaldados por unidades de estado sólido (SSD) y se implementan en el tipo de **cuenta de almacenamiento de FileStorage**. Los recursos compartidos de archivos Prémium proporcionan un alto rendimiento y una baja latencia de forma coherente en menos de 10 milisegundos en la mayoría de las operaciones de E/S para las cargas de trabajo con un uso intensivo de E/S, lo que hace que sean adecuados para una amplia variedad de cargas de trabajo, como bases de datos, hospedaje de sitios web, y entornos de desarrollo. 
- **Optimizado para transacciones**: Los recursos compartidos de archivos con el nivel Optimizado para transacciones permiten cargas de trabajo con muchas transacciones que no necesitan la latencia que ofrecen los recursos compartidos de archivos Premium. Los recursos compartidos de archivos con el nivel Optimizado para transacciones se ofrecen en el hardware de almacenamiento estándar con el respaldo de las unidades de disco duro (HDD) y se implementan en el tipo de **cuenta de almacenamiento de uso general versión 2 (GPv2)** . Este nivel de almacenamiento se ha llamado históricamente "estándar"; sin embargo, realmente es el tipo de soporte físico de almacenamiento, en lugar del propio nivel (tanto el acceso frecuente como el esporádico también son niveles "estándar", ya que se encuentran en el hardware de almacenamiento estándar).
- **Acceso frecuente**: Los recursos compartidos de nivel de acceso frecuente ofrecen almacenamiento optimizado para escenarios de uso compartido de archivos de uso general, como los recursos compartidos de los equipos y Azure File Sync. Los recursos compartidos de nivel de acceso frecuente se ofrecen en el hardware de almacenamiento estándar que usa unidades de disco duro (HDD) y se implementan en el tipo de **cuenta de almacenamiento de uso general versión 2 (GPv2)** .
- **Acceso esporádico**: Los recursos compartidos de archivos de acceso esporádico ofrecen un almacenamiento económico optimizado para escenarios de almacenamiento de archivo en línea. Azure File Sync también puede servir perfectamente para cargas de trabajo con menor renovación. Los recursos compartidos de nivel de acceso esporádico se ofrecen en el hardware de almacenamiento estándar que usa unidades de disco duro (HDD) y se implementan en el tipo de **cuenta de almacenamiento de uso general versión 2 (GPv2)** .

Los recursos compartidos de archivos Premium solo están disponibles en un modelo de facturación aprovisionada. Para más información sobre el modelo de facturación aprovisionado para recursos compartidos de archivos Premium, consulte [Planeamiento de una implementación de Azure Files](../articles/storage/files/storage-files-planning.md#understanding-provisioning-for-premium-file-shares). Los recursos compartidos de archivos Estándar, que incluyen los recursos compartidos de archivos optimizados para las transacciones, de nivel de acceso frecuente y de nivel de acceso esporádico, están disponibles en un modelo de pago por uso.

Los recursos compartidos de archivos de nivel de acceso frecuente y de nivel de acceso esporádico están disponibles en el siguiente subconjunto de regiones públicas (los recursos compartidos de archivos optimizados para transacciones están disponibles en todas las regiones de Azure):

- Centro de Australia
- Centro de Australia 2
- Este de Australia
- Sudeste de Australia
- Sur de Brasil
- Este de Canadá
- Centro de Canadá
- Centro de Francia
- Sur de Francia
- Norte de Alemania (público)
- Centro-oeste de Alemania (público)
- India central
- Sur de India
- India occidental
- Japón Oriental
- Japón Occidental
- Centro de Corea del Sur
- Corea del Sur
- Este de Noruega
- Oeste de Noruega
- Norte de Sudáfrica
- Oeste de Sudáfrica
- Norte de Suiza
- Oeste de Suiza
- Centro de Emiratos Árabes Unidos
- Norte de Emiratos Árabes Unidos
- Sur de Reino Unido
- Oeste de Reino Unido
- Centro-Norte de EE. UU
- Centro-sur de EE. UU.

Para implementar un recurso compartido de archivos de nivel de acceso frecuente o nivel de acceso esporádico, consulte el artículo sobre la [creación de recursos compartido de archivos de nivel de acceso frecuente o nivel de acceso esporádico](../articles/storage/files/storage-how-to-create-file-share.md#create-a-hot-or-cool-file-share). 