---
title: Seguridad de Azure Database for MySQL
description: Información general sobre las características de seguridad de Azure Database for MySQL.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 5caed64beb43cbb1e884a3dde6bc0052c617f83e
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378787"
---
# <a name="security-in-azure-database-for-mysql"></a>Seguridad en Azure Database for MySQL

Existen varios niveles de seguridad disponibles para proteger los datos en el servidor de Azure Database for MySQL. En este artículo se describen esas opciones de seguridad.

## <a name="information-protection-and-encryption"></a>Protección y cifrado de información

### <a name="in-transit"></a>En tránsito
Azure Database for MySQL protege los datos mediante el cifrado de datos en tránsito con Seguridad de la capa de transporte. El cifrado (SSL/TLS) se aplica de forma predeterminada.

### <a name="at-rest"></a>En reposo
El servicio Azure Database for MySQL usa el módulo criptográfico con validación FIPS 140-2 para el cifrado del almacenamiento de los datos en reposo. Los datos, incluidas las copias de seguridad, se cifran en el disco, junto con los archivos temporales creados mientras se ejecutan las consultas. El servicio usa el cifrado AES de 256 bits que se incluye en el cifrado de almacenamiento de Azure y las claves las administra el sistema. El cifrado de almacenamiento siempre está activado y no se puede deshabilitar.


## <a name="network-security"></a>Seguridad de las redes
Las conexiones a un servidor de Azure Database for MySQL se enrutan primero a través de una puerta de enlace regional. La puerta de enlace tiene una dirección IP accesible públicamente, mientras que las direcciones IP del servidor están protegidas. Para obtener más información sobre la puerta de enlace, consulte el [artículo referente a la arquitectura de conectividad](concepts-connectivity-architecture.md).  

Un servidor de Azure Database for MySQL recién creado tiene un firewall que bloquea todas las conexiones externas. Aunque lleguen a la puerta de enlace, no tienen permiso para conectarse al servidor. 

### <a name="ip-firewall-rules"></a>Reglas de firewall de IP
Las reglas de firewall de IP otorgan acceso a los servidores según la dirección IP de origen de cada solicitud. Consulte la [información general sobre las reglas de firewall](concepts-firewall-rules.md) para obtener más información.

### <a name="virtual-network-firewall-rules"></a>Reglas de firewall de red virtual
Los puntos de conexión de servicio de red virtual amplían la conectividad de la red virtual a través de la red troncal de Azure. Cuando se usan reglas de red virtual, el servidor de Azure Database for MySQL se puede habilitar para permitir conexiones desde subredes seleccionadas en una red virtual. Para obtener más información, consulte la [información general sobre los puntos de conexión de servicio de red virtual](concepts-data-access-and-security-vnet.md).

### <a name="private-ip"></a>Dirección IP privada
Private Link le permite conectarse a la instancia de Azure Database for MySQL en Azure mediante un punto de conexión privado. En esencia, Azure Private Link incorpora los servicios de Azure dentro de su red virtual privada (VNet). Se puede acceder a los recursos de PaaS mediante la dirección IP privada, al igual que cualquier otro recurso de la red virtual. Para más información, vea la [información general de Private Link](concepts-data-access-security-private-link.md).

## <a name="access-management"></a>Administración de acceso

Al crear el servidor de Azure Database for MySQL, se deben proporcionar las credenciales de un usuario administrador. Este administrador se puede usar para crear más usuarios de MySQL.


## <a name="threat-protection"></a>Protección contra amenazas

Puede optar por usar [Advanced Threat Protection](concepts-data-access-and-security-threat-protection.md), que detecta actividades anómalas que indican intentos poco habituales y posiblemente dañinos de acceder a sus servidores o de aprovechar sus vulnerabilidades.

Existe un [registro de auditoría](concepts-audit-logs.md) disponible para realizar un seguimiento de las actividades en las bases de datos. 


## <a name="next-steps"></a>Pasos siguientes
- Habilite las reglas de firewall de [direcciones IP](concepts-firewall-rules.md) o de [redes virtuales](concepts-data-access-and-security-vnet.md).
