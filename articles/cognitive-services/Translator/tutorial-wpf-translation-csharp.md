---
title: 'Tutorial: Creación de una aplicación de traducción con WPF y C#: Traductor'
titleSuffix: Azure Cognitive Services
description: En este tutorial, se va a crear una aplicación de WPF para realizar la traducción de texto, la detección de idioma y la corrección ortográfica con una clave de suscripción única.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: tutorial
ms.date: 05/26/2020
ms.author: swmachan
ms.custom: devx-track-csharp
ms.openlocfilehash: ef5384abd63dcd9aeb4789dc4955f4b80068d330
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88921246"
---
# <a name="tutorial-create-a-translation-app-with-wpf"></a>Tutorial: Creación de una aplicación de traducción con WPF

En este tutorial, se va a crear una aplicación de [Windows Presentation Foundation (WPF)](https://docs.microsoft.com/visualstudio/designers/getting-started-with-wpf?view=vs-2019) que utiliza Azure Cognitive Services para la traducción de texto, la detección de idioma y la corrección ortográfica con una única clave de suscripción. En concreto, la aplicación llamará a las API de Traductor y de [Bing Spell Check](https://azure.microsoft.com/services/cognitive-services/spell-check/).

¿Qué es WPF? Es un plataforma de interfaz de usuario que crea aplicaciones de cliente de escritorio. La plataforma de desarrollo WPF admite un amplio conjunto de características de desarrollo de aplicaciones, incluidos un modelo de aplicación, recursos, controles, gráficos, diseño, enlace de datos, documentos y seguridad. Es un subconjunto de .NET Framework, por lo que si ya ha creado aplicaciones con .NET Framework mediante ASP.NET o Windows Forms, la experiencia de programación debería resultarle familiar. WPF utiliza el lenguaje XAML para proporcionar un modelo declarativo para la programación de aplicaciones, que revisaremos en las próximas secciones.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de un proyecto WPF en Visual Studio
> * Adición de paquetes NuGet y ensamblados al proyecto
> * Creación de la interfaz de usuario de la aplicación con XAML
> * Uso de Traductor para obtener idiomas, traducir texto y detectar el idioma de origen
> * Uso de Bing Spell Check API para validar la entrada y mejorar la precisión de la traducción
> * Ejecución de la aplicación de WPF

### <a name="cognitive-services-used-in-this-tutorial"></a>Servicios Cognitive Services usados en este tutorial

En esta lista se incluyen los servicios Cognitive Services utilizados en este tutorial. Siga el vínculo para buscar la referencia de API de cada característica.

| Servicio | Característica | Descripción |
|---------|---------|-------------|
| Traductor | [Obtener idiomas](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-languages) | Recupera una lista completa de los idiomas admitidos para la traducción de texto. |
| Traductor | [Traducir](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-translate) | Traduce texto en más de 70 idiomas. |
| Traductor | [Detectar](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-detect) | Detecta el idioma del texto de entrada. Incluye la puntuación de confianza para la detección. |
| Bing Spell Check | [Corrector ortográfico](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v7-reference) | Corrige los errores de ortografía para mejorar la precisión de la traducción. |

## <a name="prerequisites"></a>Prerrequisitos

Antes de continuar, necesitará lo siguiente:

* Una suscripción a Azure Cognitive Services. [Obtenga una clave de Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#create-a-new-azure-cognitive-services-resource).
* Un equipo Windows
* [Visual Studio 2019](https://www.visualstudio.com/downloads/): Community o Enterprise

> [!NOTE]
> Se recomienda crear la suscripción en la región Oeste de EE. UU. para este tutorial. De lo contrario, tendrá que cambiar los puntos de conexión y las regiones en el código mientras trabaja en este ejercicio.  

## <a name="create-a-wpf-app-in-visual-studio"></a>Creación de una aplicación de WPF en Visual Studio

Lo primero que debemos hacer es configurar nuestro proyecto en Visual Studio.

1. Abra Visual Studio. Seleccione **Crear un proyecto**.
1. En **Crear un proyecto**, busque y seleccione **Aplicación de WPF (.NET Framework)** . Puede seleccionar C# en **Lenguaje** para restringir las opciones.
1. Seleccione **Siguiente** y, después, asigne un nombre al proyecto `MSTranslatorDemo`.
1. Establezca la versión de la plataforma en **.NET Framework 4.7.2 o posterior** y, después, haga clic en **Crear**.
   ![Escriba el nombre y la versión de la plataforma en Visual Studio](media/name-wpf-project-visual-studio.png)

Se ha creado el proyecto. Observará que hay dos pestañas abiertas: `MainWindow.xaml` y `MainWindow.xaml.cs`. A lo largo de este tutorial, iremos agregando código a estos dos archivos. Modificaremos `MainWindow.xaml` para la interfaz de usuario de la aplicación. Modificaremos `MainWindow.xaml.cs` para las llamadas a Traductor y Bing Spell Check.
   ![Revisión del entorno](media/blank-wpf-project.png)

En la siguiente sección vamos a agregar ensamblados y un paquete NuGet a nuestro proyecto para una funcionalidad adicional, como el análisis de JSON.

## <a name="add-references-and-nuget-packages-to-your-project"></a>Adición de referencias y paquetes NuGet al proyecto

Nuestro proyecto requiere un grupo de ensamblados de .NET Framework y NewtonSoft.Json, que se van a instalar mediante el administrador de paquetes NuGet.

### <a name="add-net-framework-assemblies"></a>Adición de ensamblados de .NET Framework

Agreguemos ensamblados a nuestro proyecto para serializar y deserializar objetos, y para administrar solicitudes y respuestas HTTP.

1. Busque el proyecto en el Explorador de soluciones de Visual Studio. Haga clic con el botón derecho en el proyecto y seleccione **Agregar > Referencia**, que abrirá **Administrador de referencias**.
1. La pestaña **Ensamblados** muestra todos los ensamblados de .NET Framework que están disponibles como referencia. Use la barra de búsqueda de la esquina superior derecha para buscar referencias.
   ![Adición de referencias de ensamblados](media/add-assemblies-2019.png)
1. Seleccione las siguientes referencias para el proyecto:
   * [System.Runtime.Serialization](https://docs.microsoft.com/dotnet/api/system.runtime.serialization)
   * [System.Web](https://docs.microsoft.com/dotnet/api/system.web)
   * System.Web.Extensions
   * [System.Windows](https://docs.microsoft.com/dotnet/api/system.windows)
1. Después de agregar estas referencias al proyecto, haga clic en **Aceptar** para cerrar **Administrador de referencias**.

> [!NOTE]
> Para más información sobre las referencias de ensamblado, consulte [Procedimiento: Agregar o quitar referencias con el Administrador de referencias](https://docs.microsoft.com/visualstudio/ide/how-to-add-or-remove-references-by-using-the-reference-manager?view=vs-2019).

### <a name="install-newtonsoftjson"></a>Instalación de NewtonSoft.Json

Nuestra aplicación usará NewtonSoft.Json para deserializar los objetos JSON. Siga estas instrucciones para instalar el paquete.

1. Busque el proyecto en el Explorador de soluciones de Visual Studio y haga clic con el botón derecho en el proyecto. Seleccione **Administrar paquetes NuGet**.
1. Busque y seleccione la pestaña **Examinar**.
1. Escriba [NewtonSoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/) en la barra de búsqueda.

    ![Búsqueda e instalación de Newtonsoft.Json](media/nuget-package-manager.png)

1. Seleccione el paquete y haga clic en **Instalar**.
1. Cuando la instalación se haya completado, cierre la pestaña.

## <a name="create-a-wpf-form-using-xaml"></a>Creación de un formulario WPF mediante XAML

Para usar la aplicación, va a necesitar una interfaz de usuario. Con XAML, vamos a crear un formulario que permite a los usuarios seleccionar los idiomas de entrada y de traducción, introducir el texto que se va a traducir y mostrar el resultado de la traducción.

Veamos lo que estamos creando.

![Interfaz de usuario de WPF XAML](media/translator-text-csharp-xaml.png)

La interfaz de usuario incluye los siguientes componentes:

| Nombre | Tipo | Descripción |
|------|------|-------------|
| `FromLanguageComboBox` | ComboBox | Muestra una lista de los idiomas admitidos por Microsoft Translator para la traducción de texto. El usuario selecciona el idioma del que está traduciendo. |
| `ToLanguageComboBox` | ComboBox | Muestra la misma lista de idiomas que `FromComboBox`, pero se utiliza para seleccionar el idioma al que se traduce. |
| `TextToTranslate` | TextBox | Permite al usuario escribir texto que se va a traducir. |
| `TranslateButton` | Botón | Use este botón para traducir texto. |
| `TranslatedTextLabel` | Etiqueta | Muestra la traducción. |
| `DetectedLanguageLabel` | Etiqueta | Muestra el idioma detectado del texto que se va a traducir (`TextToTranslate`). |

> [!NOTE]
> Estamos creando este formulario utilizando el código fuente de XAML; sin embargo, puede crear el formulario con el editor en Visual Studio.

Vamos a agregar el código al proyecto.

1. En Visual Studio, seleccione la pestaña para `MainWindow.xaml`.
1. Copie este código en el proyecto y, después, seleccione **Archivo > Guardar MainWindow.xaml** para guardar los cambios.
   ```xaml
   <Window x:Class="MSTranslatorDemo.MainWindow"
           xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
           xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
           xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
           xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
           xmlns:local="clr-namespace:MSTranslatorDemo"
           mc:Ignorable="d"
           Title="Microsoft Translator" Height="400" Width="700" BorderThickness="0">
       <Grid>
           <Label x:Name="label" Content="Microsoft Translator" HorizontalAlignment="Left" Margin="39,6,0,0" VerticalAlignment="Top" Height="49" FontSize="26.667"/>
           <TextBox x:Name="TextToTranslate" HorizontalAlignment="Left" Height="23" Margin="42,160,0,0" TextWrapping="Wrap" VerticalAlignment="Top" Width="600" FontSize="14" TabIndex="3"/>
           <Label x:Name="EnterTextLabel" Content="Text to translate:" HorizontalAlignment="Left" Margin="40,129,0,0" VerticalAlignment="Top" FontSize="14"/>
           <Label x:Name="toLabel" Content="Translate to:" HorizontalAlignment="Left" Margin="304,58,0,0" VerticalAlignment="Top" FontSize="14"/>

           <Button x:Name="TranslateButton" Content="Translate" HorizontalAlignment="Left" Margin="39,206,0,0" VerticalAlignment="Top" Width="114" Height="31" Click="TranslateButton_Click" FontSize="14" TabIndex="4" IsDefault="True"/>
           <ComboBox x:Name="ToLanguageComboBox"
                   HorizontalAlignment="Left"
                   Margin="306,88,0,0"
                   VerticalAlignment="Top"
                   Width="175" FontSize="14" TabIndex="2">

           </ComboBox>
           <Label x:Name="fromLabel" Content="Translate from:" HorizontalAlignment="Left" Margin="40,58,0,0" VerticalAlignment="Top" FontSize="14"/>
           <ComboBox x:Name="FromLanguageComboBox"
               HorizontalAlignment="Left"
               Margin="42,88,0,0"
               VerticalAlignment="Top"
               Width="175" FontSize="14" TabIndex="1"/>
           <Label x:Name="TranslatedTextLabel" Content="Translation is displayed here." HorizontalAlignment="Left" Margin="39,255,0,0" VerticalAlignment="Top" Width="620" FontSize="14" Height="85" BorderThickness="0"/>
           <Label x:Name="DetectedLanguageLabel" Content="Autodetected language is displayed here." HorizontalAlignment="Left" Margin="39,288,0,0" VerticalAlignment="Top" Width="620" FontSize="14" Height="84" BorderThickness="0"/>
       </Grid>
   </Window>
   ```
Ahora debería ver una vista previa de la interfaz de usuario de la aplicación en Visual Studio. El archivo debe tener un aspecto similar a la imagen anterior.

Eso es todo, el formulario está listo. Ahora vamos a escribir código para usar Translator Text y Bing Spell Check.

> [!NOTE]
> Puede adaptar este formulario o crear el suyo propio.

## <a name="create-your-app"></a>Creación de la aplicación

`MainWindow.xaml.cs` contiene el código que controla la aplicación. En las próximas secciones, se va a agregar código para rellenar los menús desplegables, y para llamar a un grupo de API expuestas por Traductor y Bing Spell Check.

* Cuando el programa se inicia y se crea una instancia de `MainWindow`, se llama al método `Languages` de Traductor para recuperar y rellenar las listas desplegables de selección de idioma. Esta tarea se realiza una vez al principio de cada sesión.
* Cuando se hace clic en el botón **Traducir**, se recupera el texto y la selección del idioma del usuario, se realiza una corrección ortográfica en la entrada y se muestran al usuario la traducción y el idioma detectado.
  * Se llama al método `Translate` de Traductor para traducir el texto de `TextToTranslate`. Esta llamada también incluye los idiomas `to` y `from` seleccionados mediante los menús desplegables.
  * Se llama al método `Detect` de Traductor para determinar el idioma del texto de `TextToTranslate`.
  * Bing Spell Check se utiliza para validar `TextToTranslate` y ajustar los errores de ortografía.

Todo nuestro proyecto se encapsula en la clase `MainWindow : Window`. Comencemos por agregar código para establecer la clave de suscripción, declarar los puntos de conexión para Traductor y Bing Spell Check, e inicializar la aplicación.

1. En Visual Studio, seleccione la pestaña para `MainWindow.xaml.cs`.
1. Reemplace las instrucciones `using` previamente rellenadas por lo siguiente.  
   ```csharp
   using System;
   using System.Windows;
   using System.Net;
   using System.Net.Http;
   using System.IO;
   using System.Collections.Generic;
   using System.Linq;
   using System.Text;
   using Newtonsoft.Json;
   ```
1. Busque la clase `MainWindow : Window` y reemplácela por este código:
   ```csharp
   {
       // This sample uses the Cognitive Services subscription key for all services. To learn more about
       // authentication options, see: https://docs.microsoft.com/azure/cognitive-services/authentication.
       const string COGNITIVE_SERVICES_KEY = "YOUR_COG_SERVICES_KEY";
       // Endpoints for Translator and Bing Spell Check
       public static readonly string TEXT_TRANSLATION_API_ENDPOINT = "https://api.cognitive.microsofttranslator.com/{0}?api-version=3.0";
       const string BING_SPELL_CHECK_API_ENDPOINT = "https://westus.api.cognitive.microsoft.com/bing/v7.0/spellcheck/";
       // An array of language codes
       private string[] languageCodes;

       // Dictionary to map language codes from friendly name (sorted case-insensitively on language name)
       private SortedDictionary<string, string> languageCodesAndTitles =
           new SortedDictionary<string, string>(Comparer<string>.Create((a, b) => string.Compare(a, b, true)));

       // Global exception handler to display error message and exit
       private static void HandleExceptions(object sender, UnhandledExceptionEventArgs args)
       {
           Exception e = (Exception)args.ExceptionObject;
           MessageBox.Show("Caught " + e.Message, "Error", MessageBoxButton.OK, MessageBoxImage.Error);
           System.Windows.Application.Current.Shutdown();
       }
       // MainWindow constructor
       public MainWindow()
       {
           // Display a message if unexpected error is encountered
           AppDomain.CurrentDomain.UnhandledException += new UnhandledExceptionEventHandler(HandleExceptions);

           if (COGNITIVE_SERVICES_KEY.Length != 32)
           {
               MessageBox.Show("One or more invalid API subscription keys.\n\n" +
                   "Put your keys in the *_API_SUBSCRIPTION_KEY variables in MainWindow.xaml.cs.",
                   "Invalid Subscription Key(s)", MessageBoxButton.OK, MessageBoxImage.Error);
               System.Windows.Application.Current.Shutdown();
           }
           else
           {
               // Start GUI
               InitializeComponent();
               // Get languages for drop-downs
               GetLanguagesForTranslate();
               // Populate drop-downs with values from GetLanguagesForTranslate
               PopulateLanguageMenus();
           }
       }
   // NOTE:
   // In the following sections, we'll add code below this.
   }
   ```
1. Agregue la clave de suscripción de Cognitive Services y guárdela.

En este bloque de código, hemos declarado dos variables de miembro que contienen información sobre los idiomas disponibles para traducción:

| Variable | Tipo | Descripción |
|----------|------|-------------|
|`languageCodes` | Matriz de cadenas |Almacena en caché los códigos de idioma. El servicio de Translator utiliza códigos cortos, como `en` para inglés, para identificar los idiomas. |
|`languageCodesAndTitles` | Diccionario ordenado | Asigna los nombres "descriptivos" en la interfaz de usuario a los códigos cortos que se utilizan en la API. Se mantienen ordenados por orden alfabético sin tener en cuenta las mayúsculas y minúsculas. |

A continuación, en el constructor `MainWindow`, hemos agregado el control de errores con `HandleExceptions`. Este control de errores asegura que se proporcione una alerta si no se controla una excepción. A continuación, se ejecuta una comprobación para confirmar que la clave de suscripción proporcionada tiene 32 caracteres. Se produce un error si la clave tiene menos o más de 32 caracteres.

Si hay claves que tengan al menos la longitud adecuada, la llamada `InitializeComponent()` da inicio a la interfaz de usuario al ubicar, cargar y crear instancias de la descripción de XAML de la ventana de la aplicación principal.

Por último, hemos agregado código a métodos de llamada para recuperar los idiomas para traducir y para rellenar los menús desplegables de la interfaz de usuario de la aplicación. Veremos el código subyacente de estas llamadas en breve.

## <a name="get-supported-languages"></a>Obtener idiomas admitidos

El Traductor admite actualmente más de 70 idiomas. Dado que, con el tiempo, se agregará nueva compatibilidad de idiomas, recomendamos llamar al recurso de idiomas que expone Traductor en lugar de codificar la lista de idiomas de la aplicación.

En esta sección, se va a crear una solicitud `GET` al recurso de idiomas, especificando que queremos una lista de idiomas disponibles para traducir.

> [!NOTE]
> El recurso de idiomas permite filtrar la compatibilidad de idiomas con los siguientes parámetros de consulta: transliteración, diccionario y traducción. Para más información, consulte la [referencia de API](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-languages).

Antes de continuar, echemos un vistazo a una salida de ejemplo para una llamada al recurso de idiomas:

```json
{
  "translation": {
    "af": {
      "name": "Afrikaans",
      "nativeName": "Afrikaans",
      "dir": "ltr"
    },
    "ar": {
      "name": "Arabic",
      "nativeName": "العربية",
      "dir": "rtl"
    }
    // Additional languages are provided in the full JSON output.
}
```

En esta salida, podemos extraer el código de idioma y el `name` de un idioma específico. La aplicación usa NewtonSoft.Json para deserializar el objeto JSON ([`JsonConvert.DeserializeObject`](https://www.newtonsoft.com/json/help/html/M_Newtonsoft_Json_JsonConvert_DeserializeObject__1.htm)).

Retomando el punto donde se dejó en la última sección, agregamos un método para obtener idiomas compatibles con la aplicación.

1. En Visual Studio, abra la pestaña para `MainWindow.xaml.cs`.
2. Agregue este código al proyecto:
   ```csharp
   // ***** GET TRANSLATABLE LANGUAGE CODES
   private void GetLanguagesForTranslate()
   {
       // Send request to get supported language codes
       string uri = String.Format(TEXT_TRANSLATION_API_ENDPOINT, "languages") + "&scope=translation";
       WebRequest WebRequest = WebRequest.Create(uri);
       WebRequest.Headers.Add("Accept-Language", "en");
       WebResponse response = null;
       // Read and parse the JSON response
       response = WebRequest.GetResponse();
       using (var reader = new StreamReader(response.GetResponseStream(), UnicodeEncoding.UTF8))
       {
           var result = JsonConvert.DeserializeObject<Dictionary<string, Dictionary<string, Dictionary<string, string>>>>(reader.ReadToEnd());
           var languages = result["translation"];

           languageCodes = languages.Keys.ToArray();
           foreach (var kv in languages)
           {
               languageCodesAndTitles.Add(kv.Value["name"], kv.Key);
           }
       }
   }
   // NOTE:
   // In the following sections, we'll add code below this.
   ```

El método `GetLanguagesForTranslate()` crea una solicitud HTTP GET y utiliza el parámetro de cadena de consulta `scope=translation` para limitar el ámbito de la solicitud a los idiomas admitidos para traducción. Se agrega el encabezado `Accept-Language` con el valor `en` para que se devuelvan los idiomas admitidos en inglés.

La respuesta JSON se analiza y se convierte en un diccionario. Después, se agregan los códigos de idioma a la variable de miembro `languageCodes`. Se recorren los pares de clave/valor que contienen los códigos de idioma y los nombres de idioma descriptivos, y se agregan a la variable miembro `languageCodesAndTitles`. En los menús desplegables del formulario se muestran los nombres descriptivos, pero se necesitan los códigos para solicitar la traducción.

## <a name="populate-language-drop-down-menus"></a>Rellenado de los menús desplegables de idioma

La interfaz de usuario se define mediante XAML, por lo que no es necesario hacer mucho para configurarla, además de llamar a `InitializeComponent()`. Lo único que necesita hacer es agregar los nombres de idiomas descriptivos en los menús desplegables **Translate from** (Traducir de) y **Translate to** (Traducir a). El método `PopulateLanguageMenus()` agrega los nombres.

1. En Visual Studio, abra la pestaña para `MainWindow.xaml.cs`.
2. Agregue este código al proyecto debajo del método `GetLanguagesForTranslate()`:
   ```csharp
   private void PopulateLanguageMenus()
   {
       // Add option to automatically detect the source language
       FromLanguageComboBox.Items.Add("Detect");

       int count = languageCodesAndTitles.Count;
       foreach (string menuItem in languageCodesAndTitles.Keys)
       {
           FromLanguageComboBox.Items.Add(menuItem);
           ToLanguageComboBox.Items.Add(menuItem);
       }

       // Set default languages
       FromLanguageComboBox.SelectedItem = "Detect";
       ToLanguageComboBox.SelectedItem = "English";
   }
   // NOTE:
   // In the following sections, we'll add code below this.
   ```

Este método itera el diccionario `languageCodesAndTitles` y agrega cada clave a ambos menús. Una vez rellenados los menús, los idiomas de origen y destino predeterminados se establecen en **Detectar** e **Inglés**, respectivamente.

> [!TIP]
> Sin una selección predeterminada para los menús, el usuario puede hacer clic en **Translate** (Traducir) sin elegir el idioma de origen ni el de destino. Los valores predeterminados eliminan la necesidad de tratar este problema.

Ahora que `MainWindow` se ha inicializado y se ha creado la interfaz de usuario, este código no se ejecutará hasta que se haga clic en el botón **Translate** (Traducir).

## <a name="detect-language-of-source-text"></a>Detección del idioma del texto de origen

Ahora vamos a crear un método para detectar el idioma del texto de origen (texto introducido en nuestra área de texto) mediante Traductor. El valor devuelto por esta solicitud se usará más adelante en la solicitud de traducción.

1. En Visual Studio, abra la pestaña para `MainWindow.xaml.cs`.
2. Agregue este código al proyecto debajo del método `PopulateLanguageMenus()`:
   ```csharp
   // ***** DETECT LANGUAGE OF TEXT TO BE TRANSLATED
   private string DetectLanguage(string text)
   {
       string detectUri = string.Format(TEXT_TRANSLATION_API_ENDPOINT ,"detect");

       // Create request to Detect languages with Translator
       HttpWebRequest detectLanguageWebRequest = (HttpWebRequest)WebRequest.Create(detectUri);
       detectLanguageWebRequest.Headers.Add("Ocp-Apim-Subscription-Key", COGNITIVE_SERVICES_KEY);
       detectLanguageWebRequest.Headers.Add("Ocp-Apim-Subscription-Region", "westus");
       detectLanguageWebRequest.ContentType = "application/json; charset=utf-8";
       detectLanguageWebRequest.Method = "POST";

       // Send request
       var serializer = new System.Web.Script.Serialization.JavaScriptSerializer();
       string jsonText = serializer.Serialize(text);

       string body = "[{ \"Text\": " + jsonText + " }]";
       byte[] data = Encoding.UTF8.GetBytes(body);

       detectLanguageWebRequest.ContentLength = data.Length;

       using (var requestStream = detectLanguageWebRequest.GetRequestStream())
           requestStream.Write(data, 0, data.Length);

       HttpWebResponse response = (HttpWebResponse)detectLanguageWebRequest.GetResponse();

       // Read and parse JSON response
       var responseStream = response.GetResponseStream();
       var jsonString = new StreamReader(responseStream, Encoding.GetEncoding("utf-8")).ReadToEnd();
       dynamic jsonResponse = serializer.DeserializeObject(jsonString);

       // Fish out the detected language code
       var languageInfo = jsonResponse[0];
       if (languageInfo["score"] > (decimal)0.5)
       {
           DetectedLanguageLabel.Content = languageInfo["language"];
           return languageInfo["language"];
       }
       else
           return "Unable to confidently detect input language.";
   }
   // NOTE:
   // In the following sections, we'll add code below this.
   ```

Este método crea una solicitud HTTP `POST` en el recurso de detección. Toma un único argumento, `text`, que se pasa como el cuerpo de la solicitud. Más tarde, cuando creamos nuestra solicitud de traducción, el texto introducido en la interfaz de usuario pasará a este método para la detección del idioma.

Además, este método evalúa la puntuación de confianza de la respuesta. Si la puntuación es superior a `0.5`, el idioma detectado se muestra en la interfaz de usuario.

## <a name="spell-check-the-source-text"></a>Corrección ortográfica del texto de origen

Ahora vamos a crear un método para corregir la ortografía del texto de origen con Bing Spell Check API. El corrector ortográfico garantiza que obtendremos traducciones precisas de Traductor. Todas las correcciones del texto de origen se pasan a la solicitud de traducción cuando se hace clic en el botón **Traducir**.

1. En Visual Studio, abra la pestaña para `MainWindow.xaml.cs`.
2. Agregue este código al proyecto debajo del método `DetectLanguage()`:

```csharp
// ***** CORRECT SPELLING OF TEXT TO BE TRANSLATED
private string CorrectSpelling(string text)
{
    string uri = BING_SPELL_CHECK_API_ENDPOINT + "?mode=spell&mkt=en-US";

    // Create a request to Bing Spell Check API
    HttpWebRequest spellCheckWebRequest = (HttpWebRequest)WebRequest.Create(uri);
    spellCheckWebRequest.Headers.Add("Ocp-Apim-Subscription-Key", COGNITIVE_SERVICES_KEY);
    spellCheckWebRequest.Method = "POST";
    spellCheckWebRequest.ContentType = "application/x-www-form-urlencoded"; // doesn't work without this

    // Create and send the request
    string body = "text=" + System.Web.HttpUtility.UrlEncode(text);
    byte[] data = Encoding.UTF8.GetBytes(body);
    spellCheckWebRequest.ContentLength = data.Length;
    using (var requestStream = spellCheckWebRequest.GetRequestStream())
        requestStream.Write(data, 0, data.Length);
    HttpWebResponse response = (HttpWebResponse)spellCheckWebRequest.GetResponse();

    // Read and parse the JSON response; get spelling corrections
    var serializer = new System.Web.Script.Serialization.JavaScriptSerializer();
    var responseStream = response.GetResponseStream();
    var jsonString = new StreamReader(responseStream, Encoding.GetEncoding("utf-8")).ReadToEnd();
    dynamic jsonResponse = serializer.DeserializeObject(jsonString);
    var flaggedTokens = jsonResponse["flaggedTokens"];

    // Construct sorted dictionary of corrections in reverse order (right to left)
    // This ensures that changes don't impact later indexes
    var corrections = new SortedDictionary<int, string[]>(Comparer<int>.Create((a, b) => b.CompareTo(a)));
    for (int i = 0; i < flaggedTokens.Length; i++)
    {
        var correction = flaggedTokens[i];
        var suggestion = correction["suggestions"][0];  // Consider only first suggestion
        if (suggestion["score"] > (decimal)0.7)         // Take it only if highly confident
            corrections[(int)correction["offset"]] = new string[]   // dict key   = offset
                { correction["token"], suggestion["suggestion"] };  // dict value = {error, correction}
    }

    // Apply spelling corrections, in order, from right to left
    foreach (int i in corrections.Keys)
    {
        var oldtext = corrections[i][0];
        var newtext = corrections[i][1];

        // Apply capitalization from original text to correction - all caps or initial caps
        if (text.Substring(i, oldtext.Length).All(char.IsUpper)) newtext = newtext.ToUpper();
        else if (char.IsUpper(text[i])) newtext = newtext[0].ToString().ToUpper() + newtext.Substring(1);

        text = text.Substring(0, i) + newtext + text.Substring(i + oldtext.Length);
    }
    return text;
}
// NOTE:
// In the following sections, we'll add code below this.
```

## <a name="translate-text-on-click"></a>Traducción de texto al hacer clic

Lo último que tenemos que hacer es crear un método que se invoque cuando se hace clic en el botón **Traducir** de la interfaz de usuario.

1. En Visual Studio, abra la pestaña para `MainWindow.xaml.cs`.
1. Agregue este código al proyecto debajo del método `CorrectSpelling()` y guárdelo:  
   ```csharp
   // ***** PERFORM TRANSLATION ON BUTTON CLICK
   private async void TranslateButton_Click(object sender, EventArgs e)
   {
       string textToTranslate = TextToTranslate.Text.Trim();

       string fromLanguage = FromLanguageComboBox.SelectedValue.ToString();
       string fromLanguageCode;

       // auto-detect source language if requested
       if (fromLanguage == "Detect")
       {
           fromLanguageCode = DetectLanguage(textToTranslate);
           if (!languageCodes.Contains(fromLanguageCode))
           {
               MessageBox.Show("The source language could not be detected automatically " +
                   "or is not supported for translation.", "Language detection failed",
                   MessageBoxButton.OK, MessageBoxImage.Error);
               return;
           }
       }
       else
           fromLanguageCode = languageCodesAndTitles[fromLanguage];

       string toLanguageCode = languageCodesAndTitles[ToLanguageComboBox.SelectedValue.ToString()];

       // spell-check the source text if the source language is English
       if (fromLanguageCode == "en")
       {
           if (textToTranslate.StartsWith("-"))    // don't spell check in this case
               textToTranslate = textToTranslate.Substring(1);
           else
           {
               textToTranslate = CorrectSpelling(textToTranslate);
               TextToTranslate.Text = textToTranslate;     // put corrected text into input field
           }
       }
       // handle null operations: no text or same source/target languages
       if (textToTranslate == "" || fromLanguageCode == toLanguageCode)
       {
           TranslatedTextLabel.Content = textToTranslate;
           return;
       }

       // send HTTP request to perform the translation
       string endpoint = string.Format(TEXT_TRANSLATION_API_ENDPOINT, "translate");
       string uri = string.Format(endpoint + "&from={0}&to={1}", fromLanguageCode, toLanguageCode);

       System.Object[] body = new System.Object[] { new { Text = textToTranslate } };
       var requestBody = JsonConvert.SerializeObject(body);

       using (var client = new HttpClient())
       using (var request = new HttpRequestMessage())
       {
           request.Method = HttpMethod.Post;
           request.RequestUri = new Uri(uri);
           request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
           request.Headers.Add("Ocp-Apim-Subscription-Key", COGNITIVE_SERVICES_KEY);
           request.Headers.Add("Ocp-Apim-Subscription-Region", "westus");
           request.Headers.Add("X-ClientTraceId", Guid.NewGuid().ToString());

           var response = await client.SendAsync(request);
           var responseBody = await response.Content.ReadAsStringAsync();

           var result = JsonConvert.DeserializeObject<List<Dictionary<string, List<Dictionary<string, string>>>>>(responseBody);
           var translation = result[0]["translations"][0]["text"];

           // Update the translation field
           TranslatedTextLabel.Content = translation;
       }
   }
   ```

El primer paso es obtener los idiomas de origen y destino, así como el texto que el usuario introdujo en nuestro formulario. Si el idioma de origen se establece en **Detect** (Detectar), se llama a `DetectLanguage()` para determinar el idioma del texto de origen. El texto puede estar en un idioma que Traductor no admita. En ese caso, se muestra un mensaje para informar al usuario y devolver el texto sin traducir.

Si el idioma de origen es el inglés (así se haya especificado o detectado), se revisa la ortografía del texto con `CorrectSpelling()` y se aplican las correcciones. El texto corregido se agrega de nuevo en el área de texto para que el usuario vea que se ha realizado una corrección.

El código para traducir el texto debe parecer familiar: crear el identificador URI, crear una solicitud, enviarla y analizar la respuesta. La matriz JSON puede contener más de un objeto para traducir; sin embargo, esta aplicación solo requiere uno.

Después de una solicitud correcta, `TranslatedTextLabel.Content` se reemplaza por `translation`, que actualiza la interfaz de usuario para mostrar el texto traducido.

## <a name="run-your-wpf-app"></a>Ejecución de la aplicación de WPF

Y eso es todo, ahora tenemos una aplicación de traducción que funciona creada con WPF. Para ejecutar la aplicación, haga clic en el botón **Iniciar** en Visual Studio.

## <a name="source-code"></a>Código fuente

El código fuente del proyecto está disponible en GitHub.

* [Exploración del código fuente](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-C-Sharp-Tutorial)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Referencia sobre Traductor de Microsoft](https://docs.microsoft.com/azure/cognitive-services/Translator/reference/v3-0-reference)
