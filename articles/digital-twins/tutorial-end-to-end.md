---
title: 'Tutorial: Conexión de una solución de un extremo a otro.'
titleSuffix: Azure Digital Twins
description: Tutorial para crear soluciones de Azure Digital Twins de un extremo a otro controladas por los datos de los dispositivos.
author: baanders
ms.author: baanders
ms.date: 4/15/2020
ms.topic: tutorial
ms.service: digital-twins
ms.openlocfilehash: 0b7e277518337072659bf5ccddd3436c05ff5201
ms.sourcegitcommit: 6e1124fc25c3ddb3053b482b0ed33900f46464b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90563818"
---
# <a name="tutorial-build-out-an-end-to-end-solution"></a>Tutorial: Creación de soluciones de un extremo a otro

Para configurar una solución de un extremo a otro controlada por los datos en directo de su entorno, puede conectar su instancia de Azure Digital Twins a otros servicios de Azure para la administración tanto de los dispositivos como de los datos.

En este tutorial:
> [!div class="checklist"]
> * Configurará una instancia de Azure Digital Twins.
> * Obtendrá información acerca del escenario del edificio de ejemplo y creará instancias de los componentes que se han escrito previamente.
> * Usará una aplicación de [Azure Functions](../azure-functions/functions-overview.md) para enrutar los datos de telemetría simulados de un dispositivo de [IoT Hub](../iot-hub/about-iot-hub.md) en las propiedades de gemelos digitales.
> * Propagará los cambios con el **grafo de gemelos**, mediante el procesamiento de las notificaciones de los gemelos digitales con Azure Functions, puntos de conexión y rutas.

