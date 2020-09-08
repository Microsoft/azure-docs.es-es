---
title: 'Inicio rápido: Creación y uso de bucles de aprendizaje con el SDK: Personalizer'
description: En este inicio rápido se muestra cómo crear y administrar la base de conocimiento mediante la biblioteca cliente de Personalizer.
ms.topic: quickstart
ms.date: 08/27/2020
ms.custom: cog-serv-seo-aug-2020
keywords: personalizer, Azure personalizer, machine learning
zone_pivot_groups: programming-languages-set-six
ms.openlocfilehash: dccdec0888f2968fb7089c4ff80c9338215de135
ms.sourcegitcommit: 420c30c760caf5742ba2e71f18cfd7649d1ead8a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89055973"
---
# <a name="quickstart-personalizer-client-library"></a>Inicio rápido: Biblioteca cliente de Personalizer

Muestre contenido personalizado en este inicio rápido con el servicio Personalizer.

Comience a trabajar con la biblioteca cliente de Personalizer. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas.

 * La API Rank: selecciona el mejor de todos los elementos de las acciones, en función de la información en tiempo real que se proporciona sobre el contenido y el contexto.
 * API Reward: determine la puntuación de recompensa en función de sus necesidades empresariales y, luego, envíela a Personalizer con esta API. Esa puntuación puede ser un solo valor, por ejemplo, 1 para bueno y 0 para malo, o un algoritmo que cree en función de sus necesidades empresariales.

::: zone pivot="programming-language-csharp"
[!INCLUDE [Get intent with C# SDK](./includes/quickstart-sdk-csharp.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [Get intent with Node.js SDK](./includes/quickstart-sdk-nodejs.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Get intent with Python SDK](./includes/quickstart-sdk-python.md)]
::: zone-end

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Cómo funciona Personalizer](how-personalizer-works.md)
> [Dónde usar Personalizer](where-can-you-use-personalizer.md)
