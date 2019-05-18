---
title: 'Información general: ¿Qué es Azure Time Series Insights? | Microsoft Docs'
description: Introducción a Azure Time Series Insights, un nuevo servicio de análisis de datos de series temporales y soluciones de IoT
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.workload: big-data
ms.topic: overview
ms.date: 04/26/2019
ms.custom: seodec18
ms.openlocfilehash: 8c7a24c4c4e071271d16de036ef6896e9d172913
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406322"
---
# <a name="what-is-azure-time-series-insights"></a>¿Qué es Azure Time Series Insights?

Azure Time Series Insights (TSI) se ha compilado para almacenar, visualizar y consultar grandes cantidades de datos de series temporales, como los generados por los dispositivos de IoT.  Si desea almacenar, administrar, consultar o visualizar los datos de una serie temporal en la nube, probablemente Time Series Insights sea lo que busca.  

![Diagrama de flujo de Time Series Insights](media/overview/time-series-insights-flowchart.png)

Time Series Insights tiene cuatro trabajos clave:

1. Está totalmente integrado con las puertas de enlace en la nube, como Azure IoT Hub y Azure Event Hubs. Y se conecta fácilmente a estos orígenes de eventos y analiza JSON desde mensajes y estructuras que tienen datos en columnas y filas limpias. Combina metadatos con telemetría e indexa los datos en un almacén en columnas.
1. TSI administra el almacenamiento de los datos. Para garantizar que se puede acceder fácilmente a los datos en todo momento, almacena los datos en memoria y en SSD hasta 400 días. Puede consultar interactivamente miles de millones de eventos en segundos, a petición.
1. TSI proporciona visualización lista para usar a través del explorador de TSI.  
1. TSI proporciona un servicio de consulta, tanto en el explorador de TSI como mediante el uso de API que son fáciles de integrar para incrustar datos de una serie temporal en aplicaciones personalizadas.  

Si va a compilar una aplicación, para consumo interno o para que la usen clientes externos, TSI puede utilizarse como back-end para la indexación, el almacenamiento y la agregación de datos de series temporales. A partir de aquí, puede compilar una experiencia de visualización y usuario personalizada mediante el [SDK de cliente](tutorial-explore-js-client-lib.md). TSI también está equipado con varias [API de consulta](how-to-shape-query-json.md) para habilitar estos escenarios personalizados.  

Los datos de serie temporal representan cómo cambia un recurso o un proceso con el tiempo. Los datos de las series temporales, por tanto, se indexan por las marcas de tiempo, y la hora es el eje más significativo en el que se organizan dichos datos. Normalmente, los datos de serie temporal llegan en orden secuencial y suelen tratarse como inserción en lugar de actualización de la base de datos como resultado.

En grandes volúmenes, almacenar, indexar, consultar, analizar y visualizar datos de series temporales pueden suponer un desafío.
Sin embargo, Azure Time Series Insights captura y almacena los eventos nuevos como filas y el cambio se mide eficazmente a lo largo del tiempo, lo cual permite buscar hacia atrás para extraer enseñanzas del pasado y predecir cambios.

## <a name="video"></a>Vídeo

### <a name="learn-more-about-azure-time-series-insights-the-cloud-based-iot-analytics-platformbr"></a>Más información sobre Azure Time Series Insights, la plataforma de análisis de IoT basada en la nube.</br>

