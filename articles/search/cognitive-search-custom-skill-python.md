---
title: Ejemplo de aptitud personalizada (Python)
titleSuffix: Azure Cognitive Search
description: Pensado para que los desarrolladores de Python conozcan las herramientas y técnicas para crear una aptitud personalizada mediante Azure Functions y Visual Studio. Las aptitudes personalizadas contienen modelos o lógicas definidos por el usuario que puede agregar a una canalización de indexación enriquecida con inteligencia artificial en Azure Cognitive Search.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/15/2020
ms.custom: devx-track-python
ms.openlocfilehash: aac184b9dc7eae8afcf796376d2f7dca684b4367
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852793"
---
# <a name="example-create-a-custom-skill-using-python"></a>Ejemplo: Creación de una aptitud personalizada con Python

En este ejemplo de un conjunto de aptitudes de Azure Cognitive Search, aprenderá a crear una aptitud personalizada de API web con Python y Visual Studio Code. Este ejemplo usa una [función de Azure](https://azure.microsoft.com/services/functions/) que implementa la [interfaz de la aptitud personalizada](cognitive-search-custom-skill-interface.md).

La aptitud personalizada es sencilla por diseño (concatena dos cadenas) para que pueda centrarse en las herramientas y tecnologías que se usan para el desarrollo personalizado de aptitudes en Python. Una vez que crea correctamente una aptitud sencilla, puede atreverse con escenarios más complejos.

## <a name="prerequisites"></a>Prerrequisitos

+ Repase el artículo sobre la [interfaz de una aptitud personalizada](cognitive-search-custom-skill-interface.md) para ver una introducción sobre la interfaz de entrada/salida que debe implementar una aptitud personalizada.

+ Configure el entorno. Hemos seguido [este tutorial de principio a fin](https://docs.microsoft.com/azure/python/tutorial-vs-code-serverless-python-01) para configurar una instancia de Azure Function sin servidor con las extensiones de Visual Studio Code y Python. Este tutorial le guía a través de la instalación de las herramientas y los componentes siguientes: 

  + [Python 3.75](https://www.python.org/downloads/release/python-375/)
  + [Visual Studio Code](https://code.visualstudio.com/)
  + [Extensión de Python para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  + [Azure Functions Core Tools](https://docs.microsoft.com/azure/azure-functions/functions-run-local#v2)
  + [Extensión de Azure Functions para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)

## <a name="create-an-azure-function"></a>Creación de una Función de Azure

En este ejemplo se usa una función de Azure para mostrar el concepto de hospedaje de una API web, pero hay otros enfoques posibles. Siempre que cumpla con los [requisitos de interfaz de una aptitud cognitiva](cognitive-search-custom-skill-interface.md), no importa el enfoque que quiera usar. Sin embargo, Azure Functions facilita la creación de una aptitud personalizada.

### <a name="create-a-function-app"></a>Creación de una aplicación de función

La plantilla del proyecto de Azure Functions de Visual Studio Code crea un proyecto que se puede publicar en una aplicación de función en Azure. Una aplicación de función permite agrupar funciones como una unidad lógica para la administración, la implementación y el uso compartido de recursos.

1. En Visual Studio Code, presione F1 para abrir la paleta de comandos. En la paleta de comandos, busque y seleccione `Azure Functions: Create new project...`.

1. Elija una ubicación de directorio para el área de trabajo del proyecto y elija **Seleccionar**.

    > [!NOTE]
    > Estos pasos se han diseñado para completarse fuera de un área de trabajo. Por este motivo, no seleccione una carpeta de proyecto que forme parte de un área de trabajo.

1. Seleccione un lenguaje para el proyecto de la aplicación de funciones. Para este tutorial, seleccione **Python**.
1. Seleccione la versión 3.7.5 de Python ya que es la que admite Azure Functions.
1. Seleccione una plantilla para la primera función de su proyecto. Seleccione **Desencadenador HTTP** para crear una función desencadenada por HTTP en la nueva aplicación de funciones.
1. Proporcione un nombre de función. En este caso, vamos a usar **Concatenator** 
1. Seleccione **Función** como nivel de autorización. Esto significa que proporcionaremos una [clave de función](../azure-functions/functions-bindings-http-webhook-trigger.md#authorization-keys) para llamar al punto de conexión HTTP de la función. 
1. Seleccione cómo desea que se abra el proyecto. En este paso, seleccione **Add to workspace** (Agregar al área de trabajo) para crear la aplicación de funciones en el área de trabajo actual.

Visual Studio Code crea el proyecto de la aplicación de función en una nueva área de trabajo. Este proyecto contiene los archivos de configuración [host.json](../azure-functions/functions-host-json.md) y [local.settings.json](../azure-functions/functions-run-local.md#local-settings-file), además de los archivos de proyecto específicos del lenguaje. 

También se crea una nueva función desencadenada por HTTP en la carpeta **Concatenator** del proyecto de la aplicación de funciones. Dentro de ella habrá un archivo denominado "\_\_init__.py" con este contenido:

```py
import logging

import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello {name}!")
    else:
        return func.HttpResponse(
             "Please pass a name on the query string or in the request body",
             status_code=400
        )

```

A continuación, vamos a modificar el código para que siga la [interfaz de la aptitud personalizada](cognitive-search-custom-skill-interface.md)). Modifique el código con el siguiente contenido:

```py
import logging
import azure.functions as func
import json

def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    try:
        body = json.dumps(req.get_json())
    except ValueError:
        return func.HttpResponse(
             "Invalid body",
             status_code=400
        )
    
    if body:
        result = compose_response(body)
        return func.HttpResponse(result, mimetype="application/json")
    else:
        return func.HttpResponse(
             "Invalid body",
             status_code=400
        )


def compose_response(json_data):
    values = json.loads(json_data)['values']
    
    # Prepare the Output before the loop
    results = {}
    results["values"] = []
    
    for value in values:
        output_record = transform_value(value)
        if output_record != None:
            results["values"].append(output_record)
    return json.dumps(results, ensure_ascii=False)

## Perform an operation on a record
def transform_value(value):
    try:
        recordId = value['recordId']
    except AssertionError  as error:
        return None

    # Validate the inputs
    try:         
        assert ('data' in value), "'data' field is required."
        data = value['data']        
        assert ('text1' in data), "'text1' field is required in 'data' object."
        assert ('text2' in data), "'text2' field is required in 'data' object."
    except AssertionError  as error:
        return (
            {
            "recordId": recordId,
            "errors": [ { "message": "Error:" + error.args[0] }   ]       
            })

    try:                
        concatenated_string = value['data']['text1'] + " " + value['data']['text2']  
        # Here you could do something more interesting with the inputs

    except:
        return (
            {
            "recordId": recordId,
            "errors": [ { "message": "Could not complete operation for record." }   ]       
            })

    return ({
            "recordId": recordId,
            "data": {
                "text": concatenated_string
                    }
            })
```

El método **transform_value** realiza una operación en un único registro. Puede modificar el método para que satisfaga sus necesidades específicas. No olvide realizar todas las validaciones de entradas necesarias y devolver los errores y advertencias que se produjeron si la operación no se pudo completar para el registro.

### <a name="debug-your-code-locally"></a>Depuración local del código

Visual Studio Code facilita la depuración del código. Presione F5 o vaya al menú **Depurar** y seleccione **Iniciar depuración**.

Puede establecer todos los puntos de interrupción que desee en el código presionando "F9" en la línea de su interés.

Una vez iniciada la depuración, la función se ejecutará localmente. Puede usar una herramienta como Postman o Fiddler para emitir la solicitud a localhost. Anote la ubicación del punto de conexión local en la ventana de terminal. 

## <a name="publish-your-function"></a>Publicación de la función

Cuando esté satisfecho con el comportamiento de la función, puede publicarla.

1. En Visual Studio Code, presione F1 para abrir la paleta de comandos. En la paleta de comandos, busque y seleccione **Deploy to Function App...** (Implementar en la aplicación de funciones...). 

1. Seleccione la suscripción de Azure en la que desea implementar la aplicación.

1. Seleccione **+ Create New Function App in Azure** (Crear aplicación de funciones en Azure).

1. Escriba un nombre único global para la aplicación de funciones.

1. Seleccione la versión de Python (Python 3.7.x funciona para esta función).

1. Seleccione una ubicación para el nuevo recurso (por ejemplo, Oeste de EE. UU. 2).

En este momento, se crearán los recursos necesarios en la suscripción de Azure para hospedar la nueva función en Azure. Espere a que la implementación se complete. En la ventana de salida se mostrará el estado del proceso de implementación.

1. En [Azure Portal](https://portal.azure.com), vaya a **Todos los recursos** y busque la función que ha publicado por su nombre. Si la ha denominado **Concatenator**, seleccione el recurso.

1. Haga clic en el botón **</> Obtener la dirección URL de la función**. Esto le permitirá copiar la dirección URL para llamar a la función.

## <a name="test-the-function-in-azure"></a>Pruebe la función en Azure

Ahora que tiene la clave de host predeterminada, pruebe la función de la siguiente manera:

```http
POST [Function URL you copied above]
```

### <a name="request-body"></a>Cuerpo de la solicitud
```json
{
    "values": [
        {
            "recordId": "e1",
            "data":
            {
                "text1":  "Hello",
                "text2":  "World"
            }
        },
        {
            "recordId": "e2",
            "data": "This is an invalid input"
        }
    ]
}
```

Este ejemplo debería producir el mismo resultado que vio anteriormente al ejecutar la función en el entorno local.

## <a name="connect-to-your-pipeline"></a>Conéctese a la canalización

Ahora que tiene una nueva aptitud personalizada, puede agregarla al conjunto de aptitudes. En el ejemplo siguiente se muestra cómo llamar a la aptitud para concatenar el título y el autor del documento en un único campo al que llamaremos merged_title_author. Reemplace `[your-function-url-here]` por la dirección URL de la nueva función de Azure.

```json
{
    "skills": [
      "[... your existing skills remain here]",  
      {
        "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
        "description": "Our new search custom skill",
        "uri": "https://[your-function-url-here]",        
          "context": "/document/merged_content/organizations/*",
          "inputs": [
            {
              "name": "text1",
              "source": "/document/metadata_title"
            },
            {
              "name": "text2",
              "source": "/document/metadata_author"
            },
          ],
          "outputs": [
            {
              "name": "text",
              "targetName": "merged_title_author"
            }
          ]
      }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes
Felicidades. Acaba de crear su primera aptitud personalizada. Si quiere agregar su propia funcionalidad personalizada, puede seguir el mismo patrón. Para más información, haga clic en los siguientes vínculos:

+ [Aptitudes avanzadas: un repositorio de aptitudes personalizadas](https://github.com/Azure-Samples/azure-search-power-skills)
+ [Incorporación de una aptitud personalizada a una canalización de enriquecimiento de inteligencia artificial](cognitive-search-custom-skill-interface.md)
+ [Definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md)
+ [Crear un conjunto de aptitudes (REST)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)
+ [Cómo asignar campos enriquecidos](cognitive-search-output-field-mapping.md)
