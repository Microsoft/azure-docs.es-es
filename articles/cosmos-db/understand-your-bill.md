---
title: Introducción a la factura de Azure Cosmos DB
description: En este artículo se explica cómo entender la factura de Azure Cosmos DB con algunos ejemplos.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/19/2020
ms.reviewer: sngun
ms.openlocfilehash: bf041163c6b2759b3d38e48ee98a0d528ec601db
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88606904"
---
# <a name="understand-your-azure-cosmos-db-bill"></a>Entienda la factura de Azure Cosmos DB

Como servicio de bases de datos nativo de la nube totalmente administrado, Azure Cosmos DB simplifica la facturación al cobrar solo las operaciones de la base de datos y el almacenamiento consumido. No existen tarifas de licencia adicionales, costos de hardware, de utilidades ni de instalaciones en comparación con las alternativas locales u hospedadas en IaaS. Al considerar las funcionalidades de varias regiones de Azure Cosmos DB, el servicio de base de datos proporciona una considerable reducción de costos en comparación con soluciones locales o IaaS.

- **Operaciones de la base de datos**: la forma en que se le cobran las operaciones de base de datos depende del tipo de cuenta de Azure Cosmos que se use.

  - **Rendimiento aprovisionado**: la facturación se realiza por hora al rendimiento máximo aprovisionado para una concreta, en incrementos de 100 RU/s.
  - **Sin servidor**: la facturación se realiza por hora a la cantidad total de unidades de solicitud usada por las operaciones de base de datos.

- **Almacenamiento**: se le cobra una tarifa plana por la cantidad total de almacenamiento (GB) usada por los datos y los índices en una hora concreta.

Consulte la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/) para obtener la información sobre los precios más reciente.

En este artículo se usan algunos ejemplos para ayudarle a entender los detalles que ve en la factura mensual. Los números que se muestran en los ejemplos pueden ser diferentes si los contenedores de Azure Cosmos tienen una cantidad diferente de rendimiento aprovisionado, si abarcan varias regiones o se ejecutan para un periodo diferente durante un mes. En todos los ejemplos de este artículo se calcula la factura según la información de precios que se muestra en la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/).

> [!NOTE]
> La facturación es para cualquier parte de una hora de reloj, no una duración de 60 minutos. Todos los ejemplos que se muestran en este documento se basan en el precio de una cuenta de Azure Cosmos implementada en una región no gubernamental de Estados Unidos. Los precios y el cálculo varían en función de la región que use; consulte la [página de precios de Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/) para obtener más información.

## <a name="billing-examples"></a>Ejemplos de facturación

### <a name="billing-example---provisioned-throughput-on-a-container-full-month"></a>Ejemplo de facturación: rendimiento aprovisionado en un contenedor (mes completo)

* Supongamos que configura un rendimiento de 1000 RU/s en un contenedor, y existe durante 24 horas * 30 días del mes = 720 horas en total.  

* 1000 RU/s son 10 unidades de 100 RU/s por hora por cada hora que existe el contenedor (es decir, 1000/100 = 10). 

* Multiplicar 10 unidades por hora por el costo de 0,008 USD (por 100 RU/s por hora) = 0,08 USD por hora. 

* Multiplicar los 0,08 USD por hora por el número de horas del mes equivale a 0,08 USD * 24 horas * 30 días = 57,60 USD para el mes.  

* La factura mensual total mostrará 7200 unidades (de 100 RU), lo que costará 57,60 USD.

### <a name="billing-example---provisioned-throughput-on-a-container-partial-month"></a>Ejemplo de facturación: rendimiento aprovisionado en un contenedor (mes parcial)

* Supongamos que creamos un contenedor con rendimiento aprovisionado de 2500 RU/s. El contenedor tiene una duración de 24 horas durante el mes (por ejemplo, lo eliminamos 24 horas después de crearlo).  

* Veremos 600 unidades en la factura (2500 RU/s / 100 RU/s/unidad * 24 horas). El costo será de 4,80 USD (600 unidades * 0,008 USD/unidad).

* La factura total para el mes será de 4,80 USD.

### <a name="billing-example---serverless-container"></a>Ejemplo de facturación: contenedor sin servidor

* Supongamos que creamos un contenedor sin servidor. 

