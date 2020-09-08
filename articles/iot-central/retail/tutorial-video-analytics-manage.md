---
title: 'Tutorial: Supervisión de vídeo mediante la plantilla de aplicación de análisis de vídeo con detección de objetos y movimiento de Azure IoT Central'
description: En este tutorial se muestra cómo usar los paneles de la plantilla de aplicación de análisis de vídeo con detección de objetos y movimiento para administrar las cámaras y supervisar el vídeo.
services: iot-central
ms.service: iot-central
ms.subservice: iot-central-retail
ms.topic: tutorial
ms.author: nandab
author: KishorIoT
ms.date: 07/31/2020
ms.openlocfilehash: 76e72e8fd134c65cc9334e635375cc25e9b09a75
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88719077"
---
# <a name="tutorial-monitor-and-manage-a-video-analytics---object-and-motion-detection-application"></a>Tutorial: Supervisión y administración de una aplicación de análisis de vídeo con detección de objetos y movimiento

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Agregar cámaras de detección de objetos y movimiento a la aplicación de IoT Central.
> * Administrar las secuencias de vídeo y reproducirlas cuando se detecten eventos de interés.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, debe completar:

* El tutorial [Creación de una aplicación de análisis de vídeo en directo en Azure IoT Central](./tutorial-video-analytics-create-app.md).
* Uno de los anteriores tutoriales, [Creación de una instancia de IoT Edge para análisis de vídeo en directo (máquina virtual Linux)](tutorial-video-analytics-iot-edge-vm.md) o [Creación de una instancia de IoT Edge para análisis de vídeo en directo (Intel NUC)](tutorial-video-analytics-iot-edge-nuc.md).

