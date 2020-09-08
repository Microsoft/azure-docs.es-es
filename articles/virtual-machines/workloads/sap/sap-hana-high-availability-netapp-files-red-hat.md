---
title: Alta disponibilidad del escalado vertical de SAP HANA con ANF en RHEL | Microsoft Docs
description: Establezca la alta disponibilidad de SAP HANA con ANF en máquinas virtuales de Azure.
services: virtual-machines-linux
documentationcenter: ''
author: rdeltcheva
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 08/11/2020
ms.author: radeltch
ms.openlocfilehash: 030677276fa077c06a95e7c677fec956b9c2a947
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88556197"
---
# <a name="high-availability-of-sap-hana-scale-up-with-azure-netapp-files-on-red-hat-enterprise-linux"></a>Alta disponibilidad del escalado vertical de SAP HANA con Azure NetApp Files en Red Hat Enterprise Linux

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

En este artículo se describe cómo configurar la replicación del sistema de SAP HANA en una implementación de escalado vertical, cuando los sistemas de archivos de HANA se montan mediante NFS con Azure NetApp Files (ANF). En las configuraciones de ejemplo y comandos de instalación, se utiliza el número de instancia **03** y el identificador del sistema de HANA **HN1**. La replicación de SAP HANA se realiza con un nodo principal y al menos uno secundario.

Si los pasos de este documento se marcan con los siguientes prefijos, el significado es el siguiente:

- **[A]** : El paso se aplica a todos los nodos.
- **[1]** : El paso solo se aplica al nodo 1.
- **[2]** : El paso solo se aplica al nodo 2.
 
Lea primero las notas y los documentos de SAP siguientes:

