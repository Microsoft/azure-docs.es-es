---
title: Comprobación de la instancia del contenedor de salud
titleSuffix: Azure Cognitive Services
description: Aprenda a comprobar la instancia del contenedor de salud.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 07/07/2020
ms.author: aahi
ms.openlocfilehash: 1a8ce0bc94c61a0cfe6cdad11375763ba954957d
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88122309"
---
### <a name="verify-that-a-container-is-running"></a>Compruebe que haya un contenedor en ejecución

1. Seleccione la pestaña **Información general** y copie la dirección IP.
1. Abra una nueva pestaña del explorador y escriba la dirección IP. Por ejemplo, escriba `http://<IP-address>:5000 (http://55.55.55.55:5000`). Se mostrará la página principal del contenedor, que le permitirá saber que el contenedor se está ejecutando.

    ![Visualización de la página principal del contenedor para comprobar que se está ejecutando](../media/how-tos/container-instance/swagger-docs-on-container.png)

1. Seleccione el vínculo de **Descripción de API de servicio** para ir a la página Swagger del contenedor.

1. Elija cualquiera de las API **POST** y seleccione **Probar**. Se muestran los parámetros, que incluyen la entrada de este ejemplo.

Hay varias direcciones URL que también puede usar para comprobar que el contenedor se está ejecutando.

|Solicitud|Propósito|
|--|--|
|`http://localhost:5000/`|El contenedor ofrece una página principal.|
|`http://localhost:5000/ready`|Solicitado con GET, proporciona una comprobación de que el contenedor está listo para aceptar una consulta para el modelo. Esta solicitud se puede usar con los [sondeos de ejecución y preparación](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) de Kubernetes.|
|`http://localhost:5000/status`|Solicitada con GET, al igual que el punto de conexión /ready, valida que el contenedor se está ejecutando sin incurrir en una consulta para el modelo. Esta solicitud se puede usar con los [sondeos de ejecución y preparación](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) de Kubernetes.|
|`http://localhost:5000/swagger`|Mediante esta dirección URL, el contenedor cuenta con un completo conjunto de documentación sobre los puntos de conexión y con una característica `Try it now`. Esta característica le permite especificar la configuración en un formulario HTML basado en web y realizar la consulta sin necesidad de escribir código. Una vez que la consulta devuelve resultados, se proporciona un ejemplo del comando CURL para mostrar los encabezados HTTP y el formato de cuerpo requeridos. |
|`http://localhost:5000/demo`| Solicitada mediante un explorador, esta característica proporciona una visualización interactiva de los resultados de las consultas de ejemplos de texto de entrada o uno que proporcione.  |

Use esta dirección URL de solicitud (`http://localhost:5000/text/analytics/v3.2-preview.1/entities/health`) para enviar una consulta al contenedor.
