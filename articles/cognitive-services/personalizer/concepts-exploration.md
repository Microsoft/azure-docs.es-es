---
title: 'Exploración: Personalizer'
titleSuffix: Azure Cognitive Services
description: Con la exploración, Personalizer puede continuar ofreciendo buenos resultados, incluso a medida que cambia el comportamiento del usuario. Elegir una configuración de exploración es una decisión empresarial sobre la proporción de interacciones de los usuarios con las que explorar a fin de mejorar el modelo.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 10/23/2019
ms.openlocfilehash: 00b9cc881d9414e957d7503bb6156095bb821c06
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87128155"
---
# <a name="exploration-and-exploitation"></a>Exploración y explotación

Con la exploración, Personalizer puede continuar ofreciendo buenos resultados, incluso a medida que cambia el comportamiento del usuario.

Cuando Personalizer recibe una llamada de Rank, devuelve un RewardActionID que hace una de estas acciones:
* Utiliza la explotación para buscar una coincidencia con el comportamiento más probable del usuario en función del modelo de Machine Learning actual.
* Utiliza la exploración, que no coincide con la acción que tiene la mayor probabilidad en la clasificación.

Personalizer usa actualmente un algoritmo llamado *epsilon-greedy* para explorar. 

## <a name="choosing-an-exploration-setting"></a>Selección de un ajuste de exploración

Configure el porcentaje de tráfico para usar en la exploración en la página **Configuración** de Azure Portal para Personalizer. Esta configuración determina el porcentaje de llamadas a Rank que realiza la exploración. 

Personalizer determina si explorar o explotar con esta probabilidad en cada llamada a Rank. Esto es diferente del comportamiento en algunos marcos A/B que bloquean un tratamiento en determinados identificadores de usuario.

## <a name="best-practices-for-choosing-an-exploration-setting"></a>Procedimiento recomendado para seleccionar un ajuste de exploración

Elegir una configuración de exploración es una decisión empresarial sobre la proporción de interacciones de los usuarios con las que explorar a fin de mejorar el modelo. 

Un valor de cero anularía muchas de las ventajas de Personalizer. Con esta configuración, Personalizer no utiliza las interacciones del usuario para detectar las mejores interacciones del usuario. Esto conduce al estancamiento del modelo, un desfase y, en última instancia, un menor rendimiento.

Un valor demasiado alto anularía las ventajas del aprendizaje a partir de comportamiento del usuario. Establecerlo al 100 % implica una aleatorización constante, y cualquier comportamiento aprendido de los usuarios no influiría en el resultado.

Es importante no cambiar el comportamiento de la aplicación en función de si Personalizer está explorando o explotando. Esto llevaría a sesgos de aprendizaje que en última instancia disminuirían el rendimiento potencial.

## <a name="next-steps"></a>Pasos siguientes

[Aprendizaje de refuerzo](concepts-reinforcement-learning.md) 