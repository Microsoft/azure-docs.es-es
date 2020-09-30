---
title: Determinación del tipo de aplicación en Hiperescala (Citus) para Azure Database for PostgreSQL
description: Identificación de la aplicación para un modelado de datos distribuido eficaz
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 07/17/2020
ms.openlocfilehash: 92333857177d33307d6997bfcbdf79787d3ab127
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90895961"
---
# <a name="determining-application-type"></a>Determinación del tipo de aplicación

Para ejecutar consultas eficaces en un grupo de servidores de Hiperescala (Citus), es necesario que las tablas se distribuyan correctamente entre los servidores. La distribución recomendada varía según el tipo de aplicación y sus patrones de consulta.

En general, hay dos tipos de aplicaciones que funcionan bien en Hiperescala (Citus). El primer paso para el modelado de datos es identificar qué tipo se parece más a la aplicación.

## <a name="at-a-glance"></a>De un vistazo

| Aplicaciones multiinquilino                                 | Aplicaciones en tiempo real                                |
|-----------------------------------------------------------|-------------------------------------------------------|
| A veces, decenas o centenares de tablas en el esquema          | Pocas tablas                                |
| Consultas relacionadas con un inquilino (empresa o almacén) a la vez | Consultas de análisis relativamente sencillas con agregaciones |
| Cargas de trabajo de OLTP para prestar servicio a clientes web                    | Gran volumen de ingesta de datos principalmente inmutables           |
| Cargas de trabajo de OLAP que atienden consultas de análisis por inquilino   | Enfoque habitual en una gran tabla de eventos            |

## <a name="examples-and-characteristics"></a>Ejemplos y características

**Aplicación multiinquilino**

> Suelen ser aplicaciones SaaS que sirven para otras compañías, cuentas u organizaciones. La mayoría de las aplicaciones SaaS son intrínsecamente relacionales. Tienen una dimensión natural en la que distribuir los datos entre los nodos: solo con particiones por tenant\_id.
>
> Hiperescala (Citus) permite escalar horizontalmente la base de datos a millones de inquilinos sin tener que volver a diseñar la aplicación. Puede mantener la semántica relacional que necesita, como combinaciones, restricciones de clave externa, transacciones, ACID y coherencia.
>
> -   **Ejemplos:** sitios web en los que se almacenan los escaparates de otras empresas, como una solución de marketing digital o una herramienta de automatización de ventas.
> -   **Características**: consultas relacionadas con un solo inquilino en lugar de combinar información entre los inquilinos. Esto incluye las cargas de trabajo de OLTP para prestar servicio a los clientes web y las cargas de trabajo de OLAP que atienden consultas de análisis por inquilino. Tener docenas o centenares de tablas en el esquema de la base de datos también es un indicador del modelo de datos multiinquilino.
>
> El escalado de una aplicación multiinquilino con Hiperescala (Citus) también requiere cambios mínimos en el código de la aplicación. Se ofrece compatibilidad con marcos populares, como Ruby on Rails y Django.

**Análisis en tiempo real**

> Las aplicaciones necesitan paralelismos masivos y coordinan cientos de núcleos para obtener resultados rápidos en consultas numéricas, estadísticas o de recuento.  Mediante el particionamiento y la paralelización de consultas SQL en varios nodos, Hiperescala (Citus) permite realizar consultas en tiempo real en miles de millones de registros en menos de un segundo.
>
> Las tablas de los modelos de datos de análisis en tiempo real normalmente se distribuyen por columnas, como user\_id, host\_id o device\_id.
>
> -   **Ejemplos:** paneles de análisis orientados al cliente que requieren tiempos de respuesta en fracciones de segundo.
> -   **Características**: pocas tablas, que a menudo se centran en una gran tabla de eventos de dispositivo, sitio o usuario, y que, además, requieren un gran volumen de ingesta de datos principalmente inmutables. Consultas de análisis relativamente sencillas (pero de cálculo intensivo) que implican varias agregaciones e instrucciones GROUP BY.

Si su situación se parece a cualquiera de los casos anteriores, el siguiente paso es decidir cómo particionar los datos en el grupo de servidores. La elección de las columnas de distribución del administrador de bases de datos debe coincidir con los patrones de acceso de las consultas típicas para garantizar el rendimiento.

## <a name="next-steps"></a>Pasos siguientes

* [Seleccione una columna de distribución](concepts-hyperscale-choose-distribution-column.md) para las tablas de la aplicación, a fin de distribuir los datos de forma eficaz
