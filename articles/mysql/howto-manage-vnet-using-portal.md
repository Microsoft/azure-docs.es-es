---
title: Administración de puntos de conexión de red virtual de Azure Database for MySQL mediante Azure Portal
description: Creación y administración de reglas y puntos de conexión de servicio de red virtual de Azure Database for MySQL mediante Azure Portal
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: how-to
ms.date: 3/18/2020
ms.openlocfilehash: 61b08514f8d2989736ff2505f0446937757ddb99
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90884676"
---
# <a name="create-and-manage-azure-database-for-mysql-vnet-service-endpoints-and-vnet-rules-by-using-the-azure-portal"></a>Creación y administración de reglas y puntos de conexión de servicio de red virtual de Azure Database for MySQL mediante Azure Portal
Las reglas y los puntos de conexión de servicios de red virtual (VNet) amplían el espacio de direcciones privadas de una red virtual al servidor de Azure Database for MySQL. Para obtener información general sobre los puntos de conexión de servicio de red virtual de Azure Database for MySQL, incluidas las limitaciones, consulte [Azure Database for MySQL Server VNet service endpoints](concepts-data-access-and-security-vnet.md) (Puntos de conexión de servicio de red virtual del servidor de Azure Database for MySQL). Los puntos de conexión del servicio de red virtual están disponibles en todas las regiones admitidas para Azure Database for MySQL.

> [!NOTE]
> La compatibilidad con puntos de conexión de servicio de red virtual solo existe para servidores de uso general y optimizados para memoria.
> En el caso de emparejamiento de VNET, si el tráfico fluye a través de una instancia de VNet Gateway común con puntos de conexión de servicio y se supone que lo hace al elemento del mismo nivel, cree una regla de ACL o red virtual para permitir que Microsoft Azure Virtual Machines en la red virtual de puerta de enlace acceda al servidor de Azure Database for MySQL.


## <a name="create-a-vnet-rule-and-enable-service-endpoints-in-the-azure-portal"></a>Creación de una regla de red virtual y habilitación de los puntos de conexión de servicio en Azure Portal

1. En la página del servidor de MySQL, en el encabezado Configuración, haga clic en **Seguridad de conexión** para abrir la página de seguridad de conexión de Azure Database for MySQL. 

2. Asegúrese de que el control Permitir el acceso a los servicios de Azure está configurado en**Desactivado**.

> [!Important]
> Si deja el control establecido en Activado, el servidor de Azure MySQL Database aceptará la comunicación desde cualquier subred. Si deja el control establecido en Activado, el número de accesos podría ser excesivo desde un punto de vista de seguridad. La característica de punto de conexión del servicio Microsoft Azure Virtual Network, junto con la característica de la regla de red virtual de Azure Database for MySQL, pueden reducir el área expuesta de seguridad.

3. A continuación, haga clic en **+ Agregar una red virtual existente**. Si no tiene ninguna red virtual, puede hacer clic en **+ Crear nueva red virtual** para crear una. Consulte [Quickstart: Creación de una red virtual mediante Azure Portal](../virtual-network/quick-create-portal.md)

   :::image type="content" source="./media/howto-manage-vnet-using-portal/1-connection-security.png" alt-text="Azure Portal: haga clic en Seguridad de conexión":::

4. Escriba un nombre de regla de red virtual, seleccione la suscripción, el nombre de la subred y red virtual y, luego, haga clic en **Habilitar**. Esto habilita automáticamente puntos de conexión de servicio de red virtual en la subred mediante la etiqueta de servicio **Microsoft.SQL**.

   :::image type="content" source="./media/howto-manage-vnet-using-portal/2-configure-vnet.png" alt-text="Azure Portal: configuración de red virtual":::

   La cuenta debe tener todos los permisos necesarios para crear una red virtual y un punto de conexión de servicio.

   Los puntos de conexión de servicio pueden configurarse en redes virtuales de forma independiente por un usuario con acceso de escritura a la red virtual.
    
   Para proteger los recursos de servicio de Azure en una red virtual, el usuario debe tener permiso en "Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/" para las subredes que se agregan. De forma predeterminada, este permiso se incluye en los roles de administrador de servicios integrado y puede modificarse mediante la creación de roles personalizados.
    
   Obtenga más información sobre los [roles integrados](https://docs.microsoft.com/azure/active-directory/role-based-access-built-in-roles) y la asignación de permisos específicos a [roles personalizados](https://docs.microsoft.com/azure/active-directory/role-based-access-control-custom-roles).
    
   Las redes virtuales y los recursos de servicio de Azure pueden encontrarse en la misma o en diferentes suscripciones. Si los recursos de servicio de Azure y de red virtual se encuentran en distintas suscripciones, los recursos deben estar en el mismo inquilino de Active Directory (AD). Asegúrese de que ambas suscripciones tengan registrado el proveedor de recursos **Microsoft.Sql**. Para más información, consulte [resource-manager-registration][resource-manager-portal].

   > [!IMPORTANT]
   > Se recomienda encarecidamente leer este artículo sobre las configuraciones y las consideraciones de puntos de conexión de servicio antes de configurarlos. **Punto de conexión de servicio de red virtual:** un [punto de conexión de servicio de red virtual](../virtual-network/virtual-network-service-endpoints-overview.md) es una subred cuyos valores de propiedad incluyen uno o más nombres formales de tipo de servicio de Azure. Los puntos de conexión de servicio de red virtual usan el nombre de tipo de servicio **Microsoft.Sql**, que hace referencia al servicio de Azure denominado SQL Database. Esta etiqueta de servicio también se aplica a los servicios Azure SQL Database, Azure Database for PostgreSQL y MySQL. Es importante tener en cuenta que, al aplicar la etiqueta de servicio de **Microsoft.Sql** a un punto de conexión de servicio de red virtual, se configura el tráfico de punto de conexión de servicio de todos los servicios de Azure Database, incluidos los servidores de Azure SQL Database, Azure Database for PostgreSQL y Azure Database for MySQL de la subred. 
   > 

5. Una vez habilitada, haga clic en **Aceptar** y verá que se habilitan los puntos de conexión de servicio de red virtual junto con una regla de red virtual.

   :::image type="content" source="./media/howto-manage-vnet-using-portal/3-vnet-service-endpoints-enabled-vnet-rule-created.png" alt-text="Se han habilitado los puntos de conexión de servicio de red virtual y se ha creado la regla de red virtual":::

## <a name="next-steps"></a>Pasos siguientes
- De forma similar, puede crear scripts para [habilitar puntos de conexión de servicio de red virtual y crear una regla de red virtual para Azure Database for MySQL mediante la CLI de Azure](howto-manage-vnet-using-cli.md).
- Para obtener ayuda para la conexión a un servidor de Azure Database for MySQL, consulte [Bibliotecas de conexiones para Azure Database for MySQL](./concepts-connection-libraries.md).

<!-- Link references, to text, Within this same GitHub repo. --> 
[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md