---
title: Scheduled Events para máquinas virtuales Linux en Azure
description: Programe los eventos usando Azure Metadata Service para las máquinas virtuales Linux.
author: EricRadzikowskiMSFT
ms.service: virtual-machines-linux
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 06/01/2020
ms.author: ericrad
ms.reviewer: mimckitt
ms.openlocfilehash: f91b5879922fc473ff1e46f817b3d649b1b30a9c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088740"
---
# <a name="azure-metadata-service-scheduled-events-for-linux-vms"></a>Azure Metadata Service: Scheduled Events para máquinas virtuales Linux

Scheduled Events es un servicio de Azure Metadata Service que proporciona el tiempo de aplicación para prepararse para el mantenimiento de la máquina virtual. Da información sobre los eventos de mantenimiento próximos (por ejemplo, un reinicio) para que la aplicación pueda prepararlos y así limitar las interrupciones. Está disponible para todos los tipos de máquina virtual de Azure, incluidos IaaS y PaaS, tanto en Windows como en Linux. 

Para más información acerca de Scheduled Events en Windows, consulte [Scheduled Events para máquinas virtuales Windows](../windows/scheduled-events.md).

> [!Note] 
> Scheduled Events está disponible con carácter general en todas las regiones de Azure. Consulte el apartado sobre la [disponibilidad por región y versión](#version-and-region-availability) para obtener información sobre la versión más reciente.

## <a name="why-use-scheduled-events"></a>¿Por qué usar Scheduled Events?

Muchas aplicaciones pueden aprovechar el tiempo para preparar el mantenimiento de las máquinas virtuales. Este tiempo se puede usar para realizar tareas específicas de la aplicación con el fin de mejorar la disponibilidad, la confiabilidad y la capacidad de servicio, lo que incluye: 

- Punto de control y restauración.
- Purga de la conexión.
- Conmutación por error de la réplica principal.
- Eliminación de un grupo de equilibradores de carga.
- Registros de eventos.
- Cierre estable.

Con Scheduled Events, la aplicación puede detectar cuándo se producirá el mantenimiento y desencadenar tareas para limitar sus efectos.  

Eventos programados proporciona eventos en los casos de uso siguientes:

- [Mantenimiento iniciado por la plataforma](../maintenance-and-updates.md?bc=/azure/virtual-machines/linux/breadcrumb/toc.json&toc=/azure/virtual-machines/linux/toc.json) (por ejemplo, reinicio de máquina virtual, migración en vivo o actualizaciones con conservación de memoria para el host)
- La máquina virtual funciona en un [hardware de host degradado](https://azure.microsoft.com/blog/find-out-when-your-virtual-machine-hardware-is-degraded-with-scheduled-events) que se predice que fallará pronto
- Mantenimiento iniciado por el usuario (por ejemplo, el usuario reinicia o vuelve a implementar una máquina virtual)
- Expulsiones de instancias de [máquina virtual de Spot](spot-vms.md) y [conjunto de escalado de Spot](../../virtual-machine-scale-sets/use-spot.md).

## <a name="the-basics"></a>Conceptos básicos  

  Metadata Service expone información sobre la ejecución de máquinas virtuales mediante un punto de conexión de REST accesible desde la propia máquina virtual. La información se encuentra disponible a través de una dirección IP no enrutable, de modo que no se expone fuera de la máquina virtual.

### <a name="scope"></a>Ámbito
Los eventos programados se entregan a:

- Máquinas virtuales independientes.
- Todas las máquinas virtuales en un servicio de nube.
- Todas las máquinas virtuales de un conjunto de disponibilidad.
- Todas las máquinas virtuales de una zona de disponibilidad. 
- Todas las máquinas virtuales de un grupo de selección de ubicación de conjunto de escalado. 

> [!NOTE]
> En cuanto a las máquinas virtuales de una zona de disponibilidad, los eventos programados van a las máquinas virtuales individuales de una zona.
> Por ejemplo, si tiene cien máquinas virtuales en un conjunto de disponibilidad y hay una actualización de una de ellas, el evento programado irá a todas esas cien máquinas, mientras que si hay cien máquinas virtuales únicas en una zona, el evento solo irá a la máquina virtual que se ve afectada.

Por ello, revise el campo `Resources` del evento para identificar cuáles son las máquinas virtuales que se verán afectadas.

### <a name="endpoint-discovery"></a>Detección de punto de conexión
En el caso de las máquinas virtuales con red virtual habilitada, el servicio de metadatos está disponible desde una dirección IP no enrutable estática, `169.254.169.254`. El punto de conexión completo de la versión más reciente de Scheduled Events es: 

 > `http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01`

Si la máquina virtual no se crea dentro de una red virtual (la opción predeterminada para servicios en la nube y máquinas virtuales clásicas), se necesita lógica adicional para detectar la dirección IP que se va a usar. Para aprender a [detectar el punto de conexión de host](https://github.com/azure-samples/virtual-machines-python-scheduled-events-discover-endpoint-for-non-vnet-vm), consulte este ejemplo.

### <a name="version-and-region-availability"></a>Disponibilidad por región y versión
El servicio Scheduled Events tiene versiones. Las versiones son obligatorias; la actual es `2019-01-01`.

| Versión | Tipo de versión | Regions | Notas de la versión | 
| - | - | - | - | 
| 2019-08-01 | Disponibilidad general | All | <li> Compatibilidad agregada con EventSource |
| 01-04-2019 | Disponibilidad general | All | <li> Compatibilidad agregada con la descripción de eventos |
| 2019-01-01 | Disponibilidad general | All | <li> Compatibilidad agregada con conjuntos de escalado de máquinas virtuales EventType "Terminate" |
| 01-11-2017 | Disponibilidad general | All | <li> Se agregó compatibilidad para la expulsión de la máquina virtual de Azure Spot EventType 'Preempt'<br> | 
| 2017-08-01 | Disponibilidad general | All | <li> Se quitó el guion bajo antepuesto de los nombres de recursos en las máquinas virtuales de IaaS<br><li>Se aplicó el requisito de encabezado de metadatos para todas las solicitudes | 
| 2017-03-01 | Versión preliminar | All | <li>Versión inicial |


> [!NOTE] 
> Versiones preliminares de Scheduled Events admitidas {más recientes} como la versión de API. Este formato ya no es compatible y dejará de utilizarse en el futuro.

### <a name="enabling-and-disabling-scheduled-events"></a>Habilitación y deshabilitación de Scheduled Events
Scheduled Events se habilita para un servicio la primera vez que se realiza una solicitud de eventos. Debe esperar hasta dos minutos de demora en la respuesta en la primera llamada.

Scheduled Events se deshabilita para el servicio si no se realiza una solicitud durante 24 horas.

### <a name="user-initiated-maintenance"></a>Mantenimiento iniciado por el usuario
El mantenimiento de las máquinas virtuales iniciado por el usuario a través de Azure Portal, API, CLI o PowerShell, da lugar a un evento programado. Esto permite probar la lógica de preparación del mantenimiento en la aplicación, además de que la aplicación pueda prepararse para el mantenimiento iniciado por el usuario.

Si reinicia una máquina virtual, se programa un evento del tipo `Reboot`. Si vuelve a implementar una máquina virtual, se programa un evento del tipo `Redeploy`.

## <a name="use-the-api"></a>Uso de la API

### <a name="headers"></a>encabezados
Al realizar consultas a Metadata Service, debe proporcionar el encabezado `Metadata:true` para asegurarse de que la solicitud no se haya redirigido de manera involuntaria. El encabezado `Metadata:true` es necesario para todas las solicitudes de eventos programados. Un error al incluir el encabezado en la solicitud generará una respuesta del tipo "Solicitud incorrecta" de Metadata Service.

### <a name="query-for-events"></a>Consulta de eventos
Puede consultar los eventos programados; para ello, simplemente haga la siguiente llamada:

#### <a name="bash"></a>Bash
```
curl -H Metadata:true http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01
```

Una respuesta contiene una matriz de eventos programados. Una matriz vacía significa que actualmente no hay eventos programados.
En caso de que haya eventos programados, la respuesta contiene una matriz de eventos. 
```
{
    "DocumentIncarnation": {IncarnationID},
    "Events": [
        {
            "EventId": {eventID},
            "EventType": "Reboot" | "Redeploy" | "Freeze" | "Preempt" | "Terminate",
            "ResourceType": "VirtualMachine",
            "Resources": [{resourceName}],
            "EventStatus": "Scheduled" | "Started",
            "NotBefore": {timeInUTC},       
            "Description": {eventDescription},
            "EventSource" : "Platform" | "User",
        }
    ]
}
```

### <a name="event-properties"></a>Propiedades de evento
|Propiedad  |  Descripción |
| - | - |
| EventId | Es un identificador único global del evento. <br><br> Ejemplo: <br><ul><li>602d9444-d2cd-49c7-8624-8643e7171297  |
| EventType | Es el impacto causado por el evento. <br><br> Valores: <br><ul><li> `Freeze`: la máquina virtual está programada para pausarse durante unos segundos. Puede que se suspenda la conectividad de la CPU y la red, pero no afecta a la memoria ni a los archivos abiertos.<li>`Reboot`: la máquina virtual está programada para reiniciarse (se borrará la memoria no persistente). <li>`Redeploy`: la máquina virtual está programada para moverse a otro nodo (los discos efímeros se pierden). <li>`Preempt`: se está eliminando la máquina virtual de Azure Spot (se pierden los discos efímeros). <li> `Terminate`: la máquina virtual está programada para eliminarse. |
| ResourceType | Es el tipo de recurso al que este evento afecta. <br><br> Valores: <ul><li>`VirtualMachine`|
| Recursos| Es la lista de recursos a los que este evento afecta. Se garantiza que contenga máquinas de un [dominio de actualización](manage-availability.md) como máximo, pero puede no contener todas las máquinas en dicho dominio. <br><br> Ejemplo: <br><ul><li> ["FrontEnd_IN_0", "BackEnd_IN_0"] |
| EventStatus | Es el estado de este evento. <br><br> Valores: <ul><li>`Scheduled`: este evento está programado para iniciarse después de la hora especificada en la propiedad `NotBefore`.<li>`Started`: este evento se ha iniciado.</ul> Ni `Completed` ni otro estado similar se han proporcionado antes. El evento ya no vuelve cuando finaliza el evento.
| NotBefore| Hora a partir de la que puede iniciarse este evento. <br><br> Ejemplo: <br><ul><li> Lunes, 19 de septiembre de 2016, 18:29:47 GMT  |
| Descripción | Descripción de este evento. <br><br> Ejemplo: <br><ul><li> El servidor host está en mantenimiento. |
| EventSource | Iniciador del evento. <br><br> Ejemplo: <br><ul><li> `Platform`: la plataforma inicia este evento. <li>`User`: el usuario inicia este evento. |

### <a name="event-scheduling"></a>Programación de eventos
Cada evento se programa una cantidad mínima de tiempo en el futuro en función de su tipo. Este tiempo se refleja en la propiedad `NotBefore` de un evento. 

|EventType  | Aviso mínimo |
| - | - |
| Freeze| 15 minutos |
| Reboot | 15 minutos |
| Volver a implementar | 10 minutos |
| Preempt | 30 segundos |
| Terminate | [Configurable por el usuario](../../virtual-machine-scale-sets/virtual-machine-scale-sets-terminate-notification.md#enable-terminate-notifications): de 5 a 15 minutos |

> [!NOTE] 
> En algunos casos, Azure puede predecir errores en el host debidos a que el hardware está degradado e intentará mitigar la interrupción del servicio mediante la programación de una migración. Las máquinas virtuales afectadas recibirán un evento programado con un valor de `NotBefore` que habitualmente es unos días posteriores. El tiempo real varía en función de la valoración de riesgo de error predicha. Azure intenta avisar con 7 días de antelación siempre que sea posible, pero el tiempo real varía y puede ser menor si la predicción es que sea muy probable que se produzcan errores en el hardware de forma inminente. Para minimizar el riesgo para el servicio si se produce un error en el hardware antes de la migración iniciada por el sistema, se recomienda volver a implementar automáticamente la máquina virtual lo antes posible.

### <a name="start-an-event"></a>Inicio de un evento 

Cuando se haya enterado de un evento próximo y completado la lógica para llevar a cabo un apagado estable, puede aprobar el evento pendiente mediante una llamada de `POST` a Metadata Service con `EventId`. Esta llamada indica a Azure que puede acortar el tiempo de notificación mínimo (cuando sea posible). 

A continuación se muestra el JSON de ejemplo en el cuerpo de la solicitud `POST`. La solicitud debe contener una lista de `StartRequests`. Cada `StartRequest` contiene el elemento `EventId` para el evento que desea acelerar:
```
{
    "StartRequests" : [
        {
            "EventId": {EventId}
        }
    ]
}
```

#### <a name="bash-sample"></a>Ejemplo de Bash
```
curl -H Metadata:true -X POST -d '{"StartRequests": [{"EventId": "f020ba2e-3bc0-4c40-a10b-86575a9eabd5"}]}' http://169.254.169.254/metadata/scheduledevents?api-version=2019-01-01
```

> [!NOTE] 
> Reconocer un evento permite a este continuar para todos sus elementos `Resources`, no solo para la máquina virtual que lo reconoce. Por tanto, puede optar por elegir un líder para que coordine el reconocimiento. Este puede ser tan sencillo como la primera máquina del campo `Resources`.

## <a name="python-sample"></a>Ejemplo de Python 

En el siguiente ejemplo se realiza una consulta a Metadata Service para eventos programados y se aprueban todos los eventos pendientes:

```python
#!/usr/bin/python

import json
import socket
import urllib2

metadata_url = "http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01"
this_host = socket.gethostname()


def get_scheduled_events():
    req = urllib2.Request(metadata_url)
    req.add_header('Metadata', 'true')
    resp = urllib2.urlopen(req)
    data = json.loads(resp.read())
    return data


def handle_scheduled_events(data):
    for evt in data['Events']:
        eventid = evt['EventId']
        status = evt['EventStatus']
        resources = evt['Resources']
        eventtype = evt['EventType']
        resourcetype = evt['ResourceType']
        notbefore = evt['NotBefore'].replace(" ", "_")
    description = evt['Description']
    eventSource = evt['EventSource']
        if this_host in resources:
            print("+ Scheduled Event. This host " + this_host +
                " is scheduled for " + eventtype + 
        " by " + eventSource + 
        " with description " + description +
        " not before " + notbefore)
            # Add logic for handling events here


def main():
    data = get_scheduled_events()
    handle_scheduled_events(data)


if __name__ == '__main__':
    main()
```

## <a name="next-steps"></a>Pasos siguientes 
- Mire [Scheduled Events on Azure Friday](https://channel9.msdn.com/Shows/Azure-Friday/Using-Azure-Scheduled-Events-to-Prepare-for-VM-Maintenance) (Scheduled Events en Azure Friday) para ver una demostración. 
- Repase los ejemplos de código de Scheduled Events en el [repositorio de GitHub Azure Instance Metadata Scheduled Events](https://github.com/Azure-Samples/virtual-machines-scheduled-events-discover-endpoint-for-non-vnet-vm) (Scheduled Events de Azure Instance Metadata).
- En [Instance Metadata Service](instance-metadata-service.md), puede obtener más información sobre las API disponibles.
- Obtenga información sobre cómo realizar [el mantenimiento planeado para máquinas virtuales Linux en Azure](../maintenance-and-updates.md?bc=/azure/virtual-machines/linux/breadcrumb/toc.json&toc=/azure/virtual-machines/linux/toc.json).
