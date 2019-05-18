---
title: 'Ejemplo de script de la CLI de Azure: creación de una transformación | Microsoft Docs'
description: Utilice el script de la CLI de Azure para crear una transformación.
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
ms.openlocfilehash: 86f455f25bb41833774bb7ad7d81688aad812b5a
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236817"
---
# <a name="cli-example-create-a-transform"></a>Ejemplo de la CLI: Cree una transformación

El script de la CLI de Azure de este artículo muestra cómo crear una transformación. Las transformaciones describen un flujo de trabajo simple de tareas para procesar los archivos de vídeo o audio (que se conoce habitualmente como "receta"). Siempre debe comprobar si ya existe una transformación con el nombre y la "receta" elegidos. Si es así, debe volver a utilizarlos.

## <a name="prerequisites"></a>Requisitos previos 

[Cree una cuenta de Media Services](create-account-cli-how-to.md).

[!INCLUDE [media-services-cli-instructions.md](../../../includes/media-services-cli-instructions.md)]

## <a name="example-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/media-services/create-transform/Create-Transform.sh "Create a transform")]

## <a name="next-steps"></a>Pasos siguientes

[Introducción a Media Services](media-services-overview.md)
