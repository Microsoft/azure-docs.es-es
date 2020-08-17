---
title: Conceptos básicos de Apache Spark
description: En este artículo se proporciona una introducción a Apache Spark en Azure Synapse Analytics y los diferentes conceptos.
services: synapse-analytics
author: euangMS
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: spark
ms.date: 04/15/2020
ms.author: euang
ms.reviewer: euang
ms.openlocfilehash: 806f4dff49e9650dba073721109e7d54a18ecbbe
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87052330"
---
# <a name="apache-spark-in-azure-synapse-analytics-core-concepts"></a>Conceptos básicos de Apache Spark en Azure Synapse Analytics

Apache Spark es una plataforma de procesamiento paralelo que admite el procesamiento en memoria para mejorar el rendimiento de aplicaciones de análisis de macrodatos. Apache Spark en Azure Synapse Analytics es una de las implementaciones de Microsoft de Apache Spark en la nube. 

Azure Synapse facilita la creación y configuración de funcionalidades de Spark en Azure. Azure Synapse proporciona una implementación diferente de las funcionalidades de Spark que se documentan aquí.

## <a name="spark-pools-preview"></a>Grupos de Spark (versión preliminar)

Un grupo de Spark (versión preliminar) se crea en Azure Portal. Es la definición de un grupo de Spark que, cuando se crean instancias, se utiliza para crear una instancia de Spark que procesa datos. Cuando se crea un grupo de Spark, solo existe como metadatos; no se consumen, ejecutan ni cobran recursos. Un grupo de Spark tiene una serie de propiedades que controlan las características de una instancia de Spark. Estas características incluyen, entre otras, el nombre, el tamaño, el comportamiento de escalado y el período de vida.

Como no hay ningún costo de recursos asociado a la creación de grupos de Spark, se puede crear cualquier cantidad de ellos con cualquier número de configuraciones diferentes. Los permisos también se pueden aplicar a los grupos de Spark, lo que permite a los usuarios acceder a algunos y a otros no.

Un procedimiento recomendado consiste en crear grupos de Spark más pequeños que se puedan usar para el desarrollo y la depuración y, después, otros más grandes para ejecutar cargas de trabajo de producción.

Puede consultar cómo crear un grupo de Spark y ver todas sus propiedades en [Introducción a los grupos de Spark en Synapse Analytics](../quickstart-create-apache-spark-pool-portal.md).

## <a name="spark-instances"></a>Instancias de Spark

Las instancias de Spark se crean al conectarse a un grupo de Spark, crear una sesión y ejecutar un trabajo. Como varios usuarios pueden acceder a un solo grupo de Spark, se crea una nueva instancia de Spark para cada usuario que se conecta. 

Cuando se envía un segundo trabajo y, si hay capacidad en el grupo, la instancia de Spark existente también tiene capacidad y la instancia existente procesará el trabajo. Si no es así y hay capacidad en el nivel del grupo, se creará una nueva instancia de Spark.

## <a name="examples"></a>Ejemplos

### <a name="example-1"></a>Ejemplo 1

- Va a crear un grupo de Spark llamado SP1. Tiene un tamaño de clúster fijo de 20 nodos.
- También va a enviar un trabajo de Notebook, J1, que usa 10 nodos, y a crear una instancia de Spark, SI1, para procesar el trabajo.
- Ahora envía otro trabajo, J2, que usa 10 nodos porque todavía hay capacidad en el grupo y la instancia, J2, la procesa SI1.
- Si J2 hubiera solicitado 11 nodos, no habría habido capacidad en SP1 ni en SI1. En este caso, si J2 procede de un cuaderno, se rechazará el trabajo. Si J2 procede de un trabajo por lotes, se pondrá en cola.

### <a name="example-2"></a>Ejemplo 2

- Crea una llamada a un grupo de Spark, SP2. Este tiene un escalado automático habilitado de 10 a 20 nodos.
- También va a enviar un trabajo de Notebook, J1, que usa 10 nodos, y a crear una instancia de Spark, SI1, para procesar el trabajo.
- Ahora envía otro trabajo, J2, que usa 10 nodos porque todavía hay capacidad en el grupo y la instancia crece automáticamente hasta los 20 nodos y procesa a J2.

### <a name="example-3"></a>Ejemplo 3

- Va a crear un grupo de Spark llamado SP1. Tiene un tamaño de clúster fijo de 20 nodos.
- También va a enviar un trabajo de Notebook, J1, que usa 10 nodos, y a crear una instancia de Spark, SI1, para procesar el trabajo.
- Otro usuario, U2, envía un trabajo, J3, que usa 10 nodos y una nueva instancia de Spark, SI2, se crea para procesar el trabajo.
- Ahora envía otro trabajo, J2, que usa 10 nodos porque todavía hay capacidad en el grupo y la instancia, J2, la procesa SI1.

## <a name="next-steps"></a>Pasos siguientes

- [Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics)
- [Documentación de Apache Spark](https://spark.apache.org/docs/2.4.4/)
