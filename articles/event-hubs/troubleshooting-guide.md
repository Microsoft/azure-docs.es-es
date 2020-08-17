---
title: 'Solución de problemas de conectividad: Azure Event Hubs | Microsoft Docs'
description: En este artículo se proporciona información sobre cómo solucionar problemas de conectividad con Azure Event Hubs.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: b85c0895d1c8f165f494d29013adea014187dd23
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87039334"
---
# <a name="troubleshoot-connectivity-issues---azure-event-hubs"></a>Solución de problemas de conectividad: Azure Event Hubs
Hay varias razones por las que las aplicaciones cliente no pueden conectarse a un centro de eventos. Los problemas de conectividad que experimenta pueden ser permanentes o transitorios. Si el problema sucede todo el tiempo (permanente), puede que desee comprobar la cadena de conexión, la configuración del firewall de la organización, la configuración del firewall de IP, la configuración de seguridad de la red (puntos de conexión de servicio, puntos de conexión privados, etc.), etc. En el caso de problemas transitorios, la actualización a la versión más reciente del SDK, la ejecución de comandos para comprobar los paquetes descartados y la obtención de seguimientos de red pueden ayudar a solucionar los problemas. 

En este artículo se proporcionan consejos sobre cómo solucionar problemas de conectividad con Azure Event Hubs. 

## <a name="troubleshoot-permanent-connectivity-issues"></a>Solución de problemas de conectividad permanentes
Si la aplicación no puede conectarse al centro de eventos, siga los pasos de esta sección para solucionar el problema. 

