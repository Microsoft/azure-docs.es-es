---
title: 'ML Studio (clásico): administración de datos de Azure AI Gallery (Azure)'
description: Puede exportar y eliminar los datos de usuario integrados de Azure AI Gallery con la interfaz o con la API Catalog de AI Gallery. Este artículo le muestra cómo.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 05/25/2018
ms.reviewer: jmartens, mldocs
ms.openlocfilehash: 8255a4c6556dc1a36d763e48f6bcc5eec012b553
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87292984"
---
# <a name="view-and-delete-in-product-user-data-from-azure-ai-gallery"></a>Visualización y eliminación de los datos de usuario integrados de Azure AI Gallery

[!INCLUDE [Notebook deprecation notice](../../../includes/aml-studio-notebook-notice.md)]

Puede ver y eliminar los datos de usuario integrados de Azure AI Gallery con la interfaz o con la API Catalog de AI Gallery. En este artículo se indica cómo hacerlo.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-dsr-and-stp-note.md)]

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

## <a name="view-your-data-in-ai-gallery-with-the-ui"></a>Visualización de los datos en AI Gallery con la interfaz de usuario

Puede ver los elementos publicados en la interfaz de usuario del sitio web de Azure AI Gallery. Los usuarios pueden ver soluciones, proyectos, experimentos y otros elementos publicados y también los no enumerados.

1.    Inicie sesión en [Azure AI Gallery](https://gallery.azure.ai/).
2.    Haga clic en la imagen de perfil en la esquina superior derecha y, a continuación, en el nombre de cuenta para cargar la página de perfil.
3.    La página de perfil muestra todos los elementos publicados en la galería, incluidas las entradas no enumeradas.

## <a name="use-the-ai-gallery-catalog-api-to-view-your-data"></a>Uso de la API Catalog de AI Gallery para visualizar los datos

Puede ver los datos recopilados mediante programación con la API Catalog de AI Gallery, a la que se puede acceder en https://catalog.cortanaanalytics.com/entities. Para visualizar los datos, necesita el identificador de autor. Para visualizar las entidades no enumeradas con la API Catalog, se necesita un token de acceso.

Las respuestas del catálogo se devuelven en formato JSON.

### <a name="get-an-author-id"></a>Obtención de un identificador de autor
El identificador de autor se basa en la dirección de correo electrónico utilizada al publicar en Azure AI Gallery. Este valor no cambia:

1. Inicie sesión en [Azure AI Gallery](https://gallery.azure.ai/).
2. Haga clic en la imagen de perfil en la esquina superior derecha y, a continuación, en el nombre de cuenta para cargar la página de perfil.
3. La dirección URL de la barra de direcciones muestra el siguiente identificador alfanumérico `authorId=`. Por ejemplo, para la dirección URL:\
    `https://gallery.azure.ai/Home/Author?authorId=99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA`

    Identificador del autor:\
    `99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA`

### <a name="get-your-access-token"></a>Obtención del token de acceso

Para visualizar las entidades no enumeradas con la API Catalog, se necesita un token de acceso. Sin token de acceso, los usuarios todavía pueden ver las entidades públicas y otra información del usuario.

Para obtener un token de acceso, debe inspeccionar el encabezado `DataLabAccessToken` de una solicitud HTTP hecha por el explorador a la API Catalog con la sesión iniciada:

1.    Inicie sesión en [Azure AI Gallery](https://gallery.azure.ai/).
2.    Haga clic en la imagen de perfil en la esquina superior derecha y, a continuación, en el nombre de cuenta para cargar la página de perfil.
3.    Abra el panel Herramientas de desarrollo del explorador presionando F12, seleccione la pestaña Red y actualice la página. 
4. Filtre las solicitudes por la cadena *catalog* escribiendo en el cuadro de texto de filtro.
5.    En las solicitudes a la dirección URL `https://catalog.cortanaanalytics.com/entities`, busque una solicitud GET y seleccione la pestaña *Encabezados*. Desplácese hacia abajo hasta la sección *Encabezados de solicitud*.
6.    El token alfanumérico está en el encabezado `DataLabAccessToken`. Para ayudar a proteger los datos, no comparta este token.

### <a name="view-user-information"></a>Visualización de la información de usuario
Con el identificador de autor que obtuvo en los pasos anteriores, puede ver la información de un perfil de usuario mediante la sustitución de `[AuthorId]` en la dirección URL siguiente:

`https://catalog.cortanaanalytics.com/users/[AuthorID]`

Por ejemplo, esta solicitud URL:

`https://catalog.cortanaanalytics.com/users/99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA`

Devuelve una respuesta como:

```json
{"entities_count":9,"contribution_score":86.351575190956922,"scored_at":"2018-05-07T14:30:25.9305671+00:00","contributed_at":"2018-05-07T14:26:55.0381756+00:00","created_at":"2017-12-15T00:49:15.6733094+00:00","updated_at":"2017-12-15T00:49:15.6733094+00:00","name":"First Last","slugs":["First-Last"],"tenant_id":"14b2744cf8d6418c87ffddc3f3127242","community_id":"9502630827244d60a1214f250e3bbca7","id":"99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA","_links":{"self":"https://catalog.azureml.net/tenants/14b2744cf8d6418c87ffddc3f3127242/communities/9502630827244d60a1214f250e3bbca7/users/99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA"},"etag":"\"2100d185-0000-0000-0000-5af063010000\""}
```

### <a name="view-public-entities"></a>Visualización de entidades públicas

La API Catalog almacena información sobre las entidades publicadas en Azure AI Gallery que también puede ver directamente en el [sitio web de AI Gallery](https://gallery.azure.ai/). 

Para ver las entidades publicadas, visite la siguiente dirección URL, reemplazando `[AuthorId]` por el identificador de autor obtenido anteriormente en [Obtención de un identificador de autor](#get-an-author-id).

`https://catalog.cortanaanalytics.com/entities?$filter=author/id eq '[AuthorId]'`

Por ejemplo:

`https://catalog.cortanaanalytics.com/entities?$filter=author/id eq '99F1F5C6260295F1078187FA179FBE08B618CB62129976F09C6AF0923B02A5BA'`

### <a name="view-unlisted-and-public-entities"></a>Visualización de las entidades públicas y no enumeradas

Esta consulta muestra solo las entidades públicas. Para ver todas las entidades, incluso las no enumeradas, proporcione el token de acceso obtenido en la sección anterior.

1.    Mediante una herramienta como [Postman](https://www.getpostman.com), cree una solicitud HTTP GET a la dirección URL del catálogo, como se describe en [Obtención del token de acceso](#get-your-access-token).
2.    Cree un encabezado de solicitud HTTP llamado `DataLabAccessToken`, con el valor establecido para el token de acceso.
3.    Envíe la solicitud HTTP.

> [!TIP]
> Si las entidades no enumeradas no aparecen en las respuestas de la API Catalog, el usuario podría tener un token de acceso no válido o expirado. Cierre la sesión de Azure AI Gallery y, a continuación, repita los pasos de la sección [Obtención del token de acceso](#get-your-access-token) para renovar el token. 
