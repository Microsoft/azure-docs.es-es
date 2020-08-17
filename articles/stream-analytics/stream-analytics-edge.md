---
title: Azure Stream Analytics en IoT Edge
description: Cree trabajos perimetrales en Azure Stream Analytics e impleméntelos en dispositivos que ejecutan Azure IoT Edge.
ms.service: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.topic: how-to
ms.date: 03/16/2020
ms.custom: seodec18
ms.openlocfilehash: 136d0627e701104e9958d51b2e37256de5659f25
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87271423"
---
# <a name="azure-stream-analytics-on-iot-edge"></a>Azure Stream Analytics en IoT Edge
 
Azure Stream Analytics (ASA) en IoT Edge permite a los desarrolladores desarrollar inteligencia analítica casi en tiempo real más próxima a los dispositivos IoT para que puedan desbloquear el valor total de los datos generados por los dispositivos. Azure Stream Analytics está diseñado con los objetivos de baja latencia, resiliencia, uso eficiente de ancho de banda y cumplimiento. Las empresas pueden implementar ahora la lógica de control cerca de las operaciones industriales y complementar los análisis de macrodatos que se realizan en la nube.  

Azure Stream Analytics en IoT Edge se ejecuta en el marco de [Azure IoT Edge](https://azure.microsoft.com/campaigns/iot-edge/). Una vez que se crea el trabajo en ASA, puede implementarlo y administrarlo mediante IoT Hub.

## <a name="scenarios"></a>Escenarios
![Diagrama general de IoT Edge](media/stream-analytics-edge/ASAedge-highlevel-diagram.png)

* **Comando de baja latencia y control**: Por ejemplo, los sistemas de seguridad en la fabricación deben responder a los datos operativos con una latencia muy baja. Con ASA en IoT Edge, puede analizar los datos de sensores casi en tiempo real y emitir comandos cuando detecte anomalías para detener una máquina o desencadenar alertas.
*   **Conectividad limitada a la nube**: Los sistemas críticos, como un equipo de minería remoto, buques conectados o las perforaciones petrolíferas en mar abierto necesitan analizar y reaccionar ante los datos, incluso cuando la conectividad a la nube sea intermitente. Con el ASA, la lógica de streaming funciona independientemente de la conectividad de red y puede elegir lo que envía a la nube para su posterior procesamiento o almacenamiento.
* **Ancho de banda limitado**: El volumen de datos que generan los motores de jet o los automóviles conectados puede ser tan grande que los datos se deben filtrar o procesar con anterioridad antes de enviarlos a la nube. ASA puede filtrar o agregar los datos que se envían a la nube.
* **Cumplimiento**: El cumplimiento de las normas puede requerir que se agreguen algunos datos o que se oculte su identidad localmente antes de enviarlos a la nube.

## <a name="edge-jobs-in-azure-stream-analytics"></a>Trabajos de Edge en Azure Stream Analytics
### <a name="what-is-an-edge-job"></a>¿Qué es un trabajo "perimetral"?

Los trabajos de ASA en Edge se ejecutan en contenedores implementados en [dispositivos de Azure IoT Edge](https://docs.microsoft.com/azure/iot-edge/how-iot-edge-works). Constan de dos partes:
1.  Una parte en la nube que es responsable de la definición del trabajo: los usuarios definen las entradas, la salida, las consultas y otros valores (eventos que no funcionan, etc.) en la nube.
2.  Un módulo que se ejecuta en dispositivos IoT. Contiene el motor de ASA y recibe la definición del trabajo de la nube. 

ASA usa IoT Hub para implementar los trabajos perimetrales en los dispositivos. Para más información acerca de [la implementación de IoT Edge, consulte aquí](https://docs.microsoft.com/azure/iot-edge/module-deployment-monitoring).

![Trabajo en Edge de Azure Stream Analytics](media/stream-analytics-edge/stream-analytics-edge-job.png)


### <a name="installation-instructions"></a>Instrucciones de instalación
Los valores posibles se describen en la tabla siguiente. En las secciones siguientes encontrará más información.

| Paso | Notas |
| --- | --- |
| **Creación de un contenedor de almacenamiento** | Los contenedores de almacenamiento se utilizan para guardar la definición de trabajo donde pueden obtenerla sus dispositivos de IoT. <br>  Todos los contenedor de almacenamiento existente se pueden reutilizar. |
| **Creación de un trabajo perimetral de ASA** | Cree un nuevo trabajo y seleccione **Edge** como **entorno de hospedaje**. <br> Estos trabajos se crean o administran desde la nube y se ejecutan en sus propios dispositivos de IoT Edge. |
| **Configuración de un entorno de IoT Edge en los dispositivos** | Instrucciones para [Windows](https://docs.microsoft.com/azure/iot-edge/quickstart) o [Linux](https://docs.microsoft.com/azure/iot-edge/quickstart-linux).|
| **Implementación de ASA en dispositivos de IoT Edge** | La definición del trabajo ASA se exporta al contenedor de almacenamiento que creó anteriormente. |

Puede seguir [este tutorial paso a paso](https://docs.microsoft.com/azure/iot-edge/tutorial-deploy-stream-analytics) para implementar su primer trabajo de ASA en IoT Edge. El vídeo siguiente debería ayudarle a entender el proceso para ejecutar un trabajo de Stream Analytics en un dispositivo con IoT Edge:  


> [!VIDEO https://channel9.msdn.com/Events/Connect/2017/T157/player]

#### <a name="create-a-storage-container"></a>Creación de un contenedor de almacenamiento
Se requiere un contenedor de almacenamiento es necesario para exportar la consulta ASA compilada y la configuración del trabajo. Se utiliza para configurar la imagen de Docker ASA con una consulta específica. 
1. Siga [estas instrucciones](https://docs.microsoft.com/azure/storage/common/storage-create-storage-account) para crear una cuenta de almacenamiento desde Azure Portal. Puede mantener todas las opciones predeterminadas para usar esta cuenta con ASA.
2. En la cuenta recién creada, cree un contenedor de almacenamiento de blobs:
    1. Haga clic en **Blobs** y, después, en **+ Contenedor**. 
    2. Escriba un nombre y mantenga el contenedor como **Privado**.

#### <a name="create-an-asa-edge-job"></a>Creación de un trabajo de Edge de ASA
> [!Note]
> Este tutorial se centra en la creación de un trabajo ASA mediante Azure Portal. También puede [usar el complemento de Visual Studio para crear un trabajo de Edge de ASA](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-tools-for-visual-studio-edge-jobs)

1. En Azure Portal, cree un nuevo "trabajo de Stream Analytics". [Vínculo directo para crear un nuevo trabajo de ASA aquí](https://ms.portal.azure.com/#create/Microsoft.StreamAnalyticsJob).

2. En la pantalla de creación, seleccione **Edge** como **entorno de hospedaje** (vea la imagen siguiente)

   ![Creación del trabajo de Stream Analytics en Edge](media/stream-analytics-edge/create-asa-edge-job.png)
3. Definición de trabajo
    1. **Definir flujos de entrada**. Defina uno o varios flujos de entrada para el trabajo.
    2. Definición de datos de referencia (opcional).
    3. **Definir flujos de salida**. Defina uno o varios flujos de salida para el trabajo. 
    4. **Definir consulta**. Defina la consulta ASA en la nube mediante el editor insertado. El compilador comprueba automáticamente la sintaxis habilitada para el perímetro de ASA. La consulta también se puede probar mediante la carga de datos de ejemplo. 

4. Establezca la información del contenedor de almacenamiento en el menú **Configuración de IoT Edge**.

5. Establecimiento de valores opcionales
    1. **Ordenación de eventos**. Puede configurar la directiva de fuera de servicio en el portal. La documentación está disponible [aquí](https://docs.microsoft.com/stream-analytics-query/time-skew-policies-azure-stream-analytics).
    2. **Configuración regional**. Defina el formato de internalización.



> [!Note]
> Cuando se crea una implementación, ASA exporta la definición del trabajo a un contenedor de almacenamiento. Esta definición de trabajo siguen siendo la misma mientras dure una implementación. En consecuencia, si desea actualizar un trabajo que se ejecuta en el perímetro, es preciso que lo edite en ASA y que, a continuación, cree una nueva implementación en IoT Hub.


#### <a name="set-up-your-iot-edge-environment-on-your-devices"></a>Configuración de un entorno de IoT Edge en los dispositivos
Los trabajos de Edge se pueden implementar en dispositivos que ejecuten Azure IoT Edge.
Para hacerlo, es preciso seguir estos pasos:
- Cree una instancia de IoT Hub.
- Instale Docker y el entorno de ejecución de Azure IoT Edge en los dispositivos perimetrales.
- Configure los dispositivos como **dispositivos de IoT Edge** en IoT Hub.

Estos pasos se describen en la documentación de IoT Edge para [Windows](https://docs.microsoft.com/azure/iot-edge/quickstart) o [Linux](https://docs.microsoft.com/azure/iot-edge/quickstart-linux).  


####  <a name="deployment-asa-on-your-iot-edge-devices"></a>Implementación de ASA en dispositivos de IoT Edge
##### <a name="add-asa-to-your-deployment"></a>Adición de ASA a una implementación
- En Azure Portal, abra IoT Hub, vaya a **IoT Edge** y haga clic en el dispositivo que va a ser el destino de esta implementación.
- Seleccione **Establecer módulos**, **+ Agregar** y, finalmente, **Módulo de Azure Stream Analytics**.
- Seleccione la suscripción y el trabajo Edge de ASA que ha creado. Haga clic en Guardar.
![Adición de un módulo de ASA a una implementación](media/stream-analytics-edge/add-stream-analytics-module.png)


> [!Note]
> Durante este paso, ASA crea una carpeta llamada "EdgeJobs" en el contenedor de almacenamiento (si aún no existe). Para cada implementación, se crea una nueva subcarpeta en la carpeta "EdgeJobs".
> Al implementar el trabajo en dispositivos de IoT Edge, ASA crea una firma de acceso compartido (SAS) para el archivo de definición del trabajo. La clave SAS se transmite de forma segura a los dispositivos de IoT Edge que usen un dispositivo gemelo. Esta clave expira a los tres años de haberse creado. Cuando se actualiza un trabajo de IoT Edge, la firma SAS cambia, pero la versión de la imagen permanece como está. Después de **actualizar**, siga el flujo de trabajo de implementación; se registra una notificación de actualización en el dispositivo.


Para más información sobre las implementaciones de IoT Edge, consulte [esta página](https://docs.microsoft.com/azure/iot-edge/module-deployment-monitoring).


##### <a name="configure-routes"></a>Configuración de rutas
IoT Edge proporciona una manera de enrutar de forma declarativa mensajes tanto entre los módulos como entre los módulos e IoT Hub. La sintaxis completa se describe [aquí](https://docs.microsoft.com/azure/iot-edge/module-composition).
Los nombres de las entradas y salidas creadas en el trabajo de ASA se pueden utilizar como puntos de conexión para el enrutamiento.  

###### <a name="example"></a>Ejemplo

```json
{
    "routes": {
        "sensorToAsa":   "FROM /messages/modules/tempSensor/* INTO BrokeredEndpoint(\"/modules/ASA/inputs/temperature\")",
        "alertsToCloud": "FROM /messages/modules/ASA/* INTO $upstream",
        "alertsToReset": "FROM /messages/modules/ASA/* INTO BrokeredEndpoint(\"/modules/tempSensor/inputs/control\")"
    }
}

```
En este ejemplo se muestran las rutas del escenario descrito en la siguiente ilustración. Contiene un trabajo perimetral denominado "**ASA**", con una entrada denominada "**temperature**" y una salida denominada "**alert**".
![Ejemplo de diagrama de enrutamiento de mensajes](media/stream-analytics-edge/edge-message-routing-example.png)

En este ejemplo se definen las siguientes rutas:
- Todos los mensaje de **tempSensor** se envían al módulo denominado **ASA**, a la entrada denominada **temperature**.
- Todas las salidas del módulo de **ASA** se envían a la instancia de IoT Hub vinculada a este dispositivo ($upstream).
- Todas las salidas del módulo de **ASA** se envían al punto de conexión **control** de **tempSensor**.


## <a name="technical-information"></a>Información técnica
### <a name="current-limitations-for-iot-edge-jobs-compared-to-cloud-jobs"></a>Limitaciones actuales de los trabajos de IoT Edge en comparación con los trabajos en la nube
El objetivo es tener paridad entre los trabajos de IoT Edge y los trabajos en la nube. Se admite la mayoría de las características del lenguaje de consulta SQL, lo que permite ejecutar la misma lógica en la nube y en IoT Edge.
Sin embargo, las siguientes características aún no se admiten en los trabajos perimetrales:
* Funciones definidas por el usuario (UDF) en JavaScript. Las UDF están disponibles en [ C# para trabajos de IoT Edge](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-edge-csharp-udf) (versión preliminar).
* Agregados definidos por el usuario (UDA).
* Funciones de Azure Machine Learning.
* Uso de más de 14 agregados en un solo paso.
* Formato AVRO para la entrada y salida. En este momento solo se admiten CSV y JSON.
* Los siguientes operadores SQL:
    * PARTITION BY
    * GetMetadataPropertyValue
* Directiva de llegada tardía

### <a name="runtime-and-hardware-requirements"></a>Requisitos de runtime y hardware
Para ejecutar ASA en IoT Edge, se necesitan dispositivos que puedan ejecutar [Azure IoT Edge](https://azure.microsoft.com/campaigns/iot-edge/). 

ASA y Azure IoT Edge usan contenedores de **Docker** para proporcionar una solución portátil que se ejecuta en sistemas operativos que admiten múltiples hosts (Windows, Linux).

ASA en IoT Edge está disponible en forma de imágenes de Windows y Linux que se ejecutan en arquitecturas x86-64 o ARM (Advanced RISC Machines). 


### <a name="input-and-output"></a>Entrada y salida
#### <a name="input-and-output-streams"></a>Flujos de entrada y salida
Los trabajos de Edge de ASA pueden obtener entradas y salidas de otros módulos que se ejecutan en dispositivos de IoT Edge. Para conectarse a determinados módulos, y desde ellos, puede establecer la configuración de enrutamiento en el momento de la implementación. Se puede encontrar más información en [la documentación de composición del módulo de IoT Edge](https://docs.microsoft.com/azure/iot-edge/module-composition).

En las entradas y salidas, se admiten los formatos CSV y JSON.

Por cada flujo de entrada y salida que se crea en un trabajo de ASA, se crea un punto de conexión correspondiente en el módulo implementado. Estos puntos de conexión se pueden utilizar en las rutas de la implementación.

En este momento, los únicos tipos de entrada y salida de flujo admitidos son Centro de Microsoft Edge. La entrada de referencia admite el tipo de archivo de referencia. Se puede llegar a otras salidas mediante un trabajo de nube de nivel inferior. Por ejemplo, un trabajo de Stream Analytics hospedado en Edge envía una salida al Centro de Microsoft Edge, el cual puede enviar luego una salida a IoT Hub. Puede usar un segundo trabajo de Azure Stream Analytics hospedado en la nube con entrada desde IoT Hub y salida a Power BI u otro tipo de salida.



##### <a name="reference-data"></a>Datos de referencia
Los datos de referencia (que también se conocen como tabla de búsqueda) son un conjunto finito de datos estáticos o que, por naturaleza, cambian lentamente. Se utilizan para realizar búsquedas o para ponerlos en correlación con su flujo de datos. Para usar los datos de referencia en el trabajo de Azure Stream Analytics, por lo general usará una instrucción [JOIN de los datos de referencia](https://docs.microsoft.com/stream-analytics-query/reference-data-join-azure-stream-analytics) en la consulta. Para obtener más información, vea [Uso de datos de referencia para las búsquedas en Stream Analytics](stream-analytics-use-reference-data.md).

Solo se admiten datos de referencia locales. Cuando un trabajo se implementa en el dispositivo IoT Edge, carga los datos de referencia de la ruta de acceso del archivo definida por el usuario.

Para crear un trabajo con datos de referencia en Edge:

1. Cree una nueva entrada para el trabajo.

2. Elija **Datos de referencia** como **tipo de origen**.

3. Tenga un archivo de datos de referencia preparado en el dispositivo. Para un contenedor de Windows, coloque el archivo de datos de referencia en la unidad local y comparta la unidad local con el contenedor de Docker. Para un contenedor de Linux, cree un volumen de Docker y rellene el archivo de datos en el volumen.

4. Establezca la ruta de acceso del archivo. Para un sistema operativo host de Windows y un contenedor de Windows, use la ruta de acceso absoluta: `E:\<PathToFile>\v1.csv`. Para un sistema operativo host de Windows y un contenedor de Linux, o un sistema operativo Linux y un contenedor de Linux, use la ruta de acceso en el volumen: `<VolumeName>/file1.txt`.

![Nueva entrada de datos de referencia para el trabajo de Azure Stream Analytics en IoT Edge](./media/stream-analytics-edge/Reference-Data-New-Input.png)

Los datos de referencia en la actualización de IoT Edge se desencadenan mediante una implementación. Una vez activado, el módulo ASA toma los datos actualizados sin necesidad de detener el trabajo en ejecución.

Hay dos maneras de actualizar los datos de referencia:
* Actualizar la ruta de acceso de los datos de referencia en el trabajo ASA desde Azure Portal.
* Actualizar la implementación de IoT Edge.

## <a name="license-and-third-party-notices"></a>Licencia y avisos de terceros
* [Licencia de Azure Stream Analytics en IoT Edge](https://go.microsoft.com/fwlink/?linkid=862827). 
* [Aviso de terceros para Azure Stream Analytics en IoT Edge](https://go.microsoft.com/fwlink/?linkid=862828).

## <a name="azure-stream-analytics-module-image-information"></a>Información de la imagen del módulo de Azure Stream Analytics 

Esta información de versión se actualizó por última vez el 27 de junio de 2019:

- Imagen: `mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.5-linux-amd64`
   - imagen base: microsoft/dotnet:2.1.6-runtime-alpine3.7
   - plataforma:
      - arquitectura: amd64
      - so: linux
  
- Imagen: `mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.5-linux-arm32v7`
   - imagen base: microsoft/dotnet:2.1.6-runtime-bionic-arm32v7
   - plataforma:
      - arquitectura: arm
      - so: linux
  
- Imagen: `mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.5-windows-amd64`
   - imagen base: microsoft/dotnet:2.1.6-runtime-nanoserver-1809
   - plataforma:
      - arquitectura: amd64
      - so: windows
      
      
## <a name="get-help"></a>Obtener ayuda
Para más ayuda, pruebe la [Página de preguntas y respuestas de Microsoft sobre Azure Stream Analytics](https://docs.microsoft.com/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Pasos siguientes

* [Más información acerca de Azure IoT Edge](https://docs.microsoft.com/azure/iot-edge/how-iot-edge-works)
* [Tutorial de ASA en IoT Edge](https://docs.microsoft.com/azure/iot-edge/tutorial-deploy-stream-analytics)
* [Desarrollo de trabajos para dispositivos perimetrales de Stream Analytics mediante herramientas de Visual Studio](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-tools-for-visual-studio-edge-jobs)
* [Implement CI/CD for Stream Analytics using APIs](stream-analytics-cicd-api.md) (Implementación de CI/CD para Stream Analytics mediante API)

<!--Link references-->
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.query.language.reference]: https://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: https://go.microsoft.com/fwlink/?LinkId=517301
