---
title: Condiciones de coincidencia del motor de reglas de Azure Front Door
description: En este artículo se proporciona una lista de las distintas condiciones de coincidencia disponibles con el motor de reglas de Azure Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/14/2020
ms.author: duau
ms.openlocfilehash: 1b2b891a0b6b67efef38005d3a4d67eecf41afbd
ms.sourcegitcommit: 03662d76a816e98cfc85462cbe9705f6890ed638
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90531872"
---
# <a name="azure-front-door-rules-engine-match-conditions"></a>Condiciones de coincidencia del motor de reglas de Azure Front Door

En el [motor de reglas de Azure Front Door](front-door-rules-engine.md) una regla consta de cero o más condiciones de coincidencia y de una acción. En este artículo se muestran descripciones detalladas de las condiciones de coincidencia que puede usar en el motor de reglas de Azure Front Door.

La primera parte de una regla es una condición de coincidencia o un conjunto de condiciones de coincidencia. Una regla puede contener hasta 10 condiciones de coincidencia. Una condición de coincidencia identifica tipos específicos de solicitudes para los que se realizan las acciones definidas. Si usa varias condiciones de coincidencia, estas se agruparán mediante la lógica AND. Para todas las condiciones de coincidencia que admiten varios valores (que se indican a continuación como "separadas por espacios"), se supone el operador "OR".

Por ejemplo, puede usar una condición de coincidencia para:

- Filtrar las solicitudes en función de una dirección IP específica, un país o una región.
- Filtrar las solicitudes por la información de encabezado.
- Filtrar las solicitudes de los dispositivos móviles o de escritorio.

Las siguientes condiciones de coincidencia están disponibles para su uso en el motor de reglas de Azure Front Door.  

## <a name="device-type"></a>Tipo de dispositivo 

Identifica las solicitudes realizadas desde un dispositivo móvil o un dispositivo de escritorio.  

#### <a name="required-fields"></a>Campos obligatorios

Operator | Valores admitidos
---------|----------------
Es igual a, No es igual a | Móvil, Escritorio

## <a name="post-argument"></a>Argumento POST

Identifica las solicitudes en función de los argumentos definidos para el método POST que se usa en la solicitud. 

#### <a name="required-fields"></a>Campos obligatorios

