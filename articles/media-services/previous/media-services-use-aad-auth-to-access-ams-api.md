---
title: Acceso a Azure Media Services API con la autenticación de Azure Active Directory | Microsoft Docs
description: Información acerca de los conceptos y los pasos que se deben seguir para usar Azure Active Directory (Azure AD) a fin de autenticar el acceso a Azure Media Services API.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/01/2019
ms.author: juliako
ms.openlocfilehash: 9d73c280e3f5639b11e0d07b9ef2f775f48cd3c3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87038495"
---
# <a name="access-the-azure-media-services-api-with-azure-ad-authentication"></a>Acceso a Azure Media Services API con la autenticación de Azure AD  

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Azure Media Services API es una API RESTful. Puede usarla para realizar operaciones en recursos multimedia a través de una API de REST o mediante los SDK del cliente disponibles. Azure Media Services ofrece un SDK del cliente de Media Services para Microsoft .NET. Para ser autorizado a acceder a recursos de Media Services y a Media Services API, se debe autenticar primero. 

Media Services admite la [autenticación basada en Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md). El servicio Azure Media REST requiere que el usuario o la aplicación que hace las solicitudes a la API de REST tenga el rol de **colaborador** o **propietario** para acceder a los recursos. Para más información, consulte [Introducción al control de acceso basado en roles en Azure Portal](../../role-based-access-control/overview.md).  

Este documento proporciona información general sobre cómo obtener acceso a Media Services API mediante las API de .NET o REST.

> [!NOTE]
> La autorización de Access Control quedó en desuso el 1 de junio de 2018.

## <a name="access-control"></a>Control de acceso

Para que la solicitud de REST de Azure Media Services tenga éxito, el usuario que realiza la llamada debe tener el rol de colaborador o propietario de la cuenta de Media Services a la que está intentando obtener acceso.  
Solo un usuario con el rol de propietario puede dar acceso a los recursos multimedia (cuenta) a nuevos usuarios o aplicaciones. El rol de colaborador solo puede tener acceso al recurso multimedia.
Las solicitudes no autorizadas producirán un error, con el código de estado 401. Si ve este código de error, compruebe si el usuario tiene el rol de colaborador o propietario asignado para la cuenta de usuario de Media Services. Puede comprobarlo en Azure Portal. Busque la cuenta de Media Services y, a continuación, haga clic en la pestaña **Control de acceso**. 

![Pestaña Control de acceso](./media/media-services-use-aad-auth-to-access-ams-api/media-services-access-control.png)

## <a name="types-of-authentication"></a>Tipos de autenticación 
 
Al utilizar la autenticación de Azure AD con Azure Media Services, tiene dos opciones de autenticación:

- **Autenticación de usuario**. Autenticar a alguien que usa la aplicación para interactuar con los recursos de Media Services. La aplicación interactiva, en primer lugar, debe solicitar al usuario las credenciales. Un ejemplo es una aplicación de consola de administración que usan los usuarios autorizados para supervisar trabajos de codificación o streaming en vivo. 
- **Autenticación de entidad de servicio**. Autenticar un servicio. Las aplicaciones que normalmente utilizan este método de autenticación son las que ejecutan servicios de demonio, servicios de nivel intermedio o trabajos programados. Algunos ejemplos son las Web Apps, las Function Apps, las Logic Apps, las API y los microservicios.

### <a name="user-authentication"></a>Autenticación de usuarios 

Las aplicaciones que deben usar el método de autenticación de usuario son las aplicaciones nativas de administración o supervisión: de dispositivos móviles, de Windows y de consola. Este tipo de solución es útil cuando desee la interacción humana con el servicio en uno de los siguientes escenarios:

- Panel de supervisión para los trabajos de codificación.
- Panel de supervisión para Live Stream.
- Aplicación de administración para los usuarios de escritorio o dispositivos móviles para administrar los recursos en una cuenta de Media Services.

> [!NOTE]
> Este método de autenticación no debe usarse para las aplicaciones orientadas al consumidor. 

Una aplicación nativa debe adquirir primero un token de acceso de Azure AD y, a continuación, usarlo al realizar solicitudes HTTP en la API de REST de Media Services. Agregue el token de acceso para el encabezado de solicitud. 

El siguiente diagrama muestra un flujo de autenticación de una aplicación interactiva típica: 

![Diagrama de aplicaciones nativas](./media/media-services-use-aad-auth-to-access-ams-api/media-services-native-aad-app1.png)

En el diagrama anterior, los números representan el flujo de las solicitudes en orden cronológico.

> [!NOTE]
> Cuando se usa el método de autenticación de usuario, todas las aplicaciones comparten el mismo identificador de cliente de aplicación nativa (valor predeterminado) y el URI de redireccionamiento de aplicación nativa. 

