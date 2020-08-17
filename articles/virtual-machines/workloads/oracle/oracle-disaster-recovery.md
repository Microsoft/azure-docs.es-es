---
title: Introducción a un escenario de recuperación ante desastres de Oracle en el entorno de Azure | Microsoft Docs
description: Un escenario de recuperación ante desastres para Oracle Database 12c en el entorno de Azure
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
ms.openlocfilehash: 6d1c7a48332aa49bc5c506ceeae287bc3affef16
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074073"
---
# <a name="disaster-recovery-for-an-oracle-database-12c-database-in-an-azure-environment"></a>Recuperación ante desastres para Oracle Database 12c en el entorno de Azure

## <a name="assumptions"></a>Supuestos

- Tiene conocimientos sobre el diseño de Oracle Data Guard y el entorno de Azure.


## <a name="goals"></a>Objetivos
- Diseñar la topología y configuración que satisfacen las necesidades de recuperación ante desastres.

## <a name="scenario-1-primary-and-dr-sites-on-azure"></a>Escenario 1: sitios principal y de recuperación ante desastres en Azure

Un cliente tiene una instalación de base de datos de Oracle en el sitio principal. El sitio de recuperación ante desastres (DR) se encuentra en una región diferente. El cliente utiliza Oracle Data Guard para la recuperación rápida entre estos sitios. El sitio principal tiene también una base de datos secundaria para informes y otros usos. 

### <a name="topology"></a>Topología

A continuación, se muestra un resumen de la configuración de Azure:

- Dos sitios (un sitio principal y un sitio de recuperación ante desastres)
- Dos redes virtuales
- Dos bases de datos de Oracle con Data Guard (principal y en espera)
- Dos bases de datos de Oracle con Golden Gate o Data Guard (solo para el sitio principal)
- Dos servicios de aplicación, uno en el sitio principal y uno en el sitio de recuperación ante desastres
- Un *conjunto de disponibilidad* que se usa para la base de datos y el servicio de aplicación en el sitio principal
- Un Jumpbox en cada sitio, que restringe el acceso a la red privada y solo permite el inicio de sesión al administrador
- El Jumpbox, el servicio de aplicación, la base de datos y la puerta de enlace de VPN en subredes independientes
- El grupo de seguridad de red se aplica en las subredes de la aplicación y de base de datos

![Captura de pantalla de la página de la topología de recuperación ante desastres](./media/oracle-disaster-recovery/oracle_topology_01.png)

## <a name="scenario-2-primary-site-on-premises-and-dr-site-on-azure"></a>Escenario 2: sitio principal en ubicación local y sitio de recuperación ante desastres en Azure

Un cliente tiene una instalación de base de datos de Oracle en un sitio local (sitio principal). Un sitio de recuperación ante desastres en Azure. Oracle Data Guard se usa para la recuperación rápida entre estos sitios. El sitio principal tiene también una base de datos secundaria para informes y otros usos. 

Existen dos enfoques para esta instalación.

### <a name="approach-1-direct-connections-between-on-premises-and-azure-requiring-open-tcp-ports-on-the-firewall"></a>Enfoque 1: conexiones directas entre el sitio local y Azure, lo que requiere abrir puertos TCP en el firewall 

No se recomiendan las conexiones directas porque exponen los puertos TCP al mundo exterior.

#### <a name="topology"></a>Topología

A continuación, se muestra un resumen de la configuración de Azure:

- Un sitio de recuperación ante desastres 
- Una red virtual
- Una base de datos de Oracle con Data Guard (activo)
- Un servicio de aplicación en el sitio de recuperación ante desastres
- Un Jumpbox, que restringe el acceso a la red privada y solo permite el inicio de sesión al administrador
- El Jumpbox, el servicio de aplicación, la base de datos y la puerta de enlace de VPN en subredes independientes
- El grupo de seguridad de red se aplica en las subredes de la aplicación y de base de datos
- Una directiva o regla de NSG para permitir el tráfico entrante en el puerto TCP 1521 (o el definido por el usuario)
- Una directiva o regla de NSG para restringir el acceso a la red virtual solo a la dirección o direcciones IP locales (base de datos o aplicación)

![Captura de pantalla de la página de la topología de recuperación ante desastres](./media/oracle-disaster-recovery/oracle_topology_02.png)

### <a name="approach-2-site-to-site-vpn"></a>Enfoque 2: VPN de sitio a sitio
Un mejor enfoque consiste en el uso de una VPN de sitio a sitio. Para más información acerca de cómo configurar una VPN, consulte [Creación de una red virtual con una conexión VPN de sitio a sitio mediante la CLI](../../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli.md).

#### <a name="topology"></a>Topología

A continuación, se muestra un resumen de la configuración de Azure:

- Un sitio de recuperación ante desastres 
- Una red virtual 
- Una base de datos de Oracle con Data Guard (activo)
- Un servicio de aplicación en el sitio de recuperación ante desastres
- Un Jumpbox, que restringe el acceso a la red privada y solo permite el inicio de sesión al administrador
- El Jumpbox, el servicio de aplicación, la base de datos y la puerta de enlace de VPN se encuentran en subredes independientes
- El grupo de seguridad de red se aplica en las subredes de la aplicación y de base de datos
- Conexión VPN de sitio a sitio entre sistemas locales y Azure

![Captura de pantalla de la página de la topología de recuperación ante desastres](./media/oracle-disaster-recovery/oracle_topology_03.png)

## <a name="additional-reading"></a>Lecturas adicionales

- [Diseño e implementación de una base de datos de Oracle en Azure](oracle-design.md)
- [Configuración de Oracle Data Guard](configure-oracle-dataguard.md)
- [Configuración de Oracle Golden Gate](configure-oracle-golden-gate.md)
- [Copia de seguridad y recuperación de Oracle](oracle-backup-recovery.md)


## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Creación de máquinas virtuales de alta disponibilidad](../../linux/create-cli-complete.md)
- [Ejemplos de la CLI de Azure para implementación de máquinas virtuales](../../linux/cli-samples.md)