* A lo largo de un mes, emitimos solicitudes de base de datos que usan un total de 500 000 unidades de solicitud. El costo será de 0,125 USD (500 000 * 0,25 USD/millón).

* La factura total para el mes será de 0,125 USD.

### <a name="billing-rate-if-storage-size-changes"></a>Tasa de facturación si cambia el tamaño de almacenamiento

La capacidad de almacenamiento se factura en unidades de la cantidad máxima de datos almacenados por hora (en GB) durante el periodo de un mes. Por ejemplo, si ha utilizado 100 GB de almacenamiento durante medio mes y 50 GB durante la segunda mitad del mes, se le facturará un uso medio de 75 GB de almacenamiento ese mes.

### <a name="billing-rate-when-container-or-a-set-of-containers-are-active-for-less-than-an-hour"></a>Tasa de facturación cuando el contenedor o un conjunto de contenedores están activos durante menos de una hora

Se le cobra la tarifa plana por cada hora durante la cual exista el contenedor o la base de datos, independientemente del uso o de si el contenedor o la base de datos están activos durante menos de una hora. Por ejemplo, si crea un contenedor o una base de datos y los elimina a los 5 minutos, en la factura aparecerá como 1 hora.

### <a name="billing-rate-when-provisioned-throughput-on-a-container-or-database-scales-updown"></a>Tasa de facturación cuando el rendimiento aprovisionado en un contenedor o base de datos se escala o reduce verticalmente

Si aumenta el rendimiento aprovisionado a las 9:30 de la mañana de 400 RU/s a 1000 RU/s y lo vuelve a reducir a las 10:45 a 400 RU/s, se le cobrarán dos horas de 1000 RU/s. 

Si aumenta el rendimiento aprovisionado para un contenedor o un conjunto de contenedores a las 9:30 de la mañana de 100 000 RU/s a 200 000 RU/s y luego reduce el rendimiento aprovisionado a las 10:45 nuevamente a 100 000 RU/s, se le cobrarán dos horas de 200 000 RU/s.

### <a name="billing-example-multiple-containers-each-with-dedicated-provisioned-throughput-mode"></a>Ejemplo de facturación: varios contenedores, cada uno con un modo de rendimiento aprovisionado dedicado

* Si crea una cuenta de Azure Cosmos en la región Este de EE. UU. 2 con dos contenedores que tienen aprovisionado un rendimiento de 500 RU/s y 700 RU/s respectivamente, tiene un rendimiento aprovisionado total de 1200 RU/s.  

* Se le cobrarán 1200/100 * 0,008 USD = 0,096 USD/hora. 

* Si sus necesidades de rendimiento cambiaron y la capacidad de cada contenedor aumentó en 500 RU/s a la vez que se crea un nuevo contenedor ilimitado con 20 000 RU/s, la capacidad aprovisionada general sería de 22 200 RU/s (1000 RU/s + 1200 RU/s + 20 000 RU/s).  

* La factura cambiaría a: 0,008 USD x 222 = 1,776 USD/hora. 

* En un mes de 720 horas (24 horas * 30 días), si el rendimiento aprovisionado durante 500 horas fue de 1200 RU/s y para las 220 horas restantes fue de 22 200 RU/s, la factura del mes indicaría: 500 x 0,096 USD/hora + 220 x 1,776 USD/hora = 438,72 USD/mes.

:::image type="content" source="./media/understand-your-bill/bill-example1.png" alt-text="Ejemplo de factura de rendimiento dedicado":::

### <a name="billing-example-containers-with-shared-provisioned-throughput-mode"></a>Ejemplo de facturación: contenedores con un modo de rendimiento (aprovisionado) compartido

* Si crea una cuenta de Azure Cosmos en Este de EE. UU. 2 con dos bases de datos de Azure Cosmos (con un conjunto de contenedores que comparten el rendimiento en el nivel de la base de datos) con un rendimiento aprovisionado de 50 000 RU/s y 70 000 RU/s, respectivamente, tendría un rendimiento aprovisionado total de 120 000 RU/s.  

* Se le cobrarán 1200 x 0,008 USD = 9,60 USD/hora. 

