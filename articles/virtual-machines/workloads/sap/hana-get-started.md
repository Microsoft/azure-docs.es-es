---
title: Instalación de SAP HANA en máquinas virtuales de Azure | Microsoft Docs
description: Guía de instalación de SAP HANA en máquinas virtuales de Azure
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: c51a2a06-6e97-429b-a346-b433a785c9f0
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/19/2020
ms.author: juergent
ms.openlocfilehash: f2a62cb08fcce6597f02c080231f5e1808794054
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88509967"
---
# <a name="installation-of-sap-hana-on-azure-virtual-machines"></a>Instalación de SAP HANA en máquinas virtuales de Azure
## <a name="introduction"></a>Introducción
Esta guía le ayuda a apuntar a los recursos adecuados para implementar HANA en máquinas virtuales de Azure correctamente. Esta guía le indicará recursos de documentación que debe consultar antes de instalar SAP HANA en una máquina virtual de Azure. De este modo, podrá realizar los pasos adecuados para, finalmente, tener una configuración compatible de SAP HANA en máquinas virtuales de Azure.  

> [!NOTE]
> En ella se describen las implementaciones de SAP HANA en máquinas virtuales de Azure. Para obtener información sobre cómo implementar SAP HANA en instancias grandes de HANA, consulte [Instalación y configuración de SAP HANA (instancias grandes) en Azure](./hana-installation.md).
 
## <a name="prerequisites"></a>Prerrequisitos
También se da por supuesto que está familiarizado con:
* SAP HANA y SAP NetWeaver y cómo instalarlos en el entorno local.
* Cómo instalar y usar SAP HANA y las instancias de aplicación SAP en Azure.
* Los conceptos y procedimientos documentados en:
   * Planeamiento de la implementación de SAP en Azure, que incluye el planeamiento de Azure Virtual Network y el uso de Azure Storage. Consulte [SAP NetWeaver en Azure Virtual Machines: guía de planeamiento e implementación](./planning-guide.md).
   * Principios de implementación y maneras de implementar máquinas virtuales en Azure. Consulte [Implementación de Azure Virtual Machines para SAP](./deployment-guide.md).
   * Los conceptos sobre alta disponibilidad para SAP HANA, tal como están documentados en [Alta disponibilidad de SAP HANA para máquinas virtuales de Azure](./sap-hana-availability-overview.md).

## <a name="step-by-step-before-deploying"></a>Guía paso a paso antes de la implementación
En esta sección se enumeran los distintos pasos que debe realizar antes de comenzar la instalación de SAP HANA en una máquina virtual de Azure. Están organizados de forma numérica y, por ello, deben seguirse en ese orden:

