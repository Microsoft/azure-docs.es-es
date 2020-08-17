---
title: 'Reglas de firewall en Azure Database for PostgreSQL: servidor único'
description: 'En este artículo se describe cómo usar reglas de firewall para conectarse a Azure Database for PostgreSQL: servidor único.'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 07/17/2020
ms.openlocfilehash: 38edbfcb8800843b43678e99d6817595ccba3235
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87071545"
---
# <a name="firewall-rules-in-azure-database-for-postgresql---single-server"></a>Reglas de firewall de Azure Database for PostgreSQL con un único servidor
El firewall del servidor de Azure Database for PostgreSQL impide todo acceso al servidor de base de datos, excepto a aquellos equipos a los que haya concedido permiso. Asimismo, otorgan acceso al servidor según la dirección IP de origen de cada solicitud.
Para configurar su firewall, cree reglas de firewall que especifiquen intervalos de direcciones IP aceptables. Puede crear reglas de firewall a nivel de servidor.

**Reglas de firewall:** estas reglas permiten a los clientes tener acceso a todo el servidor de Azure Database for PostgreSQL; es decir, a todas las bases de datos que se encuentren en el mismo servidor lógico. Las reglas de firewall de nivel de servidor pueden configurarse a través de Azure Portal o los comandos de la CLI de Azure. Para crear reglas de firewall en el nivel del servidor, debe ser propietario o colaborador de la suscripción.

## <a name="firewall-overview"></a>Información general de firewalls
Todos los accesos de base de datos al servidor de Azure Database for PostgreSQL se bloquean por el firewall de manera predeterminada. Para comenzar a usar el servidor desde otro equipo, debe especificar una o varias reglas de firewall a nivel de servidor para habilitar el acceso a su servidor. Use las reglas de firewall para especificar los intervalos de direcciones IP de Internet que se permitirán. La acción de las reglas de firewall no repercute sobre el acceso al propio sitio web de Azure Portal.
Los intentos de conexión desde Internet y Azure deben atravesar primero el firewall antes de poder alcanzar PostgreSQL Database, tal y como se muestra en el siguiente diagrama:

![Ejemplo de flujo de cómo funciona el firewall](media/concepts-firewall-rules/1-firewall-concept.png)

## <a name="connecting-from-the-internet"></a>Conexión desde Internet
Las reglas de firewall de nivel de servidor se aplican a todas las bases de datos del mismo servidor de Azure Database for PostgreSQL. Si la dirección IP de la solicitud está comprendida en uno de los intervalos especificados en las reglas de firewall de nivel de servidor, la conexión se concede.
Si la dirección IP de la solicitud no se encuentra dentro de los intervalos especificados en cualquiera de las reglas de firewall de nivel de servidor, la solicitud de la conexión genera error.
Por ejemplo, si la aplicación se conecta con un controlador JDBC para PostgreSQL, puede encontrar este error al intentar conectarse cuando el firewall está bloqueando la conexión.
> java.util.concurrent.ExecutionException: java.lang.RuntimeException: org.postgresql.util.PSQLException: Error grave: no hay entrada pg\_hba.conf para el host "123.45.67.890", usuario "adminuser", base de datos "postgresql", SSL

## <a name="connecting-from-azure"></a>Conexión desde Azure
Se recomienda encontrar la dirección IP saliente de cualquier aplicación o servicio, y permitir explícitamente el acceso a las direcciones IP individuales o a los rangos. Por ejemplo, puede encontrar la dirección IP saliente de Azure App Service, o bien usar una IP pública asociada a una máquina virtual u otro recurso (a continuación encontrará información acerca de cómo conectar con la IP privada de una máquina virtual a través de puntos de conexión de servicio). 

Si no hay ninguna dirección IP saliente fija disponible para su servicio de Azure, puede habilitar las conexiones de todas las direcciones IP del centro de datos de Azure. Este valor se puede habilitar desde Azure Portal. Para ello, en la opción **Permitir el acceso a servicios de Azure**, seleccione **Activado** en el panel **Seguridad de conexión** y pulse **Guardar**. En la CLI de Azure, un valor de regla de firewall en que la dirección inicial y final sean 0.0.0.0 surte el mismo efecto. Si no se permite el intento de conexión, la solicitud no llega al servidor Azure Database for PostgreSQL.

