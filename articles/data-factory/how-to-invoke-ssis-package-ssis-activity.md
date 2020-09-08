---
title: Ejecución de un paquete SSIS con la actividad de ejecución de paquetes SSIS
description: En este artículo se describe cómo ejecutar un paquete de SQL Server Integration Services (SSIS) desde una canalización de Azure Data Factory mediante la actividad de ejecución de paquetes SSIS.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: powershell
ms.topic: conceptual
ms.author: sawinark
author: swinarko
ms.reviewer: douglasl
manager: mflasko
ms.custom: seo-lt-2019, devx-track-azurepowershell
ms.date: 07/20/2020
ms.openlocfilehash: 901693c512ddfcf5d3c4dafaec71b1606b5dc5f1
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89077856"
---
# <a name="run-an-ssis-package-with-the-execute-ssis-package-activity-in-azure-data-factory"></a>Ejecución de un paquete de SSIS mediante la actividad Ejecutar paquete SSIS de Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se describe cómo ejecutar un paquete de SQL Server Integration Services (SSIS) desde una canalización de Azure Data Factory mediante la actividad de ejecución de paquetes SSIS. 

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Cree una instancia de Azure-SSIS Integration Runtime (IR) si no tiene ya una. Para ello, siga las instrucciones paso a paso del [Tutorial: Aprovisionamiento de Azure-SSIS IR](tutorial-create-azure-ssis-runtime-portal.md).

## <a name="run-a-package-in-the-azure-portal"></a>Ejecución de un paquete en Azure Portal
En esta sección, usará la interfaz de usuario de Data Factory o la aplicación para crear una canalización de Data Factory con una actividad de ejecución de paquetes SSIS que ejecuta un paquete SSIS.

### <a name="create-a-pipeline-with-an-execute-ssis-package-activity"></a>Creación de una canalización con una actividad Ejecutar paquete de SSIS
En este paso, usa la interfaz de usuario de Data Factory o aplicación para crear una canalización. Agregará una actividad Ejecutar paquete de SSIS a la canalización y la configurará para ejecutar el paquete de SSIS. 

1. En la información general de Data Factory o en la página principal de Azure Portal, seleccione el icono **Crear y supervisar** para iniciar la interfaz de usuario de Data Factory o la aplicación en una pestaña independiente. 

   ![Página principal de Data Factory](./media/how-to-invoke-ssis-package-stored-procedure-activity/data-factory-home-page.png)

   En la página **Let's get started** (Introducción) seleccione **Create pipeline** (Crear canalización). 

   ![Página de introducción](./media/how-to-invoke-ssis-package-stored-procedure-activity/get-started-page.png)

1. En el cuadro de herramientas **Activities** (Actividades), expanda **General**. Después, arrastre la actividad **Ejecutar paquete SSIS** a la superficie del diseñador 

   ![Arrastrar la actividad Execute SSIS Package (Ejecutar paquete de SSIS) a la superficie del diseñador](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-designer.png) 

   Seleccione el objeto Execute SSIS Package activity (Ejecutar una actividad de paquete SSISS) para configurar sus pestañas **General**, **Settings** (Configuración), **SSIS Parameters** (Parámetros de SSIS), **Connection Managers** (Administradores de conexiones) y **Property Overrides** (Invalidaciones de propiedad).

#### <a name="general-tab"></a>Pestaña General

En la pestaña **General** de la actividad Execute SSIS Package (Ejecutar paquete SSIS), realice los pasos siguientes.

![Establecimiento de propiedades en la pestaña General](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-general.png)

   1. En **Name** (Nombre), escriba el nombre de la actividad Execute SSIS Package (Ejecutar paquete de SSIS).

   1. En **Description** (Descripción), escriba la descripción de la actividad Execute SSIS Package (Ejecutar paquete de SSIS).

   1. En **Timeout** (Tiempo de espera), escriba la cantidad máxima de tiempo que se puede ejecutar la actividad Execute SSIS Package (Ejecutar paquete de SSIS). El valor predeterminado es 7 días y el formato es D.HH:MM:SS.

   1. En **Retry** (Reintentar), escriba el número máximo de reintentos para la actividad Execute SSIS Package (Ejecutar paquete de SSIS).

   1. En **Retry interval** (Intervalo de reintento), escriba el número de segundos entre cada reintento de la actividad Execute SSIS Package (Ejecutar paquete de SSIS). El valor predeterminado es 30 segundos.

   1. Active la casilla **Secure output** (Salida segura) para elegir si quiere excluir del registro la salida de la actividad Execute SSIS Package (Ejecutar paquete de SSIS).

   1. Active la casilla **Secure input** (Entrada segura) para elegir si quiere excluir del registro la entrada de la actividad Execute SSIS Package (Ejecutar paquete de SSIS).

#### <a name="settings-tab"></a>Pestaña Settings

En la pestaña **Settings** (Configuración) de la actividad Execute SSIS Package (Ejecutar paquete de SSIS), haga lo siguiente.

![Establecer propiedades de la pestaña Settings (Configuración): Automatic (Automática)](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-settings.png)

   1. En **Azure-SSIS IR**, seleccione la instancia de Azure-SSIS IR designada para ejecutar la actividad Execute SSIS Package (Ejecutar paquete de SSIS).

   1. En **Description** (Descripción), escriba la descripción de la actividad Execute SSIS Package (Ejecutar paquete de SSIS).

   1. Active la casilla **Windows authentication** (Autenticación de Windows) para elegir si quiere usar la autenticación de Windows para acceder a los almacenes de datos, por ejemplo, a los servidores o a los recursos compartidos de archivos locales de SQL o a Azure Files.
   
      Si activa esta casilla, escriba los valores de las credenciales de ejecución del paquete en los cuadros **Domain** (Dominio), **Username** (Nombre de usuario) y **Password** (Contraseña). Por ejemplo, para acceder a Azure Files, el dominio es `Azure`, el nombre de usuario es `<storage account name>` y la contraseña es `<storage account key>`.

      También puede usar como valores los secretos almacenados en la instancia de Azure Key Vault. Para hacerlo, active la casilla **AZURE KEY VAULT** que aparece al lado. Seleccione o edite el servicio vinculado del almacén de claves existente o cree uno nuevo. Luego, seleccione el nombre del secreto y la versión del valor. Al crear o editar el servicio vinculado del almacén de claves, puede seleccionar o editar el almacén de claves existente o crear uno nuevo. Asegúrese de conceder a Data Factory el acceso de la identidad administrada a su almacén de claves si aún no lo ha hecho. Los secretos también se pueden escribir directamente en el siguiente formato: `<key vault linked service name>/<secret name>/<secret version>`.
      
   1. Si el paquete necesita el entorno de ejecución de 32 bits para funcionar, active la casilla **32-Bit runtime** (Entorno de ejecución de 32 bits).

   1. En **Package location** (Ubicación del paquete), seleccione **SSISDB**, **File System (Package)** (Sistema de archivos [paquete]), **File System (Project)** (Sistema de archivos [proyecto]), **Embedded package** (Paquete insertado) o **Package store** (Almacén de paquetes). 

##### <a name="package-location-ssisdb"></a>Ubicación del paquete: SSISDB

