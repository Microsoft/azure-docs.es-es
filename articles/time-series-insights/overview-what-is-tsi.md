---
title: 'Introducción: ¿Qué es Azure Time Series Insights Gen2? Azure Time Series Insights Gen2 | Microsoft Docs'
description: Más información sobre los cambios, mejoras y características de Azure Time Series Insights Gen2.
ms.service: time-series-insights
services: time-series-insights
author: shreyasharmamsft
ms.author: shresha
manager: dpalled
ms.workload: big-data
ms.topic: overview
ms.date: 07/07/2020
ms.custom: seodec18
ms.openlocfilehash: 7da33aeceff3d4d822de0d05e4643826219af8a0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87101296"
---
# <a name="what-is-azure-time-series-insights-gen2"></a>¿Qué es Azure Time Series Insights Gen2?

Azure Time Series Insights Gen2 es un servicio de análisis de IoT de un extremo a otro abierto y escalable que incluye las mejores experiencias de usuario y API enriquecidas para integrar sus eficaces capacidades con su flujo de trabajo o aplicación existentes. 

Se puede usar para recopilar, procesar, almacenar, consultar y visualizar los datos en la escala del Internet de las cosas (IoT), que están muy contextualizados y optimizados para las series temporales. 

Azure Time Series Insights Gen2 está diseñado para la exploración de datos ad hoc y el análisis operativo, lo que le permite descubrir tendencias ocultas, detectar anomalías y realizar análisis de causa raíz. Es una oferta abierta y flexible que satisface las necesidades de las implementaciones de IoT industriales. 

## <a name="video"></a>Vídeo

Más información acerca de Azure Time Series Insights Gen2.

> [!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Using-Azure-Time-Series-Insights-to-create-an-Industrial-IoT-analytics-platform/player]

## <a name="definition-of-iot-data"></a>Definición de datos de IoT

A menudo, los datos de IoT industriales en organizaciones con un uso intensivo de recursos carecen de coherencia estructural debido a la naturaleza variable de los dispositivos y sensores en una configuración industrial. Los datos procedentes de estos flujos se caracterizan por incluir vacíos significativos y, a veces, mensajes dañados y lecturas falsas. Los datos de IoT suelen ser significativos en el contexto de las entradas de datos adicionales que proceden de orígenes propios o de terceros, como CRM o ERP, que agregan contexto para flujos de trabajo de un extremo a otro. Las entradas de orígenes de datos de terceros, como datos meteorológicos, pueden ayudar a aumentar los flujos de telemetría en una instalación determinada. 

Todo esto implica que solo una fracción de los datos se usa para fines operativos y empresariales, y el análisis requiere contextualización. Los datos industriales a menudo se historían para un análisis en profundidad durante períodos de tiempo más largos para comprender y correlacionar tendencias. Para convertir los datos de IoT recopilados en información útil, se requiere: 

* Procesamiento de datos para limpiar, filtrar, interpolar, transformar y preparar los datos para su análisis.
* Una estructura para poder explorar y comprender los datos, a fin de normalizarlos y contextualizarlos.
* Almacenamiento rentable para una retención duradera o infinita de datos procesados (o derivados) y sin procesar.

Estos datos proporcionan una información coherente, completa, actualizada y correcta que resulta necesaria para los informes y los análisis empresariales.

La imagen siguiente muestra un flujo de datos de IoT típico.

