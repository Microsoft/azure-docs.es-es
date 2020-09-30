---
title: Análisis de los datos del dispositivo en la aplicación de Azure IoT Central | Microsoft Docs
description: Analice los datos del dispositivo en la aplicación de Azure IoT Central.
author: ankitscribbles
ms.author: ankitgup
ms.date: 11/27/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: abjork
ms.openlocfilehash: 8f329b6377915a3df6e3f3e212cbd8d41f9ed80d
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90017667"
---
# <a name="how-to-use-analytics-to-analyze-device-data"></a>Cómo usar análisis para analizar los datos del dispositivo

*Este artículo se aplica a los administradores, operadores y compiladores.*

Azure IoT Central ofrece funcionalidad de análisis enriquecida para analizar tendencias históricas y correlacionar diferentes telemetrías de los dispositivos. Para comenzar, visite **Analytics** en el panel izquierdo.

## <a name="understanding-the-analytics-ui"></a>Descripción de la interfaz de usuario de Analytics
La interfaz de usuario de Analytics consta de tres componentes principales:
- **Panel de configuración de datos:** En el panel de configuración, comience seleccionando el grupo de dispositivos cuyos datos quiere analizar. A continuación, seleccione la telemetría que quiere analizar y el método de agregación para cada telemetría. El control **Dividir por** ayuda a agrupar los datos al usar las propiedades del dispositivo como dimensiones.

- **Control de tiempo:** El control de tiempo se usa para seleccionar la duración para la que desea analizar los datos. Puede arrastrar cualquier extremo del control deslizante de tiempo para seleccionar el intervalo de tiempo. El control de tiempo también tiene un control deslizante de **Tamaño de intervalo** que controla el cubo o el tamaño del intervalo usado para agregar los datos. 

- **Control de gráficos:** El control de gráficos visualiza los datos como un gráfico de líneas. Puede alternar la visibilidad de líneas específicas interactuando con la leyenda del gráfico. 


  ![Información general sobre la interfaz de usuario de Analytics](media/howto-create-analytics/analyticsui.png)


## <a name="querying-your-data"></a>Consulta de los datos

Deberá empezar por elegir un **grupo de dispositivos** y la telemetría que quiere analizar. Cuando haya terminado, seleccione **Analizar** para comenzar a visualizar los datos.

- **Grupo de dispositivos:** Un [grupo de dispositivos](tutorial-use-device-groups.md) es un conjunto de dispositivos definido por el usuario. Por ejemplo, todos los refrigeradores en Oakland o todas las turbinas eólicas versión 2.0.

- **Telemetría**: Seleccione la telemetría que quiere analizar y explorar. Puede seleccionar varias telemetrías para analizarlas juntas. El método de agregación predeterminado se establece en Average para el tipo de datos numérico y como Count para el tipo de datos de cadena, respectivamente. Los métodos de agregación admitidos para los tipos de datos numéricos son Average, Maximum, Minimum, Count y Sum.  Los métodos de agregación admitidos para el tipo de datos de cadena son Count.

- **Dividir por**: El control "Dividir por" ayuda a agrupar los datos al usar las propiedades del dispositivo como dimensiones. Los valores de las propiedades de la nube y el dispositivo se unen con la telemetría a medida que se envía por el dispositivo. Si la propiedad de la nube o del dispositivo se ha actualizado, verá la telemetría agrupada por diferentes valores en el gráfico.

    > [!TIP]
    > Para ver los datos de cada dispositivo por separado, seleccione Id. de dispositivo en el control "Dividir por".

## <a name="interacting-with-your-data"></a>Interacción con los datos

Cuando haya consultado los datos, podrá comenzar a visualizarlos en el gráfico de líneas. Puede mostrar u ocultar la telemetría, cambiar la duración de la misma y ver la telemetría en una cuadrícula de datos.