La ubicación del paquete **SSISDB** se selecciona automáticamente si Azure-SSIS IR se aprovisionó con un catálogo de SSIS (SSISDB) hospedado en el servidor de base de datos o la instancia administrada de Azure SQL; también la puede seleccionar el usuario. Si está seleccionada, siga estos pasos.

   1. Si la instancia de Azure-SSIS IR está en ejecución y la casilla **Manual entries** (Entradas manuales) está desactivada, puede examinar y seleccionar sus carpetas, proyectos, paquetes y entornos existentes de SSISDB. Seleccione **Refresh** (Actualizar) para capturar las carpetas, proyectos, paquetes y entornos recién agregados de SSISDB, de forma que estén disponibles para su examen y selección. Para examinar y seleccionar los entornos para la ejecución de paquetes, debe configurar los proyectos antes de agregar esos entornos como referencias desde las mismas carpetas en SSISDB. Para más información, consulte [Creación y asignación de entornos SSIS](https://docs.microsoft.com/sql/integration-services/packages/deploy-integration-services-ssis-projects-and-packages).

   1. En **Logging level** (Nivel de registro), seleccione un ámbito predefinido de registro para la ejecución de su paquete. Active la casilla **Customized** (Personalizado), si quiere escribir en su lugar un nombre de registro personalizado. 

   1. Si la instancia de Azure-SSIS IR no se está ejecutando o la casilla **Manual entries** (Entradas manuales) está activada, escriba las rutas de acceso del paquete y el entorno de SSISDB directamente en los siguientes formatos: `<folder name>/<project name>/<package name>.dtsx` y `<folder name>/<environment name>`.

      ![Establecer las propiedades en la pestaña Settings (Configuración): manual](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-settings2.png)

##### <a name="package-location-file-system-package"></a>Ubicación del paquete: File System (Package) (Sistema de archivos [paquete])

La ubicación del paquete **File System (Package)** (Sistema de archivos [paquete]) se selecciona automáticamente si Azure-SSIS IR se aprovisionó sin SSISDB; también la puede seleccionar el usuario. Si está seleccionada, siga estos pasos.

![Establecimiento de las propiedades en la pestaña Properties (Propiedades): File System (Package) (Sistema de archivos [paquete])](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-settings3.png)
   
   1. Especifique el paquete que se va a ejecutar, para lo que debe proporcionar una ruta de acceso UNC (Convención de nomenclatura universal) al archivo de paquete (con `.dtsx`) en el cuadro **Package path** (Ruta de acceso del paquete). Para examinar y seleccionar el paquete, elija **Browse file storage** (Examinar almacenamiento de archivos) o escriba su ruta de acceso manualmente. Por ejemplo, si almacena el paquete en Azure Files, su ruta de acceso es `\\<storage account name>.file.core.windows.net\<file share name>\<package name>.dtsx`. 
   
   1. Si configura el paquete en un archivo independiente, también deberá proporcionar una ruta de acceso UNC para el archivo de configuración (`.dtsConfig`) en el cuadro **Configuration path** (Ruta de configuración). Para examinar y seleccionar la configuración, elija **Browse file storage** (Examinar almacenamiento de archivos) o escriba su ruta de acceso manualmente. Por ejemplo, si almacena la configuración en Azure Files, su ruta de acceso es `\\<storage account name>.file.core.windows.net\<file share name>\<configuration name>.dtsConfig`.

   1. Especifique las credenciales de acceso al paquete y a los archivos de configuración. Si anteriormente ha escrito los valores de las credenciales de ejecución del paquete (para la **autenticación de Windows**), puede volver a usarlas si selecciona la casilla **Same as package execution credentials** (Igual que las credenciales de ejecución del paquete). En caso contrario, escriba los valores de las credenciales de acceso del paquete en los cuadros **Domain** (Dominio), **Username** (Nombre de usuario) y **Password** (Contraseña). Por ejemplo, si almacena el paquete y la configuración en Azure Files, el dominio es `Azure`, el nombre de usuario es `<storage account name>` y la contraseña es `<storage account key>`. 

      También puede usar como valores los secretos almacenados en la instancia de Azure Key Vault. Para hacerlo, active la casilla **AZURE KEY VAULT** que aparece al lado. Seleccione o edite el servicio vinculado del almacén de claves existente o cree uno nuevo. Luego, seleccione el nombre del secreto y la versión del valor. Al crear o editar el servicio vinculado del almacén de claves, puede seleccionar o editar el almacén de claves existente o crear uno nuevo. Asegúrese de conceder a Data Factory el acceso de la identidad administrada a su almacén de claves si aún no lo ha hecho. Los secretos también se pueden escribir directamente en el siguiente formato: `<key vault linked service name>/<secret name>/<secret version>`. 

      Estas credenciales también se usan para acceder a los paquetes secundarios de la tarea de ejecución del paquete a la que se hace referencia mediante su propia ruta de acceso y otras configuraciones especificadas en los paquetes. 

   1. Si ha usado el nivel de protección **EncryptAllWithPassword** o **EncryptSensitiveWithPassword** al crear el paquete mediante SQL Server Data Tools (SSDT), escriba el valor de la contraseña en el cuadro **Encryption password** (Contraseña de cifrado). También puede usar como valor un secreto almacenado en su instancia de Azure Key Vault (consulte anteriormente).
      
      Si ha usado el nivel de protección **EncryptSensitiveWithUserKey**, vuelva a escribir los valores confidenciales en los archivos de configuración en las pestañas **SSIS Parameters** (Parámetros SSIS), **Connection Managers** (Administradores de conexiones) o **Property Overrides** (Invalidaciones de propiedad) (consulte a continuación).
      
      Si ha usado el nivel de protección **EncryptAllWithUserKey** , no se admite. Debe volver a configurar el paquete para usar otro nivel de protección mediante SSDT o la utilidad de línea de comandos `dtutil`. 

   1. En **Logging level** (Nivel de registro), seleccione un ámbito predefinido de registro para la ejecución de su paquete. Active la casilla **Customized** (Personalizado), si quiere escribir en su lugar un nombre de registro personalizado. 
   
   1. Si quiere registrar las ejecuciones del paquete mediante el uso de los proveedores de registro estándar que se pueden especificar en el paquete, especifique la carpeta de registro mediante el suministro de su ruta de acceso UNC en el cuadro **Logging path** (Ruta de acceso a registros). Para examinar y seleccionar la carpeta de registro, elija **Browse file storage** (Examinar almacenamiento de archivos) o escriba su ruta de acceso manualmente. Por ejemplo, si almacena los registros en Azure Files, su ruta de acceso al registro es `\\<storage account name>.file.core.windows.net\<file share name>\<log folder name>`. En esta ruta de acceso se crea una subcarpeta para cada ejecución de paquete, designada tras un identificador de ejecución de la actividad Execute SSIS Package (Ejecutar paquete de SSIS), y en la que se generan archivos de registro cada cinco minutos. 
   
   1. Especifique las credenciales para acceder a la carpeta de registro. Si previamente ha escrito los valores de las credenciales de acceso del paquete (consulte más arriba), puede volver a usarlas mediante la activación de la casilla **Same as package access credentials** (Igual que las credenciales de acceso al paquete). En caso contrario, escriba los valores de las credenciales de acceso del registro en los cuadros **Domain** (Dominio), **Username** (Nombre de usuario) y **Password** (Contraseña). Por ejemplo, si almacena los registros en Azure Files, el dominio es `Azure`, el nombre de usuario es `<storage account name>` y la contraseña es `<storage account key>`. También puede usar como valores los secretos almacenados en Azure Key Vault.
   
Para todas las rutas de acceso UNC mencionadas anteriormente, el nombre de archivo completo debe tener menos de 260 caracteres. El nombre del directorio debe tener menos de 248 caracteres.

##### <a name="package-location-file-system-project"></a>Ubicación del paquete: File System (Project) (Sistema de archivos [proyecto])

Si selecciona **File System (Project)** (Sistema de archivos [proyecto]) como ubicación del paquete, realice los pasos siguientes.

![Establecimiento de las propiedades en la pestaña Settings (Configuración): File System (Project) (Sistema de archivos [proyecto])](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-settings4.png)

   1. Especifique el paquete que quiere ejecutar; para ello, proporcione una ruta de acceso UNC al archivo de proyecto (con `.ispac`) en el cuadro **Project path** (Ruta de acceso del proyecto) y un archivo de paquete (con `.dtsx`) del proyecto en el cuadro **Package name** (Nombre del paquete). Para examinar y seleccionar el proyecto, elija **Browse file storage** (Examinar almacenamiento de archivos) o escriba su ruta de acceso manualmente. Por ejemplo, si almacena el proyecto en Azure Files, su ruta de acceso será `\\<storage account name>.file.core.windows.net\<file share name>\<project name>.ispac`.

   1. Especifique las credenciales para acceder a los archivos del proyecto y del paquete. Si anteriormente ha escrito los valores de las credenciales de ejecución del paquete (para la **autenticación de Windows**), puede volver a usarlas si selecciona la casilla **Same as package execution credentials** (Igual que las credenciales de ejecución del paquete). En caso contrario, escriba los valores de las credenciales de acceso del paquete en los cuadros **Domain** (Dominio), **Username** (Nombre de usuario) y **Password** (Contraseña). Por ejemplo, si almacena su proyecto y paquete en Azure Files, el dominio es `Azure`, el nombre de usuario es `<storage account name>` y la contraseña es `<storage account key>`. 

      También puede usar como valores los secretos almacenados en la instancia de Azure Key Vault. Para hacerlo, active la casilla **AZURE KEY VAULT** que aparece al lado. Seleccione o edite el servicio vinculado del almacén de claves existente o cree uno nuevo. Luego, seleccione el nombre del secreto y la versión del valor. Al crear o editar el servicio vinculado del almacén de claves, puede seleccionar o editar el almacén de claves existente o crear uno nuevo. Asegúrese de conceder a Data Factory el acceso de la identidad administrada a su almacén de claves si aún no lo ha hecho. Los secretos también se pueden escribir directamente en el siguiente formato: `<key vault linked service name>/<secret name>/<secret version>`. 

      Estas credenciales también se usan acceder a los paquetes secundarios de la tarea de ejecución de paquetes a la que se hace referencia desde el mismo proyecto. 

   1. Si ha usado el nivel de protección **EncryptAllWithPassword** o **EncryptSensitiveWithPassword** al crear el paquete a través de SSDT, escriba el valor de la contraseña en el cuadro **Encryption password** (Contraseña de cifrado). También puede usar como valor un secreto almacenado en su instancia de Azure Key Vault (consulte anteriormente).
      
      Si ha usado el nivel de protección **EncryptSensitiveWithUserKey**, vuelva a escribir los valores confidenciales en las pestañas **SSIS Parameters** (Parámetros SSIS), **Connection Managers** (Administradores de conexiones) o **Property Overrides** (Invalidaciones de propiedad).
      
      Si ha usado el nivel de protección **EncryptAllWithUserKey** , no se admite. Debe volver a configurar el paquete para usar otro nivel de protección mediante SSDT o la utilidad de línea de comandos `dtutil`. 

   1. En **Logging level** (Nivel de registro), seleccione un ámbito predefinido de registro para la ejecución de su paquete. Active la casilla **Customized** (Personalizado), si quiere escribir en su lugar un nombre de registro personalizado. 
   
   1. Si quiere registrar las ejecuciones del paquete mediante el uso de los proveedores de registro estándar que se pueden especificar en el paquete, especifique la carpeta de registro mediante el suministro de su ruta de acceso UNC en el cuadro **Logging path** (Ruta de acceso a registros). Para examinar y seleccionar la carpeta de registro, elija **Browse file storage** (Examinar almacenamiento de archivos) o escriba su ruta de acceso manualmente. Por ejemplo, si almacena los registros en Azure Files, su ruta de acceso al registro es `\\<storage account name>.file.core.windows.net\<file share name>\<log folder name>`. En esta ruta de acceso se crea una subcarpeta para cada ejecución de paquete, designada tras un identificador de ejecución de la actividad Execute SSIS Package (Ejecutar paquete de SSIS), y en la que se generan archivos de registro cada cinco minutos. 
   
   1. Especifique las credenciales para acceder a la carpeta de registro. Si previamente ha escrito los valores de las credenciales de acceso del paquete (consulte más arriba), puede volver a usarlas mediante la activación de la casilla **Same as package access credentials** (Igual que las credenciales de acceso al paquete). En caso contrario, escriba los valores de las credenciales de acceso del registro en los cuadros **Domain** (Dominio), **Username** (Nombre de usuario) y **Password** (Contraseña). Por ejemplo, si almacena los registros en Azure Files, el dominio es `Azure`, el nombre de usuario es `<storage account name>` y la contraseña es `<storage account key>`. También puede usar como valores los secretos almacenados en Azure Key Vault.
   
Para todas las rutas de acceso UNC mencionadas anteriormente, el nombre de archivo completo debe tener menos de 260 caracteres. El nombre del directorio debe tener menos de 248 caracteres.

##### <a name="package-location-embedded-package"></a>Ubicación del paquete: Embedded package (Paquete insertado)

Si selecciona **Embedded package** (Paquete insertado) como ubicación del paquete, realice los pasos siguientes.

![Establecimiento de propiedades en la pestaña Configuración: Embedded package (Paquete insertado)](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-settings5.png)

   1. Arrastre y coloque el archivo de paquete (con `.dtsx`) o **cárguelo** desde una carpeta de archivos en el cuadro proporcionado. El paquete se comprimirá e insertará automáticamente en la carga de la actividad. Una vez insertado, puede **descargar** el paquete más adelante para su edición. También puede **parametrizar** el paquete insertado si lo asigna a un parámetro de canalización que se puede usar en varias actividades, por lo que se optimiza el tamaño de la carga de la canalización. Actualmente no se admite la inserción de archivos de proyecto (con `.ispac`), por lo que no se pueden usar parámetros de SSIS ni administradores de conexión con el ámbito de nivel de proyecto en los paquetes insertados.
   
   1. Si el paquete insertado no está cifrado por completo y se detecta el uso de la tarea Ejecutar paquete (EPT) en su interior, la casilla **Tarea Ejecutar paquete** se seleccionará automáticamente y se agregarán automáticamente los paquetes secundarios a los que hace referencia la ruta de acceso del sistema de archivos, así que también puede insertarlos.
   
      Si no se puede detectar el uso de EPT, tendrá que seleccionar la casilla **Tarea Ejecutar paquete** de forma manual y agregar los paquetes secundarios a los que hacer referencia la ruta de acceso del sistema de archivos de uno en uno, para poder insertarlos también. Si los paquetes secundarios se almacenan en una base de datos de SQL Server (MSDB), no puede insertarlos, por lo que debe asegurarse de que Azure-SSIS IR pueda tener acceso a MSDB para capturarlos con sus referencias de SQL Server. Actualmente no se admite la inserción de archivos de proyecto (con `.ispac`), por lo que no se pueden usar referencias basadas en proyectos para los paquetes secundarios.
   
   1. Si ha usado el nivel de protección **EncryptAllWithPassword** o **EncryptSensitiveWithPassword** al crear el paquete a través de SSDT, escriba el valor de la contraseña en el cuadro **Encryption password** (Contraseña de cifrado). 
   
      También puede usar como valor un secreto almacenado en Azure Key Vault. Para ello, active la casilla **AZURE KEY VAULT** que aparece al lado. Seleccione o edite el servicio vinculado del almacén de claves existente o cree uno nuevo. Luego, seleccione el nombre del secreto y la versión del valor. Al crear o editar el servicio vinculado del almacén de claves, puede seleccionar o editar el almacén de claves existente o crear uno nuevo. Asegúrese de conceder a Data Factory el acceso de la identidad administrada a su almacén de claves si aún no lo ha hecho. Los secretos también se pueden escribir directamente en el siguiente formato: `<key vault linked service name>/<secret name>/<secret version>`.
      
      Si ha usado el nivel de protección **EncryptSensitiveWithUserKey**, vuelva a escribir los valores confidenciales en los archivos de configuración en las pestañas **SSIS Parameters** (Parámetros SSIS), **Connection Managers** (Administradores de conexiones) o **Property Overrides** (Invalidaciones de propiedad) (consulte a continuación).
      
      Si ha usado el nivel de protección **EncryptAllWithUserKey** , no se admite. Debe volver a configurar el paquete para usar otro nivel de protección mediante SSDT o la utilidad de línea de comandos `dtutil`.

   1. En **Logging level** (Nivel de registro), seleccione un ámbito predefinido de registro para la ejecución de su paquete. Active la casilla **Customized** (Personalizado), si quiere escribir en su lugar un nombre de registro personalizado. 
   
   1. Si quiere registrar las ejecuciones del paquete mediante el uso de los proveedores de registro estándar que se pueden especificar en el paquete, especifique la carpeta de registro mediante el suministro de su ruta de acceso UNC en el cuadro **Logging path** (Ruta de acceso a registros). Para examinar y seleccionar la carpeta de registro, elija **Browse file storage** (Examinar almacenamiento de archivos) o escriba su ruta de acceso manualmente. Por ejemplo, si almacena los registros en Azure Files, su ruta de acceso al registro es `\\<storage account name>.file.core.windows.net\<file share name>\<log folder name>`. En esta ruta de acceso se crea una subcarpeta para cada ejecución de paquete, designada tras un identificador de ejecución de la actividad Execute SSIS Package (Ejecutar paquete de SSIS), y en la que se generan archivos de registro cada cinco minutos. 
   
   1. Especifique las credenciales de acceso a la carpeta de registro. para ello, escriba sus valores en los cuadros **Domain** (Dominio), **Username** (Nombre de usuario) y **Password** (Contraseña). Por ejemplo, si almacena los registros en Azure Files, el dominio es `Azure`, el nombre de usuario es `<storage account name>` y la contraseña es `<storage account key>`. También puede usar como valores los secretos almacenados en Azure Key Vault.
   
Para todas las rutas de acceso UNC mencionadas anteriormente, el nombre de archivo completo debe tener menos de 260 caracteres. El nombre del directorio debe tener menos de 248 caracteres.

##### <a name="package-location-package-store"></a>Ubicación del paquete: Package store (Almacén de paquetes)

Si selecciona **Package store** (Almacén de paquetes) como ubicación del paquete, realice los pasos siguientes.

![Establecimiento de las propiedades de la pestaña Settings (Configuración): Package store (Almacén de paquetes)](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-settings6.png)
   
   1. En **Package store name** (Nombre del almacén de paquetes), seleccione un almacén de paquetes existente que esté asociado a su instancia de Azure-SSIS IR.

   1. Especifique el paquete que se va a ejecutar; para ello, proporcione la ruta de acceso (sin `.dtsx`) del almacén de paquetes seleccionado en el cuadro **Package path** (Ruta de acceso del paquete). Si el almacén de paquetes seleccionado está encima del sistema de archivos o de Azure Files, puede examinar y seleccionar el paquete con **Browse file storage** (Examinar almacenamiento de archivos); en caso contrario, puede escribir su ruta de acceso en el formato `<folder name>\<package name>`. También puede importar paquetes nuevos en el almacén de paquetes seleccionado a través de SQL Server Management Studio (SSMS) de forma similar al [almacén de paquetes SSIS heredado](https://docs.microsoft.com/sql/integration-services/service/package-management-ssis-service?view=sql-server-2017). Para más información, consulte el artículo [Administración de paquetes SSIS con almacenes de paquetes de Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/azure-ssis-integration-runtime-package-store).

   1. Si configura el paquete en un archivo aparte, también deberá proporcionar una ruta de acceso UNC al archivo de configuración (`.dtsConfig`) en el cuadro **Configuration path** (Ruta de configuración). Para examinar y seleccionar la configuración, elija **Browse file storage** (Examinar almacenamiento de archivos) o escriba su ruta de acceso manualmente. Por ejemplo, si almacena la configuración en Azure Files, su ruta de acceso es `\\<storage account name>.file.core.windows.net\<file share name>\<configuration name>.dtsConfig`.

   1. Active la casilla **Configuration access credentials** (Credenciales de acceso de configuración) para elegir si quiere especificar las credenciales de acceso al archivo de configuración por separado. Esto es necesario cuando el almacén de paquetes seleccionado se encuentra encima de la base de datos de SQL Server (MSDB) hospedada en Instancia administrada de Azure SQL o no almacena tampoco el archivo de configuración.
   
      Si anteriormente ha escrito los valores de las credenciales de ejecución del paquete (para la **autenticación de Windows**), puede volver a usarlas si selecciona la casilla **Same as package execution credentials** (Igual que las credenciales de ejecución del paquete). En caso contrario, escriba los valores de las credenciales de acceso de configuración en los cuadros **Domain** (Dominio), **Username** (Nombre de usuario) y **Password** (Contraseña). Por ejemplo, si almacena su configuración en Azure Files, el dominio es `Azure`, el nombre de usuario es `<storage account name>` y la contraseña es `<storage account key>`. 

      También puede usar como valores los secretos almacenados en la instancia de Azure Key Vault. Para hacerlo, active la casilla **AZURE KEY VAULT** que aparece al lado. Seleccione o edite el servicio vinculado del almacén de claves existente o cree uno nuevo. Luego, seleccione el nombre del secreto y la versión del valor. Al crear o editar el servicio vinculado del almacén de claves, puede seleccionar o editar el almacén de claves existente o crear uno nuevo. Asegúrese de conceder a Data Factory el acceso de la identidad administrada a su almacén de claves si aún no lo ha hecho. Los secretos también se pueden escribir directamente en el siguiente formato: `<key vault linked service name>/<secret name>/<secret version>`.

   1. Si ha usado el nivel de protección **EncryptAllWithPassword** o **EncryptSensitiveWithPassword** al crear el paquete a través de SSDT, escriba el valor de la contraseña en el cuadro **Encryption password** (Contraseña de cifrado). También puede usar como valor un secreto almacenado en su instancia de Azure Key Vault (consulte anteriormente).
      
      Si ha usado el nivel de protección **EncryptSensitiveWithUserKey**, vuelva a escribir los valores confidenciales en los archivos de configuración en las pestañas **SSIS Parameters** (Parámetros SSIS), **Connection Managers** (Administradores de conexiones) o **Property Overrides** (Invalidaciones de propiedad) (consulte a continuación).
      
      Si ha usado el nivel de protección **EncryptAllWithUserKey** , no se admite. Debe volver a configurar el paquete para usar otro nivel de protección mediante SSDT o la utilidad de línea de comandos `dtutil`. 

   1. En **Logging level** (Nivel de registro), seleccione un ámbito predefinido de registro para la ejecución de su paquete. Active la casilla **Customized** (Personalizado), si quiere escribir en su lugar un nombre de registro personalizado. 
   
   1. Si quiere registrar las ejecuciones del paquete mediante el uso de los proveedores de registro estándar que se pueden especificar en el paquete, especifique la carpeta de registro mediante el suministro de su ruta de acceso UNC en el cuadro **Logging path** (Ruta de acceso a registros). Para examinar y seleccionar la carpeta de registro, elija **Browse file storage** (Examinar almacenamiento de archivos) o escriba su ruta de acceso manualmente. Por ejemplo, si almacena los registros en Azure Files, su ruta de acceso al registro es `\\<storage account name>.file.core.windows.net\<file share name>\<log folder name>`. En esta ruta de acceso se crea una subcarpeta para cada ejecución de paquete, designada tras un identificador de ejecución de la actividad Execute SSIS Package (Ejecutar paquete de SSIS), y en la que se generan archivos de registro cada cinco minutos. 
   
   1. Especifique las credenciales de acceso a la carpeta de registro. para ello, escriba sus valores en los cuadros **Domain** (Dominio), **Username** (Nombre de usuario) y **Password** (Contraseña). Por ejemplo, si almacena los registros en Azure Files, el dominio es `Azure`, el nombre de usuario es `<storage account name>` y la contraseña es `<storage account key>`. También puede usar como valores los secretos almacenados en Azure Key Vault.
   
Para todas las rutas de acceso UNC mencionadas anteriormente, el nombre de archivo completo debe tener menos de 260 caracteres. El nombre del directorio debe tener menos de 248 caracteres.

#### <a name="ssis-parameters-tab"></a>Pestaña SSIS Parameters (Parámetros de SSIS)

En la pestaña **SSIS Parameters** (Parámetros de SSIS) de la actividad Execute SSIS Package (Ejecutar paquete de SSIS), haga lo siguiente.

![Establecer propiedades en la pestaña SSIS Parameters (Parámetros de SSIS)](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-ssis-parameters.png)

   1. Si la instancia de Azure-SSIS IR está en ejecución, se selecciona **SSISDB** como la ubicación del paquete y la casilla **Manual entries** (Entradas manuales) de la pestaña **Settings** (Configuración) está desactivada, se muestran los parámetros de SSIS existentes en el proyecto o paquete seleccionados de SSISDB para que les asigne valores. De lo contrario, puede escribirlos uno por uno para asignarles valores manualmente. Asegúrese de que existen y que se han escrito correctamente para que la ejecución del paquete se realice satisfactoriamente. 
   
   1. Si ha usado el nivel de protección **EncryptSensitiveWithUserKey** al crear el paquete mediante SSDT y se ha seleccionado **File System (Package)** (Sistema de archivos [paquete]), **File System (Project)** (Sistema de archivos [proyecto]), **Embedded package** (Paquete insertado) o **Package store** (Almacén de paquetes) como ubicación del paquete, también debe volver a escribir las propiedades confidenciales del paquete para asignarles valores en esta pestaña. 
   
Al asignar valores a los parámetros, puede agregar contenido dinámico mediante expresiones, funciones, variables del sistema de Data Factory y parámetros o variables de canalización de Data Factory.

También puede usar como valores los secretos almacenados en la instancia de Azure Key Vault. Para hacerlo, active la casilla **AZURE KEY VAULT** que aparece al lado. Seleccione o edite el servicio vinculado del almacén de claves existente o cree uno nuevo. Luego, seleccione el nombre del secreto y la versión del valor. Al crear o editar el servicio vinculado del almacén de claves, puede seleccionar o editar el almacén de claves existente o crear uno nuevo. Asegúrese de conceder a Data Factory el acceso de la identidad administrada a su almacén de claves si aún no lo ha hecho. Los secretos también se pueden escribir directamente en el siguiente formato: `<key vault linked service name>/<secret name>/<secret version>`. 

#### <a name="connection-managers-tab"></a>Pestaña Administradores de conexiones

En la pestaña **Connection Managers** (Administradores de conexones) de la actividad Execute SSIS Package (Ejecutar paquete de SSIS), haga lo siguiente.

![Establecer propiedades en la pestaña Connection Managers (Administradores de conexiones)](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-connection-managers.png)

   1. Si la instancia de Azure-SSIS IR está en ejecución, se selecciona **SSISDB** como ubicación del paquete y la casilla **Manual entries** (Entradas manuales) de la pestaña **Settings** (Configuración) está desactivada, se muestran los administradores de conexiones existentes del proyecto y del paquete seleccionados en SSISDB para que se asignen valores a sus propiedades. De lo contrario, puede escribirlos uno por uno para asignar valores a sus propiedades manualmente. Asegúrese de que existen y que se han escrito correctamente para que la ejecución del paquete se realice satisfactoriamente. 
   
      Puede obtener los nombres correctos de**SCOPE**, **NAME** y **PROPERTY** de cualquier administrador de conexiones; para ello, abra el paquete que lo contiene en SSDT. Una vez abierto el paquete, seleccione el administrador de conexiones correspondiente para mostrar los nombres y valores de todas sus propiedades en la ventana **Propiedades** de SSDT. Con esta información, puede invalidar los valores de las propiedades del administrador de conexiones en tiempo de ejecución. 

      ![Obtención de las propiedades del administrador de conexiones de SSDT](media/how-to-invoke-ssis-package-ssis-activity/ssdt-connection-manager-properties.png)

      Por ejemplo, sin modificar el paquete original en SSDT, puede convertir sus flujos de datos del entorno local al entorno local que se ejecutan en SQL Server en flujos de datos del entorno local a la nube que se ejecutan en SSIS IR en ADF mediante la invalidación de los valores de las propiedades **ConnectByProxy**, **ConnectionString** y **ConnectUsingManagedIdentity** en los administradores de conexiones existentes en tiempo de ejecución.
      
      Estas invalidaciones en tiempo de ejecución pueden habilitar IR autohospedado (SHIR) como proxy para SSIS IR al acceder a los datos locales, consulte [Configuración de IR autohospedado como proxy para Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/self-hosted-integration-runtime-proxy-ssis), y las conexiones de Instancia administrada/Azure SQL Database con el controlador MSOLEDBSQL más reciente que, a su vez, habilita la autenticación de Azure Active Directory (AAD) con la identidad administrada de ADF, consulte [Configuración de la autenticación de AAD con la identidad administrada de ADF para conexiones OLEDB](https://docs.microsoft.com/sql/integration-services/connection-manager/ole-db-connection-manager?view=sql-server-ver15#managed-identities-for-azure-resources-authentication).

      ![Definición de propiedades desde SSDT en la pestaña Administradores de conexiones](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-connection-managers2.png)
   
   1. Si ha usado el nivel de protección **EncryptSensitiveWithUserKey** al crear el paquete mediante SSDT y se ha seleccionado **File System (Package)** (Sistema de archivos [paquete]), **File System (Project)** (Sistema de archivos [proyecto]), **Embedded package** (Paquete insertado) o **Package store** (Almacén de paquetes) como ubicación del paquete, también debe volver a escribir las propiedades confidenciales del administrador de conexiones para asignarles valores en esta pestaña. 

Al asignar valores a las propiedades del administrador de conexiones, puede agregar contenido dinámico mediante expresiones, funciones, variables del sistema de Data Factory y parámetros o variables de canalización de Data Factory. 

También puede usar como valores los secretos almacenados en la instancia de Azure Key Vault. Para hacerlo, active la casilla **AZURE KEY VAULT** que aparece al lado. Seleccione o edite el servicio vinculado del almacén de claves existente o cree uno nuevo. Luego, seleccione el nombre del secreto y la versión del valor. Al crear o editar el servicio vinculado del almacén de claves, puede seleccionar o editar el almacén de claves existente o crear uno nuevo. Asegúrese de conceder a Data Factory el acceso de la identidad administrada a su almacén de claves si aún no lo ha hecho. Los secretos también se pueden escribir directamente en el siguiente formato: `<key vault linked service name>/<secret name>/<secret version>`. 

#### <a name="property-overrides-tab"></a>Pestaña Property Overrides (Invalidaciones de propiedad)

En la pestaña **Property Overrides** (Invalidaciones de propiedad) de la actividad Execute SSIS Package (Ejecutar paquete de SSIS), siga estos pasos.

![Establecer propiedades en la pestaña Property Overrides (Reemplazos de propiedad)](media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-property-overrides.png)

   1. Escriba las rutas de acceso de las propiedades existentes en el paquete seleccionado una por una para asignarles valores manualmente. Asegúrese de que existen y que se han escrito correctamente para que la ejecución del paquete se realice satisfactoriamente. Por ejemplo, para invalidar el valor de la variable de usuario, escriba su ruta de acceso en el formato `\Package.Variables[User::<variable name>].Value`. 

      Para obtener la **RUTA DE ACCESO DE LA PROPIEDAD** correcta para cualquier propiedad del paquete, puede abrir el paquete que la contiene en SSDT. Una vez abierto el paquete, seleccione su flujo de control y la propiedad **Configurations** en la ventana **Propiedades** de SSDT. Después, seleccione el botón de puntos suspensivos ( **…** ) situado junto a su propiedad **Configurations** para abrir el **Organizador de configuraciones de paquetes** que se usa normalmente para [crear configuraciones de paquetes en el modelo de implementación de paquetes](https://docs.microsoft.com/sql/integration-services/packages/legacy-package-deployment-ssis#create-package-configurations). 

      ![Obtención de las propiedades de paquete desde la propiedad Configurations de SSDT](media/how-to-invoke-ssis-package-ssis-activity/ssdt-package-properties.png)

      En el **Organizador de configuraciones de paquetes**, active la casilla **Habilitar configuraciones de paquetes** y el botón **Agregar…** para abrir el **Asistente para la configuración de paquetes**. 
      
      En el **Asistente para la configuración de paquetes**, seleccione el elemento **Archivo de configuración XML** en el menú desplegable **Tipo de configuración** y el botón **Especificar valores de configuración  directamente**, escriba el nombre del archivo de configuración y seleccione el botón **Siguiente >** . 

      ![Obtención de las propiedades de paquete desde el Organizador de configuraciones de SSDT](media/how-to-invoke-ssis-package-ssis-activity/ssdt-package-properties2.png)

      Por último, seleccione las propiedades del paquete cuya ruta de acceso desea y el botón **Siguiente >** .  Ahora puede ver, copiar & pegar las rutas de acceso de las propiedades del paquete que desee y guardarlas en el archivo de configuración. Con esta información, puede invalidar los valores de las propiedades del paquete en tiempo de ejecución. 

      ![Obtención de las propiedades de paquete desde el Asistente para la configuración de SSDT](media/how-to-invoke-ssis-package-ssis-activity/ssdt-package-properties3.png)
   
   1. Si ha usado el nivel de protección **EncryptSensitiveWithUserKey** al crear el paquete mediante SSDT y se ha seleccionado **File System (Package)** (Sistema de archivos [paquete]), **File System (Project)** (Sistema de archivos [proyecto]), **Embedded package** (Paquete insertado) o **Package store** (Almacén de paquetes) como ubicación del paquete, también debe volver a escribir las propiedades confidenciales del paquete para asignarles valores en esta pestaña. 
   
Al asignar valores a las propiedades del paquete, puede agregar contenido dinámico mediante expresiones, funciones, variables del sistema de Data Factory y parámetros o variables de canalización de Data Factory.

Los valores asignados en los archivos de configuración y en la pestaña **SSIS Parameters** (Parámetros de SSIS) se pueden invalidar mediante las pestañas **Connection Managers** (Administradores de conexiones) o **Property Overrides** (Reemplazos de propiedad). Los valores asignados en la pestaña **Connection Managers** (Administradores de conexiones) también se pueden usar con la pestaña **Property Overrides** (Reemplazos de propiedad).

Para validar la configuración de la canalización, seleccione **Validate** (Validar) en la barra de herramientas. Para cerrar **Pipeline Validation Report** (Informe de comprobación de la canalización), seleccione **>>** .

Para publicar la canalización en Data Factory, seleccione **Publish All** (Publicar todo). 

### <a name="run-the-pipeline"></a>Ejecución de la canalización
En este paso, desencadenará una ejecución de canalización. 

1. Para desencadenar una ejecución de canalización, seleccione **Trigger** (Desencadenar) en la barra de herramientas y en **Trigger now** (Desencadenar ahora). 

   ![Trigger now (Desencadenar ahora)](./media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-trigger.png)

2. En la ventana **Pipeline Run** (Ejecución de canalización), seleccione **Finish** (Finalizar). 

### <a name="monitor-the-pipeline"></a>Supervisar la canalización

1. Cambie a la pestaña **Monitor** (Supervisar) de la izquierda. Verá la ejecución de canalización y su estado junto con otro tipo de información, como la hora de **inicio de la ejecución**. Para actualizar la vista, seleccione **Refresh** (Actualizar).

   ![Ejecuciones de la canalización](./media/how-to-invoke-ssis-package-stored-procedure-activity/pipeline-runs.png)

2. Seleccione el vínculo **View Activity Runs** (Ver ejecuciones de actividad) en la columna **Actions** (Acciones). Solo verá una ejecución de actividad porque la canalización solo tiene una actividad. Es la actividad de ejecución de paquetes de SSIS.

   ![Ejecuciones de actividad](./media/how-to-invoke-ssis-package-ssis-activity/ssis-activity-runs.png)

3. Ejecute la consulta siguiente en la base de datos SSISDB en el servidor de SQL para comprobar la ejecución del paquete. 

   ```sql
   select * from catalog.executions
   ```

   ![Comprobación de las ejecuciones del paquete](./media/how-to-invoke-ssis-package-stored-procedure-activity/verify-package-executions.png)

4. También puede obtener el identificador de ejecución de SSISDB desde la salida de la ejecución de la actividad de canalización, y usar el identificador para comprobar registros de ejecución y mensajes de error más completos en SQL Server Management Studio.

   ![Obtenga el identificador de ejecución.](media/how-to-invoke-ssis-package-ssis-activity/get-execution-id.png)

### <a name="schedule-the-pipeline-with-a-trigger"></a>Programación de la canalización con un desencadenador

También puede crear un desencadenador programado para la canalización de manera que esta se ejecute según una programación, como por hora o cada día. Para ver un ejemplo, consulte [Create a data factory - Data Factory UI](quickstart-create-data-factory-portal.md#trigger-the-pipeline-on-a-schedule) (Creación de una factoría de datos: interfaz de usuario de Data Factory).

## <a name="run-a-package-with-powershell"></a>Ejecución de un paquete con PowerShell
En esta sección, usará Azure PowerShell para crear una canalización de Data Factory con una actividad de ejecución de paquetes de SSIS que ejecuta un paquete SSIS. 

Instale los módulos de Azure PowerShell más recientes siguiendo las instrucciones paso a paso que se indican en [Cómo instalar y configurar Azure PowerShell](/powershell/azure/install-az-ps).

### <a name="create-a-data-factory-with-azure-ssis-ir"></a>Creación de una factoría de datos con Azure-SSIS IR
Puede usar una factoría de datos existente que ya tenga Azure-SSIS IR aprovisionado o crear una nueva factoría de datos con Azure-SSIS IR. Siga las instrucciones detalladas en el [Tutorial: Implementación de paquetes SSIS en Azure mediante PowerShell](https://docs.microsoft.com/azure/data-factory/tutorial-deploy-ssis-packages-azure-powershell).

### <a name="create-a-pipeline-with-an-execute-ssis-package-activity"></a>Creación de una canalización con una actividad Ejecutar paquete de SSIS 
En este paso se crea una canalización con una actividad Ejecutar paquete de SSIS. La actividad ejecuta el paquete de SSIS. 

1. Cree un archivo JSON denominado `RunSSISPackagePipeline.json` en la carpeta `C:\ADF\RunSSISPackage` con contenido similar al del siguiente ejemplo.

   > [!IMPORTANT]
   > Reemplace los nombres de objeto, descripciones, rutas de acceso, valores de propiedades y parámetros, contraseñas y otros valores de variables antes de guardar el archivo. 
    
   ```json
   {
       "name": "RunSSISPackagePipeline",
       "properties": {
           "activities": [{
               "name": "MySSISActivity",
               "description": "My SSIS package/activity description",
               "type": "ExecuteSSISPackage",
               "typeProperties": {
                   "connectVia": {
                       "referenceName": "MyAzureSSISIR",
                       "type": "IntegrationRuntimeReference"
                   },
                   "executionCredential": {
                       "domain": "MyExecutionDomain",
                       "username": "MyExecutionUsername",
                       "password": {
                           "type": "SecureString",
                           "value": "MyExecutionPassword"
                       }
                   },
                   "runtime": "x64",
                   "loggingLevel": "Basic",
                   "packageLocation": {
                       "type": "SSISDB",
                       "packagePath": "MyFolder/MyProject/MyPackage.dtsx"
                   },
                   "environmentPath": "MyFolder/MyEnvironment",
                   "projectParameters": {
                       "project_param_1": {
                           "value": "123"
                       },
                       "project_param_2": {
                           "value": {
                               "value": "@pipeline().parameters.MyProjectParameter",
                               "type": "Expression"
                           }
                       }
                   },
                   "packageParameters": {
                       "package_param_1": {
                           "value": "345"
                       },
                       "package_param_2": {
                           "value": {
                               "type": "AzureKeyVaultSecret",
                               "store": {
                                   "referenceName": "myAKV",
                                   "type": "LinkedServiceReference"
                               },
                               "secretName": "MyPackageParameter"
                           }
                       }
                   },
                   "projectConnectionManagers": {
                       "MyAdonetCM": {
                           "username": {
                               "value": "MyConnectionUsername"
                           },
                           "password": {
                               "value": {
                                   "type": "SecureString",
                                   "value": "MyConnectionPassword"
                               }
                           }
                       }
                   },
                   "packageConnectionManagers": {
                       "MyOledbCM": {
                           "username": {
                               "value": {
                                   "value": "@pipeline().parameters.MyConnectionUsername",
                                   "type": "Expression"
                               }
                           },
                           "password": {
                               "value": {
                                   "type": "AzureKeyVaultSecret",
                                   "store": {
                                       "referenceName": "myAKV",
                                       "type": "LinkedServiceReference"
                                   },
                                   "secretName": "MyConnectionPassword",
                                   "secretVersion": "MyConnectionPasswordVersion"
                               }
                           }
                       }
                   },
                   "propertyOverrides": {
                       "\\Package.MaxConcurrentExecutables": {
                           "value": 8,
                           "isSensitive": false
                       }
                   }
               },
               "policy": {
                   "timeout": "0.01:00:00",
                   "retry": 0,
                   "retryIntervalInSeconds": 30
               }
           }]
       }
   }
   ```

   Para ejecutar los paquetes almacenados en un sistema de archivos o Azure Files, escriba los valores del paquete y las propiedades de ubicación del registro de la siguiente manera:

   ```json
   {
       {
           {
               {
                   "packageLocation": {
                       "type": "File",
                       "packagePath": "//MyStorageAccount.file.core.windows.net/MyFileShare/MyPackage.dtsx",
                       "typeProperties": {
                           "packagePassword": {
                               "type": "SecureString",
                               "value": "MyEncryptionPassword"
                           },
                           "accessCredential": {
                               "domain": "Azure",
                               "username": "MyStorageAccount",
                               "password": {
                                   "type": "SecureString",
                                   "value": "MyAccountKey"
                               }
                           }
                       }
                   },
                   "logLocation": {
                       "type": "File",
                       "logPath": "//MyStorageAccount.file.core.windows.net/MyFileShare/MyLogFolder",
                       "typeProperties": {
                           "accessCredential": {
                               "domain": "Azure",
                               "username": "MyStorageAccount",
                               "password": {
                                   "type": "AzureKeyVaultSecret",
                                   "store": {
                                       "referenceName": "myAKV",
                                       "type": "LinkedServiceReference"
                                   },
                                   "secretName": "MyAccountKey"
                               }
                           }
                       }
                   }
               }
           }
       }
   }
   ```

   Para ejecutar los paquetes de proyectos almacenados en un sistema de archivos o Azure Files, escriba los valores de las propiedades de ubicación del paquete de la siguiente manera:

   ```json
   {
       {
           {
               {
                   "packageLocation": {
                       "type": "File",
                       "packagePath": "//MyStorageAccount.file.core.windows.net/MyFileShare/MyProject.ispac:MyPackage.dtsx",
                       "typeProperties": {
                           "packagePassword": {
                               "type": "SecureString",
                               "value": "MyEncryptionPassword"
                           },
                           "accessCredential": {
                               "domain": "Azure",
                               "userName": "MyStorageAccount",
                               "password": {
                                   "type": "SecureString",
                                   "value": "MyAccountKey"
                               }
                           }
                       }
                   }
               }
           }
       }
   }
   ```

   Para ejecutar paquetes insertados, escriba los valores de las propiedades de ubicación del paquete de la siguiente manera:

   ```json
   {
       {
           {
               {
                   "packageLocation": {
                       "type": "InlinePackage",
                       "typeProperties": {
                           "packagePassword": {
                               "type": "SecureString",
                               "value": "MyEncryptionPassword"
                           },
                           "packageName": "MyPackage.dtsx",
                           "packageContent":"My compressed/uncompressed package content",
                           "packageLastModifiedDate": "YYYY-MM-DDTHH:MM:SSZ UTC-/+HH:MM"
                       }
                   }
               }
           }
       }
   }
   ```

   Para ejecutar paquetes almacenados en almacenes de paquetes, escriba los valores del paquete y las propiedades de ubicación de configuración de la siguiente manera:

   ```json
   {
       {
           {
               {
                   "packageLocation": {
                       "type": "PackageStore",
                       "packagePath": "myPackageStore/MyFolder/MyPackage",
                       "typeProperties": {
                           "packagePassword": {
                               "type": "SecureString",
                               "value": "MyEncryptionPassword"
                           },
                           "accessCredential": {
                               "domain": "Azure",
                               "username": "MyStorageAccount",
                               "password": {
                                   "type": "SecureString",
                                   "value": "MyAccountKey"
                               }
                           },
                           "configurationPath": "//MyStorageAccount.file.core.windows.net/MyFileShare/MyConfiguration.dtsConfig",
                           "configurationAccessCredential": {
                               "domain": "Azure",
                               "userName": "MyStorageAccount",
                               "password": {
                                   "type": "AzureKeyVaultSecret",
                                   "store": {
                                       "referenceName": "myAKV",
                                       "type": "LinkedServiceReference"
                                   },
                                   "secretName": "MyAccountKey"
                               }
                           }
                       }
                   }
               }
           }
       }
   }
   ```

2. En Azure PowerShell, cambie a la carpeta `C:\ADF\RunSSISPackage`.

3. Para crear la canalización **RunSSISPackagePipeline**, ejecute el cmdlet **Set-AzDataFactoryV2Pipeline**.

   ```powershell
   $DFPipeLine = Set-AzDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName `
                                                  -ResourceGroupName $ResGrp.ResourceGroupName `
                                                  -Name "RunSSISPackagePipeline"
                                                  -DefinitionFile ".\RunSSISPackagePipeline.json"
   ```

   Esta es la salida de ejemplo:

   ```
   PipelineName      : Adfv2QuickStartPipeline
   ResourceGroupName : <resourceGroupName>
   DataFactoryName   : <dataFactoryName>
   Activities        : {CopyFromBlobToBlob}
   Parameters        : {[inputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification], [outputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification]}
   ```

### <a name="run-the-pipeline"></a>Ejecución de la canalización
Use el cmdlet **Invoke-AzDataFactoryV2Pipeline** para ejecutar la canalización. El cmdlet devuelve el identificador de ejecución de la canalización para realizar una supervisión en un futuro.

```powershell
$RunId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName `
                                             -ResourceGroupName $ResGrp.ResourceGroupName `
                                             -PipelineName $DFPipeLine.Name
```

### <a name="monitor-the-pipeline"></a>Supervisar la canalización

Ejecute el script de PowerShell siguiente para comprobar continuamente el estado de ejecución de la canalización hasta que termine de copiar los datos. Copie y pegue el siguiente script en la ventana de PowerShell y seleccione Entrar. 

```powershell
while ($True) {
    $Run = Get-AzDataFactoryV2PipelineRun -ResourceGroupName $ResGrp.ResourceGroupName `
                                               -DataFactoryName $DataFactory.DataFactoryName `
                                               -PipelineRunId $RunId

    if ($Run) {
        if ($run.Status -ne 'InProgress') {
            Write-Output ("Pipeline run finished. The status is: " +  $Run.Status)
            $Run
            break
        }
        Write-Output  "Pipeline is running...status: InProgress"
    }

    Start-Sleep -Seconds 10
}   
```

También puede supervisar la canalización mediante Azure Portal. Para ver instrucciones paso a paso, consulte [Supervisar la canalización](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline).

### <a name="schedule-the-pipeline-with-a-trigger"></a>Programación de la canalización con un desencadenador
En el paso anterior, ha ejecutado la canalización a petición. También puede crear un desencadenador de programación para ejecutar la canalización en una programación, como cada hora o cada día.

1. Cree un archivo JSON denominado `MyTrigger.json` en la carpeta `C:\ADF\RunSSISPackage` con el siguiente contenido: 
        
   ```json
   {
       "properties": {
           "name": "MyTrigger",
           "type": "ScheduleTrigger",
           "typeProperties": {
               "recurrence": {
                   "frequency": "Hour",
                   "interval": 1,
                   "startTime": "2017-12-07T00:00:00-08:00",
                   "endTime": "2017-12-08T00:00:00-08:00"
               }
           },
           "pipelines": [{
               "pipelineReference": {
                   "type": "PipelineReference",
                   "referenceName": "RunSSISPackagePipeline"
               },
               "parameters": {}
           }]
       }
   }    
   ```
    
1. En Azure PowerShell, cambie a la carpeta `C:\ADF\RunSSISPackage`.
1. Ejecute el cmdlet **Set-AzDataFactoryV2Trigger**, que crea el desencadenador. 

   ```powershell
   Set-AzDataFactoryV2Trigger -ResourceGroupName $ResGrp.ResourceGroupName `
                                   -DataFactoryName $DataFactory.DataFactoryName `
                                   -Name "MyTrigger" -DefinitionFile ".\MyTrigger.json"
   ```
1. De manera predeterminada, el desencadenador está en estado detenido. Inicie el desencadenador al ejecutar el cmdlet **Start-AzDataFactoryV2Trigger**. 

   ```powershell
   Start-AzDataFactoryV2Trigger -ResourceGroupName $ResGrp.ResourceGroupName `
                                     -DataFactoryName $DataFactory.DataFactoryName `
                                     -Name "MyTrigger" 
   ```
1. Confirme que el desencadenador se ha iniciado al ejecutar el cmdlet **Get-AzDataFactoryV2Trigger**. 

   ```powershell
   Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName `
                                   -DataFactoryName $DataFactoryName `
                                   -Name "MyTrigger"     
   ```    
1. Ejecute el comando siguiente al comenzar la hora siguiente. Por ejemplo, si la hora actual es 15:25 UTC, ejecute el comando a las 16:00 UTC. 
    
   ```powershell
   Get-AzDataFactoryV2TriggerRun -ResourceGroupName $ResourceGroupName `
                                      -DataFactoryName $DataFactoryName `
                                      -TriggerName "MyTrigger" `
                                      -TriggerRunStartedAfter "2017-12-06" `
                                      -TriggerRunStartedBefore "2017-12-09"
   ```

   Ejecute la consulta siguiente en la base de datos SSISDB en el servidor de SQL para comprobar la ejecución del paquete. 

   ```sql
   select * from catalog.executions
   ```

## <a name="next-steps"></a>Pasos siguientes
Vea la siguiente entrada de blog:
- [Modernización y ampliación de los flujos de trabajo ETL/ETL con actividades de SSIS en las canalizaciones de Azure Data Factory](https://techcommunity.microsoft.com/t5/SQL-Server-Integration-Services/Modernize-and-Extend-Your-ETL-ELT-Workflows-with-SSIS-Activities/ba-p/388370)