1. Pida al usuario las credenciales.
2. Solicite un token de acceso de Azure AD con los siguientes parámetros:  

   * Punto de conexión de inquilino de Azure AD.

       La información del inquilino se puede recuperar desde Azure Portal. Coloque el cursor sobre el nombre del usuario con sesión iniciada en la esquina superior derecha.
   * URI del recurso de Media Services. 

       Este URI es el mismo para las cuentas de Media Services que se encuentran en el mismo entorno de Azure (por ejemplo, https:\//rest.media.azure.net).

   * Id. de cliente de aplicación de Media Services (nativo).
   * URI de redireccionamiento de aplicación de Media Services (nativo).
   * URI del recurso de Media Services de REST.
        
       El URI representa el punto de conexión de API REST (por ejemplo, https://test03.restv2.westus.media.azure.net/api/).

     Para obtener los valores de estos parámetros, vea [Uso de Azure Portal para acceder a la configuración de autenticación de Azure AD](media-services-portal-get-started-with-aad.md) con la opción de autenticación de usuario.

3. El token de acceso de Azure AD se envía al cliente.
4. El cliente envía una solicitud a la API de REST de Azure Media Services con el token de acceso de Azure AD.
5. El cliente recibe los datos de Media Services.

Para más información sobre cómo usar la autenticación de Azure AD para comunicarse con las solicitudes de REST mediante el SDK del cliente para Media Services .NET, vea [Uso de la autenticación de Azure AD para tener acceso a Media Services API con .NET](media-services-dotnet-get-started-with-aad.md). 

Si no usa el SDK del cliente para Media Services .NET, debe crear manualmente una solicitud de token de acceso de Azure AD con los parámetros descritos en el paso 2. Para más información, vea [Procedimiento para usar la Biblioteca de autenticación de Azure AD para obtener el token de Azure AD](../../active-directory/azuread-dev/active-directory-authentication-libraries.md).

### <a name="service-principal-authentication"></a>Autenticación de entidad de servicio

Las aplicaciones que normalmente utilizan este método de autenticación son las que ejecutan servicios de nivel intermedio o trabajos programados: Web Apps, Function Apps, Logic Apps, API o microservicios. Este método de autenticación también es adecuado para las aplicaciones interactivas en las que puede preferir usar una cuenta de servicio para administrar los recursos.

Al utilizar el método de autenticación de entidad de servicio para crear escenarios para el consumidor, la autenticación normalmente se controla en el nivel intermedio (a través de alguna API) y no directamente en una aplicación de escritorio o dispositivo móvil. 

Para usar este método, cree una aplicación de Azure AD y la entidad de seguridad en su propio inquilino. Después de crear la aplicación, asigne al rol colaborador o propietario de la aplicación acceso a la cuenta de Media Services. Puede hacerlo en Azure Portal mediante la CLI de Azure o con un script de PowerShell. También puede usar una aplicación de Azure AD existente. Puede registrar y administrar la aplicación de Azure AD y la entidad de servicio [en Azure Portal](media-services-portal-get-started-with-aad.md). También puede usar la [CLI de Azure](media-services-use-aad-auth-to-access-ams-api.md) o [PowerShell](media-services-powershell-create-and-configure-aad-app.md). 

![Aplicaciones de nivel intermedio](./media/media-services-use-aad-auth-to-access-ams-api/media-services-principal-service-aad-app1.png)

Después de crear la aplicación de Azure AD, se obtienen valores de las siguientes opciones. Necesitará estos valores para la autenticación:

- Id. de cliente 
- Secreto del cliente 

En la ilustración anterior, los números representan el flujo de las solicitudes en orden cronológico:
    
1. Una aplicación de nivel intermedio (API de web o aplicación web) solicita un token de acceso de Azure AD que tiene los siguientes parámetros:  

   * Punto de conexión de inquilino de Azure AD.

       La información del inquilino se puede recuperar desde Azure Portal. Coloque el cursor sobre el nombre del usuario con sesión iniciada en la esquina superior derecha.
   * URI del recurso de Media Services. 

       Este URI es el mismo para las cuentas de Media Services que se encuentran en el mismo entorno de Azure (por ejemplo, https:\//rest.media.azure.net).

   * URI del recurso de Media Services de REST.

       El URI representa el punto de conexión de API REST (por ejemplo, https://test03.restv2.westus.media.azure.net/api/).

   * Valores de aplicación de Azure AD: el Id. de cliente y el secreto de cliente.
    
     Para obtener los valores de estos parámetros, vea [Uso de Azure Portal para acceder a la configuración de autenticación de Azure AD](media-services-portal-get-started-with-aad.md) con la opción de autenticación de entidad de servicio.

2. El token de acceso de Azure AD se envía al nivel intermedio.
4. El nivel intermedio envía una solicitud a la API de REST de Azure Media Services con el token de Azure AD.
5. El nivel intermedio recibe los datos de Media Services.

Para más información sobre cómo usar la autenticación de Azure AD para comunicarse con las solicitudes de REST mediante el SDK del cliente para Media Services .NET, vea [Uso de la autenticación de Azure AD para tener acceso a Media Services API con .NET](media-services-dotnet-get-started-with-aad.md). 

Si no usa el SDK del cliente para Media Services .NET, debe crear manualmente una solicitud de token de acceso de Azure AD con los parámetros descritos en el paso 1. Para más información, vea [Procedimiento para usar la Biblioteca de autenticación de Azure AD para obtener el token de Azure AD](../../active-directory/azuread-dev/active-directory-authentication-libraries.md).

## <a name="troubleshooting"></a>Solución de problemas

Excepción: "El servidor remoto devolvió un error: (401) No autorizado".

Solución: Para que la solicitud de REST de Media Services sea correcta, el usuario que hace la llamada debe tener el rol de colaborador o propietario en la cuenta de Media Services a la que intenta acceder. Para más información, consulte [Control de acceso](media-services-use-aad-auth-to-access-ams-api.md#access-control).

## <a name="resources"></a>Recursos

Los artículos siguientes son información general de conceptos de autenticación de Azure AD: 

- [Escenarios de autenticación abordados por Azure AD](../../active-directory/develop/authentication-vs-authorization.md)
- [Incorporación, actualización o eliminación de una aplicación en Azure AD](../../active-directory/develop/quickstart-register-app.md)
- [Configuración y administración del control de acceso basado en rol con Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)

## <a name="next-steps"></a>Pasos siguientes

* Uso de Azure Portal para [acceder a la autenticación de Azure para usar Azure Media Services API](media-services-portal-get-started-with-aad.md).
* Uso de la autenticación de Azure AD para [acceder a Azure Media Services API con .NET](media-services-dotnet-get-started-with-aad.md).
