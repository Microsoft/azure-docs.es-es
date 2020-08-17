---
title: Bloqueo del tráfico de salida
description: Obtenga información sobre la integración con Azure Firewall para proteger el tráfico saliente desde dentro de un entorno de App Service.
author: ccompy
ms.assetid: 955a4d84-94ca-418d-aa79-b57a5eb8cb85
ms.topic: article
ms.date: 07/13/2020
ms.author: ccompy
ms.custom: seodec18, references_regions
ms.openlocfilehash: 9407e388a44ec78fa1e089542fc5ea3ef9387d60
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073758"
---
# <a name="locking-down-an-app-service-environment"></a>Bloqueo de una instancia de App Service Environment

App Service aislado tiene varias dependencias externas a las que requiere acceso para que funcionen correctamente. El plan de App Service aislado reside en la red de Azure Virtual Network (VNet) del cliente. Los clientes deben permitir el tráfico de dependencia de App Service aislado, que es un problema para aquellos que desean bloquear todo el tráfico de salida de su red virtual.

Hay una serie de puntos de conexión de entrada que se usan para administrar una instancia de ASE. El tráfico entrante de administración no se puede enviar a través de un dispositivo de firewall. Las direcciones de origen de este tráfico se conocen y están publicadas en el documento [Direcciones de administración de App Service aislado](https://docs.microsoft.com/azure/app-service/environment/management-addresses). También hay una etiqueta de servicio denominada AppServiceManagement que se puede usar con grupos de seguridad de red (NSG) para proteger el tráfico entrante.

Las dependencias de salida de App Service aislado se definen casi por completo con los FQDN, que no tienen direcciones estáticas tras ellos. La falta de direcciones estáticas significa que no se pueden usar grupos de seguridad de red para bloquear el tráfico saliente de una instancia de ASE. Las direcciones cambian con tal frecuencia que no se pueden configurar reglas en función de la resolución actual ni usarlas para crear grupos de seguridad de red. 

La solución para proteger las direcciones de salida se encuentra en el uso de un dispositivo de firewall que puede controlar el tráfico saliente basándose en los nombres de dominio. Azure Firewall puede restringir el tráfico saliente HTTP y HTTPS basándose en el FQDN de destino.  

## <a name="system-architecture"></a>Arquitectura del sistema

Para implementar un ASE con tráfico saliente que pasa por un dispositivo de firewall, es necesario cambiar las rutas en la subred de ASE. Las rutas funcionan en el nivel de IP. Si no tiene cuidado al definir las rutas, puede forzar a que el tráfico de respuesta TCP salga de otra dirección. Cuando la dirección de respuesta es diferente de la dirección a la que se envió el tráfico, el problema se denomina enrutamiento asimétrico e interrumpirá el TCP.

Para que el tráfico que entrad al ASE pueda responder de la misma forma en que se originó, debe haber rutas definidas. Las rutas se deben definir para las solicitudes de administración de entrada y las solicitudes de aplicación de entrada.

El tráfico hacia y desde una instancia de ASE debe cumplir las convenciones siguientes:

* El tráfico a Azure SQL, Storage y el centro de eventos no se admite con el uso de un dispositivo de firewall. Este tráfico debe enviarse directamente a esos servicios. Para ello, es necesario configurar los puntos de conexión de servicio para esos tres servicios. 
* Se deben definir reglas de tabla de rutas que envíen el tráfico de administración de vuelta al lugar donde se originó.
* Se deben definir reglas de tabla de rutas que envíen el tráfico de aplicaciones de vuelta al lugar donde se originó. 
* El resto del tráfico que sale de ASE se puede enviar al dispositivo de firewall con una regla de la tabla de rutas.

![ASE con flujo de conexión de Azure Firewall][5]

## <a name="locking-down-inbound-management-traffic"></a>Bloqueo del tráfico de administración de entrada

Si la subred de ASE todavía no tiene un grupo de seguridad de red asignado, cree uno. En el grupo de seguridad de red, establezca la primera regla para permitir el tráfico desde la etiqueta de servicio llamada AppServiceManagement en los puertos 454 y 455. La regla para permitir el acceso desde la etiqueta AppServiceManagement es lo único que las direcciones IP públicas necesitan para administrar la instancia de ASE. Las direcciones que están detrás de esa etiqueta de servicio solo se usan para administrar Azure App Service. El tráfico de administración que fluye a través de estas conexiones se cifra y protege con certificados de autenticación. El tráfico habitual en este canal incluye elementos como los comandos iniciados por el cliente y los sondeos de estado. 

Los ASE que se crean mediante el portal con una nueva subred lo hacen con un grupo de seguridad de red que contiene la regla de permiso para la etiqueta AppServiceManagement.  

La instancia de ASE también debe permitir las solicitudes entrantes procedentes de la etiqueta Load Balancer en el puerto 16001. Las solicitudes procedentes de Load Balancer en el puerto 16001 son comprobaciones de conexiones persistentes entre Load Balancer y los servidores front-end de ASE. Si el puerto 16001 está bloqueado, la instancia de ASE adoptará un estado incorrecto.

## <a name="configuring-azure-firewall-with-your-ase"></a>Configuración de Azure Firewall con App Service aislado 

A continuación, se indican los pasos para bloquear la salida de la instancia existente de App Service Environment con Azure Firewall:

1. Habilite puntos de conexión de servicio para SQL, Event Hub y Storage en la subred de ASE. Para habilitar los puntos de conexión de servicio, vaya al portal de red > subredes y seleccione Microsoft.EventHub, Microsoft.SQL y Microsoft.Storage en la lista desplegable de puntos de conexión de servicio. Cuando tenga puntos de conexión de servicio habilitados para Azure SQL, todas las dependencias de Azure SQL que tengan sus aplicaciones deben configurarse también como puntos de conexión de servicio. 

   ![selección de puntos de conexión de servicio][2]
  
1. Cree una subred denominada AzureFirewallSubnet en la red virtual donde exista su ASE. Siga las instrucciones de la [documentación de Azure Firewall](https://docs.microsoft.com/azure/firewall/) para crear su instancia de Azure Firewall.

1. En la interfaz de usuario de Azure Firewall, vaya a Reglas > Recopilación de reglas de aplicación y seleccione Agregar una colección de reglas de aplicación. Proporcione un nombre y una prioridad y establezca Permitir. En la sección de etiquetas FQDN, proporcione un nombre, establezca las direcciones de origen en * y seleccione la etiqueta FQDN de App Service Environment y la actualización de Windows. 
   
   ![Adición de regla de aplicación][1]
   
1. En la interfaz de usuario de Azure Firewall, vaya a Reglas > Recopilación de reglas de aplicación y seleccione Agregar una colección de reglas de aplicación. Proporcione un nombre y una prioridad y establezca Permitir. En la sección de reglas, en Direcciones IP, proporcione un nombre, seleccione un protocolo **Cualquiera**, establezca * como direcciones de origen y de destino y defina los puertos como 123. Esta regla permite que el sistema realice la sincronización del reloj mediante NTP. Cree otra regla de la misma manera para el puerto 12000 para ayudar a evaluar los problemas del sistema. 

   ![Adición de regla de red NTP][3]
   
1. En la interfaz de usuario de Azure Firewall, vaya a Reglas > Recopilación de reglas de aplicación y seleccione Agregar una colección de reglas de aplicación. Proporcione un nombre y una prioridad y establezca Permitir. En la sección Reglas de Etiquetas de servicio, proporcione un nombre, seleccione un protocolo **Cualquiera**, establezca * como las direcciones de origen, seleccione una etiqueta de servicio de AzureMonitor y configure los puertos como 80 y 443. Esta regla permite que el sistema suministre a Azure Monitor información de estado y métricas.

   ![Adición de una regla de red de etiqueta de servicio NTP][6]
   
1. Cree una tabla de rutas con las direcciones de administración de [Direcciones de administración de App Service aislado]( https://docs.microsoft.com/azure/app-service/environment/management-addresses) con un próximo salto de Internet. Las entradas de la tabla de rutas son necesarias para evitar problemas de enrutamiento asimétrico. Agregue rutas a las dependencias de dirección IP que se indican a continuación en las dependencias de dirección IP con un próximo salto de Internet. Agregue una ruta de aplicación virtual a la tabla de rutas para 0.0.0.0/0 con el próximo salto como dirección IP privada de Azure Firewall. 

   ![Creación de una tabla de rutas][4]
   
1. Asigne la tabla de rutas que creó a la subred de ASE.

#### <a name="deploying-your-ase-behind-a-firewall"></a>Implementación de ASE tras un firewall

Los pasos para implementar su ASE tras un firewall son iguales que los pasos para configurar el ASE existente con una instancia de Azure Firewall, con la salvedad de que necesitará crear una subred de ASE y, a continuación, seguir los pasos anteriores. Para crear su ASE en una subred existente, deberá usar una plantilla de Resource Manager, como se describe en el documento sobre cómo [crear su ASE con una plantilla de Resource Manager](https://docs.microsoft.com/azure/app-service/environment/create-from-template).

## <a name="application-traffic"></a>Tráfico de la aplicación 

Los pasos anteriores permitirán que la instancia de App Service aislado funcione sin problemas. Deberá configurar todo para adaptarse a las necesidades de la aplicación. Hay dos problemas para las aplicaciones de una instancia de App Service aislado configurada con Azure Firewall.  

- Las dependencias de la aplicación deben agregarse a Azure Firewall o a la tabla de rutas. 
- Se deben crear rutas para el tráfico de la aplicación para evitar problemas de enrutamiento asimétrico.

Si las aplicaciones tienen dependencias, deben agregarse a Azure Firewall. Crear reglas de aplicación para permitir el tráfico HTTP/HTTPS y reglas de red para todo lo demás. 

Si conoce el intervalo de direcciones del que provendrá el tráfico de la solicitud de la aplicación, puede agregarlo a la tabla de rutas asignada a la subred de App Service aislado. Si el intervalo de direcciones es grande o no se especifica, puede usar un dispositivo de red, como Application Gateway, para proporcionarle una dirección para agregar a la tabla de rutas. Para obtener más información sobre cómo configurar una instancia de Application Gateway con su instancia de App Service aislado con ILB, lea [Integración de App Service aislado con ILB con Azure Application Gateway](https://docs.microsoft.com/azure/app-service/environment/integrate-with-application-gateway)

Este uso de Application Gateway es solo un ejemplo de cómo configurar el sistema. Si ha seguido este procedimiento, deberá agregar una ruta a la tabla de rutas de subred de ASE para que el tráfico de respuesta enviado a Application Gateway vaya allí directamente. 

## <a name="logging"></a>Registro 

Azure Firewall puede enviar registros a Azure Storage, Event Hub o a registros de Azure Monitor. Para integrar la aplicación con cualquier destino admitido, vaya al portal de Azure Firewall > Registros de diagnóstico y habilite los registros para el destino deseado. Si realiza la integración con los registros de Azure Monitor, podrá ver el registro de todo el tráfico enviado a Azure Firewall. Para ver el tráfico que se va a denegar, abra el portal del área de trabajo de Log Analytics > Registros y escriba una consulta como: 

```kusto
AzureDiagnostics | where msg_s contains "Deny" | where TimeGenerated >= ago(1h)
```

La integración de Azure Firewall con los registros de Azure Monitor resulta útil la primera vez que se pone una aplicación en funcionamiento, cuando aún no se conocen todas las dependencias de la aplicación. Puede obtener más información acerca de los registros de Azure Monitor en [Análisis de datos de registro en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/log-query-overview).
 
## <a name="dependencies"></a>Dependencias

La siguiente información solo es necesaria si desea configurar un dispositivo de firewall distinto a Azure Firewall. 

- Los servicios compatibles con puntos de conexión de servicio deben configurarse con puntos de conexión de servicio.
- Las dependencias de dirección IP son para tráfico que no sea HTTP/HTTPS (tráfico TCP y UDP).
- Los puntos de conexión HTTP/HTTPS de FQDN se pueden colocar en el dispositivo de firewall.
- Los puntos de conexión HTTP/HTTPS de carácter comodín son dependencias que pueden variar con App Service aislado en función del número de calificadores. 
- Las dependencias de Linux solo son un problema si va a implementar aplicaciones de Linux en App Service aislado. No es necesario agregar estas direcciones al firewall si no va a implementar aplicaciones de Linux en App Service aislado. 

#### <a name="service-endpoint-capable-dependencies"></a>Dependencias compatibles con los puntos de conexión de servicio 

| Punto de conexión |
|----------|
| Azure SQL |
| Azure Storage |
| Centro de eventos de Azure |

#### <a name="ip-address-dependencies"></a>Dependencias de dirección IP

| Punto de conexión | Detalles |
|----------| ----- |
| \*:123 | Comprobación de reloj NTP. El tráfico se comprueba en varios puntos de conexión en el puerto 123. |
| \*:12000 | Este puerto se usa para algunas tareas de supervisión del sistema. Si se bloquea, algunos problemas serán más difíciles de evaluar, pero el ASE seguirá funcionando. |
| 40.77.24.27:80 | Necesario para supervisar y alertar sobre problemas del ASE. |
| 40.77.24.27:443 | Necesario para supervisar y alertar sobre problemas del ASE. |
| 13.90.249.229:80 | Necesario para supervisar y alertar sobre problemas del ASE. |
| 13.90.249.229:443 | Necesario para supervisar y alertar sobre problemas del ASE. |
| 104.45.230.69:80 | Necesario para supervisar y alertar sobre problemas del ASE. |
| 104.45.230.69:443 | Necesario para supervisar y alertar sobre problemas del ASE. |
| 13.82.184.151:80 | Necesario para supervisar y alertar sobre problemas del ASE. |
| 13.82.184.151:443 | Necesario para supervisar y alertar sobre problemas del ASE. |

Con una instancia de Azure Firewall, se configurará automáticamente todo lo siguiente con etiquetas FQDN. 

#### <a name="fqdn-httphttps-dependencies"></a>Dependencias HTTP/HTTPS de FQDN 

| Punto de conexión |
|----------|
|graph.microsoft.com:443 |
|login.live.com:443 |
|login.windows.com:443 |
|login.windows.net:443 |
|login.microsoftonline.com:443 |
|client.wns.windows.com:443 |
|definitionupdates.microsoft.com:443 |
|go.microsoft.com:80 |
|go.microsoft.com:443 |
|www.microsoft.com:80 |
|www.microsoft.com:443 |
|wdcpalt.microsoft.com:443 |
|wdcp.microsoft.com:443 |
|ocsp.msocsp.com:443 |
|mscrl.microsoft.com:443 |
|mscrl.microsoft.com:80 |
|crl.microsoft.com:443 |
|crl.microsoft.com:80 |
|www.thawte.com:443 |
|crl3.digicert.com:80 |
|ocsp.digicert.com:80 |
|csc3-2009-2.crl.verisign.com:80 |
|crl.verisign.com:80 |
|ocsp.verisign.com:80 |
|cacerts.digicert.com:80 |
|azperfcounters1.blob.core.windows.net:443 |
|azurewatsonanalysis-prod.core.windows.net:443 |
|global.metrics.nsatc.net:80 |
|global.metrics.nsatc.net:443 |
|az-prod.metrics.nsatc.net:443 |
|antares.metrics.nsatc.net:443 |
|azglobal black.azglobal.metrics.nsatc.net:443 |
|azglobal-red.azglobal.metrics.nsatc.net:443 |
|antares-black.antares.metrics.nsatc.net:443 |
|antares-red.antares.metrics.nsatc.net:443 |
|prod.microsoftmetrics.com:443 |
|maupdateaccount.blob.core.windows.net:443 |
|clientconfig.passport.net:443 |
|packages.microsoft.com:443 |
|schemas.microsoft.com:80 |
|schemas.microsoft.com:443 |
|management.core.windows.net:443 |
|management.core.windows.net:80 |
|management.azure.com:443 |
|www.msftconnecttest.com:80 |
|shavamanifestcdnprod1.azureedge.net:443 |
|validation-v2.sls.microsoft.com:443 |
|flighting.cp.wd.microsoft.com:443 |
|dmd.metaservices.microsoft.com:80 |
|admin.core.windows.net:443 |
|prod.warmpath.msftcloudes.com:443 |
|prod.warmpath.msftcloudes.com:80 |
|gcs.prod.monitoring.core.windows.net:80|
|gcs.prod.monitoring.core.windows.net:443|
|azureprofileruploads.blob.core.windows.net:443 |
|azureprofileruploads2.blob.core.windows.net:443 |
|azureprofileruploads3.blob.core.windows.net:443 |
|azureprofileruploads4.blob.core.windows.net:443 |
|azureprofileruploads5.blob.core.windows.net:443 |
|azperfmerges.blob.core.windows.net:443 |
|azprofileruploads1.blob.core.windows.net:443 |
|azprofileruploads10.blob.core.windows.net:443 |
|azprofileruploads2.blob.core.windows.net:443 |
|azprofileruploads3.blob.core.windows.net:443 |
|azprofileruploads4.blob.core.windows.net:443 |
|azprofileruploads6.blob.core.windows.net:443 |
|azprofileruploads7.blob.core.windows.net:443 |
|azprofileruploads8.blob.core.windows.net:443 | 
|azprofileruploads9.blob.core.windows.net:443 |
|azureprofilerfrontdoor.cloudapp.net:443 |
|settings-win.data.microsoft.com:443 |
|maupdateaccount2.blob.core.windows.net:443 |
|maupdateaccount3.blob.core.windows.net:443 |
|dc.services.visualstudio.com:443 |
|gmstorageprodsn1.blob.core.windows.net:443 |
|gmstorageprodsn1.file.core.windows.net:443 |
|gmstorageprodsn1.queue.core.windows.net:443 |
|gmstorageprodsn1.table.core.windows.net:443 |
|rteventservice.trafficmanager.net:443 |
|ctldl.windowsupdate.com:80 |
|ctldl.windowsupdate.com:443 |

#### <a name="wildcard-httphttps-dependencies"></a>Dependencias HTTP/HTTPS de carácter comodín 

| Punto de conexión |
|----------|
|gr-Prod-\*.cloudapp.net:443 |
| \*.management.azure.com:443 |
| \*.update.microsoft.com:443 |
| \*.windowsupdate.microsoft.com:443 |
| \*.identity.azure.net:443 |
| \*.ctldl.windowsupdate.com:80 |
| \*.ctldl.windowsupdate.com:443 |

#### <a name="linux-dependencies"></a>Dependencias de Linux 

| Punto de conexión |
|----------|
|wawsinfraprodbay063.blob.core.windows.net:443 |
|registry-1.docker.io:443 |
|auth.docker.io:443 |
|production.cloudflare.docker.com:443 |
|download.docker.com:443 |
|us.archive.ubuntu.com:80 |
|download.mono-project.com:80 |
|packages.treasuredata.com:80|
|security.ubuntu.com:80 |
|oryx-cdn.microsoft.io:443 |
| \*.cdn.mscr.io:443 |
|mcr.microsoft.com:443 |
|\*.data.mcr.microsoft.com:443 |
|packages.fluentbit.io:80 |
|packages.fluentbit.io:443 |
|apt-mo.trafficmanager.net:80 |
|apt-mo.trafficmanager.net:443 |
|azure.archive.ubuntu.com:80 |
|azure.archive.ubuntu.com:443 |
|changelogs.ubuntu.com:80 |
|13.74.252.37:11371 |
|13.75.127.55:11371 |
|13.76.190.189:11371 |
|13.80.10.205:11371 |
|13.91.48.226:11371 |
|40.76.35.62:11371 |
|104.215.95.108:11371 |

## <a name="us-gov-dependencies"></a>Dependencias de US Gov

En el caso de ASE de regiones Gov (US), siga las instrucciones de la sección [Configuración de Azure Firewall con el ASE](https://docs.microsoft.com/azure/app-service/environment/firewall-integration#configuring-azure-firewall-with-your-ase) de este documento para configurar una instancia de Azure Firewall con el ASE.

Si desea usar un dispositivo que no sea Azure Firewall en Gov (US) 

* Los servicios compatibles con puntos de conexión de servicio deben configurarse con puntos de conexión de servicio.
* Los puntos de conexión HTTP/HTTPS de FQDN se pueden colocar en el dispositivo de firewall.
* Los puntos de conexión HTTP/HTTPS de carácter comodín son dependencias que pueden variar con App Service aislado en función del número de calificadores.

Linux no está disponible en regiones Gov (US) y, por tanto, no aparece como una configuración opcional.

#### <a name="service-endpoint-capable-dependencies"></a>Dependencias compatibles con los puntos de conexión de servicio ####

| Punto de conexión |
|----------|
| Azure SQL |
| Azure Storage |
| Centro de eventos de Azure |

#### <a name="ip-address-dependencies"></a>Dependencias de dirección IP

| Punto de conexión | Detalles |
|----------| ----- |
| \*:123 | Comprobación de reloj NTP. El tráfico se comprueba en varios puntos de conexión en el puerto 123. |
| \*:12000 | Este puerto se usa para algunas tareas de supervisión del sistema. Si se bloquea, algunos problemas serán más difíciles de evaluar, pero el ASE seguirá funcionando. |
| 40.77.24.27:80 | Necesario para supervisar y alertar sobre problemas del ASE. |
| 40.77.24.27:443 | Necesario para supervisar y alertar sobre problemas del ASE. |
| 13.90.249.229:80 | Necesario para supervisar y alertar sobre problemas del ASE. |
| 13.90.249.229:443 | Necesario para supervisar y alertar sobre problemas del ASE. |
| 104.45.230.69:80 | Necesario para supervisar y alertar sobre problemas del ASE. |
| 104.45.230.69:443 | Necesario para supervisar y alertar sobre problemas del ASE. |
| 13.82.184.151:80 | Necesario para supervisar y alertar sobre problemas del ASE. |
| 13.82.184.151:443 | Necesario para supervisar y alertar sobre problemas del ASE. |

#### <a name="dependencies"></a>Dependencias ####

| Punto de conexión |
|----------|
| \*.ctldl.windowsupdate.com:80 |
| \*.management.usgovcloudapi.net:80 |
| \*.update.microsoft.com:80 |
|admin.core.usgovcloudapi.net:80 |
|azperfmerges.blob.core.windows.net:80 |
|azperfmerges.blob.core.windows.net:80 |
|azprofileruploads1.blob.core.windows.net:80 |
|azprofileruploads10.blob.core.windows.net:80 |
|azprofileruploads2.blob.core.windows.net:80 |
|azprofileruploads3.blob.core.windows.net:80 |
|azprofileruploads4.blob.core.windows.net:80 |
|azprofileruploads5.blob.core.windows.net:80 |
|azprofileruploads6.blob.core.windows.net:80 |
|azprofileruploads7.blob.core.windows.net:80 |
|azprofileruploads8.blob.core.windows.net:80 |
|azprofileruploads9.blob.core.windows.net:80 |
|azureprofilerfrontdoor.cloudapp.net:80 |
|azurewatsonanalysis.usgovcloudapp.net:80 |
|cacerts.digicert.com:80 |
|client.wns.windows.com:80 |
|crl.microsoft.com:80 |
|crl.verisign.com:80 |
|crl3.digicert.com:80 |
|csc3-2009-2.crl.verisign.com:80 |
|ctldl.windowsupdate.com:80 |
|definitionupdates.microsoft.com:80 |
|download.windowsupdate.com:80 |
|fairfax.warmpath.usgovcloudapi.net:80 |
|flighting.cp.wd.microsoft.com:80 |
|gcwsprodgmdm2billing.queue.core.usgovcloudapi.net:80 |
|gcwsprodgmdm2billing.table.core.usgovcloudapi.net:80 |
|global.metrics.nsatc.net:80 |
|go.microsoft.com:80 |
|gr-gcws-prod-bd3.usgovcloudapp.net:80 |
|gr-gcws-prod-bn1.usgovcloudapp.net:80 |
|gr-gcws-prod-dd3.usgovcloudapp.net:80 |
|gr-gcws-prod-dm2.usgovcloudapp.net:80 |
|gr-gcws-prod-phx20.usgovcloudapp.net:80 |
|gr-gcws-prod-sn5.usgovcloudapp.net:80 |
|login.live.com:80 |
|login.microsoftonline.us:80 |
|management.core.usgovcloudapi.net:80 |
|management.usgovcloudapi.net:80 |
|maupdateaccountff.blob.core.usgovcloudapi.net:80 |
|mscrl.microsoft.com
|ocsp.digicert.0 |
|ocsp.msocsp.co|
|ocsp.verisign.0 |
|rteventse.trafficmanager.net:80 |
|settings-n.data.microsoft.com:80 |
|shavamafestcdnprod1.azureedge.net:80 |
|shavanifestcdnprod1.azureedge.net:80 |
|v10ortex-win.data.microsoft.com:80 |
|wp.microsoft.com:80 |
|dcpalt.microsoft.com:80 |
|www.microsoft.com:80 |
|www.msftconnecttest.com:80 |
|www.thawte.com:80 |
|\*ctldl.windowsupdate.com:443 |
|\*.management.usgovcloudapi.net:443 |
|\*.update.microsoft.com:443 |
|admin.core.usgovcloudapi.net:443 |
|azperfmerges.blob.core.windows.net:443 |
|azperfmerges.blob.core.windows.net:443 |
|azprofileruploads1.blob.core.windows.net:443 |
|azprofileruploads10.blob.core.windows.net:443 |
|azprofileruploads2.blob.core.windows.net:443 |
|azprofileruploads3.blob.core.windows.net:443 |
|azprofileruploads4.blob.core.windows.net:443 |
|azprofileruploads5.blob.core.windows.net:443 |
|azprofileruploads6.blob.core.windows.net:443 |
|azprofileruploads7.blob.core.windows.net:443 |
|azprofileruploads8.blob.core.windows.net:443 |
|azprofileruploads9.blob.core.windows.net:443 |
|azureprofilerfrontdoor.cloudapp.net:443 |
|azurewatsonanalysis.usgovcloudapp.net:443 |
|cacerts.digicert.com:443 |
|client.wns.windows.com:443 |
|crl.microsoft.com:443 |
|crl.verisign.com:443 |
|crl3.digicert.com:443 |
|csc3-2009-2.crl.verisign.com:443 |
|ctldl.windowsupdate.com:443 |
|definitionupdates.microsoft.com:443 |
|download.windowsupdate.com:443 |
|fairfax.warmpath.usgovcloudapi.net:443 |
|flighting.cp.wd.microsoft.com:443 |
|gcwsprodgmdm2billing.queue.core.usgovcloudapi.net:443 |
|gcwsprodgmdm2billing.table.core.usgovcloudapi.net:443 |
|global.metrics.nsatc.net:443 |
|go.microsoft.com:443 |
|gr-gcws-prod-bd3.usgovcloudapp.net:443 |
|gr-gcws-prod-bn1.usgovcloudapp.net:443 |
|gr-gcws-prod-dd3.usgovcloudapp.net:443 |
|gr-gcws-prod-dm2.usgovcloudapp.net:443 |
|gr-gcws-prod-phx20.usgovcloudapp.net:443 |
|gr-gcws-prod-sn5.usgovcloudapp.net:443 |
|login.live.com:443 |
|login.microsoftonline.us:443 |
|management.core.usgovcloudapi.net:443 |
|management.usgovcloudapi.net:443 |
|maupdateaccountff.blob.core.usgovcloudapi.net:443 |
|mscrl.microsoft.com:443 |
|ocsp.digicert.com:443 |
|ocsp.msocsp.com:443 |
|ocsp.verisign.com:443 |
|rteventservice.trafficmanager.net:443 |
|settings-win.data.microsoft.com:443 |
|shavamanifestcdnprod1.azureedge.net:443 |
|shavamanifestcdnprod1.azureedge.net:443 |
|v10.vortex-win.data.microsoft.com:443 |
|wdcp.microsoft.com:443 |
|wdcpalt.microsoft.com:443 |
|www.microsoft.com:443 |
|www.msftconnecttest.com:443 |
|www.thawte.com:443 |

<!--Image references-->
[1]: ./media/firewall-integration/firewall-apprule.png
[2]: ./media/firewall-integration/firewall-serviceendpoints.png
[3]: ./media/firewall-integration/firewall-ntprule.png
[4]: ./media/firewall-integration/firewall-routetable.png
[5]: ./media/firewall-integration/firewall-topology.png
[6]: ./media/firewall-integration/firewall-ntprule-monitor.png
