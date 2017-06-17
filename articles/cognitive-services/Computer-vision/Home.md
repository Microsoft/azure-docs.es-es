--- 
title: Computer Vision API for Microsoft Cognitive Services | Microsoft Docs
description: Use advanced algorithms in the Computer Vision API to help you process images and return information in Microsoft Cognitive Services.
services: cognitive-services
author: JuliaNik
manager: ytkuo
ms.service: cognitive-services
ms.technology: computer-vision
ms.topic: article
ms.date: 02/06/2017
ms.author: juliakuz
ms.translationtype: Human Translation
ms.sourcegitcommit: d9ae8e8948d82b9695d7d144d458fe8180294084
ms.openlocfilehash: 47eb043d07b0cbaa9f2a0232ff56872fab8fbcad
ms.contentlocale: es-es
ms.lasthandoff: 05/23/2017

--- 
 
# <a name="computer-vision-api-version-10"></a>API de Computer Vision Versión 1.0 
La API de Computer Vision basada en la nube provee a los desarrolladores de acceso a algoritmos avanzados para el procesamiento de imágenes y retorno de información. Al subir una imagen o especificando una URL de imagen, los algoritmos del servicio de Computer Vision pueden analizar el contenido visual de diferentes maneras basándose en lo ingresado y las opciones del usuario. Con la API de Computer Vision los usuarios pueden analizar imágenes para:
* [Etiquetar imágenes basado en su contenido.](#Tagging)
* [Categorizar imágenes .](#Categorizing)
* [Identificar el tipo y la calidad de las imágenes .](#Identifying)
* [Detectar rostros humanos y regresar sus coordinadas.](#Faces)
* [Reconocer el dominio específico del dominio.](#Domain-Specific)
* [Generar descripciones de contenido.](#Descriptions)
* [Reconimiento de texto manuscrito.](#RecognizeText)
* [Distinguir esquemas de color.](#Color)
* [Marcar contenido sólo para adultos.](#Adult)
* [Utilizar reconocimiento óptico de caracteres para identificar texto encontrado dentro de las imagenes.](#OCR)
* [Recortar fotografías que pueden usarse como miniaturas.](#Thumbnails)

## <a name="requirements"></a>Requisitos
- Métodos de entrada soportados: Binarios de imagen RAW en el formato de un stream de application/octet stream o la URL de imagen.
- Formatos de imagen soportados: JPEG, PNG, GIF, BMP.
- Tamaño del archivo de imagen: Menor a 4 MB.
- Dimensión de la imagen: Más de 50 x 50 pixeles

## <a name="Tagging">Etiquetado de imágenes</a>
La API de Computer Vision regresa etiquetas basadas en más de 2000 objetos reconocidos, seres vivos, paisajes y acciones. Cuando las etiquetas son ambiguas o no muy comunes, la respuesta de la API regresa 'pistas' para aclarar el significado de la etiqueta en el contexto de una configuración conocida. 

Las etiquetas no están organizadas por su taxonomía y no existen jerarquías de herencia. Una colección de etiquetas de contenido es la base de la 'descripción' en la imagen mostrado en lenguaje legible formateado en oraciones completas. Como nota, por el momento inglés es el único lenguaje disponible para la lectura de la descripción de la imagen.

Después de subir una imagen o especificar la URL de una imagen, los algoritmos de la API de Computer Vision regresa etiquetas basadas en los objetos, seres vivos, paisajes y acciones identificadas en la imagen. El etiquetado no se limita al sujeto principal, como una persona en primer plano, también incluyen configuraciones (interior o exterior), muebles, herramientas, plantas, animales, accesorios, gadgets, etc.

### <a name="example"></a>Ejemplo
![House_Yard](./Images/house_yard.jpg) '

```json
Json de respuesta
{
   'tags':[
      {
         "name":"grass",
         "confidence":0.999999761581421
      },
      {
         "name":"outdoor",
         "confidence":0.999970674514771
      },
      {
         "name":"sky",
         "confidence":999289751052856
      },
      {
         "name":"building",
         "confidence":0.996463239192963
      },
      {
         "name":"house",
         "confidence":0.992798030376434
      },
      {
         "name":"lawn",
         "confidence":0.822680294513702
      },
      {
         "name":"green",
         "confidence":0.641222536563873
      },
      {
         "name":"residential",
         "confidence":0.314032256603241
      },
   ],
}
```
## <a name="Categorizing">Categorización de imagenes</a>
Además del etiquetado y las descripciones, la API de Computer Vision regresa categorías basadas en la taxonomía de versiones anteriores. Estas categorías son organizadas como una taxonomía con herencia de padre/hijo hereditarias. Todas las categorías están en inglés. Se pueden usar solas o con nuestros nuevos modelos. 

### <a name="the-86-category-concept"></a>El concepto de las 86 categorías
Basándose en una lista de 86 conceptos que vemos en el siguiente diagrama, las características visuales que se encuentran en una imagen se pueden categorizar desde lo más amplio a lo más específico. Para el texto completo de la taxonomía, revisa [Taxonomía de las categorías](https://docs.microsoft.com/en-us/azure/cognitive-services/computer-vision/category-taxonomy).

![Analyze Categories](./Images/analyze_categories.jpg)

Imagen                                                 | Respuesta
------------------------------------------------------ | ----------------
![Woman Roof](./Images/woman_roof.jpg)                 | people
![Family Photo](./Images/family_photo.jpg)             | people_crowd
![Cute Dog](./Images/cute_dog.jpg)                     | animal_dog
![Outdoor Mountain](./Images/mountain_vista.jpg)       | outdoor_mountain
![Vision Analyze Food Bread](./Images/bread.jpg)       | food_bread

## <a name="Identifying">Identificando los tipos de imagen</a>
Hay varias maneras para categorizar las imágenes. La API de Computer Vision puede tener un identificador booleano para mostrar si una imagen está en blanco y negro o color. También se puede tener un identificador booleano para mostrar si una imagen es un dibujo o no. También para indicar si una imagen es un clip art o no indicando su calidad como escala de 0-3.

### <a name="clip-art-type"></a>Tipo Clip-art 
Detecta si una imagen es un clip art o no.  

Valor | Significado
----- | --------------
0     | Non-clip-art
1     | ambiguous
2     | normal-clip-art
3     | good-clip-art

Imagen|Respuesta
----|----
![Vision Analyze Cheese Clip Art](./Images/cheese_clipart.jpg)|3 good-clip-art
![Vision Analyze House Yard](./Images/house_yard.jpg)|0 Non-clip-art

### <a name="line-drawing-type"></a>Tipo dibujo
Detecta su imagen es un dibujo o no.  

Imagen|Respuesta
----|----
![Vision Analyze Lion Drawing](./Images/lion_drawing.jpg)|True
![Vision Analyze Flower](./Images/flower.jpg)|False

### <a name="Faces">Rostros</a>
Detecta rostros humanos dentro de una imagen y genera las coordinadas faciales, el rectángulo de un rostro, sexo y edad. Estas características visuales son un subconjunto de metadatos generados por rostro. Para un metadata más extensivo para rostros (identificación facial, detección de posición y más), usa la API de Face.

Imagen|Respuesta
----|----
![Vision Analyze Woman Roof Face](./Images/woman_roof_face.png) | [ { "age": 23, "gender": "Female", "faceRectangle": { "left": 1379, "top": 320, "width": 310, "height": 310 } } ]
![Vision Analyze Mom Daughter Face](./Images/mom_daughter_face.png) | [ { "age": 28, "gender": "Female", "faceRectangle": { "left": 447, "top": 195, "width": 162, "height": 162 } }, { "age": 10, "gender": "Male", "faceRectangle": { "left": 355, "top": 87, "width": 143, "height": 143 } } ]
![Vision Analyze Family Phot Face](./Images/family_photo_face.png) | [ { "age": 11, "gender": "Male", "faceRectangle": { "left": 113, "top": 314, "width": 222, "height": 222 } }, { "age": 11, "gender": "Female", "faceRectangle": { "left": 1200, "top": 632, "width": 215, "height": 215 } }, { "age": 41, "gender": "Male", "faceRectangle": { "left": 514, "top": 223, "width": 205, "height": 205 } }, { "age": 37, "gender": "Female", "faceRectangle": { "left": 1008, "top": 277, "width": 201, "height": 201 } } ]


## <a name="Domain-Specific">Contenido de un dominio específico</a>
Adicional al etiquetado y a la categorización de un nivel superior, la API de Computer Vision también soporta la especialización (o dominio específico) de la información. Información especializada puede ser implementada como un método autónomo o con la categorización de alto nivel. Funciona como un método para mejorar la taxonomía de las 86 categorías a través de la adición de modelos de dominio específicos.

Actualmente, la única información especializada soportada es el reconocimiento de celebridades y puntos de interés. Son refinamientos específicos para las categorías de personas y grupos de personas y puntos de interés en el mundo.

Hay dos opciones para el uso de los modelos de dominios específicos:

### <a name="option-one---scoped-analysis"></a>Opción Uno - Análisis del alcance
Analizando solo un modelo elegido, por llamada en un HTTP POST. Para esta opción, si conoces qué tipo de modelo quieres usar, especificas el nombre del modelo y sólo obtienes información relevante para ese modelo. Por ejemplo, puedes usar esta opción para usarlo en reconocimiento de celebridades. La respuesta contiene una lista potencial de celebridades, acompañada de su puntuación.

### <a name="option-two---enhanced-analysis"></a>Opción Dos - Análisis mejorado
Analiza para proporcionar detalles adicionales relacionados a las categorías de la taxonomía de las 86 categorías. Esta opción está disponible para el uso en aplicaciones donde los usuarios quieren obtener análisis genérico de imágenes adicional a los detalles de uno o más modelos de dominio específico. Cuando este método es llamado, el clasificador de la taxonomía de las 86 categorías es llamado primero. Si cualquiera de las categorías concuerda con los modelos conocidos/similares, el segundo paso es el clasificador de las llamadas. Por ejemplo, si 'details=all' o 'details' incluyen 'celebrities', el método llama al clasificador de celebridades después de que la clasificación de las 86 categorías es llamada. El resultado incluirá las etiquetas empezando por 'people_'.

## <a name="Descriptions">Generando descripciones</a>   
Los algoritmos de la API de Computer Vision analizan el contenido de una imagen. Este análisis forma la base de la 'descripción' mostrada en lenguaje legible formateado en oraciones completas. Esta descripción resume qué fue encontrado en la imagen. Los algoritmos de la API de Computer Vision generan varias descripciones basándose en los objetos identificados en la imagen. Estas descripciones son cada una evaluadas y un puntaje de confianza es generado. Se regresa una lista ordenando desde el confiable al menos confiable. Por ejemplo, en un bot se puede usar esta tecnología para generar la descripción de una imagen como [aquí](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-ImageCaption).  

### <a name="example-description-generation"></a>Ejemplo de generación de descripción
![B&W Buildings](./Images/bw_buildings.jpg) '
```json
 Json de respuesta

'description':{
   "captions":[
      {
         "type":"phrase",
         'text':'a black and white photo of a large city',
         'confidence':0.607638706850331
      }
   ]   
   "captions":[
      {
         "type":"phrase",
         'text':'a photo of a large city',
         'confidence':0.577256764264197
      }
   ]   
   "captions":[
      {
         "type":"phrase",
         'text':'a black and white photo of a city',
         'confidence':0.538493271791207
      }
   ]   
   'description':[
      "tags":{
         "outdoor",
         "city",
         "building",
         "photo",
         "large",
      }
   ]
}
```

## <a name="Color">Percibiendo esquemas de color</a>
El algoritmo de Computer Vision extrae colores de una imagen. Los colores son analizados en tres diferentes contextos: primer plano, fondo y todo. Están agrupados dentro de 12 colores de acento dominantes. Estos acentos son negro, azul, café, gris, verde, naranja, rosa, purpura, rojo, verde azulado, blanco y amarillo. Dependiendo de los colores en la imagen, los colores blanco y negro y los de acento pueden ser regresados en hexadecimal. 

Imagen                                                       | Primer Plano |Fondo| Colores
----------------------------------------------------------- | --------- | ------- | ------
![Outdoor Mountain](./Images/mountain_vista.jpg)            | Negro     | Negro   | Blanco
![Vision Analyze Flower](./Images/flower.jpg)               | Negro     | Blanco   | Blanco, Negro, Verde
![Vision Analyze Train Station](./Images/train_station.jpg) | Negro     | Negro   | Negro

### <a name="accent-color"></a>Color de acento
Color extraído de una imagen para representar el color más predominante al ojo a los usuarios por una mezcla de colores dominantes y saturación.

Imagen                                                       | Respuesta
----------------------------------------------------------- | ----
![Outdoor Mountain](./Images/mountain_vista.jpg)            | #BC6F0F
![Vision Analyze Flower](./Images/flower.jpg)               | #CAA501
![Vision Analyze Train Station](./Images/train_station.jpg) | #484B83


### <a name="black--white"></a>Black & White
Boolean flag that indicates whether an image is black&white or not.

Imagen                                                      | Respuesta
---------------------------------------------------------- | ----
![Vision Analyze Building](./Images/bw_buildings.jpg)      | Verdadero
![Vision Analyze House Yard](./Images/house_yard.jpg)      | Falso

## <a name="Adult">Marcando contenido sólo para adultos</a>
Entre las categorías visuales está el contenido para adultos y racismo, que permite la detección de contenido sólo para adultos y restringe la visualización de las imágenes que contienen contenido sexual. El filtro para adultos y racismo puede ser establecido en una escala para acomodar las preferencias del usuario.

## <a name="OCR">Reconocimiento Óptico de Caracteres (OCR)</a>
La tecnología OCR ('Optical Character Recognition' por sus siglas en inglés) detecta el texto contenido en una imagen y extrae el texto identificado en un stream legible. Puedes usar el resultado para búsqueda y otros propósitos tales como medicina, seguridad y bancarios. Automáticamente detecta el lenguaje. OCR ahorra tiempo y ofrece una posibilidad de tomar fotos de textos, en lugar de transcribir el texto.

OCR soporta 21 lenguajes. Estos lenguajes son: Chino Simplificado, Chino Tradicional, Checo, Danés, Holandés, Inglés, Finlandés, Francés, Alemán, Griego, Húngaro, Italiano, Japonés, Coreano, Noruego, Polaco, Portugués, Ruso, Español, Sueco y Turco.

Si es necesario, el OCR corrige la rotación del texto reconocido, en grados, alrededor del eje horizontal de la imagen. OCR proporciona las coordenadas de cada palabra como se muestra en la siguiente ilustración. 

![OCR Overview](./Images/vision-overview-ocr.png) Requisitos para OCR:
- El tamaño de la imagen ingresada de ser entre 40 x 40 y 3200 x 3200 píxeles.
- La imagen no puede ser mayor a 10 megapixeles.

La imagen ingresada se puede rotar por cualquier multiplo de 90 grados más un pequeño ángulo de hasta 40 grados.

La presición del reconocimiento del texto depende de la calidad de la imagen. Una lectura incorrecta puede ser causada por las siguientes situaciones: 

- Imagenes borrosas.
- Texto a mano o letras cursivas.
- Estilos de fuente artísticos
- Tamaño de fuente pequeño.
- Fondos complejos, sombras, o luz sobre el texto o una mala perspectiva.
- Letra mayúscula grande o falta de letras maúsculas en el inicio de las palabras.
- Subíndice, superíndice o texto tachado.

Limitaciones: En fotos donde el texto es dominante, los falsos positivos pueden venir de palabras parcialmente reconocidas. En algunas fotos, especialmente las que no tienen texto, la precisión puede variar dependiendo del tipo de imagen.

## <a name="RecognizeText">Reconimiento de texto manuscrito</a>
Esta tecnología permite detectar y extraer texto escrito a mano de notas, cartas, ensayos, pizarrones, formatos, etc. Funciona con diferentes tipos de superficies y fondos, tales como papel blanco, notas adhesivas y pizarrones blancos. 

El reconocimiento de texto manuscrito ahorra tiempo y esfuerzo y puede ayudarte en la productividad permitiendo tomar imágenes de texto, en lugar de transcribirlo. También hace posible el digitalizar las notas. Esta digitalización permite implementar una búsqueda rápida y sencilla, así como reducción del uso de papel.

Requisitos de entrada:
- Formatos de imagen soportados: JPEG, PNG, and BMP.
- El tamaño de la imagen debe ser menor a 4 MB.
- Las dimensiones de la imagen deben ser de al menos 40 x 40 y hasta 3200 x 3200.

Nota: Esta tecnología está actualmente en pruebas y sólo está disponible para texto en inglés.

## <a name="Thumbnails">Generación de miniaturas</a>
Una miniatura es una representación pequeña de una imagen de tamaño completo. Múltiples dispositivos como teléfonos, tablets y PCs crearon la necesidad de para una experiencia de usuario diferente. Mediante el recorte inteligente, esta característica de la API de Computer Vision ayudan a resolver el problema.

Después de subir una imagen, una miniatura de alta calidad es generada y el algoritmo de la API de Computer Vision analiza los objetos de la imagen. Después recorta la imagen para ajustarse a los requisitos de la "Región de Interés" ('ROI' por sus siglas en inglés). La salida se muestra dentro de un marco especial como se muestra en la imagen. La miniatura generada puede ser mostrada usando una relación de aspecto que es diferente a la relación de aspecto original de la imagen para ajustarse a las necesidades del usuario.

El algoritmo de la miniatura funciona de la siguiente manera:

1. Elimina los elementos distractores de la imagen y reconoce el objeto principal, la "región de interés" (ROI).
2. Se recorta la imagen basándose en la identificación de la región de interés. 
3. Cambia la relación de aspecto para ajustarse a las dimensiones de las miniaturas requeridas.

![Thumbnails](./Images/thumbnail-demo.png)