Nombre de argumento | Operator | Valor de argumento | Transformación de mayúsculas y minúsculas
--------------|----------|----------------|---------------
String | [Lista de operadores estándar](#standard-operator-list) | Cadena, Int | Minúsculas, mayúsculas, recortar, quitar espacio en blanco, codificación de URL, descodificación de URL

## <a name="query-string"></a>Cadena de consulta

Identifica las solicitudes que contienen el parámetro de cadena de consulta especificado. Este parámetro se establece en un valor que coincide con un patrón específico. Los parámetros de cadena de consulta (por ejemplo, **parameter=value**) de la dirección URL de la solicitud determinan si se cumple esta condición. Esta condición de coincidencia identifica un parámetro de cadena de consulta por su nombre y acepta uno o varios valores para el valor del parámetro.

#### <a name="required-fields"></a>Campos obligatorios

Operator | Cadena de consulta | Transformación de caso
---------|--------------|---------------
[Lista de operadores estándar](#standard-operator-list) | Cadena, Int | Minúsculas, mayúsculas, recortar, quitar espacio en blanco, codificación de URL, descodificación de URL

## <a name="remote-address"></a>Dirección remota

Identifica las solicitudes en función de la ubicación o la dirección IP del solicitante.

#### <a name="required-fields"></a>Campos obligatorios

Operator | Valores admitidos
---------|-----------------
Coincidencia geográfica | Código de país
Coincidencia de IP | Dirección IP (separada por espacios)
No tiene ninguna coincidencia geográfica | Código de país
No tiene ninguna coincidencia de IP | Dirección IP (separada por espacios)

#### <a name="key-information"></a>Información importante

- Utilice la notación CIDR.
- Para especificar varias direcciones IP y bloques de direcciones IP, utilice un solo espacio entre los valores:
  - **Ejemplo IPv4**: *1.2.3.4 10.20.30.40* coincide con las solicitudes que llegan desde la dirección 1.2.3.4 o la 10.20.30.40.
  - **Ejemplo IPv6**: *1:2:3:4:5:6:7:8 10:20:30:40:50:60:70:80* coincide con las solicitudes que llegan desde la dirección 1:2:3:4:5:6:7:8 o la 10:20:30:40:50:60:70:80.
- La sintaxis de un bloque de direcciones IP es la dirección IP de base seguida por una barra diagonal y el tamaño del prefijo. Por ejemplo:
  - **Ejemplo IPv4**: *5.5.5.64/26* coincide con las solicitudes que llegan desde la dirección 5.5.5.64 hasta la 5.5.5.127.
  - **Ejemplo IPv6**: *1:2:3:/48* coincide con las solicitudes que llegan desde la dirección 1:2:3:0:0:0:0:0 hasta la 1:2:3:ffff:ffff:ffff:ffff:ffff.

## <a name="request-body"></a>Cuerpo de la solicitud

Identifica las solicitudes en función del texto específico que aparece en el cuerpo de la solicitud.

#### <a name="required-fields"></a>Campos obligatorios

Operator | Cuerpo de la solicitud | Transformación de mayúsculas y minúsculas
---------|--------------|---------------
[Lista de operadores estándar](#standard-operator-list) | Cadena, Int | Minúsculas, mayúsculas, recortar, quitar espacio en blanco, codificación de URL, descodificación de URL

## <a name="request-header"></a>Encabezado de solicitud

Identifica las solicitudes que usan un encabezado específico en la solicitud.

#### <a name="required-fields"></a>Campos obligatorios

Nombre de encabezado | Operator | Valor de encabezado | Transformación de mayúsculas y minúsculas
------------|----------|--------------|---------------
String | [Lista de operadores estándar](#standard-operator-list) | Cadena, Int | Minúsculas, mayúsculas, recortar, quitar espacio en blanco, codificación de URL, descodificación de URL

## <a name="request-method"></a>Método de solicitud

Identifica las solicitudes que usan el método de solicitud especificado.

#### <a name="required-fields"></a>Campos obligatorios

Operator | Valores admitidos
---------|----------------
Es igual a, No es igual a | GET, POST, PUT, DELETE, HEAD, OPTIONS, TRACE

#### <a name="key-information"></a>Información importante

- Solo el método de solicitud GET puede generar contenido almacenado en caché en Azure Front Door. Los demás métodos de solicitud los procesa el proxy mediante la red. 

## <a name="request-protocol"></a>Protocolo de solicitud

Identifica las solicitudes que usan el protocolo especificado.

#### <a name="required-fields"></a>Campos obligatorios

Operator | Valores admitidos
---------|----------------
Es igual a, No es igual a | HTTP, HTTPS

## <a name="request-url"></a>URL de la solicitud

Identifica las solicitudes que coinciden con la dirección URL especificada.

#### <a name="required-fields"></a>Campos obligatorios

Operator | URL de la solicitud | Transformación de mayúsculas y minúsculas
---------|-------------|---------------
[Lista de operadores estándar](#standard-operator-list) | Cadena, Int | Minúsculas, mayúsculas, recortar, quitar espacio en blanco, codificación de URL, descodificación de URL

#### <a name="key-information"></a>Información importante

- Cuando use esta condición de regla, asegúrese de incluir la información del protocolo. Por ejemplo, *https://www.\<yourdomain\>.com*.

## <a name="request-file-extension"></a>Solicitar extensión de archivo

Identifica las solicitudes que incluyen la extensión de archivo especificada en el nombre de archivo de la dirección URL de la solicitud.

#### <a name="required-fields"></a>Campos obligatorios

Operator | Extensión | Transformación de mayúsculas y minúsculas
---------|-----------|---------------
[Lista de operadores estándar](#standard-operator-list) | Cadena, Int | Minúsculas, mayúsculas, recortar, quitar espacio en blanco, codificación de URL, descodificación de URL

#### <a name="key-information"></a>Información importante

- En el caso de la extensión, no incluya un punto inicial; por ejemplo, utilice *html* en lugar de *.html*.

## <a name="request-file-name"></a>Solicitar nombre de archivo

Identifica las solicitudes que incluyen el nombre de archivo especificado en la dirección URL de la solicitud.

#### <a name="required-fields"></a>Campos obligatorios

Operator | Nombre de archivo | Transformación de mayúsculas y minúsculas
---------|-----------|---------------
[Lista de operadores estándar](#standard-operator-list) | Cadena, Int | Minúsculas, mayúsculas, recortar, quitar espacio en blanco, codificación de URL, descodificación de URL

#### <a name="key-information"></a>Información importante

- Para especificar varios nombres de archivo, separe cada nombre de archivo presionando la tecla ENTRAR. 

## <a name="request-path"></a>Ruta de acceso de solicitud

Identifica las solicitudes que incluyen la ruta de acceso especificada en la dirección URL de la solicitud.

#### <a name="required-fields"></a>Campos obligatorios

Operator | Value | Transformación de caso
---------|-------|---------------
[Lista de operadores estándar](#standard-operator-list) | Cadena, Int | Minúsculas, mayúsculas, recortar, quitar espacio en blanco, codificación de URL, descodificación de URL

## <a name="standard-operator-list"></a>Lista de operadores estándar

En el caso de las reglas que aceptan valores de la lista de operadores estándar, son válidos los siguientes operadores:

- Any
- Equals 
- Contains 
- Empieza por 
- Termina por 
- Menor que
- Menor o igual que
- Mayor que
- Mayor o igual que
- Ninguno
- No contiene
- No empieza por 
- No termina con 
- No es menor que
- No es menor o igual que
- No es mayor que
- No es mayor o igual que

En el caso de los operadores numéricos, como *Menor que* o *Mayor o igual que*, la comparación usada se basa en la longitud. En este caso, el valor de la condición de coincidencia debe ser un entero, igual a la longitud que quiere comparar. 


## <a name="next-steps"></a>Pasos siguientes

- Aprenda a configurar su primera [configuración del motor de reglas](front-door-tutorial-rules-engine.md). 
- Obtenga más información sobre las [acciones del motor de reglas](front-door-rules-engine-actions.md).
- Obtenga más información acerca del [motor de reglas de Azure Front Door](front-door-rules-engine.md).
