---
title: 'Ejemplo de script de la CLI de Azure: publicación de un recurso | Microsoft Docs'
description: En este artículo se muestra cómo utilizar el script de la CLI de Azure para publicar un recurso.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: ''
ms.service: media-services
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-azurecli
ms.openlocfilehash: 3b3c358a84dd74595c476f029a1c8f28bc3c901f
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89295876"
---
# <a name="cli-example-publish-an-asset"></a>Ejemplo de CLI: publicación de un recurso

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

El script de la CLI de Azure de este artículo muestra cómo crear un localizador de streaming y regresar a las direcciones URL de streaming. 

## <a name="prerequisites"></a>Prerequisites 

[Cree una cuenta de Media Services](./create-account-howto.md).

[!INCLUDE [media-services-cli-instructions.md](../../../includes/media-services-cli-instructions.md)]

## <a name="example-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/media-services/publish-asset/Publish-Asset.sh "Publish an asset")]

## <a name="next-steps"></a>Pasos siguientes

[Introducción a Media Services](media-services-overview.md)
