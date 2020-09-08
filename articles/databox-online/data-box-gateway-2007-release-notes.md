---
title: Notas de la versión 2007 de Azure Stack Edge y Azure Data Box Gateway | Microsoft Docs
description: Describe los problemas críticos pendientes y los problemas solucionados de la versión 2007 de Azure Stack Edge y Data Box Gateway.
services: databox
author: twooley
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 07/14/2020
ms.author: twooley
ms.openlocfilehash: f9a08cda1466dbe284a0ac3c0e34b109be236883
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88783933"
---
# <a name="azure-stack-edge-and-azure-data-box-gateway-2007-release-notes"></a>Notas de la versión 2007 de Azure Stack Edge y Azure Data Box Gateway

En las notas de la versión siguientes, se identifican los problemas críticos pendientes y los problemas resueltos de la versión 2007 de Azure Stack Edge y Data Box Gateway.

Las notas de la versión se actualizan continuamente y se van agregando a medida que se descubren problemas críticos que requieren una solución alternativa. Antes de implementar Azure Stack Edge o Data Box Gateway, revise detenidamente la información que encontrará en las notas de la versión.

Esta versión se corresponde con las versiones de software siguientes:

- **Azure Stack Edge 2007 (1.6.1280.1667)** : KB 4566549
- **Data Box Gateway 2007 (1.6.1280.1667)** : KB 4566550

> [!NOTE]
> La actualización 2007 solo se puede aplicar a los dispositivos que ejecutan las versiones de disponibilidad general del software o versiones posteriores.

## <a name="whats-new"></a>Novedades

Esta versión contiene la siguiente corrección de errores:

- **Problema de carga**: Esta versión corrige un problema de carga en el que los reinicios de carga debido a errores pueden ralentizar la velocidad de finalización de la carga. Este problema puede producirse al cargar un conjunto de datos que se compone principalmente de archivos de gran tamaño en relación con el ancho de banda disponible, en especial, entre otras cosas, cuando está activo el límite de ancho de banda. Este cambio garantiza que se proporcione una oportunidad suficiente para completar la carga antes de reiniciar la carga de un archivo determinado.

Esta versión también contiene las siguientes actualizaciones:

- Se actualizó la imagen base para el VHD de Windows.
- Se incluyen todas las actualizaciones acumulativas de Windows y actualizaciones de .NET Framework que se lanzaron hasta mayo de 2020.
- Esta versión admite IoT Edge 1.0.9.3 en dispositivos de Azure Stack Edge.

## <a name="known-issues-in-this-release"></a>Problemas conocidos en esta versión

No se ha notificado ningún problema nuevo para esta versión. Todos los problemas notificados de la versión provienen de versiones anteriores. Para ver una lista de los problemas conocidos, vaya a [Problemas conocidos en la versión de disponibilidad general](data-box-gateway-release-notes.md#known-issues-in-ga-release).

## <a name="next-steps"></a>Pasos siguientes

- [Preparación de la implementación de Azure Stack Edge](data-box-edge-deploy-prep.md)
- [Preparación para implementar Azure Data Box Gateway](data-box-gateway-deploy-prep.md)
