---
title: 'Información general sobre redes: Servidor flexible de Azure Database for PostgreSQL'
description: Obtenga información sobre las opciones de conectividad y redes en la opción de implementación Servidor flexible para Azure Database for PostgreSQL
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 963c9c06409eca2b2f836388b94f8b80484a671a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931847"
---
# <a name="networking-overview---azure-database-for-postgresql---flexible-server"></a>Información general sobre redes: Servidor flexible de Azure Database for PostgreSQL

> [!IMPORTANT]
> Servidor flexible de Azure Database for PostgreSQL está en versión preliminar

En este artículo se describen los conceptos de conectividad y redes para Servidor flexible de Azure Database for PostgreSQL. 

## <a name="choosing-a-networking-option"></a>Elección de una opción de red
Tiene dos opciones de red para Servidor flexible de Azure Database for PostgreSQL. Las opciones son **acceso privado (integración con red virtual)** y **acceso público (direcciones IP permitidas)** . Al crear el servidor, debe elegir una opción. 

> [!NOTE]
> La opción de red no se puede cambiar una vez que se haya creado el servidor. 

* **Acceso privado (Integración con red virtual)** : puede implementar el servidor flexible en la instancia de [Azure Virtual Network](../../virtual-network/virtual-networks-overview.md). Las redes virtuales de Azure proporcionan una comunicación de red privada y segura. Los recursos de una red virtual se pueden comunicar mediante direcciones IP privadas.

   Elija la opción Integración con red virtual si quiere las funcionalidades siguientes:
   * Conexión desde recursos de Azure en la misma red virtual a un servidor flexible mediante direcciones IP privadas
   * Uso de VPN o ExpressRoute para conectarse desde recursos que no son de Azure al servidor flexible
   * El servidor flexible no tiene un punto de conexión público

* **Acceso público (direcciones IP permitidas)** : se accede al servidor flexible a través de un punto de conexión público. El punto de conexión público es una dirección DNS que se puede resolver públicamente. La frase "direcciones IP permitidas" hace referencia a un intervalo de direcciones IP a las que decida conceder permiso para acceder al servidor. Estos permisos se denominan **reglas de firewall**. 

   Elija el método de acceso público si quiere las funcionalidades siguientes:
   * Conexión desde recursos de Azure que no admiten redes virtuales
   * Conexión desde recursos externos de Azure que no están conectados mediante VPN o ExpressRoute 
   * El servidor flexible tiene un punto de conexión público

