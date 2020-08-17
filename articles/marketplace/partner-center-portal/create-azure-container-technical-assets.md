---
title: 'Conceptos técnicos de las ofertas de contenedores de Azure: marketplace comercial de Microsoft'
description: Recursos técnicos e instrucciones para ayudarle a configurar una oferta de contenedor en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 04/09/2020
ms.openlocfilehash: 46102305e99c94fc71ad0934a2c0063fdcbeec4e
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87317797"
---
# <a name="create-an-azure-container-offer"></a>Creación de una oferta de contenedor de Azure

En este artículo se proporcionan recursos técnicos y recomendaciones para ayudarle a crear una oferta de contenedor en Azure Marketplace.

## <a name="before-you-begin"></a>Antes de empezar

Para ver guías de inicio rápido, tutoriales y ejemplos, consulte [Documentación de Azure Container Instances](../../container-instances/index.yml).

## <a name="fundamental-technical-knowledge"></a>Conocimientos técnicos básicos

El diseño, la compilación y las pruebas de estos recursos lleva tiempo y requiere conocimientos técnicos de la plataforma de Azure y las tecnologías que se usan para crear la oferta.

Además del dominio de la solución, el equipo de ingeniería debe tener conocimientos sobre las siguientes tecnologías de Microsoft:

- Conocimientos básicos de los [Servicios de Azure](https://azure.microsoft.com/services/).
- Cómo [diseñar y estructurar las aplicaciones de Azure](https://azure.microsoft.com/solutions/architecture/).
- Conocimientos prácticos de [Azure Virtual Machines](https://azure.microsoft.com/services/virtual-machines/), [Azure Storage](https://azure.microsoft.com/services/?filter=storage) y [redes de Azure](https://azure.microsoft.com/services/?filter=networking).
- Conocimientos prácticos de [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/).
- Conocimientos prácticos de [JSON](https://www.json.org/).

## <a name="suggested-tools"></a>Herramientas sugeridas

Elija uno o ambos de los entornos de scripting siguientes para ayudar a administrar la imagen de contenedor:

- [Azure PowerShell](https://docs.microsoft.com/powershell/azure/?view=azps-3.7.0&viewFallbackFrom=azps-3.6.1)
- [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest).

Se recomienda agregar estas herramientas al entorno de desarrollo:

- [Explorador de Azure Storage](../../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows)
- [Visual Studio Code](https://code.visualstudio.com/)
  - Extensión: [Herramientas de Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)
  - Extensión: [Beautify](https://marketplace.visualstudio.com/items?itemName=HookyQR.beautify)
  - Extensión: [Prettify JSON](https://marketplace.visualstudio.com/items?itemName=mohsen1.prettify-json).

Revise las herramientas disponibles en la página [Herramientas de desarrollo de Azure](https://azure.microsoft.com/). Si usa Visual Studio, revise las herramientas disponibles en [Visual Studio Marketplace](https://marketplace.visualstudio.com/).

## <a name="create-the-container-image"></a>Creación de la imagen de contenedor

Para obtener más información, consulte los siguientes tutoriales:

- [Tutorial: Creación de una imagen de contenedor para su implementación en Azure Container Instances](../../container-instances/container-instances-tutorial-prepare-app.md)
- [Tutorial: Compilación e implementación de imágenes de contenedor en la nube con Azure Container Registry Tasks](../../container-registry/container-registry-tutorial-quick-task.md).

## <a name="next-steps"></a>Pasos siguientes

- [Creación de la oferta de contenedor](create-azure-container-offer.md).
