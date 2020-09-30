---
title: API del módulo de seguridad para Azure RTOS
description: API de referencia para el módulo de seguridad para Azure RTOS.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/07/2020
ms.author: mlottner
ms.openlocfilehash: 8191d68f701a3be6a37d8623d2d900f6aa6c1851
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932313"
---
# <a name="security-module-for-azure-rtos-api-preview"></a>API del módulo de seguridad para Azure RTOS (versión preliminar)

Esta API está diseñada para su uso solo con el módulo de seguridad para Azure RTOS. Para obtener recursos adicionales, consulte el [recurso de GitHub del módulo de seguridad para Azure RTOS](https://github.com/azure-rtos/azure-iot-preview/releases). 

## <a name="enable-security-module-for-azure-rtos"></a>Habilitación del módulo de seguridad para Azure RTOS

**nx_azure_iot_security_module_enable**

### <a name="prototype"></a>Prototipo

```c
UINT nx_azure_iot_security_module_enable(NX_AZURE_IOT *nx_azure_iot_ptr);
```

### <a name="description"></a>Descripción

Esta rutina habilita el subsistema del módulo de seguridad de IoT de Azure. Una máquina de estados interna administra la recopilación de eventos de seguridad y los envía a Azure IoT Hub. Solo se necesita una instancia de NX_AZURE_IOT_SECURITY_MODULE para administrar la recopilación de datos.

### <a name="parameters"></a>Parámetros

| Nombre | Descripción |
|---------|---------|
| nx_azure_iot_ptr  [in]    | Puntero a un objeto `NX_AZURE_IOT`.  |

### <a name="return-values"></a>Valores devueltos

|Valores devueltos  |Descripción |
|---------|---------|
|NX_AZURE_IOT_SUCCESS|   Módulo de seguridad de IoT de Azure habilitado correctamente.     |
|NX_AZURE_IOT_FAILURE   |  No se pudo habilitar el módulo de seguridad de IoT de Azure debido a un error interno.    |
|NX_AZURE_IOT_INVALID_PARAMETER   |  El módulo de seguridad requiere una instancia de #NX_AZURE_IOT válida.      |

### <a name="allowed-from"></a>Permitido desde

Subprocesos

## <a name="disable-azure-iot-security-module"></a>Deshabilitación del módulo de seguridad de IoT de Azure

**nx_azure_iot_security_module_disable**


### <a name="prototype"></a>Prototipo

```c
UINT nx_azure_iot_security_module_disable(NX_AZURE_IOT *nx_azure_iot_ptr);
```

### <a name="description"></a>Descripción

Esta rutina deshabilita el subsistema del módulo de seguridad de IoT de Azure.

### <a name="parameters"></a>Parámetros

| Nombre | Descripción |
|---------|---------|
| nx_azure_iot_ptr  [in]    | Puntero a `NX_AZURE_IOT`. Si es NULL, la instancia del singleton está deshabilitada. |

### <a name="return-values"></a>Valores devueltos

|Valores devueltos  |Descripción |
|---------|---------|
|NX_AZURE_IOT_SUCCESS     |   Correcto cuando el módulo de seguridad de IoT de Azure se ha deshabilitado correctamente.      |
|NX_AZURE_IOT_INVALID_PARAMETER   |  La instancia de IoT Hub de Azure es diferente de la instancia compuesta de singleton.       |
|NX_AZURE_IOT_FAILURE    |  No se pudo deshabilitar el módulo de seguridad de IoT de Azure debido a un error interno.       |

### <a name="allowed-from"></a>Permitido desde

Subprocesos


## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de los primeros pasos con el módulo de seguridad para Azure RTOS, consulte los artículos siguientes:

- Revise la [información general](iot-security-azure-rtos.md) del módulo de seguridad para RTOS de Defender para IoT.