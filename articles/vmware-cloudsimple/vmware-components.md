---
title: Componentes de VMware de nube privada
titleSuffix: Azure VMware Solution by CloudSimple
description: Vea cómo el servicio CloudSimple permite implementar de forma nativa VMware en ubicaciones de Azure. Las nubes privadas se integran con el resto de la nube de Azure.
author: sharaths-cs
ms.author: dikamath
ms.date: 08/15/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: ff9aec1904be754990958869666e9d67038e4fb3
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88142509"
---
# <a name="private-cloud-vmware-components"></a>Componentes de VMware de nube privada

Una nube privada es un entorno de pila de VMware aislado (hosts ESXi, vCenter, vSAN y NSX) que administra un servidor vCenter en un dominio de administración.  El servicio CloudSimple permite implementar de forma nativa VMware en una infraestructura de Azure sin sistema operativo en ubicaciones de Azure.  Las nubes privadas se integran con el resto de la nube de Azure.  Una nube privada se implementa con los siguientes componentes de la pila de VMware:

* **VMware ESXi:** hipervisor en nodos dedicados de Azure
* **VMware vCenter:** dispositivo para la administración centralizada del entorno de nube privada vSphere
* **vSAN de VMware:** solución de infraestructura hiperconvergida
* **NSX Data Center de VMware:** virtualización de redes y software de seguridad  

## <a name="vmware-component-versions"></a>Versiones de componentes de VMware

Una pila de VMware de nube privada se implementa con la versión de software siguiente.

| Componente | Versión | Versión con licencia |
|-----------|---------|------------------|
| ESXi | 6.7U2 | Enterprise Plus |
| vCenter | 6.7U2 | vCenter estándar |
| vSAN | 6.7 | Enterprise |
| NSX Data Center | 2.4.1 | Avanzado |

## <a name="esxi"></a>ESXi

VMware ESXi se instala en los nodos de CloudSimple aprovisionados al crear una nube privada.  ESXi proporciona el hipervisor para implementar máquinas virtuales (VM) de carga de trabajo.  Los nodos proporcionan infraestructuras hiperconvergidas (proceso y almacenamiento) en la nube privada.  Los nodos forman parte del clúster de vSphere en la nube privada.  Cada nodo tiene cuatro interfaces de red físicas conectadas a la red subyacente.  Se usan dos interfaces de red físicas para crear un **conmutador distribuido de vSphere (VDS)** en vCenter y dos para crear un **conmutador virtual distribuido administrado por NSX (N-VDS)**.  Las interfaces de red se configuran en modo activa/activa para la alta disponibilidad.

Más información sobre VMware ESXi

## <a name="vcenter-server-appliance"></a>Dispositivo de servidor de vCenter

El dispositivo de servidor de vCenter (VCSA) proporciona las funciones de autenticación, administración y orquestación para VMware Solution by CloudSimple. Al crear la nube privada, se implementa VCSA con el controlador de servicios de plataforma (PSC) insertado.  VCSA se implementa en el clúster de vSphere que se crea al implementar la nube privada.  Cada nube privada tiene su propio VCSA.  La expansión de una nube privada agrega los nodos al VCSA en la nube privada.

### <a name="vcenter-single-sign-on"></a>Inicio de sesión único de vCenter

El Controlador de servicios de plataforma insertado en VCSA está asociado con un **dominio de inicio de sesión único de vCenter**.  El nombre de dominio es **cloudsimple.local**.  Se crea un usuario predeterminado **CloudOwner@cloudsimple.com** para que acceda a vCenter.  Puede agregar los [orígenes de identidad para vCenter](set-vcenter-identity.md) locales o de Azure Active Directory.

## <a name="vsan-storage"></a>Almacenamiento de vSAN

Las nubes privadas se crean con almacenamiento de vSAN all-flash totalmente configurado, de forma local en el clúster.  Se necesita un mínimo de tres nodos de la misma SKU para crear un clúster de vSphere con el almacén de datos de vSAN.  En el almacén de datos de vSAN la desduplicación y la compresión están habilitadas de forma predeterminada.  Se crean dos grupos de discos en cada nodo del clúster de vSphere. Cada grupo de discos contiene un disco de caché y tres discos de capacidad.

Una directiva de almacenamiento de vSAN predeterminada se crea en el clúster de vSphere y se aplica al almacén de datos de vSAN.  Esta directiva determina cómo se aprovisionan los objetos de almacenamiento de máquina virtual y se asignan en el almacén de datos para garantizar el nivel de servicio requerido.  La directiva de almacenamiento define los **errores tolerables (FTT)** y el **método de tolerancia a errores**.  Puede crear directivas de almacenamiento y aplicarlas a las máquinas virtuales. Para mantener el SLA, se debe mantener el 25 % de la capacidad de reserva en el almacén de datos de vSAN.  

