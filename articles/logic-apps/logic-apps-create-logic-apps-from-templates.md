---
title: Creación de flujos de trabajo de aplicaciones lógicas más rápidos mediante plantillas precompiladas
description: Creación rápida de flujos de trabajo de aplicaciones lógicas mediante plantillas precompiladas proporcionadas por Azure Logic Apps
services: logic-apps
ms.suite: integration
author: kevinlam1
ms.author: klam
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 10/15/2017
ms.openlocfilehash: 7eca9b4a8cf09f2f35d56dc22fc25169be192f47
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87066247"
---
# <a name="create-logic-app-workflows-from-prebuilt-templates"></a>Creación de flujos de trabajo de aplicaciones lógicas a partir de plantillas precompiladas

Para que pueda empezar a crear flujos de trabajo más rápidamente, Logic Apps proporciona plantillas, que son aplicaciones lógicas precompiladas que siguen patrones de uso frecuente. Use estas plantillas tal como se proporcionan o edítelas para ajustarlas a su escenario.

Estas son algunas categorías de plantillas:

| Tipo de plantilla | Descripción | 
| ------------- | ----------- | 
| Plantillas de empresa en la nube | Para integrar Azure Blob, Dynamics CRM, Salesforce, Box y el blob de Azure; y también incluyen otros conectores para sus necesidades empresariales en la nube. Por ejemplo, puede usar estas plantillas para organizar clientes potenciales de empresa o realizar copias de seguridad de datos de archivos corporativos. | 
| Plantillas de productividad personal | Mejore la productividad personal al establecer avisos diarios, convertir los elementos de trabajo importantes en listas de tareas pendientes y automatizar las tareas largas con un único paso de aprobación de usuario. | 
| Plantillas en la nube del consumidor | Para integrar servicios de medios sociales como Twitter, Slack y correos electrónicos. Resulta útil para reforzar las iniciativas de marketing de medios sociales. Estas plantillas también incluyen tareas como la copia de la nube, que permite aumentar la productividad mediante el ahorro del tiempo invertido en tareas habitualmente repetitivas. | 
| Plantillas de Enterprise Integration Pack | Para configurar canalizaciones VETER (validación, extracción, transformación, enriquecimiento, enrutamiento), la recepción de documentos X12 EDI a través de AS2 y la transformación en XML, así como el control de mensajes X12, EDIFACT y AS2. | 
| Plantillas de patrón de protocolo | Para implementar patrones de protocolo, como el patrón solicitud-respuesta a través de HTTP, e integraciones a través de FTP y SFTP. Use estas plantillas tal como se proporcionan, o compile a partir de estas para obtener patrones de protocolo complejos. | 
||| 

