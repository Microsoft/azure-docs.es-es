---
title: 'Tutorial: Creación de un entorno de Azure Time Series Insights | Microsoft Docs'
description: Aprenda a crear un entorno de Time Series Insights, rellenado con datos de dispositivos simulados.
services: time-series-insights
author: ashannon7
ms.service: time-series-insights
ms.topic: tutorial
ms.date: 04/26/2019
ms.author: anshan
manager: cshankar
ms.custom: seodec18
ms.openlocfilehash: 42a7ba0c66bd603b19d60c7b3407ae5ca80db28e
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2019
ms.locfileid: "65210178"
---
# <a name="tutorial-create-an-azure-time-series-insights-environment"></a>Tutorial: Creación de un entorno de Azure Time Series Insights

Este tutorial le guiará a través del proceso de creación de un entorno de Time Series Insight (TSI), rellenado con datos de dispositivos simulados. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de un entorno de TSI 
> * Crear una solución Simulación de dispositivo que contiene una instancia de IoT Hub
> * Conectar el entorno de TSI con IoT Hub
> * Ejecución de una simulación de dispositivo para transmitir datos al entorno de TSI
> * Comprobar los datos de telemetría simulados

## <a name="video"></a>Vídeo

### <a name="learn-how-to-use-an-azure-iot-solution-accelerator-to-generate-data-and-get-started-with-time-series-insights-br"></a>Aprenda cómo usar un acelerador de soluciones de IoT de Azure para generar datos que se puedan usar para comenzar a trabajar con Time Series Insights. </br>

> [!VIDEO https://www.youtube.com/embed/6ehNf6AJkFo]

## <a name="prerequisites"></a>Requisitos previos

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/).

* La cuenta de inicio de sesión de Azure también debe ser miembro del rol **Propietario** de la suscripción. Para obtener detalles, consulte [Administración del acceso mediante RBAC y Azure Portal](/azure/role-based-access-control/role-assignments-portal).

## <a name="overview"></a>Información general

El entorno de ETI es donde se recopilan y almacenan los datos del dispositivo. Una vez almacenados en el entorno de TSI, puede utilizar el [explorador de TSI](time-series-quickstart.md) y la [API de consulta de TSI](/rest/api/time-series-insights/ga-query-api) para consultar y analizar los datos. IoT Hub es el punto de conexión que utilizan todos los dispositivos (simulados o físicos) para conectar y transmitir datos de forma segura a la nube de Azure. En [Introducción a Time Series Insights](time-series-insights-overview.md), se explica que IoT Hub también sirve como origen de eventos para la transmisión de datos a un entorno de TSI. En este tutorial se utiliza un [acelerador de soluciones de IoT](/azure/iot-accelerators/) para generar y transmitir datos de telemetría de ejemplo a IoT Hub.

>[!TIP]
> Los aceleradores de soluciones de IoT proporcionan soluciones preconfiguradas de nivel empresarial que permiten acelerar el desarrollo de soluciones de IoT personalizadas.

## <a name="create-a-tsi-environment"></a>Creación de un entorno de TSI

