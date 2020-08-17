---
title: 'Inicio rápido: Creación de servidor: Azure Portal: Azure Database for PostgreSQL (servidor único)'
description: 'Guía de inicio rápido para crear y administrar una instancia de Azure Database for PostgreSQL: servidor único con la interfaz de usuario de Azure Portal.'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.custom: mvc
ms.topic: quickstart
ms.date: 06/27/2020
ms.openlocfilehash: 9741e00cdcc4907c547ebfcc6f8e7df7a3b32891
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86529695"
---
# <a name="quickstart-create-an-azure-database-for-postgresql-server-in-the-azure-portal"></a>Inicio rápido: Creación de un servidor de Azure Database for PostgreSQL en Azure Portal

Azure Database for PostgreSQL es un servicio administrado que usa para ejecutar, administrar y escalar bases de datos de PostgreSQL de alta disponibilidad en la nube. Esta guía de inicio rápido muestra cómo crear un servidor de Azure Database for PostgreSQL en unos cinco minutos en Azure Portal.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal
Abra el explorador web y vaya al [portal](https://portal.azure.com/). Introduzca sus credenciales para iniciar sesión en el portal. La vista predeterminada es el panel del servicio.

## <a name="create-an-azure-database-for-postgresql-server"></a>Creación de un servidor de Azure Database for PostgreSQL

Un servidor de Azure Database for PostgreSQL se crea con un conjunto configurado de [recursos de proceso y almacenamiento](./concepts-pricing-tiers.md). El servidor se crea dentro de un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md).

Para crear un servidor de Azure Database for PostgreSQL, siga los pasos siguientes:
1. En la esquina superior izquierda del portal, seleccione **Crear un recurso** (+).

2. Seleccione **Bases de datos** > **Azure Database for PostgreSQL**.

   > [!div class="mx-imgBorder"]
   > !["Azure Database for PostgreSQL" en el menú](./media/quickstart-create-database-portal/1-create-database.png)

3. Seleccione la opción de implementación de **servidor único**.

   > [!div class="mx-imgBorder"]
   > ![Seleccione la opción de implementación Azure Database for PostgreSQL: servidor único](./media/quickstart-create-database-portal/select-deployment-option.png)

