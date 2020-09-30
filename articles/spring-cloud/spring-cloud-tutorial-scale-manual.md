---
title: 'Tutorial: Escalado de una aplicación en Azure Spring Cloud | Microsoft Docs'
description: Aprenda a escalar una aplicación con Azure Spring Cloud en Azure Portal
ms.service: spring-cloud
ms.topic: tutorial
ms.author: brendm
author: bmitchell287
ms.date: 10/06/2019
ms.custom: devx-track-java
ms.openlocfilehash: 33d25edafa9249300406365699930ded1c8ec75b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906844"
---
# <a name="tutorial-scale-an-application-in-azure-spring-cloud"></a>Tutorial: Escalado de una aplicación en Azure Spring Cloud

**Este artículo se aplica a:** ✔️ Java ✔️ C#

En este tutorial se muestra cómo escalar cualquier aplicación de microservicios mediante el panel de Azure Spring Cloud de Azure Portal.

Escale y reduzca verticalmente la aplicación mediante la modificación de su número de CPU virtuales (vCPU) y la cantidad de memoria. Escale y reduzca horizontalmente la aplicación mediante la modificación de su número de instancias.

Cuando termine, sabrá cómo realizar cambios manuales rápidos en cada aplicación del servicio. El escalado surte efecto en segundos y no requiere cambios en el código o nuevas implementaciones.

## <a name="prerequisites"></a>Requisitos previos

Para seguir estos procedimientos, necesita:

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar. 
* Una instancia del servicio Azure Spring Cloud implementada.  Como comienzo, siga el [inicio rápido para implementar una aplicación mediante la CLI de Azure](spring-cloud-quickstart.md).
* Al menos una aplicación ya creada en su instancia de servicio.

## <a name="navigate-to-the-scale-page-in-the-azure-portal"></a>Vaya a la página Escalar de Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Vaya a la página **Información general** de Azure Spring Cloud.

1. Seleccione el grupo de recursos que contiene el servicio.

1. En el menú del lado izquierdo de la página, en **Settings** (Configuración), seleccione la pestaña **Apps** (Aplicaciones).

1. Seleccione la aplicación que desea escalar. En este ejemplo, seleccione la aplicación llamada **account-service**. A continuación, verá la página **Overview** (Información general) de la aplicación.

1. En el menú del lado izquierdo de la página, en **Settings** (Configuración), seleccione la pestaña **Scale** (Escalar). Verá opciones para escalar los atributos que se muestran en la sección siguiente.

## <a name="scale-your-application"></a>Escalado de la aplicación

Si modifica los atributos de escalado, tenga en cuenta lo siguiente:

* **CPU**: el número máximo de CPU por instancia de aplicación es cuatro. El número total de CPU para una aplicación es el valor establecido aquí multiplicado por el número de instancias de la aplicación.

* **Memoria/GB**: la cantidad máxima de memoria permitida por instancia de aplicación es 8 GB. La cantidad total de memoria para una aplicación es el valor establecido aquí multiplicado por el número de instancias de la aplicación.

* **Recuento de instancias de aplicaciones**: en el nivel estándar, puede escalar horizontalmente hasta 20 instancias como máximo. Este valor cambia el número de instancias independientes en ejecución de la aplicación de microservicios.

Asegúrese de seleccionar **Save** (Guardar) para aplicar la configuración de escalado.

![El servicio Scale en Azure Portal](media/spring-cloud-tutorial-scale-manual/scale-up-out.png)

Tras unos segundos, se muestran los cambios realizados en la página **Overview** (Información general), y puede encontrar más detalles en la pestaña **Application instances** (Instancias de la aplicación). El escalado no requiere cambios en el código ni nuevas implementaciones.

## <a name="upgrade-to-the-standard-tier"></a>Actualización al nivel Estándar
Si está en el nivel Básico y se le imponen una o varias de estas [limitaciones](spring-cloud-quotas.md), puede actualizar al nivel Estándar. Para hacerlo, seleccione primero la columna de nivel Estándar y haga clic en el botón **Actualizar** para ir al menú Plan de tarifa.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a escalar manualmente una aplicación de Azure Spring Cloud. Para información sobre cómo supervisar una aplicación mediante la configuración de alertas, continúe con el siguiente tutorial.

> [!div class="nextstepaction"]
> [Aprenda a configurar las alertas](spring-cloud-tutorial-alerts-action-groups.md)
