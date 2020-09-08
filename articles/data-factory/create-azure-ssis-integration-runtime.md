---
title: Crear una instancia de Integration Runtime de Azure SSIS en Azure Data Factory
description: Obtenga información sobre cómo crear un entorno de ejecución de integración de Azure SSIS en Azure Data Factory para que pueda implementar y ejecutar paquetes SSIS en Azure.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/11/2020
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: mflasko
ms.openlocfilehash: 07cfb0048e6027b0bac219b3fe28018db2d10257
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88185271"
---
# <a name="create-an-azure-ssis-integration-runtime-in-azure-data-factory"></a>Crear una instancia de Integration Runtime de Azure SSIS en Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se describen los pasos para aprovisionar una instancia de Azure SQL Server Integration Services (SSIS) Integration Runtime (IR) en Azure Data Factory (ADF). Azure-SSIS Integration Runtime admite:

- La ejecución de paquetes implementados en el catálogo de SSIS (SSISDB) hospedados por un servidor de Azure SQL Database o por Instancia administrada (modelo de implementación de proyectos)
- La ejecución de paquetes implementados en el sistema de archivos, en Azure Files o en una base de datos de SQL Server (MSDB) hospedados por Instancia administrada de Azure SQL (modelo de implementación de paquetes).

Después de aprovisionar una instancia de Azure-SSIS IR, puede usar herramientas conocidas para implementar y ejecutar los paquetes en Azure. Estas herramientas ya están habilitadas para Azure e incluyen SQL Server Data Tools (SSDT), SQL Server Management Studio (SSMS) y utilidades de la línea de comandos, como [dtutil](https://docs.microsoft.com/sql/integration-services/dtutil-utility?view=sql-server-2017) y [AzureDTExec](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-azure-enabled-dtexec).

En el tutorial [Aprovisionamiento de Azure-SSIS IR ](tutorial-create-azure-ssis-runtime-portal.md) se muestra cómo crear una instancia de Azure-SSIS IR mediante Azure Portal o la aplicación Data Factory. También se muestra cómo usar de manera opcional un servidor o una instancia administrada de Azure SQL Database para hospedar SSISDB. Este artículo es una extensión del tutorial y en él se describe cómo hacer las siguientes tareas opcionales:

- Usar un servidor de Azure SQL Database con reglas de firewall de IP/puntos de conexión de servicio de red virtual o una instancia administrada con un punto de conexión privado para hospedar SSISDB. Como requisito previo, debe configurar los permisos y valores de la red virtual para que la instancia de Azure-SSIS IR se una a una red virtual.

- Usar la autenticación de Azure Active Directory (AAD) con la identidad administrada de la factoría de datos para conectarse a un servidor o a una instancia administrada de Azure SQL Database. Como requisito previo, debe agregar la identidad administrada de la factoría de datos como un usuario de base de datos capaz de crear instancias de SSISDB.

- Unir la instancia de Azure-SSIS IR a una red virtual o configurar IR autohospedado como proxy para que Azure-SSIS IR tenga acceso a los datos locales.

En este artículo se muestra cómo aprovisionar una instancia de Azure-SSIS IR mediante Azure Portal, Azure PowerShell y una plantilla de Azure Resource Manager.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

- **Suscripción de Azure**. Si aún no tiene una suscripción, puede crear una cuenta de [evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

- **Servidor Azure SQL Database o SQL Managed Instance (opcional)** . Si aún no tiene un servidor de bases de datos o una instancia administrada, cree uno en Azure Portal antes de empezar. A su vez, Data Factory creará una instancia de SSISDB en este servidor de bases de datos. 

  Le recomendamos que cree el servidor de bases de datos o la instancia administrada en la misma región de Azure que el entorno de ejecución de integración. Esta configuración permite que el entorno de ejecución de integración escriba registros de ejecución en SSISDB sin traspasar regiones de Azure.

  Tenga en cuenta los siguientes puntos:

  - La instancia de SSISDB puede crearse en su nombre como una base de datos única, como parte de un grupo elástico o en una instancia administrada. Se puede acceder a ella en una red pública o mediante la unión a una red virtual. Para más información sobre cómo elegir entre SQL Database y SQL Managed Instance para hospedar SSISDB, consulte la sección [Comparación entre SQL Database y SQL Managed Instance](#comparison-of-sql-database-and-sql-managed-instance) de este artículo. 
  
    Si usa un servidor de Azure SQL Database con reglas de firewall de IP/puntos de conexión de servicio de red virtual o una instancia administrada de SQL con punto de conexión privado para hospedar SSISDB, o si necesita acceder a datos locales sin configurar IR autohospedado, debe unir la instancia de Azure-SSIS IR a una red virtual. Para más información, consulte [Unión de una instancia de Azure-SSIS IR a una red virtual](https://docs.microsoft.com/azure/data-factory/join-azure-ssis-integration-runtime-virtual-network).

  - Confirme que el valor **Permitir el acceso a servicios de Azure** está habilitado para el servidor de bases de datos. Esta configuración no es aplicable si para hospedar SSISDB se usa el servidor de Azure SQL Database con reglas de firewall de red/puntos de conexión de servicio de red virtual o una instancia administrada de SQL con punto de conexión privado. Para más información, consulte [Protección de Azure SQL Database](../sql-database/sql-database-security-tutorial.md#create-firewall-rules). Para habilitar este valor mediante PowerShell, consulte [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule).

  - Agregue la dirección IP de la máquina cliente, o un intervalo de direcciones IP que la incluya, a la lista de direcciones IP de cliente de la configuración del firewall del servidor de bases de datos. Para más información, consulte [Reglas de firewall de nivel de base de datos y de nivel de servidor de Azure SQL Database](../sql-database/sql-database-firewall-configure.md).

  - Puede conectarse al servidor de bases de datos mediante la autenticación de SQL con sus credenciales de administrador del servidor o por medio de la autenticación de Azure AD con la identidad administrada de la factoría de datos. En el último de los casos, debe agregar la identidad administrada de la factoría de datos a un grupo de Azure AD con permisos de acceso al servidor de bases de datos. Para más información, consulte [Habilitar la autenticación de Azure AD para una instancia de Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/enable-aad-authentication-azure-ssis-ir).

  - Confirme que el servidor de bases de datos no tiene aún una instancia de SSISDB. El aprovisionamiento de Azure-SSIS IR no admite el uso de una instancia de SSISDB existente.

- **Red virtual de Azure Resource Manager (opcional)** . Debe tener una red virtual de Azure Resource Manager si se cumple al menos una de las siguientes condiciones:

  - Va a hospedar SSISDB en un servidor de Azure SQL Database con reglas de firewall de IP/puntos de conexión de servicio de red virtual o una instancia administrada con un punto de conexión privado.

  - Quiere conectarse a almacenes de datos locales desde paquetes SSIS que se ejecutan en Azure-SSIS IR sin configurar IR autohospedado.

- **Azure PowerShell (opcional)** . Siga las instrucciones que se indican en [Instalación y configuración de Azure PowerShell](/powershell/azure/install-az-ps), si quiere ejecutar un script de PowerShell para aprovisionar Azure-SSIS IR.

### <a name="regional-support"></a>Compatibilidad regional

Para ver una lista de regiones de Azure en las que Data Factory y Azure-SSIS IR están disponibles, consulte [Disponibilidad de Data Factory y SSIS IR por región](https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all).

### <a name="comparison-of-sql-database-and-sql-managed-instance"></a>Comparación entre SQL Database e Instancia administrada de SQL

En la tabla siguiente se comparan determinadas características de un servidor de Azure SQL Database y SQL Managed Instance y su relación con Azure SSIR IR:

| Característica | SQL Database| SQL Managed Instance |
|---------|--------------|------------------|
| **Programación** | El agente SQL Server no está disponible.<br/><br/>Consulte [Programar una ejecución de paquetes en una canalización de Data Factory](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-schedule-packages?view=sql-server-2017#activity).| El agente de Instancia administrada está disponible. |
| **Autenticación** | Puede crear una instancia de SSISDB con un usuario de base de datos independiente que represente cualquier grupo de Azure AD con la identidad administrada de la factoría de datos como miembro del rol **db_owner**.<br/><br/>Consulte [Habilitación de la autenticación de Azure AD para crear una instancia de SSISDB en el servidor de Azure SQL Database](enable-aad-authentication-azure-ssis-ir.md#enable-azure-ad-on-azure-sql-database). | Puede crear una instancia de SSISDB con un usuario de base de datos independiente que represente la identidad administrada de la factoría de datos. <br/><br/>Consulte [Habilitación de la autenticación de Azure AD para crear una instancia de SSISDB en Instancia administrada de Azure SQL](enable-aad-authentication-azure-ssis-ir.md#enable-azure-ad-on-sql-managed-instance). |
| **Nivel de servicio** | Al crear una instancia de Azure-SSIS IR con el servidor de Azure SQL Database, puede seleccionar el nivel de servicio de SSISDB. Hay varios niveles de servicio. | Cuando crea una instancia de Azure-SSIS IR con la instancia administrada, no puede seleccionar el nivel de servicio de SSISDB. Todas las bases de datos de la instancia administrada comparten el mismo recurso asignado a esa instancia. |
| **Red virtual** | Su instancia de Azure-SSIS IR puede unirse a una red virtual de Azure Resource Manager si usa un servidor de Azure SQL Database con reglas de firewall de IP/puntos de conexión de servicio de red virtual. | Su instancia de Azure-SSIS IR puede unirse a una red virtual Azure Resource Manager si usa una instancia administrada con un punto de conexión privado. La red virtual es necesaria cuando no se habilita un punto de conexión público para la instancia administrada.<br/><br/>Si une la instancia de Azure-SSIS IR a la misma red virtual que la instancia administrada, asegúrese de que ambas instancias se encuentran en subredes diferentes. Si une la instancia de Azure-SSIS IR a una red virtual diferente de aquella de la instancia administrada, se recomienda realizar un emparejamiento de red virtual o una conexión de red virtual a red virtual. Consulte [Conexión de la aplicación a una instancia administrada de Azure SQL Database](../sql-database/sql-database-managed-instance-connect-app.md). |
| **Transacciones distribuidas** | Esta característica se admite mediante transacciones elásticas. No se admiten las transacciones del Coordinador de transacciones distribuidas de Microsoft (MSDTC). Si los paquetes SSIS usan MSDTC para coordinar las transacciones distribuidas, considere la posibilidad de migrar a transacciones elásticas de Azure SQL Database. Para más información, consulte [Transacciones distribuidas en bases de datos en la nube](../sql-database/sql-database-elastic-transactions-overview.md). | No compatible. |
| | | |

## <a name="use-the-azure-portal-to-create-an-integration-runtime"></a>Uso de Azure Portal para crear un entorno de ejecución de integración

En esta sección, se usa Azure Portal, y en concreto la interfaz de usuario o la aplicación de Data Factory, para crear una instancia de Azure-SSIS IR.

### <a name="create-a-data-factory"></a>Crear una factoría de datos

Para crear la factoría de datos mediante Azure Portal, siga las instrucciones paso a paso que se indican en [Creación de una factoría de datos mediante la interfaz de usuario](https://docs.microsoft.com/azure/data-factory/quickstart-create-data-factory-portal#create-a-data-factory). Seleccione **Anclar al panel** mientras lo hace, para permitir el acceso rápido después de su creación. 

Después de crear la factoría de datos, abra su página de información general en Azure Portal. Seleccione el icono **Author & monitor**  (Creación y supervisión) para abrir su página de **inicio** en otra pestaña. Ahí puede continuar con la creación de la instancia de Azure-SSIS IR.   

### <a name="provision-an-azure-ssis-integration-runtime"></a>Aprovisionamiento de una instancia de Integration Runtime para la integración de SSIS en Azure

En la página **Let's get started** (Comencemos), seleccione el icono **Configure SSIS Integration Runtime** (Configuración de Azure-SSIS Integration Runtime) para abrir el panel **Integration runtime setup** (Configuración de Integration Runtime).

   ![Icono Configure SSIS Integration Runtime (Configuración de Integration Runtime de SSIS)](./media/tutorial-create-azure-ssis-runtime-portal/configure-ssis-integration-runtime-tile.png)

   El panel **Integration runtime setup** (Configuración de Integration Runtime) tiene tres páginas en las que pueden configurar opciones generales, de implementación y avanzadas respectivamente.

#### <a name="general-settings-page"></a>Página General settings (Configuración general)

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

#### <a name="deployment-settings-page"></a>Página Deployment settings (Configuración de implementación)

En la página **Deployment settings** (Configuración de implementación) del panel **Integration runtime setup** (Configuración de Integration Runtime), tiene las opciones para crear almacenes de paquetes de SSISDB o de Azure-SSIS IR.

##### <a name="creating-ssisdb"></a>Creación de SSISDB

En la página **Deployment settings** (Configuración de implementación) de **Integration runtime setup** (Configuración de Integration Runtime), si quiere implementar los paquetes en SSISDB (modelo de implementación de proyectos), active la casilla **Create SSIS catalog (SSISDB) hosted by Azure SQL Database server/Managed Instance to store your projects/packages/environments/execution logs** (Crear catálogo de SSIS [SSISDB] hospedado en el servidor de Azure SQL Database/Instancia administrada para almacenar los proyectos/paquetes/entornos/registros de ejecución). De forma alternativa, no necesita crear una instancia de SSISDB ni activar la casilla si quiere implementar los paquetes en el sistema de archivos, en Azure Files o en una base de datos de SQL Server (MSDB) hospedados por Azure SQL Managed Instance (modelo de implementación de paquetes).

Con independencia del modelo de implementación, active esta casilla de todos modos si quiere usar Agente SQL Server hospedado por Azure SQL Managed Instance para orquestar o programar las ejecuciones de paquetes, ya que está habilitado por SSISDB. Para obtener más información, consulte el artículo [Programación de ejecuciones de paquetes SSIS mediante el agente de Instancia administrada de Azure SQL](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-managed-instance-agent).
   
Si activa la casilla, realice los pasos siguientes para traer su propio servidor de bases de datos para hospedar la instancia de SSISDB que se creará y administrará en su nombre.

   ![Configuración de implementación de SSISDB](./media/tutorial-create-azure-ssis-runtime-portal/deployment-settings.png)
   
   1. En **Subscription** (Suscripción), seleccione la suscripción de Azure que tiene el servidor de bases de datos que va a hospedar SSISDB. 

   1. En **Location** (Ubicación), seleccione la ubicación del servidor de bases de datos que va a hospedar SSISDB. Es recomendable seleccionar la misma ubicación del entorno de ejecución de integración.

   1. En **Catalog Database Server Endpoint** (Punto de conexión del servidor de bases de datos de catálogo), seleccione el punto de conexión de su servidor de bases de datos que va a hospedar SSISDB. 
   
      En función del servidor de bases de datos seleccionado, la instancia de SSISDB puede crearse en su nombre como una base de datos única, como parte de un grupo elástico o en una instancia administrada. Se puede acceder a ella en una red pública o mediante la unión a una red virtual. Para obtener asesoramiento acerca de cómo elegir el tipo de servidor de bases de datos que va a hospedar SSISDB, consulte [Comparación entre SQL Database e Instancia administrada de SQL](../data-factory/create-azure-ssis-integration-runtime.md#comparison-of-sql-database-and-sql-managed-instance).   

      Si selecciona un servidor de Azure SQL Database con reglas de firewall de IP/puntos de conexión de servicio de red virtual o una instancia administrada con un punto de conexión privado para hospedar SSISDB, o si necesita acceder a los datos locales sin configurar IR autohospedado, debe unir la instancia de Azure-SSIS IR a una red virtual. Para más información, consulte [Creación de una instancia de Azure-SSIS IR en una red virtual](https://docs.microsoft.com/azure/data-factory/create-azure-ssis-integration-runtime).

   1. Active la casilla **Use AAD authentication with the managed identity for your ADF** (Usar la autenticación de AAD con la identidad administrada en su ADF) para seleccionar el método de autenticación del servidor de bases de datos para hospedar SSISDB. Elegirá la autenticación de SQL o la autenticación de Azure AD con la identidad administrada para su factoría de datos.

      Si activa la casilla, deberá agregar la identidad administrada de la factoría de datos a un grupo de Azure AD con permisos de acceso al servidor de bases de datos. Para más información, consulte [Creación de una instancia de Azure-SSIS IR con autenticación de Azure AD](https://docs.microsoft.com/azure/data-factory/create-azure-ssis-integration-runtime).
   
   1. En **Admin Username** (Nombre de usuario del administrador), escriba el nombre de usuario de autenticación de SQL del servidor de bases de datos para hospedar SSISDB. 

   1. En **Admin Password** (Contraseña del administrador), escriba la contraseña de la autenticación de SQL del servidor de bases de datos para hospedar SSISDB. 

   1. En **Catalog Database Service Tier** (Nivel de servicio de bases de datos de catálogo), seleccione el nivel de servicio del servidor de bases de datos para hospedar SSISDB. Seleccione el nivel de servicio Basic, Standard o Premium, o seleccione un nombre de grupo elástico.

Seleccione **Test connection** (Prueba de conexión) cuando proceda y, si la prueba se realiza correctamente, seleccione **Next** (Siguiente).

##### <a name="creating-azure-ssis-ir-package-stores"></a>Creación de almacenes de paquetes de Azure-SSIS IR

En la página **Deployment settings** (Configuración de implementación) del panel **Integration runtime setup** (Configuración de Integration Runtime), active la casilla **Create package stores to manage your packages that are deployed into file system/Azure Files/SQL Server database (MSDB) hosted by Azure SQL Managed Instance** (Crear almacenes de paquetes para administrar los paquetes implementados en el sistema de archivos, en Azure Files o en una base de datos de SQL Server [MSDB] hospedados por Instancia administrada de Azure SQL) si quiere administrar los paquetes que se implementan en MSDB, el sistema de archivos o Azure Files (modelo de implementación de paquetes).
   
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

#### <a name="advanced-settings-page"></a>Página Advanced settings (Configuración avanzada)

En la página **Advanced settings** (Configuración avanzada) del panel **Integration runtime setup** (Configuración de Integration Runtime), realice los pasos siguientes.

   ![Configuración avanzada](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings.png)

   1. En **Maximum Parallel Executions Per Node** (Número máximo de ejecuciones en paralelo por nodo), seleccione el número máximo de paquetes que se van a ejecutar simultáneamente por nodo en el clúster del entorno de ejecución de integración. Se muestran solo los números de paquetes admitidos. Seleccione un número bajo si quiere usar más de un núcleo para ejecutar un único paquete grande o pesado con un uso intensivo de memoria o proceso. Seleccione un número alto si quiere ejecutar uno o varios paquetes pequeños en un único núcleo.

   1. Active la casilla **Customize your Azure-SSIS Integration Runtime with additional system configurations/component installations** (Personalizar Azure-SSIS Integration Runtime con configuraciones de sistema/instalación de componentes adicionales) para elegir si quiere agregar instalaciones personalizadas estándar/rápidas a Azure-SSIS IR. Para más información, consulte [Instalación personalizada de una instancia de Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/how-to-configure-azure-ssis-ir-custom-setup).

      Si activa esta casilla, haga lo siguiente:

      ![Configuración avanzada con instalaciones personalizadas](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings-custom.png)
   
      1. En **Custom setup container SAS URI** (URI de SAS del contenedor de instalación personalizada), escriba el URI de SAS del contenedor en el que se almacenan los scripts y los archivos asociados para las instalaciones personalizadas estándar.

      1. En **Express custom setup** (Instalación personalizada rápida), seleccione **Nueva** para abrir el panel **Agregar instalación personalizada rápida** y después seleccione cualquier tipo en el menú desplegable **Express custom setup type** (Tipo de instalación personalizada rápida); por ejemplo, **Run cmdkey command** (Ejecutar comando cmdkey), **Agregar variable de entorno**, **Install licensed component** (Instalar componente con licencia), etc.

         Si selecciona el tipo **Install licensed component** (Instalar componente con licencia), puede seleccionar los componentes integrados de nuestros partners de ISV en el menú desplegable **Component name** (Nombre del componente) y, si fuera necesario, puede escribir la clave de licencia del producto o cargar el archivo de licencia que ha comprado en el cuadro **License key**/**License file** (Clave de licencia/Archivo de licencia).
  
         Las instalaciones rápidas personalizadas que agregue aparecerán en la página **Advanced settings** (Configuración avanzada). Para quitarlas, puede activar las casillas de verificación y seleccionar **Eliminar**.

   1. Seleccione la casilla **Select a VNet for your Azure-SSIS Integration Runtime to join, allow ADF to create certain network resources, and optionally bring your own static public IP addresses** (Seleccionar una red virtual para unir Azure-SSIS Integration Runtime, permitir que ADF cree determinados recursos de red y opcionalmente traer sus direcciones IP públicas propias) para elegir si quiere unir el entorno de ejecución de integración a una red virtual. 

      Active la casilla si usa un servidor de Azure SQL Database con reglas de firewall de IP/puntos de conexión de servicio de red virtual o una instancia administrada con un punto de conexión privado para hospedar SSISDB o si necesita acceder a los datos locales (es decir, tiene orígenes o destinos de datos locales en los paquetes de SSIS) sin configurar un IR autohospedado. Para obtener más información, consulte [Unión de una instancia de Azure-SSIS IR a una red virtual](https://docs.microsoft.com/azure/data-factory/join-azure-ssis-integration-runtime-virtual-network). 

      Si activa esta casilla, haga lo siguiente:

      ![Configuración avanzada con una red virtual](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings-vnet.png)

      1. En **Subscription** (Suscripción), seleccione la suscripción de Azure que tiene la red virtual.

      1. En **Location** (Ubicación), está seleccionada la misma ubicación del entorno de ejecución de integración.

      1. En **Type** (Tipo), seleccione el tipo de red virtual: clásica o de Azure Resource Manager. Se recomienda seleccionar una red virtual de Azure Resource Manager, puesto que las redes virtuales clásicas dejarán de utilizarse pronto.

      1. En **VNet Name** (Nombre de red virtual), seleccione el nombre de la red virtual. Debe ser la misma que se usa para el servidor de Azure SQL Database con puntos de conexión de servicio de red virtual o instancia administrada con un punto de conexión privado para hospedar SSISDB. O bien, la misma que está conectada a la red local. De lo contrario, puede ser cualquier red virtual para traer sus propias direcciones IP públicas estáticas de Azure-SSIS IR.

      1. En **Subnet Name** (Nombre de subred), seleccione el nombre de la subred en la red virtual. Debe ser la misma que se usa para el servidor de Azure SQL Database con puntos de conexión de servicio de red virtual para hospedar SSISDB. O bien, debe ser una subred diferente de la que se usa en la instancia administrada con un punto de conexión privado para hospedar SSISDB. De lo contrario, puede ser cualquier subred para traer sus propias direcciones IP públicas estáticas de Azure-SSIS IR.

      1. Active la casilla **Bring static public IP addresses for your Azure-SSIS Integration Runtime** (Traer direcciones IP públicas estáticas para Azure-SSIS Integration Runtime) para elegir si quiere traer sus propias direcciones IP públicas estáticas para Azure-SSIS IR, de modo que pueda habilitarlas en el firewall para los orígenes de datos.

         Si activa esta casilla, haga lo siguiente:

         1. En **First static public IP address** (Primera dirección IP pública estática), seleccione la primera dirección IP pública estática que cumpla con los requisitos de su instancia de Azure-SSIS IR. Si no tiene ninguna, haga clic en el vínculo **Crear nueva** para crear direcciones IP públicas estáticas en Azure Portal y, a continuación, haga clic en el botón actualizar aquí para poder seleccionarlas.
      
         1. En **Second static public IP address** (Segunda dirección IP pública estática), seleccione la primera dirección IP pública estática que cumpla con los requisitos de su instancia de Azure-SSIS IR. Si no tiene ninguna, haga clic en el vínculo **Crear nueva** para crear direcciones IP públicas estáticas en Azure Portal y, a continuación, haga clic en el botón actualizar aquí para poder seleccionarlas.

   1. Active la casilla **Set up Self-Hosted Integration Runtime as a proxy for your Azure-SSIS Integration Runtime** (Configurar el entorno de ejecución de integración autohospedado como proxy para su instancia de Azure-SSIS IR) para elegir si quiere configurar IR autohospedado como proxy para su instancia de Azure-SSIS IR. Para obtener más información, consulte [Configuración de un entorno de ejecución de integración autohospedado como proxy](https://docs.microsoft.com/azure/data-factory/self-hosted-integration-runtime-proxy-ssis). 

      Si activa esta casilla, haga lo siguiente:

      ![Configuración avanzada con IR autohospedado](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings-shir.png)

      1. En **Self-Hosted Integration Runtime** (Entorno de ejecución de integración autohospedado), seleccione la instancia de IR autohospedado existente como proxy para Azure-SSIS IR.

      1. En **Staging Storage Linked Service** (Servicio vinculado de almacenamiento provisional), seleccione el servicio vinculado de Azure Blob Storage existente o cree uno para el almacenamiento provisional.

      1. En **Ruta de acceso provisional**, especifique un contenedor de blobs en la instancia seleccionada de Azure Blob Storage o deje este campo vacío para usar uno predeterminado para el almacenamiento provisional.

   1. Seleccione **VNet Validation** (Validación de red virtual)  > **Continuar**. 

En la sección **Resumen**, revise todos los valores de aprovisionamiento, marque los vínculos a la documentación recomendada y seleccione **Finalizar** para empezar la creación del entorno de ejecución de integración.

   > [!NOTE]
   > Aparte del tiempo de configuración personalizada, este proceso debería realizarse en 5 minutos. Sin embargo, Azure-SSIS IR puede tardar entre 20 y 30 minutos en unirse a una red virtual.
   >
   > Si usa SSISDB, el servicio Data Factory se conectará al servidor de bases de datos para prepararlo. También configura los permisos o los valores de la red virtual, si se especifica, y une la instancia de Azure-SSIS IR a la red virtual.
   > 
   > Al aprovisionar Azure-SSIS IR, también se instalan el acceso redistribuible y el paquete de característica de Azure para SSIS. Estos componentes proporcionan conectividad a archivos de Excel y Access, y a diversos orígenes de datos de Azure, además de a aquellos que ya admiten los componentes integrados. Para obtener más información sobre los componentes integrados o preinstalados, vea el artículo [Componentes integrados y preinstalados en Azure-SSIS Integration Runtime](https://docs.microsoft.com/azure/data-factory/built-in-preinstalled-components-ssis-integration-runtime). Para obtener más información sobre los componentes adicionales que puede instalar, consulte [Personalización de la instalación en una instancia de Azure-SSIS Integration Runtime](https://docs.microsoft.com/azure/data-factory/how-to-configure-azure-ssis-ir-custom-setup).

#### <a name="connections-pane"></a>Panel Connections (Conexiones)

En el panel **Connections** (Conexiones) del **centro de administración**, cambie a la página **Integration Runtimes** (Entornos de ejecución de integración) y seleccione **Refresh** (Actualizar). 

   ![Panel Connections (Conexiones)](./media/tutorial-create-azure-ssis-runtime-portal/connections-pane.png)

   Puede editar y volver a configurar su instancia de Azure-SSIS IR; para ello, seleccione su nombre. También puede seleccionar los botones pertinentes para supervisar, iniciar, detener o eliminar su instancia de Azure-SSIS IR; puede generar automáticamente una canalización de ADF con la actividad Ejecutar paquete de SSIS para que se ejecute en su instancia de Azure-SSIS IR; y puede ver el código o la carga JSON de Azure-SSIS IR.  Solo puede editar o eliminar su instancia de Azure-SSIS IR si está detenida.

### <a name="azure-ssis-integration-runtimes-in-the-portal"></a>Instancias de Integration Runtime de SSIS de Azure en el portal

1. En la interfaz de usuario de Azure Data Factory, cambie a la pestaña **Edit** (Editar) y seleccione **Connections** (Conexiones). A continuación, cambie a la pestaña **Integration Runtimes** (Entornos de ejecución de integración) para ver los entornos de ejecución de integración existentes en la factoría de datos.

   ![Visualización de las instancias de IR existentes](./media/tutorial-create-azure-ssis-runtime-portal/view-azure-ssis-integration-runtimes.png)

1. Seleccione **New** (Nuevo) para crear una nueva instancia de Azure-SSIS IR y abrir el panel **Integration runtime setup** (Configuración de Integration Runtime).

   ![Integration Runtime a través del menú](./media/tutorial-create-azure-ssis-runtime-portal/edit-connections-new-integration-runtime-button.png)

1. En el panel **Integration runtime setup** (Configuración de Integration Runtime), seleccione el icono **Lift-and-shift existing SSIS packages to execute in Azure** (Migrar mediante lift-and-shift los paquetes de SSIS existentes para ejecutarlos en Azure) y haga clic en **Next** (Siguiente).

   ![Especificación del tipo de instancia de Integration Runtime](./media/tutorial-create-azure-ssis-runtime-portal/integration-runtime-setup-options.png)

1. Para conocer el resto de pasos de configuración de Azure-SSIS IR, consulte la sección [Aprovisionamiento de una instancia de Azure-SSIS IR](#provision-an-azure-ssis-integration-runtime).

## <a name="use-azure-powershell-to-create-an-integration-runtime"></a>Uso de Azure PowerShell para crear un entorno de ejecución de integración

En esta sección, se usa Azure PowerShell para crear una instancia de Azure-SSIS IR.

### <a name="create-variables"></a>Creación de variables

Copie y pegue el siguiente script: Especifique valores para las variables. 

```powershell
### Azure Data Factory info
# If your input contains a PSH special character like "$", precede it with the escape character "`" - for example, "`$"
$SubscriptionName = "[your Azure subscription name]"
$ResourceGroupName = "[your Azure resource group name]"
# Data factory name - must be globally unique
$DataFactoryName = "[your data factory name]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$DataFactoryLocation = "EastUS"

### Azure-SSIS integration runtime info - This is a Data Factory compute resource for running SSIS packages.
$AzureSSISName = "[your Azure-SSIS IR name]"
$AzureSSISDescription = "[your Azure-SSIS IR description]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$AzureSSISLocation = "EastUS"
# For supported node sizes, see https://azure.microsoft.com/pricing/details/data-factory/ssis/
$AzureSSISNodeSize = "Standard_D8_v3"
# 1-10 nodes are currently supported
$AzureSSISNodeNumber = 2
# Azure-SSIS IR edition/license info: Standard or Enterprise
$AzureSSISEdition = "Standard" # Standard by default, whereas Enterprise lets you use advanced features on your Azure-SSIS IR
# Azure-SSIS IR hybrid usage info: LicenseIncluded or BasePrice
$AzureSSISLicenseType = "LicenseIncluded" # LicenseIncluded by default, whereas BasePrice lets you bring your own on-premises SQL Server license with Software Assurance to earn cost savings from Azure Hybrid Benefit option
# For a Standard_D1_v2 node, up to 4 parallel executions per node are supported. For other nodes, up to (2 x number of cores) are currently supported.
$AzureSSISMaxParallelExecutionsPerNode = 8
# Custom setup info: Standard/express custom setups
$SetupScriptContainerSasUri = "" # OPTIONAL to provide a SAS URI of blob container for standard custom setup where your script and its associated files are stored
$ExpressCustomSetup = "[RunCmdkey|SetEnvironmentVariable|InstallAzurePowerShell|SentryOne.TaskFactory|oh22is.SQLPhonetics.NET|oh22is.HEDDA.IO|KingswaySoft.IntegrationToolkit|KingswaySoft.ProductivityPack|Theobald.XtractIS|AecorSoft.IntegrationService or leave it empty]" # OPTIONAL to configure an express custom setup without script
# Virtual network info: Classic or Azure Resource Manager
$VnetId = "[your virtual network resource ID or leave it empty]" # REQUIRED if you use an Azure SQL Database server with IP firewall rules/virtual network service endpoints or a managed instance with private endpoint to host SSISDB, or if you require access to on-premises data without configuring a self-hosted IR. We recommend an Azure Resource Manager virtual network, because classic virtual networks will be deprecated soon.
$SubnetName = "[your subnet name or leave it empty]" # WARNING: Use the same subnet as the one used for your Azure SQL Database server with virtual network service endpoints, or a different subnet from the one used for your managed instance with a private endpoint
# Public IP address info: OPTIONAL to provide two standard static public IP addresses with DNS name under the same subscription and in the same region as your virtual network
$FirstPublicIP = "[your first public IP address resource ID or leave it empty]"
$SecondPublicIP = "[your second public IP address resource ID or leave it empty]"

### SSISDB info
$SSISDBServerEndpoint = "[your Azure SQL Database server name.database.windows.net or managed instance name.DNS prefix.database.windows.net or managed instance name.public.DNS prefix.database.windows.net,3342 or leave it empty if you do not use SSISDB]" # WARNING: If you use SSISDB, ensure that there's no existing SSISDB on your database server, so we can prepare and manage one on your behalf
# Authentication info: SQL or Azure AD
$SSISDBServerAdminUserName = "[your server admin username for SQL authentication or leave it empty for Azure AD authentication]"
$SSISDBServerAdminPassword = "[your server admin password for SQL authentication or leave it empty for Azure AD authentication]"
# For the basic pricing tier, specify "Basic," not "B." For standard, premium, and elastic pool tiers, specify "S0," "S1," "S2," "S3," etc. See https://docs.microsoft.com/azure/sql-database/sql-database-resource-limits-database-server.
$SSISDBPricingTier = "[Basic|S0|S1|S2|S3|S4|S6|S7|S9|S12|P1|P2|P4|P6|P11|P15|…|ELASTIC_POOL(name = <elastic_pool_name>) for Azure SQL Database server or leave it empty for managed instance]"

### Self-hosted integration runtime info - This can be configured as a proxy for on-premises data access 
$DataProxyIntegrationRuntimeName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingLinkedServiceName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingPath = "" # OPTIONAL to configure a proxy for on-premises data access 
```

### <a name="sign-in-and-select-a-subscription"></a>Inicie sesión y seleccione una suscripción.

Agregue el siguiente script para iniciar sesión y seleccione la suscripción de Azure.

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName $SubscriptionName
```

### <a name="validate-the-connection-to-database-server"></a>Validación de la conexión al servidor de bases de datos

Agregue el siguiente script para validar el servidor o la instancia administrada de Azure SQL Database.

```powershell
# Validate only if you use SSISDB and you don't use virtual network or Azure AD authentication
if(![string]::IsNullOrEmpty($SSISDBServerEndpoint))
{
    if([string]::IsNullOrEmpty($VnetId) -and [string]::IsNullOrEmpty($SubnetName))
    {
        if(![string]::IsNullOrEmpty($SSISDBServerAdminUserName) -and ![string]::IsNullOrEmpty($SSISDBServerAdminPassword))
        {
            $SSISDBConnectionString = "Data Source=" + $SSISDBServerEndpoint + ";User ID=" + $SSISDBServerAdminUserName + ";Password=" + $SSISDBServerAdminPassword
            $sqlConnection = New-Object System.Data.SqlClient.SqlConnection $SSISDBConnectionString;
            Try
            {
                $sqlConnection.Open();
            }
            Catch [System.Data.SqlClient.SqlException]
            {
                Write-Warning "Cannot connect to your Azure SQL Database server, exception: $_";
                Write-Warning "Please make sure the server you specified has already been created. Do you want to proceed? [Y/N]"
                $yn = Read-Host
                if(!($yn -ieq "Y"))
                {
                    Return;
                }
            }
        }
    }
}
```

### <a name="configure-the-virtual-network"></a>Configuración de la red virtual

Agregue el siguiente script para configurar automáticamente los permisos y las opciones de red virtual para unir la instancia del entorno de ejecución de integración de Azure SSIS.

```powershell
# Make sure to run this script against the subscription to which the virtual network belongs
if(![string]::IsNullOrEmpty($VnetId) -and ![string]::IsNullOrEmpty($SubnetName))
{
    # Register to the Azure Batch resource provider
    $BatchApplicationId = "ddbf3205-c6bd-46ae-8127-60eb93363864"
    $BatchObjectId = (Get-AzADServicePrincipal -ServicePrincipalName $BatchApplicationId).Id
    Register-AzResourceProvider -ProviderNamespace Microsoft.Batch
    while(!(Get-AzResourceProvider -ProviderNamespace "Microsoft.Batch").RegistrationState.Contains("Registered"))
    {
    Start-Sleep -s 10
    }
    if($VnetId -match "/providers/Microsoft.ClassicNetwork/")
    {
        # Assign the VM contributor role to Microsoft.Batch
        New-AzRoleAssignment -ObjectId $BatchObjectId -RoleDefinitionName "Classic Virtual Machine Contributor" -Scope $VnetId
    }
}
```

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) con el comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran recursos de Azure como un grupo.

Si ya existe el grupo de recursos, no copie este código en el script. 

```powershell
New-AzResourceGroup -Location $DataFactoryLocation -Name $ResourceGroupName
```

### <a name="create-a-data-factory"></a>Crear una factoría de datos

Ejecute el comando siguiente para crear una factoría de datos.

```powershell
Set-AzDataFactoryV2 -ResourceGroupName $ResourceGroupName `
    -Location $DataFactoryLocation `
    -Name $DataFactoryName
```

### <a name="create-an-integration-runtime"></a>Creación de una instancia de Integration Runtime

Ejecute los siguientes comandos para crear una instancia de Integration Runtime de SSIS de Azure que ejecute paquetes de SSIS en Azure.

Si no usa SSISDB, puede omitir los parámetros `CatalogServerEndpoint`, `CatalogPricingTier` y `CatalogAdminCredential`.

Si no usa un servidor de Azure SQL Database con reglas de firewall de IP/puntos de conexión de servicio de red virtual o una instancia administrada con un punto de conexión privado para hospedar SSISDB ni necesita acceso a los datos locales, puede omitir los parámetros `VNetId` y `Subnet` o pasarles valores vacíos. También puede omitirlos si configura un IR autohospedado como proxy para el acceso de la instancia de Azure-SSIS IR a los datos locales. En los demás casos no puede omitirlos y debe pasar valores válidos de la configuración de red virtual. Para más información, consulte [Unión de una instancia de Azure-SSIS IR a una red virtual](https://docs.microsoft.com/azure/data-factory/join-azure-ssis-integration-runtime-virtual-network).

Si usa una instancia administrada para hospedar SSISDB, puede omitir el parámetro `CatalogPricingTier` o pasarle un valor vacío. De lo contrario, no puede omitirlo y debe pasar un valor válido de la lista de planes de tarifa admitidos para Azure SQL Database. Para más información, consulte [Límites de recursos de SQL Database](../sql-database/sql-database-resource-limits.md).

Si usa la autenticación de Azure AD con la identidad administrada de su factoría de datos para conectarse al servidor de bases de datos, puede omitir el parámetro `CatalogAdminCredential`. Sin embargo, debe agregar la identidad administrada de la factoría de datos a un grupo de Azure AD con permisos de acceso al servidor de bases de datos. Para más información, consulte [Habilitar la autenticación de Azure AD para una instancia de Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/enable-aad-authentication-azure-ssis-ir). En caso contrario, no puede omitirlo y debe pasar un objeto válido formado por su nombre de usuario y contraseña de administrador del servidor para la autenticación de SQL.

```powershell
Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Description $AzureSSISDescription `
    -Type Managed `
    -Location $AzureSSISLocation `
    -NodeSize $AzureSSISNodeSize `
    -NodeCount $AzureSSISNodeNumber `
    -Edition $AzureSSISEdition `
    -LicenseType $AzureSSISLicenseType `
    -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode `
    -VnetId $VnetId `
    -Subnet $SubnetName
       
# Add the CatalogServerEndpoint, CatalogPricingTier, and CatalogAdminCredential parameters if you use SSISDB
if(![string]::IsNullOrEmpty($SSISDBServerEndpoint))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -CatalogServerEndpoint $SSISDBServerEndpoint `
        -CatalogPricingTier $SSISDBPricingTier

    if(![string]::IsNullOrEmpty($SSISDBServerAdminUserName) –and ![string]::IsNullOrEmpty($SSISDBServerAdminPassword)) # Add the CatalogAdminCredential parameter if you don't use Azure AD authentication
    {
        $secpasswd = ConvertTo-SecureString $SSISDBServerAdminPassword -AsPlainText -Force
        $serverCreds = New-Object System.Management.Automation.PSCredential($SSISDBServerAdminUserName, $secpasswd)

        Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
            -DataFactoryName $DataFactoryName `
            -Name $AzureSSISName `
            -CatalogAdminCredential $serverCreds
    }
}

# Add custom setup parameters if you use standard/express custom setups
if(![string]::IsNullOrEmpty($SetupScriptContainerSasUri))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -SetupScriptContainerSasUri $SetupScriptContainerSasUri
}
if(![string]::IsNullOrEmpty($ExpressCustomSetup))
{
    if($ExpressCustomSetup -eq "RunCmdkey")
    {
        $addCmdkeyArgument = "YourFileShareServerName or YourAzureStorageAccountName.file.core.windows.net"
        $userCmdkeyArgument = "YourDomainName\YourUsername or azure\YourAzureStorageAccountName"
        $passCmdkeyArgument = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourPassword or YourAccessKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.CmdkeySetup($addCmdkeyArgument, $userCmdkeyArgument, $passCmdkeyArgument)
    }
    if($ExpressCustomSetup -eq "SetEnvironmentVariable")
    {
        $variableName = "YourVariableName"
        $variableValue = "YourVariableValue"
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.EnvironmentVariableSetup($variableName, $variableValue)
    }
    if($ExpressCustomSetup -eq "InstallAzurePowerShell")
    {
        $moduleVersion = "YourAzModuleVersion"
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.AzPowerShellSetup($moduleVersion)
    }
    if($ExpressCustomSetup -eq "SentryOne.TaskFactory")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "oh22is.SQLPhonetics.NET")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "oh22is.HEDDA.IO")
    {
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup)
    }
    if($ExpressCustomSetup -eq "KingswaySoft.IntegrationToolkit")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "KingswaySoft.ProductivityPack")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }    
    if($ExpressCustomSetup -eq "Theobald.XtractIS")
    {
        $jsonData = Get-Content -Raw -Path YourLicenseFile.json
        $jsonData = $jsonData -replace '\s',''
        $jsonData = $jsonData.replace('"','\"')
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString($jsonData)
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "AecorSoft.IntegrationService")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    # Create an array of one or more express custom setups
    $setups = New-Object System.Collections.ArrayList
    $setups.Add($setup)

    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -ExpressCustomSetup $setups
}

# Add self-hosted integration runtime parameters if you configure a proxy for on-premises data accesss
if(![string]::IsNullOrEmpty($DataProxyIntegrationRuntimeName) -and ![string]::IsNullOrEmpty($DataProxyStagingLinkedServiceName))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -DataProxyIntegrationRuntimeName $DataProxyIntegrationRuntimeName `
        -DataProxyStagingLinkedServiceName $DataProxyStagingLinkedServiceName

    if(![string]::IsNullOrEmpty($DataProxyStagingPath))
    {
        Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
            -DataFactoryName $DataFactoryName `
            -Name $AzureSSISName `
            -DataProxyStagingPath $DataProxyStagingPath
    }
}

# Add public IP address parameters if you bring your own static public IP addresses
if(![string]::IsNullOrEmpty($FirstPublicIP) -and ![string]::IsNullOrEmpty($SecondPublicIP))
{
    $publicIPs = @($FirstPublicIP, $SecondPublicIP)
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -PublicIPs $publicIPs
}
```

### <a name="start-the-integration-runtime"></a>Inicio del entorno de ejecución de integración

Ejecute los siguientes comandos para iniciar la instancia de Azure-SSIS Integration Runtime.

```powershell
write-host("##### Starting #####")
Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Force

write-host("##### Completed #####")
write-host("If any cmdlet is unsuccessful, please consider using -Debug option for diagnostics.")
```

> [!NOTE]
> Aparte del tiempo de configuración personalizada, este proceso debería realizarse en 5 minutos. Sin embargo, Azure-SSIS IR puede tardar entre 20 y 30 minutos en unirse a una red virtual.
>
> Si usa SSISDB, el servicio Data Factory se conectará al servidor de bases de datos para prepararlo. También configura los permisos o los valores de la red virtual, si se especifica, y une la instancia de Azure-SSIS IR a la red virtual.
> 
> Al aprovisionar Azure-SSIS IR, también se instalan el acceso redistribuible y el paquete de característica de Azure para SSIS. Estos componentes proporcionan conectividad a archivos de Excel y Access, y a diversos orígenes de datos de Azure, además de a aquellos que ya admiten los componentes integrados. Para obtener más información sobre los componentes integrados o preinstalados, vea el artículo [Componentes integrados y preinstalados en Azure-SSIS Integration Runtime](https://docs.microsoft.com/azure/data-factory/built-in-preinstalled-components-ssis-integration-runtime). Para obtener más información sobre los componentes adicionales que puede instalar, consulte [Personalización de la instalación en una instancia de Azure-SSIS Integration Runtime](https://docs.microsoft.com/azure/data-factory/how-to-configure-azure-ssis-ir-custom-setup).

### <a name="full-script"></a>Script completo

Este es el script completo con el que se crea un entorno de ejecución de integración de Azure SSIS.

```powershell
### Azure Data Factory info
# If your input contains a PSH special character like "$", precede it with the escape character "`" - for example, "`$"
$SubscriptionName = "[your Azure subscription name]"
$ResourceGroupName = "[your Azure resource group name]"
# Data factory name - must be globally unique
$DataFactoryName = "[your data factory name]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$DataFactoryLocation = "EastUS"

### Azure-SSIS integration runtime info - This is a Data Factory compute resource for running SSIS packages.
$AzureSSISName = "[your Azure-SSIS IR name]"
$AzureSSISDescription = "[your Azure-SSIS IR description]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$AzureSSISLocation = "EastUS"
# For supported node sizes, see https://azure.microsoft.com/pricing/details/data-factory/ssis/
$AzureSSISNodeSize = "Standard_D8_v3"
# 1-10 nodes are currently supported
$AzureSSISNodeNumber = 2
# Azure-SSIS IR edition/license info: Standard or Enterprise
$AzureSSISEdition = "Standard" # Standard by default, whereas Enterprise lets you use advanced features on your Azure-SSIS IR
# Azure-SSIS IR hybrid usage info: LicenseIncluded or BasePrice
$AzureSSISLicenseType = "LicenseIncluded" # LicenseIncluded by default, whereas BasePrice lets you bring your own on-premises SQL Server license with Software Assurance to earn cost savings from the Azure Hybrid Benefit option
# For a Standard_D1_v2 node, up to four parallel executions per node are supported. For other nodes, up to (2 x number of cores) are currently supported.
$AzureSSISMaxParallelExecutionsPerNode = 8
# Custom setup info: Standard/express custom setups
$SetupScriptContainerSasUri = "" # OPTIONAL to provide a SAS URI of blob container for standard custom setup where your script and its associated files are stored
$ExpressCustomSetup = "[RunCmdkey|SetEnvironmentVariable|InstallAzurePowerShell|SentryOne.TaskFactory|oh22is.SQLPhonetics.NET|oh22is.HEDDA.IO|KingswaySoft.IntegrationToolkit|KingswaySoft.ProductivityPack|Theobald.XtractIS|AecorSoft.IntegrationService or leave it empty]" # OPTIONAL to configure an express custom setup without script
# Virtual network info: Classic or Azure Resource Manager
$VnetId = "[your virtual network resource ID or leave it empty]" # REQUIRED if you use an Azure SQL Database server with IP firewall rules/virtual network service endpoints or a managed instance with private endpoint to host SSISDB, or if you require access to on-premises data without configuring a self-hosted IR. We recommend an Azure Resource Manager virtual network, because classic virtual networks will be deprecated soon.
$SubnetName = "[your subnet name or leave it empty]" # WARNING: Use the same subnet as the one used for your Azure SQL Database server with virtual network service endpoints, or a different subnet from the one used for your managed instance with a private endpoint
# Public IP address info: OPTIONAL to provide two standard static public IP addresses with DNS name under the same subscription and in the same region as your virtual network
$FirstPublicIP = "[your first public IP address resource ID or leave it empty]"
$SecondPublicIP = "[your second public IP address resource ID or leave it empty]"

### SSISDB info
$SSISDBServerEndpoint = "[your Azure SQL Database server name.database.windows.net or managed instance name.DNS prefix.database.windows.net or managed instance name.public.DNS prefix.database.windows.net,3342 or leave it empty if you do not use SSISDB]" # WARNING: If you use SSISDB, ensure that there's no existing SSISDB on your database server, so we can prepare and manage one on your behalf
# Authentication info: SQL or Azure AD
$SSISDBServerAdminUserName = "[your server admin username for SQL authentication or leave it empty for Azure AD authentication]"
$SSISDBServerAdminPassword = "[your server admin password for SQL authentication or leave it empty for Azure AD authentication]"
# For the basic pricing tier, specify "Basic," not "B." For standard, premium, and elastic pool tiers, specify "S0," "S1," "S2," "S3," etc. See https://docs.microsoft.com/azure/sql-database/sql-database-resource-limits-database-server.
$SSISDBPricingTier = "[Basic|S0|S1|S2|S3|S4|S6|S7|S9|S12|P1|P2|P4|P6|P11|P15|…|ELASTIC_POOL(name = <elastic_pool_name>) for Azure SQL Database server or leave it empty for managed instance]"

### Self-hosted integration runtime info - This can be configured as a proxy for on-premises data access 
$DataProxyIntegrationRuntimeName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingLinkedServiceName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingPath = "" # OPTIONAL to configure a proxy for on-premises data access 

### Sign in and select a subscription
Connect-AzAccount
Select-AzSubscription -SubscriptionName $SubscriptionName

### Validate the connection to the database server
# Validate only if you use SSISDB and don't use a virtual network or Azure AD authentication
if(![string]::IsNullOrEmpty($SSISDBServerEndpoint))
{
    if([string]::IsNullOrEmpty($VnetId) -and [string]::IsNullOrEmpty($SubnetName))
    {
        if(![string]::IsNullOrEmpty($SSISDBServerAdminUserName) -and ![string]::IsNullOrEmpty($SSISDBServerAdminPassword))
        {
            $SSISDBConnectionString = "Data Source=" + $SSISDBServerEndpoint + ";User ID=" + $SSISDBServerAdminUserName + ";Password=" + $SSISDBServerAdminPassword
            $sqlConnection = New-Object System.Data.SqlClient.SqlConnection $SSISDBConnectionString;
            Try
            {
                $sqlConnection.Open();
            }
            Catch [System.Data.SqlClient.SqlException]
            {
                Write-Warning "Cannot connect to your Azure SQL Database server, exception: $_";
                Write-Warning "Please make sure the server you specified has already been created. Do you want to proceed? [Y/N]"
                $yn = Read-Host
                if(!($yn -ieq "Y"))
                {
                    Return;
                }
            }
        }
    }
}

### Configure a virtual network
# Make sure to run this script against the subscription to which the virtual network belongs
if(![string]::IsNullOrEmpty($VnetId) -and ![string]::IsNullOrEmpty($SubnetName))
{
    # Register to the Azure Batch resource provider
    $BatchApplicationId = "ddbf3205-c6bd-46ae-8127-60eb93363864"
    $BatchObjectId = (Get-AzADServicePrincipal -ServicePrincipalName $BatchApplicationId).Id
    Register-AzResourceProvider -ProviderNamespace Microsoft.Batch
    while(!(Get-AzResourceProvider -ProviderNamespace "Microsoft.Batch").RegistrationState.Contains("Registered"))
    {
    Start-Sleep -s 10
    }
    if($VnetId -match "/providers/Microsoft.ClassicNetwork/")
    {
        # Assign the VM contributor role to Microsoft.Batch
        New-AzRoleAssignment -ObjectId $BatchObjectId -RoleDefinitionName "Classic Virtual Machine Contributor" -Scope $VnetId
    }
}

### Create a data factory
Set-AzDataFactoryV2 -ResourceGroupName $ResourceGroupName `
    -Location $DataFactoryLocation `
    -Name $DataFactoryName

### Create an integration runtime
Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Description $AzureSSISDescription `
    -Type Managed `
    -Location $AzureSSISLocation `
    -NodeSize $AzureSSISNodeSize `
    -NodeCount $AzureSSISNodeNumber `
    -Edition $AzureSSISEdition `
    -LicenseType $AzureSSISLicenseType `
    -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode `
    -VnetId $VnetId `
    -Subnet $SubnetName
       
# Add CatalogServerEndpoint, CatalogPricingTier, and CatalogAdminCredential parameters if you use SSISDB
if(![string]::IsNullOrEmpty($SSISDBServerEndpoint))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -CatalogServerEndpoint $SSISDBServerEndpoint `
        -CatalogPricingTier $SSISDBPricingTier

    if(![string]::IsNullOrEmpty($SSISDBServerAdminUserName) –and ![string]::IsNullOrEmpty($SSISDBServerAdminPassword)) # Add the CatalogAdminCredential parameter if you don't use Azure AD authentication
    {
        $secpasswd = ConvertTo-SecureString $SSISDBServerAdminPassword -AsPlainText -Force
        $serverCreds = New-Object System.Management.Automation.PSCredential($SSISDBServerAdminUserName, $secpasswd)

        Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
            -DataFactoryName $DataFactoryName `
            -Name $AzureSSISName `
            -CatalogAdminCredential $serverCreds
    }
}

# Add custom setup parameters if you use standard/express custom setups
if(![string]::IsNullOrEmpty($SetupScriptContainerSasUri))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -SetupScriptContainerSasUri $SetupScriptContainerSasUri
}
if(![string]::IsNullOrEmpty($ExpressCustomSetup))
{
    if($ExpressCustomSetup -eq "RunCmdkey")
    {
        $addCmdkeyArgument = "YourFileShareServerName or YourAzureStorageAccountName.file.core.windows.net"
        $userCmdkeyArgument = "YourDomainName\YourUsername or azure\YourAzureStorageAccountName"
        $passCmdkeyArgument = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourPassword or YourAccessKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.CmdkeySetup($addCmdkeyArgument, $userCmdkeyArgument, $passCmdkeyArgument)
    }
    if($ExpressCustomSetup -eq "SetEnvironmentVariable")
    {
        $variableName = "YourVariableName"
        $variableValue = "YourVariableValue"
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.EnvironmentVariableSetup($variableName, $variableValue)
    }
    if($ExpressCustomSetup -eq "InstallAzurePowerShell")
    {
        $moduleVersion = "YourAzModuleVersion"
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.AzPowerShellSetup($moduleVersion)
    }
    if($ExpressCustomSetup -eq "SentryOne.TaskFactory")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "oh22is.SQLPhonetics.NET")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "oh22is.HEDDA.IO")
    {
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup)
    }
    if($ExpressCustomSetup -eq "KingswaySoft.IntegrationToolkit")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "KingswaySoft.ProductivityPack")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }    
    if($ExpressCustomSetup -eq "Theobald.XtractIS")
    {
        $jsonData = Get-Content -Raw -Path YourLicenseFile.json
        $jsonData = $jsonData -replace '\s',''
        $jsonData = $jsonData.replace('"','\"')
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString($jsonData)
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "AecorSoft.IntegrationService")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    # Create an array of one or more express custom setups
    $setups = New-Object System.Collections.ArrayList
    $setups.Add($setup)

    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -ExpressCustomSetup $setups
}

# Add self-hosted integration runtime parameters if you configure a proxy for on-premises data accesss
if(![string]::IsNullOrEmpty($DataProxyIntegrationRuntimeName) -and ![string]::IsNullOrEmpty($DataProxyStagingLinkedServiceName))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -DataProxyIntegrationRuntimeName $DataProxyIntegrationRuntimeName `
        -DataProxyStagingLinkedServiceName $DataProxyStagingLinkedServiceName

    if(![string]::IsNullOrEmpty($DataProxyStagingPath))
    {
        Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
            -DataFactoryName $DataFactoryName `
            -Name $AzureSSISName `
            -DataProxyStagingPath $DataProxyStagingPath
    }
}

# Add public IP address parameters if you bring your own static public IP addresses
if(![string]::IsNullOrEmpty($FirstPublicIP) -and ![string]::IsNullOrEmpty($SecondPublicIP))
{
    $publicIPs = @($FirstPublicIP, $SecondPublicIP)
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -PublicIPs $publicIPs
}

### Start the integration runtime
write-host("##### Starting #####")
Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Force

write-host("##### Completed #####")
write-host("If any cmdlet is unsuccessful, please consider using -Debug option for diagnostics.")
```

## <a name="use-an-azure-resource-manager-template-to-create-an-integration-runtime"></a>Uso de una plantilla de Azure Resource Manager para crear un entorno de ejecución de integración

En esta sección, se usa una plantilla de Azure Resource Manager para crear un entorno de ejecución de integración de Azure SSIS. Aquí se muestra un tutorial de ejemplo:

1. Cree un archivo JSON con la siguiente plantilla de Azure Resource Manager. Reemplace los valores de los corchetes angulares (marcadores de posición) por sus propios valores.

    ```json
    {
        "contentVersion": "1.0.0.0",
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "parameters": {},
        "variables": {},
        "resources": [{
            "name": "<Specify a name for your data factory>",
            "apiVersion": "2018-06-01",
            "type": "Microsoft.DataFactory/factories",
            "location": "East US",
            "properties": {},
            "resources": [{
                "type": "integrationruntimes",
                "name": "<Specify a name for your Azure-SSIS IR>",
                "dependsOn": [ "<The name of the data factory you specified at the beginning>" ],
                "apiVersion": "2018-06-01",
                "properties": {
                    "type": "Managed",
                    "typeProperties": {
                        "computeProperties": {
                            "location": "East US",
                            "nodeSize": "Standard_D8_v3",
                            "numberOfNodes": 1,
                            "maxParallelExecutionsPerNode": 8
                        },
                        "ssisProperties": {
                            "catalogInfo": {
                                "catalogServerEndpoint": "<Azure SQL Database server name>.database.windows.net",
                                "catalogAdminUserName": "<Azure SQL Database server admin username>",
                                "catalogAdminPassword": {
                                    "type": "SecureString",
                                    "value": "<Azure SQL Database server admin password>"
                                },
                                "catalogPricingTier": "Basic"
                            }
                        }
                    }
                }
            }]
        }]
    }
    ```

2. Para implementar la plantilla de Resource Manager, ejecute el comando `New-AzResourceGroupDeployment`, tal y como se muestra en el ejemplo siguiente. En el ejemplo, `ADFTutorialResourceGroup` es el nombre del grupo de recursos. `ADFTutorialARM.json` es el archivo que contiene la definición de JSON de la factoría de datos y del entorno de ejecución de integración de Azure SSIS.

    ```powershell
    New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFTutorialARM.json
    ```

    Este comando crea la factoría de datos y la instancia de Azure-SSIS IR en ella, pero no inicia el entorno de ejecución de integración.

3. Para iniciar Azure-SSIS IR, ejecute el comando `Start-AzDataFactoryV2IntegrationRuntime`:

    ```powershell
    Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName "<Resource Group Name>" `
        -DataFactoryName "<Data Factory Name>" `
        -Name "<Azure SSIS IR Name>" `
        -Force
    ```

> [!NOTE]
> Aparte del tiempo de configuración personalizada, este proceso debería realizarse en 5 minutos. Sin embargo, Azure-SSIS IR puede tardar entre 20 y 30 minutos en unirse a una red virtual.
>
> Si usa SSISDB, el servicio Data Factory se conectará al servidor de bases de datos para prepararlo. También configura los permisos o los valores de la red virtual, si se especifica, y une la instancia de Azure-SSIS IR a la red virtual.
> 
> Al aprovisionar Azure-SSIS IR, también se instalan el acceso redistribuible y el paquete de característica de Azure para SSIS. Estos componentes proporcionan conectividad a archivos de Excel y Access, y a diversos orígenes de datos de Azure, además de a aquellos que ya admiten los componentes integrados. Para obtener más información sobre los componentes integrados o preinstalados, vea el artículo [Componentes integrados y preinstalados en Azure-SSIS Integration Runtime](https://docs.microsoft.com/azure/data-factory/built-in-preinstalled-components-ssis-integration-runtime). Para obtener más información sobre los componentes adicionales que puede instalar, consulte [Personalización de la instalación en una instancia de Azure-SSIS Integration Runtime](https://docs.microsoft.com/azure/data-factory/how-to-configure-azure-ssis-ir-custom-setup).

## <a name="deploy-ssis-packages"></a>Implementación de paquetes de SSIS

Si usa SSISDB, puede implementar los paquetes en esta base de datos y ejecutarlos en Azure-SSIS IR mediante las herramientas de SSDT o de SSMS habilitadas para Azure. Estas herramientas se conectan al servidor de bases de datos mediante su punto de conexión de servidor: 

- En el caso de un servidor de Azure SQL Database, el formato del punto de conexión de servidor es `<server name>.database.windows.net`.
- En el caso de una instancia administrada con punto de conexión privado, el formato del punto de conexión de servidor es `<server name>.<dns prefix>.database.windows.net`.
- En el caso de una instancia administrada con punto de conexión público, el formato del punto de conexión de servidor es `<server name>.public.<dns prefix>.database.windows.net,3342`. 

Si no usa SSISDB, puede implementar los paquetes en sistemas de archivos, en Azure Files o en MSDB hospedados por Azure SQL Managed Instance y ejecutarlos en Azure-SSIS IR mediante las utilidades de la línea de comandos [dtutil](https://docs.microsoft.com/sql/integration-services/dtutil-utility?view=sql-server-2017) y [AzureDTExec](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-azure-enabled-dtexec). 

Para obtener más información, vea [Implementación de proyectos o paquetes en SSIS](https://docs.microsoft.com/sql/integration-services/packages/deploy-integration-services-ssis-projects-and-packages?view=sql-server-ver15).

En ambos casos, también puede ejecutar los paquetes implementados en Azure-SSIS IR mediante la actividad de ejecución de paquetes SSIS de las canalizaciones de Data Factory. Para más información, consulte [Invocación de la ejecución de paquetes SSIS como una actividad de Data Factory de primera clase](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity).

## <a name="next-steps"></a>Pasos siguientes

Consulte otros temas sobre Azure-SSIS IR en esta documentación:

- [Integration Runtime de SSIS de Azure](concepts-integration-runtime.md#azure-ssis-integration-runtime). En este artículo se proporciona información sobre los entornos de ejecución de integración en general, incluido Azure-SSIS IR.
- [Monitor an Azure-SSIS IR](monitor-integration-runtime.md#azure-ssis-integration-runtime) (Supervisión de una instancia de Integration Runtime de SSIS de Azure). En este artículo se muestra cómo recuperar y comprender la información acerca de Azure-SSIS IR.
- [Administración de Integration Runtime de SSIS de Azure](manage-azure-ssis-integration-runtime.md). En este artículo se muestra cómo iniciar, detener o eliminar Azure-SSIS IR. También se muestra cómo escalar horizontalmente la instancia de Azure SSIS IR mediante la adición de más nodos.
- [Implementación, ejecución y supervisión de paquetes de SSIS en Azure](/sql/integration-services/lift-shift/ssis-azure-deploy-run-monitor-tutorial)   
- [Conexión a SSISDB en Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database)
- [Connect to on-premises data sources with Windows authentication](/sql/integration-services/lift-shift/ssis-azure-connect-with-windows-auth) (Conexión a orígenes de datos locales con autenticación de Windows) 
- [Programación de la ejecución de paquetes en Azure](/sql/integration-services/lift-shift/ssis-azure-schedule-packages)