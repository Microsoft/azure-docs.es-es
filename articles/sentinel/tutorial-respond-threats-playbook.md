---
title: 'Tutorial: Ejecución de un cuaderno de estrategias en Azure Sentinel'
description: Este tutorial le ayuda a usar cuadernos de estrategias de seguridad en Azure Sentinel para establecer respuestas automatizadas frente a amenazas en problemas relacionados con la seguridad.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: e4afc5c8-ffad-4169-8b73-98d00155fa5a
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/18/2019
ms.author: yelevin
ms.openlocfilehash: f75731fab9a238ffcac2e620235c9d8c5da97549
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88053497"
---
# <a name="tutorial-set-up-automated-threat-responses-in-azure-sentinel"></a>Tutorial: Configuración de respuestas automatizadas frente a amenazas en Azure Sentinel



Este tutorial le ayuda a usar cuadernos de estrategias de seguridad en Azure Sentinel para establecer respuestas automatizadas frente a amenazas en problemas relacionados con la seguridad detectados por Azure Sentinel.


> [!div class="checklist"]
> * Comprender los cuadernos de estrategias
> * Crear un cuaderno de estrategias
> * Ejecutar un cuaderno de estrategias
> * Automatizar las respuestas frente a amenazas


## <a name="what-is-a-security-playbook-in-azure-sentinel"></a>¿Qué es un cuaderno de estrategias de seguridad en Azure Sentinel?

