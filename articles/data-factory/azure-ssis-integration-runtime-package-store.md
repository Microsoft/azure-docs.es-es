---
title: Administración de paquetes con el almacén de paquetes de Azure-SSIS Integration Runtime
description: Aprenda a administrar paquetes con el almacén de paquetes de Azure-SSIS Integration Runtime.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: mflasko
ms.custom: seo-lt-2019
ms.date: 07/20/2020
ms.openlocfilehash: 6455c186e05fc98b1ec340c152f9b3e5710f1dd5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087921"
---
# <a name="manage-packages-with-azure-ssis-integration-runtime-package-store"></a>Administración de paquetes con el almacén de paquetes de Azure-SSIS Integration Runtime

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Para migrar mediante lift-and-shift las cargas de trabajo locales de SQL Server Integration Services (SSIS) a la nube, puede aprovisionar Azure-SSIS Integration Runtime (IR) en Azure Data Factory (ADF). Para obtener más información, vea [Aprovisionamiento de Azure-SSIS Integration Runtime](https://docs.microsoft.com/azure/data-factory/tutorial-deploy-ssis-packages-azure). Azure-SSIS Integration Runtime admite:

- La ejecución de paquetes implementados en el catálogo de SSIS (SSISDB) hospedados por un servidor de Azure SQL Database o por Instancia administrada (modelo de implementación de proyectos)
- La ejecución de paquetes implementados en el sistema de archivos, en Azure Files o en una base de datos de SQL Server (MSDB) hospedados por Instancia administrada de Azure SQL (modelo de implementación de paquetes)

Cuando se usa el modelo de implementación de paquetes, puede elegir aprovisionar la instancia de Azure-SSIS IR con almacenes de paquetes. Estos almacenes proporcionan una capa de administración de paquetes sobre el sistema de archivos, Azure Files o una base de datos MSDB hospedada por Azure SQL Managed Instance. El almacén de paquetes de Azure-SSIS IR permite importar, exportar, eliminar y ejecutar paquetes, así como supervisar y detener los paquetes en ejecución mediante SQL Server Management Studio (SSMS) de forma similar al [almacén de paquetes de SSIS heredado](https://docs.microsoft.com/sql/integration-services/service/package-management-ssis-service?view=sql-server-2017). 

## <a name="connect-to-azure-ssis-ir"></a>Conexión a Azure-SSIS IR

Una vez que se ha aprovisionado Azure-SSIS IR, puede conectarse a este para examinar los almacenes de paquetes en SSMS.

![Conexión a Azure-SSIS IR](media/azure-ssis-integration-runtime-package-store/ssms-package-store-connect.png)

En la ventana **Explorador de objetos** de SSMS, seleccione **Azure-SSIS Integration Runtime** en el menú desplegable **Conectar**. A continuación, inicie sesión en Azure y seleccione la suscripción y las instancias de ADF y Azure-SSIS IR pertinentes que ha aprovisionado con almacenes de paquetes. Aparecerá Azure-SSIS IR con los nodos **Paquetes en ejecución** y **Paquetes almacenados** situados debajo. Expanda el nodo **Paquetes almacenados** para ver debajo los almacenes de paquetes. Expanda los almacenes de paquetes para ver debajo las carpetas y los paquetes. Es posible que se le pida que escriba las credenciales de acceso para los almacenes de paquetes, en caso de que SSMS no se pueda conectar a ellos automáticamente. Por ejemplo, si expande un almacén de paquetes encima de MSDB, es posible que se le pida que se conecte primero a Instancia administrada de Azure SQL.

![Conexión a Instancia administrada de Azure SQL](media/azure-ssis-integration-runtime-package-store/ssms-package-store-connect2.png)

## <a name="manage-folders-and-packages"></a>Administración de carpetas y paquetes

Al conectarse a la instancia de Azure-SSIS IR en SSMS, puede hacer clic con el botón derecho en cualquier almacén de paquetes, carpeta o paquete y aparecerá un menú emergente. Seleccione **Nueva carpeta**, **Importar paquete**, **Exportar paquete**, **Eliminar** o **Actualizar**.

   ![Administración de carpetas y paquetes](media/azure-ssis-integration-runtime-package-store/ssms-package-store-manage.png)

   *  Seleccione **Nueva carpeta** para crear una carpeta para los paquetes importados.

   *  Seleccione **Importar paquete** para importar paquetes del **Sistema de archivos**, **SQL Server** (MSDB) o el **Almacén de paquetes de SSIS** heredado en el almacén de paquetes.

      ![Importación de paquete](media/azure-ssis-integration-runtime-package-store/ssms-package-store-import.png)

      En función de la **Ubicación del paquete** desde la que se va a importar, seleccione el valor pertinente de **Servidor**/**Tipo de autenticación**, escriba las credenciales de acceso si es necesario, seleccione la **Ruta de acceso del paquete** y escriba el nuevo **Nombre del paquete**. Al importar paquetes, no se puede cambiar su nivel de protección. Para cambiarlo, use SQL Server Data Tools (SSDT) o la utilidad de línea de comandos `dtutil`.

      > [!NOTE]
      > La importación de paquetes SSIS en almacenes de paquetes de Azure-SSIS IR solo se puede realizar de uno en uno, y simplemente se copian en el sistema de archivos o en las instancias de MSDB o Azure Files subyacentes, mientras se conserva la versión de SQL Server o SSIS. 
      >
      > Como Azure-SSIS IR tiene actualmente un nivel de compatibilidad predeterminado de 140, que equivale a **SQL Server 2017**, la ejecución de paquetes de versiones inferiores en él los actualizará a paquetes de SSIS 2017 en tiempo de ejecución. No se admite la ejecución de paquetes de versiones posteriores.
      >
      > Además, como los almacenes de paquetes SSIS heredados están vinculados a una versión de SQL Server específica y solo se puede tener acceso a ellos en SSMS con esa versión, los paquetes de versiones inferiores de los almacenes de paquetes SSIS heredados deben exportarse primero al sistema de archivos mediante la versión de SSMS designada antes de poderse importar en los almacenes de paquetes de Azure-SSIS IR mediante SSMS 2019 o versiones posteriores.
      >
      > Como alternativa, para importar varios paquetes SSIS en almacenes de paquetes de Azure-SSIS IR y cambiar su nivel de protección, puede usar la utilidad de línea de comandos [dtutil](https://docs.microsoft.com/sql/integration-services/dtutil-utility?view=sql-server-2017). Consulte [Implementación de varios paquetes con dtutil](#deploying-multiple-packages-with-dtutil).

   *  Seleccione **Exportar paquete** para exportar paquetes del almacén de paquetes al **Sistema de archivos**, **SQL Server** (MSDB) o el **Almacén de paquetes de SSIS** heredado.

      ![Exportación de paquetes](media/azure-ssis-integration-runtime-package-store/ssms-package-store-export.png)

      En función de la **Ubicación del paquete** a la que se va a exportar, seleccione el valor pertinente de **Servidor**/**Tipo de autenticación**, escriba las credenciales de acceso si es necesario y seleccione la **Ruta de acceso del paquete**. Al exportar paquetes, si están cifrados, escriba primero las contraseñas para descifrarlos. Luego, puede cambiar su nivel de protección, por ejemplo, para evitar almacenar datos confidenciales o para cifrar todos los datos con una clave de usuario o contraseña.

      > [!NOTE]
      > La exportación de paquetes SSIS desde almacenes de paquetes de Azure-SSIS IR solo se puede hacer uno a uno y, cuando se hace sin cambiar su nivel de protección, simplemente se copian, al tiempo que se mantiene la versión de SQL Server o SSIS; de lo contrario, se actualizan a paquetes de SSIS 2019 o versiones posteriores.
      >
      > Como Azure-SSIS IR tiene actualmente un nivel de compatibilidad predeterminado de 140, que equivale a **SQL Server 2017**, la ejecución de paquetes de versiones inferiores en él los actualizará a paquetes de SSIS 2017 en tiempo de ejecución. No se admite la ejecución de paquetes de versiones posteriores.
      >
      > Como alternativa, para exportar varios paquetes SSIS desde almacenes de paquetes de Azure-SSIS IR y cambiar al mismo tiempo su nivel de protección, puede usar la utilidad de línea de comandos [dtutil](https://docs.microsoft.com/sql/integration-services/dtutil-utility?view=sql-server-2017). Consulte [Implementación de varios paquetes con dtutil](#deploying-multiple-packages-with-dtutil).

   *  Seleccione **Eliminar** para eliminar los paquetes o las carpetas del almacén de paquetes.

   *  Seleccione **Actualizar** para mostrar los paquetes y las carpetas recién agregados al almacén de paquetes.

## <a name="execute-packages"></a>Ejecución de paquetes

Al conectarse a Azure-SSIS IR en SSMS, puede hacer clic con el botón derecho en cualquier paquete almacenado y aparecerá un menú emergente. Seleccione **Ejecutar paquete**.  Se abrirá el cuadro de diálogo **Utilidad de ejecución de paquetes**, donde podrá configurar las ejecuciones de paquetes en Azure-SSIS IR como actividades de tipo Ejecutar paquete de SSIS en canalizaciones de ADF.

![Páginas 1 y 2 de Utilidad de ejecución de paquetes](media/azure-ssis-integration-runtime-package-store/ssms-package-store-execute.png)

![Páginas 3 y 4 de Utilidad de ejecución de paquetes](media/azure-ssis-integration-runtime-package-store/ssms-package-store-execute2.png)

Las páginas **General**, **Configuraciones**, **Opciones de ejecución** y **Registro** del cuadro de diálogo **Utilidad de ejecución de paquetes** corresponden a la pestaña **Configuración** de la actividad Ejecutar paquete de SSIS. En estas páginas, puede escribir la contraseña de cifrado del paquete y acceder a la información del archivo de configuración del paquete. También puede especificar las propiedades y las credenciales de ejecución del paquete, así como la información de acceso de la carpeta de registro.  La página **Establecer valores** del cuadro de diálogo **Utilidad de ejecución de paquetes** se corresponde con la pestaña **Invalidaciones de propiedad** de la actividad Ejecutar paquete de SSIS, donde se pueden especificar las propiedades del paquete que se van a invalidar. Para obtener información, vea [Ejecución de un paquete de SSIS mediante la actividad Ejecutar paquete de SSIS en canalizaciones de ADF](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity).

Al seleccionar el botón **Ejecutar**, se genera y se desencadena automáticamente una nueva canalización de ADF con la actividad Ejecutar paquete de SSIS. Si ya existe una canalización de ADF con la misma configuración, se volverá a ejecutar y no se generará una nueva canalización. La canalización de ADF y la actividad Ejecutar paquete de SSIS se denominarán `Pipeline_SSMS_YourPackageName_HashString` y `Activity_SSMS_YourPackageName`, respectivamente.

![Botón de la Utilidad de ejecución de paquetes](media/azure-ssis-integration-runtime-package-store/ssms-package-store-execute3.png)

![Ejecutar una actividad de paquete SSIS](media/azure-ssis-integration-runtime-package-store/ssis-activity-package-store.png)

## <a name="monitor-and-stop-running-packages"></a>Supervisión y detención de los paquetes en ejecución

Después de conectarse a la instancia de Azure-SSIS IR en SSMS, puede expandir el nodo **Running Packages** (Paquetes en ejecución) para ver los paquetes que se están ejecutando.  Haga clic con el botón derecho en cualquiera de ellos para que aparezca un menú emergente y seleccione **Detener** o **Actualizar**.

   ![Supervisión y detención de los paquetes en ejecución](media/azure-ssis-integration-runtime-package-store/ssms-package-store-monitor.png)

   *  Seleccione **Detener** para cancelar la canalización de ADF que se está ejecutando actualmente y que ejecuta el paquete como una actividad Ejecutar paquete de SSIS.

   *  Seleccione **Actualizar** para mostrar los paquetes en ejecución recientemente en los almacenes de paquetes.

## <a name="monitor-azure-ssis-ir-and-edit-package-stores"></a>Supervisión de Azure-SSIS IR y edición de los almacenes de paquetes

Al conectarse a la instancia de Azure-SSIS IR en SSMS, puede hacer clic con el botón derecho en ella y aparecerá un menú emergente. Seleccione **Go to Azure Data Factory portal** (Ir al portal de Azure Data Factory) o **Actualizar**.

   ![Ir al portal de ADF](media/azure-ssis-integration-runtime-package-store/ssms-package-store-monitor2.png)

   *  Seleccione **Go to Azure Data Factory portal** (Ir al portal de Azure Data Factory) para abrir la página **Integration runtimes** (Entornos de ejecución de integración) del centro de supervisión de ADF, donde puede supervisar Azure-SSIS IR. En el icono **PACKAGE STORES** (ALMACENES DE PAQUETES), puede ver el número de almacenes de paquetes que están conectados a Azure-SSIS IR.  Al seleccionar ese número, aparecerá una ventana emergente donde puede editar los servicios vinculados de ADF que almacenan la información de acceso de los almacenes de paquetes.

      ![Edición de los almacenes de paquetes](media/azure-ssis-integration-runtime-package-store/ssms-package-store-monitor3.png)

   *  Seleccione **Actualizar** para mostrar las carpetas o paquetes recién agregados a los almacenes de paquetes y los paquetes en ejecución en los almacenes de paquetes.

## <a name="deploying-multiple-packages-with-dtutil"></a>Implementación de varios paquetes con dtutil

Para migrar mediante "lift-and-shift" las cargas de trabajo SSIS locales a SSIS en ADF y mantener al mismo tiempo el modelo de implementación de paquetes heredado, debe implementar los paquetes desde el sistema de archivos, la base de datos MSDB hospedada por SQL Server o los almacenes de paquetes SSIS heredados a Azure Files, la base de datos MSDB hospedada por Azure SQL Managed Instance o los almacenes de paquetes de Azure-SSIS IR. Al mismo tiempo y, si aún no lo ha hecho, debe cambiar el nivel de protección de cifrado mediante la clave de usuario a sin cifrar o cifrado mediante contraseña.

Puede usar la utilidad de línea de comandos [dtutil](https://docs.microsoft.com/sql/integration-services/dtutil-utility?view=sql-server-2017) que se incluye con la instalación de SQL Server o SSIS para implementar varios paquetes en lotes. Como está vinculada a una versión específica de SSIS, si se usa para implementar paquetes de versiones inferiores sin cambiar el nivel de protección, simplemente se copian, mientras se conserva su versión de SSIS. Si la usa para implementarlos y cambiar su nivel de protección al mismo tiempo, los actualizará a su versión de SSIS.

 Como Azure-SSIS IR tiene actualmente un nivel de compatibilidad predeterminado de 140, que equivale a **SQL Server 2017**, la ejecución de paquetes de versiones inferiores en él los actualizará a paquetes de SSIS 2017 en tiempo de ejecución. No se admite la ejecución de paquetes de versiones posteriores.

Por lo tanto, para evitar actualizaciones en tiempo de ejecución, en la implementación de paquetes que se van a ejecutar en Azure-SSIS IR en el modelo de implementación de paquetes se debe usar dtutil 2017 que se incluye con la instalación de SQL Server o SSIS 2017. Para este fin, puede descargar e instalar la [edición para desarrolladores de SQL Server o SSIS 2017](https://go.microsoft.com/fwlink/?linkid=853016), que es gratuita. Una vez instalada, encontrará dtutil 2017 en esta carpeta: `YourLocalDrive:\Program Files\Microsoft SQL Server\140\DTS\Binn`.

### <a name="deploying-multiple-packages-from-file-system-on-premises-into-azure-files-with-dtutil"></a>Implementación de varios paquetes desde el sistema de archivos local en Azure Files con dtutil

 Para implementar varios paquetes desde el sistema de archivos en Azure Files y cambiar su nivel de protección al mismo tiempo, puede ejecutar los siguientes comandos en un símbolo del sistema. Reemplace todas las cadenas específicas de su caso.
  
```dos
REM Persist the access credentials for Azure Files on your local machine
cmdkey /ADD:YourStorageAccountName.file.core.windows.net /USER:azure\YourStorageAccountName /PASS:YourStorageAccountKey

REM Connect Azure Files to a drive on your local machine
net use Z: \\YourStorageAccountName.file.core.windows.net\YourFileShare /PERSISTENT:Yes

REM Go to a local folder where you store your packages
cd YourLocalDrive:\...\YourPackageFolder

REM Run dtutil in a loop to deploy your packages from the local folder into Azure Files while switching their protection level
for %f in (*.dtsx) do dtutil.exe /FILE %f /ENCRYPT FILE;Z:\%f;2;YourEncryptionPassword
```

Para ejecutar los comandos anteriores en un archivo por lotes, reemplace `%f` por `%%f`.

Para implementar varios paquetes de almacenes de paquetes SSIS heredados en el sistema de archivos en Azure Files y cambiar el nivel de protección al mismo tiempo, puede usar los mismos comandos, pero reemplace `YourLocalDrive:\...\YourPackageFolder` por una carpeta local usada por los almacenes de paquetes SSIS heredados: `YourLocalDrive:\Program Files\Microsoft SQL Server\YourSQLServerDefaultCompatibilityLevel\DTS\Packages\YourPackageFolder`. Por ejemplo, si el almacén de paquetes SSIS heredado está vinculado a SQL Server 2016, vaya a `YourLocalDrive:\Program Files\Microsoft SQL Server\130\DTS\Packages\YourPackageFolder`.  Puede encontrar el valor de `YourSQLServerDefaultCompatibilityLevel` en una [lista de niveles de compatibilidad predeterminados de SQL Server](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-compatibility-level?view=sql-server-ver15#arguments).

Si ha configurado almacenes de paquetes de Azure-SSIS IR en Azure Files, los paquetes implementados aparecerán en ellos cuando se conecte a la instancia de Azure-SSIS IR en SSMS 2019 o versiones posteriores.

### <a name="deploying-multiple-packages-from-msdb-on-premises-into-msdb-in-azure-with-dtutil"></a>Implementación de varios paquetes de la base de datos MSDB local en la base de datos MSDB de Azure con dtutil

 Para implementar varios paquetes desde la base de datos MSDB hospedada por SQL Server o los almacenes de paquetes SSIS heredados de MSDB en la base de datos MSDB hospedada por Azure SQL Managed Instance y cambiar su nivel de protección al mismo tiempo, puede conectarse a SQL Server en SSMS, hacer clic con el botón derecho en el nodo `Databases->System Databases->msdb` en el **Explorador de objetos** de SSMS para abrir una ventana **Nueva consulta** y ejecutar el siguiente script T-SQL. Reemplace todas las cadenas específicas de su caso.  
  
```sql
BEGIN
  SELECT 'dtutil /SQL '+f.foldername+'\'+NAME+' /ENCRYPT SQL;'+f.foldername+'\'+NAME+';2;YourEncryptionPassword /DestServer YourSQLManagedInstanceEndpoint /DestUser YourSQLAuthUsername /DestPassword YourSQLAuthPassword'
  FROM msdb.dbo.sysssispackages p
  inner join msdb.dbo.sysssispackagefolders f
  ON p.folderid = f.folderid
END
```

Para usar el punto de conexión público o privado de Azure SQL Managed Instance, reemplace `YourSQLManagedInstanceEndpoint` por `YourSQLMIName.YourDNSPrefix.database.windows.net`/`YourSQLMIName.public.YourDNSPrefix.database.windows.net,3342`, respectivamente.

El script generará líneas de comandos de dtutil para todos los paquetes de MSDB que se puedan seleccionar de una vez, copiar y pegar y ejecutar en un símbolo del sistema.

![Generación de líneas de comandos de dtutil](media/azure-ssis-integration-runtime-package-store/sql-server-msdb-to-sql-mi-msdb.png)

```dos
dtutil /SQL YourFolder\YourPackage1 /ENCRYPT SQL;YourFolder\YourPackage1;2;YourEncryptionPassword /DestServer YourSQLManagedInstanceEndpoint /DestUser YourUserName /DestPassword YourPassword
dtutil /SQL YourFolder\YourPackage2 /ENCRYPT SQL;YourFolder\YourPackage2;2;YourEncryptionPassword /DestServer YourSQLManagedInstanceEndpoint /DestUser YourUserName /DestPassword YourPassword
dtutil /SQL YourFolder\YourPackage3 /ENCRYPT SQL;YourFolder\YourPackage3;2;YourEncryptionPassword /DestServer YourSQLManagedInstanceEndpoint /DestUser YourUserName /DestPassword YourPassword
```

Si ha configurado almacenes de paquetes de Azure-SSIS IR de MSDB, los paquetes implementados aparecerán en ellos cuando se conecte a la instancia de Azure-SSIS IR en SSMS 2019 o versiones posteriores.

### <a name="deploying-multiple-packages-from-msdb-on-premises-into-azure-files-with-dtutil"></a>Implementación de varios paquetes desde MSDB local en Azure Files con dtutil

 Para implementar varios paquetes desde la base de datos MSDB hospedada por SQL Server o los almacenes de paquetes SSIS heredados de MSDB en Azure Files y cambiar su nivel de protección al mismo tiempo, puede conectarse a SQL Server en SSMS, hacer clic con el botón derecho en el nodo `Databases->System Databases->msdb` en el **Explorador de objetos** de SSMS para abrir una ventana **Nueva consulta** y ejecutar el siguiente script T-SQL. Reemplace todas las cadenas específicas de su caso.  
  
```sql
BEGIN
  SELECT 'dtutil /SQL '+f.foldername+'\'+NAME+' /ENCRYPT FILE;Z:\'+f.foldername+'\'+NAME+'.dtsx;2;YourEncryptionPassword' 
  FROM msdb.dbo.sysssispackages p
  inner join msdb.dbo.sysssispackagefolders f
  ON p.folderid = f.folderid
END
```

El script generará líneas de comandos de dtutil para todos los paquetes de MSDB que se puedan seleccionar de una vez, copiar y pegar y ejecutar en un símbolo del sistema.

```dos
REM Persist the access credentials for Azure Files on your local machine
cmdkey /ADD:YourStorageAccountName.file.core.windows.net /USER:azure\YourStorageAccountName /PASS:YourStorageAccountKey

REM Connect Azure Files to a drive on your local machine
net use Z: \\YourStorageAccountName.file.core.windows.net\YourFileShare /PERSISTENT:Yes

REM Multiselect, copy & paste, and run the T-SQL-generated dtutil command lines to deploy your packages from MSDB on premises into Azure Files while switching their protection level
dtutil /SQL YourFolder\YourPackage1 /ENCRYPT FILE;Z:\YourFolder\YourPackage1.dtsx;2;YourEncryptionPassword
dtutil /SQL YourFolder\YourPackage2 /ENCRYPT FILE;Z:\YourFolder\YourPackage2.dtsx;2;YourEncryptionPassword
dtutil /SQL YourFolder\YourPackage3 /ENCRYPT FILE;Z:\YourFolder\YourPackage3.dtsx;2;YourEncryptionPassword
```

Si ha configurado almacenes de paquetes de Azure-SSIS IR en Azure Files, los paquetes implementados aparecerán en ellos cuando se conecte a la instancia de Azure-SSIS IR en SSMS 2019 o versiones posteriores.

## <a name="next-steps"></a>Pasos siguientes

Puede volver a ejecutar o editar las canalizaciones de ADF generadas automáticamente con actividades de tipo Ejecutar paquete de SSIS, o bien crear otras nuevas en el portal de ADF. Para obtener información, vea [Ejecución de un paquete de SSIS mediante la actividad Ejecutar paquete de SSIS en canalizaciones de ADF](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity).
