---
title: Diseño e implementación de una base de datos de Oracle en Azure | Microsoft Docs
description: Diseño e implementación de una base de datos de Oracle en el entorno de Azure.
services: virtual-machines-linux
documentationcenter: virtual-machines
author: rgardler
manager: ''
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 08/02/2018
ms.author: rogardle
ms.openlocfilehash: 78eedb9bd4f12644a1bc992d0786a43b8af767a9
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86507937"
---
# <a name="design-and-implement-an-oracle-database-in-azure"></a>Diseño e implementación de una base de datos de Oracle en Azure

## <a name="assumptions"></a>Supuestos

- Planea la migración de una base de datos Oracle del entorno local a Azure.
- Tiene el [paquete de diagnósticos](https://docs.oracle.com/cd/E11857_01/license.111/e11987/database_management.htm) para la instancia de Oracle Database que quiere migrar
- Conoce las diversas métricas de los informes de AWR de Oracle.
- Tiene conocimientos básicos sobre el rendimiento de la aplicación y el uso de la plataforma.

## <a name="goals"></a>Objetivos

- Entender cómo optimizar la implementación de Oracle en Azure.
- Explorar las opciones de optimización del rendimiento de la base de datos de Oracle en el entorno de Azure.

## <a name="the-differences-between-an-on-premises-and-azure-implementation"></a>Diferencias entre una implementación local y la implementación en Azure 

Las siguientes son algunas cuestiones importantes a tener en cuenta cuando se realiza la migración de aplicaciones locales a Azure. 

Una diferencia importante es que en una implementación en Azure, los recursos, como las máquinas virtuales, discos y redes virtuales, se comparten con otros clientes. Además, los recursos pueden estar limitados en función de los requisitos. En lugar de centrarse en evitar errores (MTBF), Azure se centra más bien en superar los errores (MTTR).

En la tabla siguiente se enumeran algunas de las diferencias entre una implementación local y una implementación en Azure de una base de datos de Oracle.


|  | Implementación local | Implementación en Azure |
| --- | --- | --- |
| **Redes** |LAN/WAN  |SDN (Redes definidas por software)|
| **Grupo de seguridad** |Herramientas de restricción de IP y puerto |[Grupo de seguridad de red (NSG)](https://azure.microsoft.com/blog/network-security-groups) |
| **Resistencia** |MTBF (tiempo medio entre errores) |MTTR (tiempo medio para recuperación)|
| **Mantenimiento planeado** |Aplicación de revisiones/actualizaciones|[Conjuntos de disponibilidad](../../windows/infrastructure-example.md) (aplicación de revisiones o actualizaciones administradas por Azure) |
| **Recurso** |Dedicado  |Compartido con otros clientes|
| **Regiones** |Centros de datos |[Pares de región](../../regions.md#region-pairs)|
| **Storage** |SAN/discos físicos |[Almacenamiento administrado por Azure](https://azure.microsoft.com/pricing/details/managed-disks/?v=17.23h)|
| **Escala** |Escalado vertical |Escalado horizontal|


### <a name="requirements"></a>Requisitos

- Determinar el tamaño de la base de datos y la tasa de crecimiento.
- Determinar los requisitos de IOPS, que pueden calcularse en función de los informes de AWR de Oracle o de otras herramientas de supervisión de red.

## <a name="configuration-options"></a>Opciones de configuración

Hay cuatro áreas posibles que se pueden ajustar para mejorar el rendimiento en el entorno de Azure:

- Tamaño de la máquina virtual
- Capacidad de proceso de la red
- Tipos y configuraciones de disco
- Configuración de la caché de disco

### <a name="generate-an-awr-report"></a>Generar un informe de AWR

Si tiene una base de datos de Oracle y está planeando migrar a Azure, tiene varias opciones. Si tiene el [paquete de diagnósticos](https://www.oracle.com/technetwork/oem/pdf/511880.pdf) para sus instancias de Oracle, puede ejecutar el informe de AWR de Oracle para obtener las métricas (IOPS, Mbps, GiBs, etc.). A continuación, elija la máquina virtual en función de las métricas recopiladas. También puede ponerse en contacto con el equipo de infraestructura para obtener información similar.

Puede considerar la posibilidad de ejecutar el informe de AWR durante la carga de trabajo normal y máxima, para poder comparar ambas. En función de estos informes, puede ajustar el tamaño de las máquinas virtuales según la carga de trabajo media o máxima.

A continuación, se muestra un ejemplo de cómo generar un informe de AWR (genere sus informes de AWR con Oracle Enterprise Manager, si su instalación actual lo tiene disponible):

```bash
$ sqlplus / as sysdba
SQL> EXEC DBMS_WORKLOAD_REPOSITORY.CREATE_SNAPSHOT;
SQL> @?/rdbms/admin/awrrpt.sql
```

### <a name="key-metrics"></a>Métricas clave

Estas son las métricas que puede obtener del informe de AWR:

- Número total de núcleos
- Velocidad del reloj de la CPU
- Memoria total en GB
- Uso de CPU
- Velocidad de transferencia de datos máxima
- Frecuencia de cambios de E/S (lectura/escritura)
- Velocidad de los registros de rehacer (Mbps)
- Capacidad de proceso de la red
- Tasa de latencia de red (baja/alta)
- Tamaño de la base de datos en GB
- Bytes recibidos a través de SQL*Net desde/hacia el cliente

### <a name="virtual-machine-size"></a>Tamaño de la máquina virtual

#### <a name="1-estimate-vm-size-based-on-cpu-memory-and-io-usage-from-the-awr-report"></a>1. La estimación del tamaño de la máquina virtual se basa en el uso de CPU, memoria y E/S según el informe de AWR

Puede consultar los primeros cinco eventos de primer plano cronometrados, que indican dónde están los cuellos de botella del sistema.

Por ejemplo, en el diagrama siguiente, la sincronización de archivos de registro está en la primera posición. Indica el número de esperas necesarias para que LGWR escriba el búfer de registro en el archivo de registro de puesta al día. Estos resultados indican que son necesarios almacenamiento o discos de mejor rendimiento. Además, también se muestra en el diagrama el número de CPU (núcleos) y la memoria.

![Captura de pantalla de la página del informe de AWR](./media/oracle-design/cpu_memory_info.png)

El siguiente diagrama muestra el total de E/S de lectura y escritura. Hubo 59 GB de lectura y 247,3 GB de escritura durante la realización del informe.

![Captura de pantalla de la página del informe de AWR](./media/oracle-design/io_info.png)

#### <a name="2-choose-a-vm"></a>2. Elegir una máquina virtual

En función de la información recopilada en el informe de AWR, el paso siguiente consiste en elegir una máquina virtual de un tamaño similar que cumpla los requisitos. Puede encontrar una lista de máquinas virtuales disponibles en el artículo [Optimización de memoria](../../sizes-memory.md).

#### <a name="3-fine-tune-the-vm-sizing-with-a-similar-vm-series-based-on-the-acu"></a>3. Ajustar el tamaño de la máquina virtual con una serie de máquina virtual similar en función del valor de ACU

Una vez que haya elegido la máquina virtual, preste atención a la ACU de la máquina virtual. Puede elegir una máquina virtual diferente en función del valor de ACU que mejor se adapte a sus necesidades. Para más información, consulte [Unidad de proceso de Azure (ACU)](../../acu.md).

![Captura de pantalla de la página de ACU](./media/oracle-design/acu_units.png)

### <a name="network-throughput"></a>Capacidad de proceso de la red

El diagrama siguiente muestra la relación que existe entre el rendimiento y las E/S por segundo (IOPS):

![Captura de pantalla del rendimiento](./media/oracle-design/throughput.png)

El rendimiento total de la red se calcula en función de la siguiente información:
- Tráfico de SQL*Net
- MBps multiplicado por el número de servidores (secuencia de salida, como Oracle Data Guard)
- Otros factores, como la replicación de aplicaciones

![Captura de pantalla del rendimiento de SQL*Net](./media/oracle-design/sqlnet_info.png)

En función de los requisitos de ancho de banda de red, puede elegir diferentes tipos de puerta de enlace. Entre los tipos se incluyen Básica, VpnGw y Azure ExpressRoute. Para más información, consulte la [página de precios de VPN Gateway](https://azure.microsoft.com/pricing/details/vpn-gateway/?v=17.23h).

**Recomendaciones**

- La latencia de red es superior en comparación con una implementación local. El rendimiento puede mejorar considerablemente si se reducen los recorridos de ida y vuelta de red.
- Consolide las aplicaciones que tengan transacciones elevadas o estén "fragmentadas" en la misma máquina virtual para reducir los recorridos de ida y vuelta.
- Use una instancia de Virtual Machines con las [redes aceleradas](../../../virtual-network/create-vm-accelerated-networking-cli.md) para mejorar el rendimiento de la red.
- En cuanto a ciertas distribuciones de Linux, habilite la [compatibilidad con TRIM/UNMAP](../../linux/configure-lvm.md#trimunmap-support).
- Instale [Oracle Enterprise Manager](https://www.oracle.com/technetwork/oem/enterprise-manager/overview/index.html) en una máquina virtual individual.
- Las páginas de gran tamaño no están habilitadas en Linux de forma predeterminada. Habilite las páginas de gran tamaño y establezca `use_large_pages = ONLY` en Oracle DB. Esto puede ayudarle a mejorar el rendimiento. Puede encontrar más información [aquí](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/refrn/USE_LARGE_PAGES.html#GUID-1B0F4D27-8222-439E-A01D-E50758C88390).

### <a name="disk-types-and-configurations"></a>Tipos y configuraciones de disco

- *Discos de sistema operativo predeterminados*: estos tipos de disco ofrecen datos persistentes y almacenamiento en caché. Estos discos están optimizados para el acceso del sistema operativo en tiempo de inicio y no están diseñados para cargas de trabajo transaccionales o de almacenamiento de datos (analíticas).

- *Discos no administrados*: con estos tipos de disco, administra las cuentas de almacenamiento que almacenan los archivos de disco duro virtual (VHD) que se corresponden con los discos de la máquina virtual. Los archivos VHD se almacenan como blobs de páginas en las cuentas de Azure Storage.

- *Managed Disks*: Azure administra las cuentas de almacenamiento utilizadas para los discos de la máquina virtual. Especifique el tipo de disco (Premium o Standard) y el tamaño de disco que necesite. Azure crea y administra el disco en su nombre.

- *Discos de Premium Storage*: estos tipos de disco son más adecuados para las cargas de trabajo de producción. Premium Storage admite discos de máquina virtual que se pueden conectar a series específicas de máquina virtual, como las series DS, DSv2, GS y F. El disco Premium tiene diferentes tamaños entre los que puede elegir, desde 32 GB a 4,096 GB. Cada tamaño de disco tiene sus propias especificaciones de rendimiento. En función de los requisitos de la aplicación puede conectar uno o varios discos a la VM.

Cuando se crea un nuevo disco administrado desde el portal, puede elegir el **Tipo de cuenta** para el tipo de disco que quiere usar. Es importante saber que en el menú desplegable no se muestran todos los discos disponibles. Después de elegir un tamaño de máquina virtual determinado, el menú muestra solo las SKU de Premium Storage disponibles que se basan en ese tamaño de máquina virtual.

![Captura de pantalla de la página de disco administrado](./media/oracle-design/premium_disk01.png)

Una vez configurado el almacenamiento en una máquina virtual, tal vez le interese realizar una prueba de carga en los discos antes de crear una base de datos. Puede ser útil conocer la velocidad de E/S en términos de latencia y rendimiento para ayudarle a determinar si las máquinas virtuales admiten el rendimiento esperado con objetivos de latencia.

Existen diversas herramientas para realizar pruebas de carga de aplicaciones, como Oracle Orion, Sysbench y Fio.

Vuelva a ejecutar la prueba de carga después de implementar una base de datos de Oracle. Inicie las cargas de trabajo normales y máximas y los resultados mostrarán la línea de base de su entorno.

Puede ser más importante determinar el tamaño del almacenamiento en función de la tasa de E/S por segundo, en vez de según el tamaño de almacenamiento. Por ejemplo, si los IOPS necesarios son 5000, pero solo necesita 200 GB, puede elegir el disco Premium P30 aunque se ofrece con más de 200 GB de almacenamiento.

La tasa de IOPS puede obtenerse de los informes de AWR. Viene determinado por el registro de recuperación, las lecturas físicas y la velocidad de escritura.

![Captura de pantalla de la página del informe de AWR](./media/oracle-design/awr_report.png)

Por ejemplo: el tamaño del registro de recuperación es 12 200 000 bytes por segundo, lo que equivale a 11,63 MBPs.
El número de IOPS es 12 200 000 / 2 358 = 5 174.

Una vez que se haya hecho una idea de los requisitos de E/S, puede elegir la combinación de unidades más adecuada para cumplirlos.

**Recomendaciones**

- Para el espacio de tabla de datos, reparta la carga de trabajo de E/S entre varios discos mediante el almacenamiento administrado u Oracle ASM.
- Agregue más discos de datos a medida que aumente el tamaño del bloque de E/S, para las operaciones intensivas de lectura y escritura.
- Aumente el tamaño de bloque para procesos secuenciales de gran tamaño.
- Use la compresión de datos para reducir la E/S (para datos e índices).
- Separe los registros de rehacer, sistema, temporales y de deshacer TS en discos de datos independientes.
- No colocar ningún archivo de aplicación en el disco predeterminado del sistema operativo (/dev/sda). Estos discos están optimizados para un tiempo de arranque rápido de la máquina virtual y podría no proporcionar un buen rendimiento para la aplicación.
- Al usar VM de la serie M en el almacenamiento Premium, habilite el [acelerador de escritura](../../linux/how-to-enable-write-accelerator.md) en el disco de registros de fase de puesta al día.

### <a name="disk-cache-settings"></a>Configuración de la caché de disco

Existen tres opciones para el almacenamiento en caché de host:

- *ReadOnly*: todas las solicitudes se almacenan en caché para lecturas futuras. Todas las escrituras se guardan de modo permanente directamente en Azure Blob Storage.

- *ReadWrite*: se trata de un algoritmo de "lectura anticipada". Las lecturas y las escrituras se almacenan en caché para lecturas futuras. Si no son escrituras a través, se guardan primero en la caché local. También proporciona la menor latencia de disco para cargas de trabajo ligeras. El uso de la memoria caché ReadWrite con una aplicación que no administre la persistencia de los datos necesarios puede provocar la pérdida de los datos, si se bloquea la máquina virtual.

- *Ninguno* (deshabilitado): con esta opción, puede omitir la memoria caché. Todos los datos se transfieren al disco y se guardan en Azure Storage. Este método proporciona la mayor velocidad de E/S para cargas de trabajo intensivas de E/S. También debe tener en cuenta el "costo de transacción".

**Recomendaciones**

Para maximizar el rendimiento, se recomienda comenzar con la opción **Ninguno** para el almacenamiento en caché de host. En el caso de Premium Storage, es importante tener en cuenta que debe deshabilitar las "barreras" al montar el sistema de archivos con las opciones **Solo lectura** o **Ninguno**. Actualice el archivo/etc/fstab con el UUID en los discos.

![Captura de pantalla de la página de disco administrado](./media/oracle-design/premium_disk02.png)

- En los discos del sistema operativo, use el almacenamiento en caché predeterminado **Lectura y escritura**.
- En el caso de SYSTEM, TEMP y UNDO, use la opción **Ninguno** para el almacenamiento en caché.
- Para los datos, utilice **Ninguno** para el almacenamiento en caché. Pero si la base de datos es de solo lectura o de lectura intensiva, use el almacenamiento en caché de **Solo lectura**.

Una vez que se haya guardado la configuración del disco de datos, no se puede cambiar la configuración de la caché de host, a menos que se desmonte la unidad en el nivel de sistema operativo y se vuelva a montar después de cambiarla.

## <a name="security"></a>Seguridad

Después de que se instala y configura el entorno de Azure, el siguiente paso es proteger la red. Estas son algunas recomendaciones:

- *Directiva del NSG*: el grupo de seguridad de red (NSG) se puede definir como una subred o NIC. Es más fácil controlar el acceso en el nivel de subred para garantizar la seguridad y forzar el enrutamiento de elementos como el firewall de aplicaciones.

- *Jumpbox*: Para un acceso más seguro, los administradores no deben conectarse directamente con el servicio de aplicación o la base de datos. Se usa un Jumpbox como un medio entre la máquina del administrador y los recursos de Azure.
![Captura de pantalla de la página de topología de Jumpbox](./media/oracle-design/jumpbox.png)

    La máquina del administrador debería tener un acceso restringido a la dirección IP del Jumpbox únicamente. El Jumpbox debe tener acceso a la aplicación y la base de datos.

- *Red privada* (subredes): se recomienda que tenga el servicio de aplicación y la base de datos en subredes independientes, para que pueda establecerse un mejor control con la directiva de NSG.


## <a name="additional-reading"></a>Lecturas adicionales

- [Configuración de ASM de Oracle](configure-oracle-asm.md)
- [Configuración de Oracle Data Guard](configure-oracle-dataguard.md)
- [Configuración de Oracle Golden Gate](configure-oracle-golden-gate.md)
- [Copia de seguridad y recuperación de Oracle](oracle-backup-recovery.md)

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Creación de máquinas virtuales de alta disponibilidad](../../linux/create-cli-complete.md)
- [Ejemplos de la CLI de Azure para implementación de máquinas virtuales](../../linux/cli-samples.md)