Un cuaderno de estrategias de seguridad es una colección de procedimientos que se pueden ejecutar desde Azure Sentinel en respuesta a una alerta. Un cuaderno de estrategias de seguridad puede ayudar a automatizar y orquestar la respuesta y se puede ejecutar manualmente o establecerse para que se ejecute automáticamente cuando se desencadenan alertas específicas. Los cuadernos de estrategias de seguridad de Azure Sentinel se basan en [Azure Logic Apps](https://docs.microsoft.com/azure/logic-apps/logic-apps-what-are-logic-apps), lo que significa que obtendrá toda la versatilidad, personalización y plantillas integradas de Logic Apps. Cada cuaderno de estrategias se crea para la suscripción específica que elija, pero cuando observe la página Cuadernos de estrategias, verá todos los cuadernos de estrategias en las suscripciones seleccionadas.

> [!NOTE]
> Los cuadernos de estrategias aprovechan Azure Logic Apps, por lo que se aplican cargos. Visite la página de precios de [Azure Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/) para más información.

Por ejemplo, si le preocupa que los atacantes malintencionados accedan a los recursos de red, puede establecer una alerta que busque direcciones IP malintencionadas que accedan a la red. A continuación, puede crear un cuaderno de estrategias que haga lo siguiente:
1. Cuando se desencadene la alerta, abra un vale en ServiceNow o en cualquier otro sistema de vales de TI.
2. Envíe un mensaje al canal de operaciones de seguridad en Microsoft Teams o Slack para asegurarse de que los analistas de seguridad son conscientes del incidente.
3. Envíe toda la información de la alerta al administrador de seguridad y al administrador de red sénior. El mensaje de correo electrónico también incluye dos botones de opción usuario: **Bloquear** u **Omitir**.
4. El cuaderno de estrategias continúa ejecutándose después de que se recibe una respuesta de los administradores.
5. Si los administradores eligen **Bloquear**, la dirección IP se bloquea en el firewall y el usuario se deshabilita en Azure AD.
6. Si elige los administradores **omitir**, se cierra la alerta en Azure Sentinel y se cierra el incidente de ServiceNow.

Los cuadernos de estrategias de seguridad se pueden ejecutar de forma manual o automática. La ejecución manual de los mismos significa que cuando obtiene una alerta, puede optar por ejecutar un cuaderno de estrategias a petición como respuesta a la alerta seleccionada. Si se ejecutan automáticamente, al crear la regla de correlación, se establece para que ejecute uno o más cuadernos de estrategias automáticamente cuando la alerta se desencadena.


## <a name="create-a-security-playbook"></a>Creación de un cuaderno de estrategias de seguridad

Para crear un nuevo cuaderno de estrategias de seguridad en Azure Sentinel, siga estos pasos:

1. Abra el panel de **Azure Sentinel**.
2. En **Configuración**, seleccione **Playbooks** (Cuadernos de estrategias).

   ![Aplicación lógica](./media/tutorial-respond-threats-playbook/playbookimg.png)

3. En la página **Azure Sentinel - Playbooks** (Azure Sentinel - Cuadernos de estrategias), haga clic en el botón **Add** (Agregar).

   ![Creación de la aplicación lógica](./media/tutorial-respond-threats-playbook/create-playbook.png) 

4. En la página **Crear aplicación lógica**, escriba la información solicitada para crear la nueva aplicación lógica y haga clic en el **Crear**. 

5. En [**Diseñador de aplicación lógica** ](../logic-apps/logic-apps-overview.md), seleccione la plantilla que desea usar. Si selecciona una plantilla que necesite credenciales, tendrá que proporcionarlas. Como alternativa, puede crear un nuevo cuaderno de estrategias en blanco desde cero. Seleccione **Aplicación lógica en blanco**. 

   ![Diseñador de aplicación lógica](./media/tutorial-respond-threats-playbook/playbook-template.png)

6. Se abrirá el Diseñador de aplicación lógica donde puede crear una plantilla nueva o editarla. Más información sobre cómo crear un cuaderno de estrategias con [Logic Apps](../logic-apps/logic-apps-create-logic-apps-from-templates.md).

7. Si crea un cuaderno de estrategias en blanco, en el campo **Buscar todos los conectores y desencadenadores** , escriba *Azure Sentinel* y seleccione **When a response to an Azure Sentinel alert is triggered** (Cuando se desencadena una respuesta a una alerta de Azure Sentinel). <br>Después de crearlo, el nuevo cuaderno de estrategias aparece en la lista **Cuadernos de estrategias**. Si no aparece, haga clic en **Actualizar**.

1. Use las funciones **Obtener entidades**, que permiten obtener las entidades pertinentes desde dentro de la lista **Entidades**, como cuentas, direcciones IP y hosts. Esto permitirá ejecutar acciones en entidades específicas.

7. Ahora puede definir lo que ocurre cuando se desencadena el playbook. Puede agregar una acción, una condición lógica, condiciones de casos de conmutador o bucles.

   ![Diseñador de aplicación lógica](./media/tutorial-respond-threats-playbook/logic-app.png)

## <a name="how-to-run-a-security-playbook"></a>Ejecución de un cuaderno de estrategias de seguridad

Puede ejecutar un cuaderno de estrategias a petición.

Para ejecutar un cuaderno de estrategias a petición:

1. En la página **incidents** (incidentes), seleccione un incidente y haga clic en **View full details** (Ver detalles completos).

2. En la pestaña **Alertas**, haga clic en la alerta que desee para ejecutar el cuaderno de estrategias, desplácese completamente a la derecha, haga clic en **Ver cuadernos de estrategias** y seleccione un cuaderno de estrategias para **ejecutar** en el lista de cuadernos de estrategias disponibles en la suscripción. 



## <a name="automate-threat-responses"></a>Automatizar las respuestas frente a amenazas

Los equipos de SIEM y SOC pueden verse inundados con alertas de seguridad periódicamente. El volumen de las alertas generadas es tan grande que los administradores de seguridad disponibles están desbordados. Esto genera, con demasiada frecuencia, situaciones en las que no se pueden investigar muchas alertas, lo que hace que la organización sea vulnerable a ataques que pasan desapercibidos. 

Muchas de estas alertas, si no la mayoría, se ajustan a los patrones recurrentes que se pueden resolver mediante acciones de corrección específicas y definidas. Azure Sentinel ya le permite definir la corrección en los cuadernos de estrategias. También es posible establecer la automatización en tiempo real como parte de la definición del cuaderno de estrategias para que pueda automatizar completamente una respuesta definida ante alertas de seguridad determinadas. Con la automatización en tiempo real, los equipos de respuesta pueden reducir de forma significativa su carga de trabajo al automatizar completamente las respuestas rutinarias a tipos de alertas recurrentes, lo que le permite concentrarse más en las alertas singulares, en el analizar patrones, en la búsqueda de amenazas y mucho más.

Para automatizar las respuestas:

1. Seleccione la alerta para la que quiere automatizar la respuesta.
1. En la página**Editar regla de alerta**, en **Real-time automation** (Automatización en tiempo real), elija el **Triggered playbook** (Cuaderno de estrategias desencadenado) que quiera ejecutar cuando se cumpla esta regla de alerta.
1. Seleccione **Guardar**.

   ![automatización en tiempo real](./media/tutorial-detect-threats/rt-configuration.png)






## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió cómo ejecutar un cuaderno de estrategias en Azure Sentinel. Continúe con la sección sobre [cómo buscar proactivamente amenazas](hunting.md) mediante Azure Sentinel.


