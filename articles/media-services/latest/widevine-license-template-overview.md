---
title: Introducción a Azure Media Services v3 con plantilla de licencia de Widevine
description: En este tema se proporciona información general sobre una plantilla de licencia de Widevine que se usó para configurar las licencias de Widevine.
author: IngridAtMicrosoft
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/07/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 8ea0eab6d420a4b74de93a1d35e178d6c57c2ad9
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89298936"
---
# <a name="media-services-v3-with-widevine-license-template-overview"></a>Introducción a Media Services v3 con plantilla de licencia de Widevine

Azure Media Services le permite cifrar el contenido HLS con **Widevine de Google**. Asimismo, Media Services también proporciona un servicio para entregar licencias de Widevine. Puede usar las API de Azure Media Services para configurar y solicitar licencias de Widevine. Cuando el reproductor intenta reproducir contenido protegido de Widevine, se envía una solicitud al servicio de entrega de licencias para obtener una licencia. Si el servicio de licencias aprueba la solicitud, el servicio emite la licencia. A continuación, se envía al cliente y se usa para descifrar y reproducir el contenido especificado.

Una solicitud de licencia de Widevine tiene el formato de un mensaje JSON.  

>[!NOTE]
> Puede crear un mensaje vacío y sin valores usando simplemente "{}". A continuación, se crea una plantilla de licencia con los valores predeterminados. La configuración predeterminada funciona para la mayoría de los casos. Los escenarios de entrega de licencia basados en Microsoft deben utilizar siempre los valores predeterminados. Si tiene que establecer los valores de "provider" y "content_id", el proveedor debe coincidir con las credenciales de Widevine.

```json
{  
    "payload":"<license challenge>",
    "content_id": "<content id>"
    "provider": "<provider>"
    "allowed_track_types":"<types>",
    "content_key_specs":[  
        {  
            "track_type":"<track type 1>"
        },
        {  
            "track_type":"<track type 2>"
        },
        …
    ],
    "policy_overrides":{  
        "can_play":<can play>,
        "can persist":<can persist>,
        "can_renew":<can renew>,
        "rental_duration_seconds":<rental duration>,
        "playback_duration_seconds":<playback duration>,
        "license_duration_seconds":<license duration>,
        "renewal_recovery_duration_seconds":<renewal recovery duration>,
        "renewal_server_url":"<renewal server url>",
        "renewal_delay_seconds":<renewal delay>,
        "renewal_retry_interval_seconds":<renewal retry interval>,
        "renew_with_usage":<renew with usage>
    }
}
```

## <a name="json-message"></a>Mensaje JSON

| Nombre | Value | Descripción |
| --- | --- | --- |
| payload |Cadena codificada en Base64 |La solicitud de licencia enviada por un cliente. |
| content_id |Cadena codificada en Base64 |Identificador utilizado para derivar el id. de la clave y la clave de contenido de cada elemento content_key_specs.track_type. |
| provider |string |Utilizado para buscar directivas y claves de contenido. En el caso de que se use la entrega de claves de Microsoft para entregar licencias de Widevine, este parámetro se omite. |
| policy_name |string |Nombre de una directiva previamente registrada. Opcional. |
| allowed_track_types |enum |SD_ONLY o SD_HD. Controla qué claves de contenido deben incluirse en una licencia. |
| content_key_specs |Matriz de estructuras JSON; consulte "Especificaciones de clave de contenido".  |Un control más preciso sobre qué claves de contenido se devolverán. Para obtener más información, consulte la sección "Especificaciones de clave de contenido". Solo se puede especificar uno de los valores de allowed_track_types y content_key_specs. |
| use_policy_overrides_exclusively |Valor booleano (true o false) |Utilice los atributos de directiva que especifica policy_overrides y omita todas las directivas almacenadas previamente. |
| policy_overrides |Estructura JSON; consulte "Invalidaciones de directivas". |Configuración de directiva para esta licencia.  En caso de que este activo tenga una directiva definida previamente, se usarán estos valores especificados. |
| session_init |Estructura JSON; consulte "Inicialización de la sesión". |Los datos opcionales se pasan a la licencia. |
| parse_only |Valor booleano (true o false) |Se analiza la solicitud de licencia, pero no se emite ninguna licencia. Sin embargo, los valores de la solicitud de licencia se devuelven en la respuesta. |