[!INCLUDE [Azure Digital Twins tutorial: sample prerequisites](../../includes/digital-twins-tutorial-sample-prereqs.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

### <a name="set-up-cloud-shell-session"></a>Configuración de una sesión de Cloud Shell
[!INCLUDE [Cloud Shell for Azure Digital Twins](../../includes/digital-twins-cloud-shell.md)]

[!INCLUDE [Azure Digital Twins tutorial: configure the sample project](../../includes/digital-twins-tutorial-sample-configure.md)]

## <a name="get-started-with-the-building-scenario"></a>Primeros pasos con el escenario de compilación

El proyecto de ejemplo que se usa en este tutorial representa un **escenario de un edificio** real, que contiene una planta, una habitación y un dispositivo de termostato. Estos componentes se representarán digitalmente en una instancia de Azure Digital Twins, que posteriormente se conectará a [IoT Hub](../iot-hub/about-iot-hub.md), [Event Grid](../event-grid/overview.md) y dos instancias de [Azure Functions](../azure-functions/functions-overview.md) para facilitar el movimiento de los datos.

El siguiente diagrama representa todo el escenario. 

En primer lugar, creará la instancia de Azure Digital Twins (la **sección A** del diagrama) y configurará el flujo de datos de telemetría en los gemelos digitales (**flecha B**); después, configurará la propagación de los datos con el grafo de gemelos (**flecha C**).

:::image type="content" source="media/tutorial-end-to-end/building-scenario.png" alt-text="Gráfico de todo el escenario del edificio. Muestra los datos que fluyen desde un dispositivo a IoT Hub, a través de una función de Azure (flecha B) hasta una instancia de Azure Digital Twins (sección A), y luego salen a través de Event Grid a otra función de Azure para el procesamiento (flecha C)":::

Para recorrer el escenario, interactuará con los componentes de la aplicación escrita previamente que descargó antes.

Estos son los complementos que implementa la aplicación de ejemplo *AdtSampleApp* del escenario del edificio:
* Autenticación de dispositivos 
* Ejemplos de uso del [SDK de .NET (C#)](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core) (se encuentran en *CommandLoop.cs*).
* Interfaz de consola para llamar a la API de Azure Digital Twins.
* *SampleClientApp*: una solución de Azure Digital Twins de ejemplo.
* *SampleFunctionsApp*: una aplicación de Azure Functions que actualiza su grafo de Azure Digital Twins con los datos de telemetría de los eventos de IoT Hub y Azure Digital Twins.

El proyecto de ejemplo también contiene un componente de autorización interactivo. Cada vez que inicie el proyecto, se abrirá una ventana del explorador, en la que se le solicitará que inicie sesión con su cuenta de Azure.

### <a name="instantiate-the-pre-created-twin-graph"></a>Instanciación del grafo de gemelos creado previamente

En primer lugar, usará la solución *AdtSampleApp* del proyecto de ejemplo para crear la parte de Azure Digital Twins del escenario de un extremo a otro (**sección A**):

:::image type="content" source="media/tutorial-end-to-end/building-scenario-a.png" alt-text="Un extracto del gráfico del escenario completo del edificio que resalta la sección A, la instancia de Azure Digital Twins":::

En la ventana de Visual Studio en la que está abierto el proyecto _**AdtE2ESample**_, ejecute el proyecto con este botón de la barra de herramientas:

:::image type="content" source="media/tutorial-end-to-end/start-button-sample.png" alt-text="Botón de inicio de Visual Studio (proyecto SampleClientApp)":::

Se abre una ventana de la consola, se lleva a cabo la autenticación y se espera un comando. En esta consola, ejecute el siguiente comando para crear una instancia de la solución de Azure Digital Twins de ejemplo.

> [!IMPORTANT]
> Si ya tiene gemelos digitales y relaciones en su instancia de Azure Digital Twins, la ejecución de este comando los eliminará y los sustituirá por los gemelos y relaciones del escenario de ejemplo.

```cmd/sh
SetupBuildingScenario
```

La salida de este comando es una serie de mensajes de confirmación cuando se crean y conectan tres [**gemelos digitales**](concepts-twins-graph.md) en su instancia de Azure Digital Twins: una planta llamada *floor1*, una habitación llamada *room21* y un sensor de temperatura llamado *thermostat67*. Estos gemelos digitales representan las entidades que existirían en un entorno real.

Se conectan mediante relaciones en el siguiente [**grafo de gemelos**](concepts-twins-graph.md). El grafo de gemelos representa el entorno como un todo, incluida la forma en que las entidades interactúan entre ellas y se relacionan entre sí.

:::image type="content" source="media/tutorial-end-to-end/building-scenario-graph.png" alt-text="Un grafo que muestra que floor1 contiene room21 y room21 contiene thermostat67" border="false":::

Para comprobar los gemelos que se crearon, ejecute el siguiente comando, que consulta la instancia de Azure Digital Twins conectada en todos los gemelos digitales que contiene:

```cmd/sh
Query
```

Después puede dejar de ejecutar el proyecto. No obstante, mantenga la solución abierta en Visual Studio, ya que la usará más veces en el tutorial.

## <a name="set-up-the-sample-function-app"></a>Configuración de la aplicación de funciones de ejemplo

El siguiente paso es configurar una [aplicación de Azure Functions](../azure-functions/functions-overview.md) que se usará en este tutorial para procesar los datos. La aplicación de funciones, *SampleFunctionsApp*, contiene dos funciones:
* *ProcessHubToDTEvents*: procesa los datos entrantes de IoT Hub y actualiza Azure Digital Twins.
* *ProcessDTRoutedData*: procesa los datos de gemelos digitales y actualiza los gemelos principales de Azure Digital Twins.

En esta sección, publicará la aplicación de funciones previamente escrita y se asegurará de que esta pueda acceder a Azure Digital Twins, asignándole una identidad de Azure Active Directory (Azure AD). Si se realizan estos pasos, el resto del tutorial podrá usar las funciones dentro de la aplicación de funciones. 

De vuelta en la ventana de Visual Studio en la que está abierto el proyecto _**AdtE2ESample**_ la aplicación de funciones se encuentra en el archivo del proyecto _**SampleFunctionsApp**_. Puede verla en el panel *Explorador de soluciones*.

### <a name="update-dependencies"></a>Actualización de las dependencias

Antes de publicar la aplicación, se recomienda asegurarse de que las dependencias están actualizadas y de que tiene la versión más reciente de todos los paquetes incluidos.

En el panel *Explorador de soluciones*, expanda *SampleFunctionsApp > Dependencias*. Haga clic con el botón derecho en *Paquetes* y elija *Administrar paquetes NuGet...* .

:::image type="content" source="media/tutorial-end-to-end/update-dependencies-1.png" alt-text="Visual Studio: Administrar paquetes NuGet para el proyecto SampleFunctionsApp" border="false":::

Se abrirá el Administrador de paquetes NuGet. Seleccione la pestaña *Actualizaciones* y, si hay paquetes que actualizar, active la casilla *Seleccionar todos los paquetes*. A continuación, pulse *Actualizar*.

:::image type="content" source="media/tutorial-end-to-end/update-dependencies-2.png" alt-text="Visual Studio: Seleccionar la actualización de todos los paquetes en el Administrador de paquetes NuGet":::

### <a name="publish-the-app"></a>Publicación de la aplicación

De vuelta a la ventana de Visual Studio en la que el proyecto _**AdtE2ESample**_ está abierto, en el *Explorador de soluciones*, seleccione con el botón derecho el archivo del proyecto _**SampleFunctionsApp**_ y pulse **Publicar**.

:::image type="content" source="media/tutorial-end-to-end/publish-azure-function-1.png" alt-text="Visual Studio: publicar proyecto":::

En la página *Publicar* que aparece a continuación, deje el destino predeterminado **Azure** y presione *Siguiente*. 

Para seleccionar un destino concreto, elija **Azure Function App (Windows)** y presione *Siguiente*.

:::image type="content" source="media/tutorial-end-to-end/publish-azure-function-2.png" alt-text="Publicar una función de Azure en Visual Studio: destino concreto":::

En la página *Functions instance* (Instancia de Functions), elija su suscripción. Se rellenará un cuadro con los *grupos de recursos* de la suscripción.

Seleccione el grupo de recursos de la instancia y presione *+ Create a new Azure Function...* (Crear una función de Azure).

:::image type="content" source="media/tutorial-end-to-end/publish-azure-function-3.png" alt-text="Publicar una función de Azure en Visual Studio: instancia de Functions (antes de la aplicación de funciones)":::

En la ventana *Function App (Windows) - Create new* [Aplicación de funciones (Windows): Crear nueva], rellene los siguientes campos:
* **Name** (Nombre) es el nombre del plan de consumo que Azure va a usar para hospedar la aplicación de Azure Functions. También será el nombre de la aplicación de funciones que contiene la función real. Puede elegir su propio valor o dejar el que se sugiere, que es el predeterminado.
* Asegúrese de que el valor del campo **Subscription** (Suscripción) coincida con la suscripción que desea usar. 
* Asegúrese de que el valor del campo **Resource group** (Grupos de recursos) coincida con el grupo de recursos que desea utilizar.
* En **Plan type** (Tipo de plan), deje *Consumption* (Consumo).
* En el campo **Location** (Ubicación), elija la ubicación del grupo de recursos.
* Cree un recurso de **Azure Storage** mediante el vínculo *New...* (Nuevo...). Establezca la ubicación que coincida con la del grupo de recursos, use los restantes valores predeterminados y pulse "Ok" (Aceptar).

:::image type="content" source="media/tutorial-end-to-end/publish-azure-function-4.png" alt-text="Publicar una función de Azure en Visual Studio: Function App (Windows) - Create new"::: [Aplicación de funciones (Windows): Crear nueva]

Seleccione **Crear**.

Debería volver a la página *Functions instance* (Instancia de Functions), donde la nueva aplicación de funciones ya se ve debajo del grupo de recursos. Pulse *Finish* (Finalizar).

:::image type="content" source="media/tutorial-end-to-end/publish-azure-function-5.png" alt-text="Publicar una función de Azure en Visual Studio: instancia de Functions (después de la aplicación de funciones)":::

En el panel *Publish* (Publicar) que se abre en la ventana principal de Visual Studio, compruebe que toda la información parece correcta y seleccione **Publish** (Publicar).

:::image type="content" source="media/tutorial-end-to-end/publish-azure-function-6.png" alt-text="Publicar una función de Azure en Visual Studio: publicar":::

> [!NOTE]
> Si ve un elemento emergente similar al siguiente: :::image type="content" source="media/tutorial-end-to-end/publish-azure-function-7.png" alt-text="Publicar una función de Azure en Visual Studio: publicar credenciales" border="false":::
> Seleccione **Attempt to retrieve credentials from Azure** (Intentar recuperar credenciales de Azure) y **Save** (Guardar).
>
> Si ve una advertencia en la que se indica que *actualice la versión de Functions en Azure* o que *la versión del entorno de ejecución de Functions no coincide con la versión que se ejecuta en Azure*:
>
> siga las indicaciones para actualizar a la versión más reciente del entorno de ejecución de Azure Functions. Este problema puede producirse si usa una versión de Visual Studio anterior a la recomendada en la sección *Requisitos previos* al principio de este tutorial.

### <a name="assign-permissions-to-the-function-app"></a>Asignación de permisos a la aplicación de funciones

Para permitir que la aplicación de funciones acceda a Azure Digital Twins, el siguiente paso es establecer una configuración de la aplicación, asignar a la aplicación una identidad de Azure AD administrada por el sistema y asignar a esta identidad el rol *Propietario de Azure Digital Twins (versión preliminar)* en la instancia de Azure Digital Twins. Este rol es necesario para cualquier usuario o función que desee realizar muchas actividades en el plano de datos en la instancia. Más información sobre la seguridad y las asignaciones de roles en [*Conceptos: Seguridad para las soluciones de Azure Digital Twins*](concepts-security.md).

En Azure Cloud Shell, use el siguiente comando para establecer una configuración de la aplicación que la aplicación de funciones usará para hacer referencia a la instancia de Azure Digital Twins.

```azurecli
az functionapp config appsettings set -g <your-resource-group> -n <your-App-Service-(function-app)-name> --settings "ADT_SERVICE_URL=<your-Azure-Digital-Twins-instance-URL>"
```

Use el siguiente comando para crear la identidad administrada por el sistema. Anote el valor del campo *principalId* de la salida.

```azurecli
az functionapp identity assign -g <your-resource-group> -n <your-App-Service-(function-app)-name>
```

Use el valor de *principalId* de la salida en el siguiente comando para asignar la identidad de la aplicación de funciones al rol *Propietario de Azure Digital Twins (versión preliminar)* en la instancia de Azure Digital Twins:

```azurecli
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<principal-ID>" --role "Azure Digital Twins Owner (Preview)"
```

El resultado de este comando es la información de salida acerca de la asignación de roles que ha creado. Ahora, la aplicación de funciones tiene los permisos necesarios para acceder a su instancia de Azure Digital Twins.

## <a name="process-simulated-telemetry-from-an-iot-hub-device"></a>Procesamiento de datos de telemetría simulados de un dispositivo de IoT Hub

Los grafos de Azure Digital Twins los controlan los datos de telemetría de los dispositivos reales. 

En este paso, conectará un dispositivo termostato simulado registrado en [IoT Hub](../iot-hub/about-iot-hub.md) al gemelo digital que lo representa en Azure Digital Twins. Cuando el dispositivo simulado emita datos de telemetría, los datos pasarán por la función de Azure *ProcessHubToDTEvents*, que desencadenará la correspondiente actualización en el gemelo digital. De esta forma, el gemelo digital permanece actualizado con los datos del dispositivo real. En Azure Digital Twins, el proceso de dirigir datos de eventos de un lugar a otro se denomina [**enrutamiento de eventos**](concepts-route-events.md).

Esto sucede en esta parte del escenario de un extremo a otro (**flecha B**):

:::image type="content" source="media/tutorial-end-to-end/building-scenario-b.png" alt-text="Un extracto del gráfico del escenario completo del edificio que resalta la flecha B, los elementos antes de Azure Digital Twins: el dispositivo, el centro de IoT y la primera función de Azure":::

Estas son las acciones que se deben realizar para configurar la conexión de este dispositivo:
1. Crear un centro de IoT que administrará el dispositivo simulado.
2. Conectar el centro de IoT a la función de Azure apropiada mediante la configuración de una suscripción al evento.
3. Registrar el dispositivo simulado en el centro de IoT.
4. Ejecutar el dispositivo simulado y generar datos de telemetría.
5. Consultar Azure Digital Twins para ver los resultados en directo

### <a name="create-an-iot-hub-instance"></a>Creación de una instancia de IoT Hub

Azure Digital Twins está diseñado para trabajar con [IoT Hub](../iot-hub/about-iot-hub.md), un servicio de Azure para administrar dispositivos y sus datos. En este paso, configurará un centro de IoT que administrará el dispositivo de ejemplo en este tutorial.

En Azure Cloud Shell, use este comando para crear un centro de IoT:

```azurecli
az iot hub create --name <name-for-your-IoT-hub> -g <your-resource-group> --sku S1
```

La salida de este comando es información sobre el centro de IoT que se ha creado.

Guarde el nombre que asignó al centro de IoT. Lo usará más adelante.

### <a name="connect-the-iot-hub-to-the-azure-function"></a>Conexión del centro de IoT a la función de Azure

A continuación, conecte su centro de IoT a la función de Azure *ProcessHubToDTEvents* en la aplicación de funciones que publicó antes, con el fin de que los datos puedan fluir desde el dispositivo de IoT Hub a través de la función, que actualiza Azure Digital Twins.

Para ello, creara una **suscripción de eventos** en su centro de IoT, con la función de Azure como punto de conexión. Así se "subscribe" la función a los eventos que suceden en IoT Hub.

En [Azure Portal](https://portal.azure.com/), vaya al centro de IoT recién creado, para lo que debe buscar su nombre en la barra de búsqueda superior. Seleccione *Eventos* en el menú del centro y seleccione *+ Suscripción de eventos*.

:::image type="content" source="media/tutorial-end-to-end/event-subscription-1.png" alt-text="Azure Portal: Suscripción de eventos de IoT Hub":::

Aparecerá la página *Crear suscripción de eventos*.

:::image type="content" source="media/tutorial-end-to-end/event-subscription-2.png" alt-text="Azure Portal: Crear suscripción de eventos":::

Rellene los campos como se indica a continuación (no se mencionan los campos rellenos de forma predeterminada):
* *DETALLES DE SUSCRIPCIONES DE EVENTOS* > **Nombre**: asigne un nombre a su suscripción de eventos.
* *DETALLES DEL TEMA* > **Nombre del tema del sistema**: asigne un nombre que se utilizará para el tema del sistema. 
* *TIPOS DE EVENTO* > **Filtro para tipos de evento**: Seleccione *Telemetría de dispositivo* en las opciones de menú.
* *DETALLES DE PUNTO DE CONEXIÓN* > **Tipo de punto de conexión**: Seleccione *Función de Azure* en las opciones del menú.
* *DETALLES DE PUNTO DE CONEXIÓN* > **Punto de conexión**: Haga clic en el vínculo *Seleccione un punto de conexión*. Se abrirá la ventana *Seleccionar la función de Azure*: :::image type="content" source="media/tutorial-end-to-end/event-subscription-3.png" alt-text="Suscripción de eventos de Azure Portal: seleccionar función de Azure" border="false":::
    - Rellene los campos **Suscripción**, **Grupo de recursos**, **Aplicación de funciones** y **Función** (*ProcessHubToDTEvents*). Algunos de estos campos es posible que se rellenen automáticamente después de seleccionar la suscripción.
    - Pulse **Confirmar selección**.

De nuevo en la página *Crear suscripción de eventos*, pulse **Crear**.

### <a name="register-the-simulated-device-with-iot-hub"></a>Registro del dispositivo simulado en el centro de IoT 

En esta sección se crea una representación de un dispositivo en IoT Hub con el identificador *thermostat67*. El dispositivo simulado se conectará a ella y así es como los eventos de telemetría pasan del dispositivo a IoT Hub, donde la función de Azure suscrita del paso anterior está escuchando, lista para seleccionar los eventos y continuar el procesamiento.

En Azure Cloud Shell, cree un dispositivo en IoT Hub con el siguiente comando:

```azurecli
az iot hub device-identity create --device-id thermostat67 --hub-name <your-IoT-hub-name> -g <your-resource-group>
```

La salida es información acerca del dispositivo creado.

### <a name="configure-and-run-the-simulation"></a>Configuración y ejecución de la simulación

A continuación, configure el simulador de dispositivos para enviar datos a su instancia de IoT Hub.

Para empezar, obtenga la *cadena de conexión de IoT Hub* con este comando:

```azurecli
az iot hub show-connection-string -n <your-IoT-hub-name>
```

Luego, obtenga la *cadena de conexión del dispositivo* con este comando:

```azurecli
az iot hub device-identity show-connection-string --device-id thermostat67 --hub-name <your-IoT-hub-name>
```

Conectará estos valores al código de simulador de dispositivo en el proyecto local para conectar el simulador a este centro de IoT y este dispositivo de IoT Hub.

En una nueva ventana de Visual Studio, abra (desde la carpeta de la solución descargada) _Device Simulator (Simulador de dispositivos) > **DeviceSimulator.sln**_.

>[!NOTE]
> Ahora debería tener dos ventanas de Visual Studio, una con _**DeviceSimulator.sln**_ y otra anterior con _**AdtE2ESample.sln**_.

En el panel del *Explorador de soluciones* de esta nueva ventana de Visual Studio, seleccione _DeviceSimulator/**AzureIoTHub.cs**_ para abrirlo en la ventana de edición. Cambie los siguientes valores de la cadena de conexión por los valores que recopiló anteriormente:

```csharp
connectionString = <Iot-hub-connection-string>
deviceConnectionString = <device-connection-string>
```

Guarde el archivo.

Ahora, para ver los resultados de la simulación de datos que ha configurado, ejecute el proyecto **DeviceSimulator** con este botón en la barra de herramientas:

:::image type="content" source="media/tutorial-end-to-end/start-button-simulator.png" alt-text="Botón de inicio de Visual Studio (proyecto DeviceSimulator)":::

Se abrirá una ventana de la consola y se mostrarán los mensajes de los datos de telemetría de temperatura simulados. Estos se envían a IoT Hub, donde la función de Azure los recoge y procesa.

:::image type="content" source="media/tutorial-end-to-end/console-simulator-telemetry.png" alt-text="Salida de la consola del simulador de dispositivos que muestra los datos de telemetría de temperatura que se envían":::

En esta consola no es preciso hacer nada más, solo dejar que se ejecute mientras se completan los pasos siguientes.

### <a name="see-the-results-in-azure-digital-twins"></a>Visualización de los resultados en Azure Digital Twins

La función *ProcessHubToDTEvents* que publicó anteriormente escucha los datos de IoT Hub y llama a una API de Azure Digital Twins para actualizar la propiedad *Temperature* en el gemelo *thermostat67*.

Para ver los datos de Azure Digital Twins, vaya a la ventana de Visual Studio donde está abierto el proyecto _**AdtE2ESample**_ y ejecútelo.

En la ventana de la consola del proyecto que se abre, ejecute el siguiente comando para obtener las temperaturas que se indican en el gemelo digital *thermostat67*:

```cmd
ObserveProperties thermostat67 Temperature
```

Verá que las temperaturas se actualizan en directo *desde la instancia de Azure Digital Twins* y se registran en la consola cada 10 segundos.

:::image type="content" source="media/tutorial-end-to-end/console-digital-twins-telemetry.png" alt-text="Salida de la consola que muestra el registro de mensajes de temperatura del gemelo digital thermostat67":::

Una vez que haya comprobado el correcto funcionamiento, puede dejar de ejecutar ambos proyectos. Mantenga abierta la ventana de Visual Studio, ya que se utilizará durante el resto del tutorial.

## <a name="propagate-azure-digital-twins-events-through-the-graph"></a>Propagación de eventos de Azure Digital Twins a través del grafo

Hasta ahora, en este tutorial ha visto cómo se puede actualizar Azure Digital Twins a partir de datos de dispositivos externos. A continuación, verá cómo se pueden propagar los cambios que se realicen en un gemelo digital mediante el grafo de Azure Digital Twins (es decir, cómo actualizar los gemelos a partir de los datos internos del servicio).

Para ello, usará la función de Azure *ProcessDTRoutedData* para actualizar un gemelo *Room* cuando el gemelo *Thermostat* conectado esté actualizado. Esto sucede en esta parte del escenario de un extremo a otro (**flecha C**):

:::image type="content" source="media/tutorial-end-to-end/building-scenario-c.png" alt-text="Un extracto del gráfico del escenario completo del edificio que resalta la flecha C, los elementos después de Azure Digital Twins: Event Grid y la segunda función de Azure":::

Estas son las acciones que se deben realizar para configurar este flujo de datos:
1. Crear un punto de conexión de Azure Digital Twins que conecte la instancia a Event Grid.
2. Configurar una ruta en Azure Digital Twins para enviar eventos de cambio de propiedades de los gemelos al punto de conexión.
3. Implementar una aplicación de Azure Functions que realice escuchas (a través de [Event Grid](../event-grid/overview.md)) en el punto de conexión y actualice otros gemelos como corresponda.
4. Ejecutar el dispositivo simulado y consultar Azure Digital Twins para ver los resultados en directo.

### <a name="set-up-endpoint"></a>Configuración del punto de conexión

[Event Grid](../event-grid/overview.md) es un servicio de Azure que ayuda a enrutar y enviar eventos procedentes de Azure Services a otros lugares de Azure. Puede crear un [tema de Event Grid](../event-grid/concepts.md) para recopilar determinados eventos de un origen y, después, los suscriptores pueden escuchar el tema para recibir los eventos a medida que llegan.

En esta sección, creará un tema de Event Grid y, después, creará un punto de conexión en Azure Digital Twins que apuntará a ese tema (enviará eventos). 

En Azure Cloud Shell, ejecute el comando siguiente para crear un tema de Event Grid:

```azurecli
az eventgrid topic create -g <your-resource-group> --name <name-for-your-event-grid-topic> -l <region>
```

> [!TIP]
> Para generar una lista de nombres de regiones de Azure que se pueden utilizar en los comandos de la CLI de Azure, ejecute este comando:
> ```azurecli
> az account list-locations -o table
> ```

La salida de este comando es información sobre el tema de Event Grid que ha creado.

Luego, cree un punto de conexión de Azure Digital Twins que apunte al tema de Event Grid. Use el comando siguiente y rellene los campos de marcadores de posición cuando sea necesario:

```azurecli
az dt endpoint create eventgrid --dt-name <your-Azure-Digital-Twins-instance> --eventgrid-resource-group <your-resource-group> --eventgrid-topic <your-event-grid-topic> --endpoint-name <name-for-your-Azure-Digital-Twins-endpoint>
```

La salida de este comando es información sobre el punto de conexión que ha creado.

También puede comprobar que la creación del punto de conexión se ha realizado correctamente ejecutando el siguiente comando para consultar en la instancia de Azure Digital Twins si existe este punto de conexión:

```azurecli
az dt endpoint show --dt-name <your-Azure-Digital-Twins-instance> --endpoint-name <your-Azure-Digital-Twins-endpoint> 
```

Busque el campo `provisioningState` en la salida y compruebe que el valor es "Succeeded" (Correcto). El valor también puede ser "Provisioning" (En aprovisionamiento), lo que significa que el punto de conexión todavía se está creando. En este caso, espere unos segundos y vuelva a ejecutar el comando para comprobar que se ha completado correctamente.

:::image type="content" source="media/tutorial-end-to-end/output-endpoints.png" alt-text="Resultado de la consulta del punto de conexión que muestra el punto de conexión con un provisioningState de Succeeded":::

Guarde los nombres que dio tanto al tema de Event Grid como al punto de conexión de Azure Digital Twins, ya que los usará más adelante.

### <a name="set-up-route"></a>Configuración de una ruta

A continuación, cree una ruta de Azure Digital Twins que envíe eventos al punto de conexión de Azure Digital Twins que acaba de crear.

[!INCLUDE [digital-twins-known-issue-cloud-shell](../../includes/digital-twins-known-issue-cloud-shell.md)]

```azurecli
az dt route create --dt-name <your-Azure-Digital-Twins-instance> --endpoint-name <your-Azure-Digital-Twins-endpoint> --route-name <name-for-your-Azure-Digital-Twins-route>
```

La salida de este comando es información sobre la ruta que ha creado.

>[!NOTE]
>Los puntos de conexión (del paso anterior) deben finalizar el aprovisionamiento antes de poder configurar una ruta de eventos que los use. Si se produce un error en la creación de la ruta porque los puntos de conexión no están listos, espere unos minutos y vuelva a intentarlo.

#### <a name="connect-the-function-to-event-grid"></a>Conexión de la función a Event Grid

A continuación, suscriba la función de Azure *ProcessDTRoutedData* al tema de Event Grid que creó anteriormente, con el fin de que los datos de telemetría puedan fluir desde el gemelo *thermostat67* a través del tema de Event Grid hasta la función, que vuelve a Azure Digital Twins y actualiza el gemelo *room21* en consecuencia.

Para ello, creará una **suscripción de Event Grid** desde el tema de Event Grid a la función de Azure *ProcessDTRoutedData* como punto de conexión.

En [Azure Portal](https://portal.azure.com/), busque el nombre de su tema de Event Grid en la barra de búsqueda superior para ir a él. Seleccione *+ Suscripción de eventos*.

:::image type="content" source="media/tutorial-end-to-end/event-subscription-1b.png" alt-text="Azure Portal: Suscripción de eventos de Event Grid":::

Los pasos para crear esta suscripción de eventos son similares a los que dio cuando suscribió la primera función de Azure a IoT Hub en este mismo tutorial. Esta vez no es preciso especificar *Telemetría del dispositivo* como el tipo de evento que hay que escuchar y que se conectará a otra función de Azure.

En la página *Crear suscripción de eventos*, rellene los campos como se indica a continuación (no se mencionan los campos rellenos de forma predeterminada):
* *DETALLES DE SUSCRIPCIONES DE EVENTOS* > **Nombre**: asigne un nombre a su suscripción de eventos.
* *DETALLES DE PUNTO DE CONEXIÓN* > **Tipo de punto de conexión**: Seleccione *Función de Azure* en las opciones del menú.
* *DETALLES DE PUNTO DE CONEXIÓN* > **Punto de conexión**: Haga clic en el vínculo *Seleccione un punto de conexión*. Se abrirá la ventana *Seleccionar la función de Azure*:
    - Rellene los campos **Suscripción**, **Grupo de recursos**, **Aplicación de funciones** y **Función** (*ProcessDTRoutedData*). Algunos de estos campos es posible que se rellenen automáticamente después de seleccionar la suscripción.
    - Pulse **Confirmar selección**.

De nuevo en la página *Crear suscripción de eventos*, pulse **Crear**.

### <a name="run-the-simulation-and-see-the-results"></a>Ejecución de la simulación y visualización de los resultados

Ya puede ejecutar el simulador de dispositivos para iniciar el nuevo flujo de eventos que ha configurado. Vaya a la ventana de Visual Studio en que esté abierto el proyecto _**DeviceSimulator**_ y ejecútelo.

Igual que pasó cuando ejecutó el simulador de dispositivo, se abrirá una ventana de la consola y se mostrarán los mensajes de los datos de telemetría de temperatura simulados. Estos eventos atraviesan el flujo que configuró anteriormente para actualizar el gemelo *thermostat67* y, después, atraviesan el flujo configurado recientemente para actualizar el gemelo *room21* para que coincidan.

:::image type="content" source="media/tutorial-end-to-end/console-simulator-telemetry.png" alt-text="Salida de la consola del simulador de dispositivos que muestra los datos de telemetría de temperatura que se envían":::

En esta consola no es preciso hacer nada más, solo dejar que se ejecute mientras se completan los pasos siguientes.

Para ver los datos de Azure Digital Twins, vaya a la ventana de Visual Studio donde está abierto el proyecto _**AdtE2ESample**_ y ejecútelo.

En la ventana de la consola del proyecto que se abre, ejecute el siguiente comando para obtener las temperaturas que se indican en **ambos** gemelos digitales, *thermostat67* y *room21*.

```cmd
ObserveProperties thermostat67 Temperature room21 Temperature
```

Verá que las temperaturas se actualizan en directo *desde la instancia de Azure Digital Twins* y se registran en la consola cada 10 segundos. Tenga en cuenta que la temperatura de *room21* se está actualizando para que coincida con las actualizaciones de *thermostat67*.

:::image type="content" source="media/tutorial-end-to-end/console-digital-twins-telemetry-b.png" alt-text="Salida de la consola que muestra el registro de mensajes de temperatura de un termostato y una habitación":::

Una vez que haya comprobado el correcto funcionamiento, puede dejar de ejecutar ambos proyectos. También puede cerrar las ventanas de Visual Studio, ya que ahora se ha completado el tutorial.

## <a name="review"></a>Revisar

Esta es una revisión del escenario que se ha creado en este tutorial.

1. Una instancia de Azure Digital Twins representa de forma digital una planta, una habitación y un termostato (representado por **sección A** en el diagrama siguiente)
2. Los datos de telemetría del dispositivo simulados se envían a IoT Hub, donde la función de Azure *ProcessHubToDTEvents* escucha los eventos de telemetría. La función de Azure *ProcessHubToDTEvents* usa la información de estos eventos para establecer la propiedad *Temperature* en *thermostat67* (**flecha B** en el diagrama).
3. Los eventos de cambio de propiedad de Azure Digital Twins se enrutan a un tema de Event Grid, donde la función de Azure *ProcessDTRoutedData* escucha los eventos. La función de Azure *ProcessDTRoutedData* usa la información de estos eventos para establecer la propiedad *Temperature* en *room21* (**flecha C** en el diagrama).

:::image type="content" source="media/tutorial-end-to-end/building-scenario.png" alt-text="Gráfico de todo el escenario del edificio. Muestra los datos que fluyen desde un dispositivo a IoT Hub, a través de una función de Azure (flecha B) hasta una instancia de Azure Digital Twins (sección A), y luego salen a través de Event Grid a otra función de Azure para el procesamiento (flecha C)":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos creados en este tutorial, siga estos pasos para eliminarlos. 

Con [Azure Cloud Shell](https://shell.azure.com), puede eliminar todos los recursos de Azure de un grupo mediante el comando [az group delete](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-delete). Así se quitan el grupo de recursos, la instancia de Azure Digital Twins, el centro de IoT y el registro del dispositivo del centro, el tema de Event Grid y las suscripciones asociadas, así como la aplicación de Azure Functions, incluidas las funciones y los recursos asociados, como el almacenamiento.

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible. El grupo de recursos y todos los recursos contenidos en él se eliminan permanentemente. Asegúrese de no eliminar por accidente el grupo de recursos o los recursos equivocados. 

```azurecli
az group delete --name <your-resource-group>
```

A continuación, use este comando para eliminar el registro de aplicación de Azure AD que creó para la aplicación cliente:

```azurecli
az ad app delete --id <your-application-ID>
```

Por último, elimine la carpeta de ejemplo del proyecto que descargó en la máquina local.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un escenario de un extremo a otro que muestra la forma en que los datos de dispositivos activos controlan Azure Digital Twins.

A continuación, consulte la documentación sobre conceptos para más información sobre los elementos con los que ha trabajado en el tutorial:

> [!div class="nextstepaction"]
> [*Conceptos: Modelos personalizados*](concepts-models.md)