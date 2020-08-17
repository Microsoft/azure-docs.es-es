---
title: Inicio de sesión en aplicaciones de MSAL | Azure
titleSuffix: Microsoft identity platform
description: Aprenda a registrar aplicaciones de la biblioteca de autenticación de Microsoft (MSAL).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/11/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: 2d41b48613ef7ba883a6a51b0fa67407fb730719
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87846231"
---
# <a name="logging-in-msal-applications"></a>Inicio de sesión en aplicaciones de MSAL

Las aplicaciones de la biblioteca de autenticación de Microsoft (MSAL) generan mensajes de registro que pueden ayudar a diagnosticar problemas. Una aplicación puede configurar el registro con unas cuentas líneas de código y tener un control personalizado sobre el nivel de detalle y si se registran o no datos de la organización y personales. Se recomienda establecer una devolución de llamada del registro de MSAL y proporcionar una forma de que los usuarios envíen registros cuando tengan problemas de autenticación.

## <a name="logging-levels"></a>Niveles de registro

MSAL ofrece varios niveles de detalle de registro:

- Error: indica que algo no ha funcionado bien y se ha generado un error. Se usa para la depuración y la identificación de problemas.
- Advertencia: No se ha producido necesariamente un error, pero es necesario realizar un diagnóstico e identificar los problemas.
- Info: MSAL registrará los eventos de carácter informativo, no relacionados necesariamente con la depuración.
- Verbose: Predeterminada. MSAL registra todos los detalles del comportamiento de la biblioteca.

## <a name="personal-and-organizational-data"></a>Datos personales y organizativos

De forma predeterminada, el registrador de MSAL no captura datos personales u organizativos sumamente confidenciales. La biblioteca ofrece la opción de habilitar el registro de datos personales y organizativos si decide hacerlo.

Para obtener más información sobre el registro de MSAL en un lenguaje determinado, elija la pestaña que coincida con su lenguaje:

## <a name="net"></a>[.NET](#tab/dotnet)

## <a name="logging-in-msalnet"></a>Registro en MSAL.NET

 > [!NOTE]
 > Consulte en la [wiki de MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki) ejemplos del registro de MSAL.NET, entre otros temas.

En MSAL 3.x, el registro se establece por aplicación en el momento de crear la aplicación mediante el modificador de generador `.WithLogging`. Este método toma parámetros opcionales:

- `Level` le permite decidir el nivel de registro deseado. Si se establece en Errors solo se reciben errores.
- `PiiLoggingEnabled` permite registrar datos personales y organizativos si se establece en true. De forma predeterminada, se establece en false, por lo que la aplicación no registra datos personales.
- `LogCallback` establece en un delegado que realiza el registro. Si `PiiLoggingEnabled` es true, este método recibirá los mensajes dos veces: una con el parámetro `containsPii` igual a false y el mensaje sin datos personales y una segunda con el parámetro `containsPii` igual a true y el mensaje podría contener datos personales. En algunos casos (cuando el mensaje no contiene datos personales), el mensaje será el mismo.
- `DefaultLoggingEnabled` habilita el registro predeterminado para la plataforma. De forma predeterminada, es "false". Si se establece en true se usa el seguimiento de eventos en aplicaciones de escritorio o UWP, NSLog en iOS y logcat en Android.

```csharp
class Program
 {
  private static void Log(LogLevel level, string message, bool containsPii)
  {
     if (containsPii)
     {
        Console.ForegroundColor = ConsoleColor.Red;
     }
     Console.WriteLine($"{level} {message}");
     Console.ResetColor();
  }

  static void Main(string[] args)
  {
    var scopes = new string[] { "User.Read" };

    var application = PublicClientApplicationBuilder.Create("<clientID>")
                      .WithLogging(Log, LogLevel.Info, true)
                      .Build();

    AuthenticationResult result = application.AcquireTokenInteractive(scopes)
                                             .ExecuteAsync().Result;
  }
 }
 ```

## <a name="android"></a>[Android](#tab/android)

## <a name="logging-in-msal-for-android-using-java"></a>Registro en MSAL para Android con Java

Active el registro al crear la aplicación mediante la creación de una devolución de llamada de inicio de sesión. La devolución de llamada toma estos parámetros:

