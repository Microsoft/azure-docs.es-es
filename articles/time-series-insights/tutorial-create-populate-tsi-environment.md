---
title: 'Tutorial: Creación de un entorno: Azure Time Series Insights | Microsoft Docs'
description: Aprenda a crear un entorno de Azure Time Series Insights que se rellena con datos de dispositivos simulados.
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.service: time-series-insights
ms.topic: tutorial
ms.date: 06/30/2020
ms.custom: seodec18
ms.openlocfilehash: 9f74be239bee1d6da3dfdb516c4fc410669e338d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87020658"
---
# <a name="tutorial-create-an-azure-time-series-insights-gen1-environment"></a>Tutorial: Creación de un entorno de Azure Time Series Insights Gen1

Este tutorial le guía a través del proceso de creación de un entorno de Azure Time Series Insights que se rellena con datos de dispositivos simulados. En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Crear un entorno de Azure Time Series Insights.
> * Crear una solución de simulación de dispositivos que contenga un centro de IoT.
> * Conectar el entorno de Azure Time Series Insights al centro de IoT.
> * Ejecutar una simulación de dispositivo para transmitir en secuencias datos al entorno de Azure Time Series Insights.
> * Comprobar los datos de telemetría simulados.

> [!IMPORTANT]
> Regístrese para obtener [suscripción gratuita a Azure](https://azure.microsoft.com/free/), en caso de que no tenga ninguna.

## <a name="prerequisites"></a>Prerrequisitos

* La cuenta de inicio de sesión de Azure también debe ser miembro del rol **Propietario** de la suscripción. Para más información, consulte el artículo sobre [administración del acceso mediante el control de acceso basado en roles y Azure Portal](../role-based-access-control/role-assignments-portal.md).

## <a name="review-video"></a>Revisión del vídeo

### <a name="learn-how-to-use-an-azure-iot-solution-accelerator-to-generate-data-and-get-started-with-azure-time-series-insights-br"></a>Aprenda a usar un acelerador de soluciones de Azure IoT para generar datos y empiece a trabajar con Azure Time Series Insights. </br>

> [!VIDEO https://www.youtube.com/embed/6ehNf6AJkFo]

## <a name="overview"></a>Información general

El entorno de Azure Time Series Insights es donde se recopilan y almacenan los datos del dispositivo. Una vez almacenados, el [explorador de Azure Time Series Insights](time-series-quickstart.md) y la [API de consulta de Azure Time Series Insights](/rest/api/time-series-insights/ga-query-api) se pueden usar para consultar y analizar los datos.

Azure IoT Hub es el origen del evento que usan todos los dispositivos (simulados o físicos) en el tutorial para conectarse de manera segura y transmitir los datos a la nube de Azure.

En este tutorial también se utiliza un [acelerador de soluciones de IoT](https://www.azureiotsolutions.com) para generar y transmitir datos de telemetría de ejemplo a IoT Hub.

>[!TIP]
> Los [aceleradores de soluciones de IoT](https://www.azureiotsolutions.com) proporcionan soluciones preconfiguradas de nivel empresarial que se pueden usar puede usar para acelerar el desarrollo de soluciones de IoT personalizadas.

## <a name="create-a-device-simulation"></a>Creación de una simulación de dispositivo

En primer lugar, cree la solución de simulación de dispositivo, que genera datos de prueba para rellenar el entorno de Azure Time Series Insights.

1. En una ventana o pestaña independientes, vaya a [azureiotsolutions.com](https://www.azureiotsolutions.com). Inicie sesión con la misma cuenta de suscripción de Azure y seleccione el acelerador **Device Simulation** (Simulación de dispositivos).

   [![Ejecución del acelerador Simulación de dispositivos](media/tutorial-create-populate-tsi-environment/iot-solution-accelerators-landing-page.png)](media/tutorial-create-populate-tsi-environment/iot-solution-accelerators-landing-page.png#lightbox)

1. Seleccione **Try Now** (Intentar ahora). A continuación, escriba los parámetros necesarios en la página **Crear la solución Simulación de dispositivos**.

   Parámetro|Descripción
   ---|---
   **Nombre de la implementación** | Este valor único se usa para crear un grupo de recursos. Los recursos de Azure enumerados se crean y se asigna al grupo de recursos.
   **Suscripción de Azure** | Especifique la misma suscripción que se usó para crear el entorno de Azure Time Series Insights en la sección anterior.
   **Opciones de implementación** | Seleccione **Provision new IoT Hub** (Aprovisionar nuevo centro de IoT) para crear un centro de IoT específico para este tutorial.
   **Ubicación de Azure** | Especifique la misma región que se usó para crear el entorno de Azure Time Series Insights en la sección anterior.

   Cuando termine, seleccione **Crear** para aprovisionar los recursos de Azure de la solución. Este proceso puede tardar hasta 20 minutos en completarse.

   [![Aprovisionamiento de la solución de simulación de dispositivos](media/tutorial-create-populate-tsi-environment/iot-solution-accelerators-configuration.png)](media/tutorial-create-populate-tsi-environment/iot-solution-accelerators-configuration.png#lightbox)

1. Una vez finalizado el aprovisionamiento, se mostrarán dos actualizaciones que le notifican que el estado de la implementación ha pasado de **Aprovisionamiento** a **Listo**.

   >[!IMPORTANT]
   > No escriba todavía su acelerador de soluciones. Mantenga abierta esta página web, ya que volverá a ella más adelante.

   [![Aprovisionamiento de la solución Simulación de dispositivos completada](media/tutorial-create-populate-tsi-environment/iot-solution-accelerator-ready.png)](media/tutorial-create-populate-tsi-environment/iot-solution-accelerator-ready.png#lightbox)

1. Ahora, inspeccione los recursos recién creados en Azure Portal. En la página **Grupos de recursos**, se ve que se ha creado un nuevo grupo de recursos, para lo que se ha usado el **nombre de solución** proporcionado en el último paso. Tome nota de los recursos que se crearon para la simulación de dispositivos.

   [![Recursos de la simulación de dispositivos](media/tutorial-create-populate-tsi-environment/tsi-device-sim-solution-resources.png)](media/tutorial-create-populate-tsi-environment/tsi-device-sim-solution-resources.png#lightbox)

## <a name="create-an-environment"></a>Creación de un entorno

En segundo lugar, cree un entorno de Azure Time Series Insights en la suscripción de Azure.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de suscripción de Azure.
1. Seleccione **+ Crear un recurso** en la esquina superior izquierda.
1. Seleccione la categoría **Internet de las cosas** y, después, elija **Time Series Insights**.

   [![Selección del recurso del entorno de Azure Time Series Insights](media/tutorial-create-populate-tsi-environment/tsi-create-new-environment.png)](media/tutorial-create-populate-tsi-environment/tsi-create-new-environment.png#lightbox)

1. En la página **Entorno de Time Series Insights**, rellene los parámetros necesarios.

   Parámetro|Descripción
   ---|---
   **Nombre del entorno** | Elija un nombre único para el entorno de Azure Time Series Insights. El explorador de Time Series Insights y las [API de consulta](https://docs.microsoft.com/rest/api/time-series-insights/ga-query) utilizan los nombres.
   **Suscripción** | Las suscripciones son contenedores para recursos de Azure. Elija una suscripción para crear el entorno de Azure Time Series Insights.
   **Grupos de recursos** | Un grupo de recursos es un contenedor para recursos de Azure. Elija un grupo de recursos existente o cree uno para el recurso del entorno de Azure Time Series Insights.
   **Ubicación** | Elija una región del centro de datos para el entorno de Azure Time Series Insights. Para evitar una mayor latencia, cree el entorno de Azure Time Series Insights en la misma región que los otros recursos de IoT.
   **Nivel** | Elija el rendimiento requerido. Seleccione **S1**.
   **Capacity** | La capacidad es el multiplicador aplicado a la tasa de entrada y la capacidad de almacenamiento asociada a la SKU seleccionada. Puede cambiar la capacidad después de la creación. Seleccione una capacidad de **1**.

   Cuando termine, seleccione **Siguiente: Origen del evento** para ir al paso siguiente.

   [![Creación del recurso del entorno de Azure Time Series Insights](media/tutorial-create-populate-tsi-environment/tsi-create-resource-tsi-params.png)](media/tutorial-create-populate-tsi-environment/tsi-create-resource-tsi-params.png#lightbox)

1. Ahora, conecte el entorno de Azure Time Series Insights al centro de IoT creado por el acelerador de soluciones. Establezca **Seleccionar un centro** en `Select existing`. Luego, elija el centro de IoT creado por el Acelerador de soluciones al establecer el **nombre del centro de IoT**.

   [![Conexión del entorno de Azure Time Series Insights al centro de IoT creado](media/tutorial-create-populate-tsi-environment/tsi-create-resource-iot-hub.png)](media/tutorial-create-populate-tsi-environment/tsi-create-resource-iot-hub.png#lightbox)

   Por último, seleccione **Revisar y crear**.

1. Compruebe el panel **Notificaciones** para supervisar la finalización de la implementación.

   [![Implementación correcta del entorno de Azure Time Series Insights](media/tutorial-create-populate-tsi-environment/create-resource-tsi-deployment-succeeded.png)](media/tutorial-create-populate-tsi-environment/create-resource-tsi-deployment-succeeded.png#lightbox)

## <a name="run-device-simulation"></a>Simulación de ejecución de dispositivo

Ahora que se han completado la implementación y la configuración inicial, rellene el entorno de Azure Time Series Insights con los datos de ejemplo de los [dispositivos simulados que creó el acelerador](#create-a-device-simulation).

Junto con el centro de IoT, se generó una aplicación web de Azure App Service para crear y transmitir la telemetría de los dispositivos simulados.

1. Vuelva al [panel de aceleradores de soluciones](https://www.azureiotsolutions.com/Accelerators#dashboard). Vuelva a iniciar sesión si fuera necesario, y use la misma cuenta de Azure que ha usado en este tutorial. Seleccione la "solución de dispositivo" y, a continuación, **Go to your solution accelerator** (Ir al acelerador de soluciones) para iniciar la solución implementada.

   [![Panel de aceleradores de soluciones](media/tutorial-create-populate-tsi-environment/iot-solution-accelerator-ready.png)](media/tutorial-create-populate-tsi-environment/iot-solution-accelerator-ready.png#lightbox)

1. La aplicación web de simulación de dispositivos empieza pidiéndole que le conceda a la aplicación web el permiso **Iniciar sesión y leer su perfil**. Este permiso permite a la aplicación recuperar la información de perfil de usuario necesaria para admitir el funcionamiento de la aplicación.

   [![Consentimiento de la aplicación web de simulación de dispositivos](media/tutorial-create-populate-tsi-environment/sawa-signin-consent.png)](media/tutorial-create-populate-tsi-environment/sawa-signin-consent.png#lightbox)

1. Seleccione **+ New simulation** (+ Nueva simulación). Después de que se cargue la página **Configuración de simulación**, escriba los parámetros necesarios.

   Parámetro|Descripción
   ---|---
   **Target IoT Hub** (IoT Hub de destino) | Seleccione **Use pre-provisioned IoT Hub** (Usar IoT Hub aprovisionado previamente).
   **Device model** (Modelo de dispositivo) | Seleccione **Refrigerador**.
   **Number of devices** (Número de dispositivos)  | Escriba `10` en **Amount** (Cantidad).
   **Telemetry frequency** (Frecuencia de telemetría) | Escriba `10` segundos.
   **Simulation duration** (Duración de la simulación) | Seleccione **End in:** (Termina en) y escriba `5` minutos.

   Cuando haya terminado, seleccione **Iniciar simulación**. La simulación se ejecuta durante 5 minutos. Genera datos de 1000 dispositivos simulados cada 10 segundos.

   [![Configuración de la simulación de dispositivos](media/tutorial-create-populate-tsi-environment/sawa-simulation-setup.png)](media/tutorial-create-populate-tsi-environment/sawa-simulation-setup.png#lightbox)

1. Mientras se ejecuta la simulación, se ve que los campos **Total messages** (Total de mensajes) y **Messages per second** (Mensajes por segundo) se actualizan, aproximadamente, cada 10 segundos. La simulación finaliza aproximadamente a los 5 minutos y vuelve a **Configuración de simulación**.

   [![Ejecución de la simulación de dispositivos](media/tutorial-create-populate-tsi-environment/sawa-simulation-running.png)](media/tutorial-create-populate-tsi-environment/sawa-simulation-running.png#lightbox)

## <a name="verify-the-telemetry-data"></a>Comprobación de los datos de telemetría

En esta sección final, va a comprobar que los datos de telemetría se generaron y almacenaron en el entorno de Azure Time Series Insights. Para comprobar los datos, se usa el explorador de Azure Time Series Insights, que se utiliza para consultar y analizar datos de telemetría.

1. Vuelva a la página **Información general** del grupo de recursos del entorno de Azure Time Series Insights. Seleccione el entorno de Azure Time Series Insights.

   [![Entorno y grupo de recursos del entorno de Azure Time Series Insights](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-rg.png)](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-rg.png#lightbox)

1. En la página **Información general** del entorno de Azure Time Series Insights, seleccione la **dirección URL del explorador de Time Series Insights** para abrir el explorador de Azure Time Series Insights.

   [![Explorador de Azure Time Series Insights](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-explorer-url.png)](media/tutorial-create-populate-tsi-environment/ap-view-tsi-env-explorer-url.png#lightbox)

1. El explorador de Azure Time Series Insights se carga y se autentica mediante la cuenta de Azure Portal. Al principio, aparece el área del gráfico con la que se rellenó el entorno de Azure Time Series Insights, junto con sus datos de telemetría simulados. Para filtrar por un rango de tiempo más reducido, seleccione el menú desplegable en la esquina superior izquierda. Escriba un rango de tiempo lo suficientemente amplio como para abarcar la duración de la simulación del dispositivo. A continuación, seleccione la lupa de buscar.

   [![Filtro de intervalo de tiempo del explorador de Azure Time Series Insights](media/tutorial-create-populate-tsi-environment/tsie-filter-time-range.png)](media/tutorial-create-populate-tsi-environment/tsie-filter-time-range.png#lightbox)

1. La reducción del intervalo de tiempo permite al gráfico acercarse a las distintas ráfagas de transferencia de datos, al centro de IoT y al entorno de Azure Time Series Insights. Observe también el texto **Se ha completado la transmisión** en la esquina superior derecha, que muestra el número total de eventos encontrados. También puede arrastrar el control deslizante **Tamaño de intervalo** para controlar la granularidad del gráfico.

   [![Vista filtrada del intervalo de tiempo del explorador de Azure Time Series Insights](media/tutorial-create-populate-tsi-environment/tsie-view-time-range.png)](media/tutorial-create-populate-tsi-environment/tsie-view-time-range.png#lightbox)

1. Por último, también puede hacer clic en una región para filtrar un rango. Después, haga clic con el botón derecho y utilice **Explorar eventos** para mostrar los detalles del evento en la vista tabular **Eventos**.

   [![Vista filtrada y eventos del intervalo de tiempo del explorador de Azure Time Series Insights](media/tutorial-create-populate-tsi-environment/tsie-view-time-range-events.png)](media/tutorial-create-populate-tsi-environment/tsie-view-time-range-events.png#lightbox)

## <a name="clean-up-resources"></a>Limpieza de recursos

En este tutorial se crean varios servicios de Azure activos para respaldar tanto el entorno de Azure Time Series Insights como la solución de simulación de dispositivos. Para quitarlos, vuelva a Azure Portal.

En el menú de la izquierda de Azure Portal:

1. Seleccione el icono **Grupos de recursos**. Luego, seleccione el grupo de recursos que creó para el entorno de Azure Time Series Insights. En la parte superior de la página, seleccione **Eliminar grupo de recursos**, escriba el nombre del grupo de recursos y seleccione **Eliminar**.

1. Seleccione el icono **Grupos de recursos**. Luego, seleccione el grupo de recursos que creó el acelerador de soluciones de simulación de dispositivos. En la parte superior de la página, seleccione **Eliminar grupo de recursos**, escriba el nombre del grupo de recursos y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
>
> * Crear un entorno de Azure Time Series Insights.
> * Crear una solución de simulación de dispositivos que contenga un centro de IoT.
> * Conectar el entorno de Azure Time Series Insights al centro de IoT.
> * Ejecutar una simulación de dispositivo para transmitir en secuencias datos al entorno de Azure Time Series Insights.
> * Comprobar los datos de telemetría simulados.

Ahora que sabe cómo crear su propio entorno de Azure Time Series Insights, aprenda a crear una aplicación web que consuma datos de este entorno:

> [!div class="nextstepaction"]
> [Consulte ejemplos de visualización del SDK de cliente hospedado](https://tsiclientsample.azurewebsites.net/)
