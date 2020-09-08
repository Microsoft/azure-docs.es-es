---
title: ¿Qué es Personalizer?
description: Personalizer es un servicio basado en la nube que permite elegir la mejor experiencia, mostrársela a los usuarios y aprender de su comportamiento en tiempo real.
ms.topic: overview
ms.date: 08/27/2020
ms.custom: cog-serv-seo-aug-2020
keywords: personalizer, Azure personalizer, machine learning
ms.openlocfilehash: 93b1f0487818cf783cc4a19f79618cfe2285cbd1
ms.sourcegitcommit: 420c30c760caf5742ba2e71f18cfd7649d1ead8a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89055624"
---
# <a name="what-is-personalizer"></a>¿Qué es Personalizer?

Azure Personalizer es un servicio basado en la nube que ayuda a las aplicaciones a elegir el mejor elemento de contenido y mostrárselo a los usuarios. El servicio Personalizer se puede usar para determinar qué producto sugerir a los compradores o para averiguar la posición óptima de un anuncio. Después de que el contenido se muestra al usuario, el sistema supervisa el comportamiento del usuario en tiempo real y devuelve una puntuación de recompensa al servicio Personalizer, lo que garantiza tanto una mejora continua del modelo de Machine Learning como la capacidad de Personalizer para seleccionar el mejor elemento de contenido en función de la información contextual que recibe.

> [!TIP]
> El contenido es cualquier unidad de información, como texto, imágenes, direcciones URL o correos electrónicos que quiera seleccionar y mostrar a los usuarios.

