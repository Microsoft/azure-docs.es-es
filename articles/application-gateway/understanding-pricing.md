---
title: Descripción de los precios de Azure Application Gateway
description: En este artículo se describen el proceso de facturación de Azure Application Gateway y el firewall de aplicaciones web para las SKU v1 y v2.
services: application-gateway
author: azhar2005
ms.service: application-gateway
ms.topic: conceptual
ms.custom: references_regions
ms.date: 09/01/2020
ms.author: azhussai
ms.openlocfilehash: 07169c7e748bd427998f39f619496610bd09faa1
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89410374"
---
# <a name="understanding-pricing-for-azure-application-gateway-and-web-application-firewall"></a>Descripción de los precios de Azure Application Gateway y el firewall de aplicaciones web

>[!NOTE]
>Los precios que se muestran en este artículo son de ejemplo y tienen fines meramente ilustrativos. Para obtener información sobre los precios en su región, consulte la [página de precios](https://azure.microsoft.com/pricing/details/application-gateway/).

Azure Application Gateway es una solución de equilibrio de carga de capa 7 que permite distribuir aplicaciones web escalables, seguras y con una alta disponibilidad en Azure.

No hay costos iniciales ni costos de terminación asociados a Application Gateway.
Únicamente se facturará al usuario por los recursos previamente aprovisionados y utilizados en función del consumo real por hora. Los costos asociados a Application Gateway son de dos tipos: fijos y variables. Los costos reales de cada tipo variarán según la SKU que se utilice.

En este artículo se describen los costos asociados a cada SKU y se recomienda que los usuarios utilicen este documento para planear y administrar los costos asociados a Azure Application Gateway.

> [!NOTE]
> Una hora parcial se facturará como una hora completa.

## <a name="v1-skus"></a>SKU v1

Las SKU estándar de Application Gateway y WAF V1 se facturarán con arreglo a una combinación de:

* Costos fijos

    Los costos basados en el tiempo que un tipo determinado de instancia de Application Gateway/WAF se aprovisiona y ejecuta para procesar solicitudes.
    En los costos fijos, se tienen en cuenta las siguientes consideraciones:
    * Nivel: instancia de Application Gateway o WAF estándar
    * Tamaño: pequeño, mediano y grande
    * Recuento de instancias: el número de instancias para implementar

    En el caso de N instancias, los costos asociados equivaldrán al costo N * de una instancia para una combinación determinada de nivel (estándar o WAF) y tamaño (pequeño, mediano o grande).

* Costos variables

    Los costos basados en la cantidad de datos procesados por parte de Application Gateway/WAF. Los bytes de solicitud y de respuesta procesados por Application Gateway se tendrán en cuenta a efectos de facturación.

Costos totales = costos fijos + costos variables

### <a name="standard-application-gateway"></a>Instancia de Application Gateway estándar

Los costos fijos y los costos variables se calcularán en función del tipo de instancia de Application Gateway.
En la tabla siguiente se muestran precios de ejemplo basados en una ilustración de los precios del Este de EE. UU., con fines meramente ilustrativos.

#### <a name="fixed-costs-east-us-region-pricing"></a>Costos fijos (precios de la región Este de EE. UU.)

|              Tipo de instancia de Application Gateway             |  Costos ($/hora)  |
| ------------------------------------------------- | ---------------|
|                     Pequeña                         |    0,025 $      |
|                     Media                        |    0,07 USD       |
|                     grande                         |    0,32 $       |

Las estimaciones del precio mensual se basan en 730 horas de uso al mes.

#### <a name="variable-costs-east-us-region-pricing"></a>Costos variables (precios de la región Este de EE. UU.)

|              Datos procesados             |  Pequeño ($/GB)  |  Medio ($/GB) |  Grande ($/GB) |
| --------------------------------------- | -------------- | -------------- | ------------- |
| Primeros 10 TB/mes                       |     0,008 USD     |      Gratuito      |     Gratuito      |
| Siguientes 30 TB (10 – 40 TB)/mes             |     0,008 USD     |     0,007 $     |     Gratuito      |
| Más de 40 TB/mes                        |     0,008 USD     |     0,007 $     |     0,0035 $   |

Para obtener más información sobre los precios según su región, consulte la [página de precios](https://azure.microsoft.com/pricing/details/application-gateway/).

### <a name="waf-v1"></a>WAF V1

Los costos fijos y los costos variables se calcularán en función del tipo de instancia de Application Gateway que se haya aprovisionado.

En la tabla siguiente se muestran precios de ejemplo basados en una instantánea de los precios del Este de EE. UU., con fines meramente ilustrativos.

#### <a name="fixed-costs-east-us-region-pricing"></a>Costos fijos (precios de la región Este de EE. UU.)

|              Tipo de instancia de Application Gateway             |  Costos ($/hora)  |
| ------------------------------------------------- | ---------------|
|                     Pequeña                         |       N/D       |
|                     Media                        |     0,126 $     |
|                     grande                         |     0,448 $     |

Las estimaciones del precio mensual se basan en 730 horas de uso al mes.

#### <a name="variable-costs-east-us-region-pricing"></a>Costos variables (precios de la región Este de EE. UU.)

|              Datos procesados             |  Pequeño ($/GB)  |  Medio ($/GB) |  Grande ($/GB) |
| --------------------------------------- | -------------- | -------------- | ------------- |
| Primeros 10 TB/mes                       |     0,008 USD     |      Gratuito      |     Gratuito      |
| Siguientes 30 TB (10 – 40 TB)/mes             |     0,008 USD     |     0,007 $     |     Gratuito      |
| Más de 40 TB/mes                        |     0,008 USD     |     0,007 $     |     0,0035 $   |

Para obtener más información sobre los precios en su región, consulte la [página de precios](https://azure.microsoft.com/pricing/details/application-gateway/).

> [!NOTE]
> Transferencias de datos de salida: los datos que salgan de los centros de datos de Azure desde puertas de enlace de aplicaciones se cobrarán de acuerdo con las [tarifas de transferencia de datos](https://azure.microsoft.com/pricing/details/bandwidth/) estándar.

### <a name="example-1-a--standard-application-gateway-with-1-instance-count"></a>Ejemplo 1 (a): Application Gateway estándar con una instancia

Supongamos que ha aprovisionado una instancia de Application Gateway estándar de tipo medio con una sola instancia y que procesa 500 GB en un mes. Los costos de Application Gateway con arreglo a los precios mencionados anteriormente se calcularían de la manera siguiente:

Precio fijo = 0,07 $ * 730 horas = 51,1 $ Las estimaciones del precio mensual se basan en 730 horas de uso al mes.

Costos variables = gratis (el nivel medio no tiene costos por los primeros 10 TB procesados por mes) Costos totales = 51,1 $ + 0 = 51,1 $ 

> [!NOTE]
> Para admitir escenarios de alta disponibilidad, es necesario configurar al menos 2 instancias para SKU V1.

### <a name="example-1-b--standard-application-gateway-with--1-instance-count"></a>Ejemplo 1 (b): Application Gateway estándar con un recuento de instancias superior a 1

Supongamos que ha aprovisionado una Application Gateway estándar de tipo medio, con cinco instancias y que procesa 500 GB en un mes. Los costos de Application Gateway con arreglo a los precios mencionados anteriormente se calcularían de la manera siguiente:

Precio fijo = 5 instancias * 0,07 $ * 730 horas = 255,5 $ Las estimaciones del precio mensual se basan en 730 horas de uso al mes.

Costos variables = gratis (el nivel medio no tiene costos por los primeros 10 TB procesados al mes) Costos totales = 255,5 $ + 0 = 255,5 $ 

### <a name="example-2--waf-application-gateway"></a>Ejemplo 2: Instancia de Application Gateway de tipo WAF

Supongamos que ha aprovisionado una instancia de Application Gateway estándar pequeña y una instancia de Application Gateway grande de tipo WAF para los primeros 15 días del mes. La puerta de enlace de aplicaciones pequeña procesa 15 TB durante el tiempo que está activa, y la puerta de enlace de aplicaciones grande de tipo WAF procesa 100 TB durante el tiempo que está activa. Los costos de Application Gateway con arreglo a los precios mencionados anteriormente se calcularían de la manera siguiente: 

###### <a name="small-instance-standard-application-gateway"></a>Instancia de Application Gateway estándar pequeña

24 horas * 15 días = 360 horas

Precio fijo = 0,025 $ * 360 horas = 9 $

Costos variables = 10 * 1000 * 0,008 $/GB + 5 * 1000 * 0,008 $/GB = 120 $

Costos totales= 9 $ + 120 $ = 129 $

###### <a name="large-instance-waf-application-gateway"></a>Instancia de Application Gateway grande de tipo WAF
24 horas * 15 días = 360 horas

Precio fijo = 0,448 $ * 360 horas = 161,28 $

Costos variables = 60 * 1000 * 0,0035 $/GB = 210 $ (el nivel grande no tiene costos por los primeros 40 TB procesados por mes)

Costos totales= 161,28 $ + 210 $ = 371,28 $

## <a name="v2-skus"></a>SKU V2  

Las instancias de Application Gateway V2 y WAF V2 admiten el escalado automático y garantizan una alta disponibilidad de forma predeterminada.

### <a name="key-terms"></a>Términos clave

##### <a name="capacity-unit"></a>Unidad de capacidad 
Una unidad de capacidad (CU) mide el uso de capacidad para una instancia de Application Gateway en relación con varios parámetros.

Una sola unidad de capacidad consta de los siguientes parámetros:
* 2500 conexiones persistentes
* Rendimiento de 2,22 Mbps
* 1 unidad de proceso

Si se supera uno de estos parámetros, se necesitarán unidades de capacidad adicionales, aunque los otros dos parámetros no superen los límites de la unidad de capacidad única.
El parámetro con el uso más elevado entre los tres anteriores se utilizará internamente para calcular las unidades de capacidad, que posteriormente se facturarán.

##### <a name="compute-unit"></a>Unidad de proceso
La unidad de proceso mide la capacidad de procesador consumida. Entre los factores que afectan a la unidad de proceso, se encuentran las conexiones TLS/seg, los cálculos de reescritura de direcciones URL y el procesamiento de reglas de WAF. El número de solicitudes que puede manejar una unidad de proceso depende de varios criterios, como el tamaño de la clave del certificado TLS, el algoritmo de intercambio de claves, las reescrituras de encabezados y, en el caso de WAF, el tamaño de la solicitud entrante.

Orientación para la unidad de proceso:
* Standard_v2: cada unidad de proceso tiene capacidad para aproximadamente 50 conexiones por segundo con el certificado TLS de la clave RSA de 2048 bits.

* WAF_v2: cada unidad de proceso puede admitir aproximadamente 10 solicitudes simultáneas por segundo para una combinación de tráfico del 70-30 %, con el 70 % de solicitudes de menos de 2 KB de GET/POST y el resto más. El rendimiento de WAF no se ve afectado por el tamaño de respuesta actualmente.

##### <a name="instance-count"></a>Recuento de instancias 
 El aprovisionamiento previo de recursos para las SKU de Application Gateway V2 se define en función del recuento de instancias. Cada instancia garantiza al menos 10 unidades de capacidad en términos de capacidad de procesamiento. La misma instancia podría admitir más de 10 unidades de capacidad para diferentes patrones de tráfico en función de los parámetros de la unidad de capacidad.

Los límites y el escalado definidos manualmente para el escalado automático (mínimo o máximo) se establecen en función del recuento de instancias. El escalado definido manualmente para el recuento de instancias y el recuento mínimo de instancias en la configuración de escalado automático reservarán 10 unidades de capacidad por instancia. Estos valores de unidades de capacidad (CU) reservados se facturarán siempre que la instancia de Application Gateway esté activa, independientemente del consumo real de los recursos. Si el consumo real supera el umbral de 10 unidades de capacidad por instancia, se facturarán unidades de capacidad adicionales con arreglo al elemento variable.

Las SKU V2 se facturan en función del consumo y tienen dos elementos:

* Costos fijos

    Los costos basados en el tiempo que la instancia de Application Gateway V2 o WAF V2 se aprovisiona y está disponible para procesar solicitudes. De este modo, se garantiza una alta disponibilidad, incluso si se han reservado 0 instancias al especificar 0 unidades de escalado mínimas como parte del escalado automático. 
    
    Los costos fijos también incluyen el costo asociado a la dirección IP pública vinculada a la instancia de Application Gateway.

    El número de instancias que se ejecutan en un momento dado no es un factor que se tenga en cuenta en los costos fijos de las SKU V2. Los costos fijos asociados a la ejecución de una instancia Standard_V2 (o WAF_V2) serían los mismos por hora, independientemente del número de instancias que se ejecuten dentro de la misma región de Azure.

* Costos de unidades de capacidad 

    Los costos basados en el número de unidades de capacidad reservadas o creadas con escalado automático en función de las necesidades para procesar las solicitudes entrantes.  Los costos basados en el consumo se calculan por hora.

Costos totales = costos fijos + costos de unidades de capacidad

> [!NOTE]
> Una hora parcial se facturará como una hora completa.

En la tabla siguiente se muestran precios de ejemplo basados en una instantánea de los precios del Este de EE. UU., con fines meramente ilustrativos.

#### <a name="fixed-costs-east-us-region-pricing"></a>Costos fijos (precios de la región Este de EE. UU.)

|              SKU V2             |  Costos ($/hora)  |
| ------------------------------- | ---------------|
|            Standard_v2          |     0,246 $     |
|              WAF_V2             |     0,443 $     |

Las estimaciones del precio mensual se basan en 730 horas de uso al mes.

#### <a name="variable-costs-east-us-region-pricing"></a>Costos variables (precios de la región Este de EE. UU.)

|        Unidad de capacidad         |  Standard_V2 ($/hora)  |  WAF_V2 ($/hora) |
| ---------------------------- | -------------------- | -------------- |
|             1 unidad de capacidad             |        0,008 USD        |     0,0144 $    |

Para obtener más información sobre los precios en su región, consulte la [página de precios](https://azure.microsoft.com/pricing/details/application-gateway/).

> [!NOTE]
> Transferencias de datos de salida: los datos que salgan de los centros de datos de Azure desde puertas de enlace de aplicaciones se cobrarán de acuerdo con las [tarifas de transferencia de datos](https://azure.microsoft.com/pricing/details/bandwidth/) estándar.

### <a name="example-1-a--manual-scaling"></a>Ejemplo 1 (a): Escalado manual 
Supongamos que ha aprovisionado una instancia Standard_V2 de Application Gateway, con el escalado manual establecido en 8 unidades de escalado para todo el mes. Durante este tiempo, recibe una transferencia de datos media de 88,8 Mbps.

Los costos de Application Gateway con arreglo a los precios mencionados anteriormente se calcularían de la manera siguiente:

1 unidad de capacidad puede administrar un rendimiento de 2,22 Mbps.

Unidades de capacidad necesarias para administrar 88,8 Mbps = 88,8/2,22 = 40 unidades de capacidad 

Unidades de capacidad aprovisionadas previamente = 8 instancias * 10 = 80 

Dado que 80 unidades de capacidad reservada > 40 unidades de capacidad necesaria, no se requieren unidades de capacidad adicionales 

Precio fijo = 0,246 $ * 730 horas = 179,58 $

Costos variables = 0,008 $ * 8 unidades de instancia * 10 unidades de capacidad * 730 horas = 467,2 $

Costos totales = 179,58 $ + 467,2 $ = 646,78 $

![Diagrama de escalado manual 1.](./media/pricing/manual-scale-1.png)

### <a name="example-1-b--manual-scaling-with-traffic-going-beyond-set-capacity"></a>Ejemplo 1 (b): Escalado manual con un tráfico que supera la capacidad establecida

Supongamos que ha aprovisionado una instancia Standard_V2 de Application Gateway, con el escalado manual establecido en 3 unidades de escalado para todo el mes. Durante este tiempo, recibe una transferencia de datos media de 88,8 Mbps.

Los costos de Application Gateway con arreglo a los precios mencionados anteriormente se calcularían de la manera siguiente:

1 unidad de capacidad puede administrar un rendimiento de 2,22 Mbps.

Unidades de capacidad necesarias para administrar 88,8 Mbps = 88,8/2,22 = 40 

Unidades de capacidad aprovisionadas previamente = 3 instancias * 10 = 30 

Dado que 40 unidades de capacidad necesaria > 30 unidades de capacidad reservada, se requieren unidades de capacidad adicionales.
El número de unidades de capacidad adicionales que se utilicen dependerá de la capacidad libre disponible con cada instancia.

Si había capacidad de procesamiento disponible equivalente a 5 unidades de capacidad adicionales para utilizar en las 3 instancias reservadas.

Precio fijo = 0,246 $ * 730 horas = 179,58 $

Costos variables = 0,008 $ * (3 unidades de instancia * 10 unidades de capacidad + 5 unidades de capacidad adicionales) * 730 horas = 204,4 $

Costos totales = 179,58 $ + 204,4 $ = 383,98 $

![Diagrama de escalado manual 2.](./media/pricing/manual-scale-2.png)

> [!NOTE]
> En el caso del escalado manual, se descartarán las solicitudes adicionales que superen la capacidad de procesamiento máxima de las instancias reservadas. En situaciones con un elevado volumen de cargas, las instancias reservadas pueden proporcionar más de 10 unidades de capacidad de procesamiento en función de la configuración y del tipo de solicitudes entrantes.

### <a name="example-2--waf_v2-instance-with-autoscaling"></a>Ejemplo 2: Instancia de WAF_V2 con escalado automático

Supongamos que ha aprovisionado una instancia WAF_V2 con el escalado automático habilitado y ha establecido la configuración de unidades de escalado mínimas en 6 unidades para todo el mes. La carga de solicitudes ha provocado que la instancia de WAF se escale horizontalmente y use 65 unidades de capacidad (escalabilidad horizontal de 5 unidades de capacidad, con 60 unidades reservadas) para todo el mes.
Los costos de Application Gateway con arreglo a los precios mencionados anteriormente se calcularían de la manera siguiente:

Las estimaciones del precio mensual se basan en 730 horas de uso al mes.

Precio fijo = 0,443 $ * 730 horas = 323,39 $

Costos variables = 0,0144 $ * 65 unidades de capacidad * 730 horas = 683,28 $

Costos totales = 323,39 $ + 683,28 $ = 1006,67 $

![Diagrama de escalado automático 2.](./media/pricing/auto-scale-1.png)

> [!NOTE]
> No es probable que el tráfico real observado para su instancia de Application Gateway tenga un patrón de tráfico constante; y la carga observada en su instancia de Application Gateway fluctuaría según el uso real.

### <a name="example-3-a--waf_v2-instance-with-autoscaling-and-0-min-scale-config"></a>Ejemplo 3 (a): Instancia de WAF_V2 con escalado automático y una configuración de escalado mínimo igual a 0

Supongamos que ha aprovisionado una instancia WAF_V2 con el escalado automático habilitado y ha establecido la configuración de unidades de escalado mínimas en 0 unidades para todo el mes. La carga de solicitudes en la instancia de WAF es mínima, pero está presente de forma constante por hora para todo el mes. La carga está por debajo de la capacidad de una única unidad de capacidad.
Los costos de Application Gateway con arreglo a los precios mencionados anteriormente se calcularían de la manera siguiente:

Las estimaciones del precio mensual se basan en 730 horas de uso al mes.

Precio fijo = 0,443 $ * 730 horas = 323,39 $

Costos variables = 0,0144 $ * 1 unidad de capacidad * 730 horas = 10,512 $

Costos totales = 323,39 $ + 10,512 $ = 333,902 $

### <a name="example-3-b--waf_v2-instance-with-autoscaling-with-0-min-scale-config"></a>Ejemplo 3 (b): Instancia de WAF_V2 con escalado automático y una configuración de escalado mínimo igual a 0 

Supongamos que ha aprovisionado una instancia WAF_V2 con el escalado automático habilitado y ha establecido la configuración de unidades de escalado mínimas en 0 unidades para todo el mes. Sin embargo, no hay ningún tráfico dirigido a la instancia de WAF durante todo el mes.
Los costos de Application Gateway con arreglo a los precios mencionados anteriormente se calcularían de la manera siguiente:

Precio fijo = 0,443 $ * 730 horas = 323,39 $

Costos variables = 0,0144 $ * 0 unidades de capacidad * 730 horas = 0 $

Costos totales = 323,39 $ + 0 $ = 323,39 $

### <a name="example-3-c--waf_v2-instance-with-manual-scale-set-to-1-instance"></a>Ejemplo 3 (C): Instancia de WAF_V2 con escalado manual establecido en 1 instancia

Supongamos que ha aprovisionado una instancia de WAF_V2 y la establece en el escalado manual con el valor mínimo aceptable de 1 unidad de instancia para todo el mes. Sin embargo, no hay ningún tráfico dirigido a la instancia de WAF durante todo el mes.
Los costos de Application Gateway con arreglo a los precios mencionados anteriormente se calcularían de la manera siguiente:

Las estimaciones del precio mensual se basan en 730 horas de uso al mes.

Precio fijo = 0,443 $ * 730 horas = 323,39 $

Costos variables = 0,0144 $ * 1 recuento de instancias * 10 unidades de capacidad * 730 horas = 105,12 $ (1 unidad de instancia reservada)

Costos totales = 323,39 $ + 105,12 $ = 428,51 $

### <a name="example-4--waf_v2-with-autoscaling-capacity-unit-calculations"></a>Ejemplo 4: Instancia de WAF_V2 con escalado automático y cálculos basados en la unidad de capacidad

Supongamos que ha aprovisionado una instancia WAF_V2 con el escalado automático habilitado y ha establecido la configuración del recuento de instancias mínimas en 0 para todo el mes. Durante este tiempo, recibe 25 nuevas conexiones TLS por segundo, un promedio de transferencia de datos de 8,88 Mbps.
Los costos de Application Gateway con arreglo a los precios mencionados anteriormente se calcularían de la manera siguiente:

Las estimaciones del precio mensual se basan en 730 horas de uso al mes.

Precio fijo = 0,443 $ * 730 horas = 323,39 $

Costos variables = 0,0144 $ * 730 horas * {Máx. (25/50, 8,88/2,22)} = 23,36 $ (4 unidades de capacidad necesarias para administrar 8,88 Mbps)

Costos totales = 323,39 $ + 23,36 $ = 346,75 $

### <a name="example-5-a--standard_v2-with-autoscaling-time-based-calculations"></a>Ejemplo 5 (a): Standard_V2 con escalado automático y cálculos basados en tiempo

Supongamos que ha aprovisionado una instancia standard_V2 con el escalado automático habilitado y ha establecido la configuración de unidades de escalado mínimas en 0 unidades. Esta instancia de Application Gateway está activa durante 2 horas.
Durante la primera hora, recibe tráfico que se puede administrar mediante 10 unidades de capacidad y, durante la segunda hora, recibe tráfico que requiere 20 unidades de capacidad para administrar la carga.
Los costos de Application Gateway con arreglo a los precios mencionados anteriormente se calcularían de la manera siguiente:

Precio fijo = 0,246 $ * 2 horas = 0,492 $

Costos variables = 0,008 $ * 10 unidades de capacidad * 1 hora + 0,008 $ * 20 unidades de capacidad * 1 hora = 0,24 $

Costos totales = 0,492 $ + 0,24 $ = 0,732 $


## <a name="monitoring-billed-usage"></a>Supervisión del uso facturado

Es posible ver el consumo realizado en relación con diferentes parámetros (unidad de proceso, rendimiento y conexiones persistentes), así como las unidades de proceso utilizadas como parte de las métricas de Application Gateway en la sección de **supervisión**.

![Diagrama de la sección de métricas.](./media/pricing/metrics-1.png)

### <a name="useful-metrics-for-cost-estimation"></a>Métricas utilizadas en la estimación de costos

* Unidades de capacidad actuales

    El número de unidades de capacidad consumidas para equilibrar la carga del tráfico entre los tres parámetros: conexiones persistentes, rendimiento y unidad de proceso.

* Unidades de capacidad facturables fijas

    El número mínimo de unidades de capacidad que se mantienen aprovisionadas según la configuración de Unidades de escalado mínimas (una instancia se traduce en 10 unidades de capacidad) en la configuración de Application Gateway.

* Unidades de capacidad facturadas estimadas

    Las unidades de capacidad facturadas estimadas indican el número de unidades de capacidad que se usa para estimar la facturación. Se calcula como el valor mayor entre las unidades de capacidad actuales (unidades de capacidad necesarias para equilibrar la carga del tráfico) y las unidades de capacidad facturables fijas (unidades de capacidad mínimas mantenidas).

Hay disponibles otras métricas como el rendimiento, las conexiones actuales y las unidades de proceso para comprender los cuellos de botella y calcular el número de unidades de capacidad necesarias. Encontrará información detallada en: [Métricas para Application Gateway](https://docs.microsoft.com/azure/application-gateway/application-gateway-metrics#application-gateway-metrics)

#### <a name="example-1-a---estimating-capacity-units-being-utilized"></a>Ejemplo 1 (a): Estimación de las unidades de capacidad en uso

**Métricas observadas:**

Unidades de proceso = 22,25

Rendimiento = 16 Mbps (2000000B)

Conexiones actuales= 178 250

Unidades de capacidad calculadas = máx. (22,25; 16/2,22; 178,25 * 1000/2500) = 71,3

Unidades de capacidad observadas en las métricas = 72,3

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos para saber cómo funcionan los precios en Azure Application Gateway:

* [Página de precios de Azure Application Gateway](https://azure.microsoft.com/pricing/details/application-gateway/)
* [Calculadora de precios de Azure Application Gateway](https://azure.microsoft.com/en-us/pricing/calculator/?service=application-gateway)
