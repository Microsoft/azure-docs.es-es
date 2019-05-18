---
title: Condiciones de coincidencia del motor de reglas de Azure CDN | Microsoft Docs
description: Documentación de referencia sobre las condiciones de coincidencia del motor de reglas de Azure Content Delivery Network
services: cdn
documentationcenter: ''
author: Lichard
manager: akucer
editor: ''
ms.assetid: 669ef140-a6dd-4b62-9b9d-3f375a14215e
ms.service: cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/21/2017
ms.author: rli
ms.openlocfilehash: f87667516d6c76bc1c21ba5e175b3b2a7ebc3f39
ms.sourcegitcommit: 9e8dfa1169a55c3c8af93a6c5f4e0dace4de48b2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65556327"
---
# <a name="azure-cdn-rules-engine-match-conditions"></a>Condiciones de coincidencia del motor de reglas de Azure CDN 
En este artículo se muestran descripciones detalladas de las condiciones de coincidencia disponibles para el [motor de reglas](cdn-rules-engine.md) de Azure Content Delivery Network (CDN).

La segunda parte de una regla es la condición de coincidencia. Una condición de coincidencia identifica tipos específicos de solicitudes para los que se ejecutará un conjunto de características.

Por ejemplo, puede usar una condición de coincidencia para:
- Filtrar las solicitudes de contenido de una ubicación concreta.
- Filtrar las solicitudes procedentes de una determinada dirección IP o país o región.
- Filtrar las solicitudes por la información de encabezado.

## <a name="always-match-condition"></a>Condición de coincidencia Siempre

La condición de coincidencia Siempre aplica un conjunto predeterminado de características a todas las solicitudes.