- Nota de SAP [1928533](https://launchpad.support.sap.com/#/notes/1928533), que incluye:
    - La lista de tamaños de máquina virtual de Azure que se admiten para la implementación del software de SAP.
    - Información importante sobre capacidad para los tamaños de máquina virtual de Azure.
    - El software de SAP admitido y combinaciones de sistema operativo y base de datos.
    - La versión del kernel de SAP necesaria para Windows y Linux en Microsoft Azure.
- La nota de SAP [2015553](https://launchpad.support.sap.com/#/notes/2015553) enumera los requisitos previos para las implementaciones de software de SAP admitidas por SAP en Azure.
- La nota de SAP [405827](https://launchpad.support.sap.com/#/notes/405827) enumera el sistema de archivos recomendado para el entorno de HANA.
- La nota de SAP [2002167](https://launchpad.support.sap.com/#/notes/2002167) recomienda la configuración del sistema operativo para Red Hat Enterprise Linux.
- La nota de SAP [2009879](https://launchpad.support.sap.com/#/notes/2009879) contiene las instrucciones de SAP HANA para Red Hat Enterprise Linux.
- La nota de SAP [2178632](https://launchpad.support.sap.com/#/notes/2178632) contiene información detallada sobre todas las métricas de supervisión notificadas para SAP en Azure.
- La nota de SAP [2191498](https://launchpad.support.sap.com/#/notes/2191498) incluye la versión de SAP Host Agent necesaria para Linux en Azure.
- La nota de SAP [2243692](https://launchpad.support.sap.com/#/notes/2243692) incluye información acerca de las licencias de SAP en Linux en Azure.
- La nota de SAP [1999351](https://launchpad.support.sap.com/#/notes/1999351) contiene más soluciones de problemas de la extensión de supervisión mejorada de Azure para SAP.
- La [wiki de la comunidad SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contiene todas las notas de SAP que se necesitan para Linux.
- [Planeación e implementación de Azure Virtual Machines para SAP en Linux][planning-guide]
- [Implementación de Azure Virtual Machines para SAP en Linux][deployment-guide]
- [Implementación de DBMS de Azure Virtual Machines para SAP en Linux][dbms-guide]
- [Replicación del sistema de SAP HANA en un clúster de Pacemaker](https://access.redhat.com/articles/3004101)
- Documentación general de RHEL
    - [Introducción al complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_overview/index)
    - [Administración del complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/index)
    - [Referencia del complemento de alta disponibilidad](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)
    - [Configuración de la replicación del sistema SAP HANA en el escalado vertical de un clúster de Pacemaker cuando los sistemas de archivos de HANA están en recursos compartidos de NFS](https://access.redhat.com/solutions/5156571)
- Documentación de RHEL específica para Azure:
    - [Directivas de compatibilidad para clústeres de alta disponibilidad RHEL: Microsoft Azure Virtual Machines como miembros del clúster](https://access.redhat.com/articles/3131341)
    - [Instalación y configuración de un clúster de alta disponibilidad de Red Hat Enterprise Linux 7.4 (y versiones posteriores) en Microsoft Azure](https://access.redhat.com/articles/3252491)
    - [Instalación de SAP HANA en Red Hat Enterprise Linux para su uso en Microsoft Azure](https://access.redhat.com/solutions/3193782)0
    - [Configuración de la replicación del sistema de escalado vertical de SAP HANA para clústeres de Pacemaker en recursos compartidos NFS](https://access.redhat.com/solutions/5156571)
- [Aplicaciones de NetApp SAP en Microsoft Azure que usan Azure NetApp Files](https://www.netapp.com/us/media/tr-4746.pdf)

## <a name="overview"></a>Información general

Tradicionalmente, en el entorno de escalado vertical todos los sistemas de archivos para SAP HANA se montan desde el almacenamiento local. La configuración de la alta disponibilidad de la replicación del sistema de SAP HANA en Red Hat Enterprise Linux se publica en la guía [Alta disponibilidad de SAP HANA en máquinas virtuales de Azure en Red Hat Enterprise Linux](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-high-availability-rhel)

Para lograr la alta disponibilidad de SAP HANA del sistema de escalado vertical en los recursos compartidos de NFS en [Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/), se necesita una configuración de recursos adicional en el clúster para que los recursos de HANA se recuperen cuando un nodo pierde el acceso a los recursos compartidos de NFS en ANF.  El clúster administra los montajes NFS, lo que le permite supervisar el estado de los recursos. Se aplican las dependencias entre los montajes del sistema de archivos y los recursos de SAP HANA.  

![Escalado vertical de alta disponibilidad de SAP HANA en ANF](./media/sap-hana-high-availability-rhel/sap-hana-scale-up-netapp-files-red-hat.png)

Los sistemas de archivos de SAP HANA se montan en recursos compartidos de NFS con Azure NetApp Files en cada nodo. Los sistemas de archivos /hana/data, /hana/log y /hana/shared son únicos para cada nodo. 

Montado en el nodo 1 (**hanadb1**)

- 10.32.2.4:/**hanadb1**-data-mnt00001 en /hana/data
- 10.32.2.4:/**hanadb1**-log-mnt00001 en /hana/log
- 10.32.2.4:/**hanadb1**-shared-mnt00001 en /hana/shared

Montado en el nodo 2 (**hanadb2**)

- 10.32.2.4:/**hanadb2**-data-mnt00001 en /hana/data
- 10.32.2.4:/**hanadb2**-log-mnt00001 en /hana/log
- 10.32.2.4:/**hanadb2**-shared-mnt00001 en /hana/shared

> [!NOTE]
> Los sistemas de archivos /hana/shared, /hana/data y /hana/log no se comparten entre los dos nodos. Cada nodo del clúster tiene sus propios sistemas de archivos independientes.   

En la configuración de la replicación del sistema de SAP HANA se usa un nombre de host virtual dedicado y direcciones IP virtuales. En Azure, se requiere un equilibrador de carga para usar una dirección IP virtual. En la lista siguiente se muestra la configuración del equilibrador de carga:

- Configuración de front-end: Dirección IP 10.32.0.10 para hn1-db
- Configuración de back-end: Se conecta a interfaces de red principales de todas las máquinas virtuales que deben formar parte de la Replicación del sistema de HANA.
- Puerto de sondeo: Puerto 62503
- Reglas de equilibrio de carga: 30313 TCP, 30315 TCP, 30317 TCP, 30340 TCP, 30341 TCP, 30342 TCP (si se utiliza Azure Load Balancer Básico)  

## <a name="set-up-the-azure-netapp-file-infrastructure"></a>Configuración de la infraestructura de Azure NetApp Files

Antes de continuar con la configuración de la infraestructura de Azure NetApp Files, familiarícese con la [documentación correspondiente](https://docs.microsoft.com/azure/azure-netapp-files/).

Azure NetApp Files está disponible en varias [regiones de Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp). Compruebe si la región de Azure seleccionada ofrece Azure NetApp Files.

Para obtener información sobre la disponibilidad de Azure NetApp Files según la región de Azure, consulte [Disponibilidad de Azure NetApp Files por región de Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all).

Antes de implementar Azure NetApp Files, solicite la incorporación a este servicio de acuerdo con las [instrucciones para registrarse en Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register).

### <a name="deploy-azure-netapp-files-resources"></a>Implementación de recursos de Azure NetApp Files

En las siguientes instrucciones se supone que ya ha implementado la [red virtual de Azure](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview). Los recursos de Azure NetApp Files y las VM en las que esos recursos se montarán se deben implementar en la misma red virtual de Azure o en redes virtuales de Azure emparejadas.

1. Si aún no implementó los recursos, solicite la [incorporación a Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-register).

2. Cree una cuenta de NetApp en la región de Azure seleccionada. Para ello, siga las instrucciones en [Creación de una cuenta de NetApp](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-create-netapp-account).

3.  Configure el grupo de capacidad de Azure NetApp Files según disponibles en [Configuración en un grupo de capacidad de Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-set-up-capacity-pool).

    La arquitectura de HANA que se presenta en este artículo utiliza un único grupo de capacidad de Azure NetApp Files, el nivel de servicio *Ultra*. Para las cargas de trabajo HANA en Azure, se recomienda usar el [nivel de servicio](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-service-levels) *Ultra* o *Premium* de Azure NetApp Files.

4.  Delegue una subred en Azure NetApp Files tal como se describe en las instrucciones de [Delegación de una subred en Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-delegate-subnet).

5.  Implemente los volúmenes de Azure NetApp Files según las instrucciones de [Creación de un volumen de NFS para Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-create-volumes).

    Cuando vaya a implementar los volúmenes, asegúrese de seleccionar la versión NFSv4.1. Implemente los volúmenes en la subred de Azure NetApp Files designada. Las direcciones IP de los volúmenes de Azure NetApp se asignan automáticamente.

    Tenga en cuenta que los recursos de Azure NetApp Files y las VM de Azure deben estar en la misma red virtual de Azure o en redes virtuales de Azure emparejadas. Por ejemplo, hanadb1-data-mnt00001, hanadb1-log-mnt00001, etc. son los nombres de los volúmenes y nfs://10.32.2.4/hanadb1-data-mnt00001, nfs://10.32.2.4/hanadb1-log-mnt00001, etc. son las rutas de acceso de archivo de los volúmenes de Azure NetApp Files.
    
    En **hanadb1**

    - Volumen hanadb1-data-mnt00001 (nfs://10.32.2.4:/hanadb1-data-mnt00001) 
    - Volumen hanadb1-log-mnt00001 (nfs://10.32.2.4:/hanadb1-log-mnt00001)
    - Volumen hanadb1-shared-mnt00001 (nfs://10.32.2.4:/hanadb1-shared-mnt00001)
    
    En **hanadb2**

    - Volumen hanadb2-data-mnt00001 (nfs://10.32.2.4:/hanadb2-data-mnt00001)
    - Volumen hanadb2-log-mnt00001 (nfs://10.32.2.4:/hanadb2-log-mnt00001)
    - Volumen hanadb2-shared-mnt00001 (nfs://10.32.2.4:/hanadb2-shared-mnt00001)

### <a name="important-considerations"></a>Consideraciones importantes

A medida que se crea la instancia de Azure NetApp Files para los sistemas de escalado vertical de SAP HANA, tenga en cuenta lo siguiente:

- El grupo de capacidad mínima es 4 tebibytes (TiB).
- El tamaño del volumen mínimo es de 100 gigabytes (GiB).
- Azure NetApp Files y todas las máquinas virtuales en las que los volúmenes de Azure NetApp Files se montarán se deben implementar en la misma red virtual de Azure o en [redes virtuales emparejadas](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview) de la misma región.
- La red virtual seleccionada debe tener una subred delegada en Azure NetApp Files.
- El rendimiento de un volumen de Azure NetApp Files es una función de la cuota del volumen y del nivel de servicio, como se documenta en [Nivel de servicio para Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-service-levels). Al ajustar el tamaño de los volúmenes de Azure NetApp de HANA, asegúrese de que el rendimiento resultante cumpla los requisitos del sistema HANA.
- Con la [directiva de exportación](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-configure-export-policy) de Azure NetApp Files, puede controlar los clientes permitidos y el tipo de acceso (lectura y escritura, solo lectura, etc.).
- La característica Azure NetApp Files no depende aún de la zona. En la actualidad, la característica no se implementa en todas las zonas de disponibilidad de una región de Azure. Tenga en cuenta las posibles implicaciones de latencia en algunas regiones de Azure.

> [!IMPORTANT]
> Para las cargas de trabajo de SAP HANA, una baja latencia resulta fundamental. Trabaje con su representante de Microsoft para asegurarse de que las máquinas virtuales y los volúmenes de Azure NetApp Files se implementan en ubicaciones cercanas entre sí.

### <a name="sizing-of-hana-database-on-azure-netapp-files"></a>Dimensionamiento de la base de datos HANA en Azure NetApp Files

El rendimiento de un volumen de Azure NetApp Files es una función del tamaño del volumen y del nivel de servicio, como se documenta en [Nivel de servicio para Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-service-levels).

Al diseñar la infraestructura de SAP en Azure, tenga en cuenta los requisitos mínimos de almacenamiento de SAP, que se traducen en características de rendimiento mínimas:

- Lectura y escritura en /hana/log de 250 megabytes por segundo (MB/s) con tamaños de E/S de 1 MB.
- Actividad de lectura de al menos 400 MB/s para /hana/data con tamaños de E/S de 16 MB y 64 MB.
- Actividad de escritura de al menos 250 MB/s para /hana/data con tamaños de E/S de 16 MB y 64 MB.

Los [límites de rendimiento de Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-service-levels) por 1 TiB de cuota de volumen son los siguientes:

- Capa de almacenamiento Premium: 64 MiB/s
- Capa de almacenamiento Ultra: 128 MiB/s

Para cumplir los requisitos de rendimiento mínimo de SAP para /hana/data y /hana/log, y de acuerdo con las directrices para /hana/shared, los tamaños recomendados serían los siguientes:

|    Volumen    | Tamaño de la capa de almacenamiento Premium | Tamaño de la capa de almacenamiento Ultra | Protocolo NFS admitido |
| :----------: | :--------------------------: | :------------------------: | :--------------------: |
|  /hana/log   |            4 TiB             |           2 TiB            |          v4.1          |
|  /hana/data  |           6,3 TiB            |          3,2 TiB           |          v4.1          |
| /hana/shared |           1 x RAM            |          1 x RAM           |          v3 o v4.1    |


> [!NOTE]
> Las recomendaciones de tamaño de Azure NetApp Files que indicadas aquí tienen como objetivo satisfacer los requisitos mínimos que SAP recomienda para sus proveedores de infraestructura. En escenarios reales de implementaciones de clientes y cargas de trabajo, es posible que estos tamaños no sean suficientes. Utilice estas recomendaciones como punto de partida y adáptelas en función de los requisitos de la carga de trabajo específica.

> [!TIP]
> Puede cambiar el tamaño de los volúmenes de Azure NetApp Files de manera dinámica, sin necesidad de *desmontar* los volúmenes, detener las máquinas virtuales ni detener SAP HANA. Este método permite la flexibilidad necesaria para satisfacer las demandas de rendimiento esperadas e imprevistas de la aplicación.

> [!NOTE]
> Todos los comandos para montar /hana/shared en este artículo se presentan para los volúmenes de /hana/shared de NFSv4.1.
> Si implementó los volúmenes /hana/shared como volúmenes de NFSv3, no olvide ajustar los comandos de montaje de /hana/shared para NFSv3.


## <a name="deploy-linux-virtual-machine-via-azure-portal"></a>Implementación de máquinas virtuales Linux en Azure Portal 

Primero deberá crear los volúmenes de Azure NetApp Files. Luego, siga estos pasos:

1.  Cree un grupo de recursos.
2.  Cree una red virtual.
3.  Cree un conjunto de disponibilidad. Establezca el dominio máximo de actualización.
4.  Cree un equilibrador de carga (interno). Se recomienda que sea un equilibrador de carga estándar.
    Seleccione la red virtual que creó en el paso 2.
5.  Cree la máquina virtual 1 (**hanadb1**). 
6.  Cree la máquina virtual 2 (**hanadb2**).  
7.  Al crear la máquina virtual, no agregaremos ningún disco, ya que todos nuestros puntos de montaje estarán en los recursos compartidos de NFS de Azure NetApp Files. 
8.  Si usa Standard Load Balancer, siga estos pasos de configuración:
    1.  Primero, cree un grupo de direcciones IP de front-end:
        1.  Abra el equilibrador de carga, seleccione **frontend IP pool** (Grupo de direcciones IP de front-end) y haga clic en **Agregar**.
        1.  Escriba el nombre del nuevo grupo de direcciones IP de front-end (por ejemplo, **hana-front-end**).
        1.  Establezca **Asignación** en **Estática** y escriba la dirección IP (por ejemplo, **10.32.0.10**).
        1.  Seleccione **Aceptar**.
        1.  Una vez creado el nuevo grupo de direcciones IP de front-end, anote la dirección IP del grupo.
    1.  A continuación, cree un grupo de back-end:
        1.  Abra el equilibrador de carga, seleccione **Grupos de back-end** y haga clic en **Agregar**.
        1.  Escriba el nombre del nuevo grupo de back-end (por ejemplo, **hana-backend**).
        1.  Seleccione **Agregar una máquina virtual**.
        1.  Seleccione **Máquina virtual**.
        1.  Seleccione las máquinas virtuales del clúster de SAP HANA y sus direcciones IP.
        1.  Seleccione **Agregar**.
    1.  A continuación, cree un sondeo de estado:
        1.  Abra el equilibrador de carga, seleccione **Sondeos de estado** y haga clic en **Agregar**.
        1.  Escriba el nombre del sondeo de estado nuevo (por ejemplo **hana-hp**).
        1.  Seleccione TCP como protocolo y el puerto 625**03**. Mantenga el valor de **Intervalo** en 5 y el valor de **Umbral incorrecto** en 2.
        1.  Seleccione **Aceptar**.
    1.  Luego cree las reglas de equilibrio de carga:
        1.  Abra el equilibrador de carga, seleccione **Reglas de equilibrio de carga** y haga clic en **Agregar**.
        1.  Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, **hana-lb**).
        1.  Seleccione la dirección IP de front-end, el grupo de back-end y el sondeo de estado que ha creado anteriormente (por ejemplo, **hana-frontend**, **hana-backend** y **hana-hp**).
        1.  Seleccione **Puertos HA**.
        1.  Aumente el **tiempo de espera de inactividad** a 30 minutos.
        1.  Asegúrese de **habilitar la dirección IP flotante**.
        1.  Seleccione **Aceptar**.

> [!NOTE] 
> Cuando las máquinas virtuales sin direcciones IP públicas se colocan en el grupo de back-end de Standard Load Balancer interno (sin dirección IP pública), no hay conectividad saliente de Internet, a menos que se realice una configuración adicional para permitir el enrutamiento a puntos de conexión públicos. Para obtener más información sobre cómo obtener conectividad saliente, vea [Conectividad de punto de conexión público para máquinas virtuales con Azure Standard Load Balancer en escenarios de alta disponibilidad de SAP](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-standard-load-balancer-outbound-connections).

9. Como alternativa, si el escenario dicta el uso de Basic Load Balancer, siga estos pasos de configuración:
    1.  Configure el equilibrador de carga. Primero, cree un grupo de direcciones IP de front-end:
        1.  Abra el equilibrador de carga, seleccione **frontend IP pool** (Grupo de direcciones IP de front-end) y haga clic en **Agregar**.
        1.  Escriba el nombre del nuevo grupo de direcciones IP de front-end (por ejemplo, **hana-front-end**).
        1.  Establezca **Asignación** en **Estática** y escriba la dirección IP (por ejemplo, **10.32.0.10**).
        1.  Seleccione **Aceptar**.
        1.  Una vez creado el nuevo grupo de direcciones IP de front-end, anote la dirección IP del grupo.
    1.  A continuación, cree un grupo de back-end:
        1.  Abra el equilibrador de carga, seleccione **Grupos de back-end** y haga clic en **Agregar**.
        1.  Escriba el nombre del nuevo grupo de back-end (por ejemplo, **hana-backend**).
        1.  Seleccione **Agregar una máquina virtual**.
        1.  Seleccione el conjunto de disponibilidad creado en el paso 3.
        1.  Seleccione las máquinas virtuales del clúster de SAP HANA
        1.  Seleccione **Aceptar**.
    1.  A continuación, cree un sondeo de estado:
        1.  Abra el equilibrador de carga, seleccione **Sondeos de estado** y haga clic en **Agregar**.
        1.  Escriba el nombre del sondeo de estado nuevo (por ejemplo **hana-hp**).
        1.  Seleccione **TCP** como protocolo y el puerto 625**03**. Mantenga el valor de **Intervalo** en 5 y el valor de **Umbral incorrecto** en 2.
        1.  Seleccione **Aceptar**.
    1.  Para SAP HANA 1.0, cree las reglas de equilibrio de carga:
        1.  Abra el equilibrador de carga, seleccione **Reglas de equilibrio de carga** y haga clic en **Agregar**.
        1.  Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, hana-lb-3**03**15).
        1.  Seleccione la dirección IP de front-end, el grupo de back-end y el sondeo de estado que creó anteriormente (por ejemplo, **hana-frontend**).
        1.  Mantenga el valor de **Protocolo** en **TCP** y escriba el puerto 3**03**15.
        1.  Aumente el **tiempo de espera de inactividad** a 30 minutos.
        1.  Asegúrese de **habilitar la dirección IP flotante**.
        1.  Seleccione **Aceptar**.
        1.  Repita estos pasos para el puerto 3**03**17.
    1.  Para SAP HANA 2.0, cree reglas de equilibrio de carga para la base de datos del sistema:
        1.  Abra el equilibrador de carga, seleccione **Reglas de equilibrio de carga** y haga clic en **Agregar**.
        1.  Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, hana-lb-3**03**13).
        1.  Seleccione la dirección IP de front-end, el grupo de back-end y el sondeo de estado que creó anteriormente (por ejemplo, **hana-frontend**).
        1.  Mantenga el valor de **Protocolo** en **TCP** y escriba el puerto 3**03**13.
        1.  Aumente el **tiempo de espera de inactividad** a 30 minutos.
        1.  Asegúrese de **habilitar la dirección IP flotante**.
        1.  Seleccione **Aceptar**.
        1.  Repita estos pasos para el puerto 3**03**14.
    1.  Para SAP HANA 2.0, primero cree las reglas de equilibrio de carga para la base de datos de inquilino:
        1.  Abra el equilibrador de carga, seleccione **Reglas de equilibrio de carga** y haga clic en **Agregar**.
        1.  Escriba el nombre de la nueva regla del equilibrador de carga (por ejemplo, hana-lb-3**03**40).
        1.  Seleccione la dirección IP de front-end, el grupo de back-end y el sondeo de estado que creó anteriormente (por ejemplo, **hana-frontend**).
        1.  Mantenga el valor de **Protocolo** en **TCP** y escriba el puerto 3**03**40.
        1.  Aumente el **tiempo de espera de inactividad** a 30 minutos.
        1.  Asegúrese de **habilitar la dirección IP flotante**.
        1.  Seleccione **Aceptar**.
        1.  Repita estos pasos para los puertos 3**03**41 y 3**03**42.

Para más información sobre los puertos necesarios para SAP HANA, consulte el capítulo [Conexiones a las bases de datos de inquilino](https://help.sap.com/viewer/78209c1d3a9b41cd8624338e42a12bf6/latest/en-US/7a9343c9f2a2436faa3cfdb5ca00c052.html) de la guía [Bases de datos de inquilino de SAP HANA](https://help.sap.com/viewer/78209c1d3a9b41cd8624338e42a12bf6) o la nota de SAP [2388694](https://launchpad.support.sap.com/#/notes/2388694).

> [!IMPORTANT]
> No habilite las marcas de tiempo TCP en VM de Azure que se encuentren detrás de Azure Load Balancer. Si habilita las marcas de tiempo TCP provocará un error en los sondeos de estado. Establezca el parámetro **net.ipv4.tcp_timestamps** a **0**. Consulte [Sondeos de estado de Load Balancer](https://docs.microsoft.com/azure/load-balancer/load-balancer-custom-probe-overview) para obtener más información. Consulte también la nota de SAP [2382421](https://launchpad.support.sap.com/#/notes/2382421).

## <a name="mount-the-azure-netapp-files-volume"></a>Montaje de los volúmenes de Azure NetApp Files

1. **[A]** Cree puntos de montaje para los volúmenes de bases de datos HANA. 

    ```
    mkdir -p /hana/data
    mkdir -p /hana/log
    mkdir -p /hana/shared
    ```

2. **[A]** Compruebe la configuración del dominio NFS. Asegúrese de que el dominio esté configurado como dominio predeterminado de Azure NetApp Files, es decir, **defaultv4iddomain.com** y de que la asignación se haya establecido en **nobody**.

    ```
    sudo cat /etc/idmapd.conf
    # Example
    [General]
    Domain = defaultv4iddomain.com
    [Mapping]
    Nobody-User = nobody
    Nobody-Group = nobody
    ```

    > [!IMPORTANT]    
    > Asegúrese de establecer el dominio NFS en /etc/idmapd.conf en la máquina virtual para que coincida con la configuración de dominio predeterminada en Azure NetApp Files: **defaultv4iddomain.com**. Si hay alguna discrepancia entre la configuración de dominio del cliente NFS (es decir, la máquina virtual) y el servidor NFS (es decir la configuración de Azure NetApp), los permisos de archivos en volúmenes de Azure NetApp que estén montados en las máquinas virtuales se mostrarán como "nobody".
    

3. **[1]** Monte los volúmenes específicos del nodo en el nodo 1 (**hanadb1**). 

    ```
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb1-shared-mnt00001 /hana/shared
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb1-log-mnt00001 /hana/log
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb1-data-mnt00001 /hana/data
    ```
    
4.  **[2]** Monte los volúmenes específicos del nodo en el nodo 2 (**hanadb2**).
    
    ```
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb2-shared-mnt00001 /hana/shared
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb2-log-mnt00001 /hana/log
    sudo mount -o rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys 10.32.2.4:/hanadb2-data-mnt00001 /hana/data
    ```

5. **[A]** Compruebe que todos los volúmenes de HANA están montados con la versión del protocolo NFS NFSv4.

    ```
    sudo nfsstat -m
    
    # Verify that flag vers is set to 4.1 
    # Example from hanadb1
    
    /hana/log from 10.32.2.4:/hanadb1-log-mnt00001
    Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.32.0.4,local_lock=none,addr=10.32.2.4
    /hana/data from 10.32.2.4:/hanadb1-data-mnt00001
    Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.32.0.4,local_lock=none,addr=10.32.2.4
    /hana/shared from 10.32.2.4:/hanadb1-shared-mnt00001
    Flags: rw,noatime,vers=4.1,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.32.0.4,local_lock=none,addr=10.32.2.4
    ```

6. **[A]** Compruebe **nfs4_disable_idmapping**. Debe establecerse en **S**. Para crear la estructura de directorio en la que se encuentra **nfs4_disable_idmapping**, ejecute el comando mount. No podrá crear manualmente el directorio en /sys/modules, ya que el acceso está reservado para el kernel o los controladores.

    ```
    # Check nfs4_disable_idmapping 
    cat /sys/module/nfs/parameters/nfs4_disable_idmapping
    
    # If you need to set nfs4_disable_idmapping to Y
    echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping
    
    # Make the configuration permanent
    echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
    ```

   Para más información sobre cómo cambiar el parámetro **nfs_disable_idmapping**, consulte [https://access.redhat.com/solutions/1749883](https://access.redhat.com/solutions/1749883). 


## <a name="sap-hana-installation"></a>Instalación de SAP HANA

1. **[A]** Configuración de la resolución del nombre de todos los hosts.

   Puede usar un servidor DNS o modificar el archivo /etc/hosts en todos los nodos. En este ejemplo se muestra cómo usar el archivo /etc/hosts. Reemplace la dirección IP y el nombre de host en los siguientes comandos:

   ```
   sudo vi /etc/hosts
   # Insert the following lines in the /etc/hosts file. Change the IP address and hostname to match your environment  
   10.32.0.4   hanadb1
   10.32.0.5   hanadb2
   ```

2. **[A]** Configuración de RHEL para HANA

   Configure RHEL tal y como se describe en la nota de SAP siguiente según la versión de RHEL.

   - [2292690 - SAP HANA DB: Recommended OS settings for RHEL 7](https://launchpad.support.sap.com/#/notes/2292690) (2292690 - SAP HANA DB: configuraciones de sistema operativo recomendadas para RHEL 7)
   - [2777782 - SAP HANA DB: Recommended OS settings for RHEL 8](https://launchpad.support.sap.com/#/notes/2777782) (SAP HANA DB: configuración recomendada del sistema operativo para RHEL 8).
   - [2455582 - Linux: Running SAP applications compiled with GCC 6.x](https://launchpad.support.sap.com/#/notes/2455582) (Nota de compatibilidad de SAP n.º 2455582 - Linux: Ejecución de aplicaciones SAP compiladas con GCC 6.x)
   - [2593824 - Linux: Ejecución de aplicaciones SAP compiladas con GCC 7.x](https://launchpad.support.sap.com/#/notes/2593824) 
   - [2886607 - Linux: Ejecución de aplicaciones SAP compiladas con GCC 9.x](https://launchpad.support.sap.com/#/notes/2886607)

3. **[A]** Instalación de SAP HANA

   Iniciado con HANA 2.0 SPS 01, MDC es la opción predeterminada. Al instalar el sistema HANA, se crearán SYSTEMDB y un inquilino con el mismo SID. En algunos casos, no se quiere el inquilino predeterminado. En caso de que no desee crear un inquilino inicial junto con la instalación, puede seguir la nota de SAP [2629711](https://launchpad.support.sap.com/#/notes/2629711).

    Ejecute el programa **hdblcm** del DVD de HANA. Escriba los siguientes valores en el símbolo del sistema:  
    Elija la instalación: Escriba **1** (para la instalación).  
    Seleccione los componentes adicionales para la instalación: Especifique **1**.  
    Escriba la ruta de acceso de instalación [/hana/shared]: presione Entrar para aceptar el valor predeterminado.  
    Enter Local Host Name [..]: Presione ENTRAR para aceptar el valor predeterminado.  
    ¿Desea agregar hosts adicionales al sistema? (s/n) [n]: **n**  
    Escriba el identificador del sistema de SAP HANA: Escriba **HN1**.  
    Escriba el número de instancia [00]: Escriba **03**.  
    Seleccione el modo de base de datos o escriba el índice [1]: presione Entrar para aceptar el valor predeterminado.  
    Seleccione el uso del sistema o especificar el índice [4]: escriba **4** (para personalizar).  
    Escriba la ubicación de los volúmenes de datos [/hana/data/HN1]: presione Entrar para aceptar el valor predeterminado.  
    Escriba la ubicación de los volúmenes de registro [/hana/log]: presione Entrar para aceptar el valor predeterminado.  
    ¿Restringir la asignación de memoria máxima? [n]: Presione ENTRAR para aceptar el valor predeterminado.  
    Escriba el nombre de host del certificado para el host '...' [...]: presione Entrar para aceptar el valor predeterminado.  
    Escriba la contraseña del usuario del agente de host de SAP (sapadm): Escriba la contraseña de usuario del agente de host.  
    Confirme la contraseña del usuario del agente de host de SAP (sapadm): Escriba de nuevo la contraseña de usuario del agente de host para confirmarla.  
    Escriba la contraseña del administrador del sistema (hn1adm): Escriba la contraseña de administrador del sistema.  
    Confirme la contraseña del administrador del sistema (hn1adm): Escriba de nuevo la contraseña de administrador del sistema para confirmarla.  
    Escribir directorio principal de administrador del sistema [/usr/sap/HN1/home]: presione Entrar para aceptar el valor predeterminado.  
    Escriba el shell de inicio de sesión de administrador del sistema [/bin/sh]: presione Entrar para aceptar el valor predeterminado.  
    Escriba el identificador de usuario del administrador del sistema [1001]: presione Entrar para aceptar el valor predeterminado.  
    Escriba el identificador del grupo de usuarios (sapsys) [79]: presione Entrar para aceptar el valor predeterminado.  
    Escriba la contraseña del usuario (SYSTEM) de la base de datos: Escriba la contraseña de usuario de base de datos.  
    Confirme la contraseña del usuario (SYSTEM) de la base de datos: Escriba de nuevo la contraseña de usuario de base de datos para confirmarla.  
    ¿Reiniciar el sistema tras el reinicio de la máquina? [n]: Presione ENTRAR para aceptar el valor predeterminado.  
    ¿Desea continuar? (s/n): valide el resumen. Escriba **s** para continuar.  

4. **[A]** Actualice el agente de host de SAP

   Descargue el archivo más reciente del agente de host de SAP desde [SAP Software Center](https://launchpad.support.sap.com/#/softwarecenter) y ejecute el siguiente comando para actualizar el agente. Reemplace la ruta de acceso al archivo para que apunte al archivo que descargó:

   ```
   sudo /usr/sap/hostctrl/exe/saphostexec -upgrade -archive <path to SAP Host Agent SAR>
   ```

5. **[A]** Configuración del firewall

   Cree la regla de firewall para el puerto de sondeo del equilibrador de carga de Azure.

   ```
   sudo firewall-cmd --zone=public --add-port=62503/tcp
   sudo firewall-cmd --zone=public --add-port=62503/tcp –permanent
   ```

## <a name="configure-sap-hana-system-replication"></a>Configure SAP HANA system replication (Configuración de la replicación del sistema de SAP HANA)

Siga los pasos descritos en [Configuración de la Replicación del sistema de SAP HANA 2.0](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-high-availability-rhel#configure-sap-hana-20-system-replication) para configurar la replicación del sistema de SAP HANA. 

## <a name="cluster-configuration"></a>Configuración del clúster

En esta sección se describen los pasos necesarios para que el clúster funcione sin problemas cuando SAP HANA se instala en recursos compartidos de NFS con Azure NetApp Files. 

### <a name="create-a-pacemaker-cluster"></a>Creación de un clúster de Pacemaker

Siga los pasos de [Configuración de Pacemaker en Red Hat Enterprise Linux en Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-rhel-pacemaker) para crear un clúster de Pacemaker básico para este servidor HANA.

### <a name="configure-filesystem-resources"></a>Configuración de recursos del sistema de archivos

En este ejemplo, cada nodo de clúster tiene sus propios sistemas de archivos NFS de HANA /hana/shared, /hana/data y /hana/log.   

1. **[1]** Ponga el clúster en modo de mantenimiento.

   ```
   pcs property set maintenance-mode=true
   ```

2. **[1]** Cree los recursos del sistema de archivos para los montajes **hanadb1**.

    ```
    pcs resource create hana_data1 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb1-data-mnt00001 directory=/hana/data fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb1_nfs
    pcs resource create hana_log1 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb1-log-mnt00001 directory=/hana/log fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb1_nfs
    pcs resource create hana_shared1 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb1-shared-mnt00001 directory=/hana/shared fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb1_nfs
    ```

3. **[2]** Cree los recursos del sistema de archivos para los montajes **hanadb2**.

    ```
    pcs resource create hana_data2 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb2-data-mnt00001 directory=/hana/data fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb2_nfs
    pcs resource create hana_log2 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb2-log-mnt00001 directory=/hana/log fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb2_nfs
    pcs resource create hana_shared2 ocf:heartbeat:Filesystem device=10.32.2.4:/hanadb2-shared-mnt00001 directory=/hana/shared fstype=nfs options=rw,vers=4,minorversion=1,hard,timeo=600,rsize=262144,wsize=262144,intr,noatime,lock,_netdev,sec=sys op monitor interval=20s on-fail=fence timeout=40s OCF_CHECK_LEVEL=20 --group hanadb2_nfs
    ```

    El atributo `OCF_CHECK_LEVEL=20` se agrega a la operación de supervisión para que cada monitor realice una prueba de lectura o escritura en el sistema de archivos. Sin este atributo, la operación de supervisión solo comprueba que el sistema de archivos esté montado. Esto puede ser un problema porque, cuando se pierde la conectividad, el sistema de archivos puede permanecer montado a pesar de que no se pueda tener acceso a él.

    El atributo `on-fail=fence` también se agrega a la operación de supervisión. Con esta opción, si se produce un error en la operación de supervisión en un nodo, ese nodo se delimita inmediatamente. Sin esta opción, el comportamiento predeterminado es detener todos los recursos que dependen del recurso con errores, reiniciar el recurso con errores y, después, iniciar todos los recursos que dependen del recurso con error. Este comportamiento no solo puede tardar mucho tiempo cuando un recurso de SAP Hana depende del recurso con errores, pero también se puede producir un error general. El recurso de SAP Hana no se puede detener correctamente si el servidor NFS que contiene los archivos ejecutables de HANA es inaccesible.

4. **[1]** Configuración de restricciones de ubicación

   Configure las restricciones de ubicación para asegurarse de que los recursos que administran montajes únicos de hanadb1 nunca se puedan ejecutar en hanadb2 y viceversa.

    ```
    pcs constraint location hanadb1_nfs rule score=-INFINITY resource-discovery=never \#uname eq hanadb2
    pcs constraint location hanadb2_nfs rule score=-INFINITY resource-discovery=never \#uname eq hanadb1
    ```

    Se establece la opción `resource-discovery=never` porque los montajes únicos de cada nodo comparten el mismo punto de montaje. Por ejemplo, `hana_data1` usa el punto de montaje `/hana/data` y `hana_data2` también usa `/hana/data` como punto de montaje. Esto puede producir un falso positivo para una operación de sondeo, cuando se comprueba el estado del recurso en el inicio del clúster y esto puede provocar un comportamiento de recuperación innecesario. Esto se puede evitar mediante el establecimiento de `resource-discovery=never`.

5. **[1]** Configuración de recursos de atributo

   Configure los recursos de atributo. Estos atributos se establecerán en true si todos los montajes NFS de un nodo (/hana/data, /hana/log y /hana/data) se montan y se establecerán en false en caso contrario.

   ```
   pcs resource create hana_nfs1_active ocf:pacemaker:attribute active_value=true inactive_value=false name=hana_nfs1_active
   pcs resource create hana_nfs2_active ocf:pacemaker:attribute active_value=true inactive_value=false name=hana_nfs2_active
   ```

6. **[1]** Configuración de restricciones de ubicación

   Configure restricciones de ubicación para asegurarse de que el recurso de atributo de hanadb1 nunca se ejecute en hanadb2 y viceversa.

   ```
   pcs constraint location hana_nfs1_active avoids hanadb2
   pcs constraint location hana_nfs2_active avoids hanadb1
   ```

7. **[1]** Creación de restricciones de ordenación

   Configure restricciones de ordenación para que los recursos de atributo de un nodo se inicien solo después de montar todos los montajes NFS del nodo.
   
    ```
    pcs constraint order hanadb1_nfs then hana_nfs1_active
    pcs constraint order hanadb2_nfs then hana_nfs2_active
    ```

   > [!TIP]
   > Si la configuración incluye sistemas de archivos, fuera del grupo `hanadb1_nfs` o `hanadb2_nfs`, incluya la opción `sequential=false`, de modo que no haya dependencias de ordenación entre los sistemas de archivos. Todos los sistemas de archivos deben iniciarse antes de `hana_nfs1_active`, pero no tienen que iniciarse en ningún orden en relación con los demás. Para más información, consulte [Configuración de la replicación del sistema SAP HANA en el escalado vertical de un clúster de Pacemaker cuando los sistemas de archivos de HANA están en recursos compartidos de NFS](https://access.redhat.com/solutions/5156571)

### <a name="configure-sap-hana-cluster-resources"></a>Configuración de recursos de clúster de SAP HANA

1. Siga los pasos descritos en [Creación de recursos de clúster de SAP HANA](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-high-availability-rhel#create-sap-hana-cluster-resources) para crear los recursos de SAP HANA en el clúster. Cuando se crean los recursos de SAP HANA, es necesario crear una restricción de regla de ubicación entre los recursos de SAP HANA y los archivos de sistema (montajes NFS).

2. **[1]** Configuración de restricciones entre los recursos de SAP HANA y los montajes NFS

   Las restricciones de la regla de ubicación se establecerán para que los recursos de SAP HANA se puedan ejecutar en un nodo solo si todos los montajes NFS del nodo están montados.

    ```
    pcs constraint location SAPHanaTopology_HN1_03-clone rule score=-INFINITY hana_nfs1_active ne true and hana_nfs2_active ne true
    pcs constraint location SAPHana_HN1_03-master rule score=-INFINITY hana_nfs1_active ne true and hana_nfs2_active ne true
    # Take the cluster out of maintenance mode
    sudo pcs property set maintenance-mode=false
    ```

   Compruebe el estado del clúster y todos los recursos.

    ```
    sudo pcs status
    
    Online: [ hanadb1 hanadb2 ]
    
    Full list of resources:
    
    rsc_hdb_azr_agt(stonith:fence_azure_arm):  Started hanadb1
    
    Resource Group: hanadb1_nfs
    hana_data1 (ocf::heartbeat:Filesystem):Started hanadb1
    hana_log1  (ocf::heartbeat:Filesystem):Started hanadb1
    hana_shared1   (ocf::heartbeat:Filesystem):Started hanadb1
    
    Resource Group: hanadb2_nfs
    hana_data2 (ocf::heartbeat:Filesystem):Started hanadb2
    hana_log2  (ocf::heartbeat:Filesystem):Started hanadb2
    hana_shared2   (ocf::heartbeat:Filesystem):Started hanadb2
    
    hana_nfs1_active   (ocf::pacemaker:attribute): Started hanadb1
    hana_nfs2_active   (ocf::pacemaker:attribute): Started hanadb2
    
    Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
    Started: [ hanadb1 hanadb2 ]
    
    Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
    Masters: [ hanadb1 ]
    Slaves: [ hanadb2 ]
    
    Resource Group: g_ip_HN1_03
    nc_HN1_03  (ocf::heartbeat:azure-lb):  Started hanadb1
    vip_HN1_03 (ocf::heartbeat:IPaddr2):   Started hanadb1
    ```

## <a name="test-the-cluster-setup"></a>Prueba de la configuración del clúster

En esta sección se describe cómo se puede probar la configuración. 

1. Antes de comenzar la prueba, asegúrese de que Pacemaker no tenga acciones con error (mediante pcs status), no haya restricciones de ubicación inesperadas (por ejemplo, restos de una prueba de migración) y que la replicación del sistema HANA se encuentre en estado de sincronización, por ejemplo con systemReplicationStatus:

    ```
    sudo su - hn1adm -c "python /usr/sap/HN1/HDB03/exe/python_support/systemReplicationStatus.py"
    ```

2. Compruebe la configuración del clúster para un escenario de error cuando un nodo pierde el acceso al recurso compartido de NFS (/hana/shared).

   Los agentes de recursos de SAP HANA dependen de los archivos binarios, almacenados en `/hana/shared` para realizar operaciones durante la conmutación por error. El sistema de archivos `/hana/shared` se monta en NFS en el escenario presentado.  
   Es difícil simular un error, en el que uno de los servidores pierde el acceso al recurso compartido NFS. Una prueba que se puede realizar consiste en volver a montar el sistema de archivos como de solo lectura.
   Este enfoque valida que el clúster pueda realizar la conmutación por error, si se pierde el acceso a `/hana/shared` en el nodo activo.     


   **Resultado previsto:** Al convertir a `/hana/shared` en un sistema de archivos de solo lectura, se producirá un error en el atributo `OCF_CHECK_LEVEL` del recurso `hana_shared1` que realiza la operación de lectura o escritura en el sistema de archivos, ya que no puede escribir nada en el sistema de archivos y realizará la conmutación por error de recursos de HANA.  Se espera el mismo resultado cuando el nodo de HANA pierde el acceso a los recursos compartidos de NFS. 

   Estado del recurso antes de iniciar la prueba:

    ```
    sudo pcs status
    # Example output
    Full list of resources:
     rsc_hdb_azr_agt        (stonith:fence_azure_arm):      Started hanadb1

     Resource Group: hanadb1_nfs
         hana_data1 (ocf::heartbeat:Filesystem):    Started hanadb1
         hana_log1  (ocf::heartbeat:Filesystem):    Started hanadb1
         hana_shared1       (ocf::heartbeat:Filesystem):    Started hanadb1

    Resource Group: hanadb2_nfs
         hana_data2 (ocf::heartbeat:Filesystem):    Started hanadb2
         hana_log2  (ocf::heartbeat:Filesystem):    Started hanadb2
         hana_shared2       (ocf::heartbeat:Filesystem):    Started hanadb2

     hana_nfs1_active       (ocf::pacemaker:attribute):     Started hanadb1
     hana_nfs2_active       (ocf::pacemaker:attribute):     Started hanadb2

     Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
         Started: [ hanadb1 hanadb2 ]

     Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
         Masters: [ hanadb1 ]
         Slaves: [ hanadb2 ]

     Resource Group: g_ip_HN1_03
         nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hanadb1
         vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hanadb1
    ```

   Puede colocar /hana/shared en modo de solo lectura en el nodo de clúster activo con el siguiente comando:

    ```
    sudo mount -o ro 10.32.2.4:/hanadb1-shared-mnt00001 /hana/shared
    ```

   hanadb1 se reiniciará o se desactivará en función de la acción establecida en stonith (`pcs property show stonith-action`).  Cuando el servidor (hanadb1) esté inactivo, el recurso de HANA se mueve a hanadb2. Puede comprobar el estado del clúster desde hanadb2.

    ```
    pcs status

    Full list of resources:

     rsc_hdb_azr_agt        (stonith:fence_azure_arm):      Started hanadb2

     Resource Group: hanadb1_nfs
         hana_data1 (ocf::heartbeat:Filesystem):    Stopped
         hana_log1  (ocf::heartbeat:Filesystem):    Stopped
         hana_shared1       (ocf::heartbeat:Filesystem):    Stopped

     Resource Group: hanadb2_nfs
         hana_data2 (ocf::heartbeat:Filesystem):    Started hanadb2
         hana_log2  (ocf::heartbeat:Filesystem):    Started hanadb2
         hana_shared2       (ocf::heartbeat:Filesystem):    Started hanadb2

     hana_nfs1_active       (ocf::pacemaker:attribute):     Stopped
     hana_nfs2_active       (ocf::pacemaker:attribute):     Started hanadb2

     Clone Set: SAPHanaTopology_HN1_03-clone [SAPHanaTopology_HN1_03]
         Started: [ hanadb2 ]
         Stopped: [ hanadb1 ]

     Master/Slave Set: SAPHana_HN1_03-master [SAPHana_HN1_03]
         Masters: [ hanadb2 ]
         Stopped: [ hanadb1 ]

     Resource Group: g_ip_HN1_03
         nc_HN1_03  (ocf::heartbeat:azure-lb):      Started hanadb2
         vip_HN1_03 (ocf::heartbeat:IPaddr2):       Started hanadb2
    ```

   Se recomienda probar exhaustivamente la configuración del clúster de SAP HANA, también mediante la pruebas descritas en [Prueba de la configuración del clúster](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-high-availability-rhel#test-the-cluster-setup).   