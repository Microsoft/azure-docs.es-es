---
title: Azure Media Services v3 con plantilla de licencia de Microsoft PlayReady
description: En este tema se ofrece información general sobre la plantilla de licencia de PlayReady que se usa para configurar las licencias de PlayReady.
author: juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/16/2018
ms.author: juliako
ms.openlocfilehash: ba4b8de4b8cf1d1d1b8124185a8006eb7695b21e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074513"
---
# <a name="media-services-v3-with-playready-license-template"></a>Media Services v3 con plantilla de licencia de PlayReady 

Azure Media Services le permite cifrar el contenido HLS con **Microsoft PlayReady**. Asimismo, Media Services proporciona un servicio para entregar licencias de PlayReady. Puede usar la API de Media Services para configurar las licencias de PlayReady. Cuando el reproductor intenta reproducir contenido protegido de PlayReady, se envía una solicitud al servicio de entrega de licencias para obtener una licencia. Si el servicio de licencia aprueba la solicitud, emite la licencia, la que se envía al cliente y se usa para descifrar y reproducir el contenido especificado.

Las licencias de PlayReady contienen los derechos y restricciones que desea aplicar en el entorno de tiempo de ejecución de administración de derechos digitales (DRM) de PlayReady cuando un usuario intenta reproducir contenido protegido. A continuación se muestran algunos ejemplos de restricciones de licencia de PlayReady que puede especificar:

