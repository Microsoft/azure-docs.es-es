---
title: 'Conectividad de servicio de la solución Monitor de rendimiento de red: Azure Log Analytics'
description: Use la funcionalidad del Monitor de conectividad de servicio en Network Performance Monitor para supervisar la conectividad de red con cualquier punto de conexión que tenga un puerto TCP abierto.
ms.subservice: logs
ms.topic: conceptual
author: abshamsft
ms.author: absha
ms.date: 02/20/2018
ms.openlocfilehash: dd8dcbe136558e997ca16a2d9754cbf1c4e7620b
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326110"
---
# <a name="service-connectivity-monitor"></a>Monitor de conectividad de servicio

Puede usar la funcionalidad Monitor de conectividad de servicio en [Network Performance Monitor](network-performance-monitor.md) para supervisar la conectividad de red con cualquier punto de conexión que tenga un puerto TCP abierto. Dichos puntos de conexión incluyen sitios web, aplicaciones SaaS, aplicaciones PaaS y bases de datos SQL. 

Puede realizar las funciones siguientes con el Monitor de conectividad de servicio: 

- Supervisar la conectividad de red con sus aplicaciones y servicios de red desde varias sucursales o ubicaciones. Entre las aplicaciones y servicios de red se incluyen Office 365, Dynamics CRM, aplicaciones de línea de negocio internas y bases de datos SQL.
- Usar pruebas integradas para supervisar la conectividad de red con puntos de conexión de Office365 y Dynamics365. 
- Determinar el tiempo de respuesta, la latencia de la red y la pérdida de paquetes al conectarse con el punto de conexión.
- Determinar si el rendimiento de una aplicación es deficiente debido a la red o porque hay algún problema en el extremo del proveedor de la aplicación.
- Identificar las zonas activas de la red que podrían estar generando un rendimiento deficiente de la aplicación mediante la visualización de la latencia aportada por cada salto en un mapa de topología.


![Monitor de conectividad de servicio](media/network-performance-monitor-service-endpoint/service-endpoint-intro.png)


## <a name="configuration"></a>Configuración 
Para abrir la configuración de Network Performance Monitor, abra la [solución de Network Performance Monitor](network-performance-monitor.md) y seleccione **Configurar**.

![Configuración de Network Performance Monitor](media/network-performance-monitor-service-endpoint/npm-configure-button.png)


### <a name="configure-log-analytics-agents-for-monitoring"></a>Configurar agentes de Log Analytics para la supervisión
Habilite las reglas de firewall siguientes en los nodos que se usan para la supervisión de manera que la solución pueda detectar la topología desde los nodos al punto de conexión de servicio: 

```
netsh advfirewall firewall add rule name="NPMDICMPV4Echo" protocol="icmpv4:8,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV6Echo" protocol="icmpv6:128,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV4DestinationUnreachable" protocol="icmpv4:3,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV6DestinationUnreachable" protocol="icmpv6:1,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV4TimeExceeded" protocol="icmpv4:11,any" dir=in action=allow 
netsh advfirewall firewall add rule name="NPMDICMPV6TimeExceeded" protocol="icmpv6:3,any" dir=in action=allow 
```

### <a name="create-service-connectivity-monitor-tests"></a>Creación de pruebas del Monitor de conectividad de servicio 

Comience a crear las pruebas para supervisar la conectividad de red con los puntos de conexión de servicio.

