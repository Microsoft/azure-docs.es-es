---
title: 'Inicio rápido: Creación de un servidor mediante Azure Portal en Azure Database for PostgreSQL con la opción Servidor flexible'
description: Guía de inicio rápido para crear y administrar una instancia de Azure Database for PostgreSQL con la opción Servidor flexible mediante la interfaz de usuario de Azure Portal.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.custom: mvc
ms.topic: quickstart
ms.date: 09/22/2020
ms.openlocfilehash: 25c3f875717c9c064af0ce27bdab735db67d2f5d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945138"
---
# <a name="quickstart-create-an-azure-database-for-postgresql---flexible-server-in-the-azure-portal"></a>Inicio rápido: Creación de una instancia de Azure Database for PostgreSQL con la opción Servidor flexible mediante Azure Portal

> [!IMPORTANT]
> Azure Database for PostgreSQL con la opción Servidor flexible se encuentra en versión preliminar.

Azure Database for PostgreSQL es un servicio administrado que usa para ejecutar, administrar y escalar bases de datos de PostgreSQL de alta disponibilidad en la nube. En esta guía de inicio rápido se muestra cómo crear un servidor de Azure Database for PostgreSQL con la opción Servidor flexible en unos cinco minutos mediante Azure Portal.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Abra el explorador web y vaya al [portal](https://portal.azure.com/). Introduzca sus credenciales para iniciar sesión en el portal. La vista predeterminada es el panel del servicio.

## <a name="create-an-azure-database-for-postgresql-server"></a>Creación de un servidor de Azure Database for PostgreSQL

Un servidor de Azure Database for PostgreSQL se crea con un conjunto configurado de [recursos de proceso y almacenamiento](./concepts-compute-storage.md). El servidor se crea dentro de un [grupo de recursos de Azure](https://docs.microsoft.com/azure/azure-resource-manager/management/overview).

Para crear un servidor de Azure Database for PostgreSQL, siga los pasos siguientes:

1. En la esquina superior izquierda del portal, seleccione **Crear un recurso** (+).

2. Seleccione **Bases de datos** > **Azure Database for PostgreSQL**.

    :::image type="content" source="./media/quickstart-create-database-portal/1-create-database.png" alt-text="Azure Database for PostgreSQL en el menú":::

3. Seleccione la opción de implementación **Servidor flexible**.

   :::image type="content" source="./media/quickstart-create-database-portal/2-select-deployment-option.png" alt-text="Seleccione la opción de implementación Servidor flexible de Azure Database for PostgreSQL.":::

4. Complete el formulario de **aspectos básicos** con la información siguiente:

    :::image type="content" source="./media/quickstart-create-database-portal/3-create-basics.png" alt-text="Creación de un servidor":::

    Configuración|Valor sugerido|Descripción
    ---|---|---
    Subscription|Nombre de la suscripción|La suscripción de Azure que desea usar para el servidor. Si tiene varias suscripciones, elija aquella en la que quiere que se le facture el recurso.
    Resource group|*myresourcegroup*| Un nuevo nombre de grupo de recursos o uno existente de la suscripción.
    Nombre de servidor |*mydemoserver*|Un nombre único que identifique al servidor de Azure Database for PostgreSQL. El nombre de dominio *postgres.database.azure.com* se anexa al nombre del servidor proporcionado. El servidor solo puede contener letras minúsculas, números y el carácter de guion (-). Debe contener al menos entre tres y 63 caracteres.
    Nombre de usuario administrador |*myadmin*| Su propia cuenta de inicio de sesión para usarla al conectarse al servidor. El nombre de inicio de sesión del administrador no puede ser **azure_superuser,** **azure_pg_admin,** **admin,** **administrator,** **root,** **guest,** ni **public**. No puede empezar por **pg_** .
    Contraseña |La contraseña| Una contraseña nueva para la cuenta de administrador del servidor. Debe tener entre 8 y 128 caracteres. La contraseña debe contener caracteres de tres de las siguientes categorías: Letras del alfabeto inglés mayúsculas y minúsculas, números (0 a 9) y caracteres no alfanuméricos (!, $, #, %, etc.).
    Location|Región más cercana a los usuarios| La ubicación más cercana a los usuarios.
    Versión|La versión principal más reciente| La versión principal más reciente de PostgreSQL, a menos que tenga requisitos específicos.
    Proceso y almacenamiento | **De uso general**, **4 núcleos virtuales**, **512 GB** y **7 días** | Configuración de los recursos de proceso, almacenamiento y copia de seguridad para el nuevo servidor. Seleccione **Configurar servidor**. *De uso general*, *4 núcleos virtuales*, *512 GB*y *7 días* son los valores predeterminados para el **nivel de proceso**, el **núcleo virtual**, el **almacenamiento** y el **período de retención de copia de seguridad**. Puede dejar esos controles deslizantes tal como están o ajustarlos. Para guardar el plan de tarifa elegido, seleccione **Aceptar**. La captura de pantalla siguiente muestra estas opciones seleccionadas.

    :::image type="content" source="./media/quickstart-create-database-portal/4-pricing-tier.png" alt-text="Panel Plan de tarifa":::
    
5. Configuración de las opciones de red

    En la pestaña Red, puede elegir cómo se puede tener acceso al servidor. Azure Database for PostgreSQL crea un firewall en el nivel de servidor. Evita que herramientas y aplicaciones externas se conecten al servidor o a las bases de datos de este, a menos que cree una regla para abrir el firewall para direcciones IP concretas. Se recomienda que el servidor sea accesible públicamente:

    :::image type="content" source="./media/quickstart-create-database-portal/5-networking.png" alt-text="Panel Redes":::

    A continuación, restrínjalo a su propia dirección IP del cliente:

    :::image type="content" source="./media/quickstart-create-database-portal/6-add-client-ip.png" alt-text="Seleccione Agregar dirección IP del cliente actual":::.

6. Seleccione **Review + create** (Revisar y crear) para revisar las selecciones. Seleccione **Crear** para realizar el aprovisionamiento del servidor. Esta operación puede tardar algunos minutos.

7. En la barra de herramientas, seleccione el símbolo de **Notificaciones** (una campana) para supervisar el proceso de implementación. Una vez realizada la implementación, puede seleccionar **Anclar al panel** para crear un icono para este servidor en el panel de Azure Portal como un acceso directo a la página **Información general** del servidor. Al seleccionar **Ir al recurso**, se abre la página **Información general** del servidor.

    :::image type="content" source="./media/quickstart-create-database-portal/7-notifications.png" alt-text="Panel Notificaciones":::

   De forma predeterminada, una base de datos de **postgres** se crea en el servidor. La base de datos de [postgres](https://www.postgresql.org/docs/12/static/app-initdb.html) es una base de datos predeterminada pensada para que la usen los usuarios, las utilidades y aplicaciones de otros fabricantes. (La otra base de datos predeterminada es **azure_maintenance**. Su función consiste en separar los procesos de servicio administrados de las acciones del usuario. No se puede acceder esta base de datos).

    > [!NOTE]
    > Las conexiones al servidor de Azure Database for PostgreSQL se comunican a través de puerto 5432. Si intenta conectarse desde una red corporativa, es posible que el firewall de la red no permita el tráfico saliente a través del puerto 5432. En ese caso no podrá conectarse al servidor, salvo que el departamento de TI abra el puerto 5432.
    >

## <a name="get-the-connection-information"></a>Obtención de la información de conexión

Al crear el servidor de Azure Database for PostgreSQL, también se crea la base de datos predeterminada denominada **postgres**. Para conectarse al servidor de base de datos, necesita las credenciales de inicio de sesión de administrador y el nombre de servidor completo. Es posible que anotara dichos valores en el artículo de la guía de inicio rápido. En caso de que no lo hiciera, encontrará fácilmente el nombre del servidor y la información de inicio de sesión en la página **Información general** del servidor en el portal.

Abra la página **Información general** del servidor. Tome nota del **Nombre del servidor** y del **Nombre de inicio de sesión del administrador del servidor**. Desplace el cursor sobre cada campo y el símbolo de copiar aparecerá a la derecha del texto. Seleccione el símbolo de copiar según sea necesario para copiar los valores.

 :::image type="content" source="./media/quickstart-create-database-portal/8-server-name.png" alt-text="Página Información general del servidor":::

## <a name="connect-to-the-postgresql-database-using-psql"></a>Conexión a la base de datos de PostgreSQL mediante psql

Hay una serie de aplicaciones que se pueden usar para conectarse al servidor de Azure Database for PostgreSQL. Si el equipo cliente tiene PostgreSQL instalado, puede usar una instancia local de [psql](https://www.postgresql.org/docs/current/static/app-psql.html) para conectarse a un servidor Azure PostgreSQL. Ahora vamos a usar la utilidad de línea de comandos psql para conectarnos al servidor Azure PostgreSQL.

1. Ejecute el comando psql siguiente para conectarse a un servidor de Azure Database for PostgreSQL

   ```bash
   psql --host=<servername> --port=<port> --username=<user> --dbname=<dbname>
   ```

   Por ejemplo, el siguiente comando se conecta a la base de datos predeterminada llamada **postgres** en el servidor PostgreSQL **mydemoserver.postgres.database.azure.com** con las credenciales de acceso. Escriba el valor de `<server_admin_password>` que eligió cuando se le solicitó una contraseña.
  
   ```bash
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin --dbname=postgres
   ```

   Tras conectarse, la utilidad psql muestra un símbolo del sistema de postgres donde escribir comandos sql. En la salida de la conexión inicial, puede aparecer una advertencia, ya que la versión de psql que usa puede diferir de la versión del servidor de Azure Database for PostgreSQL.

   Ejemplo de salida de psql:

   ```bash
   psql (11.3, server 12.1)
   WARNING: psql major version 11, server major version 12.
            Some psql features might not work.
   SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
   Type "help" for help.

   postgres=>
   ```

   > [!TIP]
   > Si el firewall no está configurado para permitir la dirección IP de su cliente, se produce el siguiente error:
   >
   > "psql: FATAL:  no pg_hba.conf entry for host `<IP address>`, user "myadmin", database "postgres", SSL on FATAL: SSL connection is required. Specify SSL options and retry.
   >
   > Confirme que la IP de su cliente se permite en el paso anterior de las reglas de firewall.

2. Escriba el comando siguiente para crear una base de datos en blanco llamada "mypgsqldb" en el símbolo del sistema:

    ```bash
    CREATE DATABASE mypgsqldb;
    ```

3. En el símbolo del sistema, ejecute el siguiente comando para cambiar las conexiones a la base de datos **mypgsqldb** recién creada:

    ```bash
    \c mypgsqldb
    ```

4. Escriba `\q` y presione la tecla Entrar para salir de psql.

Ya está conectado al servidor de Azure Database for PostgreSQL a través de psql y ha creado una base de datos de usuarios en blanco.

## <a name="clean-up-resources"></a>Limpieza de recursos

Hay dos formas de eliminar los recursos que ha creado en la guía de inicio rápido. Puede eliminar el grupo de recursos de Azure, lo que incluye todos los recursos del grupo de recursos. Si desea mantener intactos los restantes recursos, elimine solo el recurso del servidor.

> [!TIP]
> Otras guías de inicio rápido de esta colección se basan en esta. Si tiene previsto seguir usando otras guías de inicio rápido, no elimine los recursos que ha creado en esta. Si no tiene previsto continuar, siga estos pasos para eliminar los recursos creados en esta guía de inicio rápido en el portal.

Para eliminar todo el grupo de recursos, incluido el servidor recién creado:

1. Elimine el grupo de recursos en el portal. En el menú de la izquierda, seleccione **Grupos de recursos**. A continuación, seleccione el nombre de su grupo de recursos (en el ejemplo, **myresourcegroup**).

2. En la página del grupo de recursos, seleccione **Eliminar**. Escriba el nombre del grupo de recursos, en nuestro ejemplo **myresourcegroup**, en el cuadro de texto para confirmar la eliminación. Seleccione **Eliminar**.

Para eliminar solo el servidor recién creado:

1. Busque el servidor en el portal si no lo tiene abierto. En el menú de la izquierda, seleccione **Todos los recursos**. Luego busque el servidor que ha creado.

2. En la página **Información general**, seleccione **Eliminar**.

    :::image type="content" source="./media/quickstart-create-database-portal/9-delete.png" alt-text="Botón Eliminar":::

3. Confirme el nombre del servidor que desea eliminar y vea las bases de datos que incluye y que resultan afectadas. Escriba el nombre del servidor en el cuadro de texto, como el del ejemplo, **mydemoserver**. Seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Implementación de una aplicación de Django con App Service y PostgreSQL](tutorial-django-app-service-postgres.md)
