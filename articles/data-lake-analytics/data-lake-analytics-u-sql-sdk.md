---
title: 'Ejecución de trabajos de U-SQL localmente: SDK de U-SQL para Azure Data Lake'
description: Obtenga información sobre cómo ejecutar y probar localmente trabajos de U-SQL mediante la línea de comandos y las interfaces de programación en la estación de trabajo local.
services: data-lake-analytics
ms.service: data-lake-analytics
author: yanacai
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 03/01/2017
ms.openlocfilehash: daf72fcf7baba289b4145d06d878c8a7232f1c6a
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132422"
---
# <a name="run-and-test-u-sql-with-azure-data-lake-u-sql-sdk"></a>Ejecución y prueba de U-SQL con el SDK de U-SQL para Azure Data Lake

Al desarrollar el script U-SQL, es frecuente ejecutarlo y probarlo localmente antes de enviarlo a la nube. Azure Data Lake proporciona un paquete Nuget denominado SDK de U-SQL para Azure Data Lake para esta situación, por lo que le resultará muy fácil realizar el escalado de prueba y la ejecución de U-SQL. También es posible integrar esta prueba de U-SQL con el sistema de CI (integración continua) para automatizar la compilación y la prueba.

Si está interesado en cómo ejecutar y depurar localmente y de forma manual el script U-SQL con las herramientas GUI, entonces puede usar las Herramientas de Azure Data Lake para Visual Studio. Puede obtener más información sobre esto [aquí](data-lake-analytics-data-lake-tools-local-run.md).

## <a name="install-azure-data-lake-u-sql-sdk"></a>Instalación del SDK de U-SQL para Azure Data Lake

Puede obtener el SDK de U-SQL para Azure Data Lake [aquí](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/) en Nuget.org. Y, antes de usarlo, debe asegurarse de que dispone de las siguientes dependencias.

### <a name="dependencies"></a>Dependencias

El SDK de U-SQL para Data Lake requiere las siguientes dependencias:

