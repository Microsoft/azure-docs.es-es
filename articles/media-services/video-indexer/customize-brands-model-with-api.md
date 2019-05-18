---
title: Uso de Azure Video Indexer para personalizar el modelo de marcas
titlesuffix: Azure Media Services
description: En este artículo se muestra cómo usar Azure Video Indexer para personalizar el modelo de marcas.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: anzaman
ms.openlocfilehash: 8d0806bc0262cd45a49e4f97ea629683ac239aa8
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799637"
---
# <a name="customize-a-brands-model-with-the-video-indexer-api"></a>Personalización de un modelo de las marcas con la API de Video Indexer

Video Indexer permite la detección de marcas por voz y texto visual durante la indexación y la reindexación de contenido de audio y vídeo. La característica de detección de marcas identifica menciones de productos, servicios y compañías sugeridas por la base de datos de marcas de Bing. Por ejemplo, si Microsoft se menciona en un contenido de audio o vídeo, o si se muestra en texto visual en un vídeo, Video Indexer lo detectará como una marca en el contenido. Un modelo de marcas personalizado permite excluir determinadas marcas de la detección e incluir otras que deben ser parte del modelo que podrían no estar en la base de datos de marcas de Bing.

Para información general detallada, consulte [Introducción](customize-brands-model-overview.md).

Puede utilizar las API de Video Indexer para crear, usar y editar modelos de marcas personalizados detectados en un vídeo, como se describe en este tema. También puede usar el sitio web de Video Indexer, como se describe en [Customize Brands model using the Video Indexer website](customize-brands-model-with-api.md) (Personalización de un modelo de marcas mediante el sitio web de Video Indexer).

## <a name="create-a-brand"></a>Creación de una marca

Puede crear una marca personalizada y agregarla al modelo de marcas personalizado de la cuenta especificada.

### <a name="request-url"></a>URL de la solicitud

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Brands?accessToken={accessToken}
```

[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Create-Brand).

### <a name="request-parameters"></a>Parámetros de solicitud

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountId|string|Sí|Identificador único global de la cuenta.|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

Además de estos parámetros, debe proporcionar un objeto JSON de cuerpo de solicitud que facilite información sobre la nueva marca, siguiendo el formato del ejemplo siguiente.

```json
{
  "name": "Example",
  "enabled": true,
  "tags": ["Tag1", "Tag2"],
  "description": "This is an example",
  "referenceUrl": "https://en.wikipedia.org/wiki/Example"
}
```

Al establecer **enabled** en true la marca se coloca en la lista *Include* (Incluir) para que Video Indexer la detecte. Al establecer **enabled** en false la marca se coloca en la lista *Exclude* (Excluir), así que Video Indexer no la detectará.

El valor **referenceUrl** puede ser cualquier sitio web de referencia para la marca, como un vínculo a su página de Wikipedia.

El valor **tags** es una lista de etiquetas de la marca. Este valor se muestra en el campo *Category* (Categoría) de la marca del sitio web de Video Indexer. Por ejemplo, la marca "Azure" se puede etiquetar o clasificar como "Nube".

### <a name="response"></a>Respuesta

La respuesta proporciona información sobre la marca que acaba de crear, siguiendo el formato del ejemplo siguiente.

```json
{
  "referenceUrl": "https://en.wikipedia.org/wiki/Example",
  "id": 97974,
  "name": "Example",
  "accountId": "SampleAccountId",
  "lastModifierUserName": "SampleUserName",
  "created": "2018-04-25T14:59:52.7433333",
  "lastModified": "2018-04-25T14:59:52.7433333",
  "enabled": true,
  "description": "This is an example",
  "tags": [
    "Tag1",
    "Tag2"
  ]
}
```

## <a name="delete-a-brand"></a>Eliminación de una marca

Puede quitar una marca del modelo de marcas personalizado de la cuenta especificada. La cuenta se especifica en el parámetro **accountId**. Una vez que se llama correctamente, la marca ya no está en las listas de marcas *Include* (Incluir) o *Exclude* (Excluir).

### <a name="request-url"></a>URL de la solicitud

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Brands/{id}?accessToken={accessToken}
```

