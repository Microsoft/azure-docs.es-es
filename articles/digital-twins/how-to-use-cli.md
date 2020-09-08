---
title: Uso de la CLI de Azure Digital Twins
titleSuffix: Azure Digital Twins
description: Consulte el procedimiento de inicio y para usar la CLI de Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 05/25/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 2c642b2441d1f30c31e707a237732e028f548ac5
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89298188"
---
# <a name="use-the-azure-digital-twins-cli"></a>Uso de la CLI de Azure Digital Twins

Además de administrar la instancia de Azure Digital Twins en Azure Portal, Azure Digital Twins tiene una **interfaz de la línea de comandos (CLI)** que puede usar para realizar la mayoría de las acciones principales con el servicio, entre las que se incluyen:
* Administración de una instancia de Azure Digital Twins
* Administración de modelos
* Administración de gemelos digitales
* Administración de relaciones gemelas
* Configuración de puntos de conexión
* Administración de [rutas](concepts-route-events.md)
* Configuración de la [seguridad](concepts-security.md) mediante el control de acceso basado en rol (RBAC)

[!INCLUDE [digital-twins-known-issue-cloud-shell](../../includes/digital-twins-known-issue-cloud-shell.md)]

## <a name="uses-deploy-and-validate"></a>Usos (implementación y validación)

Además de administrar la instancia, generalmente, la CLI es también una herramienta útil para la implementación y la validación.
* Los comandos de plano de control se pueden usar para hacer que la implementación de una nueva instancia sea repetible o automatizada.
* Los comandos de plano de datos se pueden usar para comprobar rápidamente los valores de la instancia y que las operaciones se completaron según lo previsto.

## <a name="get-the-extension"></a>Obtener la extensión

Los comandos de Azure Digital Twins forman parte de la [extensión de Azure IoT para la CLI de Azure](https://github.com/Azure/azure-iot-cli-extension). Puede ver la documentación de referencia de estos comandos como parte del conjunto de comandos `az iot`: [az dt](https://docs.microsoft.com/cli/azure/ext/azure-iot/dt?view=azure-cli-latest).

Puede asegurarse de que tiene la última versión de la extensión con estos pasos. Puede ejecutar estos comandos en [Azure Cloud Shell](../cloud-shell/overview.md) o en una [CLI de Azure local](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

[!INCLUDE [digital-twins-cloud-shell-extensions.md](../../includes/digital-twins-cloud-shell-extensions.md)]

## <a name="next-steps"></a>Pasos siguientes

Para obtener una alternativa a los comandos de la CLI, consulte el procedimiento para administrar una instancia de Azure Digital Twins mediante las API y los SDK:
* [*Procedimiento: Uso de las API y los SDK de Azure Digital Twins*](how-to-use-apis-sdks.md)
