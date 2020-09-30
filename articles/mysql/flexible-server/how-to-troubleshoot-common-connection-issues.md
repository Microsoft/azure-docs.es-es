---
title: 'Solución de problemas de conexión: Azure Database for MySQL con servidor flexible'
description: Obtenga información sobre la solución de problemas de conexión a Azure Database for MySQL con servidor flexible.
keywords: conexión de MySQL, cadena de conexión, problemas de conectividad, error persistente, error de conexión
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: troubleshooting
ms.date: 9/21/2020
ms.openlocfilehash: b328da01cd3b2ecb3eb5f183296eab808de27399
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931896"
---
# <a name="troubleshoot-connection-issues-to-azure-database-for-mysql---flexible-server"></a>Solución de problemas de conexión a Azure Database for MySQL: servidor flexible

> [!IMPORTANT]
> Actualmente, Azure Database for MySQL con servidor flexible se encuentra en versión preliminar pública.

Los problemas de conexión pueden deberse a una variedad de causas, como:

* Configuración de firewall
* Tiempo de espera de conexión agotado
* Información de inicio de sesión incorrecta
* Límite máximo alcanzado en algunos recursos de Azure Database for MySQL con servidor flexible

En este artículo, veremos cómo puede solucionar algunos de los errores más comunes y los pasos a seguir para resolver estos errores.

## <a name="troubleshoot-common-errors"></a>Solución de errores comunes

Si la aplicación no se puede conectar a Azure Database for MySQL con servidor flexible de forma persistente, esto normalmente indica un problema con uno de los siguientes elementos:

* Conexión cifrada mediante TLS/SSL: el servidor flexible solo admite conexiones cifradas mediante la Seguridad de la capa de transporte (TLS 1.2) y **se denegarán todas las conexiones entrantes con TLS 1.0 y TLS 1.1**. No se puede deshabilitar o cambiar la versión de TLS. Obtenga más información sobre la [conectividad cifrada con Seguridad de la capa de transporte (TLS 1.2) en Azure Database for MySQL con servidor flexible](./how-to-connect-tls-ssl.md).
- Servidor flexible en *Acceso privado (integración con red virtual)* : asegúrese de conectarse desde la misma red virtual que el servidor flexible. Consulte la información sobre [redes virtuales en Azure Database for MySQL con servidor flexible].<!--(./concepts-networking-virtual-network.md)-->
- Servidor flexible con *Acceso público (direcciones IP permitidas)* : asegúrese de que el firewall está configurado para permitir conexiones desde el cliente. Para obtener más información, consulte [Creación y administración de reglas de servidor flexible mediante Azure Portal](./how-to-manage-firewall-portal.md).
* Configuración del firewall del cliente: El firewall en el cliente debe permitir las conexiones con el servidor de bases de datos. La direcciones IP y los puertos del servidor se deben permitir, así como los nombres de aplicación como MySQL en algunos servidores de seguridad.
* Error del usuario: puede haber escrito incorrectamente los parámetros de conexión, como el nombre del servidor en la cadena de conexión.

### <a name="resolve-connectivity-issues"></a>Resolución de problemas de conectividad

* Consulte [Conectividad cifrada con Seguridad de la capa de transporte (TLS 1.2) en Azure Database for MySQL con servidor flexible](./how-to-connect-tls-ssl.md) para obtener más información acerca de las conexiones cifradas.
* Si usa la opción **Acceso público (direcciones IP permitidas)** , configure [reglas de firewall](./how-to-manage-firewall-portal.md) para permitir la dirección IP del cliente. Con fines temporales de prueba solo, configure una regla de firewall empleando 0.0.0.0 como dirección IP inicial y 255.255.255.255 como dirección IP final. Se abrirá el servidor a todas las direcciones IP. Si se resuelve el problema de conectividad, quite esta regla y cree una regla de firewall para una dirección IP o intervalo de direcciones apropiadamente limitados.
* En todos los firewalls entre el cliente e Internet, asegúrese de que el puerto 3306 está abierto para las conexiones salientes.
* Compruebe la cadena de conexión y otras opciones de conexión. Consulte las cadenas de conexión predefinidas en la página **Cadenas de conexión** que están disponibles para su servidor en Azure Portal para los lenguajes comunes.

## <a name="next-steps"></a>Pasos siguientes
- [Use MySQL Workbench para conectarse a datos y consultarlos en Azure Database for MySQL con servidor flexible](./connect-workbench.md).
- [Use PHP para conectarse a datos y consultarlos en Azure Database for MySQL con servidor flexible](./connect-php.md).
- [Use Python para conectarse a datos y consultarlos en Azure Database for MySQL con servidor flexible](./connect-python.md).
