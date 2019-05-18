---
title: 'Notas de la versión: Custom Vision Service'
titlesuffix: Azure Cognitive Services
services: cognitive-services
author: anrothMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 04/03/2019
ms.author: anroth
ms.openlocfilehash: 4b4e449b9260debbe90523141cacc4ed76219490
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65787950"
---
# <a name="custom-vision-service-release-notes"></a>Notas de la versión de Custom Vision Service

## <a name="may-2-2019-and-may-10-2019"></a>2 de mayo de 2019 y 10 de mayo de 2019

- Mejoras de correcciones de errores y de back-end

## <a name="may-23-2019"></a>23 de mayo de 2019

- Portal mejorada experiencia UX relacionados con las suscripciones de Azure, lo que facilita seleccionar los directorios de Azure.

## <a name="april-18-2019"></a>18 de abril de 2019 

- Se ha agregado la exportación de detección de objetos para la visión de inteligencia artificial Dev Kit.
- Ajustes de la interfaz de usuario, incluidos la búsqueda de proyecto.

## <a name="april-3-2019"></a>3 de abril de 2019

- Aumento del límite del número de rectángulos por imagen a 200. 
- Correcciones de errores, incluida la actualización de rendimiento sustancial para los modelos exportado a TensorFlow. 

## <a name="march-26-2019"></a>26 de marzo de 2019

