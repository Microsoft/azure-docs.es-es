---
title: Uso de Herramientas de Azure Data Lake para Visual Studio Code
description: Obtenga información sobre cómo usar Herramientas de Azure Data Lake para Visual Studio Code para crear, probar y ejecutar scripts U-SQL.
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: dc9b21d8-c5f4-4f77-bcbc-eff458f48de2
ms.topic: how-to
ms.date: 02/09/2018
ms.openlocfilehash: b554869864ff71fb31b6c4e2b891fd42c4a380b1
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87126489"
---
# <a name="use-azure-data-lake-tools-for-visual-studio-code"></a>Uso de Herramientas de Azure Data Lake para Visual Studio Code

En este artículo, obtendrá información sobre cómo usar Herramientas de Azure Data Lake para Visual Studio Code (VS Code) para crear, probar y ejecutar scripts U-SQL. La información también se explica en el vídeo siguiente:

[![Reproductor de vídeo: Herramientas de Azure Data Lake para VS Code](media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-video.png)](https://channel9.msdn.com/Series/AzureDataLake/Azure-Data-Lake-Tools-for-VSCode?term=ADL%20Tools%20for%20VSCode")

## <a name="prerequisites"></a>Requisitos previos

Herramientas de Azure Data Lake para VS Code es compatible con Windows, Linux y macOS. La ejecución y depuración locales de U-SQL solo funcionan en Windows.

- [Visual Studio Code](https://www.visualstudio.com/products/code-vs.aspx)

En MacOS y Linux:

- [.NET Core SDK 2.0](https://www.microsoft.com/net/download/core)
- [Mono 5.2.x](https://www.mono-project.com/download/)

## <a name="install-azure-data-lake-tools"></a>Instalación de las herramientas de Azure Data Lake

Después de instalar los requisitos previos, puede instalar Herramientas de Azure Data Lake para VS Code.

### <a name="to-install-azure-data-lake-tools"></a>Para instalar Herramientas de Azure Data Lake

1. Abra Visual Studio Code.
2. En el panel izquierdo, seleccione **Extensiones**. Escriba **Herramientas de Azure Data Lake** en el cuadro de búsqueda.
3. Seleccione **Instalar** junto a **Herramientas de Azure Data Lake**.

   ![Selecciones para la instalación de Herramientas de Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extensions.png)

   Después de unos segundos, el botón **Instalar** cambia a **Recargar**.
4. Seleccione **Recargar** para activar la extensión **Herramientas de Azure Data Lake**.
5. Haga clic en **Recargar ventana** para confirmar. Puede ver **Herramientas de Azure Data Lake** en el panel **Extensiones**.

## <a name="activate-azure-data-lake-tools"></a>Activación de Azure Data Lake Tools

Cree un archivo .usql o abra uno ya existente para activar la extensión.

## <a name="work-with-u-sql"></a>Trabajo con U-SQL

Para trabajar con U-SQL, es preciso abrir un archivo U-SQL o una carpeta.

### <a name="to-open-the-sample-script"></a>Para abrir el script de ejemplo

Abra la paleta de comandos (Ctrl+Mayús+P) y escriba **ADL: Open Sample Script**. Se abre otra instancia de este ejemplo. En esta instancia también puede editar, configurar y enviar un script.

### <a name="to-open-a-folder-for-your-u-sql-project"></a>Para abrir una carpeta para el proyecto U-SQL

1. En Visual Studio Code, seleccione el menú **Archivo** y, luego, seleccione **Abrir carpeta**.
2. Especifique una carpeta y, luego, seleccione **Seleccionar carpeta**.
3. Seleccione el menú **Archivo** y, luego, seleccione **Nuevo**. Se agregará un archivo Sin título-1 al proyecto.
4. Escriba el código siguiente en el archivo Sin título-1:

   ```usql
   @departments  =
       SELECT * FROM
           (VALUES
               (31,    "Sales"),
               (33,    "Engineering"),
               (34,    "Clerical"),
               (35,    "Marketing")
           ) AS
                 D( DepID, DepName );
   ```

   OUTPUT @departments     TO "/Output/departments.csv" USING Outputters.Csv();

    El script crea un archivo departments.csv con algunos datos incluidos en la carpeta /output.

5. Guarde el archivo como **myUSQL.usql** en la carpeta abierta.

### <a name="to-compile-a-u-sql-script"></a>Para compilar un script U-SQL

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: Compile Script**. Los resultados de la compilación aparecen en la ventana **Salida**. También puede hacer clic con el botón derecho en un archivo de script y, luego, seleccionar **ADL: Compile Script** para compilar un trabajo de U-SQL. El resultado de la compilación aparece en el panel **Salida**.

### <a name="to-submit-a-u-sql-script"></a>Para enviar un script de U-SQL

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: Submit Job**. También puede hacer clic con el botón derecho en un archivo de script y, luego, seleccionar **ADL: Submit Job**.

Después de enviar un trabajo de U-SQL, los registros de envío aparecen en la ventana **Salida** de VS Code. La vista del trabajo aparece en el panel derecho. Si el envío se realiza correctamente, la dirección URL del trabajo también se muestra. Puede abrir la dirección URL del trabajo en un explorador web para realizar el seguimiento de estado del trabajo en tiempo real.

En la pestaña **Resumen** de la vista del trabajo puede ver los detalles del trabajo. Las funciones principales incluyen la opción de reenviar un script, duplicarlo o abrirlo en el portal. En la pestaña **Datos** de la vista del trabajo puede consultar los archivos de entrada, los archivos de salida y los archivos de recursos. Los archivos se pueden descargar en el equipo local.

![Pestaña Resumen en la vista del trabajo](./media/data-lake-analytics-data-lake-tools-for-vscode/job-view-summary.png)

![Pestaña Datos en la vista del trabajo](./media/data-lake-analytics-data-lake-tools-for-vscode/job-view-data.png)

### <a name="to-set-the-default-context"></a>Para establecer el contexto predeterminado

Si no estableció los parámetros en sus archivos individualmente, puede establecer el contexto predeterminado para aplicar esta configuración a todos los archivos de script.

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: Set Default Context**. Haga clic con el botón derecho en el editor de scripts y seleccione **ADL: Set Default Context**.
3. Elija la cuenta, la base de datos y el esquema que desee. La configuración se guarda en el archivo de configuración xxx_settings.json.

   ![Cuenta, base de datos y esquema establecidos como el contexto predeterminado](./media/data-lake-analytics-data-lake-tools-for-vscode/default-context-sequence.png)

### <a name="to-set-script-parameters"></a>Para establecer los parámetros del script

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: Set Script Parameters**.
3. Se abre el archivo xxx_settings.json con las propiedades siguientes:

   - **account**: una cuenta de Azure Data Lake Analytics en la suscripción de Azure que se necesita para compilar y ejecutar los trabajos de U-SQL. Debe configurar la cuenta del equipo antes de compilar y ejecutar trabajos de U-SQL.
   - **database**: una base de datos de la cuenta. El valor predeterminado es **master**.
   - **schema**: un esquema de la base de datos. El valor predeterminado es **dbo**.
   - **optionalSettings**:
        - **priority**: el intervalo de prioridades va de 1 a 1000, siendo 1 la prioridad más alta. El valor predeterminado es **1000**.
        - **degreeOfParallelism**: el intervalo de paralelismo va de 1 a 150. El valor predeterminado es el paralelismo máximo que se permite en su cuenta de Azure Data Lake Analytics.

   ![Contenido del archivo JSON](./media/data-lake-analytics-data-lake-tools-for-vscode/default-context-setting.png)

> [!NOTE]
> Una vez guardada la configuración, la información de la cuenta, de la base de datos y del esquema aparece en la barra de estado que se encuentra en la esquina inferior izquierda del archivo .usql correspondiente, si no tiene configurado un contexto predeterminado.

### <a name="to-set-git-ignore"></a>Para establecer Git ignore

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: Set Git Ignore**.

   - Si el archivo **.gitIgnore** no existe en la carpeta de trabajo de VS Code, se creará un archivo llamado **.gitIgnore** en la carpeta. De forma predeterminada, se agregarán al archivo cuatro elementos (**usqlCodeBehindReference**, **usqlCodeBehindGenerated**, **.cache** y **obj**). Puede realizar más actualizaciones si es necesario.
   - Si ya hay un archivo **.gitIgnore** en la carpeta de trabajo de VS Code, la herramienta agregará cuatro elementos (**usqlCodeBehindReference**, **usqlCodeBehindGenerated**, **.cache** y **obj**) al archivo **.gitIgnore** si los cuatro elementos no se encuentran en el archivo.

   ![Elementos del archivo .gitIgnore](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-gitignore.png)

## <a name="work-with-code-behind-files-c-sharp-python-and-r"></a>Trabajo con archivos de código subyacente: C#, Python y R

Herramientas de Azure Data Lake admite varios códigos personalizados. Para obtener instrucciones, consulte [Desarrollo de trabajos U-SQL con Python, R y CSharp para Azure Data Lake Analytics en VS Code](data-lake-analytics-u-sql-develop-with-python-r-csharp-in-vscode.md).

## <a name="work-with-assemblies"></a>Trabajo con ensamblados

Para información sobre el desarrollo de ensamblados, consulte [Desarrollo de ensamblados U-SQL para trabajos de Azure Data Lake Analytics](data-lake-analytics-u-sql-develop-assemblies.md).

Puede usar Herramientas de Data Lake para registrar ensamblados de código personalizados en el catálogo de Data Lake Analytics.

### <a name="to-register-an-assembly"></a>Para registrar un ensamblado

Puede registrar el ensamblado mediante el comando **ADL: Register Assembly** o **ADL: Register Assembly (Advanced)** .

### <a name="to-register-through-the-adl-register-assembly-command"></a>Para realizar el registro mediante el comando ADL: Register Assembly

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: Register Assembly**.
3. Especifique la ruta de acceso de ensamblado local.
4. Seleccione una cuenta de Data Lake Analytics.
5. Seleccione una base de datos.

El portal se abre en un explorador y muestra el proceso de registro del ensamblado.  

Una manera más cómoda de desencadenar el comando **ADL: Register Assembly** es hacer clic con el botón derecho en el archivo .dll en el Explorador de archivos.

### <a name="to-register-through-the-adl-register-assembly-advanced-command"></a>Para realizar el registro mediante el comando ADL: Register Assembly (Advanced)

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: Register Assembly (Advanced)**
3. Especifique la ruta de acceso de ensamblado local.
4. Se muestra el archivo JSON. Revise y edite las dependencias de ensamblado y los parámetros de recurso si es necesario. Las instrucciones se muestran en la ventana **Salida**. Para continuar con el registro del ensamblado, guarde (Ctrl+S) el archivo JSON.

   ![Archivo JSON con las dependencias de ensamblado y los parámetros de recursos](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-register-assembly-advance.png)

>[!NOTE]
>
>- Herramientas de Azure Data Lake detecta automáticamente si el archivo DLL tiene alguna dependencia de ensamblado. Las dependencias se muestran en el archivo JSON una vez que se detectan.
>- Puede cargar los recursos DLL (por ejemplo, .txt, .png y .csv) como parte del registro del ensamblado.

Otra manera de desencadenar el comando **ADL: Register Assembly (Advanced)** es hacer clic con el botón derecho en el archivo .dll en el Explorador de archivos.

El siguiente código de U-SQL muestra cómo llamar a un ensamblado. En el ejemplo, el nombre del ensamblado es *test*.

```usql
REFERENCE ASSEMBLY [test];
@a =
    EXTRACT
        Iid int,
    Starts DateTime,
    Region string,
    Query string,
    DwellTime int,
    Results string,
    ClickedUrls string
    FROM @"Sample/SearchLog.txt"
    USING Extractors.Tsv();
@d =
    SELECT DISTINCT Region
    FROM @a;
@d1 =
    PROCESS @d
    PRODUCE
        Region string,
    Mkt string
    USING new USQLApplication_codebehind.MyProcessor();
OUTPUT @d1
    TO @"Sample/SearchLogtest.txt"
    USING Outputters.Tsv();
```

## <a name="use-u-sql-local-run-and-local-debug-for-windows-users"></a>Uso de la ejecución y depuración locales de U-SQL para usuarios de Windows

La ejecución local de U-SQL prueba los datos locales y valida el script de manera local antes de publicar el código en Data Lake Analytics. Puede usar la característica de depuración local para completar las tareas siguientes antes de enviar el código a Data Lake Analytics:

- Depure el código subyacente de C#.
- Recorra el código.
- Valide localmente el script.

Las características de ejecución y depuración locales solo funcionan en entornos Windows y no se admiten en macOS y sistemas operativos basados en Linux.

Para instrucciones sobre la ejecución y la depuración locales, consulte [Ejecución y depuración locales de U-SQL con Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md).

## <a name="connect-to-azure"></a>Conexión con Azure

Para poder compilar y ejecutar scripts de U-SQL en Data Lake Analytics, debe conectarse a su cuenta de Azure.

<a id="sign-in-by-command"></a>

### <a name="to-connect-to-azure-by-using-a-command"></a>Para conectarse a Azure mediante un comando

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.

2. Escriba **ADL: Login** (HDInsight: inicio de sesión). La información de inicio de sesión aparece en la esquina inferior derecha.

   ![Entrada del comando de inicio de sesión](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extension-login.png)

   ![Notificaciones sobre el inicio de sesión y la autenticación](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-login-info.png)

3. Seleccione **Copiar y abrir** para abrir la [página de inicio de sesión](https://aka.ms/devicelogin). Pegue el código en el cuadro de texto y seleccione **Continuar**.

    ![Página de inicio de sesión](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extension-login-paste-code.png)  

4. Siga las instrucciones para iniciar sesión desde la página web. Cuando se conecte, el nombre de la cuenta de Azure aparecerá en la barra de estado que se encuentra en la esquina inferior izquierda de la ventana de VS Code.

> [!NOTE]
>
> - Herramientas de Data Lake iniciará sesión automáticamente la próxima vez si no cierra la sesión.
> - Si la cuenta tiene habilitada la autenticación de dos pasos, se recomienda usar la autenticación por teléfono en lugar de usar un PIN.

Para cerrar la sesión, escriba el comando **ADL: Logout**.

### <a name="to-connect-to-azure-from-the-explorer"></a>Para conectarse a Azure desde el explorador

Expanda **Azure Data Lake**, seleccione **Iniciar sesión en Azure** y, a continuación, siga los pasos 3 y 4 de [Para conectarse a Azure mediante un comando](#sign-in-by-command).

![Selección de "Iniciar sesión en Azure" en el explorador](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-sign-in-from-explorer.png )  

No se puede cerrar la sesión desde el explorador. Para cerrar sesión, consulte [Para conectarse a Azure mediante un comando](#sign-in-by-command).

## <a name="create-an-extraction-script"></a>Creación de un script de extracción

Puede crear un script de extracción para los archivos .csv, .tsv y .txt mediante el comando **ADL: Create EXTRACT Script** o desde el explorador de Azure Data Lake.

### <a name="to-create-an-extraction-script-by-using-a-command"></a>Para crear un script de extracción mediante un comando

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos y escriba **ADL: Create EXTRACT Script**.
2. Especifique la ruta de acceso completa de un archivo de Azure Storage y presione Entrar.
3. Seleccione una cuenta.
4. Para un archivo .txt, seleccione un delimitador para extraer el archivo.

![Proceso de creación de un script de extracción](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-process.png)

El script de extracción se genera en función de sus entradas. Para un script que no puede detectar las columnas, elija una de las dos opciones. Si no lo hace así, se generará un solo script.

![Resultado de la creación de un script de extracción](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-result.png)

### <a name="to-create-an-extraction-script-from-the-explorer"></a>Para crear un script de extracción desde el explorador

Otra forma de crear un script de extracción es haciendo clic con el botón derecho (menú contextual) en el archivo .csv, .tsv o .txt en Azure Data Lake Store o Azure Blob Storage.

![Comando "Create EXTRACT Script" en el menú contextual](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-from-context-menu.png)

## <a name="integrate-with-azure-data-lake-analytics-through-a-command"></a>Integración con Azure Data Lake Analytics mediante un comando

Puede acceder a los recursos de Azure Data Lake Analytics para enumerar cuentas, acceder a los metadatos y ver los trabajos de análisis.

### <a name="to-list-the-azure-data-lake-analytics-accounts-under-your-azure-subscription"></a>Para enumerar las cuentas de Azure Data Lake Analytics en su suscripción a Azure

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: List Accounts**. Las cuentas aparecen en el panel **Salida**.

### <a name="to-access-azure-data-lake-analytics-metadata"></a>Para acceder a los metadatos de Azure Data Lake Analytics

1. Seleccione Ctrl+Mayús+P y, luego, escriba **ADL: List Tables**.
2. Seleccione una de las cuentas de Data Lake Analytics.
3. Seleccione una de las bases de datos de Data Lake Analytics.
4. Seleccione uno de los esquemas. Puede ver la lista de tablas.

### <a name="to-view-azure-data-lake-analytics-jobs"></a>Para ver los trabajos de Azure Data Lake Analytics

1. Abra la paleta de comandos (Ctrl+Mayús+P) y seleccione **ADL: Show Jobs**.
2. Seleccione una cuenta de Data Lake Analytics o una cuenta local.
3. Espere a que se muestre la lista de trabajos de la cuenta.
4. Seleccione un trabajo de la lista de trabajos. Herramientas de Data Lake abre la vista del trabajo en el panel derecho y muestra alguna información en la salida de VS Code.

    ![Lista de trabajos](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-show-job.png)

## <a name="integrate-with-azure-data-lake-store-through-a-command"></a>Integración con Azure Data Lake Store mediante un comando

Puede usar comandos relacionados con Azure Data Lake Store para:

- [Examinar los recursos de Azure Data Lake Store](#list-the-storage-path)
- [Obtener una vista previa del archivo de Azure Data Lake Store](#preview-the-storage-file)
- Cargar el archivo directamente en Azure Data Lake Store en VS Code
- Descargar el archivo directamente de Azure Data Lake Store en VS Code

### <a name="list-the-storage-path"></a>Enumeración de la ruta de acceso de almacenamiento

### <a name="to-list-the-storage-path-through-the-command-palette"></a>Para enumerar la ruta de acceso de almacenamiento a través de la paleta de comandos

1. Haga clic con el botón derecho en el editor de scripts y seleccione **ADL: List Path**.
2. Elija la carpeta en la lista o seleccione **Indicar ruta de acceso** o **Examinar desde ruta de acceso raíz**. (Usamos **Indicar ruta de acceso** como un ejemplo).
3. Seleccione la cuenta de Data Lake Analytics.
4. Busque o escriba la ruta de acceso de la carpeta de almacenamiento (por ejemplo, /output/).  

En la paleta de comandos se muestra la información de la ruta de acceso en función de las entradas.

![Resultados de la ruta de acceso de almacenamiento](./media/data-lake-analytics-data-lake-tools-for-vscode/list-storage-path.png)

Una manera más cómoda de enumerar la ruta de acceso relativa es mediante el menú contextual.

### <a name="to-list-the-storage-path-through-the-shortcut-menu"></a>Para enumerar la ruta de acceso de almacenamiento mediante el menú contextual

Haga clic con el botón derecho en la cadena de la ruta de acceso y seleccione **List Path** (Enumerar ruta de acceso).

!["List Path" en el menú contextual](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-right-click-path.png)

### <a name="preview-the-storage-file"></a>Vista previa del archivo de almacenamiento

1. Haga clic con el botón derecho en el editor de scripts y seleccione **ADL: Preview File**.
2. Seleccione la cuenta de Data Lake Analytics.
3. Escriba una ruta de acceso del archivo de Azure Storage (por ejemplo, /output/SearchLog.txt).

El archivo se abre en VS Code.

![Pasos y resultado para obtener una vista previa del archivo de almacenamiento](./media/data-lake-analytics-data-lake-tools-for-vscode/preview-storage-file.png)

Otra manera de obtener una vista previa del archivo es con el menú contextual en la ruta de acceso completa o relativa del archivo en el editor de scripts.

### <a name="upload-a-file-or-folder"></a>Carga de un archivo o una carpeta

1. Haga clic con el botón derecho en el editor de scripts y seleccione **Cargar archivo** o **Cargar carpeta**.
2. Elija uno o varios archivos si ha seleccionado **Cargar archivo** o elija toda la carpeta si ha seleccionado **Cargar carpeta**. A continuación, seleccione **Cargar**.
3. Elija la carpeta de almacenamiento en la lista o seleccione **Indicar ruta de acceso** o **Examinar desde ruta de acceso raíz**. (Usamos **Indicar ruta de acceso** como un ejemplo).
4. Seleccione la cuenta de Data Lake Analytics.
5. Busque o escriba la ruta de acceso de la carpeta de almacenamiento (por ejemplo, /output/).
6. Seleccione **Elegir la carpeta actual** para especificar el destino de la carga.

![Pasos y resultado para cargar un archivo o una carpeta](./media/data-lake-analytics-data-lake-tools-for-vscode/upload-file.png)

Otra manera de cargar archivos en el almacenamiento es con el menú contextual en la ruta de acceso completa o relativa del archivo en el editor de scripts.

También puede [supervisar el estado de la carga](#check-storage-tasks-status).

### <a name="download-a-file"></a>Descarga de un archivo

Puede descargar un archivo mediante el comando **ADL: Download File** o **ADL: Download File (Advanced)** .

### <a name="to-download-a-file-through-the-adl-download-file-advanced-command"></a>Para descargar un archivo mediante el comando ADL: Download File (Advanced)

1. Haga clic con el botón derecho en el editor de scripts y después seleccione **Download File (Advanced)** (Descargar archivo [Avanzado]).
2. VS Code muestra un archivo JSON. Puede especificar las rutas de acceso de archivo y descargar varios archivos al mismo tiempo. Las instrucciones se muestran en la ventana **Salida**. Para continuar con la descarga del archivo o los archivos, guarde (Ctrl+S) el archivo JSON.

    ![Archivo JSON con rutas de acceso para la descarga de archivos](./media/data-lake-analytics-data-lake-tools-for-vscode/download-multi-files.png)

La ventana **Salida** muestra el estado de la descarga del archivo.

![Ventana de salida con el estado de la descarga](./media/data-lake-analytics-data-lake-tools-for-vscode/download-multi-file-result.png)

También puede [supervisar el estado de la descarga](#check-storage-tasks-status).

### <a name="to-download-a-file-through-the-adl-download-file-command"></a>Para descargar un archivo mediante el comando ADL: Download File

1. Haga clic con el botón derecho en el editor de scripts, seleccione **Descargar archivo**y, luego, elija la carpeta de destino en el cuadro de diálogo **Seleccionar carpeta**.

1. Elija la carpeta en la lista o seleccione **Indicar ruta de acceso** o **Examinar desde ruta de acceso raíz**. (Usamos **Indicar ruta de acceso** como un ejemplo).

1. Seleccione la cuenta de Data Lake Analytics.

1. Busque o escriba la ruta de acceso de la carpeta de almacenamiento (por ejemplo: /output/) y, a continuación, elija el archivo que desea descargar.

![Pasos y resultado para descargar un archivo](./media/data-lake-analytics-data-lake-tools-for-vscode/download-file.png)

Otra manera de descargar archivos del almacenamiento es con el menú contextual en la ruta de acceso completa o relativa del archivo en el editor de scripts.

También puede [supervisar el estado de la descarga](#check-storage-tasks-status).

### <a name="check-storage-tasks-status"></a>Comprobación del estado de las tareas de almacenamiento

El estado de la carga y la descarga aparece en la barra de estado. Seleccione la barra de estado y, a continuación, el estado aparece en el pestaña **Salida**.

![Barra de estado y detalles de la salida](./media/data-lake-analytics-data-lake-tools-for-vscode/storage-status.png)

## <a name="integrate-with-azure-data-lake-analytics-from-the-explorer"></a>Integración con Azure Data Lake Analytics desde el explorador

Después de iniciar sesión, todas las suscripciones de la cuenta de Azure aparecen en el panel izquierdo de **Azure Data Lake**.

![Explorador de Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/datalake-explorer.png)

### <a name="data-lake-analytics-metadata-navigation"></a>Exploración de los metadatos de Data Lake Analytics

Expanda la suscripción de Azure. En el nodo **Bases de datos U-SQL**, puede desplazarse por la base de datos U-SQL y ver las carpetas, como **Esquemas**, **Credenciales**, **Ensamblados**, **Tablas** e **Índice**.

### <a name="data-lake-analytics-metadata-entity-management"></a>Administración de entidades de metadatos de Data Lake Analytics

Expanda **Bases de datos U-SQL**. Puede crear una base de datos, un esquema, una tabla, un tipo de tabla, un índice o una estadística haciendo clic con el botón derecho en el nodo correspondiente y, a continuación, seleccionando **Script para crear** en el menú contextual. En la página del script que se abre, edite el script según sus necesidades. A continuación, envíe el trabajo haciendo clic con el botón derecho en él y seleccionando **ADL: Submit Job**.

Cuando termine de crear el elemento, haga clic con el botón derecho en el nodo y, a continuación, seleccione **Actualizar** para mostrar el elemento. También puede eliminar el elemento haciendo clic con el botón derecho en el elemento y seleccionando **Eliminar**.

![Comando "Script para crear" en el menú contextual del Explorador de Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-code-explorer-script-create.png)

![Página del script para el nuevo elemento](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-code-explorer-script-create-snippet.png)

### <a name="data-lake-analytics-assembly-registration"></a>Registro de ensamblados de Data Lake Analytics

Puede registrar un ensamblado en la base de datos correspondiente haciendo clic con el botón derecho en el nodo **Ensamblados** y, a continuación, seleccionando **Registrar ensamblado**.

![Comando "Registrar ensamblado" en el menú contextual del nodo Ensamblados](./media/data-lake-analytics-data-lake-tools-for-vscode/datalake-explorer-register-assembly.png)

## <a name="integrate-with-azure-data-lake-store-from-the-explorer"></a>Integración con Azure Data Lake Store desde el explorador

Vaya a **Data Lake Store**:

- Puede hacer clic con el botón derecho en el nodo de la carpeta y, a continuación, utilizar los comandos **Actualizar**, **Eliminar**, **Cargar**, **Cargar carpeta**, **Copiar ruta de acceso relativa** y **Copiar ruta de acceso completa** del menú contextual.

   ![Comandos del menú contextual para un nodo de carpeta en el Explorador de Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/storage-account-folder-menu.png)

- Puede hacer clic con el botón derecho en el nodo del archivo y, a continuación, usar los comandos **Vista previa**, **Descargar**, **Eliminar**, **Crear script de extracción** (solo disponible para archivos CSV, TSV y TXT files), **Copiar ruta de acceso relativa** y **Copiar ruta de acceso completa** del menú contextual.

   ![Comandos del menú contextual para un nodo de archivo en el Explorador de Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/storage-account-extract.png)

## <a name="integrate-with-azure-blob-storage-from-the-explorer"></a>Integración con Azure Blob Storage desde el explorador

Vaya a Blob Storage:

- Puede hacer clic con el botón derecho en el nodo del contenedor de blobs y, a continuación, utilizar los comandos **Actualizar**, **Eliminar contenedor de blobs** y **Cargar blob** del menú contextual.

   ![Comandos del menú contextual para un nodo de contenedor de blobs en Blob Storage](./media/data-lake-analytics-data-lake-tools-for-vscode/blob-storage-blob-container-node.png)

- Puede hacer clic con el botón derecho en el nodo de la carpeta y, a continuación, usar los comandos **Actualizar** y **Cargar blob** del menú contextual.

   ![Comandos del menú contextual para un nodo de carpeta en Blob Storage](./media/data-lake-analytics-data-lake-tools-for-vscode/blob-storage-folder-node.png)

- Puede hacer clic con el botón derecho en el nodo del archivo y, a continuación, usar los comandos **Vista previa/Editar**, **Descargar**, **Eliminar**, **Crear script de extracción** (solo disponible para archivos CSV, TSV y TXT files), **Copiar ruta de acceso relativa** y **Copiar ruta de acceso completa** del menú contextual.

    ![Comandos del menú contextual para un nodo de archivo en Blob Storage](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-from-context-menu-2.png)

## <a name="open-the-data-lake-explorer-in-the-portal"></a>Apertura del Explorador de Data Lake en el portal

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **Abrir el explorador web de Azure Storage** o haga clic con el botón derecho en una ruta de acceso relativa o completa en el editor de scripts y, luego, seleccione **Abrir el explorador web de Azure Storage**.
3. Seleccione una cuenta de Data Lake Analytics.

Herramientas de Data Lake abre la ruta de acceso de Azure Storage en Azure Portal. Puede encontrar la ruta de acceso y obtener la vista previa del archivo desde la Web.

## <a name="additional-features"></a>Características adicionales

Herramientas de Data Lake para VS Code es compatible con las características siguientes:

- **Autocompletar de IntelliSense**: aparecen sugerencias en ventanas emergentes en torno a elementos como palabras clave, métodos y variables. Los distintos iconos representan diferentes tipos de objetos:

  - Tipo de datos de escala
  - Tipo de datos complejo
  - UDT integrada
  - Clases y colección .NET
  - Expresiones de C#
  - UDF, UDO y UDAAG integrados de C#
  - Funciones de U-SQL
  - Funciones de ventana de U-SQL

    ![Tipos de objeto de IntelliSense](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-auto-complete-objects.png)

- **Autocompletar de IntelliSense en los metadatos de Data Lake Analytics**: Data Lake Tools descarga la información de metadatos de Data Lake Analytics localmente. La característica IntelliSense rellena automáticamente los objetos a partir de los metadatos de Data Lake Analytics. Estos objetos incluyen base de datos, esquema, tabla, vista, función con valores de tabla, procedimientos y ensamblados de C#.

  ![Metadatos de IntelliSense](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-auto-complete-metastore.png)

- **Marcador de error de IntelliSense**: Data Lake Tools subraya los errores de edición para U-SQL U y C#.
- **Resaltado de la sintaxis**: Herramientas de Data Lake usa colores distintos para diferenciar elementos como variables, palabras clave, tipo de datos y funciones.

    ![Sintaxis con distintos colores](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-syntax-highlights.png)

> [!NOTE]
> Se recomienda que actualice a Herramientas de Azure Data Lake para Visual Studio versión 2.3.3000.4 o posterior. Las versiones anteriores no están disponibles para su descarga y han quedado en desuso.  

## <a name="next-steps"></a>Pasos siguientes

- [Desarrollo de trabajos U-SQL con Python, R y CSharp para Azure Data Lake Analytics en VS Code](data-lake-analytics-u-sql-develop-with-python-r-csharp-in-vscode.md)
- [Ejecución y depuración locales de U-SQL con Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md)
- [Tutorial: Introducción a Azure Data Lake Analytics](data-lake-analytics-get-started-portal.md)
- [Tutorial: Desarrollo de scripts U-SQL mediante Data Lake Tools para Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)
