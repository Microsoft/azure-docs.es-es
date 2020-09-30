---
title: Descripción del estado de las aplicaciones en Azure Spring Cloud
description: Más información sobre las categorías de estados de las aplicaciones en Azure Spring Cloud
author: MikeDodaro
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 04/10/2020
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: ec87c5523d6f608363ca615037f20365705609df
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90892854"
---
# <a name="understanding-app-status-in-azure-spring-cloud"></a>Descripción del estado de las aplicaciones en Azure Spring Cloud

**Este artículo se aplica a:** ✔️ Java ✔️ C#

La interfaz de usuario de Azure Spring Cloud ofrece información sobre el estado de las aplicaciones en ejecución.  Hay una opción **Aplicaciones** para cada grupo de recursos de una suscripción que muestra el estado general de los tipos de aplicaciones.  Para cada tipo de aplicación, se muestran las **instancias de aplicación**.

## <a name="apps-status"></a>Estado de las aplicaciones
Para ver el estado general de un tipo de aplicación, seleccione **Aplicaciones** en el panel de navegación izquierdo de un grupo de recursos. El resultado muestra el estado de la aplicación implementada:

* **Estado de aprovisionamiento** muestra el estado de aprovisionamiento de la implementación
* **Instancia en ejecución** muestra el número de instancias de aplicación que se están ejecutando y el número de instancias de aplicación que se desea. Si se ha detenido la aplicación, esta columna muestra *detenida*.
* **Instancia registrada** muestra el número de instancias de aplicación que se han registrado en Eureka y el número de instancias de aplicación que se desea. Si se ha detenido la aplicación, esta columna muestra *detenida*.


 ![Estado de las aplicaciones](media/spring-cloud-concept-app-status/apps-ui-status.png)

**El estado de la implementación se notifica mediante uno de los siguientes valores:**

| Enum | Definición |
|:--:|:----------------:|
| En ejecución | La implementación ESTÁ en ejecución. |
| Detenido | La implementación ESTÁ detenida. |

**El estado de aprovisionamiento solo es accesible desde la CLI.  Se notifica mediante uno de los siguientes valores:**

| Enum | Definición |
|:--:|:----------------:|
| Creating | El recurso se está creando. |
| Actualizando | El recurso se está actualizando. |
| Correcto | Se han proporcionado correctamente los recursos e implementado el binario. |
| Con error | No se pudo lograr el objetivo *Correcto*. |
| Eliminando | Se está eliminando el recurso. Esto impide la operación y el recurso no está disponible en este estado. |

## <a name="app-instances-status"></a>Estado de las instancias de aplicación

Para ver el estado de una instancia específica de una aplicación implementada, haga clic en el **nombre** de la aplicación en la interfaz de usuario de **Aplicaciones**. Aparecerán los resultados:
* **Estado**: Si la instancia se está ejecutando o cuál es su estado
* **DiscoveryStatus**: El estado registrado de la instancia de aplicación en el servidor de Eureka

 ![Estado de las instancias de aplicación](media/spring-cloud-concept-app-status/apps-ui-instance-status.png)

**El estado de la instancia se notifica mediante uno de los siguientes valores:**

| Enum | Definición |
|:--:|:----------------:|
| Iniciando | El binario se ha implementado correctamente en la instancia especificada. No se puede realizar el arranque del archivo jar de la instancia porque el archivo jar no se ejecuta correctamente. |
| En ejecución | La instancia funciona. |
| Con error | La instancia de aplicación no pudo iniciar el archivo binario del usuario después de varios reintentos. |
| Terminando | La instancia de aplicación se está cerrando. |

**El estado de detección de la instancia se notifica mediante uno de los siguientes valores:**

| Enum | Definición |
|:--:|:----------------:|
| UP | La instancia de aplicación está registrada en Eureka y está lista para recibir tráfico. |
| OUT_OF_SERVICE | La instancia de aplicación está registrada en Eureka y puede recibir tráfico, pero se ha cerrado al tráfico de forma intencionada. |
| ABAJO | La instancia de aplicación no está registrada en Eureka o está registrada pero no puede recibir tráfico. |


## <a name="see-also"></a>Consulte también
* [Preparación de una aplicación de Spring para su implementación en Azure Spring Cloud](spring-cloud-tutorial-prepare-app-deployment.md)