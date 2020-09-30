---
title: 'Tutorial de respuesta a incidentes: Azure Security Center'
description: En este tutorial aprenderá a realizar una evaluación de prioridades de las alertas de seguridad, a determinar la causa raíz y el ámbito de un incidente, y a buscar datos de seguridad.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 181e3695-cbb8-4b4e-96e9-c4396754862f
ms.service: security-center
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/30/2018
ms.author: memildin
ms.openlocfilehash: c508591fe9909578dcc04b0922c0b76691898743
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90901092"
---
# <a name="tutorial-respond-to-security-incidents"></a>Tutorial: Respuesta a incidentes de seguridad
Security Center analiza continuamente las cargas de trabajo de la nube híbrida y usa el análisis avanzado e inteligencia de amenazas para alertarle de cualquier actividad malintencionada. Además, en Security Center se pueden integrar alertas de otros servicios y productos de seguridad, así como crear alertas personalizadas basadas en indicadores u orígenes de inteligencia propios. Una vez que se genera una alerta, es necesario investigarla y tomar las medidas correctoras de inmediato. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Evaluar las prioridades de las alertas de seguridad
> * Investigar en profundidad para determinar la causa raíz y el alcance de un incidente de seguridad
> * Buscar en los datos de seguridad para facilitar la investigación

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos
Para recorrer las características descritas en este tutorial, debe tener habilitado Azure Defender. Puede probar Azure Defender sin costo alguno. Para más información, consulte la [página de precios](https://azure.microsoft.com/pricing/details/security-center/). La guía de inicio rápido [Introducción a Security Center](security-center-get-started.md) le guiará en la actualización.

## <a name="scenario"></a>Escenario
Contoso migró recientemente algunos de sus recursos locales a Azure, incluidas algunas cargas de trabajo empresariales basadas en máquinas virtuales y bases de datos SQL. Actualmente el equipo de respuesta a incidentes de seguridad informática central de Contoso (CSIRT) tiene un problema para investigar asuntos de seguridad, debido a la falta de inteligencia de seguridad integrada en sus herramientas actuales de respuesta a incidentes. Esta falta de integración presenta un problema durante las fases de detección (demasiados falsos positivos), evaluación y diagnóstico. Como parte de esta migración, decidió participar en Security Center para ayudar a solucionar el problema.

La primera fase de migración terminó después de incorporar todos los recursos y abordar todas las recomendaciones de seguridad de Security Center. El equipo CSIRT de Contoso es crucial para tratar los incidentes de seguridad informática. El equipo está formado por un grupo de personas encargadas de manejar los incidentes de seguridad. Los miembros del equipo tienen tareas claramente definidas para asegurarse de que no quede sin cubrir ninguna área de respuesta.

En este escenario, nos vamos a centrar en los roles de las siguientes personas que forman parte del equipo CSIRT de Contoso:

![Ciclo de vida de respuesta a incidentes](./media/tutorial-security-incident/security-center-incident-response.png)

Judy se encarga de las operaciones de seguridad. Entre sus responsabilidades están:

* Supervisar y responder a amenazas de seguridad durante las 24 horas.
* Remitirlas al propietario de la carga de trabajo en la nube o al analista de seguridad, según sea necesario.

Sam es analista de seguridad y sus responsabilidades incluyen:

* Investigar los ataques.
* Solucionar las alertas.
* Trabajar con los propietarios de la carga de trabajo para determinar y aplicar mitigaciones.

Como puede ver, Judy y Sam tienen responsabilidades diferentes, y deben trabajar juntos para compartir la información de Security Center.

## <a name="triage-security-alerts"></a>Evaluar las prioridades de las alertas de seguridad
Security Center proporciona una vista unificada de todas las alertas de seguridad. Las alertas de seguridad se clasifican en función de su gravedad y cuando se combinan varias posibles alertas relacionadas, se genera un incidente de seguridad. Cuando la evolución de las prioridades de las alertas e incidentes debería:

- Descartar las alertas para las que no se necesitan acciones adicionales, por ejemplo, si la alerta es un falso positivo
- Actuar para corregir los ataques conocidos, por ejemplo bloqueando el tráfico de red procedente de una dirección IP malintencionada
- Determinar las alertas que requieren una posterior investigación


1. En el menú principal de Security Center, en **DETECCIÓN**, seleccione **Alertas de seguridad**:

   ![Alertas de seguridad](./media/tutorial-security-incident/tutorial-security-incident-fig1.png)

2. En la lista de alertas, seleccione un incidente de seguridad, que es una colección de alertas, para más información acerca de este incidente. Se abre **Security incident detected** (Incidente de seguridad detectado).

   ![Security incident detected (Incidente de seguridad detectado)](./media/tutorial-security-incident/tutorial-security-incident-fig2.png)

3. En esta pantalla, la descripción del incidente de seguridad se encuentra en la parte superior y en el resto la lista de alertas que forman parte de este incidente. Para más información, haga clic en la alerta que desea investigar en profundidad.

   ![Detalles de alerta del incidente](./media/tutorial-security-incident/tutorial-security-incident-fig3.png)

   El tipo de alerta puede variar. Para más información acerca del tipo de alerta y los posibles pasos para corregirla, lea el artículo [Comprensión de las alertas de seguridad en Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-alerts-type). En las alertas que se puedan descartar de forma segura puede hacer clic con el botón derecho en la alerta y seleccione la opción **Descartar**:

   ![Alerta](./media/tutorial-security-incident/tutorial-security-incident-fig4.png)

4. Si no se conocen la causa raíz ni el alcance de la actividad malintencionada, continúe con el paso siguiente para investigar la alerta en profundidad.

## <a name="investigate-an-alert-or-incident"></a>Investigación de una alerta o un incidente
1. En la página **Alerta de seguridad**, haga clic en el botón **Iniciar investigación** (si ya la ha iniciado, el nombre cambia a **Continuar investigación**).

   ![Investigación](./media/tutorial-security-incident/tutorial-security-incident-fig5.png)

   El mapa de la investigación es una representación gráfica de las entidades que conectadas a esta alerta o incidente de seguridad. Al hacer clic en cualquiera de las entidades en la asignación, la información acerca de ella mostrará nuevas entidades, lo que hará que el mapa se expanda. Las propiedades de la entidad seleccionada en el mapa están resaltadas en el panel del lado derecho de la página. La información disponible en cada pestaña variará en función de la entidad seleccionada. Durante el proceso de investigación, lea toda la información pertinente para conocer mejor el movimiento del atacante.

2. Si necesita más pruebas o debe investigar más las entidades que se encontraron durante la investigación, continúe con el paso siguiente.

## <a name="search-data-for-investigation"></a>Búsqueda de datos para la investigación

Las funcionalidades de búsqueda de Security Center se pueden usar no solo para encontrar más evidencia de sistemas en peligro sino también para obtener más detalles acerca de las entidades que forman parte de la investigación.

Para realizar una búsqueda, abra el panel de **Security Center**, haga clic en **Buscar** en el panel de navegación izquierdo, seleccione el área de trabajo que contenga las entidades que desea buscar, escriba la consulta de la búsqueda y haga clic en el botón Buscar.

## <a name="clean-up-resources"></a>Limpieza de recursos

Otras guías de inicio rápido y tutoriales de esta colección se basan en los valores de esta. Si tiene previsto seguir trabajando con las guías de inicio rápido y los tutoriales posteriores, mantenga el aprovisionamiento automático y Azure Defender habilitados. Si no tiene previsto continuar o desea deshabilitar Azure Defender:

1. Vuelva al menú principal de Security Center y seleccione **Precios y configuración**.
1. Seleccione la suscripción para la que desee cambiar a una versión anterior.
1. Establezca **Azure Defender** en Off (Desactivado).
1. Seleccione **Guardar**.

Si desea deshabilitar el aprovisionamiento automático:

1. Vuelva al menú principal de Security Center y seleccione **Directiva de seguridad**.
2. Seleccione la suscripción en la que quiere deshabilitar el aprovisionamiento automático.
3. En **Directiva de seguridad: Colección de datos**, en **Incorporación**, seleccione **Desactivado** para deshabilitar el aprovisionamiento automático.
4. Seleccione **Guardar**.

>[!NOTE]
> La deshabilitación del aprovisionamiento automático no quita el agente de Log Analytics de las máquinas virtuales de Azure en las que se ha aprovisionado. La deshabilitación del aprovisionamiento automático limita la supervisión de seguridad de los recursos.
>

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha obtenido información acerca de las características de Security Center que se deben usar cuando se responde a un incidente de seguridad, como:

> [!div class="checklist"]
> * Incidente de seguridad, que es una agregación de las alertas relacionadas de un recurso
> * Mapa de la investigación, que es una representación gráfica de las entidades que conectadas a un alerta o incidente de seguridad
> * Capacidades de búsqueda, para buscar más evidencia de los sistemas en peligro