---
title: Protección de aplicaciones demonio
titleSuffix: Azure Maps
description: Use Azure Portal para administrar la autenticación para configurar una aplicación demonio de confianza.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: cc39f8250ddc1b2fb1baaf073969f6aab5b1372c
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87531378"
---
# <a name="secure-a-daemon-application"></a>Protección de una aplicación demonio

La guía siguiente es para los procesos en segundo plano, temporizadores y trabajos que se hospedan en un entorno seguro y de confianza. Algunos ejemplos son los trabajos web de Azure, las aplicaciones de Azure Functions, los servicios de Windows y cualquier otro servicio en segundo plano de confianza.

> [!Tip]
> Microsoft recomienda implementar Azure Active Directory (Azure AD) y el control de acceso basado en rol (RBAC) para las aplicaciones de producción. Para obtener información general sobre los conceptos, consulte [Autenticación de Azure Maps](./azure-maps-authentication.md).

[!INCLUDE [authentication details](./includes/view-authentication-details.md)]

## <a name="scenario-shared-key-authentication"></a>Escenario: Autenticación de clave compartida

Después de crear la cuenta de Azure Maps, se generan las claves principal y secundaria. Se recomienda que use la clave principal como clave de suscripción cuando [llame a Azure Maps mediante la autenticación de clave compartida](https://docs.microsoft.com/azure/azure-maps/azure-maps-authentication#shared-key-authentication). Puede usar la clave secundaria, por ejemplo, para realizar cambios de clave graduales. Para más información, consulte [Autenticación con Azure Maps](https://aka.ms/amauth).

### <a name="securely-store-shared-key"></a>Almacenamiento seguro de la clave compartida

Las claves principal y secundaria permiten la autorización a todas las API de la cuenta de Maps. Las aplicaciones deben almacenar las claves en un almacén seguro como Azure Key Vault. La aplicación debe recuperar la clave compartida como un secreto de Azure Key Vault para evitar almacenar la clave compartida en texto sin formato en la configuración de la aplicación. Para saber cómo configurar un almacén de Azure Key Vault, consulte [Guía del desarrollador de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/developers-guide).

En los pasos siguientes se describe este proceso:

1. Cree un almacén de Azure Key Vault.
2. Cree una entidad de servicio de Azure AD mediante la creación de un registro de aplicación o una identidad administrada, la entidad de seguridad creada es responsable del acceso a Azure Key Vault.
3. Asigne a la entidad de servicio acceso a los secretos de Azure Key Vault con el permiso `Get`.
4. Asigne temporalmente acceso a los secretos con el permiso `Set` para el desarrollador.
5. Establezca la clave compartida en los secretos de Key Vault, haga referencia al identificador de secreto como configuración de la aplicación demonio y elimine el permiso `Set` del secreto.
6. Implemente la autenticación de Azure AD en la aplicación demonio para recuperar el secreto de la clave compartida desde Azure Key Vault.
7. Cree una solicitud a la API REST de Azure Maps con la clave compartida.

> [!Tip]
> Si la aplicación se hospeda en el entorno de Azure, debería implementar una identidad administrada para reducir el costo y la complejidad de la administración de un secreto para autenticarse en Azure Key Vault. Consulte el siguiente [tutorial para la conexión mediante una identidad administrada](https://docs.microsoft.com/azure/key-vault/general/tutorial-net-create-vault-azure-web-app) de Azure Key Vault.

La aplicación demonio es responsable de recuperar la clave compartida de un almacenamiento seguro. La implementación con Azure Key Vault requiere la autenticación mediante Azure AD para acceder al secreto. En su lugar, se recomienda la autenticación de RBAC de Azure AD para Azure Maps debido a la complejidad y los requisitos operativos adicionales que supone el uso de la autenticación de clave compartida.

> [!IMPORTANT]
> Para simplificar la regeneración de claves, se recomienda que las aplicaciones usen una clave cada vez. Después, las aplicaciones pueden regenerar la clave no utilizada e implementar la nueva clave regenerada en un almacén secreto seguro como Azure Key Vault.

## <a name="scenario-azure-ad-role-based-access-control"></a>Escenario: control de acceso basado en rol de Azure AD

Una vez creada una cuenta de Azure Maps, se muestra el valor de `x-ms-client-id` de Azure Maps en la página de detalles de autenticación de Azure Portal. Este valor representa la cuenta que se utilizará para las solicitudes a la API REST. Este valor se debe almacenar en la configuración de la aplicación y recuperarse antes de realizar las solicitudes HTTP. El objetivo del escenario es permitir que la aplicación demonio se autentique en Azure AD y llame a las API REST de Azure Maps.

> [!Tip]
> Se recomienda el hospedaje en Azure Virtual Machines, Virtual Machine Scale Sets o App Services para habilitar las ventajas de los componentes de las identidades administradas.

### <a name="daemon-hosted-on-azure-resources"></a>Demonio hospedado en recursos de Azure

Cuando la ejecución se realiza en recursos de Azure, configure identidades administradas de Azure para posibilitar un esfuerzo mínimo en la administración de credenciales y un bajo costo. 

Consulte [¿Qué son las identidades administradas de recursos de Azure?](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) para habilitar el acceso de la aplicación a una identidad administrada.

Ventajas de la identidad administrada:

* Autenticación de criptografía de clave pública de certificado X509 administrada por el sistema de Azure.
* Seguridad de Azure AD con certificados X509 en lugar de secretos de cliente.
* Azure administra y renueva todos los certificados asociados al recurso de la identidad administrada.
* Administración simplificada de las operaciones de credenciales que elimina la necesidad de un servicio de almacén de secretos seguro como Azure Key Vault. 

### <a name="daemon-hosted-on-non-azure-resources"></a>Demonio hospedado en recursos que no son de Azure

Cuando la ejecución se realiza en un entorno que no es de Azure, las identidades administradas no están disponibles. Por lo tanto, debe configurar una entidad de servicio mediante un registro de aplicación de Azure AD para la aplicación demonio.

1. En Azure Portal, en la lista de servicios de Azure, seleccione **Azure Active Directory** > **Registros de aplicaciones** > **Nuevo registro**.  

    > [!div class="mx-imgBorder"]
    > ![Registro de aplicaciones](./media/how-to-manage-authentication/app-registration.png)

2. Si ya ha registrado la aplicación, continúe en el siguiente paso. Si no la ha registrado, especifique un **nombre**, seleccione el **tipo de cuenta admitido** y haga clic en **Registrar**.  

    > [!div class="mx-imgBorder"]
    > ![Detalles del registro de la aplicación](./media/how-to-manage-authentication/app-create.png)

3. Para asignar permisos de API delegados a Azure Maps, vaya a la aplicación. A continuación, en **Registros de aplicaciones**, seleccione **Permisos de API** > **Add a permission** (Agregar un permiso). En **API usadas en mi organización**, busque y seleccione **Azure Maps**.

    > [!div class="mx-imgBorder"]
    > ![Adición de permisos de API a la aplicación](./media/how-to-manage-authentication/app-permissions.png)

4. Active la casilla situada junto a **Access Azure Maps** (Acceder a Azure Maps) y seleccione **Agregar permisos**.

    > [!div class="mx-imgBorder"]
    > ![Selección de permisos de API de la aplicación](./media/how-to-manage-authentication/select-app-permissions.png)

5. Complete los pasos siguientes para crear un secreto de cliente o configurar el certificado.

    * Si la aplicación utiliza la autenticación del servidor o de la aplicación, en la página de registro de la aplicación, vaya a **Certificates & secrets** (Certificados y secretos). Cargue un certificado de clave pública o cree una contraseña seleccionando **Nuevo secreto de cliente**.

        > [!div class="mx-imgBorder"]
        > ![Creación de un secreto de cliente](./media/how-to-manage-authentication/app-keys.png)

    * Después de seleccionar **Agregar**, copie el secreto y almacénelo de forma segura en un servicio como Azure Key Vault. Consulte [Guía del desarrollador de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/developers-guide) para almacenar de forma segura el certificado o el secreto. Usará este secreto para obtener tokens de Azure AD.

        > [!div class="mx-imgBorder"]
        > ![Adición de un secreto de cliente](./media/how-to-manage-authentication/add-key.png)

### <a name="grant-role-based-access-for-the-daemon-application-to-azure-maps"></a>Concesión de acceso basado en rol para la aplicación demonio a Azure Maps

El *control de acceso basado en rol* (RBAC) se concede mediante la asignación de la identidad administrada creada o de la entidad de servicio a una o más definiciones de roles de control de acceso de Azure Maps. Para ver las definiciones de roles de Azure disponibles para Azure Maps, vaya a **Control de acceso (IAM)** . Seleccione **Roles** y busque los roles que comienzan con *Azure Maps*. Esos son los roles de Azure Maps a los que puede conceder acceso.

> [!div class="mx-imgBorder"]
> ![Visualización de roles disponibles](./media/how-to-manage-authentication/how-to-view-avail-roles.png)

1. Vaya a su **cuenta de Azure Maps**. Seleccione **Control de acceso (IAM)**  > **Asignaciones de roles**.

    > [!div class="mx-imgBorder"]
    > ![Concesión de RBAC](./media/how-to-manage-authentication/how-to-grant-rbac.png)

2. En la pestaña **Asignaciones de roles**, seleccione **Agregar** para agregar una asignación de roles. 
    
    > [!div class="mx-imgBorder"]
    > ![Agregar asignación de roles](./media/how-to-manage-authentication/add-role-assignment.png)

3. Seleccione una definición de roles de Azure Maps integrada como **Lector de datos de Azure Maps** o **Colaborador de datos de Azure Maps**. En **Asignar acceso a**, seleccione **Usuario, grupo o entidad de servicio de Azure AD** o identidad administrada con **Identidad administrada asignada por el usuario** / **Identidad administrada asignada por el sistema**. Seleccione la entidad de seguridad. Después, seleccione **Guardar**.

    > [!div class="mx-imgBorder"]
    > ![Agregar asignación de roles](./media/how-to-manage-authentication/how-to-add-role-assignment.png)

4. Puede confirmar que se aplicó la asignación de roles en la pestaña Asignación de roles.

## <a name="request-token-with-managed-identity"></a>Solicitud de token con una identidad administrada

Una vez configurada una identidad administrada para el recurso de hospedaje, use el SDK o la API REST de Azure para adquirir un token de Azure Maps. Mas información en [Adquisición de un token de acceso](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token). Según la guía, se espera que se devuelva un token de acceso que se puede utilizar en las solicitudes a la API REST.

## <a name="request-token-with-application-registration"></a>Solicitud de un token con un registro de aplicación

Después de registrar la aplicación y asociarla a Azure Maps, puede solicitar los tokens de acceso.

* Identificador de recurso de Azure AD `https://atlas.microsoft.com/`
* Identificador de aplicación de Azure AD
* Identificador de inquilino de Azure AD
* Secreto de cliente de un registro de aplicación de Azure AD

Solicitud:

```http
POST /<Azure AD Tenant ID>/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=<Azure AD App ID>&resource=https://atlas.microsoft.com/&client_secret=<client secret>&grant_type=client_credentials
```

Respuesta:

```json
{
    "token_type": "Bearer",
    "expires_in": "...",
    "ext_expires_in": "...",
    "expires_on": "...",
    "not_before": "...",
    "resource": "https://atlas.microsoft.com/",
    "access_token": "ey...gw"
}
```

Para obtener ejemplos más detallados, consulte [Escenarios de autenticación en Azure AD](https://docs.microsoft.com/azure/active-directory/develop/authentication-scenarios).

## <a name="next-steps"></a>Pasos siguientes

Descubra las métricas de uso de API de la cuenta de Azure Maps:
> [!div class="nextstepaction"]
> [Visualización de métricas de uso](how-to-view-api-usage.md)

Explore ejemplos acerca de cómo integrar Azure AD con Azure Maps:
> [!div class="nextstepaction"]
> [Ejemplos de Azure Maps](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples)