* Fecha y hora desde las cuales la licencia es válida.
* Fecha y hora cuando caduca la licencia. 
* Para que la licencia se guarde en un almacenamiento persistente en el cliente. Las licencias permanentes se usan normalmente para permitir la reproducción sin conexión de contenido.
* El nivel de seguridad mínimo que debe tener un reproductor para reproducir contenido. 
* El nivel de protección de salida de los controles de salida para el contenido de audio y video. 
* Para más información, consulte la sección Controles de salida (3.5) en el documento [Reglas de cumplimiento normativo de PlayReady](https://www.microsoft.com/playready/licensing/compliance/).

> [!NOTE]
> Actualmente, solo puede configurar el derecho de reproducción PlayRight de la licencia de PlayReady. Este derecho es obligatorio. PlayRight da al cliente la capacidad de reproducir el contenido. También puede usar PlayRight para configurar restricciones específicas para la reproducción. 
> 

En este tema se describe cómo configurar las licencias de PlayReady con Media Services.

## <a name="basic-streaming-license-example"></a>Ejemplo de licencia básica de streaming

En el ejemplo siguiente se muestra la plantilla más sencilla (y común) que configura una licencia básica de streaming. Con esta licencia, los clientes pueden reproducir el contenido protegido con PlayReady. 

El código XML se ajusta al esquema XML de la plantilla de licencias de PlayReady definido en la sección [Esquema XML de la plantilla de licencias de PlayReady](#schema).

```xml
<?xml version="1.0" encoding="utf-8"?>
<PlayReadyLicenseResponseTemplate xmlns:i="https://www.w3.org/2001/XMLSchema-instance" 
                                  xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1">
    <LicenseTemplates>
    <PlayReadyLicenseTemplate>
        <ContentKey i:type="ContentEncryptionKeyFromHeader" />
        <PlayRight />
    </PlayReadyLicenseTemplate>
    </LicenseTemplates>
</PlayReadyLicenseResponseTemplate>
```

## <a name="use-media-services-apis-to-configure-license-templates"></a><a id="classes"></a>Usar Media Services API para configurar las plantillas de licencia

Media Services proporciona tipos que puede usar para configurar una plantilla de licencias de PlayReady. 

El fragmento de código siguiente usa las clases de .NET de Media Services para configurar la plantilla de licencias de PlayReady. Las clases se definen en el espacio de nombres [Microsoft.Azure.Management.Media.Models](/dotnet/api/microsoft.azure.management.media.models?view=azure-dotnet). El fragmento de código configura el derecho de reproducción PlayRight de la licencia de PlayReady. PlayRight concede al usuario la capacidad de reproducir contenido sujeto a cualquier restricción configurada en la licencia y en el propio derecho PlayRight (para la directiva de reproducción específica). Gran parte de la directiva de PlayRight tiene que ver con las restricciones de salida que controlan los tipos de salidas sobre las que se puede reproducir el contenido. También incluye las restricciones que se deben incluir cuando se usa una salida específica. Por ejemplo, si la opción DigitalVideoOnlyContentRestriction está habilitada, el entorno de tiempo de ejecución de DRM permite que el vídeo se muestre sobre las salidas digitales. (Las salidas de vídeo analógicas no pueden transmitir el contenido).

> [!IMPORTANT]
> La licencia de PlayReady tiene restricciones que son muy eficaces. Si las protecciones de salida son demasiado restrictivas, puede que el contenido no se reproduzca en algunos clientes. Para más información, consulte el documento [PlayReady Compliance Rules](https://www.microsoft.com/playready/licensing/compliance/).

### <a name="configure-playready-license-template-with-net"></a>Configurar la plantilla de licencia de PlayReady con .NET

```csharp
ContentKeyPolicyPlayReadyLicense objContentKeyPolicyPlayReadyLicense;
objContentKeyPolicyPlayReadyLicense = new ContentKeyPolicyPlayReadyLicense
{
    AllowTestDevices = true,
    BeginDate = new DateTime(2016, 1, 1),
    ContentKeyLocation = new ContentKeyPolicyPlayReadyContentEncryptionKeyFromHeader(),
    ContentType = ContentKeyPolicyPlayReadyContentType.UltraVioletStreaming,
    LicenseType = drmSettings.EnableOfflineMode ? ContentKeyPolicyPlayReadyLicenseType.Persistent : ContentKeyPolicyPlayReadyLicenseType.NonPersistent,
    PlayRight = new ContentKeyPolicyPlayReadyPlayRight
    {
        ImageConstraintForAnalogComponentVideoRestriction = true,
        ExplicitAnalogTelevisionOutputRestriction = new ContentKeyPolicyPlayReadyExplicitAnalogTelevisionRestriction(true, 2),
        AllowPassingVideoContentToUnknownOutput = ContentKeyPolicyPlayReadyUnknownOutputPassingOption.Allowed,
        FirstPlayExpiration = TimeSpan.FromSeconds(20.0),
    }
};
```

## <a name="playready-license-template-xml-schema"></a><a id="schema"></a>Esquema XML de la plantilla de licencia de PlayReady

```xml
<?xml version="1.0" encoding="utf-8"?>
<xs:schema xmlns:tns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1" xmlns:ser="http://schemas.microsoft.com/2003/10/Serialization/" elementFormDefault="qualified" targetNamespace="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1" xmlns:xs="https://www.w3.org/2001/XMLSchema">
    <xs:import namespace="http://schemas.microsoft.com/2003/10/Serialization/" />
    <xs:complexType name="AgcAndColorStripeRestriction">
        <xs:sequence>
            <xs:element minOccurs="0" name="ConfigurationData" type="xs:unsignedByte" />
        </xs:sequence>
    </xs:complexType>
    <xs:element name="AgcAndColorStripeRestriction" nillable="true" type="tns:AgcAndColorStripeRestriction" />
    <xs:simpleType name="ContentType">
        <xs:restriction base="xs:string">
            <xs:enumeration value="Unspecified" />
            <xs:enumeration value="UltravioletDownload" />
            <xs:enumeration value="UltravioletStreaming" />
        </xs:restriction>
    </xs:simpleType>
    <xs:element name="ContentType" nillable="true" type="tns:ContentType" />
    <xs:complexType name="ExplicitAnalogTelevisionRestriction">
        <xs:sequence>
            <xs:element minOccurs="0" name="BestEffort" type="xs:boolean" />
            <xs:element minOccurs="0" name="ConfigurationData" type="xs:unsignedByte" />
        </xs:sequence>
    </xs:complexType>
    <xs:element name="ExplicitAnalogTelevisionRestriction" nillable="true" type="tns:ExplicitAnalogTelevisionRestriction" />
    <xs:complexType name="PlayReadyContentKey">
        <xs:sequence />
    </xs:complexType>
    <xs:element name="PlayReadyContentKey" nillable="true" type="tns:PlayReadyContentKey" />
    <xs:complexType name="ContentEncryptionKeyFromHeader">
        <xs:complexContent mixed="false">
            <xs:extension base="tns:PlayReadyContentKey">
                <xs:sequence />
            </xs:extension>
        </xs:complexContent>
    </xs:complexType>
    <xs:element name="ContentEncryptionKeyFromHeader" nillable="true" type="tns:ContentEncryptionKeyFromHeader" />
    <xs:complexType name="ContentEncryptionKeyFromKeyIdentifier">
        <xs:complexContent mixed="false">
            <xs:extension base="tns:PlayReadyContentKey">
                <xs:sequence>
                    <xs:element minOccurs="0" name="KeyIdentifier" type="ser:guid" />
                </xs:sequence>
            </xs:extension>
        </xs:complexContent>
    </xs:complexType>
    <xs:element name="ContentEncryptionKeyFromKeyIdentifier" nillable="true" type="tns:ContentEncryptionKeyFromKeyIdentifier" />
    <xs:complexType name="PlayReadyLicenseResponseTemplate">
        <xs:sequence>
            <xs:element name="LicenseTemplates" nillable="true" type="tns:ArrayOfPlayReadyLicenseTemplate" />
            <xs:element minOccurs="0" name="ResponseCustomData" nillable="true" type="xs:string">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
        </xs:sequence>
    </xs:complexType>
    <xs:element name="PlayReadyLicenseResponseTemplate" nillable="true" type="tns:PlayReadyLicenseResponseTemplate" />
    <xs:complexType name="ArrayOfPlayReadyLicenseTemplate">
        <xs:sequence>
            <xs:element minOccurs="0" maxOccurs="unbounded" name="PlayReadyLicenseTemplate" nillable="true" type="tns:PlayReadyLicenseTemplate" />
        </xs:sequence>
    </xs:complexType>
    <xs:element name="ArrayOfPlayReadyLicenseTemplate" nillable="true" type="tns:ArrayOfPlayReadyLicenseTemplate" />
    <xs:complexType name="PlayReadyLicenseTemplate">
        <xs:sequence>
            <xs:element minOccurs="0" name="AllowTestDevices" type="xs:boolean" />
            <xs:element minOccurs="0" name="BeginDate" nillable="true" type="xs:dateTime">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element name="ContentKey" nillable="true" type="tns:PlayReadyContentKey" />
            <xs:element minOccurs="0" name="ContentType" type="tns:ContentType">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="ExpirationDate" nillable="true" type="xs:dateTime">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="GracePeriod" nillable="true" type="ser:duration">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="LicenseType" type="tns:PlayReadyLicenseType" />
            <xs:element minOccurs="0" name="PlayRight" nillable="true" type="tns:PlayReadyPlayRight" />
        </xs:sequence>
    </xs:complexType>
    <xs:element name="PlayReadyLicenseTemplate" nillable="true" type="tns:PlayReadyLicenseTemplate" />
    <xs:simpleType name="PlayReadyLicenseType">
        <xs:restriction base="xs:string">
            <xs:enumeration value="Nonpersistent" />
            <xs:enumeration value="Persistent" />
        </xs:restriction>
    </xs:simpleType>
    <xs:element name="PlayReadyLicenseType" nillable="true" type="tns:PlayReadyLicenseType" />
    <xs:complexType name="PlayReadyPlayRight">
        <xs:sequence>
            <xs:element minOccurs="0" name="AgcAndColorStripeRestriction" nillable="true" type="tns:AgcAndColorStripeRestriction">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="AllowPassingVideoContentToUnknownOutput" type="tns:UnknownOutputPassingOption">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="AnalogVideoOpl" nillable="true" type="xs:int">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="CompressedDigitalAudioOpl" nillable="true" type="xs:int">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="CompressedDigitalVideoOpl" nillable="true" type="xs:int">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="DigitalVideoOnlyContentRestriction" type="xs:boolean">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="ExplicitAnalogTelevisionOutputRestriction" nillable="true" type="tns:ExplicitAnalogTelevisionRestriction">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="FirstPlayExpiration" nillable="true" type="ser:duration">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="ImageConstraintForAnalogComponentVideoRestriction" type="xs:boolean">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="ImageConstraintForAnalogComputerMonitorRestriction" type="xs:boolean">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="ScmsRestriction" nillable="true" type="tns:ScmsRestriction">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="UncompressedDigitalAudioOpl" nillable="true" type="xs:int">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
            <xs:element minOccurs="0" name="UncompressedDigitalVideoOpl" nillable="true" type="xs:int">
                <xs:annotation>
                    <xs:appinfo>
                        <DefaultValue EmitDefaultValue="false" xmlns="http://schemas.microsoft.com/2003/10/Serialization/" />
                    </xs:appinfo>
                </xs:annotation>
            </xs:element>
        </xs:sequence>
    </xs:complexType>
    <xs:element name="PlayReadyPlayRight" nillable="true" type="tns:PlayReadyPlayRight" />
    <xs:simpleType name="UnknownOutputPassingOption">
        <xs:restriction base="xs:string">
            <xs:enumeration value="NotAllowed" />
            <xs:enumeration value="Allowed" />
            <xs:enumeration value="AllowedWithVideoConstriction" />
        </xs:restriction>
    </xs:simpleType>
    <xs:element name="UnknownOutputPassingOption" nillable="true" type="tns:UnknownOutputPassingOption" />
    <xs:complexType name="ScmsRestriction">
        <xs:sequence>
            <xs:element minOccurs="0" name="ConfigurationData" type="xs:unsignedByte" />
        </xs:sequence>
    </xs:complexType>
    <xs:element name="ScmsRestriction" nillable="true" type="tns:ScmsRestriction" />
</xs:schema>
```

## <a name="next-steps"></a>Pasos siguientes

Consulte cómo [proteger con DRM](protect-with-drm.md).
