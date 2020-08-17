---
title: Selección de un dispositivo de entrada de audio con el SDK de Voz
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre la selección de dispositivos de entrada de audio en el SDK de Voz (C++, C#, Python, Objective-C, Java, JavaScript) mediante la obtención de los identificadores de los dispositivos de audio conectados a un sistema.
services: cognitive-services
author: chlandsi
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/05/2019
ms.author: chlandsi
ms.custom: devx-track-javascript
ms.openlocfilehash: 80671a40b5933a89962fa07b267d7e99e475657f
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406697"
---
# <a name="how-to-select-an-audio-input-device-with-the-speech-sdk"></a>Procedimientos: Selección de un dispositivo de entrada de audio con el SDK de Voz

La versión 1.3.0 del SDK de Voz presenta una API para seleccionar la entrada de audio. En este artículo se describe cómo obtener los identificadores de los dispositivos de audio conectados a un sistema. El SDK de Voz puede usar estos posteriormente configurando el dispositivo de audio mediante el objeto `AudioConfig`:

```C++
audioConfig = AudioConfig.FromMicrophoneInput("<device id>");
```

```cs
audioConfig = AudioConfig.FromMicrophoneInput("<device id>");
```

```Python
audio_config = AudioConfig(device_name="<device id>");
```

```objc
audioConfig = AudioConfiguration.FromMicrophoneInput("<device id>");
```

```Java
audioConfig = AudioConfiguration.fromMicrophoneInput("<device id>");
```

```JavaScript
audioConfig = AudioConfiguration.fromMicrophoneInput("<device id>");
```

> [!Note]
> El uso del micrófono no está disponible para JavaScript cuando se ejecuta en Node.js.

## <a name="audio-device-ids-on-windows-for-desktop-applications"></a>Id. de dispositivos de audio en Windows para aplicaciones de escritorio

Las [cadena del identificador de punto de conexión](/windows/desktop/CoreAudio/endpoint-id-strings) del dispositivo de audio se puede recuperar del objeto [`IMMDevice`](/windows/desktop/api/mmdeviceapi/nn-mmdeviceapi-immdevice) de Windows para las aplicaciones de escritorio.

En el siguiente ejemplo de código se muestra cómo usarlo para enumerar los dispositivos de audio de C++:

```cpp
#include <cstdio>
#include <mmdeviceapi.h>

#include <Functiondiscoverykeys_devpkey.h>

const CLSID CLSID_MMDeviceEnumerator = __uuidof(MMDeviceEnumerator);
const IID IID_IMMDeviceEnumerator = __uuidof(IMMDeviceEnumerator);

constexpr auto REFTIMES_PER_SEC = (10000000 * 25);
constexpr auto REFTIMES_PER_MILLISEC = 10000;

#define EXIT_ON_ERROR(hres)  \
              if (FAILED(hres)) { goto Exit; }
#define SAFE_RELEASE(punk)  \
              if ((punk) != NULL)  \
                { (punk)->Release(); (punk) = NULL; }

void ListEndpoints();

int main()
{
    CoInitializeEx(NULL, COINIT_MULTITHREADED);
    ListEndpoints();
}

//-----------------------------------------------------------
// This function enumerates all active (plugged in) audio
// rendering endpoint devices. It prints the friendly name
// and endpoint ID string of each endpoint device.
//-----------------------------------------------------------
void ListEndpoints()
{
    HRESULT hr = S_OK;
    IMMDeviceEnumerator *pEnumerator = NULL;
    IMMDeviceCollection *pCollection = NULL;
    IMMDevice *pEndpoint = NULL;
    IPropertyStore *pProps = NULL;
    LPWSTR pwszID = NULL;

    hr = CoCreateInstance(CLSID_MMDeviceEnumerator, NULL, CLSCTX_ALL, IID_IMMDeviceEnumerator, (void**)&pEnumerator);
    EXIT_ON_ERROR(hr);

    hr = pEnumerator->EnumAudioEndpoints(eCapture, DEVICE_STATE_ACTIVE, &pCollection);
    EXIT_ON_ERROR(hr);

    UINT  count;
    hr = pCollection->GetCount(&count);
    EXIT_ON_ERROR(hr);

    if (count == 0)
    {
        printf("No endpoints found.\n");
    }

    // Each iteration prints the name of an endpoint device.
    PROPVARIANT varName;
    for (ULONG i = 0; i < count; i++)
    {
        // Get pointer to endpoint number i.
        hr = pCollection->Item(i, &pEndpoint);
        EXIT_ON_ERROR(hr);

        // Get the endpoint ID string.
        hr = pEndpoint->GetId(&pwszID);
        EXIT_ON_ERROR(hr);

        hr = pEndpoint->OpenPropertyStore(
            STGM_READ, &pProps);
        EXIT_ON_ERROR(hr);

        // Initialize container for property value.
        PropVariantInit(&varName);

        // Get the endpoint's friendly-name property.
        hr = pProps->GetValue(PKEY_Device_FriendlyName, &varName);
        EXIT_ON_ERROR(hr);

        // Print endpoint friendly name and endpoint ID.
        printf("Endpoint %d: \"%S\" (%S)\n", i, varName.pwszVal, pwszID);
    }

Exit:
    CoTaskMemFree(pwszID);
    pwszID = NULL;
    PropVariantClear(&varName);
    SAFE_RELEASE(pEnumerator);
    SAFE_RELEASE(pCollection);
    SAFE_RELEASE(pEndpoint);
    SAFE_RELEASE(pProps);
}
```

En C#, la biblioteca [NAudio](https://github.com/naudio/NAudio) se puede utilizar para acceder a CoreAudio API y enumerar los dispositivos de la siguiente manera:

```cs
using System;

using NAudio.CoreAudioApi;

namespace ConsoleApp
{
    class Program
    {
        static void Main(string[] args)
        {
            var enumerator = new MMDeviceEnumerator();
            foreach (var endpoint in
                     enumerator.EnumerateAudioEndPoints(DataFlow.Capture, DeviceState.Active))
            {
                Console.WriteLine("{0} ({1})", endpoint.FriendlyName, endpoint.ID);
            }
        }
    }
}
```

Este sería un ejemplo de id. de dispositivo: `{0.0.1.00000000}.{5f23ab69-6181-4f4a-81a4-45414013aac8}`.

## <a name="audio-device-ids-on-uwp"></a>Id. de dispositivos de audio en UWP

En la Plataforma universal de Windows (UWP), se pueden obtener dispositivos de entrada de audio mediante la propiedad `Id()` del objeto [`DeviceInformation`](/uwp/api/windows.devices.enumeration.deviceinformation) correspondiente.

Los siguientes ejemplos de código muestran cómo hacer esto en C++ y C#:

```cpp
#include <winrt/Windows.Foundation.h>
#include <winrt/Windows.Devices.Enumeration.h>

using namespace winrt::Windows::Devices::Enumeration;

void enumerateDeviceIds()
{
    auto promise = DeviceInformation::FindAllAsync(DeviceClass::AudioCapture);

    promise.Completed(
        [](winrt::Windows::Foundation::IAsyncOperation<DeviceInformationCollection> const& sender,
           winrt::Windows::Foundation::AsyncStatus /* asyncStatus */ ) {
        auto info = sender.GetResults();
        auto num_devices = info.Size();

        for (const auto &device : info)
        {
            std::wstringstream ss{};
            ss << "looking at device (of " << num_devices << "): " << device.Id().c_str() << "\n";
            OutputDebugString(ss.str().c_str());
        }
    });
}
```

```cs
using Windows.Devices.Enumeration;
using System.Linq;

namespace helloworld {
    private async void EnumerateDevices()
    {
        var devices = await DeviceInformation.FindAllAsync(DeviceClass.AudioCapture);

        foreach (var device in devices)
        {
            Console.WriteLine($"{device.Name}, {device.Id}\n");
        }
    }
}
```

Este sería un ejemplo de id. de dispositivo: `\\\\?\\SWD#MMDEVAPI#{0.0.1.00000000}.{5f23ab69-6181-4f4a-81a4-45414013aac8}#{2eef81be-33fa-4800-9670-1cd474972c3f}`.

## <a name="audio-device-ids-on-linux"></a>Id. de dispositivos de audio en Linux

Los identificadores de dispositivo se seleccionan mediante identificadores de dispositivo ALSA estándar.

Los identificadores de las entradas conectadas al sistema se encuentran en la salida del comando `arecord -L`.
Como alternativa, estas se pueden obtener mediante la [biblioteca C de ALSA](https://www.alsa-project.org/alsa-doc/alsa-lib/).

Los identificadores de ejemplo son `hw:1,0` y `hw:CARD=CC,DEV=0`.

## <a name="audio-device-ids-on-macos"></a>Id. de dispositivos de audio en macOS

La siguiente función que se implementa en Objective-C crea una lista de los nombres e identificadores de los dispositivos de audio conectados a un Mac.

La cadena `deviceUID` se usa para identificar un dispositivo en el SDK de Voz de macOS.

```objc
#import <Foundation/Foundation.h>
#import <CoreAudio/CoreAudio.h>

CFArrayRef CreateInputDeviceArray()
{
    AudioObjectPropertyAddress propertyAddress = {
        kAudioHardwarePropertyDevices,
        kAudioObjectPropertyScopeGlobal,
        kAudioObjectPropertyElementMaster
    };

    UInt32 dataSize = 0;
    OSStatus status = AudioObjectGetPropertyDataSize(kAudioObjectSystemObject, &propertyAddress, 0, NULL, &dataSize);
    if (kAudioHardwareNoError != status) {
        fprintf(stderr, "AudioObjectGetPropertyDataSize (kAudioHardwarePropertyDevices) failed: %i\n", status);
        return NULL;
    }

    UInt32 deviceCount = (uint32)(dataSize / sizeof(AudioDeviceID));

    AudioDeviceID *audioDevices = (AudioDeviceID *)(malloc(dataSize));
    if (NULL == audioDevices) {
        fputs("Unable to allocate memory", stderr);
        return NULL;
    }

    status = AudioObjectGetPropertyData(kAudioObjectSystemObject, &propertyAddress, 0, NULL, &dataSize, audioDevices);
    if (kAudioHardwareNoError != status) {
        fprintf(stderr, "AudioObjectGetPropertyData (kAudioHardwarePropertyDevices) failed: %i\n", status);
        free(audioDevices);
        audioDevices = NULL;
        return NULL;
    }

    CFMutableArrayRef inputDeviceArray = CFArrayCreateMutable(kCFAllocatorDefault, deviceCount, &kCFTypeArrayCallBacks);
    if (NULL == inputDeviceArray) {
        fputs("CFArrayCreateMutable failed", stderr);
        free(audioDevices);
        audioDevices = NULL;
        return NULL;
    }

    // Iterate through all the devices and determine which are input-capable
    propertyAddress.mScope = kAudioDevicePropertyScopeInput;
    for (UInt32 i = 0; i < deviceCount; ++i) {
        // Query device UID
        CFStringRef deviceUID = NULL;
        dataSize = sizeof(deviceUID);
        propertyAddress.mSelector = kAudioDevicePropertyDeviceUID;
        status = AudioObjectGetPropertyData(audioDevices[i], &propertyAddress, 0, NULL, &dataSize, &deviceUID);
        if (kAudioHardwareNoError != status) {
            fprintf(stderr, "AudioObjectGetPropertyData (kAudioDevicePropertyDeviceUID) failed: %i\n", status);
            continue;
        }

        // Query device name
        CFStringRef deviceName = NULL;
        dataSize = sizeof(deviceName);
        propertyAddress.mSelector = kAudioDevicePropertyDeviceNameCFString;
        status = AudioObjectGetPropertyData(audioDevices[i], &propertyAddress, 0, NULL, &dataSize, &deviceName);
        if (kAudioHardwareNoError != status) {
            fprintf(stderr, "AudioObjectGetPropertyData (kAudioDevicePropertyDeviceNameCFString) failed: %i\n", status);
            continue;
        }

        // Determine if the device is an input device (it is an input device if it has input channels)
        dataSize = 0;
        propertyAddress.mSelector = kAudioDevicePropertyStreamConfiguration;
        status = AudioObjectGetPropertyDataSize(audioDevices[i], &propertyAddress, 0, NULL, &dataSize);
        if (kAudioHardwareNoError != status) {
            fprintf(stderr, "AudioObjectGetPropertyDataSize (kAudioDevicePropertyStreamConfiguration) failed: %i\n", status);
            continue;
        }

        AudioBufferList *bufferList = (AudioBufferList *)(malloc(dataSize));
        if (NULL == bufferList) {
            fputs("Unable to allocate memory", stderr);
            break;
        }

        status = AudioObjectGetPropertyData(audioDevices[i], &propertyAddress, 0, NULL, &dataSize, bufferList);
        if (kAudioHardwareNoError != status || 0 == bufferList->mNumberBuffers) {
            if (kAudioHardwareNoError != status)
                fprintf(stderr, "AudioObjectGetPropertyData (kAudioDevicePropertyStreamConfiguration) failed: %i\n", status);
            free(bufferList);
            bufferList = NULL;
            continue;
        }

        free(bufferList);
        bufferList = NULL;

        // Add a dictionary for this device to the array of input devices
        CFStringRef keys    []  = { CFSTR("deviceUID"),     CFSTR("deviceName")};
        CFStringRef values  []  = { deviceUID,              deviceName};

        CFDictionaryRef deviceDictionary = CFDictionaryCreate(kCFAllocatorDefault,
                                                              (const void **)(keys),
                                                              (const void **)(values),
                                                              2,
                                                              &kCFTypeDictionaryKeyCallBacks,
                                                              &kCFTypeDictionaryValueCallBacks);

        CFArrayAppendValue(inputDeviceArray, deviceDictionary);

        CFRelease(deviceDictionary);
        deviceDictionary = NULL;
    }

    free(audioDevices);
    audioDevices = NULL;

    // Return a non-mutable copy of the array
    CFArrayRef immutableInputDeviceArray = CFArrayCreateCopy(kCFAllocatorDefault, inputDeviceArray);
    CFRelease(inputDeviceArray);
    inputDeviceArray = NULL;

    return immutableInputDeviceArray;
}
```

Por ejemplo, el id. de usuario para el micrófono integrado es `BuiltInMicrophoneDevice`.

## <a name="audio-device-ids-on-ios"></a>Id. de dispositivos de audio en iOS

La selección de dispositivos de audio con el SDK de Voz no se admite en iOS. No obstante, las aplicaciones que usan el SDK pueden influir en el enrutamiento de audio mediante la plataforma [`AVAudioSession`](https://developer.apple.com/documentation/avfoundation/avaudiosession?language=objc).

Por ejemplo, la instrucción

```objc
[[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategoryRecord
    withOptions:AVAudioSessionCategoryOptionAllowBluetooth error:NULL];
```

permite el uso de un auricular Bluetooth para una aplicación habilitada para voz.

## <a name="audio-device-ids-in-javascript"></a>Identificadores de dispositivos de audio en JavaScript

En JavaScript, el método [MediaDevices.enumerateDevices()](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/enumerateDevices) puede usarse para enumerar los dispositivos multimedia y buscar un identificador de dispositivo para pasar a `fromMicrophone(...)`.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Exploración de ejemplos en GitHub](https://aka.ms/csspeech/samples)

## <a name="see-also"></a>Consulte también

- [Personalización de modelos acústicos](how-to-customize-acoustic-models.md)
- [Personalización de modelos de lenguaje](how-to-customize-language-model.md)
