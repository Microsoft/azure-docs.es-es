---
title: 'PREGUNTAS MÁS FRECUENTES: SRE y codificación | Microsoft Docs'
titleSuffix: Azure
description: 'PREGUNTAS MÁS FRECUENTES: Descripción de la relación entre SRE y la codificación'
author: dnblankedelman
manager: ScottCa
ms.service: site-reliability-engineering
ms.topic: article
ms.date: 09/14/2020
ms.author: dnb
ms.openlocfilehash: b8865fdd53f4947b17a3621a128fc83f3d93d3e2
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90089078"
---
# <a name="frequently-asked-questions-do-i-need-to-know-how-to-code-to-get-involved-with-sre"></a>Preguntas más frecuentes: ¿Necesito saber cómo codificar para participar con SRE?

Cuando los usuarios están pensando en participar en SRE y los equipos piensan incorporar procedimientos de SRE, una pregunta habitual es "¿Necesita saber cómo codificar?"

La respuesta corta es "sí", 

pero la respuesta completa tiene un poco más de matices. Echemos un vistazo a tres lugares en los que la codificación entra en juego con la ingeniería de confiabilidad de sitios junto con el nivel de experiencia en codificación que se necesita para cada uno. Esta lista no está completa, pero estos escenarios son algunos de los casos de uso más comunes.

## <a name="scenario-1-removing-toil-through-automation"></a>Escenario 1: Eliminación de las tareas tediosas a través de la automatización

Los ingenieros de confiabilidad de sitios y otros usuarios que usan los procedimientos de SRE intentan todo lo posible para eliminar las tareas tediosas. Las "tareas" se refieren a algo específico en SRE. Las tareas tediosas hacen referencia a las operaciones que realiza una persona que tiene ciertas características. No tienen valor compensatorio a largo plazo. No hacen avanzar el servicio de manera significativa. A menudo son repetitivas y extremadamente manuales (aunque se podrían automatizar). A medida que el servicio o los sistemas aumentan con el tiempo, la cantidad de solicitudes de ese sistema también aumentará probablemente de manera proporcional y requerirá incluso un mayor trabajo manual.

Por ejemplo, si un servicio requiere que el equipo de SRE restablezca algo cada semana, aprovisione nuevas cuentas y espacio en disco de forma manual, o bien lo reinicie repetidamente a mano, se trata de una carga operativa que es una tarea tediosa. Completar esas acciones no ha hecho que el servicio sea mejor de ningún modo persistente a largo plazo. Es probable que dichas acciones se tengan que repetir varias veces.

A los SRE no les gustan las tareas tediosas. Trabajan para eliminarlas siempre que sea posible y adecuado. Esta es una de las ubicaciones en las que se produce la automatización en SRE. Si estas solicitudes se pueden controlar automáticamente, se libera el equipo para que trabaje con más recompensas e impacto.

*Experiencia en codificación*: la automatización requiere cierta experiencia en la codificación, pero no es necesario que requiera aptitudes completas de ingeniería de software. Si puede escribir scripts pequeños (quizás en PowerShell o en Bourne Shell) o incluso si crea un objeto [https://docs.microsoft.com/azure/logic-apps/logic-apps-overview](Azure Logic app) con casi cualquier código, esta aplicación de todos modos puede ayudar a eliminar las tareas tediosas.

## <a name="scenario-2-control-through-apisdomain-specific-languages-dslstemplates"></a>Escenario 2: Control a través de API/lenguajes específicos de dominio (DSL)/plantillas

Aunque no es estrictamente necesario para el trabajo de SRE, poder controlar los entornos a través de API, DSL y plantillas (especialmente entornos de nube) permite a que los SRE escalen verticalmente su trabajo. El aprovisionamiento y el desaprovisionamiento de la infraestructura, la configuración de la supervisión y la integración de varios servicios es mucho más eficaz a través de la codificación.

*Experiencia en codificación*: al igual que en el escenario anterior, requiere cierta experiencia en la codificación, pero no es necesario que requiera aptitudes completas de ingeniería de software. Además de los scripts y las aplicaciones lógicas mencionadas anteriormente, también es posible usar [https://docs.microsoft.com/azure/azure-resource-manager/templates/overview](Azure Resource Manager Templates) con un mínimo de experiencia en codificación.

## <a name="scenario-3-fixing-the-code"></a>Escenario 3: Corrección del código

Los ingenieros de confiabilidad de sitios buscan mejorar la confiabilidad de un sistema. Este objetivo a veces requiere profundizar en el código fuente de un sistema, determinar el problema y, a menudo, contribuir con una corrección al código base. Aunque el nivel de sofisticación de este trabajo puede variar considerablemente en función de la situación, la experiencia en codificación es un requisito definitivo en estos casos.

*Experiencia en codificación*: a menudo se requiere experiencia en ingeniería de software completa en este escenario.


## <a name="next-steps"></a>Pasos siguientes

¿Le interesa obtener más información sobre la ingeniería de confiabilidad de sitios y el trabajo con poco código? Revise el [centro de ingeniería de confiabilidad de sitios](../index.yml), la documentación del producto vinculada anteriormente.