Si no tiene ninguna suscripción a Azure, [cree una cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar. Para obtener más información acerca de cómo compilar una aplicación lógica, consulte [Crear una aplicación de lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="create-logic-apps-from-templates"></a>Creación de aplicaciones lógicas a partir de plantillas

1. Inicie sesión en [Azure Portal](https://portal.azure.com "Azure Portal") si aún no lo ha hecho.

2. En el menú principal de Azure Portal, elija **Crear un recurso** > **Enterprise Integration** > **Logic App**.

   ![Portal de Azure, Nuevo, Enterprise Integration, Logic App](./media/logic-apps-create-logic-apps-from-templates/azure-portal-create-logic-app.png)

3. Cree la aplicación lógica con la configuración que se incluye en la tabla situada bajo la imagen:

   ![Proporcione los detalles de la aplicación lógica](./media/logic-apps-create-logic-apps-from-templates/logic-app-settings.png)

   | Configuración | Value | Descripción | 
   | ------- | ----- | ----------- | 
   | **Nombre** | *nombre-de-la-aplicación-lógica* | Proporcione un nombre único de aplicación lógica. | 
   | **Suscripción** | *your-Azure-subscription-name* | Seleccione la suscripción de Azure que quiera usar. | 
   | **Grupos de recursos** | *your-Azure-resource-group-name* | Cree o seleccione un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) para esta aplicación lógica y organice todos los recursos asociados a esta aplicación. | 
   | **Ubicación** | *your-Azure-datacenter-region* | Seleccione la región del centro de datos para implementar la aplicación lógica, por ejemplo, Oeste de EE.UU. | 
   | **Log Analytics** | **Off** (Desactivar) (valor predeterminado) o **On** (Activar) | Configure el [registro de diagnóstico](../logic-apps/monitor-logic-apps-log-analytics.md) para su aplicación lógica mediante los [registros de Azure Monitor](../azure-monitor/log-query/log-query-overview.md). Es necesario que ya tenga un área de trabajo de Log Analytics. | 
   |||| 

4. Cuando esté listo, elija **Anclar al panel**. De este modo, la aplicación lógica aparecerá automáticamente en el panel de Azure y se abrirá después de la implementación. Seleccione **Create**.

   > [!NOTE]
   > Si no quiere anclar la aplicación lógica, búsquela y ábrala manualmente después de la implementación para continuar.

   Una vez que Azure ha implementado la aplicación lógica, el Diseñador de Logic Apps se abre y muestra una página con un vídeo de introducción. 
   Bajo el vídeo encontrará plantillas de los patrones de aplicación lógica más comunes. 

5. Pase el vídeo de introducción y los desencadenadores comunes hasta **Plantillas**. Elija una plantilla precompilada. Por ejemplo:

   ![Elegir una plantilla de aplicación lógica](./media/logic-apps-create-logic-apps-from-templates/choose-logic-app-template.png)

   > [!TIP]
   > Para crear la aplicación lógica desde cero, elija **Aplicación lógica en blanco**.

   Cuando se selecciona una plantilla precompilada, puede ver más información acerca de dicha plantilla. 
   Por ejemplo:

   ![Elegir una plantilla precompilada](./media/logic-apps-create-logic-apps-from-templates/logic-app-choose-prebuilt-template.png)

6. Para continuar con la plantilla seleccionada, elija **Usar esta plantilla**. 

7. En función de los conectores de la plantilla, se le solicitará que realice cualquiera de estos pasos:

   * Inicie sesión con sus credenciales en los sistemas o servicios a los que la plantilla hace referencia.

   * Cree conexiones para todos los servicios o sistemas a los que hace referencia la plantilla. Para crear una conexión, proporcione un nombre para la conexión y, si es necesario, seleccione el recurso que quiere usar. 

   * Si ya configuró estas conexiones, elija **Continuar**.

   Por ejemplo:

   ![Crear conexiones](./media/logic-apps-create-logic-apps-from-templates/logic-app-create-connection.png)

   Cuando haya terminado, la aplicación lógica se abrirá y aparecerá en el Diseñador de Logic Apps.

   > [!TIP]
   > Para volver al visor de plantillas, elija **Plantillas** en la barra de herramientas del diseñador. Esta acción descarta los cambios no guardados, por lo que aparece un mensaje de advertencia para confirmar la solicitud.

8. Siga compilando su aplicación lógica.

   > [!NOTE] 
   > Muchas de las plantillas incluyen conectores que puede que ya se hayan rellenado previamente con las propiedades necesarias. Sin embargo, es posible que algunas plantillas todavía necesiten proporcionar valores antes de que pueda implementar correctamente la aplicación lógica. Si intenta implementar sin completar los campos de propiedades que faltan, recibirá un mensaje de error. 

## <a name="update-logic-apps-with-templates"></a>Actualizar aplicaciones lógicas con plantillas

1. En [Azure Portal](https://portal.azure.com "Azure Portal"), busque y abra la aplicación lógica en el Diseñador de aplicaciones lógicas.

2. En la barra de herramientas del diseñador, haga clic en **Plantillas**. Esta acción descarta los cambios no guardados, por lo que aparece un mensaje de advertencia para confirmar que quiere continuar. Para confirmar, elija **Aceptar**. Por ejemplo:

   ![Elegir "Plantillas"](./media/logic-apps-create-logic-apps-from-templates/logic-app-update-existing-with-template.png)

3. Pase el vídeo de introducción y los desencadenadores comunes hasta **Plantillas**. Elija una plantilla precompilada. Por ejemplo:

   ![Elegir una plantilla de aplicación lógica](./media/logic-apps-create-logic-apps-from-templates/choose-logic-app-template.png)

   Cuando se selecciona una plantilla precompilada, puede ver más información acerca de dicha plantilla. 
   Por ejemplo:

   ![Elegir una plantilla precompilada](./media/logic-apps-create-logic-apps-from-templates/logic-app-choose-prebuilt-template.png)

4. Para continuar con la plantilla seleccionada, elija **Usar esta plantilla**. 

5. En función de los conectores de la plantilla, se le solicitará que realice cualquiera de estos pasos:

   * Inicie sesión con sus credenciales en los sistemas o servicios a los que la plantilla hace referencia.

   * Cree conexiones para todos los servicios o sistemas a los que hace referencia la plantilla. Para crear una conexión, proporcione un nombre para la conexión y, si es necesario, seleccione el recurso que quiere usar. 

   * Si ya configuró estas conexiones, elija **Continuar**.

   ![Crear conexiones](./media/logic-apps-create-logic-apps-from-templates/logic-app-create-connection.png)

   La aplicación lógica se abrirá y aparecerá en el Diseñador de Logic Apps.

8. Siga compilando su aplicación lógica. 

   > [!TIP]
   > Si no guardó los cambios, puede descartar el trabajo y volver a la aplicación lógica anterior. En la barra de herramientas del diseñador, haga clic en **Descartar**.

> [!NOTE] 
> Muchas de las plantillas incluyen conectores que puede que ya hayan rellenado previamente las propiedades necesarias. Sin embargo, es posible que algunas plantillas todavía necesiten proporcionar valores antes de que pueda implementar correctamente la aplicación lógica. Si intenta implementar sin completar los campos de propiedades que faltan, recibirá un mensaje de error.

## <a name="deploy-logic-apps-built-from-templates"></a>Implementar aplicaciones lógicas compiladas a partir de plantillas

Después de realizar los cambios en la plantilla, puede guardarlos. Esta acción también publica automáticamente la aplicación lógica.

En la barra de herramientas del diseñador, haga clic en **Guardar**.

![Guardar y publicar la aplicación lógica](./media/logic-apps-create-logic-apps-from-templates/logic-app-save.png)  

## <a name="get-support"></a>Obtención de soporte técnico

* Si tiene alguna duda, visite la [página de preguntas y respuestas de Microsoft sobre Azure Logic Apps](/answers/topics/azure-logic-apps.html).
* Para enviar ideas sobre características o votar sobre ellas, visite el [sitio de comentarios de los usuarios de Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca de cómo compilar aplicaciones lógicas a través de ejemplos, escenarios, casos de clientes y tutoriales.

> [!div class="nextstepaction"]
> [Escenarios comunes, ejemplos, tutoriales y guías detalladas de Azure Logic Apps](../logic-apps/logic-apps-examples-and-scenarios.md)