En primer lugar, cree un entorno de TSI en la suscripción de Azure:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) mediante la cuenta de suscripción de Azure.  
1. Seleccione **+ Crear un recurso** en la esquina superior izquierda.  
1. Seleccione la categoría **Internet de las cosas** y elija **Time Series Insights**.  

   [![Selección del recurso de entorno de Time Series Insights](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi.png)](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi.png#lightbox)

1. En la página **Entorno de Time Series Insights**, rellene los parámetros necesarios:

   Parámetro|DESCRIPCIÓN
   ---|---
   **Nombre del entorno** | Elija un nombre único para el entorno de TSI. El nombre lo utilizan el explorador de TSI y las API de consulta.
   **Suscripción** | Las suscripciones son contenedores para recursos de Azure. Elija una suscripción para crear el entorno de TSI.
   **Grupos de recursos** | Un grupo de recursos es un contenedor para recursos de Azure. Elija un grupo de recursos existente o cree uno nuevo para el recurso del entorno de TSI.
   **Ubicación** | Elija una región del centro de datos para el entorno de ETI. Para evitar costos de ancho de banda agregados, es mejor mantener el entorno de TSI en la misma región que otros recursos de IoT.
   **SKU de precios** | Elija el rendimiento requerido. Para que los costos y capacidad de inicio sean menores, seleccione `S1`.
   **Capacity** | La capacidad es el multiplicador que se aplica a la tasa de entrada, la capacidad de almacenamiento y el costo asociado con la SKU seleccionada.  Puede cambiar la capacidad después de la creación. Para que los costos sean menores, seleccione una capacidad de 1.

   Cuando finalice, haga clic en **Crear** para comenzar el proceso de aprovisionamiento.

   [![Creación del recurso de entorno de Time Series Insights](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi-params.png)](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi-params.png#lightbox)

1. Compruebe el panel **Notificaciones** para supervisar la finalización de la implementación.  

   [![Implementación correcta del entorno de Time Series Insights](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi-deployment-succeeded.png)](media/tutorial-create-populate-tsi-environment/ap-create-resource-tsi-deployment-succeeded.png#lightbox)

## <a name="create-a-device-simulation"></a>Creación de una simulación de dispositivo

A continuación, cree la solución Simulación de dispositivo, que generará datos de prueba para rellenar su entorno de TSI:

1. En una ventana o pestaña independiente, vaya a [azureiotsolutions.com](https://www.azureiotsolutions.com). Inicie sesión con la misma cuenta de suscripción de Azure y seleccione el acelerador **Simulación de dispositivos**.

   [![Ejecución del acelerador Simulación de dispositivos](media/tutorial-create-populate-tsi-environment/sa-main.png)](media/tutorial-create-populate-tsi-environment/sa-main.png#lightbox)

1. Escriba los parámetros necesarios en la página **Crear la solución Simulación de dispositivos**.

   Parámetro|DESCRIPCIÓN
   ---|---
   **Nombre de la solución** | Un valor único que se utiliza para la creación de un nuevo grupo de recursos. Los recursos de Azure enumerados se crean y se asigna al grupo de recursos.
   **Suscripción** | Especifique la misma suscripción usada para la creación del entorno de TSI, en la sección anterior.
   **Región** | Especifique la misma región usada para la creación del entorno de TSI, en la sección anterior.
   **Implementación de recursos opcionales de Azure** | Deje activado **IoT Hub**, ya que los dispositivos simulados lo utilizarán para conectarse y transmitir datos.

   Cuando termine, haga clic en **Crear solución** para aprovisionar los recursos de Azure de la solución. Este proceso puede tardar de 6 a 7 minutos en completarse.

   [![Aprovisionamiento de la solución de simulación de dispositivos](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution.png)](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution.png#lightbox)

1. Una vez finalizado el aprovisionamiento, el texto sobre la nueva solución cambiará de **Aprovisionando...** a **Listo**:

   >[!IMPORTANT]
   > No haga clic todavía en el botón **Iniciar**. Pero mantenga abierta esta página web, ya que volverá a ella más adelante.

   [![Aprovisionamiento de la solución Simulación de dispositivos completada](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution-dashboard-ready.png)](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution-dashboard-ready.png#lightbox)

1. Ahora vuelva a Azure Portal e inspecciona los recursos recién creados en la suscripción. En la página **Grupos de recursos** del portal, verá que se ha creado un nuevo grupo de recursos mediante el **nombre de solución** proporcionado en el último paso. Tenga en cuenta también todos los recursos creados para admitir la solución Simulación de dispositivos:

   [![Recursos de la solución de simulación de dispositivos](media/tutorial-create-populate-tsi-environment/ap-device-sim-solution-resources.png)](media/tutorial-create-populate-tsi-environment/ap-device-sim-solution-resources.png#lightbox)

## <a name="connect-the-tsi-environment-to-the-iot-hub"></a>Conectar el entorno de TSI con IoT Hub

En este momento, ha aprendido cómo crear dos conjuntos de recursos, cada uno en su propio grupo de recursos:

- Un entorno de TSI vacío.
- Los recursos de la solución Simulación de dispositivos, incluida una instancia de IoT Hub, generados por un acelerador de soluciones.

Recuerde que los dispositivos simulados necesitan conectarse a una instancia de IoT Hub para transmitir datos de dispositivo. Para que los datos fluyan al entorno de TSI, es necesario realizar cambios de configuración tanto en la instancia de IoT Hub como en el entorno de TSI.

### <a name="iot-hub-configuration-define-a-consumer-group"></a>Configuración de IoT Hub: definición de un grupo de consumidores

IoT Hub proporciona varios puntos de conexión para compartir la funcionalidad con otros actores. El punto de conexión "Eventos" proporciona una forma de que otras aplicaciones consuman datos, ya que se transmite por streaming a una instancia de IoT Hub. En concreto, los "grupos de consumidores" proporcionan un mecanismo para que las aplicaciones escuchen y extraigan datos de IoT Hub.

A continuación, defina una nueva propiedad **grupo de consumidores** en el **Punto de conexión de eventos** de IoT Hub de la solución Simulación de dispositivos.

1. En Azure Portal, vaya a la página **Información general** del grupo de recursos que ha creado para la solución Simulación de dispositivo y seleccione el recurso de IoT Hub:

   [![Grupo de recursos de la solución Simulación de dispositivo](media/tutorial-create-populate-tsi-environment/ap-add-iot-hub-consumer-group-view-rg.png)](media/tutorial-create-populate-tsi-environment/ap-add-iot-hub-consumer-group-view-rg.png#lightbox)

   También tome nota del **nombre** del recurso de IoT Hub generado para la solución, ya que se hará referencia a él más adelante.

1. Desplácese hacia abajo y seleccione la página **Puntos de conexión**; después, seleccione el punto de conexión **Eventos**. En la página **Propiedades** del punto de conexión, introduzca un nombre único para el punto de conexión en el grupo de consumidores "$Default" y, después, haga clic en **Guardar**:

   [![Puntos de conexión de IoT Hub de la solución de simulación de dispositivos](media/tutorial-create-populate-tsi-environment/ap-add-iot-hub-consumer-group-create.png)](media/tutorial-create-populate-tsi-environment/ap-add-iot-hub-consumer-group-create.png#lightbox)

### <a name="tsi-configuration-define-an-event-source"></a>Configuración de TSI: definición de un origen de eventos

Conecte ahora el nuevo punto de conexión de eventos **grupo de consumidores** de IoT Hub al entorno de TSI, como un **origen de eventos**.

1. Vaya a la página **Información general** del grupo de recursos que ha creado para el entorno de TSI; después, seleccione el entorno de TSI:

   [![Entorno y grupo de recursos del entorno de TSI](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-view-rg.png)](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-view-rg.png#lightbox)

1. En la página Entorno de TSI, seleccione **Orígenes de eventos**; a continuación, haga clic en **+ Agregar**.

   [![Información general del entorno de TSI](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-add.png)](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-add.png#lightbox)

1. Escriba los parámetros necesarios en la página **Nuevo origen de eventos**.

   Parámetro|DESCRIPCIÓN
   ---|---
   **Nombre de origen de evento** | Requiere un valor único, que se utiliza para denominar el origen del evento.
   **Origen** | Seleccione **IoT Hub**.
   **Opción de importación** | Seleccione la opción predeterminada `Use IoT hub from available subscriptions`. Esta opción hará que la siguiente lista desplegable se llene con las suscripciones disponibles.
   **Suscripción** | Seleccione la misma suscripción en la que creó los recursos de Simulación de dispositivo y de entorno de TSI.
   **Nombre de la instancia de IoT Hub** | De forma predeterminada, debe ser el nombre de la instancia de IoT Hub que ha anotado anteriormente. Si no es así, seleccione la instancia de IoT Hub correcta.
   **Nombre de la directiva del IoT Hub** | seleccione **iothubowner**.
   **Grupo de consumidores de IoT Hub** | De forma predeterminada, debe ser el nombre del grupo de consumidores de IoT Hub que ha creado anteriormente. En caso contrario, seleccione el nombre del grupo de consumidor correcto.
   **Formato de serialización de eventos** | Deje el valor predeterminado de `JSON`.
   **Nombre de propiedad de marca de tiempo** | Especifique `timestamp`.

   Cuando finalice, haga clic en **Crear** para agregar el origen del evento. Cuando regrese a la página **Información general** del grupo de recursos, junto con su recurso de entorno de TSI, verá un nuevo recurso "Origen del evento de Time Series Insights".

   [![Nuevo origen de eventos del entorno de TSI](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-add-event-source.png)](media/tutorial-create-populate-tsi-environment/ap-add-env-event-source-add-event-source.png#lightbox)

## <a name="run-device-simulation-to-stream-data-into-tsi"></a>Ejecución de una simulación de dispositivo para transmitir datos a TSI

Ahora que se ha completado todo el trabajo de configuración, es el momento de rellenar el entorno de TSI con datos de ejemplo de los dispositivos simulados.

Puede que recuerde de la sección [Creación de una simulación de dispositivo](#create-a-device-simulation) que se crearon varios recursos de Azure por el acelerador para admitir la solución. Junto con la instancia de IoT Hub tratada anteriormente, se ha generado una aplicación web de Azure App Service para crear y transmitir telemetría de los dispositivos simulados.

1. Vuelva al [panel de aceleradores de soluciones](https://www.azureiotsolutions.com/Accelerators#dashboard). Inicie sesión de nuevo si es necesario, usando la misma cuenta de Azure que ha usado en este tutorial. Ahora puede hacer clic en el botón **Iniciar** debajo de la solución "Simulación de dispositivos".

     [![Panel de aceleradores de soluciones](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution-dashboard.png)](media/tutorial-create-populate-tsi-environment/sa-create-device-sim-solution-dashboard.png#lightbox)

1. La aplicación web de simulación de dispositivo se iniciará en este punto, y puede tardar varios segundos con la carga inicial. También se le pedirá su consentimiento para conceder a la aplicación web el permiso Iniciar sesión y leer su perfil. Este permiso permite a la aplicación recuperar la información de perfil de usuario necesaria para admitir el funcionamiento de la aplicación.

     [![Consentimiento de la aplicación web de simulación de dispositivos](media/tutorial-create-populate-tsi-environment/sawa-signin-consent.png)](media/tutorial-create-populate-tsi-environment/sawa-signin-consent.png#lightbox)

1. Cuando se carga la página **Configuración de simulación**, escriba los parámetros necesarios.

   Parámetro|DESCRIPCIÓN
   ---|---
   **Target IoT Hub** (IoT Hub de destino) | Seleccione **Use pre-provisioned IoT Hub** (Usar IoT Hub aprovisionado previamente).
   **Device model** (Modelo de dispositivo) | Seleccione **Refrigerador**.
   **Number of devices** (Número de dispositivos)  | Escriba `1000` en **Amount** (Cantidad).
   **Telemetry frequency** (Frecuencia de telemetría) | Escriba `10` segundos.
   **Simulation duration** (Duración de la simulación) | Seleccione **End in:** (Termina en) y escriba `5` minutos.

   Cuando termine, haga clic en **Start Simulation** (Iniciar simulación). La simulación se ejecutará durante un total de 5 minutos, generando datos desde 1000 dispositivos simulados, cada 10 segundos.  

   [![Configuración de la simulación de dispositivos](media/tutorial-create-populate-tsi-environment/sawa-simulation-setup.png)](media/tutorial-create-populate-tsi-environment/sawa-simulation-setup.png#lightbox)

1. Mientras se ejecuta la simulación, verá que los campos **Total messages** (Total de mensajes) y **Messages per second** (Mensajes por segundo) se actualizan, aproximadamente cada 10 segundos. La simulación finalizará después de aproximadamente 5 minutos y volverá a **Configuración de simulación**.

   [![Ejecución de la simulación de dispositivos](media/tutorial-create-populate-tsi-environment/sawa-simulation-running.png)](media/tutorial-create-populate-tsi-environment/sawa-simulation-running.png#lightbox)

## <a name="verify-the-telemetry-data"></a>Comprobación de los datos de telemetría

En esta sección final, va a comprobar que los datos de telemetría se han generado y se han almacenado en el entorno de TSI. Para comprobar los datos, se utiliza el explorador de Time Series Insights, que se utiliza para consultar y analizar datos de telemetría.

1. Vuelva a la página **Información general** del grupo de recursos del entorno de TSI. Seleccione el entorno de TSI.

   [![Entorno y grupo de recursos del entorno de TSI](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-rg.png)](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-rg.png#lightbox)

1. En la página **Información general** del entorno de TSI, haga clic en **URL del explorador de Time Series Insights** para abrir el explorador de TSI.

   [![Explorador de TSI](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-explorer-url.png)](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-explorer-url.png#lightbox)

1. El explorador de TSI se carga y se autentica con su cuenta de Azure Portal. En la vista inicial, se puede ver en el área del gráfico que el entorno de TSI estaba efectivamente relleno con datos de telemetría simulados. Para filtrar por un intervalo de tiempo más reducido, seleccione el menú desplegable en la parte superior izquierda. A continuación, escriba un intervalo de tiempo lo suficientemente amplio como para abarcar la duración de la simulación del dispositivo. A continuación, haga clic en la clase de ampliación de la búsqueda.

   [![Filtro de intervalo de tiempo del explorador de TSI](media/tutorial-create-populate-tsi-environment/tsie-filter-time-range.png)](media/tutorial-create-populate-tsi-environment/tsie-filter-time-range.png#lightbox)

1. La reducción del intervalo de tiempo permite al gráfico acercarse a las distintivas ráfagas de transferencia de datos, a la instancia de IoT Hub y al entorno de TSI. Observe también el texto **Se ha completado la transmisión.** en la parte superior derecha, que muestra el número total de eventos encontrados. También puede arrastrar el control deslizante **Tamaño de intervalo** para controlar la granularidad del gráfico.

   [![Vista filtrada del intervalo de tiempo del explorador de TSI](media/tutorial-create-populate-tsi-environment/tsie-view-time-range.png)](media/tutorial-create-populate-tsi-environment/tsie-view-time-range.png#lightbox)

1. Por último, también puede hacer clic con el botón izquierdo en una región para filtrar un intervalo y, después, hacer clic con el botón derecho y utilizar **Explorar eventos** para mostrar los detalles del evento en la vista tabular **Eventos**.

   [![Vista filtrada y eventos del intervalo de tiempo del explorador de TSI](media/tutorial-create-populate-tsi-environment/tsie-view-time-range-events.png)](media/tutorial-create-populate-tsi-environment/tsie-view-time-range-events.png#lightbox)

## <a name="clean-up-resources"></a>Limpieza de recursos

En este tutorial se crean varios servicios de Azure activos, para admitir la solución Simulación de dispositivo y el entorno de TSI. Si desea abandonar o retrasar la finalización de esta serie de tutoriales, le recomendamos eliminar todos los recursos para no incurrir en costos innecesarios.

En el menú izquierdo de Azure Portal:

1. Haga clic en el icono **Grupos de recursos** y, después, seleccione el grupo de recursos que ha creado para el entorno de TSI. En la parte superior de la página, haga clic en **Eliminar grupo de recursos**, escriba el nombre del grupo de recursos y haga clic en **Eliminar**.

1. Haga clic en el icono **Grupos de recursos** y, a continuación, seleccione el grupo de recursos que creó el acelerador de la solución Simulación de dispositivo. En la parte superior de la página, haga clic en **Eliminar grupo de recursos**, escriba el nombre del grupo de recursos y haga clic en **Eliminar**

## <a name="next-steps"></a>Pasos siguientes

En este tutorial aprendió lo siguiente:

> [!div class="checklist"]
> * Creación de un entorno de TSI 
> * Crear una solución Simulación de dispositivo que contiene una instancia de IoT Hub
> * Conectar el entorno de TSI con IoT Hub
> * Ejecución de una simulación de dispositivo para transmitir datos al entorno de TSI
> * Comprobar los datos de telemetría simulados

Ahora que sabe cómo crear su propio entorno de TSI puede aprender a crear una aplicación web que consuma datos de un entorno de TSI:

> [!div class="nextstepaction"]
> [Creación de una aplicación web de página única de Azure Time Series Insights](tutorial-create-tsi-sample-spa.md)