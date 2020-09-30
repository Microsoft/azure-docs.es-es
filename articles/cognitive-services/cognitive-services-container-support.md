---
title: Contenedores de Azure Cognitive Services
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo los contenedores de Docker pueden obtener Cognitive Services más próximos a los datos.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.topic: article
ms.date: 09/10/2020
ms.author: aahi
ms.openlocfilehash: bda6fae31e3f5ef63d2c917937d80b2c1ea4fc48
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906992"
---
# <a name="azure-cognitive-services-containers"></a>Contenedores de Azure Cognitive Services

> [!WARNING]
> El 11 de junio de 2020 Microsoft anunció que no venderá tecnología de reconocimiento facial a los departamentos de policía de Estados Unidos hasta que se promulgue un reglamento estricto cimentado en los derechos humanos. Por lo tanto, es posible que los clientes no usen las características o la funcionalidad del reconocimiento facial incluidas en los servicios de Azure, como Face o Video Indexer, si un es un departamento de policía de Estados Unidos o permite el uso de dichos servicios por parte de cualquiera de ellos.

La compatibilidad con contenedores en Azure Cognitive Services permite a los desarrolladores usar las mismas API enriquecidas que están disponibles en Azure, y permite flexibilidad en cuanto a dónde implementar y hospedar los servicios que vienen con los [contenedores de Docker](https://www.docker.com/what-container). La compatibilidad con contenedores está disponible actualmente en versión preliminar para un subconjunto de servicios de Azure Cognitive Services, incluidas las partes de:

> [!div class="checklist"]
> * [Anomaly Detector][ad-containers]
> * [Computer Vision][cv-containers]
> * [Face][fa-containers]
> * [Form Recognizer][fr-containers]
> * [Language Understanding (LUIS)][lu-containers]
> * [Speech Service API][sp-containers]
> * [Text Analytics][ta-containers]

> [!VIDEO https://www.youtube.com/embed/hdfbn4Q8jbo]

La creación de contenedores es un enfoque de distribución de software en el que una aplicación o servicio, incluidas sus dependencias y la configuración, se empaqueta como una imagen de contenedor. La imagen de contenedor puede implementarse en un host de contenedor con pocas o ningunas modificaciones. Los contenedores están aislados entre sí y del sistema operativo subyacente, con una superficie menor que una máquina virtual. Se pueden crear instancias de contenedores a partir de las imágenes de contenedor para las tareas a corto plazo y quitarlas cuando ya no se necesiten.

Los recursos de Cognitive Services están disponibles en [Microsoft Azure](https://azure.microsoft.com). Inicie sesión en [Azure Portal](https://portal.azure.com/) para crear y explorar recursos de Azure para estos servicios.

## <a name="features-and-benefits"></a>Características y ventajas

- **Infraestructura inmutable**: permita que los equipos de DevOps aprovechen un conjunto coherente y confiable de parámetros del sistema conocidos, al tiempo que pueden adaptarse a los cambios. Los contenedores proporcionan la flexibilidad para dinamizar dentro de un ecosistema predecible y evitar el desfase de la configuración.
- **Control sobre datos**: permita que los clientes elijan dónde Cognitive Services procesa los datos. Esto es esencial para los clientes que no pueden enviar datos a la nube, pero necesitan tener acceso a la tecnología de Cognitive Services. Admite la coherencia en entornos híbridos: a través de datos, administración, identidad y seguridad.
- **Control sobre las actualizaciones del modelo**: proporcione flexibilidad a los clientes sobre el control de versiones y la actualización de los modelos implementados en sus soluciones.
- **Arquitectura portátil**: habilite la creación de una arquitectura de aplicación portátil que se pueda implementar en Azure, en el entorno local y en la red perimetral. Los contenedores se pueden implementar directamente en [Azure Kubernetes Service](../aks/index.yml), [Azure Container Instances](../container-instances/index.yml) o en un clúster de [Kubernetes](https://kubernetes.io/) implementado en [Azure Stack](/azure-stack/operator). Para obtener más información, consulte [Implementación de Kubernetes en Azure Stack](/azure-stack/user/azure-stack-solution-template-kubernetes-deploy).
- **Alto rendimiento y baja latencia**: proporcione a los clientes la capacidad de escalar para los requisitos de alto rendimiento y baja latencia permitiendo que Cognitive Services se ejecute físicamente cerca de sus datos y lógica de aplicación. Los contenedores no limitan las transacciones por segundo (TPS) y se pueden escalar tanto vertical como horizontalmente para controlar la demanda si se proporcionan los recursos de hardware necesarios.
- **Escalabilidad**: Con la popularidad cada vez mayor del software de creación y orquestación de contenedores, como Kubernetes; la escalabilidad está a la vanguardia de los avances tecnológicos. Basado en un clúster escalable, el desarrollo de aplicaciones se encarga de la alta disponibilidad.

## <a name="containers-in-azure-cognitive-services"></a>Contenedores en Azure Cognitive Services

Los contenedores de Azure Cognitive Services proporcionan el siguiente conjunto de contenedores de Docker, cada uno de los cuales contiene un subconjunto de funcionalidades de servicios de Azure Cognitive Services:

| Servicio | Plan de tarifa compatible | Contenedor | Descripción |
|--|--|--|--|
| [Anomaly Detector][ad-containers] | F0, S0 | **Anomaly Detector** ([imagen](https://hub.docker.com/_/azure-cognitive-services-decision-anomaly-detector))  | Anomaly Detector API permite supervisar y detectar anomalías en datos de serie temporal con aprendizaje automático.<br>[Solicitar acceso][request-access] |
| [Computer Vision][cv-containers] | F0, S1 | **Lectura** | Extrae texto impreso de imágenes que muestren diversos objetos con diferentes superficies y fondos, como recibos, pósteres y tarjetas de visita. El contenedor de lectura también detecta *texto manuscrito* en las imágenes y es compatible con los formatos PDF, TIFF y multipágina.<br/><br/>**Importante:** Actualmente, el contenedor de lectura solo funciona en inglés. |
| [Face][fa-containers] | F0, S0 | **Face** | Detecta caras humanas en imágenes e identifica atributos, incluidos faciales (como narices y ojos), sexo, edad y otras características faciales previstas por la máquina. Además de la detección, Face puede comprobar si dos caras en la misma o en diferentes imágenes son iguales mediante una puntuación de confianza, o bien comparar caras en una base de datos para ver si ya existe un aspecto similar o una cara idéntica. También puede organizar caras similares en grupos mediante rasgos visuales compartidos.<br>[Solicitar acceso][request-access] |
| [Form Recognizer][fr-containers] | F0, S0 | **Form Recognizer** | Form Understanding aplica tecnología de aprendizaje automático para identificar y extraer pares clave-valor de los formularios.<br>[Solicitar acceso][request-access] |
| [LUIS][lu-containers] | F0, S0 | **LUIS** ([imagen](https://go.microsoft.com/fwlink/?linkid=2043204&clcid=0x409)) | Carga un modelo de Language Understanding entrenado o publicado, lo que también se conoce como aplicación de LUIS, en un contenedor de Docker y proporciona acceso a las predicciones de consulta de los puntos de conexión de la API del contenedor. Puede recopilar registros de consultas en el contenedor y cargarlos de nuevo en el [portal de LUIS](https://www.luis.ai) para mejorar la precisión de predicción de la aplicación. |
| [Speech Service API][sp-containers-stt] | F0, S0 | **Conversión de voz en texto** ([imagen](https://hub.docker.com/_/azure-cognitive-services-speechservices-speech-to-text)) | Permite transcribir en tiempo real voz en texto. |
| [Speech Service API][sp-containers-cstt] | F0, S0 | **Conversión de voz en texto personalizada** ([imagen](https://hub.docker.com/_/azure-cognitive-services-speechservices-custom-speech-to-text)) | Permite transcribir en tiempo real voz en texto mediante un modelo personalizado. |
| [Speech Service API][sp-containers-tts] | F0, S0 | **Texto a voz** ([imagen](https://hub.docker.com/_/azure-cognitive-services-speechservices-text-to-speech)) | Convierte el texto a una voz que parece natural. |
| [Speech Service API][sp-containers-ctts] | F0, S0 | **Conversión de texto a voz personalizada** ([imagen](https://hub.docker.com/_/azure-cognitive-services-speechservices-custom-text-to-speech)) | Convierte el texto en una voz que parece natural mediante un modelo personalizado. |
| [Speech Service API][sp-containers-ntts] | F0, S0 | **Texto a voz neuronal** ([imagen](https://hub.docker.com/_/azure-cognitive-services-speechservices-neural-text-to-speech)) | Convierte texto en voz con un sonido natural utilizando una tecnología de red neuronal profunda, lo que permite obtener una voz sintetizada más natural. |
| [Text Analytics][ta-containers-keyphrase] | F0, S | **Extracción de frases clave** ([imagen](https://go.microsoft.com/fwlink/?linkid=2018757&clcid=0x409)) | Extrae las frases clave para identificar los puntos principales. Por ejemplo, si el texto de entrada es "La comida estaba deliciosa y el personal era maravilloso", la API devuelve los principales puntos de conversación: "comida" y "personal maravilloso". |
| [Text Analytics][ta-containers-language] | F0, S | **Detección de idioma** ([imagen](https://go.microsoft.com/fwlink/?linkid=2018759&clcid=0x409)) | Se detecta el idioma (120 como máximo) en que está escrito el texto de entrada y se usa un código de idioma único para informar acerca de cada documento enviado en la solicitud. El código de idioma se empareja con una puntuación que indica la intensidad de esta. |
| [Text Analytics][ta-containers-sentiment] | F0, S | **Análisis de sentimiento, versión 3** ([imagen](https://go.microsoft.com/fwlink/?linkid=2018654&clcid=0x409)) | Analiza el texto sin formato para obtener pistas sobre opiniones positivas o negativas. Esta versión del análisis de sentimiento devuelve etiquetas de sentimiento (por ejemplo, *positivo* o *negativo*) para cada documento y oración que contiene. |
| [Text Analytics][ta-containers-health] | F0, S | **Text Analytics para el estado** | Extraiga y etiquete la información médica del texto clínico no estructurado. |
| [Análisis espacial][spa-containers] | S0 | **Análisis espacial** | Extraiga y etiquete la información médica del texto clínico no estructurado. |

<!--
|[Personalizer](https://go.microsoft.com/fwlink/?linkid=2083923&clcid=0x409) |F0, S0|**Personalizer** ([image](https://go.microsoft.com/fwlink/?linkid=2083928&clcid=0x409))|Azure Personalizer is a cloud-based API service that allows you to choose the best experience to show to your users, learning from their real-time behavior.|
-->

Además, se admiten algunos contenedores en las claves del recurso de la [**oferta todo en uno**](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAllInOne) de Cognitive Services. Puede crear un único recurso todo en uno de Cognitive Services y usar la misma clave de facturación en los servicios compatibles para los siguientes servicios:

* Computer Vision
* Caras
* LUIS
* Text Analytics

## <a name="container-availability-in-azure-cognitive-services"></a>Disponibilidad de contenedores en Azure Cognitive Services

Los contenedores de Azure Cognitive Services están disponibles públicamente a través de su suscripción a Azure y se pueden extraer imágenes de contenedor de Docker desde Microsoft Container Registry o Docker Hub. Puede usar el comando [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) para descargar una imagen de contenedor desde el registro apropiado.

[!INCLUDE [Container repositories and images](containers/includes/cognitive-services-container-images.md)]

## <a name="prerequisites"></a>Requisitos previos

Debe cumplir los siguientes requisitos previos para poder utilizar contenedores de Azure Cognitive Services:

**Motor de Docker**: debe tener el motor de Docker instalado localmente. Docker proporciona paquetes que configuran el entorno de Docker en [macOS](https://docs.docker.com/docker-for-mac/), [Linux](https://docs.docker.com/engine/installation/#supported-platforms) y [Windows](https://docs.docker.com/docker-for-windows/). En Windows, Docker debe configurarse para admitir los contenedores de Linux. Los contenedores de Docker también se pueden implementar directamente en [Azure Kubernetes Service](../aks/index.yml) o [Azure Container Instances](../container-instances/index.yml).

Docker debe configurarse para permitir que los contenedores se conecten con Azure y envíen datos de facturación a dicho servicio.

**Familiaridad con Microsoft Container Registry y Docker**: debe tener un conocimiento básico de los conceptos de Microsoft Container Registry y Docker, como los registros, los repositorios, los contenedores y las imágenes de contenedor, así como de los comandos `docker` básicos.

Para conocer los principios básicos de Docker y de los contenedores, consulte [Introducción a Docker](https://docs.docker.com/engine/docker-overview/).

Los contenedores individuales también pueden tener sus propios requisitos, incluidos los requisitos de asignación de memoria y servidor.

[!INCLUDE [Cognitive Services container security](containers/includes/cognitive-services-container-security.md)]

[!INCLUDE [Discoverability of more container information](../../includes/cognitive-services-containers-discoverability.md)]

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre las [recetas del contenedor](containers/container-reuse-recipe.md) que puede usar con Cognitive Services.

Instale y explore la funcionalidad proporcionada por los contenedores en Azure Cognitive Services:

* [Contenedores de Anomaly Detector][ad-containers]
* [Contenedores de Computer Vision][cv-containers]
* [Contenedores de Face][fa-containers]
* [Contenedores de Form Recognizer][fr-containers]
* [Contenedores de Language Understanding (LUIS)][lu-containers]
* [Contenedores de Speech Service API][sp-containers]
* [Contenedores de Text Analytics][ta-containers]

<!--* [Personalizer containers](https://go.microsoft.com/fwlink/?linkid=2083928&clcid=0x409)
-->

[ad-containers]: anomaly-Detector/anomaly-detector-container-howto.md
[cv-containers]: computer-vision/computer-vision-how-to-install-containers.md
[fa-containers]: face/face-how-to-install-containers.md
[fr-containers]: form-recognizer/form-recognizer-container-howto.md
[lu-containers]: luis/luis-container-howto.md
[sp-containers]: speech-service/speech-container-howto.md
[spa-containers]: https://docs.microsoft.com/azure/cognitive-services/computer-vision/spatial-analysis-container
[sp-containers-stt]: speech-service/speech-container-howto.md?tabs=stt
[sp-containers-cstt]: speech-service/speech-container-howto.md?tabs=cstt
[sp-containers-tts]: speech-service/speech-container-howto.md?tabs=tts
[sp-containers-ctts]: speech-service/speech-container-howto.md?tabs=ctts
[sp-containers-ntts]: speech-service/speech-container-howto.md?tabs=ntts
[ta-containers]: text-analytics/how-tos/text-analytics-how-to-install-containers.md
[ta-containers-keyphrase]: text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=keyphrase
[ta-containers-language]: text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=language
[ta-containers-sentiment]: text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=sentiment
[ta-containers-health]: text-analytics/how-tos/text-analytics-how-to-install-containers.md?tabs=health
[request-access]: https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRyQZ7B8Cg2FEjpibPziwPcZUNlQ4SEVORFVLTjlBSzNLRlo0UzRRVVNPVy4u
