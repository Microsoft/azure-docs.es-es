---
title: Introducción a Azure Media Services | Microsoft Docs
description: Microsoft Azure Media Services es una plataforma extensible basada en la nube que permite a los desarrolladores crear aplicaciones de administración y entrega de contenido multimedia escalables. En este artículo se proporciona información general sobre Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/19/2019
ms.author: juliako
ms.openlocfilehash: 0d896b0ce23116f1b87718f492d407ac094b9e6a
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89261162"
---
# <a name="azure-media-services-overview"></a>Introducción a Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector" title1="Seleccione la versión de Media Services que se está usando:"]
> * [Versión 3](../latest/media-services-overview.md)
> * [Versión 2](media-services-overview.md)

> [!NOTE]
> No se van agregar características a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Microsoft Azure Media Services (AMS) es una plataforma extensible basada en la nube que permite a los desarrolladores crear aplicaciones de administración y entrega de contenido multimedia escalables. Media Services se basa en las API de REST, que permiten cargar, almacenar, codificar y empaquetar de forma segura contenido de audio o vídeo para la entrega bajo demanda y de streaming en vivo a varios clientes (por ejemplo, televisión, PC y dispositivos móviles).

Puede crear flujos de trabajo de un extremo a otro usando solamente Media Services. También puede usar componentes de terceros para algunas partes del flujo de trabajo. Por ejemplo, codifique mediante un codificador de terceros. A continuación, cargue, proteja, empaquete y entregue con Media Services. Puede elegir entre transmitir en secuencias el contenido en directo o entregar el contenido a petición. 


## <a name="compliance-privacy-and-security"></a>Cumplimiento, privacidad y seguridad

Como recordatorio importante, debe cumplir todas las leyes aplicables en el uso de Azure Media Services, y no puede usar Media Services ni ningún otro servicio de Azure de forma que infrinja los derechos de otros o que puedan ser perjudiciales para otros.

Antes de cargar un vídeo o una imagen en Media Services, debe tener todos los derechos adecuados para usar ese vídeo o imagen, incluidos (cuando lo exija la ley) todos los consentimientos necesarios de las personas (si las hay) que aparecen en ellos, para el uso, el procesamiento y el almacenamiento de sus datos en Media Services y Azure. Algunas jurisdicciones pueden imponer requisitos legales especiales sobre la recopilación, el procesamiento en línea y el almacenamiento de determinadas categorías de datos, como los datos biométricos. Antes de usar Media Services y Azure para el procesamiento y el almacenamiento de datos sujetos a requisitos legales especiales, debe garantizar el cumplimiento de tales requisitos legales que puedan aplicarse en su caso.

Para información sobre el cumplimiento, la privacidad y la seguridad en Media Services, visite el [Centro de confianza de Microsoft](https://www.microsoft.com/trust-center/?rtc=1). En lo que respecta a las obligaciones de privacidad y a las prácticas de retención y tratamiento de los datos de Microsoft, revise la [declaración de privacidad](https://privacy.microsoft.com/PrivacyStatement), los [términos de Online Services](https://www.microsoft.com/licensing/product-licensing/products?rtc=1) ("OST") y el [anexo de procesamiento de datos](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=67) ("DPA") de Microsoft. Con el uso de Media Services, acepta estar vinculado por el OST, el DPA y la declaración de privacidad.
 
## <a name="prerequisites"></a>Requisitos previos

Para empezar a usar Azure Media Services, debe tener lo siguiente:

* Una cuenta de Azure. En caso de no tener cuenta, puede crear una de evaluación gratuita en tan solo unos minutos. Para obtener más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com).
* Una cuenta de Azure Media Services. Para obtener más información, consulte [Creación de una cuenta](media-services-portal-create-account.md).
* (Opcional) Configurar el entorno de desarrollo. Elija .NET o API de REST para el entorno de desarrollo. Para obtener más información, consulte [Configuración del entorno](media-services-dotnet-how-to-use.md).

    Además, aprenda a [conectarse mediante programación a la API de AMS](media-services-use-aad-auth-to-access-ams-api.md).
* Un punto de conexión de streaming estándar o premium en estado iniciado.  Para más información, consulte [Administración de puntos de conexión de streaming](media-services-portal-manage-streaming-endpoints.md).

## <a name="sdks-and-tools"></a>SDK y herramientas

Para compilar soluciones de Media Services, puede usar:

* [API de REST de Media Services](/rest/api/media/operations/azure-media-services-rest-api-reference)
* Uno de los SDK de cliente disponibles:
    * SDK de Azure Media Services para .NET
    
        * [Paquete NuGet](https://www.nuget.org/packages/windowsazure.mediaservices/)
        * [Código fuente de GitHub](https://github.com/Azure/azure-sdk-for-media-services)
    * [SDK de Azure para Java](https://github.com/Azure/azure-sdk-for-java)
    * [SDK de Azure para PHP](https://github.com/Azure/azure-sdk-for-php)
    * [Azure Media Services para Node.js](https://github.com/michelle-becker/node-ams-sdk/blob/master/lib/request.js) (es decir, una versión de un SDK de Node.js que no sea de Microsoft. Su mantenimiento corre a cargo de una comunidad y actualmente no tiene una cobertura del 100 % de las API de AMS).
* Herramientas existentes:
    * [Azure Portal](https://portal.azure.com/)
    * [Azure-Media-Services-Explorer](https://github.com/Azure/Azure-Media-Services-Explorer) (El Explorador de Azure Media Services (AMSE) es una aplicación Winforms/C# para Windows)

> [!NOTE]
> Para obtener la versión más reciente del SDK de Java y empezar a realizar desarrollos con Java, consulte [Introducción al SDK de cliente de Java para Azure Media Services](./media-services-java-how-to-use.md). <br/>
> Para descargar el SDK de PHP más reciente para Media Services, busque la versión 0.5.7 del paquete de Microsoft/WindowAzure en el [repositorio Packagist](https://packagist.org/packages/microsoft/windowsazure#v0.5.7).  

## <a name="code-samples"></a>Ejemplos de código

Busque varios ejemplos de código en la galería de **ejemplos de código de Azure**: [Ejemplos de código de Azure Media Services](https://azure.microsoft.com/resources/samples/?service=media-services&sort=0).

## <a name="concepts"></a>Conceptos

Para conocer los conceptos de Azure Media Services, consulte [Conceptos](media-services-concepts.md).

## <a name="supported-scenarios-and-availability-of-media-services-across-data-centers"></a>Escenarios admitidos y disponibilidad de Media Services en centros de datos

Para más información, consulte [Scenarios and availability of Media Services features across datacenters](scenarios-and-availability.md) (Escenarios y disponibilidad de Media Services en centros de datos).

## <a name="service-level-agreement-sla"></a>Contrato de nivel de servicio (SLA)

Para obtener más información, consulte el [Contrato de nivel de servicio (SLA) de Microsoft Azure](https://azure.microsoft.com/support/legal/sla/).

Para obtener información acerca de la disponibilidad en los centros de datos, consulte la sección [Disponibilidad](scenarios-and-availability.md#availability).

## <a name="support"></a>Soporte técnico

[Soporte técnico de Azure](https://azure.microsoft.com/support/options/) proporciona opciones de soporte técnico para Azure, incluido Media Services.

## <a name="provide-feedback"></a>Envío de comentarios

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
