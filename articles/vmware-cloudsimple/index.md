---
title: Azure VMware Solution by CloudSimple
description: Más información acerca de Azure VMware Solutions by CloudSimple entre la que se incluye una introducción, inicios rápidos, conceptos, tutoriales y guías paso a paso.
author: sharaths-cs
ms.author: b-mashar
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 0ea962469d9f1dafb51cf047306d99758c3cc118
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88140707"
---
# <a name="azure-vmware-solution-by-cloudsimple"></a>Azure VMware Solution by CloudSimple

Bienvenido al portal integral de ayuda de Azure VMware Solution by CloudSimple.
En el sitio de documentación puede obtener información sobre los temas siguientes:

## <a name="overview"></a>Información general

Más información sobre Azure VMware Solution by CloudSimple

* Para más información sobre características, ventajas y escenarios de uso, consulte [¿Qué es Azure VMware Solution by CloudSimple?](cloudsimple-vmware-solutions-overview.md)
* Revise los [conceptos clave de administración](key-concepts.md)

## <a name="quickstart"></a>Guía de inicio rápido

Más información sobre cómo empezar a usar la solución

* Información sobre cómo [inicializar el servicio y adquirir capacidad](quickstart-create-cloudsimple-service.md)
* Aprenda a crear un nuevo entorno de VMware en [Configuración del entorno de una nube privada](quickstart-create-private-cloud.md)
* Para aprender a unificar la administración entre VMware y Azure, revise el artículo [Uso de máquinas virtuales de VMware en Azure](quickstart-create-vmware-virtual-machine.md)

## <a name="concepts"></a>Conceptos

Más información sobre los siguientes conceptos

* Un [servicio CloudSimple](cloudsimple-service.md) (también conocido como "Azure VMware Solution by CloudSimple: Servicio"). Este recurso se debe crear una vez por región.
* Para adquirir capacidad para el entorno, cree uno o varios recursos de [nodo de CloudSimple](cloudsimple-node.md). Estos recursos también se conocen como "Azure VMware Solution by CloudSimple: Nodo".
* Inicialice y configure el entorno de VMware mediante las [nubes privadas](cloudsimple-private-cloud.md).
* Unifique la administración mediante [máquinas virtuales de CloudSimple](cloudsimple-virtual-machines.md) (también conocidas como "Azure VMware Solution by CloudSimple: Máquina virtual").
* Diseñe la red subyacente mediante [VLAN o subredes](cloudsimple-vlans-subnets.md).
* Segmente y proteja la red subyacente mediante el recurso de [tabla de firewall](cloudsimple-firewall-tables.md).
* Obtenga acceso seguro a los entornos de VMware a través de la WAN mediante [VPN Gateway](cloudsimple-vpn-gateways.md).
* Habilite el acceso público para cargas de trabajo mediante una [dirección IP pública](cloudsimple-public-ip-address.md).
* Establezca conectividad con las redes virtuales de Azure y las redes locales mediante la [conexión de red de Azure](cloudsimple-azure-network-connection.md).
* Configure los destinos de correos electrónicos de alerta mediante la [administración de cuentas](cloudsimple-account.md).
* Vea los registros de actividades del usuario y del sistema mediante las pantallas de [administración de actividades](cloudsimple-activity.md).
* Descripción de los diferentes [componentes de VMware](vmware-components.md).

## <a name="tutorials"></a>Tutoriales

Aprenda a realizar tareas comunes, como:

* [Crear un servicio de CloudSimple](create-cloudsimple-service.md), uno por cada región donde quiera implementar entornos de VMware.
* Administrar la funcionalidad de servicio principal en el [portal de CloudSimple](access-cloudsimple-portal.md).
* Habilitar capacidad y optimizar la facturación de la infraestructura mediante la [adquisición de nodos de CloudSimple](create-nodes.md).
* Administrar configuraciones de entorno de VMware mediante nubes privadas. Puede [crear](create-private-cloud.md), [administrar](manage-private-cloud.md), [expandir](expand-private-cloud.md) o [reducir](shrink-private-cloud.md) nubes privadas.
* Habilitar la administración unificada mediante la [asignación de suscripciones de Azure](azure-subscription-mapping.md).
* Supervisar la actividad del usuario y del sistema mediante las [páginas de actividad](monitor-activity.md).
* Configurar las redes de los entornos mediante la [creación y administración de subredes](create-vlan-subnet.md).
* Segmentar y proteger el entorno mediante [tablas y reglas de firewall](firewall.md).
* Habilitar el acceso entrante a Internet para las cargas de trabajo mediante la [asignación de direcciones IP públicas](public-ips.md).
* Habilitar la conectividad desde las redes internas o las estaciones de trabajo cliente mediante la [configuración de VPN](vpn-gateway.md).
* Habilitar las comunicaciones desde los [entornos locales](on-premises-connection.md) hacia las [redes virtuales de Azure](virtual-network-connection.md).
* Configurar destinos de alerta y ver la capacidad total comprada en el [resumen de la cuenta](account.md).
* Ver los [usuarios](users.md) que han accedido al portal de CloudSimple.
* Administrar máquinas virtuales de VMware desde Azure Portal:
    * [Crear máquinas virtuales](azure-create-vm.md) en Azure Portal.
    * [Administrar las máquinas virtuales](azure-manage-vm.md) que ha creado.

## <a name="how-to-guides"></a>Guías de procedimientos

Estas guías describen soluciones para conseguir estos objetivos:

* [Protección del entorno](private-cloud-secure.md)
* Instalación de herramientas de terceros, habilitación de usuarios adicionales y origen de autenticación externo en VSphere mediante [elevación de privilegios](escalate-privileges.md).
* Configuración del acceso a varios servicios de VMware mediante la [configuración de DNS local](on-premises-dns-setup.md).
* Habilitación de la asignación de nombres y direcciones para las cargas de trabajo mediante la [configuración de DNS y DHCP de la carga de trabajo](dns-dhcp-setup.md).
* Comprensión de cómo el servicio garantiza la seguridad y funcionalidad de la plataforma mediante las [actualizaciones](vmware-components.md#updates-and-upgrades) del servicio.
* Ahorro en el costo total de propiedad mediante la creación de una arquitectura de copia de seguridad de ejemplo con un [software de copia de seguridad de terceros como Veeam](backup-workloads-veeam.md).
* Creación de un entorno seguro mediante la habilitación del cifrado en reposo con un [software de cifrado de KMS de terceros](vsan-encryption.md).
* Ampliación de la administración de Azure Active Directory (Azure AD) en VMware mediante la configuración del [origen de identidades de Azure AD](azure-ad.md).