[![Flujo de datos de IoT](media/v2-update-overview/overview-one.png)](media/v2-update-overview/overview-one.png#lightbox)

## <a name="azure-time-series-insights-gen2-for-industrial-iot"></a>Azure Time Series Insights Gen2 para IoT industrial

El panorama de IoT es diverso con clientes que abarcan una gran variedad de segmentos del sector, como fabricación, automoción, energía, utilidades, edificios inteligentes y consultoría. En esta amplia gama de mercados de IoT industrial, las soluciones nativas para la nube que proporcionan análisis completos orientados a datos de IoT a gran escala siguen evolucionando. 

Azure Time Series Insights Gen2 aborda esta necesidad de mercado al proporcionar una solución de análisis de IoT de un extremo a otro y llave en mano con modelos semánticos enriquecidos para contextualización de datos de series temporales, conclusiones basadas en recursos y la mejor experiencia de usuario en lo que a detección, tendencias, detección de anomalías e inteligencia operativa se refiere. 

Una plataforma de análisis operativo enriquecida combinada con nuestras funcionalidades de exploración de datos interactivas, puede usar Azure Time Series Insights Gen2 para obtener más valor de los datos recopilados de los recursos de IoT. La oferta de Gen2 admite: 

* Una solución de almacenamiento multicapa con compatibilidad con análisis intermedios y en frío que proporcionan a los clientes la opción de enrutar los datos entre intermedios y en frío para el análisis interactivo sobre datos intermedios, así como la inteligencia operativa en décadas de datos históricos. 

    *    Una solución de análisis intermedio muy interactiva para realizar numerosas y frecuentes consultas a través de datos con intervalos de tiempo más cortos. 
    *    Un lago de datos de serie temporal escalable, de gran rendimiento y optimizado para costo basado en Azure Storage que permite a los clientes analizar tendencias de años de datos de series temporales en cuestión de segundos. 

* Compatibilidad con el modelo semántico que describe el dominio y los metadatos asociados con las señales derivadas y sin procesar de los recursos y dispositivos.

* Plataforma de análisis flexible para almacenar datos históricos de series temporales en la cuenta de Azure Storage propiedad del cliente, lo que permite a los clientes tener la propiedad de sus datos de IoT. Los datos se almacenan con el formato de Apache Parquet de código abierto, que permite la conectividad y la interoperabilidad en distintos escenarios de datos, incluidos el análisis predictivo, el aprendizaje automático y otros cálculos personalizados realizados mediante tecnologías conocidas, como Spark y Databricks. 

* Análisis enriquecidos con mejores API de consulta y experiencia del usuario que combina conclusiones de datos basadas en activos con análisis de datos ad hoc enriquecidos con compatibilidad con interpolación, funciones escalares y de agregado, variables categóricas, gráficos de dispersión y señales de series temporales con pausas en directo para un análisis en profundidad.

*    Plataforma de nivel empresarial para admitir las necesidades de escalado, rendimiento, seguridad y confiabilidad de nuestros clientes de IoT empresariales.

* Extensibilidad y compatibilidad con la integración para el análisis de un extremo a otro. Azure Time Series Insights Gen2 proporciona una plataforma de análisis extensible para una amplia gama de escenarios de datos. El conector de Power BI permite a los clientes llevar las consultas que realizan en Azure Time Series Insights Gen2 directamente a Power BI para obtener una vista unificada de sus análisis de inteligencia empresarial y series temporales en una sola pantalla.

En el siguiente diagrama se muestra el flujo de datos de alto nivel.

  [![Principales funcionalidades](media/v2-update-overview/overview-two.png)](media/v2-update-overview/overview-two.png#lightbox)

Azure Time Series Insights Gen2 proporciona un modelo de precios de pago por uso escalable para el procesamiento de datos, el almacenamiento (datos y metadatos) y la consulta, lo que permite a los clientes ajustar su uso para satisfacer sus necesidades empresariales. 
 
Gracias a la introducción de estas funcionalidades clave de IoT industrial, Azure Time Series Insights Gen2 también ofrece las siguientes ventajas importantes:  

| Capacidad | Ventaja |
| ---| ---|
| Almacenamiento multicapa para datos de series temporales a escala de IoT | Con una canalización de procesamiento de datos compartida para la ingesta de datos, puede ingerir datos en almacenes intermedios y en frío. Use el almacén intermedio para las consultas interactivas y el almacén en frío para almacenar grandes volúmenes de datos. Para más información sobre cómo aprovechar las consultas basadas en recursos de alto rendimiento, vea [Consultas de datos](./concepts-query-overview.md). |
| Modelo de serie temporal para contextualizar datos de telemetría sin procesar y derivar información basada en recursos | Puede usar el modelo de serie temporal para crear instancias, jerarquías, tipos y variables para los datos de series temporales. Para más información sobre el modelo de Time Series consulte [Modelo de Time Series](./concepts-model-overview.md).  |
| Integración suave y continua con otras soluciones de datos | Los datos del almacén en reposo de Azure Time Series Insights Gen2 se [almacenan](./concepts-storage.md) en archivos de Apache Parquet de código abierto. Esto permite la integración de datos con otras soluciones de datos, de primera entidad o de terceros, en escenarios que incluyen inteligencia empresarial, aprendizaje automático avanzado y análisis predictivo. |
| Exploración de datos casi en tiempo real | La experiencia de usuario del [explorador de Azure Time Series Insights Gen2](./time-series-insights-update-explorer.md) proporciona la visualización de todos los flujos de datos mediante la canalización de ingesta. Después de conectarse a un origen del evento, puede ver, explorar y consultar datos del evento. De este modo, puede validar si un dispositivo emite los datos según lo previsto. También puede supervisar el mantenimiento, la productividad y la eficacia general de un recurso de IoT. | 
| Extensibilidad e integración | La integración del conector de Power BI está directamente disponible en la experiencia de usuario del Explorador de Time Series con la opción **Exportar**, que permite a los clientes exportar las consultas de series temporales que crean en nuestra experiencia de usuario directamente en el escritorio de Power BI y así ver los gráficos de series temporales junto con otros análisis de inteligencia empresarial. Esto abre la puerta a una nueva clase de escenarios para empresas de IoT industriales que han invertido en Power BI proporcionando un único panel en análisis de diversos orígenes de datos, incluida la serie temporal de IoT. | 
| Aplicaciones personalizadas creadas en la plataforma de Azure Time Series Insights Gen2 | Azure Time Series Insights Gen2 admite el [SDK de JavaScript](https://github.com/microsoft/tsiclient/blob/master/docs/API.md). El SDK ofrece controles enriquecidos y acceso simplificado a las consultas. Use el SDK para compilar aplicaciones personalizadas de IoT en Azure Time Series Insights Gen2 para satisfacer sus necesidades empresariales. También puede usar las [API de consulta](./concepts-query-overview.md) de Azure Time Series Insights Gen2 directamente para dirigir datos a las aplicaciones de IoT personalizadas. |

## <a name="next-steps"></a>Pasos siguientes

Introducción a Azure Time Series Insights Gen2:

> [!div class="nextstepaction"]
> [Guía de inicio rápido](./time-series-insights-update-quickstart.md)

Conozca los casos de uso:

> [!div class="nextstepaction"]
> [Casos de uso de Azure Time Series Insights Gen2](./time-series-insights-update-use-cases.md)