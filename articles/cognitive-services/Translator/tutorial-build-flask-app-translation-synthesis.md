---
title: 'Tutorial: Compilación de una aplicación de Flask para traducir, sintetizar y analizar texto: Translator'
titleSuffix: Azure Cognitive Services
description: En este tutorial, compilará una aplicación web basada en Flask para traducir texto, realizar análisis de opinión y sintetizar texto traducido a voz.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: tutorial
ms.date: 05/26/2020
ms.author: swmachan
ms.custom: devx-track-python, devx-track-javascript
ms.openlocfilehash: 6a81e52b833a59f51f6961a0bd41d52b040050b2
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876893"
---
# <a name="tutorial-build-a-flask-app-with-azure-cognitive-services"></a>Tutorial: Compilación de una aplicación de Flask con Azure Cognitive Services

En este tutorial, compilará una aplicación web de Flask que usa Azure Cognitive Services para traducir texto, realizar análisis de opinión y sintetizar texto traducido a voz. Aunque se prestará especial atención al código de Python y las rutas de Flask que habilitan nuestra aplicación, también se verá el código HTML y JavaScript que conforma la aplicación. Si experimenta algún problema, utilice el botón de comentarios de la parte inferior para informarnos.

Este tutorial abarca lo siguiente:

> [!div class="checklist"]
> * Obtención de las claves de suscripción a Azure
> * Configuración del entorno de desarrollo e instalación de las dependencias
> * Creación de una aplicación de Flask
> * Uso de Translator para traducir texto
> * Uso de Text Analytics para analizar opiniones positivas o negativas de texto de entrada y traducciones
> * Uso de Speech Services para convertir texto traducido en voz sintetizada
> * Ejecución local de la aplicación de Flask

> [!TIP]
> Si prefiere ver directamente todo el código, en [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-Flask-App-Tutorial) se encuentra disponible el ejemplo completo junto con las instrucciones de compilación.

## <a name="what-is-flask"></a>¿Qué es Flask?

Flask es un micromarco para crear aplicaciones web. Esto significa que Flask proporciona las herramientas, bibliotecas y tecnologías necesarias para compilar una aplicación web. Dicha aplicación web puede ser un conjunto de páginas web, un blog, una wiki o incluso una aplicación de calendario basada en web o un sitio web comercial.

Si desea obtener información detallada después de este tutorial, consulte estos vínculos útiles:

