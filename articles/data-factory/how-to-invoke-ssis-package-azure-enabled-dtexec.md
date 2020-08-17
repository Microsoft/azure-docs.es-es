---
title: Ejecución de paquetes de SQL Server Integration Services (SSIS) con la utilidad dtexec habilitada para Azure
description: Obtenga información sobre cómo ejecutar paquetes de SQL Server Integration Services (SSIS) con la utilidad dtexec habilitada para Azure.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 04/12/2020
author: swinarko
ms.author: sawinark
manager: mflasko
ms.reviewer: douglasl
ms.openlocfilehash: 94b581f677e370911a60db08276ff7dd0eb45486
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87927086"
---
# <a name="run-sql-server-integration-services-packages-with-the-azure-enabled-dtexec-utility"></a>Ejecución de paquetes de SQL Server Integration Services con la utilidad dtexec habilitada para Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se describe la utilidad de símbolo del sistema dtexec (AzureDTExec) habilitada para Azure. Se utiliza para ejecutar paquetes de SQL Server Integration Services en Azure-SSIS Integration Runtime (IR) de Azure Data Factory.

La utilidad dtexec tradicional está incluida con SQL Server. Para obtener más información, consulte [utilidad dtexec](https://docs.microsoft.com/sql/integration-services/packages/dtexec-utility?view=sql-server-2017). A menudo, los orquestadores o programadores de terceros, como ActiveBatch o Control-M, entre otros, invocan esta utilidad para ejecutar paquetes SSIS de forma local. 

La utilidad AzureDTExec moderna se incluye en la herramienta SQL Server Management Studio (SSMS). También pueden invocarla los orquestadores o programadores de terceros para ejecutar paquetes SSIS en Azure. Facilita la migración o las operaciones de lift-and-shift de los paquetes de SSIS a la nube. Después de la migración, si quiere seguir usando orquestadores o programadores de terceros en las operaciones cotidianas, ahora pueden invocar AzureDTExec en lugar de dtexec.

AzureDTExec ejecuta los paquetes como actividades Ejecutar paquete de SSIS en las canalizaciones de Data Factory. Para obtener más información, consulte [Ejecución paquetes de SSIS como actividades de Azure Data Factory](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity). 

AzureDTExec se puede configurar mediante SSMS para usar una aplicación de Azure Active Directory (Azure AD) que genera canalizaciones en la factoría de datos. También puede configurarse para acceder a los sistemas de archivos, recursos compartidos de archivos o Azure Files donde se almacenan los paquetes. En función de los valores que proporcione para sus opciones de invocación, AzureDTExec genera y ejecuta una canalización de Data Factory única con la actividad Ejecutar paquete SSIS. Al invocar AzureDTExec con los mismos valores para sus opciones, se vuelve a ejecutar la canalización existente.

## <a name="prerequisites"></a>Prerrequisitos
Para usar AzureDTExec, descargue e instale la versión más reciente de SSMS, que es la versión 18.3 o posterior. Descárguelo de [este sitio web](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-2017).

## <a name="configure-the-azuredtexec-utility"></a>Configuración de la utilidad AzureDTExec
La instalación de SSMS en la máquina local también instala AzureDTExec. Para configurar las opciones, inicie SSMS con la opción **Ejecutar como administrador**. A continuación, seleccione **Herramientas** > **Migrar a Azure** > **Configurar DTExec habilitado para Azure**.

![Configuración del menú de dtexec habilitado para Azure](media/how-to-invoke-ssis-package-azure-enabled-dtexec/ssms-azure-enabled-dtexec-menu.png)

Esta acción muestra la ventana **AzureDTExecConfig** que debe abrirse con privilegios administrativos para que pueda escribir en el archivo *AzureDTExec.settings*. Si no ha ejecutado SSMS como administrador, se abre una ventana de Control de cuentas de usuario (UAC). Escriba la contraseña de administrador para elevar sus privilegios.

![Configuración de las opciones de dtexec habilitadas para Azure](media/how-to-invoke-ssis-package-azure-enabled-dtexec/ssms-azure-enabled-dtexec-settings.png)

En la ventana **AzureDTExecConfig**, especifique las opciones de configuración como se indica a continuación:

- **ApplicationId**: escriba el identificador único de la aplicación de Azure AD que cree con los permisos adecuados para generar canalizaciones en la factoría de datos. Para más información, consulte [Creación de una aplicación de Azure AD y entidad de servicio mediante Azure Portal](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal).
- **AuthenticationKey**: escriba la clave de autenticación de la aplicación de Azure AD.
- **TenantId**: escriba el identificador único del inquilino de Azure AD, en el que se crea la aplicación de Azure AD.
- **DataFactory**: escriba el nombre de la factoría de datos, en la que las canalizaciones únicas con la actividad Ejecutar paquete de SSIS en ellas se generan en función de los valores de las opciones proporcionadas al invocar AzureDTExec.
- **IRName**: escriba el nombre de Azure-SSIS IR en la factoría de datos, en la que se ejecutarán los paquetes especificados en la ruta de acceso de la convención de nomenclatura universal (UNC) al invocar AzureDTExec.
- **PipelineNameHashStrLen**: escriba la longitud de las cadenas hash que se generarán a partir de los valores de las opciones que se proporcionan al invocar AzureDTExec. Las cadenas se usan para formar nombres únicos para las canalizaciones de Data Factory que ejecutan los paquetes en Azure-SSIS IR. Normalmente una longitud de 32 caracteres es suficiente.
- **ResourceGroup**: escriba el nombre del grupo de recursos de Azure en el que se creó la factoría de datos.
- **SubscriptionId**: escriba el identificador único de la suscripción de Azure, en el que se crea la factoría de datos.
- **LogAccessDomain**: Escriba la credencial de dominio para acceder a la carpeta de registro en la ruta de acceso UNC cuando se escriban los archivos de registro, necesaria cuando se especifica **LogPath** y **LogLevel** no es un valor **null**.
- **LogAccessPassword**: Escriba la credencial de contraseña para acceder a la carpeta de registro en la ruta de acceso UNC cuando se escriban los archivos de registro, necesaria cuando se especifica **LogPath** y **LogLevel** no es un valor **null**.
- **LogAccessUserName**: Escriba la credencial de nombre de usuario para acceder a la carpeta de registro en la ruta de acceso UNC cuando se escriban los archivos de registro, necesaria cuando se especifica **LogPath** y **LogLevel** no es un valor **null**.
- **LogLevel**: escriba el ámbito seleccionado de registro de las opciones predefinidas **null**, **Basic**, **Verbose** o **Performance** para las ejecuciones de paquetes en Azure-SSIS IR.
- **LogPath**: escriba la ruta de acceso UNC de la carpeta de registro, en la que se escribirán los archivos de registro de las ejecuciones del paquete en Azure-SSIS IR.
- **PackageAccessDomain**: escriba la credencial de dominio para acceder a los paquetes en su ruta de acceso UNC especificada al invocar AzureDTExec.
- **PackageAccessPassword**: escriba la credencial de contraseña para acceder a los paquetes en su ruta de acceso UNC especificada al invocar AzureDTExec.
- **PackageAccessUserName**: escriba la credencial de nombre de usuario para acceder a los paquetes en su ruta de acceso UNC especificada al invocar AzureDTExec.

Para almacenar los paquetes y archivos de registro en los sistemas de archivos o recursos compartidos de archivos de forma local, una la instancia de Azure-SSIS IR a una red virtual conectada a la red local. De este modo, esta última puede capturar los paquetes y escribir los archivos de registro. Para más información, consulte [Unión de una instancia de Azure-SSIS IR a una red virtual](https://docs.microsoft.com/azure/data-factory/join-azure-ssis-integration-runtime-virtual-network).

Para evitar que se muestren los valores confidenciales escritos en el archivo *AzureDTExec.settings* en texto sin formato, se codifican en cadenas de codificación en Base64. Al invocar AzureDTExec, todas las cadenas codificadas en Base64 se descodifican de vuelta a sus valores originales. Para proteger aún más el archivo *AzureDTExec.settings*, limite las cuentas que pueden acceder a él.

## <a name="invoke-the-azuredtexec-utility"></a>Invocación de la utilidad AzureDTExec
Puede invocar AzureDTExec en el símbolo de la línea de comandos y proporcionar los valores pertinentes para las opciones específicas en el escenario de caso de uso.

La utilidad está instalada en `{SSMS Folder}\Common7\IDE\CommonExtensions\Microsoft\SSIS\150\Binn`. Puede agregar su ruta de acceso a la variable de entorno 'PATH' para que se invoque desde cualquier lugar.

```dos
> cd "C:\Program Files (x86)\Microsoft SQL Server Management Studio 18\Common7\IDE\CommonExtensions\Microsoft\SSIS\150\Binn"
> AzureDTExec.exe  ^
  /F \\MyStorageAccount.file.core.windows.net\MyFileShare\MyPackage.dtsx  ^
  /Conf \\MyStorageAccount.file.core.windows.net\MyFileShare\MyConfig.dtsConfig  ^
  /Conn "MyConnectionManager;Data Source=MyDatabaseServer.database.windows.net;User ID=MyAdminUsername;Password=MyAdminPassword;Initial Catalog=MyDatabase"  ^
  /Set \package.variables[MyVariable].Value;MyValue  ^
  /De MyEncryptionPassword
```

La invocación de AzureDTExec ofrece opciones similares al invocar dtexec. Para obtener más información, consulte [utilidad dtexec](https://docs.microsoft.com/sql/integration-services/packages/dtexec-utility?view=sql-server-2017). Estas son las opciones que se admiten actualmente:

- **/F[ile]** : carga un paquete que está almacenado en el sistema de archivos, recurso compartido de archivos o Azure Files. Como valor de esta opción, puede especificar la ruta de acceso UNC para el archivo de paquete en el sistema de archivos, recurso compartido de archivos o Azure Files con la extensión .dtsx. Si la ruta de acceso UNC especificada contiene algún espacio, coloque las comillas alrededor de la ruta de acceso completa.
- **/Conf[igFile]** : Especifica un archivo de configuración del que se van a extraer los valores. Con esta opción, puede establecer una configuración en tiempo de ejecución para el paquete que difiere de la especificada en tiempo de diseño. Puede almacenar valores diferentes en un archivo de configuración XML y, después, cargarlos antes de la ejecución del paquete. Para obtener más información, consulte [Configuraciones de paquetes de SSIS](https://docs.microsoft.com/sql/integration-services/packages/package-configurations?view=sql-server-2017). Para especificar el valor de esta opción, use la ruta de acceso UNC para el archivo de configuración en el sistema de archivos, recurso compartido de archivos o Azure Files con la extensión dtsConfig. Si la ruta de acceso UNC especificada contiene algún espacio, coloque las comillas alrededor de la ruta de acceso completa.
- **/Conn[ection]** : Especifica las cadenas de conexión para los administradores de conexiones existentes en el paquete. Con esta opción, puede establecer las cadenas de conexión en tiempo de ejecución para los administradores de conexiones existentes en el paquete que difieren de las especificadas en tiempo de diseño. Especifique el valor de esta opción de la siguiente manera: `connection_manager_name_or_id;connection_string [[;connection_manager_name_or_id;connection_string]...]`.
- **/Set**: Invalida la configuración de un parámetro, variable, propiedad, contenedor, proveedor de registro, enumerador Foreach o conexión en el paquete. Esta opción se puede especificar varias veces. Especifique el valor de esta opción de la siguiente manera: `property_path;value`. Por ejemplo, `\package.variables[counter].Value;1` invalida el valor de la variable `counter` como 1. Puede usar el asistente para la **configuración de paquetes** para buscar, copiar y pegar el valor de `property_path` para los elementos del paquete cuyo valor quiere invalidar. Para obtener más información, consulte [Asistente para la configuración de paquetes](https://docs.microsoft.com/sql/integration-services/packages/legacy-package-deployment-ssis).
- **/De[crypt]** : establece la contraseña de descifrado para el paquete que está configurado con el nivel de protección **EncryptAllWithPassword**/**EncryptSensitiveWithPassword**.

> [!NOTE]
> Al invocar AzureDTExec con nuevos valores para sus opciones, se genera una nueva canalización excepto para la opción **/De[cript]** .

## <a name="next-steps"></a>Pasos siguientes

Después de que las canalizaciones únicas con la actividad Ejecutar paquete de SSIS en ellas se generen y ejecuten cuando se invoca AzureDTExec, se pueden supervisar en el portal de Data Factory. También puede asignarles desencadenadores de Data Factory si desea orquestarlas o programarlas mediante Data Factory. Para obtener más información, consulte [Ejecución de paquetes de SSIS como actividades de Data Factory](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity).

> [!WARNING]
> Se espera que únicamente AzureDTExec use la canalización generada. Sus propiedades o parámetros podrían cambiar en el futuro, así que no debe modificarlos o reutilizarlos para otros fines. Las modificaciones podrían interrumpir AzureDTExec. Si esto ocurre, elimine la canalización. AzureDTExec genera una nueva canalización la próxima vez que se invoca.