---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/27/2020
ms.author: trbye
ms.openlocfilehash: 82c45919892721d689bd90b7480158c4eea16c03
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87374944"
---
Al desarrollar para macOS, hay tres SDK de voz disponibles.

- El SDK de voz de Objective-C está disponible de forma nativa como un paquete de CocoaPod.
- El SDK de voz de .NET podría usarse con **Xamarin.Mac**, ya que implementa .NET Standard 2.0.
- El SDK de voz de Python está disponible como un módulo de PyPI.

> [!TIP]
> Para más información sobre el uso del SDK de voz de Objective-C con Swift, vea <a href="https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_objective-c_into_swift" target="_blank">Importación de Objective-C en Swift <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

### <a name="system-requirements"></a>Requisitos del sistema

- Una versión macOS 10.13 o posterior

# <a name="xcode"></a>[Xcode](#tab/mac-xcode)

:::row:::
    :::column span="3":::
        El paquete CocoaPod de macOS está disponible para su descarga y uso con el entorno de desarrollo integrado (IDE) de <a href="https://apps.apple.com/us/app/xcode/id497799835" target="_blank">Xcode 9.4.1 o posterior <span class="docon docon-navigate-external x-hidden-focus"></span></a>. En primer lugar, <a href="https://aka.ms/csspeech/macosbinary" target="_blank">descargue el archivo binario de CocoaPod <span class="docon docon-navigate-external x-hidden-focus"></span></a>. Extraiga el pod en el mismo directorio para su uso previsto, cree un *Podfile* y enumere `pod` como `target`.
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Xcode" src="https://docs.microsoft.com/media/logos/logo_xcode.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

```
platform :ios, '9.3'
use_frameworks!

target 'MyApp' do
  pod 'MicrosoftCognitiveServicesSpeech', '~> 1.13.0'
end
```

# <a name="xamarinmac"></a>[Xamarin.Mac](#tab/mac-xamarin)

:::row:::
    :::column span="3":::
        Xamarin.Mac expone el SDK de macOS completo para que los desarrolladores de .NET puedan compilar aplicaciones Mac nativas mediante C#. Para más información, vea <a href="https://docs.microsoft.com/xamarin/mac/" target="_blank">Xamarin.Mac <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Xamarin" src="https://docs.microsoft.com/media/logos/logo_xamarin.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

[!INCLUDE [Get .NET Speech SDK](get-speech-sdk-dotnet.md)]

# <a name="python"></a>[Python](#tab/mac-python)

[!INCLUDE [Get Python Speech SDK](get-speech-sdk-python.md)]

---

#### <a name="additional-resources"></a>Recursos adicionales

- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/objectivec/macos" target="_blank">Código fuente de Objective-C en la guía de inicio rápido del SDK de voz de macOS <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/swift/macos" target="_blank">Código fuente de Swift en la guía de inicio rápido del SDK de voz de macOS <span class="docon docon-navigate-external x-hidden-focus"></span></a>