Debe tener [Docker](https://www.docker.com/products/docker-desktop) instalado en el equipo local para ejecutar la aplicación del visor de vídeo.

## <a name="add-an-object-detection-camera"></a>Adición de una cámara de detección de objetos

En la aplicación de IoT Central, vaya al dispositivo **LVA Gateway 001** que creó anteriormente. A continuación, seleccione la pestaña **Commands** (Comandos).

Utilice los valores de la tabla siguiente como valores de parámetros para el comando **Add Camera Request** (Agregar solicitud de cámara). En los valores que se muestran en la tabla se supone que usa la cámara simulada en la máquina virtual de Azure, ajuste los valores de forma adecuada si usa una cámara real:

| Campo| Descripción| Valor de ejemplo|
|---------|---------|---------|
| Camera ID (Identificador de la cámara)      | Identificador de dispositivo para el aprovisionamiento | camera-003 |
| Camera Name (Nombre de la cámara)    | Nombre descriptivo           | Cámara de detección de objetos |
| RTSP Url (Dirección URL de RTSP)       | Dirección de la secuencia   | RTSP://10.0.0.4:554/media/camera-300s.mkv|
| RTSP Username (Nombre de usuario de RTSP)  |                         | usuario    |
| RTSP Password (Contraseña de RTSP)  |                         | password    |
| Tipo de detección | Menú desplegable                | Detección de objetos       |

Seleccione **Run** (Ejecutar) para agregar el dispositivo de cámara:

:::image type="content" source="media/tutorial-video-analytics-manage/add-camera.png" alt-text="Agregar cámara":::

> [!NOTE]
> La plantilla de dispositivo **LVA Edge Object Detector** (Detector de objetos LVA Edge) ya existe en la aplicación.

## <a name="add-a-motion-detection-camera-optional"></a>Adición de una cámara de detección de movimiento (opcional)

Si tiene dos cámaras conectadas al dispositivo de puerta de enlace IoT Edge, repita los pasos anteriores para agregar una cámara de detección de movimiento a la aplicación. Utilice valores diferentes para los parámetros **Identificador de cámara**, **Nombre de cámara** y **Dirección URL de RTSP**.

## <a name="view-the-downstream-devices"></a>Visualización de los dispositivos de bajada

Seleccione la pestaña **Downstream Devices** (Dispositivos de bajada) del dispositivo **LVA Gateway 001** para ver los dispositivos de cámara que acaba de agregar:

:::image type="content" source="media/tutorial-video-analytics-manage/inspect-downstream.png" alt-text="Inspeccionar":::

Los dispositivos de cámara también aparecen en la lista de la página **Devices** (Dispositivos) de la aplicación.

## <a name="configure-and-manage-the-camera"></a>Configuración y administración de la cámara

Vaya al dispositivo **camera-003** y seleccione la pestaña **Manage** (Administrar).

Use los valores predeterminados o modifíquelos si necesita personalizar las propiedades del dispositivo:

**Configuración de la cámara**

| Propiedad | Descripción | Valor sugerido |
|-|-|-|
| Video Playback Host (Host de reproducción de vídeo) | Host del visor de Azure Media Player | http://localhost:8094 |

**Operaciones y diagnósticos de LVA**

| Propiedad | Descripción | Valor sugerido |
|-|-|-|
| Inicio automático | Iniciar la detección de objetos cuando se reinicia la puerta de enlace de LVA | Activada |
| Debug Telemetry (Depurar telemetría) | Event Traces (Seguimientos de eventos) | Opcional |
|Inference Timeout (sec) (Tiempo de espera de inferencia [segundos])| Cantidad de tiempo que se usa para determinar que las inferencias se han detenido | 20 |

**Detección de objetos de IA**

| Propiedad | Descripción | Valor sugerido |
|-|-|-|
| Max Inference Capture Time (sec) (Tiempo máximo de captura de inferencia [segundos]) | Tiempo máximo de captura | 15 |
| Detection Classes (Clases de detección) | Cadenas, delimitadas por comas, con las etiquetas de detección. Para más información, consulte[Lista de etiquetas admitidas](https://github.com/Azure/live-video-analytics/blob/master/utilities/video-analysis/yolov3-onnx/tags.txt). | truck, bus (camión, autobús) |
| Confidence Threshold (Umbral de confianza) | Porcentaje de calificación para determinar si la detección de objetos es válida | 70 |
| Inference Frame Sample Rate (fps) (Velocidad de muestreo de marcos de la inferencia [marcos por segundo]) | [Agregar descripción aquí] | 2 |

Seleccione **Guardar**.

Después de unos segundos, verá el mensaje de confirmación **Accepted** (Aceptado) para cada configuración:

:::image type="content" source="media/tutorial-video-analytics-manage/object-detection.png" alt-text="Detección de objetos":::

## <a name="start-lva-processing"></a>Inicio del procesamiento de LVA

Vaya al dispositivo **camera-003** y seleccione la pestaña **Commands** (Comandos).

Ejecute el comando **Start LVA Processing** (Iniciar procesamiento de LVA).

Cuando se complete el comando, consulte el historial de comandos para asegurarse de que no hay errores:

:::image type="content" source="media/tutorial-video-analytics-manage/start-processing.png" alt-text="Comando Start LVA processing (Iniciar procesamiento de LVA)":::

## <a name="monitor-the-cameras"></a>Supervisión de las cámaras

Vaya al dispositivo **camera-003** y seleccione la pestaña **Dashboard** (Panel):

:::image type="content" source="media/tutorial-video-analytics-manage/camera-dashboard.png" alt-text="Panel de la cámara":::

El icono **Detection Count** (Recuento de detecciones) muestra el recuento medio de detecciones de cada uno de los objetos de clases de detección seleccionados durante un intervalo de detección de un segundo.

El gráfico circular **Detection Classes** (Clases de detección) muestra el porcentaje de detecciones para cada tipo de clase.

**Inference Event Video** (Vídeo de evento de inferencia) es una lista de vínculos a los recursos de Azure Media Services que contienen las detecciones. El vínculo usa el reproductor de host que se describe en la sección siguiente.

## <a name="start-the-streaming-endpoint"></a>Inicio del punto de conexión de streaming

Inicie el punto de conexión de Media Services para permitir que la aplicación cliente de Media Player transmita el vídeo grabado:

* En Azure Portal, vaya al grupo de recursos **lva-rg**.
* Abra el recurso **Punto de conexión de streaming**.
* En la página **Detalles del punto de conexión de streaming**, seleccione **Iniciar**. Verá una advertencia que indica que la facturación comienza cuando se inicia el punto de conexión.

## <a name="view-stored-video"></a>Visualización de vídeo almacenado

Los días de observar las cámaras y reaccionar ante las imágenes sospechosas han terminado. Con el etiquetado automático de eventos y los vínculos directos al vídeo almacenado con la detección inferida, los operadores de seguridad pueden encontrar los eventos de interés en una lista y, a continuación, seguir el vínculo para ver el vídeo.

Puede usar el [reproductor de vídeo AMP](https://github.com/Azure/live-video-analytics/tree/master/utilities/amp-viewer) para ver el vídeo almacenado en la cuenta de Azure Media Services.

La aplicación de IoT Central almacena el vídeo en Azure Media Services, desde donde puede transmitirlo. Necesita un reproductor de vídeo para reproducir el vídeo almacenado en Azure Media Services.

Asegúrese de que **Docker** está en ejecución en el equipo local.

Abra un símbolo del sistema y use el siguiente comando para ejecutar el reproductor de vídeo en un contenedor de Docker en el equipo local. Reemplace los marcadores de posición del comando por los valores que anotó previamente en el archivo *scratchpad.txt*. Anotó los valores de `amsAadClientId`, `amsAadSecret`, `amsAadTenantId`, `amsSubscriptionId` y `amsAccountName` cuando creó la entidad de servicio para la cuenta de Media Services:

```bash
docker run -it --rm -e amsAadClientId="<FROM_AZURE_PORTAL>" -e amsAadSecret="<FROM_AZURE_PORTAL>" -e amsAadTenantId="<FROM_AZURE_PORTAL>" -e amsArmAadAudience="https://management.core.windows.net" -e amsArmEndpoint="https://management.azure.com" -e amsAadEndpoint="https://login.microsoftonline.com" -e amsSubscriptionId="<FROM_AZURE_PORTAL>" -e amsResourceGroup="<FROM_AZURE_PORTAL>" -e amsAccountName="<FROM_AZURE_PORTAL>" -p 8094:8094 mcr.microsoft.com/lva-utilities/amp-viewer:1.0-amd64
```

|Parámetro de Docker | Acceso a la API de AMS (JSON)|
|-|-|
|amsAadClientId| AadClientId|
|amsAadSecret| AadSecret |
|amsAadTenantId| AadTenantId |
|amsSubscriptionId| SubscriptionId|
|amsResourceGroup| ResourceGroup |
|amsAccountName| AccountName|

Vaya al dispositivo **camera-003** y seleccione la pestaña **Dashboard** (Panel). A continuación, haga clic en uno de los hipervínculos de detección de objetos capturados en el icono **Inference Event Video** (Vídeo de evento de inferencia). El reproductor de vídeo local muestra el vídeo en una página:

:::image type="content" source="media/tutorial-video-analytics-manage/video-snippet.png" alt-text="Fragmento de vídeo":::

## <a name="change-the-simulated-devices-in-application-dashboards"></a>Cambio de los dispositivos simulados en los paneles de la aplicación

Los paneles de la aplicación se rellenan originalmente con los datos de telemetría y las propiedades generadas a partir de los dispositivos simulados de IoT Central. Para configurar los iconos para la telemetría desde cámaras reales o el simulador Live555, siga estos pasos:

1. Vaya al panel de aplicación **(Sample) Real Camera Monitor** ([Ejemplo] Monitor de cámara real).
1. Seleccione **Editar**.
1. Seleccione el icono **Note** (Nota) y elimínelo.
1. Cambie el título del panel por *Real Camera Monitor* (Monitor de cámara real).
1. En el icono **Inference Count** (Recuento de inferencias), seleccione el icono de configuración.
1. En la sección **Configure Chart** (Configurar gráfico), seleccione una o varias cámaras reales en el grupo de dispositivos **LVA Edge Object Detector** (Detector de objetos LVA Edge).
1. Seleccione el campo de telemetría `AI Inference Interface/Inference Count`.
1. Seleccione **Actualizar**.

1. Repita los pasos para los iconos siguientes:
    1. El gráfico circular **Detection** (Detección) usa el tipo de telemetría `AI Inference Interface/Inference/entity/tag/value`.
    1. **Inference** (Inferencia) usa el último valor conocido de `AI Inference Interface/Inference/entity/tag/value`.
    1. **Confidence %** (% de confianza) usa el último valor conocido de `AI Inference Interface/Inference/entity/tag/confidence`.
    1. **Snapshot** (Instantánea) usa `AI Inference Interface/Inference Image`, que se muestra como una imagen.
    1. **Inference Event Video** (Vídeo de evento de inferencia) usa `AI Inference Interface/Inference Event Video`, que se muestra como un vínculo.
1. Seleccione **Guardar**.

En el panel **Real Camera Monitor** (Monitor de cámara real) ahora se muestran los valores del dispositivo de cámara real:

:::image type="content" source="media/tutorial-video-analytics-manage/update-real-cameras.png" alt-text="Panel de la aplicación con cámaras reales":::

## <a name="pause-processing"></a>Pausa del procesamiento

Puede pausar el procesamiento de análisis de vídeo en directo en la aplicación:

1. En la aplicación, vaya a la página **Commands** (Comandos) de la cámara de detección de objetos. A continuación, ejecute el comando **Stop LVA Processing** (Detener procesamiento de LVA).

1. Detenga el punto de conexión de streaming de la cuenta de Media Services:
    * En Azure Portal, vaya al grupo de recursos **lva-rg**.
    * Haga clic en el recurso **Punto de conexión de streaming**.
    * En la página **Detalles del punto de conexión de streaming**, seleccione **Detener**.

## <a name="tidy-up"></a>Limpieza

Si ha terminado con la aplicación, puede eliminar todos los recursos que ha creado de la siguiente manera:

1. En la aplicación de IoT Central, vaya a la página **Your application** (Su aplicación) en la sección **Administration** (Administración). A continuación, seleccione **Eliminar**.
1. En Azure Portal, elimine el grupo de recursos **lva-rg**.
1. En el equipo local, detenga el contenedor de Docker **amp-viewer**.

## <a name="next-steps"></a>Pasos siguientes

Ahora ha aprendido a agregar cámaras a la aplicación de IoT Central y a configurarlas para la detección de objetos y movimiento.

Para obtener información sobre cómo personalizar el código fuente de los módulos IoT Edge:

> [!div class="nextstepaction"]
> [Modificación y compilación de los módulos de la puerta de enlace de análisis de vídeo en directo](./tutorial-video-analytics-build-module.md)
