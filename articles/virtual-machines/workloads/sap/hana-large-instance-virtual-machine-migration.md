---
title: Migración de SAP HANA en Azure (instancias grandes) a máquinas virtuales de Azure | Microsoft Docs
description: Procedimiento para migrar SAP HANA en Azure (instancias grandes) a máquinas virtuales de Azure
services: virtual-machines-linux
documentationcenter: ''
author: bentrin
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 02/11/2020
ms.author: bentrin
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: db51ec682f43366f5637c461e3fe4037dec8e364
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87085221"
---
# <a name="sap-hana-on-azure-large-instance-migration-to-azure-virtual-machines"></a>Migración de SAP HANA en instancias grandes de Azure a máquinas virtuales de Azure
En este artículo se describen posibles escenarios de implementación de instancias grandes de Azure, y se ofrece un enfoque de planeación y migración con un tiempo de inactividad de transición minimizado

## <a name="overview"></a>Información general
Desde el anuncio de SAP HANA en Azure (instancias grandes) (HLI), en septiembre de 2016, muchos clientes han adoptado este hardware como servicio para su plataforma de proceso en memoria.  En los últimos años, la extensión del tamaño de las VM de Azure, combinada con la compatibilidad de implementación de escalado horizontal de HANA, ha superado la demanda de capacidad de base de datos ERP de la mayoría de los clientes empresariales.  Empezamos a ver que los clientes expresan el interés de migrar su carga de trabajo de SAP HANA de servidores físicos a VM de Azure.
Esta guía no es un documento de configuración paso a paso. Aquí se describen los modelos de implementación comunes y se ofrecen sugerencias de planeación y migración.  La intención es destacar a las consideraciones necesarias para la preparación a fin de minimizar el tiempo de inactividad de la transición.

## <a name="assumptions"></a>Supuestos
En este artículo se da por supuesto lo siguiente:
- El único interés que se tiene en cuenta es una migración del servicio de proceso de base de datos de HANA homogénea de la Instancia grande de HANA (HLI) a VM de Azure sin necesidad de aplicar revisiones o actualizaciones de software significativas. Estas actualizaciones secundarias incluyen el uso de una versión de sistema operativo más reciente o una versión de HANA que se indica explícitamente como compatible con las notas de SAP pertinentes. 
- Es necesario realizar todas las actividades de actualización o actualización antes o después de la migración.  Por ejemplo, implementación de conversión de MCOS a MDC de SAP HANA. 
- El enfoque de migración que ofrecería el menor tiempo de inactividad es la replicación del sistema de SAP HANA. Otros métodos de migración no forman parte del ámbito de este documento.
- Esta guía se aplica tanto a las SKU de Rev3 como de Rev4 de HLI.
- En su mayor parte, la arquitectura de implementación de HANA permanece inalterada durante la migración.  Es decir, un sistema con una única instancia de DR seguirá siendo igual en el destino.
- Los clientes han revisado y comprendido el contrato de nivel de servicio (SLA) de la arquitectura de destino. 
- Los términos comerciales entre HLI y VM son diferentes. Los clientes deben supervisar el uso de sus VM para la administración de costos.
- Los clientes entienden que HLI es una plataforma de proceso dedicada, mientras que las VM se ejecutan en una infraestructura compartida, aunque aislada.
- Los clientes han validado que las VM de destino admiten la arquitectura deseada. Para ver todas las SKU de VM admitidas certificadas para la implementación de SAP HANA, consulte el [directorio de hardware de SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure).
- Los clientes han validado el diseño y el plan de migración.
- Planee la VM de recuperación ante desastres junto con el sitio primario.  Los clientes no pueden usar HLI como nodo DR para el sitio primario que se ejecuta en VM después de la migración.
- Los clientes copiaron los archivos de copia de seguridad necesarios en las VM de destino, en función de los requisitos de cumplimiento y capacidad de recuperación de la empresa. Con las copias de seguridad accesibles desde VM, se permite una recuperación a un momento dado durante el período de transición.
- En el caso de alta disponibilidad de HSR, los clientes deben instalar y configurar el dispositivo STONITH según las guías de alta disponibilidad de SAP HANA para [SLES](./high-availability-guide-suse-pacemaker.md) y [RHEL](./high-availability-guide-rhel-pacemaker.md).  No está preconfigurada como en el caso de HLI.
- Este enfoque de migración no cubre las SKU de HLI con la configuración de Optane.

