---
title: 'ML Studio (clásico): Uso de los conjuntos de datos de ejemplo: Azure'
description: Descripciones de los conjuntos de datos que se usan en los modelos de ejemplo incluidos en Machine Learning Studio (clásico). Puede usar estos conjuntos de datos de ejemplo para los experimentos.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: sample
author: likebupt
ms.author: keli19
ms.custom: previous-author=heatherbshapiro, previous-ms.author=hshapiro
ms.date: 01/19/2018
ms.openlocfilehash: 2bb10b55aeeaa2d40d4ef4398dc06c9d4091f5ea
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432087"
---
# <a name="use-the-sample-datasets-in-azure-machine-learning-studio-classic"></a>Uso de los conjuntos de datos de ejemplo en Azure Machine Learning Studio (clásico)

**SE APLICA A:**  ![yes](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


[top]: #machine-learning-sample-datasets

Cuando se crea un área de trabajo en Azure Machine Learning Studio (clásico), de manera predeterminada se incluyen diversos conjuntos de datos y experimentos de ejemplo. Los modelos de ejemplo de la [Galería de Azure AI](https://gallery.azure.ai/) utilizan muchos de estos conjuntos de datos de ejemplo. Otros se incluyen como ejemplos de distintos tipos de datos que se usan normalmente en el aprendizaje automático.

Algunos de estos conjuntos de datos están disponibles en Azure Blob Storage. Para dichos conjuntos de datos, la tabla siguiente proporciona un vínculo directo. Puede usar estos conjuntos de datos en sus experimentos mediante el módulo [Import Data][import-data] (Importar datos).

El resto de estos conjuntos de datos de ejemplo están disponibles en el área de trabajo en **Conjuntos de datos guardados**. Puede encontrar esta opción en la paleta del módulo a la izquierda del lienzo del experimento en Machine Learning Studio (clásico).
Para usar cualquiera de estos conjuntos de datos en su propio experimento, arrástrelo a su lienzo de experimentos.

## <a name="datasets"></a>Conjuntos de datos

<table>

<tr>
  <th>Nombre del conjunto de datos</th>
  <th>Descripción del conjunto de datos</th>
</tr>

<tr>
  <td>Conjunto de datos de clasificación binaria de ingresos en el censo de adultos</td>
  <td>
Subconjunto de la base de datos del censo de 1994 en el que se utilizan adultos trabajadores a partir de 16 años con un índice de ingresos ajustado de > 100.
<p></p>
<b>Uso:</b> Clasificar personas utilizando valores demográficos para predecir si una persona gana más de 50 000 al año.
<p></p>
<b>Investigación relacionada:</b> Kohavi, R., Becker, B., (1996). Repositorio de aprendizaje automático de UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: Universidad de California, Facultad de Ciencias de la Computación y de la Información </td>
</tr>

<tr>
  <td>Conjunto de datos de códigos de aeropuerto</td>
  <td>
Códigos para aeropuertos de EE. UU.
<p></p>
Este conjunto de datos contiene una fila para cada aeropuerto de EE. UU., donde se indica el identificador y el nombre del aeropuerto, además de la ciudad y el estado en que se encuentra.
  </td>
</tr>

<tr>
  <td>Información sobre los precios de los automóviles (datos sin procesar)</td>
  <td>
Información acerca de automóviles por marca y modelo, incluido el precio, características como el número de cilindros y el consumo en relación a la distancia recorrida, así como una puntuación de riesgo para aseguradoras.
<p></p>
La puntuación de riesgo está asociada inicialmente a un precio automático. Después, se ajusta a un riesgo real en un proceso que los actuarios conocen como simbología. Un valor de +3 indica que es arriesgado, y un valor de -3 indica que probablemente es seguro.
<p></p>
<b>Uso:</b> Predecir la puntuación de riesgo por características utilizando una clasificación de regresión o de múltiples variantes. 
<p></p>
<b>Investigación relacionada:</b> Schlimmer, J.C. (1987). Repositorio de aprendizaje automático de UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: Universidad de California, Facultad de Ciencias de la Computación y de la Información </td>
</tr>

<tr>
  <td>Conjunto de datos UCI de alquiler de bicicletas</td>
  <td>
Conjunto de datos UCI Bike Rental basados en datos reales tomados de la empresa Capital Bikeshare, que mantiene una red de alquiler de bicicletas en Washington DC.
<p></p>
El conjunto de datos tiene una fila para cada hora de cada día en 2011 y 2012, totalizando 17.379 filas. El rango de alquileres de bicicletas por horas va de 1 a 977.

  </td>
</tr>

<tr>
  <td>Imagen RGB de Bill Gates</td>
  <td>
Archivo de imagen disponible públicamente convertido a datos CSV.
<p></p>
El código para convertir la imagen se proporciona en la página de detalles de modelos sobre la <strong>cuantificación de color mediante la agrupación en clústeres K-Means</strong>.
  </td>
</tr>

<tr>
  <td>Datos sobre donación de sangre</td>
  <td>
Subconjunto de datos de la base de datos de donantes de sangre del Centro de servicios de transfusiones de sangre de Hsin-Chu City (Taiwán).
<p></p>
Los datos de los donantes incluyen los meses transcurridos desde la última donación y la frecuencia, o el número total de donaciones, el tiempo transcurrido desde la última donación y la cantidad de sangre donada.
<p></p>
<b>Uso:</b> el objetivo es predecir mediante la clasificación si el donante donó sangre en marzo de 2007, donde 1 hace referencia a un donante durante el período objetivo y 0 a un no donante. 
<p></p>
<b>Investigación relacionada:</b> Yeh, I.C., (2008). Repositorio de aprendizaje automático de UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: Universidad de California, Facultad de Ciencias de la Computación y de la Información
<p></p>
Yeh, I-Cheng, Yang, King-Jang y Ting, Tao-Ming, "Knowledge discovery on RFM model using Bernoulli sequence" (Detección de conocimiento en el modelo de RFM mediante la secuencia de Bernoulli) Expert Systems with Applications, 2008, <a href="https://dx.doi.org/10.1016/j.eswa.2008.07.018">https://dx.doi.org/10.1016/j.eswa.2008.07.018</a>
  </td>
</tr>

<tr>
  <td>Datos sobre cáncer de mama</td>
  <td>
Uno de los tres conjuntos de datos relacionados con el cáncer facilitados por el Instituto de oncología que aparece frecuentemente en la literatura sobre aprendizaje automático. Combina información de diagnóstico con características de análisis de laboratorio de unas 300 muestras de tejido.
<p></p>
<b>Uso:</b> Clasificar el tipo de cáncer basándose en nueve atributos, algunos de los cuales son lineales y otros categóricos. 
<p></p>
<b>Investigación relacionada:</b> Wohlberg, W.H., Street, W.N., & Mangasarian, O.L. (1995). Repositorio de aprendizaje automático de UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: Universidad de California, Facultad de Ciencias de la Computación y de la Información </td>
</tr>

<tr>
  <td>Características de cáncer de mama <td>
El conjunto de datos contiene información de 102.000 regiones sospechosas (candidatas) de imágenes de rayos X, cada una de las cuales está descrita por 117 características. Las características son privadas y los creadores del conjunto de datos no revelan su significado (Siemens Healthcare). 
  </td>
</tr>

<tr>
  <td>Información sobre cáncer de mama</td>
  <td>
El conjunto de datos contiene información adicional para cada región sospechosa de imagen de rayos X. Cada ejemplo proporciona información (por ejemplo, etiqueta, identificador de paciente,coordenadas de la zona a tratar en relación con toda la imagen) sobre el número de fila correspondiente en el conjunto de datos Características de cáncer de mama. Cada paciente tiene una serie de ejemplos. Para los pacientes con cáncer, algunos ejemplos son positivos y otros son negativos. Para los pacientes que no tienen cáncer, todos los ejemplos son negativos. El conjunto de datos tiene 102 000 ejemplos. El conjunto de datos está sesgado, el 0,6% de los puntos es positivo y el resto es negativo. Siemens Healthcare proporcionó el conjunto de datos.
  </td>
</tr>

<tr>
  <td>Etiquetas de apetencia CRM compartidas</td>
  <td>
Etiquetas del concurso sobre la previsión de relaciones con los clientes de KDD Cup 2009 (<a href="http://www.sigkdd.org/site/2009/files/orange_small_train_appetency.labels">orange_small_train_appetency.labels</a>).
  </td>
</tr>

<tr>
  <td>Etiquetas de rotación de clientes de CRM compartidas</td>
  <td>
Etiquetas del concurso sobre la previsión de relaciones con los clientes de KDD Cup 2009 (<a href="http://www.sigkdd.org/site/2009/files/orange_small_train_churn.labels">orange_small_train_churn.labels</a>).
  </td>
</tr>

<tr>
  <td>Conjunto de datos CRM compartido</td>
  <td>
Estos datos están relacionados con el concurso sobre la previsión de relaciones con los clientes de KDD Cup 2009 (<a href="http://www.sigkdd.org/site/2009/files/orange_small_train.data.zip">orange_small_train.data.zip</a>).
<p></p>
El conjunto de datos contiene 50.000 clientes de la empresa de telecomunicaciones francesa Orange. Cada cliente tiene 230 características anónimas, 190 de las cuales son numéricas y 40, categóricas. Las características están muy dispersas.
  </td>
</tr>

<tr>
  <td>Etiquetas de mejora de ventas de CRM compartidas</td>
  <td>
Etiquetas del concurso sobre la previsión de relaciones con los clientes de KDD Cup 2009 (<a href="http://www.sigkdd.org/site/2009/files/orange_large_train_upselling.labels">orange_large_train_upselling.labels</a>).
  </td>
</tr>

<tr>
  <td>Datos de regresión de eficiencia energética</td>
  <td>
Colección de perfiles energéticos simulados, con base en 12 formas de edificios diferentes. Los edificios se diferencian por ocho características. Se incluyen la superficie vidriada, la distribución de la superficie vidriada y la orientación.
<p></p>
<b>Uso:</b> Usar la regresión o la clasificación para predecir la clasificación de eficiencia energética en función de una de las dos respuestas con valores reales. En el caso de una clasificación de varias clases, la variable de respuesta se redondea al entero más cercano. 
<p></p>
<b>Investigación relacionada:</b> Xifara, A. & Tsanas, A. (2012). Repositorio de aprendizaje automático de UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: Universidad de California, Facultad de Ciencias de la Computación y de la Información </td>
</tr>

<tr>
  <td>Datos de retrasos de vuelos</td>
  <td>
Datos de rendimiento sobre puntualidad en vuelos de pasajeros recopilados en TranStats por el Departamento  de Transporte de EE.UU. (<a href="https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time">On-Time</a>).
<p></p>
El conjunto de datos abarca el periodo comprendido entre abril y octubre de 2013. Antes de cargarlo en Azure Machine Learning Studio (clásico), el conjunto de datos se ha procesado de la manera siguiente:
<ul>
  <li>Se filtró el conjunto de datos para centrarse en los 70 aeropuertos con más tráfico del territorio continental de EE. UU.</li>
  <li>Los vuelos cancelados se etiquetaron como retrasados más de 15 minutos.</li>
  <li>Los vuelos desviados se quitaron de la muestra.</li>
  <li>Se seleccionaron las siguientes columnas: Year, Month, DayofMonth, DayOfWeek, Carrier, OriginAirportID, DestAirportID, CRSDepTime, DepDelay, DepDel15, CRSArrTime, ArrDelay, ArrDel15, Cancelled</li>
</ul>
</td>
</tr>

<tr>
  <td>Información sobre puntualidad de vuelos (datos sin procesar)</td>
  <td>
Registros de llegadas y salidas de aviones dentro de Estados Unidos desde octubre de 2011.
<p></p>
<b>Uso:</b> Predecir retrasos en los vuelos. 
<p></p>
<b>Investigación relacionada:</b> Del departamento de transporte de Estados Unidos <a href="https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time">https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time</a>.
  </td>
</tr>

<tr>
  <td>Datos de incendios forestales</td>
  <td>
Contiene los datos meteorológicos, como los índices de temperatura y humedad y la velocidad del viento. Los datos se extraen de un área del Nordeste de Portugal, combinados con registros de incendios forestales.
<p></p>
<b>Uso:</b> Se trata de una tarea de regresión difícil, cuyo objetivo es predecir el área quemada por incendios forestales. 
<p></p>
<b>Investigación relacionada:</b> Cortez, P. & Morais, A. (2008). Repositorio de aprendizaje automático de UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: Universidad de California, Facultad de Ciencias de la Computación y de la Información
<p></p>
[Cortez y Morais, 2007] P. Cortez y A. Morais. A Data Mining Approach to Predict Forest Fires using Meteorological Data. En J. Neves, M. F. Santos and J. Machado Eds., Nuevas tendencias en inteligencia artificial, Procedimientos de la 13ª EPIA 2007 - Conferencia en portugués sobre inteligencia artificial, diciembre, Guimarães, Portugal, páginas 512-523, 2007. APPIA, ISBN-13 978-989-95618-0-9. Disponible en: <a href="http://www.dsi.uminho.pt/~pcortez/fires.pdf">http://www.dsi.uminho.pt/~pcortez/fires.pdf</a>.
  </td>
</tr>

<tr>
  <td>Conjunto de datos UCI de tarjeta de crédito alemana</td>
  <td>
El conjunto de datos Statlog de UCI (tarjeta de crédito alemana) (<a href="https://archive.ics.uci.edu/ml/datasets/Statlog+(German+Credit+Data)">Statlog+German+Credit+Data</a>), con el archivo german.data.
<p></p>
El conjunto de datos clasifica a las personas, descritas por un conjunto de atributos, según si su riesgo de crédito es bajo o alto. Cada ejemplo representa a una persona. Hay 20 características, tanto numéricas como categóricas, y una etiqueta binaria (el valor del riesgo de crédito). Las partidas de riesgo de crédito alto tienen la etiqueta = 2, mientras que las partidas con riesgo de crédito bajo tienen la etiqueta = 1. El coste derivado de clasificar incorrectamente como alto un ejemplo de riesgo bajo es 1, mientras que, en el caso de clasificar incorrectamente como bajo un ejemplo de riesgo alto, el coste es 5.
  </td>
</tr>

<tr>
  <td>Títulos de películas en IMDB</td>
  <td>
El conjunto de datos contiene información sobre películas que se han valorado en tweets de Twitter: Identificador de la película en IMDB, nombre, género y año de producción. Hay 17.000 películas en el conjunto de datos. El conjunto de datos se introdujo en el documento "S. Dooms, T. De Pessemier y L. Martens. MovieTweetings: un conjunto de datos sobre valoración de películas recopilado de Twitter. Taller de micromecenazgo y cálculo humano para sistemas de recomendación, CrowdRec en RecSys 2013."
  </td>
</tr>

<tr>
  <td>Datos sobre iris de dos clases</td>
  <td>
Esta es quizá la base de datos más conocida entre la literatura de reconocimiento de modelos. El conjunto de datos es relativamente pequeño, con 50 ejemplos de cada una de las mediciones de pétalos de tres variedades de iris.
<p></p>
<b>Uso:</b> Predecir el tipo de iris a partir de las mediciones.  
<p></p>
<b>Investigación relacionada:</b> Fisher, R.A. (1988). Repositorio de aprendizaje automático de UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: Universidad de California, Facultad de Ciencias de la Computación y de la Información </td>
</tr>

<tr>
  <td>Tweets sobre películas</td>
  <td>
El conjunto de datos es una versión extendida del conjunto de datos Tweets sobre películas. El conjunto de datos incluye 170 000 valoraciones de películas extraídas de tweets bien estructurados en Twitter. Cada instancia representa un tweet y es una tupla: id. de usuario, id. de película IMDB, valoración, marca de tiempo, número de favoritos para ese tweet y número de retweets de ese tweet. A. Said, S. Dooms, B. Loni y D. Tikk proporcionaron el conjunto de datos para el Concurso de sistemas de recomendación 2014.
  </td>
</tr>

<tr>
  <td>Datos sobre consumo de combustible por distancia recorrida para varios automóviles</td>
  <td>
Este conjunto de datos es una versión ligeramente diferente del conjunto de datos ofrecido por la biblioteca StatLib de la Universidad Carnegie Mellon. El conjunto de datos se utilizó en la exposición de la asociación estadounidense de estadística de 1983.
<p></p>
Los datos muestran el consumo de combustible de varios automóviles en millas por galón. También incluye información como el número de cilindros, la cilindrada, los caballos de potencia, el peso total y la aceleración.
<p></p>
<b>Uso:</b> Predecir el ahorro de combustible en función de tres atributos discretos multivalor y cinco atributos continuos. 
<p></p>
<b>Investigación relacionada:</b> StatLib, Carnegie Mellon University, (1993). Repositorio de aprendizaje automático de UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: Universidad de California, Facultad de Ciencias de la Computación y de la Información </td>
</tr>

<tr>
  <td>Conjunto de datos de clasificación binaria sobre diabetes en indios pima</td>
  <td>
Subconjunto de datos de la base de datos del Instituto Nacional de la Diabetes y las Enfermedades Digestivas y Renales. El conjunto de datos se filtró para centrarse en pacientes femeninos con herencia de indios pima. Los datos incluyen información médica como niveles de glucosa e insulina y factores de estilo de vida.
<p></p>
<b>Uso:</b> Predecir si el sujeto tiene diabetes (clasificación binaria). 
<p></p>
<b>Investigación relacionada:</b> Sigillito, V. (1990). Repositorio de aprendizaje de automático de UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml"</a>. Irvine, CA: Universidad de California, Facultad de Ciencias de la Computación y de la Información </td>
</tr>

<tr>
  <td>Datos de clientes de restaurantes</td>
  <td>
Conjunto de metadatos sobre clientes que incluye información demográfica y preferencias.
<p></p>
<b>Uso:</b> Usar este conjunto de datos, en combinación con los otros dos conjuntos de datos de restaurantes, para entrenar y probar un sistema de recomendaciones. 
<p></p>
<b>Investigación relacionada:</b> Bache, K. y Lichman, M. (2013). Repositorio de aprendizaje automático de UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: Universidad de California, Facultad de Ciencias de la Computación y de la Información.
  </td>
</tr>

<tr>
  <td>Datos de características de restaurantes</td>
  <td>
Un conjunto de metadatos acerca de restaurantes y sus características, como el tipo de comida, el estilo de comedor y la ubicación.
<p></p>
<b>Uso:</b> Usar este conjunto de datos, en combinación con los otros dos conjuntos de datos de restaurantes, para entrenar y probar un sistema de recomendaciones. 
<p></p>
<b>Investigación relacionada:</b> Bache, K. y Lichman, M. (2013). Repositorio de aprendizaje automático de UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: Universidad de California, Facultad de Ciencias de la Computación y de la Información.
  </td>
</tr>

<tr>
  <td>Valoraciones de restaurantes</td>
  <td>
Contiene valoraciones que los usuarios realizan sobre restaurantes en una escala de 0 a 2.
<p></p>
<b>Uso:</b> Usar este conjunto de datos, en combinación con los otros dos conjuntos de datos de restaurantes, para entrenar y probar un sistema de recomendaciones. 
<p></p>
<b>Investigación relacionada:</b> Bache, K. y Lichman, M. (2013). Repositorio de aprendizaje automático de UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: Universidad de California, Facultad de Ciencias de la Computación y de la Información.
  </td>
</tr>

<tr>
  <td>Conjunto de datos de varias clases de recocido de acero</td>
  <td>
Este conjunto de datos contiene una serie de registros de pruebas de recocido de acero con los atributos físicos, como anchura, grosor o tipo (rollo, lámina, etc.) de los tipos de acero resultantes.
<p></p>
<b>Uso:</b> Predecir cualquiera de los dos atributos de clase numéricos: dureza o resistencia. También puede analizar las correlaciones entre atributos.
<p></p>
Los grados del acero siguen una norma establecida, definida por la SAE y otras organizaciones. Está buscando un "grado" determinado (la variable de clase) y desea conocer los valores requeridos. 
<p></p>
<b>Investigación relacionada:</b> Sterling, D. & Buntine, W. (NA). Repositorio de aprendizaje automático de UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: Universidad de California, Facultad de Ciencias de la Computación y de la Información
<p></p>
Aquí encontrará una guía útil para los grados del acero: <a href="https://www.steamforum.com/pictures/Outokumpu-steel-grades-properties-global-standards.pdf">https://www.steamforum.com/pictures/Outokumpu-steel-grades-properties-global-standards.pdf</a>
  </td>
</tr>

<tr>
  <td>Datos de telescopio</td>
  <td>
Registro de explosiones de partículas gamma de alta energía junto con ruido de fondo, ambos simulados utilizando un proceso de Monte Carlo.
<p></p>
El objetivo de la simulación era mejorar la precisión de los telescopios gamma Cherenkov atmosféricos terrestres, utilizando métodos estadísticos para diferenciar entre la señal deseada (duchas de radiación de Cherenkov) y el ruido de fondo (duchas hadrónicas iniciadas por los rayos cósmicos en la atmósfera superior).
<p></p>
Los datos se sometieron a un procesado previo para crear un grupo alargado con el eje central orientado hacia el centro de la cámara. Las características de esta elipse (llamada en ocasiones parámetros de Hillas) se encuentran entre los parámetros de imagen que se pueden utilizar para la discriminación.
<p></p>
<b>Uso:</b> Predecir si la imagen de una ducha representa señal o ruido de fondo.
<p></p>
<b>Notas:</b> La precisión de clasificación sencilla no es significativa para estos datos, ya que clasificar un evento de fondo como señal es peor que clasificar un evento de señal como fondo. Para comparar los diferentes clasificadores se debe utilizar el gráfico ROC. La probabilidad de aceptar un evento de fondo como señal tiene que estar por debajo de uno de los umbrales siguientes: 0,01, 0,02, 0,05, 0,1 o 0,2.
<p></p>
Además, tenga en cuenta que el número de eventos de fondo (h, para las duchas hadrónicas) está subestimado, mientras que en las medidas reales, la clase h o ruido representa la mayoría de eventos. 
<p></p>
<b>Investigación relacionada:</b> Bock, R.K. (1995). Repositorio de aprendizaje automático de UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: Universidad de California, Facultad de la Información </td>
</tr>

<tr>
  <td>Conjunto de datos del tiempo</td>
  <td>
Observaciones meteorológicas en tierra por hora de la NOAA (<a href="https://az754797.vo.msecnd.net/data/WeatherDataset.csv">datos combinados del 201304 al 201310</a>).
<p></p>
Los datos meteorológicos abarcan las observaciones de las estaciones meteorológicas de los aeropuertos, durante el período que comprende desde abril hasta octubre de 2013. Antes de cargarlo en Azure Machine Learning Studio (clásico), el conjunto de datos se ha procesado de la manera siguiente:
<ul>
  <li>Se asignaron los identificadores de las estaciones meteorológicas a los identificadores de aeropuerto correspondientes.</li>
  <li>Se excluyeron las estaciones meteorológicas no asociadas a los 70 aeropuertos con mayor tráfico.</li>
  <li>La columna Date se separó en columnas independientes con los valores Year, Month y Day.</li>
  <li>Se seleccionaron las siguientes columnas: AirportID, Year, Month, Day, Time, TimeZone, SkyCondition, Visibility, WeatherType, DryBulbFarenheit, DryBulbCelsius, WetBulbFarenheit, WetBulbCelsius, DewPointFarenheit, DewPointCelsius, RelativeHumidity, WindSpeed, WindDirection, ValueForWindCharacter, StationPressure, PressureTendency, PressureChange, SeaLevelPressure, RecordType, HourlyPrecip, Altimeter</li>
</ul>
  </td>
</tr>

<tr>
  <td>Conjunto de datos de SP 500 de Wikipedia</td>
  <td>
Los datos se han extraído de Wikipedia (<a href="https://www.wikipedia.org/">https://www.wikipedia.org/</a>) y se basan en artículos de cada empresa del índice S&P 500, almacenados como datos XML.
<p></p>
Antes de cargarlo en Azure Machine Learning Studio (clásico), el conjunto de datos se ha procesado de la manera siguiente:
<ul>
  <li>Se extrajo el contenido de texto para cada empresa específica.</li>
  <li>Se eliminó el formato wiki.</li>
  <li>Se eliminaron los caracteres no alfanuméricos</li>
  <li>Se convirtió todo el texto a minúscula.</li>
  <li>Se agregaron las categorías de empresas conocidas.</li>
</ul>
<p></p>
Tenga en cuenta que no se ha encontrado ningún artículo para algunas empresas, por lo que el número de registros es inferior a 500.
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/direct_marketing.csv">direct_marketing.csv</a></td>
  <td>
El conjunto de datos contiene datos de los clientes e indicaciones sobre sus respuestas a una campaña de correo directa. Cada fila representa a un cliente. El conjunto de datos contiene nueve características sobre datos demográficos del usuario y conducta en el pasado, y tres columnas de etiqueta (visita, conversión y gasto).  Visita es una columna binaria que indica que un cliente ha hecho una visita después de la campaña de marketing. Conversión indica que un cliente adquirió algún artículo. Gasto es el importe que se gastó.  Kevin Hillstrom proporcionó el conjunto de datos para el Reto de minería de datos y análisis de correo electrónico de MineThatData.
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/lyrl2004_tokens_test.csv">lyrl2004_tokens_test.csv</a></td>
  <td>
Características de los ejemplos de prueba del conjunto de datos de noticias de Reuters RCV1 V2. El conjunto de datos tiene 781 000 artículos nuevos junto con sus identificadores (primera columna del conjunto de datos). Los artículos están acortados, excluyen palabras reservadas y su contenido se reduce a la raíz de cada palabra. David proporcionó el conjunto de datos. D. Lewis.
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/lyrl2004_tokens_train.csv">lyrl2004_tokens_train.csv</a></td>
  <td>
Características de ejemplos de aprendizaje en el conjunto de datos de noticias de Reuters RCV1 V2. El conjunto de datos tiene 23 000 artículos nuevos junto con sus identificadores (primera columna del conjunto de datos). Los artículos están acortados, excluyen palabras reservadas y su contenido se reduce a la raíz de cada palabra. David proporcionó el conjunto de datos. D. Lewis.
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/network_intrusion_detection.csv">network_intrusion_detection.csv</a><br></td>
  <td>
Conjunto de datos de la competición de minería de datos y detección de conocimientos KDD Cup 1999 (<a href="https://kdd.ics.uci.edu/databases/kddcup99/kddcup99.html">kddcup99.html</a>).
<p></p>
El conjunto de datos se descargó y almacenó en Azure Blob Storage (<a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/network_intrusion_detection.csv">network_intrusion_detection.csv</a>) y comprende conjuntos de datos de entrenamiento y pruebas. El conjunto de datos de entrenamiento tiene aproximadamente 126.000 filas y 43 columnas, incluidas las etiquetas. Tres columnas forman parte de la información de etiquetas, y 40 columnas, compuestas de características numéricas y de cadena/categóricas, se encuentran disponibles para el entrenamiento del modelo. Los datos de prueba tienen aproximadamente 22.500 ejemplos de prueba (con las mismas 43 columnas en datos de entrenamiento).
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/rcv1-v2.topics.qrels.csv">rcv1-v2.topics.qrels.csv</a></td>
  <td>
Asignaciones de tema para artículos de prensa en el conjunto de datos de noticias de Reuters RCV1 V2. Un artículo de prensa se puede asignar a varios temas. El formato de cada fila es "&lt;nombre del tema&gt;&lt;identificador del documento&gt; 1". El conjunto de datos contiene 2,6 millones de asignaciones de tema. David proporcionó el conjunto de datos. D. Lewis.
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/student_performance.txt">student_performance.txt</a></td>
  <td>
Estos datos se han extraído del concurso de evaluación del rendimiento de los estudiantes KDD Cup 2010 (<a href="https://www.kdd.org/kdd-cup/view/kdd-cup-2010-student-performance-evaluation">evaluación del rendimiento de estudiantes</a>). Los datos utilizados se corresponden con el conjunto de formación de álgebra 2008 y 2009 (Stamper, J., Niculescu-Mizil, A., Ritter, S., Gordon, G.J., & Koedinger, K.R. (2010). Álgebra I 2008-2009. Conjunto de datos obtenidos de la minería de datos educativos de KDD Cup 2010. Lo encontrará en <a href="https://pslcdatashop.web.cmu.edu/KDDCup/downloads.jsp">downloads.jsp</a>.
<p></p>
El conjunto de datos se ha descargado y almacenado en Azure Blob Storage (<a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/student_performance.txt">student_performance.txt</a>) y contiene archivos de registro de un sistema de tutoría de estudiantes. Las características suministradas incluyen el identificador del problema y una breve descripción, el identificador del alumno, la marca de tiempo y cuántos intentos realizó el estudiante antes de solucionar el problema de la manera correcta. El conjunto de datos original contiene 8,9 millones de registros; este conjunto de datos se ha muestreado para las primeras 100.000 filas. El conjunto de datos tiene 23 columnas de varios tipos separadas por tabulaciones: numéricas, categóricas y con marcas de tiempo.
  </td>
</tr>

</table>

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Ponga en marcha sus experimentos con ejemplos](sample-experiments.md)

<!-- Module References -->
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