1. Seleccione la pestaña **Monitor de conectividad de servicio**.
2. Seleccione **Agregar prueba** y escriba el nombre y la descripción de la prueba. Puede crear un máximo de 450 pruebas por área de trabajo. 
3. Seleccione el tipo de prueba:<br>

    * Seleccione **Web** para supervisar la conectividad con un servicio que responde a las solicitudes HTTP/S, como outlook.office365.com o bing.com.<br>
    * Seleccione **Red** para supervisar la conectividad con un servicio que responde a solicitudes TCP pero no a solicitudes HTTP/S, como SQL Server, servidor FTP o puerto SSH. 
    * Por ejemplo: Para crear una prueba web con una cuenta de Blob Storage, seleccione **Web** y especifique el destino como *yourstorageaccount*. blob.core.windows.net. De forma similar, puede crear pruebas para Table Storage, Queue Storage y Azure Files mediante [este vínculo](../../storage/common/storage-account-overview.md#storage-account-endpoints).
4. Si no desea realizar mediciones de red, como la latencia de la red, la pérdida de paquetes y la detección de topología, desactive la casilla **Realizar mediciones de red**. Manténgala activada para aprovechar al máximo la funcionalidad. 
5. En **Destino**, escriba la dirección IP, el FQDN o la dirección URL en que desea supervisar la conectividad de red.
6. En **Número de puerto**, escriba el número de puerto del servicio de destino. 
7. En **Frecuencia de prueba**, escriba un valor para la frecuencia con la que desea que se ejecute la prueba. 
8. Seleccione los nodos desde los cuales quiere supervisar la conectividad de red con el servicio. Asegúrese de que el número de agentes agregados por prueba es menor que 150. Los agentes solo pueden probar 150 puntos de conexión/agentes como máximo.

    >[!NOTE]
    > En el caso de los nodos basados en Windows Server, la funcionalidad usa solicitudes basadas en TCP para hacer las medidas en la red. En el caso de los nodos basados en el cliente Windows, la funcionalidad usa solicitudes basadas en ICMP para hacer las medidas en la red. En algunos casos, la aplicación de destino bloquea las solicitudes basadas en ICMP entrantes cuando los nodos se basan en clientes de Windows. La solución no puede realizar mediciones de red. Recomendamos usar los nodos basados en Windows Server en esos casos. 

9. Si no quiere crear eventos de estado para los elementos seleccionados, desactive la opción **Habilitar el seguimiento de estado en los destinos que abarca esta prueba**. 
10. Elija las condiciones de supervisión. Puede establecer umbrales personalizados para la generación de eventos de estado escribiendo valores de umbral. Siempre que el valor de la condición supere el umbral seleccionado para el par de red/subred seleccionado, se generará un evento de estado. 
11. Para guardar la configuración, seleccione **Guardar**. 

    ![Configuraciones de prueba del Monitor de conectividad de servicio](media/network-performance-monitor-service-endpoint/service-endpoint-configuration.png)



## <a name="walkthrough"></a>Tutorial 

Vaya a la vista del panel Network Performance Monitor. Para ver un resumen del estado de las distintas pruebas que ha creado, consulte la página **Monitor de conectividad de servicio**. 

![Página Monitor de conectividad de servicio](media/network-performance-monitor-service-endpoint/service-endpoint-blade.png)

Seleccione el icono para ver los detalles de las pruebas en la página **Pruebas**. En la tabla de la izquierda, puede ver el estado en un momento dado y el valor del tiempo de respuesta del servicio, la latencia de red y la pérdida de paquetes en todas las pruebas. Use el control Grabadora de estado de la red para ver una instantánea de la red en otro momento anterior. Seleccione la prueba de la tabla que desea investigar. En los gráficos del panel de la derecha, puede ver la tendencia histórica de los valores de pérdida, latencia y tiempo de respuesta. Seleccione el vínculo **Detalles de la prueba** para ver el rendimiento de cada nodo.

![Pruebas del Monitor de conectividad de servicio](media/network-performance-monitor-service-endpoint/service-endpoint-tests.png)

En la vista **Nodos de prueba**, puede observar la conectividad de red desde cada nodo. Seleccione el nodo con degradación del rendimiento. Este es el nodo donde se observa que la aplicación se ejecuta lentamente.

Determine si el rendimiento de una aplicación es deficiente debido a la red o a un problema en el extremo del proveedor de la aplicación mediante la observación de la correlación entre el tiempo de respuesta de la aplicación y la latencia de red. 

* **Problema de la aplicación:** un pico en el tiempo de respuesta, pero coherencia en la latencia de red sugiere que la red funciona correctamente y el problema se debe a un error en el extremo de la aplicación. 

    ![Problema de aplicación del Monitor de conectividad de servicio](media/network-performance-monitor-service-endpoint/service-endpoint-application-issue.png)

* **Problema de red:** un pico en el tiempo de respuesta junto con un pico correspondiente en la latencia de la red sugiere que el aumento en el tiempo de respuesta podría deberse a un aumento en la latencia de la red. 

    ![Problema de red del Monitor de conectividad de servicio](media/network-performance-monitor-service-endpoint/service-endpoint-network-issue.png)

Una vez que determina que el problema se debe a la red, seleccione el vínculo de vista **Topología** para identificar el salto problemático en el mapa de topología. Un ejemplo se muestra en la imagen siguiente. De una latencia total de 105 ms entre el nodo y el punto de conexión de la aplicación, 96 ms se debe al salto marcado en rojo. Una vez que identifica el salto problemático, puede realizar alguna acción correctiva. 

![Pruebas del Monitor de conectividad de servicio](media/network-performance-monitor-service-endpoint/service-endpoint-topology.png)

## <a name="diagnostics"></a>Diagnóstico 

Si observa un comportamiento anormal, siga estos pasos:

* Si el tiempo de respuesta del servicio, la pérdida de paquetes y la latencia aparecen como N/D, puede deberse a una o varias de las razones siguientes:

    - La aplicación está inactiva.
    - El nodo que se usa para comprobar la conectividad de red con el servicio está inactivo.
    - El destino que se escribió en la configuración de prueba no es correcto.
    - El nodo no tiene conectividad de red ninguna.

* Si aparece un tiempo de respuesta del servicio válido, pero tanto la pérdida de red como la latencia aparecen como N/D, puede deberse a una o varias de las razones siguientes:

    - Si el nodo que se usa para comprobar la conectividad de red con el servicio es un equipo cliente Windows, independientemente de si el servicio de destino bloquea las solicitudes ICMP o un firewall de red bloquea las solicitudes ICMP que se originan del nodo.
    - La casilla **Realizar mediciones de red** está en blanco en la configuración de prueba. 

* Si el tiempo de respuesta del servicio es N/D, pero tanto la pérdida de red como la latencia son válidas, el servicio de destino podría no ser una aplicación web. Edite la configuración de prueba y elija el tipo de prueba como **Red** en lugar de **Web**. 

* Si la aplicación se ejecuta demasiado lento, determine si el rendimiento de una aplicación es deficiente debido a la red o a un problema en el extremo del proveedor de la aplicación.

## <a name="gcc-office-urls-for-us-government-customers"></a>Direcciones URL de Office para GCC para los clientes del US Gov
En la región de Virginia del US Gov, solo las direcciones de DOD están incorporadas en el NPM. Los clientes que usen las direcciones URL de GCC deben crear pruebas personalizadas y agregar individualmente cada dirección URL.

| Campo | GCC |
|:---   |:--- |
| Portal de Office 365 y recursos compartidos | portal.apps.mil |
| Identidad y autenticación de Office 365 | * login.microsoftonline.us <br> * api.login.microsoftonline.com <br> * clientconfig.microsoftonline-p.net <br> * login.microsoftonline.com <br> * login.microsoftonline-p.com <br> * login.windows.net <br> * loginex.microsoftonline.com <br> * login-us.microsoftonline.com <br> * nexus.microsoftonline-p.com <br> * mscrl.microsoft.com <br> * secure.aadcdn.microsoftonline-p.com |
| Office en línea | * adminwebservice.gov.us.microsoftonline.com <br>  * adminwebservice-s1-bn1a.microsoftonline.com <br> * adminwebservice-s1-dm2a.microsoftonline.com <br> * becws.gov.us.microsoftonline.com <br> * provisioningapi.gov.us.microsoftonline.com <br> * officehome.msocdn.us <br> * prod.msocdn.us <br> * portal.office365.us <br> * webshell.suite.office365.us <br> * www\.office365.us <br> * activation.sls.microsoft.com <br> * crl.microsoft.com <br> * go.microsoft.com <br> * insertmedia.bing.office.net <br> * ocsa.officeapps.live.com <br> * ocsredir.officeapps.live.com <br> * ocws.officeapps.live.com <br> * office15client.microsoft.com <br>* officecdn.microsoft.com <br> * officecdn.microsoft.com.edgesuite.net <br> * officepreviewredir.microsoft.com <br> * officeredir.microsoft.com <br> * ols.officeapps.live.com  <br> * r.office.microsoft.com <br> * cdn.odc.officeapps.live.com <br> * odc.officeapps.live.com <br> * officeclient.microsoft.com |
| Exchange Online | * outlook.office365.us <br> * attachments.office365-net.us <br> * autodiscover-s.office365.us <br> * manage.office365.us <br> * scc.office365.us |
| Equipos de MS | gov.teams.microsoft.us | 

## <a name="next-steps"></a>Pasos siguientes
[Buscar registros](../log-query/log-query-overview.md) para ver datos detallados de rendimiento de red.

