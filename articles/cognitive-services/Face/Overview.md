---
title: ¿Qué es el servicio Face?
titleSuffix: Azure Cognitive Services
description: El servicio Face de Azure Cognitive Services ofrece algoritmos que se utilizan para detectar, reconocer y analizar caras humanas en imágenes.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: overview
ms.date: 9/01/2020
ms.author: pafarley
ms.openlocfilehash: e1b5ca8cfb9a6bc0fec7d85f609c411f38d1bbbc
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89280081"
---
# <a name="what-is-the-azure-face-service"></a>¿Qué es el servicio Azure Face?
<!--
> [!IMPORTANT]
> On June 11, 2020, Microsoft announced that it will not sell facial recognition technology to police departments in the United States until strong regulation, grounded in human rights, has been enacted. As such, customers may not use facial recognition features or features included in Azure Services, such as Face or Video Indexer, if a customer is, or is allowing use of such services by or for, a police department in the United States.
-->
![Información legal ](./media/overview/banner.png)

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

El servicio Face de Azure Cognitive Services ofrece algoritmos para detectar, reconocer y analizar caras humanas en imágenes. La capacidad de procesar la información de caras humanas es importante en muchos escenarios de software diferentes. Algunos escenarios de ejemplo son el de la seguridad, la interfaz de usuario natural, el análisis y la administración del contenido de imagen, las aplicaciones móviles y la robótica.

El servicio Face proporciona varias funciones distintas, que se describen en las secciones siguientes.

## <a name="face-detection"></a>Detección de caras

El servicio Face detecta rostros humanos en una imagen y devuelve las coordenadas del rectángulo de sus ubicaciones. Si lo desea, la detección de caras puede extraer una serie de atributos relacionados con la cara. Algunos ejemplos son la posición de la cabeza, el sexo, la edad, las emociones, el vello facial y las gafas.

> [!NOTE]
> La característica de la detección de caras también está disponible con [Computer Vision API](https://docs.microsoft.com/azure/cognitive-services/computer-vision/home). Si desea realizar más operaciones con datos faciales, use el servicio Face, que es el servicio descrito en este artículo.

![Una imagen de una mujer y un hombre, con rectángulos dibujados en torno a sus caras y edad y sexo.](./Images/Face.detection.jpg)

Para más información sobre la detección de caras, vea el artículo sobre los conceptos de la [Detección de caras](concepts/face-detection.md). Vea también la documentación de referencia de [Detect API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).

## <a name="face-verification"></a>Verificación de caras

Verify API realiza una autenticación con dos caras detectadas o a partir de una cara detectada en un objeto personal. En la práctica, evalúa si dos caras pertenecen a la misma persona. Esta funcionalidad puede ser muy útil en escenarios de seguridad. Para más información, vea la guía de conceptos de [reconocimiento de facial](concepts/face-recognition.md) o la documentación de referencia sobre [Verify API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a).

## <a name="find-similar-faces"></a>Búsqueda de caras similares

Find Similar API compara una cara objetivo con una serie de caras candidatas para buscar un conjunto más pequeño de las caras que más se asemejan a la cara objetivo. Se admiten dos modos de funcionamiento: matchPerson y matchFace. El modo matchPerson devuelve las caras parecidas tras filtrar a la misma persona mediante [Verify API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a). El modo matchFace ignora el filtro de la misma persona. Devuelve una lista de caras similares candidatas que pueden o no pertenecer a la misma persona.

En el siguiente ejemplo se muestra la cara objetivo:

![Una mujer sonriendo](./Images/FaceFindSimilar.QueryFace.jpg)

Y estas imágenes son las caras candidatas:

![Cinco imágenes de personas sonriendo. Las imágenes a y b muestran a la misma persona.](./Images/FaceFindSimilar.Candidates.jpg)

Al buscar cuatro caras parecidas, el modo matchPerson devuelve a y b, que muestran a la misma persona que la cara objetivo. El modo matchFace devuelve a, b, c y d&mdash; exactamente cuatro caras candidatas, aunque algunas no sean la misma persona que el objetivo o el nivel de similitud sea bajo. Para más información, vea la guía de conceptos de [reconocimiento de facial](concepts/face-recognition.md) o la documentación de referencia sobre [Find Similar API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237).

## <a name="face-grouping"></a>Agrupación de caras

Group API divide un conjunto de caras desconocidas en varios grupos en función de la similitud. Cada grupo es un subconjunto apropiado separado del conjunto original de caras. Todas las caras de un grupo probablemente pertenecen a la misma persona. Puede haber varios grupos diferentes para una sola persona. Los grupos se diferencian por otro factor, por ejemplo, la expresión. Para más información, vea la guía de conceptos de [reconocimiento de facial](concepts/face-recognition.md) o la documentación de referencia sobre [Group API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395238).

## <a name="person-identification"></a>Identificación de personas

Identify API se usa para identificar una cara detectada en una base de datos de personas. Esta característica puede resultar útil para el etiquetado automático de imágenes en software de administración de fotografías. Cree la base de datos con antelación y puede modificarla con el tiempo.

La siguiente imagen muestra un ejemplo de una base de datos llamada `"myfriends"`. Cada grupo puede contener hasta un millón de objetos de persona diferentes. Cada objeto de persona puede tener hasta 248 caras registradas.

![Una cuadrícula con tres columnas para diferentes personas, cada una con tres filas de imágenes de caras.](./Images/person.group.clare.jpg)

Una vez creada y entrenada la base de datos, puede realizar la identificación en el grupo con una cara nueva detectada. Si la cara se identifica como una persona en el grupo, se devuelve el objeto de persona.

Para más información sobre la identificación de personas, vea la guía de conceptos de [reconocimiento de facial](concepts/face-recognition.md) o la documentación de referencia sobre [Identify API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239).

## <a name="containers"></a>Contenedores

[Utilice el contenedor de caras](face-how-to-install-containers.md) para detectar, reconocer e identificar caras, mediante la instalación de un contenedor de Docker estándar más cercano a los datos.

## <a name="sample-apps"></a>Aplicaciones de ejemplo

Las aplicaciones de ejemplo siguientes muestran algunas formas de usar el servicio Face:

- [API Face: biblioteca de cliente Windows y ejemplo](https://github.com/Microsoft/Cognitive-Face-Windows) es una aplicación de WPF que demuestra varios escenarios de detección, análisis e identificación de caras.
- [Aplicación FamilyNotes UWP](https://github.com/Microsoft/Windows-appsample-familynotes) es una aplicación de la Plataforma universal de Windows (UWP) que usa la identificación facial junto con la voz, Cortana, la escritura manuscrita y la cámara en un escenario de uso compartido de notas en familia.

## <a name="data-privacy-and-security"></a>Seguridad y privacidad de datos

Al igual que sucede con todos los recursos de Cognitive Services, los desarrolladores que usan el servicio Face deben estar al tanto de las directivas de Microsoft sobre los datos de los clientes. Para más información, vea la [página de Cognitive Services](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) en Microsoft Trust Center.

## <a name="next-steps"></a>Pasos siguientes

Siga una guía de inicio rápido para implementar un escenario de identificación facial en el código:

- [Inicio rápido: Detección de caras en una imagen mediante el SDK de .NET con C#](quickstarts/csharp.md). Hay otros lenguajes disponibles.