- [Microsoft .NET Framework 4.6 o superior](https://www.microsoft.com/download/details.aspx?id=17851).
- Microsoft Visual C++ 14 y el SDK de Windows 10.0.10240.0 o posterior (que se denomina CppSDK en este artículo). Existen dos formas de obtener CppSDK:

  - Instale [Visual Studio Community Edition](https://developer.microsoft.com/downloads/vs-thankyou). Debe tener una carpeta \Windows Kits\10 en la carpeta de archivos de programa, por ejemplo, C:\Program Files (x86)\Windows Kits\10\. También encontrará la versión del SDK de Windows 10 en \Windows Kits\10\Lib. Si no ve estas carpetas, vuelva a instalar Visual Studio y asegúrese de seleccionar el SDK de Windows 10 durante la instalación. Si ya lo tiene instalado con Visual Studio, el compilador local de U-SQL lo encontrará automáticamente.

    ![SDK de Windows 10 para ejecución local de Data Lake Tools para Visual Studio](./media/data-lake-analytics-data-lake-tools-local-run/data-lake-tools-for-visual-studio-local-run-windows-10-sdk.png)

  - Instale [Data Lake Tools para Visual Studio](https://aka.ms/adltoolsvs). Puede encontrar los archivos de Visual C++ y Windows SDK preempaquetados en `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\ADL Tools\X.X.XXXX.X\CppSDK.`.

    En este caso, el script del compilador local de U-SQL no podrá encontrar estas dependencias automáticamente. Debe especificar la ruta de acceso de CppSDK para él. Puede copiar los archivos en otra ubicación o simplemente usarla como está.

## <a name="understand-basic-concepts"></a>Conceptos básicos

### <a name="data-root"></a>Raíz de datos

La carpeta raíz de datos es un "almacén local" de la cuenta de proceso local. Es equivalente a la cuenta de Azure Data Lake Store de una cuenta de Data Lake Analytics. Cambiar a una carpeta raíz de datos diferente es como que cambiar a otra cuenta de almacén. Si desea tener acceso a datos compartidos normalmente con carpetas raíz de datos diferentes, debe usar rutas de acceso absolutas en los scripts. O bien, cree vínculos simbólicos del sistema de archivos (por ejemplo, **mklink** en NTFS) en la carpeta raíz de datos para que apunte a los datos compartidos.

La carpeta raíz de datos se utiliza para:

- Almacenar metadatos, lo que incluye bases de datos, tablas, funciones con valores de tabla (TVF) y ensamblados.
- Buscar las rutas de acceso de entrada y salida que se definen como rutas de acceso relativas en U-SQL. El uso de rutas de acceso relativas facilita la implementación de sus proyectos U-SQL en Azure.

### <a name="file-path-in-u-sql"></a>Ruta de archivo de U-SQL

Puede usar tanto una ruta de acceso relativa como una ruta de acceso absoluta local en los scripts U-SQL. La ruta de acceso relativa es relativa a la ruta de acceso de la carpeta raíz de datos especificada. Se recomienda usar "/" como separador de ruta de acceso para que los scripts sean compatibles con el lado servidor. Estos son algunos ejemplos de rutas de acceso relativas y sus rutas de acceso absolutas equivalentes. En estos ejemplos, C:\LocalRunDataRoot es la carpeta raíz de datos.

|Ruta de acceso relativa|Ruta de acceso absoluta|
|-------------|-------------|
|/abc/def/input.csv |C:\LocalRunDataRoot\abc\def\input.csv|
|abc/def/input.csv  |C:\LocalRunDataRoot\abc\def\input.csv|
|D:/abc/def/input.csv |D:\abc\def\input.csv|

### <a name="working-directory"></a>Directorio de trabajo

Al ejecutar localmente el script U-SQL, se crea un directorio de trabajo durante la compilación en el directorio que se está ejecutando actualmente. Además de las salidas de compilación, se creará una instantánea de los archivos del runtime necesarios para la ejecución local en este directorio de trabajo. La carpeta de raíz del directorio de trabajo se denomina "ScopeWorkDir" y los archivos dentro el directorio de trabajo son los siguientes:

|Directorio o archivo|Directorio o archivo|Directorio o archivo|Definición|Descripción|
|--------------|--------------|--------------|----------|-----------|
|C6A101DDCB470506| | |Cadena de hash de la versión del runtime|Instantánea de los archivos del runtime necesarios para la ejecución local|
| |Script_66AE4909AA0ED06C| |Nombre de script + cadena de hash de la ruta de acceso del script|Resultados de compilación y registro de los pasos de ejecución|
| | |\_script\_.abr|Salida del compilador|Archivo de álgebra|
| | |\_ScopeCodeGen\_.*|Salida del compilador|Código administrado generado|
| | |\_ScopeCodeGenEngine\_.*|Salida del compilador|Código nativo generado|
| | |ensamblados de referencia|Referencia de ensamblado|Archivos de ensamblados de referencia|
| | |deployed_resources|Implementación de recursos|Archivos de implementación de recursos|
| | |xxxxxxxx.xxx[1..n]\_\*.\*|Registro de ejecución|Registro de pasos de ejecución|

## <a name="use-the-sdk-from-the-command-line"></a>Uso del SDK desde la línea de comandos

### <a name="command-line-interface-of-the-helper-application"></a>Interfaz de la línea de comandos del asistente

El archivo LocalRunHelper.exe se encuentra en el directorio del SDK, en build/runtime, y es el asistente de línea de comandos que proporciona interfaces a la mayoría de las funciones de ejecución local de uso frecuente. Tenga en cuenta que tanto el comando como los modificadores de argumentos distinguen entre mayúsculas y minúsculas. Para invocarla:

```console
LocalRunHelper.exe <command> <Required-Command-Arguments> [Optional-Command-Arguments]
```

Ejecute LocalRunHelper.exe sin argumentos o con el modificador **help** para mostrar la información de ayuda:

```console
> LocalRunHelper.exe help
    Command 'help' :  Show usage information
    Command 'compile' :  Compile the script
    Required Arguments :
        -Script param
                Script File Path
    Optional Arguments :
        -Shallow [default value 'False']
                Shallow compile
```

En la información de ayuda:

- **Command** proporciona el nombre del comando.  
- **Required Argument** enumera los argumentos que se deben proporcionar.  
- **Optional Argument** enumera argumentos que son opcionales, con los valores predeterminados.  Los argumentos opcionales booleanos no tienen parámetros y sus repeticiones implican un valor negativo de sus valores predeterminados.

### <a name="return-value-and-logging"></a>Valor devuelto y registro

El asistente devuelve **0** si se ha realizado correctamente y **-1** en caso contrario. De forma predeterminada, el asistente envía todos los mensajes a la consola actual. Sin embargo, la mayoría de los comandos admiten el argumento opcional **-MessageOut ruta_de_acceso_al_archivo_de_registro** que redirige las salidas a un archivo de registro.

### <a name="environment-variable-configuring"></a>Configuración de la variable de entorno

La ejecución local de U-SQL requiere una raíz de datos explícita como cuenta de almacenamiento local, así como una ruta de acceso de CppSDK explícita para las dependencias. Puede establecer el argumento en la línea de comandos o bien puede establecer la variable de entorno para ellos.

- Establezca la variable de entorno **SCOPE_CPP_SDK**.

  Si obtiene Microsoft Visual C++ y Windows SDK con la instalación de Data Lake Tools para Visual Studio, compruebe que dispone de la siguiente carpeta:

    `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\Microsoft Azure Data Lake Tools for Visual Studio 2015\X.X.XXXX.X\CppSDK`

  Defina una nueva variable de entorno denominada **SCOPE_CPP_SDK** para que apunte a este directorio. O copie la carpeta en otra ubicación y especifique **SCOPE_CPP_SDK** como eso.

  Además de establecer la variable de entorno, también puede especificar el argumento **-CppSDK** cuando se utiliza la línea de comandos. Este argumento sobrescribe la variable de entorno CppSDK predeterminada.

- Establezca la variable de entorno **LOCALRUN_DATAROOT**.

  Defina una nueva variable de entorno denominada **LOCALRUN_DATAROOT** que apunte a la raíz de datos.

  Además de establecer la variable de entorno, puede especificar el argumento **-DataRoot** con la ruta de acceso raíz de datos cuando se utiliza la línea de comandos. Este argumento sobrescribe la variable de entorno raíz de datos predeterminada. Necesita agregar este argumento a cada línea de comandos que esté ejecutando para que pueda sobrescribir la variable de entorno raíz de datos predeterminada para todas las operaciones.

### <a name="sdk-command-line-usage-samples"></a>Ejemplos de uso de línea de comandos del SDK

#### <a name="compile-and-run"></a>Compilación y ejecución

El comando **run** se utiliza para compilar el script y ejecutar después los resultados compilados. Sus argumentos de línea de comandos son una combinación de los de **compile** y **execute**.

```console
LocalRunHelper run -Script path_to_usql_script.usql [optional_arguments]
```

A continuación se muestran los argumentos opcionales para **run**:

|Argumento|Valor predeterminado|Descripción|
|--------|-------------|-----------|
|-CodeBehind|False|El script tiene código subyacente .cs|
|-CppSDK| |Directorio CppSDK|
|-DataRoot| Variable de entorno de DataRoot|DataRoot para ejecución local; el valor predeterminado es la variable de entorno 'LOCALRUN_DATAROOT'|
|-MessageOut| |Volcar mensajes de la consola en un archivo|
|-Parallel|1|Ejecutar el plan con el paralelismo especificado|
|-References| |Lista de rutas de acceso a los ensamblados de referencia adicionales o archivos de datos de código subyacente, separadas por ';'|
|-UdoRedirect|False|Generar la configuración de redirección de ensamblado de Udo|
|-UseDatabase|maestro|Base de datos que se utilizará para el registro de ensamblados temporal de código subyacente|
|-Verbose|False|Mostrar resultados detallados del runtime|
|-WorkDir|Directorio actual|Directorio para las salidas y el uso del compilador|
|-RunScopeCEP|0|Modo de ScopeCEP que se utilizará|
|-ScopeCEPTempPath|temp|Ruta de acceso temporal que se usará para el streaming de datos|
|-OptFlags| |Lista de marcas de optimizador separadas por comas|

Este es un ejemplo:

`LocalRunHelper run -Script d:\test\test1.usql -WorkDir d:\test\bin -CodeBehind -References "d:\asm\ref1.dll;d:\asm\ref2.dll" -UseDatabase testDB –Parallel 5 -Verbose`

Además de combinar **compile** y **execute**, puede compilar y ejecutar los ejecutables compilados por separado.

#### <a name="compile-a-u-sql-script"></a>Compilación de un script U-SQL

El comando **compile** se utiliza para compilar un script U-SQL en los archivos ejecutables.

```console
LocalRunHelper compile -Script path_to_usql_script.usql [optional_arguments]
```

A continuación, se muestran los argumentos opcionales para **compile**:

|Argumento|Descripción|
|--------|-----------|
| -CodeBehind [valor predeterminado 'False']|El script tiene código subyacente .cs|
| -CppSDK [valor predeterminado '']|Directorio CppSDK|
| -DataRoot [valor predeterminado 'DataRoot environment variable']|DataRoot para ejecución local; el valor predeterminado es la variable de entorno 'LOCALRUN_DATAROOT'|
| -MessageOut [valor predeterminado '']|Volcar mensajes de la consola en un archivo|
| -References [valor predeterminado '']|Lista de rutas de acceso a los ensamblados de referencia adicionales o archivos de datos de código subyacente, separadas por ';'|
| -Shallow [valor predeterminado 'False']|Compilación superficial|
| -UdoRedirect [valor predeterminado 'False']|Generar la configuración de redirección de ensamblado de Udo|
| -UseDatabase [valor predeterminado 'master']|Base de datos que se utilizará para el registro de ensamblados temporal de código subyacente|
| -WorkDir [valor predeterminado 'Current Directory']|Directorio para las salidas y el uso del compilador|
| -RunScopeCEP [valor predeterminado '0']|Modo de ScopeCEP que se utilizará|
| -ScopeCEPTempPath [valor predeterminado 'temp']|Ruta de acceso temporal que se usará para el streaming de datos|
| -OptFlags [valor predeterminado '']|Lista de marcas de optimizador separadas por comas|

Estos son algunos ejemplos de uso.

Compile un script U-SQL:

```console
LocalRunHelper compile -Script d:\test\test1.usql
```

Compile un script U-SQL y establezca la carpeta raíz de datos. Tenga en cuenta que esta acción sobrescribirá la variable de entorno del conjunto.

```console
LocalRunHelper compile -Script d:\test\test1.usql –DataRoot c:\DataRoot
```

Compile un script U-SQL y establezca el directorio de trabajo, el ensamblado de referencia y la base de datos:

```console
LocalRunHelper compile -Script d:\test\test1.usql -WorkDir d:\test\bin -References "d:\asm\ref1.dll;d:\asm\ref2.dll" -UseDatabase testDB
```

#### <a name="execute-compiled-results"></a>Ejecución de resultados compilados

El comando **execute** se usa para ejecutar resultados compilados.

```console
LocalRunHelper execute -Algebra path_to_compiled_algebra_file [optional_arguments]
```

A continuación, se muestran los argumentos opcionales para **execute**:

|Argumento|Valor predeterminado|Descripción|
|--------|-------------|-----------|
|-DataRoot | '' |Raíz de datos para la ejecución de metadatos. De manera predeterminada, es la variable de entorno **LOCALRUN_DATAROOT**.|
|-MessageOut | '' |Volcar mensajes de la consola en un archivo.|
|-Parallel | '1' |Indicador para ejecutar los pasos de ejecución local generados con el nivel de paralelismo especificado.|
|-Verbose | 'False' |Indicador para mostrar las salidas detalladas del tiempo de ejecución.|

Presentamos un ejemplo de uso:

```console
LocalRunHelper execute -Algebra d:\test\workdir\C6A101DDCB470506\Script_66AE4909AA0ED06C\__script__.abr –DataRoot c:\DataRoot –Parallel 5
```

## <a name="use-the-sdk-with-programming-interfaces"></a>Uso del SDK con las interfaces de programación

Las interfaces de programación se encuentran en LocalRunHelper.exe. Puede utilizarlas para integrar la funcionalidad del SDK de U-SQL y la plataforma de pruebas de C# para escalar la prueba local del script U-SQL. En este artículo, utilizaré el proyecto estándar de prueba unitaria de C# para mostrar cómo se usan estas interfaces para probar el script U-SQL.

### <a name="step-1-create-c-unit-test-project-and-configuration"></a>Paso 1: crear la configuración y el proyecto de prueba unitaria de C#

- Cree un proyecto de prueba unitaria de C# en Archivo > Nuevo > Proyecto > Visual C# > Probar > Proyecto de prueba unitaria.
- Agregue LocalRunHelper.exe como referencia para el proyecto. LocalRunHelper.exe se encuentra en \build\runtime\LocalRunHelper.exe en el paquete de Nuget.

   ![Agregar referencia del SDK de U-SQL para Azure Data Lake](./media/data-lake-analytics-u-sql-sdk/data-lake-analytics-u-sql-sdk-add-reference.png)

- El SDK de U-SQL **solo** admite entornos x64. Asegúrese de establecer el destino de la plataforma de compilación en x64. Se puede establecer en Propiedad del proyecto > Compilar > Destino de la plataforma.

   ![Configurar proyecto x64 del SDK de U-SQL para Azure Data Lake](./media/data-lake-analytics-u-sql-sdk/data-lake-analytics-u-sql-sdk-configure-x64.png)

- Asegúrese de establecer el entorno de prueba en x64. En Visual Studio, se puede establecer en Probar > Configuración de pruebas > Arquitectura del procesador predeterminada > x64.

   ![Configurar el entorno de prueba x64 del SDK de U-SQL para Azure Data Lake](./media/data-lake-analytics-u-sql-sdk/data-lake-analytics-u-sql-sdk-configure-test-x64.png)

- Asegúrese de copiar todos los archivos de dependencias del directorio build\runtime\ del paquete de Nuget en el directorio de trabajo del proyecto que se suele encontrar en [carpetaDelProyecto]\bin\x64\Debug.

### <a name="step-2-create-u-sql-script-test-case"></a>Paso 2: crear un caso de prueba del script U-SQL

A continuación, se muestra el ejemplo de código para la prueba del script U-SQL. Para las pruebas, debe preparar los scripts, los archivos de entrada y los archivos de resultados esperados.

```usql
using System;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System.IO;
using System.Text;
using System.Security.Cryptography;
using Microsoft.Analytics.LocalRun;
namespace UnitTestProject1
{
    [TestClass]
    public class USQLUnitTest
    {
        [TestMethod]
        public void TestUSQLScript()
        {
            //Specify the local run message output path
            StreamWriter MessageOutput = new StreamWriter("../../../log.txt");
            LocalRunHelper localrun = new LocalRunHelper(MessageOutput);
            //Configure the DateRoot path, Script Path and CPPSDK path
            localrun.DataRoot = "../../../";
            localrun.ScriptPath = "../../../Script/Script.usql";
            localrun.CppSdkDir = "../../../CppSDK";
            //Run U-SQL script
            localrun.DoRun();
            //Script output
            string Result = Path.Combine(localrun.DataRoot, "Output/result.csv");
            //Expected script output
            string ExpectedResult = "../../../ExpectedOutput/result.csv";
            Test.Helpers.FileAssert.AreEqual(Result, ExpectedResult);
            //Don't forget to close MessageOutput to get logs into file
            MessageOutput.Close();
        }
    }
}
namespace Test.Helpers
{
    public static class FileAssert
    {
        static string GetFileHash(string filename)
        {
            Assert.IsTrue(File.Exists(filename));
            using (var hash = new SHA1Managed())
            {
                var clearBytes = File.ReadAllBytes(filename);
                var hashedBytes = hash.ComputeHash(clearBytes);
                return ConvertBytesToHex(hashedBytes);
            }
        }
        static string ConvertBytesToHex(byte[] bytes)
        {
            var sb = new StringBuilder();
            for (var i = 0; i < bytes.Length; i++)
            {
                sb.Append(bytes[i].ToString("x"));
            }
            return sb.ToString();
        }
        public static void AreEqual(string filename1, string filename2)
        {
            string hash1 = GetFileHash(filename1);
            string hash2 = GetFileHash(filename2);
            Assert.AreEqual(hash1, hash2);
        }
    }
}
```

### <a name="programming-interfaces-in-localrunhelperexe"></a>Interfaces de programación de LocalRunHelper.exe

LocalRunHelper.exe proporciona las interfaces de programación de compilación local ejecución, etc. de U-SQL. A continuación, se enumeran las interfaces.

### <a name="constructor"></a>Constructor

public LocalRunHelper([System.IO.TextWriter messageOutput = null])

|Parámetro|Tipo|Descripción|
|---------|----|-----------|
|messageOutput|System.IO.TextWriter|para los mensajes de salida; establézcalo en null para usar la consola|

### <a name="properties"></a>Propiedades

|Propiedad|Tipo|Descripción|
|--------|----|-----------|
|AlgebraPath|string|La ruta de acceso al archivo álgebra (el archivo álgebra es uno de los resultados de compilación)|
|CodeBehindReferences|string|Si el script tiene referencias adicionales de código subyacente, especifique las rutas de acceso separadas por ";".|
|CppSdkDir|string|Directorio CppSDK|
|CurrentDir|string|Directorio actual|
|DataRoot|string|Ruta de acceso raíz de datos|
|DebuggerMailPath|string|La ruta de acceso al buzón interproceso del depurador|
|GenerateUdoRedirect|bool|Si desea generar carga la configuración de invalidación de redirección de carga de ensamblados|
|HasCodeBehind|bool|Si el script tiene código subyacente|
|InputDir|string|Directorio de datos de entrada|
|MessagePath|string|Ruta de acceso del archivo de volcado de memoria de mensajes|
|OutputDir|string|Directorio de datos de salida|
|Paralelismo|int|Paralelismo para ejecutar el álgebra|
|ParentPid|int|PID del elemento primario en el que se supervisa el servicio; para salir se establece en 0 o en un valor negativo para omitirlo.|
|ResultPath|string|Ruta de acceso del archivo de volcado de memoria de resultados|
|RuntimeDir|string|Directorio del entorno en tiempo de ejecución|
|scriptPath|string|Lugar donde se puede encontrar el script|
|Shallow|bool|Compilación superficial o no|
|TempDir|string|Directorio temporal|
|UseDataBase|string|Especifique la base de datos que se utilizará para el registro de ensamblados temporal de código subyacente; la maestra de forma predeterminada.|
|WorkDir|string|Directorio de trabajo preferido|

### <a name="method"></a>Método

|Método|Descripción|Valor devuelto|Parámetro|
|------|-----------|------|---------|
|public bool DoCompile()|Compilación del script U-SQL|Si se realiza correctamente, devuelve True.| |
|public bool DoExec()|Ejecución del resultado compilado|Si se realiza correctamente, devuelve True.| |
|public bool DoRun()|Ejecución del script U-SQL (compilación + ejecución)|Si se realiza correctamente, devuelve True.| |
|public bool IsValidRuntimeDir(string path)|Comprobación de si la ruta de acceso especificada es la ruta de acceso válida al entorno en tiempo de ejecución|True para válida|La ruta de acceso del directorio del entorno en tiempo de ejecución|

## <a name="faq-about-common-issue"></a>Preguntas más frecuentes sobre problemas comunes

### <a name="error-1"></a>Error 1

E_CSC_SYSTEM_INTERNAL: Error interno. No se pudo cargar el archivo o ensamblado 'ScopeEngineManaged.dll' ni una de sus dependencias. No se pudo encontrar el módulo especificado.

Compruebe lo siguiente:

- Asegúrese de que tiene un entorno x64. La plataforma de destino de compilación y el entorno de prueba deben ser x64. Consulte el **Paso 1: crear la configuración y el proyecto de prueba unitaria de C#** más arriba.
- Asegúrese de que ha copiado todos los archivos de dependencias del directorio \build\runtime\ del paquete Nuget al directorio de trabajo del proyecto.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre U-SQL, consulte [Introducción al lenguaje U-SQL de Análisis de Azure Data Lake](data-lake-analytics-u-sql-get-started.md).
- Para registrar información de diagnóstico, consulte [Acceso a los registros de diagnóstico de Azure Data Lake Analytics](data-lake-analytics-diagnostic-logs.md).
- Para ver una consulta más compleja, consulte [Tutorial: Análisis de registros de sitios web mediante Azure Data Lake Analytics](data-lake-analytics-analyze-weblogs.md).
- Para ver los detalles del trabajo, consulte [Usar el explorador de trabajos y la vista de trabajos para trabajos de Azure Data Lake Analytics](data-lake-analytics-data-lake-tools-view-jobs.md).
- Para usar la vista de ejecución de vértices, vea [Uso de la vista de ejecución de vértices de Data Lake Tools para Visual Studio](data-lake-analytics-data-lake-tools-use-vertex-execution-view.md).