4. Complete el formulario de **aspectos básicos** con la información siguiente:

   > [!div class="mx-imgBorder"]
   > ![Creación de un servidor](./media/quickstart-create-database-portal/create-basics.png)

   Configuración|Valor sugerido|Descripción
   ---|---|---
   Subscription|Nombre de la suscripción|La suscripción de Azure que desea usar para el servidor. Si tiene varias suscripciones, elija aquella en la que se factura el recurso.
   Resource group|*myresourcegroup*| Un nuevo nombre de grupo de recursos o uno existente de la suscripción.
   Nombre de servidor |*mydemoserver*|Un nombre único que identifique al servidor de Azure Database for PostgreSQL. El nombre de dominio *postgres.database.azure.com* se anexa al nombre del servidor proporcionado. El servidor solo puede contener letras minúsculas, números y el carácter de guion (-). Debe contener al menos entre tres y 63 caracteres.
   Origen de datos | *None* | Seleccione *None* para crear un servidor desde cero. (Seleccione *Copia de seguridad* si va a crear un servidor a partir de una copia de seguridad con redundancia geográfica de un servidor existente de Azure Database for PostgreSQL).
   Nombre de usuario administrador |*myadmin*| Su propia cuenta de inicio de sesión para usarla al conectarse al servidor. El nombre de inicio de sesión del administrador no puede ser **azure_superuser,** **azure_pg_admin,** **admin,** **administrator,** **root,** **guest,** ni **public**. No puede empezar por **pg_** .
   Contraseña |La contraseña| Una contraseña nueva para la cuenta de administrador del servidor. Debe tener entre 8 y 128 caracteres. La contraseña debe contener caracteres de tres de las siguientes categorías: Letras del alfabeto inglés mayúsculas y minúsculas, números (0 a 9) y caracteres no alfanuméricos (!, $, #, %, etc.).
   Location|Región más cercana a los usuarios| La ubicación más cercana a los usuarios.
   Versión|La versión principal más reciente| La versión principal más reciente de PostgreSQL, a menos que tenga requisitos específicos.
   Proceso y almacenamiento | **Uso general**, **Gen 5**, **2 núcleos virtuales**, **5 GB**, **7 días**, **Redundancia geográfica** | Configuración de los recursos de proceso, almacenamiento y copia de seguridad para el nuevo servidor. Seleccione **Configurar servidor**. A continuación, seleccione el plan de tarifa adecuado; para más información, consulte [Detalles de precios](https://azure.microsoft.com/pricing/details/postgresql/server/). Para habilitar las copias de seguridad del servidor en el almacenamiento con redundancia geográfica, seleccione **Redundancia geográfica** en **Opciones de redundancia de copia de seguridad**. Seleccione **Aceptar**.

   > [!NOTE]
   > Considere la posibilidad de usar el plan de tarifa Básico si menos proceso y E/S resultan adecuados para su carga de trabajo. Tenga en cuenta que los servidores que creó en el plan de tarifa Básico no se podrán escalar más adelante a De uso general u Optimizada para memoria. 
   
5. Seleccione **Review + create** (Revisar y crear) para revisar las selecciones. Seleccione **Crear** para realizar el aprovisionamiento del servidor. Esta operación puede tardar algunos minutos.

6. En la barra de herramientas, seleccione el símbolo de **Notificaciones** (una campana) para supervisar el proceso de implementación. Una vez finalizada la implementación, seleccione **Ir al recurso** para abrir la página **Información general** del servidor.

Se ha creado una base de datos vacía llamada **postgres**. También encontrará la base de datos **azure_maintenance**, que se usa para separar los procesos del servicio administrado de las acciones del usuario. No se puede acceder a la base de datos **azure_maintenance**.

## <a name="configure-a-server-level-firewall-rule"></a>Configuración de una regla de firewall de nivel de servidor
De forma predeterminada, el servidor creado no es accesible públicamente y debe conceder permisos a su dirección IP. Para conceder acceso a su dirección IP, vaya al recurso de servidor en Azure Portal y seleccione **Seguridad de la conexión** en el menú del lado izquierdo del recurso de servidor. Si no está seguro de cómo encontrar el recurso, consulte [Apertura de recursos](https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resources-portal#open-resources).

> [!div class="mx-imgBorder"]
> ![Seguridad de la conexión: reglas de firewall](./media/quickstart-create-database-portal/add-current-ip-firewall.png)
  
Ahora, seleccione **Agregar dirección IP del cliente actual** y, a continuación, seleccione **Guardar**. Puede agregar direcciones IP adicionales o proporcionar un intervalo de direcciones IP para conectarse al servidor desde esas direcciones IP. Para más información, consulte [Cómo administrar reglas de firewall](./concepts-firewall-rules.md).
   
> [!NOTE]
> Compruebe si la red permite el tráfico de salida por el puerto 5432, utilizado por Azure Database for PostgreSQL para evitar problemas de conectividad.  

## <a name="connect-to-azure-database-for-postgresql-server-using-psql"></a>Conexión a Azure Database for PostgreSQL mediante psql

Puede usar [psql](http://postgresguide.com/utilities/psql.html) o [pgAdmin](https://www.pgadmin.org/docs/pgadmin4/latest/connecting.html), que son clientes populares de PostgreSQL. En esta guía de inicio rápido, se conectará con psql en [Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview) dentro de Azure Portal.

1. Anote el nombre del servidor, el nombre de inicio de sesión del administrador del servidor, la contraseña y el identificador de la suscripción del servidor recién creado de la sección **Información general** del servidor, tal como se muestra en la imagen siguiente.

2. Inicie Azure Cloud Shell en el portal; para ello, seleccione el icono de la parte superior izquierda, tal como se resalta en la imagen siguiente.

   > [!NOTE]
   > Si va a iniciar Cloud Shell por primera vez, verá un símbolo del sistema para crear un grupo de recursos y una cuenta de almacenamiento. Esto es un paso único y se adjuntará automáticamente en todas las sesiones. 

   > [!div class="mx-imgBorder"]
   > ![Apertura de Azure Cloud Shell](media/quickstart-create-database-portal/use-in-cloud-shell.png)

3. Ejecute este comando en el terminal de Azure Cloud Shell. Reemplace los valores por el nombre del servidor real y el nombre de inicio de sesión del usuario administrador. Use la base de datos vacía **postgres** con el usuario administrador con el formato <nombre_del_usuario_administrador>@<servername>, como se muestra a continuación para Azure Database for PostgreSQL.

   ```azurecli-interactive
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
   ```
 
   Este es el aspecto de la experiencia en el terminal de Cloud Shell.
   
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
4. En el mismo terminal de Azure Cloud Shell, cree la base de datos **guest** (invitado).
   ```bash
   postgres=> CREATE DATABASE guest;
   ```

5. Ahora, vamos a cambiar las conexiones a la base de datos **guest** recién creada.

   ```bash
   \c guest
   ```
6. Escriba `\q` y presione la tecla Entrar para salir de psql. 

## <a name="clean-up-resources"></a>Limpieza de recursos
Ha creado correctamente un servidor de Azure Database for PostgreSQL en un grupo de recursos.  Si no cree que vaya a necesitar estos recursos en el futuro, puede eliminarlos mediante la eliminación del grupo de recursos o bien elimine simplemente el servidor de PostgreSQL. Para eliminar el grupo de recursos, siga estos pasos:

1. En Azure Portal, busque y seleccione **Grupos de recursos**. 
2. En la lista de los grupos de recursos, elija el nombre del grupo de recursos.
3. En la página de información general del grupo de recursos, seleccione **Eliminar grupo de recursos**.
4. En el cuadro de diálogo de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.

Para eliminar el servidor, puede hacer clic en botón **Eliminar** de la página **Información general** del servidor, como se muestra a continuación:
> [!div class="mx-imgBorder"]
> ![Eliminación de los recursos](media/quickstart-create-database-portal/12-delete.png)

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Migración de una base de datos mediante exportación e importación](./howto-migrate-using-export-and-import.md)