## <a name="content-key-specs"></a>Especificaciones de clave de contenido
Si existe una directiva anterior, no es necesario especificar ninguno de los valores en las especificaciones de clave de contenido. La directiva anterior que estaba asociada con este contenido se usa para determinar la protección de salida como, por ejemplo, Protección de contenido digital de ancho de banda alto (HDCP) y Copy General Management System (CGMS). Si no hay una directiva anterior registrada con el servidor de licencias de Widevine, el proveedor de contenido puede insertar los valores en la solicitud de licencia.   

Cada valor de content_key_specs debe especificarse en todas las pistas, independientemente de la opción use_policy_overrides_exclusively. 

| Nombre | Value | Descripción |
| --- | --- | --- |
| content_key_specs. track_type |string |Un nombre de tipo de pista. Si se especifica content_key_specs en la solicitud de licencia, asegúrese de especificar todos los tipos de pista explícitamente. Si no lo hace, se producirán errores en la reproducción transcurridos 10 segundos. |
| content_key_specs  <br/> security_level |uint32 |Define los requisitos de solidez del cliente para la reproducción. <br/> - Se requiere criptografía de caja blanca basada en software. <br/> - Se requiere criptografía de software y un descodificador de ofuscación. <br/> - Las operaciones de criptografía y material clave deben realizarse en un entorno de ejecución de confianza con respaldo del hardware. <br/> - La criptografía y la descodificación del contenido deben realizarse dentro de un entorno de ejecución de confianza con respaldo del hardware.  <br/> - La criptografía, la descodificación y todo el tratamiento de los medios (comprimidos y descomprimidos) deben administrarse dentro de un entorno de ejecución de confianza con respaldo del hardware. |
| content_key_specs <br/> required_output_protection.hdc |cadena, una de HDCP_NONE, HDCP_V1, HDCP_V2 |Indica si se requiere HDCP. |
| content_key_specs <br/>key |Base64-<br/>cadena codificada |Clave de contenido que se utilizará para esta pista. Si se especifica, se requiere track_type o key_id. Esta opción permite que el proveedor de contenido inserte la clave de contenido para esta pista en lugar de permitir que el servidor de licencias de Widevine genere o busque una clave. |
| content_key_specs.key_id |Binario de cadena codificada en Base64, 16 bytes |Identificador único para la clave. |

## <a name="policy-overrides"></a>Invalidaciones de directivas
| Nombre | Value | Descripción |
| --- | --- | --- |
| policy_overrides&#46;can_play |Valor booleano (true o false) |Indica que la reproducción del contenido está permitida. El valor predeterminado es False. |
| policy_overrides&#46;can_persist |Valor booleano (true o false) |Indica que la licencia puede conservarse en el almacenamiento no volátil para su uso sin conexión. El valor predeterminado es False. |
| policy_overrides&#46;can_renew |Valor booleano (true o false) |Indica que se permite la renovación de la presente licencia. Si es true, se puede ampliar la duración de la licencia mediante latido. El valor predeterminado es False. |
| policy_overrides&#46;license_duration_seconds |int64 |Indica el período de tiempo para esta licencia específica. Un valor de 0 indica que no hay ningún límite para la duración. El valor predeterminado es 0. |
| policy_overrides&#46;rental_duration_seconds |int64 |Indica el período de tiempo en el que se permite la reproducción. Un valor de 0 indica que no hay ningún límite para la duración. El valor predeterminado es 0. |
| policy_overrides&#46;playback_duration_seconds |int64 |El período de tiempo de visualización una vez que la reproducción comienza en el plazo de duración de la licencia. Un valor de 0 indica que no hay ningún límite para la duración. El valor predeterminado es 0. |
| policy_overrides&#46;renewal_server_url |string |Todas las solicitudes de latido (renovación) de esta licencia se dirigirán a la dirección URL especificada. Este campo solo se usa si can_renew es "true". |
| policy_overrides&#46;renewal_delay_seconds |int64 |El número de segundos después de license_start_time, antes de intentar la renovación por primera vez. Este campo solo se usa si can_renew es "true". El valor predeterminado es 0. |
| policy_overrides&#46;renewal_retry_interval_seconds |int64 |Especifica el plazo en segundos entre las posteriores solicitudes de renovación de licencia, en caso de error. Este campo solo se usa si can_renew es "true". |
| policy_overrides&#46;renewal_recovery_duration_seconds |int64 |El período de tiempo en el que la reproducción puede continuar mientras se intenta la renovación, aunque no se realice correctamente debido a problemas de back-end con el servidor de licencias. Un valor de 0 indica que no hay ningún límite para la duración. Este campo solo se usa si can_renew es "true". |
| policy_overrides&#46;renew_with_usage |Valor booleano (true o false) |Indica que la licencia se enviará para renovación cuando se inicie el uso. Este campo solo se usa si can_renew es "true". |

