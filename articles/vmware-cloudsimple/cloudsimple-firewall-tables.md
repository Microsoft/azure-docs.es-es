---
title: 'Azure VMware Solution by CloudSimple: tablas de firewall'
description: Obtenga información sobre las reglas de firewall y las tablas de firewall de nube privada de CloudSimple, incluidas las reglas predeterminadas que se crean en cada tabla de firewall.
author: sharaths-cs
ms.author: dikamath
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 8c44c39f66a0a0161eea8a7e9656bbe0e3d1015c
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88140877"
---
# <a name="firewall-tables-overview"></a>Información general sobre las tablas de firewall

Una tabla de firewall enumera las reglas para filtrar el tráfico de red hacia y desde los recursos de la nube privada. Puede aplicar tablas de firewall a una VLAN o subred. Las reglas controlan el tráfico de red entre una red de origen o una dirección IP y una red de destino o una dirección IP.

## <a name="firewall-rules"></a>Reglas de firewall

En la siguiente tabla se describen los parámetros de una regla de firewall.

| Propiedad | Detalles |
| ---------| --------|
| **Nombre** | Nombre que identifica de forma única la regla de firewall y su finalidad. |
| **Prioridad** | Número entre 100 y 4096, donde 100 es la prioridad más alta. Las reglas se procesan en el orden de prioridad. Cuando el tráfico encuentra una coincidencia de regla, el procesamiento de la regla se detiene. Como resultado, las reglas con menor prioridad que tengan los mismos atributos que las reglas con una prioridad mayor no se procesarán.  Procure evitar las reglas en conflicto. |
| **Seguimiento de estado** | El seguimiento puede ser sin estado (nube privada, Internet o VPN) o con estado (IP pública).  |
| **Protocolo** | Algunas opciones son Cualquiera, TCP o UDP. Si necesita ICMP, use Cualquiera. |
| **Dirección** | Si la regla se aplica al tráfico entrante o al saliente. |
| **Acción** | Permita o deniegue el tipo de tráfico definido en la regla. |
| **Origen** | Una dirección IP, un bloque de enrutamiento de interdominios sin clases (CIDR) (por ejemplo, 10.0.0.0/24) o Cualquiera.  La especificación de un intervalo, una etiqueta de servicio o un grupo de seguridad de aplicaciones le permite crear menos reglas de seguridad. |
| **Puerto de origen** | Puerto de red del que procede el tráfico.  Puede especificar un puerto individual o un intervalo de puertos, como 443 u 8000-8080. La especificación de intervalos le permite crear menos reglas de seguridad. |
| **Destino** | Una dirección IP, un bloque de enrutamiento de interdominios sin clases (CIDR) (por ejemplo, 10.0.0.0/24) o Cualquiera.  La especificación de un intervalo, una etiqueta de servicio o un grupo de seguridad de aplicaciones le permite crear menos reglas de seguridad.  |
| **Puerto de destino** | Puerto al que fluye el tráfico de red.  Puede especificar un puerto individual o un intervalo de puertos, como 443 u 8000-8080. La especificación de intervalos le permite crear menos reglas de seguridad.|

### <a name="stateless"></a>Sin estado

Una regla sin estado solo examina paquetes individuales y los filtra en función de la regla.  
Es posible que se necesiten reglas adicionales para el flujo de tráfico en la dirección inversa.  Use reglas sin estado para el tráfico entre los siguientes puntos:

* Subredes de nubes privadas
* Subred local y una subred de nube privada
* Tráfico de Internet de nubes privadas

### <a name="stateful"></a>Con estado

 Una regla con estado tienen en cuenta las que pasan por ella. Se crea un registro de flujo para las conexiones existentes. Se permite o deniega la comunicación en función del estado de conexión del registro de flujo.  Use este tipo de regla con las direcciones IP públicas para filtrar el tráfico de Internet.

### <a name="default-rules"></a>Reglas predeterminadas

Las reglas predeterminadas siguientes se crean en cada tabla de firewall.

|Priority|Nombre|Seguimiento de estado|Dirección|Tipo de tráfico|Protocolo|Source|Puerto de origen|Destination|Puerto de destino|Acción|
|--------|----|--------------|---------|------------|--------|------|-----------|-----------|----------------|------|
|65000|allow-all-to-internet|Con estado|Salida|IP pública o tráfico de Internet|All|Any|Any|Any|Any|Allow|
|65001|deny-all-from-internet|Con estado|Entrada|IP pública o tráfico de Internet|All|Any|Any|Any|Any|Denegar|
|65002|allow-all-to-intranet|Sin estado|Salida|Nube privada interna o tráfico de VPN|All|Any|Any|Any|Any|Allow|
|65003|allow-all-from-intranet|Sin estado|Entrada|Nube privada interna o tráfico de VPN|All|Any|Any|Any|Any|Allow|

## <a name="next-steps"></a>Pasos siguientes

* [Configuración de tablas y reglas de firewall](firewall.md)