* Si cambiaron sus necesidades de rendimiento y aumentó el rendimiento aprovisionado de cada base de datos en 10 000 RU/s para cada base de datos, y agrega un nuevo contenedor para la primera base de datos con el modo de rendimiento dedicado de 15 000 RU/s para la base de datos de rendimiento compartido, la capacidad aprovisionada total sería de 155 000 RU/s (60 000 RU/s + 80 000 RU/s + 15 000 RU/s).  

* La factura cambiaría a: 1550 * 0,008 USD = 12,40 USD/hora.  

* En un mes de 720 horas, si para 300 horas el rendimiento aprovisionado fue de 120 000 RU/s y para las 420 horas restantes el rendimiento aprovisionado fue de 155 000 RU/s, la factura del mes indicaría: 300 x 9,60 USD/hora + 420 x 12,40 USD/hora = 2,880 USD + 5,208 USD/mes = 8,088 USD/mes. 

:::image type="content" source="./media/understand-your-bill/bill-example2.png" alt-text="Ejemplo de factura de rendimiento compartido":::

## <a name="billing-examples-with-geo-replication-and-multi-master"></a>Ejemplos de facturación con replicación geográfica y arquitectura multimaestro  

Puede agregar o quitar regiones de Azure de cualquier parte del mundo en la cuenta de base de datos de Azure Cosmos en cualquier momento. El rendimiento que ha configurado para diferentes contenedores y bases de datos de Azure Cosmos se reserva en cada una de las regiones de Azure asociadas con la cuenta de base de datos de Azure Cosmos. Si la suma del rendimiento aprovisionado (RU/s) configurado en todas las bases de datos y los contenedores dentro de la cuenta de bases de datos de Azure Cosmos (aprovisionada por hora) es T y el número de regiones de Azure asociadas con su cuenta de base de datos es N, el rendimiento aprovisionado total para una hora dada, para su cuenta de base de datos, (a) configurado con una única región de escritura es igual a T x N RU/s y (b) configurado con todas las regiones capaces de procesar las operaciones de escritura es igual a T x (N+1) RU/s, respectivamente. El rendimiento aprovisionado (una sola región de escritura) cuesta 0,008 USD/hora por cada 100 RU/s, y el rendimiento aprovisionado con varias regiones de escritura (configuración de arquitectura multimaestro) cuesta 0,016 USD/hora por cada 100 RU/s (consulte la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/)). Ya sea que se trate de una sola región de escritura o de varias regiones de escritura, Azure Cosmos DB le permite leer datos desde cualquier región.

### <a name="billing-example-multi-region-azure-cosmos-account-single-region-writes"></a>Ejemplo de facturación: cuenta de Azure Cosmos de varias regiones, escrituras en una sola región

Supongamos que tiene un contenedor de Azure Cosmos en Oeste de EE. UU. El contenedor se crea con un rendimiento de 10 000 RU/s y almacena 1 TB de datos este mes. Supongamos que agrega tres regiones (Este de EE. UU., Norte de Europa y Este de Asia) a su cuenta de Azure Cosmos, cada una con la misma cantidad de rendimiento y almacenamiento. Su factura total mensual (suponiendo que un mes tiene 30 días) sería como sigue: 

|**Elemento** |**Uso (mes)** |**Tarifa** |**Costo mensual** |
|---------|---------|---------|-------|
|Factura de rendimiento por el contenedor en Oeste de EE. UU.      | 10 000  RU/s * 24 * 30    |0,008 USD por 100 RU/s por hora   |576 USD|
|Factura de rendimiento para 3 regiones adicionales: Este de EE. UU., Norte de Europa y Este de Asia       | 3 * 10 000 RU/s * 24 * 30    |0,008 USD por 100 RU/s por hora  |1728 USD|
|Factura de almacenamiento por el contenedor en Oeste de EE. UU.      | 250 GB    |0,25 USD/GB  |62,50 USD|
|Factura de almacenamiento para 3 regiones adicionales: Este de EE. UU., Norte de Europa y Este de Asia      | 3 * 250 GB    |0,25 USD/GB  |187,50 USD|
|**Total**     |     |  |**2554 USD**|

*Suponga que hace salir 100 GB de datos todos los meses del contenedor de la región Oeste de EE. UU. para replicar datos en las regiones Este de EE. UU., Norte de Europa y Este de Asia. Se le facturará la salida según las tarifas de transferencia de datos.*