[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Delete-Brand?).

### <a name="request-parameters"></a>Parámetros de solicitud

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountId|string|Sí|Identificador único global de la cuenta.|
|id|integer|Sí|El identificador de la marca (generado cuando se creó la marca).|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

No se devuelve ningún contenido cuando la marca se elimina correctamente.

## <a name="get-a-specific-brand"></a>Obtención de una hoja específica

Puede buscar los detalles de una marca en el modelo de marcas personalizado de la cuenta especificada mediante el identificador de marca.

### <a name="request-url"></a>URL de la solicitud

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Brands?accessToken={accessToken}
```

[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Brand?).

### <a name="request-parameters"></a>Parámetros de solicitud

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountId|string|Sí|Identificador único global de la cuenta.|
|id|integer|Sí|El identificador de la marca (generado cuando se creó la marca).|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

La respuesta proporciona información sobre la marca que ha buscado (mediante el identificador de marca), siguiendo el formato del ejemplo siguiente.

```json
{
  "referenceUrl": "https://en.wikipedia.org/wiki/Example",
  "id": 128846,
  "name": "Example",
  "accountId": "SampleAccountId",
  "lastModifierUserName": "SampleUserName",
  "created": "2018-01-06T13:51:38.3666667",
  "lastModified": "2018-01-11T13:51:38.3666667",
  "enabled": true,
  "description": "This is an example",
  "tags": [
    "Tag1",
    "Tag2"
  ]
}
```

> [!NOTE]
> Si **enabled** se establece en **true** significa que la marca está en la lista *Include* (Incluir) para que Video Indexer la detecte; si **enabled** es false significa que la marca está en la lista *Exclude* (Excluir), así que Video Indexer no la detectará.

## <a name="update-a-specific-brand"></a>Actualización de una marca específica

Puede buscar los detalles de una marca en el modelo de marcas personalizado de la cuenta especificada mediante el identificador de marca.

### <a name="request-url"></a>URL de la solicitud

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Brands/{id}?accessToken={accessToken}
```

[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Update-Brand?).

### <a name="request-parameters"></a>Parámetros de solicitud

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountId|string|Sí|Identificador único global de la cuenta.|
|id|integer|Sí|El identificador de la marca (generado cuando se creó la marca).|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

Además de estos parámetros, tiene que proporcionar un objeto JSON de cuerpo de solicitud que facilite información actualizada sobre la marca que quiere actualizar siguiendo el formato del ejemplo siguiente.

```json
{
  "name": "Example",
  "enabled": false,
  "tags": ["Tag1", "NewTag2"],
  "description": "This is an update example",
  "referenceUrl": "https://en.wikipedia.org/wiki/Example",
  "lastModifierUserName": "SampleUserName",
  "created": "2018-04-25T14:59:52.7433333",
  "lastModified": "2018-04-28T15:52:22.3413983",
}
```

> [!NOTE]
> En este ejemplo, la marca que se creó en el cuerpo de solicitud de ejemplo en la sección **Creación de una marca** se actualiza aquí con una nueva etiqueta y una nueva descripción. El valor **enabled** también se ha cambiado a false para colocarla en la lista *Exclude* (Excluir).

### <a name="response"></a>Respuesta

La respuesta proporciona la información actualizada sobre la marca que actualizó, siguiendo el formato del ejemplo siguiente.

```json
{
  "referenceUrl": null,
  "id": 97974,
  "name": "Example",
  "accountId": "SampleAccountId",
  "lastModifierUserName": "SampleUserName",
  "Created": "2018-04-25T14:59:52.7433333",
  "lastModified": "2018-04-25T15:37:50.67",
  "enabled": false,
  "description": "This is an update example",
  "tags": [
    "Tag1",
    "NewTag2"
  ]
}
```

## <a name="get-all-of-the-brands"></a>Obtención de todas las marcas

