---
title: Back-end y grupos de back-end en Azure Front Door | Microsoft Docs
description: En este artículo se describe qué back-end y grupos de back-end se encuentran en la configuración de Front Door.
services: front-door
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/10/2018
ms.author: duau
ms.openlocfilehash: 66767d4329a0a757de99308e1f586b56b327a515
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89399929"
---
# <a name="backends-and-backend-pools-in-azure-front-door"></a>Back-end y grupos de back-end en Azure Front Door
En este artículo se describen conceptos acerca de cómo asignar la implementación de aplicaciones con Azure Front Door. También se explican los distintos términos en la configuración de Front Door sobre back-end de aplicación.

## <a name="backends"></a>Backends
Un back-end es igual a la instancia de implementación de una aplicación en una región. Front Door es compatible con back-ends de Azure y que no son de Azure, por lo que la región no está restringida solo a las regiones de Azure. Además, puede ser su centro de datos local o una instancia de aplicación en otra nube.

Los back-end de Front Door hacen referencia al nombre de host o a la dirección IP pública de la aplicación, que puede atender solicitudes de cliente. Los back-ends no deben confundirse con el nivel de base de datos, la capa de almacenamiento, etc. Los back-ends deben considerarse como el punto de conexión público del back-end de aplicación. Al agregar un back-end en un grupo de back-end de Front Door, también debe agregar lo siguiente:

- **Tipo de host de back-end**. El tipo de recurso que desea agregar. Front Door es compatible con la detección automática de back-end de aplicación del servicio de aplicación, el servicio en la nube o el almacenamiento. Si desea usar un recurso diferente en Azure o incluso un back-end ajeno a Azure, seleccione **Host personalizado**.

    >[!IMPORTANT]
    >Durante la configuración, las API no validan si el back-end no es accesible desde los entornos de Front Door. Asegúrese de que Front Door puede llegar a su back-end.

- **Suscripción y nombre de host de back-end**. Si no ha seleccionado **Host personalizado** para el tipo de host de back-end, seleccione el back-end eligiendo la suscripción adecuada y el nombre de host de back-end correspondiente en la interfaz de usuario.

