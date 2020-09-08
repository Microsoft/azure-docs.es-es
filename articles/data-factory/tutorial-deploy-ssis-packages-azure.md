---
title: Aprovisionamiento de Azure-SSIS Integration Runtime
description: Aprenda a aprovisionar Integration Runtime de Azure SSIS en Azure Data Factory para que pueda implementar y ejecutar paquetes SSIS en Azure.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: tutorial
ms.custom: seo-lt-2019
ms.date: 08/11/2020
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: mflasko
ms.openlocfilehash: 840ccb00fdc91cc44fee46500bbc7237fe55ff2a
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88185526"
---
# <a name="provision-the-azure-ssis-integration-runtime-in-azure-data-factory"></a>Aprovisionamiento de Azure-SSIS Integration Runtime en Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este tutorial se describen los pasos necesarios para usar Azure Portal para aprovisionar el entorno de ejecución de integración de Azure-SQL Server Integration Services (ISSIS) en Azure Data Factory (ADF). Azure-SSIS Integration Runtime admite:

- La ejecución de paquetes implementados en el catálogo de SSIS (SSISDB) hospedados por un servidor de Azure SQL Database o por Instancia administrada (modelo de implementación de proyectos)
- La ejecución de paquetes implementados en el sistema de archivos, en Azure Files o en una base de datos de SQL Server (MSDB) hospedados por Instancia administrada de Azure SQL (modelo de implementación de paquetes).

