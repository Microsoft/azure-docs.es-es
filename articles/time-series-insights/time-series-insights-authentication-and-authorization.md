---
title: 'Autenticación y autorización de la API: Azure Time Series Insights | Microsoft Docs'
description: En este artículo se describe cómo configurar la autenticación y la autorización para una aplicación personalizada que llama a la API de Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: shresha
manager: dpalled
ms.reviewer: v-mamcge, jasonh, kfile
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 06/30/2020
ms.custom: seodec18, has-adal-ref
ms.openlocfilehash: 3441a540cb00dc9b85de54543484b1723fa4226e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080781"
---
# <a name="authentication-and-authorization-for-azure-time-series-insights-api"></a>Autenticación y autorización para la API de Azure Time Series Insights

En este documento se describe cómo registrar una aplicación en Azure Active Directory mediante la nueva hoja de Azure Active Directory. Las aplicaciones registradas en Azure Active Directory permiten a los usuarios autenticarse y recibir permiso para usar la API de Azure Time Series Insight asociada a un entorno de Azure Time Series Insights.

## <a name="service-principal"></a>Entidad de servicio

En las siguientes secciones se explica cómo configurar una aplicación para acceder a la API de Azure Time Series Insights en nombre de una aplicación. Después, la aplicación puede consultar o publicar datos de referencia en el entorno de Azure Time Series Insights con sus propias credenciales de aplicación mediante Azure Active Directory.

## <a name="summary-and-best-practices"></a>Resumen y procedimientos recomendados

El flujo de registro de una aplicación de Azure Active Directory conlleva tres pasos principales.