### <a name="billing-example-multi-region-azure-cosmos-account-multi-region-writes"></a>Ejemplo de facturación: cuenta de Azure Cosmos de varias regiones, escrituras en varias regiones

Supongamos que crea un contenedor de Azure Cosmos en Oeste de EE. UU. El contenedor se crea con un rendimiento de 10 000 RU/s y almacena 1 TB de datos este mes. Suponga que agrega tres regiones (Este de EE. UU., Norte de Europa y Este de Asia), cada una con el mismo almacenamiento y rendimiento, y quiere ser capaz de escribir en los contenedores de todas las regiones asociadas con su cuenta de Azure Cosmos. Su factura total mensual (suponiendo que un mes tiene 30 días) será:

|**Elemento** |**Uso (mes)**|**Tarifa** |**Costo mensual** |
|---------|---------|---------|-------|
|Factura de rendimiento para un contenedor en Oeste de EE. UU. (se puede escribir en todas las regiones)       | 10 000  RU/s * 24 * 30    |0,016 USD por 100 RU/s por hora    |1152 USD |
|Factura de rendimiento para 3 regiones adicionales: Este de EE. UU., Norte de Europa y Este de Asia (se puede escribir en todas las regiones)        | (3 + 1) * 10 000 RU/s * 24 * 30    |0,016 USD por 100 RU/s por hora   |4608 USD |
|Factura de almacenamiento por el contenedor en Oeste de EE. UU.      | 250 GB    |0,25 USD/GB  |62,50 USD|
|Factura de almacenamiento para 3 regiones adicionales: Este de EE. UU., Norte de Europa y Este de Asia      | 3 * 250 GB    |0,25 USD/GB  |187,50 USD|
|**Total**     |     |  |**6010 USD**|

*Suponga que hace salir 100 GB de datos todos los meses del contenedor de la región Oeste de EE. UU. para replicar datos en las regiones Este de EE. UU., Norte de Europa y Este de Asia. Se le facturará la salida según las tarifas de transferencia de datos.*

### <a name="billing-example-azure-cosmos-account-with-multi-master-database-level-throughput-including-dedicated-throughput-mode-for-some-containers"></a>Ejemplo de facturación: Cuenta de Azure Cosmos con rendimiento en el nivel de la base de datos, arquitectura multimaestro, incluido el modo de rendimiento dedicado para algunos contenedores

Consideremos el ejemplo siguiente, donde se tiene una cuenta de Azure Cosmos de varias regiones donde se puede escribir en todas las regiones (configuración de arquitectura multimaestro). Para simplificar, supondremos que el tamaño de almacenamiento permanece constante y no cambia, y lo omitiremos aquí para que el ejemplo sea más sencillo. El rendimiento aprovisionado durante el mes varió como sigue (suponiendo 30 días o 720 horas)\: 

[0-100 horas]\:  

* Creamos una cuenta de Azure Cosmos de tres regiones (Oeste de EE. UU., Este de EE. UU., Norte de Europa), donde se puede escribir en todas las regiones 

* Creamos una base de datos (D1) con un rendimiento compartido de 10 000 RU/s 

* Creamos una base de datos (D2) con un rendimiento compartido de 30 000 RU/s  

* Creamos un contenedor (C1) con un rendimiento dedicado de 20 000 RU/s 

[101-200 horas]\:  

* Escalamos verticalmente la base de datos (D1) a 50 000 RU/s 

* Escalamos verticalmente la base de datos (D2) a 70 000 RU/s  

* Eliminamos el contenedor (C1)  

[201-300 horas]\:  

* Creamos el contenedor (C1) nuevamente con un rendimiento dedicado de 20 000 RU/s 

[301-400 horas]\:  

* Quitamos una de las regiones de la cuenta de Azure Cosmos (el número de regiones de escritura ahora es 2) 

* Redujimos verticalmente la base de datos (D1) a 10 000 RU/s 

* Escalamos verticalmente la base de datos (D2) a 80 000 RU/s  

* Eliminamos el contenedor (C1) nuevamente 

[401-500 horas]\:  

* Redujimos verticalmente la base de datos (D2) a 10 000 RU/s  

* Creamos el contenedor (C1) nuevamente con un rendimiento dedicado de 20 000 RU/s 

[501-700 horas]\:  