### <a name="default-vsan-storage-policy"></a>Directiva de almacenamiento de vSAN predeterminada

En la tabla siguiente se muestran los parámetros de la directiva de almacenamiento de vSAN predeterminada.

| Número de nodos en el clúster de vSphere | FTT | Método de tolerancia a errores |
|------------------------------------|-----|--------------------------|
| 3 y 4 nodos | 1 | RAID 1 (creación de reflejo): crea 2 copias |
| 5 a 16 nodos | 2 | RAID 1 (creación de reflejo): crea 3 copias |

## <a name="nsx-data-center"></a>NSX Data Center

NSX Data Center proporciona funciones de virtualización de red, microsegmentación y seguridad de red en la nube privada.  Puede configurar todos los servicios compatibles con NSX Data Center en la nube privada a través de NSX.  Al crear una nube privada, se instalan y configuran los siguientes componentes de NSX.

* NSXT Manager (Administrador de NSXT)
* Zonas de transporte
* Perfil de vínculo superior de host y Edge
* Conmutador lógico para Transporte perimetral, Ext1 y Ext2
* Grupo de IP para nodos de transporte de ESXi
* Grupo de IP para nodos de transporte perimetral
* Nodos perimetrales
* Regla de antiafinidad de DRS para el controlador y las máquinas virtuales de Edge
* Enrutador de nivel 0
* Habilitar BGP en el enrutador de nivel 0

## <a name="vsphere-cluster"></a>Clúster de vSphere

Los hosts de ESXi se configuran como un clúster para garantizar la alta disponibilidad de la nube privada.  Al crear una nube privada, los componentes de administración de vSphere se implementan en el primer clúster.  Se crea un grupo de recursos para los componentes de administración y todas las máquinas virtuales de administración se implementan en este grupo de recursos. No se puede eliminar el primer clúster para reducir la nube privada.  El clúster de vSphere proporciona alta disponibilidad para máquinas virtuales mediante **vSphere HA**.  Los errores tolerables se basan en el número de nodos disponibles en el clúster.  Puede usar la fórmula ```Number of nodes = 2N+1```, donde ```N``` es el número de errores tolerables.

### <a name="vsphere-cluster-limits"></a>Límites del clúster de vSphere

| Resource | Límite |
|----------|-------|
| Número mínimo de nodos para crear una nube privada (primer clúster de vSphere) | 3 |
| Número máximo de nodos en un clúster de vSphere en una nube privada | 16 |
| Número máximo de nodos en una nube privada | 64 |
| Número máximo de clústeres de vSphere en una nube privada | 21 |
| Número mínimo de nodos en un clúster de vSphere nuevo | 3 |

## <a name="vmware-infrastructure-maintenance"></a>Mantenimiento de la infraestructura de VMware

En ocasiones, es necesario realizar cambios en la configuración de la infraestructura de VMware. En la actualidad, estos intervalos se pueden producir cada 1 o 2 meses, pero se espera que la frecuencia se reduzca con el tiempo. Normalmente, este tipo de mantenimiento se puede realizar sin interrumpir el consumo normal de los servicios de CloudSimple. Durante un intervalo de mantenimiento de VMware, los servicios siguientes seguirán funcionando sin ningún impacto:

* Las aplicaciones y el plano de administración de VMware
* El acceso a vCenter
* Todas las redes y el almacenamiento
* Todo el tráfico de Azure

## <a name="updates-and-upgrades"></a>Actualizaciones

CloudSimple es responsable de la administración del ciclo de vida del software de VMware (ESXi, vCenter, PSC y NSX) en la nube privada.

Entre las actualizaciones de software se incluyen las siguientes:

* **Revisiones**. Revisiones de seguridad o correcciones de errores publicadas por VMware.
* **Actualizaciones**. Cambio de versión secundaria de un componente de la pila de VMware.
* **Actualizaciones**. Cambio de versión principal de un componente de la pila de VMware.

CloudSimple prueba una revisión de seguridad crítica en cuanto esté disponible desde VMware. Según el SLA, CloudSimple implementa la revisión de seguridad en los entornos de nube privada en el plazo de una semana.

CloudSimple proporciona actualizaciones de mantenimiento trimestrales para los componentes de software de VMware. Cuando hay una nueva versión principal disponible del software de VMware, CloudSimple trabaja con los clientes para coordinar una ventana de mantenimiento adecuada para la actualización.  

## <a name="next-steps"></a>Pasos siguientes

* [Mantenimiento y actualizaciones de CloudSimple](cloudsimple-maintenance-updates.md)
