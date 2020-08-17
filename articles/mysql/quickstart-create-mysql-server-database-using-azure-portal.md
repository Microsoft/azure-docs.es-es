---
title: 'Inicio rápido: Creación de un servidor: Azure Portal (Azure Database for MySQL)'
description: Este artículo le ayudará a usar Azure Portal para crear rápidamente un servidor de Azure Database for MySQL de ejemplo, en unos cinco minutos.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 7/15/2020
ms.openlocfilehash: 61da81dd96078e0577e00be37fe27211a5b62154
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084434"
---
# <a name="quickstart-create-an-azure-database-for-mysql-server-in-the-azure-portal"></a>Inicio rápido: Creación de un servidor de Azure Database for MySQL mediante Azure Portal

Azure Database for MySQL es un servicio administrado que se usa para ejecutar, administrar y escalar bases de datos MySQL de alta disponibilidad en la nube. Esta guía de inicio rápido muestra cómo crear un servidor de Azure Database for MySQL en unos cinco minutos mediante Azure Portal.  

Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal
Abra el explorador web y vaya a [Azure Portal](https://portal.azure.com/). Introduzca sus credenciales para iniciar sesión en el portal. La vista predeterminada es el panel del servicio.

## <a name="create-an-azure-database-for-mysql-server"></a>Creación de un servidor de Azure Database for MySQL
Cree un servidor de Azure Database for MySQL con un conjunto definido de [recursos de proceso y almacenamiento](./concepts-compute-unit-and-storage.md). Cree el servidor dentro en un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md).

Para crear un servidor de Azure Database for MySQL, siga estos pasos:

1. En la esquina superior izquierda del portal, seleccione **Crear un recurso** (+).

2. Seleccione **Bases de datos** > **Azure Database for MySQL**. También puede escribir **MySQL** en el cuadro de búsqueda para encontrar el servicio.

  
>[!div class="mx-imgBorder"]
> ![Opción Azure Database for MySQL](./media/quickstart-create-mysql-server-database-using-azure-portal/2_navigate-to-mysql.png)

3. Rellene el formulario del nuevo servidor con la siguiente información:
    
>[!div class="mx-imgBorder"]
> ![Formulario de creación del servidor](./media/quickstart-create-mysql-server-database-using-azure-portal/4-create-form.png)

