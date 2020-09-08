---
title: Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP | Microsoft Docs
description: Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 12/04/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 15c0368b2d0bd85f6fee65ffa2c9d6776d07f162
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88650622"
---
# <a name="considerations-for-azure-virtual-machines-dbms-deployment-for-sap-workload"></a>Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP
[1114181]:https://launchpad.support.sap.com/#/notes/1114181
[1409604]:https://launchpad.support.sap.com/#/notes/1409604
[1597355]:https://launchpad.support.sap.com/#/notes/1597355
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2039619]:https://launchpad.support.sap.com/#/notes/2039619
[2069760]:https://launchpad.support.sap.com/#/notes/2069760
[2171857]:https://launchpad.support.sap.com/#/notes/2171857
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2233094]:https://launchpad.support.sap.com/#/notes/2233094
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[deployment-guide]:deployment-guide.md
[deployment-guide-3]:deployment-guide.md#b3253ee3-d63b-4d74-a49b-185e76c4088e
[planning-guide]:planning-guide.md

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png


[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

Esta guía forma parte de la documentación sobre la implementación del software de SAP en Microsoft Azure. Antes de leer esta guía, vea la [Guía de planeamiento e implementación][planning-guide]. En este documento se describen los aspectos de implementación genéricos de los sistemas DBMS relacionados con SAP en Microsoft Azure Virtual Machines (VM) mediante las funcionalidades de infraestructura como servicio (IaaS) de Azure.

El documento complementa la documentación de instalación de SAP y las Notas de SAP, que representan los principales recursos para las instalaciones e implementaciones de software de SAP en las plataformas proporcionadas.

En este documento, se presentan las consideraciones sobre la ejecución de sistemas DBMS relacionados con SAP en máquinas virtuales de Azure. En este capítulo, se hacen algunas referencias a sistemas DBMS específicos. En su lugar, en este artículo se abordan los sistemas DBMS específicos, después de este documento.

## <a name="definitions"></a>Definiciones
En todo el documento, se usan estos términos:

* **IaaS**: infraestructura como servicio.
* **PaaS**: plataforma como servicio.
* **SaaS**: software como servicio.
* **Componente de SAP**: una aplicación de SAP individual, por ejemplo, ERP Central Component (ECC), Business Warehouse (BW), Solution Manager o Enterprise Portal (EP). Los componentes de SAP pueden basarse en tecnologías tradicionales, como ABAP o Java, o en una aplicación no basada en NetWeaver, como Business Objects.
* **Entorno de SAP**: uno o varios componentes de SAP agrupados lógicamente para desempeñar una función empresarial, como desarrollo, control de calidad, aprendizaje, recuperación ante desastres o producción.
* **Infraestructura de SAP**: Este término hace referencia a todos los recursos de SAP de la infraestructura de TI de un cliente. La infraestructura de SAP incluye todos los entornos, tanto los que son de producción como los que no.
* **Sistema SAP**: la combinación de la capa de DBMS y el nivel de aplicación de, por ejemplo, un sistema de desarrollo de SAP ERP, un sistema de prueba de SAP BW y un sistema de producción de SAP CRM. Las implementaciones de Azure no admiten la división de estas dos capas entre la infraestructura local y la de Azure. En consecuencia, un sistema SAP debe implementarse de forma local o en Azure, pero no en ambos. Los diferentes sistemas de un entorno de SAP pueden implementarse en Azure o de forma local. Por ejemplo, pueden implementarse los sistemas de desarrollo y pruebas de SAP CRM en Azure, a la vez que se implementa el de producción de forma local.
* **Entre locales**: describe un escenario donde se implementan máquinas virtuales en una suscripción de Azure con conexión de sitio a sitio, entre varios sitios o de Azure ExpressRoute entre los centros de datos locales y Azure. En la documentación habitual de Azure, este tipo de implementaciones se denominan "escenarios entre locales". 

    El motivo de la conexión es ampliar los dominios locales, Active Directory local y DNS local a Azure. La infraestructura local se extiende a los recursos de Azure de la suscripción. Con esta extensión, las máquinas virtuales pueden formar parte del dominio local. Los usuarios del dominio local pueden acceder a los servidores y ejecutar servicios en esas máquinas virtuales (por ejemplo, servicios de DBMS). Es posible la comunicación y resolución de nombres entre máquinas virtuales implementadas de forma local y en Azure. Este es el escenario más habitual para implementar recursos de SAP en Azure. Para obtener más información, vea [Planeamiento y diseño de VPN Gateway](../../../vpn-gateway/vpn-gateway-about-vpngateways.md).

> [!NOTE]
> En sistemas de producción SAP, se admiten implementaciones entre locales de sistemas SAP donde las máquinas virtuales de Azure que ejecuten sistemas SAP sean miembros de un dominio local. Las configuraciones entre locales se admiten para la implementación completa o parcial de infraestructuras de SAP en Azure. Incluso para ejecutar la infraestructura completa de SAP en Azure, es necesario que esas máquinas virtuales formen parte de un dominio local y Active Directory o LDAP. 
>
> En versiones anteriores de la documentación, se mencionaron los escenarios de TI híbrida. El término *híbrida* se basa en el hecho de que hay una conectividad entre locales entre el entorno local y Azure. En este caso, híbrido también significa que las máquinas virtuales de Azure forman parte de Active Directory local.
>
>

Algunos documentos de Microsoft describen escenarios entre locales de un modo ligeramente distinto, en especial en configuraciones de DBMS de alta disponibilidad. En el caso de los documentos relacionados con SAP, el escenario entre locales se reduce a una conectividad de sitio a sitio o [ExpressRoute](https://azure.microsoft.com/services/expressroute/), y al hecho de que la infraestructura de SAP se distribuye entre medios locales y Azure.

## <a name="resources"></a>Recursos
Hay otros artículos disponibles sobre la carga de trabajo SAP en Azure. Comience con [Carga de trabajo SAP en Azure: Introducción](./get-started.md) y, a continuación, elija el área de interés.

Las notas de SAP siguientes están relacionadas con SAP en Azure con respecto al área que se describe en este documento.

| Número de nota | Título |
| --- | --- |
| [1928533] |SAP applications on Azure: Supported Products and Azure VM types (Nota de SAP 1928533: Aplicaciones SAP en Azure: productos admitidos y tipos de máquina virtual de Azure) |
| [2015553] |SAP on Microsoft Azure: Support Prerequisites (Nota de soporte técnico 2015553 de SAP en Microsoft Azure: requisitos previos de soporte técnico) |
| [1999351] |Troubleshooting Enhanced Azure Monitoring for SAP (Solución de problemas de la supervisión mejorada de Azure para SAP) |
| [2178632] |Key Monitoring Metrics for SAP on Microsoft Azure (Métricas de supervisión clave para SAP en Microsoft Azure) |
| [1409604] |Virtualization on Windows: Enhanced Monitoring (SAP en Linux con Azure: supervisión mejorada) |
| [2191498] |SAP on Linux with Azure: Enhanced Monitoring (SAP en Linux con Azure: supervisión mejorada) |
| [2039619] |SAP Applications on Microsoft Azure using the Oracle Database: Supported products and versions (Aplicaciones de SAP en Microsoft Azure con Base de datos de Oracle: versiones y productos compatibles) |
| [2233094] |DB6: aplicaciones SAP en Azure con IBM DB2 para Linux, UNIX y Windows: Información adicional |
| [2243692] |Linux on Microsoft Azure (IaaS) VM: SAP license issues (Linux y máquinas virtuales de Microsoft Azure (IaaS): problemas de licencia de SAP) |
| [1984787] |SUSE LINUX Enterprise Server 12: Notas de instalación |
| [2002167] |Red Hat Enterprise Linux 7.x: Instalación y actualización |
| [2069760] |Instalación y actualización de SAP en Oracle Linux 7.x |
| [1597355] |Recomendación de espacio de intercambio para Linux |
| [2171857] |Oracle Database 12c: compatibilidad con sistema de archivos en Linux |
| [1114181] |Oracle Database 11g: compatibilidad con sistema de archivos en Linux |


Para información sobre todas las notas de SAP para Linux, consulte la [wiki de la comunidad SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes).

Debe tener conocimientos prácticos sobre la arquitectura de Microsoft Azure y cómo se implementan y usan las máquinas virtuales de Microsoft Azure. Para más información, consulte la [documentación de Azure](../../../index.yml).

En general, el proceso de instalación y configuración de DBMS, Windows y Linux es prácticamente el mismo en cualquier máquina virtual o máquina sin sistema operativo que se instala en un entorno local. Hay algunas decisiones de implementación de administración de sistemas y arquitecturas que difieren cuando se usa IaaS de Azure. En este documento se explican las diferencias de administración del sistema y arquitectura específicas que se deben tener en cuenta cuando se usa IaaS de Azure.


## <a name="storage-structure-of-a-vm-for-rdbms-deployments"></a><a name="65fa79d6-a85f-47ee-890b-22e794f51a64"></a>Estructura de almacenamiento de una máquina virtual para las implementaciones de RDBMS
Para seguir este capítulo, lea y comprenda la información presentada en [este capítulo][deployment-guide-3] de la [Guía de implementación][deployment-guide]. Antes de leer este capítulo, es necesario que comprenda y conozca las diferentes series de máquinas virtuales y las diferencias entre el almacenamiento estándar y premium. 

Para más información sobre Azure Storage para las máquinas virtuales de Azure, consulte [Introducción a los discos administrados para máquinas virtuales de Azure](../../managed-disks-overview.md).

En una configuración básica, normalmente se recomienda una estructura de implementación en la que el sistema operativo, el DBMS y los archivos binarios finales de SAP sean independientes de los archivos de base de datos. Se recomienda que los sistemas SAP que se ejecutan en máquinas virtuales de Azure tengan instalado el disco duro virtual base (o disco) con el sistema operativo, los archivos ejecutables del sistema de administración de bases de datos y los archivos ejecutables de SAP. 

Los archivos de datos y de registro de DBMS se almacenan en almacenamiento estándar o premium. Se almacenan en discos distintos y se asocian como discos lógicos a la máquina virtual de la imagen del sistema operativo de Azure original. En el caso de implementaciones de Linux, se documentan distintas recomendaciones, especialmente para SAP HANA.

Al planear el diseño del disco, encuentre el mejor equilibrio entre estos elementos:

* El número de archivos de datos
* El número de discos que contienen los archivos.
* Las cuotas de IOPS de un solo disco.
* El rendimiento por disco.
* El número de discos de datos adicionales posibles por tamaño de máquina virtual.
* El rendimiento de almacenamiento global que puede proporcionar una máquina virtual
* La latencia que pueden proporcionar los diferentes tipos de Azure Storage.
* SLA de máquina virtual.

Azure impone una cuota de IOPS por disco de datos. Estas cuotas son diferentes para los discos hospedados en almacenamiento estándar y almacenamiento premium. La latencia de E/S también es diferente entre los dos tipos de almacenamiento. El almacenamiento premium ofrece una mejor latencia de E/S. 

Cada uno de los distintos tipos de máquina virtual tiene un número limitado de discos de datos que se pueden asociar. Otra restricción es que solo determinados tipos de máquinas virtuales pueden usar almacenamiento premium. Normalmente, la decisión de usar un determinado tipo de máquina virtual se basa en los requisitos de CPU y memoria. También puede considerar los requisitos de IOPS, latencia y rendimiento de disco que normalmente se escalan con el número de discos o el tipo de discos de almacenamiento premium. El número de IOPS y el rendimiento que se debe conseguir por cada disco podrían determinar el tamaño del disco, especialmente con el almacenamiento premium.

> [!NOTE]
> Para las implementaciones de DBMS, se recomienda el uso de almacenamiento premium para los archivos de datos, registro de transacciones o fase de puesta al día. No importa si se quieren implementar sistemas de producción o que no sean de producción.

> [!NOTE]
> Para beneficiarse del [contrato de nivel de servicio de máquina virtual única](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/), todos los discos que están asociados deben tener el tipo de almacenamiento premium, que incluye el disco duro virtual base.

> [!NOTE]
> No se permite hospedar archivos de bases de datos principales (archivos de datos y de registro) de bases de datos SAP en hardware de almacenamiento ubicado en centros de datos de terceros colocados y subyacentes a centros de datos de Azure. Para cargas de trabajo SAP, solo se admite el almacenamiento representado como un servicio nativo de Azure para los archivos de datos y de registro de transacciones de bases de datos SAP.

La colocación de los archivos de base de datos y los archivos de registro y fase de puesta al día, y el tipo de almacenamiento de Azure que se use, se debe definir en función de los requisitos de IOPS, latencia y rendimiento. Con el fin de disponer de IOPS suficiente, es posible que tenga que usar varios discos o un disco de almacenamiento premium de mayor tamaño. Si usa varios discos, cree una sección de software en todos los discos que contenga los archivos de datos o los archivos de registro y fase de puesta al día. En esos casos, los contratos de nivel de servicio de IOPS y rendimiento de los discos de almacenamiento premium subyacentes o la cantidad máxima de IOPS que se puede alcanzar de los discos de almacenamiento estándar son acumulativos para el espacio seccionado resultante.

Como ya se ha indicado, si el requisito de IOPS es superior a lo que puede proporcionar un solo disco duro virtual, equilibre el número de operaciones de IOPS necesarias para los archivos de base de datos entre varios discos duros virtuales. La forma más sencilla de distribuir la carga de IOPS entre los discos consiste en crear una sección de software por los diferentes discos. Después, coloque una serie de archivos de datos del DBMS de SAP en los LUN extraídos de la sección de software. El número de discos de la sección está controlado por las demandas de IOPS, rendimiento y volumen.


---
> ![Windows][Logo_Windows] Windows
>
> Se recomienda usar espacios de almacenamiento de Windows para crear conjuntos seccionados entre varios discos duros virtuales de Azure. Use al menos Windows Server 2012 R2 o Windows Server 2016.
>
> ![Linux][Logo_Linux] Linux
>
> Para crear un software RAID en Linux, solo se admiten MDADM y LVM (Logical Volume Manager). Para más información, consulte:
>
> - [Configuración del software RAID en Linux](../../linux/configure-raid.md) con MDADM.
> - [Configuración del LVM en una máquina virtual Linux en Azure](../../linux/configure-lvm.md) con LVM.
>
>

---

> [!NOTE]
> Dado que Azure Storage mantiene tres imágenes de los discos duros virtuales, no tiene sentido configurar una redundancia al seccionar. Solo tiene que configurar el seccionamiento para que las operaciones de E/S se distribuyan entre los diferentes discos duros virtuales.
>

### <a name="managed-or-nonmanaged-disks"></a>Discos administrados o no administrados
Las cuentas de Azure Storage son una construcción administrativa y también están sujetas a limitaciones. Las limitaciones difieren entre cuentas de almacenamiento estándar y cuentas de almacenamiento premium. Para información sobre las funcionalidades y limitaciones, consulte [Objetivos de escalabilidad y rendimiento de Azure Storage](../../../storage/common/scalability-targets-standard-account.md).

Con el almacenamiento estándar, recuerde que hay un límite en el número de IOPS por cuenta de almacenamiento. Consulte la fila que contiene **Tasa de solicitud total** del artículo [Objetivos de escalabilidad y rendimiento de Azure Storage](../../../storage/common/scalability-targets-standard-account.md). Además, hay un límite inicial en el número de cuentas de almacenamiento por suscripción de Azure. Con infraestructuras de SAP más grandes, equilibre los discos duros virtuales entre diferentes cuentas de almacenamiento para evitar alcanzar los límites de estas cuentas de almacenamiento. Este es un trabajo tedioso cuando se trata de varios cientos de máquinas virtuales con más de mil discos duros virtuales.

Puesto que no se recomienda usar almacenamiento estándar en implementaciones de DBMS en combinación con una carga de trabajo de SAP, las referencias y recomendaciones para el almacenamiento estándar se limitan a este breve [artículo](/archive/blogs/mast/configuring-azure-virtual-machines-for-optimal-storage-performance).

Para evitar el trabajo administrativo de planear e implementar discos duros virtuales entre varias cuentas de Azure Storage, Microsoft presentó [Azure Managed Disks](https://azure.microsoft.com/services/managed-disks/) en 2017. Los discos administrados están disponibles para almacenamiento estándar y almacenamiento premium. Las principales ventajas de los discos administrados en comparación con los discos no administrados son las siguientes:

- Para los discos administrados, Azure distribuye los diferentes discos duros virtuales entre diferentes cuentas de almacenamiento de forma automática durante la implementación. De esta manera, no se alcanzan los límites de la cuenta de almacenamiento en cuanto al volumen de datos, el rendimiento de E/S y el número de IOPS.
- Con los discos administrados, Azure Storage hace honor a los conceptos de conjuntos de disponibilidad de Azure. Si la máquina virtual forma parte de un conjunto de disponibilidad de Azure, el disco duro virtual base y el disco asociado de una máquina virtual se implementan en diferentes dominios de error y actualización.


> [!IMPORTANT]
> Dadas las ventajas de Azure Managed Disks, se recomienda usar Azure Managed Disks para las implementaciones de DBMS y las de SAP en general.
>

Para convertir discos no administrados a discos administrados, consulte:

- [Conversión de una máquina virtual Windows con discos no administrados a discos administrados](../../windows/convert-unmanaged-to-managed-disks.md).
- [Conversión de una máquina virtual Linux con discos no administrados a discos administrados](../../linux/convert-unmanaged-to-managed-disks.md).


### <a name="caching-for-vms-and-data-disks"></a><a name="c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f"></a>Almacenamiento en caché de máquinas virtuales y discos de datos
Al montar discos en máquinas virtuales, se puede elegir almacenar en caché el tráfico de E/S entre la máquina virtual y esos discos ubicados en Azure Storage. El almacenamiento estándar y el almacenamiento premium emplean dos tecnologías diferentes para este tipo de almacenamiento en caché.

En las recomendaciones siguientes se da por hecho estas características de E/S para DBMS estándar:

- Es principalmente una carga de trabajo de lectura en los archivos de datos de una base de datos. Estas operaciones de lectura son críticas para el rendimiento del sistema DBMS.
- La escritura en los archivos de datos se produce en ráfagas en función de puntos de comprobación o un flujo constante. Se promedia durante un día, hay menos escrituras que lecturas. Al contrario que las operaciones de lectura de archivos de datos, estas operaciones de escritura son asincrónicas y no bloquean transacciones de usuario.
- Apenas hay lecturas del registro de transacciones o de los archivos de fase de puesta al día. Las excepciones son las operaciones de E/S de gran tamaño al realizar copias de seguridad del registro de transacciones.
- La carga principal en los archivos de registro de transacciones o de fase de puesta al día es una operación de escritura. Según la naturaleza de la carga de trabajo, puede tener operaciones de E/S de solo 4 KB o llegar a superar 1 MB.
- Todas las operaciones de escritura se deben conservar en el disco de forma confiable.

Para el almacenamiento estándar, estos son los tipos posibles de caché:

* None
* Lectura
* Lectura/Escritura

Para obtener un rendimiento coherente y determinista, establezca el almacenamiento en caché del almacenamiento estándar para todos los discos que contengan archivos de datos, archivos de registro y fase de puesta al día, y espacio de tablas relacionados con DBMS en **NINGUNO**. Se puede conservar el valor de almacenamiento en caché predeterminado del disco duro virtual base.

Para el almacenamiento premium, existen las opciones de almacenamiento en caché siguientes:

* None
* Lectura
* Lectura/escritura
* Ninguno + Acelerador de escritura, que es solo para máquinas virtuales de la serie M de Azure
* Lectura + Acelerador de escritura, que es solo para máquinas virtuales de la serie M de Azure

Para el almacenamiento premium, se recomienda usar el **almacenamiento en caché de lectura para archivos de datos** de la base de datos de SAP y no elegir **almacenamiento en caché para los discos de archivos de registro**.

Para las implementaciones de la serie M, se recomienda usar el Acelerador de escritura de Azure para la implementación de DBMS. Para más información, incluidas las restricciones y la implementación del Acelerador de escritura de Azure, consulte [Habilitar el acelerador de escritura](../../how-to-enable-write-accelerator.md).


### <a name="azure-nonpersistent-disks"></a>Discos de Azure no persistentes
Las máquinas virtuales de Azure ofrecen discos no persistentes después de implementar una máquina virtual. En el caso de un reinicio de la máquina virtual, se borrará todo el contenido de esas unidades. Es evidente que los archivos de datos y los archivos de registro y fase de puesta al día de las bases de datos no se deben colocar bajo ninguna circunstancia en esas unidades de disco no persistentes. Es posible que haya excepciones para algunas de las bases de datos, donde estas unidades de disco no persistentes podrían ser adecuadas para espacios de tablas tempdb y temp. Evite usar esas unidades con máquinas virtuales de la serie A, ya que esas unidades no persistentes tienen un límite de rendimiento con esa familia de máquinas virtuales. 

Para más información, consulte [Understand the temporary drive on Windows VMs in Azure](/archive/blogs/mast/understanding-the-temporary-drive-on-windows-azure-virtual-machines) (Comprender la unidad temporal en máquinas virtuales Windows en Azure).

---
> ![Windows][Logo_Windows] Windows
>
> La unidad D:\ de una máquina virtual de Azure es una unidad no persistente respaldada por algunos discos locales del nodo de ejecución de Azure. Como es persistente, todos los cambios realizados en el contenido de la unidad D se pierden cuando se reinicia la máquina virtual. Los cambios incluyen archivos que se almacenaron, directorios que se crearon y aplicaciones que se instalaron.
>
> ![Linux][Logo_Linux] Linux
>
> Las máquinas virtuales Linux de Azure montan automáticamente una unidad en /mnt/resource, que es una unidad no persistente respaldada por discos locales del nodo de ejecución de Azure. Como no es persistente, los cambios realizados en el contenido en /mnt/resource se pierden cuando se reinicia la máquina virtual. Los cambios incluyen archivos que se almacenaron, directorios que se crearon y aplicaciones que se instalaron.
>
>

---



### <a name="microsoft-azure-storage-resiliency"></a><a name="10b041ef-c177-498a-93ed-44b3441ab152"></a>Resistencia de Microsoft Azure Storage
Microsoft Azure Storage almacena el disco duro virtual base (con SO) y blobs o los discos asociados en, al menos, tres nodos de almacenamiento independientes. Este tipo de almacenamiento se denomina almacenamiento con redundancia local (LRS). LRS es el valor predeterminado para todos los tipos de almacenamiento en Azure.

Hay otros métodos de redundancia. Para más información, consulte [Replicación de Azure Storage](../../../storage/common/storage-redundancy.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json).

> [!NOTE]
>El almacenamiento premium es el tipo de almacenamiento recomendado para máquinas virtuales de DBMS y los discos que almacenan los archivos de base de datos, registro y fase de puesta al día. El único método de redundancia disponible para el almacenamiento premium es LRS. Como resultado, debe configurar métodos de base de datos para habilitar la replicación de datos de base de datos en otra zona de disponibilidad o región de Azure. Los métodos de base de datos incluyen SQL Server Always On, Oracle Data Guard y replicación del sistema HANA.


> [!NOTE]
> Para implementaciones de DBMS, no se recomienda el uso de almacenamiento con redundancia geográfica (GRS) para el almacenamiento estándar. GRS afecta gravemente al rendimiento y no respeta el orden de escritura entre diferentes discos duros virtuales que están asociados a una máquina virtual. El hecho de no respetarse el orden de escritura entre diferentes discos duros virtuales, puede llevar a incoherencias de las bases de datos en el destino de replicación. Esta situación se produce si los archivos de base de datos, registro y fase de puesta al día se reparten entre varios discos duros virtuales, como suele ser el caso, en la máquina virtual de origen.



## <a name="vm-node-resiliency"></a>Resistencia de nodos de máquina virtual
Azure ofrece varios contratos de nivel de servicio distintos para las máquinas virtuales. Para más información, consulte la versión más reciente de [Contrato de nivel de servicio para Máquinas virtuales](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/). Dado que la capa de DBMS es normalmente fundamental para la disponibilidad en un sistema SAP, debe comprender los conjuntos disponibilidad, las zonas de disponibilidad y los eventos de mantenimiento. Para más información sobre estos conceptos, consulte [Administración de la disponibilidad de las máquinas virtuales Windows en Azure](../../windows/manage-availability.md) y [Administración de la disponibilidad de las máquinas virtuales Linux en Azure](../../linux/manage-availability.md).

La recomendación mínima para escenarios DBMS de producción con una carga de trabajo SAP es:

- Implemente dos máquinas virtuales en un conjunto de disponibilidad independiente en la misma región de Azure.
- Ejecute estas dos máquinas virtuales en la misma red virtual de Azure y cuente con NIC asociadas fuera de las mismas subredes.
- Use los métodos de la base de datos para mantener una espera activa con la segunda máquina virtual. Los métodos pueden ser SQL Server Always On, Oracle Data Guard o Replicación del sistema de HANA.

También puede implementar una tercera máquina virtual en otra región de Azure y usar los mismos métodos de base de datos para proporcionar una réplica asincrónica en otra región de Azure.

Para información sobre cómo configurar conjuntos de disponibilidad de Azure, consulte [este tutorial](../../windows/tutorial-availability-sets.md).



## <a name="azure-network-considerations"></a>Consideraciones sobre la red de Azure
En las implementaciones de SAP a gran escala, use el proyecto de [Azure Virtual Datacenter](/azure/architecture/vdc/networking-virtual-datacenter). Utilícelo para la configuración de la red virtual y las asignaciones de roles y permisos a diferentes partes de su organización.

Estos procedimientos recomendados son el resultado de cientos de implementaciones de clientes:

- Las redes virtuales en las que se implementa la aplicación de SAP no tienen acceso a Internet.
- Las máquinas virtuales de base de datos se ejecutan en la misma red virtual que la capa de aplicación.
- Las máquinas virtuales dentro de la red virtual tienen una asignación estática de la dirección IP privada. Para más información, consulte [Tipos de direcciones IP y métodos de asignación en Azure](../../../virtual-network/public-ip-addresses.md).
- Las restricciones de enrutamiento a y desde las máquinas virtuales de DBMS *no* se establecen con firewalls instalados en las máquinas virtuales de DBMS locales. En su lugar, el enrutamiento del tráfico se define mediante [grupos de seguridad de red (NSG)](../../../virtual-network/security-overview.md).
- Para separar y aislar el tráfico a la máquina virtual de DBMS, asigne diferentes NIC a la máquina virtual. Cada NIC recibe una dirección IP diferente y cada NIC se asigna a una subred de red virtual diferente. Cada subred tiene diferentes reglas de NSG. El aislamiento o la separación del tráfico de red es una medida para el enrutamiento. No se usa para establecer cuotas para el rendimiento de la red.

> [!NOTE]
> Asignar direcciones IP estáticas mediante Azure significa asignarlas NIC virtuales individuales. No asigne direcciones IP estáticas dentro del sistema operativo invitado a una NIC virtual. Algunos servicios de Azure como Azure Backup se basan en el hecho de que al menos la NIC virtual principal está establecida en DHCP y no en direcciones IP estáticas. Para más información, consulte [Solución de problemas de copia de seguridad de máquinas virtuales de Azure](../../../backup/backup-azure-vms-troubleshoot.md#networking). Para asignar varias direcciones IP estáticas a una máquina virtual, asigne varias NIC virtuales a una máquina virtual.
>


> [!IMPORTANT]
> No se admite la configuración de [aplicaciones virtuales de red](https://azure.microsoft.com/solutions/network-appliances/) en la ruta de comunicación entre la aplicación de SAP y la capa de DBMS de un sistema SAP basado en SAP NetWeaver, Hybris o S/4HANA. Esta restricción es por motivos de rendimiento y funcionalidad. La comunicación entre la capa de la aplicación de SAP y la capa de DBMS debe ser directa. La restricción no incluye las [ reglas del grupo de seguridad de aplicaciones (ASG) y las reglas NSG](../../../virtual-network/security-overview.md) si estas permiten una ruta de acceso de comunicación directa. 
>
> Otros escenarios donde no se admiten aplicaciones virtuales de red se encuentran en:
>
> * Rutas de comunicación entre máquinas virtuales de Azure que representan los nodos de clúster de Linux Pacemaker y los dispositivos SBD, según se describe en [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications](./high-availability-guide-suse.md).
> * Rutas de comunicación entre máquinas virtuales de Azure y el Servidor de archivos de escalabilidad horizontal (SOFS) de Windows Server configuradas como se describe en [Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un recurso compartido de archivos en Azure](./sap-high-availability-guide-wsfc-file-share.md). 
>
> Las aplicaciones virtuales de red de las rutas de comunicación pueden duplicar fácilmente la latencia de red entre dos asociados de comunicación. También pueden restringir el rendimiento en las rutas críticas entre la capa de aplicación de SAP y la capa de DBMS. En algunos escenarios de clientes, las aplicaciones virtuales de red pueden provocar un error en los clústeres de Linux Pacemaker. Hay casos donde las comunicaciones entre los nodos del clúster de Linux Pacemaker se comunican con su dispositivo SBD mediante una aplicación virtual de red.
>

> [!IMPORTANT]
> Otro diseño que *no* se admite es la segregación de la capa de la aplicación de SAP ni la capa de DBMS en diferentes redes virtuales de Azure que no están [emparejadas](../../../virtual-network/virtual-network-peering-overview.md) entre sí. Se recomienda separar la capa de aplicación de SAP y la capa de DBMS mediante subredes dentro de una red virtual de Azure, en lugar de usar diferentes redes virtuales de Azure. 
>
> Si decide no seguir la recomendación y separa las dos capas en redes virtuales diferentes, las dos redes virtuales deben estar [emparejadas](../../../virtual-network/virtual-network-peering-overview.md). 
>
> Tenga en cuenta que el tráfico entre dos redes virtuales de Azure [emparejadas](../../../virtual-network/virtual-network-peering-overview.md) está sujeto a costos de transferencia. Un enorme volumen de datos que consta de muchos terabytes se intercambia entre la capa de aplicación de SAP y la capa de DBMS. Puede acumular costos sustanciales si la capa de aplicación de SAP y la capa de DBMS se separan entre dos redes virtuales de Azure emparejadas.

Use dos máquinas virtuales para la implementación de DBMS en producción dentro de un conjunto de disponibilidad de Azure. Use también enrutamiento independiente para la capa de aplicación de SAP y el tráfico de administración y operaciones a las dos máquinas virtuales de DBMS. Consulte la siguiente imagen:

![Diagrama de dos máquinas virtuales en dos subredes](./media/virtual-machines-shared-sap-deployment-guide/general_two_dbms_two_subnets.PNG)


### <a name="use-azure-load-balancer-to-redirect-traffic"></a>Uso de Azure Load Balancer para redirigir el tráfico
El uso de direcciones IP virtuales privadas en funcionalidades como SQL Server Always On o Replicación de sistema de HANA requiere la configuración de una instancia de Azure Load Balancer. El equilibrador de carga usa puertos de sondeo para determinar el nodo de DBMS activo y enrutar el tráfico exclusivamente a ese nodo de base de datos activo. 

Si hay una conmutación por error del nodo de base de datos, no hay necesidad de volver a configurar la aplicación de SAP. En su lugar, las arquitecturas de aplicaciones de SAP más comunes se vuelven a conectar con la dirección IP virtual privada. Mientras tanto, el equilibrador de carga reacciona a la conmutación por error del nodo mediante la redirección del tráfico con la dirección IP virtual privada al segundo nodo.

Azure ofrece dos [SKU de equilibrador de carga](../../../load-balancer/load-balancer-overview.md) diferentes: una SKU básica y una SKU estándar. A menos que quiera implementar en varias zonas de disponibilidad de Azure, la SKU de equilibrador de carga básica es suficiente.

¿El tráfico entre las máquinas virtuales de DBMS y la capa de aplicación de SAP siempre se enruta a través del equilibrador? La respuesta depende de cómo se configure el equilibrador de carga. 

En este momento, el tráfico entrante a la máquina virtual de DBMS siempre se enruta a través del equilibrador de carga. La ruta de tráfico saliente desde la máquina virtual de DBMS hasta la máquina virtual de la capa de aplicación depende de la configuración del equilibrador de carga. 

El equilibrador de carga ofrece una opción de DirectServerReturn. Si se configura esa opción, el tráfico dirigido desde la máquina virtual de DBMS hasta la capa de aplicación de SAP *NO* se enruta a través del equilibrador de carga. En su lugar, pasa directamente a la capa de aplicación. Si no se configura DirectServerReturn, el tráfico de retorno a la capa de aplicación de SAP se enruta a través del equilibrador de carga.

Se recomienda configurar DirectServerReturn en combinación con equilibradores de carga que estén colocados entre la capa de aplicación de SAP y la capa de DBMS. Esta configuración reduce la latencia de red entre las dos capas.

Para ver un ejemplo de cómo realizar esta configuración con SQL Server Always On, consulte [Configuración de un cliente de escucha ILB para grupos de disponibilidad AlwaysOn en Azure](/previous-versions/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener).

Si usa las plantillas JSON publicadas en GitHub como referencia para las implementaciones de infraestructura SAP en Azure, debe estudiar esta [plantilla para un sistema SAP de tres niveles](https://github.com/Azure/azure-quickstart-templates/tree/4099ad9bee183ed39b88c62cd33f517ae4e25669/sap-3-tier-marketplace-image-converged-md). En esta plantilla, también puede ver la configuración correcta del equilibrador de carga.

### <a name="azure-accelerated-networking"></a>Redes aceleradas de Azure
Para reducir aún más la latencia de red entre máquinas virtuales de Azure, se recomienda elegir [Redes aceleradas de Azure](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/). Use esta opción al implementar máquinas virtuales de Azure para una carga de trabajo SAP, especialmente para la capa de aplicación de SAP y la capa de DBMS de SAP.

> [!NOTE]
> No todos los tipos de máquina virtual admiten redes aceleradas. En el artículo anterior se enumeran los tipos de máquina virtual que admiten redes aceleradas.
>

---
> ![Windows][Logo_Windows] Windows
>
> Para información sobre cómo implementar máquinas virtuales con redes aceleradas para Windows, consulte [Creación de una máquina virtual Windows con redes aceleradas](../../../virtual-network/create-vm-accelerated-networking-powershell.md).
>
> ![Linux][Logo_Linux] Linux
>
> Para más información sobre la distribución de Linux, consulte [Creación de una máquina virtual Linux con redes aceleradas](../../../virtual-network/create-vm-accelerated-networking-cli.md).
>
>

---

> [!NOTE]
> En el caso de SUSE, Red Hat y Oracle Linux, Redes aceleradas se admite con las versiones recientes. Las versiones anteriores como SLES 12 SP2 o RHEL 7.2 no son compatibles con redes aceleradas de Azure.
>


## <a name="deployment-of-host-monitoring"></a>Implementación de la supervisión del host
Para el uso en producción de aplicaciones de SAP en máquinas virtuales de Azure, SAP necesita que se puedan obtener datos de supervisión de los hosts físicos que se ejecutan en dichas máquinas. Se requiere un nivel de revisión específico del agente de host de SAP que permita esta funcionalidad en SAPOSCOL y el agente de host de SAP. El nivel de revisión exacto se menciona en la nota de SAP [1409604].

Para más información sobre la implementación de componentes que proporcionen datos de host a SAPOSCOL y el agente de host de SAP, así como sobre la administración del ciclo de vida de esos componentes, vea la [Guía de implementación][deployment-guide].


## <a name="next-steps"></a>Pasos siguientes
Para más información sobre un DBMS concreto, consulte:

- [Implementación de DBMS de Azure Virtual Machines de SQL Server para una carga de trabajo de SAP](dbms_guide_sqlserver.md)
- [Implementación de DBMS de Azure Virtual Machines de Oracle para una carga de trabajo de SAP](dbms_guide_oracle.md)
- [Implementación de DBMS de Azure Virtual Machines de IBM DB2 para una carga de trabajo de SAP](dbms_guide_ibm.md)
- [Implementación de DBMS de Azure Virtual Machines de SAP ASE para una carga de trabajo de SAP](dbms_guide_sapase.md)
- [Implementación de SAP maxDB, Live Cache y del servidor de contenido en Azure](dbms_guide_maxdb.md)
- [SAP HANA en la guía de operaciones de Azure](hana-vm-operations.md)
- [Alta disponibilidad de SAP HANA para máquinas virtuales de Azure](sap-hana-availability-overview.md)
- [Guía de copia de seguridad de SAP HANA en Azure Virtual Machines](sap-hana-backup-guide.md)