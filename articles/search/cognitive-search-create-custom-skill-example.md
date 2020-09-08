---
title: Ejemplo de una aptitud personalizada con Bing Entity Search API
titleSuffix: Azure Cognitive Search
description: Aquí se muestra cómo usar el servicio Bing Entity Search en una aptitud personalizada asignada a una canalización de indexación enriquecida con IA en Azure Cognitive Search.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 5755e14e53d359fd8b322939bf1325d21536d593
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020191"
---
# <a name="example-create-a-custom-skill-using-the-bing-entity-search-api"></a>Ejemplo: Creación de una aptitud personalizada con Bing Entity Search API

En este ejemplo, obtiene información sobre cómo crear una aptitud personalizada de la API web. Esta aptitud aceptará ubicaciones, personajes públicos y organizaciones, y devolverá sus descripciones. En el ejemplo se usa una [función de Azure](https://azure.microsoft.com/services/functions/) para encapsular [Bing Entity Search API](https://azure.microsoft.com/services/cognitive-services/bing-entity-search-api/) para que implemente la interfaz de la aptitud personalizada.

## <a name="prerequisites"></a>Prerrequisitos

+ Si no está familiarizado con la interfaz de entrada o salida que debe implementar una aptitud personalizada, lea el artículo sobre la [interfaz de aptitud personalizada](cognitive-search-custom-skill-interface.md).

+ [!INCLUDE [cognitive-services-bing-entity-search-signup-requirements](../../includes/cognitive-services-bing-entity-search-signup-requirements.md)]

+ Instale [Visual Studio 2019](https://www.visualstudio.com/vs/) o posterior, incluida la carga de trabajo de desarrollo de Azure.

## <a name="create-an-azure-function"></a>Creación de una Función de Azure

Aunque este ejemplo utiliza una función de Azure para hospedar una API web, esta acción no es necesaria.  Siempre que cumpla con los [requisitos de interfaz de una aptitud cognitiva](cognitive-search-custom-skill-interface.md), no importa el enfoque que quiera usar. Sin embargo, Azure Functions facilita la creación de una aptitud personalizada.

### <a name="create-a-function-app"></a>Creación de una aplicación de función

1. En Visual Studio, seleccione **Nuevo** > **Proyecto** en el menú Archivo.

1. En el cuadro de diálogo Nuevo proyecto, seleccione **Instalado**, expanda **Visual C#**  > **Nube**, seleccione **Azure Functions**, escriba un nombre para el proyecto y haga clic en **Aceptar**. El nombre de la aplicación de funciones debe ser válido como espacio de nombres de C#, por lo que no debe usar guiones bajos, guiones ni otros caracteres no alfanuméricos.

1. Seleccione **Azure Functions v2 (.NET Core)** . También puede hacerlo con la versión 1, pero el código que se escribe a continuación utiliza la plantilla de la versión 2.

1. Asegúrese de que el tipo es **Desencadenador HTTP**.

1. En la cuenta de almacenamiento, seleccione **Ninguno**, ya que no necesitará ningún almacenamiento para esta función.

1. Seleccione **Aceptar** para crear el proyecto de función y la función que se desencadena mediante HTTP.

### <a name="modify-the-code-to-call-the-bing-entity-search-service"></a>Modificación del código para llamar al servicio Bing Entity Search

Visual Studio crea un proyecto y dentro de él una clase que contiene código reutilizable para el tipo de función seleccionada. El atributo *FunctionName* en el método establece el nombre de la función. El atributo *HttpTrigger* especifica que una solicitud HTTP desencadena la función.

A continuación, reemplace todo el contenido del archivo *Function1.cs* con el siguiente código:

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;

namespace SampleSkills
{
    /// <summary>
    /// Sample custom skill that wraps the Bing entity search API to connect it with a 
    /// AI enrichment pipeline.
    /// </summary>
    public static class BingEntitySearch
    {
        #region Credentials
        // IMPORTANT: Make sure to enter your credential and to verify the API endpoint matches yours.
        static readonly string bingApiEndpoint = "https://api.cognitive.microsoft.com/bing/v7.0/entities/";
        static readonly string key = "<enter your api key here>";  
        #endregion

        #region Class used to deserialize the request
        private class InputRecord
        {
            public class InputRecordData
            {
                public string Name { get; set; }
            }

            public string RecordId { get; set; }
            public InputRecordData Data { get; set; }
        }

        private class WebApiRequest
        {
            public List<InputRecord> Values { get; set; }
        }
        #endregion

        #region Classes used to serialize the response

        private class OutputRecord
        {
            public class OutputRecordData
            {
                public string Name { get; set; } = "";
                public string Description { get; set; } = "";
                public string Source { get; set; } = "";
                public string SourceUrl { get; set; } = "";
                public string LicenseAttribution { get; set; } = "";
                public string LicenseUrl { get; set; } = "";
            }

            public class OutputRecordMessage
            {
                public string Message { get; set; }
            }

            public string RecordId { get; set; }
            public OutputRecordData Data { get; set; }
            public List<OutputRecordMessage> Errors { get; set; }
            public List<OutputRecordMessage> Warnings { get; set; }
        }

        private class WebApiResponse
        {
            public List<OutputRecord> Values { get; set; }
        }
        #endregion

        #region Classes used to interact with the Bing API
        private class BingResponse
        {
            public BingEntities Entities { get; set; }
        }
        private class BingEntities
        {
            public BingEntity[] Value { get; set; }
        }

        private class BingEntity
        {
            public class EntityPresentationinfo
            {
                public string[] EntityTypeHints { get; set; }
            }

            public class License
            {
                public string Url { get; set; }
            }

            public class ContractualRule
            {
                public string _type { get; set; }
                public License License { get; set; }
                public string LicenseNotice { get; set; }
                public string Text { get; set; }
                public string Url { get; set; }
            }

            public ContractualRule[] ContractualRules { get; set; }
            public string Description { get; set; }
            public string Name { get; set; }
            public EntityPresentationinfo EntityPresentationInfo { get; set; }
        }
        #endregion

        #region The Azure Function definition

        [FunctionName("EntitySearch")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("Entity Search function: C# HTTP trigger function processed a request.");

            var response = new WebApiResponse
            {
                Values = new List<OutputRecord>()
            };

            string requestBody = new StreamReader(req.Body).ReadToEnd();
            var data = JsonConvert.DeserializeObject<WebApiRequest>(requestBody);

            // Do some schema validation
            if (data == null)
            {
                return new BadRequestObjectResult("The request schema does not match expected schema.");
            }
            if (data.Values == null)
            {
                return new BadRequestObjectResult("The request schema does not match expected schema. Could not find values array.");
            }

            // Calculate the response for each value.
            foreach (var record in data.Values)
            {
                if (record == null || record.RecordId == null) continue;

                OutputRecord responseRecord = new OutputRecord
                {
                    RecordId = record.RecordId
                };

                try
                {
                    responseRecord.Data = GetEntityMetadata(record.Data.Name).Result;
                }
                catch (Exception e)
                {
                    // Something bad happened, log the issue.
                    var error = new OutputRecord.OutputRecordMessage
                    {
                        Message = e.Message
                    };

                    responseRecord.Errors = new List<OutputRecord.OutputRecordMessage>
                    {
                        error
                    };
                }
                finally
                {
                    response.Values.Add(responseRecord);
                }
            }

            return (ActionResult)new OkObjectResult(response);
        }

        #endregion

        #region Methods to call the Bing API
        /// <summary>
        /// Gets metadata for a particular entity based on its name using Bing Entity Search
        /// </summary>
        /// <param name="entityName">The name of the entity to extract data for.</param>
        /// <returns>Asynchronous task that returns entity data. </returns>
        private async static Task<OutputRecord.OutputRecordData> GetEntityMetadata(string entityName)
        {
            var uri = bingApiEndpoint + "?q=" + entityName + "&mkt=en-us&count=10&offset=0&safesearch=Moderate";
            var result = new OutputRecord.OutputRecordData();

            using (var client = new HttpClient())
            using (var request = new HttpRequestMessage {
                Method = HttpMethod.Get,
                RequestUri = new Uri(uri)
            })
            {
                request.Headers.Add("Ocp-Apim-Subscription-Key", key);

                HttpResponseMessage response = await client.SendAsync(request);
                string responseBody = await response?.Content?.ReadAsStringAsync();

                BingResponse bingResult = JsonConvert.DeserializeObject<BingResponse>(responseBody);
                if (bingResult != null)
                {
                    // In addition to the list of entities that could match the name, for simplicity let's return information
                    // for the top match as additional metadata at the root object.
                    return AddTopEntityMetadata(bingResult.Entities?.Value);
                }
            }

            return result;
        }

        private static OutputRecord.OutputRecordData AddTopEntityMetadata(BingEntity[] entities)
        {
            if (entities != null)
            {
                foreach (BingEntity entity in entities.Where(
                    entity => entity?.EntityPresentationInfo?.EntityTypeHints != null
                        && (entity.EntityPresentationInfo.EntityTypeHints[0] == "Person"
                            || entity.EntityPresentationInfo.EntityTypeHints[0] == "Organization"
                            || entity.EntityPresentationInfo.EntityTypeHints[0] == "Location")
                        && !String.IsNullOrEmpty(entity.Description)))
                {
                    var rootObject = new OutputRecord.OutputRecordData
                    {
                        Description = entity.Description,
                        Name = entity.Name
                    };

                    if (entity.ContractualRules != null)
                    {
                        foreach (var rule in entity.ContractualRules)
                        {
                            switch (rule._type)
                            {
                                case "ContractualRules/LicenseAttribution":
                                    rootObject.LicenseAttribution = rule.LicenseNotice;
                                    rootObject.LicenseUrl = rule.License.Url;
                                    break;
                                case "ContractualRules/LinkAttribution":
                                    rootObject.Source = rule.Text;
                                    rootObject.SourceUrl = rule.Url;
                                    break;
                            }
                        }
                    }

                    return rootObject;
                }
            }

            return new OutputRecord.OutputRecordData();
        }
        #endregion
    }
}
```

Asegúrese de escribir su propio valor de *clave* en la constante `key`, según la clave que ha obtenido al registrarse en Bing Entity Search API.

En este ejemplo se incluye todo el código necesario en un solo archivo para mayor comodidad. Puede encontrar una versión algo más estructurada de esa misma aptitud en el [repositorio de aptitudes avanzadas](https://github.com/Azure-Samples/azure-search-power-skills/tree/master/Text/BingEntitySearch).

Por supuesto, puede cambiar el nombre del archivo de `Function1.cs` a `BingEntitySearch.cs`.

## <a name="test-the-function-from-visual-studio"></a>Pruebe la función desde Visual Studio

Presione **F5** para ejecutar el programa y probar los comportamientos de la función. En este caso, usaremos la siguiente función para buscar dos entidades. Use Postman o Fiddler para emitir una llamada como la que se muestra a continuación:

```http
POST https://localhost:7071/api/EntitySearch
```

### <a name="request-body"></a>Cuerpo de la solicitud
```json
{
    "values": [
        {
            "recordId": "e1",
            "data":
            {
                "name":  "Pablo Picasso"
            }
        },
        {
            "recordId": "e2",
            "data":
            {
                "name":  "Microsoft"
            }
        }
    ]
}
```

### <a name="response"></a>Response
Debería ver una respuesta similar a la del siguiente ejemplo:

```json
{
    "values": [
        {
            "recordId": "e1",
            "data": {
                "name": "Pablo Picasso",
                "description": "Pablo Ruiz Picasso was a Spanish painter [...]",
                "source": "Wikipedia",
                "sourceUrl": "http://en.wikipedia.org/wiki/Pablo_Picasso",
                "licenseAttribution": "Text under CC-BY-SA license",
                "licenseUrl": "http://creativecommons.org/licenses/by-sa/3.0/"
            },
            "errors": null,
            "warnings": null
        },
        "..."
    ]
}
```

## <a name="publish-the-function-to-azure"></a>Publicar la función en Azure

Cuando esté satisfecho con el comportamiento de la función, puede publicarla.

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Publicar**. Elija **Crear nueva** > **Publicar**.

1. Si todavía no tiene conectado Visual Studio a la cuenta de Azure, seleccione **Agregar una cuenta...**

1. Siga las instrucciones que aparecen en pantalla. Se le pedirá que especifique un nombre único para el servicio de aplicaciones, la suscripción de Azure, el grupo de recursos, el plan de hospedaje y la cuenta de almacenamiento que quiera usar. Puede crear un nuevo grupo de recursos, un nuevo plan de hospedaje y una cuenta de almacenamiento si aún no los tiene. Cuando haya terminado, seleccione **Crear**.

1. Una vez completada la implementación, fíjese en la dirección URL del sitio. Es la dirección de la aplicación de función en Azure. 

1. En [Azure Portal](https://portal.azure.com), vaya al grupo de recursos y busque la función `EntitySearch` que ha publicado. En la sección **Administrar**, verá las claves de host. Seleccione el ícono **Copiar** de la clave de host *predeterminada*.  

## <a name="test-the-function-in-azure"></a>Pruebe la función en Azure

Ahora que tiene la clave de host predeterminada, pruebe la función de la siguiente manera:

```http
POST https://[your-entity-search-app-name].azurewebsites.net/api/EntitySearch?code=[enter default host key here]
```

### <a name="request-body"></a>Cuerpo de la solicitud
```json
{
    "values": [
        {
            "recordId": "e1",
            "data":
            {
                "name":  "Pablo Picasso"
            }
        },
        {
            "recordId": "e2",
            "data":
            {
                "name":  "Microsoft"
            }
        }
    ]
}
```

Este ejemplo debería producir el mismo resultado que vio anteriormente al ejecutar la función en el entorno local.

## <a name="connect-to-your-pipeline"></a>Conéctese a la canalización
Ahora que tiene una nueva aptitud personalizada, puede agregarla al conjunto de aptitudes. En el ejemplo siguiente se muestra cómo llamar a la aptitud para agregar descripciones a las organizaciones del documento (esto podría ampliarse para que también funcione con ubicaciones y personas). Reemplace `[your-entity-search-app-name]` por el nombre de la aplicación.

```json
{
    "skills": [
      "[... your existing skills remain here]",  
      {
        "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
        "description": "Our new Bing entity search custom skill",
        "uri": "https://[your-entity-search-app-name].azurewebsites.net/api/EntitySearch?code=[enter default host key here]",
          "context": "/document/merged_content/organizations/*",
          "inputs": [
            {
              "name": "name",
              "source": "/document/merged_content/organizations/*"
            }
          ],
          "outputs": [
            {
              "name": "description",
              "targetName": "description"
            }
          ]
      }
  ]
}
```

En este caso, contamos con la [aptitud de reconocimiento de entidades](cognitive-search-skill-entity-recognition.md) integrada que debe estar presente en el conjunto de aptitudes y haber completado el documento con la lista de organizaciones. Como referencia, esta es una configuración de aptitudes de extracción de entidades que sería suficiente para generar los datos que necesitamos:

```json
{
    "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
    "name": "#1",
    "description": "Organization name extraction",
    "context": "/document/merged_content",
    "categories": [ "Organization" ],
    "defaultLanguageCode": "en",
    "inputs": [
        {
            "name": "text",
            "source": "/document/merged_content"
        },
        {
            "name": "languageCode",
            "source": "/document/language"
        }
    ],
    "outputs": [
        {
            "name": "organizations",
            "targetName": "organizations"
        }
    ]
},
```

## <a name="next-steps"></a>Pasos siguientes
Felicidades. Acaba de crear su primera aptitud personalizada. Si quiere agregar su propia funcionalidad personalizada, puede seguir el mismo patrón. Para más información, haga clic en los siguientes vínculos:

+ [Aptitudes avanzadas: un repositorio de aptitudes personalizadas](https://github.com/Azure-Samples/azure-search-power-skills)
+ [Incorporación de una aptitud personalizada a una canalización de enriquecimiento de inteligencia artificial](cognitive-search-custom-skill-interface.md)
+ [Definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md)
+ [Crear un conjunto de aptitudes (REST)](/rest/api/searchservice/create-skillset)
+ [Cómo asignar campos enriquecidos](cognitive-search-output-field-mapping.md)