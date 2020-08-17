---
title: Introducción al registro de flujo para NSG
titleSuffix: Azure Network Watcher
description: En esta página se explica cómo usar la característica de registros de flujo de los grupos de seguridad de red de Azure Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: b73727e6bd824b80fbc3897055d71f6b9c632a61
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084371"
---
# <a name="introduction-to-flow-logging-for-network-security-groups"></a>Introducción al registro de flujo de grupos de seguridad de red

## <a name="introduction"></a>Introducción

Los [registros de flujos de grupos de seguridad de red](https://docs.microsoft.com/azure/virtual-network/security-overview#security-rules) (NSG) son una característica de Azure Network Watcher que permite registrar información acerca del tráfico IP que pasa por un grupo de seguridad de red. Los datos del flujo se envían a cuentas de Azure Storage desde donde puede acceder a ellos y exportarlos a cualquier herramienta de visualización, SIEM o IDS de su elección.

![información general de los registros de flujo](./media/network-watcher-nsg-flow-logging-overview/homepage.jpg)

## <a name="why-use-flow-logs"></a>¿Por qué usar registros de flujo?

Es esencial supervisar, administrar y conocer su propia red para no poner en peligro la seguridad, el cumplimiento y el rendimiento. Conocer su propio entorno es de gran importancia para protegerlo y optimizarlo. A menudo, es necesario conocer el estado actual de la red, quién se conecta, desde dónde se conecta, qué puertos están abiertos a Internet, el comportamiento esperado de la red, comportamientos irregulares de esta o picos repentinos de tráfico.

Los registros de flujo son el origen único de toda la actividad de red del entorno en la nube. Tanto para una startup que intenta optimizar recursos como para una gran empresa que busca detectar intrusiones, los registros de flujo son la mejor opción. Puede usarlos para optimizar flujos de red, supervisar el rendimiento, comprobar el cumplimiento, detectar intrusiones y mucho más.

## <a name="common-use-cases"></a>Casos de uso comunes

**Supervisión de red**: identifique tráfico desconocido o no deseado. Supervise los niveles de tráfico y el consumo de ancho de banda. Filtre los registros de flujo por IP y puerto para comprender el comportamiento de la aplicación. Exporte registros de flujo a las herramientas de visualización y análisis que desee para configurar paneles de supervisión.

**Supervisión y optimización del uso:** identifique desde dónde se envían más datos en la red. Combine datos de GeoIP para identificar el tráfico entre regiones. Comprenda el crecimiento del tráfico para realizar previsiones de capacidad. Use datos para quitar reglas de tráfico excesivamente restrictivas.

**Cumplimiento**: uso de datos de flujo para comprobar el aislamiento de red y el cumplimiento con las reglas de acceso empresariales.

**Análisis forense y de seguridad de la red**: analice flujos de red de interfaces de red y direcciones IP comprometidas. Exporte los registros de flujo a cualquier herramienta SIEM o IDS de su elección.

## <a name="how-logging-works"></a>Cómo funciona el registro

**Propiedades clave**

- Los registros de flujo operan en el [nivel 4](https://en.wikipedia.org/wiki/OSI_model#Layer_4:_Transport_Layer) y registran todos los flujos de IP que entran y salen de un grupo de seguridad de red.
- Los registros se recopilan mediante la plataforma Azure y no afectan a los recursos del cliente ni al rendimiento de la red de ningún modo.
- Los registros se escriben en formato JSON y muestran flujos entrantes y salientes por cada regla de grupo de seguridad de red.
- Cada entrada del registro contiene la interfaz de red (NIC) a la que se aplica el flujo, información de 5-tupla, la decisión de tráfico e información de rendimiento (solo en la versión 2). Consulte _Formato del registro_ más adelante para obtener información detallada.
- Los registros de flujo tienen una característica de retención que permite la eliminación automática de los registros hasta un año después de su creación. 

> [!NOTE]
> La retención solo está disponible si usa [cuentas de almacenamiento de uso general v2 (GPv2)](https://docs.microsoft.com/azure/storage/common/storage-account-overview#types-of-storage-accounts). 

**Conceptos principales**

- Las redes definidas por software están organizadas en torno a redes virtuales y subredes. La seguridad de estas redes virtuales y subredes se puede administrar mediante un grupo de seguridad de red.
- Un grupo de seguridad de red (NSG) contiene una lista de _reglas de seguridad_ que permiten o deniegan el tráfico de red en recursos a los que está conectado. Los grupos de seguridad de red se pueden asociar a subredes, máquinas virtuales individuales o interfaces de red (NIC) individuales conectadas a máquinas virtuales (Resource Manager). Para más información, consulte [Introducción a los grupos de seguridad de red](https://docs.microsoft.com/azure/virtual-network/security-overview?toc=%2Fazure%2Fnetwork-watcher%2Ftoc.json).
- Todos los flujos de tráfico de la red se evalúan mediante las reglas de los grupos de seguridad de red correspondientes.
- El resultado de estas evaluaciones son los registros de flujo de NSG. Los registros de flujo se recopilan mediante la plataforma Azure y no requieren ningún cambio en los recursos del cliente.
- Nota: Las reglas son de dos tipos: de terminación y no de terminación, cada una con distintos comportamientos de registro.
- - Las reglas de denegación de NSG son de terminación. El NSG que deniega el tráfico lo registrará en los registros de flujo, y el procesamiento, en este caso, se detendrá después de que NSG deniegue el tráfico. 
- - Las reglas de permiso de NSG son de no terminación, lo que significa que, aunque un NSG lo permita, el procesamiento continuará hasta el siguiente NSG. El último NSG que permita el tráfico registrará el tráfico en los registros de flujo.
- Los registros de flujo de NSG se escriben en cuentas de almacenamiento desde donde se puede acceder a ellos.
- Puede exportar, procesar, analizar y visualizar registros de flujo mediante herramientas como TA, Splunk, Grafana, Stealthwatch, etc.

## <a name="log-format"></a>Formato de registro

Los registros de flujo incluyen las siguientes propiedades:

* **time**: la hora en la que se registró el evento
* **systemId**: El identificador del sistema del grupo de seguridad de red.
* **category**: la categoría del evento. La categoría es siempre **NetworkSecurityGroupFlowEvent**
* **resourceid**: el identificador del recurso del grupo de seguridad de red.
* **operationName**: siempre es NetworkSecurityGroupFlowEvents
* **properties**: una recopilación de las propiedades del flujo
    * **Version**: número de versión del esquema de eventos del registro de flujos
    * **flows**: una recopilación de flujos. Esta propiedad tiene varias entradas para diferentes reglas
        * **rule**: regla para la que se muestran los flujos
            * **flows**: una recopilación de flujos
                * **mac**: la dirección MAC de la NIC de la máquina virtual en la que se recopiló el flujo
                * **flowTuples**: una cadena que contiene varias propiedades de la tupla de flujo en un formato separado por comas
                    * **Time Stamp**: este valor es la marca de tiempo de cuando se produjo el flujo en formato de época de UNIX
                    * **Source IP**: la IP de origen
                    * **Destination IP**: la IP de destino
                    * **Source Port**: el puerto de origen
                    * **Destination Port**: el puerto de destino
                    * **Protocol**: el protocolo del flujo. Los valores válidos son **T** para TCP y **U** para UDP
                    * **Traffic Flow**: la dirección del flujo de tráfico. Los valores válidos son **I** para el correo entrante y **O** para el saliente.
                    * **Decisión de tráfico**: indica si el tráfico se permitió o se denegó. Los valores válidos son **A** para permitido y **D** para denegado.
                    * **Estado de flujo, solo versión 2**: captura el estado del flujo. Los estados posibles son**B**: Begin (Comienzo), cuando se crea el flujo. No se proporcionan las estadísticas. **C**: Continuación de un flujo en curso. Las estadísticas se proporcionan a intervalos de cinco minutos. **E**: End (Final), cuando termina el flujo. Se proporcionan las estadísticas.
                    * **Paquetes: origen a destino, solo versión 2** El número total de paquetes TCP o UDP enviados desde el origen al destino desde la última actualización.
                    * **Bytes enviados: origen a destino, solo versión 2** El número total de bytes de paquetes TCP o UDP enviados desde el origen al destino desde la última actualización. Los bytes de paquete incluyen el encabezado y la carga del paquete.
                    * **Paquetes: destino a origen, solo versión 2** El número total de paquetes TCP o UDP enviados desde el destino al origen desde la última actualización.
                    * **Bytes enviados: destino a origen, solo versión 2** El número total de bytes de paquetes TCP y UDP enviados desde el destino al origen desde la última actualización. Los bytes de paquete incluyen el encabezado y la carga del paquete.


**Versión 2 de los registros de flujo de NSG (frente a la versión 1)** 

La versión 2 de los registros presenta el concepto de estado de flujo. Puede configurar qué versión de los registros de flujo recibe.

El estado de flujo _B_ se registra cuando se inicia un flujo. El estado de flujo _C_ y el estado de flujo _E_ son estados que marcan la continuación de un flujo y la terminación de este, respectivamente. Los estados _C_ y _E_ contienen información sobre el ancho de banda del tráfico.

### <a name="sample-log-records"></a>Entradas de registro de ejemplo

El texto que sigue es un ejemplo de un registro de flujo. Como puede ver, hay varios registros que siguen la lista de propiedades que se describe en la sección anterior.

> [!NOTE]
> Los valores de la propiedad *flowTuples* son una lista separada por comas.
 
**Muestra de formato de la versión 1 del registro de flujo de NSG**
```json
{
    "records": [
        {
            "time": "2017-02-16T22:00:32.8950000Z",
            "systemId": "2c002c16-72f3-4dc5-b391-3444c3527434",
            "category": "NetworkSecurityGroupFlowEvent",
            "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
            "operationName": "NetworkSecurityGroupFlowEvents",
            "properties": {
                "Version": 1,
                "flows": [
                    {
                        "rule": "DefaultRule_DenyAllInBound",
                        "flows": [
                            {
                                "mac": "000D3AF8801A",
                                "flowTuples": [
                                    "1487282421,42.119.146.95,10.1.0.4,51529,5358,T,I,D"
                                ]
                            }
                        ]
                    },
                    {
                        "rule": "UserRule_default-allow-rdp",
                        "flows": [
                            {
                                "mac": "000D3AF8801A",
                                "flowTuples": [
                                    "1487282370,163.28.66.17,10.1.0.4,61771,3389,T,I,A",
                                    "1487282393,5.39.218.34,10.1.0.4,58596,3389,T,I,A",
                                    "1487282393,91.224.160.154,10.1.0.4,61540,3389,T,I,A",
                                    "1487282423,13.76.89.229,10.1.0.4,53163,3389,T,I,A"
                                ]
                            }
                        ]
                    }
                ]
            }
        },
        {
            "time": "2017-02-16T22:01:32.8960000Z",
            "systemId": "2c002c16-72f3-4dc5-b391-3444c3527434",
            "category": "NetworkSecurityGroupFlowEvent",
            "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
            "operationName": "NetworkSecurityGroupFlowEvents",
            "properties": {
                "Version": 1,
                "flows": [
                    {
                        "rule": "DefaultRule_DenyAllInBound",
                        "flows": [
                            {
                                "mac": "000D3AF8801A",
                                "flowTuples": [
                                    "1487282481,195.78.210.194,10.1.0.4,53,1732,U,I,D"
                                ]
                            }
                        ]
                    },
                    {
                        "rule": "UserRule_default-allow-rdp",
                        "flows": [
                            {
                                "mac": "000D3AF8801A",
                                "flowTuples": [
                                    "1487282435,61.129.251.68,10.1.0.4,57776,3389,T,I,A",
                                    "1487282454,84.25.174.170,10.1.0.4,59085,3389,T,I,A",
                                    "1487282477,77.68.9.50,10.1.0.4,65078,3389,T,I,A"
                                ]
                            }
                        ]
                    }
                ]
            }
        },
    "records":
    [
        
        {
             "time": "2017-02-16T22:00:32.8950000Z",
             "systemId": "2c002c16-72f3-4dc5-b391-3444c3527434",
             "category": "NetworkSecurityGroupFlowEvent",
             "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
             "operationName": "NetworkSecurityGroupFlowEvents",
             "properties": {"Version":1,"flows":[{"rule":"DefaultRule_DenyAllInBound","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282421,42.119.146.95,10.1.0.4,51529,5358,T,I,D"]}]},{"rule":"UserRule_default-allow-rdp","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282370,163.28.66.17,10.1.0.4,61771,3389,T,I,A","1487282393,5.39.218.34,10.1.0.4,58596,3389,T,I,A","1487282393,91.224.160.154,10.1.0.4,61540,3389,T,I,A","1487282423,13.76.89.229,10.1.0.4,53163,3389,T,I,A"]}]}]}
        }
        ,
        {
             "time": "2017-02-16T22:01:32.8960000Z",
             "systemId": "2c002c16-72f3-4dc5-b391-3444c3527434",
             "category": "NetworkSecurityGroupFlowEvent",
             "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
             "operationName": "NetworkSecurityGroupFlowEvents",
             "properties": {"Version":1,"flows":[{"rule":"DefaultRule_DenyAllInBound","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282481,195.78.210.194,10.1.0.4,53,1732,U,I,D"]}]},{"rule":"UserRule_default-allow-rdp","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282435,61.129.251.68,10.1.0.4,57776,3389,T,I,A","1487282454,84.25.174.170,10.1.0.4,59085,3389,T,I,A","1487282477,77.68.9.50,10.1.0.4,65078,3389,T,I,A"]}]}]}
        }
        ,
        {
             "time": "2017-02-16T22:02:32.9040000Z",
             "systemId": "2c002c16-72f3-4dc5-b391-3444c3527434",
             "category": "NetworkSecurityGroupFlowEvent",
             "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
             "operationName": "NetworkSecurityGroupFlowEvents",
             "properties": {"Version":1,"flows":[{"rule":"DefaultRule_DenyAllInBound","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282492,175.182.69.29,10.1.0.4,28918,5358,T,I,D","1487282505,71.6.216.55,10.1.0.4,8080,8080,T,I,D"]}]},{"rule":"UserRule_default-allow-rdp","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282512,91.224.160.154,10.1.0.4,59046,3389,T,I,A"]}]}]}
        }
        
        
```
**Muestra de formato de la versión 2 del registro de flujo de NSG**
```json
 {
    "records": [
        {
            "time": "2018-11-13T12:00:35.3899262Z",
            "systemId": "a0fca5ce-022c-47b1-9735-89943b42f2fa",
            "category": "NetworkSecurityGroupFlowEvent",
            "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
            "operationName": "NetworkSecurityGroupFlowEvents",
            "properties": {
                "Version": 2,
                "flows": [
                    {
                        "rule": "DefaultRule_DenyAllInBound",
                        "flows": [
                            {
                                "mac": "000D3AF87856",
                                "flowTuples": [
                                    "1542110402,94.102.49.190,10.5.16.4,28746,443,U,I,D,B,,,,",
                                    "1542110424,176.119.4.10,10.5.16.4,56509,59336,T,I,D,B,,,,",
                                    "1542110432,167.99.86.8,10.5.16.4,48495,8088,T,I,D,B,,,,"
                                ]
                            }
                        ]
                    },
                    {
                        "rule": "DefaultRule_AllowInternetOutBound",
                        "flows": [
                            {
                                "mac": "000D3AF87856",
                                "flowTuples": [
                                    "1542110377,10.5.16.4,13.67.143.118,59831,443,T,O,A,B,,,,",
                                    "1542110379,10.5.16.4,13.67.143.117,59932,443,T,O,A,E,1,66,1,66",
                                    "1542110379,10.5.16.4,13.67.143.115,44931,443,T,O,A,C,30,16978,24,14008",
                                    "1542110406,10.5.16.4,40.71.12.225,59929,443,T,O,A,E,15,8489,12,7054"
                                ]
                            }
                        ]
                    }
                ]
            }
        },
        {
            "time": "2018-11-13T12:01:35.3918317Z",
            "systemId": "a0fca5ce-022c-47b1-9735-89943b42f2fa",
            "category": "NetworkSecurityGroupFlowEvent",
            "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
            "operationName": "NetworkSecurityGroupFlowEvents",
            "properties": {
                "Version": 2,
                "flows": [
                    {
                        "rule": "DefaultRule_DenyAllInBound",
                        "flows": [
                            {
                                "mac": "000D3AF87856",
                                "flowTuples": [
                                    "1542110437,125.64.94.197,10.5.16.4,59752,18264,T,I,D,B,,,,",
                                    "1542110475,80.211.72.221,10.5.16.4,37433,8088,T,I,D,B,,,,",
                                    "1542110487,46.101.199.124,10.5.16.4,60577,8088,T,I,D,B,,,,",
                                    "1542110490,176.119.4.30,10.5.16.4,57067,52801,T,I,D,B,,,,"
                                ]
                            }
                        ]
                    }
                ]
            }
        }
        
```
**Tupla de registro explicada**

![información general de los registros de flujo](./media/network-watcher-nsg-flow-logging-overview/tuple.png)

**Ejemplo de cálculo de ancho de banda**

tuplas de flujo de una conversación TCP entre 185.170.185.105:35370 y 10.2.0.4:23:

"1493763938,185.170.185.105,10.2.0.4,35370,23,T,I,A,B,,,," "1493695838,185.170.185.105,10.2.0.4,35370,23,T,I,A,C,1021,588096,8005,4610880" "1493696138,185.170.185.105,10.2.0.4,35370,23,T,I,A,E,52,29952,47,27072"

Para los estados de continuación _C_ y finalización _E_, los recuentos de paquetes y bytes son recuentos agregados desde el momento del registro de la tupla de flujo anterior. Con referencia a la conversación de ejemplo anterior, el número total de paquetes transferidos es 1021+52+8005+47 = 9125. El número total de bytes transferidos es 588096+29952+4610880+27072 = 5256000.


## <a name="enabling-nsg-flow-logs"></a>Deshabilitar registros de flujo de NSG

Use el vínculo correspondiente a continuación para consultar guías sobre cómo habilitar los registros de flujo.

- [Azure Portal](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal)
- [PowerShell](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-powershell)
- [CLI](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-cli)
- [REST](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-rest)
- [Azure Resource Manager](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-azure-resource-manager)

## <a name="updating-parameters"></a>Actualizar parámetros

**Azure Portal**

En Azure Portal, vaya a la sección Registro de flujos de NSG en Network Watcher. A continuación, haga clic en el nombre del grupo de seguridad de red. Se abrirá el panel de configuración del registro de flujo. Cambie los parámetros que desee y presione **Guardar** para implementar los cambios.

**PS/CLI/REST/ARM**

Para actualizar parámetros mediante herramientas de línea de comandos, use el mismo comando que se usa para habilitar los registros de flujo (arriba), pero con los parámetros actualizados que desea cambiar.

## <a name="working-with-flow-logs"></a>Trabajar con registros de flujo

*Lectura y exportación de registros de flujo*

- [Descarga y visualización de registros de flujo desde el portal](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal#download-flow-log)
- [Lectura de registros de flujo mediante las funciones de PowerShell](https://docs.microsoft.com/azure/network-watcher/network-watcher-read-nsg-flow-logs)
- [Exportación de registros de flujo de NSG a Splunk](https://www.splunk.com/en_us/blog/tips-and-tricks/splunking-microsoft-azure-network-watcher-data.html)

Aunque los registros de flujo tienen como destino los NSG, no se muestran como los demás registros. Los registros de flujo se almacenan solo dentro de una cuenta de almacenamiento y siguen la ruta de acceso del registro que se muestra en el ejemplo siguiente:

```
https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/resourceId=/SUBSCRIPTIONS/{subscriptionID}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json
```

*Visualización de registros de flujo*

- [Análisis de tráfico de Azure](https://docs.microsoft.com/azure/network-watcher/traffic-analytics) es un servicio nativo de Azure para procesar registros de flujo, extraer información y visualizar registros de flujo. 
- [[Tutorial] Visualización de registros de flujo de NSG con Power BI](https://docs.microsoft.com/azure/network-watcher/network-watcher-visualize-nsg-flow-logs-power-bi)
- [[Tutorial] Visualización de registros de flujo de NSG con Elastic Stack](https://docs.microsoft.com/azure/network-watcher/network-watcher-visualize-nsg-flow-logs-open-source-tools)
- [[Tutorial] Administración y análisis de registros de flujo de NSG con Grafana](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-grafana)
- [[Tutorial] Administración y análisis de registros de flujo de NSG con Graylog](https://docs.microsoft.com/azure/network-watcher/network-watcher-analyze-nsg-flow-logs-graylog)


## <a name="nsg-flow-logging-considerations"></a>Consideraciones acerca del registro de flujo de NSG

**Consideraciones de la cuenta de almacenamiento**: 

- Ubicación: la cuenta de almacenamiento usada debe estar en la misma región que NSG.
- Rotación de claves autoadministrable: si se cambian o rotan las claves de acceso a su cuenta de almacenamiento, los registros de flujos de NSG dejarán de funcionar. Para solucionar este problema, se deben deshabilitar y volver a habilitar los registros de flujos de NSG.

**Costos del registro de flujo**: El registro de flujos de NSG se factura según el volumen de registros generados. Un volumen de tráfico elevado puede producir un volumen de registro de flujo elevado, con los costos asociados. Los precios del registro de flujos de NSG no incluyen los costos de almacenamiento subyacentes. El uso de la característica de directiva de retención con el registro de flujos de NSG implica que se incurre en costos de almacenamiento independientes durante prolongados períodos de tiempo. Si no necesita la característica de directiva de retención, se recomienda que establezca este valor en 0. Para obtener más información, consulte [precios de Network Watcher](https://azure.microsoft.com/pricing/details/network-watcher/) y [precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/) para obtener más detalles.

**Problemas con las reglas TCP de entrada definidas por el usuario**: [Los grupos de seguridad de red (NSG)](https://docs.microsoft.com/azure/virtual-network/security-overview) se implementan como [firewall con estado](https://en.wikipedia.org/wiki/Stateful_firewall?oldformat=true). Sin embargo, debido a las limitaciones actuales de la plataforma, las reglas definidas por el usuario que afectan a los flujos TCP de entrada se implementan en modo sin estado. Debido a esto, los flujos afectados por las reglas de entrada definidas por el usuario pasan a no ser de terminación. Además, los recuentos de bytes y de paquetes no se registran para estos flujos. Por lo tanto, el número de bytes y paquetes que se indican en los registros de flujo de NSG (y Análisis de tráfico) podrían ser diferentes de los números reales. Está previsto que una marca de suscripción que corrige estos problemas esté disponible a finales de diciembre de 2020. Mientras tanto, los clientes que se enfrentan a problemas graves debido a este comportamiento pueden solicitar la suscripción mediante el soporte técnico. Presente una solicitud de soporte técnico en Network Watcher > NSG Flow Logs (Registros de flujo de grupo de seguridad de red).  

**Flujos entrantes registrados desde direcciones IP de Internet a VM sin direcciones IP públicas**: Las VM que no tienen una dirección IP pública asignada a través de una dirección IP pública asociada con la NIC como dirección IP pública de nivel de instancia, o que forman parte de un grupo de back-end de equilibrador de carga básico, usan [SNAT predeterminada](../load-balancer/load-balancer-outbound-connections.md) y tiene una dirección IP asignada por Azure para facilitar la conectividad de salida. Como consecuencia, es posible que vea las entradas de registro de flujo para los flujos desde las direcciones IP de Internet, si el flujo está destinado a un puerto en el intervalo de puertos asignados para SNAT. Si bien Azure no permitirá estos flujos a la VM, el intento se registra y aparece en el registro de flujos de NSG de Network Watcher por diseño. Se recomienda que el tráfico entrante de Internet no deseado se bloquee explícitamente con NSG.

## <a name="best-practices"></a>Procedimientos recomendados

**Habilitar en redes virtuales o subredes críticas**: los registros de flujo deben habilitarse en todas las redes virtuales o subredes críticas de la suscripción como procedimiento recomendado de seguridad y auditoría. 

**Habilitar el registro de flujo de NSG en todos los NSG asociados a un recurso**: En Azure, el registro de flujo en Azure se configura en el recurso de NSG. Un flujo solo se asociará a una regla de NSG. En escenarios en los que se utilizan varios grupos de seguridad de red, se recomienda habilitar los registros de flujo de NSG en todos los grupos de seguridad de red aplicados a la interfaz de red o subred de un recurso para garantizar que todo el tráfico se registre. Para más información, consulte [cómo se evalúa el tráfico](../virtual-network/security-overview.md#how-traffic-is-evaluated) en los grupos de seguridad de red.

**Aprovisionamiento de almacenamiento**: el almacenamiento debe aprovisionarse en línea con el volumen de registros de flujo esperado.

## <a name="troubleshooting-common-issues"></a>Solución de problemas habituales

**No puedo habilitar los registros de flujo de NSG**

- El proveedor de recursos **Microsoft.Insights** no está registrado

Si ha recibido un error _AuthorizationFailed_ o _GatewayAuthenticationFailed_, es posible que no haya habilitado el proveedor de recursos Microsoft Insights en su suscripción. [Siga las instrucciones](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal#register-insights-provider) para habilitar el proveedor Microsoft Insights.

**He habilitado los registros de flujo de NSG pero no veo los datos en mi cuenta de almacenamiento**

- **Tiempo de instalación**

Los registros de flujo de NSG pueden tardar hasta 5 minutos en aparecer en la cuenta de almacenamiento (si se configuró correctamente). Aparecerá un archivo PT1H. JSON, al que puede acceder [tal y como se describe aquí](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal#download-flow-log).

- **No hay tráfico en los NSG**

En ocasiones, no verá registros porque las máquinas virtuales no están activas o hay filtros ascendentes en una puerta de enlace de aplicaciones u otros dispositivos que bloquean el tráfico a su NSG.

**Deseo automatizar los registros de flujo de NSG**

Actualmente las plantillas de ARM no permiten automatizar los registros de flujo de NSG. Consulte el [anuncio de característica](https://azure.microsoft.com/updates/arm-template-support-for-nsg-flow-logs/) para obtener más información.

## <a name="faq"></a>Preguntas más frecuentes

**¿Qué hace Registro de flujo de NSG?**

Los recursos de red de Azure se pueden combinar y administrar mediante [grupos de seguridad de red](https://docs.microsoft.com/azure/virtual-network/security-overview). Registro de flujo de NSG permite registrar información de flujo de una tupla de 5 sobre todo el tráfico de los grupos de seguridad de red. Los registros de flujo sin procesar se escriben en una cuenta de Azure Storage desde donde se pueden procesar, analizar, consultar o exportar según sea necesario.

**¿El uso de registros de flujo afecta a la latencia o el rendimiento de la red?**

Los datos de los registros de flujo se recopilan fuera de la ruta del tráfico de red y, por tanto, no afectan al rendimiento o la latencia de la red. Puede crear o eliminar registros de flujo sin riesgo de que afecte al rendimiento de la red.

**¿Cómo se usan los registros de flujo de grupo de seguridad de red en una cuenta de Storage con un firewall?**

Para usar una cuenta de Storage con un firewall, debe especificar una excepción para que los Servicios de Microsoft de confianza accedan a la cuenta de almacenamiento:

- Para ir a la cuenta de almacenamiento, escriba el nombre de la misma en la búsqueda global del portal o en la [página de cuentas de almacenamiento](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.Storage%2FStorageAccounts)
- En la sección **CONFIGURACIÓN**, seleccione **Firewalls y redes virtuales**.
- En **Permitir el acceso desde**, seleccione **Redes seleccionadas**. Después, en **Excepciones**, active la casilla situada junto a ****Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento****.
- Si ya está seleccionada, no hay que hacer ningún cambio.
- Busque el grupo de seguridad de red de destino en la [página de introducción a los registros de flujo de NSG](https://ms.portal.azure.com/#blade/Microsoft_Azure_Network/NetworkWatcherMenuBlade/flowLogs) y habilite los registros de flujo de NSG con la cuenta de almacenamiento anterior seleccionada.

Puede comprobar los registros de almacenamiento después de unos minutos; verá una marca de tiempo actualizada o un nuevo archivo JSON.

**¿Cómo se usan los registros de flujo de grupo de seguridad de red con una cuenta de Storage detrás de un punto de conexión de servicio?**

Los registros de flujo de NSG son compatibles con los puntos de conexión de servicio sin necesidad de ninguna configuración adicional. Consulte el [tutorial sobre cómo habilitar puntos de conexión de servicio](https://docs.microsoft.com/azure/virtual-network/tutorial-restrict-network-access-to-resources#enable-a-service-endpoint) en su red virtual.

**¿Cuál es la diferencia entre la versión 1 y la versión 2 de los registros de flujo?**

La versión 2 de los registros de flujo presenta el concepto de _estado del flujo_ y almacena información sobre los bytes y los paquetes transmitidos. [Más información](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-overview#log-file)

## <a name="pricing"></a>Precios

Los registros de flujo de NSG se cobran por GB de registros recopilados e incluyen un plan gratuito de 5 GB/mes por suscripción. Para conocer los precios actuales de su región, consulte la [página de precios de Network Watcher](https://azure.microsoft.com/pricing/details/network-watcher/).

El almacenamiento de registros se cobra por separado; consulte la [página de precios de blob en bloques de Azure Storage](https://azure.microsoft.com/pricing/details/storage/blobs/) para ver los precios correspondientes.
 
