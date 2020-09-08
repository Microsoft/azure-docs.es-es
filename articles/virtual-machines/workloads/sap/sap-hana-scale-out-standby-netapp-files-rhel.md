---
title: Escalabilidad horizontal de SAP HANA con modo de espera mediante Azure NetApp Files en RHEL | Microsoft Docs
description: Guía de alta disponibilidad para SAP NetWeaver en Red Hat Enterprise Linux con Azure NetApp Files para aplicaciones SAP
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 5e514964-c907-4324-b659-16dd825f6f87
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 06/15/2020
ms.author: radeltch
ms.openlocfilehash: 9978137edb7874a8b93e0c9a5f1f9979ce449277
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88893177"
---
# <a name="deploy-a-sap-hana-scale-out-system-with-standby-node-on-azure-vms-by-using-azure-netapp-files-on-red-hat-enterprise-linux"></a>Implementación de un sistema de escalabilidad horizontal de SAP HANA con nodo en espera en VM de Azure mediante Azure NetApp Files en Red Hat Enterprise Linux 

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md

[anf-azure-doc]:https://docs.microsoft.com/azure/azure-netapp-files/
[anf-avail-matrix]:https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all 
[anf-register]:https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register
[anf-sap-applications-azure]:https://www.netapp.com/us/media/tr-4746.pdf

[2205917]:https://launchpad.support.sap.com/#/notes/2205917
[1944799]:https://launchpad.support.sap.com/#/notes/1944799
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[1410736]:https://launchpad.support.sap.com/#/notes/1410736
[1900823]:https://launchpad.support.sap.com/#/notes/1900823
[2292690]:https://launchpad.support.sap.com/#/notes/2292690
[2455582]:https://launchpad.support.sap.com/#/notes/2455582
[2593824]:https://launchpad.support.sap.com/#/notes/2593824
[2009879]:https://launchpad.support.sap.com/#/notes/2009879

[sap-swcenter]:https://support.sap.com/en/my-support/software-downloads.html

[sap-hana-ha]:sap-hana-high-availability.md
[nfs-ha]:high-availability-guide-suse-nfs.md


En este artículo se describe cómo implementar un sistema SAP HANA de alta disponibilidad en una configuración de escalabilidad horizontal con modo de espera en máquinas virtuales (VM) de Azure Red Hat Enterprise Linux mediante [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) para los volúmenes de almacenamiento compartido.  

En las configuraciones de ejemplo, como en los comandos de instalación, la instancia de HANA es **03** y el identificador del sistema de HANA es **HN1**. Los ejemplos se basan en HANA 2.0 SP4 y Red Hat Enterprise Linux para SAP 7.6. 

Antes de comenzar, consulte las siguientes notas y documentos de SAP:

* [Documentación sobre Azure NetApp Files][anf-azure-doc] 
* Nota de SAP [1928533], que incluye:  
  * Una lista de los tamaños de máquina virtual de Azure que se admiten para la implementación de software de SAP
  * Información importante sobre la capacidad para los tamaños de máquina virtual de Azure
  * Software de SAP admitido y combinaciones de sistema operativo y base de datos
  * La versión del kernel de SAP necesaria para Windows y Linux en Microsoft Azure.