## <a name="session-initialization"></a>Inicialización de la sesión
| Nombre | Value | Descripción |
| --- | --- | --- |
| provider_session_token |Cadena codificada en Base64 |Este token de sesión se pasa de nuevo en la licencia y existirá en renovaciones posteriores. El token de sesión no se conservará una vez agotadas las sesiones. |
| provider_client_token |Cadena codificada en Base64 |Token de cliente para devolver en la respuesta de licencia. Si la solicitud de licencia contiene un token de cliente, este valor se omite. El token del cliente se conservará una vez agotadas las sesiones de licencia. |
| override_provider_client_token |Valor booleano (true o false) |Si es false y la solicitud de licencia contiene un token de cliente, use el token de la solicitud incluso si se especificó un token de cliente en esta estructura. Si es true, utilice siempre el token especificado en esta estructura. |

## <a name="configure-your-widevine-license-with-net"></a>Configuración de la licencia de Widevine con .NET 

Media Services proporciona una clase que le permite configurar una licencia de Widevine. Para construir la licencia, pase el JSON a [WidevineTemplate](/dotnet/api/microsoft.azure.management.media.models.contentkeypolicywidevineconfiguration.widevinetemplate?view=azure-dotnet#Microsoft_Azure_Management_Media_Models_ContentKeyPolicyWidevineConfiguration_WidevineTemplate).

Para configurar la plantilla puede:

### <a name="directly-construct-a-json-string"></a>Crear directamente una cadena JSON.

Este método puede ser propenso a errores. Se recomienda utilizar otro método, tal como se describe en la sección [Definir las clases necesarias y serializarlas a JSON](#classes).

```csharp
ContentKeyPolicyWidevineConfiguration objContentKeyPolicyWidevineConfiguration = new ContentKeyPolicyWidevineConfiguration
{
    WidevineTemplate = @"{""allowed_track_types"":""SD_HD"",""content_key_specs"":[{""track_type"":""SD"",""security_level"":1,""required_output_protection"":{""hdcp"":""HDCP_V2""}}],""policy_overrides"":{""can_play"":true,""can_persist"":true,""can_renew"":false}}"
};
```

### <a name="define-needed-classes-and-serialize-to-json"></a><a id="classes"></a> Definir las clases necesarias y serializarlas a JSON

#### <a name="define-classes"></a>Definir las clases

En el ejemplo siguiente se muestra un ejemplo de definiciones de clases que se asignan al esquema JSON de Widevine. Puede crear instancias de las clases antes de serializarlas a la cadena JSON.  

```csharp
/// <summary>
/// Widevine PolicyOverrides class.
/// </summary>
public class PolicyOverrides
{
    /// <summary>
    /// Gets or sets a value indicating whether playback of the content is allowed. Default is false.
    /// </summary>
    [JsonProperty("can_play")]
    public bool CanPlay { get; set; }

    /// <summary>
    /// Gets or sets a value indicating whether the license might be persisted to nonvolatile storage for offline use. Default is false.
    /// </summary>
    [JsonProperty("can_persist")]
    public bool CanPersist { get; set; }

    /// <summary>
    /// Gets or sets a value indicating whether renewal of this license is allowed. If true, the duration of the license can be extended by heartbeat. Default is false.
    /// </summary>
    [JsonProperty("can_renew")]
    public bool CanRenew { get; set; }

    /// <summary>
    /// Gets or sets the time window while playback is permitted. A value of 0 indicates that there is no limit to the duration. Default is 0.
    /// </summary>
    [JsonProperty("rental_duration_seconds")]
    public int RentalDurationSeconds { get; set; }

    /// <summary>
    /// Gets or sets the viewing window of time after playback starts within the license duration. A value of 0 indicates that there is no limit to the duration. Default is 0.
    /// </summary>
    [JsonProperty("playback_duration_seconds")]
    public int PlaybackDurationSeconds { get; set; }

    /// <summary>
    /// Gets or sets the time window for this specific license. A value of 0 indicates that there is no limit to the duration. Default is 0.
    /// </summary>
    [JsonProperty("license_duration_seconds")]
    public int LicenseDurationSeconds { get; set; }
}

/// <summary>
/// Widevine ContentKeySpec class.
/// </summary>
public class ContentKeySpec
{
    /// <summary>
    /// Gets or sets track type.
    /// If content_key_specs is specified in the license request, make sure to specify all track types explicitly.
    /// Failure to do so results in failure to play back past 10 seconds.
    /// </summary>
    [JsonProperty("track_type")]
    public string TrackType { get; set; }

    /// <summary>
    /// Gets or sets client robustness requirements for playback.
    /// Software-based white-box cryptography is required.
    /// Software cryptography and an obfuscated decoder are required.
    /// The key material and cryptography operations must be performed within a hardware-backed trusted execution environment.
    /// The cryptography and decoding of content must be performed within a hardware-backed trusted execution environment.
    /// The cryptography, decoding, and all handling of the media (compressed and uncompressed) must be handled within a hardware-backed trusted execution environment.
    /// </summary>
    [JsonProperty("security_level")]
    public int SecurityLevel { get; set; }

    /// <summary>
    /// Gets or sets the OutputProtection.
    /// </summary>
    [JsonProperty("required_output_protection")]
    public OutputProtection RequiredOutputProtection { get; set; }
}

/// <summary>
/// OutputProtection Widevine class.
/// </summary>
public class OutputProtection
{
    /// <summary>
    /// Gets or sets HDCP protection.
    /// Supported values : HDCP_NONE, HDCP_V1, HDCP_V2
    /// </summary>
    [JsonProperty("hdcp")]
    public string HDCP { get; set; }

    /// <summary>
    /// Gets or sets CGMS.
    /// </summary>
    [JsonProperty("cgms_flags")]
    public string CgmsFlags { get; set; }
}

/// <summary>
/// Widevine template.
/// </summary>
public class WidevineTemplate
{
    /// <summary>
    /// Gets or sets the allowed track types.
    /// SD_ONLY or SD_HD.
    /// Controls which content keys are included in a license.
    /// </summary>
    [JsonProperty("allowed_track_types")]
    public string AllowedTrackTypes { get; set; }

    /// <summary>
    /// Gets or sets a finer-grained control on which content keys to return.
    /// For more information, see the section "Content key specs."
    /// Only one of the allowed_track_types and content_key_specs values can be specified.
    /// </summary>
    [JsonProperty("content_key_specs")]
    public ContentKeySpec[] ContentKeySpecs { get; set; }

    /// <summary>
    /// Gets or sets policy settings for the license.
    /// In the event this asset has a predefined policy, these specified values are used.
    /// </summary>
    [JsonProperty("policy_overrides")]
    public PolicyOverrides PolicyOverrides { get; set; }
}
```

#### <a name="configure-the-license"></a>Configuración de la licencia

Utilice las clases definidas en la sección anterior para crear el archivo JSON que se usa para configurar [WidevineTemplate](/dotnet/api/microsoft.azure.management.media.models.contentkeypolicywidevineconfiguration.widevinetemplate?view=azure-dotnet#Microsoft_Azure_Management_Media_Models_ContentKeyPolicyWidevineConfiguration_WidevineTemplate):

```csharp
private static ContentKeyPolicyWidevineConfiguration ConfigureWidevineLicenseTempate()
{
    WidevineTemplate template = new WidevineTemplate()
    {
        AllowedTrackTypes = "SD_HD",
        ContentKeySpecs = new ContentKeySpec[]
        {
            new ContentKeySpec()
            {
                TrackType = "SD",
                SecurityLevel = 1,
                RequiredOutputProtection = new OutputProtection()
                {
                    HDCP = "HDCP_V2"
                }
            }
        },
        PolicyOverrides = new PolicyOverrides()
        {
            CanPlay = true,
            CanPersist = true,
            CanRenew = false,
            RentalDurationSeconds = 2592000,
            PlaybackDurationSeconds = 10800,
            LicenseDurationSeconds = 604800,
        }
    };

    ContentKeyPolicyWidevineConfiguration objContentKeyPolicyWidevineConfiguration = new ContentKeyPolicyWidevineConfiguration
    {
        WidevineTemplate = Newtonsoft.Json.JsonConvert.SerializeObject(template)
    };
    return objContentKeyPolicyWidevineConfiguration;
}
```

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="next-steps"></a>Pasos siguientes

Consulte cómo [proteger con DRM](protect-with-drm.md).
