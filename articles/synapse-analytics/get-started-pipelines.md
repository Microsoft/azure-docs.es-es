---
title: 'Tutorial: Introducción a la organización con canalizaciones'
description: En este tutorial, aprenderá a organizar canalizaciones y actividades mediante Synapse Studio.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.date: 07/20/2020
ms.openlocfilehash: 5e32a6a9817f2a3176e96e39c5e261875e8f4ed1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87093794"
---
# <a name="orchestrate-with-pipelines"></a>Organización con canalizaciones

En este tutorial, aprenderá a organizar canalizaciones y actividades mediante Synapse Studio. 

## <a name="overview"></a>Información general

Puede organizar una gran variedad de tareas en Azure Synapse.

1. En Synapse Studio, vaya al centro **Orchestrate** (Organización).
1. Seleccione **+**  > **Canalización** para crear una canalización.
1. Vaya al centro **Develop** (Desarrollo) y busque el cuaderno que creó anteriormente.
1. Arrastre el cuaderno a la canalización.
1. En la canalización, seleccione **Agregar desencadenador** > **Nuevo/editar**.
1. En **Choose trigger** (Elegir desencadenador), seleccione **Nuevo** y, después, en el apartado en que se elige la **periodicidad**, configure el desencadenador para que se ejecute cada hora.
1. Seleccione **Aceptar**.
1. Seleccione **Publish All** (Publicar todo). La canalización se ejecuta cada hora.
1. Para que la canalización se ejecute ahora sin esperar a la próxima hora, seleccione **Agregar desencadenador** > **Nuevo/editar**.



## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Visualización de datos con Power BI](get-started-visualize-power-bi.md)
                                 
