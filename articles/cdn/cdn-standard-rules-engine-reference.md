---
title: Referencia del motor de reglas estándar de Azure CDN | Microsoft Docs
description: Documentación de referencia para las condiciones de coincidencia y las acciones del motor de reglas estándar de Azure Content Delivery Network (Azure CDN).
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: article
ms.date: 08/04/2020
ms.author: allensu
ms.openlocfilehash: f729176d3f79c2a1f6fabb5631d49747219db48f
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87760097"
---
# <a name="standard-rules-engine-reference-for-azure-cdn"></a>Referencia del motor de reglas estándar de Azure CDN

En el [motor de reglas estándar](cdn-standard-rules-engine.md) de Azure Content Delivery Network (Azure CDN), una regla consta de una o varias condiciones de coincidencia y una acción. En este artículo se muestran descripciones detalladas de las condiciones de coincidencia y las características disponibles en el motor de reglas estándar para Azure CDN.

El motor de reglas está diseñado para ser la autoridad definitiva sobre cómo Azure CDN estándar procesa los tipos específicos de solicitudes.

**Usos comunes de las reglas**:

- Invalidar o definir una directiva de memoria caché personalizada.
- Redirigir solicitudes.
- Modificar los encabezados de solicitudes y respuestas HTTP.

## <a name="terminology"></a>Terminología

Para definir una regla en el motor de reglas, establezca las [condiciones de coincidencia](cdn-standard-rules-engine-match-conditions.md) y las [acciones](cdn-standard-rules-engine-actions.md):

 ![Estructura de reglas de Azure CDN](./media/cdn-standard-rules-engine-reference/cdn-rules-structure.png)

Cada regla puede tener hasta diez condiciones de coincidencia y cinco acciones. Cada punto de conexión de Azure CDN puede tener hasta 25 reglas. 

En este límite se incluye una *regla global* predeterminada. La regla global no tiene condiciones de coincidencia, y las acciones que se definen en una regla global siempre se desencadenan.

## <a name="limits-and-pricing"></a>Límites y precios 

Cada punto de conexión de Azure CDN puede tener hasta 25 reglas. Cada regla puede tener hasta diez condiciones de coincidencia y cinco acciones. Los precios del motor de reglas siguen las dimensiones a continuación: 
- Reglas: 1 USD por regla al mes 
- Solicitudes procesadas: 0,60 USD por millón de solicitudes
- Las primeras 5 reglas seguirán siendo gratuitas

## <a name="syntax"></a>Sintaxis

La forma en que los caracteres especiales se tratan en una regla varía en función de cómo las diferentes condiciones de coincidencia y acciones controlan los valores de texto. Una condición de coincidencia o acción puede interpretar el texto de una de las siguientes maneras:

- [Valores literales](#literal-values)
- [Valores de carácter comodín](#wildcard-values)


### <a name="literal-values"></a>Valores literales

El texto que se interpreta como un valor literal trata todos los caracteres especiales, *excepto el símbolo %* , como parte del valor que debe coincidir en una regla. Por ejemplo, una condición de coincidencia literal establecida en `'*'` se cumple solo cuando se encuentra el valor exacto `'*'`.

Se usa un símbolo de porcentaje para indicar la codificación de direcciones URL (p. ej., `%20`).

### <a name="wildcard-values"></a>Valores de carácter comodín

Actualmente se admite el carácter comodín en la **condición de coincidencia UrlPath** en el motor de reglas estándar. El carácter \* es un asterisco que representa uno o más caracteres. 

## <a name="next-steps"></a>Pasos siguientes

- [Condiciones de coincidencia en el motor de reglas estándar](cdn-standard-rules-engine-match-conditions.md)
- [Acciones en el motor de reglas estándar](cdn-standard-rules-engine-actions.md)
- [Aplicación de HTTPS mediante el motor de reglas estándar](cdn-standard-rules-engine.md)
- [Información general de Azure CDN](cdn-overview.md)