Después de aprovisionar una instancia de Azure-SSIS IR, puede usar herramientas conocidas para implementar y ejecutar los paquetes en Azure. Estas herramientas ya están habilitadas para Azure e incluyen SQL Server Data Tools (SSDT), SQL Server Management Studio (SSMS) y utilidades de la línea de comandos, como [dtutil](https://docs.microsoft.com/sql/integration-services/dtutil-utility?view=sql-server-2017) y [AzureDTExec](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-azure-enabled-dtexec).

Para obtener información conceptual acerca de Integration Runtime para la integración de SSIS en Azure, consulte [Introducción a Integration Runtime de SSIS de Azure](concepts-integration-runtime.md#azure-ssis-integration-runtime).

En este tutorial, va a completar los siguientes pasos:

> [!div class="checklist"]
> * Creación de una factoría de datos.
> * Aprovisionamiento de una instancia de Integration Runtime de SSIS en Azure.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

- **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

- **Servidor de Azure SQL Database (opcional)** . Si aún no tiene un servidor de bases de datos, cree uno en Azure Portal antes de empezar. A su vez, Data Factory creará una instancia de SSISDB en este servidor de bases de datos. 

  Le recomendamos que cree el servidor de bases de datos en la misma región de Azure que el entorno de ejecución de integración. Esta configuración permite que el entorno de ejecución de integración escriba registros de ejecución en SSISDB sin traspasar regiones de Azure.

  Tenga en cuenta los siguientes puntos:

  - En función del servidor de bases de datos seleccionado, la instancia de SSISDB puede crearse en su nombre como una base de datos única, como parte de un grupo elástico o en una instancia administrada. Se puede acceder a ella en una red pública o mediante la unión a una red virtual. Para obtener asesoramiento acerca de cómo elegir el tipo de servidor de bases de datos que va a hospedar SSISDB, consulte [Comparación entre SQL Database e Instancia administrada de SQL](../data-factory/create-azure-ssis-integration-runtime.md#comparison-of-sql-database-and-sql-managed-instance). 
  
    Si usa un servidor de Azure SQL Database con reglas de firewall de IP/puntos de conexión de servicio de red virtual o una instancia administrada con punto de conexión privado para hospedar SSISDB, o si necesita acceder a datos locales sin configurar IR autohospedado, debe unir la instancia de Azure-SSIS IR a una red virtual. Para más información, consulte [Creación de una instancia de Azure-SSIS IR en una red virtual](https://docs.microsoft.com/azure/data-factory/create-azure-ssis-integration-runtime).

  - Confirme que el valor **Permitir el acceso a servicios de Azure** está habilitado para el servidor de bases de datos. Esta configuración no es aplicable si para hospedar SSISDB se usa el servidor de Azure SQL Database con reglas de firewall de red/puntos de conexión de servicio de red virtual o una instancia administrada con punto de conexión privado. Para más información, consulte [Protección de Azure SQL Database](../sql-database/sql-database-security-tutorial.md#create-firewall-rules). Para habilitar este valor mediante PowerShell, consulte [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule).

  - Agregue la dirección IP de la máquina cliente, o un intervalo de direcciones IP que la incluya, a la lista de direcciones IP de cliente de la configuración del firewall del servidor de bases de datos. Para más información, consulte [Reglas de firewall de nivel de base de datos y de nivel de servidor de Azure SQL Database](../sql-database/sql-database-firewall-configure.md).

  - Puede conectarse al servidor de bases de datos mediante la autenticación de SQL con sus credenciales de administrador del servidor o por medio de la autenticación de Azure AD con la identidad administrada de la factoría de datos. En el último de los casos, debe agregar la identidad administrada de la factoría de datos a un grupo de Azure AD con permisos de acceso al servidor de bases de datos. Para más información, consulte [Creación de una instancia de Azure-SSIS IR con autenticación de Azure AD](https://docs.microsoft.com/azure/data-factory/create-azure-ssis-integration-runtime).

  - Confirme que el servidor de bases de datos no tiene aún una instancia de SSISDB. El aprovisionamiento de Azure-SSIS IR no admite el uso de una instancia de SSISDB existente.

> [!NOTE]
> Para ver una lista de las regiones de Azure en las que Data Factory y Azure-SSIS IR están disponibles actualmente, consulte [Disponibilidad de Data Factory y SSIS IR por región](https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all). 

## <a name="create-a-data-factory"></a>Crear una factoría de datos

Para crear la factoría de datos mediante Azure Portal, siga las instrucciones paso a paso que se indican en [Creación de una factoría de datos mediante la interfaz de usuario](https://docs.microsoft.com/azure/data-factory/quickstart-create-data-factory-portal#create-a-data-factory). Seleccione **Anclar al panel** mientras lo hace, para permitir el acceso rápido después de su creación. 

Después de crear la factoría de datos, abra su página de información general en Azure Portal. Seleccione el icono **Author & monitor**  (Creación y supervisión) para abrir su página de **inicio** en otra pestaña. Ahí puede continuar con la creación de la instancia de Azure-SSIS IR.

## <a name="create-an-azure-ssis-integration-runtime"></a>Creación de un entorno de ejecución de integración de SSIS para Azure

### <a name="from-the-data-factory-overview"></a>Desde la información general de Data Factory

1. En la página **Let's get started** (Comencemos), seleccione el icono **Configure SSIS Integration Runtime** (Configuración de Integration Runtime de SSIS). 

   ![Icono "Configure SSIS Integration Runtime" (Configuración de Integration Runtime de SSIS)](./media/tutorial-create-azure-ssis-runtime-portal/configure-ssis-integration-runtime-tile.png)

1. Para el resto de pasos de configuración de Integration Runtime para la integración de SSIS en Azure, consulte la sección [Aprovisionamiento de Integration Runtime de SSIS de Azure](#provision-an-azure-ssis-integration-runtime). 

### <a name="from-the-authoring-ui"></a>Desde la interfaz de usuario de creación

1. En la interfaz de usuario de Azure Data Factory, cambie a la pestaña **Edit** (Editar) y seleccione **Connections** (Conexiones). A continuación, cambie a la pestaña **Integration Runtimes** (Entornos de ejecución de integración) para ver los entornos de ejecución de integración existentes en la factoría de datos. 

   ![Opciones para ver las instancias de Integration Runtime existentes](./media/tutorial-create-azure-ssis-runtime-portal/view-azure-ssis-integration-runtimes.png)

1. Seleccione **New** (Nuevo) para crear una instancia de Azure-SSIS IR y abrir el panel **Integration runtime setup** (Configuración del entorno de ejecución de integración). 

   ![Integration Runtime a través del menú](./media/tutorial-create-azure-ssis-runtime-portal/edit-connections-new-integration-runtime-button.png)

1. En el panel **Integration runtime setup** (Configuración de Integration Runtime), seleccione el icono **Lift-and-shift existing SSIS packages to execute in Azure** (Migrar mediante lift-and-shift los paquetes de SSIS existentes para ejecutarlos en Azure) y haga clic en **Next** (Siguiente).

   ![Especificación del tipo de instancia de Integration Runtime](./media/tutorial-create-azure-ssis-runtime-portal/integration-runtime-setup-options.png)

1. Para el resto de pasos de configuración de Integration Runtime para la integración de SSIS en Azure, consulte la sección [Aprovisionamiento de Integration Runtime de SSIS de Azure](#provision-an-azure-ssis-integration-runtime). 

## <a name="provision-an-azure-ssis-integration-runtime"></a>Aprovisionamiento de una instancia de Integration Runtime para la integración de SSIS en Azure

El panel **Integration runtime setup** (Configuración del entorno de ejecución de integración) tiene tres páginas en las que puede configurar opciones generales, de implementación y avanzadas respectivamente.

### <a name="general-settings-page"></a>Página General settings (Configuración general)

En la página **General settings** (Configuración general) del panel **Integration runtime setup** (Configuración de Integration Runtime), realice los pasos siguientes. 

   ![Configuración general](./media/tutorial-create-azure-ssis-runtime-portal/general-settings.png)

   1. En **Name** (Nombre), escriba el nombre del entorno de ejecución de integración. 

   1. En **Description** (Descripción), escriba la descripción del entorno de ejecución de integración. 

   1. En **Location** (Ubicación), seleccione la ubicación del entorno de ejecución de integración. Se muestran solo las ubicaciones admitidas. Le recomendamos que seleccione la misma ubicación del servidor de bases de datos que va a hospedar SSISDB. 

   1. En **Node Size** (Tamaño de nodo), seleccione el tamaño de nodo del clúster del entorno de ejecución de integración. Se muestran solo los tamaños de nodo admitidos. Seleccione un tamaño de nodo grande (escalado vertical) si quiere ejecutar muchos paquetes de uso intensivo de proceso o memoria. 

   1. En **Node Number** (Número de nodos), seleccione el número de nodos del clúster del entorno de ejecución de integración. Se muestran solo los números de nodos admitidos. Seleccione un clúster de grande con muchos nodos (escalado horizontal), si quiere ejecutar muchos paquetes en paralelo. 

   1. En **Edition/License** (Edición o licencia), seleccione la edición de SQL Server para el entorno de ejecución de integración: Standard o Enterprise. Seleccione Enterprise si quiere usar características avanzadas en el entorno de ejecución de integración. 

   1. En **Save Money** (Ahorrar dinero), seleccione la opción Azure Hybrid Benefit (Ventaja híbrida de Azure) para el entorno de ejecución de integración: **Yes** (Sí) o **No**. Seleccione **Yes** (Sí) si quiere que su propia licencia de SQL Server con Software Assurance se beneficie de los ahorros con el uso híbrido. 

   1. Seleccione **Next** (Siguiente). 

### <a name="deployment-settings-page"></a>Página Deployment settings (Configuración de implementación)

En la página **Deployment settings** (Configuración de implementación) del panel **Integration runtime setup** (Configuración de Integration Runtime), tiene las opciones para crear los almacenes de paquetes de Azure-SSIS IR o SSISDB.

#### <a name="creating-ssisdb"></a>Creación de SSISDB

En la página **Deployment settings** (Configuración de implementación) del panel **Integration runtime setup** (Configuración de Integration Runtime), si desea implementar los paquetes en SSISDB (Modelo de implementación de proyectos), active la casilla **Create SSIS catalog (SSISDB) hosted by Azure SQL Database server/Managed Instance to store your projects/packages/environments/execution logs** (Crear catálogo de SSIS [SSISDB] hospedado en el servidor de Azure SQL Database/Instancia administrada para almacenar los proyectos/paquetes/entornos/registros de ejecución). De forma alternativa, no necesita crear una instancia de SSISDB ni activar la casilla, si quiere implementar los paquetes en el sistema de archivos, en Azure Files o en una base de datos de SQL Server (MSDB) hospedados por Azure SQL Managed Instance (modelo de implementación de paquetes).

Con independencia del modelo de implementación, active esta casilla de todos modos si quiere usar Agente SQL Server hospedado por Azure SQL Managed Instance para orquestar o programar las ejecuciones de paquetes, ya que está habilitado por SSISDB. Para obtener más información, consulte el artículo [Programación de ejecuciones de paquetes SSIS mediante el agente de Instancia administrada de Azure SQL](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-managed-instance-agent).
   
Si activa la casilla, realice los pasos siguientes para traer su propio servidor de bases de datos y hospedar la instancia de SSISDB que se creará y administrará en su nombre.

   ![Configuración de implementación de SSISDB](./media/tutorial-create-azure-ssis-runtime-portal/deployment-settings.png)
   
   1. En **Subscription** (Suscripción), seleccione la suscripción de Azure que tiene el servidor de bases de datos que va a hospedar SSISDB. 

   1. En **Location** (Ubicación), seleccione la ubicación del servidor de bases de datos que va a hospedar SSISDB. Es recomendable seleccionar la misma ubicación del entorno de ejecución de integración.

   1. En **Catalog Database Server Endpoint** (Punto de conexión del servidor de bases de datos de catálogo), seleccione el punto de conexión de su servidor de bases de datos que va a hospedar SSISDB. 
   
      En función del servidor de bases de datos seleccionado, la instancia de SSISDB puede crearse en su nombre como una base de datos única, como parte de un grupo elástico o en una instancia administrada. Se puede acceder a ella en una red pública o mediante la unión a una red virtual. Para obtener asesoramiento acerca de cómo elegir el tipo de servidor de bases de datos que va a hospedar SSISDB, consulte [Comparación entre SQL Database e Instancia administrada de SQL](../data-factory/create-azure-ssis-integration-runtime.md#comparison-of-sql-database-and-sql-managed-instance).   

      Si selecciona un servidor de Azure SQL Database con reglas de firewall de IP/puntos de conexión de servicio de red virtual o una instancia administrada con un punto de conexión privado para hospedar SSISDB, o si necesita acceder a los datos locales sin configurar IR autohospedado, debe unir la instancia de Azure-SSIS IR a una red virtual. Para más información, consulte [Creación de una instancia de Azure-SSIS IR en una red virtual](https://docs.microsoft.com/azure/data-factory/create-azure-ssis-integration-runtime).

   1. Active la casilla **Use Azure AD authentication with the managed identity for your ADF** (Usar la autenticación de AAD con la identidad administrada en su ADF) para seleccionar el método de autenticación del servidor de bases de datos donde hospedar SSISDB. Elegirá la autenticación de SQL o la autenticación de Azure AD con la identidad administrada para su factoría de datos.

      Si activa la casilla, deberá agregar la identidad administrada de la factoría de datos a un grupo de Azure AD con permisos de acceso al servidor de bases de datos. Para más información, consulte [Creación de una instancia de Azure-SSIS IR con autenticación de Azure AD](https://docs.microsoft.com/azure/data-factory/create-azure-ssis-integration-runtime).
   
   1. En **Admin Username** (Nombre de usuario del administrador), escriba el nombre de usuario de autenticación de SQL del servidor de bases de datos para hospedar SSISDB. 

   1. En **Admin Password** (Contraseña del administrador), escriba la contraseña de la autenticación de SQL del servidor de bases de datos para hospedar SSISDB. 

   1. En **Catalog Database Service Tier** (Nivel de servicio de bases de datos de catálogo), seleccione el nivel de servicio del servidor de bases de datos para hospedar SSISDB. Seleccione el nivel de servicio Basic, Standard o Premium, o seleccione un nombre de grupo elástico.

Seleccione **Test connection** (Prueba de conexión) cuando proceda y, si la prueba se realiza correctamente, seleccione **Next** (Siguiente).

#### <a name="creating-azure-ssis-ir-package-stores"></a>Creación de almacenes de paquetes de Azure-SSIS IR

En la página **Deployment settings** (Configuración de implementación) del panel **Integration runtime setup** (Configuración de Integration Runtime), si desea administrar los paquetes implementados en MSDB, el sistema de archivos o Azure Files (Modelo de implementación de paquetes) con almacenes de paquetes de Azure-SSIS IR, active la casilla **Create package stores to manage your packages that are deployed into file system/Azure Files/SQL Server database (MSDB) hosted by Azure SQL Managed Instance** (Crear almacenes de paquetes para administrar los paquetes implementados en el sistema de archivos, en Azure Files o en una base de datos de SQL Server [MSDB] hospedados por Instancia administrada de Azure SQL).
   
Los almacenes de paquetes de Azure-SSIS IR permiten importar, exportar, eliminar y ejecutar paquetes, así como supervisar y detener los paquetes en ejecución mediante SSMS de forma similar al [almacén de paquetes de SSIS heredado](https://docs.microsoft.com/sql/integration-services/service/package-management-ssis-service?view=sql-server-2017). Para más información, vea el artículo [Administración de paquetes SSIS mediante almacenes de paquetes de Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/azure-ssis-integration-runtime-package-store).
   
Si activa esta casilla, puede seleccionar **New** (Nuevo) para agregar varios almacenes de paquetes a una instancia de Azure-SSIS IR. Por otro lado, un almacén de paquetes se puede compartir con varias instancias de Azure-SSIS IR.

![Configuración de implementación para MSDB, un sistema de archivos o Azure Files](./media/tutorial-create-azure-ssis-runtime-portal/deployment-settings2.png)

En el panel **Add package store** (Adición de un almacén de paquetes), siga estos pasos:
   
   1. En **Package store name** (Nombre del almacén de paquetes), escriba el nombre del almacén de paquetes. 

   1. En **Package store linked service** (Servicio vinculado de almacén de paquetes), seleccione el servicio vinculado existente que almacena la información de acceso del sistema de archivos, de Azure Files o de Instancia administrada de Azure SQL, donde se implementan los paquetes, o cree un servicio vinculado nuevo; para ello, seleccione **New** (Nuevo). En el panel **New linked service** (Nuevo servicio vinculado), realice los pasos siguientes. 

      ![Configuración de implementación de servicios vinculados](./media/tutorial-create-azure-ssis-runtime-portal/deployment-settings-linked-service.png)

      1. En **Name** (Nombre), escriba el nombre del servicio vinculado. 
         
      1. En **Description** (Descripción), escriba la descripción del servicio vinculado. 
         
      1. En **Type** (Tipo), seleccione **Azure File Storage**, **Azure SQL Managed Instance** (Instancia administrada de Azure SQL) o **File System** (Sistema de archivos).

      1. Puede omitir la opción **Connect via integration runtime** (Conectarse a través de IR), ya que siempre se usa su instancia de Azure-SSIS IR para obtener la información de acceso de los almacenes de paquetes.

      1. Si selecciona **Azure File Storage**, haga lo siguiente: 

         1. En **Account selection method** (Método de selección de cuenta), seleccione **From Azure subscription** (Desde la suscripción de Azure) o **Enter manually** (Especificar manualmente).
         
         1. Si selecciona **From Azure subscription** (Desde la suscripción de Azure), escoja la **suscripción de Azure**, el **nombre de la cuenta de almacenamiento** y el **recurso compartido de archivos** correspondientes.
            
         1. Si selecciona **Enter manually** (Especificar manualmente), escriba `\\<storage account name>.file.core.windows.net\<file share name>` en **Host**, `Azure\<storage account name>` en **Username** (Nombre de usuario), y `<storage account key>` en **Password** (Contraseña), o bien seleccione su instancia de **Azure Key Vault** donde se almacena la contraseña como secreto.

      1. Si selecciona **Azure SQL Managed Instance** (Instancia administrada de Azure SQL), siga estos pasos: 

         1. Seleccione **Connection string** (Cadena de conexión) para escribir la cadena de conexión manualmente o elija la instancia de **Azure Key Vault** donde se almacena como secreto.
         
         1. En caso de seleccionar **Connection string** (Cadena de conexión), realice los siguientes pasos: 

            1. En **Fully qualified domain name** (Nombre de dominio completo), escriba `<server name>.<dns prefix>.database.windows.net` o `<server name>.public.<dns prefix>.database.windows.net,3342` como el punto de conexión público o privado (respectivamente) de Instancia administrada de Azure SQL. Si escribe el punto de conexión privado, no se puede usar la **prueba de conexión**, ya que la interfaz de usuario de ADF no puede acceder a ella.

            1. En **Database name** (Nombre de la base de datos), escriba `msdb`.
               
            1. En **Authentication type** (Tipo de autenticación), seleccione **SQL Authentication** (Autenticación de SQL), **Managed Identity** (Identidad administrada) o **Service Principal** (Entidad de servicio).

            1. Si selecciona **SQL Authentication** (Autenticación de SQL), escriba el **nombre de usuario** y la **contraseña** correspondientes, o bien seleccione la instancia de **Azure Key Vault** donde se almacena la contraseña como secreto.

            1. Si selecciona **Managed Identity** (Identidad administrada), conceda a la identidad administrada de ADF acceso a su Instancia administrada de Azure SQL.

            1. Si selecciona **Service Principal** (Entidad de servicio), escriba el **identificador de entidad de servicio** y la **clave de entidad de servicio** correspondientes, o bien seleccione la instancia de **Azure Key Vault** donde se almacena la clave como secreto.

      1. Si selecciona **File system** (Sistema de archivos), escriba en **Host** la ruta de acceso UNC de la carpeta en la que se implementan los paquetes y especifique el **nombre de usuario** y la **contraseña** correspondientes, o bien seleccione la instancia de **Azure Key Vault** donde se almacena la contraseña como secreto.

      1. Seleccione **Test connection** (Prueba de conexión) cuando proceda y, si la prueba se realiza correctamente, seleccione **Create** (Crear).

   1. Los almacenes de paquetes agregados aparecerán en la página **Deployment settings** (Configuración de implementación). Para quitarlas, active sus casillas y, luego, seleccione **Eliminar**.

Seleccione **Test connection** (Prueba de conexión) cuando proceda y, si la prueba se realiza correctamente, seleccione **Next** (Siguiente).

### <a name="advanced-settings-page"></a>Página Advanced settings (Configuración avanzada)

En la página **Advanced settings** (Configuración avanzada) del panel **Integration runtime setup** (Configuración de Integration Runtime), realice los pasos siguientes. 

   ![Configuración avanzada](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings.png)

   1. En **Maximum Parallel Executions Per Node** (Número máximo de ejecuciones en paralelo por nodo), seleccione el número máximo de paquetes que se van a ejecutar simultáneamente por nodo en el clúster del entorno de ejecución de integración. Se muestran solo los números de paquetes admitidos. Seleccione un número bajo si quiere usar más de un núcleo para ejecutar un único paquete grande o pesado con un uso intensivo de memoria o proceso. Seleccione un número alto si quiere ejecutar uno o varios paquetes pequeños en un único núcleo. 

   1. Active la casilla **Customize your Azure-SSIS Integration Runtime with additional system configurations/component installations** (Personalizar Azure-SSIS Integration Runtime con configuraciones de sistema/instalación de componentes adicionales) para elegir si quiere agregar instalaciones personalizadas estándar/rápidas a Azure-SSIS IR. Para más información, consulte [Instalación personalizada de una instancia de Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/how-to-configure-azure-ssis-ir-custom-setup).
   
   1. Seleccione la casilla **Select a VNet for your Azure-SSIS Integration Runtime to join, allow ADF to create certain network resources, and optionally bring your own static public IP addresses** (Seleccionar una red virtual para Azure-SSIS Integration Runtime para unirse, permitir que ADF cree determinados recursos de red y, opcionalmente traer sus direcciones IP públicas propias) para elegir si desea unir la instancia de Azure-SSIS IR a una red virtual.

      Seleccione esta casilla si usa un servidor de Azure SQL Database con reglas de firewall de IP/puntos de conexión de servicio de red virtual o una instancia administrada con punto de conexión privado para hospedar SSISDB, o si necesita acceder a datos locales sin configurar un IR autohospedado. Para más información, consulte [Creación de una instancia de Azure-SSIS IR en una red virtual](https://docs.microsoft.com/azure/data-factory/create-azure-ssis-integration-runtime). 
   
   1. Active la casilla **Set up Self-Hosted Integration Runtime as a proxy for your Azure-SSIS Integration Runtime** (Configurar el entorno de ejecución de integración autohospedado como proxy para su instancia de Azure-SSIS IR) para elegir si quiere configurar IR autohospedado como proxy para su instancia de Azure-SSIS IR. Para más información, consulte [Configuración de un entorno de ejecución de integración autohospedado como proxy](https://docs.microsoft.com/azure/data-factory/self-hosted-integration-runtime-proxy-ssis).   

   1. Seleccione **Continuar**. 

En la página **Summary** (Resumen) del panel **Integration runtime setup** (Configuración de Integration Runtime), revise todos los valores de aprovisionamiento, marque los vínculos a la documentación recomendada y seleccione **Finish** (Finalizar) para empezar la creación del entorno de ejecución de integración. 

   > [!NOTE]
   > Aparte del tiempo de configuración personalizada, este proceso debería realizarse en 5 minutos.
   >
   > Si usa SSISDB, el servicio Data Factory se conectará al servidor de bases de datos para prepararlo. 
   > 
   > Al aprovisionar Azure-SSIS IR, también se instalan el acceso redistribuible y el paquete de característica de Azure para SSIS. Estos componentes proporcionan conectividad a archivos de Excel y Access, y a diversos orígenes de datos de Azure, además de a aquellos que ya admiten los componentes integrados. Para obtener más información sobre los componentes integrados o preinstalados, vea el artículo [Componentes integrados y preinstalados en Azure-SSIS Integration Runtime](https://docs.microsoft.com/azure/data-factory/built-in-preinstalled-components-ssis-integration-runtime). Para obtener más información sobre los componentes adicionales que puede instalar, consulte [Personalización de la instalación en una instancia de Azure-SSIS Integration Runtime](https://docs.microsoft.com/azure/data-factory/how-to-configure-azure-ssis-ir-custom-setup).

### <a name="connections-pane"></a>Panel Connections (Conexiones)

En el panel **Connections** (Conexiones) del **centro de administración**, cambie a la página **Integration Runtimes** (Entornos de ejecución de integración) y seleccione **Refresh** (Actualizar). 

   ![Panel Connections (Conexiones)](./media/tutorial-create-azure-ssis-runtime-portal/connections-pane.png)

   Puede editar y volver a configurar su instancia de Azure-SSIS IR; para ello, seleccione su nombre. También puede seleccionar los botones pertinentes para supervisar, iniciar, detener o eliminar su instancia de Azure-SSIS IR; puede generar automáticamente una canalización de ADF con la actividad Ejecutar paquete de SSIS para que se ejecute en su instancia de Azure-SSIS IR; y puede ver el código o la carga JSON de Azure-SSIS IR.  Solo puede editar o eliminar su instancia de Azure-SSIS IR si está detenida.

## <a name="deploy-ssis-packages"></a>Implementación de paquetes de SSIS

Si usa SSISDB, puede implementar los paquetes en esta base de datos y ejecutarlos en Azure-SSIS IR mediante las herramientas de SSDT o de SSMS habilitadas para Azure. Estas herramientas se conectan al servidor de bases de datos mediante su punto de conexión de servidor: 

- En el caso de un servidor de Azure SQL Database, el formato del punto de conexión de servidor es `<server name>.database.windows.net`.
- En el caso de una instancia administrada con punto de conexión privado, el formato del punto de conexión de servidor es `<server name>.<dns prefix>.database.windows.net`.
- En el caso de una instancia administrada con punto de conexión público, el formato del punto de conexión de servidor es `<server name>.public.<dns prefix>.database.windows.net,3342`. 

Si no usa SSISDB, puede implementar los paquetes en sistemas de archivos, en Azure Files o en MSDB hospedados por Azure SQL Managed Instance y ejecutarlos en Azure-SSIS IR mediante las utilidades de la línea de comandos [dtutil](https://docs.microsoft.com/sql/integration-services/dtutil-utility?view=sql-server-2017) y [AzureDTExec](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-azure-enabled-dtexec). 

Para obtener más información, vea [Implementación de proyectos o paquetes en SSIS](https://docs.microsoft.com/sql/integration-services/packages/deploy-integration-services-ssis-projects-and-packages?view=sql-server-ver15).

En ambos casos, también puede ejecutar los paquetes implementados en Azure-SSIS IR mediante la actividad de ejecución de paquetes SSIS de las canalizaciones de Data Factory. Para más información, consulte [Invocación de la ejecución de paquetes SSIS como una actividad de Data Factory de primera clase](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity).

Consulte también la documentación de SSIS siguiente: 

- [Implementación, ejecución y supervisión de paquetes de SSIS en Azure](/sql/integration-services/lift-shift/ssis-azure-deploy-run-monitor-tutorial) 
- [Conexión a SSISDB en Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database) 
- [Programación de la ejecución de paquetes en Azure](/sql/integration-services/lift-shift/ssis-azure-schedule-packages) 
- [Connect to on-premises data sources with Windows authentication](/sql/integration-services/lift-shift/ssis-azure-connect-with-windows-auth) (Conexión a orígenes de datos locales con autenticación de Windows) 

## <a name="next-steps"></a>Pasos siguientes

Para obtener información acerca de la personalización del entorno de ejecución de integración de SSIS de Azure, vaya al siguiente artículo: 

> [!div class="nextstepaction"]
> [Personalización de Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/how-to-configure-azure-ssis-ir-custom-setup)