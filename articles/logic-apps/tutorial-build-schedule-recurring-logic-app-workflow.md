---
title: Creación de flujos de trabajo automatizados basados en una programación
description: 'Tutorial: Creación de un flujo de trabajo automatizado, periódico y basado en una programación mediante Azure Logic Apps'
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: tutorial
ms.custom: mvc
ms.date: 09/12/2019
ms.openlocfilehash: 41f7b1309a9c7fa9a5f2abb3e2e59f08ef31382d
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87124857"
---
# <a name="tutorial-create-automated-schedule-based-recurring-workflows-by-using-azure-logic-apps"></a>Tutorial: Creación de un flujo de trabajo automatizado, periódico y basado en una programación mediante Azure Logic Apps

En este tutorial se muestra cómo crear una [aplicación lógica](../logic-apps/logic-apps-overview.md) y automatizar un flujo de trabajo periódico que se ejecuta según una programación. En concreto, esta aplicación lógica de ejemplo se ejecuta cada día por la mañana y comprueba el tiempo de desplazamiento, incluido el tráfico, entre dos lugares. Si el tiempo supera un límite específico, la aplicación lógica envía un correo electrónico con el tiempo de desplazamiento y el tiempo adicional necesario para el destino.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una aplicación lógica en blanco.
> * Agregar un desencadenador de periodicidad que especifique la programación de la aplicación lógica.
> * Agregar una acción de Bing Maps que obtenga el tiempo de desplazamiento para una ruta.
> * Agregar una acción que cree una variable, convierta el tiempo de desplazamiento de segundos a minutos y guarde ese resultado en la variable.
> * Agregar una condición que compara el tiempo de desplazamiento con un límite especificado.
> * Agregar una acción que envíe un correo electrónico si el tiempo de desplazamiento supera el límite.

Cuando haya terminado, la aplicación lógica se parecerá a este flujo de trabajo, en un alto nivel:

