---
title: Automatización de flujos de trabajo en Azure Security Center | Microsoft Docs
description: Aprenda a crear y automatizar flujos de trabajo en Azure Security Center
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 09/12/2020
ms.author: memildin
ms.openlocfilehash: 7933cc692ebc3b40e5f608a917dce51f5298fbe3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904619"
---
# <a name="create-automatic-responses-to-alerts-and-recommendations-with-workflow-automation"></a>Creación de respuestas automáticas a alertas y recomendaciones con automatización del flujo de trabajo

Cada programa de seguridad incluye varios flujos de trabajo para la respuesta a incidentes. Estos procesos pueden incluir notificaciones para las partes interesadas competentes, iniciar un proceso de administración de cambios y aplicar pasos de corrección específicos. Los expertos en seguridad recomiendan automatizar tantos pasos de esos procedimientos como sea posible. Recuerde que la automatización reduce la sobrecarga. También puede mejorar la seguridad asegurándose de que los pasos del proceso se realizan de forma rápida, coherente y según sus requisitos predefinidos.

En este artículo se describe la característica de automatización de flujos de trabajo de Azure Security Center. Esta característica puede desencadenar una instancia de Logic Apps sobre alertas y recomendaciones de seguridad. Por ejemplo, si quiere que Security Center envíe un correo electrónico a un usuario específico cuando se produce una alerta. También aprenderá a crear instancias de Logic Apps con [Azure Logic Apps](https://docs.microsoft.com/azure/logic-apps/logic-apps-overview).

> [!NOTE]
> Si ya se ha usado la vista Cuadernos de estrategias (versión preliminar) de la barra lateral, se encontrarán las mismas características junto con la función expandida en la página nueva de automatización del flujo de trabajo.



## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponible con carácter general|
|Precios:|Gratuito|
|Roles y permisos necesarios:|**Rol Administrador de seguridad** o **Propietario** en el grupo de recursos<br>También debe tener permisos de escritura para el recurso de destino.<br><br>Para trabajar con flujos de trabajo de Azure Logic Apps, también debe tener los siguientes roles o permisos de Logic Apps:<br> Son necesarios los permisos de - [Operador de aplicación lógica](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#logic-app-operator) o el acceso de lectura o desencadenamiento de aplicación lógica (este rol no puede crear ni editar aplicaciones lógicas, solo *ejecutar* las existentes).<br> Los permisos de - [Colaborador de la aplicación lógica](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#logic-app-contributor) son necesarios para la creación y modificación de aplicaciones lógicas.<br>Si quiere usar conectores de aplicaciones lógicas, es posible que necesite credenciales adicionales para iniciar sesión en sus servicios respectivos (por ejemplo, en las instancias de Outlook, Teams o Slack).|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) US Gov<br>![Sí](./media/icons/yes-icon.png) China Gov, otros gobiernos|
|||



## <a name="create-a-logic-app-and-define-when-it-should-automatically-run"></a>Crear una aplicación lógica y definir cuándo debe ejecutarse automáticamente 

1. En la barra lateral de Security Center, seleccione **Automatización de flujos de trabajo**.

    :::image type="content" source="./media/workflow-automation/list-of-workflow-automations.png" alt-text="Lista de automatizaciones de flujos de trabajo":::

    Desde esta página puede crear nuevas reglas de automatización, así como habilitar, deshabilitar o eliminar las existentes.

1. Para definir un nuevo flujo de trabajo, haga clic en **Add workflow automation** (Agregar automatización de flujos de trabajo). 

    Aparecerá un panel con las opciones de la nueva automatización. Aquí puede escribir lo siguiente:
    1. Un nombre y descripción para la automatización.
    1. Los desencadenadores que iniciarán este flujo de trabajo automático. Por ejemplo, cuando quiera que la aplicación lógica se ejecute cuando se genere una alerta de seguridad que contenga "SQL".
    1. La aplicación lógica que se ejecutará cuando se cumplan las condiciones del desencadenador. 

        :::image type="content" source="./media/workflow-automation/add-workflow.png" alt-text="Panel para agregar automatizaciones del flujo de trabajo":::

1. En la sección Acciones, haga clic en **Crear una nueva** para comenzar el proceso de creación de la aplicación lógica.

    Se le dirigirá a Azure Logic Apps.

    [![Creación de una aplicación lógica nueva](media/workflow-automation/logic-apps-create-new.png)](media/workflow-automation/logic-apps-create-new.png#lightbox)

1. Escriba un nombre, un grupo de recursos y una ubicación, y haga clic en **Crear**.

1. En la nueva aplicación lógica, puede decidir si quiere usar plantillas predefinidas integradas en la categoría de seguridad. También puede definir un flujo de eventos personalizado para que se active cuando se desencadene este proceso.

    En el diseñador de la aplicación lógica se admiten los siguientes desencadenadores de los conectores de Security Center:

    * **Cuando se crea o se desencadena una recomendación de Azure Security Center**
    * **Cuando se crea o se desencadena una alerta de Azure Security Center** 
    
    > [!TIP]
    > Se puede personalizar el desencadenador para que haga referencia solo a las alertas con los niveles de gravedad que le interesen.
    
    > [!NOTE]
    > Si utiliza el desencadenador heredado "Cuando se desencadena una respuesta a una alerta de Azure Security Center", la característica de automatización de flujos de trabajo no iniciará la instancia de Logic Apps. En su lugar, use cualquiera de los desencadenadores mencionados anteriormente. 

    [![Aplicación lógica de ejemplo](media/workflow-automation/sample-logic-app.png)](media/workflow-automation/sample-logic-app.png#lightbox)

1. Una vez que haya definido la aplicación lógica, vuelva al panel de la definición de automatización del flujo de trabajo ("Agregar automatización de flujos de trabajo"). Haga clic en **Actualizar** para asegurarse de que la nueva aplicación lógica está disponible en la selección.

    ![Actualizar](media/workflow-automation/refresh-the-list-of-logic-apps.png)

1. Seleccione la aplicación lógica y guarde la automatización. Tenga en cuenta que la lista desplegable de aplicaciones lógicas solo muestra las aplicaciones lógicas con los conectores de Security Center complementarios mencionados anteriormente.


## <a name="manually-trigger-a-logic-app"></a>Desencadenar manualmente una aplicación lógica

También puede ejecutar Logic Apps manualmente al ver una alerta o recomendación de seguridad.

Para ejecutar manualmente una aplicación lógica, abra una alerta o recomendación y haga clic en **Desencadenar aplicación lógica**:

[![Desencadenar manualmente una aplicación lógica](media/workflow-automation/manually-trigger-logic-app.png)](media/workflow-automation/manually-trigger-logic-app.png#lightbox)

## <a name="data-types-schemas"></a>Esquemas de tipos de datos

Para ver los esquemas de eventos sin procesar de las alertas de seguridad o los eventos de recomendaciones que se pasan a la instancia de aplicación lógica, consulte los [Esquemas de tipos de datos de automatización de flujo de trabajo](https://aka.ms/ASCAutomationSchemas). Puede resultar útil en los casos en que no se usan los conectores de aplicación lógica integrados de Security Center mencionados anteriormente, sino que alternativamente se usa el conector HTTP genérico de la aplicación lógica; puede usar el esquema JSON del evento para analizarlo manualmente como considere oportuno.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre cómo crear aplicaciones lógicas, automatizar su ejecución en Security Center y ejecutarlas manualmente. 

Para obtener material relacionado, consulte: 

- [Módulo de Microsoft Learn sobre cómo la automatización de flujos de trabajo para automatizar una respuesta de seguridad](https://docs.microsoft.com/learn/modules/resolve-threats-with-azure-security-center/)
- [Recomendaciones de seguridad en Azure Security Center](security-center-recommendations.md)
- [Alertas de seguridad en el Centro de seguridad de Azure](security-center-alerts-overview.md)
- [Acerca de Azure Logic Apps](https://docs.microsoft.com/azure/logic-apps/logic-apps-overview)
- [Conectores de Aplicaciones lógicas](https://docs.microsoft.com/connectors/)
- [Esquemas de tipos de datos de automatización de flujo de trabajo](https://aka.ms/ASCAutomationSchemas)
