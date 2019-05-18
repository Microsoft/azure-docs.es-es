---
title: Creación de una cuenta de Video Indexer en Azure Portal
titlesuffix: Azure Media Services
description: En este artículo se muestra cómo crear una cuenta de Video Indexer en Azure Portal.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: juliako
ms.openlocfilehash: 33493f1bdff6071737aad4bfb8c7d0e5e22896db
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799842"
---
# <a name="create-a-video-indexer-account-connected-to-azure"></a>Creación de una cuenta de Video Indexer conectada a Azure

Al crear una cuenta de Video Indexer, puede elegir una cuenta de evaluación gratuita (donde obtendrá un número determinado de minutos gratuitos de indexación) o una opción de pago (donde no está limitado por la cuota). Con la versión de evaluación gratuita, Video Indexer proporciona hasta 600 horas de indexación gratuita a los usuarios de sitios web y hasta 2400 horas de indexación gratuita a los usuarios de API. Con la opción de pago, se crea una cuenta de Video Indexer que está conectada a su suscripción de Azure y a una cuenta de Azure Media Services. Se paga por minutos de indexación y por los cargos relacionados con la cuenta de Media Services. 

En este artículo se muestra cómo crear una cuenta de Video Indexer que está vinculada a su suscripción de Azure y a una cuenta de Azure Media Services. Se proporcionan los pasos para conectarse a Azure mediante el flujo automático (predeterminado). También se muestra cómo conectarse a Azure manualmente (avanzado).

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción de Azure.

    Si aún no tiene una suscripción de Azure, [regístrese para obtener una cuenta de evaluación gratuita de Azure](https://azure.microsoft.com/free/).

* Un dominio de Azure Active Directory (AD).

    Si no tiene un dominio de Azure AD, créelo con su suscripción de Azure. Para más información, consulte [Administración de nombres de dominio personalizados en Azure Active Directory](../../active-directory/users-groups-roles/domains-manage.md).

* Un usuario y un miembro del dominio de Azure AD. Usará a este miembro para conectar su cuenta de Video Indexer a Azure.

    Este usuario debe ser un usuario de Azure AD con una cuenta profesional o educativa, no una cuenta personal, como outlook.com, live.com o hotmail.com.

    ![todos los usuarios de AAD](./media/create-account/all-aad-users.png)

### <a name="additional-prerequisites-for-automatic-flow"></a>Requisitos previos adicionales para el flujo automático

Un usuario y un miembro del dominio de Azure AD. Usará a este miembro para conectar su cuenta de Video Indexer a Azure.

Este usuario debe ser miembro de la suscripción de Azure con un rol de **Propietario**, o con los roles de **Colaborador** y **Administrador de acceso de usuario**. Un usuario se puede agregar dos veces, con 2 roles. Una vez como Colaborador y otra como Administrador de acceso de usuario.

![control de acceso](./media/create-account/access-control-iam.png)

### <a name="additional-prerequisites-for-manual-flow"></a>Requisitos previos adicionales para el flujo manual

Registrar el proveedor de recursos EventGrid mediante Azure Portal.

En [Azure Portal](https://portal.azure.com/), vaya a **Suscripciones**->[suscripción]->**ResourceProviders**. 

Busque **Microsoft.Media** y **Microsoft.EventGrid**. Si no se encuentra en el estado "Registrado", haga clic en **Registrarse**. Tarda unos minutos en registrarse.

![EventGrid](./media/create-account/event-grid.png)

## <a name="connect-to-azure"></a>Conéctese a Azure

1. Vaya al sitio web de [Video Indexer](https://www.videoindexer.ai/) e inicie sesión.

2. Haga clic en el **crear nueva cuenta** botón:

    ![conectarse a Azure](./media/create-account/connect-to-azure.png)

3. Cuando aparezca la lista de suscripciones, seleccione la suscripción que desea usar.

    ![conectar Video Indexer a Azure](./media/create-account/connect-vi-to-azure-subscription.png)

4. Seleccione una región de Azure de las ubicaciones admitidas: Oeste de EE. UU. 2, Europa del Norte o Asia Oriental.
5. En **Cuenta de Azure Media Services**, elija una de estas opciones:

    * Para crear una cuenta de Media Services nueva, seleccione **Crear nuevo grupo de recursos**. Escriba un nombre para el grupo de recursos.

        Azure creará la nueva cuenta en su suscripción, incluida una nueva cuenta de Azure Storage. La nueva cuenta de Media Services tiene una configuración inicial predeterminada con un punto de conexión de streaming y 10 unidades reservadas S3.
    * Para usar una cuenta de Media Services existente, seleccione **Usar el recurso existente**. En la lista de cuentas, seleccione su cuenta.

        La cuenta de Media Services debe tener la misma región que la cuenta de Video Indexer. 

        > [!NOTE]
        > Para minimizar la duración de la indexación y el bajo rendimiento, es muy recomendable ajustar el tipo y el número de [unidades reservadas](../previous/media-services-scale-media-processing-overview.md ) a **10 unidades reservadas S3** en su cuenta de Media Services. Consulte [Uso del portal para cambiar las unidades reservadas](../previous/media-services-portal-scale-media-processing.md).

    * Para configurar manualmente la conexión, haga clic en el vínculo **Switch to manual configuration** (Cambiar a configuración manual).

        Para más información, consulte la sección [Conexión manual de Azure](#connect-to-azure-manually-advanced-option) (opción avanzada) que viene a continuación.
6. Cuando termine, elija **Conectar**. Esta operación puede tardar unos minutos. 

    Después de conectarse a Azure, la nueva cuenta de Video Indexer aparece en la lista de cuentas:

    ![nueva cuenta](./media/create-account/new-account.png)

7. Vaya a la nueva cuenta

## <a name="connect-to-azure-manually-advanced-option"></a>Conexión manual a Azure (opción avanzada)

Si se produjo un error en la conexión a Azure, puede intentar solucionar el problema mediante la conexión manual.

> [!NOTE]
> Se recomienda encarecidamente tener las siguientes tres cuentas en la misma región: la cuenta de Video Indexer a la que se conecta con la cuenta de Media Services, así como la cuenta de almacenamiento de Azure conectada a la misma cuenta de Media Services.

### <a name="create-and-configure-a-media-services-account"></a>Creación y configuración de una cuenta de Media Services

1. Use [Azure Portal](https://portal.azure.com/) para crear una cuenta de Azure Media Services, como se describe en [Creación de una cuenta](../previous/media-services-portal-create-account.md).

    Al crear una cuenta de almacenamiento para la cuenta de Media Services, seleccione **StorageV2** para el tipo de cuenta y **con redundancia geográfica (GRS)** para los campos de la replicación.

    ![nueva cuenta de ASM](./media/create-account/create-ams-account1.png)

    > [!NOTE]
    > Asegúrese de anotar los nombres de recurso y cuenta de Media Services. Los necesitará para los pasos de la siguiente sección.

2. Ajustar el tipo y número de [unidades reservadas](../previous/media-services-scale-media-processing-overview.md ) a **10 unidades reservadas S3** en la cuenta de Media Services que ha creado. Consulte [Uso del portal para cambiar las unidades reservadas](../previous/media-services-portal-scale-media-processing.md).
3. Para poder reproducir los vídeos en la aplicación web de Video Indexer, debe iniciar el **punto de conexión de streaming** predeterminado de la nueva cuenta de Media Services.

    En la nueva cuenta de Media Services, haga clic en **Puntos de conexión de streaming**. Seleccione el punto de conexión de streaming y presione Iniciar.

    ![nueva cuenta de ASM](./media/create-account/create-ams-account2.png)

4. Para que Video Indexer se autentique con la API de Media Services, es necesario crear una aplicación de AD. Los pasos siguientes le guían por el proceso de autenticación de Azure AD que se describe en [Introducción a la autenticación de Azure AD mediante Azure Portal](../previous/media-services-portal-get-started-with-aad.md):

    1. En la nueva cuenta de Media Services, seleccione **Acceso de API**.
    2. Seleccione [Service principal authentication method](../previous/media-services-portal-get-started-with-aad.md#service-principal-authentication) (Método de autenticación de la entidad de servicio).
    3. Obtenga el identificador de cliente y el secreto de cliente, como se describe en la sección [Obtención del identificador de cliente y del secreto de cliente](../previous/media-services-portal-get-started-with-aad.md#get-the-client-id-and-client-secret).

        Después de seleccionar **Configuración**->**Claves**, agregar una **descripción** y presionar **Guardar**, el valor de la clave se rellena.

        Si la clave expira, el propietario de la cuenta tendrá que ponerse en contacto con el soporte técnico de Video Indexer para renovar la clave.

        > [!NOTE]
        > Asegúrese de anotar el valor de clave y el identificador de aplicación. Los necesitará para los pasos de la siguiente sección.

### <a name="connect-manually"></a>Conexión manual

En el cuadro de diálogo **Connect Video Indexer to an Azure subscription** (Conectar Video Indexer a una suscripción de Azure) de su página de [Video Indexer](https://www.videoindexer.ai/), seleccione el vínculo **Switch to manual configuration** (Cambiar a configuración manual).

En el cuadro de diálogo, proporcione la siguiente información:

|Configuración|DESCRIPCIÓN|
|---|---|
|Región de la cuenta de Video Indexer|El nombre de la región de la cuenta de Video Indexer. Para un mejor rendimiento y reducir los costos, se recomienda encarecidamente especificar el nombre de la región donde se encuentran el recurso de Azure Media Services y la cuenta de Azure Storage. |
|Inquilino de Azure Active Directory (AAD)|El nombre del inquilino de Azure AD, por ejemplo, "contoso.onmicrosoft.com". La información del inquilino se puede recuperar desde Azure Portal. Coloque el cursor sobre el nombre del usuario con sesión iniciada en la esquina superior derecha. Busque el nombre a la derecha de **Domain** (Dominio).|
|Id. de suscripción|La suscripción de Azure en la que debe crearse esta conexión. El id. de suscripción del inquilino se puede recuperar de Azure Portal. Haga clic en **Todos los servicios** en el panel izquierdo y busque "suscripciones". Seleccione **Subscriptions** (Suscripciones) y elija el identificador deseado de la lista de las suscripciones.|
|Nombre del grupo de recursos de Azure Media Services|El nombre del grupo de recursos en el que creó la cuenta de Media Services.|
|Nombre del recurso de Media Services|El nombre de la cuenta de Azure Media Services que creó en la sección anterior.|
|Identificador de aplicación|El identificador de aplicación de Azure AD (con permisos para la cuenta de Media Services especificada) que creó en la sección anterior.|
|Clave de aplicación|La clave de aplicación de Azure AD que creó en la sección anterior. |

## <a name="considerations"></a>Consideraciones

Tenga en cuenta las siguientes consideraciones con relación a Azure Media Services:

* Si se conecta automáticamente, verá un grupo de recursos, una cuenta de Media Services y una cuenta de almacenamiento nuevos en su suscripción de Azure.
* Si se conecta automáticamente, Video Indexer establece las **unidades reservadas** en 10 unidades de S3:

    ![Unidades reservadas de Media Services](./media/create-account/ams-reserved-units.png)

* Si se conecta a una cuenta de Media Services existente, Video Indexer no cambia la configuración existente de **unidades reservadas**.

   Es posible que tenga que ajustar el tipo y número de unidades reservadas, según la carga planeada. Tenga en cuenta que si la carga es alta y no tiene suficientes unidades o velocidad, el procesamiento de los vídeos pueden producir errores de tiempo de espera.

* Si se conecta a una nueva cuenta de Media Services, Video Indexer inicia automáticamente en ella el **punto de conexión de streaming** predeterminado:

    ![Punto de conexión de streaming de Media Services](./media/create-account/ams-streaming-endpoint.png)

    Los puntos de conexión de streaming tienen un tiempo de inicio considerable. Por lo tanto, pueden transcurrir varios minutos desde que conecta su cuenta a Azure hasta que los vídeos se pueden transmitir y ver en la aplicación web de Video Indexer.

* Si se conecta a una cuenta de Media Services existente, Video Indexer no cambia la configuración del punto de conexión de streaming predeterminado. Si no hay ningún **punto de conexión de streaming** en ejecución, no podrá ver vídeos desde esta cuenta de Media Services ni en Video Indexer.

## <a name="next-steps"></a>Pasos siguientes

Para interactuar mediante programación con su cuenta de evaluación gratuita o con sus cuentas de Video Indexer conectadas a Azure, siga las instrucciones descritas en: [Uso de las API](video-indexer-use-apis.md).

Debe utilizar el mismo usuario de Azure AD que usa al conectarse a Azure.


