---
title: ¿Qué es Oracle WebLogic Server en Azure?
description: Obtenga información sobre cómo ejecutar Oracle WebLogic Server en Microsoft Azure.
services: virtual-machines-linux
documentationcenter: ''
author: edburns
manager: gwallace
tags: azure-resource-management
ms.assetid: ''
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 05/11/2020
ms.author: edburns
ms.openlocfilehash: fe247e75040f658beb94a66176f802993268a7d8
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87851872"
---
# <a name="what-is-oracle-weblogic-server-on-azure"></a>¿Qué es Oracle WebLogic Server en Azure?

En esta página se describen las soluciones para ejecutar WebLogic Server (WLS) en Azure Virtual Machines.  Estas soluciones las desarrollan en conjunto Oracle y Microsoft.

Oracle WebLogic Server es el primer servidor de aplicaciones de la plataforma Java nativo de nube del mundo para desarrollar e implementar aplicaciones empresariales distribuidas de varios niveles. Las ofertas de Azure WebLogic Server le permiten adoptar la informática en la nube.  Obtendrá una mayor elección y flexibilidad para la migración de WebLogic, incluida la migración mediante lift-and-shift de las aplicaciones de Java EE a la nube de Azure.   WLS en Azure genera grandes efectos con un pequeño esfuerzo. Las ofertas permiten iniciar rápidamente sus aplicaciones de línea de negocio (LOB).  Cada oferta aprovisiona automáticamente la red virtual, el almacenamiento y los recursos de Linux.  WebLogic Server se instala sin ningún esfuerzo.  WLS en Azure configura la seguridad con un grupo de seguridad de red, el equilibrio de carga con Azure App Gateway, la autenticación con Azure Active Directory y se conecta automáticamente a la base de datos existente.

:::image type="content" source="media/oracle-weblogic/wls-on-azure.gif" alt-text="Puede usar Azure Portal para implementar WebLogic Server en Azure":::

Hay cuatro ofertas disponibles para satisfacer distintos escenarios: un solo nodo sin un servidor de administración, un solo nodo con un servidor de administración, un clúster y un clúster dinámico.  Pruebe las ofertas, ya que están disponibles de forma gratuita.

_Estas ofertas son de tipo "traiga su propia licencia"_ . Se supone que ya adquirió las licencias adecuadas con Oracle y que tiene licencias adecuadas para ejecutar ofertas en Microsoft Azure.

_Si quiere trabajar estrechamente en los escenarios de migración con el equipo de ingeniería que desarrolla estas ofertas, seleccione el botón [PONERSE EN CONTACTO CONMIGO](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/oracle.oraclelinux-wls-cluster?tab=Overview)_ en [Azure Marketplace](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/oracle.oraclelinux-wls-cluster?tab=Overview). Los administradores de programas, los arquitectos y los ingenieros se pondrán en contacto con usted en breve para iniciar la colaboración.

## <a name="oracle-weblogic-server-single-node"></a>Nodo único de Oracle WebLogic Server

Esta oferta aprovisiona una máquina virtual única y le instala WLS. No crea un dominio ni inicia el servidor de administración. El nodo único resulta útil en escenarios con una configuración de dominio muy personalizada.

## <a name="oracle-weblogic-server-with-admin-server"></a>Oracle WebLogic Server con servidor de administración

Esta oferta aprovisiona una máquina virtual única y le instala WLS. Crea un dominio e inicia el servidor de administración, lo que le permite administrar el dominio.

## <a name="oracle-weblogic-server-cluster"></a>Clúster de Oracle WebLogic Server

Esta oferta crea un clúster de alta disponibilidad de máquinas virtuales de WLS. El servidor de administración y todos los servidores administrados se inician de forma predeterminada, lo que le permite administrar el dominio.

## <a name="oracle-weblogic-server-dynamic-cluster"></a>Clúster dinámico de Oracle WebLogic Server

Esta oferta crea un clúster dinámico escalable y de alta disponibilidad de máquinas virtuales de WLS. El servidor de administración y todos los servidores administrados se inician de forma predeterminada, lo que le permite administrar el dominio.

## <a name="next-steps"></a>Pasos siguientes

Explore las ofertas en Azure Marketplace.

> [!div class="nextstepaction"]
> [Nodo único de Oracle WebLogic Server](https://portal.azure.com/#create/oracle.20191001-arm-oraclelinux-wls20191001-arm-oraclelinux-wls)

> [!div class="nextstepaction"]
> [Oracle WebLogic Server con servidor de administración](https://portal.azure.com/#create/oracle.20191009-arm-oraclelinux-wls-admin20191009-arm-oraclelinux-wls-admin)

> [!div class="nextstepaction"]
> [Clúster de Oracle WebLogic Server](https://portal.azure.com/#create/oracle.20191007-arm-oraclelinux-wls-cluster20191007-arm-oraclelinux-wls-cluster)

> [!div class="nextstepaction"]
> [Clúster dinámico de Oracle WebLogic Server](https://portal.azure.com/#create/oracle.20191021-arm-oraclelinux-wls-dynamic-cluster20191021-arm-oraclelinux-wls-dynamic-cluster)
