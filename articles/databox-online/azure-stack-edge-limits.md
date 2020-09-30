---
title: Límites de Azure Stack Edge Pro | Microsoft Docs
description: Obtenga información sobre los límites y tamaños recomendados a medida que implementa y opera Azure Stack Edge Pro, incluidos los límites de servicio, dispositivos y almacenamiento.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: 5f3c39ce7334145d3ffc1d54badb1f7b766da70f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904458"
---
# <a name="azure-stack-edge-pro-limits"></a>Límites de Azure Stack Edge Pro

Tenga en cuenta estos límites cuando implemente y use su solución de Microsoft Azure Stack Edge Pro. 

## <a name="azure-stack-edge-service-limits"></a>Límites del servicio de Azure Stack Edge

[!INCLUDE [data-box-edge-gateway-service-limits](../../includes/data-box-edge-gateway-service-limits.md)]

## <a name="azure-stack-edge-device-limits"></a>Límites del dispositivo Azure Stack Edge

En la siguiente tabla se describen los límites de los dispositivos Azure Stack Edge Pro. 

| Descripción | Value |
|---|---|
|No. de archivos por dispositivo |100 millones |
|No. de recursos compartidos por dispositivo |24 |
|No. de recursos compartidos por contenedor |1 |
|Tamaño máximo de archivo escrito en un recurso compartido| 5 TB |

## <a name="azure-storage-limits"></a>Límites de almacenamiento de Azure

[!INCLUDE [data-box-edge-gateway-storage-limits](../../includes/data-box-edge-gateway-storage-limits.md)]

## <a name="data-upload-caveats"></a>Advertencias al cargar los datos

[!INCLUDE [data-box-edge-gateway-storage-data-upload-caveats](../../includes/data-box-edge-gateway-storage-data-upload-caveats.md)]

## <a name="azure-storage-account-size-and-object-size-limits"></a>Límites de tamaño objeto y de tamaño de cuenta de almacenamiento en Azure

[!INCLUDE [data-box-edge-gateway-storage-acct-limits](../../includes/data-box-edge-gateway-storage-acct-limits.md)]


## <a name="azure-object-size-limits"></a>Límites de tamaño de objeto de Azure

[!INCLUDE [data-box-edge-gateway-storage-object-limits](../../includes/data-box-edge-gateway-storage-object-limits.md)]

## <a name="next-steps"></a>Pasos siguientes

- [Preparación de la implementación de Azure Stack Edge Pro](azure-stack-edge-deploy-prep.md)
