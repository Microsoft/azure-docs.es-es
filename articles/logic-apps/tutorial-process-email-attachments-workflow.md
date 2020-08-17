---
title: Automatización de tareas con varios servicios de Azure
description: 'Tutorial: Creación de los flujos de trabajo automatizados para procesar correos electrónicos con Azure Logic Apps, Azure Storage y Azure Functions'
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: tutorial
ms.custom: mvc
ms.date: 02/27/2020
ms.openlocfilehash: 925759b63d1225c720ad439f15b82632a4921cbb
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132337"
---
# <a name="tutorial-automate-tasks-to-process-emails-by-using-azure-logic-apps-azure-functions-and-azure-storage"></a>Tutorial: Automatización de tareas para procesar correos electrónicos mediante Azure Logic Apps, Azure Functions y Azure Storage

Azure Logic Apps le ayuda a automatizar los flujos de trabajo y a integrar los datos entre servicios de Azure, servicios de Microsoft, otras aplicaciones de software como servicio (SaaS) y sistemas locales. En este tutorial se muestra cómo puede crear una [aplicación lógica](../logic-apps/logic-apps-overview.md) que controla los correos electrónicos entrantes y sus datos adjuntos. Esta aplicación lógica analiza el contenido de correo electrónico, lo guarda en Azure Storage y envía notificaciones para revisarlo.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar [Azure Storage](../storage/common/storage-introduction.md) y el Explorador de Storage para comprobar los correos electrónicos y los datos adjuntos guardados.
> * Crear una [función de Azure](../azure-functions/functions-overview.md) que elimina el código HTML de los correos electrónicos. Este tutorial incluye el código que puede usar para esta función.
> * Crear una aplicación lógica en blanco.
> * Agregar un desencadenador que supervise los datos adjuntos en los correos electrónicos
> * Agregar una condición que compruebe si los correos electrónicos tienen datos adjuntos
> * Agregar una acción que llama a la función de Azure cuando un correo electrónico tiene datos adjuntos
> * Agregar una acción que crea blobs de almacenamiento para correos electrónicos y datos adjuntos
> * Agregar una acción que envía notificaciones por correo electrónico

Cuando haya terminado, la aplicación lógica se parecerá a este flujo de trabajo, en un alto nivel:

