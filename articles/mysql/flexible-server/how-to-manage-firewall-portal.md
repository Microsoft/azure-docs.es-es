---
title: Administración de reglas de firewall del servidor flexible de Azure Database for MySQL mediante Azure Portal
description: Creación y administración de reglas de firewall para un servidor flexible de Azure Database for MySQL mediante Azure Portal
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: 132319575147c2ff1075881b1f1faec8bc5029f4
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932493"
---
# <a name="create-and-manage-firewall-rules-for-azure-database-for-mysql---flexible-server-using-the-azure-portal"></a>Creación y administración de reglas de firewall para un servidor flexible de Azure Database for MySQL mediante Azure Portal

> [!IMPORTANT]
> Actualmente, la opción de implementación Servidor flexible de Azure Database for MySQL se encuentra en versión preliminar pública.

Se admiten dos métodos de conectividad de red mutuamente excluyentes para conectarse a su servidor flexible de Azure Database for MySQL. Las dos opciones son:

1. Acceso público (direcciones IP permitidas)
2. Acceso privado (integración con red virtual)

Este artículo gira en torno a la creación de un servidor MySQL con **Acceso público (direcciones IP permitidas)** mediante Azure Portal y proporciona información general sobre la administración de reglas de firewall después de la creación de un servidor flexible. Con *Acceso público (direcciones IP permitidas)* , las conexiones con el servidor MySQL se restringen únicamente a las direcciones IP permitidas. Las direcciones IP del cliente se deben permitir en reglas de firewall. Para obtener más información al respecto, vea [Acceso público (direcciones IP permitidas)](./concepts-networking.md#public-access-allowed-ip-addresses). Las reglas de firewall se pueden definir en el momento de crear el servidor (recomendado), pero también se pueden agregar después. En este artículo, se proporcionará información general sobre cómo crear y administrar reglas de firewall con acceso público (direcciones IP permitidas).

## <a name="create-a-firewall-rule-when-creating-a-server"></a>Creación de una regla de firewall al crear un servidor

1. En la esquina superior izquierda del portal, seleccione **Crear un recurso** (+).
2. Seleccione **Bases de datos** > **Azure Database for MySQL**. También puede escribir **MySQL** en el cuadro de búsqueda para encontrar el servicio.
3. Seleccione **Servidor flexible** como opción de implementación.
4. Rellene el formulario **Datos básicos**.
5. Vaya a la pestaña **Redes** para configurar cómo se quiere conectar al servidor.
6. En **Método de conectividad**, seleccione *Acceso público (direcciones IP permitidas)* . Para crear las **reglas de firewall**, especifique el nombre de la regla de firewall y la dirección IP única, o bien un intervalo de direcciones. Si quiere limitar la regla a una única dirección IP, escriba la misma en los campos Dirección IP inicial y Dirección IP final. La apertura del firewall permite a administradores, usuarios y aplicaciones acceder a cualquier base de datos del servidor MySQL para la que tengan credenciales válidas.
   > [!Note]
   > Servidor flexible de Azure Database for MySQL crea un firewall en el nivel de servidor. Evita que herramientas y aplicaciones externas se conecten al servidor o a las bases de datos de este, a menos que cree una regla para abrir el firewall para direcciones IP concretas.

7. Seleccione **Revisar y crear** para revisar la configuración del servidor flexible.
8.  Seleccione **Crear** para realizar el aprovisionamiento del servidor. El aprovisionamiento puede tardar unos minutos.

## <a name="create-a-firewall-rule-after-server-is-created"></a>Creación de una regla de firewall después de crear el servidor

1. En [Azure Portal](https://portal.azure.com/), seleccione el servidor flexible de Azure Database for MySQL al que desea agregar reglas de firewall.
2. En la página del servidor flexible, en el título **Configuración**, haga clic en **Redes** para abrir la página Redes del servidor flexible.

   <!--:::image type="content" source="./media/howto-manage-firewall-portal/1-connection-security.png" alt-text="Azure portal - click Connection Security":::-->

3. Haga clic en **Agregar dirección IP del cliente actual** en las reglas de firewall. Se crea automáticamente una regla de firewall con la dirección IP pública del equipo, según la percibe el sistema de Azure.

   <!--:::image type="content" source="./media/howto-manage-firewall-portal/2-add-my-ip.png" alt-text="Azure portal - click Add My IP":::-->

4. Compruebe la dirección IP antes de guardar la configuración. En algunos casos, la dirección IP observada por Azure Portal difiere de la dirección IP utilizada para acceder a Internet y a los servidores de Azure. Por tanto, es posible que tenga que cambiar las direcciones IP inicial y final para que la regla funcione según lo previsto.

   Puede usar un motor de búsqueda u otra herramienta en línea para comprobar la dirección IP propia. Por ejemplo, busque "¿cuál es mi IP?".

   <!--:::image type="content" source="./media/howto-manage-firewall-portal/3-what-is-my-ip.png" alt-text="Bing search for What is my IP":::-->

5. Agregue intervalos de direcciones adicionales. En las reglas de firewall del servidor flexible de Azure Database for MySQL, puede especificar una dirección IP única o un intervalo de direcciones. Si quiere limitar la regla a una única dirección IP, escriba la misma en los campos Dirección IP inicial y Dirección IP final. La apertura del firewall permite a administradores, usuarios y aplicaciones acceder a cualquier base de datos del servidor MySQL para la que tengan credenciales válidas.

   <!--:::image type="content" source="./media/howto-manage-firewall-portal/4-specify-addresses.png" alt-text="Azure portal - firewall rules":::-->

6. Haga clic en **Guardar** en la barra de herramientas para guardar esta regla de firewall. Espere la confirmación de que la actualización de las reglas de firewall se realizó correctamente.

   <!--:::image type="content" source="./media/howto-manage-firewall-portal/5-save-firewall-rule.png" alt-text="Azure portal - click Save":::-->

## <a name="connect-from-azure"></a>Conexión desde Azure

Es posible que quiera habilitar los recursos o las aplicaciones implementadas en Azure para conectarse al servidor flexible. Esto incluye las aplicaciones web hospedadas en Azure App Service, que se ejecutan en una máquina virtual de Azure, una puerta de enlace de administración de datos de Azure Data Factory y muchas más.

Cuando una aplicación se intenta conectar al servidor desde Azure, el firewall comprueba que se permiten las conexiones de Azure. Puede habilitar este valor si selecciona la opción **Permitir acceso público a este servidor desde los recursos y servicios de Azure dentro de Azure** en el portal, en la pestaña **Redes** y presiona **Guardar**.

No es necesario que los recursos se encuentren en la misma red virtual (VNET) o grupo de recursos para que la regla de firewall habilite las conexiones. Si no se permite el intento de conexión, la solicitud no llega al servidor flexible de Azure Database for MySQL.

> [!IMPORTANT]
> Esta opción configura el firewall para permitir todas las conexiones de Azure, incluidas las de las suscripciones de otros clientes. Al seleccionar esta opción, asegúrese de que los permisos de usuario y el inicio de sesión limiten el acceso solamente a los usuarios autorizados.
>
> Se recomienda elegir **Acceso privado (integración con red virtual)** para acceder de forma segura al servidor flexible.
>

## <a name="manage-existing-firewall-rules-through-the-azure-portal"></a>Administración de reglas de firewall existentes a través de Azure Portal

Repita los pasos siguientes para administrar las reglas de firewall.

- Para agregar el equipo actual, haga clic en + **Agregar dirección IP del cliente actual** en las reglas de firewall. Haga clic en **Guardar** para guardar los cambios.
- Para agregar direcciones IP adicionales, escriba el nombre de la regla, la dirección IP inicial y la dirección IP final. Haga clic en **Guardar** para guardar los cambios.
- Para modificar una regla existente, haga clic en cualquiera de los campos de la regla y realice la modificación. Haga clic en **Guardar** para guardar los cambios.
- Para eliminar una regla existente, haga clic en el botón de puntos suspensivos […] y luego en **Eliminar** para quitar la regla. Haga clic en **Guardar** para guardar los cambios.

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre la [conexión de red en servidores flexibles de Azure Database for MySQL](./concepts-networking.md).
- Para más información, consulte [Reglas de firewall de servidor flexible de Azure Database for MySQL](./concepts-networking.md#public-access-allowed-ip-addresses).
- [Creación y administración de reglas de firewall de Azure Database for MySQL mediante la CLI de Azure](./how-to-manage-firewall-cli.md).