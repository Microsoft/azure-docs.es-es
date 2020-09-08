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
ms.date: 08/10/2020
ms.openlocfilehash: 006b7db9f63f5ba74fee936383206b18c42aa038
ms.sourcegitcommit: 1a0dfa54116aa036af86bd95dcf322307cfb3f83
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88041874"
---
# <a name="execute-ssis-packages-in-azure-from-ssdt"></a>Ejecución de paquetes SSIS en Azure desde SSDT

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se describe la característica de los proyectos de SQL Server Integration Services (SSIS) habilitados para Azure en SQL Server Data Tools (SSDT). Permite evaluar la compatibilidad con la nube de los paquetes de SSIS y ejecutarlos en Azure-SSIS Integration Runtime (IR) en Azure Data Factory (ADF). Esta característica se puede usar para probar los paquetes existentes antes de migrarlos mediante lift-and-shift a Azure o desarrollar nuevos paquetes para ejecutarlos en Azure.

Con esta característica, puede conectar una instancia reciente o existente de Azure-SSIS Integration Runtime con proyectos de SSIS y, después, ejecutar los paquetes allí.  Se admite la ejecución de paquetes que se implementarán en el catálogo de SSIS (SSISDB) hospedados por un servidor de Azure SQL Database o Managed Instance en el modelo de implementación de proyectos. También se admite la ejecución de paquetes que se implementarán en el sistema de archivos, en Azure Files o en una base de datos de SQL Server (MSDB) hospedados por Azure SQL Managed Instance en el modelo de implementación de paquetes. 

## <a name="prerequisites"></a>Prerrequisitos

