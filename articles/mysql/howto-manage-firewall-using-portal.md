---
title: Administración de reglas de firewall mediante Azure Portal en Azure Database for MySQL
description: Creación y administración de reglas de firewall de Azure Database for MySQL mediante Azure Portal
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: how-to
ms.date: 3/18/2020
ms.openlocfilehash: 7c5bc010653a936c00c5995142b5b34829591d24
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90884706"
---
# <a name="create-and-manage-azure-database-for-mysql-firewall-rules-by-using-the-azure-portal"></a>Creación y administración de reglas de firewall de Azure Database for MySQL mediante Azure Portal
Las reglas de firewall de nivel de servidor se pueden usar para administrar el acceso a un servidor de Azure Database for MySQL desde una dirección IP o desde un intervalo de direcciones IP específicos. 

Las reglas de Virtual Network (VNet) también se pueden usar para proteger el acceso a su servidor. Obtenga más información sobre la [creación y la administración de reglas y puntos de conexión de servicio de red virtual mediante Azure Portal](howto-manage-vnet-using-portal.md).

## <a name="create-a-server-level-firewall-rule-in-the-azure-portal"></a>Creación de una regla de firewall de nivel a servidor en Azure Portal

1. En la página del servidor de MySQL, en el encabezado Configuración, haga clic en **Seguridad de conexión** para abrir la página de seguridad de conexión para Azure Database for MySQL.

   :::image type="content" source="./media/howto-manage-firewall-using-portal/1-connection-security.png" alt-text="Azure Portal: haga clic en Seguridad de conexión":::

2. Haga clic en **Agregar mi dirección IP** en la barra de herramientas. Se crea automáticamente una regla de firewall con la dirección IP pública del equipo, según la percibe el sistema de Azure.

   :::image type="content" source="./media/howto-manage-firewall-using-portal/2-add-my-ip.png" alt-text="Azure Portal: haga clic en Agregar mi dirección IP":::

3. Compruebe la dirección IP antes de guardar la configuración. En algunos casos, la dirección IP observada por Azure Portal difiere de la dirección IP utilizada para acceder a Internet y a los servidores de Azure. Por tanto, puede necesitar cambiar las direcciones IP inicial y final para que la regla funcione según lo previsto.

   Use un motor de búsqueda u otra herramienta en línea para comprobar su propia dirección IP. Por ejemplo, busque "¿cuál es mi dirección IP?".

4. Agregue intervalos de direcciones adicionales. En las reglas de firewall de Azure Database for MySQL, puede especificar una sola dirección IP o un intervalo de direcciones. Si desea limitar la regla a una única dirección IP, escriba la misma dirección en los campos de dirección IP inicial y dirección IP final. Abrir el firewall permite a los administradores, los usuarios y la aplicación tener acceso a cualquier base de datos del servidor MySQL para el que tengan credenciales válidas.

   :::image type="content" source="./media/howto-manage-firewall-using-portal/4-specify-addresses.png" alt-text="Azure Portal: reglas de firewall":::

5. Haga clic en **Guardar**, en la barra de herramientas, para guardar esta regla de firewall de nivel de servidor. Espere la confirmación de que la actualización de las reglas de firewall se ha realizado correctamente.

   :::image type="content" source="./media/howto-manage-firewall-using-portal/5-save-firewall-rule.png" alt-text="Azure Portal: haga clic en Guardar":::

## <a name="connecting-from-azure"></a>Conexión desde Azure
Para permitir que las aplicaciones de Azure se conecten al servidor Azure Database for MySQL, deben habilitarse las conexiones de Azure. Por ejemplo, para hospedar una aplicación Azure Web Apps o una aplicación que se ejecute en una VM de Azure, o para conectarse desde una puerta de enlace de administración de datos de Azure Data Factory. No es necesario que los recursos se encuentren en la misma red virtual (VNET) o grupo de recursos para que la regla de firewall habilite las conexiones. Cuando una aplicación desde Azure intenta conectarse a su servidor de s de datos, el firewall comprueba que se permiten las conexiones de Azure. Hay un par de métodos para habilitar estos tipos de conexiones. Una configuración del firewall con dirección inicial y final igual a 0.0.0.0 indica que se permiten estas conexiones. Como alternativa, puede establecer la opción **Permitir el acceso a servicios de Azure** en **ACTIVADO** en el portal desde el panel **Seguridad de conexión** y seleccionar **Guardar**. Si no se permite el intento de conexión, la solicitud no llega al servidor Azure Database for MySQL.

> [!IMPORTANT]
> Esta opción configura el firewall para permitir todas las conexiones de Azure, incluidas las de las suscripciones de otros clientes. Al seleccionar esta opción, asegúrese de que los permisos de usuario y el inicio de sesión limiten el acceso solamente a los usuarios autorizados.
> 

## <a name="manage-existing-server-level-firewall-rules-by-using-the-azure-portal"></a>Administración de reglas de firewall de nivel de servidor existentes mediante Azure Portal
Repita los pasos para administrar las reglas de firewall.
* Para agregar el equipo actual, haga clic en **+ Agregar mi dirección IP**. Haga clic en **Guardar** para guardar los cambios.
* Para agregar direcciones IP adicionales, escriba el **nombre de la regla**, la **dirección IP inicial** y la **dirección IP final**. Haga clic en **Guardar** para guardar los cambios.
* Para modificar una regla existente, haga clic en cualquiera de los campos de la regla y luego realice la modificación. Haga clic en **Guardar** para guardar los cambios.
* Para eliminar una regla existente, haga clic en los puntos suspensivos […] y luego haga clic en **Eliminar**. Haga clic en **Guardar** para guardar los cambios.


## <a name="next-steps"></a>Pasos siguientes
- De modo similar, puede utilizar scripts para la [Creación y administración de reglas de firewall de Azure Database for MySQL mediante la CLI de Azure](howto-manage-firewall-using-cli.md).
- Proteja más el acceso al servidor con [la creación y la administración de reglas y puntos de conexión de servicio de red virtual mediante Azure Portal](howto-manage-vnet-using-portal.md).
- Para obtener ayuda para la conexión a un servidor de Azure Database for MySQL, consulte [Bibliotecas de conexiones para Azure Database for MySQL](./concepts-connection-libraries.md).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 