* Nota de SAP [2015553]: enumera los requisitos previos para las implementaciones de software de SAP admitidas por SAP en Azure.
* La nota de SAP [2002167] recomienda la configuración del sistema operativo para Red Hat Enterprise Linux,
* La nota de SAP [2009879] contiene las instrucciones de SAP HANA para Red Hat Enterprise Linux
* Nota de SAP [2178632]: contiene información detallada sobre todas las métricas de supervisión notificadas para SAP en Azure.
* Nota de SAP [2191498]: incluye la versión de SAP Host Agent necesaria para Linux en Azure.
* Nota de SAP [2243692]: contiene información sobre las licencias de SAP en Linux en Azure.
* Nota de SAP [1999351]: contiene información adicional sobre la solución de problemas de la extensión de supervisión mejorada de Azure para SAP.
* Nota de SAP [1900823]: incluye información sobre los requisitos de almacenamiento de SAP HANA.
* [SAP Community Wiki](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) (wiki de la comunidad de SAP): Contiene todas las notas de SAP que se requieren para Linux.
* [Planeación e implementación de Azure Virtual Machines para SAP en Linux][planning-guide]
* [Implementación de Azure Virtual Machines para SAP en Linux][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP en Linux][dbms-guide]
* Documentación general de RHEL
  * [Introducción al complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
  * [Administración del complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
  * [Referencia del complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
  * [Guía de redes de Red Hat Enterprise Linux](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide)
* Documentación de RHEL específica para Azure:
  * [Instalación de SAP HANA en Red Hat Enterprise Linux para su uso en Microsoft Azure](https://access.redhat.com/public-cloud/microsoft-azure)
* [Aplicaciones de NetApp SAP en Microsoft Azure que usan Azure NetApp Files][anf-sap-applications-azure]


## <a name="overview"></a>Información general

Un método para lograr la alta disponibilidad de HANA es mediante la configuración de la conmutación por error automática del host. Para configurar la conmutación por error automática del host, agregue una o más máquinas virtuales al sistema HANA y configúrelas como nodos en espera. Cuando se produce un error en el nodo activo, un nodo en espera toma automáticamente el control. En la configuración presentada con máquinas virtuales de Azure, la conmutación por error automática se consigue mediante [NFS en Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md).  

> [!NOTE]
> El nodo en espera debe tener acceso a todos los volúmenes de la base de datos. Los volúmenes HANA deben montarse como volúmenes NFSv4. El mecanismo de bloqueo mejorado basado en la concesión de archivos en el protocolo NFSv4 se usa para el vallado `I/O`. 

> [!IMPORTANT]
> Para compilar la configuración admitida, debe implementar los volúmenes de datos y de registro de HANA como volúmenes NFSv 4.1 y montarlos mediante el protocolo NFSv 4.1. La configuración de conmutación por error automática del host de HANA con nodo en espera no se admite con NFSv3.

![Información general sobre la alta disponibilidad de SAP NetWeaver](./media/sap-hana-high-availability-rhel/sap-hana-scale-out-standby-netapp-files-rhel.png)

En el diagrama anterior, que sigue las recomendaciones de red de SAP HANA, se representan tres subredes dentro de una red virtual de Azure: 
* Para la comunicación del cliente
* Para la comunicación con el sistema de almacenamiento
* Para la comunicación interna entre nodos de HANA

Los volúmenes de Azure NetApp están en una subred independiente y [delegan a Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).  

En esta configuración de ejemplo, las subredes son:  

  - `client` 10.9.1.0/26  
  - `storage` 10.9.3.0/26  
  - `hana` 10.9.2.0/26  
  - `anf` 10.9.0.0/26 (subred delegada en Azure NetApp Files)

## <a name="set-up-the-azure-netapp-files-infrastructure"></a>Configuración de la infraestructura de Azure NetApp Files 

Antes de continuar con la configuración de la infraestructura de Azure NetApp Files, familiarícese con la [documentación correspondiente][anf-azure-doc]. 

Azure NetApp Files está disponible en varias [regiones de Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp). Compruebe si la región de Azure seleccionada ofrece Azure NetApp Files.  

Para obtener información sobre la disponibilidad de Azure NetApp Files según la región de Azure, consulte [Disponibilidad de Azure NetApp Files por región de Azure][anf-avail-matrix].  

Antes de implementar Azure NetApp Files, solicite la incorporación a este servicio de acuerdo con las [instrucciones para registrarse en Azure NetApp Files][anf-register]. 

### <a name="deploy-azure-netapp-files-resources"></a>Implementación de recursos de Azure NetApp Files  

En las siguientes instrucciones se supone que ya ha implementado la [red virtual de Azure](../../../virtual-network/virtual-networks-overview.md). Los recursos de Azure NetApp Files y las VM en las que esos recursos se montarán se deben implementar en la misma red virtual de Azure o en redes virtuales de Azure emparejadas.  

1. Si aún no implementó los recursos, solicite la [incorporación a Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).  

2. Cree una cuenta de NetApp en la región de Azure seleccionada. Para ello, siga las instrucciones en [Creación de una cuenta de NetApp](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md).  

3. Configure el grupo de capacidad de Azure NetApp Files según disponibles en [Configuración en un grupo de capacidad de Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md).  

   La arquitectura de HANA que se presenta en este artículo utiliza un único grupo de capacidad de Azure NetApp Files, el nivel de *servicio Ultra*. Para las cargas de trabajo HANA en Azure, se recomienda usar el [nivel de servicio](../../../azure-netapp-files/azure-netapp-files-service-levels.md) *Ultra* o *Premium* de Azure NetApp Files.  

4. Delegue una subred en Azure NetApp Files tal como se describe en las instrucciones de [Delegación de una subred en Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).  

5. Implemente los volúmenes de Azure NetApp Files según las instrucciones de [Creación de un volumen de NFS para Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).  

   Cuando vaya a implementar los volúmenes, asegúrese de seleccionar la versión **NFSv 4.1**. Implemente los volúmenes en la [subred](/rest/api/virtualnetwork/subnets) de Azure NetApp Files designada. Las direcciones IP de los volúmenes de Azure NetApp se asignan automáticamente. 
   
   Tenga en cuenta que los recursos de Azure NetApp Files y las VM de Azure deben estar en la misma red virtual de Azure o en redes virtuales de Azure emparejadas. Por ejemplo, **HN1**-data-mnt00001, **HN1**-log-mnt00001, etc. son los nombres de los volúmenes y nfs://10.9.0.4/**HN1**-data-mnt00001, nfs://10.9.0.4/**HN1**-log-mnt00001, etc. son las rutas de acceso de archivo de los volúmenes de Azure NetApp Files.  

   * volumen **HN1**-data-mnt00001 (nfs://10.9.0.4/**HN1**-data-mnt00001)
   * volumen **HN1**-data-mnt00002 (nfs://10.9.0.4/**HN1**-data-mnt00002)
   * volumen **HN1**-log-mnt00001 (nfs://10.9.0.4/**HN1**-log-mnt00001)
   * volumen **HN1**-log-mnt00002 (nfs://10.9.0.4/**HN1**-log-mnt00002)
   * volumen **HN1**-shared (nfs://10.9.0.4/**HN1**-shared)
   
   En este ejemplo, usamos instancias de Azure NetApp Files independientes para cada volumen de datos y registro de HANA. Para una configuración más optimizada para los costos en sistemas más pequeños o no productivos, es posible colocar todos los montajes de datos en un único volumen y todos los montajes de registros en un solo volumen distinto.  

### <a name="important-considerations"></a>Consideraciones importantes

Durante la creación de la instancia de Azure NetApp Files para la escalabilidad horizontal de SAP HANA con el escenario de nodos en espera, tenga en cuenta los siguientes aspectos importantes:

- El grupo de capacidad mínima es 4 tebibytes (TiB).  
- El tamaño del volumen mínimo es de 100 gigabytes (GiB).
- Azure NetApp Files y todas las máquinas virtuales en las que los volúmenes de Azure NetApp Files se montarán se deben implementar en la misma red virtual de Azure o en [redes virtuales emparejadas](../../../virtual-network/virtual-network-peering-overview.md) de la misma región.  
- La red virtual seleccionada debe tener una subred delegada en Azure NetApp Files.
- El rendimiento de un volumen de Azure NetApp Files es una función de la cuota del volumen y del nivel de servicio, como se documenta en [Nivel de servicio para Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). Al ajustar el tamaño de los volúmenes de Azure NetApp de HANA, asegúrese de que el rendimiento resultante cumpla los requisitos del sistema HANA.  
- Con la [directiva de exportación](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md) de Azure NetApp Files, puede controlar los clientes permitidos y el tipo de acceso (lectura y escritura, solo lectura, etc.). 
- La característica Azure NetApp Files no depende aún de la zona. En la actualidad, la característica no se implementa en todas las zonas de disponibilidad de una región de Azure. Tenga en cuenta las posibles implicaciones de latencia en algunas regiones de Azure.  

> [!IMPORTANT]
> Para las cargas de trabajo de SAP HANA, una baja latencia resulta fundamental. Trabaje con su representante de Microsoft para asegurarse de que las máquinas virtuales y los volúmenes de Azure NetApp Files se implementan en ubicaciones muy cercanas entre sí.  

### <a name="sizing-for-hana-database-on-azure-netapp-files"></a>Ajuste del tamaño de la base de datos HANA en Azure NetApp Files

El rendimiento de un volumen de Azure NetApp Files es una función del tamaño del volumen y del nivel de servicio, como se documenta en [Nivel de servicio para Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). 

Al diseñar la infraestructura de SAP en Azure, tenga en cuenta los requisitos mínimos de almacenamiento de SAP, que se traducen en características de rendimiento mínimas:

- Lectura y escritura en /hana/log de 250 megabytes por segundo (MB/s) con tamaños de E/S de 1 MB.  
- Actividad de lectura de al menos 400 MB/s para /hana/data con tamaños de E/S de 16 MB y 64 MB.  
- Actividad de escritura de al menos 250 MB/s para /hana/data con tamaños de E/S de 16 MB y 64 MB. 

Los [límites de rendimiento de Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md) por 1 TiB de cuota de volumen son los siguientes:
- Capa de almacenamiento Premium: 64 MiB/s  
- Capa de almacenamiento Ultra: 128 MiB/s  

Para cumplir los requisitos de rendimiento mínimo de SAP para los datos y el registro, y de acuerdo con las directrices para /hana/shared, los tamaños recomendados serían los siguientes:

| Volumen | Tamaño de<br>Capa Premium Storage | Tamaño de<br>Capa de almacenamiento Ultra | Protocolo NFS admitido |
| --- | --- | --- | --- |
| /hana/log/ | 4 TiB | 2 TiB | v4.1 |
| /hana/data | 6,3 TiB | 3,2 TiB | v4.1 |
| /hana/shared | 1 x RAM por cuatro nodos de trabajo | 1 x RAM por cuatro nodos de trabajo | v3 o v4.1 |

La configuración de SAP HANA para el diseño que se presenta en este artículo, con la capa de almacenamiento Ultra de Azure NetApp Files, sería la siguiente:

| Volumen | Tamaño de<br>Capa de almacenamiento Ultra | Protocolo NFS admitido |
| --- | --- | --- |
| /hana/log/mnt00001 | 2 TiB | v4.1 |
| /hana/log/mnt00002 | 2 TiB | v4.1 |
| /hana/data/mnt00001 | 3,2 TiB | v4.1 |
| /hana/data/mnt00002 | 3,2 TiB | v4.1 |
| /hana/shared | 2 TiB | v3 o v4.1 |

> [!NOTE]
> Las recomendaciones de tamaño de Azure NetApp Files que indicadas aquí tienen como objetivo satisfacer los requisitos mínimos que SAP recomienda para sus proveedores de infraestructura. En escenarios reales de implementaciones de clientes y cargas de trabajo, es posible que estos tamaños no sean suficientes. Utilice estas recomendaciones como punto de partida y adáptelas en función de los requisitos de la carga de trabajo específica.  

> [!TIP]
> Puede cambiar el tamaño de los volúmenes de Azure NetApp Files de manera dinámica, sin necesidad de *desmontar* los volúmenes, detener las máquinas virtuales ni detener SAP HANA. Este método permite la flexibilidad necesaria para satisfacer las demandas de rendimiento esperadas e imprevistas de la aplicación.

## <a name="deploy-linux-virtual-machines-via-the-azure-portal"></a>Implementación de máquinas virtuales Linux en Azure Portal

Primero deberá crear los volúmenes de Azure NetApp Files. Luego, siga estos pasos:
1. Cree las [subredes de la red virtual de Azure](../../../virtual-network/virtual-network-manage-subnet.md) en la [red virtual de Azure](../../../virtual-network/virtual-networks-overview.md). 
1. Implemente las VM. 
1. Cree las interfaces de red adicionales y conecte las interfaces de red a las VM correspondientes.  

   Cada máquina virtual tiene tres interfaces de red, que corresponden a las tres subredes de la red virtual de Azure (`client`, `storage` y `hana`). 

   Para más información, consulte [Creación de una máquina virtual Linux en Azure con varias tarjetas de interfaz de red](../../linux/multiple-nics.md).  

> [!IMPORTANT]
> Para las cargas de trabajo de SAP HANA, una baja latencia resulta fundamental. Para lograr una baja latencia, trabaje con su representante de Microsoft para asegurarse de que las máquinas virtuales y los volúmenes de Azure NetApp Files se implementen en ubicaciones muy cercanas entre sí. Envíe la información necesaria al [incorporar un nuevo sistema SAP HANA](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxjSlHBUxkJBjmARn57skvdUQlJaV0ZBOE1PUkhOVk40WjZZQVJXRzI2RC4u), que usa Azure NetApp Files de SAP HANA. 
 
En las instrucciones siguientes, se supone que ya ha creado el grupo de recursos, la red virtual de Azure y las tres subredes de la red virtual de Azure: `client`, `storage` y `hana`. Al implementar las máquinas virtuales, seleccione la subred de cliente, de modo que la interfaz de red de cliente sea la interfaz principal de las VM. También tendrá que configurar una ruta explícita a la subred delegada de Azure NetApp Files mediante la puerta de enlace de la subred de almacenamiento. 

> [!IMPORTANT]
> Asegúrese de que el sistema operativo seleccionado esté certificado por SAP para SAP HANA en los tipos específicos de VM que use. Para obtener una lista de los tipos de VM certificados para SAP HANA y las versiones de sistema operativo correspondientes, consulte el sitio [Plataformas de IaaS certificadas para SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Haga clic en los detalles del tipo de VM de la lista para obtener la lista completa de las versiones de sistema operativo compatibles con SAP HANA para ese tipo.  

1. Cree un conjunto de disponibilidad para SAP HANA. Asegúrese de establecer el dominio máximo de actualización.  

2. Cree tres máquinas virtuales (**hanadb1**, **hanadb2**, **hanadb3**). Para ello, haga lo siguiente:  

   a. Use una imagen de Red Hat Enterprise Linux en la galería de Azure que sea compatible con SAP HANA. En este ejemplo se usa una imagen RHEL-SAP-HA 7.6.  

   b. Seleccione el conjunto de disponibilidad que creó anteriormente para SAP HANA.  

   c. Seleccione la subred de la red virtual de Azure del cliente. Seleccione [Accelerated Networking](../../../virtual-network/create-vm-accelerated-networking-cli.md).  

   Al implementar las máquinas virtuales, el nombre de la interfaz de red se genera automáticamente. Para simplificar, en estas instrucciones haremos referencia a las interfaces de red generadas automáticamente, que están conectadas a la subred de la red virtual de Azure del cliente, como **hanadb1-client**, **hanadb2-client** y **hanadb3-client**. 

3. Cree tres interfaces de red, una para cada máquina virtual, para la subred de la red virtual `storage` (en este ejemplo **hanadb1-storage**, **hanadb2-storage** y **hanadb3-storage**).  

4. Cree tres interfaces de red, una para cada máquina virtual, para la subred de red virtual `hana` (en este ejemplo **hanadb1-hana**, **hanadb2-hana** y **hanadb3-hana**).  

5. Conecte las interfaces de red virtual recién creadas a las máquinas virtuales correspondientes. Para ello, siga estos pasos:  

    a. Vaya a la máquina virtual en [Azure Portal](https://portal.azure.com/#home).  

    b. En el menú de la izquierda, seleccione **Máquinas virtuales**. Filtre por el nombre de máquina virtual (por ejemplo, **hanadb1**) y luego seleccione la máquina virtual.  

    c. En el panel **Información general**, seleccione **Detener** para desasignar la máquina virtual.  

    d. Seleccione **Redes** y adjunte la interfaz de red. En la lista desplegable **Adjuntar interfaz de red**, seleccione las interfaces de red ya creadas para las subredes `storage` y `hana`.  
    
    e. Seleccione **Guardar**. 
 
    f. Repita los pasos del b al e para las demás máquinas virtuales (en nuestro ejemplo, **hanadb2** y **hanadb3**).
 
    g. Deje las máquinas virtuales en estado detenido por ahora. A continuación, habilitaremos la opción [Redes aceleradas](../../../virtual-network/create-vm-accelerated-networking-cli.md) para todas las interfaces de red que se acaban de conectar.  

6. Habilite redes aceleradas para las interfaces de red adicionales de las subredes `storage` y `hana`. Para ello, siga estos pasos:  

    a. Abra [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/) en [Azure Portal](https://portal.azure.com/#home).  

    b. Ejecute los siguientes comandos para habilitar las redes aceleradas para las interfaces de red adicionales, conectadas a las subredes `storage` y `hana`.  

    <pre><code>
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb1-storage</b> --accelerated-networking true
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb2-storage</b> --accelerated-networking true
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb3-storage</b> --accelerated-networking true
    
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb1-hana</b> --accelerated-networking true
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb2-hana</b> --accelerated-networking true
    az network nic update --id /subscriptions/<b>your subscription</b>/resourceGroups/<b>your resource group</b>/providers/Microsoft.Network/networkInterfaces/<b>hanadb3-hana</b> --accelerated-networking true

    </code></pre>

7. Inicie las máquinas virtuales. Para ello, siga estos pasos:  

    a. En el menú de la izquierda, seleccione **Máquinas virtuales**. Filtre por el nombre de máquina virtual (por ejemplo, **hanadb1**) y luego seleccione la máquina virtual.  

    b. En el panel **Información general**, seleccione **Iniciar**.  

## <a name="operating-system-configuration-and-preparation"></a>Configuración y preparación del sistema operativo

Las instrucciones de las secciones siguientes tienen como prefijo una de las siguientes opciones:
* **[A]** : Aplicable a todos los nodos
* **[1]** : Aplicable solo al nodo 1
* **[2]** : Aplicable solo al nodo 2
* **[3]** : aplicable solo al nodo 3

Configure y prepare el sistema operativo. Para ello, siga estos pasos:

1. **[A]** Mantenga los archivos de host en las máquinas virtuales. Incluya las entradas para todas las subredes. En este ejemplo se agregaron las siguientes entradas a `/etc/hosts`.  

    <pre><code>
    # Storage
    10.9.3.4   hanadb1-storage
    10.9.3.5   hanadb2-storage
    10.9.3.6   hanadb3-storage
    # Client
    10.9.1.5   hanadb1
    10.9.1.6   hanadb2
    10.9.1.7   hanadb3
    # Hana
    10.9.2.4   hanadb1-hana
    10.9.2.5   hanadb2-hana
    10.9.2.6   hanadb3-hana
    </code></pre>

3. **[A]** Agregue una ruta de red, de modo que la comunicación con Azure NetApp Files pase a través de la interfaz de red de almacenamiento.  

   En este ejemplo, se utilizará `Networkmanager` para configurar la ruta de red adicional. En las instrucciones siguientes se presupone que la interfaz de red de almacenamiento es `eth1`.  
   En primer lugar, determine el nombre de la conexión del dispositivo `eth1`. En este ejemplo, el nombre de la conexión del dispositivo `eth1` es `Wired connection 1`.  

    <pre><code>
    # Execute as root
    nmcli connection
    # Result
    #NAME                UUID                                  TYPE      DEVICE
    #System eth0         5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03  ethernet  eth0
    #Wired connection 1  4b0789d1-6146-32eb-83a1-94d61f8d60a7  ethernet  eth1
    </code></pre>

   A continuación, configure la ruta adicional en la red delegada de Azure NetApp Files a través de `eth1`.  

    <pre><code>
    # Add the following route 
    # ANFDelegatedSubnet/cidr via StorageSubnetGW dev StorageNetworkInterfaceDevice
    nmcli connection modify <b>"Wired connection 1"</b> +ipv4.routes <b>"10.9.0.0/26 10.9.3.1"</b>
    </code></pre>

    Reinicie la máquina virtual para activar los cambios.  

3. **[A]** Instale el paquete de cliente NFS.  

    <pre><code>
    yum install nfs-utils
    </code></pre>

3. **[A]** Prepare el sistema operativo para ejecutar SAP HANA en Azure NetApp con NFS, como se describe en [Aplicaciones de NetApp SAP en Microsoft Azure que usan Azure NetApp Files][anf-sap-applications-azure]. Cree el archivo de configuración */etc/sysctl.d/netapp-hana.conf* para las opciones de configuración de NetApp.  

    <pre><code>
    vi /etc/sysctl.d/netapp-hana.conf
    # Add the following entries in the configuration file
    net.core.rmem_max = 16777216
    net.core.wmem_max = 16777216
    net.core.rmem_default = 16777216
    net.core.wmem_default = 16777216
    net.core.optmem_max = 16777216
    net.ipv4.tcp_rmem = 65536 16777216 16777216
    net.ipv4.tcp_wmem = 65536 16777216 16777216
    net.core.netdev_max_backlog = 300000 
    net.ipv4.tcp_slow_start_after_idle=0 
    net.ipv4.tcp_no_metrics_save = 1
    net.ipv4.tcp_moderate_rcvbuf = 1
    net.ipv4.tcp_window_scaling = 1
    net.ipv4.tcp_timestamps = 1
    net.ipv4.tcp_sack = 1
    </code></pre>

4. **[A]** Cree el archivo de configuración */etc/sysctl.d/ms-az.conf* con las opciones de optimización adicionales.  

    <pre><code>
    vi /etc/sysctl.d/ms-az.conf
    # Add the following entries in the configuration file
    net.ipv6.conf.all.disable_ipv6 = 1
    net.ipv4.tcp_max_syn_backlog = 16348
    net.ipv4.ip_local_port_range = 40000 65300
    net.ipv4.conf.all.rp_filter = 0
    sunrpc.tcp_slot_table_entries = 128
    vm.swappiness=10
    </code></pre>

5. **[A]** Ajuste la configuración de sunrpc, tal como se recomienda en [Aplicaciones de NetApp SAP en Microsoft Azure que usan Azure NetApp Files][anf-sap-applications-azure].  

    <pre><code>
    vi /etc/modprobe.d/sunrpc.conf
    # Insert the following line
    options sunrpc tcp_max_slot_table_entries=128
    </code></pre>

6. **[A]** Configuración de Red Hat para HANA.

    Configure RHEL tal y como se describe en las notas de SAP [2292690], [2455582], [2593824] y <https://access.redhat.com/solutions/2447641>.

    > [!NOTE]
    > Si instala HANA 2.0 SP04, deberá instalar el paquete `compat-sap-c++-7` como se describe en la nota de SAP [2593824], antes de poder instalar SAP HANA. 

## <a name="mount-the-azure-netapp-files-volumes"></a>Montaje de los volúmenes de Azure NetApp Files

1. **[A]** Cree puntos de montaje para los volúmenes de bases de datos HANA.  

    <pre><code>
    mkdir -p /hana/data/<b>HN1</b>/mnt00001
    mkdir -p /hana/data/<b>HN1</b>/mnt00002
    mkdir -p /hana/log/<b>HN1</b>/mnt00001
    mkdir -p /hana/log/<b>HN1</b>/mnt00002
    mkdir -p /hana/shared
    mkdir -p /usr/sap/<b>HN1</b>
    </code></pre>

2. **[1]** Cree directorios específicos de nodo para /usr/sap en el recurso compartido **HN1**.  

    <pre><code>
    # Create a temporary directory to mount <b>HN1</b>-shared
    mkdir /mnt/tmp
    # if using NFSv3 for this volume, mount with the following command
    mount <b>10.9.0.4</b>:/<b>HN1</b>-shared /mnt/tmp
    # if using NFSv4.1 for this volume, mount with the following command
    mount -t nfs -o sec=sys,vers=4.1 <b>10.9.0.4</b>:/<b>HN1</b>-shared /mnt/tmp
    cd /mnt/tmp
    mkdir shared usr-sap-<b>hanadb1</b> usr-sap-<b>hanadb2</b> usr-sap-<b>hanadb3</b>
    # unmount /hana/shared
    cd
    umount /mnt/tmp
    </code></pre>

3. **[A]** Compruebe la configuración del dominio NFS. Asegúrese de que el dominio esté configurado como dominio predeterminado de Azure NetApp Files, es decir, **`defaultv4iddomain.com`** y que la asignación se haya establecido en **nobody**.  

    > [!IMPORTANT]
    > Asegúrese de establecer el dominio NFS de `/etc/idmapd.conf` en la máquina virtual para que coincida con la configuración de dominio predeterminada en Azure NetApp Files: **`defaultv4iddomain.com`** . Si hay alguna discrepancia entre la configuración de dominio del cliente NFS (es decir, la máquina virtual) y el servidor NFS (es decir la configuración de Azure NetApp), los permisos de archivos en volúmenes de Azure NetApp que estén montados en las máquinas virtuales se mostrarán como `nobody`.  

    <pre><code>
    sudo cat /etc/idmapd.conf
    # Example
    [General]
    Domain = <b>defaultv4iddomain.com</b>
    [Mapping]
    Nobody-User = <b>nobody</b>
    Nobody-Group = <b>nobody</b>
    </code></pre>

4. **[A]** Compruebe `nfs4_disable_idmapping`. Debe establecerse en **S**. Para crear la estructura de directorio en la que se encuentra `nfs4_disable_idmapping`, ejecute el comando mount. No podrá crear manualmente el directorio en /sys/modules, ya que el acceso está reservado para el kernel o los controladores.  

    <pre><code>
    # Check nfs4_disable_idmapping 
    cat /sys/module/nfs/parameters/nfs4_disable_idmapping
    # If you need to set nfs4_disable_idmapping to Y
    mkdir /mnt/tmp
    mount 10.9.0.4:/HN1-shared /mnt/tmp
    umount  /mnt/tmp
    echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping
    # Make the configuration permanent
    echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
    </code></pre>

   Para obtener más información sobre cómo cambiar el parámetro `nfs4_disable_idmapping`, consulte https://access.redhat.com/solutions/1749883.

6. **[A]** Monte los volúmenes de Azure NetApp Files compartidos.  

    <pre><code>
    sudo vi /etc/fstab
    # Add the following entries
    10.9.0.4:/<b>HN1</b>-data-mnt00001 /hana/data/<b>HN1</b>/mnt00001  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    10.9.0.4:/<b>HN1</b>-data-mnt00002 /hana/data/<b>HN1</b>/mnt00002  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    10.9.0.4:/<b>HN1</b>-log-mnt00001 /hana/log/<b>HN1</b>/mnt00001  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    10.9.0.4:/<b>HN1</b>-log-mnt00002 /hana/log/HN1/mnt00002  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    10.9.0.4:/<b>HN1</b>-shared/shared /hana/shared  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    # Mount all volumes
    sudo mount -a 
    </code></pre>

7. **[1]** Monte los volúmenes específicos del nodo en **hanadb1**.  

    <pre><code>
    sudo vi /etc/fstab
    # Add the following entries
    10.9.0.4:/<b>HN1</b>-shared/usr-sap-<b>hanadb1</b> /usr/sap/<b>HN1</b>  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    # Mount the volume
    sudo mount -a 
    </code></pre>

8. **[2]** Monte los volúmenes específicos del nodo en **hanadb2**.  

    <pre><code>
    sudo vi /etc/fstab
    # Add the following entries
    10.9.0.4:/<b>HN1</b>-shared/usr-sap-<b>hanadb2</b> /usr/sap/<b>HN1</b>  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    # Mount the volume
    sudo mount -a 
    </code></pre>

9. **[3]** Monte los volúmenes específicos del nodo en **hanadb3**.  

    <pre><code>
    sudo vi /etc/fstab
    # Add the following entries
    10.9.0.4:/<b>HN1</b>-shared/usr-sap-<b>hanadb3</b> /usr/sap/<b>HN1</b>  nfs   rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys  0  0
    # Mount the volume
    sudo mount -a 
    </code></pre>

10. **[A]** Compruebe que todos los volúmenes de HANA están montados con la versión del protocolo NFS **NFSv4**.  

    <pre><code>
    sudo nfsstat -m
    # Verify that flag vers is set to <b>4.1</b> 
    # Example from <b>hanadb1</b>
    /hana/data/<b>HN1</b>/mnt00001 from 10.9.0.4:/<b>HN1</b>-data-mnt00001
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.9.3.4,local_lock=none,addr=10.9.0.4
    /hana/log/<b>HN1</b>/mnt00002 from 10.9.0.4:/<b>HN1</b>-log-mnt00002
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.9.3.4,local_lock=none,addr=10.9.0.4
    /hana/data/<b>HN1</b>/mnt00002 from 10.9.0.4:/<b>HN1</b>-data-mnt00002
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.9.3.4,local_lock=none,addr=10.9.0.4
    /hana/log/<b>HN1</b>/mnt00001 from 10.9.0.4:/<b>HN1</b>-log-mnt00001
    Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.9.3.4,local_lock=none,addr=10.9.0.4
    /usr/sap/<b>HN1</b> from 10.9.0.4:/<b>HN1</b>-shared/usr-sap-<b>hanadb1</b>
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.9.3.4,local_lock=none,addr=10.9.0.4
    /hana/shared from 10.9.0.4:/<b>HN1</b>-shared/shared
     Flags: rw,noatime,vers=<b>4.1</b>,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.9.3.4,local_lock=none,addr=10.9.0.4
    </code></pre>

## <a name="installation"></a>Instalación  

En este ejemplo para implementar SAP HANA en la configuración de escalabilidad horizontal con un nodo en espera con Azure, usamos HANA 2.0 SP4.  

### <a name="prepare-for-hana-installation"></a>Preparación de la instalación de HANA

1. **[A]** Antes de realizar la instalación de HANA, establezca la contraseña raíz. Puede deshabilitar la contraseña raíz una vez completada la instalación. Ejecute como `passwd` del comando `root`.  

2. **[1]** Compruebe que puede iniciar sesión a través de SSH en **hanadb2** y **hanadb3**, sin que se le pida una contraseña.  

    <pre><code>
    ssh root@<b>hanadb2</b>
    ssh root@<b>hanadb3</b>
    </code></pre>

3. **[A]** Instale paquetes adicionales, necesarios para HANA 2.0 SP4. Para más información, consulte la nota de SAP [2593824](https://launchpad.support.sap.com/#/notes/2593824). 

    <pre><code>
    yum install libgcc_s1 libstdc++6 compat-sap-c++-7 libatomic1 
    </code></pre>

4. **[2], [3]** Cambie la propiedad de los directorios `data` y `log` de SAP HANA a **hn1**adm.   

    <pre><code>
    # Execute as root
    sudo chown hn1adm:sapsys /hana/data/<b>HN1</b>
    sudo chown hn1adm:sapsys /hana/log/<b>HN1</b>
    </code></pre>

5. **[A]** Deshabilite el firewall temporalmente para que no interfiera con la instalación de HANA. Puede volver a habilitarlo después de realizar la instalación de HANA. 
   <pre><code>
    # Execute as root
    systemctl stop firewalld
    systemctl disable firewalld
   </code></pre>

### <a name="hana-installation"></a>Instalación de HANA

1. **[1]** Instale SAP HANA. Para ello, siga las instrucciones de la [guía de instalación y actualización de SAP HANA 2.0](https://help.sap.com/viewer/2c1988d620e04368aa4103bf26f17727/2.0.04/en-US/7eb0167eb35e4e2885415205b8383584.html). En este ejemplo, se instala la escalabilidad horizontal de SAP HANA con el nodo maestro, un nodo de trabajo y uno en espera.  

   a. Inicie el programa **hdblcm** desde el directorio de software de instalación de HANA. Use el parámetro `internal_network` y pase el espacio de direcciones a la subred que se usa para la comunicación interna entre nodos de HANA.  

    <pre><code>
    ./hdblcm --internal_network=10.9.2.0/26
    </code></pre>

   b. Escriba los siguientes valores en el símbolo del sistema:

     * En **Elegir una acción**: escriba **1** (para la instalación)
     * En **Additional components for installation** (Componentes adicionales para la instalación): especifique **2, 3**
     * Para la ruta de instalación: presione Entrar (el valor predeterminado es/Hana/Shared).
     * En **Local Host Name** (Nombre de host local): presione Entrar para aceptar el valor predeterminado.
     * En **Do you want to add hosts to the system?** (¿Desea agregar hosts al sistema): escriba **y**.
     * En **comma-separated host names to add** (nombres de host separados por comas que se deben agregar): escriba **hanadb2, hanadb3**
     * En **Root User Name** (Nombre de usuario raíz) [raíz]: presione Entrar para aceptar el valor predeterminado.
     * Para los roles de host hanadb2: escriba **1** (para el rol de trabajo)
     * En **Host Failover Group** (Grupo de conmutación por error de host) para el host hanadb2 [valor predeterminado]: presione Entrar para aceptar el valor predeterminado.
     * En **Storage Partition Number** (Número de partición de almacenamiento) para el host hanadb2 [<<assign automatically>>]: presione Entrar para aceptar el valor predeterminado.
     * En **Worker Group** (Grupo de roles de trabajo) para el host hanadb2 [valor predeterminado]: presione Entrar para aceptar el valor predeterminado.
     * En **Select roles** (Seleccionar roles) para host hanadb3: escriba **2** (para el modo de espera).
     * En **Host Failover Group** (Grupo de conmutación por error de host) para el host hanadb3 [valor predeterminado]: presione Entrar para aceptar el valor predeterminado.
     * En **Worker Group** (Grupo de roles de trabajo) para el host hanadb3 [valor predeterminado]: presione Entrar para aceptar el valor predeterminado.
     * En **SAP HANA ID** (Id. de SAP HANA) del sistema: escriba **HN1**.
     * En **Instance number** (Número de instancia) [00]: escriba **03**.
     * En **Local Host Worker Group** (Grupo de trabajo de host local) [valor predeterminado]: presione Entrar para aceptar el valor predeterminado.
     * En **Select System Usage / Enter index [4]** (Seleccionar uso del sistema o especificar el índice [4]): escriba **4** (para personalizar).
     * En **Location of Data Volumes** (Ubicación de los volúmenes de datos) [/hana/data/HN1]: presione Entrar para aceptar el valor predeterminado.
     * En **Location of Log Volumes** (Ubicación de los volúmenes de registro) [/hana/log/HN1]: presione Entrar para aceptar el valor predeterminado.
     * En **Restrict maximum memory allocation?** (¿Restringir la asignación de memoria máxima?) [n]: especifique **n**.
     * En **Certificate Host Name For Host hanadb1** (Nombre de host del certificado para el host hanadb1) [hanadb1]: presione Entrar para aceptar el valor predeterminado.
     * En **Certificate Host Name For Host hanadb2** (Nombre de host del certificado para el host hanadb2) [hanadb2]: presione Entrar para aceptar el valor predeterminado.
     * En **Certificate Host Name For hanadb3** (Nombre de host del certificado para el host hanadb3) [hanadb3]: presione Entrar para aceptar el valor predeterminado.
     * En **System Administrator (hn1adm) Password** (Contraseña del administrador del sistema (hn1adm)): escriba la contraseña.
     * En **System Database User (system) Password** (Contraseña de usuario de base de datos del sistema (sistema)): escriba la contraseña del sistema.
     * En **Confirm System Database User (system) Password** (Confirmar la contraseña del usuario (sistema) de la base de datos del sistema): escriba la contraseña del sistema.
     * En **Restart system after machine reboot?** (¿Reiniciar el sistema tras el reinicio de la máquina?) [n]: especifique **n**. 
     * En **Do you want to continue (y/n)** (¿Desea continuar (sí/no)?): valide el resumen y, si todo es correcto, especifique **s**.


2. **[1]** Compruebe global.ini.  

   Muestre global.ini y asegúrese de que la configuración de la comunicación interna entre nodos de SAP HANA esté presente. Compruebe la sección de **communication**. Debe tener el espacio de direcciones para la subred `hana`, y `listeninterface` debe estar establecido en `.internal`. Compruebe la sección **internal_hostname_resolution**. Debe tener las direcciones IP de las máquinas virtuales de HANA que pertenezcan a la subred `hana`.  

   <pre><code>
    sudo cat /usr/sap/<b>HN1</b>/SYS/global/hdb/custom/config/global.ini
    # Example 
    #global.ini last modified 2019-09-10 00:12:45.192808 by hdbnameserve
    [communication]
    internal_network = <b>10.9.2.0/26</b>
    listeninterface = .internal
    [internal_hostname_resolution]
    <b>10.9.2.4</b> = <b>hanadb1</b>
    <b>10.9.2.5</b> = <b>hanadb2</b>
    <b>10.9.2.6</b> = <b>hanadb3</b>
   </code></pre>

3. **[1]** Agregue la asignación de host para asegurarse de que las direcciones IP del cliente se utilizan para la comunicación del cliente. Agregue la sección `public_host_resolution` y agregue las direcciones IP correspondientes de la subred del cliente.  

   <pre><code>
    sudo vi /usr/sap/HN1/SYS/global/hdb/custom/config/global.ini
    #Add the section
    [public_hostname_resolution]
    map_<b>hanadb1</b> = <b>10.9.1.5</b>
    map_<b>hanadb2</b> = <b>10.9.1.6</b>
    map_<b>hanadb3</b> = <b>10.9.1.7</b>
   </code></pre>

4. **[1]** Reinicie SAP HANA para activar los cambios.  

   <pre><code>
    sudo -u <b>hn1</b>adm /usr/sap/hostctrl/exe/sapcontrol -nr <b>03</b> -function StopSystem HDB
    sudo -u <b>hn1</b>adm /usr/sap/hostctrl/exe/sapcontrol -nr <b>03</b> -function StartSystem HDB
   </code></pre>

5. **[1]** Compruebe que la interfaz de cliente utilizará las direcciones IP de la subred `client` para la comunicación.  

   <pre><code>
    # Execute as hn1adm
    /usr/sap/HN1/HDB03/exe/hdbsql -u SYSTEM -p "<b>password</b>" -i 03 -d SYSTEMDB 'select * from SYS.M_HOST_INFORMATION'|grep net_publicname
    # Expected result
    "<b>hanadb3</b>","net_publicname","<b>10.9.1.7</b>"
    "<b>hanadb2</b>","net_publicname","<b>10.9.1.6</b>"
    "<b>hanadb1</b>","net_publicname","<b>10.9.1.5</b>"
   </code></pre>

   Para más información sobre cómo comprobar la configuración, consulte la nota de SAP [2183363 - Configuration of SAP HANA internal network](https://launchpad.support.sap.com/#/notes/2183363) (Configuración de la red interna de SAP HANA).  

5. **[A]** Vuelva a habilitar el firewall.  
   - Detener HANA
       <pre><code>
        sudo -u <b>hn1</b>adm /usr/sap/hostctrl/exe/sapcontrol -nr <b>03</b> -function StopSystem HDB
       </code></pre>
   - Volver a habilitar el firewall
       <pre><code>
        # Execute as root
        systemctl start firewalld
        systemctl enable firewalld
       </code></pre>

   - Abrir los puertos del firewall necesarios

       > [!IMPORTANT]
       > Cree reglas de firewall para permitir la comunicación entre nodos de HANA y el tráfico del cliente. Los puertos necesarios se enumeran en [TCP/IP Ports of All SAP Products](https://help.sap.com/viewer/ports) (Puertos TCP/IP de todos los productos de SAP). Los siguientes comandos son solo un ejemplo. En este escenario con el número de sistema usado 03.

       <pre><code>
        # Execute as root
        sudo firewall-cmd --zone=public --add-port=30301/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30301/tcp
        sudo firewall-cmd --zone=public --add-port=30303/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30303/tcp
        sudo firewall-cmd --zone=public --add-port=30306/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30306/tcp
        sudo firewall-cmd --zone=public --add-port=30307/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30307/tcp
        sudo firewall-cmd --zone=public --add-port=30313/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30313/tcp
        sudo firewall-cmd --zone=public --add-port=30315/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30315/tcp
        sudo firewall-cmd --zone=public --add-port=30317/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30317/tcp
        sudo firewall-cmd --zone=public --add-port=30340/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30340/tcp
        sudo firewall-cmd --zone=public --add-port=30341/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30341/tcp
        sudo firewall-cmd --zone=public --add-port=30342/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30342/tcp
        sudo firewall-cmd --zone=public --add-port=1128/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=1128/tcp
        sudo firewall-cmd --zone=public --add-port=1129/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=1129/tcp
        sudo firewall-cmd --zone=public --add-port=40302/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40302/tcp
        sudo firewall-cmd --zone=public --add-port=40301/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40301/tcp
        sudo firewall-cmd --zone=public --add-port=40307/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40307/tcp
        sudo firewall-cmd --zone=public --add-port=40303/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40303/tcp
        sudo firewall-cmd --zone=public --add-port=40340/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=40340/tcp
        sudo firewall-cmd --zone=public --add-port=50313/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=50313/tcp
        sudo firewall-cmd --zone=public --add-port=50314/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=50314/tcp
        sudo firewall-cmd --zone=public --add-port=30310/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30310/tcp
        sudo firewall-cmd --zone=public --add-port=30302/tcp --permanent
        sudo firewall-cmd --zone=public --add-port=30302/tcp
       </code></pre>

   - Iniciar HANA
       <pre><code>
        sudo -u <b>hn1</b>adm /usr/sap/hostctrl/exe/sapcontrol -nr <b>03</b> -function StartSystem HDB
       </code></pre>

6. Para optimizar SAP HANA para el almacenamiento de Azure NetApp Files subyacente, establezca los siguientes parámetros de SAP HANA:

   - `max_parallel_io_requests` **128**
   - `async_read_submit`**activado**
   - `async_write_submit_active`**activado**
   - `async_write_submit_blocks` **todo**

   Para obtener más información, consulte [Aplicaciones de NetApp SAP en Microsoft Azure que usan Azure NetApp Files][anf-sap-applications-azure]. 

   A partir de los sistemas SAP HANA 2.0, puede establecer los parámetros en `global.ini`. Para más información, consulte la nota de SAP [1999930](https://launchpad.support.sap.com/#/notes/1999930).  
   
   En el caso de los sistemas SAP HANA 1.0, en las versiones SPS12 y anteriores, estos parámetros se pueden establecer durante la instalación, tal y como se describe en la nota de SAP [2267798](https://launchpad.support.sap.com/#/notes/2267798).  

7. El almacenamiento que usa Azure NetApp Files tiene un límite de tamaño de archivo de 16 terabytes (TB). SAP HANA no es consciente implícitamente de la limitación de almacenamiento y no creará automáticamente un nuevo archivo de datos cuando se alcance el límite de tamaño de archivo de 16 TB. Cuando SAP HANA intente aumentar el tamaño del archivo más allá de 16 TB, se producirán errores y finalmente se bloqueará el servidor de índice. 

   > [!IMPORTANT]
   > Para evitar que SAP HANA intente aumentar los archivos de datos más allá del [límite de 16 TB](../../../azure-netapp-files/azure-netapp-files-resource-limits.md) del subsistema de almacenamiento, establezca los parámetros siguientes en `global.ini`.  
   > - datavolume_striping = true
   > - datavolume_striping_size_gb = 15000 Para más información, consulte la nota de SAP [2400005](https://launchpad.support.sap.com/#/notes/2400005).
   > Tenga en cuenta la nota de SAP [2631285](https://launchpad.support.sap.com/#/notes/2631285). 

## <a name="test-sap-hana-failover"></a>Prueba de la conmutación por error de SAP HANA 

1. Haga una simulación del bloqueo de nodos en un nodo de trabajo de SAP HANA. Haga lo siguiente: 

   a. Antes de simular el bloqueo del nodo, ejecute los siguientes comandos como **hn1**adm para capturar el estado del entorno:  

   <pre><code>
    # Check the landscape status
    python /usr/sap/HN1/HDB<b>03</b>/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | yes    | ignore |          |        |         0 |         0 | default  | default  | master 3   | slave      | standby     | standby     | standby | standby | default | -       |

    # Check the instance status
    sapcontrol -nr <b>03</b>  -function GetSystemInstanceList
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GREEN
   </code></pre>

   b. Para simular un bloqueo de nodo, ejecute el siguiente comando como raíz en el nodo de trabajo, que en este caso es **hanadb2**:  
   
   <pre><code>
    echo b > /proc/sysrq-trigger
   </code></pre>

   c. Supervise el sistema para la finalización de la conmutación por error. Cuando se haya completado la conmutación por error, capture el estado, que debería tener el aspecto siguiente:  

    <pre><code>
    # Check the instance status
    sapcontrol -nr <b>03</b>  -function GetSystemInstanceList
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GRAY
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GREEN
    # Check the landscape status
    python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | no     | info   |          |        |         2 |         0 | default  | default  | master 2   | slave      | worker      | standby     | worker  | standby | default | -       |
    | hanadb3 | yes    | info   |          |        |         0 |         2 | default  | default  | master 3   | slave      | standby     | slave       | standby | worker  | default | default |
   </code></pre>

   > [!IMPORTANT]
   > Cuando un nodo experimenta un estado de pánico del kernel, para evitar retrasos con la conmutación por error de SAP HANA, establezca `kernel.panic` en 20 segundos en *todas* las máquinas virtuales de HANA. La configuración está terminada en `/etc/sysctl`. Reinicie las máquinas virtuales para activar el cambio. Si no se realiza este cambio, la conmutación por error puede tardar 10 minutos o más cuando un nodo está experimentando un estado de pánico del kernel.  

2. Termine el servidor de nombres. Para ello, haga lo siguiente:

   a. Antes de la prueba, ejecute los siguientes comandos como **hn1**adm para comprobar el estado del entorno:  

   <pre><code>
    #Landscape status 
    python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | yes    | ignore |          |        |         0 |         0 | default  | default  | master 3   | slave      | standby     | standby     | standby | standby | default | -       |
    # Check the instance status
    sapcontrol -nr 03  -function GetSystemInstanceList
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GREEN
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
   </code></pre>

   b. Ejecute los siguientes comandos como **hn1**adm en el nodo principal activo, que en este caso es **hanadb1**:  

    <pre><code>
        hn1adm@hanadb1:/usr/sap/HN1/HDB03> HDB kill
    </code></pre>
    
    El nodo en espera **hanadb3** asumirá el control como nodo maestro. Este es el estado de los recursos una vez completada la prueba de conmutación por error:  

    <pre><code>
     # Check the instance status
     sapcontrol -nr 03  -function GetSystemInstanceList
     GetSystemInstanceList
     OK
     hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
     hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
     hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GREEN
     hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GRAY
     # Check the landscape status
     python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
     | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
     |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
     |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
     | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
     | hanadb1 | no     | info   |          |        |         1 |         0 | default  | default  | master 1   | slave      | worker      | standby     | worker  | standby | default | -       |
     | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
     | hanadb3 | yes    | info   |          |        |         0 |         1 | default  | default  | master 3   | master     | standby     | master      | standby | worker  | default | default |
    </code></pre>

   c. Reinicie la instancia de HANA en **hanadb1**, es decir, en la misma máquina virtual donde se ha terminado el servidor de nombres. El nodo **hanadb1** volverá a unirse al entorno y mantendrá su rol de nodo en espera.  

   <pre><code>
    hn1adm@hanadb1:/usr/sap/HN1/HDB03> HDB start
   </code></pre>

   Después de que se haya iniciado SAP HANA en **hanadb1**, espere el siguiente estado:  

   <pre><code>
    # Check the instance status
    sapcontrol -nr 03  -function GetSystemInstanceList
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GREEN
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    # Check the landscape status
    python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | no     | info   |          |        |         1 |         0 | default  | default  | master 1   | slave      | worker      | standby     | worker  | standby | default | -       |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | yes    | info   |          |        |         0 |         1 | default  | default  | master 3   | master     | standby     | master      | standby | worker  | default | default |
   </code></pre>

   d. Ahora, vuelva a terminar el servidor de nombres en el nodo maestro actualmente activo, es decir, en el nodo **hanadb3**.  
   
   <pre><code>
    hn1adm@hanadb3:/usr/sap/HN1/HDB03> HDB kill
   </code></pre>

   El nodo **hanadb1** reanudará el rol del nodo maestro. Una vez completada la prueba de conmutación por error, el estado tendrá el aspecto siguiente:

   <pre><code>
    # Check the instance status
    sapcontrol -nr 03  -function GetSystemInstanceList
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GRAY
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    # Check the landscape status
    python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | no     | ignore |          |        |         0 |         0 | default  | default  | master 3   | slave      | standby     | standby     | standby | standby | default | -       |
   </code></pre>

   e. Inicie SAP HANA en **hanadb3**, que estará preparado para funcionar como nodo en espera.  

   <pre><code>
    hn1adm@hanadb3:/usr/sap/HN1/HDB03> HDB start
   </code></pre>

   Una vez que SAP HANA se haya iniciado en **hanadb3**, el estado tiene el aspecto siguiente:  

   <pre><code>
    # Check the instance status
    sapcontrol -nr 03  -function GetSystemInstanceList & python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    GetSystemInstanceList
    OK
    hostname, instanceNr, httpPort, httpsPort, startPriority, features, dispstatus
    hanadb2, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    hanadb3, 3, 50313, 50314, 0.3, HDB|HDB_STANDBY, GREEN
    hanadb1, 3, 50313, 50314, 0.3, HDB|HDB_WORKER, GREEN
    # Check the landscape status
    python /usr/sap/HN1/HDB03/exe/python_support/landscapeHostConfiguration.py
    | Host    | Host   | Host   | Failover | Remove | Storage   | Storage   | Failover | Failover | NameServer | NameServer | IndexServer | IndexServer | Host    | Host    | Worker  | Worker  |
    |         | Active | Status | Status   | Status | Config    | Actual    | Config   | Actual   | Config     | Actual     | Config      | Actual      | Config  | Actual  | Config  | Actual  |
    |         |        |        |          |        | Partition | Partition | Group    | Group    | Role       | Role       | Role        | Role        | Roles   | Roles   | Groups  | Groups  |
    | ------- | ------ | ------ | -------- | ------ | --------- | --------- | -------- | -------- | ---------- | ---------- | ----------- | ----------- | ------- | ------- | ------- | ------- |
    | hanadb1 | yes    | ok     |          |        |         1 |         1 | default  | default  | master 1   | master     | worker      | master      | worker  | worker  | default | default |
    | hanadb2 | yes    | ok     |          |        |         2 |         2 | default  | default  | master 2   | slave      | worker      | slave       | worker  | worker  | default | default |
    | hanadb3 | no     | ignore |          |        |         0 |         0 | default  | default  | master 3   | slave      | standby     | standby     | standby | standby | default | -       |
   </code></pre>

## <a name="next-steps"></a>Pasos siguientes

* [Planeamiento e implementación de Azure Virtual Machines para SAP][planning-guide]
* [Implementación de Azure Virtual Machines para SAP][deployment-guide]
* [Implementación de DBMS de Azure Virtual Machines para SAP][dbms-guide]
* Para más información sobre cómo establecer la alta disponibilidad y planear la recuperación ante desastres de SAP HANA en VM de Azure, consulte [Alta disponibilidad de SAP HANA en Azure Virtual Machines (VM)][sap-hana-ha].
