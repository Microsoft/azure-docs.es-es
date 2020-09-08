---
title: 'Long Audio API (versión preliminar): servicio de voz'
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo se ha diseñado Long Audio API para la síntesis asincrónica de texto de formato largo a voz.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/11/2020
ms.author: trbye
ms.openlocfilehash: be38d3e78108a15c9f7875a15156e0eeba5a6211
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88167766"
---
# <a name="long-audio-api-preview"></a>Long Audio API (versión preliminar)

Long Audio API está diseñada para la síntesis asincrónica de texto de formato largo a voz (por ejemplo: audiolibros, artículos de noticias y documentos). Esta API no devuelve audio sintetizado en tiempo real, sino que, en su lugar, se espera que se sondeen las respuestas y se consuman las salidas a medida que estén disponibles desde el servicio. A diferencia de Text To Speech API, que usa el SDK de Voz, Long Audio API puede crear audio sintetizado de más de diez minutos, lo que resulta idóneo para los editores y las plataformas de contenido de audio.

Ventajas adicionales de Long Audio API:

* La voz sintetizada devuelta por el servicio usa las mejores voces neuronales.
* No es necesario implementar un punto de conexión de voz ya que no sintetiza las voces en ningún modo por lotes en tiempo real.

> [!NOTE]
> Long Audio API admite ahora [voces neuronales públicas](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support#neural-voices) y [voces neuronales personalizadas](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-custom-voice#custom-neural-voices).

## <a name="workflow"></a>Flujo de trabajo

Normalmente, cuando use Long Audio API, enviará un archivo de texto o archivos que se sintetizarán, sondeará el estado y, a continuación, si el estado es correcto, podrá descargar la salida de audio.

El diagrama a continuación proporciona una introducción general del flujo de trabajo.

![Diagrama de flujo de trabajo de Long Audio API](media/long-audio-api/long-audio-api-workflow.png)

## <a name="prepare-content-for-synthesis"></a>Preparación del contenido para la síntesis

Cuando prepare el archivo de texto, asegúrese de lo siguiente:

* Se trata de texto sin formato (.txt) o texto SSML (.txt).
* Está codificado como [UTF-8 con marca BOM](https://www.w3.org/International/questions/qa-utf8-bom.en#bom).
* Es un solo archivo, no un archivo ZIP.
* Contiene más de 400 caracteres para texto sin formato o 400 [caracteres facturables](https://docs.microsoft.com/azure/cognitive-services/speech-service/text-to-speech#pricing-note) para texto SSML y menos de 10 000 párrafos
  * En el caso de texto sin formato, cada párrafo se separa al presionar **Entrar**: consulte un [ejemplo de entrada de texto sin formato](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/en-US.txt).
  * En el caso de texto SSML, cada fragmento de SSML se considera un párrafo. Los fragmentos de SSML se deben separar con párrafos distintos: consulte un [ejemplo de entrada de texto SSML](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/SSMLTextInputSample.txt).
> [!NOTE]
> En el caso del chino (continental), chino (Hong Kong SAR), chino (Taiwán), japonés y coreano, una palabra se contará como dos caracteres. 

## <a name="python-example"></a>Ejemplo de Python

Esta sección contiene ejemplos de Python que muestran el uso básico de Long Audio API. Cree un nuevo proyecto de Python con su IDE o editor favorito. A continuación, copie este fragmento de código en un archivo llamado `voice_synthesis_client.py`.

```python
import argparse
import json
import ntpath
import urllib3
import requests
import time
from json import dumps, loads, JSONEncoder, JSONDecoder
import pickle

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
```

Estas bibliotecas se usan para analizar argumentos, construir la solicitud HTTP y llamar a la API REST de audios largos para convertir texto a voz.

### <a name="get-a-list-of-supported-voices"></a>Obtención de una lista de voces compatibles

Este código le permite obtener una lista completa de las voces de una región o punto de conexión en concreto que puede usar. Agregue el código a `voice_synthesis_client.py`:

```python
parser = argparse.ArgumentParser(description='Text-to-speech client tool to submit voice synthesis requests.')
parser.add_argument('--voices', action="store_true", default=False, help='print voice list')
parser.add_argument('-key', action="store", dest="key", required=True, help='the speech subscription key, like fg1f763i01d94768bda32u7a******** ')
parser.add_argument('-region', action="store", dest="region", required=True, help='the region information, could be centralindia, canadacentral or uksouth')
args = parser.parse_args()
baseAddress = 'https://%s.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0-beta1/' % args.region

def getVoices():
    response=requests.get(baseAddress+"voicesynthesis/voices", headers={"Ocp-Apim-Subscription-Key":args.key}, verify=False)
    voices = json.loads(response.text)
    return voices

if args.voices:
    voices = getVoices()
    print("There are %d voices available:" % len(voices))
    for voice in voices:
        print ("Name: %s, Description: %s, Id: %s, Locale: %s, Gender: %s, PublicVoice: %s, Created: %s" % (voice['name'], voice['description'], voice['id'], voice['locale'], voice['gender'], voice['isPublicVoice'], voice['created']))
```

Ejecute el script con el comando `python voice_synthesis_client.py --voices -key <your_key> -region <region>` y reemplace los siguientes valores:

* Reemplace `<your_key>` por la clave de suscripción del servicio Speech. Esta información está disponible en la pestaña de **información general** del recurso en [Azure Portal](https://aka.ms/azureportal).
* Reemplace `<region>` por la región donde se creó el recurso de Voz (por ejemplo: `eastus` o `westus`). Esta información está disponible en la pestaña de **información general** del recurso en [Azure Portal](https://aka.ms/azureportal).

Verá una salida similar a la siguiente:

```console
There are xx voices available:

Name: Microsoft Server Speech Text to Speech Voice (en-US, xxx), Description: xxx , Id: xxx, Locale: en-US, Gender: Male, PublicVoice: xxx, Created: 2019-07-22T09:38:14Z
Name: Microsoft Server Speech Text to Speech Voice (zh-CN, xxx), Description: xxx , Id: xxx, Locale: zh-CN, Gender: Female, PublicVoice: xxx, Created: 2019-08-26T04:55:39Z
```

Si el parámetro **PublicVoice** es **True**, se trata de una voz neuronal pública. De lo contrario, se trata de una voz neuronal personalizada.

### <a name="convert-text-to-speech"></a>Conversión de texto en voz

Prepare un archivo de texto de entrada en texto sin formato o texto SSML y, a continuación, agregue el código a `voice_synthesis_client.py`:

> [!NOTE]
> "concatenateResult" es un parámetro opcional. Si este parámetro no se establece, las salidas de audio se generarán por párrafo. También puede concatenar los audios en una salida si establece el parámetro. De forma predeterminada, la salida de audio se establece en riff-16khz-16bit-mono-pcm. Para más información sobre las salidas de audio compatibles, vea [Formatos de salida de audio](https://docs.microsoft.com/azure/cognitive-services/speech-service/long-audio-api#audio-output-formats).

```python
parser.add_argument('--submit', action="store_true", default=False, help='submit a synthesis request')
parser.add_argument('--concatenateResult', action="store_true", default=False, help='If concatenate result in a single wave file')
parser.add_argument('-file', action="store", dest="file", help='the input text script file path')
parser.add_argument('-voiceId', action="store", nargs='+', dest="voiceId", help='the id of the voice which used to synthesis')
parser.add_argument('-locale', action="store", dest="locale", help='the locale information like zh-CN/en-US')
parser.add_argument('-format', action="store", dest="format", default='riff-16khz-16bit-mono-pcm', help='the output audio format')

def submitSynthesis():
    modelList = args.voiceId
    data={'name': 'simple test', 'description': 'desc...', 'models': json.dumps(modelList), 'locale': args.locale, 'outputformat': args.format}
    if args.concatenateResult:
        properties={'ConcatenateResult': 'true'}
        data['properties'] = json.dumps(properties)
    if args.file is not None:
        scriptfilename=ntpath.basename(args.file)
        files = {'script': (scriptfilename, open(args.file, 'rb'), 'text/plain')}
    response = requests.post(baseAddress+"voicesynthesis", data, headers={"Ocp-Apim-Subscription-Key":args.key}, files=files, verify=False)
    if response.status_code == 202:
        location = response.headers['Location']
        id = location.split("/")[-1]
        print("Submit synthesis request successful")
        return id
    else:
        print("Submit synthesis request failed")
        print("response.status_code: %d" % response.status_code)
        print("response.text: %s" % response.text)
        return 0

def getSubmittedSynthesis(id):
    response=requests.get(baseAddress+"voicesynthesis/"+id, headers={"Ocp-Apim-Subscription-Key":args.key}, verify=False)
    synthesis = json.loads(response.text)
    return synthesis

if args.submit:
    id = submitSynthesis()
    if (id == 0):
        exit(1)

    while(1):
        print("\r\nChecking status")
        synthesis=getSubmittedSynthesis(id)
        if synthesis['status'] == "Succeeded":
            r = requests.get(synthesis['resultsUrl'])
            filename=id + ".zip"
            with open(filename, 'wb') as f:  
                f.write(r.content)
                print("Succeeded... Result file downloaded : " + filename)
            break
        elif synthesis['status'] == "Failed":
            print("Failed...")
            break
        elif synthesis['status'] == "Running":
            print("Running...")
        elif synthesis['status'] == "NotStarted":
            print("NotStarted...")
        time.sleep(10)
```

Ejecute el script con el comando `python voice_synthesis_client.py --submit -key <your_key> -region <region> -file <input> -locale <locale> -voiceId <voice_guid>` y reemplace los siguientes valores:

* Reemplace `<your_key>` por la clave de suscripción del servicio Speech. Esta información está disponible en la pestaña de **información general** del recurso en [Azure Portal](https://aka.ms/azureportal).
* Reemplace `<region>` por la región donde se creó el recurso de Voz (por ejemplo: `eastus` o `westus`). Esta información está disponible en la pestaña de **información general** del recurso en [Azure Portal](https://aka.ms/azureportal).
* Reemplace `<input>` por la ruta de acceso al archivo de texto que ha preparado para convertir de texto a voz.
* Reemplace `<locale>` por la configuración regional de salida deseada. Para obtener más información, consulte [Compatibilidad de idioma](language-support.md#neural-voices).
* Reemplace `<voice_guid>` por la voz de salida deseada. Use una de las voces devueltas por la llamada anterior al punto de conexión `/voicesynthesis/voices`.

Verá una salida similar a la siguiente:

```console
Submit synthesis request successful

Checking status
NotStarted...

Checking status
Running...

Checking status
Running...

Checking status
Succeeded... Result file downloaded : xxxx.zip
```

El resultado contiene el texto de entrada y los archivos de salida de audio generados por el servicio. Puede descargar estos archivos en un archivo ZIP.

> [!NOTE]
> Si tiene más de un archivo de entrada, tendrá que enviar varias solicitudes. Existen algunas limitaciones que tener en cuenta. 
> * El cliente puede enviar hasta **5** solicitudes al servidor por segundo para cada cuenta de suscripción de Azure. Si supera la limitación, el cliente obtendrá un código de error 429 (demasiadas solicitudes). Reduzca la cantidad de solicitudes por segundo.
> * El servidor puede ejecutar y poner en cola hasta **120** solicitudes para cada cuenta de suscripción de Azure. Si supera la limitación, el servidor devolverá un código de error 429 (demasiadas solicitudes). Espere y evite enviar una nueva solicitud hasta que se completen algunas solicitudes.

### <a name="remove-previous-requests"></a>Eliminación de solicitudes anteriores

El servicio mantendrá hasta **20 000** solicitudes para cada cuenta de suscripción de Azure. Si la cantidad de solicitudes supera esta limitación, elimine las solicitudes anteriores antes de realizar otras nuevas. Si no quita las solicitudes existentes, recibirá una notificación de error.

Agregue el siguiente código a `voice_synthesis_client.py`:

```python
parser.add_argument('--syntheses', action="store_true", default=False, help='print synthesis list')
parser.add_argument('--delete', action="store_true", default=False, help='delete a synthesis request')
parser.add_argument('-synthesisId', action="store", nargs='+', dest="synthesisId", help='the id of the voice synthesis which need to be deleted')

def getSubmittedSyntheses():
    response=requests.get(baseAddress+"voicesynthesis", headers={"Ocp-Apim-Subscription-Key":args.key}, verify=False)
    syntheses = json.loads(response.text)
    return syntheses

def deleteSynthesis(ids):
    for id in ids:
        print("delete voice synthesis %s " % id)
        response = requests.delete(baseAddress+"voicesynthesis/"+id, headers={"Ocp-Apim-Subscription-Key":args.key}, verify=False)
        if (response.status_code == 204):
            print("delete successful")
        else:
            print("delete failed, response.status_code: %d, response.text: %s " % (response.status_code, response.text))

if args.syntheses:
    synthese = getSubmittedSyntheses()
    print("There are %d synthesis requests submitted:" % len(synthese))
    for synthesis in synthese:
        print ("ID : %s , Name : %s, Status : %s " % (synthesis['id'], synthesis['name'], synthesis['status']))

if args.delete:
    deleteSynthesis(args.synthesisId)
```

Ejecute `python voice_synthesis_client.py --syntheses -key <your_key> -region <region>` para obtener una lista de las solicitudes de síntesis realizadas. Verá una salida similar a la siguiente:

```console
There are <number> synthesis requests submitted:
ID : xxx , Name : xxx, Status : Succeeded
ID : xxx , Name : xxx, Status : Running
ID : xxx , Name : xxx : Succeeded
```

Para eliminar una solicitud, ejecute `python voice_synthesis_client.py --delete -key <your_key> -region <Region> -synthesisId <synthesis_id>` y reemplace `<synthesis_id>` por un valor de identificador de solicitud devuelto por la solicitud anterior.

> [!NOTE]
> Las solicitudes con el estado "En ejecución" o "En espera" no se pueden quitar ni eliminar.

La solución completa `voice_synthesis_client.py` está disponible en [GitHub](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Python/voiceclient.py).

## <a name="http-status-codes"></a>Códigos de estado HTTP

En la tabla siguiente se detallan los mensajes y códigos de respuesta HTTP de la API REST.

| API | Código de estado HTTP | Descripción | Solución |
|-----|------------------|-------------|----------|
| Crear | 400 | La síntesis de voz no está habilitada en esta región. | Cambie la clave de suscripción de voz a una región admitida. |
|        | 400 | Solo es válida la suscripción **estándar** para esta región. | Cambie la clave de suscripción de voz al plan de tarifa "estándar". |
|        | 400 | Se superó el límite de solicitudes de 20 000 para la cuenta de Azure. Quite algunas solicitudes antes de enviar otras nuevas. | El servidor mantendrá hasta 20 000 solicitudes para cada cuenta de suscripción de Azure. Elimine algunas solicitudes antes de enviar otras nuevas. |
|        | 400 | Este modelo no se puede usar en la síntesis de voz: {modelID}. | Asegúrese de que el estado de {modelID} sea correcto. |
|        | 400 | La región de la solicitud no coincide con la región del modelo: {modelID}. | Asegúrese de que la región de {modelID} coincida con la región de la solicitud. |
|        | 400 | La síntesis de voz solo admite el archivo de texto en la codificación UTF-8 con el marcador de orden de bytes. | Asegúrese de que los archivos de entrada están en codificación UTF-8 con el marcador de orden de bytes. |
|        | 400 | Solo se permiten entradas SSML válidas en la solicitud de síntesis de voz. | Asegúrese de que las expresiones SSML de entrada sean correctas. |
|        | 400 | No se encuentra el nombre de voz {voiceName} en el archivo de entrada. | El nombre de voz SSML de entrada no está alineado con el id. de modelo. |
|        | 400 | El número de párrafos del archivo de entrada debe ser inferior a 10 000. | Asegúrese de que el número de párrafos del archivo sea inferior a 10 000. |
|        | 400 | El archivo de entrada debe tener más de 400 caracteres. | Asegúrese de que el archivo de entrada supere los 400 caracteres. |
|        | 404 | No se encuentra el modelo declarado en la definición de síntesis de voz: {modelID}. | Asegúrese de que {modelID} sea correcto. |
|        | 429 | Se superó el límite de síntesis de voz activa. Espere hasta que finalicen algunas solicitudes. | El servidor puede ejecutar y poner en cola hasta 120 solicitudes para cada cuenta de Azure. Espere y evite enviar nuevas solicitudes hasta que se completen algunas solicitudes. |
| All       | 429 | Hay demasiadas solicitudes. | El cliente puede enviar hasta 5 solicitudes al servidor por segundo para cada cuenta de Azure. Reduzca la cantidad de solicitudes por segundo. |
| Eliminar    | 400 | La tarea de síntesis de voz todavía está en uso. | Solo se pueden eliminar solicitudes **Completadas** o **Con errores**. |
| GetByID   | 404 | No se encuentra la entidad especificada. | Asegúrese de que el id. de síntesis sea correcto. |

## <a name="regions-and-endpoints"></a>Regiones y puntos de conexión

Long Audio API está disponible en varias regiones con puntos de conexión únicos.

| Region | Punto de conexión |
|--------|----------|
| Este de Australia | `https://australiaeast.customvoice.api.speech.microsoft.com` |
| Centro de Canadá | `https://canadacentral.customvoice.api.speech.microsoft.com` |
| Este de EE. UU. | `https://eastus.customvoice.api.speech.microsoft.com` |
| India central | `https://centralindia.customvoice.api.speech.microsoft.com` |
| Centro-sur de EE. UU. | `https://southcentralus.customvoice.api.speech.microsoft.com` |
| Sudeste de Asia | `https://southeastasia.customvoice.api.speech.microsoft.com` |
| Sur de Reino Unido | `https://uksouth.customvoice.api.speech.microsoft.com` |
| Oeste de Europa | `https://westeurope.customvoice.api.speech.microsoft.com` |
| Oeste de EE. UU. 2 | `https://westus2.customvoice.api.speech.microsoft.com` |

## <a name="audio-output-formats"></a>Formatos de salida de audio

Se admiten formatos de salida de audio flexibles. Puede generar salidas de audio por párrafo o concatenar las salidas de audio en una sola salida estableciendo el parámetro "concatenateResult". Long Audio API admite los siguientes formatos de salida de audio:

> [!NOTE]
> El formato de audio predeterminado es riff-16khz-16bit-mono-pcm.

* riff-8khz-16bit-mono-pcm
* riff-16khz-16bit-mono-pcm
* riff-24khz-16bit-mono-pcm
* riff-48khz-16bit-mono-pcm
* audio-16khz-32kbitrate-mono-mp3
* audio-16khz-64kbitrate-mono-mp3
* audio-16khz-128kbitrate-mono-mp3
* audio-24khz-48kbitrate-mono-mp3
* audio-24khz-96kbitrate-mono-mp3
* audio-24khz-160kbitrate-mono-mp3

## <a name="sample-code"></a>Código de ejemplo
Hay un ejemplo de código para Long Audio API disponible en GitHub.

* [Código de ejemplo: Python](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/CustomVoice-API-Samples/Python)
* [Código de ejemplo: C#](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/CustomVoice-API-Samples/CSharp)
* [Código de ejemplo: Java](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/)