NOMBRE | Propósito
-----|--------
[Siempre](#always) | Se aplica un conjunto predeterminado de características a todas las solicitudes.

## <a name="device-match-condition"></a>Condición de coincidencia Dispositivo

La condición de coincidencia Dispositivo identifica solicitudes realizadas desde un dispositivo móvil en función de sus propiedades.  

NOMBRE | Propósito
-----|--------
[Device](#device) | Identifica solicitudes realizadas desde un dispositivo móvil en función de sus propiedades.

## <a name="location-match-conditions"></a>Condiciones de coincidencia Ubicación

Las condiciones de coincidencia Ubicación identifican solicitudes en función de la ubicación del solicitante.

NOMBRE | Propósito
-----|--------
[Número de sistema autónomo (AS)](#as-number) | Identifica solicitudes que se originan en una red determinada.
[País](#country) | Identifica solicitudes que se originan en los países o regiones especificados.

## <a name="origin-match-conditions"></a>Condiciones de coincidencia Origen

Las condiciones de coincidencia Origen identifican solicitudes que señalan al servidor de origen de un cliente o de almacenamiento de Content Delivery Network.

NOMBRE | Propósito
-----|--------
[Origen de red CDN](#cdn-origin) | Identifica las solicitudes de contenido almacenado en el almacenamiento de Content Delivery Network.
[Origen de cliente](#customer-origin) | Identifica solicitudes de contenido almacenado en el servidor de origen de un cliente específico.

## <a name="request-match-conditions"></a>Condiciones de coincidencia Solicitud

Las condiciones de coincidencia Solicitud identifican solicitudes en función de sus propiedades.

NOMBRE | Propósito
-----|--------
[Dirección IP de cliente](#client-ip-address) | Identifica solicitudes que se originan en una dirección IP determinada.
[Parámetro de cookie](#cookie-parameter) | Busca el valor especificado en las cookies asociadas a cada solicitud.
[Regex de parámetro de cookie](#cookie-parameter-regex) | Busca la expresión regular especificada en las cookies asociadas a cada solicitud.
[Cname perimetral](#edge-cname) | Identifica solicitudes que señalan a un CNAME perimetral específico .
[Dominio de referencia](#referring-domain) | Identifica solicitudes a las que se haga referencia en los nombres de host especificados.
[Literal de encabezado de solicitud](#request-header-literal) | Identifica solicitudes que contienen el encabezado que se especifique en un valor concreto.
[Regex de encabezado de solicitud](#request-header-regex) | Identifica solicitudes que contienen el encabezado que se especifique establecido en la expresión regular especificada.
[Carácter comodín de encabezado de solicitud](#request-header-wildcard) | Identifica solicitudes que contienen el encabezado que se especifique establecido en un valor que coincide con el patrón especificado.
[Método de solicitud](#request-method) | Identifica solicitudes por su método HTTP.
[Esquema de solicitud](#request-scheme) | Identifica solicitudes por su protocolo HTTP.

## <a name="url-match-conditions"></a>Condiciones de coincidencia URL

Las condiciones de coincidencia URL identifican solicitudes en función de sus direcciones URL.

NOMBRE | Propósito
-----|--------
[Directorio de la ruta de acceso URL](#url-path-directory) | Identifica solicitudes por su ruta de acceso relativa.
[Extensión de la ruta de acceso URL](#url-path-extension) | Identifica solicitudes por su extensión de nombre de archivo.
[Nombre de archivo de la ruta de acceso URL](#url-path-filename) | Identifica solicitudes por su nombre de archivo.
[Literal de la ruta de acceso URL](#url-path-literal) | Compara la ruta de acceso relativa de una solicitud con el valor especificado.
[Regex de la ruta de acceso URL](#url-path-regex) | Compara la ruta de acceso relativa de una solicitud con la expresión regular especificada.
[Carácter comodín de la ruta de acceso URL](#url-path-wildcard) | Compara la ruta de acceso relativa de una solicitud con el patrón especificado.
[Literal de la consulta URL](#url-query-literal) | Compara la cadena de consulta de una solicitud con el valor especificado.
[Parámetro de la consulta URL](#url-query-parameter) | Identifica solicitudes que contienen el parámetro de cadena de consulta que se especifique establecido en un valor que coincide con el patrón especificado.
[Regex de la consulta URL](#url-query-regex) | Identifica solicitudes que contienen el parámetro de cadena de consulta que se especifique establecido en un valor que coincide con la expresión regular especificada.
[Carácter comodín de la consulta URL](#url-query-wildcard) | Compara los valores especificados con la cadena de consulta de la solicitud.


## <a name="reference-for-rules-engine-match-conditions"></a>Referencia de las condiciones de coincidencia del motor de reglas

<a name="main"></a>

---

### <a name="always"></a>Siempre

La condición de coincidencia Siempre aplica un conjunto predeterminado de características a todas las solicitudes.

[Volver arriba](#main)

</br>

---
### <a name="as-number"></a>Número de sistema autónomo (AS) 
La red del número de sistema autónomo se define por el número de sistema autónomo (ASN). 

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Número de sistema autónomo (AS):
- **Matches** (Coincide): requiere que el ASN de la red del cliente coincida con uno de los ASN especificados. 
- **Does Not Match** (No coincide): requiere que el ASN de la red del cliente no coincida con ninguno de los ASN especificados.

Información importante:
- Especifique varios números de sistema autónomo mediante la delimitación de cada uno con un espacio. Por ejemplo, 64514 64515 coincide con las solicitudes que llegan desde 64514 o 64515.
- Algunas solicitudes no pueden devolver un ASN válido. Un signo de interrogación (?) coincidirá con las solicitudes para las que no se pueda determinar un ASN válido.
- Especifique el ASN completo para la red deseada. No se encontrarán coincidencias con valores parciales.
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
  - Relleno de la memoria caché completa
  - Max-Age interna predeterminada
  - Forzar Max-Age interna
  - Ignorar no almacenar en caché de origen
  - Max-Stale interna

[Volver arriba](#main)

</br>

---
### <a name="cdn-origin"></a>Origen de red CDN
La condición de coincidencia Origen de red CDN se cumple cuando lo hacen ambas condiciones siguientes:
- Se solicitó contenido del almacenamiento de la red CDN.
- El identificador URI de la solicitud usa el tipo de punto de acceso al contenido (por ejemplo, /000001) que se define en esta condición de coincidencia:
  - Dirección URL de la red CDN: el identificador URI de la solicitud debe contener el punto de acceso al contenido seleccionado.
  - Dirección URL de CNAME perimetral: la configuración de CNAME perimetral correspondiente debe apuntar al punto de acceso al contenido seleccionado.
  
Información importante:
 - El punto de acceso al contenido identifica el servicio que debe atender el contenido solicitado.
 - No use una instrucción AND IF para combinar determinadas condiciones de coincidencia. Por ejemplo, la combinación de una condición de coincidencia Origen de red CDN con una Origen de cliente crearía un patrón de coincidencia que no se puede cumplir. Por esta razón, no se pueden combinar dos condiciones de coincidencia Origen de red CDN mediante una instrucción AND IF.

[Volver arriba](#main)

</br>

---
### <a name="client-ip-address"></a>Dirección IP del cliente
La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Dirección IP del cliente:
- **Matches** (Coincide): requiere que la dirección IP del cliente coincida con una de las direcciones IP especificadas. 
- **Does Not Match** (No coincide): requiere que la dirección IP del cliente no coincida con ninguna de las direcciones IP especificadas. 

Información importante:
- Utilice la notación CIDR.
- Especifique varias direcciones IP o bloques de direcciones IP al delimitarlas individualmente con un espacio. Por ejemplo:
  - **Ejemplo IPv4**: 1.2.3.4 10.20.30.40 coincide con las solicitudes que llegan desde la dirección 1.2.3.4 o 10.20.30.40.
  - **Ejemplo IPv6**: 1:2:3:4:5:6:7:8 10:20:30:40:50:60:70:80 coincide con las solicitudes que llegan desde la dirección 1:2:3:4:5:6:7:8 o 10:20:30:40:50:60:70:80.
- La sintaxis de un bloque de direcciones IP es la dirección IP de base seguida por una barra diagonal y el tamaño del prefijo. Por ejemplo:
  - **Ejemplo IPv4**: 5.5.5.64/26 coincide con las solicitudes que llegan desde la dirección 5.5.5.64 hasta la 5.5.5.127.
  - **Ejemplo IPv6**: 1:2:3:/48 coincide con las solicitudes que llegan desde la dirección 1:2:3:0:0:0:0:0 hasta la 1:2:3:ffff:ffff:ffff:ffff:ffff.
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
  - Relleno de la memoria caché completa
  - Max-Age interna predeterminada
  - Forzar Max-Age interna
  - Ignorar no almacenar en caché de origen
  - Max-Stale interna

[Volver arriba](#main)

</br>

---
### <a name="cookie-parameter"></a>Parámetro de cookie
La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Parámetro de cookie.
- **Matches** (Coincide): requiere que una solicitud contenga la cookie especificada con un valor que coincida con al menos uno de los que se definen en esta condición de coincidencia.
- **Does Not Match** (No coincide): requiere que la solicitud cumpla cualquiera de los siguientes criterios:
  - No contenga la cookie especificada.
  - Contenga la cookie especificada, pero que su valor no coincida con ninguno de los que se definen en esta condición de coincidencia.
  
Información importante:
- Nombre de la cookie: 
  - Dado que no se admiten caracteres comodín, incluidos los asteriscos (*), al especificar un nombre de cookie, solo las coincidencias de nombre de cookie exacto son aptas para la comparación.
  - Solo se puede especificar un nombre de cookie por instancia de esta condición de coincidencia.
  - Las comparaciones de nombres de cookie distinguen entre mayúsculas y minúsculas.
- Valor de la cookie: 
  - Especifique varios valores de cookie mediante la delimitación de cada uno con un espacio.
  - Un valor de cookie puede aprovechar las ventajas de los [valores de carácter comodín](cdn-rules-engine-reference.md#wildcard-values). 
  - Si no se ha especificado un valor de carácter comodín, solo una coincidencia exacta cumplirá esta condición de coincidencia. Por ejemplo, especificar que "Value" coincida con "Value", pero no "Value1" o "Value2".
  - Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si en la comparación se distinguirán mayúsculas y minúsculas en el valor de la cookie de la solicitud.
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
  - Relleno de la memoria caché completa
  - Max-Age interna predeterminada
  - Forzar Max-Age interna
  - Ignorar no almacenar en caché de origen
  - Max-Stale interna

[Volver arriba](#main)

</br>

---
### <a name="cookie-parameter-regex"></a>Regex de parámetro de cookie
La condición de coincidencia Regex de parámetro de cookie define el valor y el nombre de la cookie. Puede utilizar [expresiones regulares](cdn-rules-engine-reference.md#regular-expressions) para definir el valor de la cookie deseado. 

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Regex de parámetro de cookie.
- **Matches** (Coincide): identifica solicitudes que contienen la cookie determinada con un valor que coincida con la expresión regular especificada.
- **Does Not Match** (No coincide): requiere que la solicitud cumpla cualquiera de los siguientes criterios:
  - No contenga la cookie especificada.
  - Contenga la cookie especificada, pero que su valor no coincida con la expresión regular especificada.
  
Información importante:
- Nombre de la cookie: 
  - Dado que no se admiten expresiones regulares ni caracteres comodín, incluidos los asteriscos (*), al especificar un nombre de cookie, solo las coincidencias de nombre de cookie exacto son aptas para la comparación.
  - Solo se puede especificar un nombre de cookie por instancia de esta condición de coincidencia.
  - Las comparaciones de nombres de cookie distinguen entre mayúsculas y minúsculas.
- Valor de la cookie: 
  - Para los valores de cookie se pueden utilizar expresiones regulares.
  - Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si en la comparación se distinguirán mayúsculas y minúsculas en el valor de la cookie de la solicitud.
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
  - Relleno de la memoria caché completa
  - Max-Age interna predeterminada
  - Forzar Max-Age interna
  - Ignorar no almacenar en caché de origen
  - Max-Stale interna

[Volver arriba](#main)

</br>

---
### <a name="country"></a>País
Puede especificar un país o región a través de su código de país. 

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia País:
- **Matches** (Coincide): requiere que la solicitud contenga los valores de código de país especificados. 
- **Does Not Match** (No coincide): requiere que la solicitud no contenga los valores de código de país especificados.

Información importante:
- Especifique varios códigos de país mediante la delimitación de cada uno con un espacio.
- No se admiten caracteres comodín al especificar un código de país.
- Los códigos de país "EU" y "AP" no incluyen todas las direcciones IP de esas regiones.
- Algunas solicitudes pueden no devolver un código de país válido. Un signo de interrogación (?) coincidirá con las solicitudes para las que no se pueda determinar un código de país válido.
- Los códigos de país distinguen entre mayúsculas y minúsculas.
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
  - Relleno de la memoria caché completa
  - Max-Age interna predeterminada
  - Forzar Max-Age interna
  - Ignorar no almacenar en caché de origen
  - Max-Stale interna

#### <a name="implementing-country-filtering-by-using-the-rules-engine"></a>Implementación del filtrado por país utilizando el motor de reglas
Esta condición de coincidencia permite realizar un gran número de personalizaciones en función de la ubicación desde la que se originó una solicitud. Por ejemplo, el comportamiento de la característica de filtrado por país puede replicarse a través de la configuración siguiente:

- Coincidencia de caracteres comodín de la ruta de URL: establezca la [condición de coincidencia de caracteres comodín de la ruta de URL](#url-path-wildcard) en el directorio que se va a proteger. 
    Anexe un asterisco al final de la ruta de acceso relativa para asegurarse de que el acceso a todos sus elementos secundarios se limitará mediante esta regla.

- Coincidencia de país: Establece la condición de coincidencia de país en el conjunto deseado de países o regiones.
   - Permitir: Establezca la condición de coincidencia de país en **no coincide con** para permitir únicamente el acceso de países o regiones especificados al contenido almacenado en la ubicación definida por la condición de coincidencia de caracteres comodín de ruta de acceso de dirección URL.
   - Bloquear: Establezca la condición de coincidencia de país en **coincidencias** para bloquear los países o regiones especificados tengan acceso a contenido almacenado en la ubicación definida por la condición de coincidencia de caracteres comodín de ruta de acceso de dirección URL.

- Característica de denegación de acceso (403): habilite la [característica de denegación de acceso (403)](cdn-rules-engine-reference-features.md#deny-access-403) para replicar la parte permitidos o bloqueados de la característica de filtrado por país.

[Volver arriba](#main)

</br>

---
### <a name="customer-origin"></a>Origen de cliente

Información importante: 
- La condición de coincidencia Origen de cliente se cumplirá independientemente de si se solicita contenido mediante una dirección URL de la red CDN o una dirección URL CNAME perimetral que apunte al origen del cliente seleccionado.
- No se puede eliminar de la página Origen de cliente una configuración de origen de cliente a la que haga referencia una regla. Antes de intentar eliminar una configuración de origen de cliente, asegúrese de que las siguientes configuraciones de no hacen referencia a ella:
  - Una condición de coincidencia Origen de cliente
  - Una configuración de CNAME perimetral
- No use una instrucción AND IF para combinar determinadas condiciones de coincidencia. Por ejemplo, la combinación de una condición de coincidencia Origen de cliente con una Origen de red CDN crearía un patrón de coincidencia que no se puede cumplir. Por esta razón, no se pueden combinar dos condiciones de coincidencia Origen de cliente mediante una instrucción AND IF.

[Volver arriba](#main)

</br>

---
### <a name="device"></a>Dispositivo

La condición de coincidencia Dispositivo identifica solicitudes realizadas desde un dispositivo móvil en función de sus propiedades. La detección de dispositivos móviles se realiza a través de [WURFL](http://wurfl.sourceforge.net/). 

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Dispositivo:
- **Matches** (Coincide): requiere que el dispositivo del solicitante coincida con el valor especificado. 
- **Does Not Match** (No coincide): requiere que el dispositivo del solicitante no coincida con el valor especificado.

Información importante:

- Use la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para especificar si el valor especificado distingue mayúsculas de minúsculas.
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
  - Relleno de la memoria caché completa
  - Max-Age interna predeterminada
  - Forzar Max-Age interna
  - Ignorar no almacenar en caché de origen
  - Max-Stale interna

#### <a name="string-type"></a>Tipo de cadena
Normalmente, una funcionalidad WURFL acepta cualquier combinación de números, letras y símbolos. Debido a la naturaleza flexible de esta funcionalidad, debe elegir cómo se interpreta el valor asociado a esta condición de coincidencia. La tabla siguiente describe el conjunto de opciones disponibles:

Type     | DESCRIPCIÓN
---------|------------
Literal  | Seleccione esta opción para evitar que la mayoría de los caracteres tengan un significado especial mediante el uso de su [valor literal](cdn-rules-engine-reference.md#literal-values).
Wildcard (Carácter comodín) | Seleccione esta opción para aprovechar las ventajas de todos los [caracteres comodín] ([valores de caracteres comodín](cdn-rules-engine-reference.md#wildcard-values).
Regex    | Seleccione esta opción para usar [expresiones regulares](cdn-rules-engine-reference.md#regular-expressions). Las expresiones regulares son útiles para definir un patrón de caracteres.

#### <a name="wurfl-capabilities"></a>Funcionalidades WURFL
Una funcionalidad WURFL hace referencia a una categoría que describe los dispositivos móviles. La funcionalidad seleccionada determina el tipo de descripción de dispositivo móvil que se utiliza para identificar las solicitudes.

En la tabla siguiente se enumeran las funcionalidades WURFL y sus variables para el motor de reglas.
<br>
> [!NOTE] 
> Las variables siguientes se admiten en las funciones **Modify Client Request Header** (Modificar encabezado de solicitud de cliente) y **Modify Client Response Header** (Modificar encabezado de respuesta de cliente).

Funcionalidad | Variable | DESCRIPCIÓN | Valores de ejemplo
-----------|----------|-------------|----------------
Nombre de la organización | %{wurfl_cap_brand_name} | Una cadena que indica el nombre de marca del dispositivo. | Samsung
Sistema operativo del dispositivo | %{wurfl_cap_device_os} | Una cadena que indica el sistema operativo instalado en el dispositivo. | IOS
Versión del sistema operativo del dispositivo | %{wurfl_cap_device_os_version} | Una cadena que indica el número de versión del sistema operativo instalado en el dispositivo. | 1.0.1
Orientación dual | %{wurfl_cap_dual_orientation} | Un valor booleano que indica si el dispositivo admite la orientación dual. | true
DTD preferido de HTML | %{wurfl_cap_html_preferred_dtd} | Una cadena que indica la definición de tipo de documento (DTD) preferido del dispositivo móvil para el contenido HTML. | ninguno<br/>xhtml_basic<br/>html5
Imagen de inclusión | %{wurfl_cap_image_inlining} | Un valor booleano que indica si el dispositivo admite imágenes con codificación Base64. | false
Es Android | %{wurfl_vcap_is_android} | Un valor booleano que indica si el dispositivo usa el sistema operativo Android. | true
Es IOS | %{wurfl_vcap_is_ios} | Un valor booleano que indica si el dispositivo usa iOS. | false
Es Smart TV | %{wurfl_cap_is_smarttv} | Un valor booleano que indica si el dispositivo es una Smart TV. | false
Es smartphone | %{wurfl_vcap_is_smartphone} | Un valor booleano que indica si el dispositivo es un smartphone. | true
Es tablet | %{wurfl_cap_is_tablet} | Un valor booleano que indica si el dispositivo es una tablet. Esta descripción es independiente del sistema operativo. | true
Es dispositivo inalámbrico | %{wurfl_cap_is_wireless_device} | Un valor booleano que indica si el dispositivo se considera un dispositivo inalámbrico. | true
Nombre de marca | %{wurfl_cap_marketing_name} | Una cadena que indica el nombre de marca del dispositivo. | BlackBerry 8100 Pearl
Explorador móvil | %{wurfl_cap_mobile_browser} | Una cadena que indica el explorador que se usa para solicitar el contenido del dispositivo. | Chrome
Versión del explorador móvil | %{wurfl_cap_mobile_browser_version} | Una cadena que indica la versión del explorador que se usa para solicitar el contenido del dispositivo. | 31
Nombre del modelo | %{wurfl_cap_model_name} | Una cadena que indica el nombre del modelo del dispositivo. | s3
Descarga progresiva | %{wurfl_cap_progressive_download} | Un valor booleano que indica si el dispositivo admite la reproducción de audio y vídeo durante la descarga. | true
Fecha de lanzamiento | %{wurfl_cap_release_date} | Una cadena que indica el año y el mes en el que se agregó el dispositivo a la base de datos WURFL.<br/><br/>Formato: `yyyy_mm` | 2013_december
Altura de resolución | %{wurfl_cap_resolution_height} | Un entero que indica la altura del dispositivo en píxeles. | 768
Anchura de resolución | %{wurfl_cap_resolution_width} | Un entero que indica la anchura del dispositivo en píxeles. | 1024

[Volver arriba](#main)

</br>

---
### <a name="edge-cname"></a>Cname perimetral
Información importante: 
- La lista de CNAME perimetrales disponibles se limita a los que se han configurado en la página de CNAME perimetral que se corresponde con la plataforma donde se configura el motor de reglas.
- Antes de eliminar una configuración de CNAME perimetral, asegúrese de que no hace referencia a ella ninguna condición de coincidencia CNAME perimetral. No se pueden eliminar de la página de CNAME perimetral las configuraciones de CNAME perimetral que se hayan definido en una regla. 
- No use una instrucción AND IF para combinar determinadas condiciones de coincidencia. Por ejemplo, la combinación de una condición de coincidencia CNAME perimetral con una Origen de cliente crearía un patrón de coincidencia que no se puede cumplir. Por esta razón, no se pueden combinar dos condiciones de coincidencia CNAME perimetral mediante una instrucción AND IF.
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
  - Relleno de la memoria caché completa
  - Max-Age interna predeterminada
  - Forzar Max-Age interna
  - Ignorar no almacenar en caché de origen
  - Max-Stale interna

[Volver arriba](#main)

</br>

---
### <a name="referring-domain"></a>Dominio de referencia
El nombre de host asociado con el origen de referencia mediante el cual se solicitó contenido determina si se cumple la condición Dominio de referencia. 

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Dominio de referencia:
- **Matches** (Coincide): requiere que el nombre de host de referencia coincida con el valor especificado. 
- **Does Not Match** (No coincide): requiere que el nombre de host de referencia no coincida con el valor especificado.

Información importante:
- Especifique varios nombres de host mediante la delimitación de cada uno con un espacio.
- Esta condición de coincidencia admite [valores de caracteres comodín](cdn-rules-engine-reference.md#wildcard-values).
- Si el valor especificado no contiene asteriscos, debe ser una coincidencia exacta para el nombre de host del sitio de referencia. Por ejemplo, especificar "mydomain.com" no coincidiría con "www.mydomain.com."
- Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si en la comparación se distinguirán mayúsculas y minúsculas.
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
  - Relleno de la memoria caché completa
  - Max-Age interna predeterminada
  - Forzar Max-Age interna
  - Ignorar no almacenar en caché de origen
  - Max-Stale interna

[Volver arriba](#main)

</br>

---
### <a name="request-header-literal"></a>Literal de encabezado de solicitud
La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Literal de encabezado de solicitud.
- **Matches** (Coincide): requiere que la solicitud contenga el encabezado especificado. Su valor debe coincidir con el que se define en esta condición de coincidencia.
- **Does Not Match** (No coincide): requiere que la solicitud cumpla cualquiera de los siguientes criterios:
  - No contenga el encabezado especificado.
  - Contenga el encabezado especificado, pero que su valor no coincida con el que se define en esta condición de coincidencia.
  
Información importante:
- Las comparaciones de nombres de encabezado distinguen siempre entre mayúsculas y minúsculas. Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si las comparaciones de valores de encabezado distinguen las mayúsculas y las minúsculas.
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
  - Relleno de la memoria caché completa
  - Max-Age interna predeterminada
  - Forzar Max-Age interna
  - Ignorar no almacenar en caché de origen
  - Max-Stale interna

[Volver arriba](#main)

</br>

---
### <a name="request-header-regex"></a>Regex de encabezado de solicitud
La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Regex de encabezado de solicitud.
- **Matches** (Coincide): requiere que la solicitud contenga el encabezado especificado. Su valor debe coincidir con el patrón que se define en la [expresión regular](cdn-rules-engine-reference.md#regular-expressions) especificada.
- **Does Not Match** (No coincide): requiere que la solicitud cumpla cualquiera de los siguientes criterios:
  - No contenga el encabezado especificado.
  - Contenga el encabezado especificado, pero que su valor no coincida con la expresión regular especificada.

Información importante:
- Nombre de encabezado: 
  - Las comparaciones de nombres de encabezado distinguen entre mayúsculas y minúsculas.
  - Reemplace los espacios en el nombre del encabezado por "%20". 
- Valor de encabezado: 
  - Para los valores de encabezado se pueden utilizar expresiones regulares.
  - Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si las comparaciones de valores de encabezado distinguen las mayúsculas y las minúsculas.
  - La condición de coincidencia solo se cumple cuando un valor de encabezado coincide exactamente con al menos uno de los patrones especificados.
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
  - Relleno de la memoria caché completa
  - Max-Age interna predeterminada
  - Forzar Max-Age interna
  - Ignorar no almacenar en caché de origen
  - Max-Stale interna 

[Volver arriba](#main)

</br>

---
### <a name="request-header-wildcard"></a>Carácter comodín de encabezado de solicitud
La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Caracteres comodín de encabezado de solicitud.
- **Matches** (Coincide): requiere que la solicitud contenga el encabezado especificado. Su valor debe coincidir con al menos uno de los valores que se definen en esta condición de coincidencia.
- **Does Not Match** (No coincide): requiere que la solicitud cumpla cualquiera de los siguientes criterios:
  - No contenga el encabezado especificado.
  - Contenga el encabezado especificado, pero que su valor no coincida con ninguno de los valores especificados.
  
Información importante:
- Nombre de encabezado: 
  - Las comparaciones de nombres de encabezado distinguen entre mayúsculas y minúsculas.
  - Los espacios en el nombre del encabezado deben reemplazarse por "%20". También puede utilizar este valor para especificar espacios en un valor de encabezado.
- Valor de encabezado: 
  - Un valor de encabezado puede aprovechar las ventajas de los [valores de carácter comodín](cdn-rules-engine-reference.md#wildcard-values).
  - Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si las comparaciones de valores de encabezado distinguen las mayúsculas y las minúsculas.
  - La condición de coincidencia se cumple cuando un valor de encabezado coincide exactamente con al menos uno de los patrones especificados.
  - Especifique varios valores de encabezado mediante la delimitación de cada uno con un espacio.
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
  - Relleno de la memoria caché completa
  - Max-Age interna predeterminada
  - Forzar Max-Age interna
  - Ignorar no almacenar en caché de origen
  - Max-Stale interna

[Volver arriba](#main)

</br>

---
### <a name="request-method"></a>Método de solicitud
La condición de coincidencia Método de solicitud se cumple solo cuando los recursos se solicitan mediante el método de solicitud seleccionado. Los métodos de solicitud disponibles son:
- GET
- HEAD 
- POST 
- OPCIONES 
- PUT 
- DELETE 
- TRACE 
- CONNECT 

Información importante:
- De forma predeterminada, solo el método de solicitud GET genera contenido almacenado en caché en la red. Los demás métodos de solicitud los procesa el proxy mediante la red.
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
  - Relleno de la memoria caché completa
  - Max-Age interna predeterminada
  - Forzar Max-Age interna
  - Ignorar no almacenar en caché de origen
  - Max-Stale interna

[Volver arriba](#main)

</br>

---
### <a name="request-scheme"></a>Esquema de solicitud
La condición de coincidencia Esquema de solicitud se cumple solo cuando los recursos se solicitan mediante el protocolo seleccionado. Los protocolos disponibles son: 
- HTTP
- HTTPS

Información importante:
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
  - Relleno de la memoria caché completa
  - Max-Age interna predeterminada
  - Forzar Max-Age interna
  - Ignorar no almacenar en caché de origen
  - Max-Stale interna

[Volver arriba](#main)

</br>

---
### <a name="url-path-directory"></a>Directorio de ruta de acceso URL
Identifica una solicitud por su ruta de acceso relativa, que excluye el nombre de archivo del recurso solicitado.

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Directorio de ruta de acceso URL.
- **Matches** (Coincide): requiere que la solicitud contenga una ruta de acceso URL relativa, sin incluir el nombre de archivo, que coincida con el patrón de dirección URL especificado.
- **Does Not Match** (No coincide): requiere que la solicitud contenga una ruta de acceso URL relativa, sin incluir el nombre de archivo, que no coincida con el patrón de dirección URL especificado.

Información importante:
- Use la opción **Relative to** (Relativo a) para especificar si la comparación de la dirección URL debe comenzar antes o después del punto de acceso al contenido. El punto de acceso al contenido es la parte de la ruta de acceso que aparece entre el nombre de host de Verizon CDN y la ruta de acceso relativa al recurso solicitado (por ejemplo, /800001/CustomerOrigin). Identifica una ubicación por tipo de servidor (por ejemplo, red CDN u origen del cliente) y el número de cuenta de cliente.

   Los valores siguientes están disponibles para la opción **Relative to** (Relativo a):
   - **Root** (Raíz): indica que el punto de comparación de la dirección URL comienza directamente después del nombre de host de la red CDN. 

     Por ejemplo: http:\//wpc.0001.&lt;dominio&gt;/**800001/myorigin/myfolder**/index.htm

   - **Origin** (Origen): indica que el punto de comparación de la dirección URL comienza después del punto de acceso al contenido (por ejemplo, /000001 o /800001/myorigin). Dado que el CNAME \*. azureedge.net se crea en relación con el directorio de origen en el nombre de host de Verizon CDN de forma predeterminada, los usuarios de Azure CDN deben utilizar el valor **Origin** (Origen). 

     Por ejemplo: https:\//&lt;punto_de_conexión&gt;.azureedge.net/**myfolder**/index.htm 

     Esta dirección URL señala al nombre de host de Verizon CDN siguiente: http:\//wpc.0001.&lt;domain&gt;/800001/myorigin/**myfolder**/index.htm

- Se vuelve a escribir un dirección URL de CNAME perimetral a una dirección URL de la red CDN antes de la comparación de la dirección URL.

    Por ejemplo, las siguientes direcciones URL apuntan al mismo recurso y, por tanto, tienen la misma ruta de acceso URL.
  - Dirección URL de la red CDN: http:\//wpc.0001.&lt;domain&gt;/800001/CustomerOrigin/path/asset.htm
    
  - Dirección URL del CNAME perimetral: http:\//&lt;punto_de_conexión&gt;.azureedge.net/path/asset.htm
    
    Información adicional:
  - Dominio personalizado: https:\//my.domain.com/path/asset.htm
    
  - Ruta de acceso de dirección URL (relativa a la raíz): / 800001/CustomerOrigin/path/
    
  - Ruta de acceso de dirección URL (relativa al origen): /path/

- La parte de la dirección URL que se utiliza para la comparación de la dirección URL termina justo antes del nombre de archivo del recurso solicitado. Una barra diagonal final es el último carácter de este tipo de ruta de acceso.
    
- Reemplace los espacios en el patrón de la ruta de acceso de dirección URL por "%20".
    
- Cada patrón de ruta de acceso de dirección URL puede contener uno o varios asteriscos (*), donde cada asterisco coincide con una secuencia de uno o más caracteres.
    
- Especifique varias rutas de acceso URL en el patrón mediante la delimitación de cada una con un espacio.

    Por ejemplo: */sales/ */marketing/

- Una especificación de ruta de acceso de dirección URL puede aprovechar las ventajas de los [valores de caracteres comodín](cdn-rules-engine-reference.md#wildcard-values).

- Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si en la comparación se distinguirán mayúsculas y minúsculas.

[Volver arriba](#main)

</br>

---
### <a name="url-path-extension"></a>Extensión de ruta de acceso URL
Identifica las solicitudes por la extensión de archivo del recurso solicitado.

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Extensión de ruta de acceso URL.
- **Matches** (Coincide): requiere que la dirección URL de la solicitud contenga una extensión de archivo que coincida exactamente con el patrón especificado.

   Por ejemplo, si especifica "htm", coinciden los recursos "htm" pero no los recursos "html".  

- **Does Not Match** (No coincide): requiere que la dirección URL de la solicitud contenga una extensión de archivo que no coincida con el patrón especificado.

Información importante:
- Debe especificar las extensiones de archivo para que coincidan con el cuadro **Valor**. No incluya un punto inicial; por ejemplo, utilice htm en lugar de .htm.

- Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si en la comparación se distinguirán mayúsculas y minúsculas.

- Especifique varias extensiones de archivo delimitando cada extensión con un único espacio. 

    Por ejemplo: htm html

- Por ejemplo, la especificación "htm" coincide con los recursos "htm" pero no con los recursos "html".


#### <a name="sample-scenario"></a>Escenario de ejemplo

En la configuración del ejemplo siguiente se da por supuesto que esta condición de coincidencia se cumple cuando una solicitud coincide con una de las extensiones especificadas.

Especificación de Valor: asp aspx php html

Esta condición de coincidencia se cumple cuando se encuentran direcciones URL que terminan con las siguientes extensiones:
- .asp
- .aspx
- .php
- .html

[Volver arriba](#main)

</br>

---
### <a name="url-path-filename"></a>Nombre de archivo de ruta de acceso URL
Identifica las solicitudes por el nombre de archivo del recurso solicitado. A efectos de esta condición de coincidencia, un nombre de archivo consta del nombre del recurso solicitado, un punto y la extensión de archivo (por ejemplo, index.html).

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Nombre de archivo de ruta de acceso URL.
- **Matches** (Coincide): requiere que la solicitud contenga un nombre de archivo en la ruta de acceso URL que coincida con el patrón especificado.
- **Does Not Match** (No coincide): requiere que la solicitud contenga un nombre de archivo en la ruta de acceso URL que no coincida con el patrón especificado.

Información importante:
- Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si en la comparación se distinguirán mayúsculas y minúsculas.

- Puede especificar varias extensiones de archivo delimitando cada extensión con un único espacio.

    Por ejemplo: index.htm index.html

- Reemplace los espacios en un valor de nombre de archivo por "%20".
    
- Un valor de nombre de archivo puede aprovechar las ventajas de los [valores de carácter comodín](cdn-rules-engine-reference.md#wildcard-values). Por ejemplo, cada patrón de nombre de archivo puede contener uno o varios asteriscos (*), donde cada asterisco coincide con una secuencia de uno o más caracteres.
    
- Si no se ha especificado ningún carácter comodín, solo una coincidencia exacta cumplirá esta condición de coincidencia.

    Por ejemplo, especificar "presentation.ppt" coincide con un recurso llamado "presentation.ppt", pero no uno con nombre "presentation.pptx."

[Volver arriba](#main)

</br>

---
### <a name="url-path-literal"></a>Literal de ruta de acceso URL
Compara la ruta de acceso URL de la solicitud, incluido el nombre de archivo, con el valor especificado.

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Literal de ruta de acceso URL.
- **Matches** (Coincide): requiere que la solicitud contenga una ruta de acceso URL que coincida con el patrón especificado.
- **Does Not Match** (No coincide): requiere que la solicitud contenga una ruta de acceso URL que no coincida con el patrón especificado.

Información importante:
- Use la opción **Relative to** (Relativo a) para especificar si la comparación de la dirección URL debe comenzar antes o después del punto de acceso al contenido. 

    Los valores siguientes están disponibles para la opción **Relative to** (Relativo a):
  - **Root** (Raíz): indica que el punto de comparación de la dirección URL comienza directamente después del nombre de host de la red CDN.

    Por ejemplo: http:\//wpc.0001.&lt;domain&gt;/**800001/myorigin/myfolder/index.htm**

  - **Origin** (Origen): indica que el punto de comparación de la dirección URL comienza después del punto de acceso al contenido (por ejemplo, /000001 o /800001/myorigin). Dado que el CNAME \*. azureedge.net se crea en relación con el directorio de origen en el nombre de host de Verizon CDN de forma predeterminada, los usuarios de Azure CDN deben utilizar el valor **Origin** (Origen). 

    Por ejemplo: https:\//&lt;punto_de_conexión&gt;.azureedge.net/**myfolder/index.htm**

    Esta dirección URL señala al nombre de host de Verizon CDN siguiente: http:\//wpc.0001.&lt;domain&gt;/800001/myorigin/**myfolder/index.htm**

- Se vuelve a escribir una dirección URL de CNAME perimetral a una dirección URL de la red CDN antes de la comparación de la dirección URL.

    Por ejemplo, las siguientes direcciones URL apuntan al mismo recurso y, por tanto, tienen la misma ruta de acceso URL:
  - Dirección URL de la red CDN: http:\//wpc.0001.&lt;domain&gt;/800001/CustomerOrigin/path/asset.htm
  - Dirección URL del CNAME perimetral: http:\//&lt;punto_de_conexión&gt;.azureedge.net/path/asset.htm
    
    Información adicional:
    
  - Ruta de acceso URL (relativa a la raíz): /800001/CustomerOrigin/path/asset.htm
   
  - Ruta de acceso URL (relativa al origen): /path/asset.htm

- Se omiten las cadenas de consulta en la dirección URL.
- Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si en la comparación se distinguirán mayúsculas y minúsculas.
- El valor especificado para esta condición de coincidencia se compara con la ruta de acceso relativa de la solicitud exacta realizada por el cliente.

- Para hacer coincidir todas las solicitudes realizadas a un directorio determinado, utilice la condición de coincidencia [directorio de la ruta de acceso URL](#url-path-directory) o [carácter comodín de la ruta de acceso URL](#url-path-wildcard).

[Volver arriba](#main)

</br>

---
### <a name="url-path-regex"></a>Regex de ruta de acceso URL
Compara la ruta de acceso de una solicitud con la [expresión regular](cdn-rules-engine-reference.md#regular-expressions) especificada.

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Regex de ruta de acceso URL.
- **Matches** (Coincide): requiere que la solicitud contenga una ruta de acceso URL que coincida con la expresión regular especificada.
- **Does Not Match** (No coincide): requiere que la solicitud contenga una ruta de acceso URL que no coincida con la expresión regular especificada.

Información importante:
- Se vuelve a escribir una dirección URL de CNAME perimetral a una dirección URL de la red CDN antes de la comparación de la dirección URL. 
 
    Por ejemplo, las direcciones URL apuntan al mismo recurso y, por tanto, tienen la misma ruta de acceso URL.

  - Dirección URL de la red CDN: http:\//wpc.0001.&lt;domain&gt;/800001/CustomerOrigin/path/asset.htm

  - Dirección URL de CNAME perimetral: http:\//my.domain.com/path/asset.htm
    
    Información adicional:
    
  - Ruta de acceso URL: /800001/CustomerOrigin/path/asset.htm

- Se omiten las cadenas de consulta en la dirección URL.
    
- Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si en la comparación se distinguirán mayúsculas y minúsculas.
    
- Los espacios en la ruta de acceso URL deben reemplazarse por "%20".

[Volver arriba](#main)

</br>

---
### <a name="url-path-wildcard"></a>Carácter comodín de ruta de acceso URL
Compara la ruta de acceso URL relativa de una solicitud con el patrón de caracteres comodín especificado.

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Caracteres comodín de ruta de acceso URL.
- **Matches** (Coincide): requiere que la solicitud contenga una ruta de acceso URL que coincida con el patrón de caracteres comodín especificado.
- **Does Not Match** (No coincide): requiere que la solicitud contenga una ruta de acceso URL que no coincida con el patrón de caracteres comodín especificado.

Información importante:
- **Relative to** (Relativo a): esta opción determina si la comparación de la dirección URL debe comenzar antes o después del punto de acceso al contenido.

   Esta opción puede tener los valores siguientes:
  - **Root** (Raíz): indica que el punto de comparación de la dirección URL comienza directamente después del nombre de host de la red CDN.

    Por ejemplo: http:\//wpc.0001.&lt;domain&gt;/**800001/myorigin/myfolder/index.htm**

  - **Origin** (Origen): indica que el punto de comparación de la dirección URL comienza después del punto de acceso al contenido (por ejemplo, /000001 o /800001/myorigin). Dado que el CNAME \*. azureedge.net se crea en relación con el directorio de origen en el nombre de host de Verizon CDN de forma predeterminada, los usuarios de Azure CDN deben utilizar el valor **Origin** (Origen). 

    Por ejemplo: https:\//&lt;punto_de_conexión&gt;.azureedge.net/**myfolder/index.htm**

    Esta dirección URL señala al nombre de host de Verizon CDN siguiente: http:\//wpc.0001.&lt;domain&gt;/800001/myorigin/**myfolder/index.htm**

- Se vuelve a escribir una dirección URL de CNAME perimetral a una dirección URL de la red CDN antes de la comparación de la dirección URL.

    Por ejemplo, las siguientes direcciones URL apuntan al mismo recurso y, por tanto, tienen la misma ruta de acceso URL:
  - Dirección URL de la red CDN: http://wpc.0001.&lt;dominio&gt;/800001/CustomerOrigin/path/asset.htm
  - Dirección URL del CNAME perimetral: http:\//&lt;punto_de_conexión&gt;.azureedge.net/path/asset.htm
    
    Información adicional:
    
  - Ruta de acceso URL (relativa a la raíz): /800001/CustomerOrigin/path/asset.htm
    
  - Ruta de acceso URL (relativa al origen): /path/asset.htm
    
- Especifique varias rutas de acceso URL mediante la delimitación de cada una con un espacio único.

   Por ejemplo: /marketing/asset.* /sales/\*.htm

- Se omiten las cadenas de consulta en la dirección URL.
    
- Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si en la comparación se distinguirán mayúsculas y minúsculas.
    
- Reemplace los espacios en la ruta de acceso URL por "%20".
    
- El valor especificado para una ruta de acceso URL puede aprovechar las ventajas de los [valores de caracteres comodín](cdn-rules-engine-reference.md#wildcard-values). Cada patrón de ruta de acceso URL puede contener uno o varios asteriscos (*), donde cada asterisco coincide con una secuencia de uno o más caracteres.

#### <a name="sample-scenarios"></a>Escenarios de ejemplo

En las configuraciones de ejemplo de la tabla siguiente se da por supuesto que esta condición de coincidencia se cumple cuando una solicitud coincide con una de los patrones de URL especificados:

Valor                   | Relativo a    | Resultado 
------------------------|----------------|-------
\*/test.html \*/test.php  | Raíz u origen | Este patrón coincide con solicitudes de recursos llamados "test.html" o "test.php" en cualquier carpeta.
/80ABCD/origin/text/*   | Raíz           | Este patrón coincide cuando el recurso solicitado cumple los criterios siguientes: <br />- Debe residir en un origen de cliente que se llama "origin". <br />- La ruta de acceso relativa debe comenzar con una carpeta llamada "text". Es decir, el recurso solicitado puede residir en la carpeta "text" o una de sus subcarpetas recursivas.
*/css/* */js/*          | Raíz u origen | Este patrón coincide con todas las direcciones URL de CNAME perimetral o de la red CDN que contengan una carpeta llamada css o js.
*.jpg *.gif *.png       | Raíz u origen | Este patrón coincide con todas las direcciones URL de CNAME perimetral o de la red CDN que terminen con .jpg, .gif o .png. Una manera alternativa para especificar este patrón es con la [condición de coincidencia Extensión de ruta de acceso URL](#url-path-extension).
/images/\* /media/\*      | Origen         | Este patrón coincide con las direcciones URL de CNAME perimetral o de la red CDN cuya ruta de acceso relativa comienza por una carpeta "images" o "media". <br />- Dirección URL de la red CDN: http:\//wpc.0001.&lt;domain&gt;/800001/myorigin/images/sales/event1.png<br />- Dirección URL de CNAME perimetral de ejemplo: http:\//cdn.mydomain.com/images/sales/event1.png

[Volver arriba](#main)

</br>

---
### <a name="url-query-literal"></a>Literal de consulta de dirección URL
Compara la cadena de consulta de una solicitud con el valor especificado.

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Literal de la consulta URL.
- **Matches** (Coincide): requiere que la solicitud contenga una cadena de consulta URL que coincida con la cadena de consulta especificada.
- **Does Not Match** (No coincide): requiere que la solicitud contenga una cadena de consulta URL que no coincida con la cadena de consulta especificada.

Información importante:

- Solo las coincidencias de cadenas de consulta exactas satisfacen esta condición de coincidencia.
    
- Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si las comparaciones de cadena de consulta distinguen las mayúsculas y las minúsculas.
    
- No incluya un signo de interrogación (?) inicial en el texto del valor de la cadena de consulta.
    
- Ciertos caracteres requieren codificación URL. Utilice el símbolo de porcentaje para codificar en formato URL los caracteres siguientes:

   Character | Codificación URL
   ----------|---------
   Espacio     | %20
   &         | %25

- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
   - Relleno de la memoria caché completa
   - Max-Age interna predeterminada
   - Forzar Max-Age interna
   - Ignorar no almacenar en caché de origen
   - Max-Stale interna

[Volver arriba](#main)

</br>

---
### <a name="url-query-parameter"></a>Parámetro de consulta de dirección URL
Identifica las solicitudes que contienen el parámetro de cadena de consulta especificado. Este parámetro se establece en un valor que coincide con un patrón especificado. Los parámetros de cadena de consulta (por ejemplo, parameter=value) en la dirección URL de la solicitud determinan si se cumple esta condición. Esta condición de coincidencia identifica un parámetro de cadena de consulta por su nombre y acepta uno o varios valores para el valor del parámetro. 

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Parámetro de consulta URL.
- **Matches** (Coincide): requiere que una solicitud contenga el parámetro especificado con un valor que coincida con al menos uno de los que se definen en esta condición de coincidencia.
- **Does Not Match** (No coincide): requiere que la solicitud cumpla cualquiera de los siguientes criterios:
  - No contiene el parámetro especificado.
  - Contiene el parámetro especificado, pero su valor no coincide con ninguno de los que se definen en esta condición de coincidencia.

Esta condición de coincidencia proporciona una manera sencilla para especificar las combinaciones de nombre/valor de los parámetros. Para obtener mayor flexibilidad en las coincidencias de parámetros de cadena de consulta, considere el uso de la condición de coincidencia [Carácter comodín de la consulta URL](#url-query-wildcard) coincide con la condición.

Información importante:
- Solo se puede especificar un nombre de parámetro de la cadena de consulta por instancia de esta condición de coincidencia.
    
- Dado que no se admiten valores de caracteres comodín cuando se especifica un nombre de parámetro, solo las coincidencias de nombre de parámetro exactas son aptas para la comparación.
- Los valores de los parámetros pueden incluir [valores de caracteres comodín](cdn-rules-engine-reference.md#wildcard-values).
   - Cada patrón de valor de parámetro puede contener uno o varios asteriscos (*), donde cada asterisco coincide con una secuencia de uno o más caracteres.
   - Ciertos caracteres requieren codificación URL. Utilice el símbolo de porcentaje para codificar en formato URL los caracteres siguientes:

       Character | Codificación URL
       ----------|---------
       Espacio     | %20
       &         | %25

- Especifique varios valores de parámetro de la cadena de consulta mediante la delimitación de cada uno con un espacio. Esta condición de coincidencia se cumple cuando una solicitud contiene una de las combinaciones de nombre/valor especificadas.

   - Ejemplo 1:

     - Configuración:

       ValueA ValueB

     - Esta configuración coincide con los siguientes parámetros de la cadena de consulta:

       Parameter1=ValueA
    
       Parameter1=ValueA

   - Ejemplo 2:

     - Configuración: 

        Value%20A Value%20B

     - Esta configuración coincide con los siguientes parámetros de la cadena de consulta:

       Parameter1=Value%20A

       Parameter1=Value%20B

- Esta condición de coincidencia se cumple solo cuando hay una coincidencia exacta con al menos una de las combinaciones de nombre/valor de la cadena de consulta especificada.

   Por ejemplo, si se usa la configuración del ejemplo anterior, la combinación de nombre/valor del parámetro "Parameter1=ValueAdd" no se consideraría una coincidencia. Sin embargo, si se especifica cualquiera de los valores siguientes, coincidirá con esa combinación de nombre/valor:

   - ValueA ValueB ValueAdd
   - ValueA* ValueB

- Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si las comparaciones de cadena de consulta distinguen las mayúsculas y las minúsculas.
    
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
   - Relleno de la memoria caché completa
   - Max-Age interna predeterminada
   - Forzar Max-Age interna
   - Ignorar no almacenar en caché de origen
   - Max-Stale interna

#### <a name="sample-scenarios"></a>Escenarios de ejemplo
En el ejemplo siguiente se muestra cómo funciona esta opción en situaciones específicas:

NOMBRE  | Valor |  Resultado
------|-------|--------
Usuario  | Joe   | Este patrón coincide cuando la cadena de consulta para una dirección URL solicitada es "?user=joe".
Usuario  | *     | Este patrón coincide cuando la cadena de consulta para una dirección URL solicitada contiene el parámetro User.
Email | Joe\* | Este patrón coincide cuando la cadena de consulta para una dirección URL solicitada contiene un parámetro Email que comienza por "Joe".

[Volver arriba](#main)

</br>

---
### <a name="url-query-regex"></a>Regex de consulta de dirección URL
Identifica las solicitudes que contienen el parámetro de cadena de consulta especificado. Este parámetro se establece en un valor que coincide con una [expresión regular](cdn-rules-engine-reference.md#regular-expressions) especificada.

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Regex de consulta URL.
- **Matches** (Coincide): requiere que la solicitud contenga una cadena de consulta URL que coincida con la expresión regular especificada.
- **Does Not Match** (No coincide): requiere que la solicitud contenga una cadena de consulta URL que no coincida con la expresión regular especificada.

Información importante:
- Solo las coincidencias exactas con la expresión regular especificada satisfacen esta condición de coincidencia.
    
- Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si las comparaciones de cadena de consulta distinguen las mayúsculas y las minúsculas.
    
- A los efectos de esta opción, una cadena de consulta comienza con el primer carácter después del delimitador de signo de interrogación (?) de la cadena de consulta.
    
- Ciertos caracteres requieren codificación URL. Utilice el símbolo de porcentaje para codificar en formato URL los caracteres siguientes:

   Character | Codificación URL | Valor
   ----------|--------------|------
   Espacio     | %20          | \%20
   &         | %25          | \%25

   Tenga en cuenta que los símbolos de porcentaje deben usarse en forma de secuencia de escape.

- Realice una doble secuencia de escape en los caracteres especiales de las expresiones regulares (por ejemplo, \^$.+) para incluir una barra diagonal inversa en la expresión regular.

   Por ejemplo:

   Valor | Se interpreta como 
   ------|---------------
   \\+    | +
   \\\\+   | \\+

- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
   - Relleno de la memoria caché completa
   - Max-Age interna predeterminada
   - Forzar Max-Age interna
   - Ignorar no almacenar en caché de origen
   - Max-Stale interna


[Volver arriba](#main)

</br>

---
### <a name="url-query-wildcard"></a>Carácter comodín de consulta de dirección URL
Compara los valores especificados con la cadena de consulta de una solicitud.

La opción **Matches**/**Does Not Match** (Coincide/No coincide) determina las condiciones para que se cumpla la condición de coincidencia Caracteres comodín de la consulta URL.
- **Matches** (Coincide): requiere que la solicitud contenga una cadena de consulta URL que coincida con el valor del carácter comodín especificado.
- **Does Not Match** (No coincide): requiere que la solicitud contenga una cadena de consulta URL que no coincida con el valor del carácter comodín especificado.

Información importante:
- A los efectos de esta opción, una cadena de consulta comienza con el primer carácter después del delimitador de signo de interrogación (?) de la cadena de consulta.
- Los valores de los parámetros pueden incluir [valores de caracteres comodín](cdn-rules-engine-reference.md#wildcard-values):
   - Cada patrón de valor de parámetro puede contener uno o varios asteriscos (*), donde cada asterisco coincide con una secuencia de uno o más caracteres.
   - Ciertos caracteres requieren codificación URL. Utilice el símbolo de porcentaje para codificar en formato URL los caracteres siguientes:

     Character | Codificación URL
     ----------|---------
     Espacio     | %20
     &         | %25

- Especifique varios valores de encabezado mediante la delimitación de cada uno con un espacio.

   Por ejemplo: *Parameter1=ValueA* *ValueB* *Parameter1=ValueC&Parameter2=ValueD*

- Solo las coincidencias exactas con al menos uno de los patrones de cadena de consulta especificados satisfacen esta condición de coincidencia.
    
- Utilice la opción **Ignore Case** (No distinguir mayúsculas y minúsculas) para controlar si las comparaciones de cadena de consulta distinguen las mayúsculas y las minúsculas.
    
- Debido a la manera en que se realiza el seguimiento de la configuración de la memoria caché, esta condición de coincidencia es incompatible con las siguientes características:
   - Relleno de la memoria caché completa
   - Max-Age interna predeterminada
   - Forzar Max-Age interna
   - Ignorar no almacenar en caché de origen
   - Max-Stale interna

#### <a name="sample-scenarios"></a>Escenarios de ejemplo
En el ejemplo siguiente se muestra cómo funciona esta opción en situaciones específicas:

 NOMBRE                 | DESCRIPCIÓN
 ---------------------|------------
user=joe              | Este patrón coincide cuando la cadena de consulta para una dirección URL solicitada es "?user=joe".
\*user=\* \*optout=\* | Este patrón coincide cuando la consulta URL de la red CDN contiene el parámetro user o el parámetro optout.

[Volver arriba](#main)

</br>

## <a name="next-steps"></a>Pasos siguientes
* [Introducción a Azure Content Delivery Network](cdn-overview.md)
* [Referencia del motor de reglas](cdn-rules-engine-reference.md)
* [Expresiones condicionales del motor de reglas](cdn-rules-engine-reference-conditional-expressions.md)
* [Características del motor de reglas](cdn-rules-engine-reference-features.md)
* [Invalidación del comportamiento HTTP predeterminado mediante el motor de reglas](cdn-rules-engine.md)