- **Encabezado de host de back-end**. El valor del encabezado de host enviado al back-end para cada solicitud. Para más información, consulte [Encabezado de host de back-end](#hostheader).

- **Prioridad**. Asigne prioridades a los diferentes back-end cuando desee usar un back-end de servicio principal para todo el tráfico. Además, proporcione las copias de seguridad si los back-end principal o de copia de seguridad no están disponibles. Para más información, consulte [Prioridad](front-door-routing-methods.md#priority).

- **Peso**. Asigne pesos a sus diferentes back-end para distribuir el tráfico entre un conjunto de back-end, ya sea de manera uniforme o según los coeficientes de peso. Para más información, vea [Pesos](front-door-routing-methods.md#weighted).

### <a name="backend-host-header"></a><a name = "hostheader"></a>Encabezado de host de back-end

Las solicitudes reenviadas por Front Door a un back-end incluyen un campo de encabezado de host que el back-end usa para recuperar el recurso de destino. El valor de este campo normalmente procede del URI del back-end y tiene el host y el puerto.

Por ejemplo, una solicitud realizada para `www.contoso.com` tendrá el encabezado de host www.contoso.com. Si usa Azure Portal para configurar su back-end, el valor predeterminado para este campo es el nombre de host del back-end. Si el back-end es contoso-westus.azurewebsites.net, en Azure Portal, el valor que se rellena automáticamente el encabezado de host de back-end será contoso-westus.azurewebsites.net. En cambio, si se usan plantillas de Azure Resource Manager u otro método sin establecer explícitamente este campo, Front Door enviará el nombre de host entrante como el valor del encabezado de host. Si la solicitud se realizó para www\.contoso.com y el back-end es contoso-westus.azurewebsites.net que tiene un campo de encabezado vacío, Front Door establecerá el encabezado de host como www\.contoso.com.

La mayoría de back-end de aplicación (por ejemplo, Azure Web Apps, Blob Storage y Cloud Services) requieren que el encabezado de host coincida con el dominio del back-end. En cambio, el host de front-end que se enrute al back-end usará otro nombre de host, como www.contoso.net.

Si el back-end requiere que el encabezado de host coincida con el nombre de host de back-end, asegúrese de que el encabezado de host de back-end incluya el back-end de nombre de host.

#### <a name="configuring-the-backend-host-header-for-the-backend"></a>Configurar el encabezado de host de back-end para el back-end

Para configurar el campo **Encabezado host de back-end** para un back-end en la sección del grupo de back-end:

1. Abra el recurso de Front Door y seleccione el grupo de back-end con el back-end que se va a configurar.

2. Agregue un back-end si no lo ha hecho o edite uno existente.

3. Establezca el campo de encabezado de host de back-end en un valor personalizado o déjelo en blanco. El nombre de host de la solicitud entrante se utilizará como el valor del encabezado de host.

## <a name="backend-pools"></a>Grupos back-end
Un grupo de back-end en Front Door hace referencia al conjunto de back-end que reciben tráfico similar para las aplicaciones. En otras palabras, es una agrupación lógica de las instancias de aplicación en todo el mundo que reciben el mismo tráfico y responden con el comportamiento esperado. Estos back-end se implementan en diferentes regiones o en la misma región. Todos los back-end puede estar en modo de implementación activo/activo, o lo que se define como configuración activa/pasiva.

Un grupo de back-end define cómo se deben evaluar los diferentes back-end a través de los sondeos de estado. También define cómo se produce el equilibrio de carga entre ellos.

### <a name="health-probes"></a>Sondeos de estado
Front Door envía solicitudes periódicas de sondeo HTTP/HTTPS a cada uno de los back-end configurados. Las solicitudes de sondeo determinan la proximidad y el estado de cada back-end para equilibrar la carga de las solicitudes del usuario final. La configuración de sondeo de estado para un grupo de back-end define cómo se sondea el estado de mantenimiento de los back-end de la aplicación. Las siguientes opciones están disponibles para la configuración del equilibrio de carga:

- **Ruta de acceso**: La dirección URL utilizada para las solicitudes de sondeo para todos los back-end del grupo de back-end. Por ejemplo, si uno de los back-end es contoso-westus.azurewebsites.net y la ruta de acceso se establece en /probe/test.aspx, los entornos de Front Door, suponiendo que el protocolo está establecido en HTTP, enviarán las solicitudes de sondeo de estado a http\://contoso-westus.azurewebsites.net/probe/test.aspx.

- **Protocolo**: Define si las solicitudes de sondeo de estado desde Front Door a los back-end se enviarán a través del protocolo HTTP o HTTPS.

- **Método**: método HTTP que se va a usar para enviar sondeos de estado. Las opciones incluyen GET o HEAD (valor predeterminado).
    > [!NOTE]
    > Para reducir la carga y el costo de los back-end, Front Door recomienda el uso de solicitudes HEAD para sondeos de estado.

- **Intervalo (segundos)** : Este campo define la frecuencia de los sondeos de estado para los back-end o los intervalos en que cada uno de los entornos de Front Door envía un sondeo.

    >[!NOTE]
    >Para las conmutaciones por error rápidas, establezca el intervalo en un valor inferior. Cuanto menor sea el valor, mayor será el volumen de sondeo de estado que reciben los back-end. Por ejemplo, si el intervalo se establece en 30 segundos con, por ejemplo, 100 POP de Front Door globalmente, cada back-end recibirá en torno a 200 solicitudes de sondeo por minuto.

Para más información, consulte [Sondeos de estado](front-door-health-probes.md).

### <a name="load-balancing-settings"></a>Configuración de equilibrio de carga
La configuración del equilibrio de carga para el grupo de back-end define cómo se evalúan los sondeos de estado. Esta configuración determina si el back-end es correcto o incorrecto. También comprueba cómo equilibrar la carga del tráfico entre diferentes back-end en el grupo de back-end. Las siguientes opciones están disponibles para la configuración del equilibrio de carga:

- **Tamaño de muestra**. Identifica cuántas muestras de los sondeos de estado se deben tener en cuenta para la evaluación de estado de back-end.

- **Tamaño de muestra correcto**. Define el tamaño de muestra como se mencionó anteriormente, el número de muestras correctas necesario para llamar al back-end correcto. Por ejemplo, suponga que un intervalo de sondeo de estado de Front Door es de 30 segundos, el tamaño de muestra es 5 y el tamaño de muestra correcto es 3. Cada vez que evaluemos los sondeos de estado del back-end, nos centramos en las cinco últimas muestras sobre 150 segundos (5 x 30). Se requieren al menos tres sondeos correctos para declarar el back-end como correcto.

- **Sensibilidad de la latencia (latencia adicional)** . Define si desea que Front Door envíe la solicitud a los back-end dentro del intervalo de sensibilidad de medición de latencia o reenvíe la solicitud al back-end más cercano.

Para más información, consulte [Enrutamiento del tráfico basado en la latencia más baja](front-door-routing-methods.md#latency).

## <a name="next-steps"></a>Pasos siguientes

- [Crear un perfil de Front Door](quickstart-create-front-door.md)
- Información acerca de cómo [funciona Front Door](front-door-routing-architecture.md)