- **Panel del editor de tiempo:** De forma predeterminada, se podrán recuperar datos del último día. Puede arrastrar cualquier extremo del control deslizante de tiempo para cambiar el intervalo de tiempo. También puede utilizar el control de calendario para seleccionar uno de los cubos de hora predefinidos o seleccionar un intervalo de tiempo personalizado. El control de tiempo también tiene un control deslizante de **Tamaño de intervalo** que controla el cubo o el tamaño del intervalo usado para agregar los datos.

    ![Editor de tiempo](media/howto-create-analytics/timeeditorpanel.png)

    - **Herramienta de control deslizante de intervalos de fechas internas**: use los dos controles de punto de conexión arrastrándolos sobre el intervalo de tiempo que quiera usar. El control deslizante del intervalo de fechas externas se encarga de restringir el intervalo de fechas internas.
    
   
    - **Control deslizante de intervalos de fechas externas**: use los controles del punto de conexión para seleccionar el intervalo de fechas externas que estará disponible para el control de intervalos de fechas internas.

    - **Aumentar y disminuir los botones de intervalo de fechas**: puede aumentar o disminuir el intervalo de tiempo si selecciona cualquiera de los botones del intervalo que quiera.

    - **Control deslizante para el tamaño de intervalos**: use esta opción para acercar y alejar los intervalos durante el mismo intervalo de tiempo. Esta acción le proporciona un control más preciso del movimiento entre grandes segmentos de tiempo. Puede usar esta opción para tener vistas pormenorizadas de alta resolución de los datos, incluso en pocos milisegundos. El punto inicial predeterminado del control deslizante está establecido como la vista más óptima de los datos de la selección, lo que permite equilibrar la resolución, la velocidad de la consulta y la granularidad.
    
    - **Selector de intervalo de fechas**: con este control web, puede seleccionar fácilmente los intervalos de fecha y hora que quiera. También puede usar el control para cambiar entre zonas horarias diferentes. Después de realizar los cambios para aplicarlos al área de trabajo actual, seleccione Guardar.

    > [!TIP]
    > El tamaño del intervalo se determina dinámicamente según el intervalo de tiempo seleccionado. Los intervalos de tiempo más pequeños permitirán agregar los datos en intervalos muy pormenorizados de hasta unos segundos.


- **Leyenda del gráfico:** La leyenda de gráfico muestra la telemetría seleccionada en el gráfico. Puede mantener el mouse sobre cada elemento de la leyenda para ponerlo en el foco en el gráfico. Cuando se usa "Dividir por", la telemetría se agrupa por los respectivos valores de la dimensión seleccionada. Para alternar la visibilidad de cada telemetría específica o de todo el grupo, haga clic en el nombre del grupo.  


- **Control de formato del eje Y:** el modo del eje Y recorre las opciones disponibles de la vista del eje Y. Este control solo está disponible cuando se visualizan diferentes telemetrías. Puede establecer el eje Y eligiendo uno de los tres modos siguientes:

    - **Apilado**: Se apila un gráfico para cada telemetría y cada uno de los gráficos tiene su propio eje Y. Este modo está establecido como predeterminado.
    - **Compartida:** Se traza un gráfico para cada telemetría en el mismo eje Y.
    - **Superposición:** Úselo para apilar varias líneas en el mismo eje Y para que los datos del eje Y cambien en función de la línea seleccionada.

  ![Organización de datos en el eje Y con distintos modos de visualización](media/howto-create-analytics/yaxiscontrol.png)

- **Control de zoom:** El zoom le permite profundizar más en los datos. Si encuentra un período de tiempo en cuyo conjunto de resultados quiere centrarse, use el puntero del mouse para captar el área y arrástrela hasta el punto de conexión de su elección. A continuación, haga clic con el botón derecho en el área seleccionada y luego en Zoom.

  ![Ampliación de los datos](media/howto-create-analytics/zoom.png)

En los puntos suspensivos, hay más controles de gráfico para interactuar con los datos:

- **Mostrar cuadrícula:** los resultados están disponibles en formato de tabla para permitirle ver el valor específico para cada punto de datos.

- **Descargar como CSV:** los resultados están disponibles para exportar como un archivo de valores separados por comas (CSV). El archivo .csv contiene datos para cada dispositivo. Los resultados se exportan mediante el intervalo y el período de tiempo especificados. 

- **Colocar un marcador:** El control "Colocar un marcador" ofrece una manera de delimitar determinados punto de datos en el gráfico. Resulta útil cuando se intentan comparar datos de varias líneas en distintos períodos de tiempo.

  ![Presentación de la vista de cuadrícula para los análisis](media/howto-create-analytics/additionalchartcontrols.png)
