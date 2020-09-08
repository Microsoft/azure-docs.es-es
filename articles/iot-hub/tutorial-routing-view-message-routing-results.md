---
title: Visualización de los resultados de enrutamiento de mensajes de Azure IoT Hub (.NET) | Microsoft Docs
description: Después de configurar todos los recursos que usan la parte 1 del tutorial, agregue la capacidad de enrutar mensajes a Azure Stream Analytics y ver los resultados en Power BI.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 03/25/2018
ms.author: robinsh
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: fb86b25fc24ea505ae258cc44d5554a019723af6
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019205"
---
# <a name="tutorial-part-2---view-the-routed-messages"></a>Tutorial: Parte 2: Visualización de los mensajes enrutados

[!INCLUDE [iot-hub-include-routing-intro](../../includes/iot-hub-include-routing-intro.md)]

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="rules-for-routing-the-messages"></a>Reglas de enrutamiento de los mensajes

Estas son las reglas de enrutamiento de mensajes que se configuraron en la parte 1 del tutorial y que se verán en funcionamiento en esta segunda parte.

|Value |Resultado|
|------|------|
|level="storage" |Se escribe en Azure Storage.|
|level="critical" |Es escribe en una cola de Service Bus. Una aplicación lógica recupera el mensaje de la cola y usa Office 365 para enviarlo por correo electrónico.|
|default |Se muestran estos datos con Power BI.|

Ahora, cree los recursos a los que enrutar los mensajes, ejecute una aplicación para enviar los mensajes al centro y visualice el enrutamiento en acción.

## <a name="create-a-logic-app"></a>Crear una aplicación lógica  

La cola de Service Bus se debe usar para recibir los mensajes designados como críticos. Configure una aplicación lógica que supervise la cola de Service Bus y envíe un correo electrónico cuando se agregue un mensaje a la cola.