* Escalamos verticalmente la base de datos (D1) a 20 000 RU/s  

* Escalamos verticalmente la base de datos (D2) a 100 000 RU/s  

* Eliminamos el contenedor (C1) nuevamente  

[701-720 horas]\:  

* Redujimos verticalmente la base de datos (D2) a 50 000 RU/s  

Los cambios en el rendimiento aprovisionado total durante las 720 horas del mes se muestran visualmente en la ilustración siguiente: 

:::image type="content" source="./media/understand-your-bill/bill-example3.png" alt-text="Ejemplo de la vida real":::

La factura mensual total (suponiendo 30 días o 720 horas en un mes) se calculará como sigue:

|**Horas**  |**RU/s** |**Elemento** |**Uso (por hora)** |**Costee** |
|---------|---------|---------|-------|-------|
|[0-100] |D1:10 000 <br/>D2:30 000 <br/>C1:20 000 |Factura de rendimiento para un contenedor en Oeste de EE. UU. (se puede escribir en todas las regiones)  | `D1: 10K RU/sec/100 * $0.016 * 100 hours = $160` <br/>`D2: 30 K RU/sec/100 * $0.016 * 100 hours = $480` <br/>`C1: 20 K RU/sec/100 *$0.016 * 100 hours = $320` |960 USD  |
| | |Factura de rendimiento para 2 regiones adicionales: Este de EE. UU., Norte de Europa (se puede escribir en todas las regiones)  |`(2 + 1) * (60 K RU/sec /100 * $0.016) * 100 hours = $2,880`  |2880 USD  |
|[101-200] |D1:50 000 <br/>D2:70 000 <br/>C1: -- |Factura de rendimiento para un contenedor en Oeste de EE. UU. (se puede escribir en todas las regiones)  |`D1: 50 K RU/sec/100 * $0.016 * 100 hours = $800` <br/>`D2: 70 K RU/sec/100 * $0.016 * 100 hours = $1,120` |1920 USD  |
| | |Factura de rendimiento para 2 regiones adicionales: Este de EE. UU., Norte de Europa (se puede escribir en todas las regiones)  |`(2 + 1) * (120 K RU/sec /100 * $0.016) * 100 hours = $5,760`  |5760 USD  |
|[201-300]  |D1:50 000 <br/>D2:70 000 <br/>C1:20 000 |Factura de rendimiento para un contenedor en Oeste de EE. UU. (se puede escribir en todas las regiones)  |`D1: 50 K RU/sec/100 * $0.016 * 100 hours = $800` <br/>`D2: 70 K RU/sec/100 * $0.016 * 100 hours = $1,120` <br/>`C1: 20 K RU/sec/100 *$0.016 * 100 hours = $320` |USD 2240  |
| | |Factura de rendimiento para 2 regiones adicionales: Este de EE. UU., Norte de Europa (se puede escribir en todas las regiones)  |`(2 + 1) * (140 K RU/sec /100 * $0.016-) * 100 hours = $6,720` |6720 USD |
|[301-400] |D1:10 000 <br/>D2:80 000 <br/>C1: -- |Factura de rendimiento para un contenedor en Oeste de EE. UU. (se puede escribir en todas las regiones)  |`D1: 10K RU/sec/100 * $0.016 * 100 hours = $160` <br/>`D2: 80 K RU/sec/100 * $0.016 * 100 hours = $1,280`  |1440 USD   |
| | |Factura de rendimiento para 2 regiones adicionales: Este de EE. UU., Norte de Europa (se puede escribir en todas las regiones)  |`(1 + 1) * (90 K RU/sec /100 * $0.016) * 100 hours = $2,880`  |2880 USD  |
|[401-500] |D1:10 000 <br>D2:10 000 <br>C1:20 000 |Factura de rendimiento para un contenedor en Oeste de EE. UU. (se puede escribir en todas las regiones)  |`D1: 10K RU/sec/100 * $0.016 * 100 hours = $160` <br>`D2: 10K RU/sec/100 * $0.016 * 100 hours = $160` <br>`C1: 20 K RU/sec/100 *$0.016 * 100 hours = $320` |640 USD  |
| | |Factura de rendimiento para 2 regiones adicionales: Este de EE. UU., Norte de Europa (se puede escribir en todas las regiones)  |`(1 + 1) * (40 K RU/sec /100 * $0.016) * 100 hours = $1,280`  |1280 USD  |
|[501-700] |D1:20 000 <br>D2:100 000 <br>C1: -- |Factura de rendimiento para un contenedor en Oeste de EE. UU. (se puede escribir en todas las regiones)  |`D1: 20 K RU/sec/100 * $0.016 * 200 hours = $640` <br>`D2: 100 K RU/sec/100 * $0.016 * 200 hours = $3,200` |3840 USD  |
| | |Factura de rendimiento para 2 regiones adicionales: Este de EE. UU., Norte de Europa (se puede escribir en todas las regiones)  |`(1 + 1) * (120 K RU/sec /100 * $0.016) * 200 hours = $1,280`  |7680 USD  |
|[701-720] |D1:20 000 <br/>D2:50 000 <br/>C1: -- |Factura de rendimiento para un contenedor en Oeste de EE. UU. (se puede escribir en todas las regiones)  |`D1: 20 K RU/sec/100 *$0.016 * 20 hours = $64` <br/>`D2: 50 K RU/sec/100 *$0.016 * 20 hours = $160` |224 USD  |
| | |Factura de rendimiento para 2 regiones adicionales: Este de EE. UU., Norte de Europa (se puede escribir en todas las regiones)  |`(1 + 1) * (70 K RU/sec /100 * $0.016) * 20 hours = $448`  |224 USD  |
|| |**Costo mensual total**  | |**38 688 USD**   |