- Custom Vision Service ha entrado en Disponibilidad General en Azure.
- Se ha agregado la característica de aprendizaje avanzado con un nuevo back-end para mejorar el rendimiento, especialmente en un desafío de conjuntos de datos y clasificación específica de aprendizaje automático. Con los cursos avanzados, puede especificar que un presupuesto de tiempo de proceso para el entrenamiento y Custom Vision empíricamente identificará la mejor configuración de formación y aumento. Para las iteraciones rápidas, aún puede usar el aprendizaje rápido existente.
- Introducido en 3.0 API. Anunciado en desuso próximo API pre-3.0 en el 1 de octubre de 2019. Vea los tutoriales de documentación para [.Net](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/csharp-tutorial), [Python](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/python-tutorial), [nodo](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/node-tutorial), [Java](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/java-tutorial), o [vaya](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/go-tutorial) para obtener ejemplos acerca de cómo empezar a trabajar.
- Reemplaza "Iteraciones predeterminado" con publicar o cancelar la publicación en la versión 3.0 de API.
- Se han agregado nuevos destinos de exportación del modelo. Exportación de Dockerfile se ha actualizado para admitir ARM para Raspberry Pi 3. Compatibilidad con la exportación se ha agregado a la [Vision AI Dev Kit.](https://visionaidevkit.com/).
- Mayor límite de etiquetas por proyecto a 500 para el nivel S0. Mayor límite de imágenes de cada proyecto de 100.000 para nivel S0.
- Quitar dominio para adultos. Dominio general se recomienda en su lugar.
- Anunciado [precios](https://azure.microsoft.com/pricing/details/cognitive-services/custom-vision-service/) para Disponibilidad General.  

## <a name="february-25-2019"></a>25 de febrero de 2019

- Anunció el final de la versión de prueba limitado proyectos (no está asociado con un recurso de Azure), como Custom Vision se acerque la finalización de su migración a vista previa pública de Azure. A partir del 25 de marzo de 2019, el sitio CustomVision.ai solo admite proyectos de visualización asociados con un recurso de Azure, como el recurso de Custom Vision gratuito. A través del 1 de octubre de 2019, aún podrá tener acceso a los proyectos de prueba limitados existentes a través de la API de visión personalizado. Esto le dará tiempo para las claves de API de actualización para las aplicaciones que ha escrito con Custom Vision. Después del 1 de octubre de 2019, se eliminarán los proyectos de prueba limitados no ha movido a Azure.

## <a name="january-22-2019"></a>22 de enero de 2019

- Se ha agregado compatibilidad para nuevas regiones de Azure: Oeste de EE. UU. 2, Este de EE. UU., Este de EE. UU. 2, Oeste de Europa, Norte de Europa, Sudeste Asiático, Este de Australia, India central, Sur de Reino Unido, Japón Oriental, Centro-norte de EE. UU. Se mantiene la compatibilidad para Centro-sur de EE. UU.

## <a name="december-12-2018"></a>12 de diciembre de 2018

- Admite la exportación de modelos de detección de objetos (se ha introducido el dominio compacto de detección de objetos).
- Se han corregido varios problemas de accesibilidad para mejorar el lector de pantalla y el soporte para navegación del teclado.
- Actualizaciones de la experiencia de usuario en el visor de imágenes y se ha mejorado el etiquetado de la detección de objetos, con el fin de agilizar el etiquetado.  
- Actualización del modelo base del dominio de detección de objetos para mejorar la calidad de la detección de objetos.
- Correcciones de errores.

## <a name="november-6-2018"></a>6 de noviembre de 2018

- Se ha agregado compatibilidad con el dominio de logotipo en la detección de objetos.

## <a name="october-9-2018"></a>9 de octubre de 2018

- Detección de objetos entra en la versión preliminar de pago. Ahora puede crear proyectos de detección de objetos con un recurso de Azure.
- La característica "Mover a Azure" se ha agregado al sitio web, para que resulte más fácil actualizar un proyecto de prueba limitado para vincular a un proyecto de recursos vinculados de Azure (F0 o S0). Puede encontrarlo en la página Configuración del producto.  
- Se ha agregado exportación a ONNX 1.2, para admitir la versión Actualización de octubre de 2018 de Windows de Windows ML.
Correcciones de errores, incluidos para la exportación ONNX con caracteres especiales.

## <a name="august-14-2018"></a>14 de agosto de 2018

- Se agregó un widget de "Introducción" al sitio customvision.ai para orientar a los usuarios a lo largo del aprendizaje del proyecto.
- Otras mejoras en la canalización de aprendizaje automático para beneficiar a los proyectos multietiqueta (nueva capa de pérdida).

## <a name="june-28-2018"></a>28 de junio de 2018

- Correcciones de errores y mejoras de back-end.
- Habilitar clasificación multiclase, para los proyectos donde las imágenes tienen exactamente una etiqueta. En las predicciones para el modo de varias clases, las probabilidades se sumará a uno (todas las imágenes se clasifican entre las etiquetas especificadas).

## <a name="june-13-2018"></a>13 de junio de 2018

- Actualización de la experiencia del usuario centrada en la facilidad de uso y la accesibilidad.
- Mejoras en la canalización de aprendizaje automático para beneficiar a los proyectos multietiqueta con una gran cantidad de etiquetas.
- Se corrigió el error en la exportación de TensorFlow. Se han habilitado las versiones de modelo exportado, para que las iteraciones se puedan exportar más de una vez.

## <a name="may-7-2018"></a>7 de mayo de 2018

- Se ha incorporado la versión preliminar de la característica de detección de objetos para proyectos con versiones de prueba limitadas.
- Las API se han actualizado a la versión 2.0.
- El nivel S0 se ha ampliado hasta 250 etiquetas y 50 000 imágenes.
- Se han incorporado mejoras importantes en los componentes back-end de la canalización de aprendizaje automático para los proyectos de clasificación de imágenes. Los proyectos entrenados después del 27 de abril de 2018 se beneficiarán de estas actualizaciones.
- Se ha agregado la exportación de modelos en ONNX para que pueda usarse con aprendizaje automático de Windows.
- Se ha agregado la exportación de modelos en Dockerfile. Esto le permite descargar los artefactos necesarios para generar sus propios contenedores de Windows o Linux, como un archivo DockerFile, un modelo de TensorFlow y un código del servicio.
- En el caso de los modelos recién entrenados que se exportan a TensorFlow en los dominios General (compacto) y Landmark (compacto), [ahora los valores medios son (0,0,0)](https://github.com/azure-samples/cognitive-services-android-customvision-sample) para que sean coherentes en todos los proyectos.

## <a name="march-1-2018"></a>1 de marzo de 2018

- Vista preliminar de pago especificado e incorporarla en Azure portal. Ahora, los proyectos pueden vincularse a recursos de Azure utilizando un nivel F0 (gratis) o S0 (estándar). Se han incorporado proyectos en el nivel S0, lo que permite tener hasta 100 etiquetas y 25 000 imágenes.
- Se han realizado cambios en los componentes back-end del parámetro de normalización y la canalización de aprendizaje automático. De este modo, los clientes tendrán un mayor control sobre el equilibrio entre precisión y exhaustividad al ajustar el umbral de probabilidad. Como parte de estos cambios, el umbral de probabilidad predeterminado del portal CustomVision.ai se ha establecido en 50 %.

## <a name="december-19-2017"></a>19 de diciembre de 2017

- Se ha incorporado la exportación a Android (TensorFlow), junto con la exportación a iOS (CoreML) publicada anteriormente. Esto permite exportar un modelo compacto entrenado para ejecutarlo sin conexión en una aplicación.
- Se han agregado los dominios "compactos" Retail y Landmark para permitir la exportaciones de modelos en estos dominios.
- Se ha publicado la versión [1.2 de Training API](https://southcentralus.dev.cognitive.microsoft.com/docs/services/f2d62aa3b93843d79e948fe87fa89554/operations/5a3044ee08fa5e06b890f11f) y la versión [1.1 de Prediction API](https://southcentralus.dev.cognitive.microsoft.com/docs/services/57982f59b5964e36841e22dfbfe78fc1/operations/5a3044f608fa5e06b890f164). Las API actualizadas admiten la exportación de modelos, una nueva operación de predicción que no guarda las imágenes en "Predictions" (Predicciones). Además, se han incorporado operaciones por lotes en Training API.
- Se han realizado pequeños ajustes en la experiencia del usuario, como la capacidad para ver qué dominio se ha utilizado para entrenar una iteración.
- Se ha actualizado el [SDK y el ejemplo de C#](https://github.com/Microsoft/Cognitive-CustomVision-Windows).