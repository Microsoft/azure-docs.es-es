---
title: Puerta de enlace de datos local para Azure Analysis Services | Microsoft Docs
description: Necesitará una puerta de enlace local si el servidor de Analysis Services de Azure se debe conectar a orígenes de datos locales.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: ee332eb7dea86e07c2d8f9b75a0e152dc7482a41
ms.sourcegitcommit: 14bf4129a73de2b51a575c3a0a7a3b9c86387b2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87438822"
---
# <a name="connecting-to-on-premises-data-sources-with-on-premises-data-gateway"></a>Conexión a orígenes de datos locales con la puerta de enlace de datos local

La puerta de enlace de datos local proporciona una transferencia de datos segura entre orígenes de datos locales y los servidores de Azure Analysis Services en la nube. Además de funcionar con varios servidores de Azure Analysis Services en la misma región, la versión más reciente de la puerta de enlace también funciona con Azure Logic Apps, Power BI, Power Apps y Power Automate. Si bien la puerta de enlace que instala es la misma en todos estos servicios, Azure Analysis Services y Logic Apps tienen algunos pasos adicionales.

La información que se proporciona aquí se refiere específicamente al funcionamiento de Azure Analysis Services con la puerta de enlace de datos local. Para más información sobre la puerta de enlace en general y cómo funciona con otros servicios, consulte [¿Qué es una puerta de enlace de datos local?](/data-integration/gateway/service-gateway-onprem).

Para Azure Analysis Services, el proceso de instalación de la puerta de enlace consta de cuatro partes:

- **Descargar y ejecutar el programa de instalación**: en este paso se instala un servicio de puerta de enlace en un equipo de la organización. También se inicia sesión en Azure con la cuenta de Azure AD del [inquilino](/previous-versions/azure/azure-services/jj573650(v=azure.100)#what-is-an-azure-ad-tenant). No se pueden usar cuentas B2B (invitadas) de Azure.

- **Registrar la puerta de enlace**: en este paso se especifica el nombre y la clave de recuperación de la puerta de enlace y se selecciona la región, de modo que la puerta de enlace se registra en el servicio en la nube de la puerta de enlace. El recurso de puerta de enlace se puede registrar en cualquier región, pero es recomendable que esté en la misma que los servidores de Analysis Services. 

- **Creación de un recurso de puerta de enlace en Azure**: en este paso, creará un recurso de puerta de enlace en Azure.

- **Conexión del recurso de puerta de enlace a los servidores**: una vez que tenga un recurso de puerta de enlace, podrá comenzar a conectarle servidores. Puede conectar varios servidores y otros recursos, siempre que se encuentren en la misma región.



## <a name="how-it-works"></a>Funcionamiento
La puerta de enlace que se instala en un equipo de la organización funciona como un servicio de Windows, **Puerta de enlace de datos local**. Este servicio local se registra en el servicio en la nube de puerta de enlace a través de Azure Service Bus. Después debe crear un recurso de puerta de enlace de datos local para una suscripción de Azure. Posteriormente, los servidores de Azure Analysis Services se conectan al recurso de puerta de enlace de Azure. Cuando los modelos del servidor necesitan conectarse a los orígenes de datos locales para realizar consultas o procesamiento, un flujo de datos y consultas atraviesa el recurso de puerta de enlace, Azure Service Bus, el servicio de puerta de enlace de datos local y los orígenes de datos. 

![Funcionamiento](./media/analysis-services-gateway/aas-gateway-how-it-works.png)

Flujo de datos y consultas:

1. El servicio en la nube crea una consulta con las credenciales cifradas para el origen de datos local. A continuación, se envía a una cola de la puerta de enlace para su procesamiento.
2. El servicio en la nube de puerta de enlace analiza la consulta e inserta la solicitud en la instancia de [Azure Service Bus](https://azure.microsoft.com/documentation/services/service-bus/).
3. La puerta de enlace de datos local sondea Azure Service Bus en busca de solicitudes pendientes.
4. La puerta de enlace obtiene la consulta, descifra las credenciales y se conecta a los orígenes de datos con ellas.
5. La puerta de enlace envía la consulta al origen de datos para su ejecución.
6. Los resultados se devuelven desde el origen de datos a la puerta de enlace y luego se envían al servicio en la nube y al servidor.

## <a name="installing"></a>Instalación

En una instalación para un entorno de Azure Analysis Services, es importante que siga los pasos descritos en [Instalación y configuración de una puerta de enlace de datos local para Azure Analysis Services](analysis-services-gateway-install.md). Este artículo es específico para Azure Analysis Services. Incluye pasos adicionales necesarios para configurar un recurso de puerta de enlace de datos local en Azure y conectar el servidor de Azure Analysis Services al recurso.

## <a name="connecting-to-a-gateway-resource-in-a-different-subscription"></a>Conexión a un recurso de puerta de enlace de otra suscripción

Es conveniente que cree el recurso de puerta de enlace de Azure en la misma suscripción que el servidor. No obstante, puede configurar los servidores para que se conecten a un recurso de puerta de enlace de otra suscripción. No se puede establecer conexión con un recurso de puerta de enlace de otra suscripción cuando se configura un servidor existente o se crea un nuevo servidor en el portal, aunque sí puede configurarse mediante PowerShell. Para más información, consulte [Conexión de un recurso de puerta de enlace a un servidor](analysis-services-gateway-install.md#connect-gateway-resource-to-server).

## <a name="ports-and-communication-settings"></a>Configuración de puertos y comunicación

La puerta de enlace crea una conexión de salida con Azure Service Bus. Se comunica en los puertos de salida siguientes: TCP 443 (predeterminado), 5671, 5672 y del 9350 al 9354.  La puerta de enlace no requiere puertos de entrada.

Es posible que tenga que incluir direcciones IP de la región de datos en el firewall. Puede descargar la [lista de direcciones IP del centro de datos de Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=56519). La lista se actualiza semanalmente. Las direcciones IP mostradas en la lista de direcciones IP del centro de datos de Azure están en notación CIDR. Para obtener más información, consulte [Enrutamiento de interdominios sin clases](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

Estos son los nombres de dominio completos usados por la puerta de enlace.

| Nombres de dominio | Puertos de salida | Descripción |
| --- | --- | --- |
| *.powerbi.com |80 |HTTP que se utiliza para descargar el instalador. |
| *.powerbi.com |443 |HTTPS |
| *. analysis.windows.net |443 |HTTPS |
| *.login.windows.net, login.live.com, aadcdn.msauth.net |443 |HTTPS |
| *.servicebus.windows.net |5671-5672 |Advanced Message Queuing Protocol (AMQP) |
| *.servicebus.windows.net |443, 9350-9354 |Agentes de escucha de Service Bus Relay sobre TCP (requiere 443 para la adquisición del token de Access Control) |
| *.frontend.clouddatahub.net |443 |HTTPS |
| *.core.windows.net |443 |HTTPS |
| login.microsoftonline.com |443 |HTTPS |
| *.msftncsi.com |443 |Se utiliza para probar la conectividad a Internet si el servicio Power BI no puede acceder a la puerta de enlace. |
| *.microsoftonline-p.com |443 |Se utiliza para la autenticación dependiendo de la configuración. |
| dc.services.visualstudio.com    |443 |Usado por AppInsights para recopilar datos de telemetría. |

### <a name="forcing-https-communication-with-azure-service-bus"></a>Cómo forzar la comunicación HTTPS con Azure Service Bus

Puede hacer que la puerta de enlace se comunique con Azure Service Bus mediante HTTPS en lugar de TCP directo. Sin embargo, de ese modo puede reducir en gran medida el rendimiento. Debe modificar el archivo *Microsoft.PowerBI.DataMovement.Pipeline.GatewayCore.dll.config* cambiando el valor de `AutoDetect` por `Https`. Este archivo suele encontrarse en *C:\Archivos de programa\Puerta de enlace de datos local*.

```
<setting name="ServiceBusSystemConnectivityModeString" serializeAs="String">
    <value>Https</value>
</setting>
```

## <a name="next-steps"></a>Pasos siguientes 

Los artículos siguientes se incluyen en el contenido general de la puerta de enlace de datos local que se aplica a todos los servicios que admite la puerta de enlace:

* [Preguntas más frecuentes sobre la puerta de enlace de datos local](https://docs.microsoft.com/data-integration/gateway/service-gateway-onprem-faq)   
* [Uso de la aplicación Puerta de enlace de datos local](https://docs.microsoft.com/data-integration/gateway/service-gateway-app)   
* [Administración en el nivel de inquilino](https://docs.microsoft.com/data-integration/gateway/service-gateway-tenant-level-admin)
* [Configuración de los valores de proxy](https://docs.microsoft.com/data-integration/gateway/service-gateway-proxy)   
* [Ajuste de la configuración de comunicación](https://docs.microsoft.com/data-integration/gateway/service-gateway-communication)   
* [Configurar los archivos de registro](https://docs.microsoft.com/data-integration/gateway/service-gateway-log-files)   
* [Solución de problemas](https://docs.microsoft.com/data-integration/gateway/service-gateway-tshoot)
* [Supervisión y optimización del rendimiento de la puerta de enlace](https://docs.microsoft.com/data-integration/gateway/service-gateway-performance)
