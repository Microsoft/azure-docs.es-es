---
title: 'Diseño de identidades híbridas: requisitos de respuesta ante incidentes en Azure | Microsoft Docs'
description: Determine las funcionalidades de supervisión e informes de la solución de identidad híbrida que puede aprovechar el responsable de TI para realizar acciones de cara a identificar y mitigar las posibles amenazas.
documentationcenter: ''
services: active-directory
author: billmath
manager: daveba
editor: ''
ms.assetid: a3d2a459-599b-4b67-8e51-7369ee25082d
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/18/2017
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8c68be4e46693621f77cb25dd8f34f8e0b7d75dd
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89278418"
---
# <a name="determine-incident-response-requirements-for-your-hybrid-identity-solution"></a>Determinación de los requisitos de respuesta a incidentes para su solución de identidad híbrida
La mayoría de las organizaciones grandes y medianas contarán probablemente con una [respuesta a los incidentes de seguridad](/previous-versions/tn-archive/cc700825(v=technet.10)) para ayudar al responsable de TI a emprender acciones en función del nivel de incidente. El sistema de administración de identidades es un componente importante en el proceso de respuesta a incidentes, ya que se puede usar para ayudar a identificar quién ha realizado una acción específica en el destino. La solución de identidad híbrida debe ser capaz de proporcionar funcionalidades de supervisión e informes que TI puede aprovechar para realizar acciones con el fin de identificar y mitigar las posibles amenazas. En un plan de respuesta a incidentes típico tendrá las siguientes fases:

1. Evaluación inicial.
2. Comunicación de incidentes
3. Control de daños y reducción de riesgos
4. Identificación de lo que se ha puesto en peligro y la gravedad
5. Conservación de evidencias
6. Notificación a las partes interesadas
7. Recuperación del sistema
8. Documentación.
9. Evaluación de daños y costos
10. Revisión del proceso y el plan.

Durante la fase de identificación de lo que se pone en peligro y la gravedad, será necesario identificar los sistemas que se han puesto en peligro, los archivos a los que se ha obtenido acceso y determinar la confidencialidad de esos archivos. El sistema de identidad híbrida debe ser capaz de satisfacer esos requisitos para ayudarle a identificar el usuario que realizó dichos cambios. 

## <a name="monitoring-and-reporting"></a>Supervisión e informes
En muchas ocasiones, el sistema de identidad híbrida puede ayudar en la fase de evaluación inicial, en especial si lleva integradas funcionalidades de auditoría e informes. Durante la evaluación inicial, el administrador de TI debe poder identificar una actividad sospechosa o el sistema debe ser capaz de desencadenarla automáticamente según una tarea configurada previamente. Muchas actividades podrían indicar un posible ataque; sin embargo, en otros casos, un sistema mal configurado puede conducir a varios falsos positivos en un sistema de detección de intrusiones. 

El sistema de administración de identidades debe ayudar a los administradores de TI a identificar y notificar esas actividades sospechosas. Por lo habitual, estos requisitos técnicos se pueden satisfacer con la supervisión de todos los sistemas y una funcionalidad de informes que pueda resaltar las posibles amenazas. Las siguientes preguntas le ayudarán a diseñar la solución de identidad híbrida y a tener en cuenta los requisitos de respuesta a incidentes:

* ¿Tiene su empresa una respuesta a incidentes de seguridad?
  * En caso afirmativo, ¿se usa el sistema de administración de identidades actual como parte del proceso?
* ¿Necesita su empresa identificar los intentos de inicio de sesión sospechosos de los usuarios en distintos dispositivos?
* ¿Necesita su empresa detectar las credenciales del usuario que han podido verse comprometidas?
* ¿Necesita su empresa auditar el acceso y la acción del usuario?
* ¿Su empresa necesita saber cuándo restablece un usuario su contraseña?

## <a name="policy-enforcement"></a>Aplicación de directivas
Durante la fase de control de daños y reducción de riesgos, es importante reducir rápidamente los efectos reales y posibles de un ataque. Esa acción que realice en este punto puede marcar la diferencia entre un daño menor o uno mayor. La respuesta exacta dependerá de su organización y de la naturaleza del ataque al que se enfrenta. Si la evaluación inicial concluye que se ha puesto en peligro una cuenta, deberá aplicar la directiva para bloquear esta cuenta. Este es solo un ejemplo de dónde se puede aprovechar el sistema de administración de identidades. Las siguientes preguntas le ayudarán a diseñar la solución de identidad híbrida y a tener en cuenta el modo en que se aplicarán las directivas para reaccionar a un incidente en curso:

* ¿Dispone su empresa de directivas que impidan a los usuarios el acceso a la red si es necesario?
  * En caso afirmativo, ¿se integra la solución actual con el sistema de administración de identidad híbrida que va a adoptar?
* ¿Su empresa necesita aplicar acceso condicional a los usuarios que están en cuarentena? 

> [!NOTE]
> Asegúrese de anotar cada respuesta y de que comprende las razones que se esconden detrás. [Definición de la estrategia de protección de datos](plan-hybrid-identity-design-considerations-data-protection-strategy.md) se recorren las opciones disponibles y las ventajas y desventajas de cada una.  Las respuestas que obtenga partir de estas preguntas le servirán para seleccionar la opción que mejor se adapte a sus necesidades empresariales.
> 
> 

## <a name="next-steps"></a>Pasos siguientes
[Definición de la estrategia de protección de datos](plan-hybrid-identity-design-considerations-data-protection-strategy.md)

## <a name="see-also"></a>Consulte también
[Información general sobre las consideraciones de diseño](plan-hybrid-identity-design-considerations-overview.md)