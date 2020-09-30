---
title: Introducción al seguimiento de estado para Azure Application Gateway
description: Azure Application Gateway supervisa el estado de todos los recursos de su grupo de back-end y elimina automáticamente del grupo aquellos que se considera que están en mal estado.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 07/09/2020
ms.author: victorh
ms.openlocfilehash: f0e5a153efe26640e54f386600f07c7b3d4711d0
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89649064"
---
# <a name="application-gateway-health-monitoring-overview"></a>Información general sobre la supervisión de estado de la puerta de enlace de aplicaciones

La puerta de enlace de aplicaciones de Azure supervisa de forma predeterminada el estado de todos los recursos de su grupo de back-end y elimina automáticamente del grupo los recursos que se considera que están en mal estado. Además, continúa supervisando las instancias en mal estado y las agrega de nuevo al grupo de back-end en buen estado, una vez que están disponibles y responden a los sondeos de estado. De manera predeterminada, Application Gateway envía los sondeos de mantenimiento con el mismo puerto que se define en la configuración de HTTP de back-end. Un puerto de sondeo personalizado se puede configurar mediante un sondeo de estado personalizado.

La dirección IP de origen que usa Application Gateway para los sondeos de estado depende del grupo de back-end:
 
- Si la dirección del servidor en el grupo de back-end es un punto de conexión público, la dirección de origen es la dirección IP pública de front-end de Application Gateway.
- Si la dirección del servidor en el grupo de back-end es un punto de conexión privado, la dirección IP de origen proviene del espacio de direcciones IP privadas de la subred de Application Gateway.

![ejemplo de sondeo de Application Gateway][1]

Aparte del uso de la supervisión del sondeo de estado, también puede personalizar el sondeo de estado para adaptarlo a las necesidades de su aplicación. En este artículo trataremos ambos sondeos de estado: el personalizado y el predeterminado.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="default-health-probe"></a>Sondeo de estado predeterminado

Una puerta de enlace de aplicaciones configura automáticamente un sondeo de estado predeterminado cuando no hay ningún sondeo personalizado configurado. El comportamiento de supervisión consiste en realizar una solicitud HTTP GET a las direcciones IP o nombres de dominio completo configurados en el grupo de back-end. En el caso de los sondeos predeterminados si las opciones de HTTP del back-end se configuran para HTTPS, el sondeo usa HTTPS también para comprobar el mantenimiento de los servidores de back-end.

