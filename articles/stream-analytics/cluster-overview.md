---
title: Introducción a los clústeres de Azure Stream Analytics (versión preliminar)
description: Obtenga información sobre la oferta dedicada de un solo inquilino del clúster de Stream Analytics.
author: sidramadoss
ms.author: sidram
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: overview
ms.custom: mvc
ms.date: 09/22/2020
ms.openlocfilehash: 613cf7d9b68fe42c26f2c01cb1fb5dd1da1e1fb5
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945450"
---
# <a name="overview-of-azure-stream-analytics-cluster-preview"></a>Introducción al clúster de Azure Stream Analytics (versión preliminar)

Un clúster de Azure Stream Analytics ofrece una implementación de un solo inquilino que se puede usar para escenarios de streaming complejos y exigentes. A escala completa, los clústeres de Stream Analytics pueden procesar más de 200 MB/segundo en tiempo real. Los trabajos de Stream Analytics que se ejecutan en clústeres dedicados pueden aprovechar todas las características de la oferta estándar e incluyen compatibilidad con la conectividad de Private Link con las entradas y salidas.

Los clústeres de Stream Analytics se facturan por unidades de streaming (SU), que representan la cantidad de recursos de CPU y memoria asignados al clúster. Una unidad de streaming es igual tanto en las ofertas estándar como las dedicadas. Puede comprar 36, 72, 108, 144, 180 o 216 unidades de streaming para cada clúster. Un clúster de Stream Analytics puede proporcionar servicios como plataforma de streaming para su organización y se puede compartir entre distintos equipos que trabajan en varios casos de uso.

## <a name="what-are-stream-analytics-clusters"></a>Qué son los clústeres de Stream Analytics

Los clústeres de Stream Analytics cuentan con la tecnología del mismo motor que se usa en los trabajos de Stream Analytics que se ejecutan en un entorno de varios inquilinos. El clúster dedicado de un solo inquilino tiene las siguientes características:

* Hospedaje de un solo inquilino sin ruido de otros inquilinos. Los recursos están realmente "aislados" y se ejecutan mejor cuando hay ráfagas de tráfico.

* Escale su clúster entre 36 y 216 unidades de streaming a medida que el uso de streaming aumenta con el tiempo.

* Compatibilidad con la red virtual que permite a los trabajos de Stream Analytics conectarse a otros recursos de forma segura mediante puntos de conexión privados.

* Capacidad para crear funciones definidas por el usuario de C# y deserializadores personalizados en cualquier región.

* No tiene ningún costo de mantenimiento, lo que le permite centrar sus esfuerzos en la creación de soluciones de análisis en tiempo real.

## <a name="how-to-get-started"></a>Primeros pasos

Puede [crear un clúster de Stream Analytics](create-cluster.md) a través de [Azure Portal](https://aka.ms/asaclustercreateportal). Si tiene alguna pregunta o necesita ayuda con la incorporación, puede ponerse en contacto con el [equipo de Stream Analytics](mailto:askasa@microsoft.com).

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="how-do-i-choose-between-a-stream-analytics-cluster-and-a-stream-analytics-job"></a>¿Cómo puedo elegir entre un clúster y un trabajo de Stream Analytics?

La manera más sencilla de empezar es crear y desarrollar un trabajo de Stream Analytics a fin de familiarizarse con el servicio y ver cómo puede cumplir los requisitos de análisis.

Los trabajos de Stream Analytics por sí solos no admiten redes virtuales. Si las entradas o salidas están protegidas detrás de un firewall o de una instancia de Azure Virtual Network, tiene las dos opciones siguientes:

* Si la máquina local tiene acceso a los recursos de entrada y salida protegidos por una red virtual (por ejemplo, Azure Event Hubs o Azure SQL Database), puede [instalar herramientas de Azure Stream Analytics para Visual Studio](stream-analytics-tools-for-visual-studio-install.md) en la máquina local. Puede desarrollar y [probar trabajos de Stream Analytics localmente](stream-analytics-live-data-local-testing.md) en el dispositivo de forma gratuita. Una vez que esté listo para usar Stream Analytics en la arquitectura, puede crear un clúster de Stream Analytics, configurar puntos de conexión privados y ejecutar los trabajos a gran escala.

* Puede crear un clúster de Stream Analytics, configurar el clúster con los puntos de conexión privados necesarios para la canalización y ejecutar los trabajos de Stream Analytics en el clúster.

### <a name="what-performance-can-i-expect"></a>¿Qué rendimiento puedo esperar?

Una unidad de streaming es igual tanto en las ofertas estándar como las dedicadas. Un solo trabajo que use un clúster completo de 36 unidades de streaming puede alcanzar un rendimiento aproximado de 36 MB/segundo con una latencia de milisegundos. El número exacto depende del formato de los eventos y del tipo de análisis. Dado que el clúster de Stream Analytics es dedicado, ofrece garantías de rendimiento más confiables. Usted es el único propietario de todos los trabajos que se ejecutan en el clúster.

### <a name="can-i-scale-my-cluster"></a>¿Puedo escalar mi clúster?

Sí. Puede configurar fácilmente la capacidad de su clúster, lo que le permitirá realizar [un escalado o una reducción vertical](scale-cluster.md) según sea necesario a fin de satisfacer la demanda cambiante.

### <a name="can-i-run-my-existing-jobs-on-these-new-clusters-ive-created"></a>¿Puedo ejecutar mis trabajos existentes en estos nuevos clústeres que he creado?

Sí. Puede vincular los trabajos existentes al clúster de Stream Analytics recién creado y ejecutarlo como lo hace habitualmente. No tiene que volver a crear los trabajos de Stream Analytics existentes desde cero.

### <a name="how-much-will-these-clusters-cost-me"></a>¿Cuánto me costarán estos clústeres?

Los clústeres de Stream Analytics se cobran en función de la capacidad elegida de unidades de streaming . Los clústeres se facturan por horas y no se aplican cargos adicionales por los trabajos que se ejecuten en estos clústeres. Consulte la [página de precios del servicio Private Link](https://azure.microsoft.com/pricing/details/private-link/) para obtener actualizaciones de facturación del punto de conexión privado.

### <a name="which-inputs-and-outputs-can-i-privately-connect-to-from-my-stream-analytics-cluster"></a>¿A qué entradas y salidas puedo conectarme de forma privada desde mi clúster de Stream Analytics?

Stream Analytics admite varios tipos de entradas y salidas. Cualquiera de estos servicios que admitan instancias de Azure Private Link pueden conectarse a sus trabajos de forma privada. Puede [crear puntos de conexión privados](private-endpoints.md) en el clúster que permitan a los trabajos tener acceso a los recursos de entrada y salida.

## <a name="next-steps"></a>Pasos siguientes

Ya tiene información general acerca del clúster de Azure Stream Analytics. A continuación, puede crear el clúster y ejecutar el trabajo de Stream Analytics: 

* [Creación de un clúster de Stream Analytics](create-cluster.md)
* [Administración de puntos de conexión privados en un clúster de Azure Stream Analytics](private-endpoints.md)
* [Administración de trabajos de Stream Analytics en un clúster de Stream Analytics](manage-jobs-cluster.md)
* [Creación de un trabajo de Stream Analytics](stream-analytics-quick-create-portal.md)
