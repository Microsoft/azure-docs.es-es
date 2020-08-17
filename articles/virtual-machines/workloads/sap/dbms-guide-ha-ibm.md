---
title: Configurar IBM Db2 HADR en máquinas virtuales de (VM) de Azure | Microsoft Docs
description: Establezca la alta disponibilidad de IBM Db2 LUW en máquinas virtuales (VM) de Azure.
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: SAP
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/06/2020
ms.author: juergent
ms.openlocfilehash: 7d453fba37e62e8528ae7b4ea86d1604973b84a1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87051993"
---
# <a name="high-availability-of-ibm-db2-luw-on-azure-vms-on-suse-linux-enterprise-server-with-pacemaker"></a>Alta disponibilidad de IBM Db2 LUW en máquinas virtuales de Azure en SUSE Linux Enterprise Server con Pacemaker

IBM Db2 para Linux, UNIX y Windows (LUW) en la [configuración de alta disponibilidad y recuperación ante desastres (HADR) ](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_10.5.0/com.ibm.db2.luw.admin.ha.doc/doc/c0011267.html) consta de un nodo que ejecuta una instancia de base de datos principal y al menos un nodo que ejecuta una instancia de base de datos secundaria. Los cambios realizados en la instancia de base de datos principal se replican en una instancia de base de datos secundaria de manera sincrónica o asincrónica, según la configuración. 

En este artículo se describe cómo implementar y configurar las máquinas virtuales (VM) de Azure, instalar el marco del clúster e instalar IBM Db2 LUW con configuración de HADR. 

En el artículo no se describe cómo instalar y configurar IBM Db2 LUW con HADR o la instalación de software SAP. Para ayudarle a hacer estas tareas, proporcionamos referencias a los manuales de instalación de SAP e IBM. En este artículo nos centramos en los elementos que son específicos para el entorno de Azure. 

Las versiones de IBM Db2 admitidas son 10.5 y posteriores, tal como se documenta en la nota de SAP [1928533].

Antes de comenzar una instalación, consulte la documentación y notas SAP siguientes:

| Nota de SAP | Descripción |
| --- | --- |
| [1928533] | SAP applications on Azure: Supported Products and Azure VM types (Nota de SAP 1928533: Aplicaciones SAP en Azure: productos admitidos y tipos de máquina virtual de Azure) |
| [2015553] | SAP en Azure: Support Prerequisites (Nota de soporte técnico 2015553 de SAP en Microsoft Azure: requisitos previos de soporte técnico) |
| [2178632] | Key monitoring metrics for SAP on Azure (Métricas de supervisión clave para SAP en Azure) |
| [2191498] | SAP on Linux with Azure: Enhanced Monitoring (SAP en Linux con Azure: supervisión mejorada) |
| [2243692] | Linux on Azure (IaaS) VM: SAP license issues (Linux y máquinas virtuales de Microsoft Azure (IaaS): problemas de licencia de SAP) |
| [1984787] | SUSE LINUX Enterprise Server 12: Notas de instalación |
| [1999351] | Troubleshooting Enhanced Azure Monitoring for SAP (Solución de problemas de la supervisión mejorada de Azure para SAP) |
| [2233094] | DB6: SAP applications on Azure that use IBM Db2 for Linux, UNIX, and Windows - additional information (DB6: aplicaciones de SAP en Azure con IBM Db2 para Linux, UNIX y Windows: información adicional) |
| [1612105] | DB6: FAQ on Db2 with HADR (Preguntas más frecuentes sobre Db2 con HADR) |


| Documentación | 
| --- |
| [SAP Community Wiki](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) (wiki de la comunidad de SAP): tiene todas las notas de SAP necesarias para Linux |
| Guía [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver][planning-guide] |
| [Implementación de máquinas virtuales de Azure para SAP en Linux][deployment-guide] (este artículo) |
| Guía [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP][dbms-guide] |
| [Lista de comprobación de planeamiento e implementación de cargas de trabajo de SAP en Azure][azr-sap-plancheck] |
| [Guías de procedimientos recomendados de SUSE Linux Enterprise Server for SAP Applications 12 SP4][sles-for-sap-bp] |
| [SUSE Linux Enterprise High Availability Extension 12 SP4][sles-ha-guide] |
| [Implementación de DBMS de Azure Virtual Machines de IBM Db2 para carga de trabajo de SAP][dbms-db2] |
| [IBM Db2 HADR 11.1][db2-hadr-11.1] |
| [IBM Db2 HADR R 10.5][db2-hadr-10.5] |

## <a name="overview"></a>Información general
Para lograr alta disponibilidad, IBM Db2 LUW con HADR se instala en al menos dos máquinas virtuales de Azure, que se implementan en un [conjunto de disponibilidad de Azure](../../windows/tutorial-availability-sets.md) o entre [Azure Availability Zones](./sap-ha-availability-zones.md). 

En los gráficos siguientes se muestra una configuración de dos VM de Azure de servidor de base de datos. Ambas máquinas virtuales de Azure de servidor de base de datos tienen su propio almacenamiento conectado y están en funcionamiento. En HADR, una instancia de base de datos en una de las máquinas virtuales de Azure tiene el rol de la instancia principal. Todos los clientes se conectan a esta instancia principal. Todos los cambios en las transacciones de base de datos se almacenan localmente en el registro de transacciones de Db2. Dado que las entradas del registro de transacciones se almacenan localmente, los registros se transfieren a través de TCP/IP a la instancia de base de datos en el segundo servidor de base de datos, el servidor en espera o la instancia en espera. La instancia en espera actualiza la base de datos local al poner al día las entradas del registro de transacciones. De este modo, el servidor en espera se mantiene sincronizado con el servidor principal.

HADR es solo una funcionalidad de replicación. No tiene detección de errores ni capacidades de toma de control o conmutación por error. Una toma de control o transferencia al servidor en espera lo debe iniciar manualmente un administrador de base de datos. Para lograr una toma de control y detección de errores automáticas, puede usar la característica de agrupación en clústeres de Linux Pacemaker. Pacemaker supervisa las dos instancias de servidor de bases de datos. Cuando se bloquea la instancia del servidor de base de datos principal, Pacemaker inicia una toma de control de HADR *automática* por el servidor en espera. Pacemaker también garantiza que la dirección IP virtual se asigna al nuevo servidor principal.

