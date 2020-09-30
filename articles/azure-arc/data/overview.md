---
title: ¿Qué son los servicios de datos habilitados para Azure Arc?
description: Presentación de los servicios de datos habilitados para Azure Arc
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
ms.date: 09/22/2020
ms.topic: overview
ms.openlocfilehash: 47fef490c5ece577823a14e3fa4c415f0f613ccb
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945730"
---
# <a name="what-are-azure-arc-enabled-data-services-preview"></a>¿Qué son los servicios de datos habilitados para Azure Arc (versión preliminar)?

Azure Arc permite ejecutar servicios de datos de Azure en el entorno local, en el perímetro y en nubes públicas con Kubernetes y la infraestructura de su elección.

Actualmente, hay disponibles los siguientes servicios de datos habilitados para Azure Arc en versión preliminar:

- Instancia administrada de SQL
- Hiperescala de PostgreSQL

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="always-current"></a>Siempre actual

Los servicios de datos habilitados para Azure Arc, como la instancia administrada de SQL habilitada para Azure Arc y PostgreSQL con la opción Hiperescala habilitado para Azure Arc, reciben actualizaciones con frecuencia, como revisiones de mantenimiento y nuevas características similares a la experiencia de Azure. Usted recibe las actualizaciones de Microsoft Container Registry y debe definir las cadencias de implementación de acuerdo con sus directivas. De este modo, las bases de datos locales pueden mantenerse al día a la vez que mantiene su control. Dado que los servicios de datos habilitados para Azure Arc son de suscripción, ya no deberá se producirá ninguna otra finalización de soporte técnico para las bases de datos.

## <a name="elastic-scale"></a>Escalado elástico

La elasticidad similar a la nube local le permite escalar y reducir verticalmente las bases de datos de forma dinámica al igual que en Azure, en función de la capacidad disponible de la infraestructura. Esta funcionalidad puede ser útil en los escenarios flexibles con necesidades volátiles, incluidos aquellos escenarios que requieren la ingesta y consulta de datos en tiempo real, a cualquier escala, con un tiempo de respuesta inferior a un segundo. Además, también puede escalar horizontalmente las instancias de base de datos mediante la opción de implementación única de hiperescala de Azure Database for PostgreSQL con la opción Hiperescala. Esta funcionalidad proporciona a las cargas de trabajo de datos un aumento adicional en la optimización de la capacidad, mediante el uso de lecturas y escrituras de escalabilidad *horizontal* únicas.

## <a name="self-service-provisioning"></a>Aprovisionamiento de autoservicio

Azure Arc también proporciona otras ventajas en la nube, como la implementación y automatización rápidas a gran escala. Gracias a la orquestación basada en Kubernetes, puede implementar una base de datos en cuestión segundos mediante las herramientas GUI o CLI.

## <a name="unified-management"></a>Administración unificada

Gracias a las herramientas conocidas como Azure Portal, Azure Data Studio y la CLI de datos de Azure, ahora puede obtener una vista unificada de todos los recursos de datos implementados con Azure Arc. No solo puede ver y administrar una variedad de bases de datos relacionales en todo el entorno y Azure, sino que también puede obtener registros y telemetría de las API de Kubernetes para analizar la capacidad y el estado de la infraestructura subyacente. Además de contar con la supervisión del rendimiento y el análisis de registros localizados, ahora puede aprovechar Azure Monitor para obtener información operativa completa en todo el estado.

## <a name="disconnected-scenario-support"></a>Compatibilidad con los escenarios sin conexión

Muchos de los servicios, como el aprovisionamiento de autoservicio, las copias de seguridad automatizadas y la restauración y la supervisión se pueden ejecutar localmente en la infraestructura con una conexión directa con Azure o sin ninguna. La conexión directa a Azure abre opciones adicionales para la integración con otros servicios de Azure, como Azure Monitor y la capacidad de usar las API de Azure Portal y Azure Resource Manager desde cualquier lugar del mundo, a fin de administrar los servicios de datos habilitados para Azure Arc.

## <a name="next-steps"></a>Pasos siguientes

> **¿Solo desea realizar algunas pruebas?**  
> Empiece a trabajar rápidamente con el [Inicio rápido de Azure Arc](https://github.com/microsoft/azure_arc#azure-arc-enabled-data-services) en Azure Kubernetes Service (AKS), AWS Elastic Kubernetes Service (EKS), Google Cloud Kubernetes Engine (GKE) o en una VM de Azure.

[Instalación de las herramientas de cliente](install-client-tools.md)

[Creación del controlador de datos de Azure Arc](create-data-controller.md) (requiere primero la instalación de las herramientas de cliente).

[Creación de una instancia administrada de Azure SQL en Azure Arc](create-sql-managed-instance.md) (requiere primero la creación de un controlador de datos de Azure Arc).

[Creación de un grupo de servidores de Hiperescala de Azure Database for PostgreSQL en Azure Arc](create-postgresql-hyperscale-server-group.md) (requiere primero la creación de un controlador de datos de Azure Arc).