## <a name="billing-examples-with-free-tier-accounts"></a>Ejemplos de facturación con cuentas de nivel Gratis
Con el nivel Gratis de Azure Cosmos DB, obtendrá en la cuenta las primeras 400 RU/s y 5 GB de almacenamiento gratis, que se aplicarán en el nivel de cuenta. Las RU/s y el almacenamiento que superen, respectivamente, las 400 RU/s y los 5 GB, se facturarán según las tarifas de precios habituales de la página de precios. En la factura no verá un cargo o una partida por las 400 RU/s y los 5 GB gratis, solo por las RU/s y el almacenamiento que supere lo que cubre el nivel Gratis. Las 400 RU/s se aplican a cualquier tipo de RU/s: rendimiento aprovisionado, escalabilidad automática y arquitectura multimaestro.  

### <a name="billing-example---container-or-database-with-provisioned-throughput"></a>Ejemplo de facturación: contenedor o base de datos con capacidad de proceso aprovisionada
- Supongamos que creamos una base de datos o un contenedor en una cuenta de nivel Gratis con 400 RU/s y 5 GB de almacenamiento.
- La factura no mostrará ningún cargo por este recurso. El costo por hora y mensual será de 0 USD.
- Supongamos ahora que en la misma cuenta agregamos otra base de datos o contenedor con 1000 RU/s y 10 GB de almacenamiento.
- La factura mostrará un cargo por los 1000 RU/s y los 10 GB de almacenamiento. 

### <a name="billing-example---container-with-autoscale-throughput"></a>Ejemplo de facturación: contenedor con rendimiento de escalabilidad automática
- Supongamos que en una cuenta de nivel Gratis creamos un contenedor con escalabilidad automática habilitada, con un máximo de 4000 RU/s. Este recurso se escalará automáticamente entre 400 y 4000 RU/s. 
- Supongamos que, desde la hora 1 a la hora 10, el recurso se sitúa en el mínimo de 400 RU/s. Durante la hora 11, el recurso se escala verticalmente hasta 1000 RU/s y después vuelve a 400 RU/s en la misma hora.
- En las horas de 1 a 10 se le facturará 0 USD por la capacidad de proceso, ya que las 400 RU/s están cubiertas por el nivel Gratis. 
- En la hora 11 se le facturará por una cantidad efectiva de 1000 RU/s - 400 RU/s = 600 RU/s, ya que es el valor máximo de RU/s de la hora. Serán 6 unidades de 100 RU/s por la hora, por lo que el costo de capacidad de proceso total de la hora será de 6 unidades * 0,012 USD = 0,072 USD. 
- Cualquier almacenamiento que supere los primeros 5 GB se facturará a las tarifas de almacenamiento normales. 