> [!IMPORTANT]
> La opción **Permitir el acceso a servicios de Azure** configura el firewall para permitir todas las conexiones desde Azure, lo que incluye conexiones desde las suscripciones de otros clientes. Al seleccionar esta opción, asegúrese de que los permisos de usuario y el inicio de sesión limiten el acceso solamente a los usuarios autorizados.
> 

![Configurar el permiso de acceso a los servicios de Azure en el portal](media/concepts-firewall-rules/allow-azure-services.png)

### <a name="connecting-from-a-vnet"></a>Conexión desde una red virtual
Para conectarse de forma segura a Azure Database for PostgreSQL desde una red virtual, considere la posibilidad de usar [puntos de conexión de servicio de red virtual](./concepts-data-access-and-security-vnet.md). 

## <a name="programmatically-managing-firewall-rules"></a>Administración mediante programación de reglas de firewall
Además de Azure Portal, las reglas de firewall se pueden administrar mediante programación con la CLI de Azure.
Vea también la información sobre la [creación y la administración de reglas de firewall de Azure Database for PostgreSQL mediante la CLI de Azure](howto-manage-firewall-using-cli.md).

## <a name="troubleshooting-firewall-issues"></a>Solución de problemas de firewall
Tenga en cuenta los siguientes puntos cuando el acceso al servicio del servidor de Microsoft Azure Database for PostgreSQL no se comporte de la manera prevista:

* **Los cambios en la lista de permitidos aún no se han aplicado:** puede que se produzca un retraso de hasta cinco minutos hasta que se apliquen los cambios de configuración del firewall del servidor de Azure Database for PostgreSQL.

* **La cuenta de usuario no está autorizada o se ha usado una contraseña incorrecta:** si una cuenta de inicio de sesión no tiene los permisos en el servidor de Azure Database for PostgreSQL o la contraseña usada es incorrecta, se denegará la conexión al servidor de Azure Database for PostgreSQL. La creación de una configuración de firewall solo ofrece a los clientes una oportunidad de intentar conectarse al servidor; cada cliente debe continuar ofreciendo las credenciales de seguridad necesarias.

   Por ejemplo, mediante un cliente JDBC, puede aparecer el siguiente error.
   > java.util.concurrent.ExecutionException: java.lang.RuntimeException: org.postgresql.util.PSQLException: Error grave: error de autenticación de contraseña para el usuario "suNombreDeUsuario"

* **Dirección IP dinámica:** Si tiene una conexión a Internet con un direccionamiento IP dinámico y tiene problemas al acceder al firewall, pruebe una de las soluciones siguientes:

   * Pida a su proveedor de acceso a Internet (ISP) el intervalo de direcciones IP asignado a los equipos cliente que acceden al servidor de Azure Database for PostgreSQL y agréguelo como regla de firewall.

   * Obtenga el direccionamiento IP estático en su lugar para los equipos cliente y luego agregue la dirección IP estática como regla de firewall.

* **La dirección IP del servidor parece pública:** las conexiones con el servidor de Azure Database for PostgreSQL se redirigen mediante una puerta de enlace de Azure de acceso público. Sin embargo, la dirección IP real del servidor está protegida por el firewall. Para más información, consulte el [artículo de arquitectura de conectividad](concepts-connectivity-architecture.md).

* **No se puede conectar desde el recurso de Azure con la dirección IP permitida:** Compruebe si el punto de conexión de servicio **Microsoft.Sql** está habilitado para la subred desde la que se conecta. Si **Microsoft.Sql** está habilitado, indica que solo desea usar [reglas de punto de conexión de servicio de una red virtual](concepts-data-access-and-security-vnet.md) en esa subred.

   Por ejemplo, puede ver el siguiente error si se conecta desde una máquina virtual de Azure en una subred con **Microsoft.Sql** habilitado, pero sin ninguna regla de red virtual correspondiente: `FATAL: Client from Azure Virtual Networks is not allowed to access the server`

## <a name="next-steps"></a>Pasos siguientes
* [Create and manage Azure Database for PostgreSQL firewall rules using the Azure portal](howto-manage-firewall-using-portal.md) (Creación y administración de reglas de firewall de Azure Database for PostgreSQL mediante Azure Portal)
* [Create and manage Azure Database for PostgreSQL firewall rules using Azure CLI](howto-manage-firewall-using-cli.md) (Creación y administración de reglas de firewall de Azure Database for PostgreSQL mediante la CLI de Azure)
* [Puntos de conexión de servicio de red virtual en Azure Database for PostgreSQL](./concepts-data-access-and-security-vnet.md)
