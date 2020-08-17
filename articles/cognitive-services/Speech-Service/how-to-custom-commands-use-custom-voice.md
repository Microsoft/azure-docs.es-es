---
title: 'Instrucciones: Uso de comandos personalizados con Voz personalizada: servicio de voz'
titleSuffix: Azure Cognitive Services
description: En este artículo, especificará la voz de salida de una aplicación de comandos personalizados.
services: cognitive-services
author: singhsaumya
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/16/2020
ms.author: sausin
ms.openlocfilehash: b1e488653f210089ee5ed6757fb2f3d1e4fb0fe4
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293865"
---
# <a name="use-custom-commands-with-custom-voice"></a>Uso de Comandos personalizados con Voz personalizada

En este artículo, obtendrá información sobre cómo seleccionar una voz de salida personalizada para una aplicación de comandos personalizados.

## <a name="select-a-custom-voice"></a>Selección de una voz personalizada

1. En la aplicación de Comandos personalizados, seleccione **Configuración** en el panel izquierdo.
1. Seleccione **Voz personalizada** en el panel central.
1. Seleccione la voz pública o personalizada que desee en la tabla.
1. Seleccione **Guardar**.

> [!div class="mx-imgBorder"]
> ![Oraciones de ejemplo con parámetros](media/custom-commands/select-custom-voice.png)

> [!NOTE]
> - Para **Public voices** (Voces públicas), **Neural types** (Tipos neuronales) solo está disponible para regiones específicas. Para comprobar la disponibilidad, consulte las [voces estándar y neuronales por región o punto de conexión](https://docs.microsoft.com/azure/cognitive-services/speech-service/regions#standard-and-neural-voices).
> - Para **Custom voices** (Voces personalizadas), se pueden crear desde la página de proyecto Custom Voice (Voz personalizada). Consulte [Introducción a Voz personalizada](./how-to-custom-voice.md).

Ahora la aplicación responderá en la voz seleccionada, en lugar de en la voz predeterminada.
