---
title: Configurar la restricción del tráfico de red saliente para los clústeres de HDInsight de Azure
description: Obtenga información sobre cómo configurar la restricción del tráfico de red saliente para los clústeres de HDInsight de Azure.
services: hdinsight
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.topic: howto
ms.date: 05/13/2019
ms.openlocfilehash: f244a67abab5c7f8cd14277f87f055ac6d48b8d2
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65762426"
---
# <a name="configure-outbound-network-traffic-restriction-for-azure-hdinsight-clusters"></a>Configurar la restricción del tráfico de red saliente para los clústeres de HDInsight de Azure

En este artículo proporciona los pasos para proteger el tráfico saliente desde el clúster de HDInsight mediante Firewall de Azure. Los pasos siguientes se supone que va a configurar un Firewall de Azure para un clúster existente. Si va a implementar un nuevo clúster y detrás de un firewall, cree su clúster de HDInsight y una subred primero y, a continuación, siga los pasos descritos en esta guía.

## <a name="background"></a>Fondo

Clústeres de HDInsight de Azure se implementan normalmente en su propia red virtual. El clúster tiene dependencias en Servicios fuera de esa red virtual que requieren acceso a la red funcione correctamente.

Existen varias dependencias que requieren el tráfico entrante. El tráfico entrante de administración no se puede enviar a través de un dispositivo de firewall. Las direcciones de origen para este tráfico se conocen y se publican [aquí](hdinsight-extend-hadoop-virtual-network.md#hdinsight-ip). También puede crear reglas de grupo de seguridad de red (NSG) con esta información para proteger el tráfico entrante a los clústeres.

Las dependencias de tráfico saliente de HDInsight se definen casi por completo con el FQDN, que no tiene direcciones IP estáticas detrás de ellos. La falta de direcciones estáticas significa que no se puede usar grupos de seguridad de red (NSG) para bloquear el tráfico saliente desde un clúster. Con la suficiente frecuencia cambian las direcciones que uno no se puede configurar las reglas según la resolución de nombres actual y úsela para configurar las reglas NSG.

La solución para proteger las direcciones de salida es usar un dispositivo de firewall que puede controlar el tráfico saliente basándose en los nombres de dominio. Azure Firewall puede restringir el tráfico saliente HTTP y HTTPS basándose en el FQDN de destino.

## <a name="configuring-azure-firewall-with-hdinsight"></a>Configuración de Firewall de Azure con HDInsight

Un resumen de los pasos para bloquear la salida de la HDInsight existente con el Firewall de Azure son:
1. Habilitar puntos de conexión de servicio.
1. Creación de un firewall.
1. Agregar reglas de aplicación para el firewall
1. Agregar reglas de red al servidor de seguridad.
1. Cree una tabla de enrutamiento.

### <a name="enable-service-endpoints"></a>Habilitar puntos de conexión de servicio

Si desea omitir el servidor de seguridad (por ejemplo, para ahorrar costos de transferencia de datos), a continuación, puede habilitar los puntos de conexión de servicio para SQL y storage en la subred de HDInsight. Cuando haya puntos de conexión de servicio habilitados para SQL Azure, las dependencias SQL de Azure que tiene el clúster deben configurarse con los extremos de servicio.

Para habilitar los puntos de conexión de servicio correcto, complete los pasos siguientes:

1. Inicie sesión en el portal de Azure y seleccione la red virtual que se implementa el clúster de HDInsight en.
1. Seleccione **subredes** en **configuración**.
1. Seleccione la subred donde se implementa el clúster.
1. En la pantalla para editar la configuración de subred, haga clic en **Microsoft.SQL** o **Microsoft.Storage** desde el **los extremos del servicio**  >   **Servicios** cuadro de lista desplegable.
1. Si está usando un clúster de ESP, también debe seleccionar la **Microsoft.AzureActiveDirectory** punto de conexión de servicio.
1. Haga clic en **Save**(Guardar).

### <a name="create-a-new-firewall-for-your-cluster"></a>Creación de un firewall para el clúster nuevo

1. Crear una subred denominada **AzureFirewallSubnet** en la red virtual donde existe el clúster. 
1. Crear un nuevo firewall **prueba FW01** mediante los pasos descritos en [Tutorial: Implementación y configuración de Azure Firewall mediante Azure Portal](../firewall/tutorial-firewall-deploy-portal.md#deploy-the-firewall).
1. Seleccione el nuevo firewall de Azure portal. Haga clic en **reglas** en **configuración** > **colección de reglas de aplicación** > **agregar colección de reglas de aplicación**.

    ![Título: Agregar una colección de reglas de aplicación](./media/hdinsight-restrict-outbound-traffic/hdinsight-restrict-outbound-traffic-add-app-rule-collection.png)

### <a name="configure-the-firewall-with-application-rules"></a>Configurar el firewall con reglas de aplicación

Crear una colección de reglas de aplicación que permite que el clúster enviar y recibir comunicaciones importantes.

Seleccione el nuevo firewall **prueba FW01** desde el portal de Azure. Haga clic en **reglas** en **configuración** > **colección de reglas de aplicación** > **agregar colección de reglas de aplicación**.

En el **agregar colección de reglas de aplicación** pantalla, complete los pasos siguientes:

1. Escriba un **nombre**, **prioridad**y haga clic en **permitir** desde el **acción** menú desplegable.
1. Agregue las siguientes reglas:
    1. Una regla para permitir el tráfico de HDInsight y actualización de Windows:
        1. En el **FQDN etiquetas** sección, proporcione un **nombre**y establezca **direcciones de origen** a `*`.
        1. Seleccione **HDInsight** y **Windows Update** desde el **FQDN etiquetas** menú desplegable.
    1. Una regla para permitir que la actividad de inicio de sesión de Windows:
        1. En el **FQDN de destino** sección, proporcione un **nombre**y establezca **direcciones de origen** a `*`.
        1. Escriba `https:443` en **: puerto de protocolo** y `login.windows.net` en **FQDN de destino**.
    1. Una regla para permitir que los datos de telemetría SQM:
        1. En el **FQDN de destino** sección, proporcione un **nombre**y establezca **direcciones de origen** a `*`.
        1. Escriba `https:443` en **: puerto de protocolo** y `sqm.telemetry.microsoft.com` en **FQDN de destino**.
    1. Si el clúster está respaldado por WASB y no utilizan los puntos de conexión de servicio anteriores, a continuación, agregue una regla para WASB:
        1. En el **FQDN de destino** sección, proporcione un **nombre**y establezca **direcciones de origen** a `*`.
        1. Escriba `wasb` en **: puerto de protocolo** y `*` en **FQDN de destino**.
1. Haga clic en **Agregar**.

![Título: Escriba los detalles de recopilación de regla de aplicación](./media/hdinsight-restrict-outbound-traffic/hdinsight-restrict-outbound-traffic-add-app-rule-collection-details.png)

### <a name="configure-the-firewall-with-network-rules"></a>Configurar el firewall con las reglas de red

Cree las reglas de red para configurar correctamente el clúster de HDInsight.

> [!Important]
> Puede elegir entre el uso de etiquetas de servicio SQL en el firewall mediante reglas de red como se describe en esta sección, o una instancia de SQL extremo de servicio un descrita en [la sección sobre los puntos de conexión de servicio](#enable-service-endpoints). Si decide usar etiquetas SQL en las reglas de red, puede registrar y auditar el tráfico SQL. Uso de un extremo de servicio tendrá tráfico SQL tan alto el firewall.

1. Seleccione el nuevo firewall **prueba FW01** desde el portal de Azure.
1. Haga clic en **reglas** en **configuración** > **colección de reglas de red** > **agregar colección de reglas de red**.
1. En el **agregar colección de reglas de red** pantalla, escriba un **nombre**, **prioridad**y haga clic en **permitir** desde el **acción** menú desplegable.
1. Cree las siguientes reglas:
    1. Una regla de red que permite que el clúster realizar la sincronización de reloj mediante NTP.
        1. En el **reglas** sección, proporcione un **nombre** y seleccione **cualquier** desde el **protocolo** lista desplegable.
        1. Establecer **direcciones de origen** y **direcciones de destino** a `*`.
        1. Establecer **los puertos de destino** a 123.
    1. Si usas el paquete de seguridad de la empresa (ESP), a continuación, agregue una regla de red que permite la comunicación con AAD-DS para los clústeres de ESP.
        1. Determinar las dos direcciones IP para los controladores de dominio.
        1. En la siguiente fila de la **reglas** sección, proporcione un **nombre** y seleccione **cualquier** desde el **protocolo** lista desplegable.
        1. Establecer **direcciones de origen** `*`.
        1. Escriba todas las direcciones IP de los controladores de dominio en **direcciones de destino** separados por comas.
        1. Establecer **los puertos de destino** a `*`.
    1. Si usa Azure Data Lake Storage, a continuación, puede agregar una regla de red para solucionar un problema SNI con ADLS Gen1 y Gen2. Esta opción enrutará el tráfico al firewall que podría ocasionar mayores costes para grandes cargas de datos pero el tráfico se ha iniciado y auditable.
        1. Determinar la dirección IP de su cuenta de Data Lake Storage. Puede usar un comando de powershell, como `[System.Net.DNS]::GetHostAddresses("STORAGEACCOUNTNAME.blob.core.windows.net")` para resolver el FQDN en una dirección IP.
        1. En la siguiente fila de la **reglas** sección, proporcione un **nombre** y seleccione **cualquier** desde el **protocolo** lista desplegable.
        1. Establecer **direcciones de origen** `*`.
        1. Escriba la dirección IP de su cuenta de almacenamiento en **direcciones de destino**.
        1. Establecer **los puertos de destino** a `*`.
    1. Una regla de red para habilitar la comunicación con la clave de administración para Windows activación del servicio.
        1. En la siguiente fila de la **reglas** sección, proporcione un **nombre** y seleccione **cualquier** desde el **protocolo** lista desplegable.
        1. Establecer **direcciones de origen** `*`.
        1. Establecer **direcciones de destino** a `*`.
        1. Establecer **los puertos de destino** a `1688`.
    1. Si usa Log Analytics, a continuación, cree una regla de red para permitir la comunicación con el área de trabajo de Log Analytics.
        1. En la siguiente fila de la **reglas** sección, proporcione un **nombre** y seleccione **cualquier** desde el **protocolo** lista desplegable.
        1. Establecer **direcciones de origen** `*`.
        1. Establecer **direcciones de destino** a `*`.
        1. Establecer **los puertos de destino** a `12000`.
    1. Configurar una etiqueta de servicio de SQL que le permitirá registrar y auditar el tráfico SQL.
        1. En la siguiente fila de la **reglas** sección, proporcione un **nombre** y seleccione **cualquier** desde el **protocolo** lista desplegable.
        1. Establecer **direcciones de origen** `*`.
        1. Establecer **direcciones de destino** a `*`.
        1. Seleccione **Sql** desde el **etiquetas de servicio** lista desplegable.
        1. Establecer **los puertos de destino** a `1433,11000-11999,14000-14999`.
1. Haga clic en **agregar** para completar la creación de la colección de reglas de red.

![Título: Escriba los detalles de recopilación de regla de aplicación](./media/hdinsight-restrict-outbound-traffic/hdinsight-restrict-outbound-traffic-add-network-rule-collection.png)

### <a name="create-and-configure-a-route-table"></a>Crear y configurar una tabla de rutas

Crear una tabla de rutas con las siguientes entradas:

1. Siete direcciones desde [esta lista de direcciones IP de administración de HDInsight requiere](../hdinsight/hdinsight-extend-hadoop-virtual-network.md#hdinsight-ip) con el próximo salto de **Internet**:
    1. Cuatro direcciones IP para todos los clústeres en todas las regiones
    1. Dos direcciones IP que son específicas de la región donde se crea el clúster
    1. Una dirección IP para la resolución recursiva de Azure
1. Una ruta de aplicación Virtual para 0.0.0.0/0 direcciones IP con el próximo salto que se va a la dirección IP privada de Firewall de Azure.

Por ejemplo, para configurar la tabla de rutas para un clúster creado en la región de Estados Unidos de "Central US", use pasos:

1. Inicie sesión en el Portal de Azure.
1. Seleccione el firewall de Azure **prueba FW01**. Copia el **dirección IP privada** aparece en el **Introducción** página. En este ejemplo usaremos un **dirección de 10.1.1.4 de ejemplo**
1. Cree una nueva tabla de rutas.
1. Haga clic en **rutas** en **configuración**.
1. Haga clic en **agregar** crear rutas para las direcciones IP en la tabla siguiente.

| Nombre de ruta | Prefijo de dirección | Tipo del próximo salto | Dirección del próximo salto |
|---|---|---|---|
| 168.61.49.99 | 168.61.49.99/32 | Internet | N/D |
| 23.99.5.239 | 23.99.5.239/32 | Internet | N/D |
| 168.61.48.131 | 168.61.48.131/32 | Internet | N/D |
| 138.91.141.162 | 138.91.141.162/32 | Internet | N/D |
| 13.67.223.215 | 13.67.223.215/32 | Internet | N/D |
| 40.86.83.253 | 40.86.83.253/32 | Internet | N/D |
| 168.63.129.16 | 168.63.129.16/32 | Internet | N/D |
| 0.0.0.0 | 0.0.0.0/0 | Dispositivo virtual | 10.1.1.4 |

![Título: Creación de una tabla de rutas](./media/hdinsight-restrict-outbound-traffic/hdinsight-restrict-outbound-traffic-add-route-table.png)

Completar la configuración de la tabla de ruta:

1. Asignar la tabla de rutas que creó a la subred de HDInsight, haga clic en **subredes** en **configuración** y, a continuación, **asociar**.
1. En el **asociar subred** pantalla, seleccione la red virtual que se creó el clúster en y **AzureFirewallSubnet** creado para su uso con el firewall.
1. Haga clic en **OK**.

![Título: Creación de una tabla de rutas](./media/hdinsight-restrict-outbound-traffic/hdinsight-restrict-outbound-traffic-route-table-associate-subnet.png)

## <a name="edge-node-application-traffic"></a>Tráfico de aplicación de nodo del borde

Los pasos anteriores le permitirá que el clúster para que funcione sin problemas. Deberá configurar las dependencias para dar cabida a sus aplicaciones personalizadas que se ejecutan en los nodos perimetrales, si procede.

Dependencias de la aplicación deben estar identificadas y agregadas para el Firewall de Azure o la tabla de rutas.

Las rutas deben crearse para que el tráfico de aplicación evitar problemas de enrutamiento asimétrico.

Si sus aplicaciones tienen otras dependencias, deben agregarse al Firewall de Azure. Crear reglas de aplicación para permitir el tráfico HTTP/HTTPS y reglas de red para todo lo demás.

## <a name="logging"></a>Registro

Firewall de Azure puede enviar registros a algunos sistemas de almacenamiento diferente. Para obtener instrucciones sobre la configuración de registro para el firewall, siguen los pasos de [Tutorial: Supervisión de los registros de Firewall de Azure](../firewall/tutorial-diagnostics.md).

Una vez haya completado la configuración del registro, si está registrando datos a Log Analytics, puede ver el tráfico bloqueado con una consulta como la siguiente:

```
AzureDiagnostics | where msg_s contains "Deny" | where TimeGenerated >= ago(1h)
```

La integración del Firewall de Azure con los registros de Azure Monitor es útil al obtener primero un trabajo de la aplicación cuando no es consciente de todas las dependencias de aplicación. Puede obtener más información acerca de los registros de Azure Monitor en [Análisis de datos de registro en Azure Monitor](../azure-monitor/log-query/log-query-overview.md).

## <a name="configure-another-network-virtual-appliance"></a>Configurar otro dispositivo de red virtual

>[!Important]
> La información siguiente es **sólo** necesario si quiere configurar una aplicación virtual de red (NVA) que no sea el Firewall de Azure.

Las instrucciones anteriores le ayudan a configurar el Firewall de Azure para restringir el tráfico saliente desde el clúster de HDInsight. Firewall de Azure se configura automáticamente para permitir el tráfico para muchos de los escenarios comunes de importantes. Si desea usar otro dispositivo de red virtual, deberá configurar manualmente un número de características adicionales. Tenga en cuenta lo siguiente la configuración de la aplicación virtual de red:

* Los servicios compatibles con puntos de conexión de servicio deben configurarse con puntos de conexión de servicio.
* Las dependencias de dirección IP son para el tráfico que no sean HTTP/S (el tráfico TCP y UDP).
* Los puntos de conexión HTTP/HTTPS de FQDN se pueden colocar en el dispositivo de aplicación virtual de red.
* Los puntos de conexión HTTP/HTTPS de comodín son las dependencias que pueden variar en función de un número de calificadores.
* Asigne la tabla de rutas que cree para la subred de HDInsight.

### <a name="service-endpoint-capable-dependencies"></a>Dependencias compatibles con los puntos de conexión de servicio

| **Punto de conexión** |
|---|
| Azure SQL |
| Azure Storage |
| Azure Active Directory |

#### <a name="ip-address-dependencies"></a>Dependencias de dirección IP

| **Punto de conexión** | **Detalles** |
|---|---|
| \*:123 | Comprobación de reloj NTP. El tráfico se comprueba en varios puntos de conexión en el puerto 123. |
| Las direcciones IP que se publican [aquí](hdinsight-extend-hadoop-virtual-network.md#hdinsight-ip) | Estos son los servicios de HDInsight |
| Clústeres de AAD-DS direcciones IP privadas para ESP |
| \*: 16800 activación de KMS Windows |
| \*12000 para Log Analytics |

#### <a name="fqdn-httphttps-dependencies"></a>Dependencias HTTP/HTTPS de FQDN

>[!Important]
> La lista siguiente proporciona solo algunos de los FQDN más importantes. Puede obtener la lista completa de FQDN para la configuración de la NVA [en este archivo](https://github.com/Azure-Samples/hdinsight-fqdn-lists/blob/master/HDInsightFQDNTags.json).

| **Punto de conexión**                                                          |
|---|
| azure.archive.ubuntu.com:80                                           |
| security.ubuntu.com:80                                                |
| ocsp.msocsp.com:80                                                    |
| ocsp.digicert.com:80                                                  |
| wawsinfraprodbay063.blob.core.windows.net:443                         |
| registry-1.docker.io:443                                              |
| auth.docker.io:443                                                    |
| production.cloudflare.docker.com:443                                  |
| download.docker.com:443                                               |
| us.archive.ubuntu.com:80                                              |
| download.mono-project.com:80                                          |
| packages.treasuredata.com:80                                          |
| security.ubuntu.com:80                                                |
| azure.archive.ubuntu.com:80                                                |
| ocsp.msocsp.com:80                                                |
| ocsp.digicert.com:80                                                |

## <a name="next-steps"></a>Pasos siguientes

* [Arquitectura de red virtual de Azure HDInsight](hdinsight-virtual-network-architecture.md)
