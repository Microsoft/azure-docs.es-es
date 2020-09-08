---
title: Variables HTTP para el motor de reglas de Azure CDN | Microsoft Docs
description: Obtenga información sobre las variables HTTP, que le permiten obtener metadatos de solicitud y respuesta HTTP para algunas características del motor de reglas. Use metadatos para modificar una solicitud o respuesta.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/09/2018
ms.author: allensu
ms.openlocfilehash: a2d9fc98ba6f514afbd88e543a859a69e0fc6c6b
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88192678"
---
# <a name="http-variables-for-azure-cdn-rules-engine"></a>Variables HTTP para el motor de reglas de Azure CDN
Las variables HTTP proporcionan los medios para recuperar metadatos de respuesta y solicitudes HTTP. Estos metadatos pueden usarse después para alterar una solicitud o respuesta de forma dinámica. El uso de las variables HTTP se restringe a las siguientes características del motor de reglas:

- [Reescritura de clave de caché](https://docs.vdms.com/cdn/Content/HRE/F/Cache-Key-Rewrite.htm)
- [Modificar encabezado de solicitud de cliente](https://docs.vdms.com/cdn/Content/HRE/F/Modify-Client-Request-Header.htm)
- [Modificar encabezado de respuesta de cliente](https://docs.vdms.com/cdn/Content/HRE/F/Modify-Client-Response-Header.htm)
- [Redirección de direcciones URL](https://docs.vdms.com/cdn/Content/HRE/F/URL-Redirect.htm)
- [Reescritura de direcciones URL](https://docs.vdms.com/cdn/Content/HRE/F/URL-Rewrite.htm)

## <a name="definitions"></a>Definiciones
En la tabla siguiente se describen las variables HTTP compatibles. Se devuelve un valor en blanco cuando los metadatos de replicación geográfica (por ejemplo, el código postal) no están disponibles para una solicitud concreta.


| Nombre | Variable | Descripción | Valor de ejemplo |
| ---- | -------- | ----------- | ------------ |
| ASN (solicitante) | %{geo_asnum} | Indica el número de sistema autónomo (AS) del solicitante. <br /><br />**En desuso**: %{virt_dst_asnum}. <br />Esta variable ha quedado en desuso y se ha reemplazado por %{geo_asnum}. Aunque una regla que usa esta variable en desuso continuará funcionando, debe actualizarla para usar la nueva variable. | AS15133 |
| Ciudad (solicitante) | %{geo_city} | Indica la ciudad del solicitante. | Los Angeles |
| Continente (solicitante) | %{geo_continent} | Indica el continente del solicitante mediante su abreviatura. <br />Los valores válidos son: <br />AF: África<br />AS: Asia<br />EU: Europa<br />N/D: Norteamérica<br />OC: Oceanía<br />SA: Sudamérica<br /><br />**En desuso**: %{virt_dst_continent}. <br />Esta variable ha quedado en desuso y se ha reemplazado por %{geo_continent}. <br />Aunque una regla que usa esta variable en desuso continuará funcionando, debe actualizarla para usar la nueva variable.| N/D |
| Valor de la cookie | %{cookie_Cookie} | Devuelve el valor correspondiente a la clave de cookie identificada por el término Cookie. | Ejemplo de uso: <br />%{cookie__utma}<br /><br />Valor de ejemplo:<br />111662281.2.10.1222100123 |
| País o región (solicitante) | %{geo_country} | Indica el país o región de origen del solicitante mediante su código de país o región. <br />**En desuso**: %{virt_dst_country}. <br /><br />Esta variable ha quedado en desuso y se ha reemplazado por %{geo_country}. Aunque una regla que usa esta variable en desuso continuará funcionando, debe actualizarla para usar la nueva variable. | US |
| Área de mercado designada (solicitante) | %{geo_dma_code} |Indica el mercado de medios del solicitante mediante su código de región. <br /><br />Este campo solo es aplicable a las solicitudes que se originan en los Estados Unidos.| 745 |
| Método de solicitud HTTP | %{request_method} | Indica el método de solicitud HTTP. | GET |
| Código de estado HTTP | %{status} | Indica el código de estado HTTP de la respuesta. | 200 |
| Dirección IP (solicitante) | %{virt_dst_addr} | Indica la dirección IP del solicitante. | 192.168.1.1 |
| Latitud (solicitante) | %{geo_latitude} | Indica la latitud del solicitante. | 34.0995 |
| Longitud (solicitante) | %{geo_longitude} | Indica la longitud del solicitante. | -118.4143 |
| Área estadística metropolitana (solicitante) | %{geo_metro_code} | Indica el área metropolitana del solicitante. <br /><br />Este campo solo es aplicable a las solicitudes que se originan en los Estados Unidos.<br />| 745 |
| Puerto (solicitante) | %{virt_dst_port} | Indica el puerto efímero del solicitante. | 55885 |
| Código postal (solicitante) | %{geo_postal_code} | Indica el código postal del solicitante. | 90210 |
| Cadena de consulta encontrada | %{is_args} | El valor de esta variable varía según si la solicitud contiene una cadena de consulta.<br /><br />- Cadena de consulta encontrada: ?<br />- Sin cadena de consulta: NULL | ? |
| Parámetro de cadena de consulta encontrado | %{is_amp} | El valor de esta variable varía según si la solicitud contiene al menos un parámetro de cadena de consulta.<br /><br />- Parámetro encontrado: &<br />- Sin parámetros: NULL | & |
| Valor de parámetro de cadena de consulta | %{arg_&lt;parameter&gt;} | Devuelve el valor correspondiente al parámetro de cadena de consulta identificado por el término &lt;parámetro&gt;. | Ejemplo de uso: <br />%{arg_language}<br /><br />Parámetro de cadena de consulta de ejemplo: <br />?language=es<br /><br />Valor de ejemplo: es |
| Valor de cadena de consulta | %{query_string} | Indica el valor de cadena de consulta completo definido en la dirección URL de la solicitud. |key1=val1&key2=val2&key3=val3 |
| Dominio de origen de referencia | %{referring_domain} | Indica el dominio definido en el encabezado de solicitud del origen de referencia. | <www.google.com> |
| Región (solicitante) | %{geo_region} | Indica la región del solicitante (por ejemplo, estado o provincia) mediante su abreviatura alfanumérica. | CA |
| Valor de encabezado de solicitud | %{http_RequestHeader} | Devuelve el valor correspondiente al encabezado de solicitud identificado por el término RequestHeader. <br /><br />Si el nombre del encabezado de solicitud contiene un guión (por ejemplo, Usuario-Agente), reemplácelo por un guión bajo (por ejemplo, Usuario_Agente).| Ejemplo de uso: %{http_Connection}<br /><br />Valor de ejemplo: Keep-Alive | 
| Host de solicitud | %{host} | Indica el host definido en la dirección URL de la solicitud. | <www.mydomain.com> |
| Protocolo de solicitud | %{request_protocol} | Indica el protocolo de la solicitud. | HTTP/1.1 |
| Esquema de solicitud | %{scheme} | Indica el esquema de solicitud. |http |
| URI de solicitud (relativa) | %{request_uri} | Indica la ruta de acceso relativa, incluida la cadena de consulta, definida en el URI de solicitud. | /marketing/foo.js?loggedin=true |
| URI de solicitud (relativa sin cadena de consulta) | %{uri} | Indica la ruta de acceso relativa al contenido solicitado. <br /><br/>Información importante:<br />- Esta ruta de acceso relativa excluye la cadena de consulta.<br />- Esta ruta de acceso relativa refleja las reescrituras de la dirección URL. Una dirección URL se reescribirá en las siguientes condiciones:<br />  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- Característica de reescritura de direcciones URL: esta característica reescribe la ruta de acceso relativa definida en el URI de solicitud.<br />    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- Dirección URL de CNAME perimetral: este tipo de solicitud se reescribe para la dirección URL de CDN correspondiente. |/800001/corigin/rewrittendir/foo.js |
| URI de solicitud | %{request} | Describe la solicitud. <br />Sintaxis: &lt;Método HTTP&gt; &lt;Ruta de acceso relativa&gt; &lt;Protocolo HTTP&gt; | GET /marketing/foo.js?loggedin=true HTTP/1.1 |
| Valor de encabezado de respuesta | %{resp_&lt;ResponseHeader&gt;} | Devuelve el valor correspondiente al encabezado de respuesta identificado por el término &lt;ResponseHeader&gt;. <br /><br />Si el nombre del encabezado de respuesta contiene un guión (por ejemplo, Usuario-Agente), reemplácelo por un guión bajo (por ejemplo, Usuario_Agente). | Ejemplo de uso: %{resp_Content_Length}<br /><br />Valor de ejemplo: 100 |

## <a name="usage"></a>Uso
En la tabla siguiente se describe la sintaxis correcta para especificar una variable HTTP.


| Sintaxis | Ejemplo | Descripción |
| ------ | -------- | ---------- |
| %{&lt;HTTPVariable&gt;} | %{host} | Use esta sintaxis para obtener el valor completo correspondiente a la variable &lt;HTTPVariable&gt; especificada. |
| %{&lt;HTTPVariableDelimiter&gt;} | %{host,} | Use esta sintaxis para definir el valor completo correspondiente al valor de &lt;HTTPVariableDelimiter&gt; especificado. |
| %{&lt;HTTPVariableDelimiterExpression&gt;} | %{host/=^www\.([^\.]+)\.([^\.:]+)/cdn.$2.$3:80} | Use una expresión regular para &lt;HTTPVariableDelimiterExpression&gt;, para reemplazar, eliminar o manipular un valor de la variable HTTP. |

Los nombres de variables HTTP solo admiten guiones bajos y caracteres alfabéticos. Convierta los caracteres no admitidos en guiones bajos.

## <a name="delimiter-reference"></a>Referencia de delimitador
Se puede especificar un delimitador después de una variable HTTP para lograr cualquiera de los siguientes efectos:

- Transformar el valor asociado a la variable.

     Ejemplo: Convertir en minúscula todo el valor.

- Eliminar el valor asociado a la variable.

- Manipular el valor asociado a la variable.

     Ejemplo: use expresiones regulares para cambiar el valor asociado a la variable HTTP.

Los delimitadores se describen en la tabla siguiente.

| Delimitador | Descripción |
| --------- | ----------- |
| := | Indica que se asignará un valor predeterminado a la variable cuando: <br />- Falte. <br />- Se establezca en NULL. |
| :+ | Indica que se asignará un valor predeterminado a la variable cuando se le haya asignado un valor. |
| : | Indica que se expandirá una subcadena del valor asignado a la variable. |
| # | Indica que se debe eliminar el patrón especificado después de este delimitador cuando se encuentre al principio del valor asociado con la variable. |
| % | Indica que se debe eliminar el patrón especificado después de este delimitador cuando se encuentre al final del valor asociado con la variable. <br />Esta definición solo es aplicable si se usa el símbolo % como delimitador. |
| / | Delimita una variable HTTP o un patrón. |
| // | Buscar y reemplazar todas las instancias del patrón especificado. |
| /= | Busque, copie y reescriba todas las instancias del patrón especificado. |
| , | Convertir en minúscula el valor asociado con la variable HTTP. |
| ^ | Convertir en mayúscula el valor asociado con la variable HTTP. |
| ,, | Convertir en minúscula todas las instancias del carácter especificado en el valor asociado a la variable HTTP. |
| ^^ | Convertir en mayúscula todas las instancias del carácter especificado en el valor asociado a la variable HTTP. |

## <a name="exceptions"></a>Excepciones
En la tabla siguiente se describen las circunstancias en que el texto especificado no se trata como una variable HTTP.

| Condición | Descripción | Ejemplo |
| --------- | ----------- | --------|
| Símbolo de escape % | El símbolo de porcentaje se puede escapar mediante el uso de una barra diagonal inversa. <br />El valor de ejemplo de la derecha se tratará como un valor literal y no como una variable HTTP.| \%{host} |
| Variables desconocidas | Siempre se devuelve una cadena vacía para variables desconocidas. | %{unknown_variable} |
| Sintaxis o caracteres no válidos | Las variables que contienen sintaxis o caracteres no válidos se tratan como valores literales. <br /><br />Ejemplo 1: el valor especificado contiene un carácter no válido (por ejemplo, -). <br /><br />Ejemplo 2: el valor especificado contiene un conjunto doble de llaves. <br /><br />Ejemplo 3: al valor especificado le falta una llave de cierre.<br /> | Ejemplo 1: %{resp_user-agent} <br /><br />Ejemplo 2: %{{host}} <br /><br />Ejemplo 3: %{host |
| Falta el nombre de variable | Siempre se devuelve un valor NULL cuando no se especifica una variable. | %{} |
| Caracteres finales | Caracteres que finalizan una variable se tratan como valores literales. <br />El valor de ejemplo de la derecha contiene una llave final que se tratará como un valor literal. | %{host}} |

## <a name="setting-default-header-values"></a>Definición de valores de encabezado predeterminados
Un valor predeterminado puede asignarse a un encabezado cuando se cumple alguna de las condiciones siguientes:
- Ausencia/sin definir
- Definición en NULL

En la tabla siguiente se describe cómo definir un valor predeterminado.

| Condición | Sintaxis | Ejemplo | Descripción |
| --------- | ------ | --------| ----------- |
| Establecer un encabezado en un valor predeterminado si se cumple alguna de las siguientes condiciones: <br /><br />- Falta el encabezado <br /><br />- El valor de encabezado está establecido en NULL.| %{Variable:=Value} | %{http_referrer:=unspecified} | El encabezado del origen de referencia solo se establecerá en *unspecified* en caso de que falte o de que esté establecido en NULL. En caso de que se haya definido, no se llevará a cabo ninguna acción. |
| Establecer un encabezado en un valor predeterminado cuando está ausente. | %{Variable=Value} | %{http_referrer=unspecified} | El encabezado del origen de referencia solo se establecerá en *unspecified* en caso de que falte. En caso de que se haya definido, no se llevará a cabo ninguna acción. |
| Establecer el encabezado en un valor predeterminado si no se cumple alguna de las siguientes condiciones: <br /><br />- Falte.<br /><br /> - Se establezca en NULL. | %{Variable:+Value} | %{http_referrer:+unspecified} | El encabezado del origen de referencia solo se establecerá en *unspecified* si se le ha asignado algún valor. No se llevará a cabo ninguna acción si falta o está establecido en NULL. |

## <a name="manipulating-variables"></a>Manipulación de variables
Las variables se pueden manipular de las maneras siguientes:

- Expansión de subcadenas
- Eliminación de patrones

### <a name="substring-expansion"></a>Expansión de subcadena
De forma predeterminada, se expandirá una variable a su valor total. Use la siguiente sintaxis para expandir solo una subcadena del valor de la variable.

`%<Variable>:<Offset>:<Length>}`

Información importante:

- El valor asignado al término de desplazamiento determina el carácter inicial de la subcadena:

     - Positivo: se calcula el carácter inicial de la subcadena desde el primer carácter de la cadena.
     - Cero: el carácter inicial de la subcadena es el primer carácter de la cadena.
     - Negativo: se calcula el carácter inicial de la subcadena desde el último carácter de la cadena.

- La longitud de la subcadena se determina con el término *Longitud*:

     - Omitido: la omisión del término Longitud permite que la subcadena incluya todos los caracteres entre el carácter inicial y el final de la cadena.
     - Positivo: determina la longitud de la subcadena desde el carácter inicial de la derecha.
     - Negativo: determina la longitud de la subcadena desde el carácter inicial de la izquierda.

#### <a name="example"></a>Ejemplo:

El siguiente ejemplo se basa en la siguiente dirección URL de solicitud de ejemplo:

https:\//cdn.midominio.com/carpeta/marketing/miconsultor/propuesta.html

La cadena siguiente muestra varios métodos para manipular las variables:

https:\//www%{http_host:3}/mobile/%{request_uri:7:10}/%{request_uri:-5:-8}.htm

Según la dirección URL de solicitud de ejemplo, la manipulación de la variable anterior generará el siguiente valor:

https:\//www.midominio.com/móvil/marketing/propuesta.htm


### <a name="pattern-removal"></a>Eliminación de patrón
El texto que coincide con un patrón específico puede quitarse del principio o el final del valor de una variable.

| Sintaxis | Acción |
| ------ | ------ |
| %{Variable#Pattern} | Quitar el texto cuando el patrón especificado se encuentre al principio del valor de una variable. |
| %{Variable%Pattern} | Quitar el texto cuando el patrón especificado se encuentre al final del valor de una variable. |

#### <a name="example"></a>Ejemplo:

En este escenario de ejemplo, la variable *request_uri* se establece en:

`/800001/myorigin/marketing/product.html?language=en-US`

En la tabla siguiente se muestra cómo funciona esta sintaxis.

| Sintaxis de ejemplo | Results | Descripción |
| ------------- | ------- | --- |
| %{request_uri#/800001}/customerorigin | /customerorigin/myorigin/marketing/product.html?language=en-US | Dado que la variable empieza con el patrón, se ha reemplazado. |
| %{request_uri%html}htm | /800001/myorigin/marketing/product.html?language=en-US | Dado que la variable no termina con el patrón, no se ha introducido ningún cambio.|

### <a name="find-and-replace"></a>Buscar y reemplazar
La sintaxis de buscar y reemplazar se describe en la tabla siguiente.

| Sintaxis | Acción |
| ------ | ------ |
| %{Variable/Find/Replace} | Buscar y reemplazar la primera ocurrencia del patrón especificado. |
| %{Variable//Find/Replace} | Buscar y reemplazar todas las ocurrencias del patrón especificado. |
| %{Variable^} |Convertir todo el valor a mayúscula. |
| %{Variable^Find} | Convertir en mayúscula la primera ocurrencia del patrón especificado. |
| %{Variable,} | Convertir en minúscula todo el valor. |
| %{Variable,Find} | Convertir en minúscula la primera ocurrencia del patrón especificado. |

### <a name="find-and-rewrite"></a>Buscar y reescribir
Para una variación en Buscar y reemplazar, utilice el texto que coincide con el patrón especificado al reescribirlo. La sintaxis de buscar y reescribir se describe en la tabla siguiente.

| Sintaxis | Acción |
| ------ | ------ |
| %{Variable/=Find/Rewrite} | Busque, copie y reescriba todas las instancias del patrón especificado. |
| %{Variable/^Find/Rewrite} | Buscar, copiar y reescribir el patrón especificado cuando se produce al principio de la variable. |
| %{Variable/$Find/Rewrite} | Buscar, copiar y reescribir el patrón especificado cuando se produce al final de la variable. |
| %{Variable/Find} | Buscar y eliminar todas las ocurrencias del patrón especificado. |

Información importante:

- Expandir el texto que coincide con el patrón especificado mediante la especificación de un signo de dólar seguido de un número entero (por ejemplo, $1).

- Se pueden especificar varios patrones. El orden en que se especifica el patrón determina el entero que se le asignará. En el ejemplo siguiente, el primer patrón coincide con "www"., el segundo patrón coincide con el dominio de segundo nivel y el tercer patrón coincide con el dominio de nivel superior:

    `%{host/=^www\.([^\.]+)\.([^\.:]+)/cdn.$2.$3:80}`

- El valor de reescritura puede constar de una combinación de texto y de estos marcadores de posición.

    En el ejemplo anterior, el nombre de host se reescribe como `cdn.$2.$3:80` (por ejemplo, cdn.midominio.com:80).

- El caso de un marcador de posición de patrón (por ejemplo, $1) puede modificarse con las siguientes marcas:
     - U: en mayúscula el valor expandido.

         Sintaxis de ejemplo:

         `%{host/=^www\.([^\.]+)\.([^\.:]+)/cdn.$U2.$3:80}`

     - L: en minúscula el valor expandido.

         Sintaxis de ejemplo:

         `%{host/=^www\.([^\.]+)\.([^\.:]+)/cdn.$L2.$3:80}`

- Un operador debe especificarse antes del patrón. El operador especificado determina el comportamiento de captura del patrón:

     - `=`: indica que se deben capturar y reescribir todas las apariciones del patrón especificado.
     - `^`: indica que solo se capturará el texto que empieza con el patrón especificado.
     - `$`: indica que solo se capturará el texto que termina con el patrón especificado.
 
- Si omite el valor */Rewrite*, se eliminará el texto que coincide con el patrón.


