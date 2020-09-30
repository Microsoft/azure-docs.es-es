---
title: Preguntas frecuentes sobre los contenedores de Cognitive Services
titleSuffix: Azure Cognitive Services
description: Preguntas frecuentes y respuestas.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: aahi
ms.openlocfilehash: 3d35a1f6913d0b657956489d0e57836a05f9eb1d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90900057"
---
# <a name="azure-cognitive-services-containers-frequently-asked-questions-faq"></a>Preguntas frecuentes acerca de los contenedores de Azure Cognitive Services

## <a name="general-questions"></a>Preguntas generales

**P: ¿Qué hay disponible?**

**R:** Los contenedores de Azure Cognitive Services permiten a los desarrolladores usar las mismas API inteligentes que están disponibles en Azure, pero con las [ventajas](../cognitive-services-container-support.md#features-and-benefits) que aportan los contenedores. Algunos contenedores están disponibles como versión preliminar validada, lo que puede exigir una aplicación para acceder. Otros contenedores están disponibles públicamente como versión preliminar no validada o están disponibles con carácter general. Puede encontrar una lista completa de contenedores y su disponibilidad en el artículo [Compatibilidad con contenedores en Azure Cognitive Services](../cognitive-services-container-support.md#container-availability-in-azure-cognitive-services). También puede ver los contenedores en [Docker Hub](https://hub.docker.com/_/microsoft-azure-cognitive-services).

**P: ¿Hay alguna diferencia entre la nube de Cognitive Services y los contenedores?**

**R:** Los contenedores de Cognitive Services son una alternativa a la nube de Cognitive Services. Los contenedores ofrecen las mismas funcionalidades que los servicios en la nube correspondientes. Los clientes pueden implementar los contenedores en un entorno local o en Azure. La tecnología básica de inteligencia artificial, los planes de tarifa, las claves de API y la firma de API son los mismos entre los servicios locales y sus homólogos en la nube. Las siguientes son las [características y ventajas](../cognitive-services-container-support.md#features-and-benefits) de elegir contenedores en lugar de sus equivalentes de servicio en la nube.

**P: ¿Cómo accedo y uso un contenedor en versión preliminar validada?**

**R:** Anteriormente, los contenedores en versión preliminar validada se hospedaban en el repositorio de `containerpreview.azurecr.io`. A partir del 22 de septiembre de 2020, estos contenedores se hospedan en Microsoft Container Registry y su descarga no requiere el uso del comando docker login. Podrá ejecutar un contenedor en versión preliminar validada si el recurso de Azure se creó con el identificador de la suscripción de Azure aprobada. No podrá ejecutar el contenedor si la suscripción de Azure no se ha aprobado después de completar el [formulario de solicitud](https://aka.ms/csgate).


**P: ¿Los contenedores estarán disponibles para todos los servicios de Cognitive Services y cuál es el siguiente conjunto de contenedores que podría estar disponible?**

**R:** Nos gustaría ofrecer más servicios de Cognitive Services como ofertas de contenedores. Póngase en contacto con el administrador de la cuenta de Microsoft local para obtener actualizaciones de nuevas versiones de contenedor y otros anuncios de Cognitive Services.

**P: ¿Cuál será el Acuerdo de Nivel de Servicio (SLA) para contenedores de Cognitive Services?**

**R:** Los contenedores de Cognitive Services no tienen un Acuerdo de Nivel de Servicio.

Los clientes controlan las configuraciones de contenedores de Cognitive Services para los recursos, por lo que Microsoft no ofrecerá un Acuerdo de Nivel de Servicio para disponibilidad general (GA). Los clientes pueden implementar contenedores en el entorno local y, de este modo, definen los entornos de host.

> [!IMPORTANT]
> Para más información sobre los Acuerdos de Nivel de Servicio de Cognitive Services, [visite nuestra página de SLA](https://azure.microsoft.com/support/legal/sla/cognitive-services/v1_1/).

**P: ¿Estos contenedores están disponibles en nubes soberanas?**

**R:** No todo el mundo está familiarizado con el término "nube soberana", por tanto, empecemos por definirlo:

> La "nube soberana" consta de las nubes [Azure Government](../../azure-government/documentation-government-welcome.md), [Azure Alemania](../../germany/germany-welcome.md) y [Azure China 21Vianet](https://docs.microsoft.com/azure/china/overview-operations).

Desafortunadamente, los contenedores de Cognitive Services *no* se admiten de forma nativa en las nubes soberanas. Los contenedores se pueden ejecutar en estas nubes, pero se extraerán de la nube pública y tendrán que enviar datos de uso al punto de conexión público.

### <a name="versioning"></a>Control de versiones

**P: ¿Cómo se actualizan los contenedores a la versión más reciente?**

**R:** Los clientes pueden elegir cuándo actualizar los contenedores que han implementado. Los contenedores se marcarán con las [etiquetas de Docker](https://docs.docker.com/engine/reference/commandline/tag/) estándar, como `latest` para indicar la versión más reciente. Animamos a los clientes a que extraigan la versión más reciente de los contenedores a medida que se publiquen y comprueben los [webhooks de Azure Container Registry](../../container-registry/container-registry-webhook.md) para más información sobre cómo recibir notificaciones cuando se actualiza una imagen.
 
**P: ¿Qué versiones se admitirán?**

**R:** Se admitirá la versión actual y la versión principal más reciente del contenedor. Sin embargo, animamos a los clientes a mantenerse al día para conseguir la tecnología más moderna.
 
**P: ¿Cómo se controlan las actualizaciones de versiones?**

**R:** Los cambios de versión principales indican que hay un cambio importante en la firma de la API. Se prevé que esto normalmente coincida con cambios de la versión principal en la oferta de nube de Cognitive Services correspondiente. Los cambios de versión secundaria indican correcciones de errores, actualizaciones de modelos o nuevas características que no suponen un cambio importante en la firma de la API.

## <a name="technical-questions"></a>Preguntas técnicas

**P: ¿Cómo debo ejecutar los contenedores de Cognitive Services en dispositivos IoT?**

**R:** Tanto si no dispone de una conexión a Internet confiable o desea ahorrar en el ancho de banda, como si tiene requisitos de latencia baja o está tratando con datos confidenciales que se deban analizar en el sitio, [Azure IoT Edge con los contenedores Cognitive Services](https://azure.microsoft.com/blog/running-cognitive-services-on-iot-edge/) le proporciona coherencia con la nube.

**P: ¿Son compatibles estos contenedores con OpenShift?** 

No se prueban los contenedores con OpenShift, pero, por lo general, los contenedores de Cognitive Services se ejecutarán en cualquier plataforma que admita imágenes de Docker. Si usa OpenShift, se recomienda ejecutar los contenedores como `root-user`.

**P: ¿Cómo proporcionar comentarios sobre el producto y recomendaciones de características?**

**R:** Se anima a los clientes a que [expongan sus preocupaciones](https://cognitive.uservoice.com/) públicamente y a que voten a otros usuarios que hayan hecho lo mismo siempre que los posibles problemas se solapen. La herramienta de voz de usuario se puede usar tanto para los comentarios de los productos como para las recomendaciones de características.

**P: ¿Qué errores y mensajes de estado devuelven los contenedores de Cognitive Services?**

**R:** Consulte la tabla siguiente para ver una lista de errores y mensajes de estado.

|Situación  | Descripción  |
|---------|---------|
| `Valid` | La clave de API es válida, no se necesita ninguna acción. |
| `Invalid` |   La clave de API no es válida. Debe proporcionar una clave de API válida para ejecutar el contenedor. Busque la clave de API y la región del servicio en la sección **Claves y punto de conexión** para el recurso de Azure Cognitive Services en Azure Portal. |
| `Mismatch` | Proporcionó una clave de API o un punto de conexión para un tipo de recurso de Cognitive Services distinto. Busque la clave de API y la región del servicio en la sección **Claves y punto de conexión** para el recurso de Azure Cognitive Services. |
| `CouldNotConnect` | El contenedor no se pudo conectar al punto de conexión de facturación. Revise el valor `Retry-After` y espere que este período finalice antes de realizar solicitudes adicionales. |
| `OutOfQuota` | La clave de API superó la cuota. Puede actualizar el plan de tarifa o esperar a que haya una cuota adicional disponible. Busque el nivel en la sección **Plan de tarifa** del recurso de Azure Cognitive Services en Azure Portal. |
| `BillingEndpointBusy` | Actualmente, el punto de conexión de facturación está ocupado. Revise el valor `Retry-After` y espere que este período finalice antes de realizar solicitudes adicionales. |
| `ContainerUseUnauthorized` | La clave de API proporcionada no está autorizada para su uso con este contenedor. Lo más probable es que use un contenedor validado, por lo que debe enviar una [solicitud en línea](https://aka.ms/csgate) para asegurarse de que el id. de la suscripción de Azure esté aprobada. |
| `Unknown` | Actualmente, el servidor no puede procesar las solicitudes de facturación. |


**P: ¿Con quién debo ponerme en contacto para obtener soporte técnico?**

**R:** Los canales de asistencia al cliente son los mismos que en la oferta de la nube de Cognitive Services. Todos los contenedores de Cognitive Services incluyen características de registro que nos ayudarán a nosotros y a la comunidad a proporcionar soporte técnico a los clientes. Para obtener soporte técnico adicional, consulte las siguientes opciones.

### <a name="customer-support-plan"></a>Plan de soporte técnico para clientes

Los clientes deben consultar el [plan de soporte técnico de Azure](https://azure.microsoft.com/support/plans/) para ver con quién deben ponerse en contacto si desean obtener soporte técnico.

### <a name="azure-knowledge-center"></a>Centro de conocimiento de Azure

Los clientes pueden explorar el [centro de conocimiento de Azure](https://azure.microsoft.com/resources/knowledge-center/) para responder a preguntas y problemas de soporte técnico.

### <a name="stack-overflow"></a>Stack Overflow

> [Stack Overflow](https://en.wikipedia.org/wiki/Stack_Overflow) es un sitio de preguntas y respuestas para programadores profesionales y aficionados.

Explore las siguientes etiquetas para ver posibles preguntas y respuestas que concuerden con sus necesidades.

* [Azure Cognitive Services](https://stackoverflow.com/questions/tagged/azure-cognitive-services)
* [Microsoft Cognitive](https://stackoverflow.com/questions/tagged/microsoft-cognitive)

**P: ¿Cómo funciona la facturación?**

**R:** Se cobra a los clientes en función del consumo, igual que en la nube de Cognitive Services. Los contenedores deben configurarse para enviar datos de medición a Azure y las transacciones se facturarán según corresponda. Los recursos usados en los servicios hospedados y locales se agregarán a una única cuota con precios según niveles de servicio, teniendo en cuenta ambos usos. Para más información, consulte la página de precios de la oferta correspondiente.

* [Anomaly Detector][ad-containers-billing]
* [Computer Vision][cv-containers-billing]
* [Face][fa-containers-billing]
* [Form Recognizer][fr-containers-billing]
* [Language Understanding (LUIS)][lu-containers-billing]
* [Speech Service API][sp-containers-billing]
* [Text Analytics][ta-containers-billing]

> [!IMPORTANT]
> Los contenedores de Cognitive Services no tienen licencia para ejecutarse sin estar conectados a Azure para realizar mediciones. Los clientes tienen que habilitar los contenedores para comunicar la información de facturación con el servicio de medición en todo momento. Los contenedores de Cognitive Services no envían los datos del cliente a Microsoft.
 
**P: ¿Cuál es la garantía de soporte técnico actual para los contenedores?**

**R:** No hay ninguna garantía para las versiones preliminares. La garantía estándar de Microsoft para el software empresarial se aplica cuando los contenedores se anuncian formalmente como de disponibilidad general (GA).
 
**P: ¿Qué ocurre con los contenedores de Cognitive Services cuando se pierde la conectividad a Internet?**

**R:** Los contenedores de Cognitive Services *no tienen licencia* para ejecutarse sin estar conectados a Azure para realizar mediciones. Los clientes tienen que habilitar los contenedores para comunicarse con el servicio de medición en todo momento.

**P: ¿Cuánto tiempo puede funcionar el contenedor sin estar conectado a Azure?**

**R:** Los contenedores de Cognitive Services *no tienen licencia* para ejecutarse sin estar conectados a Azure para realizar mediciones. Los clientes tienen que habilitar los contenedores para comunicarse con el servicio de medición en todo momento.
 
**P: ¿Cuál es el hardware actual necesario para ejecutar estos contenedores?**

**R:** Los contenedores de Cognitive Services se basan en contenedores x64 que pueden ejecutar cualquier nodo, máquina virtual y dispositivo perimetral compatible con Linux que admita los contenedores de Docker de Linux x64. Todos ellos requieren procesadores de CPU. A continuación puede ver la configuración mínima y la recomendada de cada oferta de contenedor:

* [Anomaly Detector][ad-containers-recommendations]
* [Computer Vision][cv-containers-recommendations]
* [Face][fa-containers-recommendations]
* [Form Recognizer][fr-containers-recommendations]
* [Language Understanding (LUIS)][lu-containers-recommendations]
* [Speech Service API][sp-containers-recommendations]
* [Text Analytics][ta-containers-recommendations]
 
**P: ¿Se admiten actualmente estos contenedores en Windows?**

**R:** Los contenedores de Cognitive Services son también de Linux; sin embargo, hay cierta compatibilidad con contenedores de Linux en Windows. Para más información acerca de los contenedores de Linux en Windows, consulte la [documentación de Docker](https://blog.docker.com/2017/09/preview-linux-containers-on-windows/).
 
**P: ¿Cómo se descubren los contenedores?**

**R:** Los contenedores de Cognitive Services están disponibles en varias ubicaciones, como Azure Portal, el centro de Docker y los registros de contenedor de Azure. Para conocer las ubicaciones de contenedor más recientes, consulte los [repositorios e imágenes de contenedores](../cognitive-services-container-support.md#container-repositories-and-images).

**P: ¿Cómo se comparan los contenedores de Cognitive Services con las ofertas de AWS y Google?**

**R:** Microsoft es el primer proveedor de servicios en la nube que traslada sus modelos de inteligencia artificial con entrenamiento previo a los contenedores con una facturación sencilla por transacción, como si los clientes estuvieran usando un servicio en la nube. Microsoft considera que una nube híbrida ofrece más opciones a los clientes.

**P: ¿Qué certificaciones de cumplimiento tienen los contenedores?**

**R:** Los contenedores de Cognitive Services no tienen certificaciones de cumplimiento

**P: ¿En qué regiones están disponibles los contenedores de Cognitive Services?**

**R:** Los contenedores se pueden ejecutar en cualquier parte de cualquier región, pero necesitan una clave y recurrir de nuevo a Azure para las mediciones. Todas las regiones admitidas para el servicio en la nube se admiten para la llamada de medición de contenedores.

[!INCLUDE [Containers next steps](includes/containers-next-steps.md)]

[ad-containers]: ../anomaly-Detector/anomaly-detector-container-howto.md
[cv-containers]: ../computer-vision/computer-vision-how-to-install-containers.md
[fa-containers]: ../face/face-how-to-install-containers.md
[fr-containers]: ../form-recognizer/form-recognizer-container-howto.md
[lu-containers]: ../luis/luis-container-howto.md
[sp-containers]: ../speech-service/speech-container-howto.md
[ta-containers]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md

[ad-containers-billing]: ../anomaly-Detector/anomaly-detector-container-howto.md#billing
[cv-containers-billing]: ../computer-vision/computer-vision-how-to-install-containers.md#billing
[fa-containers-billing]: ../face/face-how-to-install-containers.md#billing
[fr-containers-billing]: ../form-recognizer/form-recognizer-container-howto.md#billing
[lu-containers-billing]: ../luis/luis-container-howto.md#billing
[sp-containers-billing]: ../speech-service/speech-container-howto.md#billing
[ta-containers-billing]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md#billing

[ad-containers-recommendations]: ../anomaly-Detector/anomaly-detector-container-howto.md#container-requirements-and-recommendations
[cv-containers-recommendations]: ../computer-vision/computer-vision-how-to-install-containers.md#container-requirements-and-recommendations
[fa-containers-recommendations]: ../face/face-how-to-install-containers.md#container-requirements-and-recommendations
[fr-containers-recommendations]: ../form-recognizer/form-recognizer-container-howto.md#container-requirements-and-recommendations
[lu-containers-recommendations]: ../luis/luis-container-howto.md#container-requirements-and-recommendations
[sp-containers-recommendations]: ../speech-service/speech-container-howto.md#container-requirements-and-recommendations
[ta-containers-recommendations]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md#container-requirements-and-recommendations
