---
title: Ejecución de paquetes SSIS desde SSDT
description: Aprenda a ejecutar paquetes SSIS en Azure desde SSDT.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.author: sawinark
author: swinarko
ms.reviewer: douglasl
manager: mflasko
ms.custom: seo-lt-2019
ms.date: 07/31/2019
ms.openlocfilehash: c3163d414e940d843489a34f319996b1b8ed6f4a
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86497414"
---
# <a name="execute-ssis-packages-in-azure-from-ssdt"></a>Ejecución de paquetes SSIS en Azure desde SSDT

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se describe la característica de los proyectos de SQL Server Integration Services (SSIS) habilitados para Azure en SQL Server Data Tools (SSDT), que permite ejecutar paquetes de Azure-SSIS Integration Runtime (IR) en Azure Data Factory (ADF).  Esta característica se puede usar para probar los paquetes SSIS existentes antes de migrarlos mediante lift-and-shift a Azure o desarrollar nuevos paquetes SSIS para ejecutarlos en Azure.

Con esta característica, puede crear un nuevo Azure-SSIS Integration Runtime o adjuntar uno existente a los proyectos de SSIS y, después, ejecutar los paquetes en él.  Admitimos la ejecución de paquetes que se van a implementar en el catálogo de SSIS (SSISDB) en el modelo de implementación de proyectos y los que se van a implementar en sistemas de archivos, recursos compartidos de archivos o Azure Files en el modelo de implementación de paquetes. 