### <a name="check-if-there-is-a-service-outage"></a>Compruebe si hay una interrupción del servicio
Compruebe la interrupción del servicio de Azure Event Hubs en el [sitio de estado del servicio de Azure](https://azure.microsoft.com/status/).

### <a name="verify-the-connection-string"></a>Compruebe la cadena de conexión 
Compruebe que la cadena de conexión que está usando es correcta. Consulte [Obtención de una cadena de conexión de Event Hubs](event-hubs-get-connection-string.md) para obtener la cadena de conexión mediante Azure Portal, la CLI o PowerShell. 

Para clientes de Kafka, compruebe que los archivos productor.config o consumer.config están configurados correctamente. Para más información, consulte [Envío y recepción de mensajes con Kafka en Event Hubs](event-hubs-quickstart-kafka-enabled-event-hubs.md#send-and-receive-messages-with-kafka-in-event-hubs).

### <a name="check-if-the-ports-required-to-communicate-with-event-hubs-are-blocked-by-organizations-firewall"></a>Compruebe si los puertos necesarios para comunicarse con Event Hubs están bloqueados por el firewall de la organización
Compruebe que los puertos utilizados para comunicarse con Azure Event Hubs no estén bloqueados en el firewall de la organización. Consulte en la siguiente tabla los puertos de salida que se deben abrir para comunicarse con Azure Event Hubs. 

| Protocolo | Puertos | Detalles | 
| -------- | ----- | ------- | 
| AMQP | 5671 y 5672 | Consulte la [guía del protocolo AMQP](../service-bus-messaging/service-bus-amqp-protocol-guide.md) | 
| HTTP, HTTPS | 80, 443 |  |
| Kafka | 9093 | Consulte [Uso de Azure Event Hubs desde aplicaciones de Apache Kafka](event-hubs-for-kafka-ecosystem-overview.md)

Este es un comando de ejemplo que comprueba si el puerto 5671 está bloqueado.

```powershell
tnc <yournamespacename>.servicebus.windows.net -port 5671
```

En Linux:

```shell
telnet <yournamespacename>.servicebus.windows.net 5671
```

### <a name="verify-that-ip-addresses-are-allowed-in-your-corporate-firewall"></a>Compruebe que las direcciones IP se permiten en el firewall corporativo
Cuando trabaja con Azure, en ocasiones tiene que permitir intervalos de direcciones IP específicos o direcciones URL en el firewall o proxy corporativo para tener acceso a todos los servicios de Azure que está usando o intentando usar. Compruebe que se permite el tráfico en las direcciones IP utilizadas por Event Hubs. En el caso de las direcciones IP usadas por Azure Event Hubs, vea [Intervalos de direcciones IP y etiquetas de servicio de Azure: nube pública](https://www.microsoft.com/download/details.aspx?id=56519).

Además, compruebe que se permite la dirección IP del espacio de nombres. Para buscar las direcciones IP correctas para permitirlas para las conexiones, siga estos pasos:

1. Ejecute el siguiente comando desde el símbolo del sistema: 

    ```
    nslookup <YourNamespaceName>.servicebus.windows.net
    ```
2. Anote la dirección IP devuelta en `Non-authoritative answer`. El único momento en que cambiaría es si restaurara el espacio de nombres en un clúster distinto.

Si usa la redundancia de zona para el espacio de nombres, deberá realizar algunos pasos adicionales: 

1. En primer lugar, ejecute nslookup en el espacio de nombres.

    ```
    nslookup <yournamespace>.servicebus.windows.net
    ```
2. Anote el nombre de la sección **respuesta no autoritativa**, que se encuentra en uno de los siguientes formatos: 

    ```
    <name>-s1.cloudapp.net
    <name>-s2.cloudapp.net
    <name>-s3.cloudapp.net
    ```
3. Ejecute nslookup para cada uno con los sufijos s1, s2 y s3 para obtener las direcciones IP de las tres instancias que se ejecutan en tres zonas de disponibilidad. 

### <a name="verify-that-azureeventgrid-service-tag-is-allowed-in-your-network-security-groups"></a>Compruebe que se permite la etiqueta de servicio AzureEventGrid en los grupos de seguridad de red
Si la aplicación se ejecuta dentro de una subred y hay un grupo de seguridad de red asociado, confirme si se permite la salida de Internet o la etiqueta de servicio AzureEventGrid. Vea [Etiquetas de servicio de red virtual](../virtual-network/service-tags-overview.md) y busque `EventHub`.

### <a name="check-if-the-application-needs-to-be-running-in-a-specific-subnet-of-a-vnet"></a>Comprobar si la aplicación debe ejecutarse en una subred específica de una red virtual
Confirme que la aplicación se está ejecutando en una subred de red virtual que tiene acceso al espacio de nombres. Si no es así, ejecute la aplicación en la subred que tenga acceso al espacio de nombres o agregue la dirección IP del equipo en el que se ejecuta la aplicación al [firewall de IP](event-hubs-ip-filtering.md). 

Cuando se crea un punto de conexión de servicio de red virtual para un espacio de nombres del centro de eventos, el espacio de nombres solamente acepta tráfico de la subred enlazada al punto de conexión de servicio. Hay una excepción a este comportamiento. Puede agregar direcciones IP específicas en el firewall de IP para habilitar el acceso al punto de conexión público del centro de eventos. Para obtener más información, vea [Puntos de conexión de servicio de red](event-hubs-service-endpoints.md).

### <a name="check-the-ip-firewall-settings-for-your-namespace"></a>Comprobar la configuración del firewall de IP para su espacio de nombres
Compruebe que el firewall de IP no bloquea la dirección IP pública de la máquina en la que se ejecuta la aplicación.  

De forma predeterminada, los espacios de nombres de Azure Event Hubs son accesibles desde Internet, siempre que la solicitud venga con una autenticación y una autorización válidas. Con el firewall de IP, puede restringirlo aún más a solo un conjunto de direcciones o intervalos de direcciones IPv4 en notación [CIDR (Enrutamiento de interdominios sin clases)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

Las reglas de firewall de IP se aplican en el nivel del espacio de nombres de Event Hubs. Por lo tanto, las reglas se aplican a todas las conexiones de clientes que usan cualquier protocolo admitido. Cualquier intento de conexión desde una dirección IP que no coincida con una regla IP admitida en el espacio de nombres de Event Hubs se rechaza como no autorizado. La respuesta no menciona la regla IP. Las reglas de filtro IP se aplican en orden y la primera regla que coincida con la dirección IP determina la acción de aceptar o rechazar.

Para obtener más información, consulte [Configuración de reglas de firewall de IP para un espacio de nombres de Azure Event Hubs](event-hubs-ip-filtering.md). Para comprobar si tiene problemas de filtrado de direcciones IP, red virtual o cadena de certificados, consulte [Solución de problemas relacionados con la red](#troubleshoot-network-related-issues).

#### <a name="find-the-ip-addresses-blocked-by-ip-firewall"></a>Buscar las direcciones IP bloqueadas por el firewall de IP
Habilite los registros de diagnóstico para [eventos de conexión de red virtual de Event Hubs](event-hubs-diagnostic-logs.md#event-hubs-virtual-network-connection-event-schema) siguiendo las instrucciones indicadas en [Habilitar registros de diagnóstico](event-hubs-diagnostic-logs.md#enable-diagnostic-logs). Verá la dirección IP para la conexión que se deniega.

```json
{
    "SubscriptionId": "0000000-0000-0000-0000-000000000000",
    "NamespaceName": "namespace-name",
    "IPAddress": "1.2.3.4",
    "Action": "Deny Connection",
    "Reason": "IPAddress doesn't belong to a subnet with Service Endpoint enabled.",
    "Count": "65",
    "ResourceId": "/subscriptions/0000000-0000-0000-0000-000000000000/resourcegroups/testrg/providers/microsoft.eventhub/namespaces/namespace-name",
    "Category": "EventHubVNetConnectionEvent"
}
```

### <a name="check-if-the-namespace-can-be-accessed-using-only-a-private-endpoint"></a>Comprobar si se puede acceder al espacio de nombres con solo un punto de conexión privado
Si el espacio de nombres de Event Hubs está configurado para ser accesible únicamente a través de un punto de conexión privado, confirme que la aplicación cliente accede al espacio de nombres a través del punto de conexión privado. 

El [servicio Azure Private Link](../private-link/private-link-overview.md) permite el acceso a Azure Event Hubs a través de un **punto de conexión privado** en la red virtual. Un punto de conexión privado es una interfaz de red que le conecta de forma privada y segura a un servicio con la tecnología de Azure Private Link. El punto de conexión privado usa una dirección IP privada de la red virtual para incorporar el servicio de manera eficaz a su red virtual. Todo el tráfico dirigido al servicio se puede enrutar mediante el punto de conexión privado, por lo que no se necesita ninguna puerta de enlace, dispositivos NAT, conexiones de ExpressRoute o VPN ni direcciones IP públicas. El tráfico entre la red virtual y el servicio atraviesa la red troncal de Microsoft, eliminando la exposición a la red pública de Internet. Puede conectarse a una instancia de un recurso de Azure, lo que le otorga el nivel más alto de granularidad en el control de acceso.

Para obtener más información, consulte [Configuración de puntos de conexión privados](private-link-service.md). Vea la sección **Validación de que la conexión del punto de conexión privado funciona** para confirmar que se usa un punto de conexión privado. 

### <a name="troubleshoot-network-related-issues"></a>Solución de problemas relacionados con la red
Para solucionar problemas relacionados con la red con Event Hubs, siga estos pasos: 

Desplácese a `https://<yournamespacename>.servicebus.windows.net/` o use [wget](https://www.gnu.org/software/wget/) ir allí. Le ayudará a comprobar si tiene problemas de cadena de certificados (lo más común al usar el SDK de Java), filtrado IP o red virtual.

Un ejemplo de **mensaje correcto**:

```xml
<feed xmlns="http://www.w3.org/2005/Atom"><title type="text">Publicly Listed Services</title><subtitle type="text">This is the list of publicly-listed services currently available.</subtitle><id>uuid:27fcd1e2-3a99-44b1-8f1e-3e92b52f0171;id=30</id><updated>2019-12-27T13:11:47Z</updated><generator>Service Bus 1.1</generator></feed>
```

Un ejemplo de **mensaje de error**:

```json
<Error>
    <Code>400</Code>
    <Detail>
        Bad Request. To know more visit https://aka.ms/sbResourceMgrExceptions. . TrackingId:b786d4d1-cbaf-47a8-a3d1-be689cda2a98_G22, SystemTracker:NoSystemTracker, Timestamp:2019-12-27T13:12:40
    </Detail>
</Error>
```

## <a name="troubleshoot-transient-connectivity-issues"></a>Solución de problemas de conectividad transitorios
Si tiene problemas de conectividad intermitentes, consulte las siguientes secciones para obtener sugerencias para la solución de problemas. 

### <a name="use-the-latest-version-of-the-client-sdk"></a>Usar la versión más reciente del SDK
Es posible que algunos de los problemas de conectividad transitorios se hayan corregido en las versiones posteriores del SDK de las que está usando. Asegúrese de que está usando la versión más reciente de los SDK cliente en las aplicaciones. Los SDK se han mejorado continuamente con características nuevas o actualizadas y correcciones de errores, por lo que siempre debe probar con el paquete más reciente. Compruebe las notas de la versión para conocer los problemas corregidos y las características agregadas o actualizadas. 

Para obtener información sobre los SDK cliente, consulte el artículo [Azure Event Hubs: SDK cliente](sdks.md). 

### <a name="run-the-command-to-check-dropped-packets"></a>Ejecutar el comando para comprobar los paquetes descartados
Si hay problemas de conectividad intermitentes, ejecute el siguiente comando para comprobar si hay paquetes descartados. Este comando intentará establecer 25 conexiones TCP diferentes cada segundo con el servicio. A continuación, puede comprobar cuántas de ellas se han realizado correctamente y cuántas han fallado y, además, ver la latencia de conexión TCP. Puede descargar la herramienta `psping` desde [aquí](/sysinternals/downloads/psping).

```shell
.\psping.exe -n 25 -i 1 -q <yournamespacename>.servicebus.windows.net:5671 -nobanner     
```
Puede usar comandos equivalentes si utiliza otras herramientas como `tnc`, `ping`, etc. 

Realice un seguimiento de red si los pasos anteriores no ayudan y analícelo con herramientas como [Wireshark](https://www.wireshark.org/). Si lo necesita, póngase en contacto con el [soporte técnico de Microsoft](https://support.microsoft.com/). 

### <a name="service-upgradesrestarts"></a>Actualizaciones o reinicios del servicio
Pueden producirse problemas de conectividad transitorios debido a actualizaciones y reinicios del servicio back-end. Cuando se producen, es posible que vea los síntomas siguientes: 

- Puede haber una caída en la llegada de mensajes o solicitudes entrantes.
- El archivo de registro puede contener mensajes de error.
- Puede que las aplicaciones se desconecten del servicio durante unos segundos.
- Puede que las solicitudes se limiten momentáneamente.

Si el código de aplicación usa SDK, la directiva de reintento ya está integrada y activa. La aplicación se volverá a conectar sin un impacto significativo en la aplicación o el flujo de trabajo. La detección de estos errores transitorios, la interrupción y el posterior reintento de la llamada garantizará que el código sea resistente a estos problemas transitorios.

## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes:

* [Solución de problemas de autenticación y autorización](troubleshoot-authentication-authorization.md)