1. No todos los escenarios de implementación posibles son compatibles con Azure. Por lo tanto, debe consultar el documento [Carga de trabajo de SAP en escenarios admitidos en máquinas virtuales de Azure](./sap-planning-supported-configurations.md) para buscar el escenario que tiene planeado para su implementación de SAP HANA. Si el escenario no aparece en la lista, debe suponer que no se ha probado y, por lo tanto, no se admite.
2. Suponiendo que tiene una idea aproximada de los requisitos de memoria para la implementación de SAP HANA, debe buscar una máquina virtual de Azure de adecuada. No todas las máquinas virtuales certificadas para SAP NetWeaver están certificadas para SAP HANA, como está documentado en la [Nota de compatibilidad de SAP n.º 1928533](https://launchpad.support.sap.com/#/notes/1928533). El origen de confianza de las máquinas virtuales de Azure certificadas para SAP HANA es el sitio web del [directorio de hardware de SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Las unidades que comienzan por **S** son unidades de [instancias grandes de HANA](./hana-overview-architecture.md), no máquinas virtuales de Azure.
3. Los diferentes tipos de máquinas virtuales de Azure tienen versiones mínimas de sistema operativo diferentes para SUSE Linux o Red Hat Linux. En el sitio web del [directorio de hardware de SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure), debe hacer clic en una entrada de la lista de unidades certificadas para SAP HANA para obtener información detallados sobre la unidad. Además de la carga de trabajo de HANA admitida, la lista incluye las versiones del sistema operativo compatibles con esas unidades para SAP HANA
4. En el caso de las versiones del sistema operativo, debe plantearse ciertas versiones mínimas del kernel. Estas versiones mínimas están documentadas en las siguientes notas de compatibilidad de SAP:
    - [SAP support note #2814271 SAP HANA Backup fails on Azure with Checksum Error](https://launchpad.support.sap.com/#/notes/2814271) (Nota de compatibilidad de SAP n.º 2814271: no se puede realizar la copia de seguridad de SAP HANA en Azure debido a un error de suma de comprobación)
    - [SAP support note #2753418 Potential Performance Degradation Due to Timer Fallback ](https://launchpad.support.sap.com/#/notes/2753418) (Nota de compatibilidad de SAP n.º 2753418: posible reducción del rendimiento debido a la reserva del temporizador)
    - [SAP support note #2791572 Performance Degradation Because of Missing VDSO Support For Hyper-V in Azure](https://launchpad.support.sap.com/#/notes/2791572) (Nota de compatibilidad de SAP n.º 2791572: reducción del rendimiento debido a la falta de compatibilidad de VDSO con Hyper-V en Azure)
4. En función de la versión del sistema operativo que sea compatible con el tipo de máquina virtual que elija, compruebe si la versión de SAP HANA deseada es compatible con esa versión del sistema operativo. Consulte la [nota de compatibilidad de SAP n.º 2235581](https://launchpad.support.sap.com/#/notes/2235581) para ver una matriz de compatibilidad entre las versiones de SAP HANA y las distintas versiones de un sistema operativo.
5. Ya que tal vez encontró una combinación válida de tipo de máquina virtual de Azure, versión del sistema operativo y versión de SAP HANA, debe consultar la Matriz de disponibilidad de productos SAP. En la Matriz de disponibilidad de SAP, puede averiguar si se admite el producto de SAP que quiere ejecutar en la base de datos de SAP HANA.


## <a name="step-by-step-vm-deployment-and-guest-os-considerations"></a>Guía detallada para implementar la VM y consideraciones sobre el sistema operativo invitado
En esta fase, debe seguir los pasos de implementación de las máquinas virtuales para instalar HANA y, finalmente, optimizar el sistema operativo elegido después de la instalación.

1. Seleccione la imagen base de la galería de Azure. Si quiere crear su propia imagen de sistema operativo para SAP HANA, debe conocer todos los paquetes necesarios para instalar SAP HANA correctamente. En caso contrario, se recomienda usar las imágenes de SUSE y Red Hat para SAP o SAP HANA de la galería de imágenes de Azure. Estas imágenes incluyen los paquetes necesarios para instalar HANA correctamente. En función del contrato de soporte técnico con el proveedor del sistema operativo, debe elegir una imagen en la que traiga su propia licencia. O bien, elija una imagen de sistema operativo que incluya soporte técnico.
2. Si eligió una imagen de SO invitado que requiere que traiga su propia licencia, debe registrar la imagen del sistema operativo con su suscripción. De este modo, puede descargar y aplicar las revisiones más recientes. Para este paso necesitará acceso a la red pública de Internet. A menos que configure una instancia privada, por ejemplo, del servidor de SMT en Azure.
3. Determine la configuración de red de la máquina virtual. Puede obtener más información en el documento [Configuraciones y operaciones de infraestructura de SAP HANA en Azure](./hana-vm-operations.md). Tenga en cuenta que no hay ninguna cuota de rendimiento de red que pueda asignar a las tarjetas de red virtual en Azure. Como resultado, solo dirigirá el tráfico a través de vNIC diferentes debido a las consideraciones de seguridad. Confiamos en encontrar un compromiso compatible con la complejidad del enrutamiento del tráfico a través de varias vNIC y los requisitos que exige la seguridad.
3. Aplique las revisiones más recientes al sistema operativo una vez que la máquina virtual se haya implementado y registrado. Regístrela con su propia suscripción. O bien, en caso de que haya elegido una imagen con soporte técnico de sistema operativo, la máquina virtual ya tendrá acceso a las revisiones. 
4. Aplique las optimizaciones necesarias para SAP HANA. Estas optimizaciones se incluyen en estas notas de compatibilidad de SAP:

    - [Nota de soporte técnico de SAP 2694118 - Red Hat Enterprise Linux HA Add-On on Azure](https://launchpad.support.sap.com/#/notes/2694118) (Complemento de alta disponibilidad de Red Hat Enterprise Linux en Azure)
    - [Nota de soporte técnico de SAP 1984787 - SUSE LINUX Enterprise Server 12: Installation notes](https://launchpad.support.sap.com/#/notes/1984787) (SUSE LINUX Enterprise Server 12: notas de instalación) 
    - [Nota de soporte técnico de SAP 2578899 – SUSE Linux Enterprise Server 15: Installation Note](https://launchpad.support.sap.com/#/notes/2578899) (SUSE Linux Enterprise Server 15: nota de instalación)
    - [SAP support note #2002167 - Red Hat Enterprise Linux 7.x: Installation and Upgrade](https://launchpad.support.sap.com/#/notes/0002002167) (Red Hat Enterprise Linux 7.x: instalación y actualización)
    - [SAP support note #2292690 - SAP HANA DB: Recommended OS settings for RHEL 7](https://launchpad.support.sap.com/#/notes/0002292690) (2292690 - SAP HANA DB: configuraciones de sistema operativo recomendadas para RHEL 7) 
    -  [Nota de soporte técnico de SAP 2772999 - Red Hat Enterprise Linux 8.x: Installation and Configuration](https://launchpad.support.sap.com/#/notes/2772999) (Red Hat Enterprise Linux 8.x: instalación y configuración) 
    -  [Nota de soporte técnico de SAP 2777782 - SAP HANA DB: Recommended OS settings for RHEL 8](https://launchpad.support.sap.com/#/notes/2777782) (SAP HANA DB: configuración recomendada del sistema operativo para RHEL 8).
    -  [SAP support note #2455582 - Linux: Running SAP applications compiled with GCC 6.x](https://launchpad.support.sap.com/#/notes/0002455582) (Nota de compatibilidad de SAP n.º 2455582 - Linux: Ejecución de aplicaciones SAP compiladas con GCC 6.x)
    -  [Nota de soporte técnico de SAP 2382421 - Optimizing the Network Configuration on HANA- and OS-Level](https://launchpad.support.sap.com/#/notes/2382421) (Optimización de la configuración de red en los niveles de HANA y de sistema operativo)

1. Seleccione el tipo de almacenamiento de Azure para SAP HANA. En este paso, debe decidir el diseño del almacenamiento para la instalación de SAP HANA. Deberá usar discos de Azure conectados o recursos compartidos de NFS de Azure nativos. Los tipos de almacenamiento de Azure admitidos y las combinaciones de diferentes tipos de almacenamiento de Azure que puede usar están documentados en [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md). Use las configuraciones documentadas como punto inicial. En el caso de los sistemas que no son de producción, es posible que pueda configurar un rendimiento o IOPS inferiores. En el caso de la producción, es posible que necesite configurar un poco más de rendimiento e IOPS.
2. Asegúrese de que haber configurado el [Acelerador de escritura de Azure](../../how-to-enable-write-accelerator.md) para los volúmenes que contienen los registros de transacciones de DBMS o los registros de la fase de puesta al día al usar máquinas virtuales de la serie M o Mv2. Tenga en cuenta las limitaciones documentadas del Acelerador de escritura.
2. Compruebe si las [redes aceleradas de Azure](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/) están habilitadas en las máquinas virtuales implementadas.

> [!NOTE]
> Es posible que no todos los comandos descritos en las notas o incluidos en los distintos perfiles de optimización de SAP se ejecuten correctamente en Azure. Los comandos que manipulan el modo de energía de las máquinas virtuales normalmente devuelven un error, ya que no se puede manipular el modo de energía del hardware del host de Azure subyacente.

## <a name="step-by-step-preparations-specific-to-azure-virtual-machines"></a>Preparación paso a paso específica de las máquinas virtuales de Azure
Uno de los requisitos específicos de Azure es la instalación de una extensión de máquina virtual de Azure que proporciona datos de supervisión al agente de host de SAP. Los detalles sobre la instalación de esta extensión de supervisión están documentados en:

-  En [SAP Note 2191498](https://launchpad.support.sap.com/#/notes/2191498/E) (Nota de SAP 2191498) se describe la supervisión mejorada de SAP con máquinas virtuales Linux en Azure. 
-  En [SAP Note 1102124](https://launchpad.support.sap.com/#/notes/1102124/E) (Nota de SAP 1102124) se describe información sobre SAPOSCOL en Linux. 
-  En [SAP Note 2178632](https://launchpad.support.sap.com/#/notes/2178632/E) (Nota de SAP 2178632) se describen las métricas de supervisión clave para SAP en Microsoft Azure.
-  [Implementación de Azure Virtual Machines para SAP NetWeaver](./deployment-guide.md#d98edcd3-f2a1-49f7-b26a-07448ceb60ca)

## <a name="sap-hana-installation"></a>Instalación de SAP HANA
Una vez que las máquinas virtuales se hayan implementado y los sistemas operativos se hayan registrado y configurado, podrá instalar SAP HANA en función de la instalación de SAP. Como buen punto de partida con esta documentación, comience con el sitio web de SAP para [recursos de HANA](https://www.sap.com/products/hana/implementation/resources.html).

Para obtener configuraciones de escalabilidad horizontal para SAP HANA que usan discos de Azure Premium Storage o Ultra conectados directamente, consulte los detalles en el documento [Configuraciones y operaciones de infraestructura de SAP HANA en Azure](./hana-vm-operations.md#configuring-azure-infrastructure-for-sap-hana-scale-out).


## <a name="additional-resources-for-sap-hana-backup"></a>Recursos adicionales para la copia de seguridad de SAP HANA
Para más información sobre cómo hacer una copia de seguridad de bases de datos SAP HANA en máquinas virtuales de Azure, consulte:
* [Guía de copia de seguridad de SAP HANA en Azure Virtual Machines](./sap-hana-backup-guide.md)
* [Azure Backup de SAP HANA en el nivel de archivo](./sap-hana-backup-file-level.md)

## <a name="next-steps"></a>Pasos siguientes
Consulte la documentación:

- [Configuraciones y operaciones de infraestructura de SAP HANA en Azure](./hana-vm-operations.md)
- [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md)