* [Documentación de Flask](http://flask.pocoo.org/)
* [Guía de Flask para principiantes](https://codeburst.io/flask-for-dummies-a-beginners-guide-to-flask-part-uno-53aec6afc5b1)

## <a name="prerequisites"></a>Requisitos previos

Para este tutorial, se necesita el software y las claves de suscripción siguientes.

* [Python 3.5.2 o versiones posteriores](https://www.python.org/downloads/)
* [Herramientas de Git](https://git-scm.com/downloads)
* Un editor de texto o IDE, como [Visual Studio Code](https://code.visualstudio.com/) o [Atom](https://atom.io/)  
* [Chrome](https://www.google.com/chrome/browser/) o [Firefox](https://www.mozilla.org/firefox)
* Una clave de suscripción de **Translator** (no es obligatorio seleccionar una región)
* Una clave de suscripción de **Text Analytics** en la región **Oeste de EE. UU.**
* Una clave de suscripción de **Speech Services** en la región **Oeste de EE. UU.**

## <a name="create-an-account-and-subscribe-to-resources"></a>Creación de una cuenta y suscripción a recursos

Como se ha indicado anteriormente, se necesitarán tres claves de suscripción para este tutorial. Esto significa que necesita crear un recurso en su cuenta de Azure para:
* Traductor
* Text Analytics
* Speech Services

Use [Creación de una cuenta de Cognitive Services en Azure Portal](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) para obtener instrucciones paso a paso para crear recursos.

> [!IMPORTANT]
> Para este tutorial, cree los recursos en la región Oeste de EE. UU. Si usa una región distinta, deberá ajustar la URL base en cada uno de los archivos de Python.

## <a name="set-up-your-dev-environment"></a>Configuración del entorno de desarrollo

Antes de compilar la aplicación web de Flask, deberá crear un directorio de trabajo para el proyecto e instalar algunos paquetes de Python.

### <a name="create-a-working-directory"></a>Creación de un directorio de trabajo

1. Abra un terminal (macOS o Linux) o la línea de comandos (Windows). A continuación, cree un directorio de trabajo y los subdirectorios para el proyecto:  

   ```
   mkdir -p flask-cog-services/static/scripts && mkdir flask-cog-services/templates
   ```
2. Cambie al directorio de trabajo del proyecto:  

   ```
   cd flask-cog-services
   ```

### <a name="create-and-activate-your-virtual-environment-with-virtualenv"></a>Creación y activación de un entorno virtual con `virtualenv`

Ahora creará un entorno virtual para la aplicación de Flask con `virtualenv`. El uso de un entorno virtual garantiza que dispone de un entorno limpio desde el que trabajar.

1. En el directorio de trabajo, ejecute este comando para crear un entorno virtual: **macOS/Linux:**
   ```
   virtualenv venv --python=python3
   ```
   Se ha indicado explícitamente que el entorno virtual debe usar Python 3. Esto permite garantizar que los usuarios con varias instalaciones de Python usen la versión correcta.

   **CMD de Windows/Bash de Windows:**
   ```
   virtualenv venv
   ```
   Para simplificar las cosas, se denominará al entorno virtual venv.

2. Los comandos para activar el entorno virtual variarán según la plataforma o shell:   

   | Plataforma | Shell | Get-Help |
   |----------|-------|---------|
   | macOS/Linux | bash/zsh | `source venv/bin/activate` |
   | Windows | Bash | `source venv/Scripts/activate` |
   | | Línea de comandos | `venv\Scripts\activate.bat` |
   | | PowerShell | `venv\Scripts\Activate.ps1` |

   Después de ejecutar este comando, la sesión de línea de comandos o terminal debe ir precedida por `venv`.

3. Puede desactivar la sesión en cualquier momento escribiendo lo siguiente en la línea de comandos o el terminal: `deactivate`.

> [!NOTE]
> Python cuenta con una amplia documentación para crear y administrar entornos virtuales; consulte [virtualenv](https://virtualenv.pypa.io/en/latest/).

### <a name="install-requests"></a>Instalación de Requests

Requests es un módulo popular que se usa para enviar solicitudes HTTP 1.1. No es preciso agregar manualmente cadenas de consulta a las direcciones URL ni formar y codificar los datos POST.

1. Para instalar Requests, ejecute lo siguiente:

   ```
   pip install requests
   ```

> [!NOTE]
> Si desea más información sobre Requests, consulte [Requests: HTTP for Humans](https://2.python-requests.org/en/master/) (Requests: HTTP para humanos).

### <a name="install-and-configure-flask"></a>Instalación y configuración de Flask

A continuación, se debe instalar Flask. Flask controla el enrutamiento de nuestra aplicación web y permite realizar llamadas de servidor a servidor que ocultan nuestras claves de suscripción al usuario final.

1. Para instalar Flask, ejecute lo siguiente:
   ```
   pip install Flask
   ```
   Asegúrese de que se haya instalado Flask. Ejecutar:
   ```
   flask --version
   ```
   La versión se debe imprimir en el terminal. Si no es así, algo no ha funcionado.

2. Para ejecutar la aplicación de Flask, puede usar el comando flask o el modificador -m de Python con Flask. Antes de eso, es necesario indicar al terminal con qué aplicación trabajar mediante la exportación de la variable de entorno `FLASK_APP`:

   **macOS/Linux**:
   ```
   export FLASK_APP=app.py
   ```

   **Windows**:
   ```
   set FLASK_APP=app.py
   ```

## <a name="create-your-flask-app"></a>Creación de la aplicación de Flask

En esta sección, va a crear una aplicación de Flask esencial que devuelve un archivo HTML cuando los usuarios llegan a la raíz de la aplicación. No dedique demasiado tiempo a analizar en detalle el código, ya que más adelante regresaremos sobre este archivo para actualizarlo.

### <a name="what-is-a-flask-route"></a>¿Qué es una ruta de Flask?

A continuación, se explica brevemente en qué consisten las "[rutas](http://flask.pocoo.org/docs/1.0/api/#flask.Flask.route)". El enrutamiento se usa para enlazar una dirección URL a una función específica. Flask usa elementos Decorator de ruta para registrar funciones en direcciones URL específicas. Por ejemplo, cuando un usuario navega a la raíz (`/`) de nuestra aplicación web, se representa `index.html`.  

```python
@app.route('/')
def index():
    return render_template('index.html')
```

Veamos otro ejemplo para dejarlo claro.

```python
@app.route('/about')
def about():
    return render_template('about.html')
```

Este código garantiza que cuando un usuario navega a `http://your-web-app.com/about`, se representa el archivo `about.html`.

Aunque estos ejemplos ilustran cómo representar páginas HTML para un usuario, también se pueden usar rutas para llamar a API al presionar un botón o para realizar una serie de acciones sin necesidad de salir de la página principal. Cuando cree rutas para traducción, opinión y síntesis de voz verá un ejemplo de esto en acción.

### <a name="get-started"></a>Introducción

1. Abra el proyecto en el entorno de desarrollo integrado y, después, cree un archivo denominado `app.py` en la raíz de su directorio de trabajo. A continuación, copie este código en `app.py` y guarde:

   ```python
   from flask import Flask, render_template, url_for, jsonify, request

   app = Flask(__name__)
   app.config['JSON_AS_ASCII'] = False

   @app.route('/')
   def index():
       return render_template('index.html')
   ```

   Este bloque de código indica a la aplicación que muestre `index.html` cada vez que un usuario navega a la raíz de la aplicación web (`/`).

2. A continuación, vamos a crear la interfaz de usuario de la aplicación web. Cree un archivo denominado `index.html` en el directorio `templates`. A continuación, copie este código en `templates/index.html`.

   ```html
   <!doctype html>
   <html lang="en">
     <head>
       <!-- Required metadata tags -->
       <meta charset="utf-8">
       <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
       <meta name="description" content="Translate and analyze text with Azure Cognitive Services.">
       <!-- Bootstrap CSS -->
       <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
       <title>Translate and analyze text with Azure Cognitive Services</title>
     </head>
     <body>
       <div class="container">
         <h1>Translate, synthesize, and analyze text with Azure</h1>
         <p>This simple web app uses Azure for text translation, text-to-speech conversion, and sentiment analysis of input text and translations. Learn more about <a href="https://docs.microsoft.com/azure/cognitive-services/">Azure Cognitive Services</a>.
        </p>
        <!-- HTML provided in the following sections goes here. -->

        <!-- End -->
       </div>

       <!-- Required Javascript for this tutorial -->
       <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
       <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
       <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q" crossorigin="anonymous"></script>
       <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl" crossorigin="anonymous"></script>
       <script type = "text/javascript" src ="static/scripts/main.js"></script>
     </body>
   </html>
   ```

3. Vamos a probar la aplicación de Flask. Desde el terminal, ejecute:

   ```
   flask run
   ```

4. Abra un explorador y vaya a la dirección URL proporcionada. Deberá ver la aplicación de página única. Presione **Ctrl + C** para terminar la aplicación.

## <a name="translate-text"></a>Traducir texto

Ahora que tiene una idea de cómo funciona una aplicación sencilla de Flask, realizará lo siguiente:

* Escribir código Python para llamar a Translator y devolver una respuesta
* Crear una ruta de Flask para llamar a su código Python
* Actualizar el código HTML con un área de entrada de texto y traducción, un selector de idioma y un botón de traducción
* Escribir código JavaScript que permita a los usuarios interactuar con la aplicación de Flask desde el código HTML

### <a name="call-the-translator"></a>Llamada a Translator

Lo primero que tiene que hacer es escribir una función para llamar a Translator. Esta función tendrá dos argumentos: `text_input` y `language_output`. Se llama a esta función cada vez que un usuario presiona el botón de traducción de la aplicación. El área de texto del código HTML se envía como `text_input`, y el valor de selección de idioma del código HTML se envía como `language_output`.

1. Para empezar, cree un archivo denominado `translate.py` en la raíz de su directorio de trabajo.
2. A continuación, agregue este código a `translate.py`. Esta función tiene dos argumentos: `text_input` y `language_output`.
   ```python
   import os, requests, uuid, json

   # Don't forget to replace with your Cog Services subscription key!
   # If you prefer to use environment variables, see Extra Credit for more info.
   subscription_key = 'YOUR_TRANSLATOR_TEXT_SUBSCRIPTION_KEY'
   
   # Don't forget to replace with your Cog Services location!
   # Our Flask route will supply two arguments: text_input and language_output.
   # When the translate text button is pressed in our Flask app, the Ajax request
   # will grab these values from our web app, and use them in the request.
   # See main.js for Ajax calls.
   def get_translation(text_input, language_output):
       base_url = 'https://api.cognitive.microsofttranslator.com'
       path = '/translate?api-version=3.0'
       params = '&to=' + language_output
       constructed_url = base_url + path + params

       headers = {
           'Ocp-Apim-Subscription-Key': subscription_key,
           'Ocp-Apim-Subscription-Region': 'location',
           'Content-type': 'application/json',
           'X-ClientTraceId': str(uuid.uuid4())
       }

       # You can pass more than one object in body.
       body = [{
           'text' : text_input
       }]
       response = requests.post(constructed_url, headers=headers, json=body)
       return response.json()
   ```
3. Agregue su clave de suscripción de Translator y guarde.

### <a name="add-a-route-to-apppy"></a>Agregar una ruta a `app.py`

A continuación, deberá crear una ruta en la aplicación de Flask que llame a `translate.py`. Se llamará a esta ruta cada vez que un usuario presione el botón de traducción en la aplicación.

Para esta aplicación, la ruta aceptará solicitudes `POST`. Esto se debe a que la función espera el texto que traducir y un idioma de salida para la traducción.

Flask proporciona funciones auxiliares para ayudarle a analizar y administrar cada solicitud. En el código proporcionado, `get_json()` devuelve los datos de la solicitud `POST` como JSON. A continuación, mediante `data['text']` y `data['to']`, los valores de texto e idioma de salida se pasan a la función `get_translation()` disponible desde `translate.py`. El último paso es devolver la respuesta como JSON, ya que necesitará mostrar estos datos en la aplicación web.

En las secciones siguientes, deberá repetir este proceso a medida que cree rutas para la síntesis de voz y el análisis de opinión.

1. Abra `app.py` y busque la instrucción de importación en la parte superior de `app.py` y agregue la siguiente línea:

   ```python
   import translate
   ```
   Ahora la aplicación de Flask puede usar el método disponible mediante `translate.py`.

2. Copie este código al final de `app.py` y guarde:

   ```python
   @app.route('/translate-text', methods=['POST'])
   def translate_text():
       data = request.get_json()
       text_input = data['text']
       translation_output = data['to']
       response = translate.get_translation(text_input, translation_output)
       return jsonify(response)
   ```

### <a name="update-indexhtml"></a>Actualizar `index.html`

Ahora que tiene una función para traducir texto y una ruta en la aplicación de Flask para llamarla, el siguiente paso es empezar a compilar el código HTML de la aplicación. El código HTML siguiente realiza varias cosas:

* Proporciona un área de texto donde los usuarios pueden escribir el texto que se traducirá.
* Incluye un selector de idioma.
* Incluye los elementos HTML para representar el idioma detectado y las puntuaciones de confianza que se devuelven durante la traducción.
* Proporciona un área de texto de solo lectura donde se muestra la salida de traducción.
* Incluye marcadores de posición para análisis de opinión y síntesis de voz que agregará a este archivo más adelante en el tutorial.

Ahora actualizaremos `index.html`.

1. Abra `index.html` y busque estos comentarios en el código:
   ```html
   <!-- HTML provided in the following sections goes here. -->

   <!-- End -->
   ```

2. Reemplace los comentarios del código por este bloque HTML:
   ```html
   <div class="row">
     <div class="col">
       <form>
         <!-- Enter text to translate. -->
         <div class="form-group">
           <label for="text-to-translate"><strong>Enter the text you'd like to translate:</strong></label>
           <textarea class="form-control" id="text-to-translate" rows="5"></textarea>
         </div>
         <!-- Select output language. -->
         <div class="form-group">
           <label for="select-language"><strong>Translate to:</strong></label>
           <select class="form-control" id="select-language">
             <option value="ar">Arabic</option>
             <option value="ca">Catalan</option>
             <option value="zh-Hans">Chinese (Simplified)</option>
             <option value="zh-Hant">Chinese (Traditional)</option>
             <option value="hr">Croatian</option>
             <option value="en">English</option>
             <option value="fr">French</option>
             <option value="de">German</option>
             <option value="el">Greek</option>
             <option value="he">Hebrew</option>
             <option value="hi">Hindi</option>
             <option value="it">Italian</option>
             <option value="ja">Japanese</option>
             <option value="ko">Korean</option>
             <option value="pt">Portuguese</option>
             <option value="ru">Russian</option>
             <option value="es">Spanish</option>
             <option value="th">Thai</option>
             <option value="tr">Turkish</option>
             <option value="vi">Vietnamese</option>
           </select>
         </div>
         <button type="submit" class="btn btn-primary mb-2" id="translate">Translate text</button></br>
         <div id="detected-language" style="display: none">
           <strong>Detected language:</strong> <span id="detected-language-result"></span><br />
           <strong>Detection confidence:</strong> <span id="confidence"></span><br /><br />
         </div>

         <!-- Start sentiment code-->

         <!-- End sentiment code -->

       </form>
     </div>
     <div class="col">
       <!-- Translated text returned by the Translate API is rendered here. -->
       <form>
         <div class="form-group" id="translator-text-response">
           <label for="translation-result"><strong>Translated text:</strong></label>
           <textarea readonly class="form-control" id="translation-result" rows="5"></textarea>
         </div>

         <!-- Start voice font selection code -->

         <!-- End voice font selection code -->

       </form>

       <!-- Add Speech Synthesis button and audio element -->

       <!-- End Speech Synthesis button -->

     </div>
   </div>
   ```

El siguiente paso es escribir código JavaScript. Este es el puente entre el código HTML y la ruta de Flask.

### <a name="create-mainjs"></a>Cree `main.js`  

El archivo `main.js` es el puente entre el código HTML y la ruta de Flask. La aplicación usará una combinación de jQuery, Ajax y XMLHttpRequest para representar el contenido y realizar solicitudes `POST` a las rutas de Flask.

En el código siguiente, el contenido del código HTML se usa para construir una solicitud a la ruta de Flask. En concreto, el contenido del área de texto y el selector de idioma se asigna a variables y, a continuación, se pasa en la solicitud a `translate-text`.

El código procesa una iteración en la respuesta y actualiza el código HTML con la traducción, el idioma detectado y la puntuación de confianza.

1. Desde el entorno de desarrollo integrado, cree un archivo denominado `main.js` en el directorio `static/scripts`.
2. Copie este código en `static/scripts/main.js`:
   ```javascript
   //Initiate jQuery on load.
   $(function() {
     //Translate text with flask route
     $("#translate").on("click", function(e) {
       e.preventDefault();
       var translateVal = document.getElementById("text-to-translate").value;
       var languageVal = document.getElementById("select-language").value;
       var translateRequest = { 'text': translateVal, 'to': languageVal }

       if (translateVal !== "") {
         $.ajax({
           url: '/translate-text',
           method: 'POST',
           headers: {
               'Content-Type':'application/json'
           },
           dataType: 'json',
           data: JSON.stringify(translateRequest),
           success: function(data) {
             for (var i = 0; i < data.length; i++) {
               document.getElementById("translation-result").textContent = data[i].translations[0].text;
               document.getElementById("detected-language-result").textContent = data[i].detectedLanguage.language;
               if (document.getElementById("detected-language-result").textContent !== ""){
                 document.getElementById("detected-language").style.display = "block";
               }
               document.getElementById("confidence").textContent = data[i].detectedLanguage.score;
             }
           }
         });
       };
     });
     // In the following sections, you'll add code for sentiment analysis and
     // speech synthesis here.
   })
   ```

### <a name="test-translation"></a>Probar la traducción

Ahora probaremos la función de traducción de la aplicación.

```
flask run
```

Vaya a la dirección del servidor proporcionada. Escriba texto en el área de entrada, seleccione un idioma y presione el botón de traducción. Deberá obtener una traducción. Si no funciona, asegúrese de que ha agregado su clave de suscripción.

> [!TIP]
> Si no se muestran los cambios realizados o la aplicación no funciona de la forma esperada, pruebe a borrar la caché o a abrir una ventana privada o de incógnito.

Presione **CTRL + C** para terminar la aplicación y, después, vaya a la sección siguiente.

## <a name="analyze-sentiment"></a>Análisis de opinión

[Text Analytics API](https://docs.microsoft.com/azure/cognitive-services/text-analytics/overview) puede usarse para realizar análisis de opinión, extraer frases clave del texto o detectar el idioma de origen. En esta aplicación, vamos a usar análisis de opinión para determinar si el texto proporcionado es negativo, neutral o positivo. La API devuelve una puntuación numérica entre 0 y 1. Las puntuaciones próximas a 1 indican una opinión positiva y las puntuaciones próximas a 0 indican una opinión negativa.

En esta sección, realizará lo siguiente:

* Escribir código Python para llamar a Text Analytics API para realizar análisis de opinión y devolver una respuesta
* Crear una ruta de Flask para llamar a su código Python
* Actualizar el código HTML con un área para las puntuaciones de opinión y un botón para realizar el análisis
* Escribir código JavaScript que permita a los usuarios interactuar con la aplicación de Flask desde el código HTML

### <a name="call-the-text-analytics-api"></a>Llamada a la API de Text Analytics

Ahora escribirá una función para llamar a Text Analytics API. Esta función tendrá cuatro argumentos: `input_text`, `input_language`, `output_text` y `output_language`. Se llama a esta función cada vez que un usuario presiona el botón de ejecución de análisis de opinión de la aplicación. Con cada solicitud se proporcionan los datos especificados por el usuario en el área de texto y el selector de idioma, así como el idioma detectado y la salida de traducción. El objeto de respuesta incluye las puntuaciones de opinión para el origen y la traducción. En las secciones siguientes, escribirá código JavaScript para analizar la respuesta y usarlo en la aplicación. Por ahora, nos centraremos en la llamada a Text Analytics API.

1. Cree un archivo denominado `sentiment.py` en la raíz de su directorio de trabajo.
2. A continuación, agregue este código a `sentiment.py`.
   ```python
   import os, requests, uuid, json

   # Don't forget to replace with your Cog Services subscription key!
   subscription_key = 'YOUR_TEXT_ANALYTICS_SUBSCRIPTION_KEY'

   # Our Flask route will supply four arguments: input_text, input_language,
   # output_text, output_language.
   # When the run sentiment analysis button is pressed in our Flask app,
   # the Ajax request will grab these values from our web app, and use them
   # in the request. See main.js for Ajax calls.

   def get_sentiment(input_text, input_language, output_text, output_language):
       base_url = 'https://westus.api.cognitive.microsoft.com/text/analytics'
       path = '/v2.0/sentiment'
       constructed_url = base_url + path

       headers = {
           'Ocp-Apim-Subscription-Key': subscription_key,
           'Content-type': 'application/json',
           'X-ClientTraceId': str(uuid.uuid4())
       }

       # You can pass more than one object in body.
       body = {
           'documents': [
               {
                   'language': input_language,
                   'id': '1',
                   'text': input_text
               },
               {
                   'language': output_language,
                   'id': '2',
                   'text': output_text
               }
           ]
       }
       response = requests.post(constructed_url, headers=headers, json=body)
       return response.json()
   ```
3. Agregue su clave de suscripción de Text Analytics y guarde.

### <a name="add-a-route-to-apppy"></a>Agregar una ruta a `app.py`

Ahora creará una ruta en la aplicación de Flask que llame a `sentiment.py`. Se llamará a esta ruta cada vez que un usuario presione el botón de ejecución de análisis de opinión de la aplicación. Al igual que la ruta para traducción, esta ruta aceptará solicitudes `POST`, ya que la función espera argumentos.

1. Abra `app.py`, busque la instrucción de importación en la parte superior de `app.py` y actualícela:

   ```python
   import translate, sentiment
   ```
   Ahora la aplicación de Flask puede usar el método disponible mediante `sentiment.py`.

2. Copie este código al final de `app.py` y guarde:
   ```python
   @app.route('/sentiment-analysis', methods=['POST'])
   def sentiment_analysis():
       data = request.get_json()
       input_text = data['inputText']
       input_lang = data['inputLanguage']
       output_text = data['outputText']
       output_lang =  data['outputLanguage']
       response = sentiment.get_sentiment(input_text, input_lang, output_text, output_lang)
       return jsonify(response)
   ```

### <a name="update-indexhtml"></a>Actualizar `index.html`

Ahora que tiene una función para ejecutar análisis de opinión y una ruta en la aplicación de Flask para llamarla, el siguiente paso es empezar a escribir el código HTML de la aplicación. El código HTML siguiente realiza varias cosas:

* Agrega un botón a la aplicación para ejecutar análisis de opinión
* Agrega un elemento que explica la puntuación de opiniones
* Agrega un elemento que muestra las puntuaciones de opinión

1. Abra `index.html` y busque estos comentarios en el código:
   ```html
   <!-- Start sentiment code-->

   <!-- End sentiment code -->
   ```

2. Reemplace los comentarios del código por este bloque HTML:
   ```html
   <button type="submit" class="btn btn-primary mb-2" id="sentiment-analysis">Run sentiment analysis</button></br>
   <div id="sentiment" style="display: none">
      <p>Sentiment scores are provided on a 1 point scale. The closer the sentiment score is to 1, indicates positive sentiment. The closer it is to 0, indicates negative sentiment.</p>
      <strong>Sentiment score for input:</strong> <span id="input-sentiment"></span><br />
      <strong>Sentiment score for translation:</strong> <span id="translation-sentiment"></span>
   </div>
   ```

### <a name="update-mainjs"></a>Actualizar `main.js`

En el código siguiente, el contenido del código HTML se usa para construir una solicitud a la ruta de Flask. En concreto, el contenido del área de texto y el selector de idioma se asigna a variables y, a continuación, se pasa en la solicitud a la ruta `sentiment-analysis`.

El código procesa una iteración en la respuesta y actualiza el código HTML con las puntuaciones de opinión.

1. Desde el entorno de desarrollo integrado, cree un archivo denominado `main.js` en el directorio `static`.

2. Copie este código en `static/scripts/main.js`:
   ```javascript
   //Run sentinment analysis on input and translation.
   $("#sentiment-analysis").on("click", function(e) {
     e.preventDefault();
     var inputText = document.getElementById("text-to-translate").value;
     var inputLanguage = document.getElementById("detected-language-result").innerHTML;
     var outputText = document.getElementById("translation-result").value;
     var outputLanguage = document.getElementById("select-language").value;

     var sentimentRequest = { "inputText": inputText, "inputLanguage": inputLanguage, "outputText": outputText,  "outputLanguage": outputLanguage };

     if (inputText !== "") {
       $.ajax({
         url: "/sentiment-analysis",
         method: "POST",
         headers: {
             "Content-Type":"application/json"
         },
         dataType: "json",
         data: JSON.stringify(sentimentRequest),
         success: function(data) {
           for (var i = 0; i < data.documents.length; i++) {
             if (typeof data.documents[i] !== "undefined"){
               if (data.documents[i].id === "1") {
                 document.getElementById("input-sentiment").textContent = data.documents[i].score;
               }
               if (data.documents[i].id === "2") {
                 document.getElementById("translation-sentiment").textContent = data.documents[i].score;
               }
             }
           }
           for (var i = 0; i < data.errors.length; i++) {
             if (typeof data.errors[i] !== "undefined"){
               if (data.errors[i].id === "1") {
                 document.getElementById("input-sentiment").textContent = data.errors[i].message;
               }
               if (data.errors[i].id === "2") {
                 document.getElementById("translation-sentiment").textContent = data.errors[i].message;
               }
             }
           }
           if (document.getElementById("input-sentiment").textContent !== '' && document.getElementById("translation-sentiment").textContent !== ""){
             document.getElementById("sentiment").style.display = "block";
           }
         }
       });
     }
   });
   // In the next section, you'll add code for speech synthesis here.
   ```

### <a name="test-sentiment-analysis"></a>Probar el análisis de opinión

Ahora probará el análisis de opinión en la aplicación.

```
flask run
```

Vaya a la dirección del servidor proporcionada. Escriba texto en el área de entrada, seleccione un idioma y presione el botón de traducción. Deberá obtener una traducción. A continuación, presione el botón de ejecución de análisis de opinión. Deberá ver dos puntuaciones. Si no funciona, asegúrese de que ha agregado su clave de suscripción.

> [!TIP]
> Si no se muestran los cambios realizados o la aplicación no funciona de la forma esperada, pruebe a borrar la caché o a abrir una ventana privada o de incógnito.

Presione **CTRL + C** para terminar la aplicación y, después, vaya a la sección siguiente.

## <a name="convert-text-to-speech"></a>Conversión de texto a voz

[Text-to-Speech API](https://docs.microsoft.com/azure/cognitive-services/speech-service/text-to-speech) permite a la aplicación convertir texto en voz sintetizada natural similar a la humana. El servicio admite voces neuronales, estándares y personalizadas. Nuestra aplicación de ejemplo usa varias de las voces disponibles; para ver una lista completa, consulte los [idiomas admitidos](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support#text-to-speech).

En esta sección, realizará lo siguiente:

* Escribir código Python para convertir texto en voz con Text-to-Speech API
* Crear una ruta de Flask para llamar a su código Python
* Actualizar el código HTML con un botón para convertir texto en voz y un elemento para la reproducción de audio
* Escribir código JavaScript que permita a los usuarios interactuar con la aplicación de Flask

### <a name="call-the-text-to-speech-api"></a>Llamar a Text-to-Speech API

Ahora escribirá una función para convertir texto en voz. Esta función tendrá dos argumentos: `input_text` y `voice_font`. Se llama a esta función cada vez que un usuario presiona el botón de convertir texto en voz de la aplicación. `input_text` es la salida de traducción devuelta por la llamada para traducir texto, `voice_font` es el valor del selector de fuente de voz en el código HTML.

1. Cree un archivo denominado `synthesize.py` en la raíz de su directorio de trabajo.

2. A continuación, agregue este código a `synthesize.py`.
   ```Python
   import os, requests, time
   from xml.etree import ElementTree

   class TextToSpeech(object):
       def __init__(self, input_text, voice_font):
           subscription_key = 'YOUR_SPEECH_SERVICES_SUBSCRIPTION_KEY'
           self.subscription_key = subscription_key
           self.input_text = input_text
           self.voice_font = voice_font
           self.timestr = time.strftime('%Y%m%d-%H%M')
           self.access_token = None

       # This function performs the token exchange.
       def get_token(self):
           fetch_token_url = 'https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken'
           headers = {
               'Ocp-Apim-Subscription-Key': self.subscription_key
           }
           response = requests.post(fetch_token_url, headers=headers)
           self.access_token = str(response.text)

       # This function calls the TTS endpoint with the access token.
       def save_audio(self):
           base_url = 'https://westus.tts.speech.microsoft.com/'
           path = 'cognitiveservices/v1'
           constructed_url = base_url + path
           headers = {
               'Authorization': 'Bearer ' + self.access_token,
               'Content-Type': 'application/ssml+xml',
               'X-Microsoft-OutputFormat': 'riff-24khz-16bit-mono-pcm',
               'User-Agent': 'YOUR_RESOURCE_NAME',
           }
           # Build the SSML request with ElementTree
           xml_body = ElementTree.Element('speak', version='1.0')
           xml_body.set('{http://www.w3.org/XML/1998/namespace}lang', 'en-us')
           voice = ElementTree.SubElement(xml_body, 'voice')
           voice.set('{http://www.w3.org/XML/1998/namespace}lang', 'en-US')
           voice.set('name', 'Microsoft Server Speech Text to Speech Voice {}'.format(self.voice_font))
           voice.text = self.input_text
           # The body must be encoded as UTF-8 to handle non-ascii characters.
           body = ElementTree.tostring(xml_body, encoding="utf-8")

           #Send the request
           response = requests.post(constructed_url, headers=headers, data=body)

           # Write the response as a wav file for playback. The file is located
           # in the same directory where this sample is run.
           return response.content
   ```
3. Agregue la clave de suscripción de Speech Services y guarde.

### <a name="add-a-route-to-apppy"></a>Agregar una ruta a `app.py`

Ahora creará una ruta en la aplicación de Flask que llame a `synthesize.py`. Se llamará a esta ruta cada vez que un usuario presione el botón de convertir texto en voz de la aplicación. Al igual que las rutas de traducción y análisis de opinión, esta ruta aceptará solicitudes `POST`, ya que la función espera dos argumentos: el texto que sintetizar y la fuente de voz para la reproducción.

1. Abra `app.py`, busque la instrucción de importación en la parte superior de `app.py` y actualícela:

   ```python
   import translate, sentiment, synthesize
   ```
   Ahora la aplicación de Flask puede usar el método disponible mediante `synthesize.py`.

2. Copie este código al final de `app.py` y guarde:

   ```Python
   @app.route('/text-to-speech', methods=['POST'])
   def text_to_speech():
       data = request.get_json()
       text_input = data['text']
       voice_font = data['voice']
       tts = synthesize.TextToSpeech(text_input, voice_font)
       tts.get_token()
       audio_response = tts.save_audio()
       return audio_response
   ```

### <a name="update-indexhtml"></a>Actualizar `index.html`

Ahora que tiene una función para convertir texto en voz y una ruta en la aplicación de Flask para llamarla, el siguiente paso es empezar a escribir el código HTML de la aplicación. El código HTML siguiente realiza varias cosas:

* Proporciona una lista desplegable de selección de voz
* Agrega un botón para convertir texto a voz
* Agrega un elemento de audio, que se usa para reproducir la voz sintetizada

1. Abra `index.html` y busque estos comentarios en el código:
   ```html
   <!-- Start voice font selection code -->

   <!-- End voice font selection code -->
   ```

2. Reemplace los comentarios del código por este bloque HTML:
   ```html
   <div class="form-group">
     <label for="select-voice"><strong>Select voice font:</strong></label>
     <select class="form-control" id="select-voice">
       <option value="(ar-SA, Naayf)">Arabic | Male | Naayf</option>
       <option value="(ca-ES, HerenaRUS)">Catalan | Female | HerenaRUS</option>
       <option value="(zh-CN, HuihuiRUS)">Chinese (Mainland) | Female | HuihuiRUS</option>
       <option value="(zh-CN, Kangkang, Apollo)">Chinese (Mainland) | Male | Kangkang, Apollo</option>
       <option value="(zh-HK, Tracy, Apollo)">Chinese (Hong Kong)| Female | Tracy, Apollo</option>
       <option value="(zh-HK, Danny, Apollo)">Chinese (Hong Kong) | Male | Danny, Apollo</option>
       <option value="(zh-TW, Yating, Apollo)">Chinese (Taiwan)| Female | Yating, Apollo</option>
       <option value="(zh-TW, Zhiwei, Apollo)">Chinese (Taiwan) | Male | Zhiwei, Apollo</option>
       <option value="(hr-HR, Matej)">Croatian | Male | Matej</option>
       <option value="(en-US, AriaRUS)">English (US) | Female | AriaRUS</option>
       <option value="(en-US, Guy24kRUS)">English (US) | Male | Guy24kRUS</option>
       <option value="(en-IE, Sean)">English (IE) | Male | Sean</option>
       <option value="(fr-FR, Julie, Apollo)">French | Female | Julie, Apollo</option>
       <option value="(fr-FR, HortenseRUS)">French | Female | Julie, HortenseRUS</option>
       <option value="(fr-FR, Paul, Apollo)">French | Male | Paul, Apollo</option>
       <option value="(de-DE, Hedda)">German | Female | Hedda</option>
       <option value="(de-DE, HeddaRUS)">German | Female | HeddaRUS</option>
       <option value="(de-DE, Stefan, Apollo)">German | Male | Apollo</option>
       <option value="(el-GR, Stefanos)">Greek | Male | Stefanos</option>
       <option value="(he-IL, Asaf)">Hebrew (Isreal) | Male | Asaf</option>
       <option value="(hi-IN, Kalpana, Apollo)">Hindi | Female | Kalpana, Apollo</option>
       <option value="(hi-IN, Hemant)">Hindi | Male | Hemant</option>
       <option value="(it-IT, LuciaRUS)">Italian | Female | LuciaRUS</option>
       <option value="(it-IT, Cosimo, Apollo)">Italian | Male | Cosimo, Apollo</option>
       <option value="(ja-JP, Ichiro, Apollo)">Japanese | Male | Ichiro</option>
       <option value="(ja-JP, HarukaRUS)">Japanese | Female | HarukaRUS</option>
       <option value="(ko-KR, HeamiRUS)">Korean | Female | Heami</option>
       <option value="(pt-BR, HeloisaRUS)">Portuguese (Brazil) | Female | HeloisaRUS</option>
       <option value="(pt-BR, Daniel, Apollo)">Portuguese (Brazil) | Male | Daniel, Apollo</option>
       <option value="(pt-PT, HeliaRUS)">Portuguese (Portugal) | Female | HeliaRUS</option>
       <option value="(ru-RU, Irina, Apollo)">Russian | Female | Irina, Apollo</option>
       <option value="(ru-RU, Pavel, Apollo)">Russian | Male | Pavel, Apollo</option>
       <option value="(ru-RU, EkaterinaRUS)">Russian | Female | EkaterinaRUS</option>
       <option value="(es-ES, Laura, Apollo)">Spanish | Female | Laura, Apollo</option>
       <option value="(es-ES, HelenaRUS)">Spanish | Female | HelenaRUS</option>
       <option value="(es-ES, Pablo, Apollo)">Spanish | Male | Pablo, Apollo</option>
       <option value="(th-TH, Pattara)">Thai | Male | Pattara</option>
       <option value="(tr-TR, SedaRUS)">Turkish | Female | SedaRUS</option>
       <option value="(vi-VN, An)">Vietnamese | Male | An</option>
     </select>
   </div>
   ```

3. A continuación, busque estos comentarios en el código:
   ```html
   <!-- Add Speech Synthesis button and audio element -->

   <!-- End Speech Synthesis button -->
   ```

4. Reemplace los comentarios del código por este bloque HTML:

```html
<button type="submit" class="btn btn-primary mb-2" id="text-to-speech">Convert text-to-speech</button>
<div id="audio-playback">
  <audio id="audio" controls>
    <source id="audio-source" type="audio/mpeg" />
  </audio>
</div>
```

5. Asegúrese de guardar los cambios.

### <a name="update-mainjs"></a>Actualizar `main.js`

En el código siguiente, el contenido del código HTML se usa para construir una solicitud a la ruta de Flask. En concreto, la traducción y la fuente de voz se asignan a variables y, a continuación, se pasan en la solicitud a la ruta `text-to-speech`.

El código procesa una iteración en la respuesta y actualiza el código HTML con las puntuaciones de opinión.

1. Desde el entorno de desarrollo integrado, cree un archivo denominado `main.js` en el directorio `static`.
2. Copie este código en `static/scripts/main.js`:
   ```javascript
   // Convert text-to-speech
   $("#text-to-speech").on("click", function(e) {
     e.preventDefault();
     var ttsInput = document.getElementById("translation-result").value;
     var ttsVoice = document.getElementById("select-voice").value;
     var ttsRequest = { 'text': ttsInput, 'voice': ttsVoice }

     var xhr = new XMLHttpRequest();
     xhr.open("post", "/text-to-speech", true);
     xhr.setRequestHeader("Content-Type", "application/json");
     xhr.responseType = "blob";
     xhr.onload = function(evt){
       if (xhr.status === 200) {
         audioBlob = new Blob([xhr.response], {type: "audio/mpeg"});
         audioURL = URL.createObjectURL(audioBlob);
         if (audioURL.length > 5){
           var audio = document.getElementById("audio");
           var source = document.getElementById("audio-source");
           source.src = audioURL;
           audio.load();
           audio.play();
         }else{
           console.log("An error occurred getting and playing the audio.")
         }
       }
     }
     xhr.send(JSON.stringify(ttsRequest));
   });
   // Code for automatic language selection goes here.
   ```
3. Ya casi ha terminado. Lo último que debe hacer es agregar código a `main.js` para seleccionar automáticamente una fuente de voz según el idioma seleccionado para la traducción. Agregue este bloque de código a `main.js`:
   ```javascript
   // Automatic voice font selection based on translation output.
   $('select[id="select-language"]').change(function(e) {
     if ($(this).val() == "ar"){
       document.getElementById("select-voice").value = "(ar-SA, Naayf)";
     }
     if ($(this).val() == "ca"){
       document.getElementById("select-voice").value = "(ca-ES, HerenaRUS)";
     }
     if ($(this).val() == "zh-Hans"){
       document.getElementById("select-voice").value = "(zh-HK, Tracy, Apollo)";
     }
     if ($(this).val() == "zh-Hant"){
       document.getElementById("select-voice").value = "(zh-HK, Tracy, Apollo)";
     }
     if ($(this).val() == "hr"){
       document.getElementById("select-voice").value = "(hr-HR, Matej)";
     }
     if ($(this).val() == "en"){
       document.getElementById("select-voice").value = "(en-US, Jessa24kRUS)";
     }
     if ($(this).val() == "fr"){
       document.getElementById("select-voice").value = "(fr-FR, HortenseRUS)";
     }
     if ($(this).val() == "de"){
       document.getElementById("select-voice").value = "(de-DE, HeddaRUS)";
     }
     if ($(this).val() == "el"){
       document.getElementById("select-voice").value = "(el-GR, Stefanos)";
     }
     if ($(this).val() == "he"){
       document.getElementById("select-voice").value = "(he-IL, Asaf)";
     }
     if ($(this).val() == "hi"){
       document.getElementById("select-voice").value = "(hi-IN, Kalpana, Apollo)";
     }
     if ($(this).val() == "it"){
       document.getElementById("select-voice").value = "(it-IT, LuciaRUS)";
     }
     if ($(this).val() == "ja"){
       document.getElementById("select-voice").value = "(ja-JP, HarukaRUS)";
     }
     if ($(this).val() == "ko"){
       document.getElementById("select-voice").value = "(ko-KR, HeamiRUS)";
     }
     if ($(this).val() == "pt"){
       document.getElementById("select-voice").value = "(pt-BR, HeloisaRUS)";
     }
     if ($(this).val() == "ru"){
       document.getElementById("select-voice").value = "(ru-RU, EkaterinaRUS)";
     }
     if ($(this).val() == "es"){
       document.getElementById("select-voice").value = "(es-ES, HelenaRUS)";
     }
     if ($(this).val() == "th"){
       document.getElementById("select-voice").value = "(th-TH, Pattara)";
     }
     if ($(this).val() == "tr"){
       document.getElementById("select-voice").value = "(tr-TR, SedaRUS)";
     }
     if ($(this).val() == "vi"){
       document.getElementById("select-voice").value = "(vi-VN, An)";
     }
   });
   ```

### <a name="test-your-app"></a>Prueba de la aplicación

Ahora probará la síntesis de voz en la aplicación.

```
flask run
```

Vaya a la dirección del servidor proporcionada. Escriba texto en el área de entrada, seleccione un idioma y presione el botón de traducción. Deberá obtener una traducción. A continuación, seleccione una voz y presione el botón de convertir texto en voz. La traducción debe reproducirse como voz sintetizada. Si no funciona, asegúrese de que ha agregado su clave de suscripción.

> [!TIP]
> Si no se muestran los cambios realizados o la aplicación no funciona de la forma esperada, pruebe a borrar la caché o a abrir una ventana privada o de incógnito.

Eso es todo. Ya tiene una aplicación funcional que realiza traducciones, analiza opiniones y genera voz sintetizada. Presione **Ctrl + C** para terminar la aplicación. No olvide consultar los demás recursos de [Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/).

## <a name="get-the-source-code"></a>Obtención del código fuente

El código fuente de este proyecto está disponible en [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-Flask-App-Tutorial).

## <a name="next-steps"></a>Pasos siguientes

* [Referencia de Translator](https://docs.microsoft.com/azure/cognitive-services/Translator/reference/v3-0-reference)
* [Referencia de Text Analytics API](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c7)
* [Referencia de Text-to-speech API](https://docs.microsoft.com/azure/cognitive-services/speech-service/rest-text-to-speech)
