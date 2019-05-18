---
title: Llamada a una función de Azure desde Microsoft Flow | Microsoft Docs
description: Cree un conector personalizado y después llame a una función mediante ese conector.
services: functions
keywords: aplicaciones de nube, servicios en la nube, Microsoft Flow, procesos empresariales, aplicación empresarial
author: ggailey777
manager: jeconnoc
ms.assetid: ''
ms.service: azure-functions
ms.topic: conceptual
ms.date: 12/14/2017
ms.author: glenga
ms.reviewer: sunayv
ms.custom: ''
ms.openlocfilehash: d3e777b5611dec382dc4eaaac5ec1594abcdab31
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65787686"
---
# <a name="call-a-function-from-microsoft-flow"></a>Llamada a una función desde Microsoft Flow

[Microsoft Flow](https://flow.microsoft.com/) facilita la automatización de los flujos de trabajo y los procesos empresariales en sus aplicaciones y servicios favoritos. Los desarrolladores profesionales pueden usar Azure Functions para ampliar las funcionalidades de Microsoft Flow, evitándoles los detalles técnicos a los compiladores de flujos.

En este tema se compilará un flujo partiendo de un escenario de mantenimiento de turbinas eólicas. En este tema, se muestra cómo llamar a la función que ha definido en [Create an OpenAPI definition for a function](functions-openapi-definition.md) (Creación de una definición de OpenAPI para una función). La función determina si resulta rentable una reparación de emergencia en una turbina eólica. Si es rentable, el flujo envía un correo electrónico para recomendar la reparación.

Para más información sobre cómo llamar a la misma función desde PowerApps, consulte [Llamada a una función desde PowerApps](functions-powerapps-scenario.md).

En este tema, aprenderá cómo:

> [!div class="checklist"]
> * Crear una lista en SharePoint.
> * Exportar una definición de API.
> * Agregar una conexión a la API.
> * Crear un flujo para enviar correo electrónico si una reparación es rentable.
> * Ejecutar el flujo.

[!INCLUDE [functions-openapi-note](../../includes/functions-openapi-note.md)]

## <a name="prerequisites"></a>Requisitos previos

+ Una [cuenta de Microsoft Flow](https://flow.microsoft.com/documentation/sign-up-sign-in/) activa con las mismas credenciales de inicio de sesión que la cuenta de Azure. 
+ SharePoint, que se usa como un origen de datos para este flujo. Si aún no tiene SharePoint, suscríbase a [una evaluación gratuita de Office 365](https://signup.microsoft.com/Signup?OfferId=467eab54-127b-42d3-b046-3844b860bebf&dl=O365_BUSINESS_PREMIUM&ali=1).
+ Complete el tutorial [Create an OpenAPI definition for a function](functions-openapi-definition.md) (Creación de una definición de OpenAPI para una función).

## <a name="create-a-sharepoint-list"></a>Creación de una lista de SharePoint
Para comenzar, cree una lista que usará como origen de datos en el flujo. La lista tiene las columnas siguientes.

| Columna de la lista     | Tipo de datos           | Notas                                    |
|-----------------|---------------------|------------------------------------------|
| **Título**           | Línea de texto única | Nombre de la turbina                      |
| **LastServiceDate** | Date                |                                          |
| **MaxOutput**       | Número              | Salida de la turbina, en KwH            |
| **ServiceRequired** | Sí/No              |                                          |
| **EstimatedEffort** | Número              | Tiempo estimado de la reparación, en horas |

1. En el sitio de SharePoint, pulse o haga clic en **Nuevo** y, luego, **Lista**.

    ![Creación de una nueva lista de SharePoint](./media/functions-flow-scenario/new-list.png)

2. Escriba el nombre `Turbines` y luego pulse o haga clic en **Crear**.

    ![Especificación de un nombre para la nueva lista](./media/functions-flow-scenario/create-list.png)

    Se crea la lista **Turbines**, con el campo **Título** predeterminado.

    ![Lista de turbinas](./media/functions-flow-scenario/initial-list.png)

3. Pulse o haga clic en el ![icono de Nuevo elemento](./media/functions-flow-scenario/icon-new.png) y luego en **Fecha**.

    ![Adición de una única línea de texto](./media/functions-flow-scenario/add-column.png)

4. Escriba el nombre `LastServiceDate`, a continuación, pulse o haga clic en **Crear**.

    ![Creación de la columna LastServiceDate](./media/functions-flow-scenario/date-column.png)

5. Repita los dos últimos pasos con las otras tres columnas de la lista:

    1. **Número** > "MaxOutput"

    2. **Sí/No** > "ServiceRequired"

    3. **Número** > "EstimatedEffort"

Y eso es todo por ahora. Debería tener una lista vacía parecida a la imagen siguiente. Los datos se agregan a la lista después de crear el flujo.

![Lista vacía](media/functions-flow-scenario/empty-list.png)

[!INCLUDE [Export an API definition](../../includes/functions-export-api-definition.md)]

## <a name="add-a-connection-to-the-api"></a>Agregar una conexión a la API
La API personalizada (también conocida como conector personalizado) está disponible en Microsoft Flow, pero se debe crear una conexión a la API antes de usarla en un flujo.

1. En [flow.microsoft.com](https://flow.microsoft.com), haga clic en el icono de engranaje (en la esquina superior derecha) y lugo haga clic en **Conexiones**.

    ![Conexiones de flujo](media/functions-flow-scenario/flow-connections.png)

1. Haga clic en **Crear conexión**, desplácese hacia abajo hasta el conector **Turbine Repair** (Reparación de turbinas) y haga clic en él.

    ![Crear conexión](media/functions-flow-scenario/create-connection.png)

1. Escriba la clave de API y haga clic en **Crear conexión**.

    ![Escritura de la clave de API y creación de la conexión](media/functions-flow-scenario/api-key.png)

> [!NOTE]
> Si comparte el flujo con otros usuarios, todas las personas que trabajen en la aplicación o lo usen también tendrán que escribir la clave de API para conectarse a la API. Este comportamiento podría cambiar en el futuro y, en ese caso, este tema se actualizaría para reflejar el cambio.


## <a name="create-a-flow"></a>Creación de un flujo

Ahora está listo para crear un flujo que use el conector personalizado y la lista de SharePoint que ha creado.

### <a name="add-a-trigger-and-specify-a-condition"></a>Adición de un desencadenador y especificación de una condición

Primero creará un flujo partiendo de cero (sin una plantilla) y agregará un *desencadenador* que se active cuando se crea un elemento en la lista de SharePoint. A continuación, agregará una *condición* para determinar qué ocurre después.

1. En [flow.microsoft.com](https://flow.microsoft.com), haga clic en **Mis flujos** y luego en **Crear desde cero**.

    ![Crear desde cero](media/functions-flow-scenario/create-from-blank.png)

2. Haga clic en el desencadenador de SharePoint **Cuando se crea un elemento**.

    ![Elegir un desencadenador](media/functions-flow-scenario/choose-trigger.png)

    Si aún no ha iniciado sesión en SharePoint, se le pedirá que lo haga.

3. En **Dirección del sitio**, escriba el nombre de su sitio de SharePoint y, en **Nombre de lista**, escriba la lista que contiene los datos de turbina.

    ![Elegir un desencadenador](media/functions-flow-scenario/site-list.png)

4. Haga clic en **Nuevo paso** y luego en **Agregar una condición**.

    ![Agregar una condición](media/functions-flow-scenario/add-condition.png)

    Microsoft Flow agrega dos ramas al flujo: **En caso afirmativo** y **si no hay ningún**. Después de definir la condición para la que quiere buscar coincidencia, puede agregar pasos a una o ambas ramas.

    ![Ramas de la condición](media/functions-flow-scenario/condition-branches.png)

5. En la tarjeta **Condición**, haga clic en el primer cuadro y seleccione **ServiceRequired** en el cuadro de diálogo **Contenido dinámico**.

    ![Selección del campo de condición](media/functions-flow-scenario/condition1-field.png)

6. Escriba un valor de `True` para la condición.

    ![Especificación de true para la condición](media/functions-flow-scenario/condition1-yes.png)

    El valor se muestra como `Yes` o `No` en la lista de SharePoint, pero se almacena como un valor *booleano*, o bien `True` o `False`. 

7. Haga clic en **Crear flujo** en la parte superior de la página. Asegúrese de hacer clic periódicamente en **Actualizar flujo**.

Para los elementos cargados en la lista, el flujo comprueba si el campo **ServiceRequired** está establecido en `Yes` y luego va a la rama **En caso afirmativo** o a la rama **En caso negativo** según corresponda. Para ahorrar tiempo, en este tema solo se van a especificar acciones para la rama **En caso afirmativo**.

### <a name="add-the-custom-connector"></a>Adición del conector personalizado

Ahora agregará el conector personalizado que llama a la función en Azure. El conector personalizado se agrega al flujo lo mismo que un conector estándar. 

1. En la rama **En caso afirmativo**, haga clic en **Agregar una acción**.

    ![Agregar una acción](media/functions-flow-scenario/condition1-yes-add-action.png)

2. En el cuadro de diálogo **Elegir una acción**, busque `Turbine Repair` y luego seleccione la acción **Turbine Repair - Calculates costs** (Reparación de turbina: calcula costos).

    ![Elegir una acción](media/functions-flow-scenario/choose-turbine-repair.png)

    En la siguiente imagen se muestra la tarjeta que se agrega al flujo. Los campos y las descripciones proceden de la definición de OpenAPI del conector.

    ![Valores predeterminados de Calcula costos](media/functions-flow-scenario/calculates-costs-default.png)

3. En la tarjeta **Calculates costs** (Calcula costos), use el cuadro de diálogo **Contenido dinámico** para seleccionar los valores de entrada de la función. Microsoft Flow muestra los campos numéricos, pero no el de fecha, puesto que la definición de OpenAPI especifica que **Horas** y **Capacidad** son numéricos.

    Para **Horas**, seleccione **EstimatedEffort** y para **Capacidad**, seleccione **MaxOutput**.

    ![Elegir una acción](media/functions-flow-scenario/calculates-costs-fields.png)

     Ahora agregará otra condición basada en la salida de la función.

4. En la parte inferior de la rama **En caso afirmativo**, haga clic en **Más** y luego en **Agregar una condición**.

    ![Agregar una condición](media/functions-flow-scenario/condition2-add.png)

5. En la tarjeta **Condición 2**, haga clic en el primer cuadro y seleccione **Mensaje** en el cuadro de diálogo **Contenido dinámico**.

    ![Selección del campo de condición](media/functions-flow-scenario/condition2-field.png)

6. Escriba un valor de `Yes`. El flujo se dirige a la siguiente rama **En caso afirmativo** o **En caso negativo** según si el mensaje que devuelve la función es afirmativo (se hace la reparación) o negativo (no se hace la reparación). 

    ![Especificación de Sí para la condición](media/functions-flow-scenario/condition2-yes.png)

El flujo debe parecerse ahora a la imagen siguiente.

![Especificación de Sí para la condición](media/functions-flow-scenario/flow-checkpoint1.png)

### <a name="send-email-based-on-function-results"></a>Envío de correo electrónico en función de los resultados de la función

En este momento del flujo, la función ha devuelto un valor de **Mensaje** de `Yes` o `No`, así como otra información sobre los costos y los ingresos potenciales. En la rama **En caso afirmativo** de la segunda condición, enviará un correo electrónico; pero podría realizar muchas otras cosas, como reescribir en la lista de SharePoint o iniciar un [proceso de aprobación](https://flow.microsoft.com/documentation/modern-approvals/).

1. En la rama **En caso afirmativo** de la segunda condición, haga clic en **Agregar una acción**.

    ![Agregar una acción](media/functions-flow-scenario/condition2-yes-add-action.png)

2. En el cuadro de diálogo **Elegir una acción**, busque `email` y luego seleccione una acción de correo electrónico según el sistema de correo electrónico que use (en este caso Outlook).

    ![Envío de un correo electrónico de Outlook](media/functions-flow-scenario/outlook-send-email.png)

3. En la tarjeta **Enviar un correo electrónico**, redacte un mensaje de correo electrónico. En el campo **Para**, escriba un nombre válido de su organización. En la imagen siguiente puede ver que los demás campos son una combinación de texto y tokens del cuadro de diálogo **Contenido dinámico**. 

    ![Campos de correo electrónico](media/functions-flow-scenario/email-fields.png)

    El token **Título** procede de la lista de SharePoint y la función devuelve **CostToFix** y **RevenueOpportunity**.

    El flujo finalizado debe parecerse a la imagen siguiente (se deja fuera la primera rama **En caso negativo** para ahorrar espacio).

    ![Flujo completado](media/functions-flow-scenario/complete-flow.png)

4. Haga clic en **Actualizar flujo** en la parte superior de la página y luego haga clic en **Listo**.

## <a name="run-the-flow"></a>Ejecución del flujo

Ahora que se ha completado el flujo, agregará una fila a la lista de SharePoint y verá cómo responde el flujo.

1. Vuelva a la lista de SharePoint y haga clic en **Edición rápida**.

    ![Edición rápida](media/functions-flow-scenario/quick-edit.png)

2. Escriba los siguientes valores en la cuadrícula de edición.

    | Columna de la lista     | Valor           |
    |-----------------|---------------------|
    | **Título**           | Turbina 60 |
    | **LastServiceDate** | 08/04/2017 |
    | **MaxOutput**       | 2.500 |
    | **ServiceRequired** | Sí |
    | **EstimatedEffort** | 10 |

3. Haga clic en **Done**(Listo).

    ![Edición rápida finalizada](media/functions-flow-scenario/quick-edit-done.png)

    Cuando se agrega el elemento, se desencadena el flujo, que verá a continuación.

4. En [flow.microsoft.com](https://flow.microsoft.com), haga clic en **Mis flujos** y haga clic en el flujo que ha creado.

    ![Mis flujos](media/functions-flow-scenario/my-flows.png)

5. En **Historial de ejecución**, haga clic en la ejecución de flujo.

    ![Hist. de eje.](media/functions-flow-scenario/run-history.png)

    Si la ejecución se realizó correctamente, puede revisar las operaciones de flujo en la siguiente página. Si por algún motivo se produce un error en la ejecución, en la página siguiente se proporciona información de solución de problemas.

6. Expanda las tarjetas para ver lo que ha sucedido durante el flujo. Por ejemplo, expanda la tarjeta **Calculates costs** (Calcula los costos) para ver las entradas y las salidas de la función. 

    ![Entradas y salidas de Calcula los costos](media/functions-flow-scenario/calculates-costs-outputs.png)

7. Compruebe la cuenta de correo electrónico de la persona que especificó en el campo **Para** de la tarjeta **Enviar un correo electrónico**. El correo electrónico enviado desde el flujo debe ser similar a la imagen siguiente.

    ![Correo electrónico de flujo](media/functions-flow-scenario/flow-email.png)

    Puede ver cómo se han reemplazado los tokens por los valores correctos de la lista de SharePoint y la función.

## <a name="next-steps"></a>Pasos siguientes
En este tema, ha aprendido cómo:

> [!div class="checklist"]
> * Crear una lista en SharePoint.
> * Exportar una definición de API.
> * Agregar una conexión a la API.
> * Crear un flujo para enviar correo electrónico si una reparación es rentable.
> * Ejecutar el flujo.

Para aprender más sobre Microsoft Flow, consulte [Introducción a Microsoft Flow](https://flow.microsoft.com/documentation/getting-started/).

Para obtener información sobre otros escenarios interesantes que usan Azure Functions, consulte [Llamada a una función desde PowerApps](functions-powerapps-scenario.md) y [Creación de una función que se integre con Azure Logic Apps](functions-twitter-email.md).
