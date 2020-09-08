---
title: Obtención de información de la clave de cifrado
titleSuffix: Azure Cognitive Search
description: Recupere el nombre y la versión de la clave de cifrado que se usan en un mapa de sinónimos o índices para poder administrar la clave en Azure Key Vault.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 08/01/2020
ms.openlocfilehash: 37ff94608e9756142f70a4f3c64d0a6f7eeea685
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88932906"
---
# <a name="get-customer-managed-key-information-from-indexes-and-synonym-maps"></a>Obtención de información de la clave administrada por el cliente de mapas de sinónimos e índices

En Azure Cognitive Search, las claves de cifrado administradas por el cliente se crean, almacenan y administran en Azure Key Vault. Si necesita determinar si un objeto está cifrado o qué nombre o versión de la clave se han usado, utilice la API REST o un SDK para recuperar la propiedad **encryptionKey** de una definición del mapa de índices o sinónimos. 

Se recomienda [habilitar el registro](../key-vault/general/logging.md) en Key Vault para poder supervisar el uso de la clave.

## <a name="get-the-admin-api-key"></a>Obtención de la clave de API de administración

Para obtener definiciones de objetos de un servicio de búsqueda, tendrá que autenticarse con derechos de administrador. La forma más sencilla de obtener la clave de API de administración es a través de Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y abra la página de información general del servicio de búsqueda.

1. En el lado izquierdo, haga clic en **Claves** y copie una API de administración. Para recuperar un mapa de sinónimos e índices, se requiere una clave de administración.

Para realizar los restantes pasos, cambie a PowerShell y la API REST. El portal no muestra mapas de asignaciones ni las propiedades de la clave de cifrado de los índices.

## <a name="use-powershell-and-rest"></a>Uso de PowerShell y REST

Ejecute los siguientes comandos para configurar las variables y obtener definiciones de los objetos.

```powershell
<# Connect to Azure #>
$Connect-AzAccount

<# Provide the admin API key used for search service authentication  #>
$headers = @{
'api-key' = '<YOUR-ADMIN-API-KEY>'
'Content-Type' = 'application/json'
'Accept' = 'application/json' }

<# List all existing synonym maps #>
$uri= 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/synonyms?api-version=2020-06-30&$select=name'
Invoke-RestMethod -Uri $uri -Headers $headers | ConvertTo-Json

<# List all existing indexes #>
$uri= 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes?api-version=2020-06-30&$select=name'
Invoke-RestMethod -Uri $uri -Headers $headers | ConvertTo-Json

<# Return a specific synonym map definition. The encryptionKey property is at the end #>
$uri= 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/synonyms/<YOUR-SYNONYM-MAP-NAME>?api-version=2020-06-30'
Invoke-RestMethod -Uri $uri -Headers $headers | ConvertTo-Json

<# Return a specific index definition. The encryptionKey property is at the end #>
$uri= 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/<YOUR-INDEX-NAME>?api-version=2020-06-30'
Invoke-RestMethod -Uri $uri -Headers $headers | ConvertTo-Json
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe qué clave y versión de cifrado se usan, puede administrar la clave en Azure Key Vault o comprobar otras opciones de configuración.

+ [Inicio rápido: Establecimiento y recuperación de un secreto de Azure Key Vault mediante PowerShell](../key-vault/secrets/quick-create-powershell.md)

+ [Configuración de claves administradas por el cliente para el cifrado de datos en Azure Cognitive Search](search-security-manage-encryption-keys.md)