Antes de empezar, no dude en probar [Personalizer con esta demostración interactiva](https://personalizationdemo.azurewebsites.net/).

<!--
![What is personalizer animation](./media/what-is-personalizer.gif)
-->

## <a name="how-does-personalizer-select-the-best-content-item"></a>¿Cómo selecciona Personalizer el mejor elemento de contenido?

Personalizer usa el **aprendizaje de refuerzo** para seleccionar el mejor elemento (_acción_) en función del comportamiento colectivo y las puntuaciones de recompensa de todos los usuarios. Las acciones son los elementos de contenido, como artículos de noticias, películas específicas o productos.

La llamada **Rank** toma el elemento de acción, junto con las características de la acción, y las características de contexto para seleccionar el elemento de acción superior:

* **Acciones con características**: elementos de contenido con características específicas de cada elemento
* **Características de contexto**: características de los usuarios, su contexto o su entorno cuando se usa la aplicación

La llamada Rank devuelve el identificador del elemento de contenido, __acción__, que se mostrará al usuario, en el campo **Reward Action ID** (Id. de acción de recompensa).

La __acción__ que se muestra al usuario se elige con los modelos de Machine Learning, que intentan maximizar la cantidad total de recompensas con el paso del tiempo.

### <a name="sample-scenarios"></a>Escenarios de ejemplo

Echemos un vistazo a algunos escenarios en los que se puede usar Personalizer para seleccionar el mejor contenido que se va a representar para un usuario.

|Tipo de contenido|Acciones (con características)|Características de contexto|Identificador de acción de recompensa que se devuelve<br>(se muestra este contenido)|
|--|--|--|--|
|Lista de noticias|a. `The president...` (nacional, política, [texto])<br>b. `Premier League ...` (global, deportes, [texto, imagen, vídeo])<br> c. `Hurricane in the ...` (regional, información meteorológica, [texto, imagen]|Las noticias del dispositivo se leen desde<br>Mes o temporada<br>|a `The president...`|
|Lista de películas|1. `Star Wars` (1977, [acción, aventura, fantasía], George Lucas)<br>2. `Hoop Dreams` (1994, [documental, deportes], Steve James<br>3. `Casablanca` (1942, [romance, drama, guerra], Michael Curtiz)|La película del dispositivo se visiona desde<br>tamaño de pantalla<br>Tipo de usuario<br>|3. `Casablanca`|
|Lista de productos|i. `Product A` (3 kg, $$$$, entrega en 24 horas)<br>ii. `Product B` (20 kg, $$, envío en 2 semanas con tasas de aduanas)<br>iii. `Product C` (3 kg, $$$, entrega en 48 horas)|El envío del dispositivo se lee desde<br>Nivel de gastos del usuario<br>Mes o temporada|ii. `Product B`|

Personalizer ha usado el aprendizaje de refuerzo para seleccionar la mejor acción individual, conocida como _identificador de acción de recompensa_. El modelo de Machine Learning usa: 

* Un modelo entrenado: información que se ha recibido previamente del servicio dfe personalización y que se usa para mejorar el modelo de Machine Learning.
* Datos actuales: acciones específicas con características y características de contexto

## <a name="when-to-use-personalizer"></a>¿Cuándo se utiliza Personalizer?

A la [API](https://go.microsoft.com/fwlink/?linkid=2092082) **Rank** de Personalizer se le llama cada vez que una aplicación presenta contenido. Esto se conoce como un **evento**, anotado con un _identificador de evento_.

Se puede llamar a **API** [Reward](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/Reward) de Personalizer en tiempo real o con retardo para que se adapte mejor a su infraestructura. La puntuación de recompensa se determinará según sus necesidades empresariales. La puntuación de recompensa oscila entre 0 y 1. Esta puntuación debe ser un solo valor, por ejemplo, 1 para bueno y 0 para malo, o un número generado por un algoritmo que cree teniendo en cuenta las métricas y los objetivos empresariales.

## <a name="content-requirements"></a>Requisitos de contenido

Use Personalizer cuando el contenido:

* Tenga un conjunto limitado de elementos (50 como máximo) para seleccionar. Si tiene una lista más grande, [use un motor de recomendaciones](where-can-you-use-personalizer.md#how-to-use-personalizer-with-a-recommendation-solution) para reducir la lista a 50 elementos.
* Contenga información que describa el contenido que quiere clasificar: _acciones con características_ y _características de contexto_.
* Tenga un número mínimo de eventos relacionados con contenido de 1 KB/día para que Personalizer funcione de forma efectiva. Si Personalizer no recibe el tráfico mínimo necesario, el servicio tarda más en determinar el mejor elemento de contenido.

Dado que Personalizer usa información colectiva casi en tiempo real para devolver el mejor elemento de contenido, el servicio no:
* Conserva ni administra la información de perfil de usuario
* Registra las preferencias ni el historial de usuarios individuales
* Necesita contenido limpio ni etiquetado

## <a name="how-to-design-for-and-implement-personalizer"></a>Diseño e implementación de Personalizer

1. [Diseñe](concepts-features.md) y planee contenido, **_acciones_** y **_contexto_**. Determine el algoritmo de recompensa para la puntuación de **_recompensa_**.
1. Cada [recurso de Personalizer](how-to-settings.md) que cree se considera un bucle de aprendizaje. El bucle recibirá las llamadas Rank y Reward para ese contenido o experiencia de usuario.

    |Tipo de recurso| Propósito|
    |--|--|
    |[Modo de aprendiz](concept-apprentice-mode.md) `E0`|Aprendizaje del modelo de Personalizer sin afectar a la aplicación existente e implementación del comportamiento de aprendizaje en línea en un entorno de producción|
    |Estándar, `S0`|Comportamiento de aprendizaje en línea en un entorno de producción|
    |Gratis, `F0`| Comportamiento de aprendizaje en línea en un entorno que no es de producción|

1. Agregue Personalizer a su aplicación, sitio web o sistema:
    1. Agregue una llamada **Rank** a Personalizer en su aplicación, sitio web o sistema para determinar el mejor elemento de _contenido_ antes de que se muestre el contenido al usuario.
    1. Muestre al usuario el mejor elemento de _contenido_, que es el _identificador de acción de recompensa_ devuelto.
    1. Aplique la _lógica de negocios_ a la información recopilada sobre cómo se comportará el usuario, para determinar la puntuación de **recompensa**, por ejemplo:

    |Comportamiento|Puntuación de recompensa calculada|
    |--|--|
    |El usuario seleccionó el mejor elemento de _contenido_ (identificador de acción de recompensa)|**1**|
    |El usuario seleccionó otro contenido|**0**|
    |El usuario hizo una pausa, se desplazó sin decidirse, antes de seleccionar el mejor elemento de _contenido_ (identificador de acción de recompensa)|**0,5**|

    1. Agregue una llamada **Reward** para enviar una puntuación de recompensa entre 0 y 1
        * Inmediatamente después de mostrar el contenido
        * O, en algún momento posterior en un sistema sin conexión
    1. [Evalúe el bucle](concepts-offline-evaluation.md) con una evaluación sin conexión después de un período de uso. La evaluación sin conexión permite probar y evaluar la eficacia del servicio Personalizer sin cambiar el código o sin que se vea afectada la experiencia del usuario.

## <a name="complete-a-quickstart"></a>Completar una guía de inicio rápido

Ofrecemos inicios rápidos de C#, JavaScript y Python. El diseño de cada inicio rápido le permite ejecutar el código en menos de 10 minutos. 

* [Inicio rápido: Procedimientos para usar la biblioteca cliente de Personalizer](sdk-learning-loop.md)

Una vez que haya tenido la oportunidad de empezar a utilizar el servicio Personalizer, pruebe nuestros tutoriales y aprenda a usar Personalizer en aplicaciones web, bots de chat o una instancia de Azure Notebook.

* [Tutorial: Uso de Personalizer en una aplicación web de .NET](tutorial-use-personalizer-web-app.md)
* [Tutorial: Uso de Personalizer en un bot de chat de .NET](tutorial-use-personalizer-chat-bot.md)
* [Tutorial: Uso de Personalizer en una instancia de Azure Notebook](tutorial-use-azure-notebook-generate-loop-data.md)

## <a name="reference"></a>Referencia 

* [SDK de C#/.NET de Personalizer](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/personalizer?view=azure-dotnet)
* [SDK de Go de Personalizer](https://github.com/Azure/azure-sdk-for-go/tree/master/services/preview/personalizer/v1.0/personalizer)
* [SDK de JavaScript de Personalizer](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-personalizer/?view=azure-node-latest)
* [SDK de Python de Personalizer](https://docs.microsoft.com/python/api/overview/azure/cognitiveservices/personalizer?view=azure-python)
* [API de REST](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/Rank)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Funcionamiento de Personalizer](how-personalizer-works.md)
> [¿Qué es el aprendizaje de refuerzo?](concepts-reinforcement-learning.md)
