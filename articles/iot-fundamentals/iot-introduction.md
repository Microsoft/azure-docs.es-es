---
title: Introducción a Internet de las cosas (IoT) de Azure
description: Esta introducción explica los fundamentos de IoT de Azure y los servicios de IoT e incluye ejemplos que ayudan a ilustrar el uso de IoT.
author: robinsh
ms.service: iot-fundamentals
services: iot-fundamentals
ms.topic: overview
ms.date: 10/11/2018
ms.author: robinsh
ms.openlocfilehash: e1cb588d68153a88d8b55b2696b376c4eb8704f5
ms.sourcegitcommit: d4f728095cf52b109b3117be9059809c12b69e32
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/10/2019
ms.locfileid: "54199291"
---
# <a name="what-is-azure-internet-of-things-iot"></a>¿Qué es Internet de las cosas (IoT) de Azure?

Internet de las cosas (IoT) de Azure es un conjunto de servicios en la nube administrados por Microsoft que permiten conectar, supervisar y controlar miles de millones de recursos de IoT. En términos más sencillos, una solución IoT se compone de uno o varios dispositivos IoT y uno o varios servicios de back-end que se ejecutan en la nube y se comunican entre sí. 

En este artículo describe los conceptos básicos de IoT, habla sobre los casos de uso y explica brevemente los ocho distintos servicios disponibles. Al comprender lo que está disponible, puede averiguar lo que desea examinar con más detalle para ayudar a diseñar su escenario.

## <a name="introduction"></a>Introducción

Las partes principales de una solución IoT son las siguientes: dispositivos, servicios de back-end y las comunicaciones entre los dos. 

### <a name="iot-devices"></a>Dispositivos IoT

Los dispositivos están formados generalmente por una placa de circuitos con sensores que se conecta a Internet. Muchos dispositivos se comunican mediante un chip Wi-Fi. Estos son algunos ejemplos de dispositivos IoT:

* Sensores de presión en una bomba de petróleo remota.
* Sensores de temperatura y humedad en una unidad de aire acondicionado.
* Acelerómetros en un ascensor.
* Sensores de presencia en una sala.

Dos dispositivos que se utilizan con frecuencia para la creación de prototipos son el chip MX de IoT Devkit básico de Microsoft y los dispositivos Raspberry PI. El chip MX de Devkit tiene sensores integrados para temperatura, presión, humedad, así como un giroscopio y un acelerómetro, un magnetómetro y un chip Wi-Fi. Raspberry PI es un dispositivo IoT al que puede conectar muchos tipos diferentes de sensores, para que pueda seleccionar exactamente lo que necesita para su escenario. 

