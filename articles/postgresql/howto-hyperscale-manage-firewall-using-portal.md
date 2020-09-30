---
title: 'Administración de reglas de firewall en Azure Database for PostgreSQL: Hiperescala (Citus)'
description: 'Creación y administración de reglas de firewall en Azure Database for PostgreSQL: Hiperescala (Citus) mediante Azure Portal'
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 9/11/2020
ms.openlocfilehash: d369614357bd62dc13073f650fbe5ce358d6dc6e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90884330"
---
# <a name="manage-firewall-rules-for-azure-database-for-postgresql---hyperscale-citus"></a>Administración de reglas de firewall en Azure Database for PostgreSQL: Hiperescala (Citus)
Se pueden usar reglas de firewall de nivel de servidor para administrar el acceso a un nodo de coordinación de Hiperescala (Citus) desde una dirección IP o un intervalo de direcciones IP especificados.

## <a name="prerequisites"></a>Requisitos previos
Para seguir esta guía, necesitará:
- Un grupo de servidores. Consulte [Creación de un grupo de recursos de Azure Database for PostgreSQL: Hiperescala (Citus)](quickstart-create-hyperscale-portal.md).

## <a name="create-a-server-level-firewall-rule-in-the-azure-portal"></a>Creación de una regla de firewall de nivel a servidor en Azure Portal

> [!NOTE]
> También se puede obtener acceso a esta configuración durante la creación de un grupo de servidores de Azure Database for PostgreSQL: Hiperescala (Citus). En la pestaña **Redes**, haga clic en **Punto de conexión público**.

> :::image type="content" source="./media/howto-hyperscale-manage-firewall-using-portal/0-create-public-access.png" alt-text="Azure Portal: pestaña Redes":::

1. En la página del grupo de servidores PostgreSQL, en el encabezado Seguridad, haga clic en **Redes** para abrir las reglas de firewall.

   :::image type="content" source="./media/howto-hyperscale-manage-firewall-using-portal/1-connection-security.png" alt-text="Azure Portal: clic en Redes":::

2. Haga clic en **Add current client IP address** (Agregar la dirección IP actual del cliente) para crear una regla de firewall con la IP pública del equipo, según la percibe el sistema de Azure.

   :::image type="content" source="./media/howto-hyperscale-manage-firewall-using-portal/2-add-my-ip.png" alt-text="Azure Portal: clic en Agregar IP de cliente":::

Como alternativa, al hacer clic en **+Agregar 0.0.0.0-255.255.255.255** (a la derecha de la opción B), no solo se permite que la dirección IP acceda al puerto 5432 del nodo de coordinador, sino todo Internet. En esta situación, los clientes todavía deben iniciar sesión con el nombre de usuario y la contraseña correctos para usar el clúster. No obstante, se recomienda permitir el acceso global solo durante breves períodos de tiempo y solo para bases de datos que no sean de producción.

3. Compruebe la dirección IP antes de guardar la configuración. En algunos casos, la dirección IP observada por Azure Portal difiere de la dirección IP utilizada para acceder a Internet y a los servidores de Azure. Por tanto, puede necesitar cambiar las direcciones IP inicial y final para que la regla funcione según lo previsto.
   Use un motor de búsqueda u otra herramienta en línea para comprobar su propia dirección IP. Por ejemplo, busque "¿cuál es mi IP?".

   :::image type="content" source="./media/howto-hyperscale-manage-firewall-using-portal/3-what-is-my-ip.png" alt-text="Búsqueda en Bing "cuál es mi ip"":::

4. Agregue intervalos de direcciones adicionales. En las reglas de firewall, puede especificar una sola dirección IP o un intervalo de direcciones. Si desea limitar la regla a una única dirección IP, escriba la misma dirección en los campos de dirección IP inicial y dirección IP final. Al abrir el firewall, los administradores, los usuarios y las aplicaciones pueden acceder al nodo de coordinación en el puerto 5432.

5. Haga clic en **Guardar**, en la barra de herramientas, para guardar esta regla de firewall de nivel de servidor. Espere la confirmación de que la actualización de las reglas de firewall se realizó correctamente.

## <a name="connecting-from-azure"></a>Conexión desde Azure

Existe una manera sencilla de conceder a la base de datos de Hiperescala acceso a las aplicaciones hospedadas en Azure (por ejemplo, aplicaciones de Azure Web Apps o las que se ejecutan en una máquina virtual de Azure). Basta con establecer la opción **Permitir que los servicios y recursos de Azure accedan a este grupo de servidores** en **Sí** en el portal, en el panel **Redes**, y hacer clic en **Guardar**.

> [!IMPORTANT]
> Esta opción configura el firewall para permitir todas las conexiones de Azure, incluidas las de las suscripciones de otros clientes. Al seleccionar esta opción, asegúrese de que los permisos de usuario y el inicio de sesión limiten el acceso solamente a los usuarios autorizados.

## <a name="manage-existing-server-level-firewall-rules-through-the-azure-portal"></a>Administración de reglas de firewall de nivel de servidor existentes a través del Portal de Azure
Repita los pasos para administrar las reglas de firewall.
* Para agregar el equipo actual, haga clic en el botón + **Add current client IP address** (Agregar la dirección IP actual del cliente). Haga clic en **Guardar** para guardar los cambios.
* Para agregar direcciones IP adicionales, escriba el nombre de la regla, la dirección IP inicial y la dirección IP final. Haga clic en **Guardar** para guardar los cambios.
* Para modificar una regla existente, haga clic en cualquiera de los campos de la regla y realice la modificación. Haga clic en **Guardar** para guardar los cambios.
* Para eliminar una regla existente, haga clic en el botón de puntos suspensivos […] y luego en **Eliminar** para quitar la regla. Haga clic en **Guardar** para guardar los cambios.

## <a name="next-steps"></a>Pasos siguientes
- Más información sobre el [concepto de reglas de firewall](concepts-hyperscale-firewall-rules.md), incluida la solución de problemas de conexión.
