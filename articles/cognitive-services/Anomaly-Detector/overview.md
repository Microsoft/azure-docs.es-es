---
title: ¿Qué es Anomaly Detector API?
titleSuffix: Azure Cognitive Services
description: Use algoritmos avanzados de Anomaly Detector API para identificar anomalías en datos de serie temporal.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: overview
ms.date: 09/10/2020
ms.author: aahi
ms.openlocfilehash: ba094709d78d92b678e3aa5ad9995923888caf00
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90887507"
---
# <a name="what-is-the-anomaly-detector-api"></a>¿Qué es Anomaly Detector API?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Anomaly Detector API permite supervisar y detectar anomalías en datos de serie temporal con aprendizaje automático. Esta API se adapta al identificar y aplicar automáticamente los modelos más idóneos para sus datos, con independencia del sector, el escenario o el volumen de datos. Con el uso de datos de serie temporal, la API determina los límites de la detección de anomalías, los valores esperados y qué puntos de datos son anomalías.

![Detectar cambios de patrón en las solicitudes de servicio](./media/anomaly_detection2.png)

Para usar Anomaly Detector, no se necesita ninguna experiencia anterior con el aprendizaje automático, y la API RESTful le permite integrar fácilmente el servicio en sus aplicaciones y procesos.

## <a name="features"></a>Características

Con Anomaly Detector, puede detectar automáticamente las anomalías existentes en los datos de serie temporal, o cuando se producen en tiempo real.

|Característica  |Descripción  |
|---------|---------|
|Detección de anomalías cuando se producen en tiempo real | Detecte anomalías en sus datos de transmisión mediante los puntos de datos vistos anteriormente para determinar si el último de ellos es una anomalía. Esta operación genera un modelo con los puntos de datos enviados y determina si el punto de destino es una anomalía. La llamada a la API con cada nuevo punto de datos que genera le permite supervisar los datos que se crean. |
|Detección de anomalías en todo el conjunto de datos como un lote | Use la serie temporal para detectar las anomalías que podrían existir en los datos. Esta operación genera un modelo con todos los datos de serie temporal, donde cada punto de analiza con el mismo modelo.         |
|Detecte los puntos de cambio en el conjunto de datos como un lote. | Use la serie temporal para detectar los puntos de cambio de tendencia que existan en los datos. Esta operación genera un modelo con todos los datos de serie temporal, donde cada punto de analiza con el mismo modelo.    |
| Obtención de información adicional sobre los datos | Obtenga detalles útiles sobre los datos y las anomalías observadas, como los valores esperados, los límites de anomalías y las posiciones. |
| Ajuste de los límites de detección de anomalías | Anomaly Detector API crea automáticamente límites para la detección de anomalías. Configure estos límites para aumentar o disminuir la sensibilidad de la API a las anomalías de datos y ajustar mejor los datos. |

## <a name="demo"></a>Demostración

Consulte esta [demostración interactiva](https://aka.ms/adDemo) para comprender cómo funciona Anomaly Detector.
Para ejecutar la demostración, debe crear un recurso de Anomaly Detector y obtener la clave de API y el punto de conexión.

## <a name="notebook"></a>Notebook

Para saber cómo llamar a Anomaly Detector API, pruebe este [cuaderno](https://aka.ms/adNotebook). En esta instancia de Jupyter Notebook se muestra cómo enviar una solicitud de API y visualizar el resultado.

Para ejecutar el cuaderno, haga lo siguiente:

1. Obtenga una clave de suscripción válida de Anomaly Detector API y un punto de conexión de API. En la siguiente sección se proporcionan instrucciones para registrarse.
1. Inicie sesión y haga clic en la opción de clonación en la esquina superior derecha.
1. Desactive la opción "público" en el cuadro de diálogo antes de completar la operación de clonación, ya que si no, el cuaderno, incluidas las claves de suscripción, será público.
1. Haga clic en **Run on Free Compute** (Ejecutar en un proceso gratuito).
1. Seleccione uno de los cuadernos.
1. Agregue su clave de suscripción válida de Anomaly Detector API a la variable `subscription_key`.
1. Cambie la variable `endpoint` por el punto de conexión. Por ejemplo: `https://westus2.api.cognitive.microsoft.com/anomalydetector/v1.0/timeseries/last/detect`
1. En la barra de menús superior, haga clic en **Cell** (celda) y, luego, en **Run All** (Ejecutar todo).

## <a name="workflow"></a>Flujo de trabajo

Anomaly Detector API es un servicio web RESTful, lo que significa que es fácil llamarlo desde cualquier lenguaje de programación que pueda realizar solicitudes HTTP y analizar código JSON.

[!INCLUDE [cognitive-services-anomaly-detector-data-requirements](../../../includes/cognitive-services-anomaly-detector-data-requirements.md)]

[!INCLUDE [cognitive-services-anomaly-detector-signup-requirements](../../../includes/cognitive-services-anomaly-detector-signup-requirements.md)]

Después del registro:

1. Tome sus datos de serie temporal y conviértalos en un formato JSON válido. Use los [procedimientos recomendados](concepts/anomaly-detection-best-practices.md) al preparar los datos para obtener los mejores resultados.
1. Envíe una solicitud a Anomaly Detector API con sus datos.
1. Analice el mensaje JSON devuelto para procesar la respuesta de API.

## <a name="algorithms"></a>Algoritmos

* Consulte los siguientes blogs técnicos para obtener información acerca de los algoritmos usados:
    * [Introducing Azure Anomaly Detector API](https://techcommunity.microsoft.com/t5/AI-Customer-Engineering-Team/Introducing-Azure-Anomaly-Detector-API/ba-p/490162) (Introducción a Anomaly Detector API de Azure)
    * [Introducción al algoritmo SR-CNN de Azure Anomaly Detector](https://techcommunity.microsoft.com/t5/AI-Customer-Engineering-Team/Overview-of-SR-CNN-algorithm-in-Azure-Anomaly-Detector/ba-p/982798)

Puede leer el artículo [Time-Series Anomaly Detection Service at Microsoft](https://arxiv.org/abs/1906.03821) (Servicio de detección de anomalías de Time Series) (aceptado por KDD 2019) para conocer los algoritmos SR-CNN desarrollados por Microsoft.


> [!VIDEO https://www.youtube.com/embed/ERTaAnwCarM]

## <a name="join-the-anomaly-detector-community"></a>Únase a la comunidad de Anomaly Detector

* Únase al [grupo de asesores de Anomaly Detector en Microsoft Teams](https://aka.ms/AdAdvisorsJoin)
* Vea el [contenido generado por usuarios](user-generated-content.md) seleccionado

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Detección de anomalías en datos de serie temporal mediante la API REST Anomaly Detector](quickstarts/detect-data-anomalies-csharp.md)
* [Demostración en línea](https://notebooks.azure.com/AzureAnomalyDetection/projects/anomalydetector) de Anomaly Detector API
* [Referencia de la API REST](https://aka.ms/anomaly-detector-rest-api-ref) de Anomaly Detector