## <a name="deployment-scenarios"></a>Escenarios de implementación
En la tabla siguiente se resumen los modelos de implementación comunes con clientes de HLI.  Es posible la migración a VM de Azure para todos los escenarios de HLI.  Para beneficiarse de los servicios complementarios de Azure disponibles, es posible que se requieran cambios de arquitectura menores.

| Id. de escenario | Escenario de HLI | ¿Migración textual a VM? | Comentario |
| --- | --- | --- | --- |
| 1 | [Nodo único con un SID](./hana-supported-scenario.md#single-node-with-one-sid) | Sí | - |
| 2 | [Nodo único con MCOS](./hana-supported-scenario.md#single-node-mcos) | Sí | - |
| 3 | [Nodo único con DR mediante replicación de almacenamiento](./hana-supported-scenario.md#single-node-with-dr-using-storage-replication) | No | La replicación de almacenamiento no está disponible con la plataforma virtual de Azure, cambie la solución actual de DR a HSR o copia de seguridad/restauración. |
| 4 | [Nodo único con DR (multipropósito) mediante replicación de almacenamiento](./hana-supported-scenario.md#single-node-with-dr-multipurpose-using-storage-replication) | No | La replicación de almacenamiento no está disponible con la plataforma virtual de Azure, cambie la solución actual de DR a HSR o copia de seguridad/restauración. |
| 5 | [HSR con STONITH para alta disponibilidad](./hana-supported-scenario.md#hsr-with-stonith-for-high-availability) | Sí | Sin ningún SBD preconfigurado para las VM de destino.  Seleccione e implemente una solución de STONITH.  Opciones posibles: Agente de barreras de Azure (compatible con [RHEL](./high-availability-guide-rhel-pacemaker.md), [SLES](./high-availability-guide-suse-pacemaker.md)), SBD |
| 6 | [Alta disponibilidad con HSR, DR con replicación de almacenamiento](./hana-supported-scenario.md#high-availability-with-hsr-and-dr-with-storage-replication) | No | Reemplace la replicación de almacenamiento para las necesidades de DR con HSR o copia de seguridad/restauración. |
| 7 | [Conmutación por error automática de host (1+1)](./hana-supported-scenario.md#host-auto-failover-11) | Sí | Use ANF el almacenamiento compartido con VM de Azure. |
| 8 | [Escalabilidad horizontal con espera](./hana-supported-scenario.md#scale-out-with-standby) | Sí | BW/4HANA con VM M128s, M416s, M416ms con ANF para almacenamiento únicamente. |
| 9 | [Escalabilidad horizontal sin espera](./hana-supported-scenario.md#scale-out-without-standby) | Sí | BW/4HANA con VM M128s, M416s, M416ms (con o sin usar ANF para almacenamiento). |
| 10 | [Escalabilidad horizontal con DR mediante replicación de almacenamiento](./hana-supported-scenario.md#scale-out-with-dr-using-storage-replication) | No | Reemplace la replicación de almacenamiento para las necesidades de DR con HSR o copia de seguridad/restauración. |
| 11 | [Nodo único con DR mediante HSR](./hana-supported-scenario.md#single-node-with-dr-using-hsr) | Sí | - |
| 12 | [HSR de nodo único a DR (optimizado para costo)](./hana-supported-scenario.md#single-node-hsr-to-dr-cost-optimized) | Sí | - |
| 13 | [Alta disponibilidad y DR con HSR](./hana-supported-scenario.md#high-availability-and-disaster-recovery-with-hsr) | Sí | - |
| 14 | [Alta disponibilidad y DR con HSR (optimizado para costo)](./hana-supported-scenario.md#high-availability-and-disaster-recovery-with-hsr-cost-optimized) | Sí | - |
| 15 | [Escalabilidad horizontal con DR mediante HSR](./hana-supported-scenario.md#scale-out-with-dr-using-hsr) | Sí | BW/4HANA con VM M128s, M416s, M416ms (con o sin usar ANF para almacenamiento). |


## <a name="source-hli-planning"></a>Planeación de origen (HLI)
Al incorporar un servidor de HLI, tanto la administración de servicios de Microsoft como los clientes han pasado por la planeación de la configuración específica de proceso, red, almacenamiento y sistema operativo para ejecutar la base de datos de SAP HANA.  Debe realizarse una planeación similar para la migración a VM de Azure.

### <a name="sap-hana-housekeeping"></a>Mantenimiento de SAP HANA 
Es un procedimiento operativo recomendado ordenar el contenido de la base de datos para que los datos no deseados, los datos obsoletos o los registros obsoletos no se migren a la nueva base de datos.  Por lo general, el mantenimiento implica la eliminación o el archivado de datos antiguos, caducados o inactivos.  Estas acciones de "higiene de datos" deban probarse en sistemas que no sean de producción para garantizar la validez de los recortes de datos antes del uso en producción.

### <a name="allow-network-connectivity-for-new-vms-and-or-virtual-network"></a>Permitir la conectividad de red para nuevas VM o la red virtual 
En la implementación de HLI de un cliente, se ha establecido la red en función de la información que se describe en el artículo [Arquitectura de red de SAP HANA (instancias grandes)](./hana-network-architecture.md). Además, el enrutamiento del tráfico de red se realiza de la manera descrita en la sección "Enrutamiento en Azure".
- Al configurar una nueva VM como destino de migración, si se coloca en la red virtual existente con los intervalos de direcciones IP ya permitidos para conectarse a HLI, no se requiere ninguna actualización de conectividad adicional.
- Si la nueva VM de Azure se coloca en una nueva instancia de Microsoft Azure Virtual Network (puede estar en otra región) y se empareja con la red virtual existente, la clave de servicio de ExpressRoute y el identificador de recurso del aprovisionamiento de HLI original se pueden usar para permitir el acceso a este nuevo intervalo de direcciones IP de la red virtual.  Coordine con la administración de servicios de Microsoft para habilitar la conectividad de la red virtual a HLI.  Nota: Para minimizar la latencia de red entre las capas de aplicación y base de datos, ambas deben estar en la misma red virtual.  

### <a name="existing-app-layer-availability-set-availability-zones-and-proximity-placement-group-ppg"></a>Conjunto de disponibilidad de la capa de aplicación existente, Availability Zones y grupos de selección de ubicación de proximidad (PPG)
El modelo actual de implementación está hecho para cumplir determinados objetivos de nivel de servicio.  En este movimiento, asegúrese de que la infraestructura de destino cumpla o supere los objetivos establecidos.  
Es más probable que los servidores de aplicaciones SAP de los clientes se coloquen en un conjunto de disponibilidad.  Si el nivel de servicio actual de implementación es satisfactorio y, además: 
- Si la VM de destino supone el nombre de host del nombre lógico de HLI, la actualización de la resolución de direcciones del servicio de nombres de dominio (DNS) que apunta a la dirección IP de la VM funcionaría sin actualizar ningún perfil de SAP.
- Si no usa PPG, asegúrese de colocar todos los servidores de aplicación y de base de datos en la misma zona para reducir la latencia de red.
- Si utiliza PPG, consulte la sección de este documento: "Planeación de destino: Conjuntos de disponibilidad, Availability Zones y grupos de selección de ubicación de proximidad (PPG)".

### <a name="storage-replication-discontinuance-process-if-used"></a>Proceso de interrupción de la replicación de almacenamiento (si se usa)
Si se usa la replicación de almacenamiento como solución de DR, debe terminarse (sin programar) una vez que se haya cerrado la aplicación de SAP.  Además, el último catálogo de SAP HANA, el archivo de registro y las copias de seguridad de los datos se han replicado en los volúmenes de almacenamiento remotos de DR de HLI.  Se hace como medida de precaución en caso de que se produzca un desastre durante la transición del servidor físico a la VM de Azure.

### <a name="data-backups-preservation-consideration"></a>Consideración de preservación de copias de seguridad de datos
Después de la migración total a SAP HANA en VM de Azure, no se podrá tener acceso fácilmente a ninguno de los datos basados en instantáneas o las copias de seguridad de registros en HLI, ni se podrán restaurar a una VM de ser necesario. En el período de transición temprano, antes de que la copia de seguridad basada en Azure cree un historial suficiente para satisfacer los requisitos de recuperación a un momento dado, se recomienda realizar copias de seguridad de nivel de archivo, además de instantáneas en HLI, días o semanas antes de la migración completa.  Haga que estas copias de seguridad se copien en una cuenta de Azure Storage a la que tenga acceso la nueva VM de SAP HANA.
Además de realizar una copia de seguridad del contenido de HLI, es prudente hacer que se pueda acceder fácilmente a las copias de seguridad completas de la entorno de SAP en caso de que se necesite una reversión.

### <a name="adjusting-system-monitoring"></a>Ajuste de la supervisión de sistemas 
Los clientes usan muchas herramientas diferentes para supervisar y enviar notificaciones de alerta a los sistemas dentro de su entorno de SAP.  Este elemento es simplemente un simple llamado a la acción adecuada al incorporar cambios para supervisar y actualizar los destinatarios de las notificaciones de alerta, de ser necesario.

### <a name="microsoft-operations-team-involvement"></a>Implicación del equipo de Microsoft Operations 
Abra un vale de Azure Portal basado en la instancia de HLI existente.  Una vez que se cree la incidencia de soporte técnico, un ingeniero de soporte técnico se pondrá en contacto con usted por correo electrónico.  

### <a name="engage-microsoft-account-team"></a>Participación del equipo de cuentas de Microsoft
Planee la migración cerca de la fecha de aniversario de renovación del contrato de HLI, con el fin de minimizar los sobrecostos innecesarios en recursos de proceso. Para retirar la hoja de HLI, es necesario coordinar la finalización del contrato y el apagado real de la unidad.

## <a name="destination-planning"></a>Planeación de destino
La creación de una nueva infraestructura para sustituir otra existente merece cierta atención para asegurar que la nueva incorporación quepa en el esquema general.  A continuación, encontrará algunos puntos clave en los que pensar.

### <a name="resource-availability-in-the-target-region"></a>Disponibilidad de recursos en la región de destino 
Normalmente, la región de implementación actual del servidor de aplicaciones de SAP está en estrecha proximidad con las instancias de HLI asociadas.  Sin embargo, las instancias de HLI se ofrecen en menos ubicaciones que las regiones de Azure disponibles.  Al migrar de la máquina física de HLI a la VM de Azure, también es un buen momento para "afinar" la distancia de proximidad de todos los servicios relacionados para optimizar el rendimiento.  Al hacerlo, una consideración clave sería asegurarse de que la región elegida tenga todos los recursos necesarios.  Por ejemplo, la disponibilidad de ciertas familias de VM o la oferta de zonas de Azure para la configuración de alta disponibilidad.

### <a name="virtual-network"></a>Virtual network 
Los clientes deben elegir si desean ejecutar la nueva base de datos de HANA en una red virtual existente o crear una nueva.  El factor de decisión principal es el diseño de red actual de conexión en red para el entorno de SAP.  Además, cuando la infraestructura pasa de una implementación de una zona a otra de dos zonas y usa PPG, impone un cambio en la arquitectura. Para obtener más información, consulte el artículo [Grupos de selección de ubicación de proximidad de Azure para una latencia de red óptima con aplicaciones SAP](./sap-proximity-placement-scenarios.md).   

### <a name="security"></a>Seguridad
Ya sea que la nueva VM de SAP HANA aterrice en una red virtual o subred nueva o existente, representa un nuevo servicio crítico para la empresa, que requiere protección.  Debe evaluarse e implementarse el control de acceso en cumplimiento de la directiva de seguridad de información de la compañía para esta nueva clase de servicio.

### <a name="vm-sizing-recommendation"></a>Recomendación de ajuste de tamaño de la VM
Esta migración también es una oportunidad para ajustar el tamaño del motor de proceso de HANA.  Puede usar las [vistas del sistema](https://help.sap.com/viewer/7c78579ce9b14a669c1f3295b0d8ca16/Cloud/3859e48180bb4cf8a207e15cf25a7e57.html) de HANA junto con HANA Studio para comprender el consumo de recursos del sistema, lo que permite dar con el tamaño adecuado para impulsar la eficacia de los gastos.

### <a name="storage"></a>Storage 
El rendimiento del almacenamiento es uno de los factores que afectan a la experiencia de usuario de la aplicación de SAP.  En función de una SKU de VM determinada, existen requisitos de diseño de almacenamiento mínimo publicados en [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md). Se recomienda revisar estas especificaciones mínimas y compararlas con las estadísticas del sistema de HLI existentes para garantizar la capacidad de E/S y el rendimiento adecuados para la nueva VM de HANA.

Si configura PPG para la nueva VM de HANA y sus servidores asociados, envíe una incidencia de soporte técnico para inspeccionar y garantizar la colocación del almacenamiento y la VM. Dado que es posible que su solución de copia de seguridad tenga que cambiar, el costo de almacenamiento también debe revisarse para evitar sorpresas en los gastos operativos.

### <a name="storage-replication-for-disaster-recovery"></a>Replicación de almacenamiento para recuperación ante desastres
Con HLI, la replicación de almacenamiento se ofrecía como la opción predeterminada para la recuperación ante desastres. Esta característica no es la opción predeterminada para SAP HANA con VM de Azure. Tenga en cuenta HSR, copia de seguridad/restauración u otras soluciones compatibles que satisfagan sus necesidades empresariales.

### <a name="availability-sets-availability-zones-and-proximity-placement-groups"></a>Conjuntos de disponibilidad, Availability Zones y grupos de selección de ubicación de proximidad 
Para acortar la distancia entre la capa de aplicación y SAP HANA a fin de mantener la latencia de red en un mínimo, la nueva VM de base de datos y los servidores de aplicaciones de SAP actuales deben colocarse en un PPG. Consulte [Grupos de selección de ubicación de proximidad](./sap-proximity-placement-scenarios.md) para aprender cómo funcionan los conjuntos de disponibilidad y Azure Availability Zones con PPG para implementaciones de SAP.
Si los miembros del sistema HANA de destino se implementan en más de una zona de Azure, los clientes deben tener una visión clara del perfil de latencia de las zonas elegidas. La colocación de los componentes del sistema SAP es óptima con respecto a la distancia proximal entre la aplicación de SAP y la base de datos.  El dominio público [herramienta de prueba de latencia de zona de disponibilidad](https://github.com/Azure/SAP-on-Azure-Scripts-and-Utilities/tree/master/AvZone-Latency-Test) ayuda a facilitar la medición.  


### <a name="backup-strategy"></a>Estrategia de copia de seguridad
Muchos clientes ya usan soluciones de copia de seguridad de terceros para SAP HANA en HLI.  En ese caso, solo deben configurarse una VM protegida adicional y bases de datos de HANA.  En adelante, los trabajos de copia de seguridad de HLI en curso se pueden desprogramar si el equipo se está dando de baja después de la migración.
Azure Backup para SAP HANA en VM ya está disponible con carácter general.  Para obtener información detallada, consulte estos vínculos: [Copia de seguridad](../../../backup/backup-azure-sap-hana-database.md), [restauración](../../../backup/sap-hana-db-restore.md), [administración](../../../backup/sap-hana-db-manage.md) de copias de seguridad de SAP HANA en VM de Azure.

### <a name="dr-strategy"></a>Estrategia de recuperación ante desastres
Si los objetivos de nivel de servicio admiten un tiempo de recuperación más largo, una copia de seguridad simple en el almacenamiento de blobs y la restauración en contexto o en una nueva VM, esta es la estrategia de recuperación ante desastres más sencilla y menos costosa.  
Al igual que la plataforma de instancias grandes en la que la recuperación ante desastres de HANA se realiza normalmente con HSR; en VM de Azure, HSR es también la solución de recuperación ante desastres más natural y nativa de SAP HANA.  Independientemente de si la implementación de origen es de instancia única o en clúster, se requiere una réplica de la infraestructura de origen en la región de DR.
Esta réplica de DR se configurará después de que se complete la migración principal de HLI a VM.  La base de datos de recuperación ante desastres de HANA se registrará en la instancia principal de SAP HANA en VM como sitio de replicación secundario.  

### <a name="sap-application-server-connectivity-destination-change"></a>Cambio de destino de conectividad del servidor de aplicaciones de SAP
La migración de HSR produce un nuevo host de base de datos de HANA y, por lo tanto, un nuevo nombre de host de base de nombres para la capa de aplicación, y los perfiles de SAP deben modificarse para reflejar el nuevo nombre de host.  Si el cambio se realiza mediante la resolución de nombres que conserva el nombre de host, no se requiere ningún cambio de perfil.

### <a name="operating-system"></a>Sistema operativo
Las imágenes de sistema operativo para HLI y VM, aunque estén en el mismo nivel de versión, por ejemplo, SLES 12 SP4, no son idénticas. Los clientes deben validar los paquetes necesarios, las correcciones, las revisiones, el kernel y las correcciones de seguridad en HLI para poder instalar los mismos paquetes en el destino.  Se admite el uso de HSR para replicar de un sistema operativo anterior en una VM con una versión más reciente del sistema operativo.  Para verificar las versiones compatibles específicas, revise [la nota de SAP 2763388](https://launchpad.support.sap.com/#/notes/2763388).

### <a name="new-sap-license-request"></a>Nueva solicitud de licencia de SAP
Una simple llamada para solicitar una nueva licencia de SAP para el nuevo sistema de HANA ahora que se ha migrado a VM.

### <a name="service-level-agreement-sla-differences"></a>Diferencias en los contratos de nivel de servicio (SLA)
Los autores quieren destacar la diferencia de SLA de disponibilidad entre HLI y VM de Azure.  Por ejemplo, los pares de alta disponibilidad en clúster de HLI ofrecen una disponibilidad del 99,99 %. Para lograr el mismo SLA, hay que implementar VM en zonas de disponibilidad. En este [artículo](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_9/) se describe la disponibilidad con arquitecturas de implementación asociadas para que los clientes puedan planear la infraestructura de destino en consecuencia.


## <a name="migration-strategy"></a>Estrategia de migración
En este documento, solo se aborda el enfoque de replicación del sistema de HANA para la migración de HLI a VM de Azure.  Dependiendo de la solución de almacenamiento de destino implementada, el proceso difiere ligeramente. A continuación se describen los pasos generales.

### <a name="vm-with-premiumultra-disks-for-data"></a>VM con discos Premium o Ultra para datos
En el caso de las VM que se implementan con discos Premium o Ultra, la configuración estándar de replicación del sistema de SAP HANA se aplica para configurar HSR.  El [artículo de ayuda de SAP](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.04/099caa1959ce4b3fa1144562fa09e163.html) proporciona información general sobre los pasos necesarios para configurar la replicación del sistema, la toma de control de un sistema secundario, la conmutación por recuperación a un sistema principal y la deshabilitación de la replicación del sistema.  A los efectos de la migración, solo se necesitarán los pasos de instalación, toma de control y deshabilitación de la replicación.  

### <a name="vm-with-anf-for-data-and-log-volumes"></a>VM con ANF para volúmenes de datos y de registro
En un nivel alto, las instantáneas más recientes de almacenamiento de HLI de los volúmenes de datos y de registro completos deben copiarse en Azure Storage, donde la VM de HANA de destino puede acceder a ellos y recuperarlos.  El proceso de copia se puede realizar con las herramientas de copia nativas de Linux.  

> [!IMPORTANT]
> La copia y la transferencia de datos pueden tardar horas en función del tamaño de la base de datos de HANA y del ancho de banda de red.  La mayor parte del proceso de copia debe realizarse antes del tiempo de inactividad principal de la base de datos de HANA.

### <a name="mcos-to-mdc-conversion"></a>Conversión de MCOS a MDC
Algunos de nuestros clientes de HLI usaron el modelo de implementación de Multiple Components in One System (MCOS).  La motivación era eludir la limitación de instantáneas de almacenamiento de Multiple Databases Container (MDC) de las versiones anteriores de SAP HANA.  En el modelo MCOS, varias instancias de SAP HANA independientes se apilan en una hoja de HLI.  El uso de HSR para la migración funcionaría bien, pero daría lugar a varias VM de HANA con una base de trabajo de inquilino cada una.  Este estado final hace posible un panorama más complejo del aquel al que un cliente podría haber estado acostumbrado.  Con SAP HANA 2.0, la implementación predeterminada es MDC, una alternativa viable es realizar un [movimiento de inquilinos de HANA](https://launchpad.support.sap.com/#/notes/2472478) después de la migración de HSR.  Este proceso "consolida" estas bases de datos de HANA independientes en los coinquilinos en un solo contenedor de HANA.  

### <a name="application-layer-consideration"></a>Consideración de la capa de aplicación
El servidor de base de datos se ve como el centro de un sistema SAP.  Todos los servidores de aplicaciones se deben ubicar cerca de la base de datos de SAP HANA.  En algunos casos, cuando se desea un nuevo uso de PPG, puede ser necesaria la reubicación de los servidores de aplicaciones existentes en el PPG donde se encuentra la VM de HANA.  Es posible que la creación de nuevos servidores de aplicaciones sea más fácil si ya tiene plantillas de implementación útiles.  
Si los servidores de aplicaciones existentes y la nueva VM de HANA se encuentran en una ubicación óptima, no es necesario compilar nuevos servidores de aplicaciones, a menos que se requiera capacidad adicional.  
Si se crea una nueva infraestructura para mejorar la disponibilidad del servicio, es posible que los servidores de aplicaciones existentes sean innecesarios y deban apagarse y eliminarse.
Si el nombre de host de la VM de destino ha cambiado y difiere del nombre de host de HLI, los perfiles de los servidores de aplicaciones de SAP deben ajustarse para que apunten al nuevo host.  Si solo ha cambiado la dirección IP de la base de datos de HANA, se necesita una actualización de registros DNS para dirigir las conexiones entrantes a la nueva VM de HANA.

### <a name="acceptance-test"></a>Prueba de aceptación
Aunque la migración de HLI a VM no realiza ningún cambio sustancial en el contenido de la base de datos en comparación con una migración heterogénea, todavía se recomienda validar las funcionalidades clave y el aspecto de rendimiento de la nueva configuración.  

### <a name="cutover-plan"></a>Plan de migración total
Si bien esta migración es sencilla, implica el retiro de una base de datos existente.  La planeación cuidadosa para conservar el sistema de origen con el contenido asociado y las imágenes de copia de seguridad es fundamental en caso de que sea necesaria la reserva.  Una buena planeación ofrece una reversión más rápida.   


## <a name="post-migration"></a>Después de la migración
El trabajo de migración no se realiza hasta que no se hayan desacoplado con seguridad los servicios o la conectividad dependientes de HLI para asegurar que se preserve la integridad de los datos.  Además, apague los servicios innecesarios.  En esta sección se describen algunos elementos que deben tenerse presentes.

### <a name="decommissioning-the-hli"></a>Retiro de HLI
Después de una migración correcta de la base de datos de HANA a VM de Azure, asegúrese de que no se ejecute ninguna transacción de negocio de producción en la base de datos de HLI.  Sin embargo, mantener la instancia de HLI en ejecución durante un período igual a su ventana de retención de datos de copia de seguridad local es una práctica segura que garantiza una recuperación de datos más rápida si es necesario.  Solo entonces se debe retirar la hoja de HLI.  Los clientes deben ponerse en contacto con sus representantes de Microsoft para finalizar contractualmente sus compromisos de HLI con Microsoft.

### <a name="remove-any-proxy-ex-iptables-bigip-configured-for-hli"></a>Eliminación de un proxy (por ejemplo: Iptables, BIGIP) configurado para HLI 
Si se usa un servicio de proxy como el de IPTables para enrutar el tráfico local hacia y desde HLI, ya no es necesario después de la migración correcta a VM.  Sin embargo, este servicio de conectividad debe conservarse mientras la hoja de HLI todavía esté en espera.  Únicamente apague el servicio después de que se haya retirado por completo la hoja de HLI.

### <a name="remove-global-reach-for-hli"></a>Eliminación de Global Reach para HLI 
Global Reach se usa para conectar la puerta de enlace de ExpressRoute de los clientes con la puerta de enlace de ExpressRoute de HLI.  Permite que el tráfico local de los clientes llegue directamente al inquilino de HLI sin usar un servicio de proxy.  Esta conexión ya no es necesaria en ausencia de la unidad de HLI después de la migración.  Al igual que el caso del servicio de proxy de IPTables, GlobalReach también debe conservarse hasta que se haya retirado por completo la hoja de HLI.

### <a name="operating-system-subscription--movereuse"></a>Suscripción al sistema operativo: movimiento y reutilización
A medida que los servidores de VM se ponen en marcha y se retiran las hojas de HLI, las suscripciones de sistema operativo se pueden reemplazar o reutilizar para evitar el doble pago de licencias del sistema operativo.



## <a name="next-steps"></a>Pasos siguientes
Consulte estos artículos:
- [Configuraciones y operaciones de infraestructura de SAP HANA en Azure](./hana-vm-operations.md).
- [Lista de comprobación de planeamiento e implementación de cargas de trabajo de SAP en Azure](./sap-deployment-checklist.md).