Para usar esta característica, descargue e instale la extensión SSDT con proyectos de SSIS para Visual Studio (VS) más reciente desde [aquí](https://marketplace.visualstudio.com/items?itemName=SSIS.SqlServerIntegrationServicesProjects). Como alternativa, también puede descargar e instalar la versión más reciente de SSDT como instalador independiente desde [aquí](https://docs.microsoft.com/sql/ssdt/download-sql-server-data-tools-ssdt?view=sql-server-2017#ssdt-for-vs-2017-standalone-installer).

## <a name="azure-enable-ssis-projects"></a>Proyectos de SSIS habilitados para Azure

### <a name="creating-new-azure-enabled-ssis-projects"></a>Creación de proyectos de SSIS habilitados para Azure

En SSDT, puede crear proyectos de SSIS habilitados para Azure mediante la plantilla **Proyecto de Integration Services (habilitado para Azure)** .

   ![Nuevo proyecto de SSIS habilitado para Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-new-project.png)

Una vez creado el proyecto habilitado para Azure, se le pedirá que se conecte a SSIS en Azure Data Factory.

   ![Símbolo del sistema de conexión de Azure-SSIS IR](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-integration-runtime-connect-prompt.png)

Si desea conectarse a su instancia de Azure-SSIS IR de inmediato, consulte [Conexión a Azure-SSIS IR](#connectssisir) para obtener más información. También puede conectarse más tarde si hace clic con el botón derecho en el nodo del proyecto en la ventana del Explorador de soluciones de SSDT para mostrar un menú. A continuación, seleccione el elemento **Connect to SSIS in Azure Data Factory** (Conectarse a SSIS en Azure Data Factory) en el submenú **SSIS en Azure Data Factory**.

### <a name="azure-enabling-existing-ssis-projects"></a><a name="azureenableproject"></a> Habilitación de proyectos de SSIS existentes para Azure

En el caso de los proyectos existentes de SSIS, puede habilitarlos con Azure si sigue estos pasos:

1. Haga clic con el botón derecho en el nodo del proyecto en la ventana Explorador de soluciones de SSDT para mostrar un menú. A continuación, seleccione el elemento **Proyecto habilitado para Azure** en el submenú **SSIS en Azure Data Factory** para iniciar el **Asistente para proyectos habilitados para Azure**.

   ![Proyecto de SSIS existente habilitado para Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-for-existing-project.png)

2. En la página **Selección de la configuración de Visual Studio**, seleccione la configuración de VS existente para aplicar la configuración de ejecución de paquetes en Azure. También puede crear una nueva si todavía no lo ha hecho, consulte [Creación de una nueva configuración de VS](https://docs.microsoft.com/visualstudio/ide/how-to-create-and-edit-configurations?view=vs-2019). Se recomienda tener al menos dos configuraciones de VS diferentes para las ejecuciones de paquetes en los entornos locales y de nube, de modo que pueda habilitar el proyecto para Azure en la configuración de la nube. De esta manera, si ha parametrizado el proyecto o los paquetes, puede asignar valores diferentes a los parámetros del paquete o proyecto en tiempo de ejecución en función de los distintos entornos de ejecución (en la máquina local o en Azure). Por ejemplo, consulte [Conmutación de los entornos de ejecución de paquetes](#switchenvironment).

   ![Selección de la configuración de Visual Studio](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-select-visual-studio-configurations.png)

3. Habilitar para Azure los proyectos de SSIS existentes requiere que establezca la versión del servidor de destino para que sea la más reciente admitida por Azure-SSIS IR. Actualmente, Azure-SSIS IR tiene un nivel de compatibilidad predeterminado de 140, que es igual a **SQL Server 2017**. Asegúrese de que los paquetes no contengan componentes adicionales que no se admitan en SQL Server 2017. Asegúrese también de que todos los componentes adicionales compatibles también se hayan instalado en Azure-SSIS IR a través de las configuraciones personalizadas, consulte [Personalización de Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/how-to-configure-azure-ssis-ir-custom-setup). Seleccione el botón **Siguiente** para continuar.

   ![Cambio de la versión del servidor de destino](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-switch-target-server-version-step.png)

4. Consulte [Conexión a Azure-SSIS IR](#connectssisir) para completar la conexión del proyecto a Azure-SSIS IR.

## <a name="connect-azure-enabled-projects-to-ssis-in-azure-data-factory"></a><a name="connectssisir"></a> Conexión de proyectos habilitados para SSIS en Azure Data Factory

Mediante la conexión de proyectos habilitados para Azure a SSIS en ADF, puede cargar sus paquetes en Azure Files y ejecutarlos en Azure-SSIS IR. Para hacerlo, siga estos pasos:

1. En la página **SSIS en la introducción de ADF**, lea la introducción y seleccione el botón **Siguiente** para continuar.

   ![SSIS en la introducción de ADF](media/how-to-invoke-ssis-package-ssdt/ssis-in-adf-connection-wizard.png)

2. En la página **Selección de SSIS IR en ADF**, seleccione los ADF y Azure-SSIS IR existentes para ejecutar paquetes. También puede crear otros nuevos si no tiene ninguno.
   - Para seleccionar el Azure-SSIS IR existente, es preciso seleccionar antes el archivo de definición de aplicación y la suscripción a Azure relevante.
   - Si selecciona un archivo de definición de aplicación existente que no tenga ninguna instancia de Azure-SSIS IR, seleccione el botón**Crear SSIS IR** para crearla en el portal de ADF. Una vez creado, puede volver a esta página para seleccionar la nueva instancia de Azure-SSIS IR.
   - Si selecciona una suscripción de Azure existente que no tiene ningún ADF, seleccione el botón **Crear SSIS IR** para iniciar el **Asistente para la creación de Integration Runtime**. En el asistente, puede especificar la ubicación y el prefijo designados para que podamos crear automáticamente un nuevo grupo de recursos de Azure, Data Factory y SSIS IR en su nombre, denominado con el siguiente patrón: **SuPrefijo-RG/DF/IR-SuHoraDeCreación**. Una vez creado, puede volver a esta página para seleccionar el nuevo ADF y Azure-SSIS IR.

   ![Selección de SSIS IR en ADF](media/how-to-invoke-ssis-package-ssdt/ssis-in-adf-connection-wizard2.png)

3. En la página **Seleccionar Azure Storage** , seleccione su cuenta de Azure Storage existente para cargar paquetes en Azure Files. También puede crear una nueva si no tiene ninguna.
   - Para seleccionar la cuenta de Azure Storage existente, seleccione antes la suscripción a Azure relevante.
   - Si selecciona la misma suscripción de Azure que la instancia de Azure-SSIS IR que no tiene ninguna cuenta de Azure Storage, seleccione el botón **Create Azure Storage** (Crear Azure Storage). Se creará automáticamente una nueva instancia en su nombre en la misma ubicación que su instancia de Azure-SSIS IR, denominada mediante la combinación de un prefijo del nombre de Azure-SSIS IR y la fecha de creación. Una vez creada, puede volver a esta página para seleccionar la nueva cuenta de Azure Storage.
   - Si selecciona una suscripción de Azure diferente que no tiene ninguna cuenta de Azure Storage, seleccione el botón **Create Azure Storage** (Crear Azure Storage) para crear una nueva instancia en Azure Portal. Una vez creada, puede volver a esta página para seleccionar la nueva cuenta de Azure Storage.

   ![Selección de Azure Storage](media/how-to-invoke-ssis-package-ssdt/ssis-in-adf-connection-wizard3.png)

4. Seleccione el botón **Conectar** para completar la conexión del proyecto a Azure-SSIS IR. En la ventana del Explorador de soluciones de SSDT, se mostrará la instancia de Azure-SSIS IR y la cuenta de Azure Storage seleccionadas en el nodo **Recursos de Azure vinculados** . También actualizaremos periódicamente y mostraremos allí el estado de la instancia de Azure-SSIS IR. Para administrar la instancia de Azure-SSIS IR, puede hacer clic con el botón derecho en su nodo para que aparezca un menú y, después, seleccionar el elemento **Start\Stop\Manage** (Iniciar\detener\administrar), que le lleva al portal de ADF para hacerlo.

## <a name="assess-ssis-projectpackages-for-executions-in-azure"></a>Evaluación de proyecto o paquetes de SSIS para las ejecuciones en Azure

### <a name="assessing-single-or-multiple-packages"></a>Evaluación de uno o varios paquetes

Antes de ejecutar los paquetes en Azure, puede evaluarlos para que surjan los posibles problemas de compatibilidad de la nube. Entre ellos se incluyen los bloqueadores de migración y la información adicional que deba tener en cuenta. 
-  Tiene las opciones para evaluar los paquetes individuales uno por uno o todos los paquetes al mismo tiempo en el proyecto.

   ![Evaluación de paquete](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assess-package.png)

   ![Evaluación del proyecto](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assess-project.png)

-  En la ventaja **Assessment Report** (Informe de evaluación) de SSDT, puede encontrar todos los posibles problemas de compatibilidad de la nube que surgen, cada uno con su propia descripción y recomendación. También puede exportar el informe de evaluación a un archivo CSV que se puede compartir con cualquier persona que deba mitigar estos problemas. 

   ![Informe de evaluación](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assess-project-result.png)

### <a name="suppressing-assessment-rules"></a>Supresión de las reglas de evaluación

Una vez que esté seguro de que algunos problemas potenciales de compatibilidad con la nube no correspondan o se hayan mitigado correctamente en los paquetes, puede suprimir las reglas de evaluación pertinentes que los exponen. Esto reducirá el ruido en los informes de evaluación posteriores.
-  Seleccione el vínculo **Configure Assessment Rule Suppression** (Configurar supresión de reglas de evaluación) en la ventana **Informe de evaluación** de SSDT para mostrar la ventana **Configuración de la supresión de reglas de valoración**, donde puede seleccionar las reglas de evaluación que se van a suprimir.

   ![Configuración de la supresión de reglas de valoración](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assessment-rule-suppression-settings.png)

-  Como alternativa, haga clic con el botón derecho en el nodo del proyecto en la ventana Explorador de soluciones de SSDT para mostrar un menú. Seleccione el elemento **Configuración habilitada para Azure** en el submenú **SSIS en Azure Data Factory** para que aparezca una ventana que contenga las páginas de propiedades del proyecto. Seleccione la propiedad **Suppressed Assessment Rule IDs** (Id. de reglas de evaluación suprimidas) en **Configuración habilitada para Azure**. Por último, seleccione el botón de puntos suspensivos ( **…** ) para mostrar la ventana de **Configuración de la supresión de reglas de valoración**, donde puede seleccionar las reglas de evaluación que se van a suprimir.

   ![Configuración habilitada para Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-azure-enabled-setting.png)

   ![Configuración de supresión de reglas de valoración mediante la configuración habilitada para Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-assessment-rule-suppression-settings-via-azure-enabled-settings.png)

## <a name="execute-ssis-packages-in-azure"></a>Ejecución de paquetes SSIS en Azure

### <a name="configuring-azure-enabled-settings"></a><a name="azureenabledsettings"></a> Opciones de configuración habilitada para Azure

Antes de ejecutar los paquetes en Azure, puede establecer la configuración habilitada para Azure para ellos. Por ejemplo, puede habilitar la autenticación de Windows en su instancia de Azure-SSIS IR para acceder a almacenes de datos locales o en la nube siguiendo estos pasos:

1. Haga clic con el botón derecho en el nodo del proyecto en la ventana Explorador de soluciones de SSDT para mostrar un menú. A continuación, seleccione el elemento **Configuración habilitada para Azure** en el submenú **SSIS en Azure Data Factory** para que aparezca una ventana que contenga las páginas de propiedades del proyecto.

   ![Configuración habilitada para Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-azure-enabled-setting.png)

2. Seleccione la propiedad **Habilitar autenticación de Windows** en **Configuración habilitada para Azure** y, a continuación, seleccione **Verdadero** en el menú desplegable. A continuación, seleccione la propiedad **credenciales de autenticación de Windows** y, a continuación, seleccione el botón de puntos suspensivos ( **...** ) para mostrar la ventana **Credenciales de autenticación de Windows**.

   ![Habilite la autenticación de Windows](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-windows-authentication-open.png)

3. Escriba las credenciales de autenticación de Windows. Por ejemplo, para tener acceso a Azure Files, puede especificar `Azure`, `YourStorageAccountName` y `YourStorageAccountKey` para **Dominio**, **Nombre de usuario** y **Contraseña**, respectivamente.

   ![Credenciales de autenticación de Windows](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-windows-authentication-credential.png)

### <a name="starting-package-executions"></a>Inicio de las ejecuciones del paquete

Después de conectar los proyectos habilitados para Azure a SSIS en ADF, evaluar la compatibilidad con la nube y mitigar los posibles problemas, puede ejecutar o probar los paquetes en Azure-SSIS IR.
-  Seleccione el botón **Iniciar** en la barra de herramientas de SSDT para desplegar un menú. A continuación, seleccione el elemento **Ejecutar en Azure**.

   ![Ejecutar en Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-execute-package.png)

-  Como alternativa, haga clic con el botón derecho en el nodo del paquete en la ventana Explorador de soluciones de SSDT para mostrar un menú. A continuación, seleccione el elemento **Ejecutar paquete en Azure**.

   ![Ejecutar paquete en Azure](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-execute-package2.png)

> [!NOTE]
> La ejecución de paquetes en Azure requiere que Azure-SSIS Integration Runtime esté en ejecución, por lo que si Azure-SSIS IR se detiene, aparecerá una ventana de diálogo para iniciarlo. Sin incluir el tiempo de configuración personalizada, este proceso debería completarse en 5 minutos, pero puede que tarde aproximadamente de 20 a 30 minutos en unir Azure-SSIS IR a una red virtual. Después de ejecutar los paquetes en Azure, puede detener Azure-SSIS IR para administrar su costo de ejecución haciendo clic con el botón derecho en su nodo en la ventana del Explorador de soluciones de SSDT para que aparezca un menú y, después, seleccionando el elemento **Start\Stop\Manage** (Iniciar\detener\administrar), que le lleva al portal de ADF para hacerlo.

### <a name="using-execute-package-task"></a>Usar la tarea Ejecutar paquete

Si los paquetes contienen tareas Ejecutar paquete que hacen referencia a paquetes secundarios almacenados en sistemas de archivos locales, es siga los pasos adicionales a continuación:

1. Cargue los paquetes secundarios en Azure Files, en la misma cuenta de Azure Storage que está conectada a los proyectos, y obtenga su nueva ruta de acceso de convención de nomenclatura universal (UNC), por ejemplo, `\\YourStorageAccountName.file.core.windows.net\ssdtexecution\YourChildPackage1.dtsx`.

2. Reemplace la ruta de acceso de archivo de los paquetes secundarios en el administrador de conexiones de archivos de las tareas Ejecutar paquete por su nueva ruta de acceso UNC.
   - Si la máquina local que ejecuta SSDT no puede acceder a la nueva ruta de acceso UNC, puede indicarla en el panel Propiedades del administrador de conexiones de archivos.
   - Como alternativa, puede usar una variable para la ruta de acceso de archivo para asignar el valor correcto en tiempo de ejecución.

Si los paquetes contienen tareas Ejecutar paquete que hacen referencia a paquetes secundarios del mismo proyecto, no es necesario completar ningún paso adicional.

### <a name="switching-package-protection-level"></a>Cambio del nivel de protección de paquetes

La ejecución de paquetes SSIS en Azure no admite los niveles de protección **EncryptSensitiveWithUserKey**/**EncryptAllWithUserKey**. Por lo tanto, si los paquetes están configurados para usarlos, se convertirán a aquellos de forma temporal mediante los niveles de protección **EncryptSensitiveWithPassword**/**EncryptAllWithPassword**, respectivamente. También se generarán aleatoriamente contraseñas de cifrado cuando se carguen los paquetes en Azure Files para las ejecuciones en la instancia de Azure-SSIS IR.

> [!NOTE]
> Si los paquetes contienen tareas Ejecutar paquete que hacen referencia a paquetes secundarios configurados para usar los niveles de protección **EncryptSensitiveWithUserKey**/**EncryptAllWithUserKey**, debe volver a configurar manualmente esos paquetes secundarios para que usen los niveles de protección **EncryptSensitiveWithPassword**/**EncryptAllWithPassword**, respectivamente, antes de ejecutar los paquetes.

Si los paquetes ya están configurados para usar los niveles de protección de datos **EncryptSensitiveWithPassword**/**EncryptAllWithPassword**, no se modificarán. Aún se generarán aleatoriamente contraseñas de cifrado cuando se carguen los paquetes en Azure Files para las ejecuciones en la instancia de Azure-SSIS IR.

### <a name="switching-package-execution-environments"></a><a name="switchenvironment"></a> Conmutación de los entornos de ejecución de paquetes

Si parametriza el proyecto o los paquetes en el modelo de implementación de proyectos, puede crear varias configuraciones de VS para conmutar los entornos de ejecución de paquetes. De esta manera, puede asignar valores específicos del entorno a los parámetros del paquete o proyecto en tiempo de ejecución. Se recomienda tener al menos dos configuraciones de VS diferentes para las ejecuciones de paquetes en los entornos locales y de nube, de modo que pueda habilitar los proyectos para Azure en la configuración de la nube. Este es un ejemplo paso a paso de cómo conmutar los entornos de ejecución de paquetes entre la máquina local y Azure:

1. Supongamos que el paquete contiene una tarea Sistema de archivos que establece los atributos de un archivo. Al ejecutarlo en la máquina local, establece los atributos de un archivo almacenado en el sistema de archivos local. Al ejecutarlo en Azure-SSIS IR, quiere establecer los atributos de un archivo almacenado en Azure Files. En primer lugar, cree un parámetro de paquete de tipo cadena y asígnele el nombre **FilePath** para contener el valor de la ruta de acceso del archivo de destino.

   ![Creación de parámetros del paquete](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-example-define-parameters.png)

2. A continuación, en la página **General** de la ventana **Editor de la tarea Sistema de archivos**, parametrice la propiedad **SourceVariable** en la sección **Conexión de origen** con el parámetro de paquete **FilePath**. 

   ![Parametrización de la conexión de origen](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-example-update-task-with-parameters.png)

3. De forma predeterminada, tiene una configuración de VS existente para las ejecuciones de paquetes en el entorno local denominada **Development**. Cree una nueva configuración de VS para las ejecuciones de paquetes en el entorno de nube denominada **Azure**, consulte [Creación de una nueva configuración de VS](https://docs.microsoft.com/visualstudio/ide/how-to-create-and-edit-configurations?view=vs-2019), si todavía no lo ha hecho.

4. Al ver los parámetros del paquete, seleccione el botón **Agregar parámetros a las configuraciones** para abrir la ventana **Administrar valores de parámetro** para el paquete. A continuación, asigne distintos valores de ruta de acceso del archivo de destino al parámetro de paquete **FilePath** en las configuraciones **Development** y **Azure**.

   ![Asignación de valores de parámetro](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-example-override-parameter.png)

5. Habilite su proyecto para Azure con la configuración de la nube, consulte [Habilitación de proyectos de SSIS existentes para Azure](#azureenableproject), si aún no lo ha hecho. Después, establezca la configuración habilitada para Azure para habilitar la autenticación de Windows para que el Azure-SSIS IR tenga acceso a Azure Files, consulte [Opciones de configuración habilitada para Azure](#azureenabledsettings), si aún no lo ha hecho.

6. Ejecute el paquete en Azure. Puede volver a cambiar el entorno de ejecución de paquetes a la máquina local si selecciona la configuración **Development**.

   ![Cambio a la configuración de Visual Studio](media/how-to-invoke-ssis-package-ssdt/ssdt-azure-enabled-example-switch-configurations.png)

### <a name="using-package-configuration-file"></a>Uso del archivo de configuración de paquetes

Si usa archivos de configuración de paquetes en el modelo de implementación de paquetes, puede asignar valores específicos del entorno a las propiedades del paquete en tiempo de ejecución. Esos archivos se cargarán automáticamente con los paquetes en Azure Files para las ejecuciones en su instancia de Azure-SSIS IR.

### <a name="checking-package-execution-logs"></a>Comprobación de los registros de ejecución de paquetes

Después de iniciar la ejecución del paquete, formatearemos y mostraremos sus registros en la ventana **Progreso** de SSDT. En el caso de un paquete de ejecución prolongada, actualizaremos periódicamente sus registros. Para cancelar de inmediato la ejecución del paquete, puede seleccionar el botón **Detener** de la barra de herramientas de SSDT. También puede buscar temporalmente los datos sin procesar de sus registros en la siguiente ruta de acceso UNC: `\\<YourStorageAccountName>.file.core.windows.net\ssdtexecution\<YourProjectName-FirstConnectTime>\<YourPackageName-tmp-ExecutionTime>\logs`, pero la limpiaremos después de un día.

## <a name="current-limitations"></a>Limitaciones actuales

-  SSDT habilitado para Azure solo es compatible con las regiones de nubes comerciales/globales y, por ahora, no es compatible con las regiones de nubes gubernamentales/nacionales.

## <a name="next-steps"></a>Pasos siguientes

Una vez que esté satisfecho con la ejecución de los paquetes en Azure desde SSDT, puede implementarlos y ejecutarlos como actividades de Ejecutar paquetes SSIS en las canalizaciones de ADF, consulte [Ejecución de paquetes de SSIS como actividades de Ejecutar paquete SSIS en canalizaciones de ADF](https://docs.microsoft.com/azure/data-factory/how-to-invoke-ssis-package-ssis-activity).
