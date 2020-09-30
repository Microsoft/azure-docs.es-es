---
title: Desarrollo de funciones de .NET Standard para trabajos de Azure Stream Analytics (versión preliminar)
description: Aprenda a escribir funciones definidas por el usuario de C# para los trabajos de Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 09/10/2020
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 9cf929a3a6f5b3752b030f449b3b24b2bdc941a1
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907278"
---
# <a name="develop-net-standard-user-defined-functions-for-azure-stream-analytics-jobs-preview"></a>Desarrollo de funciones definidas por el usuario de .NET Standard para trabajos de Azure Stream Analytics (versión preliminar)

Azure Stream Analytics ofrece un lenguaje de consulta similar a SQL para realizar transformaciones y cálculos sobre transmisiones de datos del evento. Hay muchas funciones integradas, pero algunos escenarios complejos requieren flexibilidad adicional. Con las funciones definidas por el usuario (UDF) de .NET Standard, puede invocar sus propias funciones escritas en cualquier lenguaje de .NET Standard (C#, F#, etc.) para ampliar el lenguaje de consulta de Stream Analytics. Las UDF le permiten realizar cálculos matemáticos complejos, importar modelos de aprendizaje automático personalizados mediante ML.NET y usar una lógica personalizada de imputación para datos que faltan. La característica UDF para trabajos de Stream Analytics está actualmente en versión preliminar y no debe usarse en las cargas de trabajo de producción.

La función definida por el usuario de .NET para trabajos en la nube está disponible en:
* Centro-Oeste de EE. UU.
* Norte de Europa
* Este de EE. UU.
* Oeste de EE. UU.
* Este de EE. UU. 2
* Oeste de Europa

Si está interesado en usar esta característica en cualquier otra región, puede [solicitar acceso](https://aka.ms/ccodereqregion).

## <a name="package-path"></a>Ruta de acceso de paquete

El formato de cualquier paquete UDF tiene la ruta de acceso `/UserCustomCode/CLR/*`. Las bibliotecas de vínculos dinámicos (DLL) y los recursos se copian en la carpeta `/UserCustomCode/CLR/*`, lo que ayuda a aislar los archivos DLL de usuario del sistema y los archivos DLL de Azure Stream Analytics. Esta ruta de acceso del paquete se utiliza para todas las funciones, independientemente del método empleado para usarlas.

## <a name="supported-types-and-mapping"></a>Asignación y tipos admitidos

En el caso de los valores de Azure Stream Analytics que se van a usar en C# , se deben serializar de un entorno a otro. La serialización se produce para todos los parámetros de entrada de una UDF. Cada tipo de Azure Stream Analytics tiene un tipo correspondiente en C# que se muestra en la tabla siguiente:

|**Tipo de Azure Stream Analytics** |**Tipo de C#** |
|---------|---------|
|bigint | long |
|FLOAT | double |
|nvarchar(max) | string |
|datetime | DateTime |
|Registro | Diccionario\<string, object> |
|Array | Object[] |

Lo mismo se aplica cuando es necesario serializar datos de C# a Azure Stream Analytics, lo cual sucede en el valor de salida de una UDF. En la tabla siguiente se muestran los tipos que se admiten:

|**Tipo de C#**  |**Tipo de Azure Stream Analytics**  |
|---------|---------|
|long  |  bigint   |
|double  |  FLOAT   |
|string  |  nvarchar(max)   |
|DateTime  |  dateTime   |
|struct  |  Registro   |
|object  |  Registro   |
|Object[]  |  Array   |
|Diccionario\<string, object>  |  Registro   |

## <a name="develop-a-udf-in-visual-studio-code"></a>Desarrollo de una UDF en Visual Studio Code

Las [herramientas de Visual Studio Code para Azure Stream Analytics](quick-create-visual-studio-code.md) facilitan la escritura de UDF, la prueba de los trabajos en local (incluso sin conexión) y la publicación de trabajos de Stream Analytics en Azure.

Hay dos maneras de implementar UDF de .NET Standard en las herramientas de Visual Studio Code.

* UDF a partir de DLL locales
* UDF desde un proyecto local

### <a name="local-project"></a>Proyecto local

Las funciones definidas por el usuario pueden escribirse en un ensamblado que se menciona más adelante en una consulta de Azure Stream Analytics. Esta es la opción recomendada para funciones complejas que requieren toda la eficacia de un lenguaje de .NET Standard más allá de su lenguaje de expresiones, como la lógica de procedimientos o la recursividad. También podrían utilizarse UDF de un proyecto local cuando necesite compartir la lógica de la función en varias consultas de Azure Stream Analytics. La incorporación de UDF al proyecto local da la posibilidad de depurar y probar las funciones en local.

Para hacer referencia a un proyecto local:

1. Cree una nueva biblioteca de clases de .NET Standard en la máquina local.
2. Escriba el código en la clase. Recuerde que las clases deben definirse como *pública* y los objetos deben definirse como *público estático*.
3. Agregue un nuevo archivo de configuración de la función de CSharp al proyecto de Azure Stream Analytics y haga referencia al proyecto de la biblioteca de clases de CSharp.
4. Configure la ruta de ensamblado del archivo de configuración del trabajo, `JobConfig.json`, sección **CustomCodeStorage**. Este paso no es necesario para las pruebas locales.

### <a name="local-dlls"></a>DLL locales

También se puede hacer referencia a las DLL locales que incluyen las funciones definidas por el usuario.

### <a name="example"></a>Ejemplo

En este ejemplo, **CSharpUDFProject** es un proyecto de biblioteca de clases de C# y **ASAUDFDemo** es el proyecto de Azure Stream Analytics que va a hacer referencia a **CSharpUDFProject**.

:::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-demo.png" alt-text="Proyecto de Azure Stream Analytics en Visual Studio Code":::

La siguiente UDF tiene una función que multiplica un entero por sí mismo para generar el cuadrado de dicho entero. Las clases deben definirse como *públicas* y los objetos como *públicos estáticos*.

```csharp
using System;

namespace CSharpUDFProject
{
    // 
    public class Class1
    {
        public static Int64 SquareFunction(Int64 a)
        {
            return a * a;
        }
    }
}
```

En los siguientes pasos se muestra cómo agregar la UDF de C# al proyecto de Stream Analytics.

1. Haga clic con el botón derecho en la carpeta **Funciones** y seleccione **Agregar elemento**.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-add-function.png" alt-text="Incorporación de nueva función al proyecto de Azure Stream Analytics":::

2. Agregue una función de C# **SquareFunction** al proyecto de Azure Stream Analytics.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-add-function-2.png" alt-text="Selección de función de CSharp desde el proyecto de Stream Analytics en VS Code":::

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-add-function-name.png" alt-text="Especificación del nombre de la función de CSharp en VS Code":::

3. En la configuración de la función de C#, seleccione **Choose library project path** (Elegir la ruta del proyecto de biblioteca) para seleccionar el proyecto de C# en la lista desplegable y luego **Compilar proyecto** para compilar el proyecto. Luego seleccione **Seleccionar clase** y **Seleccionar método** para seleccionar el nombre de la clase y el método relacionados en la lista desplegable. Para hacer referencia a los métodos, tipos y funciones en la consulta de Stream Analytics, se deben definir las clases como *pública* y los objetos como *público estático*.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-choose-project.png" alt-text="Configuración de la función de CSharp de Stream Analytics en VS Code":::

    Si quiere usar la UDF de C# de una DLL, seleccione **Choose library dll path** (Elegir la ruta de la DLL de biblioteca) para seleccionar la DLL. Luego seleccione **Seleccionar clase** y **Seleccionar método** para seleccionar el nombre de la clase y el método relacionados en la lista desplegable.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-choose-dll.png" alt-text="Configuración de la función de Stream Analytics C sharp":::

4. Invoque la UDF en la consulta de Azure Stream Analytics.

   ```sql
    SELECT price, udf.SquareFunction(price)
    INTO Output
    FROM Input 
   ```

5. Antes de enviar el trabajo a Azure, configure la ruta del paquete en el archivo de configuración del trabajo, `JobConfig.json`, sección **CustomCodeStorage**. Use **Seleccionar de la suscripción** en CodeLens para seleccionar la suscripción y seleccione la cuenta de almacenamiento y el nombre del contenedor en la lista desplegable. Deje **Ruta** en el valor predeterminado. Este paso no es necesario para pruebas locales.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-configure-storage-account.png" alt-text="Selección de la ruta de la biblioteca":::

## <a name="develop-a-udf-in-visual-studio"></a>Desarrollo de una UDF en Visual Studio

Hay tres maneras de implementar UDF en las herramientas de Visual Studio.

* Archivos de CodeBehind en un proyecto ASA
* UDF desde un proyecto local
* Un paquete existente desde una cuenta de almacenamiento de Azure

### <a name="codebehind"></a>CodeBehind

Puede escribir funciones definidas por el usuario en el CodeBehind **Script.asql**. Las herramientas de Visual Studio compilarán automáticamente el archivo de código subyacente en un archivo de ensamblado. Los ensamblados se empaquetan como archivos zip y se cargan en su cuenta de almacenamiento al enviar el trabajo en Azure. Puede aprender a escribir un UDF de C# con CodeBehind siguiendo el tutorial de [UDF de C# para los trabajos perimetrales de Stream Analytics](stream-analytics-edge-csharp-udf.md). 

### <a name="local-project"></a>Proyecto local

Para hacer referencia a un proyecto local en Visual Studio:

1. Cree una nueva biblioteca de clases de .NET Standard en la solución.
2. Escriba el código en la clase. Recuerde que las clases deben definirse como *pública* y los objetos deben definirse como *público estático*. 
3. Compile el proyecto. Las herramientas empaquetarán todos los artefactos de la carpeta de la papelera en un archivo ZIP y cargarán dicho archivo en la cuenta de almacenamiento. Para las referencias externas, utilice la referencia de ensamblado en lugar del paquete NuGet.
4. Haga referencia a la nueva clase en el proyecto de Azure Stream Analytics.
5. Agregue una nueva función en el proyecto de Azure Stream Analytics.
6. Configure la ruta de acceso de ensamblado en el archivo de configuración del trabajo, `JobConfig.json`. Establezca la ruta de acceso de ensamblado en **Referencia de proyecto local o de CodeBehind**.
7. Vuelva a generar el proyecto de función y el proyecto de Azure Stream Analytics.  

### <a name="example"></a>Ejemplo

En este ejemplo, **UDFTest** es un proyecto de biblioteca de clases de C# y **ASAUDFDemo** es el proyecto de Azure Stream Analytics, que hará referencia a **UDFTest**.

:::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-demo.png" alt-text="Proyecto de Azure Stream Analytics en IoT Edge en Visual Studio":::

1. Compile el proyecto de C#, que le permitirá agregar una referencia a la UDF en C# de las consultas de Azure Stream Analytics.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-build-project.png" alt-text="Compilar un proyecto de Azure Stream Analytics en IoT Edge en Visual Studio":::

2. Agregue la referencia al proyecto de C# en el proyecto de ASA. Haga clic con el botón derecho en el nodo Referencias y elija Agregar referencia.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-add-reference.png" alt-text="Agregar una referencia a un proyecto de C# en Visual Studio":::

3. Elija el nombre del proyecto de C# en la lista.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-choose-project-name.png" alt-text="Elegir el nombre del proyecto de C# en la lista de referencia":::

4. Debería ver el **UDFTest** listado en **Referencias** en el **Explorador de soluciones**.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-added-reference.png" alt-text="Ver la referencia de funciones definidas por el usuario en el Explorador de soluciones":::

5. Haga clic con el botón derecho en la carpeta **Funciones** carpeta y elija **Nuevo elemento**.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-add-csharp-function.png" alt-text="Agregar un nuevo elemento a Funciones en la solución perimetral de Azure Stream Analytics":::

6. Agregue una función de C# **SquareFunction.json** a su proyecto de Azure Stream Analytics.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-add-csharp-function-2.png" alt-text="Seleccionar una función de CSharp de la lista de elementos de Stream Analytics Edge en Visual Studio":::

7. Haga doble clic en la función en el **Explorador de soluciones** para abrir el cuadro de diálogo de configuración.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-csharp-function-config.png" alt-text="Configuración de la función C sharp en Visual Studio":::

8. En la configuración de la función de C#, elija **Cargar de la referencia de proyecto de ASA** y los nombres de método, la clase y el ensamblado relacionados en la lista desplegable. Para hacer referencia a los métodos, tipos y funciones en la consulta de Stream Analytics, se deben definir las clases como *pública* y los objetos como *público estático*.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-asa-csharp-function-config.png" alt-text="Configuración de la función de CSharp de Stream Analytics en Visual Studio":::

## <a name="existing-packages"></a>Paquetes existentes

Puede crear las UDF de .NET Standard en cualquier IDE que prefiera e invocarlos desde la consulta de Azure Stream Analytics. En primer lugar, compile el código y todos los archivos DLL del paquete. El formato del paquete tiene la ruta de acceso `/UserCustomCode/CLR/*`. Después, cargue `UserCustomCode.zip` a la raíz del contenedor en su cuenta de almacenamiento de Azure.

Una vez que se hayan cargado los paquetes zip de ensamblado a la cuenta de almacenamiento de Azure, puede usar las funciones de las consultas de Azure Stream Analytics. Todo lo que necesita hacer es incluir la información de almacenamiento en la configuración del trabajo de Stream Analytics. No se puede probar la función localmente con esta opción porque las herramientas de Visual Studio no descargarán el paquete. Se redistribuye la ruta de acceso del paquete directamente al servicio. 

Para configurar la ruta de acceso de ensamblado en el archivo de configuración del trabajo, `JobConfig.json`:

Expanda la sección **Configuración de código definido por el usuario** y rellene la configuración con los siguientes valores sugeridos:

   |**Configuración**|**Valor sugerido**|
   |-------|---------------|
   |Recurso de configuración de almacenamiento global|Selección del origen de datos desde la cuenta actual|
   |Suscripción de configuración de almacenamiento global| < su suscripción >|
   |Cuenta de almacenamiento de la configuración de almacenamiento global| < su cuenta de almacenamiento >|
   |Recurso de configuración de almacenamiento de código personalizado|Selección del origen de datos desde la cuenta actual|
   |Cuenta de almacenamiento de configuración de almacenamiento de código personalizado|< su cuenta de almacenamiento >|
   |Contenedor de configuración de almacenamiento de código personalizado|< su contenedor de almacenamiento >|
   |Origen del ensamblado de código personalizado|Paquetes de ensamblado existentes de la nube|
   |Origen del ensamblado de código personalizado|UserCustomCode.zip|

## <a name="user-logging"></a>Registro de usuarios

El mecanismo de registro le permite capturar información personalizada mientras se ejecuta un trabajo. Puede usar los datos de registro para depurar o evaluar la corrección del código personalizado en tiempo real.

La clase `StreamingContext` le permite publicar información de diagnóstico mediante la función `StreamingDiagnostics.WriteError`. El código siguiente muestra la interfaz expuesta por Azure Stream Analytics.

```csharp
public abstract class StreamingContext
{
    public abstract StreamingDiagnostics Diagnostics { get; }
}

public abstract class StreamingDiagnostics
{
    public abstract void WriteError(string briefMessage, string detailedMessage);
}
```

`StreamingContext` se pasa como un parámetro de entrada al método UDF y se puede usar en la UDF para publicar información de registro personalizada. En el ejemplo siguiente, `MyUdfMethod` define una entrada **data**, proporcionada por la consulta, y una entrada **context** como `StreamingContext`, proporcionada por el motor en tiempo de ejecución. 

```csharp
public static long MyUdfMethod(long data, StreamingContext context)
{
    // write log
    context.Diagnostics.WriteError("User Log", "This is a log message");
    
    return data;
}
```

No es necesario que la consulta SQL pase el valor de `StreamingContext`. Azure Stream Analytics proporciona un objeto de contexto automáticamente si existe un parámetro de entrada presente. El uso de `MyUdfMethod` no cambia, tal como se muestra en la consulta siguiente:

```sql
SELECT udf.MyUdfMethod(input.value) as udfValue FROM input
```

Puede acceder a los mensajes de registro a través de los [registros de diagnóstico](data-errors.md).

## <a name="limitations"></a>Limitaciones

Actualmente, la versión preliminar de UDF tiene las siguientes limitaciones:

* Las UDF de .NET Standard solo pueden crearse en Visual Studio Code o Visual Studio y publicarse en Azure. Las versiones de solo lectura de las UDF de .NET Standard pueden verse en **Funciones** en Azure Portal. No se admite la creación de funciones de .NET Standard en Azure Portal.

* El editor de consultas de Azure Portal muestra un error cuando se usan UDF de .NET Standard en el portal. 

* Dado que el código personalizado comparte el contexto con el motor de Azure Stream Analytics, el código personalizado no puede hacer referencia a todo lo que tiene un espacio de nombres o dll_name en conflicto con el código de Azure Stream Analytics. Por ejemplo, no puede hacer referencia a *Newtonsoft Json*.

* Los archivos auxiliares incluidos en el proyecto se copian en el archivo zip del código personalizado del usuario que se usa al publicar el trabajo en la nube. Todos los archivos de las subcarpetas se copian directamente en la raíz de la carpeta de código personalizado del usuario en la nube cuando se descomprimen. El código postal se "acopla" cuando se descomprime.

* El código personalizado del usuario no admite carpetas vacías. No agregue carpetas vacías a los archivos auxiliares del proyecto.

## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: escritura de una función de C# definida por el usuario para un trabajo de Azure Stream Analytics (versión preliminar)](stream-analytics-edge-csharp-udf.md)
* [Tutorial: Funciones definidas por el usuario en JavaScript para Azure Stream Analytics](stream-analytics-javascript-user-defined-functions.md)
* [Creación de un trabajo de Azure Stream Analytics en Visual Studio Code](quick-create-visual-studio-code.md)