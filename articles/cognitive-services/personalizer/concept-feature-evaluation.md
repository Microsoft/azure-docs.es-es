---
title: 'Evaluación de características: Personalizer'
titleSuffix: Azure Cognitive Services
description: Al ejecutar una evaluación en el recurso Personalizer desde Azure Portal, este proporciona información sobre las características del contexto y las acciones que influyen en el modelo.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 07/29/2019
ms.openlocfilehash: c0e47a2943cf8c934d201f76aefc41868adf0b25
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87127730"
---
# <a name="feature-evaluation"></a>Evaluación de características

Al ejecutar una evaluación en el recurso Personalizer desde [Azure Portal](https://portal.azure.com), este proporciona información sobre las características del contexto y las acciones que influyen en el modelo. 

Esto resulta útil para:

* Imaginar otras características que podría usar, inspirándose a partir de las características más importantes del modelo.
* Ver qué características no son importantes, con la posibilidad de eliminarlas o de analizar con mayor profundidad lo que puede afectar al uso.
* Ofrecer orientación a los equipos de edición o protección sobre los nuevos contenidos o productos que merece la pena incluir en el catálogo.
* Solucionar problemas y errores comunes que se producen al enviar características a Personalizer.

Las características más importantes tienen un mayor peso en el modelo. En consecuencia, suelen estar presentes cuando Personalizer obtiene una mayor recompensa.

## <a name="getting-feature-importance-evaluation"></a>Evaluación de la importancia de las características

Para ver los resultados de importancia de las características, debe ejecutarse una evaluación. La evaluación crea etiquetas de características legibles para el usuario que se basan en los nombres de las características observadas durante el período de evaluación.

La información resultante sobre la importancia de las características representa el modelo en línea actual de Personalizer. La evaluación analiza la importancia de las características del modelo guardado en la fecha de finalización del período de evaluación, después de llevar a cabo todo el entrenamiento realizado durante la evaluación, con la directiva de aprendizaje en línea actual. 

Los resultados de la importancia de las características no representan otras directivas ni modelos probados o creados durante la evaluación.  La evaluación no incluirá características enviadas a Personalizer una vez que finalice el período de evaluación.

## <a name="how-to-interpret-the-feature-importance-evaluation"></a>Interpretación de la evaluación de la importancia de las características

Para evaluar las características, Personalizer crea "grupos" de características que tienen una importancia similar. Se puede decir que un grupo tiene una importancia total mayor que otros, pero, dentro del grupo, el orden de las características es alfabético.

La información sobre cada característica incluye:

* Si la característica procede de contexto o de acciones.
* Clave y valor de la característica.

Por ejemplo, una aplicación de pedido de helados puede ver "Context.Weather.Hot" como una característica muy importante.

Personalizer muestra las correlaciones de las características que, cuando se tienen en cuenta en su conjunto, producen mayores recompensas.

Por ejemplo, puede ver "Context.Weather:Hot *con* Action.MenuItem:IceCream" y "Context.Weather:Cold *con* Action.MenuItem:WarmTea:".

## <a name="actions-you-can-take-based-on-feature-evaluation"></a>Acciones pueden realizarse en función de la evaluación de características

### <a name="imagine-additional-features-you-could-use"></a>Imaginar otras características que podrían usarse

Obtenga inspiración de las características más importantes del modelo. Por ejemplo, si ve "Context.MobileBattery:Low" en una aplicación móvil de vídeo, puede pensar que el tipo de conexión también puede influir de modo que los clientes decidan ver un vídeo tras otro. Por tanto, pueden agregar a la aplicación características sobre el tipo de conectividad y el ancho de banda.

### <a name="see-what-features-are-not-important"></a>Ver qué características no son importantes

Puede eliminar las características que no son importantes o analizar con mayor profundidad los aspectos que pueden afectar al uso. Las características pueden tener una clasificación baja por muchas razones. Una podría ser que realmente la característica no influya en el comportamiento del usuario. Pero también podría significar que la característica no es visible para el usuario. 

Por ejemplo, un sitio de vídeos podría ver que "Action.VideoResolution=4k" es una característica de poca importancia, lo que contradice la búsqueda de los usuarios. La causa podría ser que la aplicación ni siquiera menciona o muestra la resolución de vídeo, por lo que los usuarios no cambiarán su comportamiento en función de la resolución.

### <a name="provide-guidance-to-editorial-or-curation-teams"></a>Ofrecer orientación a los equipos de edición y protección

Ofrezca orientación sobre los nuevos contenidos o productos que merece la pena incluir en el catálogo. Personalizer está diseñado para ser una herramienta que aumenta la información que los equipos tienen sobre los usuarios. Una forma de hacerlo es proporcionar información a los grupos de edición sobre qué aspectos de los productos, artículos o contenidos impulsan el comportamiento. Por ejemplo, el escenario de aplicación de vídeo puede mostrar que hay una característica importante denominada "Action.VideoEntities.Cat:true", lo que induce al equipo editorial a incorporar más vídeos de gatos.

### <a name="troubleshoot-common-problems-and-mistakes"></a>Solucionar problemas y errores comunes

Los problemas y errores comunes se pueden corregir con un cambio en el código de la aplicación, de manera que no se envíen características inadecuadas o con formato incorrecto a Personalizer. 

Entre los errores comunes que se producen al enviar características se incluyen los siguientes:

* Envío de información de identificación personal. Personalizer no debe usarse con información de identificación personal específica de un individuo (como el nombre, el número de teléfono, los números de tarjetas de crédito y las direcciones IP). Si su aplicación necesita realizar un seguimiento de los usuarios, use un UUID que no permita la identificación o algún otro número de id. de usuario. En la mayoría de los escenarios, esto también resulta problemático.
* Con un gran número de usuarios, es improbable que la interacción de cada uno tenga un peso mayor que la interacción de toda la población, por lo que el envío de identificadores de usuario probablemente agregue más ruido que valor al modelo.
* Enviar campos de fecha y hora como marcas de tiempo precisas en lugar de valores de tiempo caracterizados. Características como Context.TimeStamp.Day=Monday o "Context.TimeStamp.Hour"="13" son más útiles. Habrá como máximo 7 o 24 valores de características para cada uno. Pero "Context.TimeStamp":"1985-04-12T23:20:50.52Z" es tan preciso que no habrá manera de aprender de ese valor, porque nunca volverá a ocurrir.

## <a name="next-steps"></a>Pasos siguientes

Entender la [escalabilidad y el rendimiento](concepts-scalability-performance.md) con Personalizer.