![Introducción a la alta disponibilidad de IBM Db2](./media/dbms-guide-ha-ibm/ha-db2-hadr-lb.png)

Para que los servidores de aplicaciones de SAP se conecten a la base de datos principal, se necesita un nombre de host virtual y una dirección IP virtual. Si se produce una conmutación por error, los servidores de aplicaciones de SAP se conectarán a la nueva instancia de base de datos principal. En un entorno de Azure, se necesita [Azure Load Balancer](https://microsoft.sharepoint.com/teams/WAG/AzureNetworking/Wiki/Load%20Balancing.aspx) para usar una dirección IP virtual de la manera en que se necesita para HADR de IBM Db2. 

Para ayudarle a comprender por completo cómo IBM Db2 LUW con HADR y Pacemaker se incorporan a una configuración de sistema SAP de alta disponibilidad, en la imagen siguiente se presenta una visión general de una configuración de alta disponibilidad de un sistema SAP basado en la base de datos IBM Db2. En este artículo solo se cubre IBM Db2, pero se proporcionan referencias a otros artículos sobre cómo configurar otros componentes de un sistema SAP.

![Introducción al entorno completo de alta disponibilidad de IBM DB2](.//media/dbms-guide-ha-ibm/end-2-end-ha.png)


### <a name="high-level-overview-of-the-required-steps"></a>Introducción de alto nivel de los pasos necesarios
Para implementar una configuración de IBM Db2, deberá seguir estos pasos:

  + Planificar el entorno.
  + Implemente las VM.
  + Actualizar SUSE Linux y configurar los sistemas de archivos.
  + Instalar y configurar Pacemaker.
  + Instalar [NFS de alta disponibilidad][nfs-ha].
  + Instalar [ASCS/ERS en un clúster independiente][ascs-ha].
  + Instalar la base de datos de IBM Db2 con la opción Distributed/High Availability (Distribuida/alta disponibilidad) (SWPM).
  + Instalar y crear un nodo y una instancia de base de datos secundarios y configurar HADR.
  + Confirmar que HADR funciona.
  + Aplicar la configuración de Pacemaker para controlar IBM Db2.
  + Configurar Azure Load Balancer.
  + Instalar los servidores de aplicaciones principal y de diálogo.
  + Comprobar y adaptar la configuración de los servidores de aplicaciones SAP.
  + Realizar pruebas de toma de control y conmutación por error.



## <a name="plan-azure-infrastructure-for-hosting-ibm-db2-luw-with-hadr"></a>Planificar la infraestructura de Azure para hospedar IBM Db2 LUW con HADR

Complete el proceso de planificación antes de ejecutar la implementación. La planificación crea la base para la implementación de una configuración de Db2 con HADR en Azure. En la tabla siguiente se indican los elementos clave que deben formar parte de la planificación de IBM Db2 LUW (parte de la base de datos del entorno SAP):

| Tema | Descripción breve |
| --- | --- |
| Definir los grupos de recursos de Azure | Los grupos de recursos donde se implementa la máquina virtual, red virtual, Azure Load Balancer y otros recursos. Puede ser nuevo o existente. |
| Definición de red virtual o subred | Donde se implementan las VM para IBM Db2 y Azure Load Balancer. Puede ser existente o recién creado. |
| Máquinas virtuales que hospedan IBM Db2 LUW | Tamaño de VM, almacenamiento, redes, dirección IP. |
| Nombre de host virtual y dirección IP virtual de la base de datos IBM Db2| La dirección IP virtual o nombre de host que se usa para la conexión de servidores de aplicaciones SAP. **db-virt-hostname**, **db-virt-ip**. |
| Barreras de Azure | Barreras de Azure o barreras de SBD (altamente recomendado). Método para evitar situaciones de cerebro dividido. |
| VM SBD | Tamaño de máquina virtual SBD, almacenamiento, red. |
| Azure Load Balancer | Uso de la versión Basic o Standard (recomendado), puerto de sondeo para la base de datos Db2 (nuestra recomendación 62500) **probe-port**. |
| Resolución de nombres| Funcionamiento de la resolución de nombres en el entorno. Se recomienda el servicio DNS. Se puede usar el archivo de host local. |
    
Para obtener más información sobre Linux Pacemaker en Azure, consulte [Configuración de Pacemaker en SUSE Linux Enterprise Server en Azure](./high-availability-guide-suse-pacemaker.md).

## <a name="deployment-on-suse-linux"></a>Implementación en SUSE Linux

El agente de recursos para IBM Db2 LUW se incluye en SUSE Linux Enterprise Server for SAP Applications. Para obtener la configuración que se describe en este documento, debe usar SUSE Linux Enterprise Server for SAP Applications. Azure Marketplace contiene una imagen de SUSE Enterprise Server for SAP Applications 12 que puede usar para implementar nuevas máquinas virtuales de Azure. Tenga en cuenta los distintos modelos de soporte técnico o servicio que ofrece SUSE en Azure Marketplace al elegir una imagen de VM en Azure VM Marketplace. 

### <a name="hosts-dns-updates"></a>Hosts: Actualizaciones del servicio de nombres de dominio
Haga una lista de todos los nombres de host, incluidos los nombres de host virtual, y actualice los servidores DNS para habilitar la resolución correcta de dirección IP y nombres de host. Si un servidor DNS no existe o no se pueden actualizar y crear entradas DNS, deberá usar los archivos de host local de las VM individuales que forman parte de este escenario. Si usa entradas de archivos de host, asegúrese de que las entradas se apliquen a todas las VM del entorno de sistema SAP. Sin embargo, recomendamos que use el DNS que, idealmente, se extiende a Azure.


### <a name="manual-deployment"></a>Implementación manual

Asegúrese de que el SO seleccionado sea compatible con IBM/SAP para IBM Db2 LUW. La lista de versiones de SO admitidas para máquinas virtuales de Azure y las versiones de Db2 está disponible en la nota de SAP [1928533]. La lista de las versiones de SO por versión de Db2 individual está disponible en la matriz de disponibilidad de productos de SAP. Se recomienda un mínimo de SLES 12 SP4 debido a las mejoras de rendimiento relacionados con Azure en esta versión o versiones posteriores de SUSE Linux.

1. Cree o seleccione un grupo de recursos.
1. Cree o seleccione una red virtual y subred.
1. Cree un conjunto de disponibilidad de Azure o implemente una zona de disponibilidad.
    + Para el conjunto de disponibilidad, establezca el número máximo de dominios de actualización en 2.
1. Cree la máquina Virtual 1.
    + Use SLES para la imagen SAP de Azure Marketplace.
    + Seleccione el conjunto de disponibilidad de Azure que creó en el paso 3 o seleccione Zona de disponibilidad.
1.  Cree la máquina Virtual 2.
    + Use SLES para la imagen SAP de Azure Marketplace.
    + Seleccione el conjunto de disponibilidad de Azure que creó en el paso 3 o seleccione Zona de disponibilidad (no la misma zona que seleccionó en el paso 3).
1. Agregue discos de datos a las VM y luego compruebe la recomendación de una configuración de sistema de archivos disponible en el artículo [Implementación de DBMS de Azure Virtual Machines de IBM Db2 para carga de trabajo de SAP][dbms-db2].

## <a name="create-the-pacemaker-cluster"></a>Implementar un clúster de Pacemaker
    
Para crear un clúster de Pacemaker básico para este servidor IBM Db2, consulte  [Configuración de Pacemaker en SUSE Linux Enterprise Server en Azure][sles-pacemaker]. 

## <a name="install-the-ibm-db2-luw-and-sap-environment"></a>Instalar el entorno de SAP y IBM Db2 LUW

Antes de comenzar la instalación de un entorno SAP basado en IBM Db2 LUW, consulte la siguiente documentación:

+ Documentación de Azure
+ Documentación de SAP
+ Documentación de IBM

En la sección de introducción de este artículo se proporcionan vínculos a esta documentación.

Consulte los manuales de instalación de SAP sobre la instalación de aplicaciones basadas en NetWeaver en IBM Db2 LUW.

Encontrará las guías en el portal de ayuda de SAP mediante el [buscador de guías de instalación de SAP][sap-instfind].

Puede reducir el número de guías que se muestran en el portal al establecer los siguientes filtros:

- Quiero: "Instalar un nuevo sistema"
- Mi base de datos: "IBM Db2 para Linux, Unix y Windows"
- Filtros adicionales para las versiones de SAP NetWeaver, configuración de la pila o sistema operativo

### <a name="installation-hints-for-setting-up-ibm-db2-luw-with-hadr"></a>Sugerencias de instalación para la configuración de IBM Db2 LUW con HADR

Para configurar la instancia de base de datos de IBM Db2 LUW principal:

- Use la opción distribuida o de alta disponibilidad.
- Instale la instancia de SAP ASCS/ERS y base de datos.
- Realice una copia de seguridad de la base de datos recién instalada.


> [!IMPORTANT] 
> Anote el "puerto de comunicación de la base de datos" que se establece durante la instalación. Debe ser el mismo número de puerto para ambas instancias de base de datos.

Para configurar el servidor de base de datos en espera mediante el procedimiento de copia del sistema homogéneo de SAP, ejecute estos pasos:

1. Seleccione la opción **System copy** > **Target systems** > **Distributed** > **Database instance** (Copia del sistema > Sistemas de destino > Distribuida > Instancia de base de datos).
1. Como método de copia, seleccione **Homogeneous System** (Sistema homogéneo) para que pueda usar la copia de seguridad para restaurar una copia de seguridad en la instancia de servidor en espera.
1. Cuando llegue al paso de salida para restaurar la base de datos para la copia del sistema homogéneo, cierre el instalador. Restaure la base de datos a partir de una copia de seguridad del host principal. Todas las fases de instalación posteriores se han ejecutado en el servidor de base de datos principal.
1. Configure HADR para IBM Db2.

   > [!NOTE]
   > Para la instalación y configuración específicas de Azure y Pacemaker: Durante el procedimiento de instalación a través de SAP Software Provisioning Manager, hay una pregunta sobre la alta disponibilidad para IBM Db2 LUW:
   >+ No seleccione **IBM Db2 pureScale**.
   >+ No seleccione **Install IBM Tivoli System Automation for Multiplatforms** (Instalar IBM Tivoli System Automation para multiplataformas).
   >+ No seleccione **Generate cluster configuration files** (Generar archivos de configuración de clúster).

   Cuando usa un dispositivo SBD para Linux Pacemaker, establezca los siguientes parámetros de HADR de Db2:
   + Duración de la ventana de mismo nivel de HADR (segundos) (HADR_PEER_WINDOW) = 300  
   + Valor de tiempo de espera de HADR (HADR_TIMEOUT) = 60

   Cuando usa un agente de barrera de Azure Pacemaker, establezca los parámetros siguientes:
   + Duración de la ventana de mismo nivel de HADR (segundos) (HADR_PEER_WINDOW) = 900  
   + Valor de tiempo de espera de HADR (HADR_TIMEOUT) = 60

Recomendamos los parámetros anteriores según las pruebas iniciales de conmutación por error o toma de control. Es obligatorio de hacer pruebas para el correcto funcionamiento de la conmutación por error y toma de control con estos valores de parámetro. Dado que las configuraciones individuales pueden variar, es posible que los parámetros necesiten ajuste. 

> [!IMPORTANT]
> Específico de IBM Db2 con la configuración de HADR con inicio normal: antes de poder iniciar la instancia de base de datos principal, debe estar en marcha la instancia de base de datos secundario o en espera.

Para fines de demostración y los procedimientos que se describen en este artículo, la base de datos SID es **PTR**.

#### <a name="ibm-db2-hadr-check"></a>Comprobación de HADR de IBM Db2
Tras haber configurado HADR y el estado es PEER y CONNECTED en los nodos principal y en espera, realice la comprobación siguiente:

<pre><code>
Execute command as db2&lt;sid&gt; db2pd -hadr -db &lt;SID&gt;

#Primary output:
# Database Member 0 -- Database PTR -- Active -- Up 1 days 01:51:38 -- Date 2019-02-06-15.35.28.505451
# 
#                             <b>HADR_ROLE = PRIMARY
#                           REPLAY_TYPE = PHYSICAL
#                         HADR_SYNCMODE = NEARSYNC
#                            STANDBY_ID = 1
#                         LOG_STREAM_ID = 0
#                            HADR_STATE = PEER
#                            HADR_FLAGS = TCP_PROTOCOL
#                   PRIMARY_MEMBER_HOST = azibmdb02
#                      PRIMARY_INSTANCE = db2ptr
#                        PRIMARY_MEMBER = 0
#                   STANDBY_MEMBER_HOST = azibmdb01
#                      STANDBY_INSTANCE = db2ptr
#                        STANDBY_MEMBER = 0
#                   HADR_CONNECT_STATUS = CONNECTED</b>
#              HADR_CONNECT_STATUS_TIME = 02/05/2019 13:51:47.170561 (1549374707)
#           HEARTBEAT_INTERVAL(seconds) = 15
#                      HEARTBEAT_MISSED = 0
#                    HEARTBEAT_EXPECTED = 6137
#                 HADR_TIMEOUT(seconds) = 60
#         TIME_SINCE_LAST_RECV(seconds) = 13
#              PEER_WAIT_LIMIT(seconds) = 0
#            LOG_HADR_WAIT_CUR(seconds) = 0.000
#     LOG_HADR_WAIT_RECENT_AVG(seconds) = 0.000025
#    LOG_HADR_WAIT_ACCUMULATED(seconds) = 434.595
#                   LOG_HADR_WAIT_COUNT = 223713
# SOCK_SEND_BUF_REQUESTED,ACTUAL(bytes) = 0, 46080
# SOCK_RECV_BUF_REQUESTED,ACTUAL(bytes) = 0, 374400
#             PRIMARY_LOG_FILE,PAGE,POS = S0000280.LOG, 15571, 27902548040
#             STANDBY_LOG_FILE,PAGE,POS = S0000280.LOG, 15571, 27902548040
#                   HADR_LOG_GAP(bytes) = 0
#      STANDBY_REPLAY_LOG_FILE,PAGE,POS = S0000280.LOG, 15571, 27902548040
#        STANDBY_RECV_REPLAY_GAP(bytes) = 0
#                      PRIMARY_LOG_TIME = 02/06/2019 15:34:39.000000 (1549467279)
#                      STANDBY_LOG_TIME = 02/06/2019 15:34:39.000000 (1549467279)
#               STANDBY_REPLAY_LOG_TIME = 02/06/2019 15:34:39.000000 (1549467279)
#          STANDBY_RECV_BUF_SIZE(pages) = 2048
#              STANDBY_RECV_BUF_PERCENT = 0
#            STANDBY_SPOOL_LIMIT(pages) = 0
#                 STANDBY_SPOOL_PERCENT = NULL
#                    STANDBY_ERROR_TIME = NULL
#                  PEER_WINDOW(seconds) = 300
#                       PEER_WINDOW_END = 02/06/2019 15:40:25.000000 (1549467625)
#              READS_ON_STANDBY_ENABLED = N

#Secondary output:
# Database Member 0 -- Database PTR -- Standby -- Up 1 days 01:46:43 -- Date 2019-02-06-15.38.25.644168
# 
#                             <b>HADR_ROLE = STANDBY
#                           REPLAY_TYPE = PHYSICAL
#                         HADR_SYNCMODE = NEARSYNC
#                            STANDBY_ID = 0
#                         LOG_STREAM_ID = 0
#                            HADR_STATE = PEER
#                            HADR_FLAGS = TCP_PROTOCOL
#                   PRIMARY_MEMBER_HOST = azibmdb02
#                      PRIMARY_INSTANCE = db2ptr
#                        PRIMARY_MEMBER = 0
#                   STANDBY_MEMBER_HOST = azibmdb01
#                      STANDBY_INSTANCE = db2ptr
#                        STANDBY_MEMBER = 0
#                   HADR_CONNECT_STATUS = CONNECTED</b>
#              HADR_CONNECT_STATUS_TIME = 02/05/2019 13:51:47.205067 (1549374707)
#           HEARTBEAT_INTERVAL(seconds) = 15
#                      HEARTBEAT_MISSED = 0
#                    HEARTBEAT_EXPECTED = 6186
#                 HADR_TIMEOUT(seconds) = 60
#         TIME_SINCE_LAST_RECV(seconds) = 5
#              PEER_WAIT_LIMIT(seconds) = 0
#            LOG_HADR_WAIT_CUR(seconds) = 0.000
#     LOG_HADR_WAIT_RECENT_AVG(seconds) = 0.000023
#    LOG_HADR_WAIT_ACCUMULATED(seconds) = 434.595
#                   LOG_HADR_WAIT_COUNT = 223725
# SOCK_SEND_BUF_REQUESTED,ACTUAL(bytes) = 0, 46080
# SOCK_RECV_BUF_REQUESTED,ACTUAL(bytes) = 0, 372480
#             PRIMARY_LOG_FILE,PAGE,POS = S0000280.LOG, 15574, 27902562173
#             STANDBY_LOG_FILE,PAGE,POS = S0000280.LOG, 15574, 27902562173
#                   HADR_LOG_GAP(bytes) = 0
#      STANDBY_REPLAY_LOG_FILE,PAGE,POS = S0000280.LOG, 15574, 27902562173
#        STANDBY_RECV_REPLAY_GAP(bytes) = 155
#                      PRIMARY_LOG_TIME = 02/06/2019 15:37:34.000000 (1549467454)
#                      STANDBY_LOG_TIME = 02/06/2019 15:37:34.000000 (1549467454)
#               STANDBY_REPLAY_LOG_TIME = 02/06/2019 15:37:34.000000 (1549467454)
#          STANDBY_RECV_BUF_SIZE(pages) = 2048
#              STANDBY_RECV_BUF_PERCENT = 0
#            STANDBY_SPOOL_LIMIT(pages) = 0
#                 STANDBY_SPOOL_PERCENT = NULL
#                    STANDBY_ERROR_TIME = NULL
#                  PEER_WINDOW(seconds) = 300
#                       PEER_WINDOW_END = 02/06/2019 15:43:19.000000 (1549467799)
#              READS_ON_STANDBY_ENABLED = N
</code></pre>



## <a name="db2-pacemaker-configuration"></a>Configuración de Db2 Pacemaker

Cuando usa Pacemaker la conmutación por error automática en el caso de que haya un error en el nodo, deberá configurar las instancias de Db2 y Pacemaker, según corresponda. En esta sección se describe este tipo de configuración.

Los siguientes elementos tienen los siguiente prefijos:

- **[A]** : Aplicable a todos los nodos
- **[1]** : Aplicable solo al nodo 1 
- **[2]** : Aplicable solo al nodo 2

**[A]**  Requisitos previos para la configuración de Pacemaker:
1. Apague ambos servidores de bases de datos con el usuario db2\<sid> con db2stop.
1. Cambie el entorno de shell del usuario db2\<sid> por */bin/ksh*. Recomendamos que use la herramienta de Yast. 


### <a name="pacemaker-configuration"></a>Configuración de pacemaker

> [!IMPORTANT]
> Pruebas recientes han mostrado situaciones en las que netcat deja de responder a las solicitudes debido al trabajo pendiente y a su limitación para controlar solo una conexión. El recurso netcat deja de escuchar las solicitudes del equilibrador de carga de Azure y la dirección IP flotante deja de estar disponible.  
> En el caso de los clústeres de Pacemaker existentes, en el pasado se recomendaba reemplazar netcat por socat. Actualmente se recomienda usar el agente de recursos azure-lb, que forma parte de los agentes de recursos de paquetes, con los siguientes requisitos de versión de paquete:
> - En el caso de SLES 12 SP4/SP5, la versión debe ser, al menos, resource-agents-4.3.018.a7fb5035-3.30.1.  
> - Para SLES 15/15 SP1, la versión debe ser al menos resource-agents-4.3.0184.6ee15eb2-4.13.1.  
>
> Tenga en cuenta que el cambio requerirá un breve tiempo de inactividad.  
> En el caso de los clústeres de Pacemaker existentes, si la configuración ya se ha cambiado para usar socat, como se describe en [Protección de la detección del equilibrador de carga de Azure](https://www.suse.com/support/kb/doc/?id=7024128), no hay ningún requisito para cambiar inmediatamente al agente de recursos azure-lb.

**[1]**  Configuración de Pacemaker específico de HADR para IBM Db2:
<pre><code># Put Pacemaker into maintenance mode
sudo crm configure property maintenance-mode=true
</code></pre>

**[1]** Crear los recursos de IBM Db2:
<pre><code># Replace **bold strings** with your instance name db2sid, database SID, and virtual IP address/Azure Load Balancer.

sudo crm configure primitive rsc_Db2_db2ptr_<b>PTR</b> db2 \
        params instance="<b>db2ptr</b>" dblist="<b>PTR</b>" \
        op start interval="0" timeout="130" \
        op stop interval="0" timeout="120" \
        op promote interval="0" timeout="120" \
        op demote interval="0" timeout="120" \
        op monitor interval="30" timeout="60" \
        op monitor interval="31" role="Master" timeout="60"

# Configure virtual IP - same as Azure Load Balancer IP
sudo crm configure primitive rsc_ip_db2ptr_<b>PTR</b> IPaddr2 \
        op monitor interval="10s" timeout="20s" \
        params ip="<b>10.100.0.10</b>"

# Configure probe port for Azure load Balancer
sudo crm configure primitive rsc_nc_db2ptr_<b>PTR</b> azure-lb port=<b>62500</b>

sudo crm configure group g_ip_db2ptr_<b>PTR</b> rsc_ip_db2ptr_<b>PTR</b> rsc_nc_db2ptr_<b>PTR</b>

sudo crm configure ms msl_Db2_db2ptr_<b>PTR</b> rsc_Db2_db2ptr_<b>PTR</b> \
        meta target-role="Started" notify="true"

sudo crm configure colocation col_db2_db2ptr_<b>PTR</b> inf: g_ip_db2ptr_<b>PTR</b>:Started msl_Db2_db2ptr_<b>PTR</b>:Master

sudo crm configure order ord_db2_ip_db2ptr_<b>PTR</b> inf: msl_Db2_db2ptr_<b>PTR</b>:promote g_ip_db2ptr_<b>PTR</b>:start

sudo crm configure rsc_defaults resource-stickiness=1000
sudo crm configure rsc_defaults migration-threshold=5000
</code></pre>

**[1]** Iniciar los recursos de IBM Db2:
* Saque Pacemaker del modo de mantenimiento.
<pre><code># Put Pacemaker out of maintenance-mode - that start IBM Db2
sudo crm configure property maintenance-mode=false</pre></code>

**[1]**  Asegúrese de que el estado del clúster sea correcto y que todos los recursos se hayan iniciado. No importa en qué nodo se ejecutan en los recursos.
<pre><code>sudo crm status</code>

# <a name="2-nodes-configured"></a>2 nodes configured
# <a name="5-resources-configured"></a>5 resources configured

# <a name="online--azibmdb01-azibmdb02-"></a>Online: [ azibmdb01 azibmdb02 ]

# <a name="full-list-of-resources"></a>Full list of resources:

#  <a name="stonith-sbd----stonithexternalsbd-started-azibmdb02"></a>stonith-sbd    (stonith:external/sbd): Started azibmdb02
#  <a name="resource-group-g_ip_db2ptr_ptr"></a>Resource Group: g_ip_db2ptr_PTR
#      <a name="rsc_ip_db2ptr_ptr--ocfheartbeatipaddr2-------started-azibmdb02"></a>rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
#      <a name="rsc_nc_db2ptr_ptr--ocfheartbeatazure-lb------started-azibmdb02"></a>rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
#  <a name="masterslave-set-msl_db2_db2ptr_ptr-rsc_db2_db2ptr_ptr"></a>Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
#      <a name="masters--azibmdb02-"></a>Masters: [ azibmdb02 ]
#      <a name="slaves--azibmdb01-"></a>Slaves: [ azibmdb01 ]
</pre>

> [!IMPORTANT]
> Debe administrar la instancia de DB2 en clústeres de Pacemaker mediante las herramientas de Pacemaker. Si usa comandos de db2, como db2stop, Pacemaker detecta la acción como un error de recurso. Si va a realizar mantenimiento, puede colocar los nodos en modo de mantenimiento. Pacemaker suspende la supervisión de recursos y luego se pueden usar los comandos de administración normal de db2.


### <a name="configure-azure-load-balancer"></a>Configuración de Azure Load Balancer
Para configurar Azure Load Balancer, recomendamos usar la [SKU de Azure Standard Load Balancer](../../../load-balancer/load-balancer-overview.md) y luego hacer lo siguiente:

> [!NOTE]
> La SKU de Standard Load Balancer tiene restricciones para acceder a direcciones IP públicas desde los nodos situados debajo de Load Balancer. En el artículo [Conectividad del punto de conexión público para las máquinas virtuales que usan Azure Standard Load Balancer en escenarios de alta disponibilidad de SAP](./high-availability-guide-standard-load-balancer-outbound-connections.md) se describen las distintas formas para habilitar esos nodos y tener acceso a direcciones IP públicas.

1. Crear un grupo de IP front-end:

   a. En Azure Portal, abra Azure Load Balancer, seleccione **Grupo de IP de front-end** y luego seleccione **Agregar**.

   b. Escriba el nombre del nuevo grupo de IP front-end (por ejemplo, **Db2-connection**).

   c. Establezca la opción **Asignación** en **Estático** y en **IP Virtual** escriba la dirección IP definida al principio.

   d. Seleccione **Aceptar**.

   e. Una vez creado el nuevo grupo de direcciones IP de front-end, anote la dirección IP del grupo.

1. Cree un grupo de back-end:

   a. En Azure Portal, abra Azure Load Balancer, seleccione **grupos back-end** luego seleccione **Agregar**.

   b. Escriba el nombre del nuevo grupo de back-end (por ejemplo, **Db2-backend**).

   c. Seleccione **Agregar una máquina virtual**.

   d. Seleccione el conjunto de disponibilidad o las máquinas virtuales que hospedan la base de datos IBM Db2 creada en el paso anterior.

   e. Seleccione las máquinas virtuales del clúster de IBM Db2.

   f. Seleccione **Aceptar**.

1. Cree un sondeo de estado:

   a. En Azure Portal, abra Azure Load Balancer, seleccione **sondeos de estado** y luego seleccione **Agregar**.

   b. Escriba el nombre del nuevo sondeo de estado (por ejemplo, **Db2-hp**).

   c. Seleccione **TCP** como el protocolo y el puerto **62500**. Mantenga el valor **Intervalo** establecido en **5** y mantenga el valor de **Umbral incorrecto** establecido en **2**.

   d. Seleccione **Aceptar**.

1. Cree las reglas de equilibrio de carga:

   a. En Azure Portal, abra Azure Load Balancer, seleccione **Reglas de equilibrio de carga** y luego seleccione **Agregar**.

   b. Escriba el nombre de la nueva regla de equilibrador de carga (por ejemplo, **Db2-SID**).

   c. Seleccione la dirección IP de front-end, el grupo de back-end y el sondeo de estado que creó anteriormente (por ejemplo, **Db2-frontend**).

   d. Mantenga el valor de **Protocolo** establecido en **TCP** y especifique el *puerto de comunicación de la base de datos*.

   e. Aumente el **tiempo de espera de inactividad** a 30 minutos.

   f. Asegúrese de **habilitar la dirección IP flotante**.

   g. Seleccione **Aceptar**.


### <a name="make-changes-to-sap-profiles-to-use-virtual-ip-for-connection"></a>Hacer cambios en los perfiles de SAP para usar la dirección IP virtual para la conexión
Para conectarse a la instancia principal de la configuración de HADR, el nivel de aplicación de SAP debe usar la dirección IP virtual que definió y configuró para Azure Load Balancer. Se requieren los siguientes cambios:

/sapmnt/\<SID>/profile/DEFAULT.PFL
<pre><code>SAPDBHOST = db-virt-hostname
j2ee/dbhost = db-virt-hostname
</code></pre>

/sapmnt/\<SID>/global/db6/db2cli.ini
<pre><code>Hostname=db-virt-hostname
</code></pre>



## <a name="install-primary-and-dialog-application-servers"></a>Instalar los servidores de aplicaciones principal y de diálogo

Al instalar servidores de aplicaciones principal y de diálogo con una configuración de HADR de Db2, use el nombre de host virtual que eligió para la configuración. 

Si realizó la instalación antes de haber creado la configuración de HADR de Db2, realice los cambios según se describe en la sección anterior y del modo siguiente para pilas de Java de SAP.

### <a name="abapjava-or-java-stack-systems-jdbc-url-check"></a>Comprobación URL de JDBC para sistemas de pila de ABAP+Java o Java

Use la herramienta de configuración de J2EE para comprobar o actualizar la dirección URL de JDBC. Dado que la herramienta de configuración de J2EE es una herramienta gráfica, deberá tener X server instalado:
 
1. Inicie sesión en el servidor de aplicaciones principal de la instancia de J2EE y ejecute lo siguiente: `sudo /usr/sap/*SID*/*Instance*/j2ee/configtool/configtool.sh`
1. En el marco de la izquierda, seleccione **security store**.
1. En el marco de la derecha, elija la clave jdbc/pool/\<SAPSID>/url.
1. Cambie el nombre de host de la dirección URL de JDBC al nombre de host virtual.
     `jdbc:db2://db-virt-hostname:5912/TSP:deferPrepares=0`
1. Seleccione **Agregar**.
1. Para guardar los cambios, seleccione el icono de disco de la parte superior izquierda.
1. Cierre la herramienta de configuración.
1. Reinicie la instancia de Java.

## <a name="configure-log-archiving-for-hadr-setup"></a>Configurar el archivado de registros para la configuración de HADR
Para configurar el archivado de registros de Db2 para la configuración de HADR, recomendamos que configure tanto la base de datos principal como la base de datos en espera de modo que tengan capacidad de recuperación automático de registros de todas las ubicaciones de archivo de registro. Tanto la base de datos principal y como la base de datos en espera deben poder recuperar los archivos de registro de todas las ubicaciones de archivos de registro en las que cualquiera de las instancias de base de datos podría archivar este tipo de archivo. 

El archivado de registros solo lo realiza la base de datos principal. Si cambia los roles de HADR de los servidores de base de datos o si se produce un error, la nueva base de datos principal es responsable del archivado de registros. Si ha configurado varias ubicaciones de archivo de registro, es posible que los registros se archiven dos veces. Si se produce una actualización local o remota, también es posible que tenga que copiar manualmente los registros archivados desde el servidor principal anterior a la ubicación de registros activa del nuevo servidor principal.

Recomendamos configurar un recurso compartido NFS común en el que almacenar los registros de ambos nodos. El recurso compartido NFS debe tener alta disponibilidad. 

Puede usar recursos compartidos NFS de alta disponibilidad existentes para los transportes o un directorio de perfiles. Para más información, consulte:

- [Alta disponibilidad para NFS en máquinas virtuales de Azure en SUSE Linux Enterprise Server][nfs-ha] 
- [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server con Azure NetApp Files para las aplicaciones de SAP](./high-availability-guide-suse-netapp-files.md)
- [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) (para crear recursos compartidos de NFS)


## <a name="test-the-cluster-setup"></a>Prueba de la configuración del clúster

En esta sección se describe cómo se puede probar la configuración de HADR de Db2. *En todas las pruebas, se supone que se inició sesión como usuario raíz*  y que el servidor principal de IBM Db2 se ejecutan en la máquina virtual *azibmdb01*.

El estado inicial de todos los casos de prueba se explica aquí: (crm_mon -r o crm status)

- **crm status** es una instantánea del estado de Pacemaker en tiempo de ejecución 
- **crm_mon -r** es la salida continua del estado de Pacemaker

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Stopped
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Stopped
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     rsc_Db2_db2ptr_PTR      (ocf::heartbeat:db2):   Promoting azibmdb01
     Slaves: [ azibmdb02 ]
</code></pre>

El estado original en un sistema SAP se documenta en Transaction DBACOCKPIT > Configuration > Overview (Transacción DBACOCKPIT) > Configuración > Introducción, tal como se muestra en la siguiente imagen:

![DBACockpit - Pre Migration](./media/dbms-guide-ha-ibm/hadr-sap-mgr-org.png)




### <a name="test-takeover-of-ibm-db2"></a>Probar la toma de control de IBM Db2


> [!IMPORTANT] 
> Antes de iniciar la prueba, asegúrese de lo siguiente:
> * Pacemaker no tiene acciones con error (crm status).
> * No hay limitaciones de ubicación (restos de la prueba de migración).
> * La sincronización de HADR de IBM Db2 funciona. Comprobación con el usuario db2\<sid> <pre><code>db2pd -hadr -db \<DBSID></code></pre>


Migre el nodo que ejecuta la base de datos principal de Db2. Para ello, ejecute el comando siguiente:
<pre><code>crm resource migrate msl_<b>Db2_db2ptr_PTR</b> azibmdb02</code></pre>

Una vez finalizada la migración, la salida del estado de crm tendrá este aspecto:
<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Slaves: [ azibmdb01 ]
</code></pre>

El estado original en un sistema SAP se documenta en Transaction DBACOCKPIT > Configuration > Overview (Transacción DBACOCKPIT) > Configuración > Introducción, tal como se muestra en la siguiente imagen:

![DBACockpit: posterior a la migración](./media/dbms-guide-ha-ibm/hadr-sap-mgr-post.png)

la migración de recursos con "crm resource migrate" crea restricciones de ubicación. Las restricciones de ubicación se deben eliminar. Si no se eliminan las restricciones de ubicación, el recurso no se puede hacer la conmutación por error o se pueden producir tomas de control no deseadas. 

Vuelva a migrar el recurso a *azibmdb01* y elimine las restricciones de ubicación.
<pre><code>crm resource migrate msl_<b>Db2_db2ptr_PTR</b> azibmdb01
crm resource clear msl_<b>Db2_db2ptr_PTR</b>
</code></pre>

- **crm resource migrate \<res_name> \<host>:** crea restricciones de ubicación y puede causar problemas con la toma de control.
- **crm resource clear \<res_name>** : elimina las restricciones de ubicación.
- **crm resource cleanup \<res_name>** : elimina todos los errores del recurso.

### <a name="test-the-fencing-agent"></a>Probar el agente de barrera

En este caso, hacemos una prueba de la barrera de SBD, que recomendamos que haga cuando use SUSE Linux.

<pre><code>
azibmdb01:~ # ps -ef|grep sbd
root       2374      1  0 Feb05 ?        00:00:17 sbd: inquisitor
root       2378   2374  0 Feb05 ?        00:00:40 sbd: watcher: /dev/disk/by-id/scsi-36001405fbbaab35ee77412dacb77ae36 - slot: 0 - uuid: 27cad13a-0bce-4115-891f-43b22cfabe65
root       2379   2374  0 Feb05 ?        00:01:51 sbd: watcher: Pacemaker
root       2380   2374  0 Feb05 ?        00:00:18 sbd: watcher: Cluster

azibmdb01:~ # kill -9 2374
</code></pre>

El nodo de clúster *azibmdb01* se debe reiniciar. El rol HADR principal de IBM Db2 se va a mover a *azibmdb02*. Cuando *azibmdb01* vuelva a estar en línea, la instancia de Db2 se va a mover al rol de una instancia de base de datos secundaria. 

Si el servicio de Pacemaker no se inicia automáticamente en la anterior instancia principal reiniciada, asegúrese de iniciarla manualmente con:

<code><pre>sudo service pacemaker start</code></pre>

### <a name="test-a-manual-takeover"></a>Probar una toma de control manual

Puede probar una toma de control manual al detener el servicio de Pacemaker en el nodo *azibmdb01*:
<pre><code>service pacemaker stop</code></pre>

Estado en *azibmdb02*
<pre><code>
2 nodes configured
5 resources configured

Online: [ azibmdb02 ]
OFFLINE: [ azibmdb01 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Stopped: [ azibmdb01 ]
</code></pre>

Después de la conmutación por error, puede reiniciar el servicio en *azibmdb01*.
<pre><code>service pacemaker start</code></pre>


### <a name="kill-the-db2-process-on-the-node-that-runs-the-hadr-primary-database"></a>Detener el proceso de Db2 en el nodo que ejecuta la base de datos principal de HADR

<pre><code>#Kill main db2 process - db2sysc
azibmdb01:~ # ps -ef|grep db2s
db2ptr    34598  34596  8 14:21 ?        00:00:07 db2sysc 0

azibmdb01:~ # kill -9 34598
</code></pre>

Se producirá un error en la instancia de Db2 y Pacemaker notificará el siguiente estado:

<pre><code>
2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Stopped
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Stopped
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Slaves: [ azibmdb02 ]
     Stopped: [ azibmdb01 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_demote_0 on azibmdb01 'unknown error' (1): call=157, status=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:28:19 2019', queued=40ms, exec=223ms

</code></pre>

Pacemaker reiniciará la instancia de base de datos principal de Db2 en el mismo nodo o se hará la conmutación por error al nodo que ejecuta la instancia de base de datos secundaria y se notifica un error.

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_demote_0 on azibmdb01 'unknown error' (1): call=157, status=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:28:19 2019', queued=40ms, exec=223ms
</code></pre>


### <a name="kill-the-db2-process-on-the-node-that-runs-the-secondary-database-instance"></a>Detener el proceso de Db2 en el nodo que ejecuta la instancia de base de datos secundaria

<pre><code>azibmdb02:~ # ps -ef|grep db2s
db2ptr    65250  65248  0 Feb11 ?        00:09:27 db2sysc 0

azibmdb02:~ # kill -9</code></pre>

El nodo entra en un estado de error y el error se notifica.
<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     rsc_Db2_db2ptr_PTR      (ocf::heartbeat:db2):   FAILED azibmdb02
     Masters: [ azibmdb01 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_monitor_30000 on azibmdb02 'not running' (7): call=144, status=complete, exitreason='',
last-rc-change='Tue Feb 12 14:36:59 2019', queued=0ms, exec=0ms</code></pre>

La instancia de Db2 se reinicia en el rol secundario que tenía asignado anteriormente.

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_monitor_30000 on azibmdb02 'not running' (7): call=144, status=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:36:59 2019', queued=0ms, exec=0ms</code></pre>



### <a name="stop-db-via-db2stop-force-on-the-node-that-runs-the-hadr-primary-database-instance"></a>Detener la base de datos a través de db2stop force en el nodo que ejecuta la instancia de base de datos principal de HADR

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]</code></pre>

Como usuario db2\<sid>, ejecute el comando db2stop force:
<pre><code>azibmdb01:~ # su - db2ptr
azibmdb01:db2ptr> db2stop force</code></pre>

Se detectó un error.
<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

 stonith-sbd    (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Stopped
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Stopped
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     rsc_Db2_db2ptr_PTR      (ocf::heartbeat:db2):   FAILED azibmdb01
     Slaves: [ azibmdb02 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_demote_0 on azibmdb01 'unknown error' (1): call=201, status=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:45:25 2019', queued=1ms, exec=150ms</code></pre>

La instancia de base de datos secundaria de HADR de Db2 se promovió al rol principal.
<pre><code> nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb01
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Stopped: [ azibmdb01 ]

Failed Actions:
* rsc_Db2_db2ptr_PTR_start_0 on azibmdb01 'unknown error' (1): call=205, stat
us=complete, exitreason='',
    last-rc-change='Tue Feb 12 14:45:27 2019', queued=0ms, exec=865ms</pre></code>


### <a name="crash-vm-with-restart-on-the-node-that-runs-the-hadr-primary-database-instance"></a>Bloquear la VM con reinicio en el nodo que ejecuta la instancia de base de datos principal de HADR

<pre><code>#Linux kernel panic - with OS restart
azibmdb01:~ # echo b > /proc/sysrq-trigger</code></pre>

Pacemaker promoverá la instancia secundaria al rol de instancia principal. La instancia principal anterior pasará al rol secundario después de que la VM y todos los servicios se restauren completamente después del reinicio de la VM:

<pre><code> nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]</code></pre>



### <a name="crash-the-vm-that-runs-the-hadr-primary-database-instance-with-halt"></a>Bloquear la VM que ejecuta la instancia de base de datos principal de HADR mediante "halt"

<pre><code>#Linux kernel panic - halts OS
azibmdb01:~ # echo b > /proc/sysrq-trigger</code></pre>

En tal caso, Pacemaker detectará que el nodo que ejecuta la instancia de base de datos principal no responde.

<pre><code>2 nodes configured
5 resources configured

Node azibmdb01: UNCLEAN (online)
Online: [ azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb01
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb01
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb01 ]
     Slaves: [ azibmdb02 ]</code></pre>

El paso siguiente es comprobar para ver si existe una situación tipo *cerebro dividido*. Cuando el nodo superviviente haya determinado que el nodo que ejecutó por última vez la instancia de base de datos principal está inactivo, se ejecuta una conmutación por error de los recursos.
<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb02 ]
OFFLINE: [ azibmdb01 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Stopped: [ azibmdb01 ] </code></pre>


En el caso una "parada" del nodo, el nodo con error se debe reiniciar mediante las herramientas de administración de Azure (en Azure Portal, PowerShell o la CLI de Azure). Cuando el nodo con error vuelva a estar en línea, inicia la instancia de Db2 en el rol secundario.

<pre><code>2 nodes configured
5 resources configured

Online: [ azibmdb01 azibmdb02 ]

Full list of resources:

stonith-sbd     (stonith:external/sbd): Started azibmdb02
 Resource Group: g_ip_db2ptr_PTR
     rsc_ip_db2ptr_PTR  (ocf::heartbeat:IPaddr2):       Started azibmdb02
     rsc_nc_db2ptr_PTR  (ocf::heartbeat:azure-lb):      Started azibmdb02
 Master/Slave Set: msl_Db2_db2ptr_PTR [rsc_Db2_db2ptr_PTR]
     Masters: [ azibmdb02 ]
     Slaves: [ azibmdb01 ]</code></pre>

## <a name="next-steps"></a>Pasos siguientes
- [Escenarios y arquitectura de alta disponibilidad para SAP NetWeaver](./sap-high-availability-architecture-scenarios.md)
- [Configuración de Pacemaker en SUSE Linux Enterprise Server en Azure](./high-availability-guide-suse-pacemaker.md)

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2233094]:https://launchpad.support.sap.com/#/notes/2233094
[1612105]:https://launchpad.support.sap.com/#/notes/1612105

[sles-for-sap-bp]:https://www.suse.com/documentation/sles-for-sap-12/
[db2-hadr-11.1]:https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.admin.ha.doc/doc/c0011267.html
[db2-hadr-10.5]:https://www.ibm.com/support/knowledgecenter/en/SSEPGG_10.5.0/com.ibm.db2.luw.admin.ha.doc/doc/c0011267.html
[dbms-db2]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_ibm
[sles-pacemaker]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-suse-pacemaker
[sap-instfind]:https://help.sap.com/viewer/9e41ead9f54e44c1ae1a1094b0f80712/ALL/en-US/576f5c1808de4d1abecbd6e503c9ba42.html
[nfs-ha]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-suse-nfs
[sles-ha-guide]:https://www.suse.com/releasenotes/x86_64/SLE-HA/12-SP4/
[ascs-ha]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-suse

[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md
[azr-sap-plancheck]:https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-deployment-checklist
