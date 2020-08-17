---
title: Puntos de conexión de streaming (origen)
titleSuffix: Azure Media Services
description: Obtenga información sobre los puntos de conexión de streaming (origen), un servicio dinámico de empaquetado y streaming que entrega contenido directamente a una aplicación de reproducción cliente o a una instancia de Content Delivery Network (CDN).
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 02/13/2020
ms.author: juliako
ms.openlocfilehash: 6d725ed8a69e2dfed6f5197db731f4adac57e2e2
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87446219"
---
# <a name="streaming-endpoints-origin-in-azure-media-services"></a>Puntos de conexión de streaming (origen) en Azure Media Services

En Microsoft Azure Media Services, un [punto de conexión de streaming](/rest/api/media/streamingendpoints) representa un servicio dinámico (Just-In-Time) de empaquetado y origen que puede entregar directamente el contenido en directo y a petición a una aplicación de reproducción cliente, mediante uno de los protocolos de streaming multimedia habituales (HLS o DASH). Además, el **punto de conexión de streaming** proporciona cifrado dinámico (Just-In-Time) para los sistemas de DRM líderes del sector. 

Cuando se crea una cuenta de Media Services, se genera automáticamente un punto de conexión de streaming **predeterminado** en estado detenido. No se puede eliminar el punto de conexión de streaming **predeterminado**. Es posible crear puntos de conexión de streaming adicionales en la cuenta (consulte [Cuotas y límites](limits-quotas-constraints.md)).

> [!NOTE]
> Para iniciar la transmisión de vídeos, es preciso que inicie el **punto de conexión de streaming** desde el que desee transmitir en secuencias el vídeo.
>
> Solo se le cobrará cuando el punto de conexión de streaming esté en estado de ejecución.

Asegúrese de revisar también el tema [Empaquetado dinámico](dynamic-packaging-overview.md). 

## <a name="naming-convention"></a>Convención de nomenclatura

El formato de nombre de host de la dirección URL de streaming es: `{servicename}-{accountname}-{regionname}.streaming.media.azure.net`, donde `servicename` = el nombre del punto de conexión de streaming o el nombre del evento en directo.

Al usar el punto de conexión de streaming predeterminado, se omite `servicename`, por lo que la dirección URL es: `{accountname}-{regionname}.streaming.azure.net`.

### <a name="limitations"></a>Limitaciones

* El nombre del punto de conexión de streaming tiene un valor máximo de 24 caracteres.
* El nombre debe seguir este patrón [de expresión regular](/dotnet/standard/base-types/regular-expression-language-quick-reference): `^[a-zA-Z0-9]+(-*[a-zA-Z0-9])*$`.

## <a name="types"></a>Tipos

Existen dos tipos de **puntos de conexión de streaming**: **Estándar** (versión preliminar) y **Premium**. El tipo se define por el número de unidades de escalado (`scaleUnits`) que se asignan para el punto de conexión de streaming.