![Introducción al flujo de trabajo de una aplicación lógica de alto nivel](./media/tutorial-build-scheduled-recurring-logic-app-workflow/check-travel-time-overview.png)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene ninguna suscripción, [suscríbase a una cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.

* Una cuenta de correo electrónico con un proveedor de correo electrónico compatible con Logic Apps, como Office 365 Outlook, Outlook.com o Gmail. En el caso de otros proveedores, [consulte la lista de conectores que se muestra aquí](/connectors/). Este inicio rápido utiliza una cuenta de Office 365 Outlook. Si utiliza una cuenta de correo electrónico diferente, los pasos generales siguen siendo los mismos, pero la interfaz de usuario podría ser ligeramente distinta.

  > [!IMPORTANT]
  > Si quiere usar el conector de Gmail, solo las cuentas empresariales de G-Suite pueden usar este conector sin restricciones en las aplicaciones lógicas. Si tiene una cuenta de consumidor de Gmail, puede usar este conector solo con servicios específicos aprobados por Google, o puede [crear una aplicación cliente de Google para usarla en la autenticación con el conector de Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Para más información, consulte [Directivas de privacidad y seguridad de datos de los conectores de Google en Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

* Para obtener el tiempo de desplazamiento para una ruta, necesita una clave de acceso para la API de Mapas de Bing. Para obtener esta clave, siga los pasos sobre [cómo obtener una clave de Mapas de Bing](/bingmaps/getting-started/bing-maps-dev-center-help/getting-a-bing-maps-key).

## <a name="create-your-logic-app"></a>Creación de una aplicación lógica

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con sus credenciales de su cuenta de Azure.

1. En el menú principal de Azure, seleccione **Crear un recurso** > **Integración** > **Aplicación lógica**.

   ![Creación del recurso para la aplicación lógica](./media/tutorial-build-scheduled-recurring-logic-app-workflow/create-new-logic-app-resource.png)

1. En **Crear aplicación lógica**, proporcione esta información sobre la aplicación lógica tal como se muestra y se describe a continuación. Seleccione **Crear** cuando haya terminado.

   ![Proporción de información acerca de la aplicación lógica](./media/tutorial-build-scheduled-recurring-logic-app-workflow/create-logic-app-settings.png)

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **Nombre** | LA TravelTime | El nombre de la aplicación lógica, que solo puede contener letras, números, guiones (`-`), caracteres de subrayado (`_`), paréntesis (`(`, `)`) y puntos (`.`). En este ejemplo se utiliza "LA-TravelTime". |
   | **Suscripción** | <*nombre-de-su-suscripción-a-Azure*> | El nombre de la suscripción de Azure |
   | **Grupos de recursos** | LA-TravelTime-RG | El nombre del [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) que se utiliza para organizar recursos relacionados. En este ejemplo se utiliza "LA-TravelTime-RG". |
   | **Ubicación** | Oeste de EE. UU. | La región en la que desea almacenar la información de la aplicación lógica. En este ejemplo se utiliza "Oeste de EE. UU.". |
   | **Log Analytics** | Off | Mantenga el valor **Off** para el registro de diagnóstico. |
   ||||

1. Una vez que Azure implemente la aplicación, en la barra de herramientas de Azure, seleccione **Notificaciones** > **Ir al recurso** para la aplicación lógica implementada.

   ![Vaya al nuevo recurso de la aplicación lógica](./media/tutorial-build-scheduled-recurring-logic-app-workflow/go-to-new-logic-app-resource.png)

   O también puede buscar y seleccionar la aplicación lógica escribiendo el nombre en el cuadro de búsqueda.

   El Diseñador de aplicaciones lógicas se abre y muestra una página con un vídeo de introducción y los patrones de aplicaciones lógicas y desencadenadores utilizados frecuentemente. En **Plantillas**, elija **Blank Logic App**.

   ![Selección de Aplicación lógica en blanco](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-logic-app-template.png)

A continuación, agregue el [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts) Periodicidad, que se activa en función de una programación especificada. Cada aplicación lógica debe comenzar con un desencadenador, que se activa cuando sucede un evento específico o cuando hay nuevos datos que cumplen una condición determinada. Para más información, consulte [Creación de una nueva aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="add-the-recurrence-trigger"></a>Adición del desencadenador Periodicidad

1. Cuando se abra el Diseñador de aplicaciones lógicas, en el cuadro de búsqueda, escriba el filtro "periodicidad". En la lista **Desencadenadores**, seleccione **Periodicidad**.

   ![Agregar desencadenador "Periodicidad"](./media/tutorial-build-scheduled-recurring-logic-app-workflow/add-schedule-recurrence-trigger.png)

1. En la forma **Periodicidad**, seleccione el botón de **puntos suspensivos** ( **...** ) y luego **Cambiar de nombre**. Cambie el nombre del desencadenador por esta descripción:`Check travel time every weekday morning`

   ![Cambio de nombre de la descripción del desencadenador de periodicidad](./media/tutorial-build-scheduled-recurring-logic-app-workflow/rename-recurrence-schedule-trigger.png)

1. En el desencadenador, cambie estas propiedades.

   ![Cambio del intervalo de periodicidad y la frecuencia del desencadenador](./media/tutorial-build-scheduled-recurring-logic-app-workflow/change-interval-frequency.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Intervalo** | Sí | 1 | Número de intervalos que se espera entre comprobaciones |
   | **Frecuencia** | Sí | Semana | Unidad de tiempo que se usa para la periodicidad. |
   |||||

1. En **Intervalo** y **Frecuencia**, abra la lista **Agregar nuevo parámetro** y seleccione estas propiedades para agregarlas al desencadenador.

   * **En estos días**
   * **A estas horas**
   * **En estos minutos**

   ![Incorporación de propiedades para el desencadenador de periodicidad](./media/tutorial-build-scheduled-recurring-logic-app-workflow/add-trigger-properties.png)

1. Ahora, establezca los valores de las propiedades adicionales como se muestra y se describe aquí.

   ![Indicación de detalles de programación y periodicidad](./media/tutorial-build-scheduled-recurring-logic-app-workflow/recurrence-trigger-property-values.png)

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **En estos días** | Lunes,Martes,Miércoles,Jueves,Viernes | Disponible solo cuando **Frecuencia** se establece en "Semana". |
   | **A estas horas** | 7,8,9 | Disponible solo cuando **Frecuencia** se establece en "Semana" o "Día". Se seleccionan las horas del día para ejecutar esta periodicidad. En este ejemplo, la ejecución se realiza a las marcas de hora 7, 8 y 9. |
   | **En estos minutos** | 0,15,30,45 | Disponible solo cuando **Frecuencia** se establece en "Semana" o "Día". Se seleccionan los minutos del día para ejecutar esta periodicidad. En este ejemplo, la ejecución se realiza cada 15 minutos a partir de la marca de hora cero. |
   ||||

   Este desencadenador se activa los días de entre semana cada 15 minutos, a partir de las 7:00 a.m. y hasta las 9:45 p.m. El cuadro **Vista previa** muestra la programación de periodicidad. Para más información, consulte [Programación de tareas y flujos de trabajo](../connectors/connectors-native-recurrence.md) y [Acciones y desencadenadores de flujos de trabajo](../logic-apps/logic-apps-workflow-actions-triggers.md#recurrence-trigger).

1. Para ocultar por ahora los detalles del desencadenador, haga clic dentro de la barra de título de la forma.

   ![Contraer la forma para ocultar detalles](./media/tutorial-build-scheduled-recurring-logic-app-workflow/collapse-trigger-shape.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

La aplicación lógica está ahora activa pero no hace más que repetirse. Por lo tanto, agregue una acción que responda cuando se active el desencadenador.

## <a name="get-the-travel-time-for-a-route"></a>Obtención del tiempo de desplazamiento para una ruta

Ahora que ya tiene un desencadenador, agregue una [acción](../logic-apps/logic-apps-overview.md#logic-app-concepts) que obtenga el tiempo de desplazamiento entre dos lugares. Logic Apps proporciona un conector para la API de Mapas de Bing de forma que pueda obtener fácilmente esta información. Antes de iniciar esta tarea, asegúrese de tener una clave de la API de Mapas de Bing tal como se describe en los requisitos previos de este tutorial.

1. En el Diseñador de aplicaciones lógicas, seleccione **Nuevo paso**.

1. En **Elegir una acción**, seleccione **Estándar**. En el cuadro de búsqueda, escriba "bing maps" como filtro y seleccione la acción **Get route** (Obtener ruta).

   ![Selección de la acción "Get route" (Obtener ruta)](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-get-route-action.png)

1. Si no tiene una conexión de Mapas de Bing, se le solicitará que cree una conexión. Proporcione estos detalles de conexión y seleccione **Crear**.

   ![Creación de la conexión a la API de Bing Maps](./media/tutorial-build-scheduled-recurring-logic-app-workflow/create-maps-connection.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Nombre de la conexión** | Sí | BingMapsConnection | Proporcione un nombre para la conexión. En este ejemplo se usa "BingMapsConnection". |
   | **Clave de API** | Sí | <*su-clave-de-Mapas-de-Bing*> | Escriba la clave de Mapas de Bing recibida previamente. Si no tiene una clave de Mapas de Bing, consulte [Getting a Bing Maps Key](/bingmaps/getting-started/bing-maps-dev-center-help/getting-a-bing-maps-key) (Obtención de una clave de Mapas de Bing). |
   |||||

1. Cambie el nombre de la acción por esta descripción: `Get route and travel time with traffic`

1. Dentro de la acción, abra la **lista Agregar nuevo parámetro** y seleccione estas propiedades para agregarlas a la acción.

   * **Optimize** (Optimizar)
   * **Distance unit** (Unidad de distancia)
   * **Travel mode** (Modo de desplazamiento)

   ![Incorporación de propiedades a la acción "Get route" (Obtener ruta)](./media/tutorial-build-scheduled-recurring-logic-app-workflow/add-bing-maps-action-properties.png) 

1. Ahora, establezca los valores de las propiedades de la acción como se muestra y se describe aquí.

   ![Especificación de detalles para la acción "Get route" (Obtener ruta)](./media/tutorial-build-scheduled-recurring-logic-app-workflow/get-route-action-settings.png) 

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Waypoint 1** (Punto de referencia 1) | Sí | <*ubicación-inicial*> | Origen de la ruta |
   | **Waypoint 2** (Punto de referencia 2) | Sí | <*ubicación-final*> | Destino de la ruta |
   | **Optimize** (Optimizar) | No | timeWithTraffic | Parámetro para optimizar la ruta, como distancia, tiempo de desplazamiento con tráfico actual, etc. Seleccione el parámetro "timeWithTraffic". |
   | **Distance unit** (Unidad de distancia) | No | <*su preferencia*> | Unidad de distancia para la ruta. En este ejemplo se usa "kilómetro" como unidad. |
   | **Travel mode** (Modo de desplazamiento) | No | Conducción | Modo de desplazamiento para la ruta. Seleccione el modo "Driving" (Conducción). |
   ||||

   Para más información acerca de estos parámetros, consulte [Calculate a route](/bingmaps/rest-services/routes/calculate-a-route) (Cálculo de una ruta).

1. Guarde la aplicación lógica.

A continuación, cree una variable para que pueda convertir y almacenar el tiempo de desplazamiento actual en minutos, en lugar de en segundos. De este modo, puede evitar repetir la conversión y utilizar el valor más fácilmente en pasos posteriores. 

## <a name="create-a-variable-to-store-travel-time"></a>Creación de una variable para almacenar el tiempo de desplazamiento

Tal vez desee realizar operaciones en los datos del flujo de trabajo y usar los resultados en acciones posteriores. Para guardar estos resultados, de forma que pueda volver a utilizarlos o hacer referencia a ellos fácilmente, puede crear variables para almacenar los resultados después de procesarlos. Solo puede crear variables en el nivel superior de la aplicación lógica.

De forma predeterminada, la acción anterior **Get route** (Obtener ruta) devuelve el tiempo de desplazamiento actual con el tráfico en segundos mediante la propiedad **Travel Duration Traffic** (Tráfico de duración del desplazamiento). Al convertir y almacenar este valor en minutos, facilita volver a utilizar el valor más adelante sin necesidad de convertirlo de nuevo.

1. En la acción **Get route** (Obtener ruta), seleccione **Nuevo paso**.

1. En **Elegir una acción**, seleccione **Integrado**. En el cuadro de búsqueda, escriba "variables" y seleccione la acción **Inicializar variable**.

   ![Selección de la acción "Inicializar variable"](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-initialize-variable-action.png)

1. Cambie el nombre de esta acción por esta descripción: `Create variable to store travel time`

1. Proporcione los detalles de la variable tal como se describen aquí:

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Nombre** | Sí | travelTime | El nombre de la variable. En este ejemplo se usa "travelTime". |
   | **Tipo** | Sí | Entero | Tipo de datos de la variable |
   | **Valor** | No| Expresión que convierte el tiempo de desplazamiento actual de segundos a minutos (consulte los pasos a continuación de esta tabla). | Valor inicial de la variable |
   ||||

   1. Para crear la expresión para el campo **Valor**, haga clic en el campo para que aparezca la lista de contenido dinámico. Si es necesario, amplíe el explorador hasta que aparezca la lista. En la lista de contenido dinámico, seleccione **Expresión**.

      ![Especificación de información para la acción "Inicializar variable"](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings.png)

      Al hacer clic en algunos cuadros de edición, aparece la lista de contenido dinámico. En esta lista se muestran los parámetros de las acciones anteriores que puede usar como entradas en el flujo de trabajo. La lista de contenido dinámico tiene un editor de expresiones donde puede seleccionar funciones para ejecutar operaciones. Este editor de expresiones solo aparece en la lista de contenido dinámico.

   1. En el editor de expresiones, escriba esta expresión: `div(,60)`

      ![Escriba esta expresión: "div(,60)"](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-2.png)

   1. Coloque el cursor dentro de la expresión entre el paréntesis de apertura ( **(** ) y la coma ( **,** ). 
   Seleccione **Contenido dinámico**.

      ![Colocación del cursor, selección de "Contenido dinámico"](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-3.png)

   1. En la lista de contenido dinámico, seleccione **Travel Duration Traffic** (Tráfico de duración del desplazamiento).

      ![Selección de la propiedad "Travel Duration Traffic" (Tráfico de duración del desplazamiento)](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-4.png)

   1. Una vez que el valor de la propiedad se resuelva dentro de la expresión, seleccione **Aceptar**.

      ![Selección de "Aceptar" para terminar la compilación de la expresión](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-5.png)

      El campo **Valor** aparece ahora como se muestra aquí:

      ![Propiedad "Value" con la expresión resuelta](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-6.png)

1. Guarde la aplicación lógica.

A continuación, agregue una condición que compruebe si el tiempo de desplazamiento actual es mayor que un límite específico.

## <a name="compare-the-travel-time-with-limit"></a>Comparación del tiempo de desplazamiento con límite

1. En la acción anterior, seleccione **Nuevo paso**.

1. En **Elegir una acción**, seleccione **Integrado**. En el cuadro de búsqueda, escriba "condición" como filtro. En la lista Acciones, seleccione la acción **Condición**.

   ![Selección de la acción "Condición"](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-condition-action.png)

1. Cambie el nombre de la condición por esta descripción: `If travel time exceeds limit`

1. Cree una condición que compruebe si el valor de la propiedad **travelTime** (tiempo de desplazamiento) supera el límite especificado tal y como se describe aquí:

   1. En la condición, haga clic en el cuadro **Elegir un valor**, en el lado izquierdo de la condición.

   1. En la lista de contenido dinámico que aparece, en **Variables**, seleccione la propiedad **travelTime**.

      ![Compilación del lado izquierdo de la condición](./media/tutorial-build-scheduled-recurring-logic-app-workflow/build-condition-left-side.png)

   1. En el cuadro de comparación central, seleccione el operador **es mayor que**.

   1. En el cuadro **Elegir un valor** del lado derecho de la condición, escriba este límite: `15`

      Una vez que haya terminado, la condición debe ser parecida a la de este ejemplo:

      ![Condición finalizada para comprobar el tiempo de desplazamiento](./media/tutorial-build-scheduled-recurring-logic-app-workflow/build-condition-check-travel-time.png)

1. Guarde la aplicación lógica.

A continuación, agregue la acción que desea ejecutar cuando el tiempo de desplazamiento supere el límite.

## <a name="send-email-when-limit-exceeded"></a>Envío de un correo electrónico cuando se supere el límite

Ahora, agregue una acción que se le enviará por correo electrónico cuando el tiempo de desplazamiento supere el límite. Este correo electrónico incluye el tiempo de desplazamiento actual y el tiempo adicional necesario para desplazarse por la ruta especificada.

1. En la rama **If true** (Si se cumple) de la condición, seleccione **Agregar una acción**.

1. En **Elegir una acción**, seleccione **Estándar**. En el cuadro de búsqueda, escriba "enviar un correo electrónico". La lista devuelve muchos resultados, por lo que primero debe seleccionar el conector de correo electrónico que desee, por ejemplo:

   ![Selección del conector de correo electrónico preferido](./media/tutorial-build-scheduled-recurring-logic-app-workflow/add-action-send-email.png)

   * Para las cuentas profesionales o educativas de Azure, seleccione **Office 365 Outlook**.
   * Para las cuentas de Microsoft personales, seleccione **Outlook.com**.

1. Cuando aparezcan las acciones del conector, seleccione "enviar acción de correo electrónico" que desea usar, por ejemplo:

   ![Seleccione la acción "send email"](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-send-email-action.png)

1. Si no tiene ya una conexión, se le solicita que inicie sesión en su cuenta de correo electrónico.

   Logic Apps crea una conexión a la cuenta de correo electrónico.

1. Cambie el nombre de la acción por esta descripción: `Send email with travel time`

1. En el cuadro **Para**, escriba la dirección de correo electrónico del destinatario. Para las pruebas, use su dirección de correo electrónico.

1. En el cuadro **Asunto**, especifique el asunto del correo electrónico e incluya la variable **travelTime**.

   1. Escriba el texto `Current travel time (minutes):` con un espacio final. 

   1. En la lista de contenido dinámico, en **Variables**, seleccione **Ver más**.

      ![Búsqueda de la variable "travelTime"](./media/tutorial-build-scheduled-recurring-logic-app-workflow/find-travelTime-variable.png)

   1. Una vez que **travelTime** aparezca en **Variables**, seleccione **travelTime**.

      ![Especificación del texto del asunto y de una expresión que devuelva el tiempo de desplazamiento](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-travelTime-variable.png)

1. En el cuadro **Cuerpo**, especifique el contenido del cuerpo del correo electrónico.

   1. Escriba el texto `Add extra travel time (minutes):` con un espacio final.

   1. En la lista de contenido dinámico, seleccione **Expresión**.

      ![Creación de una expresión para el cuerpo del correo electrónico](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings.png)

   1. En el editor de expresiones, escriba esta expresión para que pueda calcular el número de minutos que superen el límite: ```sub(,15)```

      ![Especificación de la expresión para calcular el tiempo del desplazamiento adicional en minutos](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-2.png)

   1. Coloque el cursor dentro de la expresión entre el paréntesis de apertura ( **(** ) y la coma ( **,** ). Seleccione **Contenido dinámico**.

      ![Continuación de la creación de la expresión para calcular el tiempo del desplazamiento adicional en minutos](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-3.png)

   1. En **Variables**, seleccione **travelTime**.

      ![Selección de la propiedad "travelTime" que se utilizará en la expresión](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-4.png)

   1. Una vez que la propiedad se resuelva dentro de la expresión, seleccione **Aceptar**.

      ![Selección de "Aceptar" una vez resuelta la propiedad "Body"](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-5.png)

      La propiedad **Body** (Cuerpo) aparece ahora como se muestra aquí:

      ![Expresión con la propiedad "Body" resuelta](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-6.png)

1. Guarde la aplicación lógica.

A continuación, pruebe la aplicación lógica, que ahora es similar a este ejemplo:

![Ejemplo de flujo de trabajo de aplicación lógica finalizado](./media/tutorial-build-scheduled-recurring-logic-app-workflow/check-travel-time-finished.png)

## <a name="run-your-logic-app"></a>Ejecución de la aplicación lógica

Para iniciar manualmente la aplicación lógica, en la barra de herramientas del diseñador, elija **Ejecutar**.

* Si el tiempo de desplazamiento actual permanece en el límite, la aplicación lógica no hace nada más y espera al siguiente intervalo antes de volver a comprobar. 

* Pero, si el tiempo de desplazamiento actual supera el límite, recibirá un correo electrónico con el tiempo de desplazamiento actual y el número de minutos por encima del límite. Este es un correo electrónico de ejemplo que la aplicación lógica envía:

![Ejemplo de correo electrónico enviado que muestra el tiempo de desplazamiento](./media/tutorial-build-scheduled-recurring-logic-app-workflow/received-example-email-notification.png)

Si no recibe ningún correo electrónico, compruebe la carpeta de correo electrónico no deseado. El filtro de correo electrónico no deseado podría redirigir esta clase de correo. Si tampoco aparece allí y no está seguro de que la aplicación lógica se ejecutara correctamente, consulte el artículo de [solución de problemas en la aplicación lógica](../logic-apps/logic-apps-diagnosing-failures.md).

Enhorabuena, acaba de crear y ejecutar una aplicación lógica periódica basada en programación. 

Para crear otras aplicaciones lógicas que usen el desencadenador **Periodicidad**, consulte estas plantillas, que están disponibles tras crear una aplicación lógica:

* Enviarle recordatorios diarios.
* Eliminar blobs de Azure anteriores.
* Agregar mensaje a una cola de Azure Storage.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite el ejemplo de aplicación lógica, elimine el grupo de recursos que contiene la aplicación lógica y los recursos relacionados. 

1. En el menú principal de Azure, vaya a **Grupos de recursos** y seleccione el grupo de recursos de la aplicación lógica.

1. En el menú del grupo de recursos, seleccione **Información general** > **Eliminar grupo de recursos**. 

   !["Introducción" > "Eliminar grupo de recursos"](./media/tutorial-build-scheduled-recurring-logic-app-workflow/delete-resource-group.png)

1. Escriba el nombre del grupo de recursos como confirmación y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha creado una aplicación de lógica que comprueba el tráfico según una programación definida (por las mañanas los días de entre semana) y realiza alguna acción (envío de correo electrónico) cuando el tiempo de desplazamiento supera un límite especificado. Ahora, aprenda a crear una aplicación lógica que envía solicitudes de lista de correo para su aprobación mediante la integración de servicios de Azure, servicios de Microsoft y otras aplicaciones SaaS.

> [!div class="nextstepaction"]
> [Administración de solicitudes de lista de distribución de correo](../logic-apps/tutorial-process-mailing-list-subscriptions-workflow.md)
