---
title: 'Inicio rápido: Creación de un servidor (Azure Portal): Azure Database for PostgreSQL (servidor único)'
description: En esta guía de inicio rápido va a crear y administrar un servidor de Azure Database for PostgreSQL desde Azure Portal.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.custom: mvc
ms.topic: quickstart
ms.date: 06/27/2020
ms.openlocfilehash: 9b3e1916301d6346defd6ebeab2f3784882f2d5f
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88798863"
---
# <a name="quickstart-create-an-azure-database-for-postgresql-server-by-using-the-azure-portal"></a>Inicio rápido: Creación de un servidor de Azure Database for PostgreSQL desde Azure Portal

Azure Database for PostgreSQL es un servicio administrado que usa para ejecutar, administrar y escalar bases de datos de PostgreSQL de alta disponibilidad en la nube. Este inicio rápido muestra cómo crear un servidor único de Azure Database for PostgreSQL en unos cinco minutos desde Azure Portal.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal
Abra el explorador web y vaya al [portal](https://portal.azure.com/). Introduzca sus credenciales para iniciar sesión en el portal. La vista predeterminada es el panel del servicio.

## <a name="create-an-azure-database-for-postgresql-server"></a>Creación de un servidor de Azure Database for PostgreSQL

Un servidor de Azure Database for PostgreSQL se crea con un conjunto configurado de [recursos de proceso y almacenamiento](./concepts-pricing-tiers.md). El servidor se crea dentro de un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md).

Para crear un servidor de Azure Database for PostgreSQL:

1. En la esquina superior izquierda del portal, seleccione **Crear un recurso**.

2. Seleccione **Bases de datos** > **Azure Database for PostgreSQL**.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla que muestra "Azure Database for PostgreSQL" en el menú.](./media/quickstart-create-database-portal/1-create-database.png)

3. Seleccione la opción de implementación de **servidor único**.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla de la selección de la opción de implementación de un servidor único en Azure Database for PostgreSQL.](./media/quickstart-create-database-portal/select-deployment-option.png)

