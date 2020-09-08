---
title: Administración de puntos de conexión de streaming con Azure Portal | Microsoft Docs
description: En este artículo se muestra cómo administrar los puntos de conexión de streaming mediante Azure Portal.
services: media-services
documentationcenter: ''
author: Juliako
writer: juliako
manager: femila
editor: ''
ms.assetid: bb1aca25-d23a-4520-8c45-44ef3ecd5371
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: b7ea61a1f09a9340661f333e452f70c5ad8cf877
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266653"
---
# <a name="manage-streaming-endpoints-with-the-azure-portal"></a>Administración de puntos de conexión de streaming con el Portal de Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

En este artículo se muestra cómo usar Azure Portal para administrar puntos de conexión de streaming. 

>[!NOTE]
>No olvide revisar el artículo de [Información general](media-services-streaming-endpoints-overview.md). 

Para obtener información sobre cómo escalar el punto de conexión de streaming, consulte [este](media-services-portal-scale-streaming-endpoints.md) artículo.

## <a name="start-managing-streaming-endpoints"></a>Inicio de la administración de puntos de conexión de streaming 

Para comenzar a administrar puntos de conexión de streaming de su cuenta, haga lo siguiente.

1. En [Azure Portal](https://portal.azure.com/), seleccione la cuenta de Azure Media Services.
2. En la hoja **Configuración**, haga clic en **Puntos de conexión de streaming**.
   
    ![extremo de streaming](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints1.png)

> [!NOTE]
> Solo se le cobrará cuando StreamingEndpoint esté en estado en ejecución.

## <a name="adddelete-a-streaming-endpoint"></a>Adición o eliminación de puntos de conexión de streaming

>[!NOTE]
>No se puede eliminar el punto de conexión de streaming predeterminado.

Para agregar/quitar el punto de conexión de streaming mediante el Portal de Azure, haga lo siguiente:

1. Para agregar un punto de conexión de streaming, haga clic en **+ Punto de conexión** en la parte superior de la página. 

    Conviene tener varios puntos de conexión de streaming si planea tener diferentes redes CDN o una red CDN y acceso directo.

2. Para eliminar un punto de conexión de streaming, haga clic en el botón **Eliminar** .      
3. Haga clic en el botón **Iniciar** para iniciar el punto de conexión de streaming.
   
    ![extremo de streaming](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints2.png)


## <a name="configuring-the-streaming-endpoint"></a><a id="configure_streaming_endpoints"></a>Configuración del extremo de streaming
Punto de conexión de streaming le permite configurar las siguientes propiedades:

* Control de acceso
* Control de caché
* Directivas de acceso en varios sitios

Para obtener información detallada acerca de estas propiedades, consulte [StreamingEndpoint](/rest/api/media/operations/streamingendpoint).

>[!NOTE]
>Cuando CDN esté habilitado, no se podrá tener acceso IP. El acceso IP solo es aplicable cuando no tiene CDN.

Puede configurar el punto de conexión de streaming haciendo lo siguiente:

1. Seleccione el punto de conexión de streaming que desea configurar.
2. Haga clic en **Configuración**.

Aparecerá una breve descripción de los campos.

![extremo de streaming](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints4.png)

1. Directiva de caché máximo: se usa para configurar la duración de la caché de los recursos entregados a través de este punto de conexión de streaming. Si no se establece ningún valor, se utiliza el valor predeterminado. Los valores predeterminados también pueden definirse directamente en el almacenamiento de Azure. Si Azure CDN está habilitado para el punto de conexión de streaming, no debe establecer el valor de la directiva de caché en menos de 600 segundos.  
2. Direcciones IP permitidas: especifique las direcciones IP que podrán conectarse al punto de conexión de streaming publicado. Si no se especificaron direcciones IP, cualquier dirección IP se podrá conectar. Las direcciones IP se pueden especificar como una dirección IP única (por ejemplo, 10.0.0.1), un intervalo de IP que usa una dirección IP y una máscara de subred CIDR (por ejemplo, 10.0.0.1/22) o un intervalo de IP que usa una máscara de subred decimal con puntos; por ejemplo, 10.0.0.1(255.255.255.0).
3. Configuración de autenticación del encabezado de firma de Akamai: se utiliza para especificar cómo se configura la solicitud de autenticación de encabezado de firma desde servidores Akamai. La expiración está en formato UTC.

## <a name="scale-your-premium-streaming-endpoint"></a>Escalado del punto de conexión de streaming premium

Para obtener más información, consulte [este](media-services-portal-scale-streaming-endpoints.md) artículo.

## <a name="enable-azure-cdn-integration"></a><a id="enable_cdn"></a>Habilitación de la integración de Azure CDN

Cuando se crea una nueva cuenta, la integración de Azure CDN de punto de conexión de streaming está habilitada de forma predeterminada.

Si más adelante desea volver a habilitar o deshabilitar la red CDN, punto de conexión de streaming debe estar en estado **stopped** (detenido). Es posible que transcurran hasta dos horas hasta que la integración de Azure CDN se habilite y los cambios se activen en todos los POP de la red CDN. Sin embargo, puede iniciar el punto de conexión de streaming y transmitir sin interrupciones desde ahí y, una vez que la integración esté completa, la transmisión se efectúa desde la red CDN. Durante el período de aprovisionamiento, el punto de conexión de streaming estará en estado **starting** (iniciando) y es posible que note una reducción en el rendimiento.

La integración de la red CDN está habilitada en todos los centros de datos de Azure excepto las regiones de China y el Gobierno Federal.

Una vez habilitada, la configuración de **Access Control**, del **nombre de host personalizado y de la **autenticación de firma de Akamai** se deshabilita.
 
> [!IMPORTANT]
> La integración de Azure Media Services con Azure CDN se implementa en **Azure CDN de Verizon** para puntos de conexión de streaming estándar. Los puntos de conexión de streaming premium pueden configurarse con todos los **proveedores y planes de tarifa de Azure CDN**. Para obtener más información sobre las características de Azure CDN, consulte la [información general de la red CDN](../../cdn/cdn-overview.md).
 
### <a name="additional-considerations"></a>Consideraciones adicionales

* Cuando la red CDN está habilitada para un extremo de streaming, los clientes no pueden solicitar contenido directamente desde el origen. Si necesita poder probar el contenido con o sin red CDN, puede crear otro punto de conexión de streaming que no tenga habilitada la red CDN.
* El nombre de host del extremo de streaming permanece igual después de habilitar la red CDN. No es necesario realizar ningún cambio en el flujo de trabajo de Servicios multimedia una vez habilitada la red CDN. Por ejemplo, si el nombre de host del extremo de streaming es strasbourg.streaming.mediaservices.windows.net, después de habilitar la red CDN se usa exactamente el mismo nombre de host.
* Para los nuevos puntos de conexión de streaming, puede habilitar la red CDN creando un nuevo punto de conexión; para los puntos de conexión de streaming existentes, deberá detener primero el punto de conexión y, después, habilitar/deshabilitar la red CDN.
* El punto de conexión de streaming estándar solo se puede configurar mediante el **proveedor de red CDN estándar de Verizon** con el portal de Azure clásico. Sin embargo, puede habilitar otros proveedores de Azure CDN mediante las API REST.

## <a name="configure-cdn-profile"></a>Configuración del perfil de la red CDN

Puede configurar el perfil de la red CDN con el botón **Administrar red de entrega de contenido** de la parte superior.

![extremo de streaming](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints6.png)

## <a name="next-steps"></a>Pasos siguientes
Consulte las rutas de aprendizaje de Media Services.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