### <a name="billing-example---multi-region-single-write-region-account"></a>Ejemplo de facturación: cuenta de varias regiones, con una sola región de escritura
- Supongamos que en una cuenta de nivel Gratis creamos una base de datos o un contenedor con 1200 RU/s y 10 GB de almacenamiento. Replicamos la cuenta en 3 regiones y contamos con una cuenta de maestro único (una sola región de escritura).
- En total, sin el nivel Gratis, se le facturarían 3 * 1200 RU/s = 3600 RU/s y 3 * 10 GB = 30 GB de almacenamiento.
- Con el descuento por nivel Gratis, después de quitar 400 RU/s y 5 GB de almacenamiento, se le facturará por una cantidad efectiva de 3200 RU/s (32 unidades) de capacidad de proceso aprovisionada con la tarifa de región de escritura única y 25 GB de almacenamiento.
- El costo mensual de RU/s sería: 32 unidades * 0,008 USD * 24 horas * 31 días = 190,46 USD. El costo mensual del almacenamiento sería: 25 GB * 0,25/GB = 6,25 USD. El costo total sería de 190,46 + 6,25 = 196,71 USD.
- Nota: Si el precio por unidad de RU/s o de almacenamiento difiere en las regiones, las 400 RU/s y los 5 GB del nivel Gratis reflejarán las tarifas de la región donde se creó la cuenta.

### <a name="billing-example---multi-region-multi-master-multiple-write-region-account"></a>Ejemplo de facturación: cuenta de varias regiones, con arquitectura multimaestro (varias regiones de escritura)

Este ejemplo refleja los [precios de la arquitectura multimaestro](https://azure.microsoft.com/pricing/details/cosmos-db/) para las cuentas creadas después del 1 de diciembre de 2019. 
- Supongamos que en una cuenta de nivel Gratis creamos una base de datos o un contenedor con 1200 RU/s y 10 GB de almacenamiento. Replicamos la cuenta en 3 regiones y contamos con una cuenta de arquitectura multimaestro (varias regiones de escritura). 
- En total, sin el nivel Gratis, se le facturarían 3 * 1200 RU/s = 3600 RU/s y 3 * 10 GB = 30 GB de almacenamiento.
- Con el descuento por nivel Gratis, después de quitar 400 RU/s y 5 GB de almacenamiento, se le facturará por una cantidad efectiva de 3200 RU/s (32 unidades) de capacidad de proceso aprovisionada con la tarifa de varias regiones de escritura y 25 GB de almacenamiento.
- El costo mensual de RU/s sería: 32 unidades * 0,016 USD * 24 horas * 31 días = 380,93 USD. El costo mensual del almacenamiento sería: 25 GB * 0,25/GB = 6,25 USD. El costo total sería de 380,93 + 6,25 = 387,18 USD.

## <a name="proactively-estimating-your-monthly-bill"></a>Cálculo proactivo de la factura mensual  

Veamos otro ejemplo, donde quiere calcular la factura antes del final del mes de forma proactiva. Puede calcular la factura como sigue:

**Costo del almacenamiento**

* Tamaño de registro promedio (KB) = 1 
* Número de registros = 100 000 000 
* Almacenamiento total (GB) = 100 
* Costo mensual por GB = 0,25 $ 
* Costo mensual previsto para almacenamiento = 25,00 $ 

**Costo de rendimiento**

|Tipo de operación| Solicitudes/s| Prom. RU/solicitud| RU necesarias|
|----|----|----|----|
|Escritura| 100 | 5 | 500|
|Lectura| 400| 1| 400|

Total de RU/s: 500 + 400 = 900 Costo por hora: 900/100 * 0,008 USD = 0,072 USD Costo mensual esperado para rendimiento (suponiendo 31 días): 0,072 USD * 24 * 31 = 53,57 USD

**Costo mensual total**

Costo mensual total = Costo mensual de almacenamiento + Costo mensual de rendimiento Costo mensual total = 25,00 USD + 53,57 USD = 78,57 USD

*Los precios pueden variar por región. Para ver los precios actualizados, consulte la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/).*

## <a name="billing-with-azure-cosmos-db-reserved-capacity"></a>Facturación con capacidad reservada de Azure Cosmos DB

