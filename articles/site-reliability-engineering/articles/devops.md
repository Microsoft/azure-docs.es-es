---
title: 'PREGUNTAS MÁS FRECUENTES: SRE y DevOps | Microsoft Docs'
titleSuffix: Azure
description: 'PREGUNTAS MÁS FRECUENTES: Descripción de la relación entre SRE y DevOps'
author: dnblankedelman
manager: ScottCa
ms.service: site-reliability-engineering
ms.topic: article
ms.date: 09/14/2020
ms.author: dnb
ms.openlocfilehash: 3d0698f2780a4ccc41eedbde70fddea1766c5f21
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90090336"
---
# <a name="frequently-asked-questions-whats-the-relationship-between-sre-and-devops"></a>Preguntas más frecuentes: ¿Cuál es la relación entre SRE y DevOps?

Hay un conjunto de preguntas comunes que surgen en torno a la relación entre SRE (ingeniería de confiabilidad de sitios) y DevOps, entre otras: "¿En qué se parecen? ¿En qué se diferencian? ¿Podemos tener ambas en nuestra organización?". En este artículo se intenta compartir algunas de las respuestas que han ofrecido las comunidades de SRE y DevOps que nos acercan a comprender esta relación.

## <a name="how-are-they-the-same"></a>¿En qué se parecen?

Tanto SRE como DevOps son prácticas de operaciones modernas que se crearon y desarrollaron en respuesta a ciertos desafíos, entre ellos:

- una complejidad cada vez mayor de los entornos de producción y los procesos de desarrollo
- un aumento de la dependencia empresarial en el funcionamiento continuo de esos entornos
- la incapacidad de escalar el personal de forma lineal con el tamaño de estos entornos
- la necesidad de moverse más rápido a la vez que conserva la estabilidad operativa

Ambas prácticas de operaciones valoran la atención a los asuntos que son cruciales para abordar estos desafíos, como la supervisión u observabilidad, la automatización, la documentación y las herramientas colaborativas de desarrollo de software.

Entre SRE y DevOps existe una superposición considerable en cuanto a las herramientas y áreas de trabajo. Como lo expresa _The Site Reliability Workbook_, "SRE cree en lo mismo que DevOps, pero para razones ligeramente diferentes".

## <a name="three-different-ways-to-compare-the-two-operations-practices"></a>Tres maneras distintas de comparar las dos prácticas de operaciones

Las similitudes entre SRE y DevOps son claras. Lo que resulta realmente interesante es en qué se difieren o divergen estas dos. Aquí se ofrecen tres maneras de pensar en su relación como forma de aportar cierto matiz a esta pregunta. Puede que no esté de acuerdo con estas respuestas, pero cada una ofrece un buen punto de partida para la discusión.

### <a name="class-sre-implements-interface-devops"></a>"la clase SRE implementa la interfaz DevOps"

_The Site Reliability Workbook_ (que se menciona en nuestra [lista de libros de recursos](../resources/books.md)) describe SRE y DevOps en el primer capítulo. En ese capítulo se usa la frase "la clase SRE implementa la interfaz DevOps" como subtítulo. Esto tiene como fin sugerir (mediante una frase dirigida a los desarrolladores) que SRE podría considerarse una implementación específica de la filosofía de DevOps. Como se destaca en el capítulo, "DevOps es relativamente silenciosa en la manera de ejecutar operaciones a nivel detallado", mientras que SRE tiene prácticas considerablemente más excluyentes. Por tanto, una posible respuesta a la pregunta de cómo se relacionan estas dos es que SRE puede considerarse como una de las muchas implementaciones posibles de DevOps.

### <a name="sre-is-to-reliability-as-devops-is-to-delivery"></a>SRE es a la confiabilidad lo que DevOps es a la entrega

Esta comparación es un poco farragosa, ya que existen varias definiciones para SRE y DevOps, pero puede seguir resultando útil. Comienza con la pregunta "Si tuviera que sintetizar cada práctica de operaciones en una o dos palabras que reflejen su preocupación principal, ¿cuál sería?".

Si usamos esta definición de SRE del [centro de ingeniería de confiabilidad de sitios](../index.yml):

> La ingeniería de confiabilidad de sitios es una disciplina de ingeniería que se dedica a ayudar a las organizaciones a lograr un nivel de confiabilidad adecuado en sus sistemas, servicios y productos.

sería fácil decir que la palabra para SRE es "confiabilidad". Que esté incluida en medio de su nombre también ofrece una excelente evidencia para esta afirmación.

Si usamos esta definición de DevOps del [centro de recursos de Azure DevOps](https://docs.microsoft.com/azure/devops/learn/):

> DevOps es la unión de personas, procesos y productos para hacer posible la entrega continua de valor a los usuarios finales.

una síntesis similar para DevOps podría ser "entrega".

De aquí que "SRE es a la confiabilidad lo que DevOps es a la entrega".

### <a name="direction-of-attention"></a>Dirección de la atención

Esta respuesta está citada o algo parafraseada a partir de una contribución de Thomas Limoncelli al libro _Seeking SRE_, mencionado en nuestra [lista de libros de recursos](../resources/books.md). Limoncelli observa que los ingenieros de DevOps se centran enormemente en la canalización del ciclo de vida de desarrollo de software, con responsabilidades ocasionales en las operaciones de producción, mientras que los ingenieros de SRE se centran en las operaciones de producción, con responsabilidades ocasionales en la canalización de SDLC.

Pero, lo que es más importante, también muestra un diagrama que comienza con el proceso de desarrollo de software en un lado y el trabajo de operaciones de producción en el otro. Los dos están conectados por la canalización habitual que se crea para tomar el código de un desarrollador, someterlo al número deseado de pruebas y fases y, a continuación, pasar el código a producción.

Limoncelli señala que los ingenieros de DevOps comienzan en el entorno de desarrollo y automatizan los pasos hacia la producción. Una vez que se ha completado, retroceden para optimizar los cuellos de botella.

Por otro lado, los ingenieros de SRE se centran en las operaciones de producción y llegan a las profundidades de la canalización como medio para mejorar el resultado final (básicamente trabajan en el sentido opuesto).

Es esta diferencia en el sentido en que SRE y DevOps se centran que puede ayudar a diferenciarlas.

## <a name="coexistence-in-the-same-organization"></a>Coexistencia en la misma organización

La pregunta final que nos gustaría abordar es: "¿Puede tenerse tanto SRE como DevOps en la misma organización?".

La respuesta a esta pregunta es un "¡Sí!" enfático.

Esperamos que las respuestas anteriores ofrezcan una idea de cómo se superponen las dos prácticas de operaciones y, cuando no se superponen, cómo pueden tener enfoques complementarios. Las organizaciones con una práctica DevOps establecida pueden experimentar con las prácticas de SRE a pequeña escala (por ejemplo, probar SLI y SLO) sin tener que comprometerse con la creación de puestos ni equipos de SRE. Este es un patrón de adopción de SRE bastante común.

## <a name="next-steps"></a>Pasos siguientes

¿Le interesa obtener más información acerca de la ingeniería de confiabilidad de sitios o DevOps? Eche un vistazo a nuestro [centro de ingeniería de confiabilidad de sitios](../index.yml) y [centro de recursos de Azure DevOps](https://docs.microsoft.com/azure/devops/learn/).
