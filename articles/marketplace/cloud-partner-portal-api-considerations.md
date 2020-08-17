---
title: 'Consideraciones sobre las API: Azure Marketplace'
description: Versiones, control de errores y problemas de autorización con las API de Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: c8d5c9365e2cedce7a6ed877bcf93ecd17ff220b
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287878"
---
# <a name="api-considerations"></a>Consideraciones sobre las API

<a name="api-versioning"></a>Control de versiones de la API
--------------

> [!NOTE]
> Las API de Cloud Partner Portal se integran con el Centro de partners y seguirán funcionando en este. La transición presenta pequeños cambios. Revise los cambios que se muestran en [Referencia de API de Cloud Partner Portal](./cloud-partner-portal-api-overview.md) para asegurarse de que el código sigue funcionando después de la transición al Centro de partners. Las API de CPP solo deben usarse para los productos existentes que ya estaban integrados antes de la transición al Centro de partners. Los nuevos productos deben usar las API de envío del Centro de partners.

Puede haber varias versiones de la API disponibles al mismo tiempo. Los clientes deben indicar qué versión quieren invocar para usarla proporcionando el parámetro `api-version` como parte de la cadena de consulta.

   `GET https://cloudpartner.azure.com/api/offerTypes?api-version=2017-10-31`

La respuesta a una solicitud con una versión de API desconocida o no válida es un código HTTP 400. Este error devuelve la colección de versiones de API conocidas en el cuerpo de la respuesta.

``` json
    {
        "error": { 
            "code":"InvalidAPIVersion",
            "message":"Invalid api version. Allowed values are [2016-08-01-preview]"
        }
    }
```            

<a name="errors"></a>Errors
------

La API responde a los errores con los códigos de estado HTTP correspondientes y, opcionalmente, información adicional en la respuesta serializada como JSON.
Cuando reciba un error, especialmente un error de clase 400, no vuelva a intentar la solicitud antes de corregir la causa subyacente. Por ejemplo, en la respuesta de ejemplo anterior, corrija el parámetro de la versión de la API antes de volver a enviar la solicitud.

<a name="authorization-header"></a>Encabezado de autorización
--------------------

Para todas las API de esta referencia, debe pasar el encabezado de autorización junto con el token de portador obtenido de Azure Active Directory (Azure AD). Este encabezado se requiere para recibir una respuesta válida. Si no está presente, se devuelve un error `401 Unauthorized`. 

``` HTTP
  GET https://cloudpartner.azure.com/api/offerTypes?api-version=2016-08-01-preview

    Accept: application/json 
    Authorization: Bearer <YOUR_TOKEN> 
```
