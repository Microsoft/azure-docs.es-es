---
author: PatrickFarley
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 11/14/2019
ms.author: pafarley
ms.openlocfilehash: 88be632e17912012618ab559f22f97487ad26c9c
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88723507"
---
## <a name="analyze-forms-for-key-value-pairs-and-tables"></a>Análisis de los formularios para obtener los pares clave-valor y las tablas

A continuación, utilizará el modelo recién entrenado para analizar un documento y extraerá de él tanto los pares clave-valor como las tablas. Llame a la API **[Analyze Form](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm)** mediante la ejecución del siguiente código en un nuevo script de Python. Antes de ejecutar el script, realice estos cambios:

1. Reemplace `<file path>` por la ruta de acceso de archivo del formulario (por ejemplo, C:\temp\file.pdf). Esta también puede ser una dirección URL a un archivo remoto. En este inicio rápido puede usar los archivos de la carpeta **Prueba** del [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2090451).
1. Reemplace `<model_id>` por el identificador de modelo que recibió en la sección anterior.
1. Reemplace `<endpoint>` por el punto de conexión que obtuvo con la clave de suscripción de Form Recognizer. Lo encontrará en la pestaña **Información general** del recurso Form Recognizer.
1. Reemplace `<file type>` por el tipo de archivo. Tipos admitidos: `application/pdf`, `image/jpeg`, `image/png`, `image/tiff`.
1. Reemplace `<subscription key>` por la clave de suscripción.

    # <a name="v20"></a>[v2.0](#tab/v2-0)
    ```python
    ########### Python Form Recognizer Async Analyze #############
    import json
    import time
    from requests import get, post
    
    # Endpoint URL
    endpoint = r"<endpoint>"
    apim_key = "<subsription key>"
    model_id = "<model_id>"
    post_url = endpoint + "/formrecognizer/v2.0/custom/models/%s/analyze" % model_id
    source = r"<file path>"
    params = {
        "includeTextDetails": True
    }
    
    headers = {
        # Request headers
        'Content-Type': '<file type>',
        'Ocp-Apim-Subscription-Key': apim_key,
    }
    with open(source, "rb") as f:
        data_bytes = f.read()
    
    try:
        resp = post(url = post_url, data = data_bytes, headers = headers, params = params)
        if resp.status_code != 202:
            print("POST analyze failed:\n%s" % json.dumps(resp.json()))
            quit()
        print("POST analyze succeeded:\n%s" % resp.headers)
        get_url = resp.headers["operation-location"]
    except Exception as e:
        print("POST analyze failed:\n%s" % str(e))
        quit() 
    ```    
    # <a name="v21-preview"></a>[Versión preliminar v2.1](#tab/v2-1)
    ```python
    ########### Python Form Recognizer Async Analyze #############
    import json
    import time
    from requests import get, post
    
    # Endpoint URL
    endpoint = r"<endpoint>"
    apim_key = "<subsription key>"
    model_id = "<model_id>"
    post_url = endpoint + "/formrecognizer/v2.1-preview.1/custom/models/%s/analyze" % model_id
    source = r"<file path>"
    params = {
        "includeTextDetails": True
    }
    
    headers = {
        # Request headers
        'Content-Type': '<file type>',
        'Ocp-Apim-Subscription-Key': apim_key,
    }
    with open(source, "rb") as f:
        data_bytes = f.read()
    
    try:
        resp = post(url = post_url, data = data_bytes, headers = headers, params = params)
        if resp.status_code != 202:
            print("POST analyze failed:\n%s" % json.dumps(resp.json()))
            quit()
        print("POST analyze succeeded:\n%s" % resp.headers)
        get_url = resp.headers["operation-location"]
    except Exception as e:
        print("POST analyze failed:\n%s" % str(e))
        quit() 
    ```    


    ---



1. Guarde el código en un archivo con una extensión .py. Por ejemplo, *form-recognizer-analyze.py*.
1. Abra una ventana de símbolo del sistema.
1. En el símbolo del sistema, utilice el comando `python` para ejecutar el ejemplo. Por ejemplo, `python form-recognizer-analyze.py`.

Cuando llame a la API **Analyze Form**, recibirá una respuesta `201 (Success)` con un encabezado **Operation-Location** (Operación-ubicación). El valor de este encabezado es un identificador que se usa para realizar el seguimiento de los resultados de la operación de análisis. El script anterior imprime el valor de este encabezado en la consola.

## <a name="get-the-analyze-results"></a>Obtención de los resultados del análisis

Agregue el código siguiente al final del script de Python. El código usa el valor de identificador de la llamada anterior en una nueva llamada API para recuperar los resultados del análisis. La operación de **análisis de formulario** es asincrónica, por lo que este script llama a la API a intervalos regulares hasta que los resultados están disponibles. Se recomienda un intervalo de uno o varios segundos.

```python 
n_tries = 15
n_try = 0
wait_sec = 5
max_wait_sec = 60
while n_try < n_tries:
    try:
        resp = get(url = get_url, headers = {"Ocp-Apim-Subscription-Key": apim_key})
        resp_json = resp.json()
        if resp.status_code != 200:
            print("GET analyze results failed:\n%s" % json.dumps(resp_json))
            quit()
        status = resp_json["status"]
        if status == "succeeded":
            print("Analysis succeeded:\n%s" % json.dumps(resp_json))
            quit()
        if status == "failed":
            print("Analysis failed:\n%s" % json.dumps(resp_json))
            quit()
        # Analysis still running. Wait and retry.
        time.sleep(wait_sec)
        n_try += 1
        wait_sec = min(2*wait_sec, max_wait_sec)     
    except Exception as e:
        msg = "GET analyze results failed:\n%s" % str(e)
        print(msg)
        quit()
print("Analyze operation did not complete within the allocated time.")
```
