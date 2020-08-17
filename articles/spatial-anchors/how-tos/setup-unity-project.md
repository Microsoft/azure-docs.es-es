---
title: Instalación de Azure Spatial Anchors para Unity
description: Configure un proyecto de Unity para que use Azure Spatial Anchors.
author: craigktreasure
manager: vriveras
services: azure-spatial-anchors
ms.author: crtreasu
ms.date: 07/31/2020
ms.topic: how-to
ms.service: azure-spatial-anchors
ms.openlocfilehash: a23cff8bcef2d62b1e415997ffd6afc80cf47793
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87812262"
---
# <a name="configuring-azure-spatial-anchors-in-a-unity-project"></a>Configuración de Azure Spatial Anchors en un proyecto de Unity

## <a name="requirements"></a>Requisitos

Azure Spatial Anchors admite actualmente Unity 2019.4 (LTS) con las siguientes configuraciones.

* Unity 2019.4 con AR Foundation 3.1 es compatible con Azure Spatial Anchors 2.4.0 y versiones posteriores.

## <a name="configuring-a-project"></a>Configuración de un proyecto

Azure Spatial Anchors para Unity se distribuye actualmente mediante un paquete de recursos de Unity (`.unitypackage`), que se puede encontrar en las [versiones de GitHub](https://github.com/Azure/azure-spatial-anchors-samples/releases).

### <a name="import-the-asset-package"></a>Importación del paquete de recursos

1. Descargue el archivo `AzureSpatialAnchors.unitypackage` de la versión que desea como destino en la página de [versiones de GitHub](https://github.com/Azure/azure-spatial-anchors-samples/releases).
2. Siga las instrucciones que se indican [aquí](https://docs.unity3d.com/Manual/AssetPackagesImport.html) para importar el paquete de recursos de Unity en el proyecto.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Cómo: Creación y localización de delimitadores en Unity](./create-locate-anchors-unity.md)
