---
title: 'Ejemplo de script de la CLI de Azure: creación de una transformación | Microsoft Docs'
description: Las transformaciones describen un flujo de trabajo de tareas simple para procesar archivos de vídeo o audio (que se conoce habitualmente como "receta"). El script de la CLI de Azure de este artículo muestra cómo crear una transformación.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: ''
ms.service: media-services
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 05/01/2019
ms.author: juliako
ms.custom: devx-track-azurecli
ms.openlocfilehash: 538f48d427a4d8b51f77ae50bb0bee95909f0b09
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494475"
---
# <a name="cli-example-create-a-transform"></a>Ejemplo de la CLI: creación de una transformación

El script de la CLI de Azure de este artículo muestra cómo crear una transformación. Las transformaciones describen un flujo de trabajo de tareas simple para procesar archivos de vídeo o audio (que se conoce habitualmente como "receta"). Siempre debe comprobar si ya existe una transformación con el nombre y la "receta" elegidos. Si es así, debe volver a utilizarlos.

## <a name="prerequisites"></a>Prerequisites 

[Cree una cuenta de Media Services](./create-account-howto.md).

[!INCLUDE [media-services-cli-instructions.md](../../../includes/media-services-cli-instructions.md)]

> [!NOTE]
> Solo puede especificar una ruta de acceso a un archivo JSON preestablecido del codificador estándar personalizado para [StandardEncoderPreset](/rest/api/media/transforms/createorupdate#standardencoderpreset), consulte el ejemplo [Codificación con una transformación personalizada](custom-preset-cli-howto.md).
>
> No se puede pasar un nombre de archivo cuando se usa [BuiltInStandardEncoderPreset](/rest/api/media/transforms/createorupdate#builtinstandardencoderpreset).

## <a name="example-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/media-services/create-transform/Create-Transform.sh "Create a transform")]

## <a name="next-steps"></a>Pasos siguientes

[az ams transform (CLI)](/cli/azure/ams/transform?view=azure-cli-latest)
