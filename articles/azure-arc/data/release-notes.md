---
title: 'Servicios de datos habilitados para Azure Arc: notas de la versión'
description: Notas de la versión más recientes
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
ms.date: 09/22/2020
ms.topic: conceptual
ms.openlocfilehash: 5908083be4e6ed389b606754ffef41a4a371c3e3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932150"
---
# <a name="release-notes---azure-arc-enabled-data-services-preview"></a>Notas de la versión: servicios de datos habilitados para Azure Arc (versión preliminar)

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="september-2020"></a>Septiembre de 2020

Los servicios de datos habilitados para Azure Arc se publican como versión preliminar pública. Los servicios de datos habilitados para Arc permiten administrar los servicios de datos en cualquier lugar.

- Instancia administrada de SQL
- Hiperescala de PostgreSQL

Para obtener instrucciones, consulte [¿Qué son los servicios de datos habilitados para Azure Arc?](overview.md)

## <a name="next-steps"></a>Pasos siguientes

> **¿Solo desea realizar algunas pruebas?**  
> Empiece a trabajar rápidamente con el [Inicio rápido de Azure Arc](https://github.com/microsoft/azure_arc#azure-arc-enabled-data-services) en Azure Kubernetes Service (AKS), AWS Elastic Kubernetes Service (EKS), Google Cloud Kubernetes Engine (GKE) o en una máquina virtual de Azure.

[Instalación de las herramientas de cliente](install-client-tools.md)

[Creación del controlador de datos de Azure Arc](create-data-controller.md) (requiere primero la instalación de las herramientas de cliente).

[Creación de una instancia administrada de Azure SQL en Azure Arc](create-sql-managed-instance.md) (requiere primero la creación de un controlador de datos de Azure Arc).

[Creación de un grupo de servidores de Hiperescala de Azure Database for PostgreSQL en Azure Arc](create-postgresql-hyperscale-server-group.md) (requiere primero la creación de un controlador de datos de Azure Arc).

## <a name="known-limitations-and-issues"></a>Problemas y limitaciones conocidos

- Los nombres de instancia de SQL Managed Instance no pueden tener más de 13 caracteres.
- No hay actualización local para el controlador de datos de Azure Arc ni las instancias de base de datos.
- Las imágenes de contenedor de los servicios de datos habilitados para Arc no están firmadas.  Es posible que deba configurar los nodos de Kubernetes para permitir la extracción de imágenes de contenedor sin firmar.  Por ejemplo, si usa Docker como entorno de ejecución del contenedor, puede establecer la variable de entorno DOCKER_CONTENT_TRUST=0 y reiniciar.  Otros entornos de ejecución de contenedores tienen opciones similares, como ocurre con [OpenShift](https://docs.openshift.com/container-platform/4.5/openshift_images/image-configuration.html#images-configuration-file_image-configuration).
- No se pueden crear instancias de SQL Managed Instance ni grupos de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc desde Azure Portal.
- Por ahora, si usa NFS, debe establecer allowRunAsRoot en true en el archivo del perfil de implementación antes de crear el controlador de datos de Azure Arc.
- Solo autenticación de inicio de sesión de SQL y PostgreSQL.  No se admiten Azure Active Directory ni Active Directory.
- La creación de un controlador de datos en OpenShift requiere restricciones de seguridad relajadas.  Para obtener información detallada, consulte la documentación.
- No se admite la _reducción_ del número de nodos de trabajo de Hiperescala de Postgres.
- No se admite la actualización a una versión de Kubernetes más reciente si usa el motor de Azure Kubernetes Service (Motor de AKS) en Azure Stack Hub con el controlador de datos de Azure Arc e instancias de base de datos. Desinstale el controlador de datos de Azure Arc y todas las instancias de base de datos antes de actualizar el clúster de Kubernetes.
- La versión preliminar no admite la copia de seguridad y restauración de la versión 11 del motor Postgres. Solo admite la copia de seguridad y restauración de la versión 12 de Postgres.