- `tag` es una cadena que la biblioteca pasa a la devolución de llamada. Se asocia a la entrada de registro y se puede usar para ordenar los mensajes de registro.
- `logLevel` le permite decidir el nivel de registro deseado. Los niveles de registro admitidos son: `Error`, `Warning`, `Info` y `Verbose`.
- `message` es el contenido de la entrada de registro.
- `containsPII` especifica si se registran los mensajes que contienen datos personales o datos de la organización. De forma predeterminada, se establece en false para que la aplicación no registre datos personales. Si `containsPII` es `true`, este método recibirá los mensajes dos veces: una con el parámetro `containsPII` igual a `false` y el `message` sin datos personales y una segunda con el parámetro `containsPii` igual a `true` y el mensaje podría contener datos personales. En algunos casos (cuando el mensaje no contiene datos personales), el mensaje será el mismo.

```java
private StringBuilder mLogs;

mLogs = new StringBuilder();
Logger.getInstance().setExternalLogger(new ILoggerCallback()
{
   @Override
   public void log(String tag, Logger.LogLevel logLevel, String message, boolean containsPII)
   {
      mLogs.append(message).append('\n');
   }
});
```

De forma predeterminada, el registrador de MSAL no capturará ninguna información de identificación personal ni información de identificación de la organización.
Para habilitar el registro de información de identificación personal o de información de identificación de la organización:

```java
Logger.getInstance().setEnablePII(true);
```

Para deshabilitar el registro de datos personales y de datos de la organización:

```java
Logger.getInstance().setEnablePII(false);
```

De forma predeterminada, el registro se deshabilita en logcat. Para habilitar las siguientes opciones:

```java
Logger.getInstance().setEnableLogcatLog(true);
```

## <a name="javascript"></a>[JavaScript](#tab/javascript)

 Puede habilitar el registro en MSAL.js (JavaScript) si pasa un objeto de registrador durante la configuración para crear una instancia de `UserAgentApplication`. El objeto de registrador tiene las siguientes propiedades:

- `localCallback`: una instancia de devolución de llamada que el desarrollador puede proporcionar para consumir y publicar registros de una manera personalizada. Implemente el método localCallback según cómo quiera redirigir los registros.
- `level` (opcional): el nivel de registro configurable. Los niveles de registro admitidos son: `Error`, `Warning`, `Info` y `Verbose`. El valor predeterminado es `Info`.
- `piiLoggingEnabled` (opcional): si se establece en true, registra los datos personales y de la organización. De forma predeterminada, se establece en false para que la aplicación no registre los datos personales. Los registros de datos personales nunca se escriben en salidas predeterminadas, como consola, Logcat o NSLog.
- `correlationId` (opcional): un identificador único, que se usa para asignar la solicitud con la respuesta para fines de depuración. El valor predeterminado es RFC4122 version 4 guid (128 bits).

```javascript
function loggerCallback(logLevel, message, containsPii) {
   console.log(message);
}

var msalConfig = {
    auth: {
        clientId: "<Enter your client id>",
    },
     system: {
             logger: new Msal.Logger(
                                loggerCallback ,{
                                     level: Msal.LogLevel.Verbose,
                                     piiLoggingEnabled: false,
                                     correlationId: '1234'
                                }
                        )
     }
}

var UserAgentApplication = new Msal.UserAgentApplication(msalConfig);
```

## <a name="objective-c"></a>[Objective-C](#tab/objc)

## <a name="msal-for-ios-and-macos-logging-objc"></a>Registro de MSAL para iOS y macOS: ObjC

Establezca una devolución de llamada para capturar el registro de MSAL e incorporarlo al registro de su propia aplicación. La firma de la devolución de llamada tiene el siguiente aspecto:

```objc
/*!
    The LogCallback block for the MSAL logger
 
    @param  level           The level of the log message
    @param  message         The message being logged
    @param  containsPII     If the message might contain Personally Identifiable Information (PII)
                            this will be true. Log messages possibly containing PII will not be
                            sent to the callback unless PIllLoggingEnabled is set to YES on the
                            logger.

 */
typedef void (^MSALLogCallback)(MSALLogLevel level, NSString *message, BOOL containsPII);
```

Por ejemplo:

```objc
[MSALGlobalConfig.loggerConfig setLogCallback:^(MSALLogLevel level, NSString *message, BOOL containsPII)
    {
        if (!containsPII)
        {
#if DEBUG
            // IMPORTANT: MSAL logs may contain sensitive information. Never output MSAL logs with NSLog, or print, directly unless you're running your application in debug mode. If you're writing MSAL logs to file, you must store the file securely.
            NSLog(@"MSAL log: %@", message);
#endif
        }
    }];
```

### <a name="personal-data"></a>Datos personales

De forma predeterminada, MSAL no captura ni registra información de identificación personal. La biblioteca permite a los desarrolladores de aplicaciones activar esta opción una propiedad en la clase MSALLogger. Al activar `pii.Enabled`, la aplicación tiene la responsabilidad de controlar la información confidencial y cumplir con los requisitos normativos de forma segura.