Para más información sobre los dispositivos IoT disponibles, consulte el [catálogo de dispositivos certificados para IoT](https://catalog.azureiotsolutions.com/alldevices) más grande del sector.

Los [SDK de dispositivo IoT](../iot-hub/iot-hub-devguide-sdks.md) le permiten crear aplicaciones que se ejecutan en los dispositivos para que puedan realizar las tareas necesarias. Con los SDK, puede enviar datos de telemetría al centro de IoT, recibir mensajes y actualizaciones desde el centro de IoT, etc.

### <a name="communication"></a>Comunicación

El dispositivo se puede comunicar con los servicios de back-end en ambas direcciones. Estos son algunos ejemplos de formas de comunicación del dispositivo con la solución de back-end.

#### <a name="examples"></a>Ejemplos 

* El dispositivo puede enviar la temperatura desde un camión refrigerador móvil cada 5 minutos a un centro de IoT. 

* El servicio de back-end puede pedir al dispositivo que envíe los datos de telemetría con más frecuencia para ayudar a diagnosticar un problema. 

* El dispositivo puede enviar alertas en función de los valores que se leen de sus sensores. Por ejemplo, si supervisa un reactor por lotes en una planta química, puede enviar una alerta cuando las temperaturas superen un determinado valor.

* El dispositivo puede enviar información a un panel para su visualización por parte de los operadores humanos. Por ejemplo, una sala de control en una refinería puede mostrar la temperatura y la presión de cada canalización, así como el volumen que fluye a través de esa canalización, lo que permite a los operadores visualizarlo. 

Estas tareas, entre otras, se pueden implementar mediante los [SDK de dispositivo IoT](../iot-hub/iot-hub-devguide-sdks.md).

#### <a name="connection-considerations"></a>Consideraciones sobre la conexión

La conexión segura de dispositivos y la confiabilidad suele ser el mayor reto de las soluciones de IoT. Esto se debe a que los dispositivos IoT tienen características diferentes a las de otros clientes, como los exploradores y las aplicaciones para dispositivos móviles. En concreto, los dispositivos de IoT:

* A menudo son sistemas insertados sin operador humano (a diferencia de un teléfono).

* Se pueden implementar en ubicaciones remotas, donde el acceso físico resulta costoso.

* Es posible que solo sean accesibles a través del back-end de soluciones. No hay ninguna otra manera de interactuar con el dispositivo.

* Es posible que tengan limitaciones de recursos de procesamiento y alimentación.

* Es posible que tengan conectividad de red intermitente, lenta o costosa.

* Es posible que necesiten usar protocolos de aplicación propios, personalizados o específicos de determinados sectores.

### <a name="back-end-services"></a>Servicios de back-end 

Estas son algunas de las funciones que puede proporcionar un servicio de back-end.

* Recibir datos de telemetría a escala desde los dispositivos y determinar cómo procesar y almacenar esos datos.

* Analizar los datos de telemetría para proporcionar información detallada, ya sea en tiempo real o después de que se produzcan los hechos.

* Enviar comandos desde la nube a un dispositivo específico. 

* Aprovisionar dispositivos y controlar qué dispositivos se pueden conectar a la infraestructura.

* Controlar el estado de los dispositivos y supervisar sus actividades.

Por ejemplo, en un escenario de mantenimiento predictivo, el back-end en la nube almacena datos de telemetría históricos. La solución utiliza estos datos para identificar posibles comportamientos anómalos en bombas concretas antes de que causen un problema real. Mediante el análisis de los datos, identifica que la solución preventiva va a devolver un comando al dispositivo para emprender una acción correctiva. Este proceso genera un bucle de realimentación automatizado entre el dispositivo y la nube que aumenta en gran medida la eficacia de la solución.

## <a name="an-iot-example"></a>Un ejemplo de IoT

Este es un ejemplo de cómo ha utilizado IoT una empresa para ahorrar millones de dólares. 

Se trata de una enorme explotación ganadera con cientos de miles de vacas. Es complicado realizar un seguimiento de tantas vacas y saber qué están haciendo y requiere una gran cantidad de horas de conducción. Han adjuntado sensores a cada vaca que envían información como las coordenadas GPS y la temperatura a un servicio de back-end que lo escribe en una base de datos.

A continuación, tienen un servicio de análisis que examina los datos de entrada y analiza los datos de cada vaca para comprobar preguntas como las siguientes:

* ¿Tiene fiebre la vaca? ¿Cuánto tiempo lleva con fiebre? Si ha pasado más de un día, obtener las coordenadas GPS, encontrar la vaca y, si procede, tratarla con antibióticos. 

* ¿Está la vaca en el mismo lugar más de un día? Si es así, obtener las coordenadas GPS y encontrar la vaca. ¿Ha caído la vaca por un precipicio? ¿Está herida la vaca? ¿La vaca necesita ayuda? 

La implementación de esta solución IoT permite a la empresa comprobar y tratar las vacas rápidamente y reducir la cantidad de tiempo que tenían que dedicar a la conducción en torno a la comprobación de los animales, con el ahorro de una gran cantidad de dinero. Para más ejemplos reales de cómo utilizan IoT las empresas, consulte [Estudios de casos técnicos para IoT de Microsoft](https://microsoft.github.io/techcasestudies/#technology=IoT&sortBy=featured). 

## <a name="iot-services"></a>Servicios de IoT

Hay varios servicios relacionados con IoT en Azure y puede resultar confuso averiguar cuál de ellos desea usar. Algunos, como IoT Central y los aceleradores de soluciones IoT, proporcionan plantillas para ayudarle a crear su propia solución y empezar a trabajar rápidamente. También puede desarrollar completamente sus propias soluciones mediante otros servicios disponibles, todo depende de cuánta ayuda y cuánto control desee. Esta es una lista de los servicios disponibles, así como sus usos posibles.

1. [**IoT Central**](../iot-central/overview-iot-central.md): se trata de una solución SaaS que le ayuda a conectar, supervisar y administrar los dispositivos IoT. Para empezar, seleccione una plantilla para el tipo de dispositivo y cree y pruebe una aplicación básica de IoT Central que utilizarán los operadores del dispositivo. La aplicación de IoT Central también le permitirá supervisar los dispositivos y aprovisionar nuevos dispositivos. Este servicio es para soluciones sencillas que no requieren personalización profunda del servicio. 

2. [**Aceleradores de soluciones IoT**](/azure/iot-suite): se trata de una colección de soluciones PaaS que puede usar para acelerar el desarrollo de una solución IoT. Comience con una solución IoT proporcionada y personalice completamente esa solución según sus requisitos. Necesita conocimientos de Java o .NET para personalizar el back-end y conocimientos de JavaScript para personalizar la visualización. 

3. [**IoT Hub**](/azure/iot-hub/): este servicio le permite conectarse desde los dispositivos a un centro de IoT y supervisar y controlar miles de millones de dispositivos IoT. Es especialmente útil si necesita comunicación bidireccional entre los dispositivos IoT y el back-end. Se trata del servicio subyacente de IoT Central y los aceleradores de soluciones IoT. 

4. [**IoT Hub Device Provisioning Service**](/azure/iot-dps/): se trata de un servicio auxiliar de IoT Hub que puede usar para aprovisionar dispositivos en el centro de IoT de forma segura. Con este servicio, se pueden aprovisionar fácilmente millones de dispositivos con rapidez, en lugar de aprovisionarlos de uno en uno. 

5. [**IoT Edge**](/azure/iot-edge/): este servicio se basa en IoT Hub. Se puede usar para analizar los datos en los dispositivos IoT en lugar de en la nube. Al mover partes de la carga de trabajo al perímetro, es necesario enviar menos mensajes a la nube. 

6. [**Azure Digital Twins**](../digital-twins/index.yml): este servicio le permite crear modelos integrales del entorno físico. Puede modelar las relaciones y las interacciones entre personas, espacios y dispositivos. Por ejemplo, puede predecir las necesidades de mantenimiento de una fábrica, analizar los requisitos de energía en tiempo real de una red eléctrica u optimizar el uso del espacio disponible en una oficina.

7. [**Time Series Insights**](/azure/time-series-insights): este servicio le permite almacenar, visualizar y consultar grandes cantidades de datos de series temporales generados por los dispositivos IoT. Puede usar este servicio con IoT Hub. 

8. [**Azure Maps**](/azure/azure-maps): este servicio proporciona información geográfica para aplicaciones web y móviles. Hay un conjunto completo de API REST, así como un control JavaScript basado en web que se puede usar para crear aplicaciones flexibles que trabajan en aplicaciones de escritorio o móviles para dispositivos Apple y Windows.

## <a name="next-steps"></a>Pasos siguientes

Para algunos casos de negocio reales y la arquitectura utilizada, consulte [Estudios de casos técnicos para IoT de Microsoft Azure](https://microsoft.github.io/techcasestudies/#technology=IoT&sortBy=featured).

Para algunos proyectos de ejemplo que puede probar con IoT DevKit, consulte el [Catálogo de proyectos de IoT DevKit](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/). 

Para obtener una explicación más completa de los distintos servicios y cómo se usan, consulte [Servicios y tecnologías de IoT de Azure](iot-services-and-technologies.md).

Para ver un análisis detallado de la arquitectura de IoT, consulte [Microsoft Azure IoT Reference Architecture](https://aka.ms/iotrefarchitecture) (Arquitectura de referencia de IoT de Microsoft Azure).
