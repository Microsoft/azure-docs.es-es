---
title: 'Tutorial: Creación de una aplicación iOS que toma una foto y la inicia en el Lector inmersivo (Swift)'
titleSuffix: Azure Cognitive Services
description: En este tutorial, creará una aplicación iOS desde cero y agregará la imagen a la funcionalidad de Lector inmersivo.
services: cognitive-services
author: metanMSFT
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: tutorial
ms.date: 01/14/2020
ms.author: metan
ms.openlocfilehash: 968f3c9fbfea1e2f04fb029605173087f6f311c0
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88516510"
---
# <a name="tutorial-create-an-ios-app-that-launches-the-immersive-reader-with-content-from-a-photo-swift"></a>Tutorial: Creación de una aplicación iOS que inicia el Lector inmersivo con contenido de una foto (Swift)

El [Lector inmersivo](https://www.onenote.com/learningtools) es una herramienta diseñada de forma inclusiva que implementa técnicas demostradas para mejorar la comprensión lectora.

La [API Read de Computer Vision de Microsoft Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-recognizing-text) detecta el contenido de texto de una imagen con los últimos modelos de reconocimiento de Microsoft y convierte el texto identificado en una secuencia de caracteres legible por una máquina.

En este tutorial, creará una aplicación iOS desde cero e integrará la API Read, así como el Lector inmersivo mediante su SDK. Se puede encontrar un completo ejemplo funcional [aquí](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/ios).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

* [Xcode](https://apps.apple.com/us/app/xcode/id497799835?mt=12)
* Un recurso del Lector inmersivo configurado para la autenticación de Azure Active Directory. Siga [estas instrucciones](./how-to-create-immersive-reader.md) para realizar la configuración. Al configurar las propiedades del proyecto de ejemplo, necesitará algunos de los valores creados aquí. Guarde la salida de la sesión en un archivo de texto para futuras referencias.
* El uso de este ejemplo requiere una suscripción de Azure a Computer Vision de Cognitive Services. [Cree un recurso de Computer Vision de Cognitive Services en Azure Portal](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision).

## <a name="create-an-xcode-project"></a>Creación de un proyecto de Xcode

Cree un proyecto nuevo en Xcode.

![Nuevo proyecto](./media/ios/xcode-create-project.png)

Elija **Single View App** (Aplicación de vista única).

![Nueva aplicación de vista única](./media/ios/xcode-single-view-app.png)

## <a name="get-the-sdk-cocoapod"></a>Obtención del SDK CocoaPod
La manera más sencilla de usar el SDK de Lector inmersivo es a través de CocoaPods. Para instalarlo a través de CocoaPods:
1. [Instale CocoaPods](http://guides.cocoapods.org/using/getting-started.html): siga la guía de introducción para instalar CocoaPods.
2. Ejecute `pod init` en el directorio raíz del proyecto de Xcode para crear un podfile.
3.  Agregue `pod 'immersive-reader-sdk', :path => 'https://github.com/microsoft/immersive-reader-sdk/tree/master/iOS/immersive-reader-sdk'` para incorporar CocoaPod al podfile. El podfile debe tener un aspecto similar al siguiente, reemplazando picture-to-immersive-reader-swift por el nombre del destino:
 ```ruby
  platform :ios, '9.0'

  target 'picture-to-immersive-reader-swift' do
  use_frameworks!
  # Pods for picture-to-immersive-reader-swift
  pod 'immersive-reader-sdk', :git => 'https://github.com/microsoft/immersive-reader-sdk.git'
  end
```
4. En el terminal, en el directorio del proyecto de Xcode, ejecute el comando `pod install` para instalar el pod del SDK de Lector inmersivo.
5. Agregue `import immersive_reader_sdk` a todos los archivos que necesitan hacer referencia al SDK.
6. Asegúrese de abrir el proyecto abriendo el archivo `.xcworkspace` y no el archivo `.xcodeproj`.

## <a name="acquire-an-azure-ad-authentication-token"></a>Adquisición de un token de autenticación de Azure AD

Para esta parte, necesitará algunos valores del paso anterior de requisitos previos de configuración de la autenticación de Azure AD. Vuelva a consultar el archivo de texto que ha guardado de esa sesión.

````text
TenantId     => Azure subscription TenantId
ClientId     => Azure AD ApplicationId
ClientSecret => Azure AD Application Service Principal password
Subdomain    => Immersive Reader resource subdomain (resource 'Name' if the resource was created in the Azure portal, or 'CustomSubDomain' option if the resource was created with Azure CLI Powershell. Check the Azure portal for the subdomain on the Endpoint in the resource Overview page, for example, 'https://[SUBDOMAIN].cognitiveservices.azure.com/')
````

En la carpeta principal del proyecto, que contiene el archivo ViewController.swift, cree un archivo de clase Swift denominado Constants.swift. Reemplace la clase por el código siguiente y agregue sus propios valores cuando proceda. Conserve este archivo como un archivo local que solo existe en el equipo y asegúrese de no confirmar este archivo en el control de código fuente, ya que contiene secretos que no deben hacerse públicos. Se recomienda no mantener secretos en la aplicación. En su lugar, se recomienda usar un servicio back-end para obtener el token; así los secretos se pueden mantener fuera de la aplicación y del dispositivo. El punto de conexión de la API de back-end debe estar protegido mediante algún tipo de autenticación (por ejemplo, [OAuth](https://oauth.net/2/)) para evitar que usuarios no autorizados obtengan tokens y los utilicen en el servicio Lector inmersivo y su facturación; este trabajo está fuera del ámbito de este tutorial.

## <a name="set-up-the-app-to-run-without-a-storyboard"></a>Configuración de la aplicación para que se ejecute sin un guion gráfico

Abra AppDelegate.swift y reemplace el archivo por el siguiente código.

## <a name="add-functionality-for-taking-and-uploading-photos"></a>Incorporación de funcionalidad para tomar y cargar fotos

Cambie el nombre de ViewController.swift a PictureLaunchViewController.swift y reemplace el archivo por el código siguiente.

## <a name="build-and-run-the-app"></a>Compilación y ejecución de la aplicación

Seleccione como destino un simulador o un dispositivo para establecer el esquema de archivo en Xcode.
![Esquema de archivo](./media/ios/xcode-archive-scheme.png)<br/>
![Selección del destino](./media/ios/xcode-select-target.png)

En Xcode, presione Ctrl+R o haga clic en el botón de reproducción para ejecutar el proyecto; la aplicación debe iniciarse en el simulador o el dispositivo especificados.

En la aplicación, debe ver lo siguiente:

![Aplicación de ejemplo](./media/ios/picture-to-immersive-reader-ipad-app.png)

Dentro de la aplicación, haga clic en el botón 'Take Photo’ (Tomar foto) o ’Choose Photo from Library’ (Elegir foto de la biblioteca) ' para tomar o cargar una foto de texto; a continuación, el Lector inmersivo abrirá el texto de la foto.

![Lector inmersivo](./media/ios/picture-to-immersive-reader-ipad.png)

## <a name="next-steps"></a>Pasos siguientes

* Lea la [Referencia del SDK del Lector inmersivo](./reference.md)
