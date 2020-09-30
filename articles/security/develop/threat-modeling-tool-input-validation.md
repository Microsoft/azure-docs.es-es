---
title: 'Validación de entrada: Microsoft Threat Modeling Tool (Azure) | Microsoft Docs'
description: Más información sobre la validación de entradas en Threat Modeling Tool. Consulte la información de mitigación y los ejemplos de código.
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.subservice: security-develop
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/07/2017
ms.author: jegeib
ms.custom: devx-track-csharp
ms.openlocfilehash: 3bb944badfbdffd703672f9e78619c70a148aae2
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89293360"
---
# <a name="security-frame-input-validation--mitigations"></a>Marco de seguridad: Validación de entrada | Mitigaciones 
| Producto o servicio | Artículo |
| --------------- | ------- |
| **Aplicación web** | <ul><li>[Deshabilite el scripting XSLT para todas las transformaciones mediante hojas de estilos no de confianza](#disable-xslt)</li><li>[Compruebe que cada página que pueda incluir contenido controlable por el usuario quede excluida del rastreo de MIME automático](#out-sniffing)</li><li>[Proteja o deshabilite la resolución de entidades XML](#xml-resolution)</li><li>[Las aplicaciones que usan http.sys realizan la comprobación de canonización de URL](#app-verification)</li><li>[Asegúrese de que los controles adecuados estén en vigor al aceptar archivos de usuarios](#controls-users)</li><li>[Asegúrese de que se usen parámetros con seguridad de tipos en la aplicación web para el acceso a datos](#typesafe)</li><li>[Use clases de enlace de modelos distintas o listas de filtros de enlace para evitar la vulnerabilidad de asignación masiva de MVC](#binding-mvc)</li><li>[Codifique la salida web que no sea de confianza antes de representarla](#rendering)</li><li>[Lleve a cabo la validación de entrada y el filtrado para todas las propiedades de modelo de tipo cadena](#typemodel)</li><li>[Se debería aplicar la comprobación de estado a los campos de formulario que acepten todos los caracteres, por ejemplo, editor de texto enriquecido](#richtext)</li><li>[No asigne elementos DOM a receptores que carezcan de codificación integrada](#inbuilt-encode)</li><li>[Valide que todos los redireccionamientos dentro de la aplicación se cierren o se realicen de forma segura](#redirect-safe)</li><li>[Implemente la validación de entrada en todos los parámetros de tipo cadena aceptados por métodos de controlador](#string-method)</li><li>[Establezca el tiempo de espera del límite superior para el procesamiento de expresiones regulares con el fin de impedir ataques DoS causados por expresiones regulares incorrectas](#dos-expression)</li><li>[No use Html.Raw en las vistas Razor](#html-razor)</li></ul> | 
| **Base de datos** | <ul><li>[No use consultas dinámicas en procedimientos almacenados](#stored-proc)</li></ul> |
| **API web** | <ul><li>[Asegúrese de que se lleve a cabo la validación del modelo en métodos de API web](#validation-api)</li><li>[Implemente la validación de entrada en todos los parámetros de tipo cadena aceptados por métodos de API web](#string-api)</li><li>[Asegúrese de que se usen parámetros con seguridad de tipos en la API web para el acceso a datos](#typesafe-api)</li></ul> | 
| **Azure Document DB** | <ul><li>[Uso de consultas SQL parametrizadas con Azure Cosmos DB](#sql-docdb)</li></ul> | 
| **WCF** | <ul><li>[Validación de entrada de WCF mediante enlaces de esquema](#schema-binding)</li><li>[Validación de entrada de WCF mediante inspectores de parámetros](#parameters)</li></ul> |

## <a name="disable-xslt-scripting-for-all-transforms-using-untrusted-style-sheets"></a><a id="disable-xslt"></a>Deshabilite el scripting XSLT para todas las transformaciones mediante hojas de estilos no de confianza

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Seguridad XSLT](https://msdn.microsoft.com/library/ms763800(v=vs.85).aspx), [Propiedad XsltSettings.EnableScript](https://msdn.microsoft.com/library/system.xml.xsl.xsltsettings.enablescript.aspx) |
| **Pasos** | XSLT es compatible con el scripting dentro de hojas de estilos mediante el elemento `<msxml:script>`. Esto permite que se usen funciones personalizadas en una transformación XSLT. El script se ejecuta en el contexto del proceso que lleva a cabo la transformación. El script XSLT debe estar deshabilitado cuando se encuentra en un entorno que no sea de confianza para evitar la ejecución de código no de confianza. *Si usa .NET:* El scripting XSLT está deshabilitado de forma predeterminada; sin embargo, debe asegurarse de que no se haya habilitado explícitamente por medio de la propiedad `XsltSettings.EnableScript`.|

### <a name="example"></a>Ejemplo 

```csharp
XsltSettings settings = new XsltSettings();
settings.EnableScript = true; // WRONG: THIS SHOULD BE SET TO false
```

### <a name="example"></a>Ejemplo
Si usa MSXML 6.0, el scripting XSLT está deshabilitado de forma predeterminada. Aún así, tiene que asegurarse de que no se haya habilitado explícitamente por medio de la propiedad AllowXsltScript del objeto DOM XML. 

```csharp
doc.setProperty("AllowXsltScript", true); // WRONG: THIS SHOULD BE SET TO false
```

### <a name="example"></a>Ejemplo
Si usa MSXML 5 o una versión anterior, el scripting XSLT está habilitado de forma predeterminada y debe deshabilitarlo explícitamente. Establezca la propiedad AllowXsltScript del objeto DOM XML en false. 

```csharp
doc.setProperty("AllowXsltScript", false); // CORRECT. Setting to false disables XSLT scripting.
```

## <a name="ensure-that-each-page-that-could-contain-user-controllable-content-opts-out-of-automatic-mime-sniffing"></a><a id="out-sniffing"></a>Compruebe que cada página que pueda incluir contenido controlable por el usuario quede excluida del rastreo de MIME automático

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [IE8 Security Part V - Comprehensive Protection](https://docs.microsoft.com/archive/blogs/ie/ie8-security-part-v-comprehensive-protection) (Seguridad de IE8 (parte V): Protección completa)  |
| **Pasos** | <p>Para cada página que podría incluir contenido controlable por el usuario, debe usar el encabezado HTTP `X-Content-Type-Options:nosniff`. Para cumplir este requisito, puede establecer el encabezado necesario página a página solo para aquellas que puedan incluir contenido controlable por el usuario o puede establecerlo globalmente para todas las páginas de la aplicación.</p><p>Cada tipo de archivo que se entregue desde un servidor web tiene asociado un [tipo MIME](https://en.wikipedia.org/wiki/Mime_type) (también llamado *content-type*) que describe la naturaleza del contenido (es decir, imagen, texto, aplicación, etc.).</p><p>El encabezado X-Content-Type-Options es un encabezado HTTP que permite a los desarrolladores especificar que su contenido no se debe someter a un rastreo de MIME. Este encabezado se ha diseñado para mitigar los ataques de rastreo de MIME. Se agregó compatibilidad con este encabezado en Internet Explorer 8 (IE8).</p><p>Los únicos que se beneficiarán de X-Content-Type-Options son los usuarios de Internet Explorer 8 (IE8). Las versiones anteriores de Internet Explorer no respetan actualmente el encabezado X-Content-Type-Options.</p><p>Internet Explorer 8 (y versiones posteriores) es el único explorador importante que implementa una característica para excluir del rastreo de MIME. Cuando otros exploradores importantes (Firefox, Safari, Chrome) implementen características similares, si lo hacen, se actualizará esta recomendación para incluir también la sintaxis de esos exploradores.</p>|

### <a name="example"></a>Ejemplo
Para habilitar de forma global el encabezado necesario para todas las páginas de la aplicación, puede realizar una de las siguientes acciones: 

* Agregue el encabezado en el archivo Web.config si la aplicación está hospedada con Internet Information Services (IIS) 7. 

```
<system.webServer> 
  <httpProtocol> 
    <customHeaders> 
      <add name=""X-Content-Type-Options"" value=""nosniff""/>
    </customHeaders>
  </httpProtocol>
</system.webServer> 
```

* Agregue el encabezado mediante el método Application\_BeginRequest global. 

``` 
void Application_BeginRequest(object sender, EventArgs e)
{
  this.Response.Headers[""X-Content-Type-Options""] = ""nosniff"";
} 
```

* Implemente un módulo HTTP personalizado 

``` 
public class XContentTypeOptionsModule : IHttpModule 
  {
    #region IHttpModule Members 
    public void Dispose() 
    { 

    } 
    public void Init(HttpApplication context)
    { 
      context.PreSendRequestHeaders += newEventHandler(context_PreSendRequestHeaders); 
    } 
    #endregion 
    void context_PreSendRequestHeaders(object sender, EventArgs e) 
      { 
        HttpApplication application = sender as HttpApplication; 
        if (application == null) 
          return; 
        if (application.Response.Headers[""X-Content-Type-Options ""] != null) 
          return; 
        application.Response.Headers.Add(""X-Content-Type-Options "", ""nosniff""); 
      } 
  } 

``` 

* Puede habilitar el encabezado necesario solo para páginas específicas si lo agrega a respuestas individuales: 

```
this.Response.Headers[""X-Content-Type-Options""] = ""nosniff""; 
``` 

## <a name="harden-or-disable-xml-entity-resolution"></a><a id="xml-resolution"></a>Proteja o deshabilite la resolución de entidades XML

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [XML Entity Expansion](https://capec.mitre.org/data/definitions/197.html) (Expansión de entidad XML), [Ataques por denegación de servicio y defensas XML](https://msdn.microsoft.com/magazine/ee335713.aspx), [Información general de seguridad MSXML](https://msdn.microsoft.com/library/ms754611(v=VS.85).aspx), [Prácticas recomendadas para proteger el código MSXML](https://msdn.microsoft.com/library/ms759188(VS.85).aspx), [NSXMLParserDelegate Protocol Reference](https://developer.apple.com/library/ios/#documentation/cocoa/reference/NSXMLParserDelegate_Protocol/Reference/Reference.html) (Referencia del protocolo NSXMLParserDelegate), [Resolver recursos externos](https://msdn.microsoft.com/library/5fcwybb2.aspx) |
| **Pasos**| <p>Aunque no está generalizada, hay una característica de XML que permite al analizador XML expandir las entidades de macro con valores definidos dentro del propio documento o desde orígenes externos. Por ejemplo, el documento podría definir una entidad "companyname" con el valor "Microsoft", de forma que, cada vez que aparezca el texto "&companyname;" en el documento, se reemplace automáticamente por el texto Microsoft. O bien, el documento podría definir una entidad "MSFTStock" que hace referencia a un servicio web externo para capturar el valor actual de las acciones de Microsoft.</p><p>Después, siempre que "&MSFTStock;" aparezca en el documento, se reemplazará automáticamente por el precio actual de las acciones. Sin embargo, se puede abusar de esta funcionalidad para crear las condiciones para una denegación de servicio (DoS). Un atacante puede anidar varias entidades para crear una bomba XML de expansión exponencial que consuma toda la memoria disponible en el sistema. </p><p>También puede crear una referencia externa que transmita de vuelta una cantidad infinita de datos o que simplemente bloquee el subproceso. Como resultado, todos los equipos deben deshabilitar por completo la resolución de entidades XML internas o externas si su aplicación no la usa o limitar de forma manual la cantidad de memoria y el tiempo que la aplicación puede consumir para la resolución de entidades si esta funcionalidad es absolutamente necesaria. Si la aplicación no necesita la resolución de entidades, deshabilítela. </p>|

### <a name="example"></a>Ejemplo
Para el código de .NET Framework, puede usar los enfoques siguientes:

```csharp
XmlTextReader reader = new XmlTextReader(stream);
reader.ProhibitDtd = true;

XmlReaderSettings settings = new XmlReaderSettings();
settings.ProhibitDtd = true;
XmlReader reader = XmlReader.Create(stream, settings);

// for .NET 4
XmlReaderSettings settings = new XmlReaderSettings();
settings.DtdProcessing = DtdProcessing.Prohibit;
XmlReader reader = XmlReader.Create(stream, settings);
```
Tenga en cuenta que el valor predeterminado de `ProhibitDtd` en `XmlReaderSettings` es true, pero en `XmlTextReader` es false. Si usa XmlReaderSettings, no es necesario establecer ProhibitDtd en true de forma explícita, pero se recomienda por seguridad. Tenga en cuenta que la clase XmlDocument permite la resolución de entidades de forma predeterminada. 

### <a name="example"></a>Ejemplo
Para deshabilitar la resolución de entidades para XmlDocuments, use la sobrecarga `XmlDocument.Load(XmlReader)` del método Load y establezca las propiedades adecuadas en el argumento XmlReader para deshabilitar la resolución, como se muestra en el código siguiente: 

```csharp
XmlReaderSettings settings = new XmlReaderSettings();
settings.ProhibitDtd = true;
XmlReader reader = XmlReader.Create(stream, settings);
XmlDocument doc = new XmlDocument();
doc.Load(reader);
```

### <a name="example"></a>Ejemplo
Si no es posible deshabilitar la resolución de entidades para su aplicación, establezca la propiedad XmlReaderSettings.MaxCharactersFromEntities en un valor razonable según las necesidades de su aplicación. Esto limitará el impacto de posibles ataques DoS de expansión exponencial. En el código siguiente se muestra un ejemplo de este enfoque: 

```csharp
XmlReaderSettings settings = new XmlReaderSettings();
settings.ProhibitDtd = false;
settings.MaxCharactersFromEntities = 1000;
XmlReader reader = XmlReader.Create(stream, settings);
```

### <a name="example"></a>Ejemplo
Si tiene que resolver entidades en línea pero no necesita resolver externas, establezca la propiedad XmlReaderSettings.XmlResolver en null. Por ejemplo: 

```csharp
XmlReaderSettings settings = new XmlReaderSettings();
settings.ProhibitDtd = false;
settings.MaxCharactersFromEntities = 1000;
settings.XmlResolver = null;
XmlReader reader = XmlReader.Create(stream, settings);
```
Tenga en cuenta que en MSXML6, ProhibitDTD está establecido en true (deshabilitar el procesamiento de DTD) de forma predeterminada. Para el código de Apple OSX e iOS, puede usar uno de estos dos analizadores XML: NSXMLParser y libXML2. 

## <a name="applications-utilizing-httpsys-perform-url-canonicalization-verification"></a><a id="app-verification"></a>Las aplicaciones que usan http.sys realizan la comprobación de canonización de URL

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>Cualquier aplicación que use http.sys debe seguir estas instrucciones:</p><ul><li>Limite la longitud de la dirección URL a no más de 16 384 caracteres (ASCII o Unicode). Se trata de la longitud máxima absoluta para una dirección URL según la configuración predeterminada de Internet Information Services (IIS) 6. Se debe procurar que los sitios web usen direcciones URL más cortas, de ser posible.</li><li>Utilice las clases de E/S de archivos estándar de .NET Framework (por ejemplo, FileStream), ya que aprovecharán las reglas de canonización en .NET FX.</li><li>Cree explícitamente una lista de nombres de archivo conocidos permitidos.</li><li>Rechace de forma explícita los tipos de archivo conocidos a los que no servirá rechazos de UrlScan: archivos exe, bat, cmd, com, htw, ida, idq, htr, idc, shtm[l], stm, ini, pol, dat y de impresora.</li><li>Detecte las siguientes excepciones:<ul><li>System.ArgumentException (para nombres de dispositivo)</li><li>System.NotSupportedException (para flujos de datos)</li><li>System.IO.FileNotFoundException (para nombres de archivo con escape no válidos)</li><li>System.IO.DirectoryNotFoundException (para directorios con escape no válidos)</li></ul></li><li>*No* llame a las API de E/S de archivos Win32. En una dirección URL no válida, devuelva un error 400 correctamente al usuario y registre el error real.</li></ul>|

## <a name="ensure-appropriate-controls-are-in-place-when-accepting-files-from-users"></a><a id="controls-users"></a>Asegúrese de que los controles adecuados estén en vigor al aceptar archivos de usuarios

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload) (Carga de archivos sin restricciones), [File Signature Table](https://www.garykessler.net/library/file_sigs.html) (Tabla de firmas de archivo) |
| **Pasos** | <p>Los archivos cargados presentan un riesgo notable para las aplicaciones.</p><p>El primer paso en muchos ataques consiste en incluir código en el sistema que va a sufrir el ataque. Después, el ataque solo necesita encontrar una manera de que se ejecute el código. El uso de la carga de archivos ayuda al atacante a llevar a cabo el primer paso. Las consecuencias de la carga de archivos sin restricciones varían, desde la toma de control de todo el sistema, la sobrecarga del sistema de archivos o la base de datos, o el reenvío de ataques a sistemas back-end hasta una simple alteración.</p><p>Depende de lo que haga la aplicación con el archivo cargado y especialmente de dónde se almacene. Falta la validación del lado servidor para las cargas de archivos. Se deberían implementar los siguientes controles de seguridad para la funcionalidad de carga de archivos:</p><ul><li>Comprobación de la extensión del archivo (solo se debe aceptar un conjunto válido de tipos de archivo permitidos)</li><li>Límite máximo de tamaño de archivo</li><li>No se debe cargar el archivo a webroot; la ubicación debe ser un directorio en una unidad que no sea del sistema.</li><li>Se debe seguir la convención de nomenclatura, por ejemplo, que el nombre del archivo cargado contenga elementos aleatorios, para evitar que se sobrescriban archivos.</li><li>Se deben examinar los archivos en busca de virus antes de escribirlos en el disco.</li><li>Asegúrese de que el nombre de archivo y cualquier otro metadato (como la ruta de acceso del archivo) se validen en busca de caracteres malintencionados.</li><li>Se debe comprobar la firma de formato del archivo para evitar que un usuario cargue un archivo enmascarado (por ejemplo, un archivo exe cuya extensión se haya cambiado a txt).</li></ul>| 

### <a name="example"></a>Ejemplo
En lo que respecta al último punto sobre la validación de la firma de formato del archivo, consulte más detalles en la clase que sigue: 

```csharp
        private static Dictionary<string, List<byte[]>> fileSignature = new Dictionary<string, List<byte[]>>
                    {
                    { ".DOC", new List<byte[]> { new byte[] { 0xD0, 0xCF, 0x11, 0xE0, 0xA1, 0xB1, 0x1A, 0xE1 } } },
                    { ".DOCX", new List<byte[]> { new byte[] { 0x50, 0x4B, 0x03, 0x04 } } },
                    { ".PDF", new List<byte[]> { new byte[] { 0x25, 0x50, 0x44, 0x46 } } },
                    { ".ZIP", new List<byte[]> 
                                            {
                                              new byte[] { 0x50, 0x4B, 0x03, 0x04 },
                                              new byte[] { 0x50, 0x4B, 0x4C, 0x49, 0x54, 0x55 },
                                              new byte[] { 0x50, 0x4B, 0x53, 0x70, 0x58 },
                                              new byte[] { 0x50, 0x4B, 0x05, 0x06 },
                                              new byte[] { 0x50, 0x4B, 0x07, 0x08 },
                                              new byte[] { 0x57, 0x69, 0x6E, 0x5A, 0x69, 0x70 }
                                                }
                                            },
                    { ".PNG", new List<byte[]> { new byte[] { 0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A } } },
                    { ".JPG", new List<byte[]>
                                    {
                                              new byte[] { 0xFF, 0xD8, 0xFF, 0xE0 },
                                              new byte[] { 0xFF, 0xD8, 0xFF, 0xE1 },
                                              new byte[] { 0xFF, 0xD8, 0xFF, 0xE8 }
                                    }
                                    },
                    { ".JPEG", new List<byte[]>
                                        { 
                                            new byte[] { 0xFF, 0xD8, 0xFF, 0xE0 },
                                            new byte[] { 0xFF, 0xD8, 0xFF, 0xE2 },
                                            new byte[] { 0xFF, 0xD8, 0xFF, 0xE3 }
                                        }
                                        },
                    { ".XLS", new List<byte[]>
                                            {
                                              new byte[] { 0xD0, 0xCF, 0x11, 0xE0, 0xA1, 0xB1, 0x1A, 0xE1 },
                                              new byte[] { 0x09, 0x08, 0x10, 0x00, 0x00, 0x06, 0x05, 0x00 },
                                              new byte[] { 0xFD, 0xFF, 0xFF, 0xFF }
                                            }
                                            },
                    { ".XLSX", new List<byte[]> { new byte[] { 0x50, 0x4B, 0x03, 0x04 } } },
                    { ".GIF", new List<byte[]> { new byte[] { 0x47, 0x49, 0x46, 0x38 } } }
                };

        public static bool IsValidFileExtension(string fileName, byte[] fileData, byte[] allowedChars)
        {
            if (string.IsNullOrEmpty(fileName) || fileData == null || fileData.Length == 0)
            {
                return false;
            }

            bool flag = false;
            string ext = Path.GetExtension(fileName);
            if (string.IsNullOrEmpty(ext))
            {
                return false;
            }

            ext = ext.ToUpperInvariant();

            if (ext.Equals(".TXT") || ext.Equals(".CSV") || ext.Equals(".PRN"))
            {
                foreach (byte b in fileData)
                {
                    if (b > 0x7F)
                    {
                        if (allowedChars != null)
                        {
                            if (!allowedChars.Contains(b))
                            {
                                return false;
                            }
                        }
                        else
                        {
                            return false;
                        }
                    }
                }

                return true;
            }

            if (!fileSignature.ContainsKey(ext))
            {
                return true;
            }

            List<byte[]> sig = fileSignature[ext];
            foreach (byte[] b in sig)
            {
                var curFileSig = new byte[b.Length];
                Array.Copy(fileData, curFileSig, b.Length);
                if (curFileSig.SequenceEqual(b))
                {
                    flag = true;
                    break;
                }
            }

            return flag;
        }
```

## <a name="ensure-that-type-safe-parameters-are-used-in-web-application-for-data-access"></a><a id="typesafe"></a>Asegúrese de que se usen parámetros con seguridad de tipos en la aplicación web para el acceso a datos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>Si usa la colección de parámetros, SQL trata la entrada como valor literal en lugar de como código ejecutable. La colección de parámetros se puede usar para aplicar restricciones de tipo y longitud a los datos de entrada. Los valores fuera del intervalo desencadenan una excepción. Si no se usan parámetros SQL con seguridad de tipos, los atacantes podrían ejecutar ataques por inyección que se insertan en la entrada sin filtrar.</p><p>Use parámetros con seguridad de tipos al construir consultas SQL para evitar posibles ataques por inyección de código SQL que pueden producirse con la entrada sin filtrar. Puede usar parámetros con seguridad de tipos con procedimientos almacenados e instrucciones SQL dinámicas. Los parámetros se tratan como valores literales en la base de datos y no como código ejecutable. También se comprueban el tipo y la longitud de los parámetros.</p>|

### <a name="example"></a>Ejemplo 
En el código siguiente, se muestra cómo usar parámetros con seguridad de tipos con SqlParameterCollection al llamar a un procedimiento almacenado. 

```csharp
using System.Data;
using System.Data.SqlClient;

using (SqlConnection connection = new SqlConnection(connectionString))
{ 
DataSet userDataset = new DataSet(); 
SqlDataAdapter myCommand = new SqlDataAdapter("LoginStoredProcedure", connection); 
myCommand.SelectCommand.CommandType = CommandType.StoredProcedure; 
myCommand.SelectCommand.Parameters.Add("@au_id", SqlDbType.VarChar, 11); 
myCommand.SelectCommand.Parameters["@au_id"].Value = SSN.Text; 
myCommand.Fill(userDataset);
}  
```
En el ejemplo de código anterior, el valor de entrada no puede tener más de 11 caracteres. Si los datos no se ajustan al tipo o la longitud definidos en el parámetro, la clase SqlParameter produce una excepción. 

## <a name="use-separate-model-binding-classes-or-binding-filter-lists-to-prevent-mvc-mass-assignment-vulnerability"></a><a id="binding-mvc"></a>Use clases de enlace de modelos distintas o listas de filtros de enlace para evitar la vulnerabilidad de asignación masiva de MVC

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | MVC5, MVC6 |
| **Atributos**              | N/D  |
| **Referencias**              | [Atributos de metadatos](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.metadatatypeattribute), [Public Key Security Vulnerability And Mitigation](https://github.com/blog/1068-public-key-security-vulnerability-and-mitigation) (Vulnerabilidad de seguridad de clave pública y mitigación), [Guía completa para la asignación masiva en ASP.NET MVC](https://odetocode.com/Blogs/scott/archive/2012/03/11/complete-guide-to-mass-assignment-in-asp-net-mvc.aspx), [Introducción a EF mediante MVC](https://www.asp.net/mvc/tutorials/getting-started-with-ef-using-mvc/implementing-basic-crud-functionality-with-the-entity-framework-in-asp-net-mvc-application#overpost) |
| **Pasos** | <ul><li>**¿Cuándo debería buscar vulnerabilidades por publicación excesiva?** Las vulnerabilidades por publicación excesiva se pueden producir en cualquier lugar donde se enlacen clases de modelo desde la entrada del usuario. Los marcos de trabajo como MVC pueden representar datos de usuario en clases .NET personalizadas, incluidos los objetos CLR estándar (POCO). MVC rellena automáticamente estas clases de modelo con datos de la solicitud y proporciona una representación cómoda para tratar la entrada del usuario. Cuando estas clases incluyen propiedades que el usuario no debería establecer, la aplicación puede ser vulnerable a ataques por publicación excesiva, que permiten que el usuario controle datos que la aplicación no pretendía que controlara. Al igual que el enlace de modelos de MVC, las tecnologías de acceso a bases de datos, como los asignadores de objetos/relacionales (por ejemplo, Entity Framework), a menudo también admiten el uso de objetos POCO para representar datos de la base de datos. Estas clases del modelo de datos proporcionan la misma comodidad al tratar con datos de la base de datos que MVC al tratar la entrada del usuario. Ya que MVC y la base de datos admiten modelos similares, como los objetos POCO, parece sencillo reutilizar las mismas clases para ambos fines. Esta práctica no respeta la separación de aspectos y es un área habitual en la que se exponen propiedades imprevistas al enlace de modelos, lo que hace posibles los ataques por publicación excesiva.</li><li>**¿Por qué no debo usar mis clases de modelo de base de datos sin filtrar como parámetros para acciones de MVC?** Porque el enlace de modelos de MVC enlazará cualquier elemento de esa clase. Incluso si los datos no aparecen en la vista, un usuario malintencionado puede enviar una solicitud HTTP con estos datos incluidos y MVC la enlazará sin reparos, porque la acción indica que la clase de base de datos tiene la forma de datos que debería aceptar para la entrada del usuario.</li><li>**¿Por qué debo preocuparme por la forma usada para el enlace de modelos?** El uso del enlace de modelos de ASP.NET MVC con modelos demasiado amplios expone a la aplicación a ataques por publicación excesiva. La publicación excesiva podría permitir que los atacantes cambiaran los datos de la aplicación más allá de lo previsto por el desarrollador, por ejemplo, invalidar el precio de un artículo o los privilegios de seguridad de una cuenta. Las aplicaciones deben utilizar modelos de enlace específicos de la acción (o listas de filtros de propiedad permitidos específicos) para proporcionar un contrato explícito sobre la entrada no de confianza que se permitirá mediante el enlace de modelos.</li><li>**¿Tener modelos de enlace independientes sirve solo para duplicar el código?** No, es cuestión de separar los aspectos. Si reutiliza modelos de base de datos en los métodos de acción, lo que indica es que el usuario puede establecer cualquier propiedad (o subpropiedad) de esa clase en una solicitud HTTP. Si no desea que MVC haga esto, necesita una lista de filtros o una forma de clase diferente para mostrar a MVC qué datos pueden proceder de la entrada del usuario.</li><li>**Si dispongo de modelos de enlace independientes para la entrada del usuario, ¿tengo que duplicar todos los atributos de anotación de datos?** No necesariamente. Puede usar MetadataTypeAttribute en la clase de modelo de base de datos para vincular a los metadatos en una clase de enlace de modelos. Tenga en cuenta que el tipo al que hace referencia MetadataTypeAttribute debe ser un subconjunto del tipo de referencia (puede tener menos propiedades, pero no más).</li><li>**Mover los datos entre los modelos de entrada de usuario y los modelos de base de datos es una tarea tediosa. ¿Puedo simplemente copiar todas las propiedades mediante reflexión?** Sí. Las únicas propiedades que aparecen en los modelos de enlace son las que haya determinado que son seguras para la entrada de usuario. No existe ninguna razón de seguridad que impida usar la reflexión para copiar todas las propiedades en común entre estos dos modelos.</li><li>**¿Cómo funciona [Bind(Exclude ="â&euro;¦")]? ¿Lo puedo usar en lugar de tener modelos de enlace independientes?** No se recomienda este enfoque. Si usa [Bind(Exclude ="â&euro;¦")], cualquier propiedad nueva se puede enlazar de forma predeterminada. Cuando se agrega una nueva propiedad, hay que recordar un paso adicional para mantener la seguridad, es decir, el diseño no es seguro de forma predeterminada. Confiar en que el desarrollador compruebe esta lista cada vez que se agregue una propiedad es arriesgado.</li><li>**¿Resulta útil [Bind(Include ="â&euro;¦")] para operaciones Edit?**  No. [Bind(Include ="â&euro;¦")] solo es aplicable a operaciones de tipo INSERT (agregar datos nuevos). Para las operaciones de estilo UPDATE (modificar datos existentes), se usa otro enfoque, como tener modelos de enlace independientes o pasar una lista explícita de propiedades permitidas a UpdateModel o TryUpdateModel. Al agregar un atributo [Bind(Include ="â&euro;¦")] a una operación Edit, MVC crea una instancia de objeto y establece solo las propiedades mencionadas, dejando el resto con sus valores predeterminados. Cuando los datos sean persistentes, reemplazarán por completo la entidad existente y restablecerán los valores predeterminados de las propiedades omitidas. Por ejemplo, si se ha omitido IsAdmin en un atributo [Bind(Include ="â&euro;¦")] en una operación Edit, cualquier usuario cuyo nombre se haya editado mediante esta acción se restablecería a IsAdmin = false (cualquier usuario editado perdería el estado de administrador). Si desea evitar que se actualicen determinadas propiedades, use uno de los otros enfoques anteriores. Tenga en cuenta que algunas versiones de las herramientas de MVC generan clases de controlador con [Bind(Include ="â&euro;¦")] en acciones Edit e implican que quitar una propiedad de esa lista impediría los ataques de publicación excesiva. Sin embargo, como se ha descrito antes, ese enfoque no funciona según lo previsto y, en su lugar, restablecerá los valores predeterminados de los datos en las propiedades omitidas.</li><li>**Para las operaciones Create, ¿existen consideraciones a la hora de usar [Bind(Include ="â&euro;¦")] en lugar de modelos de enlace independientes?**  Sí. En primer lugar, este enfoque no funciona para los escenarios de edición, por lo que es necesario mantener dos enfoques independientes para mitigar todas las vulnerabilidades por publicación excesiva. En segundo lugar, los modelos de enlace independientes aplican la separación de intereses entre la forma usada para la entrada del usuario y la forma usada para la persistencia, algo que [Bind(Include ="â&euro;¦")] no hace. En tercer lugar, tenga en cuenta que [Bind(Include ="â&euro;¦")] solo puede controlar propiedades de nivel superior; no se permiten solo partes de subpropiedades (como "Details.Name") en el atributo. Por último, y quizás lo más importante, usar [Bind(Include ="â&euro;¦")] agrega un paso adicional que se debe recordar siempre que la clase se use para el enlace de modelos. Si un nuevo método de acción enlaza directamente con la clase de datos y olvida incluir un atributo [Bind(Include ="â&euro;¦")], puede ser vulnerable a ataques de exceso de publicación, por lo que el enfoque de [Bind(Include ="â&euro;¦")] es menos seguro de forma predeterminada. Si usa [Bind(Include ="â&euro;¦")], recuerde especificarlo siempre cada vez que las clases de datos aparezcan como parámetros de método de acción.</li><li>**Para operaciones Create, ¿se puede colocar el atributo [Bind(Include ="â&euro;¦")] en la propia clase de modelo? ¿No evita este enfoque la necesidad de acordarse de colocar el atributo en cada método de acción?** Este enfoque funciona en algunos casos. Usar [Bind(Include ="â&euro;¦")] en el propio tipo de modelo (en lugar de en parámetros de acción mediante esta clase) evita la necesidad de recordar incluir el atributo [Bind(Include ="â&euro;¦")] en cada método de acción. Efectivamente, al usarse el atributo directamente en la clase, se crea un área expuesta de esta clase distinta para el enlace de modelos. Sin embargo, este enfoque solo permite una forma de enlace de modelos por cada clase de modelo. Si un método de acción necesita permitir el enlace de modelos de un campo (por ejemplo, una acción solo de administrador que actualiza roles de usuario) y otras acciones deben evitar el enlace de modelos de este campo, este enfoque no funcionará. Cada clase solo puede tener una forma de enlace de modelo. Si diferentes acciones necesitan formas de enlace de modelo diferentes, es necesario representar estas formas independientes con cualquier clase de enlace de modelo independiente o atributos [Bind(Include ="â&euro;¦")] independientes en los métodos de acción.</li><li>**¿Qué son los modelos de enlace? ¿Son lo mismo que los modelos de vista?** Se trata de dos conceptos relacionados. El término modelo de enlace hace referencia a una clase de modelo usada en una lista de parámetros de acción (la forma que se pasa del enlace de modelos de MVC al método de acción). El término modelo de vista hace referencia a una clase de modelo que se pasa de un método de acción a una vista. El uso de un modelo específico de la vista es un enfoque habitual para pasar datos de un método de acción a una vista. Con frecuencia, esta forma también es adecuada para el enlace de modelos, y el término modelo de vista puede usarse para hacer referencia al mismo modelo que se usa en ambos lugares. Para ser exactos, este procedimiento se refiere en concreto a los modelos de enlace y se centra en la forma pasada a la acción, que es lo que realmente importa para la asignación masiva.</li></ul>| 

## <a name="encode-untrusted-web-output-prior-to-rendering"></a><a id="rendering"></a>Codifique la salida web que no sea de confianza antes de representarla

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, formularios Web Forms, MVC5, MVC6 |
| **Atributos**              | N/D  |
| **Referencias**              | [Impedir el scripting entre sitios en ASP.NET](https://msdn.microsoft.com/library/ms998274.aspx), [Scripting entre sitios](https://cwe.mitre.org/data/definitions/79.html), [Hoja de referencia rápida de prevención de (scripting entre sitios) XSS](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) |
| **Pasos** | El scripting entre sitios (normalmente abreviado como XSS) es un vector de ataque para servicios en línea o cualquier aplicación o componente que consuma una entrada desde la Web. Las vulnerabilidades de XSS pueden permitir que un atacante ejecute un script en la máquina de otro usuario por medio de una aplicación web vulnerable. Los scripts malintencionados sirven para robar cookies y alterar de otras formas la máquina de la víctima mediante JavaScript. Se impide el XSS mediante la validación de la entrada del usuario para asegurarse de que esté bien formada y codificada antes de representarla en una página web. La validación de la entrada y la codificación de la salida se pueden realizar con Web Protection Library. Para el código administrado (C\#, VB.NET, etc.), use uno o varios métodos de codificación adecuados de Web Protection (Anti-XSS) Library, según el contexto donde se manifieste la entrada del usuario:| 

### <a name="example"></a>Ejemplo

```csharp
* Encoder.HtmlEncode 
* Encoder.HtmlAttributeEncode 
* Encoder.JavaScriptEncode 
* Encoder.UrlEncode
* Encoder.VisualBasicScriptEncode 
* Encoder.XmlEncode 
* Encoder.XmlAttributeEncode 
* Encoder.CssEncode 
* Encoder.LdapEncode 
```

## <a name="perform-input-validation-and-filtering-on-all-string-type-model-properties"></a><a id="typemodel"></a>Lleve a cabo la validación de entrada y el filtrado para todas las propiedades de modelo de tipo cadena

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, MVC5, MVC6 |
| **Atributos**              | N/D  |
| **Referencias**              | [Adding Validation](https://www.asp.net/mvc/overview/getting-started/introduction/adding-validation) (Incorporación de validación), [Validating Model Data in an MVC Application](https://msdn.microsoft.com/library/dd410404(v=vs.90).aspx) (Validación de datos de modelo en una aplicación de MVC), [Principios orientativos para las aplicaciones de ASP.NET MVC](https://msdn.microsoft.com/magazine/dd942822.aspx) |
| **Pasos** | <p>Todos los parámetros de entrada se deben validar antes de usarse en la aplicación para asegurarse de que la aplicación esté protegida ante las entradas de usuarios malintencionados. Compruebe los valores de entrada mediante validaciones con expresiones regulares en el lado servidor con una estrategia de validación con lista de elementos permitidos. La entrada de usuario o los parámetros sin sanear pasados a los métodos pueden ocasionar vulnerabilidades por inyección de código.</p><p>Para las aplicaciones web, los puntos de entrada también pueden ser campos de formulario, QueryStrings, cookies, encabezados HTTP y parámetros de servicio web.</p><p>Se deben realizar las siguientes comprobaciones de validación de entrada con el enlace de modelos:</p><ul><li>Las propiedades del modelo se deben anotar con la anotación RegularExpression para que acepten los caracteres permitidos y la longitud máxima permitida.</li><li>Los métodos de controlador deberían llevar a cabo la comprobación de validez ModelState.</li></ul>|

## <a name="sanitization-should-be-applied-on-form-fields-that-accept-all-characters-eg-rich-text-editor"></a><a id="richtext"></a>Se debería aplicar la comprobación de estado a los campos de formulario que acepten todos los caracteres, por ejemplo, editor de texto enriquecido

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Codificación de la entrada no segura](https://msdn.microsoft.com/library/ff647397.aspx#paght000003_step3), [HTMLSanitizer](https://github.com/mganss/HtmlSanitizer) |
| **Pasos** | <p>Identifique todas las etiquetas de marcado estático que desea usar. Es habitual restringir el formato a elementos HTML seguros, como `<b>` (negrita) e `<i>` (cursiva).</p><p>Antes de escribir los datos, codifíquelos en HTML. Esto anula el peligro de cualquier script malintencionado, ya que hace que se traten como texto y no como código ejecutable.</p><ol><li>Deshabilite la validación de solicitudes ASP.NET agregando el atributo ValidateRequest="false" a la directiva \@ Page.</li><li>Codifique la entrada de cadena con el método HtmlEncode.</li><li>Use un objeto StringBuilder y llame a su método Replace para quitar de forma selectiva la codificación de los elementos HTML que desea permitir.</li></ol><p>La página en las referencias deshabilita la validación de solicitudes ASP.NET al establecer `ValidateRequest="false"`. Se codifica en HTML la entrada y se permiten de forma selectiva los elementos `<b>` e `<i>`. O bien, también se puede usar una biblioteca .NET para la comprobación de estado de HTML.</p><p>HtmlSanitizer es una biblioteca .NET que limpia las construcciones que pueden dar lugar a ataques XSS en fragmentos y documentos HTML. Utiliza AngleSharp para analizar, manipular y representar HTML y CSS. Se puede instalar HtmlSanitizer como paquete NuGet, y se puede pasar la entrada del usuario por los métodos de comprobación de estado de HTML o CSS pertinentes, según corresponda, en el lado servidor. Tenga en cuenta que la comprobación de estado como control de seguridad se debe considerar la última opción.</p><p>La validación de entrada y la codificación de salida son los controles de seguridad que se consideran mejores.</p> |

## <a name="do-not-assign-dom-elements-to-sinks-that-do-not-have-inbuilt-encoding"></a><a id="inbuilt-encode"></a>No asigne elementos DOM a receptores que carezcan de codificación integrada

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | Muchas funciones JavaScript no codifican de forma predeterminada. Cuando se asigna una entrada que no sea de confianza a elementos DOM mediante estas funciones, es posible que se produzcan ejecuciones de scripts entre sitios (XSS).| 

### <a name="example"></a>Ejemplo
Los siguientes son ejemplos inseguros: 

```
document.getElementByID("div1").innerHtml = value;
$("#userName").html(res.Name);
return $('<div/>').html(value)
$('body').append(resHTML);   
```
No utilice `innerHtml`; en su lugar, use `innerText`. Igualmente, en lugar de `$("#elm").html()`, use `$("#elm").text()`. 

## <a name="validate-all-redirects-within-the-application-are-closed-or-done-safely"></a><a id="redirect-safe"></a>Valide que todos los redireccionamientos dentro de la aplicación se cierren o se realicen de forma segura

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [The OAuth 2.0 Authorization Framework - Open Redirectors](https://tools.ietf.org/html/rfc6749#section-10.15) (Marco de autorización OAuth 2.0: redirectores abiertos) |
| **Pasos** | <p>El diseño de aplicaciones que requieren redireccionamiento a una ubicación proporcionada por el usuario debe restringir los destinos de redireccionamiento posibles a una lista predefinida de sitios o dominios "seguros". Todos los redireccionamientos en la aplicación deben ser cerrados y seguros.</p><p>Para ello, siga estos pasos:</p><ul><li>Identifique todos los redireccionamientos.</li><li>Implemente una mitigación adecuada para cada tipo de redireccionamiento. Entre las mitigaciones adecuadas, se incluye la lista de redireccionamientos permitidos o la confirmación del usuario. Si un sitio web o servicio con una vulnerabilidad de redireccionamiento abierto usa proveedores de identidades de Facebook, OAuth u OpenID, un atacante puede robar el token de inicio de sesión de un usuario y suplantarle. Se trata de un riesgo inherente al uso de OAuth, que se describe en la RFC 6749 sobre el marco de autorización OAuth 2.0, en la sección 10.15 sobre redireccionamientos abiertos. De forma similar, las credenciales de los usuarios pueden correr peligro por ataques de suplantación de identidad (spear-phishing) que usan redireccionamientos abiertos.</li></ul>|

## <a name="implement-input-validation-on-all-string-type-parameters-accepted-by-controller-methods"></a><a id="string-method"></a>Implemente la validación de entrada en todos los parámetros de tipo cadena aceptados por métodos de controlador

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, MVC5, MVC6 |
| **Atributos**              | N/D  |
| **Referencias**              | [Validating Model Data in an MVC Application](https://msdn.microsoft.com/library/dd410404(v=vs.90).aspx) (Validación de datos de modelo en una aplicación de MVC), [Principios orientativos para las aplicaciones de ASP.NET MVC](https://msdn.microsoft.com/magazine/dd942822.aspx) |
| **Pasos** | Para los métodos que solo acepten el tipo primitivo de datos como argumento, pero no modelos, se debe realizar la validación de entrada mediante expresiones regulares. Aquí se debería usar Regex.IsMatch con un patrón de regex válido. Si la entrada no coincide con la expresión regular especificada, el control no debería continuar y se debería mostrar una advertencia adecuada con respecto al error de validación.| 

## <a name="set-upper-limit-timeout-for-regular-expression-processing-to-prevent-dos-due-to-bad-regular-expressions"></a><a id="dos-expression"></a>Establezca el tiempo de espera del límite superior para el procesamiento de expresiones regulares con el fin de impedir ataques DoS causados por expresiones regulares incorrectas

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, formularios Web Forms, MVC5, MVC6  |
| **Atributos**              | N/D  |
| **Referencias**              | [Propiedad DefaultRegexMatchTimeout](https://msdn.microsoft.com/library/system.web.configuration.httpruntimesection.defaultregexmatchtimeout.aspx) |
| **Pasos** | Para evitar los ataques por denegación de servicio contra expresiones regulares creadas incorrectamente, que causan grandes retrocesos, establezca el tiempo de espera predeterminado global. Si el tiempo de procesamiento es mayor que el límite superior definido, se produce una excepción de tiempo de espera. Si no se configura nada, el tiempo de espera sería infinito.| 

### <a name="example"></a>Ejemplo
Por ejemplo, la siguiente configuración produce una excepción RegexMatchTimeoutException si el procesamiento excede los 5 segundos: 

```csharp
<httpRuntime targetFramework="4.5" defaultRegexMatchTimeout="00:00:05" />
```

## <a name="avoid-using-htmlraw-in-razor-views"></a><a id="html-razor"></a>No use Html.Raw en las vistas Razor

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Aplicación web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | MVC5, MVC6 |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| Paso | Las páginas Web ASP.NET (Razor) realizan la codificación de HTML automáticamente. Todas las cadenas impresas por fragmentos de código insertados (bloques @) se codifican en HTML de forma automática. Sin embargo, cuando se invoca el método `HtmlHelper.Raw`, devuelve marcado que no está codificado en HTML. Si se utiliza el método auxiliar `Html.Raw()`, omite la protección de codificación automática que proporciona Razor.|

### <a name="example"></a>Ejemplo
El siguiente ejemplo es inseguro: 

```csharp
<div class="form-group">
            @Html.Raw(Model.AccountConfirmText)
        </div>
        <div class="form-group">
            @Html.Raw(Model.PaymentConfirmText)
        </div>
</div>
```
No utilice `Html.Raw()` a menos que necesite mostrar marcado. Este método no realiza la codificación de salida implícitamente. Use otros asistentes de ASP.NET, como `@Html.DisplayFor()`. 

## <a name="do-not-use-dynamic-queries-in-stored-procedures"></a><a id="stored-proc"></a>No use consultas dinámicas en procedimientos almacenados

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Base de datos | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>Un ataque por inyección de código SQL aprovecha vulnerabilidades en la validación de entrada para ejecutar comandos arbitrarios en la base de datos. Puede suceder cuando la aplicación usa una entrada para construir instrucciones SQL dinámicas para acceder a la base de datos. También puede producirse si el código usa procedimientos almacenados a los que se pasan cadenas que contienen entrada de usuario sin procesar. Mediante un ataque por inyección de código SQL, el atacante puede ejecutar comandos arbitrarios en la base de datos. Todas las instrucciones SQL (incluidas las que se encuentran en procedimientos almacenados) deben estar parametrizadas. Las instrucciones SQL parametrizadas aceptarán caracteres que tengan un significado especial en SQL (por ejemplo, las comillas simples) sin problemas porque están fuertemente tipadas. |

### <a name="example"></a>Ejemplo
El ejemplo siguiente es de un procedimiento almacenado dinámico inseguro: 

```csharp
CREATE PROCEDURE [dbo].[uspGetProductsByCriteria]
(
  @productName nvarchar(200) = NULL,
  @startPrice float = NULL,
  @endPrice float = NULL
)
AS
 BEGIN
  DECLARE @sql nvarchar(max)
  SELECT @sql = ' SELECT ProductID, ProductName, Description, UnitPrice, ImagePath' +
       ' FROM dbo.Products WHERE 1 = 1 '
       PRINT @sql
  IF @productName IS NOT NULL
     SELECT @sql = @sql + ' AND ProductName LIKE ''%' + @productName + '%'''
  IF @startPrice IS NOT NULL
     SELECT @sql = @sql + ' AND UnitPrice > ''' + CONVERT(VARCHAR(10),@startPrice) + ''''
  IF @endPrice IS NOT NULL
     SELECT @sql = @sql + ' AND UnitPrice < ''' + CONVERT(VARCHAR(10),@endPrice) + ''''

  PRINT @sql
  EXEC(@sql)
 END
```

### <a name="example"></a>Ejemplo
Este es el mismo procedimiento almacenado pero implementado de forma segura: 
```csharp
CREATE PROCEDURE [dbo].[uspGetProductsByCriteriaSecure]
(
             @productName nvarchar(200) = NULL,
             @startPrice float = NULL,
             @endPrice float = NULL
)
AS
       BEGIN
             SELECT ProductID, ProductName, Description, UnitPrice, ImagePath
             FROM dbo.Products where
             (@productName IS NULL or ProductName like '%'+ @productName +'%')
             AND
             (@startPrice IS NULL or UnitPrice > @startPrice)
             AND
             (@endPrice IS NULL or UnitPrice < @endPrice)         
       END
```

## <a name="ensure-that-model-validation-is-done-on-web-api-methods"></a><a id="validation-api"></a>Asegúrese de que se lleve a cabo la validación del modelo en métodos de API web

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | MVC5, MVC6 |
| **Atributos**              | N/D  |
| **Referencias**              | [Validación de modelos en ASP.NET Web API](https://www.asp.net/web-api/overview/formats-and-model-binding/model-validation-in-aspnet-web-api) |
| **Pasos** | Cuando un cliente envía datos a una API web, es obligatorio que valide los datos antes de realizar cualquier procesamiento. Para las API web de ASP.NET, cuando aceptan modelos como entrada, utilice anotaciones de datos en los modelos para establecer reglas de validación en las propiedades del modelo.|

### <a name="example"></a>Ejemplo
El código siguiente lo demuestra: 

```csharp
using System.ComponentModel.DataAnnotations;

namespace MyApi.Models
{
    public class Product
    {
        public int Id { get; set; }
        [Required]
        [RegularExpression(@"^[a-zA-Z0-9]*$", ErrorMessage="Only alphanumeric characters are allowed.")]
        public string Name { get; set; }
        public decimal Price { get; set; }
        [Range(0, 999)]
        public double Weight { get; set; }
    }
}
```

### <a name="example"></a>Ejemplo
En el método de acción de los controladores de API, la validez del modelo se debe comprobar explícitamente como se muestra a continuación: 

```csharp
namespace MyApi.Controllers
{
    public class ProductsController : ApiController
    {
        public HttpResponseMessage Post(Product product)
        {
            if (ModelState.IsValid)
            {
                // Do something with the product (not shown).

                return new HttpResponseMessage(HttpStatusCode.OK);
            }
            else
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, ModelState);
            }
        }
    }
}
```

## <a name="implement-input-validation-on-all-string-type-parameters-accepted-by-web-api-methods"></a><a id="string-api"></a>Implemente la validación de entrada en todos los parámetros de tipo cadena aceptados por métodos de API web

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, MVC 5, MVC 6 |
| **Atributos**              | N/D  |
| **Referencias**              | [Validating Model Data in an MVC Application](https://msdn.microsoft.com/library/dd410404(v=vs.90).aspx) (Validación de datos de modelo en una aplicación de MVC), [Principios orientativos para las aplicaciones de ASP.NET MVC](https://msdn.microsoft.com/magazine/dd942822.aspx) |
| **Pasos** | Para los métodos que solo acepten el tipo primitivo de datos como argumento, pero no modelos, se debe realizar la validación de entrada mediante expresiones regulares. Aquí se debería usar Regex.IsMatch con un patrón de regex válido. Si la entrada no coincide con la expresión regular especificada, el control no debería continuar y se debería mostrar una advertencia adecuada con respecto al error de validación.|

## <a name="ensure-that-type-safe-parameters-are-used-in-web-api-for-data-access"></a><a id="typesafe-api"></a>Asegúrese de que se usen parámetros con seguridad de tipos en la API web para el acceso a datos

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | API Web | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | N/D  |
| **Pasos** | <p>Si usa la colección de parámetros, SQL trata la entrada como valor literal en lugar de como código ejecutable. La colección de parámetros se puede usar para aplicar restricciones de tipo y longitud a los datos de entrada. Los valores fuera del intervalo desencadenan una excepción. Si no se usan parámetros SQL con seguridad de tipos, los atacantes podrían ejecutar ataques por inyección que se insertan en la entrada sin filtrar.</p><p>Use parámetros con seguridad de tipos al construir consultas SQL para evitar posibles ataques por inyección de código SQL que pueden producirse con la entrada sin filtrar. Puede usar parámetros con seguridad de tipos con procedimientos almacenados e instrucciones SQL dinámicas. Los parámetros se tratan como valores literales en la base de datos y no como código ejecutable. También se comprueban el tipo y la longitud de los parámetros.</p>|

### <a name="example"></a>Ejemplo
En el código siguiente, se muestra cómo usar parámetros con seguridad de tipos con SqlParameterCollection al llamar a un procedimiento almacenado. 

```csharp
using System.Data;
using System.Data.SqlClient;

using (SqlConnection connection = new SqlConnection(connectionString))
{ 
DataSet userDataset = new DataSet(); 
SqlDataAdapter myCommand = new SqlDataAdapter("LoginStoredProcedure", connection); 
myCommand.SelectCommand.CommandType = CommandType.StoredProcedure; 
myCommand.SelectCommand.Parameters.Add("@au_id", SqlDbType.VarChar, 11); 
myCommand.SelectCommand.Parameters["@au_id"].Value = SSN.Text; 
myCommand.Fill(userDataset);
}  
```
En el ejemplo de código anterior, el valor de entrada no puede tener más de 11 caracteres. Si los datos no se ajustan al tipo o la longitud definidos en el parámetro, la clase SqlParameter produce una excepción. 

## <a name="use-parameterized-sql-queries-for-cosmos-db"></a><a id="sql-docdb"></a>Uso de consultas SQL parametrizadas con Cosmos DB

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | Azure DocumentDB | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico |
| **Atributos**              | N/D  |
| **Referencias**              | [Announcing SQL Parameterization in Azure Cosmos DB](https://azure.microsoft.com/blog/announcing-sql-parameterization-in-documentdb/) (Anuncio de la parametrización de SQL en Azure Cosmos DB) |
| **Pasos** | Aunque Azure Cosmos DB únicamente admite consultas de solo lectura, todavía es posible inyectar código SQL si las consultas se crean mediante la concatenación con la entrada del usuario. Es posible que un usuario obtenga acceso a datos a los que no debería acceder dentro de la misma colección mediante el diseño de consultas SQL malintencionadas. Use consultas SQL parametrizadas si las consultas se construyen tomando como base la entrada del usuario. |

## <a name="wcf-input-validation-through-schema-binding"></a><a id="schema-binding"></a>Validación de entrada de WCF mediante enlaces de esquema

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, .NET Framework 3 |
| **Atributos**              | N/D  |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff647820.aspx) |
| **Pasos** | <p>La falta de validación conlleva diferentes tipos de ataque por inyección.</p><p>La validación de mensajes constituye una línea de defensa para la protección de la aplicación WCF. Con este enfoque, se validan los mensajes mediante esquemas para proteger las operaciones del servicio WCF frente al ataque de un cliente malintencionado. Valide todos los mensajes recibidos por el cliente para protegerlo frente al ataque de un servicio malintencionado. La validación de mensajes permite validar mensajes cuando las operaciones consumen contratos de mensaje o de datos, algo que no se puede llevar a cabo mediante la validación de parámetros. La validación de mensajes le permite crear lógica de validación dentro de esquemas, lo cual proporciona más flexibilidad y reduce el tiempo de desarrollo. Los esquemas se pueden reutilizar en distintas aplicaciones dentro de la organización y así crear estándares para la representación de datos. Además, la validación de mensajes permite proteger las operaciones cuando consumen tipos de datos más complejos con contratos que representan lógica de negocios.</p><p>Para realizar la validación de mensajes, primero cree un esquema que represente las operaciones del servicio y los tipos de datos consumidos por esas operaciones. Después, cree una clase .NET que implemente un inspector de mensajes de cliente personalizado y un inspector de mensajes de distribuidor personalizado para validar los mensajes enviados al servicio o recibidos desde él. A continuación, implemente un comportamiento de punto de conexión personalizado para habilitar la validación de mensajes en el cliente y el servicio. Por último, implemente un elemento de configuración personalizado en la clase que permita exponer el comportamiento de punto de conexión personalizado extendido en el archivo de configuración del servicio o el cliente.</p>|

## <a name="wcf--input-validation-through-parameter-inspectors"></a><a id="parameters"></a>Validación de entrada de WCF mediante inspectores de parámetros

| Título                   | Detalles      |
| ----------------------- | ------------ |
| **Componente**               | WCF | 
| **Fase de SDL**               | Build |  
| **Tecnologías aplicables** | Genérico, .NET Framework 3 |
| **Atributos**              | N/D  |
| **Referencias**              | [MSDN](https://msdn.microsoft.com/library/ff647875.aspx) |
| **Pasos** | <p>La validación de datos y de entrada constituye una importante línea de defensa para la protección de la aplicación WCF. Debería validar todos los parámetros expuestos en las operaciones del servicio WCF para proteger el servicio frente al ataque de un cliente malintencionado. A la inversa, debería validar todos los valores devueltos recibidos por el cliente para protegerlo frente al ataque de un servicio malintencionado.</p><p>WCF proporciona diferentes puntos de extensibilidad que permiten personalizar el comportamiento en tiempo de ejecución de WCF mediante la creación de extensiones personalizadas. Los inspectores de mensajes y de parámetros son dos mecanismos de extensibilidad que se usan para ejercer un mayor control sobre los datos que pasan entre un cliente y un servicio. Debería usar inspectores de parámetros para la validación de entrada e inspectores de mensajes solo cuando necesite inspeccionar el mensaje completo que fluye dentro y fuera de un servicio.</p><p>Para llevar a cabo la validación de entrada, generará una clase .NET e implementará un inspector de parámetros personalizado para validar los parámetros en las operaciones de su servicio. Después, implementará un comportamiento de punto de conexión personalizado para habilitar la validación tanto en el cliente como en el servicio. Por último, implementará un elemento de configuración personalizado en la clase que permita exponer el comportamiento de punto de conexión personalizado extendido en el archivo de configuración del servicio o del cliente.</p>|