[![VÍDEO](https://img.youtube.com/vi/GaARrFfjoss/0.jpg)](https://www.youtube.com/watch?v=GaARrFfjoss)

## <a name="primary-scenarios"></a>Escenarios principales

- Almacenamiento de datos de serie temporal de forma escalable.  
  - En esencia, Time Series Insights tiene una base de datos diseñada para datos de serie temporal.  Dado que es totalmente administrado y escalable, Time Series Insights realiza el trabajo de almacenar y administrar los eventos.

- Exploración de datos casi en tiempo real.  
  - Time Series Insights proporciona un explorador que visualiza todos los flujos de datos en un entorno.  Poco después de conectar un origen de eventos, los datos del evento se ven, exploran y consultan desde Time Series Insights.  Los datos son útiles para validar si un dispositivo emite los datos según lo previsto y para supervisar el estado, la productividad y la eficacia global de un recurso de IoT.  

- Análisis de las causas principales y detección de anomalías.
  - Time Series Insights tiene herramientas como patrones y vistas de perspectiva para llevar a cabo y guardar análisis de las causas principales de varios pasos.  Además, Time Series Insights funciona junto con servicios de alerta, como Azure Stream Analytics, para que las alertas y las anomalías detectadas se puedan ver casi en tiempo real en el explorador del primero.  

- Una vista global de los flujos de datos de serie temporal desde distintas ubicaciones permite la comparación entre varios recursos/sitios.
  - Puede conectar varios orígenes de eventos a un entorno de Time Series Insights.  Esto significa que los flujos de datos de varias ubicaciones dispares se pueden ver juntos casi en tiempo real.  Los usuarios pueden aprovechar esta visibilidad para compartir datos con líderes empresariales y permitir una mejor colaboración con expertos de dominio que puedan aplicar sus conocimientos para ayudar a resolver problemas, aplicar los procedimientos recomendados y compartir conocimientos.

- Compilación de una aplicación cliente a partir de Time Series Insights. 
  - Time Series Insights expone varias API de consulta de REST para que pueda compilar aplicaciones que usen datos de serie temporal.

## <a name="capabilities"></a>Capacidades

- **Introducción rápida**: Azure Time Series Insights no requiere la preparación inicial de los datos. Conéctese a millones de eventos en Azure IoT Hub o Event Hubs en minutos. Una vez conectado, visualice los datos de los sensores e interactúe con ellos para validar las soluciones de IoT con rapidez. Puede interactuar con los datos sin escribir código.
No hay que aprender ningún lenguaje nuevo; Time Series Insights proporciona una superficie de consultas granular y atextual para usuarios avanzados y permite apuntar y hacer clic como método de exploración.

- **Información casi en tiempo real**: Time Series Insights puede ingerir millones de eventos de sensor al día, con una latencia de un minuto. Time Series Insights ayuda a obtener información detallada sobre los datos de los sensores, al contribuir a detectar tendencias y anomalías, a llevar a cabo análisis de causa principal y a evitar costosos tiempos de inactividad. Al habilitar la correlación entre los datos en tiempo real e históricos, Time Series Insights le ayuda a desvelar tendencias ocultas en los datos.

- **Compilación de soluciones personalizadas**: inserte los datos de Azure Time Series Insights en aplicaciones existentes o cree soluciones personalizadas con las API de REST de Time Series Insights. Cree vistas personalizadas para compartir sus conocimientos con otros usuarios.

- **Escalabilidad:** Time Series Insights está diseñado para admitir IoT a escala. Puede incorporar de 1 a 100 millones de eventos al día, con un tiempo predeterminado de retención de 31 días. Puede visualizar y analizar flujos de datos activos casi en tiempo real, junto con datos históricos. Más adelante, las tasas de incorporación y retención aumentarán para adaptarse a una escala empresarial.

## <a name="getting-started"></a>Introducción

Con los primeros pasos se tarda menos de 5 minutos.

1. Para empezar, incorpore un entorno de Time Series Insights a Azure Portal.
1. Conecte un origen de eventos, por ejemplo, Azure IoT Hub o Event Hubs.  
1. Cargue los datos de referencia (no es un servicio adicional).
1. Consulte los datos en minutos con el explorador de Time Series Insights.

## <a name="time-series-insights-explorer"></a>Explorador de Time Series Insights

Este diagrama muestra un ejemplo de datos de Time Series Insights vistos con el explorador: ![Explorador de Time Series Insights](media/time-series-insights-explorer/explorer4.png)

## <a name="next-steps"></a>Pasos siguientes

- Explore el [entorno de demostración gratuito](./time-series-quickstart.md) en disponibilidad general de Azure Time Series Insights.

- Obtenga más información sobre [el planeamiento de su entorno de Time Series Insights](time-series-insights-environment-planning.md).
