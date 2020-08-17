---
title: Información general sobre la implementación de Azure Firewall Manager
description: Conozca los pasos de implementación de alto nivel necesarios para Azure Firewall Manager
author: vhorne
ms.service: firewall-manager
services: firewall-manager
ms.topic: overview
ms.date: 07/28/2020
ms.author: victorh
ms.openlocfilehash: 748adbcc719df2cdcf734f308bd4b083e9ca6ec0
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372372"
---
# <a name="azure-firewall-manager-deployment-overview"></a>Información general sobre la implementación de Azure Firewall Manager

Hay más de una forma de implementar Azure Firewall Manager, pero se recomienda el siguiente proceso general.

## <a name="general-deployment-process"></a>Proceso de implementación general

### <a name="hub-virtual-networks"></a>Redes virtuales del centro de conectividad

1.  Cree una directiva de firewall

    - Creación de una nueva directiva
<br>*or*<br>
    - Derive una directiva base y personalización de una directiva local
<br>*or*<br>
    - Importe reglas de una instancia de Azure Firewall existente. Asegúrese de quitar las reglas NAT de las directivas que deben aplicarse en varios firewalls.
1. Creación de una arquitectura de concentrador y radio
   - Cree una red virtual de centro de conectividad mediante Azure Firewall Manager y empareje redes virtuales radiales con ella mediante el emparejamiento de redes virtuales
<br>*or*<br>
    - Cree una red virtual, agregue conexiones de red virtual y empareje redes virtuales radiales con ella mediante el emparejamiento de redes virtuales

3. Seleccione los proveedores de seguridad y asocie la directiva de firewall. Actualmente, Azure Firewall es el único proveedor admitido.

   - Esto se hace al crear una red virtual del centro de conectividad
<br>*or*<br>
    - Convierta una red virtual existente en una red virtual del centro de conectividad. También es posible convertir varias redes virtuales.

4. Configure rutas definidas por el usuario para enrutar el tráfico al firewall de la red virtual del centro de conectividad.


### <a name="secured-virtual-hubs"></a>Centros virtuales protegidos

1. Creación de una arquitectura de concentrador y radio

   - Cree un centro virtual protegido mediante Azure Firewall Manager y agregue conexiones de red virtual.<br>*or*<br>
   - Cree un centro de conectividad de Virtual WAN y agregue conexiones de red virtual.
2. Selección de proveedores de seguridad

   - Se realiza al crear un centro virtual protegido.<br>*or*<br>
   - Convierta un centro de conectividad de Virtual WAN en un centro virtual seguro.
3. Creación de una directiva de firewall y asociación con su concentrador

   - Solo se aplica si se usa Azure Firewall.
   - Las directivas de seguridad como servicio (SECaaS) de terceros se configuran a través de la experiencia de administración de asociados.
4. Configuración de los valores de la ruta para enrutar el tráfico a su centro protegido

   - Enrute fácilmente el tráfico al centro protegido para filtrar y registrar rutas definidas por el usuario (UDR) en redes virtuales de radios mediante la página de configuración de rutas del centro virtual protegido.

> [!NOTE]
> - No se puede tener más de un concentrador por red WAN virtual por región. Sin embargo, es posible agregar varias redes WAN virtuales en la región para lograrlo.
> - No se pueden tener espacios de direcciones IP superpuestos para los concentradores de una vWAN.
> - Las conexiones de red virtual del concentrador deben estar en la misma región que el concentrador.

## <a name="convert-virtual-networks"></a>Conversión de redes virtuales

La siguiente información se aplica si convierte una red virtual existente en una red virtual de centro de conectividad:

- Si la red virtual tiene una instancia de Azure Firewall existente, seleccione una directiva de firewall para asociarla con el firewall existente. El estado de aprovisionamiento del firewall se actualizará mientras la directiva de firewall reemplaza las reglas de firewall. Durante el estado de aprovisionamiento, el firewall continúa procesando el tráfico y no hay ningún tiempo de inactividad. Puede importar las reglas existentes a una directiva de firewall mediante Firewall Manager o Azure PowerShell.
- Si la red virtual no tiene una instancia de Azure Firewall asociada, se implementará un firewall y la directiva de firewall se asociará al nuevo firewall.

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Protección de una red en la nube con Azure Firewall Manager mediante Azure Portal](secure-cloud-network.md)