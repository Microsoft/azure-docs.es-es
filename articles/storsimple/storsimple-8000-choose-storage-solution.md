---
title: Opciones para transferir datos a Azure con un dispositivo | Microsoft Docs
description: Obtenga información sobre cómo elegir el dispositivo adecuado para la transferencia de datos locales a Azure entre Data Box Edge, Azure File Sync y la serie StorSimple 8000.
services: storsimple
author: alkohli
ms.service: storsimple
ms.topic: article
ms.date: 04/01/2019
ms.author: alkohli
ms.openlocfilehash: 903342230b28f8e4fbdcdfd93543e5a0aa54975e
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88183282"
---
# <a name="compare-storsimple-with-azure-file-sync-and-data-box-edge-data-transfer-options"></a>Comparación de las opciones de transferencia de datos StorSimple con Azure File Sync y Data Box Edge 

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]
 
Este documento contiene información general sobre las opciones de transferencia de datos locales a Azure y una comparación entre Data Box Edge, Azure File Sync y StorSimple serie 8000.

- **[Data Box Edge](/azure/databox-online/data-box-edge-overview)** : Data Box Edge es un dispositivo de red local que mueve datos a y desde Azure y que tiene un proceso perimetral que admite inteligencia artificial para procesar previamente los datos durante la carga. Data Box Gateway es una versión virtual del dispositivo con las mismas funcionalidades de transferencia de datos.
- **[Azure File Sync](/azure/storage/files/storage-sync-files-deployment-guide)** : se puede usar Azure File Sync para centralizar los recursos compartidos de archivos de su organización en Azure Files sin renunciar a la flexibilidad, el rendimiento y la compatibilidad de un servidor de archivos local. Azure File Sync transforma Windows Server en una caché rápida de los recursos compartidos de archivos de Azure. La disponibilidad general de Azure File Sync se anunció anteriormente en 2018.
- **[StorSimple](/azure/storsimple/storsimple-overview)** : StorSimple es un dispositivo híbrido que ayuda a las empresas a consolidar su infraestructura de almacenamiento para almacenamiento principal, protección de datos, archivado y recuperación ante desastres en una única solución mediante una estrecha integración con Azure Storage. Puede encontrar el ciclo de vida de StorSimple [aquí](https://support.microsoft.com/lifecycle/search?alpha=Azure%20StorSimple%208000%20Series).

## <a name="comparison-summary"></a>Resumen de comparación

|                           |StorSimple 8000   |Azure File Sync   |Data Box Edge           |
|---------------------------|----------------------------------------|-------------------------------|-----------------------------------------|
|**Información general**     |Almacenamiento híbrido en capas y archivado|Almacenamiento del servidor de archivos general con nube por niveles y sincronización multisitio.  |Solución de almacenamiento para procesar previamente los datos y enviarlos a Azure a través de la red.        |
|**Escenarios**    |Servidor de archivos, archivado y destino de copia de seguridad |Servidor de archivos, archivado (multisitio)   |Transferencia de datos, procesamiento previo de datos, como ML, inferencias, IoT o archivado    |
|**Proceso perimetral** |No disponible |No disponible |Admite la ejecución de contenedores con Azure IoT Edge    |
|**Factor de forma**  |Dispositivo físico   |Agente instalado en Windows Server |Dispositivo físico   |
|**Hardware**     |Dispositivo físico que proporciona Microsoft como parte del servicio | Proporcionado por el cliente |Dispositivo físico que proporciona Microsoft como parte del servicio  |
|**Formato de datos**  |Personalizar formato   |Archivos         |Blobs o archivos    |
|**Compatibilidad con protocolos** |iSCSI          |SMB, NFS    | SMB o NFS      |
|**Precios**      |[StorSimple](https://azure.microsoft.com/pricing/details/storsimple/) |[Azure File Sync](https://azure.microsoft.com/pricing/details/storage/files/)  |[Data Box Edge](https://azure.microsoft.com/pricing/details/storage/databox/edge/)  |

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre [Azure Data Box Edge](/azure/databox-online/data-box-edge-overview) y [Azure Data Box Gateway](/azure/databox-online/data-box-gateway-overview).
- Obtenga información sobre [Azure File Sync](/azure/storage/files/storage-sync-files-deployment-guide).
