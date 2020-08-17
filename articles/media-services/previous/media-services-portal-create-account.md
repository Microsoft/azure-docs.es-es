---
title: Creación de una cuenta de Azure Media Services con Azure Portal | Microsoft Docs
description: Este tutorial le guiará por los pasos que hay que seguir para crear una cuenta de Azure Media Services con Azure Portal.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: c551e158-aad6-47b4-931e-b46260b3ee4c
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/01/2019
ms.author: juliako
ms.openlocfilehash: 68a33f2aa7420de21a606d4e38b11db2509226ed
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084575"
---
# <a name="create-a-media-services-account-using-the-azure-portal"></a>Creación de una cuenta de Media Services desde Azure Portal

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Azure Portal proporciona una forma de crear rápidamente una cuenta de Azure Media Services (AMS). Puede usar la cuenta para obtener acceso a Media Services que le permiten almacenar, cifrar, codificar, administrar y transmitir contenido multimedia en Azure. En el momento en el que crea una cuenta de Media Services, también crea una cuenta de almacenamiento asociada (o usa una ya existente). Si elimina una cuenta de Media Services, no se eliminarán los blobs de la cuenta de almacenamiento relacionada.

La cuenta de Media Services y todas las cuentas de almacenamiento asociadas deben estar en la misma suscripción de Azure. Se recomienda encarecidamente usar cuentas de almacenamiento que se encuentren en la misma ubicación que la cuenta de Media Services para evitar costos adicionales debidos a la latencia y a la salida de datos.

En este artículo se muestra cómo crear una cuenta de Media Services con Azure Portal.

> [!NOTE]
> Para obtener información acerca de la disponibilidad de las características de Azure Media Services en distintas regiones, consulte la sección [Availability of Media Services features across datacenters](scenarios-and-availability.md#availability) (Disponibilidad de las características de Media Services en los centros de datos).

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, deberá tener una cuenta de Azure. Para obtener más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/). 

## <a name="create-an-ams-account"></a>Creación de una cuenta de AMS

Los pasos de esta sección muestran cómo crear una cuenta de AMS.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Haga clic en **+ Crear un recurso** > **Multimedia** > **Media Services**.
3. En **CREAR CUENTA DE SERVICIOS MULTIMEDIA** especifique los valores obligatorios.

   1. En **Nombre de la cuenta**, especifique el nombre de la cuenta nueva de AMS. El nombre de la cuenta de Media Services debe estar compuesto totalmente de minúsculas o de números, sin espacios, y con una longitud de entre 3 y 24 caracteres.
   2. En Suscripción, seleccione entre las diferentes suscripciones de Azure a las que tiene acceso.
   3. En **Grupo de recursos**seleccione el recurso nuevo o uno ya existente.  Un grupo de recursos es una colección de recursos que comparten ciclos de vida, permisos y directivas. Obtenga más información [aquí](../../azure-resource-manager/management/overview.md#resource-groups).
   4. En **Ubicación**, seleccione la región geográfica que se usará para almacenar los registros de medios y de metadatos para la cuenta de Media Services. Esta región se utilizará para procesar y transmitir contenido multimedia. Solo las regiones de Media Services disponibles aparecen en la lista desplegable. 
   5. En **Cuenta de almacenamiento**, seleccione una cuenta de almacenamiento para proporcionar almacenamiento de blobs del contenido multimedia desde la cuenta de Media Services. Puede seleccionar una cuenta de almacenamiento existente de la misma región geográfica que la cuenta de Media Services o crearla. Se crea una nueva cuenta de almacenamiento en la misma región. Las reglas para los nombres de cuenta de almacenamiento son las mismas que para las cuentas de Media Services.
      
       Puede obtener más información acerca del almacenamiento [aquí](../../storage/common/storage-introduction.md).
   6. Seleccione **Anclar al panel** para ver el progreso de la implementación de la cuenta.
4. Haga clic en **Crear** en la parte inferior del formulario.
   
    Cuando la cuenta se crea correctamente, se carga la página de información general. En la tabla de puntos de conexión de streaming, la cuenta tendrá el punto de conexión de streaming predeterminado en el estado **Stopped** (Detenido). 

    >[!NOTE]
    >Cuando se crea la cuenta de AMS, se agrega un punto de conexión de streaming **predeterminado** a la cuenta en estado **Stopped** (Detenido). Para iniciar la transmisión del contenido y aprovechar el empaquetado dinámico y el cifrado dinámico, el punto de conexión de streaming desde el que va a transmitir el contenido debe estar en estado **Running** (En ejecución). 
   
## <a name="to-manage-your-ams-account"></a>Administración de una cuenta de AMS

Para administrar una cuenta de AMS (por ejemplo, conectarse a la API de AMS mediante programación, cargar vídeos, codificar recursos, configurar la protección de contenido o supervisar el progreso del trabajo) seleccione **Configuración** en el lado izquierdo del portal. En **Configuración**, vaya a una de las hojas disponibles (por ejemplo: **Acceso de API**, **Recursos**, **Trabajos**, **Content Protection**).

## <a name="next-steps"></a>Pasos siguientes

Ya puede cargar archivos en su cuenta de AMS. Para más información, consulte [Upload files into a Media Services account using the Azure portal](media-services-portal-upload-files.md)(Carga de archivos en una cuenta de Servicios multimedia desde el Portal de Azure).

Si planea acceder a la API de AMS mediante programación, consulte [Acceso a Azure Media Services API con la autenticación de Azure AD](media-services-use-aad-auth-to-access-ams-api.md).

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