![Aplicación lógica de alto nivel finalizada](./media/tutorial-process-email-attachments-workflow/overview.png)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Una cuenta de correo electrónico con un proveedor de correo electrónico compatible con Logic Apps, como Office 365 Outlook, Outlook.com o Gmail. En el caso de otros proveedores, [consulte la lista de conectores que se muestra aquí](/connectors/).

  Esta aplicación lógica usa una cuenta de Office 365 Outlook. Si utiliza una cuenta de correo electrónico diferente, los pasos generales siguen siendo los mismos pero la interfaz de usuario podría ser ligeramente distinta.

  > [!IMPORTANT]
  > Si quiere usar el conector de Gmail, solo las cuentas empresariales de G-Suite pueden usar este conector sin restricciones en las aplicaciones lógicas. Si tiene una cuenta de consumidor de Gmail, puede usar este conector solo con servicios específicos aprobados por Google, o puede [crear una aplicación cliente de Google para usarla en la autenticación con el conector de Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Para más información, consulte [Directivas de privacidad y seguridad de datos de los conectores de Google en Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

* Descargue e instale el [Explorador de Microsoft Azure Storage](https://storageexplorer.com/), una herramienta que es gratuita. Esta herramienta le ayuda a comprobar que el contenedor de almacenamiento esté configurado correctamente.

## <a name="set-up-storage-to-save-attachments"></a>Configuración del almacenamiento para guardar datos adjuntos

Puede guardar los correos electrónicos entrantes y los datos adjuntos como blobs en un [contenedor de Azure Storage](../storage/common/storage-introduction.md).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con sus credenciales de su cuenta de Azure.

1. Para crear un contenedor de almacenamiento, [cree una cuenta de almacenamiento](../storage/common/storage-account-create.md) con esta configuración en la pestaña **Datos básicos** de Azure Portal:

   | Configuración | Value | Descripción |
   |---------|-------|-------------|
   | **Suscripción** | <*Azure-subscription-name*> | El nombre de la suscripción a Azure |  
   | **Grupos de recursos** | <*Azure-resource-group*> | El nombre del [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) usado para organizar y administrar los recursos relacionados. En este ejemplo se utiliza "LA-tutorial-RG". <p>**Nota:** Existe un grupo de recursos dentro de una región específica. Aunque es posible que los elementos de este tutorial no estén disponibles en todas las regiones, intente usar la misma región siempre que sea posible. |
   | **Nombre de cuenta de almacenamiento** | <*Azure-storage-account-name*> | El nombre de la cuenta de almacenamiento, que tiene que tener entre 3 y 24 caracteres, y solo puede contener letras minúsculas y números. En este ejemplo se usa "attachmentstorageacct". |
   | **Ubicación** | <*Azure-region*> | La región en la que se va a almacenar la información sobre su cuenta de almacenamiento. En este ejemplo se utiliza "Oeste de EE. UU.". |
   | **Rendimiento** | Estándar | Esta configuración especifica los tipos de datos admitidos y los medios para almacenar los datos. Consulte [Tipos de cuentas de almacenamiento](../storage/common/storage-introduction.md#types-of-storage-accounts). |
   | **Tipo de cuenta** | Uso general | El [tipo de cuenta de almacenamiento](../storage/common/storage-introduction.md#types-of-storage-accounts) |
   | **Replicación** | Almacenamiento con redundancia local (LRS) | Esta configuración especifica cómo se copian, se almacenan, se administran y se sincronizan los datos. Consulte [Almacenamiento con redundancia local (LRS): redundancia de datos de bajo costo para Azure Storage](../storage/common/storage-redundancy.md). |
   | **Nivel de acceso (predeterminado)** | Mantenga la configuración actual. |
   ||||

   En la pestaña **Opciones avanzadas**, seleccione este valor:

   | Configuración | Value | Descripción |
   |---------|-------|-------------|
   | **Se requiere transferencia segura** | Disabled | Esta configuración especifica la seguridad que necesitan las solicitudes en las conexiones. Consulte [Exigencia de transferencia segura](../storage/common/storage-require-secure-transfer.md). |
   ||||

   Para crear una cuenta de almacenamiento también puede usar [Azure PowerShell](../storage/common/storage-account-create.md?tabs=powershell) o la [CLI de Azure](../storage/common/storage-account-create.md?tabs=azure-cli).

1. Seleccione **Revisar y crear** cuando haya terminado.

1. Una vez que Azure implemente su cuenta de almacenamiento, búsquela y obtenga la clave para acceder a ella:

   1. En el menú de la cuenta de almacenamiento, en **Configuración**, seleccione **Claves de acceso**.

   1. Copie el nombre de la cuenta de almacenamiento y **key1**; después, guarde dichos valores en un lugar seguro.

      ![Copiar y guardar el nombre y la clave de la cuenta de almacenamiento](./media/tutorial-process-email-attachments-workflow/copy-save-storage-name-key.png)

   Para obtener la clave de acceso de su cuenta de almacenamiento, también puede usar [Azure PowerShell](/powershell/module/az.storage/get-azstorageaccountkey) o la [CLI de Azure](/cli/azure/storage/account/keys?view=azure-cli-latest.md#az-storage-account-keys-list).

1. Cree un contenedor de almacenamiento de blobs para los datos adjuntos de correo electrónico.

   1. En el menú de la cuenta de almacenamiento, seleccione **Información general**. En el panel Información general, seleccione **Contenedores**.

      ![Agregar contenedor de Blob Storage](./media/tutorial-process-email-attachments-workflow/create-storage-container.png)

   1. Después de que se abra la página **Contenedores**, en la barra de herramientas, seleccione **Contenedor**.

   1. En **Nuevo contenedor**, escriba `attachments` como nombre del contenedor. En **Nivel de acceso público**, seleccione **Contenedor (acceso de lectura anónimo para contenedores y blobs)**  > **Aceptar**.

      Cuando haya terminado, puede encontrar el contenedor de almacenamiento en la cuenta de almacenamiento en Azure Portal:

      ![Contenedor de almacenamiento finalizado](./media/tutorial-process-email-attachments-workflow/created-storage-container.png)

   Para crear un contenedor de almacenamiento, también puede usar [Azure PowerShell](/powershell/module/az.storage/new-azstoragecontainer) o la [CLI de Azure](/cli/azure/storage/container?view=azure-cli-latest#az-storage-container-create).

A continuación, conecte el Explorador de Storage a la cuenta de almacenamiento.

## <a name="set-up-storage-explorer"></a>Configurar el Explorador de Storage

Ahora, conecte el Explorador de Storage a la cuenta de almacenamiento para que pueda confirmar que la aplicación lógica puede guardar correctamente los datos adjuntos como blobs en el contenedor de almacenamiento.

1. Inicie Explorador de Microsoft Azure Storage.

   El Explorador de Storage le solicita una conexión a su cuenta de almacenamiento.

1. En el cuadro de diálogo **Conectar a Azure Storage**, seleccione **Usar una clave y un nombre de cuenta de almacenamiento** > **Siguiente**.

   ![Explorador de Storage: conéctese a la cuenta de almacenamiento](./media/tutorial-process-email-attachments-workflow/storage-explorer-choose-storage-account.png)

   > [!TIP]
   > Si no aparece ningún mensaje, en la barra de herramientas Explorador de Storage, seleccione **Agregar una cuenta**.

1. En **Nombre para mostrar**, proporcione un nombre descriptivo para la conexión. En **Account name** (Nombre de cuenta), especifique el nombre de la cuenta de almacenamiento. En **Clave de cuenta**, especifique la clave de acceso que guardó anteriormente y seleccione **Siguiente**.

1. Confirme la información de su conexión y, después, seleccione **Conectar**.

   El Explorador de Storage crea la conexión y muestra la cuenta de almacenamiento en la ventana del Explorador en **Local y asociado** > **Cuentas de Storage**.

1. Para buscar un contenedor de almacenamiento de blobs, en **Cuentas de almacenamiento**, expanda su cuenta de almacenamiento, que es **attachmentstorageacct** aquí, y después expanda **Contenedores de blobs**, donde se encuentra el contenedor **attachments**, por ejemplo:

   ![Explorador de Azure Storage: buscar un contenedor de almacenamiento](./media/tutorial-process-email-attachments-workflow/storage-explorer-check-contianer.png)

A continuación, cree una [función de Azure](../azure-functions/functions-overview.md) que elimine el código HTML del correo electrónico entrante.

## <a name="create-function-to-clean-html"></a>Creación de una función para limpiar el código HTML

Ahora, use el fragmento de código proporcionado en estos pasos para crear una función de Azure que quite el código HTML de cada correo electrónico entrante. De este modo, el contenido del correo electrónico es más limpio y se procesa de manera más fácil. A continuación, puede llamar a esta función desde la aplicación lógica.

1. Antes de crear una función, [cree una aplicación de función](../azure-functions/functions-create-function-app-portal.md) con estos valores de configuración:

   | Configuración | Value | Descripción |
   | ------- | ----- | ----------- |
   | **Nombre de la aplicación** | <*function-app-name*> | El nombre de la aplicación de función, que debe ser único globalmente en Azure. En este ejemplo ya se usa "CleanTextFunctionApp", por lo que debe proporcionar un nombre diferente, como "MyCleanTextFunctionApp-<*su-nombre*>" |
   | **Suscripción** | <*nombre-de-su-suscripción-a-Azure*> | La misma suscripción de Azure que usó anteriormente. |
   | **Grupo de recursos** | LA-Tutorial-RG | El mismo grupo de recursos de Azure que usó anteriormente. |
   | **SISTEMA OPERATIVO** | <*su-sistema-operativo*> | Seleccione el sistema operativo que admita el lenguaje de programación de funciones que prefiera. En este ejemplo, seleccione **Windows**. |
   | **Plan de hospedaje** | Plan de consumo | Esta configuración determina cómo asignar y escalar los recursos, como la potencia de computación, para ejecutar la aplicación de función. Consulte [Comparación de planes de hospedaje](../azure-functions/functions-scale.md). |
   | **Ubicación** | Oeste de EE. UU. | La misma región que usó anteriormente. |
   | **Pila en tiempo de ejecución** | Lenguaje preferido | Seleccione un runtime que admita el lenguaje de programación de funciones que prefiera. Elija **.NET** para funciones de C# y F#. |
   | **Storage** | cleantextfunctionstorageacct | Cree una cuenta de almacenamiento para su aplicación de función. Use solo letras minúsculas y números. <p>**Nota:** Esta cuenta de almacenamiento contiene las aplicaciones de función y se diferencia de la cuenta de almacenamiento creada anteriormente para los datos adjuntos de correo electrónico. |
   | **Application Insights** | Disable | Activa la supervisión de aplicaciones con [Application Insights](../azure-monitor/app/app-insights-overview.md); sin embargo, en este tutorial, mantenga el valor de configuración **Disable** (Desactivado)  > **Apply** (Aplicar). |
   ||||

   Si la aplicación de función no se abre automáticamente tras la implementación, en el cuadro de búsqueda de [Azure Portal](https://portal.azure.com), busque **Function App** y selecciónelo. En **Function App**, seleccione su aplicación de función.

   ![Seleccionar la aplicación de función](./media/tutorial-process-email-attachments-workflow/select-function-app.png)

   En caso contrario, Azure abre automáticamente una aplicación de función como se muestra aquí:

   ![Aplicación de función creada](./media/tutorial-process-email-attachments-workflow/function-app-created.png)

   Para crear una aplicación de función, también puede usar la [CLI de Azure](../azure-functions/functions-create-first-azure-function-azure-cli.md) o [PowerShell y plantillas de Resource Manager](../azure-resource-manager/templates/deploy-powershell.md).

1. En la lista **Aplicaciones de funciones**, expanda la aplicación de función, si no está ya expandida. En su aplicación de función, seleccione **Funciones**. En la barra de herramientas de funciones, elija  **New function** (Nueva función).

   ![Crear una nueva función](./media/tutorial-process-email-attachments-workflow/function-app-new-function.png)

1. En **Choose a template below or go to the quickstart** (Elegir una plantilla a continuación o ir al inicio rápido), seleccione la plantilla **HTTP trigger**.

   ![Seleccionar la plantilla HTTP trigger](./media/tutorial-process-email-attachments-workflow/function-select-httptrigger-csharp-function-template.png)

   Azure crea una función mediante una plantilla específica del lenguaje para una función desencadenada por HTTP.

1. En el panel **New Function** (Nueva función), en **Name** (Name), escriba `RemoveHTMLFunction`. Mantenga **Nivel de autorización** establecido en **Función** y elija **Crear**.

   ![Asignar un nombre a la función](./media/tutorial-process-email-attachments-workflow/function-provide-name.png)

1. Cuando se abra el editor, sustituya el código de la plantilla por este código de ejemplo, que elimina el HTML y devuelve los resultados a la persona que llama:

   ```csharp
   #r "Newtonsoft.Json"

   using System.Net;
   using Microsoft.AspNetCore.Mvc;
   using Microsoft.Extensions.Primitives;
   using Newtonsoft.Json;
   using System.Text.RegularExpressions;

   public static async Task<IActionResult> Run(HttpRequest req, ILogger log) {

      log.LogInformation("HttpWebhook triggered");

      // Parse query parameter
      string emailBodyContent = await new StreamReader(req.Body).ReadToEndAsync();

      // Replace HTML with other characters
      string updatedBody = Regex.Replace(emailBodyContent, "<.*?>", string.Empty);
      updatedBody = updatedBody.Replace("\\r\\n", " ");
      updatedBody = updatedBody.Replace(@"&nbsp;", " ");

      // Return cleaned text
      return (ActionResult)new OkObjectResult(new { updatedBody });
   }
   ```

1. Cuando finalice, seleccione **Guardar**. Para probar la función, en el borde derecho del editor, debajo del icono de la flecha ( **<** ), elija **Probar**.

   ![Abrir el panel "Probar"](./media/tutorial-process-email-attachments-workflow/function-choose-test.png)

1. En el panel **Probar**, en **Cuerpo de la solicitud**, escriba esta línea y elija **Ejecutar**.

   `{"name": "<p><p>Testing my function</br></p></p>"}`

   ![Probar la función](./media/tutorial-process-email-attachments-workflow/function-run-test.png)

   En la ventana **Output** (Salida) se muestra el resultado de la función:

   ```json
   {"updatedBody":"{\"name\": \"Testing my function\"}"}
   ```

Después de comprobar que funciona la función, cree la aplicación lógica. Aunque en este tutorial se muestra cómo crear una función que elimina el código HTML de los mensajes de correo electrónico, Logic Apps también proporciona un conector de **HTML a texto**.

## <a name="create-your-logic-app"></a>Creación de una aplicación lógica

1. En el cuadro de búsqueda de nivel superior de Azure, escriba `logic apps` y seleccione **Logic Apps**.

   ![Buscar y seleccionar "Logic Apps"](./media/tutorial-process-email-attachments-workflow/find-select-logic-apps.png)

1. En el panel **Logic Apps**, seleccione **Agregar**.

   ![Incorporación de una nueva aplicación lógica](./media/tutorial-process-email-attachments-workflow/add-new-logic-app.png)

1. En el panel **Aplicación lógica**, proporcione los detalles sobre la aplicación lógica, como se muestra a continuación. Seleccione **Revisar y crear** cuando haya terminado.

   ![Especificación de información de la aplicación lógica](./media/tutorial-process-email-attachments-workflow/create-logic-app-settings.png)

   | Configuración | Value | Descripción |
   | ------- | ----- | ----------- |
   | **Suscripción** | <*nombre-de-su-suscripción-a-Azure*> | La misma suscripción de Azure que usó anteriormente. |
   | **Grupos de recursos** | LA-Tutorial-RG | El mismo grupo de recursos de Azure que usó anteriormente. |
   | **Nombre de la aplicación lógica** | LA-ProcessAttachment | Nombre de la aplicación lógica |
   | **Seleccione la ubicación**. | Oeste de EE. UU. | La misma región que usó anteriormente. |
   | **Log Analytics** | Desactivado | En este tutorial, seleccione el valor **Desactivado**. |
   ||||

1. Una vez que Azure implemente la aplicación, en la barra de herramientas de Azure, seleccione el icono de Notificaciones y, después, **Ir al recurso**.

   ![En la lista de notificaciones de Azure, seleccione "Ir a recurso".](./media/tutorial-process-email-attachments-workflow/go-to-new-logic-app-resource.png)

1. Una vez que el Diseñador de aplicaciones lógicas se abra, mostrará una página con un vídeo de introducción y plantillas para patrones de aplicaciones lógicas comunes. En **Plantillas**, elija **Blank Logic App**.

   ![Selección de Aplicación lógica en blanco](./media/tutorial-process-email-attachments-workflow/choose-logic-app-template.png)

A continuación, agregue un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts) que escuche los correos electrónicos entrantes que tienen datos adjuntos. Cada aplicación lógica debe comenzar con un desencadenador, que se activa cuando sucede un evento específico o cuando hay nuevos datos que cumplen una condición determinada. Para más información, consulte [Creación de una nueva aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="monitor-incoming-email"></a>Supervisión del correo electrónico entrante

1. En el cuadro de búsqueda del diseñador, escriba `when new email arrives` como filtro. Seleccione este desencadenador para su proveedor de correo electrónico: **Cuando llega un nuevo correo electrónico - <*su proveedor de correo electrónico*>**

   Por ejemplo:

   ![Seleccione este desencadenador para el proveedor de correo electrónico: "Cuando llega un nuevo correo electrónico"](./media/tutorial-process-email-attachments-workflow/add-trigger-when-email-arrives.png)

   * Para las cuentas profesionales o educativas de Azure, seleccione Office 365 Outlook.

   * Para las cuentas de Microsoft personales, seleccione Outlook.com.

1. Si se le piden credenciales, inicie sesión en su cuenta de correo electrónico para que Logic Apps pueda conectarse a ella.

1. Ahora, especifique los criterios que usa el desencadenador para filtrar el correo electrónico nuevo.

   1. Especifique la configuración que se describe a continuación para comprobar los correos electrónicos.

      ![Especificar la carpeta, el intervalo y la frecuencia de comprobación de los correos electrónicos](./media/tutorial-process-email-attachments-workflow/set-up-email-trigger.png)

      | Configuración | Value | Descripción |
      | ------- | ----- | ----------- |
      | **Carpeta** | Bandeja de entrada | La carpeta de correo electrónico para comprobar. |
      | **Tiene datos adjuntos** | Sí | Obtener solo los correos electrónicos con datos adjuntos. <p>**Nota:** El desencadenador no elimina los correos electrónicos de la cuenta, solo comprueba si existen nuevos mensajes y procesa únicamente los correos electrónicos que coinciden con el filtro del asunto. |
      | **Include Attachments** (Incluir datos adjuntos) | Sí | Obtener los datos adjuntos como entrada del flujo de trabajo, en lugar de solamente buscar datos adjuntos. |
      | **Intervalo** | 1 | Número de intervalos que se espera entre comprobaciones |
      | **Frecuencia** | Minute | La unidad de tiempo de cada intervalo entre comprobaciones |
      ||||

   1. En la lista **Agregar nuevo parámetro** lista, seleccione **Filtro de asunto**.

   1. Después de que el cuadro **Filtro de asunto** aparezca en la acción, especifique el asunto como se muestra aquí:

      | Configuración | Value | Descripción |
      | ------- | ----- | ----------- |
      | **Subject Filter** (Filtro de asunto) | `Business Analyst 2 #423501` | El texto para encontrar el asunto del correo electrónico. |
      ||||

1. Para ocultar por ahora los detalles del desencadenador, haga clic dentro de la barra de título del desencadenador.

   ![Contraer la forma para ocultar detalles](./media/tutorial-process-email-attachments-workflow/collapse-trigger-shape.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

   La aplicación lógica está ahora activa, pero no hace otra cosa que comprobar los correos electrónicos. A continuación, agregue una condición que especifique criterios para continuar el flujo de trabajo.

## <a name="check-for-attachments"></a>Comprobar datos adjuntos

Ahora, agregue una condición que seleccione solo los correos electrónicos que tienen datos adjuntos.

1. En el desencadenador, seleccione **Nuevo paso**.

   !["Nuevo paso"](./media/tutorial-process-email-attachments-workflow/add-condition-under-trigger.png)

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `condition`. Seleccione esta acción: **Condition**

   ![Seleccione "Condición"](./media/tutorial-process-email-attachments-workflow/select-condition.png)

   1. Cambie el nombre de la condición por una descripción mejor. En la barra de título de la condición, seleccione el botón de puntos suspensivos ( **...** ) > **Cambiar nombre**.

      ![Cambiar nombre de condición](./media/tutorial-process-email-attachments-workflow/condition-rename.png)

   1. Cambie el nombre de la condición con esta descripción: `If email has attachments and key subject phrase`

1. Cree una condición que busque correos electrónicos que tienen datos adjuntos.

   1. En la primera fila, debajo de **And**, haga clic en el cuadro izquierdo. En la lista de contenido dinámico que aparece, seleccione la propiedad **Tiene datos adjuntos**.

      ![Crear condición](./media/tutorial-process-email-attachments-workflow/build-condition.png)

   1. En el cuadro central, mantenga el operador **es igual que**.

   1. En el cuadro de la derecha, escriba **True** como el valor que se compara con la propiedad **Tiene datos adjuntos** del desencadenador.

      ![Crear condición](./media/tutorial-process-email-attachments-workflow/finished-condition.png)

      Si ambos valores son iguales, el correo electrónico tiene al menos un archivo adjunto, la condición pasa y el flujo de trabajo continúa.

   En la definición de la aplicación lógica subyacente, que se puede ver en la ventana del editor de código, esta condición es similar a la de este ejemplo:

   ```json
   "Condition": {
      "actions": { <actions-to-run-when-condition-passes> },
      "expression": {
         "and": [ {
            "equals": [
               "@triggerBody()?['HasAttachment']",
                 "true"
            ]
         } ]
      },
      "runAfter": {},
      "type": "If"
   }
   ```

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

### <a name="test-your-condition"></a>Prueba de la condición

Ahora, pruebe si la condición funciona correctamente:

1. Si la aplicación lógica no se está ejecutando aún, elija **Ejecutar** en la barra de herramientas del diseñador.

   Este paso inicia manualmente la aplicación lógica sin tener que esperar a que transcurra el intervalo especificado. Sin embargo, no sucede nada hasta que el correo electrónico de prueba llega a la Bandeja de entrada.

1. Envíese un correo electrónico que cumpla este criterio:

   * El asunto del correo electrónico tiene el texto que especificó en el **filtro de asunto** del desencadenador: `Business Analyst 2 #423501`

   * El correo electrónico tiene un archivo adjunto. Por ahora, cree un archivo de texto vacío y asocie ese archivo a su correo electrónico.

   Cuando llega el correo electrónico, la aplicación lógica comprueba si hay datos adjuntos y el texto de asunto especificado. Si pasa la condición, el desencadenador se activa y hace que el motor de Logic Apps cree una instancia de aplicación lógica e inicie el flujo de trabajo.

1. Para comprobar que el desencadenador se ha activado y que la aplicación lógica se ejecutó correctamente, en el menú de la aplicación lógica, elija **Información general**.

   ![Comprobar el desencadenador y el historial de ejecuciones](./media/tutorial-process-email-attachments-workflow/checkpoint-run-history.png)

   Si la aplicación lógica no se ha desencadenado o se ejecuta a pesar de que se ha desencadenado correctamente, consulte [Solución de problemas de la aplicación lógica](../logic-apps/logic-apps-diagnosing-failures.md).

A continuación, defina las acciones que se realizarán para la rama **If true**. Para guardar el correo electrónico junto con los datos adjuntos, elimine el código HTML del cuerpo del correo electrónico y, luego, cree blobs en el contenedor de almacenamiento para el correo electrónico y los datos adjuntos.

> [!NOTE]
> La lógica de aplicación no tiene que hacer nada con la rama **If false** cuando un correo electrónico no tiene datos adjuntos.
> Como ejercicio adicional, cuando termine este tutorial, puede agregar una acción adecuada que quiera realizar con la rama **If false**.

## <a name="call-removehtmlfunction"></a>Llamada a RemoveHTMLFunction

Este paso agrega la función de Azure que creó anteriormente a su aplicación lógica y pasa el contenido del cuerpo del correo electrónico del desencadenador de correo electrónico a su función.

1. En el menú de la aplicación lógica, seleccione **Logic App Designer** (Diseñador de aplicaciones lógicas). En la rama **If true**, seleccione **Agregar una acción**.

   ![Dentro de "If true", agregar acción](./media/tutorial-process-email-attachments-workflow/if-true-add-action.png)

1. En el cuadro de búsqueda, busque "azure functions" y seleccione esta acción: **Elegir una función de Azure - Azure Functions**

   ![Seleccionar una acción para "Choose an Azure function" (Elegir una función de Azure)](./media/tutorial-process-email-attachments-workflow/add-action-azure-function.png)

1. Seleccione la aplicación de función creada anteriormente, que en este ejemplo es `CleanTextFunctionApp`:

   ![Seleccionar la aplicación de función de Azure](./media/tutorial-process-email-attachments-workflow/add-action-select-azure-function-app.png)

1. Ahora seleccione la función: **RemoveHTMLFunction**

   ![Seleccionar la función de Azure](./media/tutorial-process-email-attachments-workflow/add-action-select-azure-function.png)

1. Cambie el nombre de la forma de la función con esta descripción: `Call RemoveHTMLFunction to clean email body`

1. Ahora, especifique la entrada de la función que se va a procesar.

   1. En **Request Body** (Cuerpo de la solicitud), escriba este texto con un espacio al final:

      `{ "emailBody":`

      Mientras trabaja en esta entrada en los pasos siguientes, aparece un error que indica que el código de JSON no es válido, hasta que la entrada tiene el formato JSON correcto. Cuando anteriormente probó esta función, la entrada especificada para esta función usaba notación de objetos JavaScript (JSON). Por lo tanto, el cuerpo de la solicitud también debe usar el mismo formato.

      Además, cuando el cursor está dentro del cuadro **Request body** (Cuerpo de la solicitud), aparece la lista de contenido dinámico, con el fin de que pueda seleccionar los valores de la propiedad disponibles de las acciones anteriores.

   1. En la lista de contenido dinámico, en **Cuando llega un nuevo correo electrónico**, seleccione la propiedad **Body**. Después de esta propiedad, no olvide agregar la llave de cierre: `}`

      ![Especificar el cuerpo de la solicitud para pasar a la función](./media/tutorial-process-email-attachments-workflow/add-email-body-for-function-processing.png)

   Cuando haya terminado, la entrada a la función debe ser parecida a la de este ejemplo:

   ![Cuerpo de la solicitud finalizado para pasar a la función](./media/tutorial-process-email-attachments-workflow/add-email-body-for-function-processing-2.png)

1. Guarde la aplicación lógica.

A continuación, agregue una acción que cree un blob en el contenedor de almacenamiento para que pueda guardar el cuerpo del correo electrónico.

## <a name="create-blob-for-email-body"></a>Creación del blob para el cuerpo del correo electrónico

1. En el bloque **If true** y en la función de Azure, seleccione **Agregar una acción**.

1. En el cuadro de búsqueda, escriba `create blob` como filtro y seleccione esta acción: **Crear un blob**

   ![Agregar acción para crear un blob para el cuerpo del correo electrónico](./media/tutorial-process-email-attachments-workflow/create-blob-action-for-email-body.png)

1. Cree una conexión a la cuenta de almacenamiento con esta configuración, como se muestra y se describe aquí. Seleccione **Crear** cuando haya terminado.

   ![Crear la conexión a la cuenta de almacenamiento](./media/tutorial-process-email-attachments-workflow/create-storage-account-connection-first.png)

   | Configuración | Value | Descripción |
   | ------- | ----- | ----------- |
   | **Nombre de la conexión** | AttachmentStorageConnection | Un nombre descriptivo para la conexión. |
   | **Storage Account** | attachmentstorageacct | El nombre de la cuenta de almacenamiento que creó anteriormente para guardar los datos adjuntos. |
   ||||

1. Cambie el nombre de la acción **Create blob** (Crear blob) con esta descripción:`Create blob for email body`.

1. En la acción **Create blob** (Crear blob), especifique esta información y seleccione estos campos para crear el blob, como se muestra y se describe:

   ![Proporcionar información del blob para el cuerpo del correo electrónico](./media/tutorial-process-email-attachments-workflow/create-blob-for-email-body.png)

   | Configuración | Value | Descripción |
   | ------- | ----- | ----------- |
   | **Folder path** (Ruta de acceso a la carpeta) | /attachments | La ruta de acceso y el nombre del contenedor que creó anteriormente. En este ejemplo, haga clic en el icono de la carpeta y, después, seleccione el contenedor "/attachments". |
   | **Blob name** (Nombre de blob) | Campo **From** (De) | En este ejemplo, utilice el nombre del remitente como nombre del blob. Haga clic dentro de este cuadro para que aparezca la lista de contenido dinámico y, después, seleccione el campo **De** bajo la acción **Cuando llega un nuevo correo electrónico**. |
   | **Blob content** (Contenido del blob) | Campo **Content** (Contenido) | Para este ejemplo, use el cuerpo del correo electrónico sin HTML como contenido del blob. Haga clic dentro de este cuadro para que aparezca la lista de contenido dinámico y, después, seleccione **Body** (Cuerpo) en la acción **Call RemoveHTMLFunction to clean email body** (Llamar a RemoveHTMLFunction para limpiar el cuerpo del correo electrónico). |
   ||||

   Cuando haya terminado, la acción debe ser parecida a la de este ejemplo:

   ![Finalizó la acción "Create blob" (Crear blob)](./media/tutorial-process-email-attachments-workflow/create-blob-for-email-body-done.png)

1. Guarde la aplicación lógica.

### <a name="check-attachment-handling"></a>Comprobación del tratamiento de los datos adjuntos

Ahora compruebe si la aplicación lógica trata los correos electrónicos tal y como ha especificado:

1. Si la aplicación lógica no se está ejecutando aún, elija **Ejecutar** en la barra de herramientas del diseñador.

1. Envíese un correo electrónico que cumpla este criterio:

   * El asunto del correo electrónico tiene el texto que especificó en el **filtro de asunto** del desencadenador: `Business Analyst 2 #423501`

   * El correo electrónico tiene al menos un archivo adjunto. Por ahora, cree un archivo de texto vacío y asocie ese archivo a su correo electrónico.

   * El correo electrónico tiene algún contenido de prueba en el cuerpo, por ejemplo, `Testing my logic app`

   Si la aplicación lógica no se ha desencadenado o se ejecuta a pesar de que se ha desencadenado correctamente, consulte [Solución de problemas de la aplicación lógica](../logic-apps/logic-apps-diagnosing-failures.md).

1. Compruebe que la aplicación lógica ha guardado el correo electrónico en el contenedor de almacenamiento correcto.

   1. En el Explorador de Storage, expanda **Local y asociado** > **Cuentas de almacenamiento** > **attachmentstorageacct (Clave)**  > **Contenedores de blobs** > **datos adjuntos**.

   1. Busque en el contenedor **attachments** (datos adjuntos) el correo electrónico.

      En este momento, solo el correo electrónico aparece en el contenedor porque la aplicación lógica no procesa aún los datos adjuntos.

      ![Comprobar el Explorador de Storage en busca del correo electrónico guardado](./media/tutorial-process-email-attachments-workflow/storage-explorer-saved-email.png)

   1. Cuando haya terminado, elimine el correo electrónico del Explorador de Storage.

1. Si lo desea, para probar la rama **If false**, que en este momento no hace nada, puede enviar un correo electrónico que no cumpla los criterios.

A continuación, agregue un bucle para procesar todos los datos adjuntos de correo electrónico.

## <a name="process-attachments"></a>Procesamiento de los datos adjuntos

Para procesar los datos adjuntos del correo electrónico, agregue un bucle **Para cada uno** al flujo de trabajo de la aplicación lógica.

1. En la forma **Create blob for email body** (Crear blob para el cuerpo del correo electrónico), seleccione **Agregar una acción**.

   ![Agregar bucle "para cada uno"](./media/tutorial-process-email-attachments-workflow/add-for-each-loop.png)

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `for each` como filtro y seleccione esta acción: **For each**

   ![Seleccione "Para cada uno"](./media/tutorial-process-email-attachments-workflow/select-for-each.png)

1. Cambie el nombre del bucle con esta descripción:`For each email attachment`.

1. Ahora, especifique los datos del bucle para procesar. Haga clic en el cuadro **Seleccionar una salida de los pasos anteriores** para que se abra la lista de contenido dinámico y, después, seleccione **Attachments** (Datos adjuntos).

   ![Seleccionar "Attachments" (Datos adjuntos)](./media/tutorial-process-email-attachments-workflow/select-attachments.png)

   El campo **Attachments** (Datos adjuntos) pasa una matriz que contiene todos los datos adjuntos incluidos en un correo electrónico. El bucle **For each** (Para cada uno) repite las acciones en cada elemento que se pasa con la matriz.

1. Guarde la aplicación lógica.

A continuación, agregue la acción que guarda cada archivo adjunto como un blob en el contenedor de almacenamiento **attachments** (datos adjuntos).

## <a name="create-blob-for-each-attachment"></a>Creación de un blob para cada archivo adjunto

1. En el bucle **For each email attachment** (Para cada archivo adjunto del correo electrónico), elija **Add an action** (Agregar una acción) para que pueda especificar la tarea que se realizará en cada archivo adjunto encontrado.

   ![Agregar acción al bucle](./media/tutorial-process-email-attachments-workflow/for-each-add-action.png)

1. En el cuadro de búsqueda, escriba `create blob` como filtro y seleccione esta acción: **Crear un blob**

   ![Agregar acción para crear el blob](./media/tutorial-process-email-attachments-workflow/create-blob-action-for-attachments.png)

1. Cambie el nombre de la acción **Create blob 2** (Crear blob 2) por esta descripción: `Create blob for each email attachment`.

1. En la acción **Create blob for each email attachment** (Crear blob para cada dato adjunto de correo electrónico), proporcione esta información y seleccione las propiedades de cada blob que desee crear como se muestra y se describe:

   ![Proporcionar información de blob](./media/tutorial-process-email-attachments-workflow/create-blob-per-attachment.png)

   | Configuración | Value | Descripción |
   | ------- | ----- | ----------- |
   | **Folder path** (Ruta de acceso a la carpeta) | /attachments | La ruta de acceso y el nombre del contenedor que creó anteriormente. En este ejemplo, haga clic en el icono de la carpeta y, después, seleccione el contenedor "/attachments". |
   | **Blob name** (Nombre de blob) | Campo **Name** (Nombre) | En este ejemplo, utilice el nombre del archivo adjunto como nombre del blob. Haga clic dentro de este cuadro para que aparezca la lista de contenido dinámico y, después, seleccione el campo **Nombre** bajo la acción **Cuando llega un nuevo correo electrónico**. |
   | **Blob content** (Contenido del blob) | Campo **Content** (Contenido) | En este ejemplo, use el campo **Content** (Contenido) como contenido del blob. Haga clic dentro de este cuadro para que aparezca la lista de contenido dinámico y, después, seleccione **Content** (Contenido) bajo la acción **Cuando llega un nuevo correo electrónico**. |
   ||||

   Cuando haya terminado, la acción debe ser parecida a la de este ejemplo:

   ![Finalizó la acción "Create blob" (Crear blob)](./media/tutorial-process-email-attachments-workflow/create-blob-per-attachment-done.png)

1. Guarde la aplicación lógica.

### <a name="check-attachment-handling"></a>Comprobación del tratamiento de los datos adjuntos

A continuación, compruebe si la aplicación lógica trata los datos adjuntos tal y como ha especificado:

1. Si la aplicación lógica no se está ejecutando aún, elija **Ejecutar** en la barra de herramientas del diseñador.

1. Envíese un correo electrónico que cumpla este criterio:

   * El asunto del correo electrónico tiene el texto que especificó en la propiedad **Filtro de asunto** del desencadenador: `Business Analyst 2 #423501`

   * El correo electrónico tiene al menos dos archivos adjuntos. Por ahora, cree dos archivos de texto vacío y adjunte esos archivos a su correo electrónico.

   Si la aplicación lógica no se ha desencadenado o se ejecuta a pesar de que se ha desencadenado correctamente, consulte [Solución de problemas de la aplicación lógica](../logic-apps/logic-apps-diagnosing-failures.md).

1. Compruebe que la aplicación lógica ha guardado el correo electrónico y los datos adjuntos en el contenedor de almacenamiento correcto.

   1. En el Explorador de Storage, expanda **Local y asociado** > **Cuentas de almacenamiento** > **attachmentstorageacct (Clave)**  > **Contenedores de blobs** > **datos adjuntos**.

   1. Compruebe que en el contenedor **attachments** (datos adjuntos) se encuentren el correo electrónico y los datos adjuntos.

      ![Comprobar el correo electrónico y los datos adjuntos guardados](./media/tutorial-process-email-attachments-workflow/storage-explorer-saved-attachments.png)

   1. Cuando haya terminado, elimine el correo electrónico y los datos adjuntos del Explorador de Storage.

A continuación, agregue una acción para que la aplicación lógica envíe un correo electrónico para revisar los datos adjuntos.

## <a name="send-email-notifications"></a>Envío de notificaciones por correo electrónico

1. En la rama **if true**, en el bucle **For each email attachment** (Para cada archivo adjunto de correo electrónico), seleccione **Add an action** (Agregar una acción).

   ![Agregar acción en el bucle "for each" (para cada uno)](./media/tutorial-process-email-attachments-workflow/add-action-send-email.png)

1. En el cuadro de búsqueda, escriba `send email` como filtro y, después, seleccione la acción "send email" para el proveedor de correo electrónico.

   Para filtrar la lista de acciones y encontrar un servicio específico, puede seleccionar primero el conector.

   ![Seleccionar la acción "send email" (enviar correo electrónico) para el proveedor de correo electrónico](./media/tutorial-process-email-attachments-workflow/add-action-select-send-email.png)

   * Para las cuentas profesionales o educativas de Azure, seleccione Office 365 Outlook.

   * Para las cuentas de Microsoft personales, seleccione Outlook.com.

1. Si se le piden credenciales, inicie sesión en su cuenta de correo electrónico para que Logic Apps cree una conexión a su cuenta de correo electrónico.

1. Cambie el nombre de la acción **Send an email** (Enviar un correo electrónico) por esta descripción:`Send email for review`.

1. Especifique la información de esta acción y seleccione los campos que desea incluir en el correo electrónico como se muestra y se describe. Para agregar líneas en blanco en el cuadro de edición, presione Mayús + Entrar.

   ![Enviar notificaciones por correo electrónico](./media/tutorial-process-email-attachments-workflow/send-email-notification.png)

   Si no puede encontrar campo que esperaba que estuviera en la lista de contenido dinámico, seleccione **See more** (Ver más) junto a **When a new email arrives** (Cuando llega un nuevo correo electrónico).

   | Configuración | Value | Notas |
   | ------- | ----- | ----- |
   | **To** | <*recipient-email-address*> | Para realizar pruebas, puede usar su propia dirección de correo electrónico. |
   | **Subject**  | ```ASAP - Review applicant for position:``` **Asunto** | El asunto del correo electrónico que quiere incluir. Haga clic en este cuadro, escriba el texto de ejemplo y, en la lista de contenido dinámico, seleccione el campo **Asunto** en **Cuando llega un nuevo correo electrónico**. |
   | **Cuerpo** | ```Please review new applicant:``` <p>```Applicant name:``` **De** <p>```Application file location:``` **Ruta de acceso** <p>```Application email content:``` **Cuerpo** | El contenido del cuerpo del correo electrónico. Haga clic en este cuadro, escriba el texto de ejemplo y, en la lista de contenido dinámico, seleccione estos campos: <p>- El campo **From** (De) en **When a new email arrives** (Cuando llega un nuevo correo electrónico) </br>- El campo **Path** (Ruta de acceso) en **Create blob for email body** (Crear blob para el cuerpo del correo electrónico) </br>- El campo **Body** (Cuerpo) en **Call RemoveHTMLFunction to clean email body** (Llamar a RemoveHTMLFunction para limpiar el cuerpo del correo electrónico) |
   ||||

   > [!NOTE]
   > Si selecciona un campo que contiene una matriz, como el campo **Content** (Contenido), el diseñador agrega automáticamente un bucle "For each" (Para cada uno) en la acción que haga referencia a ese campo.
   > De este modo, la aplicación lógica realiza la acción en todos los elementos de la matriz.
   > Para quitar el bucle, quite el campo de la matriz, mueva la acción de referencia fuera del bucle, seleccione el botón de puntos suspensivos ( **...** ) en la barra de título del bucle y seleccione **Delete** (Eliminar).

1. Guarde la aplicación lógica.

Ahora, pruebe la aplicación lógica, que es similar a la de este ejemplo:

![Aplicación lógica terminada](./media/tutorial-process-email-attachments-workflow/complete.png)

## <a name="run-your-logic-app"></a>Ejecución de la aplicación lógica

1. Envíese un correo electrónico que cumpla este criterio:

   * El asunto del correo electrónico tiene el texto que especificó en la propiedad **Filtro de asunto** del desencadenador: `Business Analyst 2 #423501`

   * El correo electrónico tiene uno o varios datos adjuntos. Puede volver a usar un archivo de texto vacío de la prueba anterior. Si quiere un escenario más realista, adjunte un archivo de reanudación.

   * El cuerpo del correo electrónico tiene este texto, que puede copiar y pegar:

     ```text

     Name: Jamal Hartnett

     Street address: 12345 Anywhere Road

     City: Any Town

     State or Country: Any State

     Postal code: 00000

     Email address: jamhartnett@outlook.com

     Phone number: 000-000-0000

     Position: Business Analyst 2 #423501

     Technical skills: Dynamics CRM, MySQL, Microsoft SQL Server, JavaScript, Perl, Power BI, Tableau, Microsoft Office: Excel, Visio, Word, PowerPoint, SharePoint, and Outlook

     Professional skills: Data, process, workflow, statistics, risk analysis, modeling; technical writing, expert communicator and presenter, logical and analytical thinker, team builder, mediator, negotiator, self-starter, self-managing  

     Certifications: Six Sigma Green Belt, Lean Project Management

     Language skills: English, Mandarin, Spanish

     Education: Master of Business Administration
     ```

1. Ejecute la aplicación lógica. Si la operación tiene éxito, la aplicación lógica le envía un correo electrónico parecido al de este ejemplo:

   ![Notificación de correo electrónico enviada por la aplicación lógica](./media/tutorial-process-email-attachments-workflow/email-notification.png)

   Si no recibe ningún correo electrónico, compruebe la carpeta de correo electrónico no deseado. El filtro de correo electrónico no deseado podría redirigir esta clase de correo. Si tampoco aparece allí y no está seguro de que la aplicación lógica se ejecutara correctamente, consulte el artículo de [solución de problemas en la aplicación lógica](../logic-apps/logic-apps-diagnosing-failures.md).

Enhorabuena, ahora ha creado y ejecutado una aplicación de lógica que automatiza tareas en diferentes servicios de Azure y llama a código personalizado.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite este ejemplo, elimine el grupo de recursos que contiene la aplicación lógica y los recursos relacionados.

1. En el cuadro de búsqueda de Azure de nivel superior, escriba `resources groups` y seleccione **Grupos de recursos**.

   ![Búsqueda y selección de "Grupos de recursos"](./media/tutorial-process-email-attachments-workflow/find-azure-resource-groups.png)

1. En la lista **Grupos de recursos**, seleccione el grupo de recursos para este tutorial. 

   ![Búsqueda del grupo de recursos para el tutorial](./media/tutorial-process-email-attachments-workflow/find-select-tutorial-resource-group.png)

1. En el panel **Información general**, elija **Eliminar grupo de recursos**.

   ![Eliminar grupo de recursos de aplicación lógica](./media/tutorial-process-email-attachments-workflow/delete-resource-group.png)

1. Cuando aparezca el panel de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una aplicación de lógica que procesa y almacena datos adjuntos de correo electrónico mediante la integración de servicios de Azure, como Azure Storage y Azure Functions. Ahora, conozca más sobre otros conectores que puede usar para crear aplicaciones de lógica.

> [!div class="nextstepaction"]
> [Más información sobre los conectores de Logic Apps](../connectors/apis-list.md)