La capacidad reservada de Azure Cosmos DB permite comprar rendimiento aprovisionado de antemano (una capacidad reservada o una reserva) que se puede aplicar a todas las bases de datos y contenedores de Azure Cosmos (para cualquier modelo de datos o API) en todas las regiones de Azure. Puesto que el precio del rendimiento aprovisionado varía por región, resulta útil pensar que la capacidad reservada es como un crédito monetario que adquiere con un descuento y que puede usarse para pagar el rendimiento aprovisionado al precio correspondiente en cada región. Por ejemplo, supongamos que tiene una cuenta de Azure Cosmos con un único contenedor aprovisionado con 50 000 RU/s y replicación global en dos regiones: Este de EE. UU. y Japón Oriental. Si elige la opción de pago por uso, pagaría:  

* en Este de EE. UU.: para 50 000 RU/s a la tarifa de 0,008 USD por 100 RU/s en dicha región 

* en Japón Oriental: para 50 000 RU/s a la tarifa de 0,009 USD por 100 RU/s en dicha región

La factura total (sin capacidad reservada) sería (suponiendo 30 días o 720 horas): 

|**Región**| **Precio por hora por 100 RU/s**|**Unidades (RU/s)**|**Importe facturado (por hora)**| **Importe facturado (mensual)**|
|----|----|----|----|----|
|Este de EE. UU.|0,008 USD |50 000|4 USD|2880 USD |
|Japón Oriental|0,009 USD |50 000| 4,50 USD |3240 USD |
|Total|||8,50 USD|6120 USD |

Vamos a suponer que, en su lugar, ha comprado capacidad reservada. Puede comprar capacidad reservada para 100 000 RU/s al precio de 56 064 USD durante un año (con un 20 % de descuento) o 6,40 USD por hora. Consulte los precios de capacidad reservada en la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/).  

* Costo de rendimiento (pago por uso): 100 000 RU/s/100 * 0,008 USD/hora * 8760 horas al año = 70 080 USD 

* Costo de rendimiento (con capacidad reservada) 70 080 USD con un 20 % de descuento = 56 064 USD 

Lo que realmente ha comprado es un crédito de 8 USD por hora, para 100 000 RU/s al precio de venta en Este de EE. UU., al precio de 6,40 USD por hora. Después, puede disponer de esta reserva de rendimiento de prepago por hora para cubrir la capacidad de rendimiento aprovisionado en cualquier región global de Azure a los precios establecidos para su suscripción en cada región. En este ejemplo, donde aprovisiona 50 000 RU/s en el Este de EE. UU. y Japón Oriental, podrá hacer uso de un rendimiento aprovisionado por valor de 8,00 USD por hora, y se le cobrará el uso por encima del límite a 0,50 USD por hora (o 360 USD/mes). 

|**Región**| **Precio por hora por 100 RU/s**|**Unidades (RU/s)**| **Importe facturado (por hora)**| **Importe facturado (mensual)**|
|----|----|----|----|----|
|Este de EE. UU.|0,008 USD |50 000|4 USD|2880 USD |
|Japón Oriental|0,009 USD |50 000| 4,50 USD |3240 USD |
|||Pago por uso|8,50 USD|6120 USD|
|Capacidad reservada adquirida|0,0064 USD (20 % de descuento) |100 RU/s o 8 USD de capacidad comprada previamente |-8 USD|-5760 USD |
|Importe neto|||0,50 USD |360 USD |

## <a name="next-steps"></a>Pasos siguientes

A continuación, puede obtener información sobre la optimización de costos en Azure Cosmos DB con los siguientes artículos:

* Más información sobre la [rentabilidad del modelo de precios de Cosmos DB para los clientes](total-cost-ownership.md)
* Obtenga más información sobre la [optimización para desarrollo y pruebas](optimize-dev-test.md).
* Obtenga más información sobre la [optimización del costo de la capacidad del rendimiento](optimize-cost-throughput.md).
* Obtenga más información sobre la [optimización del costo del almacenamiento](optimize-cost-storage.md).
* Obtenga más información sobre la [optimización del costo de la lectura y la escritura](optimize-cost-reads-writes.md).
* Obtenga más información sobre la [optimización del costo de las consultas](optimize-cost-queries.md).
* Obtenga más información sobre la [optimización del costo de las cuentas de Azure Cosmos de varias regiones](optimize-cost-regions.md).