4. Complete el formulario **Conceptos básicos** con la siguiente información.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla que muestra la pestaña Conceptos básicos para crear un servidor único.](./media/quickstart-create-database-portal/create-basics.png)

   Configuración|Valor sugerido|Descripción
   ---|---|---
   Subscription|Nombre de la suscripción|La suscripción de Azure que desea usar para el servidor. Si tiene varias suscripciones, elija aquella en la que se factura el recurso.
   Resource group|*myresourcegroup*| Un nuevo nombre de grupo de recursos o uno existente de la suscripción.
   Nombre de servidor |*mydemoserver*|Un nombre único que identifique al servidor de Azure Database for PostgreSQL. El nombre de dominio *postgres.database.azure.com* se anexa al nombre del servidor que especifique. El servidor solo puede contener letras minúsculas, números y el carácter de guion (-). Debe contener entre 3 y 63 caracteres.
   Origen de datos | **None** | Seleccione **None** para crear un servidor desde cero (si fuera a crear un servidor desde una copia de seguridad geográfica de un servidor de Azure Database for PostgreSQL server existente, seleccionaría **Copia de seguridad**).
   Nombre de usuario administrador |*myadmin*| Su propia cuenta de inicio de sesión para usarla al conectarse al servidor. El nombre de inicio de sesión del administrador no puede ser **azure_superuser,** **azure_pg_admin,** **admin,** **administrator,** **root,** **guest,** ni **public**. No puede empezar por **pg_** .
   Contraseña |La contraseña| Una contraseña nueva para la cuenta de administrador del servidor. Debe contener entre 8 y 128 caracteres de tres de las siguientes categorías: letras del alfabeto inglés mayúsculas y minúsculas, números (0 a 9) y caracteres no alfanuméricos (por ejemplo, !, $, # o %).
   Location|Región más cercana a los usuarios| Ubicación más cercana a los usuarios.
   Versión|La versión principal más reciente| La versión principal más reciente de PostgreSQL, a menos que tenga requisitos específicos.
   Proceso y almacenamiento | **Uso general**, **Gen 5**, **2 núcleos virtuales**, **5 GB**, **7 días**, **Redundancia geográfica** | Configuración de los recursos de proceso, almacenamiento y copia de seguridad para el nuevo servidor. Seleccione **Configurar servidor**. A continuación, seleccione el plan de tarifa adecuado. Para más información, consulte la [información de precios](https://azure.microsoft.com/pricing/details/postgresql/server/). Para habilitar las copias de seguridad del servidor en el almacenamiento con redundancia geográfica, seleccione **Redundancia geográfica** en **Opciones de redundancia de copia de seguridad**. Seleccione **Aceptar**.

   > [!NOTE]
   > Considere la posibilidad de usar el plan de tarifa Básico si menos proceso y E/S resultan adecuados para su carga de trabajo. Tenga en cuenta que los servidores que se creen en el plan de tarifa Básico no se podrán escalar más adelante a De uso general u Optimizada para memoria. 
   
5. Seleccione **Review + create** (Revisar y crear) para revisar las selecciones. Seleccione **Crear** para realizar el aprovisionamiento del servidor. Esta operación puede tardar unos minutos.

6. En la barra de herramientas, seleccione el símbolo de **Notificaciones** (una campana) para supervisar el proceso de implementación. Cuando se complete la implementación, seleccione **Ir al recurso** para abrir la página **Información general** del servidor.

Se ha creado una base de datos vacía llamada **postgres**. También encontrará la base de datos **azure_maintenance**, que se usa para separar los procesos del servicio administrado de las acciones del usuario. No se puede acceder a la base de datos **azure_maintenance**.

## <a name="configure-a-server-level-firewall-rule"></a>Configuración de una regla de firewall de nivel de servidor
De forma predeterminada, no se puede acceder públicamente al servidor que se crea. Es preciso conceder permisos a su dirección IP. Vaya al recurso de servidor en Azure Portal y seleccione **Seguridad de la conexión** en el menú izquierdo del recurso de servidor. Si no está seguro de cómo encontrar el recurso, consulte la sección [Apertura de recursos](https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resources-portal#open-resources).

> [!div class="mx-imgBorder"]
> ![Captura de pantalla que muestra las reglas de firewall para la seguridad de la conexión.](./media/quickstart-create-database-portal/add-current-ip-firewall.png)
  
Ahora, seleccione **Agregar dirección IP del cliente actual** y, después, **Guardar**. Puede agregar más direcciones IP o especificar un intervalo IP para conectarse al servidor desde dichas direcciones IP. Para más información, consulte el artículo [Reglas de firewall de Azure Database for PostgreSQL](./concepts-firewall-rules.md).
   
> [!NOTE]
> Para evitar problemas de conectividad, compruebe si la red permite el tráfico de salida a través el puerto 5432, ya que Azure Database for PostgreSQL usa ese puerto.  

## <a name="connect-to-azure-database-for-postgresql-server-by-using-psql"></a>Conexión al servidor de Azure Database for PostgreSQL mediante psql

Puede usar [psql](http://postgresguide.com/utilities/psql.html) o [pgAdmin](https://www.pgadmin.org/docs/pgadmin4/latest/connecting.html), que son clientes de PostgreSQL muy conocidos. En este inicio rápido, se conectará mediante psql a [Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview) dentro de Azure Portal.

1. Anote el nombre del servidor, el nombre de inicio de sesión del administrador del servidor, la contraseña y el identificador de la suscripción del servidor recién creado de la sección **Información general** del servidor.

2. Abra Azure Cloud Shell en el portal, para lo que debe seleccionar el icono de la parte superior izquierda.

   > [!NOTE]
   > Si es la primera vez que abre Cloud Shell, verá un símbolo del sistema para crear un grupo de recursos y una cuenta de almacenamiento. Esto es un paso único y se adjuntará automáticamente en todas las sesiones. 

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla que muestra información del servidor y el icono para abrir Azure Cloud Shell.](media/quickstart-create-database-portal/use-in-cloud-shell.png)

3. Ejecute el siguiente comando en el terminal de Azure Cloud Shell. Reemplace los valores por el nombre del servidor real y el nombre de inicio de sesión del usuario administrador. Use la base de datos vacía **postgres** con el usuario administrador en este formato: `<admin-username>@<servername>`.

   ```azurecli-interactive
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
   ```
 
   Este es el aspecto final en el terminal de Cloud Shell:
   
   ```bash
    Requesting a Cloud Shell.Succeeded.
    Connecting terminal...

    Welcome to Azure Cloud Shell
 
    Type "az" to use Azure CLI
    Type "help" to learn about Cloud Shell

    user@Azure:~$psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
    Password for user myadmin@mydemoserver.postgres.database.azure.com:
    psql (12.2 (Ubuntu 12.2-2.pgdg16.04+1), server 11.6)
    SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
    Type "help" for help.

    postgres=>
    ```
4. En el mismo terminal de Azure Cloud Shell, cree una base de datos llamada **guest**.

   ```bash
   postgres=> CREATE DATABASE guest;
   ```

5. Cambie las conexiones a la base de datos **guest** recién creada.

   ```bash
   \c guest
   ```
6. Escriba `\q` y seleccione la tecla Entrar para cerrar psql. 

## <a name="clean-up-resources"></a>Limpieza de recursos
Ha creado correctamente un servidor de Azure Database for PostgreSQL en un grupo de recursos. Si no cree que vaya a necesitar estos recursos en el futuro, puede eliminarlos mediante la eliminación del grupo de recursos o del servidor de PostgreSQL. 

Para eliminar el grupo de recursos:

1. En Azure Portal, busque y seleccione **Grupos de recursos**. 
2. En la lista de los grupos de recursos, elija el nombre del grupo de recursos.
3. En la página **Información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.
4. En el cuadro de diálogo de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.

Para eliminar el servidor, seleccione el botón **Eliminar** de la página **Información general** del servidor:

> [!div class="mx-imgBorder"]
> ![Captura de pantalla que muestra el botón para probar eliminar un servidor.](media/quickstart-create-database-portal/12-delete.png)

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Migración de una base de datos mediante exportación e importación](./howto-migrate-using-export-and-import.md)
