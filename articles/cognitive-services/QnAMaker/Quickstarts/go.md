---
title: API de REST (V4) - Go - QnA Maker
titleSuffix: Azure Cognitive Services
description: Obtenga información y ejemplos de código de Go basados en REST que le ayuden a empezar a usar rápidamente Microsoft Translator Text API en Microsoft Cognitive Services en Azure.
services: cognitive-services
author: diberry
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: article
ms.date: 02/13/2019
ms.author: diberry
ms.custom: seodec18
ms.openlocfilehash: 76be371547bcb9daf0e10b62df62df5065cd6465
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65791459"
---
# <a name="how-to-use-the-qna-maker-rest-api-with-go"></a>Uso de la API REST de QnA Maker con Go 
<a name="HOLTop"></a>

En este artículo se muestra cómo usar [Microsoft QnA Maker API](../Overview/overview.md) con Go para hacer lo siguiente.

- [Crear una base de conocimiento](#Create)
- [Actualizar una base de conocimiento existente](#Update)
- [Obtener el estado de una solicitud para crear o actualizar una base de conocimiento](#Status)
- [Publicar una base de conocimiento existente](#Publish)
- [Reemplazar el contenido de una base de conocimiento existente](#Replace)
- [Descargar el contenido de una base de conocimiento](#GetQnA)
- [Obtener respuestas a una pregunta con una base de conocimiento](#GetAnswers)
- [Obtener información acerca de una base de conocimiento](#GetKB)
- [Obtener información acerca de todas las bases de conocimiento que pertenecen al usuario especificado](#GetKBsByUser)
- [Eliminar una base de conocimiento](#Delete)
- [Obtener las claves del punto de conexión actual](#GetKeys)
- [Regenerar las claves del punto de conexión actual](#PutKeys)
- [Obtener el conjunto actual de alteraciones de palabras que no distinguen mayúsculas de minúsculas.](#GetAlterations)
- [Reemplazar el conjunto actual de alteraciones de palabras que no distinguen mayúsculas de minúsculas.](#PutAlterations)

## <a name="prerequisites"></a>Requisitos previos

Se requiere [Go 1.10.1](https://golang.org/dl/) para ejecutar este código.

Debe tener una [cuenta de Cognitive Services API](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) con **Microsoft QnA Maker API**. Se requiere una clave de suscripción de pago del [panel de Azure](https://portal.azure.com/#create/Microsoft.CognitiveServices).

<a name="Create"></a>

## <a name="create-knowledge-base"></a>Creación de una base de conocimiento

El siguiente código crea una nueva base de conocimiento, con el método [Crear](https://go.microsoft.com/fwlink/?linkid=2092179).

1. Cree un nuevo proyecto de Go en su IDE favorito.
2. Agregue el código que se le ha proporcionado a continuación.
3. Reemplace el valor `key` por una clave de acceso válida para la suscripción.
4. Ejecute el programa.

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "strconv"
    "time"
)

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
var subscriptionKey string = "ENTER KEY HERE"

var host string = "https://westus.api.cognitive.microsoft.com"
var service string = "/qnamaker/v4.0"
var method string = "/knowledgebases/create"

type Response struct {
    Headers map[string][]string
    Body    string
}

func pretty_print(content string) string {
    var obj map[string]interface{}
    json.Unmarshal([]byte(content), &obj)
    result, _ := json.MarshalIndent(obj, "", "  ")
    return string(result)
}

func post(uri string, content string) Response {
    req, _ := http.NewRequest("POST", uri, bytes.NewBuffer([]byte(content)))
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
    req.Header.Add("Content-Type", "application/json")
    req.Header.Add("Content-Length", strconv.Itoa(len(content)))
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)

    return Response {response.Header, string(body)}
}

func get(uri string) Response {
    req, _ := http.NewRequest("GET", uri, nil)
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)

    return Response {response.Header, string(body)}
}

var req string = `{
  "name": "QnA Maker FAQ",
  "qnaList": [
    {
      "id": 0,
      "answer": "You can use our REST APIs to manage your Knowledge Base. See here for details: https://go.microsoft.com/fwlink/?linkid=2092179",
      "source": "Custom Editorial",
      "questions": [
        "How do I programmatically update my Knowledge Base?"
      ],
      "metadata": [
        {
          "name": "category",
          "value": "api"
        }
      ]
    }
  ],
  "urls": [
    "https://docs.microsoft.com/azure/cognitive-services/qnamaker/faqs",
    "https://docs.microsoft.com/bot-framework/resources-bot-framework-faq"
  ],
  "files": []
}`;

func create_kb(uri string, req string) (string, string) {
    fmt.Println("Calling " + uri + ".")
    result := post(uri, req)
    return result.Headers["Location"][0], result.Body
}

func check_status(uri string) (string, string) {
    fmt.Println("Calling " + uri + ".")
    result := get(uri)
    if retry, success := result.Headers["Retry-After"]; success {
        return retry[0], result.Body
    } else {
// If the response headers did not include a Retry-After value, default to 30 seconds.
        return "30", result.Body
    }
}

func main() {
    var uri = host + service + method
    operation, body := create_kb(uri, req)
    fmt.Printf(body + "\n")
    var done bool = false
    for done == false {
        uri := host + service + operation
        wait, status := check_status(uri)
        fmt.Println(status)
        var status_obj map[string]interface{}
        json.Unmarshal([]byte(status), &status_obj)
        state := status_obj["operationState"]
// If the operation isn't finished, wait and query again.
        if state == "Running" || state == "NotStarted" {
            fmt.Printf ("Waiting " + wait + " seconds...")
            sec, _ := strconv.Atoi(wait)
            time.Sleep (time.Duration(sec) * time.Second)
        } else {
            done = true
        }
    }
}
```

**Creación de la respuesta de una base de conocimiento**

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-04-13T01:52:30Z",
  "lastActionTimestamp": "2018-04-13T01:52:30Z",
  "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
  "operationId": "e88b5b23-e9ab-47fe-87dd-3affc2fb10f3"
}
...
{
  "operationState": "Running",
  "createdTimestamp": "2018-04-13T01:52:30Z",
  "lastActionTimestamp": "2018-04-13T01:52:30Z",
  "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
  "operationId": "e88b5b23-e9ab-47fe-87dd-3affc2fb10f3"
}
...
{
  "operationState": "Succeeded",
  "createdTimestamp": "2018-04-13T01:52:30Z",
  "lastActionTimestamp": "2018-04-13T01:52:46Z",
  "resourceLocation": "/knowledgebases/b0288f33-27b9-4258-a304-8b9f63427dad",
  "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
  "operationId": "e88b5b23-e9ab-47fe-87dd-3affc2fb10f3"
}
```

[Volver arriba](#HOLTop)

<a name="Update"></a>

## <a name="update-knowledge-base"></a>Actualización de una base de conocimiento

El siguiente código actualiza una base de conocimiento existente con el método [Actualizar](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase/update).

1. Cree un nuevo proyecto de Go en su IDE favorito.
2. Agregue el código que se le ha proporcionado a continuación.
3. Reemplace el valor `key` por una clave de acceso válida para la suscripción.
4. Ejecute el programa.

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "strconv"
    "time"
)

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
var subscriptionKey string = "ENTER KEY HERE"

// NOTE: Replace this with a valid knowledge base ID.
var kb string = "ENTER ID HERE";

var host string = "https://westus.api.cognitive.microsoft.com"
var service string = "/qnamaker/v4.0"
var method string = "/knowledgebases/"

type Response struct {
    Headers map[string][]string
    Body    string
}

func pretty_print(content string) string {
    var obj map[string]interface{}
    json.Unmarshal([]byte(content), &obj)
    result, _ := json.MarshalIndent(obj, "", "  ")
    return string(result)
}

func patch(uri string, content string) Response {
    req, _ := http.NewRequest("PATCH", uri, bytes.NewBuffer([]byte(content)))
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
    req.Header.Add("Content-Type", "application/json")
    req.Header.Add("Content-Length", strconv.Itoa(len(content)))
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)

    return Response {response.Header, string(body)}
}

func get(uri string) Response {
    req, _ := http.NewRequest("GET", uri, nil)
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)

    return Response {response.Header, string(body)}
}

var req string = `{
  'add': {
    'qnaList': [
      {
        'id': 1,
        'answer': 'You can change the default message if you use the QnAMakerDialog. See this for details: https://docs.botframework.com/azure-bot-service/templates/qnamaker/#navtitle',
        'source': 'Custom Editorial',
        'questions': [
          'How can I change the default message from QnA Maker?'
        ],
        'metadata': []
      }
    ],
    'urls': [
      'https://docs.microsoft.com/azure/cognitive-services/Emotion/FAQ'
    ]
  },
  'update' : {
    'name' : 'New KB Name'
  },
  'delete': {
    'ids': [
      0
    ]
  }
}`;

func update_kb (uri string, req string) (string, string) {
    fmt.Println("Calling " + uri + ".")
    result := patch(uri, req)
    return result.Headers["Location"][0], result.Body
}

func check_status(uri string) (string, string) {
    fmt.Println("Calling " + uri + ".")
    result := get(uri)
    if retry, success := result.Headers["Retry-After"]; success {
        return retry[0], result.Body
    } else {
// If the response headers did not include a Retry-After value, default to 30 seconds.
        return "30", result.Body
    }
}

func main() {
    var uri = host + service + method + kb
    operation, body := update_kb (uri, req)
    fmt.Printf(body + "\n")
    var done bool = false
    for done == false {
        uri := host + service + operation
        wait, status := check_status(uri)
        fmt.Println(status)
        var status_obj map[string]interface{}
        json.Unmarshal([]byte(status), &status_obj)
        state := status_obj["operationState"]
// If the operation isn't finished, wait and query again.
        if state == "Running" || state == "NotStarted" {
            fmt.Printf ("Waiting " + wait + " seconds...")
            sec, _ := strconv.Atoi(wait)
            time.Sleep (time.Duration(sec) * time.Second)
        } else {
            done = true
        }
    }
}
```

**Actualización de la respuesta de una base de conocimiento**

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-04-13T01:49:48Z",
  "lastActionTimestamp": "2018-04-13T01:49:48Z",
  "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
  "operationId": "5156f64e-e31d-4638-ad7c-a2bdd7f41658"
}
...
{
  "operationState": "Succeeded",
  "createdTimestamp": "2018-04-13T01:49:48Z",
  "lastActionTimestamp": "2018-04-13T01:49:50Z",
  "resourceLocation": "/knowledgebases/140a46f3-b248-4f1b-9349-614bfd6e5563",
  "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
  "operationId": "5156f64e-e31d-4638-ad7c-a2bdd7f41658"
}
Press any key to continue.
```

[Volver arriba](#HOLTop)

<a name="Status"></a>

## <a name="get-request-status"></a>Obtención del estado de la solicitud

Puede llamar al método [Operación](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/operations/getdetails) a fin de comprobar el estado de una solicitud para crear o actualizar una base de conocimiento. Para ver cómo se utiliza este método, consulte el código de ejemplo para el método [Crear](#Create) o [Actualizar](#Update).

[Volver arriba](#HOLTop)

<a name="Publish"></a>

## <a name="publish-knowledge-base"></a>Publicación de una base de conocimiento

El siguiente código publica una base de conocimiento existente con el método [Publicar](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase/publish).

1. Cree un nuevo proyecto de Go en su IDE favorito.
2. Agregue el código que se le ha proporcionado a continuación.
3. Reemplace el valor `key` por una clave de acceso válida para la suscripción.
4. Ejecute el programa.

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "strconv"
)

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
var subscriptionKey string = "ENTER KEY HERE"

// NOTE: Replace this with a valid knowledge base ID.
var kb string = "ENTER ID HERE";

var host string = "https://westus.api.cognitive.microsoft.com"
var service string = "/qnamaker/v4.0"
var method string = "/knowledgebases/"

func pretty_print(content string) string {
    var obj map[string]interface{}
    json.Unmarshal([]byte(content), &obj)
    result, _ := json.MarshalIndent(obj, "", "  ")
    return string(result)
}

func post(uri string, content string) string {
    req, _ := http.NewRequest("POST", uri, bytes.NewBuffer([]byte(content)))
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
    req.Header.Add("Content-Type", "application/json")
    req.Header.Add("Content-Length", strconv.Itoa(len(content)))
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)

    if(response.StatusCode == 204) {
        return "{'result' : 'Success.'}"
    } else {
        return string(body)
    }
}

func publish(uri string, req string) string {
    fmt.Println("Calling " + uri + ".")
    return post(uri, req)
}

func main() {
    var uri = host + service + method + kb
    body := publish(uri, "")
    fmt.Printf(body + "\n")

}
```

**Publicación de la respuesta de una base de conocimiento**

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "result": "Success."
}
```

[Volver arriba](#HOLTop)

<a name="Replace"></a>

## <a name="replace-knowledge-base"></a>Reemplazo de la base de conocimiento

El siguiente código reemplaza el contenido de la base de conocimiento especificada con el método [Reemplazar](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase/replace).

1. Cree un nuevo proyecto de Go en su IDE favorito.
2. Agregue el código que se le ha proporcionado a continuación.
3. Reemplace el valor `key` por una clave de acceso válida para la suscripción.
4. Ejecute el programa.

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "strconv"
)

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
var subscriptionKey string = "ENTER KEY HERE"

// NOTE: Replace this with a valid knowledge base ID.
var kb string = "ENTER ID HERE";

var host string = "https://westus.api.cognitive.microsoft.com"
var service string = "/qnamaker/v4.0"
var method string = "/knowledgebases/"

func pretty_print(content string) string {
    var obj map[string]interface{}
    json.Unmarshal([]byte(content), &obj)
    result, _ := json.MarshalIndent(obj, "", "  ")
    return string(result)
}

func put(uri string, content string) string {
    req, _ := http.NewRequest("PUT", uri, bytes.NewBuffer([]byte(content)))
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
    req.Header.Add("Content-Type", "application/json")
    req.Header.Add("Content-Length", strconv.Itoa(len(content)))
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)

    if(response.StatusCode == 204) {
        return "{'result' : 'Success.'}"
    } else {
        return string(body)
    }
}

var req string = `{
  'qnaList': [
    {
      'id': 0,
      'answer': 'You can use our REST APIs to manage your Knowledge Base. See here for details: https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase/update',
      'source': 'Custom Editorial',
      'questions': [
        'How do I programmatically update my Knowledge Base?'
      ],
      'metadata': [
        {
          'name': 'category',
          'value': 'api'
        }
      ]
    }
  ]
}`;

func replace_kb (uri string, req string) (string) {
    fmt.Println("Calling " + uri + ".")
    return put(uri, req)
}

func main() {
    var uri = host + service + method + kb
    body := replace_kb (uri, req)
    fmt.Printf(body + "\n")
}
```

**Reemplazo de la respuesta de una base de conocimiento**

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
    "result": "Success."
}
```

[Volver arriba](#HOLTop)

<a name="GetQnA"></a>

## <a name="download-the-contents-of-a-knowledge-base"></a>Descarga del contenido de una base de conocimiento

El siguiente código descarga el contenido de la base de conocimiento especificada con el método [Download knowledge base](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase/download) (Descargar base de conocimiento).

1. Cree un nuevo proyecto de Go en su IDE favorito.
2. Agregue el código que se le ha proporcionado a continuación.
3. Reemplace el valor `key` por una clave de acceso válida para la suscripción.
4. Ejecute el programa.

```go
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
)

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
var subscriptionKey string = "ENTER KEY HERE"

// NOTE: Replace this with a valid knowledge base ID.
var kb string = "ENTER ID HERE";

// NOTE: Replace this with "test" or "prod".
var env string = "test";

var host string = "https://westus.api.cognitive.microsoft.com"
var service string = "/qnamaker/v4.0"
var method string = "/knowledgebases/%s/%s/qna"

func pretty_print(content string) string {
    var obj map[string]interface{}
    json.Unmarshal([]byte(content), &obj)
    result, _ := json.MarshalIndent(obj, "", "  ")
    return string(result)
}

func get(uri string) string {
    req, _ := http.NewRequest("GET", uri, nil)
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)
    return string(body)
}

func getQnA(uri string) string {
    fmt.Println("Calling " + uri + ".")
    return get(uri)
}

func main() {
    var method_with_id = fmt.Sprintf(method, kb, env)
    var uri = host + service + method_with_id
    body := getQnA(uri)
    fmt.Printf(body + "\n")
}
```

**Descarga de la respuesta de una base de conocimiento**

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "qnaDocuments": [
    {
      "id": 1,
      "answer": "You can use our REST APIs to manage your Knowledge Base. See here for details: https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase/update",
      "source": "Custom Editorial",
      "questions": [
        "How do I programmatically update my Knowledge Base?"
      ],
      "metadata": [
        {
          "name": "category",
          "value": "api"
        }
      ]
    },
    {
      "id": 2,
      "answer": "QnA Maker provides an FAQ data source that you can query from your bot or application. Although developers will find this useful, content owners will especially benefit from this tool. QnA Maker is a completely no-code way of managing the content that powers your bot or application.",
      "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/faqs",
      "questions": [
        "Who is the target audience for the QnA Maker tool?"
      ],
      "metadata": []
    },
...
  ]
}
```

[Volver arriba](#HOLTop)

<a name="GetAnswers"></a>

## <a name="get-answers-to-a-question-by-using-a-knowledge-base"></a>Obtención de respuestas de una pregunta con una base de conocimiento

El siguiente código obtiene las respuestas de una pregunta con la base de conocimiento especificada, mediante el método **Generar respuestas**.

1. Cree un nuevo proyecto de Go en su IDE favorito.
1. Agregue el código que se le ha proporcionado a continuación.
1. Reemplace el valor `host` por el nombre del sitio web de su suscripción a QnA Maker. Para más información, consulte [Create a QnA Maker service](../How-To/set-up-qnamaker-service-azure.md) (Creación de un servicio de QnA Maker).
1. Reemplace el valor `endpoint_key` por una clave de punto de conexión válida para la suscripción. Tenga en cuenta que es una clave distinta a la clave de suscripción. Puede obtener las claves de punto de conexión mediante el método [Get endpoint keys](#GetKeys) (Obtener claves de punto de conexión).
1. Reemplace el valor `kb` por el identificador de la base de conocimiento que quiere consultar para obtener respuestas. Tenga en cuenta que esta base de conocimiento ya debe haberse publicado mediante el método [Publicar](#Publish).
1. Ejecute el programa.

```go
package main

import (
    "bytes"
    "fmt"
    "io/ioutil"
    "net/http"
    "strconv"
)

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// NOTE: Replace this with a valid host name.
var host string = "ENTER HOST HERE";

// NOTE: Replace this with a valid endpoint key.
// This is not your subscription key.
// To get your endpoint keys, call the GET /endpointkeys method.
var endpoint_key string = "ENTER KEY HERE";

// NOTE: Replace this with a valid knowledge base ID.
// Make sure you have published the knowledge base with the
// POST /knowledgebases/{knowledge base ID} method.
var kb string = "ENTER KB ID HERE";

var method string = "/qnamaker/knowledgebases/" + kb + "/generateanswer"

func post(uri string, content string) string {
    req, _ := http.NewRequest("POST", uri, bytes.NewBuffer([]byte(content)))
    req.Header.Add("Authorization", "EndpointKey " + endpoint_key)
    req.Header.Add("Content-Type", "application/json")
    req.Header.Add("Content-Length", strconv.Itoa(len(content)))
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)

    return string(body)
}

var req string = `{
    'question': 'Is the QnA Maker Service free?',
    'top': 3
}`;

func get_answers(uri string, req string) string {
    fmt.Println("Calling " + uri + ".")
    return post(uri, req)
}

func main() {
    var uri = host + method
    body := get_answers(uri, req)
    fmt.Printf(body + "\n")
}
```

**Obtención de respuestas**

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "answers": [
    {
      "questions": [
        "Can I use BitLocker with the Volume Shadow Copy Service?"
      ],
      "answer": "Yes. However, shadow copies made prior to enabling BitLocker will be automatically deleted when BitLocker is enabled on software-encrypted drives. If you are using a hardware encrypted drive, the shadow copies are retained.",
      "score": 17.3,
      "id": 62,
      "source": "https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-frequently-asked-questions",
      "metadata": []
    },
...
  ]
}
```

[Volver arriba](#HOLTop)

<a name="GetKB"></a>

## <a name="get-information-about-a-knowledge-base"></a>Obtención de información acerca de una base de conocimiento

El siguiente código obtiene información sobre la base de conocimiento especificada mediante el método [Get knowledge base details](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase/getdetails) (Obtener detalles de la base de conocimiento).

1. Cree un nuevo proyecto de Go en su IDE favorito.
2. Agregue el código que se le ha proporcionado a continuación.
3. Reemplace el valor `key` por una clave de acceso válida para la suscripción.
4. Ejecute el programa.

```go
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
)

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
var subscriptionKey string = "ENTER KEY HERE"

// NOTE: Replace this with a valid knowledge base ID.
var kb string = "ENTER ID HERE";

var host string = "https://westus.api.cognitive.microsoft.com"
var service string = "/qnamaker/v4.0"
var method string = "/knowledgebases/"

func pretty_print(content string) string {
    var obj map[string]interface{}
    json.Unmarshal([]byte(content), &obj)
    result, _ := json.MarshalIndent(obj, "", "  ")
    return string(result)
}

func get(uri string) string {
    req, _ := http.NewRequest("GET", uri, nil)
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)
    return string(body)
}

func getKB(uri string) string {
    fmt.Println("Calling " + uri + ".")
    return get(uri)
}

func main() {
    var uri = host + service + method + kb
    body := getKB(uri)
    fmt.Printf(body + "\n")
}
```

**Obtención de respuesta de detalles de la base de conocimiento**

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "id": "140a46f3-b248-4f1b-9349-614bfd6e5563",
  "hostName": "https://qna-docs.azurewebsites.net",
  "lastAccessedTimestamp": "2018-04-12T22:58:01Z",
  "lastChangedTimestamp": "2018-04-12T22:58:01Z",
  "name": "QnA Maker FAQ",
  "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
  "urls": [
    "https://docs.microsoft.com/azure/cognitive-services/qnamaker/faqs",
    "https://docs.microsoft.com/bot-framework/resources-bot-framework-faq"
  ],
  "sources": [
    "Custom Editorial"
  ]
}
```

[Volver arriba](#HOLTop)

<a name="GetKBsByUser"></a>

## <a name="get-all-knowledge-bases-for-a-user"></a>Obtención de todas las bases de conocimiento para un usuario

El siguiente código obtiene información sobre todas las bases de conocimiento para un usuario especificado mediante el método [Get knowledge bases for user](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase/listall) (Obtener bases de conocimiento para el usuario).

1. Cree un nuevo proyecto de Go en su IDE favorito.
2. Agregue el código que se le ha proporcionado a continuación.
3. Reemplace el valor `key` por una clave de acceso válida para la suscripción.
4. Ejecute el programa.

```go
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
)

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
var subscriptionKey string = "ENTER KEY HERE"

var host string = "https://westus.api.cognitive.microsoft.com"
var service string = "/qnamaker/v4.0"
var method string = "/knowledgebases/"

func pretty_print(content string) string {
    var obj map[string]interface{}
    json.Unmarshal([]byte(content), &obj)
    result, _ := json.MarshalIndent(obj, "", "  ")
    return string(result)
}

func get(uri string) string {
    req, _ := http.NewRequest("GET", uri, nil)
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)
    return string(body)
}

func getKBs(uri string) string {
    fmt.Println("Calling " + uri + ".")
    return get(uri)
}

func main() {
    var uri = host + service + method
    body := getKBs(uri)
    fmt.Printf(body + "\n")
}
```

**Obtención de respuesta de bases de conocimiento para el usuario**

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "knowledgebases": [
    {
      "id": "081c32a7-bd05-4982-9d74-07ac736df0ac",
      "hostName": "https://qna-docs.azurewebsites.net",
      "lastAccessedTimestamp": "2018-04-12T11:51:58Z",
      "lastChangedTimestamp": "2018-04-12T11:51:58Z",
      "name": "QnA Maker FAQ",
      "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
      "urls": [],
      "sources": []
    },
    {
      "id": "140a46f3-b248-4f1b-9349-614bfd6e5563",
      "hostName": "https://qna-docs.azurewebsites.net",
      "lastAccessedTimestamp": "2018-04-12T22:58:01Z",
      "lastChangedTimestamp": "2018-04-12T22:58:01Z",
      "name": "QnA Maker FAQ",
      "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
      "urls": [
        "https://docs.microsoft.com/azure/cognitive-services/qnamaker/faqs",
        "https://docs.microsoft.com/bot-framework/resources-bot-framework-faq"
      ],
      "sources": [
        "Custom Editorial"
      ]
    },
...
  ]
}
Press any key to continue.
```

[Volver arriba](#HOLTop)

<a name="Delete"></a>

## <a name="delete-a-knowledge-base"></a>Eliminación de una base de conocimiento

El siguiente código elimina el contenido de la base de conocimiento especificada con el método [Delete knowledge base](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/knowledgebase/delete) (Eliminar base de conocimiento).

1. Cree un nuevo proyecto de Go en su IDE favorito.
2. Agregue el código que se le ha proporcionado a continuación.
3. Reemplace el valor `key` por una clave de acceso válida para la suscripción.
4. Ejecute el programa.

```go
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
)

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
var subscriptionKey string = "ENTER KEY HERE"

// NOTE: Replace this with a valid knowledge base ID.
var kb string = "ENTER ID HERE";

var host string = "https://westus.api.cognitive.microsoft.com"
var service string = "/qnamaker/v4.0"
var method string = "/knowledgebases/"

func pretty_print(content string) string {
    var obj map[string]interface{}
    json.Unmarshal([]byte(content), &obj)
    result, _ := json.MarshalIndent(obj, "", "  ")
    return string(result)
}

func delete(uri string) string {
    req, _ := http.NewRequest("DELETE", uri, nil)
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)

    if(response.StatusCode == 204) {
        return "{'result' : 'Success.'}"
    } else {
        return string(body)
    }
}

func deleteKB(uri string) string {
    fmt.Println("Calling " + uri + ".")
    return delete(uri)
}

func main() {
    var uri = host + service + method + kb
    body := deleteKB(uri)
    fmt.Printf(body + "\n")

}
```

**Eliminación de la respuesta de la base de conocimiento**

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "result": "Success."
}
```

[Volver arriba](#HOLTop)

<a name="GetKeys"></a>

## <a name="get-endpoint-keys"></a>Obtención de claves de punto de conexión

El siguiente código obtiene las claves del punto de conexión actual mediante el método [Get endpoint keys](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/endpointkeys/getkeys) (Obtener claves de punto de conexión).

1. Cree un nuevo proyecto de Go en su IDE favorito.
2. Agregue el código que se le ha proporcionado a continuación.
3. Reemplace el valor `key` por una clave de acceso válida para la suscripción.
4. Ejecute el programa.

```go
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
)

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
var subscriptionKey string = "ENTER KEY HERE"

var host string = "https://westus.api.cognitive.microsoft.com"
var service string = "/qnamaker/v4.0"
var method string = "/endpointkeys/"

func pretty_print(content string) string {
    var obj map[string]interface{}
    json.Unmarshal([]byte(content), &obj)
    result, _ := json.MarshalIndent(obj, "", "  ")
    return string(result)
}

func get(uri string) string {
    req, _ := http.NewRequest("GET", uri, nil)
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)
    return string(body)
}

func getEndpointKeys(uri string) string {
    fmt.Println("Calling " + uri + ".")
    return get(uri)
}

func main() {
    var uri = host + service + method
    body := getEndpointKeys(uri)
    fmt.Printf(body + "\n")
}
```

**Obtención de la respuesta de claves de punto de conexión**

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "primaryEndpointKey": "ac110bdc-34b7-4b1c-b9cd-b05f9a6001f3",
  "secondaryEndpointKey": "1b4ed14e-614f-444a-9f3d-9347f45a9206"
}
```

[Volver arriba](#HOLTop)

<a name="PutKeys"></a>

## <a name="refresh-endpoint-keys"></a>Actualización de las claves de punto de conexión

El siguiente código regenera las claves del punto de conexión actual mediante el método [Refresh endpoint keys](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/endpointkeys/refreshkeys) (Actualizar claves de punto de conexión).

1. Cree un nuevo proyecto de Go en su IDE favorito.
2. Agregue el código que se le ha proporcionado a continuación.
3. Reemplace el valor `key` por una clave de acceso válida para la suscripción.
4. Ejecute el programa.

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "strconv"
)

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
var subscriptionKey string = "ENTER KEY HERE"

// NOTE: Replace this with "PrimaryKey" or "SecondaryKey."
var key_type string = "PrimaryKey";

var host string = "https://westus.api.cognitive.microsoft.com"
var service string = "/qnamaker/v4.0"
var method string = "/endpointkeys/"

func pretty_print(content string) string {
    var obj map[string]interface{}
    json.Unmarshal([]byte(content), &obj)
    result, _ := json.MarshalIndent(obj, "", "  ")
    return string(result)
}

func patch(uri string, content string) string {
    req, _ := http.NewRequest("PATCH", uri, bytes.NewBuffer([]byte(content)))
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
    req.Header.Add("Content-Type", "application/json")
    req.Header.Add("Content-Length", strconv.Itoa(len(content)))
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)

    return string(body)
}

func refresh (uri string, req string) string {
    fmt.Println("Calling " + uri + ".")
    return patch(uri, req)
}

func main() {
    var uri = host + service + method + key_type
    body := refresh (uri, "")
    fmt.Printf(body + "\n")
}
```

**Actualización de la respuesta de claves de punto de conexión**

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "primaryEndpointKey": "ac110bdc-34b7-4b1c-b9cd-b05f9a6001f3",
  "secondaryEndpointKey": "1b4ed14e-614f-444a-9f3d-9347f45a9206"
}
```

[Volver arriba](#HOLTop)

<a name="GetAlterations"></a>

## <a name="get-word-alterations"></a>Obtención de modificaciones de palabras

El siguiente código obtiene las modificaciones de palabras actuales mediante el método [Download alterations](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/alterations/get) (Descargar modificaciones).

1. Cree un nuevo proyecto de Go en su IDE favorito.
2. Agregue el código que se le ha proporcionado a continuación.
3. Reemplace el valor `key` por una clave de acceso válida para la suscripción.
4. Ejecute el programa.

```go
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
)

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
var subscriptionKey string = "ENTER KEY HERE"

var host string = "https://westus.api.cognitive.microsoft.com"
var service string = "/qnamaker/v4.0"
var method string = "/alterations/"

func pretty_print(content string) string {
    var obj map[string]interface{}
    json.Unmarshal([]byte(content), &obj)
    result, _ := json.MarshalIndent(obj, "", "  ")
    return string(result)
}

func get(uri string) string {
    req, _ := http.NewRequest("GET", uri, nil)
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)
    return string(body)
}

func getAlterations(uri string) string {
    fmt.Println("Calling " + uri + ".")
    return get(uri)
}

func main() {
    var uri = host + service + method
    body := getAlterations(uri)
    fmt.Printf(body + "\n")
}
```

**Obtención de la respuesta de modificaciones de palabras**

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "wordAlterations": [
    {
      "alterations": [
        "botframework",
        "bot frame work"
      ]
    }
  ]
}
```

[Volver arriba](#HOLTop)

<a name="PutAlterations"></a>

## <a name="replace-word-alterations"></a>Reemplazo de modificaciones de palabras

El siguiente código reemplaza las modificaciones de palabras actuales mediante el método [Replace alterations](https://docs.microsoft.com/rest/api/cognitiveservices/qnamaker/alterations/replace) (Reemplazar modificaciones).

1. Cree un nuevo proyecto de Go en su IDE favorito.
2. Agregue el código que se le ha proporcionado a continuación.
3. Reemplace el valor `key` por una clave de acceso válida para la suscripción.
4. Ejecute el programa.

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "strconv"
)

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
var subscriptionKey string = "ENTER KEY HERE"

var host string = "https://westus.api.cognitive.microsoft.com"
var service string = "/qnamaker/v4.0"
var method string = "/alterations/"

func pretty_print(content string) string {
    var obj map[string]interface{}
    json.Unmarshal([]byte(content), &obj)
    result, _ := json.MarshalIndent(obj, "", "  ")
    return string(result)
}

func put(uri string, content string) string {
    req, _ := http.NewRequest("PUT", uri, bytes.NewBuffer([]byte(content)))
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)
    req.Header.Add("Content-Type", "application/json")
    req.Header.Add("Content-Length", strconv.Itoa(len(content)))
    client := &http.Client{}
    response, err := client.Do(req)
    if err != nil {
        panic(err)
    }

    defer response.Body.Close()
    body, _ := ioutil.ReadAll(response.Body)

    if(response.StatusCode == 204) {
        return "{'result' : 'Success.'}"
    } else {
        return string(body)
    }
}

var req string = `{
  'wordAlterations': [
    {
      'alterations': [
        'botframework',
        'bot frame work'
      ]
    }
  ]
}`;

func putAlterations (uri string, req string) (string) {
    fmt.Println("Calling " + uri + ".")
    return put(uri, req)
}

func main() {
    var uri = host + service + method
    body := putAlterations (uri, req)
    fmt.Printf(body + "\n")
}
```

**Reemplazo de la respuesta de modificaciones de palabras**

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo: 

```json
{
  "result": "Success."
}
```

[Volver arriba](#HOLTop)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Referencia de QnA Maker (V4) REST API](https://go.microsoft.com/fwlink/?linkid=2092179)

## <a name="see-also"></a>Vea también 

[Introducción de QnA Maker](../Overview/overview.md)
