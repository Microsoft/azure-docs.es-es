---
title: Listas de exclusión y límites de tamaño de solicitud del firewall de aplicaciones web en Azure Application Gateway (Azure Portal)
description: En este artículo se proporciona información acerca de la configuración de listas de exclusión y límites de tamaño de solicitud del firewall de aplicaciones web en Application Gateway con Azure Portal.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.date: 5/15/2019
ms.author: victorh
ms.topic: conceptual
ms.openlocfilehash: 5ddcdeca41e2f21fa27db25f7e0721c7ef87e491
ms.sourcegitcommit: 3675daec6c6efa3f2d2bf65279e36ca06ecefb41
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65620283"
---
# <a name="web-application-firewall-request-size-limits-and-exclusion-lists"></a>Listas de exclusión y límites de tamaño de solicitud del firewall de aplicaciones web

El firewall de aplicaciones web (WAF) de Azure Application Gateway proporciona protección a las aplicaciones web. En este artículo se describe la configuración de listas de exclusión y límites de tamaño de solicitud del WAF.

## <a name="waf-request-size-limits"></a>Límites de tamaño de solicitud del WAF

![Límites de tamaño de solicitud](media/application-gateway-waf-configuration/waf-requestsizelimit.png)

El firewall de aplicaciones web le permite configurar límites de tamaño de solicitud dentro de los límites inferior y superior. Hay disponibles dos configuraciones de límites de tamaño:

- El campo de tamaño del cuerpo de solicitud máximo se especifica en kilobytes y controles de límite de tamaño de solicitud general excluyendo cualquier archivo de carga. Este campo puede oscilar entre 1 KB como mínimo y 128 KB como máximo. El valor predeterminado para el tamaño del cuerpo de la solicitud es de 128 KB.
- El campo de límite de carga de archivo se especifica en MB y controla el tamaño máximo de carga de archivo permitido. Este campo puede tener un valor mínimo de 1 MB y un máximo de 500 MB para las instancias de SKU de grandes, mientras que las SKU medianas tiene un máximo de 100 MB. El valor predeterminado del límite de carga de archivo es 100 MB.

El WAF también ofrece un botón configurable para activar o desactivar la inspección del cuerpo de la solicitud. De forma predeterminada, la inspección del cuerpo de la solicitud está habilitada. Si se desactiva la inspección del cuerpo de solicitud, WAF no evalúa el contenido del cuerpo del mensaje HTTP. En tal caso, el WAF continúa aplicando reglas de WAF en URI, cookies y encabezados. Si se desactiva la inspección del cuerpo de la solicitud, el campo de tamaño máximo del cuerpo de la solicitud no es aplicable y no se puede definir. Al desactivar la inspección del cuerpo de la solicitud, se podrán enviar mensajes de más de 128 KB al WAF, pero no se inspecciona el cuerpo de mensaje en busca de vulnerabilidades.

## <a name="waf-exclusion-lists"></a>Listas de exclusión del WAF

![waf-exclusion.png](media/application-gateway-waf-configuration/waf-exclusion.png)

Las listas de exclusión del WAF le permiten omitir determinados atributos de solicitud de una evaluación del WAF. Un ejemplo común son los tokens insertados de Active Directory que se usan para campos de contraseña o autenticación. Estos atributos suelen contener caracteres especiales que podrían desencadenar un falso positivo de las reglas de WAF. Una vez que se agrega un atributo a la lista de exclusión del WAF, ninguna regla de WAF configurada y activa lo toma en consideración. Las listas de exclusión tienen un ámbito global.

Los siguientes atributos se pueden agregar a las listas de exclusión:

* Encabezados de solicitud
* Cookies de solicitud
* Nombre del atributo de solicitud (args)

   * Datos de varias partes del formulario
   * XML
   * JSON
   * Argumentos de la consulta de dirección URL

Puede especificar una coincidencia exacta con un atributo de cadena de consulta, una cookie, el cuerpo o el encabezado de una solicitud.  Si lo desea, también puede especificar coincidencias parciales. La exclusión siempre se aplica al campo del encabezado; nunca a su valor. Las reglas de exclusión tienen un ámbito global y se aplican a todas las páginas y todas las reglas.

Estos son los operadores de criterios de coincidencia admitidos:

- **Equals** (es igual a):  este operador se usa para una coincidencia exacta. Por ejemplo, para seleccionar el encabezado denominado **bearerToken**, utilice el operador de igualdad con el selector definido como **bearerToken**.
- **Starts with** (empieza por): este operador coincide con todos los campos que comienzan por el valor del selector especificado.
- **Ends with** (termina en):  este operador coincide con todos los campos de solicitud que terminan con el valor del selector especificado.
- **Contains** (contiene): este operador coincide con todos los campos de solicitud que contienen el valor del selector especificado.
- **Igual a cualquier**: Este operador coincide con todos los campos de solicitud. * será el valor del selector.

En todos los casos, la coincidencia distingue mayúsculas de minúsculas y no se permiten expresiones regulares como selectores.

### <a name="examples"></a>Ejemplos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Los ejemplos siguientes muestran el uso de exclusiones.

### <a name="example-1"></a>Ejemplo 1

En este ejemplo, desea excluir el encabezado user-agent. El encabezado de solicitud del agente de usuario contiene una cadena de característica que permite a los pares del protocolo de red identificar el tipo de aplicación, sistema operativo, fabricante de software o versión de software del agente de usuario solicitante de software. Para obtener más información, consulte [User-Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent).

Puede haber varias razones para deshabilitar la evaluación de este encabezado. Podría haber una cadena que el WAF ve y suponer que es malintencionado. Por ejemplo, el ataque SQL clásico "x = x" en una cadena. En algunos casos, esto puede ser tráfico legítimo. Por lo que es posible que deba excluir este encabezado de la evaluación de WAF.

El siguiente cmdlet de PowerShell de Azure excluye el encabezado user-agent de evaluación:

```azurepowershell
$exclusion1 = New-AzApplicationGatewayFirewallExclusionConfig `
   -MatchVariable "RequestHeaderNames" `
   -SelectorMatchOperator "Equals" `
   -Selector "User-Agent"
```

### <a name="example-2"></a>Ejemplo 2

En este ejemplo se excluye el valor en el *usuario* parámetro que se pasa a través de la dirección URL de la solicitud. Por ejemplo, supongamos que es común en su entorno para el campo de usuario que contenga una cadena que el Firewall se considera como contenido malintencionado, por lo que lo bloquea.  En este caso puede excluir el parámetro de usuario para que no evalúa el WAF nada en el campo.

El siguiente cmdlet de PowerShell de Azure excluye el parámetro de usuario de evaluación:

```azurepowershell
$exclusion2 = New-AzApplicationGatewayFirewallExclusionConfig `
   -MatchVariable "RequestArgNames" `
   -SelectorMatchOperator "Equals" `
   -Selector "user"
```
Por tanto, si la dirección URL **http://www.contoso.com/?user=fdafdasfda** se pasa en el firewall, no evalúa la cadena **fdafdasfda**.

## <a name="next-steps"></a>Pasos siguientes

Después de configurar el WAF, puede aprender a ver los registros del WAF. Para más información, consulte [Diagnósticos de Application Gateway](application-gateway-diagnostics.md#diagnostic-logging).