El límite máximo de unidades de streaming suele ser diez. Póngase en contacto con nosotros [aquí](https://azure.microsoft.com/support/create-ticket/) para aumentar el límite de su cuenta.

En la tabla se describen los tipos:

|Tipo|Unidades de escalado|Descripción|
|--------|--------|--------|  
|**Estándar**|0|El punto de conexión de streaming predeterminado es de tipo **Estándar** y se puede cambiar al tipo Premium mediante el ajuste de `scaleUnits`.|
|**Premium**|>0|Los puntos de conexión de streaming **Premium** son adecuados para cargas de trabajo avanzadas y proporcionan una capacidad de ancho de banda dedicada y escalable. Para pasar a un tipo **Premium** se ajusta la propiedad `scaleUnits` (unidades de streaming). La propiedad `scaleUnits` proporciona capacidad de salida dedicada que puede adquirirse en incrementos de 200 Mbps. Cuando se usa el tipo **Premium**, cada unidad habilitada proporciona capacidad de ancho de banda adicional a la aplicación. |

> [!NOTE]
> Para los clientes que quieren entregar contenido a grandes audiencias de Internet, se recomienda habilitar CDN en el punto de conexión de streaming.

Para obtener información del contrato de nivel de servicio, consulte [Precios y SLA](https://azure.microsoft.com/pricing/details/media-services/).

## <a name="comparing-streaming-types"></a>Comparación de tipos de streaming

Característica|Estándar|Premium
---|---|---
Throughput |Hasta 600 Mbps y puede proporcionar un rendimiento eficaz mucho mayor cuando se usa una red CDN.|200 Mbps por unidad de streaming. Puede proporcionar un rendimiento eficaz mucho mayor cuando se usa una red CDN.
CDN|Azure CDN, CDN de terceros o ninguna CDN.|Azure CDN, CDN de terceros o ninguna CDN.
La facturación se prorratea| Diario|Diario
Cifrado dinámico|Sí|Sí
Empaquetado dinámico|Sí|Sí
Escala|Se amplía automáticamente hasta el rendimiento objetivo.|Unidades de streaming adicionales
Filtrado de direcciones IP/G20/host personalizado <sup>1</sup>|Sí|Sí
Descarga progresiva|Sí|Sí
Uso recomendado |Se recomienda para la gran mayoría de escenarios de streaming.|Uso profesional.

<sup>1</sup> Solo se usa directamente en el punto de conexión de streaming cuando la red CDN no está habilitada en el punto de conexión.<br/>

## <a name="streaming-endpoint-properties"></a>Propiedades del punto de conexión de streaming

En esta sección se proporcionan detalles sobre algunas de las propiedades del punto de conexión de streaming. Para obtener ejemplos de cómo crear un nuevo punto de conexión de streaming y las descripciones de todas las propiedades, vea [Streaming Endpoint](/rest/api/media/streamingendpoints/create) (Punto de conexión de streaming).

- `accessControl`: Se usa para configurar las siguientes opciones de seguridad para este punto de conexión de streaming: claves de autenticación de encabezado de firma de Akamai y direcciones IP que puedan conectarse a este punto de conexión. Esta propiedad solo se puede establecer cuando la propiedad `cdnEnabled` se establece en false.

- `cdnEnabled`: indica si la integración de la red Azure CDN para este punto de conexión de streaming está habilitada (está deshabilitada de forma predeterminada). Si establece la propiedad `cdnEnabled` en true, se deshabilitarán las siguientes configuraciones: `customHostNames` y `accessControl`.

    No todos los centros de datos admiten la integración de Azure CDN. Para comprobar si su centro de datos tiene disponible la integración de la red Azure CDN, realice los pasos siguientes:

  - Intente establecer el valor de `cdnEnabled` en true.
  - Compruebe el resultado devuelto de `HTTP Error Code 412` (error de condición previa) con el mensaje de que la propiedad CdnEnabled del punto de conexión de streaming no se puede establecer en true ya que la funcionalidad de la red CDN no está disponible en la región actual.

    Si recibe este error, el centro de datos no la admite. Pruebe otro centro de datos.

- `cdnProfile`: Cuando la propiedad `cdnEnabled` está establecida en true, también puede pasar valores `cdnProfile`. El valor `cdnProfile` es el nombre del perfil CDN donde se creará el punto de conexión CDN. Puede proporcionar un cdnProfile existente o usar uno nuevo. Si el valor es NULL y `cdnEnabled` es true, se utiliza el valor predeterminado "AzureMediaStreamingPlatformCdnProfile". Si el valor `cdnProfile` proporcionado ya existe, se crea un punto de conexión debajo de él. Si el perfil no existe, se crea un nuevo perfil automáticamente.
- `cdnProvider`: Cuando la red CDN está habilitada, también se pueden pasar valores `cdnProvider`. El valor `cdnProvider` controla qué proveedor se utilizará. Actualmente, se admiten tres valores: "StandardVerizon", "PremiumVerizon" y "StandardAkamai". Si no se proporciona ningún valor y la propiedad `cdnEnabled` es true, se utiliza "StandardVerizon" (es decir, el valor predeterminado).
- `crossSiteAccessPolicies`: Se usa para especificar las directivas de acceso entre sitios para varios clientes. Para obtener más información, consulte [Cross-domain policy file specification](https://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html) (Especificación del archivo de directivas entre dominios) y [Making a Service Available Across Domain Boundaries](/previous-versions/azure/azure-services/gg185950(v=azure.100)) (Hacer que un servicio esté disponible a través de los límites del dominio). La configuración se aplica solo a Smooth Streaming.
- `customHostNames`: se usa para configurar un punto de conexión de streaming para que acepte el tráfico dirigido a un nombre de host personalizado. Esta propiedad es válida para los puntos de conexión de streaming Estándar y Premium y se puede establecer cuando `cdnEnabled`: false.

    Media Services debe confirmar la propiedad del nombre de dominio. Media Services comprueba la propiedad del nombre de dominio mediante la solicitud de un registro `CName` que contenga el identificador de cuenta de Media Services como un componente para agregarlo al dominio en uso. Por ejemplo, para que "sports.contoso.com" se use como nombre de host personalizado para el punto de conexión de streaming, se debe configurar un registro para `<accountId>.contoso.com` que apunte a uno de los nombres de host de comprobación de Media Services. El nombre de host de comprobación está formado por verifydns.\<mediaservices-dns-zone>.

    Las siguientes son las zonas DNS esperadas que se van a usar en el registro de comprobación para diferentes regiones de Azure.
  
  - Norteamérica, Europa, Singapur, RAE de Hong Kong, Japón:

    - `media.azure.net`
    - `verifydns.media.azure.net`

  - China:

    - `mediaservices.chinacloudapi.cn`
    - `verifydns.mediaservices.chinacloudapi.cn`

    Por ejemplo, un registro `CName` que asigna "945a4c4e-28ea-45cd-8ccb-a519f6b700ad.contoso.com" a "verifydns.media.azure.net" prueba que el identificador de Media Services 945a4c4e-28ea-45cd-8ccb-a519f6b700ad tiene la propiedad del dominio contoso.com, lo que permite usar cualquier nombre de contoso.com como nombre de host personalizado para un punto de conexión de streaming en esa cuenta. Para buscar el valor de id. de Media Services, vaya a [Azure Portal](https://portal.azure.com/) y seleccione su cuenta de Media Services. El **identificador de cuenta** aparece en la parte superior derecha de la página.

    Si se intenta establecer un nombre de host personalizado sin una comprobación adecuada del registro `CName`, se producirá un error de respuesta de DNS y se almacenará en caché durante algún tiempo. Una vez que haya un registro adecuado, podría tardar un tiempo hasta que se vuelva a validar la respuesta almacenada en caché. En función del proveedor de DNS para el dominio personalizado, puede tardar desde unos minutos a una hora en volver a validar el registro.

    Además del registro `CName` que asigna `<accountId>.<parent domain>` a `verifydns.<mediaservices-dns-zone>`, debe crear otro registro `CName` que asigne el nombre de host personalizado (por ejemplo, `sports.contoso.com`) al nombre de host del punto de conexión de streaming de Media Services (por ejemplo, `amstest-usea.streaming.media.azure.net`).

    > [!NOTE]
    > Los puntos de conexión de streaming ubicados en el mismo centro de datos no pueden compartir el mismo nombre de host personalizado.

    Actualmente, Media Services no admite TLS con dominios personalizados.

- `maxCacheAge`: invalida el encabezado max-age de control de caché HTTP predeterminado establecido por el punto de conexión de streaming en fragmentos multimedia y manifiestos a petición. El valor se establece en segundos.
- `resourceState` -

    - Detenido: el estado inicial de un punto de conexión de streaming después de su creación.
    - Iniciando: está realizando la transición al estado En ejecución.
    - En ejecución: capaz de hacer streaming del contenido a clientes.
    - Escalando: las unidades de escalado se están aumentando o reduciendo.
    - Deteniendo: está realizando la transición al estado Detenido.
    - Eliminando: se está eliminando.

- `scaleUnits`: proporciona capacidad de salida dedicada que se puede adquirir en incrementos de 200 Mbps. Si tiene que pasar a un tipo **premium**, ajuste la propiedad `scaleUnits`.

## <a name="why-use-multiple-streaming-endpoints"></a>¿Por qué usar varios puntos de conexión de streaming?

Un punto de conexión de streaming único puede transmitir vídeos en directo y bajo demanda, y la mayoría de los clientes solo usan un punto de conexión de streaming. En esta sección se proporcionan algunos ejemplos de por qué es posible que tenga que usar varios puntos de conexión de streaming.

* Cada unidad reservada permite 200 Mbps de ancho de banda. Si necesita más de 2000 Mbps (2 Gbps) de ancho de banda, puede usar el segundo punto de conexión de streaming y el equilibrio de carga para proporcionar ancho de banda adicional.

    Sin embargo, la red CDN es la mejor manera de alcanzar la escalabilidad horizontal para el contenido de streaming, pero si va a entregar tanto contenido que la red CDN extrae más de 2 Gbps, puede agregar puntos de conexión de streaming adicionales (orígenes). En este caso, deberá entregar las direcciones URL de contenido que estén equilibradas entre los dos puntos de conexión de streaming. Este enfoque proporciona un mejor almacenamiento en caché que intentar enviar solicitudes a cada origen de manera aleatoria (por ejemplo, a través de un administrador de tráfico). 
    
    > [!TIP]
    > Por lo general, si la red CDN extrae más de 2 Gbps, es posible que algo esté mal configurado (por ejemplo, sin blindaje de origen).
    
* Equilibrio de carga entre distintos proveedores de CDN. Por ejemplo, puede configurar el punto de conexión de streaming predeterminado para usar la red CDN de Verizon y crear un segundo para usar Akamai. Luego, agregue un equilibrio de carga entre ambos para lograr el equilibrio de múltiples CDN. 

    Sin embargo, el cliente a menudo realiza el equilibrio de carga entre varios proveedores de CDN mediante un origen único.
* Contenido mixto de streaming: en directo y vídeo bajo demanda. 

    Los patrones de acceso para contenido en vivo y bajo demanda son muy diferentes. El contenido en directo tiende a tener una gran cantidad de demanda del mismo contenido a la vez. El contenido de vídeo bajo demanda (por ejemplo, contenido de archivo de cola larga) se usa poco en el mismo contenido. Por lo tanto, el almacenamiento en caché funciona muy bien en el contenido en directo, pero no tanto en el contenido de cola larga.

    Considere un escenario en el que los clientes ven principalmente el contenido en directo pero que, en ocasiones, ven el contenido bajo demanda y se proporciona desde el mismo punto de conexión de streaming. El uso bajo de contenido bajo demanda ocuparía espacio en caché que se guardaría mejor para el contenido en directo. En este escenario, se recomienda proporcionar el contenido en directo de un punto de conexión de streaming y el contenido de cola larga desde otro punto de conexión de streaming. Esto mejorará el rendimiento del contenido del evento en directo.
    
## <a name="scaling-streaming-with-cdn"></a>Escalado de streaming con CDN

Vea los artículos siguientes:

- [Información general de la red CDN](../../cdn/cdn-overview.md)
- [Escalado de streaming con CDN](scale-streaming-cdn.md)

## <a name="ask-questions-and--get-updates"></a>Formulación de preguntas y obtención de actualizaciones

Consulte el artículo [Comunidad de Azure Media Services](media-services-community.md) para ver diferentes formas de formular preguntas, enviar comentarios y obtener actualizaciones de Media Services.

## <a name="see-also"></a>Consulte también

[Empaquetado dinámico](dynamic-packaging-overview.md)

## <a name="next-steps"></a>Pasos siguientes

[Administración de puntos de conexión de streaming](manage-streaming-endpoints-howto.md)