```objc
// By default, the `MSALLogger` doesn't capture any PII

// PII will be logged
MSALGlobalConfig.loggerConfig.piiEnabled = YES;

// PII will NOT be logged
MSALGlobalConfig.loggerConfig.piiEnabled = NO;
```

### <a name="logging-levels"></a>Niveles de registro

Para establecer el nivel de registro al iniciar sesión con MSAL para iOS y macOS, use uno de los siguientes valores:

|Nivel  |Descripción |
|---------|---------|
| `MSALLogLevelNothing`| Deshabilitar todos los registros |
| `MSALLogLevelError` | Nivel predeterminado; imprime información solo cuando se producen errores. |
| `MSALLogLevelWarning` | Advertencias |
| `MSALLogLevelInfo` |  Puntos de entrada de la biblioteca, con parámetros y varias operaciones de llavero. |
|`MSALLogLevelVerbose`     |  Seguimiento de API |

Por ejemplo:

```objc
MSALGlobalConfig.loggerConfig.logLevel = MSALLogLevelVerbose;
 ```

 ### <a name="log-message-format"></a>Formato de los mensajes de registro

En los mensajes de registro de MSAL, la parte del mensaje tiene el formato `TID = <thread_id> MSAL <sdk_ver> <OS> <OS_ver> [timestamp - correlation_id] message`.

Por ejemplo:

`TID = 551563 MSAL 0.2.0 iOS Sim 12.0 [2018-09-24 00:36:38 - 36764181-EF53-4E4E-B3E5-16FE362CFC44] acquireToken returning with error: (MSALErrorDomain, -42400) User cancelled the authorization session.`

Proporcionar los identificadores de correlación y las marcas de tiempo son útiles para el seguimiento de problemas. La información del identificador de correlación y las marcas de tiempo está disponible en el mensaje de registro. El único lugar confiable para recuperarlos es en los mensajes de registro de MSAL.

## <a name="swift"></a>[Swift](#tab/swift)

## <a name="msal-for-ios-and-macos-logging-swift"></a>Registro de MSAL para iOS y macOS: Swift

Establezca una devolución de llamada para capturar el registro de MSAL e incorporarlo al registro de su propia aplicación. La firma (representada en Objective-C) para la devolución de llamada tiene el siguiente aspecto:

```objc
/*!
    The LogCallback block for the MSAL logger
 
    @param  level           The level of the log message
    @param  message         The message being logged
    @param  containsPII     If the message might contain Personally Identifiable Information (PII)
                            this will be true. Log messages possibly containing PII will not be
                            sent to the callback unless PIllLoggingEnabled is set to YES on the
                            logger.

 */
typedef void (^MSALLogCallback)(MSALLogLevel level, NSString *message, BOOL containsPII);
```

Por ejemplo:

```swift
MSALGlobalConfig.loggerConfig.setLogCallback { (level, message, containsPII) in
    if let message = message, !containsPII
    {
#if DEBUG
    // IMPORTANT: MSAL logs may contain sensitive information. Never output MSAL logs with NSLog, or print, directly unless you're running your application in debug mode. If you're writing MSAL logs to file, you must store the file securely.
    print("MSAL log: \(message)")
#endif
    }
}
```

### <a name="personal-data"></a>Datos personales

De forma predeterminada, MSAL no captura ni registra información de identificación personal. La biblioteca permite a los desarrolladores de aplicaciones activar esta opción una propiedad en la clase MSALLogger. Al activar `pii.Enabled`, la aplicación tiene la responsabilidad de controlar la información confidencial y cumplir con los requisitos normativos de forma segura.

```swift
// By default, the `MSALLogger` doesn't capture any PII

// PII will be logged
MSALGlobalConfig.loggerConfig.piiEnabled = true

// PII will NOT be logged
MSALGlobalConfig.loggerConfig.piiEnabled = false
```

### <a name="logging-levels"></a>Niveles de registro

Para establecer el nivel de registro al iniciar sesión con MSAL para iOS y macOS, use uno de los siguientes valores:

|Nivel  |Descripción |
|---------|---------|
| `MSALLogLevelNothing`| Deshabilitar todos los registros |
| `MSALLogLevelError` | Nivel predeterminado; imprime información solo cuando se producen errores. |
| `MSALLogLevelWarning` | Advertencias |
| `MSALLogLevelInfo` |  Puntos de entrada de la biblioteca, con parámetros y varias operaciones de llavero. |
|`MSALLogLevelVerbose`     |  Seguimiento de API |

Por ejemplo:

```swift
MSALGlobalConfig.loggerConfig.logLevel = .verbose
 ```

### <a name="log-message-format"></a>Formato de los mensajes de registro