1. En [Azure Portal](https://portal.azure.com), seleccione **+ Crear un recurso**. Escriba **lógica app** en el cuadro de búsqueda y presione en Entrar. En los resultados de la búsqueda que se muestran, seleccione Logic App y seleccione **Crear** para pasar al panel **Crear aplicación lógica**. Rellene los campos.

   **Name**: este campo es el nombre de la aplicación lógica. En este tutorial se usa **ContosoLogicApp**.

   **Suscripción**: Seleccione su suscripción a Azure.

   **Grupo de recursos**: seleccione **Usar existente** y el grupo de recursos. En este tutorial se usa **ContosoResources**.

   **Ubicación**: use su ubicación. En este tutorial se usa **Oeste de EE. UU.**

   **Log Analytics**: esta opción debe estar desactivada.

   ![Pantalla de creación de la aplicación lógica](./media/tutorial-routing-view-message-routing-results/create-logic-app.png)

   Seleccione **Crear**. Puede que la aplicación tarde unos minutos en implementarse.

2. Ahora, vaya a la aplicación lógica. La manera más fácil de llegar a la aplicación lógica es seleccionar **Grupos de recursos**, el grupo de recursos (en este tutorial se usa **ContosoResources**) y la aplicación lógica en la lista de recursos. 

    Aparece la página Diseñador de aplicaciones lógicas (es posible que tenga que desplazarse hacia la derecha para ver la página completa). En la página Diseñador de aplicaciones lógicas, desplácese hacia abajo hasta que vea el icono **Aplicación lógica en blanco +** y selecciónelo. La pestaña predeterminada es "Para usted". Si este panel está en blanco, seleccione **Todos** para ver todos los conectores y desencadenadores disponibles.

3. Seleccione **Service Bus** de la lista de conectores.

   ![Lista de conectores](./media/tutorial-routing-view-message-routing-results/logic-app-connectors.png)

4. Se muestra una lista de desencadenadores. Seleccione **Cuando se recibe un mensaje en una cola (autocompletar) / Service Bus**.

   ![Lista de desencadenadores de Service Bus](./media/tutorial-routing-view-message-routing-results/logic-app-triggers.png)

5. En la siguiente pantalla, rellene el campo Nombre de la conexión. En este tutorial se usa **ContosoConnection**.

   ![Configuración de la conexión para la cola de Service Bus](./media/tutorial-routing-view-message-routing-results/logic-app-define-connection.png)

   Seleccione el espacio de nombres de Service Bus. En este tutorial se usa **ContosoSBNamespace**. Cuando se selecciona el espacio de nombres, el portal consulta el espacio de nombres de Service Bus para recuperar las claves. Seleccione **RootManageSharedAccessKey** y **Crear**.

   ![Finalización d la configuración de la conexión](./media/tutorial-routing-view-message-routing-results/logic-app-finish-connection.png)

6. En la siguiente pantalla, seleccione el nombre de la cola (en este tutorial se usa **contososbqueue**) en la lista desplegable. En los restantes campos puede dejar los valores predeterminados.

   ![Opciones de cola](./media/tutorial-routing-view-message-routing-results/logic-app-queue-options.png)

7. Ahora, configure la acción para enviar un correo electrónico cuando se recibe un mensaje en la cola. En el Diseñador de aplicaciones lógicas, seleccione **+ Nuevo paso** para agregar un paso y **Todos** para ver todas las opciones disponibles. En el panel **Elegir una acción**, busque y seleccione **Office 365 Outlook**. En la pantalla de acciones, seleccione **Enviar un correo electrónico / Office 365 Outlook**.  

   ![Opciones de Office 365](./media/tutorial-routing-view-message-routing-results/logic-app-select-outlook.png)

8. Inicie sesión con su cuenta de Office 365 para configurar la conexión. Si se agota el tiempo, simplemente vuelva a intentarlo. Especifique las direcciones de correo electrónico de los destinatarios de los mensajes de correo electrónico. Especifique también el asunto y escriba el mensaje que desea que el destinatario vea en el cuerpo. Para las pruebas, utilice su propia dirección de correo electrónico como destinatario.

   Seleccione **Agregar contenido dinámico** para mostrar el contenido del mensaje que puede incluir. Seleccione **Contenido** (incluirá el mensaje en el correo electrónico).

   ![Opciones de correo electrónico de la aplicación lógica](./media/tutorial-routing-view-message-routing-results/logic-app-send-email.png)

9. Seleccione **Guardar**. Después, cierre el Diseñador de aplicaciones lógicas.

## <a name="set-up-azure-stream-analytics"></a>Configuración de Azure Stream Analytics

Para ver los datos en una visualización de Power BI, primero es preciso configurar un trabajo de Stream Analytics que los recupere. Recuerde que los únicos mensajes que se envían al punto de conexión predeterminado y que el trabajo de Stream Analytics recuperará para la visualización de Power BI son aquellos en que **level** está establecido en **normal**.

### <a name="create-the-stream-analytics-job"></a>Creación del trabajo de Stream Analytics

1. En [Azure Portal](https://portal.azure.com), seleccione **Crear un recurso** > **Internet de las cosas** > **Stream Analytics Job**.

2. Escriba la siguiente información para el trabajo.

   **Nombre del trabajo**: Nombre del trabajo. El nombre debe ser único globalmente. En este tutorial se usa **contosoJob**.

   **Suscripción**: la suscripción de Azure que está utilizando para este tutorial.

   **Grupo de recursos**: use el mismo grupo de recursos que IoT Hub. En este tutorial se usa **ContosoResources**.

   **Ubicación**: use la misma ubicación que en el script de instalación. En este tutorial se usa **Oeste de EE. UU.**

   ![Creación del trabajo de Stream Analytics](./media/tutorial-routing-view-message-routing-results/stream-analytics-create-job.png)

3. Seleccione **Crear** para crear el trabajo. Esto puede tardar unos minutos.

    Para volver al trabajo, seleccione **Grupos de recursos**. En este tutorial se usa **ContosoResources**. Seleccione el grupo de recursos y el trabajo de Stream Analytics en la lista de recursos.

### <a name="add-an-input-to-the-stream-analytics-job"></a>Adición de una entrada al trabajo de Stream Analytics

1. En **Topología de trabajo**, seleccione **Entradas**.

2. En el panel **Entradas**, seleccione **Agregar entrada de flujo** y seleccione IoT Hub. En la pantalla que aparece, rellene los campos siguientes:

   **Alias de entrada**: En este tutorial se usa **contosoinputs**.

   **Seleccionar centro de IoT de entre las suscripciones**: seleccione esta opción de botón de radio.

   **Suscripción**: seleccione la suscripción de Azure que utiliza para este tutorial.

   **IoT Hub**: seleccione el centro de IoT. En este tutorial se usa **ContosoTestHub**.

   **Punto de conexión**: seleccione **Mensajería**. (si selecciona Supervisión de operaciones, obtendrá los datos de telemetría de IoT Hub, en lugar de los datos que va a enviar). 

   **Nombre de directiva de acceso compartido**: Seleccione **Servicio**. El portal rellena automáticamente el campo Clave de directiva de acceso compartido.

   **Grupo de consumidores**: seleccione el grupo de consumidores que configuró en la parte 1 de este tutorial. En este tutorial se usa **contosoconsumers**.
   
   En cuanto al resto de los campos, acepte los valores predeterminados. 

   ![Configuración de las entradas en el trabajo de Stream Analytics](./media/tutorial-routing-view-message-routing-results/stream-analytics-job-inputs.png)

3. Seleccione **Guardar**.

### <a name="add-an-output-to-the-stream-analytics-job"></a>Adición de una salida al trabajo de Stream Analytics

1. En **Topología de trabajo**, seleccione **Salidas**.

2. En el panel **Salidas**, seleccione **Agregar** y **Power BI**. En la pantalla que aparece, rellene los campos siguientes:

   **Alias de salida**: el alias único para la salida. En este tutorial se usa **contosooutputs**. 

   **Nombre del conjunto de datos**: nombre del conjunto de datos que se va a usar en Power BI. En este tutorial se usa **contosodataset** 

   **Nombre de la tabla**: nombre de la tabla que se va a usar en Power BI. En este tutorial se usa **contosotable**.

   En el resto de los campos, acepte los valores predeterminados.

3. Seleccione **Autorizar** e inicie sesión en su cuenta de Power BI (quizá deba intentarlo más de una vez).

   ![Configuración de las salidas en el trabajo de Stream Analytics](./media/tutorial-routing-view-message-routing-results/stream-analytics-job-outputs.png)

4. Seleccione **Guardar**.

### <a name="configure-the-query-of-the-stream-analytics-job"></a>Configuración de la consulta del trabajo de Stream Analytics

1. En **Topología de trabajo**, seleccione **Consulta**.

2. Reemplace `[YourInputAlias]` por el alias de entrada del trabajo. En este tutorial se usa **contosoinputs**.

3. Reemplace `[YourOutputAlias]` por el alias de salida del trabajo. En este tutorial se usa **contosooutputs**.

   ![Configuración de la consulta en el trabajo de Stream Analytics](./media/tutorial-routing-view-message-routing-results/stream-analytics-job-query.png)

4. Seleccione **Guardar**.

5. Cierre el panel Consulta. Volverá a la vista de recursos del grupo de recursos. Seleccione el trabajo de Stream Analytics. Este tutorial lo llama **contosoJob**.

### <a name="run-the-stream-analytics-job"></a>Ejecución del trabajo de Stream Analytics

En el trabajo de Stream Analytics, seleccione **Iniciar** > **Ahora** > **Iniciar**. Una vez que el trabajo se inicia correctamente, su estado cambia de **Detenido** a **En ejecución**.

Para configurar el informe de Power BI, necesita datos, por lo que deberá configurar Power BI después de crear el dispositivo y ejecutar la aplicación de simulación de dispositivo.

## <a name="run-simulated-device-app"></a>Ejecución de una aplicación de simulación de dispositivo

En la parte 1 de este tutorial configuró un dispositivo de simulación, para lo que usó un dispositivo IoT. En esta sección se descargará la aplicación de consola .NET que simula que el dispositivo envía mensajes de dispositivo a la nube a IoT Hub, suponiendo que no descargara ya la aplicación y los recursos en la parte 1.

Esta aplicación envía mensajes para cada uno de los distintos métodos de enrutamiento. En esa descarga hay también una carpeta que contiene la plantilla de Azure Resource Manager completa y el archivo de parámetros, así como los scripts de la CLI de Azure y PowerShell.

Si no descargó los archivos del repositorio en la parte 1 de este tutorial, hágalo ahora desde la [simulación de dispositivo IoT](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Al seleccionar este vínculo se descarga un repositorio que contiene varias aplicaciones; la solución que busca está en iot-hub/Tutorials/Routing/IoT_SimulatedDevice.sln. 

Haga doble clic en el archivo de la solución (IoT_SimulatedDevice.sln) para abrir el código en Visual Studio y abra Program.cs. Sustituya `{your hub name}` por el nombre de host de IoT Hub. El formato del nombre de host de IoT Hub es **{nombre de iot hub} .azure-devices.net**. Para este tutorial, el nombre de host del centro es **ContosoTestHub.azure devices.net**. A continuación, sustituya `{your device key}` por la clave de dispositivo que guardó anteriormente al configurar el dispositivo simulado. 

   ```csharp
        static string s_myDeviceId = "Contoso-Test-Device";
        static string s_iotHubUri = "ContosoTestHub.azure-devices.net";
        // This is the primary key for the device. This is in the portal. 
        // Find your IoT hub in the portal > IoT devices > select your device > copy the key. 
        static string s_deviceKey = "{your device key}";
   ```

## <a name="run-and-test"></a>Ejecución y prueba

Ejecución de la aplicación de consola. Espere unos minutos. Puede ver los mensajes que se envían en la pantalla de la consola de la aplicación.

La aplicación envía un nuevo mensaje de dispositivo a nube a IoT Hub cada segundo. El mensaje contiene un objeto serializado mediante JSON serializado con el identificador de dispositivo, la temperatura, la humedad y el nivel de mensaje, cuyo valor predeterminado es `normal`. Asigna de forma aleatoria un nivel de `critical` o `storage`, lo que hace que el mensaje se enrute a la cuenta de almacenamiento o a la cola de Service Bus (lo que desencadena que la aplicación lógica envíe un correo electrónico). Las lecturas predeterminadas (`normal`) se mostrarán en el informe de BI que se configura a continuación.

Si todo está configurado correctamente, debería verá los siguientes resultados:

1. Empieza a recibir correos electrónicos acerca de mensajes críticos.

   ![Correos resultantes](./media/tutorial-routing-view-message-routing-results/results-in-email.png)

   Este resultado significa que las instrucciones siguientes son reales. 

   * El enrutamiento a la cola de Service Bus funciona correctamente.
   * La aplicación lógica que recupera el mensaje de la cola de Service Bus funciona correctamente.
   * El conector de aplicación lógica con Outlook funciona correctamente. 

2. En [Azure Portal](https://portal.azure.com), seleccione **Grupos de recursos** y un grupo de recursos. En este tutorial se usa **ContosoResources**. 

    Seleccione la cuenta de almacenamiento, **Containers** y el contenedor. En este tutorial se usa **contosoresults**. Debe ver una carpeta y puede explorar en profundidad los directorios hasta que vea uno o varios archivos. Abra cualquiera de ellos; contienen las entradas enrutadas a la cuenta de almacenamiento. 

   ![Archivos resultantes en el almacenamiento](./media/tutorial-routing-view-message-routing-results/results-in-storage.png)

Este resultado significa que la instrucción siguiente es real.

   * El enrutamiento a la cuenta de almacenamiento funciona correctamente.

Con la aplicación en ejecución, configure la visualización de Power BI para ver los mensajes que proceden del enrutamiento predeterminado.

## <a name="set-up-the-power-bi-visualizations"></a>Configuración de las visualizaciones de Power BI

1. Inicie sesión en su cuenta de [Power BI](https://powerbi.microsoft.com/).

2. Vaya a **Áreas de trabajo** y seleccione la que estableció al crear la salida del trabajo de Stream Analytics. En este tutorial se usa **My Workspace**. 

3. Seleccione **Conjuntos de datos**. Si no tiene ningún conjunto de datos, espere unos minutos y vuelva a comprobarlo.

   Debería ver en la lista el conjunto de datos que especificó en el momento de crear la salida para el trabajo de Stream Analytics. En este tutorial se usa **contosodataset** (la primera vez el conjunto de datos puede tardar entre 5 y 10 minutos en mostrarse).

4. En **ACCIONES**, seleccione el primer icono para crear un informe.

   ![Área de trabajo de Power BI con el icono de Acciones e informe resaltado](./media/tutorial-routing-view-message-routing-results/power-bi-actions.png)

5. Cree un gráfico de líneas para mostrar la temperatura en tiempo real en un período determinado.

   * En la página de creación de informes, agregue un gráfico de líneas, para lo que debe seleccionar el icono del gráfico de línea.

     ![Visualizaciones y campos](./media/tutorial-routing-view-message-routing-results/power-bi-visualizations-and-fields.png)

   * En el panel **Campos**, expanda la tabla que especificó en el momento de crear la salida para el trabajo de Stream Analytics. En este tutorial se usa **contosotable**.

   * Arrastre **EventEnqueuedUtcTime** (Hora UTC de evento en cola) al **Eje** en el panel **Visualizaciones**.

   * Arrastre **temperature** (temperatura) a **Valores**.

   Se ha creado un gráfico de líneas. El eje x muestra la fecha y hora en la zona horaria UTC. El eje y muestra la temperatura del sensor.

6. Cree otro gráfico de líneas para mostrar la humedad en tiempo real en un período determinado. Para configurar el segundo gráfico, siga el mismo proceso que para el primero y coloque **EventEnqueuedUtcTime** en el eje "x" y **Axis** y **humidity** en el eje "y" (**Valores**).

   ![Informe final de Power BI con los dos gráficos](./media/tutorial-routing-view-message-routing-results/power-bi-report.png)

7. Para guardar el informe, seleccione **Guardar** y escriba un nombre para el informe si se le solicita.

Debería poder ver los datos en ambos gráficos. Este resultado significa que las instrucciones siguientes son reales:

   * El enrutamiento al punto de conexión predeterminado funciona correctamente.
   * El trabajo de Azure Stream Analytics se transmite correctamente.
   * La visualización de Power BI está configurada correctamente.

Puede actualizar los gráficos para ver los datos más recientes. Para ello, debe seleccionar el botón Actualizar, que se encuentra en la parte superior de la ventana de Power BI. 

## <a name="clean-up-resources"></a>Limpieza de recursos 

Si desea quitar todos los recursos de Azure que ha creado en las dos partes de este tutorial, elimine el grupo de recursos. Esta acción elimina también todos los recursos del grupo. En este caso, quita la instancia de IoT Hub, el espacio de nombres y la cola de Service Bus, la aplicación lógica, la cuenta de almacenamiento y el propio grupo de recursos. También puede eliminar los recursos de Power BI y borrar los mensajes de correo electrónico enviados durante el tutorial.

### <a name="clean-up-resources-in-the-power-bi-visualization"></a>Limpieza de recursos en la visualización de Power BI

Inicie sesión en su cuenta de [Power BI](https://powerbi.microsoft.com/). Vaya a su área de trabajo. En este tutorial se usa **My Workspace**. Para quitar la visualización de Power BI, vaya a Conjuntos de datos y seleccione el icono de la papelera para eliminarlo. En este tutorial se usa **contosodataset** Al quitar el conjunto de datos, se quita también el informe.

### <a name="use-the-azure-cli-to-clean-up-resources"></a>Uso de la CLI de Azure para quitar recursos

Para quitar el grupo de recursos, use el comando [az group delete](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-delete). `$resourceGroup` se estableció en **ContosoResources** al principio de este tutorial.

```azurecli-interactive
az group delete --name $resourceGroup
```

### <a name="use-powershell-to-clean-up-resources"></a>Uso de PowerShell para quitar recursos

Para quitar el grupo de recursos, use el comando [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup). `$resourceGroup` se estableció en **ContosoResources** al principio de este tutorial.

```azurepowershell-interactive
Remove-AzResourceGroup -Name $resourceGroup
```

### <a name="clean-up-test-emails"></a>Limpieza de correos electrónicos de prueba

También puede que desee eliminar los correos electrónicos de la bandeja de entrada que se generaron mediante la aplicación lógica mientras se ejecutaba la aplicación de dispositivo.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial de dos partes ha aprendido a utilizar el enrutamiento de mensajes para enrutar los mensajes de IoT Hub a diferentes destinos mediante la realización de las siguientes tareas.  

**Parte I: Creación de recursos, configuración del enrutamiento de mensajes**
> [!div class="checklist"]
> * Cree los recursos: una instancia de IoT Hub, una cuenta de almacenamiento, una cola de Service Bus y un dispositivo simulado.
> * Configure los puntos de conexión y las rutas de los mensajes en IoT Hub para la cuenta de almacenamiento y la cola de Service Bus.

**Parte II: Envío de mensajes al centro, visualización de los resultados del enrutamiento**
> [!div class="checklist"]
> * Crear una aplicación lógica que se desencadena y se envía un correo electrónico cuando se agrega un mensaje a la cola de Service Bus.
> * Descargar y ejecutar una aplicación que simule que un dispositivo IoT envía mensajes al centro para las diferentes opciones de enrutamiento.
> * Crear una visualización de Power BI para los datos enviados al punto de conexión predeterminado.
> * Vea los resultados...
> * ... en la cola de Service Bus y en los mensajes de correo electrónico.
> * ... en la cuenta de almacenamiento.
> * ... en la visualización de Power BI.

En el siguiente tutorial aprender a administrar el estado de un dispositivo IoT. 

> [!div class="nextstepaction"]
> [Configuración y uso de métricas y diagnósticos con un centro de IoT](tutorial-use-metrics-and-diags.md)
