---
title: 'Depuración de trabajos de U-SQL: Herramientas de Azure Data Lake para Visual Studio Code'
description: Obtenga información sobre cómo usar Herramientas de Azure Data Lake para Visual Studio Code para ejecutar y depurar localmente trabajos de U-SQL.
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: dc9b21d8-c5f4-4f77-bcbc-eff458f48de2
ms.topic: how-to
ms.date: 07/14/2017
ms.openlocfilehash: 4b8285281119121b509c62b11bf48a5f70a9f0f0
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87125707"
---
# <a name="run-u-sql-and-debug-locally-in-visual-studio-code"></a>Ejecutar U-SQL y depurar localmente en Visual Studio Code
En este artículo se describe cómo ejecutar trabajos de U-SQL en un equipo de desarrollo local para acelerar la primeras fases de codificación o depurar código localmente en Visual Studio Code. Para consultar instrucciones sobre las Herramientas de Azure Data Lake para Visual Studio Code, vea [Uso de Herramientas de Azure Data Lake para Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md).

Únicamente las instalaciones de Windows de las Herramientas de Azure Data Lake para Visual Studio admiten la acción para ejecutar y depurar U-SQL localmente. Las instalaciones en sistemas operativos basados en Linux y macOS no admiten esta característica.

## <a name="set-up-the-u-sql-local-run-environment"></a>Configuración del entorno de ejecución local de U-SQL

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos y luego escriba **ADL: descargar paquete de ejecución local** para descargar los paquetes.  

   ![Descarga de los paquetes de dependencia de ADL LocalRun](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/downloadtheadllocalrunpackage.png)

2. Busque los paquetes de dependencia de la ruta de acceso que se muestra en el panel **Salida** y, luego, instale BuildTools y Win10SDK 10240. A continuación se muestra una ruta de acceso de ejemplo:  
`C:\Users\xxx\AppData\Roaming\LocalRunDependency` 

   ![Ubicación de los paquetes de dependencia](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/LocateDependencyPath.png)

   2.1 Para instalar **BuildTools**, haga clic en visualcppbuildtools_full.exe en la carpeta LocalRunDependency y después siga las instrucciones del asistente.   

    ![Instalación de BuildTools](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/InstallBuildTools.png)

   2.2 Para instalar **Win10SDK 10240**, haga clic en sdksetup.exe en la carpeta LocalRunDependency/Win10SDK_10.0.10240_2 y después siga las instrucciones del asistente.  

    ![Instalación de Win10SDK 10240](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/InstallWin10SDK.png)

3. Configure la variable de entorno. Establezca la variable de entorno **SCOPE_CPP_SDK** en:  
`C:\Users\XXX\AppData\Roaming\LocalRunDependency\CppSDK_3rdparty`  


## <a name="start-the-local-run-service-and-submit-the-u-sql-job-to-a-local-account"></a>Inicio del servicio de ejecución local y envío del trabajo de U-SQL a la cuenta local 
Para los usuarios nuevos, use **ADL: descargar paquete de ejecución local** para descargar los paquetes de ejecución locales, si no ha  [configurado el entorno de ejecución local de U-SQL](#set-up-the-u-sql-local-run-environment).

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos y luego escriba **ADL: iniciar servicio de ejecución local**.   
2. Seleccione **Aceptar** para aceptar los Términos de licencia del software de Microsoft por primera vez. 

   ![Aceptación de los Términos de licencia del software de Microsoft](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/AcceptEULA.png)   
3. Se abre la consola cmd. Para los usuarios nuevos, tiene que especificar **3** y asignar la ruta de acceso a la carpeta local para la entrada y salida de datos. Si no consigue definir la ruta de acceso con barras diagonales inversas, pruebe las barras diagonales. Para otras opciones, puede usar los valores predeterminados.

   ![Cmd de ejecución local de Data Lake Tools para Visual Studio Code](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/data-lake-tools-for-vscode-local-run-cmd.png)
4. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos, escriba **ADL: enviar trabajo** y luego seleccionar **Local** para enviar el trabajo a la cuenta local.

   ![Selección local de Herramientas de Data Lake para Visual Studio Code](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/data-lake-tools-for-vscode-select-local.png)
5. Una vez que envía el trabajo, puede ver los detalles del envío. Para hacerlo, seleccione **jobUrl** en la ventana **Salida**. También puede ver el estado de envío del trabajo en la consola cmd. Escriba **7** en la consola cmd si desea más información sobre los detalles del trabajo.

   ![Salida d ejecución local de Herramientas de Data Lake para Visual Studio Code](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/data-lake-tools-for-vscode-local-run-result.png)
   ![Herramientas de Data Lake para el estado de cmd de ejecución local de Visual Studio Code](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/data-lake-tools-for-vscode-localrun-cmd-status.png) 


## <a name="start-a-local-debug-for-the-u-sql-job"></a>Inicio de una depuración local para el trabajo de U-SQL  
Para los usuarios nuevos:

1. Use **ADL: descargar paquete de ejecución local** para descargar los paquetes de ejecución locales, si no ha  [configurado el entorno de ejecución local de U-SQL](#set-up-the-u-sql-local-run-environment).
2. Instale SDK de .NET Core 2.0 como se sugiere en el cuadro de mensaje, si no está instalado.
 
  ![recordatorio de instalación de Dotnet](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/remind-install-dotnet.png)
3. Instale C# para Visual Studio Code como se sugiere en el cuadro de mensaje, si no está instalado. Haga clic en **Instalar** para continuar y después reinicie VSCode.

    ![Recordatorio de instalación de C#](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/install-csharp.png)

Siga estos pasos para realizar la depuración local:
  
1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos y luego escriba **ADL: iniciar servicio de ejecución local**. Se abre la consola cmd. Asegúrese de que se establece **DataRoot**.
2. Establezca un punto de interrupción en el código subyacente de C#.
3. Vuelva al editor de scripts, haga clic con el botón derecho y seleccione **ADL: Depuración local**.
    
   ![Resultado de la depuración local de Herramientas de Data Lake para Visual Studio Code](./media/data-lake-analytics-data-lake-tools-for-vscode-local-run-and-debug/data-lake-tools-for-vscode-local-debug-result.png)


## <a name="next-steps"></a>Pasos siguientes
* [Uso de Azure Data Lake Tools para Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md)
* [Desarrollo de trabajos U-SQL con Python, R y CSharp para Azure Data Lake Analytics en VSCode](data-lake-analytics-u-sql-develop-with-python-r-csharp-in-vscode.md)
* [Introducción a Análisis de Data Lake mediante PowerShell](data-lake-analytics-get-started-powershell.md)
* [Introducción a Análisis de Data Lake mediante el Portal de Azure](data-lake-analytics-get-started-portal.md)
* [Uso de Data Lake Tools for Visual Studio para desarrollar aplicaciones de U-SQL](data-lake-analytics-data-lake-tools-get-started.md)
* [Uso del catálogo de Data Lake Analytics (U-SQL)](data-lake-analytics-use-u-sql-catalog.md)