Las características siguientes se aplican tanto si decide usar la opción de acceso privado como la de acceso público:
* Las conexiones desde direcciones IP permitidas se deben autenticar en el servidor PostgreSQL con credenciales válidas
* Hay [cifrado de conexión](#tls-and-ssl) disponible para el tráfico de red
* El servidor tiene un nombre de dominio completo (FQDN). En el caso de la propiedad hostname en las cadenas de conexión, se recomienda usar el FQDN en lugar de una dirección IP.
* Las dos opciones controlan el acceso en el nivel de servidor, no en el nivel de base de datos o de tabla. Tendría que usar las propiedades de los roles de PostgreSQL para controlar el acceso de bases de datos, tablas y otros objetos.


## <a name="private-access-vnet-integration"></a>Acceso privado (integración con red virtual)
El acceso privado con la integración de red virtual (vnet) proporciona una comunicación segura y privada para el servidor flexible de PostgreSQL.

### <a name="virtual-network-concepts"></a>Conceptos de redes virtuales
Estos son algunos conceptos que debe conocer al usar redes virtuales con servidores flexibles de PostgreSQL.

* **Red virtual**: una instancia de Azure Virtual Network (VNet) contiene un espacio de direcciones IP privadas que se configura para su uso. Para obtener más información sobre las redes virtuales de Azure, visite la [Información general sobre Azure Virtual Network](../../virtual-network/virtual-networks-overview.md).

    La red virtual debe estar en la misma región de Azure que el servidor flexible.


* **Subred delegada**: una red virtual contiene subredes. Las subredes permiten segmentar la red virtual en espacios de direcciones más pequeños. Los recursos de Azure se implementan en subredes específicas dentro de una red virtual. 

   El servidor flexible de PostgreSQL debe estar en una subred **delegada** para uso exclusivo del servidor flexible de PostgreSQL. Esta delegación significa que solo los servidores flexibles de Azure Database for PostgreSQL pueden usar esa subred. No puede haber otros tipos de recursos de Azure en la subred delegada. Puede delegar una subred si asigna su propiedad de delegación como Microsoft.DBforPostgreSQL/flexibleServers.

Obtenga información sobre cómo crear un servidor flexible con acceso privado (integración con red virtual) en [Azure Portal](how-to-manage-virtual-network-portal.md) o [la CLI de Azure](how-to-manage-virtual-network-cli.md).


### <a name="unsupported-virtual-network-scenarios"></a>Escenarios de red virtual no admitidos
* Punto de conexión público (o dirección IP pública o DNS): un servidor flexible implementado en una red virtual no puede tener un punto de conexión público
* Una vez que se haya implementado el servidor flexible en una red virtual y una subred, no se puede trasladar a otra red virtual o subred. No se puede trasladar la red virtual a otro grupo de recursos o suscripción.
* No se puede aumentar el tamaño de la subred (espacios de direcciones) cuando existen recursos en la subred
* No se admite el emparejamiento de redes virtuales entre regiones


## <a name="public-access-allowed-ip-addresses"></a>Acceso público (direcciones IP permitidas)
Entre las características del método de acceso público se incluyen las siguientes:
* Solo las direcciones IP que permita tienen autorización para acceder al servidor flexible de PostgreSQL. De forma predeterminada, no se permite ninguna dirección IP. Puede agregar direcciones IP durante la creación del servidor o después.
* El servidor PostgreSQL tiene un nombre DNS que se puede resolver de forma pública
* El servidor flexible no está en una de las redes virtuales de Azure
* El tráfico de red hacia y desde el servidor no se realiza a través de una red privada. El tráfico usa las rutas de Internet generales.

### <a name="firewall-rules"></a>Reglas de firewall
La concesión de permisos a una dirección IP se denomina regla de firewall. Si un intento de conexión procede de una dirección IP no permitida, el cliente de origen verá un error.

Obtenga información sobre cómo crear un servidor flexible con acceso público (direcciones IP permitidas) en [Azure Portal](how-to-manage-firewall-portal.md) o [la CLI de Azure](how-to-manage-firewall-cli.md).


### <a name="allowing-all-azure-ip-addresses"></a>Permitir todas las direcciones IP de Azure
Si no hay ninguna dirección IP saliente fija disponible para su servicio de Azure, puede habilitar las conexiones de todas las direcciones IP del centro de datos de Azure.

> [!IMPORTANT]
> La opción **Permitir acceso público desde los recursos y servicios de Azure dentro de Azure** configura el firewall para permitir todas las conexiones desde Azure, incluidas las que provienen de suscripciones de otros clientes. Al seleccionar esta opción, asegúrese de que los permisos de usuario y el inicio de sesión limiten el acceso solamente a los usuarios autorizados.

### <a name="troubleshooting-public-access-issues"></a>Solución de problemas de acceso público
Tenga en cuenta los siguientes puntos cuando el acceso al servicio del servidor de Microsoft Azure Database for PostgreSQL no se comporte de la manera prevista:

* **Los cambios en la lista de permitidos aún no se han aplicado:** puede que se produzca un retraso de hasta cinco minutos hasta que se apliquen los cambios de configuración del firewall del servidor de Azure Database for PostgreSQL.

* **Error de autenticación:** si un usuario no tiene permisos en el servidor de Azure Database for PostgreSQL o la contraseña usada es incorrecta, se denegará la conexión al servidor de Azure Database for PostgreSQL. La creación de una configuración de firewall solo ofrece a los clientes una oportunidad de intentar conectarse al servidor. Cada cliente todavía tiene que proporcionar las credenciales de seguridad necesarias.

* **Dirección IP del cliente dinámica:** Si tiene una conexión a Internet con un direccionamiento IP dinámico y tiene problemas al acceder al firewall, pruebe una de las soluciones siguientes:

   * Pida a su proveedor de acceso a Internet (ISP) el intervalo de direcciones IP asignado a los equipos cliente que acceden al servidor de Azure Database for PostgreSQL y agréguelo como regla de firewall.
   * Obtenga el direccionamiento IP estático en su lugar para los equipos cliente y luego agregue la dirección IP estática como regla de firewall.


## <a name="hostname"></a>Nombre de host
Con independencia de la opción de red que elija, se recomienda usar siempre un nombre de dominio completo (FQDN) como nombre de host al conectarse al servidor flexible. No se garantiza que la dirección IP del servidor permanezca estática. El uso del FQDN le ayudará a evitar realizar cambios en la cadena de conexión. 

Un escenario en el que los cambios de IP se producen si se usa alta disponibilidad con redundancia de zona y se produce una conmutación por error entre el principal y el secundario. El uso del FQDN significa que puede reintentar las conexiones sin problemas con la misma cadena de conexión.

Ejemplo
* `hostname = servername.postgres.database.azure.com` recomendado
* Evite el uso de `hostname = 10.0.0.4` (dirección privada) o `hostname = 40.2.45.67` (dirección pública)



## <a name="tls-and-ssl"></a>TLS y SSL
Servidor flexible de Azure Database for PostgreSQL admite la conexión de las aplicaciones cliente al servicio PostgreSQL mediante Seguridad de la capa de transporte (TLS). TLS es un protocolo estándar del sector que garantiza conexiones de red cifradas entre el servidor de bases de datos y las aplicaciones cliente. TLS es un protocolo actualizado de SSL (Capa de sockets seguros).

Servidor flexible de Azure Database for PostgreSQL solo admite conexiones cifradas con Seguridad de la capa de transporte. Se denegarán todas las conexiones entrantes con TLS 1.0 y TLS 1.1. 

## <a name="next-steps"></a>Pasos siguientes
* Obtenga información sobre cómo crear un servidor flexible con **acceso privado (integración con red virtual)** en [Azure Portal](how-to-manage-virtual-network-portal.md) o [la CLI de Azure](how-to-manage-virtual-network-cli.md).
* Obtenga información sobre cómo crear un servidor flexible con **acceso público (direcciones IP permitidas)** en [Azure Portal](how-to-manage-firewall-portal.md) o [la CLI de Azure](how-to-manage-firewall-cli.md).