**Configuración** | **Valor sugerido** | **Descripción del campo** 
---|---|---
Suscripción | Su suscripción | Seleccione la suscripción de Azure que quiere usar para el servidor. Si tiene varias suscripciones, elija aquella en la que se factura el recurso.
Resource group | *myresourcegroup* | Proporcione un nombre de grupo de recursos nuevo o existente. El grupo de recursos se puede usar para organizar las dependencias que pertenecen a un único proyecto.
Nombre de servidor | Nombre de servidor único | Elija un nombre único que identifique al servidor de Azure Database for MySQL. Por ejemplo, "mysqldbserver". El nombre del servidor solo puede contener letras minúsculas, números y el carácter de guion (-). Debe contener entre 3 y 63 caracteres.
Origen de datos |*None* | Seleccione *None* para crear un servidor desde cero. (Seleccione *Copia de seguridad* si va a crear un servidor a partir de una copia de seguridad con redundancia geográfica de un servidor existente de Azure Database for MySQL).
Inicio de sesión de administrador de servidor | myadmin | Escriba un nombre de usuario para el administrador del servidor. El nombre del administrador no puede ser **azure_superuser**, **admin**, **administrator**, **root**, **guest** ni **public**.
Contraseña | *Su elección* | Proporcione una nueva contraseña para la cuenta de administrador del servidor. La contraseña debe tener entre 8 y 128 caracteres de longitud y debe contener una combinación de letras mayúsculas o minúsculas, números y caracteres no alfanuméricos (!, $, #,%, etc.).
Confirmar contraseña | *Su elección*| Confirme la contraseña de la cuenta de administrador.
Location | *Región más cercana a los usuarios*| Elija la ubicación más cercana a los usuarios o a sus otras aplicaciones de Azure.
Versión | *La versión principal más reciente*| La versión principal más reciente (a menos que tenga requisitos específicos que requieran otra versión).
Proceso y almacenamiento | **Uso general**, **Gen 5**, **2 núcleos virtuales**, **5 GB**, **7 días**, **Redundancia geográfica** |Configuración de los recursos de proceso, almacenamiento y copia de seguridad para el nuevo servidor. Seleccione **Configurar servidor**. A continuación, seleccione el plan de tarifa adecuado; para más información, consulte [Página de precios](https://azure.microsoft.com/pricing/details/mysql/). Para habilitar las copias de seguridad del servidor en el almacenamiento con redundancia geográfica, seleccione **Redundancia geográfica** en **Opciones de redundancia de copia de seguridad**. Seleccione **Aceptar**.

   > [!NOTE]
   > Considere la posibilidad de usar el plan de tarifa Básico si menos proceso y E/S resultan adecuados para su carga de trabajo. Tenga en cuenta que los servidores que creó en el plan de tarifa Básico no se podrán escalar más adelante a De uso general u Optimizada para memoria. 

4. Seleccione **Revisar y crear** para realizar el aprovisionamiento del servidor. El aprovisionamiento puede tardar hasta 20 minutos.
   
5. Haga clic en **Notificaciones** en la barra de herramientas (icono de campana) para supervisar el proceso de implementación.
   
De forma predeterminada, se crean las bases de datos siguientes en el servidor: **information_schema**, **mysql**, **performance_schema** y **sys**.

## <a name="configure-a-server-level-firewall-rule"></a>Configuración de una regla de firewall de nivel de servidor
De forma predeterminada, el servidor creado está protegido con un firewall y no se puede acceder a él públicamente. Para conceder acceso a su dirección IP, vaya al recurso de servidor en Azure Portal y seleccione **Seguridad de la conexión** en el menú del lado izquierdo del recurso de servidor. Si no sabe cómo encontrar el recurso, consulte [Apertura de recursos](https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resources-portal#open-resources).

>[!div class="mx-imgBorder"]
> ![Seguridad de la conexión: reglas de firewall](./media/quickstart-create-mysql-server-database-using-azure-portal/add-current-ip-firewall.png)
   
Ahora, seleccione **Agregar dirección IP del cliente actual** y, a continuación, seleccione **Guardar**. Puede agregar direcciones IP adicionales o proporcionar un intervalo de direcciones IP para conectarse al servidor desde esas direcciones IP. Para más información, consulte [Cómo administrar reglas de firewall del servidor de Azure Database for MySQL](./concepts-firewall-rules.md).

> [!NOTE]
> Compruebe si la red permite el tráfico de salida por el puerto 3306 que usa Azure Database for MySQL para evitar problemas de conectividad.  

## <a name="connect-to-azure-database-for-mysql-server-using-mysql-command-line-client"></a>Conexión a un servidor de Azure Database for MySQL mediante el cliente de línea de comandos de MySQL
Puede elegir [mysql.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql.html) o [MySQL Workbench](./connect-workbench.md) para conectarse al servidor desde su entorno local. En esta guía de inicio rápido, se ejecutará **mysql.exe** en [Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview) para conectarse al servidor.

1. Inicie Azure Cloud Shell en el portal; para ello, haga clic en el icono resaltado de la parte superior izquierda. Anote el nombre del servidor, el nombre de inicio de sesión del administrador del servidor, la contraseña y la suscripción desde la sección **Información general** del servidor recién creado, tal como se muestra en la imagen siguiente.

    >[!NOTE]
    >Si va a iniciar Cloud Shell por primera vez, se mostrará un símbolo del sistema para crear un grupo de recursos y una cuenta de almacenamiento. Esto es un paso único y se adjuntará automáticamente en todas las sesiones. 

   >[!div class="mx-imgBorder"]
   > ![Vista completa del portal de Cloud Shell](./media/quickstart-create-mysql-server-database-using-azure-portal/use-in-cloud-shell.png)
2. Ejecute este comando en el terminal de Azure Cloud Shell. Reemplace los valores por el nombre del servidor real y el nombre de inicio de sesión del usuario administrador. El nombre de usuario de administrador requiere "@<servername> como se muestra a continuación para Azure Database for MySQL  

  ```azurecli-interactive
  mysql --host=mydemoserver.mysql.database.azure.com --user=myadmin@mydemoserver -p 
  ```

  Este es el aspecto de la experiencia en el terminal de Cloud Shell.
  ```
  Requesting a Cloud Shell.Succeeded.
  Connecting terminal...

  Welcome to Azure Cloud Shell

  Type "az" to use Azure CLI
  Type "help" to learn about Cloud Shell

  sunitha@Azure:~$mysql -h mydemoserver.mysql.database.azure.com -u admin@wpapp-mysqldbserver -p
  Enter password:
  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 64796
  Server version: 5.6.42.0 Source distribution

  Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.

  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
  mysql>
  ```
3. En el mismo terminal de Azure Cloud Shell, cree la base de datos **guest** (invitado). 
  ```
  mysql> CREATE DATABASE guest;
  Query OK, 1 row affected (0.27 sec)
  ```
4. Cambie a la base de datos **guest**.
  ```
  mysql> USE guest;
  Database changed 
  ```
5. Escriba ```quit``` y después seleccione la tecla Entrar para salir de mysql.   

## <a name="clean-up-resources"></a>Limpieza de recursos
Ha creado correctamente un servidor de Azure Database for MySQL en un grupo de recursos.  Si no cree que vaya a necesitar estos recursos en el futuro, puede eliminarlos mediante la eliminación del grupo de recursos o bien puede eliminar simplemente el servidor de MySQL. Para eliminar el grupo de recursos, siga estos pasos:
1. En Azure Portal, busque y seleccione **Grupos de recursos**. 
2. En la lista de los grupos de recursos, elija el nombre del grupo de recursos.
3. En la página de información general del grupo de recursos, seleccione **Eliminar grupo de recursos**.
4. En el cuadro de diálogo de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.

Para eliminar el servidor, puede hacer clic en botón **Eliminar** de la página **Información general** del servidor, como se muestra a continuación:
> [!div class="mx-imgBorder"]
> ![Eliminación de los recursos](media/quickstart-create-mysql-server-database-using-azure-portal/delete-server.png)

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
>[Crear una aplicación PHP en Windows con MySQL](../app-service/app-service-web-tutorial-php-mysql.md)
>[Crear una aplicación PHP en Linux con MySQL](../app-service/containers/tutorial-php-mysql-app.md)
>[Crear una aplicación Spring basada en Java con MySQL](https://docs.microsoft.com/azure/developer/java/spring-framework/spring-app-service-e2e?tabs=bash)
