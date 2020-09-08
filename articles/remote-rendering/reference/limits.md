---
title: Limitaciones
description: Limitaciones de código para las características del SDK
author: erscorms
ms.author: erscor
ms.date: 02/11/2020
ms.topic: reference
ms.openlocfilehash: af935aeaaeee11ab50b327b7f7b5b77246cb974b
ms.sourcegitcommit: 37afde27ac137ab2e675b2b0492559287822fded
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88566053"
---
# <a name="limitations"></a>Limitaciones

Algunas características tienen limitaciones de tamaño, recuento u otras.

## <a name="azure-frontend"></a>Azure Frontend

* Número total de instancias de AzureFrontend por proceso: 16.
* Número total de instancias de AzureSession por AzureFrontend: 16.

## <a name="objects"></a>Objetos

* Número total de objetos permitidos de un único tipo (Entity, CutPlaneComponent, etc.): 16 777 215.
* Número total de planos de corte activo permitidos: 8.

## <a name="geometry"></a>Geometría

* Número total de materiales permitidos en un recurso: 65 535.
* Dimensión máxima de una sola textura: 16 384 x 16 384. El proceso de conversión reducirá la escala de las texturas de origen más grandes.

## <a name="overall-number-of-polygons"></a>Número total de polígonos

El número de polígonos permitidos para todos los modelos cargados depende del tamaño de la máquina virtual que se pasa a [la API REST de administración de sesión](../how-tos/session-rest-api.md#create-a-session):

| Tamaño del servidor | Número máximo de polígonos |
|:--------|:------------------|
|estándar| 20 millones |
|prémium| ilimitado |

Vea el capítulo [Tamaño del servidor](../reference/vm-sizes.md) para obtener información más detallada sobre esta limitación.

## <a name="platform-limitations"></a>Limitaciones de la plataforma

**Escritorio de Windows 10**

* Win32/x64 es la única plataforma Win32 compatible. Win32/x86 no es compatible.

**HoloLens 2**

* No se admite la característica [representación desde la cámara PV](https://docs.microsoft.com/windows/mixed-reality/mixed-reality-capture-for-developers#render-from-the-pv-camera-opt-in).
