---
title: 'Registro del SDK de voz: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo habilitar el registro en el SDK de Voz (C++, C#, Python, Objective-C, Java).
services: cognitive-services
author: amitkumarshukla
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/05/2019
ms.author: amishu
ms.custom: devx-track-javascript, devx-track-csharp
ms.openlocfilehash: db7bbfecdb83e12225445905c60afe0999838813
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88918645"
---
# <a name="enable-logging-in-the-speech-sdk"></a>Habilitar el registro en el SDK de voz

El registro en un archivo es una característica opcional del SDK de voz. Durante el desarrollo, el registro proporciona información adicional y diagnósticos de los componentes principales del SDK de voz. Se puede habilitar estableciendo la propiedad `Speech_LogFilename` de un objeto de configuración de voz en la ubicación y el nombre del archivo de registro. El registro se activará globalmente una vez creado un reconocedor a partir de esa configuración que no se pueda deshabilitar más tarde. No se puede cambiar el nombre de un archivo de registro durante la ejecución de una sesión de registro.

> [!NOTE]
> El registro está disponible desde la versión 1.4.0 del SDK de voz en todos los lenguajes de programación admitidos a excepción de JavaScript.

## <a name="sample"></a>Muestra

El nombre del archivo de registro se especifica en un objeto de configuración. Tomando `SpeechConfig` como ejemplo y suponiendo que ha creado una instancia denominada `config`:

```csharp
config.SetProperty(PropertyId.Speech_LogFilename, "LogfilePathAndName");
```

```java
config.setProperty(PropertyId.Speech_LogFilename, "LogfilePathAndName");
```

```C++
config->SetProperty(PropertyId::Speech_LogFilename, "LogfilePathAndName");
```

```Python
config.set_property(speechsdk.PropertyId.Speech_LogFilename, "LogfilePathAndName")
```

```objc
[config setPropertyTo:@"LogfilePathAndName" byId:SPXSpeechLogFilename];
```

Puede crear un reconocedor a partir del objeto de configuración. Esto habilitará el registro de todos los reconocedores.

> [!NOTE]
> Si crea un `SpeechSynthesizer` a partir del objeto de configuración, no se habilitará el registro. No obstante, si el registro está habilitado, también recibirá los diagnósticos desde `SpeechSynthesizer`.

## <a name="create-a-log-file-on-different-platforms"></a>Creación de un archivo de registro en distintas plataformas

Para Windows o Linux, el archivo de registro puede estar en cualquier ruta sobre la que el usuario tenga permiso de escritura. Los permisos de escritura en ubicaciones del sistema de archivos de otros sistemas operativos puede estar limitada o restringida de forma predeterminada.

### <a name="universal-windows-platform-uwp"></a>Plataforma universal de Windows (UWP)

Las aplicaciones de UWP requieren que los archivos de registro estén colocados en una de las ubicaciones de datos de la aplicación (local, roaming o temporal). Se puede crear un archivo de registro en la carpeta de la aplicación local:

```csharp
StorageFolder storageFolder = ApplicationData.Current.LocalFolder;
StorageFile logFile = await storageFolder.CreateFileAsync("logfile.txt", CreationCollisionOption.ReplaceExisting);
config.SetProperty(PropertyId.Speech_LogFilename, logFile.Path);
```

[Aquí](https://docs.microsoft.com/windows/uwp/files/file-access-permissions) hay más información sobre el permiso de acceso a los archivos de las aplicaciones de UWP.

### <a name="android"></a>Android

Puede guardar un archivo de registro en un almacenamiento interno, externo o en el directorio de la memoria caché. Los archivos creados en el almacenamiento interno o en el directorio de la memoria caché son privados para la aplicación. Es preferible crear un archivo de registro en el almacenamiento externo.

```java
File dir = context.getExternalFilesDir(null);
File logFile = new File(dir, "logfile.txt");
config.setProperty(PropertyId.Speech_LogFilename, logFile.getAbsolutePath());
```

El código anterior guarda un archivo de registro en el almacenamiento externo de la raíz de un directorio específico de la aplicación. Un usuario puede acceder al archivo con el administrador de archivos (normalmente en `Android/data/ApplicationName/logfile.txt`). El archivo se eliminará cuando se desinstale la aplicación.

También deberá solicitar el permiso `WRITE_EXTERNAL_STORAGE` sobre el archivo de manifiesto:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="...">
  ...
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  ...
</manifest>
```

[Aquí](https://developer.android.com/guide/topics/data/data-storage.html) hay disponible más información sobre almacenamiento de datos y de archivos para aplicaciones de Android.

#### <a name="ios"></a>iOS

Solo los directorios dentro del espacio aislado de la aplicación son accesibles. Los archivos se pueden crear en los directorios de documentos, biblioteca y temporales. Los archivos del directorio de documentos pueden estar a disposición de un usuario. El fragmento de código siguiente muestra la creación de un archivo de registro en el directorio de documentos de la aplicación:

```objc
NSString *filePath = [
  [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject]
    stringByAppendingPathComponent:@"logfile.txt"];
[speechConfig setPropertyTo:filePath byId:SPXSpeechLogFilename];
```

Para acceder a un archivo creado, agregue las siguientes propiedades a la lista de propiedades `Info.plist` de la aplicación:

```xml
<key>UIFileSharingEnabled</key>
<true/>
<key>LSSupportsOpeningDocumentsInPlace</key>
<true/>
```

[Aquí](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html) hay más información disponible sobre el sistema de archivos de iOS.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Exploración de ejemplos en GitHub](https://aka.ms/csspeech/samples)