En los mensajes de registro de MSAL, la parte del mensaje tiene el formato `TID = <thread_id> MSAL <sdk_ver> <OS> <OS_ver> [timestamp - correlation_id] message`.

Por ejemplo:

`TID = 551563 MSAL 0.2.0 iOS Sim 12.0 [2018-09-24 00:36:38 - 36764181-EF53-4E4E-B3E5-16FE362CFC44] acquireToken returning with error: (MSALErrorDomain, -42400) User cancelled the authorization session.`

Proporcionar los identificadores de correlación y las marcas de tiempo son útiles para el seguimiento de problemas. La información del identificador de correlación y las marcas de tiempo está disponible en el mensaje de registro. El único lugar confiable para recuperarlos es en los mensajes de registro de MSAL.

## <a name="java"></a>[Java](#tab/java)

## <a name="msal-for-java-logging"></a>Registro en MSAL para Java

MSAL para Java le permite usar la biblioteca de registro que ya usa con la aplicación, siempre y cuando sea compatible con SLF4J. MSAL para Java usa la [fachada de registro simple para Java](http://www.slf4j.org/) (SLF4J) como fachada o abstracción simple para varios marcos de registro, tales como [java.util.logging](https://docs.oracle.com/javase/7/docs/api/java/util/logging/package-summary.html), [Logback](http://logback.qos.ch/) y [Log4j](https://logging.apache.org/log4j/2.x/). SLF4J permite al usuario conectar el marco de registro deseado en el momento de la implementación.

Por ejemplo, para usar Logback como marco de registro de la aplicación, agregue la dependencia Logback al archivo POM de Maven de la aplicación:

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
```

Agregue luego el archivo de configuración Logback:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="true">

</configuration>
```

SLF4J se enlaza automáticamente a Logback en el momento de la implementación. Los registros de MSAL se escribirán en la consola.

Para obtener instrucciones sobre cómo enlazar a otros marcos de registro, consulte el [manual de SLF4J](http://www.slf4j.org/manual.html).

### <a name="personal-and-organization-information"></a>Información personal y de la organización

De forma predeterminada, el registro de MSAL no captura ni registra ningún dato personal o de la organización. En el ejemplo siguiente, el registro de datos personales o de la organización está desactivado de forma predeterminada:

```java
    PublicClientApplication app2 = PublicClientApplication.builder(PUBLIC_CLIENT_ID)
            .authority(AUTHORITY)
            .build();
```

Active el registro de datos personales y de la organización estableciendo `logPii()` en el generador de aplicaciones cliente. Si activa el registro de datos personales o de la organización, la aplicación asume la responsabilidad de controlar de forma segura datos altamente confidenciales y de cumplir los requisitos normativos.

En el ejemplo siguiente, el registro de datos personales o de la organización está habilitado:

```java
PublicClientApplication app2 = PublicClientApplication.builder(PUBLIC_CLIENT_ID)
        .authority(AUTHORITY)
        .logPii(true)
        .build();
```

## <a name="python"></a>[Python](#tab/python)

## <a name="msal-for-python-logging"></a>Registro de MSAL para Python

El registro en MSAL para Python usa el mecanismo de registro estándar de Python; por ejemplo, `logging.info("msg")` Puede configurar el registro de MSAL como se indica a continuación (y verlo en acción en [username_password_sample](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.0.0/sample/username_password_sample.py#L31L32)):

### <a name="enable-debug-logging-for-all-modules"></a>Habilitar el registro de depuración para todos los módulos

De forma predeterminada, el registro de cualquier script de Python está desactivado. Si quiere habilitar el registro de depuración para todos los módulos de su script de Python completo, use:

```python
logging.basicConfig(level=logging.DEBUG)
```

### <a name="silence-only-msal-logging"></a>Silenciar solo el registro de MSAL

Para silenciar solo el registro de la biblioteca MSAL, mientras se habilita el registro de depuración en el resto de módulos del script de Python, desactive el registrador usado por MSAL para Python:

```Python
logging.getLogger("msal").setLevel(logging.WARN)
```

### <a name="personal-and-organizational-data-in-python"></a>Datos personales y organizativos en Python

MSAL para Python no registra datos personales ni datos de la organización. No hay ninguna propiedad para activar o desactivar el registro de datos personales o de la organización.

Puede usar el registro de Python estándar para registrar lo que quiera, pero usted es responsable de controlar los datos confidenciales de forma segura y de cumplir los siguientes requisitos normativos.

Para más información sobre el registro en Python, consulte los [Procedimientos de registro](https://docs.python.org/3/howto/logging.html#logging-basic-tutorial) de Python.

---