Por ejemplo: Configure la puerta de enlace de aplicaciones para usar los servidores back-end A, B y C para recibir el tráfico de red HTTP en el puerto 80. La supervisión de estado predeterminada comprueba los tres servidores cada 30 segundos para ver que la respuesta de HTTP es correcta con un tiempo de espera de 30 segundos para cada solicitud. Una respuesta HTTP correcta tiene un [código de estado](https://msdn.microsoft.com/library/aa287675.aspx) entre 200 y 399. En este caso, la solicitud HTTP GET para el sondeo de estado tendrá el aspecto http://127.0.0.1/.

Si se produce un error en la comprobación de sondeo predeterminado para el servidor A, Application Gateway deja de reenviar solicitudes a este servidor. El sondeo predeterminado sigue comprobando el servidor A cada 30 segundos. Cuando el servidor A responde correctamente a una solicitud de un sondeo de estado predeterminado, Application Gateway comienza a reenviar las solicitudes al servidor de nuevo.

### <a name="default-health-probe-settings"></a>Configuración de sondeo de estado predeterminado

| Propiedad de sondeo | Value | Descripción |
| --- | --- | --- |
| Dirección URL de sondeo |\<protocol\>://127.0.0.1:\<port\>/ |El protocolo y el puerto se heredan de la configuración de HTTP de back-end a la que está asociado el sondeo. |
| Intervalo |30 |Cantidad de tiempo en segundos que se debe esperar antes de que se envíe el siguiente sondeo de estado.|
| Tiempo de espera |30 |Cantidad de tiempo en segundos que la puerta de enlace de la aplicación espera una respuesta de sondeo antes de marcar dicho sondeo como incorrecto. Si un sondeo devuelve un estado correcto, el back-end correspondiente se marca inmediatamente como correcto.|
| Umbral incorrecto |3 |Controla cuántos sondeos se van a enviar si se produce un error en el sondeo de estado normal. En la SKU v1, estos sondeos de estado adicionales se envían en sucesión rápida para determinar el estado del back-end rápidamente y no esperar al intervalo de sondeo. En el caso de la SKU v2, los sondeos de estado esperan el intervalo. El servidor back-end se marca como inactivo después de que el número de errores de sondeo consecutivos alcanza el umbral incorrecto. |

El sondeo predeterminado solo se fija en \<protocol\>:\//127.0.0.1:\<port\> para determinar el estado de mantenimiento. Si necesita configurar el sondeo de estado para ir a una dirección URL personalizada o modificar alguna otra configuración, debe usar sondeos personalizados. Para obtener más información sobre los sondeos HTTPS, consulte [Introducción a la terminación TLS y a TLS de extremo a extremo con Application Gateway](ssl-overview.md#for-probe-traffic).

### <a name="probe-intervals"></a>Intervalos de sondeo

Todas las instancias de Application Gateway sondean el back-end de manera independiente unas de otras. La misma configuración de sondeo se aplica a cada instancia de Application Gateway. Por ejemplo, si la configuración de sondeo es enviar sondeos de estado cada 30 segundos y Application Gateway tiene dos instancias, entonces ambas instancias envían el sondeo de estado cada 30 segundos.

También si hay varios agentes de escucha, de manera que cada uno de ellos sondea el back-end de manera independiente unos de otros. Por ejemplo, si existen dos agentes de escucha que apuntan al mismo grupo de back-ends en dos puertos diferentes (definidos por dos configuraciones http de back-end), entonces cada agente de escucha sondea el mismo back-end de forma independiente. En este caso, hay dos sondeos procedentes de cada instancia de Application Gateway para los dos agentes de escucha. Si hay dos instancias de la puerta de Application Gateway en este escenario, la máquina virtual de back-end vería cuatro sondeos por el intervalo de sondeo configurado.

## <a name="custom-health-probe"></a>Sondeo de estado personalizado

Los sondeos personalizados permiten un control más específico sobre la supervisión de estado. Cuando se usan sondeos personalizados, puede configurar un nombre de host personalizado, la ruta de acceso de la dirección URL, el intervalo de sondeo y la cantidad de respuestas erróneas que se aceptan antes de marcar la instancia del grupo de back-end como en mal estado, etc.

### <a name="custom-health-probe-settings"></a>Configuración de sondeo de estado personalizado

La siguiente tabla proporciona definiciones de las propiedades de un sondeo de mantenimiento personalizado.

| Propiedad de sondeo | Descripción |
| --- | --- |
| Nombre |Nombre del sondeo. Este nombre se usa para identificar y hacer referencia al sondeo en la configuración de HTTP de back-end. |
| Protocolo |Protocolo usado para enviar el sondeo. Tiene que coincidir con el protocolo definido en la configuración de HTTP de back-end a la que está asociado.|
| Host |Nombre de host con el que enviar el sondeo. En la SKU v1, este valor solo se usará para el encabezado de host de la solicitud de sondeo. En la SKU v2, se usarán como encabezado de host y SNI. |
| Path |Ruta de acceso relativa del sondeo. Las rutas de acceso válidas comienzan por '/' |
| Port |Si se define, se usa como puerto de destino. De lo contrario, utiliza el mismo puerto que la configuración de HTTP a la que está asociado. Esta propiedad solo está disponible en la SKU v2.
| Intervalo |Intervalo de sondeo en segundos. Este valor es el intervalo de tiempo entre dos sondeos consecutivos. |
| Tiempo de espera |Tiempo de espera del sondeo en segundos. Si no se recibe una respuesta válida dentro del período de espera, el sondeo se marca como erróneo.  |
| Umbral incorrecto |Número de reintentos de sondeo. El servidor back-end se marca como inactivo después de que el número de errores de sondeo consecutivos alcanza el umbral incorrecto. |

### <a name="probe-matching"></a>Sondeo de búsqueda de coincidencia

De forma predeterminada, una respuesta HTTP (S) con el código de estado entre 200 y 399 se considera correcta. Los sondeos de estado personalizados admiten además dos criterios de coincidencia. Los criterios de coincidencia pueden usarse para modificar opcionalmente la interpretación predeterminada de lo que constituye una respuesta correcta.

Estos son los criterios de coincidencia: 

- **Coincidencia de código de estado de respuesta HTTP**: criterio de coincidencia de sondeo para aceptar el código de respuesta, o los intervalos de códigos de respuesta, HTTP especificados por el usuario. Se admiten códigos de estado de respuesta individuales, o un intervalo de códigos de estado de respuesta, separados por coma.
- **Coincidencia de cuerpo de respuesta HTTP**: criterio de coincidencia de sondeo que examina el cuerpo de la respuesta HTTP y busca la coincidencia con una cadena especificada por el usuario. La coincidencia solo busca la existencia de la cadena especificada por el usuario en el cuerpo de la respuesta, no es una coincidencia de expresión regular completa.

Los criterios de coincidencia se pueden especificar mediante el cmdlet `New-AzApplicationGatewayProbeHealthResponseMatch`.

Por ejemplo:

```azurepowershell
$match = New-AzApplicationGatewayProbeHealthResponseMatch -StatusCode 200-399
$match = New-AzApplicationGatewayProbeHealthResponseMatch -Body "Healthy"
```
Una vez que se especifican los criterios de coincidencia, se pueden asociar a la configuración de sondeo con un parámetro `-Match` en PowerShell.

## <a name="nsg-considerations"></a>Consideraciones sobre NSG

Debe permitir el tráfico entrante de Internet en los puertos TCP 65503-65534 para la SKU de Application Gateway v1, y los puertos TCP 65200-65535 para la SKU de V2 con la subred de destino como **cualquiera** y origen como la etiqueta de servicio de **GatewayManager**. Este intervalo de puertos es necesario para la comunicación de la infraestructura de Azure.

Además, no se puede bloquear la conectividad saliente de Internet y se debe permitir el tráfico entrante desde la etiqueta **AzureLoadBalancer**.

Para más información, consulte [Introducción a la configuración de Application Gateway](configuration-infrastructure.md#network-security-groups).

## <a name="next-steps"></a>Pasos siguientes
Tras conocer todo lo referente a la supervisión del mantenimiento de Application Gateway, puede configurar un [sondeo de mantenimiento personalizado](application-gateway-create-probe-portal.md) en Azure Portal o un [sondeo de mantenimiento personalizado](application-gateway-create-probe-ps.md) mediante PowerShell y el modelo de implementación con Azure Resource Manager.

[1]: ./media/application-gateway-probe-overview/appgatewayprobe.png