## <a name="prerequisites"></a>Requisitos previos
Para usar esta característica, descargue e instale la extensión SSDT con proyectos de SSIS para Visual Studio más reciente desde [aquí](https://marketplace.visualstudio.com/items?itemName=SSIS.SqlServerIntegrationServicesProjects) o como un programa de instalación independiente desde [aquí](https://docs.microsoft.com/sql/ssdt/download-sql-server-data-tools-ssdt?view=sql-server-2017#ssdt-for-vs-2017-standalone-installer).

## <a name="azure-enable-ssis-projects"></a>Proyectos de SSIS habilitados para Azure
### <a name="create-new-azure-enabled-ssis-projects"></a>Creación de proyectos de SSIS habilitados para Azure
En SSDT, puede crear proyectos de SSIS habilitados para Azure mediante la plantilla **Proyecto de Integration Services (habilitado para Azure)** .

   ![Nuevo proyecto de SSIS habilitado para Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-new-project.png)

Una vez creado el proyecto habilitado para Azure, se le pedirá que se conecte a SSIS en Azure Data Factory.

   ![Símbolo del sistema de conexión de Azure-SSIS IR](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-integration-runtime-connect-prompt.png)

Si desea conectarse a su instancia de Azure-SSIS IR de inmediato, consulte [Conexión a Azure-SSIS IR](#irconnect) para obtener más información. También puede conectarse más tarde si hace clic con el botón derecho en el nodo del proyecto en el panel del Explorador de soluciones de SSDT para que aparezca un menú y, a continuación, selecciona el elemento de menú **Connect to SSIS in Azure Data Factory** (Conectar a SSIS en Azure Data Factory) del submenú **SSIS in Azure Data Factory** (SSIS en Azure Data Factory).

### <a name="azure-enable-existing-ssis-projects"></a>Proyectos de SSIS existentes habilitados para Azure
En el caso de los proyectos existentes de SSIS, puede habilitarlos con Azure si sigue estos pasos:

1. Haga clic con el botón derecho en el nodo del proyecto en el panel del Explorador de soluciones de SSDT para que aparezca un menú y, a continuación, seleccione el elemento de menú **Azure-Enabled Project** (Proyecto habilitado para Azure) en el submenú **SSIS en Azure Data Factory** para iniciar el **Asistente para proyectos habilitados para Azure**.

   ![Proyecto de SSIS existente habilitado para Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-for-existing-project.png)

2. En la página **Selección de la configuración de Visual Studio**, seleccione la configuración de Visual Studio para aplicar la configuración de ejecución de paquetes en Azure. Es recomendable s crear una configuración de Visual Studio para la nube y habilitar el proyecto en la configuración de la nube. Con varias configuraciones, puede asignar valores diferentes a los parámetros en función de los distintos entornos (tanto de forma local como en Azure). Para más información, consulte [este ejemplo](#example).

   ![Selección de la configuración de Visual Studio](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-select-visual-studio-configurations.png)

3. Habilitar en Azure los proyectos de SSIS existentes requiere que establezca la versión del servidor de destino para que sea la más reciente admitida por Azure-SSIS IR, que actualmente es **SQL Server 2017**. Por lo tanto, si todavía no lo ha hecho, debe comprobar si el paquete contiene componentes adicionales que no se admitan en SQL Server 2017 y hacer clic en el botón Siguiente para continuar, si no hay ningún problema.

   ![Cambio de la versión del servidor de destino](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-switch-target-server-version-step.png)

4. Consulte [Conexión a Azure-SSIS IR](#irconnect) para completar la conexión a Azure-SSIS IR.

## <a name="connect-azure-enabled-projects-to-ssis-in-azure-data-factory"></a><a name="irconnect"></a> Conexión de proyectos habilitados para SSIS en Azure Data Factory

Mediante la conexión de proyectos habilitados para Azure a SSIS en ADF, puede cargar sus paquetes en Azure Files y ejecutarlos en Azure-SSIS IR. Para ello, siga estos pasos:

1. En la página **SSIS en la introducción de ADF**, lea la introducción y haga clic en el botón **Siguiente** para continuar.

   ![SSIS en la introducción de ADF](media/how-to-invoke-ssis-package-ssdt/ssis-in-adf-connection-wizard.png)

2. En la página **Selección de SSIS IR en ADF**, seleccione los ADF y Azure-SSIS IR existentes para ejecutar paquetes o cree unos nuevos si no tiene ninguno.
   - Para seleccionar el Azure-SSIS IR existente, es preciso seleccionar antes el archivo de definición de aplicación y la suscripción a Azure relevante.
   - Si selecciona un archivo de definición de aplicación existente que no tenga ningún Azure-SSIS IR, haga clic en el botón**Crear SSIS IR** para crearlo en el portal o la aplicación de ADF.
   - Si selecciona una suscripción de Azure existente que no tenga ningún archivo de definición de aplicación, haga clic en el botón **Crear SSIS IR** para iniciar el **Asistente para la creación de Integration Runtime**, donde puede especificar la ubicación y prefijo para que se cree automáticamente un grupo de recursos de Azure, Data Factory y SSIS IR en su nombre, y para asignar el nombre se usa el siguiente patrón: **SuPrefijo-RG/DF/IR-SuHoraDeCreación**.

   ![Selección de SSIS IR en ADF](media/how-to-invoke-ssis-package-ssdt/ssis-in-adf-connection-wizard2.png)

3. En la página **Seleccionar Azure Storage** , seleccione su cuenta de Azure Storage existente para cargar paquetes en Azure Files, o si no tiene una cuenta, créela.
   - Para seleccionar la cuenta de Azure Storage existente, seleccione antes la suscripción a Azure relevante.
   - Si selecciona la misma suscripción de Azure que Azure-SSIS Integration Runtime que no tiene ninguna cuenta de Azure Storage, haga clic en el botón **Create Azure Storage** (Crear instancia de Azure Storage) para que se cree automáticamente una en su nombre en la misma ubicación que su Azure-SSIS IR, y para asignarle un nombre se combina un prefijo del nombre de Azure-SSIS Integration Runtime y su fecha de creación.
   - Si selecciona una suscripción de Azure diferente que no tiene ninguna cuenta de Azure Storage, haga clic en e,l botón **Create Azure Storage** (Crear instancia de Azure Storage) para crear una nueva en Azure Portal.

   ![Selección de Azure Storage](media/how-to-invoke-ssis-package-ssdt/ssis-in-adf-connection-wizard3.png)

4. Haga clic en el botón **Conectar** para completar la conexión.  En el panel del Explorador de soluciones de SSDT, se mostrará el Azure-SSIS IR y la cuenta de Azure Storage seleccionados en el nodo **Recursos de Azure vinculados** .  También se actualizará el estado de Azure-SSIS Integration Runtime, que puede administrar haciendo clic con el botón derecho en su nodo para que aparezca un menú y, después, seleccionando el elemento de menú **Start\Stop\Manage** (Iniciar\detener\administrar), que le lleva al portal o a la aplicación de ADF para hacerlo.

## <a name="assess-ssis-projectpackages-for-executions-in-azure"></a>Evaluación de proyecto o paquetes de SSIS para las ejecuciones en Azure
### <a name="assess-ssis-project-or-package"></a>Evaluación de un proyecto o paquete de SSIS
Antes de ejecutar los paquetes en Azure, puede evaluar el paquete para evaluar si hay algún elemento que pueda bloquear la migración o información que deba tener en cuenta. 
-  Puede evaluar todos los paquetes del proyecto, o bien un solo paquete.

   ![Evaluar proyecto](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assess-project.png)
   ![Evaluar paquete](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assess-package.png)

-  Puede obtener un informe de la valoración para comprobar todos los problemas y cada uno de ellos tendrá una descripción detallada y una recomendación. También puede exportar el informe de valoración en forma de archivo CSV. 

   ![Evaluar el resultado del proyecto](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assess-project-result.png)

### <a name="suppress-assessment-rule"></a>Supresión de reglas de valoración
Si está seguro de que no se ha aplicado alguna regla de valoración en su paquete, puede suprimirla. 
-  Puede hacer clic directamente en el vínculo **Configure Assessment Rule Suppression** (Configurar supresión de reglas de valoración) del informe Assessment (Valoración).

   ![Configuración de la supresión de reglas de valoración](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assessment-rule-suppression-settings.png)

-  También puede configurarlo desde **Azure-Enabled Settings** (Configuración habilitada para Azure).

   ![Configuración habilitada para Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-azure-enabled-setting.png)

   ![Configuración de supresión de reglas de valoración mediante la configuración habilitada para Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assessment-rule-suppression-settings-via-azure-enabled-settings.png)

## <a name="execute-ssis-packages-in-azure"></a>Ejecución de paquetes SSIS en Azure
### <a name="azure-enabled-setting"></a>Configuración habilitada para Azure
Antes de ejecutar los paquetes en Azure, puede optar por configurar las opciones de ejecución. Si desea habilitar la autenticación de Windows para los paquetes SSIS, siga estos pasos:

1. Haga clic con el botón derecho en el nodo del proyecto en el panel del Explorador de soluciones de SSDT para que aparezca un menú y, a continuación, seleccione el elemento de menú **Azure-Enabled Settings** (Configuración habilitada para Azure) en el submenú **SSIS in Azure Data Factory** (SSIS en Azure Data Factory).

   ![Configuración habilitada para Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-azure-enabled-setting.png)

2. Haga clic en la lista desplegable **Habilitar autenticación de Windows** y elija **true**. A continuación, haga clic en el botón Editar para la opción **Credenciales de autenticación de Windows** para escribir las credenciales.

   ![Habilite la autenticación de Windows](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-windows-authentication-open.png)

3. Proporcione las credenciales en el editor **Credenciales de autenticación de Windows**.

   ![Credenciales de autenticación de Windows](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-windows-authentication-credential.png)

### <a name="starting-package-executions"></a>Inicio de las ejecuciones del paquete
Después de conectar los proyectos a SSIS en ADF, puede ejecutar paquetes en Azure-SSIS IR.  Tiene dos opciones para iniciar las ejecuciones de paquetes:
-  Haga clic en el botón **Iniciar** de la barra de herramientas de SSDT para desplegar un menú y seleccione el elemento de menú **Execute in Azure** (Ejecutar en Azure) 

   ![Ejecutar en Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-execute-package.png)

-  Haga clic con el botón derecho en el nodo del paquete en el panel del Explorador de soluciones de SSDT para que aparezca un menú y seleccione el elemento de menú **Execute Package in Azure** (Ejecutar paquete en Azure).

   ![Ejecutar paquete en Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-execute-package2.png)

> [!NOTE]
> La ejecución de paquetes en Azure requiere que Azure-SSIS Integration Runtime esté en ejecución, por lo que si Azure-SSIS IR se detiene, aparecerá una ventana de diálogo para iniciarlo.  Sin incluir el tiempo de configuración personalizada, este proceso debería completarse en 5 minutos, pero puede que tarde aproximadamente de 20 a 30 minutos en unir Azure-SSIS IR a una red virtual.  Después de ejecutar los paquetes en Azure, puede detener Azure-SSIS Integration Runtime para administrar su costo de ejecución haciendo clic con el botón derecho en su nodo en el panel del Explorador de soluciones de SSDT para que aparezca un menú y, después, seleccionando el elemento de menú **Start\Stop\Manage** (Iniciar\detener\administrar), que le lleva al portal o a la aplicación de ADF para hacerlo.

### <a name="checking-package-execution-logs"></a>Comprobación de los registros de ejecución de paquetes
Al iniciar la ejecución del paquete, formatearemos y mostraremos su registro en la ventana de progreso de SSDT.  En el caso de un paquete de ejecución prolongada, actualizaremos periódicamente su registro.  Para detener la ejecución del paquete, haga clic en el botón **Detener** de la barra de herramientas de SSDT, que la cancelará de inmediato.  También puede encontrar temporalmente los datos sin procesar de registro en su ruta de acceso UNC (convención de nomenclatura universal): `\\<YourConnectedAzureStorage>.file.core.windows.net\ssdtexecution\<YourProjectName-FirstConnectTime>\<YourPackageName-tmp-ExecutionTime>\logs`, pero se limpiarán a las 24 horas.

### <a name="switching-package-protection-level"></a>Cambio del nivel de protección de paquetes
La ejecución de paquetes SSIS en Azure no admite los niveles de protección **EncryptSensitiveWithUserKey**/**EncryptAllWithUserKey**.  Por tanto, si los paquetes están configurados con ellos, los cambiaremos temporalmente a **EncryptSensitiveWithPassword**/**EncryptAllWithPassword**, respectivamente, y se generarán contraseñas de forma aleatoria cuando se cargan los paquetes en Azure Files para su ejecución en Azure-SSIS Integration Runtime.

> [!NOTE]
> Si los paquetes contienen tareas Ejecutar paquete que hacen referencia a otros paquetes configurados con los niveles de protección **EncryptSensitiveWithUserKey**/**EncryptAllWithUserKey**, debe volver a configurar manualmente los otros paquetes para que usen **EncryptSensitiveWithPassword**/**EncryptAllWithPassword**, respectivamente, antes de ejecutar los paquetes.

Si los paquetes ya están configurados con los niveles de protección **EncryptSensitiveWithPassword**/**EncryptAllWithPassword**, se mantendrán inalterados, pero se seguirán usando contraseñas generadas de forma aleatoria al cargar los paquetes en Azure Files para su ejecución en Azure-SSIS Integration Runtime.

### <a name="using-package-configuration-file"></a>Uso del archivo de configuración de paquetes
Si usa archivos de configuración de paquetes en el modelo de implementación de paquetes para cambiar los valores de las variables en tiempo de ejecución, esos archivos se cargarán automáticamente con los paquetes en Azure Files para su ejecución en Azure-SSIS Integration Runtime.

### <a name="using-execute-package-task"></a>Usar la tarea Ejecutar paquete
Si los paquetes contienen tareas Ejecutar paquete que hacen referencia a otros paquetes almacenados en sistemas de archivos locales, es preciso realizar las siguientes configuraciones adicionales:

1. Cargue los restantes paquetes en Azure Files, en la misma cuenta de Azure Storage que está conectada a los proyectos y obtenga su nueva ruta de acceso UNC, por ejemplo, `\\test.file.core.windows.net\ssdtexecution\Package1.dtsx`

2. Reemplace la ruta de acceso de archivo de los restantes paquetes en el administrador de conexiones de archivos de las tareas Ejecutar paquete por su nueva ruta de acceso UNC
   - Si el equipo que ejecuta SSDT no puede acceder a la nueva ruta de acceso UNC, puede cambiar la ruta de acceso de archivo en el panel Propiedades del administrador de conexiones de archivos
   - Como alternativa, puede usar una variable para la ruta de acceso de archivo para asignar el valor correcto en tiempo de ejecución

Si los paquetes contienen tareas Ejecutar paquete que hacen referencia a otros paquetes del mismo proyecto, no es necesario realizar ninguna configuración adicional.

## <a name="switching-package-execution-environments-with-azure-enabled-projects"></a><a name="example"></a> Cambio de los entornos de ejecución de paquetes con proyectos habilitados para Azure

Para cambiar los entornos de ejecución de paquetes con proyectos habilitados para Azure, puede crear varias configuraciones de Visual Studio y aplicar valores diferentes para los parámetros específicos del entorno. Por ejemplo, hay un paquete SSIS sencillo con una **tarea del sistema de archivos**  que establece los atributos de archivo especificado. Basta con migrar a la nube mediante los pasos siguientes:

1. Defina un parámetro **FilePath** de tipo cadena, que sea la ruta de acceso del archivo de destino.

   ![Parámetro de paquete secundario](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-example-define-parameters.png)

2. Asocie la **conexión de origen** con este parámetro. 

   ![Actualización de la conexión de origen](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-example-update-task-with-parameters.png)

3. Cree una nueva configuración de Visual Studio para la nube en Visual Studio Configuration Manager.

4. Defina los valores para este parámetro en cada configuración de Visual Studio.

   ![Invalidación de los valores de parámetros](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-example-override-parameter.png)

5. Se habilita este proyecto de SSIS en Azure con la configuración de Visual Studio para la nube.

6. Ejecute este paquete en Azure. Puede cambiar fácilmente el entorno a uno local si cambia la configuración actual de Visual Studio.

   ![Cambio a la configuración de Visual Studio](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-example-switch-configurations.png)

## <a name="current-limitations"></a>Limitaciones actuales
-  Actualmente, esta característica de SSDT no es compatible con la nube nacional.

## <a name="next-steps"></a>Pasos siguientes
Una vez que esté satisfecho con la ejecución de los paquetes en Azure desde SSDT, puede implementarlos y ejecutarlos como actividades de Ejecutar paquetes SSIS en las canalizaciones de ADF, consulte [Ejecución de paquetes de SSIS como actividades de Ejecutar paquete SSIS en canalizaciones de ADF](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity).
