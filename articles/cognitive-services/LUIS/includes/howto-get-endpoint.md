---
title: archivo de inclusión
description: archivo de inclusión
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.date: 05/06/2020
ms.subservice: language-understanding
ms.topic: include
ms.author: diberry
ms.openlocfilehash: 01d102de24185eba789e041d8534d6fdce8c260b
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87375487"
---
En la sección **Administrar** (menú de la parte superior derecha), en la página **Recursos de Azure** (menú izquierdo), copie la dirección URL de la **Consulta de ejemplo** y péguela en una nueva pestaña del explorador.

La dirección URL del punto de conexión tiene un formato similar al siguiente; aunque su propio subdominio personalizado, id. de la aplicación y clave del punto de conexión reemplazarán a APP-ID y KEY-ID:

```console
https://YOUR-CUSTOM-SUBDOMAIN.api.cognitive.microsoft.com/luis/prediction/v3.0/apps/APP-ID/slots/production/predict?subscription-key=KEY-ID&verbose=true&show-all-intents=true&log=true&query=YOUR_QUERY_HERE
```