1. [Registrar una aplicación](#azure-active-directory-app-registration) en Azure Active Directory.
1. Autorizar a la aplicación para tener [acceso a datos en el entorno de Azure Time Series Insights](#granting-data-access).
1. Usar el **identificador de la aplicación** y el **secreto de cliente** para obtener un token de `https://api.timeseries.azure.com/` en la [aplicación cliente](#client-app-initialization). Luego el token puede usarse para llamar a la API de Azure Time Series Insights.

Según el **paso 3**, distinguir entre las credenciales de la aplicación y las suyas de usuario le permite lo siguiente:

* Asignar permisos a la identidad de la aplicación que sean diferentes a los suyos propios. Normalmente, estos permisos están restringidos a exactamente aquello que la aplicación requiere. Por ejemplo, puede permitir que la aplicación lea datos de únicamente un entorno de Azure Time Series Insights determinado.
* Aislar la seguridad de la aplicación de las credenciales de autenticación del usuario, utilizando para ello un **secreto de cliente** o un certificado de seguridad. Como resultado, las credenciales de la aplicación no son dependientes de las credenciales de un usuario específico. Si el rol del usuario cambia, la aplicación no necesitará nuevas credenciales ni ninguna configuración adicional. Si el usuario cambia de contraseña, todo el acceso a la aplicación no requiere nuevas credenciales o claves.
* Ejecutar un script de instalación desatendida con un **secreto de cliente** o con un certificado de seguridad, en lugar de con las credenciales de un usuario específico (lo que requeriría estar presente).
* Usar un certificado de seguridad en lugar de una contraseña para proteger el acceso a la API de Azure Time Series Insights.

> [!IMPORTANT]
> Siga el principio de **Separación de intereses** (descrito anteriormente para este escenario) al configurar la directiva de seguridad de Azure Time Series Insights.

> [!NOTE]
> * Este artículo se centra en una aplicación de un único inquilino diseñada para ejecutarse en una sola organización.
> * Las aplicaciones de un único inquilino suelen usarse en aplicaciones de línea de negocio que se ejecutan en la organización.

## <a name="detailed-setup"></a>Configuración detallada

### <a name="azure-active-directory-app-registration"></a>Registro de la aplicación de Azure Active Directory

[!INCLUDE [Azure Active Directory app registration](../../includes/time-series-insights-aad-registration.md)]

### <a name="granting-data-access"></a>Conceder acceso a datos

1. En el entorno de Azure Time Series Insights, seleccione **Directivas de acceso a datos** y, después, **Agregar**.

   [![Adición de nueva directiva de acceso a datos al entorno de Azure Time Series Insights](media/authentication-and-authorization/time-series-insights-data-access-policies-add.png)](media/authentication-and-authorization/time-series-insights-data-access-policies-add.png#lightbox)

1. En el cuadro de diálogo **Seleccionar usuario**, pegue el **nombre de la aplicación** o el **identificador de la aplicación** desde la sección de registro de la aplicación de Azure Active Directory.

   [![Búsqueda de una aplicación en el cuadro de diálogo Seleccionar usuario](media/authentication-and-authorization/time-series-insights-data-access-policies-select-user.png)](media/authentication-and-authorization/time-series-insights-data-access-policies-select-user.png#lightbox)

1. Seleccione el rol. Seleccione **Lector** para consultar datos o **Colaborador** para consultar datos y cambiar los datos de referencia. Seleccione **Aceptar**.

   [![Selección de Lector o Colaborador en el cuadro de diálogo Seleccionar rol de usuario](media/authentication-and-authorization/time-series-insights-data-access-policies-select-role.png)](media/authentication-and-authorization/time-series-insights-data-access-policies-select-role.png#lightbox)

1. Seleccione **Aceptar** para guardar la directiva.

   > [!TIP]
   > Para más información sobre las opciones avanzadas de acceso a datos, lea [Conceder acceso a datos](./time-series-insights-data-access.md).

### <a name="client-app-initialization"></a>Inicialización de la aplicación cliente

* Los desarrolladores pueden usar la [biblioteca de autenticación de Microsoft (MSAL)] para autenticarse con Azure Time Series Insights.

* Para autenticarse mediante MSAL:

   1. Use el **identificador de la aplicación** y el **secreto de cliente** (clave de aplicación) de la sección de registro de la aplicación de Azure Active Directory para obtener el token en nombre de la aplicación.

   1. En C#, puede emplear el siguiente código para conseguir el token en nombre de la aplicación. Para obtener un ejemplo completo sobre cómo consultar datos desde un entorno de Gen1, lea [Consultar de datos mediante C#](time-series-insights-query-data-csharp.md).

        [!code-csharp[csharpquery-example](~/samples-tsi/csharp-tsi-ga-sample/Program.cs?range=170-199)]

   1. Luego el token se puede pasar en el encabezado `Authorization` cuando la aplicación llame a la API de Azure Time Series Insights.

> [!IMPORTANT]
> Si usa la [Biblioteca de autenticación de Azure Active Directory (ADAL)](https://docs.microsoft.com/azure/active-directory/azuread-dev/active-directory-authentication-libraries), lea acerca de la [migración a MSAL](https://docs.microsoft.com/azure/active-directory/develop/msal-net-migration).

## <a name="common-headers-and-parameters"></a>Parámetros y encabezados comunes

En esta sección, se describen los encabezados y parámetros comunes de la solicitud HTTP que se usan para realizar consultas frente a las API de Azure Time Series Insights Gen1 y Gen2. Los requisitos específicos de las API se describen con mayor detalle en la [documentación de referencia de la API de REST de Azure Time Series Insights](https://docs.microsoft.com/rest/api/time-series-insights/).

> [!TIP]
> Lea la [Referencia de la API de REST de Azure](https://docs.microsoft.com/rest/api/azure/) para obtener más información sobre cómo usar las API de REST, realizar solicitudes HTTP y manipular las respuestas HTTP.

### <a name="authentication"></a>Authentication

Para realizar consultas autenticadas en las [API de REST de Azure Time Series Insights](https://docs.microsoft.com/rest/api/time-series-insights/), se debe pasar un token de portador OAuth 2.0 válido en el [Encabezado de autorización](/rest/api/apimanagement/2019-12-01/authorizationserver/createorupdate) mediante un cliente de REST de su elección (Postman, JavaScript, C#).

> [!TIP]
> Lea la [visualización de muestra del SDK de cliente](https://tsiclientsample.azurewebsites.net/) hospedado de Azure Time Series Insights para aprender a autenticarse con las API de Azure Time Series Insights mediante programación, usando el [SDK de cliente de JavaScript](https://github.com/microsoft/tsiclient/blob/master/docs/API.md) junto con gráficos.

### <a name="http-headers"></a>Encabezados HTTP

A continuación se describen los encabezados de solicitud obligatorios.

| Encabezado de solicitud obligatorio | Descripción |
| --- | --- |
| Authorization | Para la autenticación en Azure Time Series Insights, es necesario pasar un token de portador de OAuth 2.0 válido en el encabezado de **autorización**. |

> [!IMPORTANT]
> El token se debe emitir exactamente en el recurso `https://api.timeseries.azure.com/` (también conocido como "audiencia" del token).
> * Por lo tanto, [Postman](https://www.getpostman.com/) **AuthURL** será: `https://login.microsoftonline.com/microsoft.onmicrosoft.com/oauth2/authorize?scope=https://api.timeseries.azure.com/.default`
> * `https://api.timeseries.azure.com/` es válido, pero `https://api.timeseries.azure.com` no.

A continuación se describen los encabezados de solicitud opcionales.

| Encabezado de solicitud opcional | Descripción |
| --- | --- |
| Content-type | Solo se admite `application/json`. |
| x-ms-client-request-id | Un id. de solicitud de cliente. El servicio registra este valor. Permite que el servicio realice el seguimiento de la operación en todos los servicios. |
| x-ms-client-session-id | Un id. de sesión de cliente. El servicio registra este valor. Permite que el servicio realice el seguimiento de un grupo de operaciones relacionadas en todos los servicios. |
| x-ms-client-application-name | Nombre de la aplicación que generó esta solicitud. El servicio registra este valor. |

A continuación se describen los encabezados de respuesta opcionales, pero recomendados.

| Encabezado de respuesta | Descripción |
| --- | --- |
| Content-type | Solo se admite `application/json`. |
| x-ms-request-id | Id. de solicitud que creó el servidor. Se puede usar para ponerse en contacto con Microsoft para investigar una solicitud. |
| x-ms-property-not-found-behavior | Encabezado de respuesta opcional de la API GA. Los valores posibles son `ThrowError` (valor predeterminado) o `UseNull`. |

### <a name="http-parameters"></a>Parámetros de HTTP

> [!TIP]
> Encontrará más detalles sobre la información de consulta necesaria y opcional en la [documentación de referencia](https://docs.microsoft.com/rest/api/time-series-insights/).

Los parámetros de la cadena de consulta de la URL obligatorios dependen de la versión de la API.

| Release | Posibles valores de versión de API |
| --- |  --- |
| Gen1 | `api-version=2016-12-12`|
| Gen2 | `api-version=2020-07-31` y `api-version=2018-11-01-preview`|

> [!IMPORTANT]
>
> La versión `api-version=2018-11-01-preview` caerá en desuso pronto. Se recomienda a los usuarios cambiar a la versión más reciente.

Los parámetros de cadena de consulta de dirección URL opcionales incluyen establecer un tiempo de espera para los tiempos de ejecución de la solicitud HTTP.

| Parámetro de consulta opcional | Descripción | Versión |
| --- |  --- | --- |
| `timeout=<timeout>` | Tiempo de espera del lado servidor para la ejecución de la solicitud. Esto es aplicable solo a las API [Get Environment Events](https://docs.microsoft.com/rest/api/time-series-insights/ga-query-api#get-environment-events-api) y [Get Environment Aggregates](https://docs.microsoft.com/rest/api/time-series-insights/ga-query-api#get-environment-aggregates-api). El valor del tiempo de espera debe estar en un formato de duración de tipo ISO 8601, por ejemplo `"PT20S"`, y debe estar en el intervalo `1-30 s`. El valor predeterminado es `30 s`. | Gen1 |
| `storeType=<storeType>` | En el caso de los entornos de Gen2 con almacenamiento intermedio habilitado, la consulta se puede ejecutar en `WarmStore` o `ColdStore`. Este parámetro de la consulta define en qué almacén debe ejecutarse la consulta. Si no se define, la consulta se ejecutará en el almacén frío. Para consultar la tienda en caliente, **storeType** debe establecerse en `WarmStore`. Si no se define, la consulta se ejecutará en el almacén frío. | Gen2 |

## <a name="next-steps"></a>Pasos siguientes

* Para obtener código de ejemplo que llama a la API de Azure Time Series Insights Gen1, lea [Consulta de datos de Gen1 mediante C#](./time-series-insights-query-data-csharp.md).

* Para obtener código de ejemplo que llama a la API de Azure Time Series Insights Gen2, lea [Consulta de datos de Gen2 mediante C#](./time-series-insights-update-query-data-csharp.md).

* Para información de referencia de la API, lea la documentación de [referencia sobre la API de consulta](https://docs.microsoft.com/rest/api/time-series-insights/ga-query-api).

* Sepa cómo [crear una entidad de servicio](../active-directory/develop/howto-create-service-principal-portal.md).