Puede devolver todas las marcas del modelo de marcas personalizado de la cuenta especificada, independientemente de si se quiere que la marca esté en las listas *Include* (Incluir) o *Exclude* (Excluir).

### <a name="request-url"></a>URL de la solicitud

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Brands?accessToken={accessToken}
```

[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Brands?).

### <a name="request-parameters"></a>Parámetros de solicitud

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountId|string|Sí|Identificador único global de la cuenta.|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

La respuesta proporciona una lista de todas las marcas de su cuenta y cada uno de sus detalles, siguiendo el formato del ejemplo siguiente.

```json
[
    {
        "ReferenceUrl": null,
        "id": 97974,
        "name": "Example",
        "accountId": "AccountId",
        "lastModifierUserName": "UserName",
        "Created": "2018-04-25T14:59:52.7433333",
        "LastModified": "2018-04-25T14:59:52.7433333",
        "enabled": true,
        "description": "This is an example",
        "tags": ["Tag1", "Tag2"]
    },
    {
        "ReferenceUrl": null,
        "id": 97975,
        "name": "Example2",
        "accountId": "AccountId",
        "lastModifierUserName": "UserName",
        "Created": "2018-04-26T14:59:52.7433333",
        "LastModified": "2018-04-26T14:59:52.7433333",
        "enabled": false,
        "description": "This is another example",
        "tags": ["Tag1", "Tag2"]
    },
]
```

> [!NOTE]
> La marca llamada *Example* está en la lista *Include* (Incluir) para que Video Indexer la detecte, y la marca llamada *Example2* está en la lista *Exclude* (Excluir), así que Video Indexer no la detectará.

## <a name="get-brands-model-settings"></a>Obtención de la configuración del modelo de marcas

Puede devolver la configuración del modelo de marcas de la cuenta especificada. La configuración del modelo de marcas representa si la detección desde la base de datos de Bing está o no habilitada. Si las marcas de Bing no están habilitadas, Video Indexer solo detectará las marcas del modelo de marcas personalizado de la cuenta especificada.

### <a name="request-url"></a>URL de la solicitud

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Brands?accessToken={accessToken}
```

[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Brands).

### <a name="request-parameters"></a>Parámetros de solicitud

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountId|string|Sí|Identificador único global de la cuenta.|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

La respuesta muestra si las marcas de Bing están habilitadas, siguiendo el formato del ejemplo siguiente.

```json
{
  "state": true,
  "useBuiltIn": true
}
```

> [!NOTE]
> Si **useBuiltIn** se establece en true representa que las marcas de Bing están habilitadas. Si *useBuiltin* es false, las marcas de Bing están deshabilitadas. El valor **state** se puede omitir porque ya no se utiliza.

## <a name="update-brands-model-settings"></a>Actualización de la configuración del modelo de marcas

Puede actualizar la configuración del modelo de marcas de la cuenta especificada. La configuración del modelo de marcas representa si la detección desde la base de datos de Bing está o no habilitada. Si las marcas de Bing no están habilitadas, Video Indexer solo detectará las marcas del modelo de marcas personalizado de la cuenta especificada.

### <a name="request-url"></a>URL de la solicitud:
```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/BrandsModelSettings?accessToken={accessToken}
```

[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Update-Brands-Model-Settings?).

### <a name="request-parameters"></a>Parámetros de solicitud

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountId|string|Sí|Identificador único global de la cuenta.|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

Además de estos parámetros, debe proporcionar un objeto JSON de cuerpo de solicitud que facilite información sobre la nueva marca, siguiendo el formato del ejemplo siguiente.

```json
{
    "useBuiltIn":true
}
```

> [!NOTE]
> Si **useBuiltIn** se establece en true representa que las marcas de Bing están habilitadas. Si *useBuiltin* es false, las marcas de Bing están deshabilitadas.

### <a name="response"></a>Respuesta

No se devuelve ningún contenido cuando la configuración del modelo de marcas se actualiza correctamente.

## <a name="next-steps"></a>Pasos siguientes

[Customize Brands model using website](customize-brands-model-with-website.md) (Personalización del modelo de marcas mediante el sitio web)
