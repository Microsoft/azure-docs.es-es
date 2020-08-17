---
title: Autenticación de una aplicación para acceder a los recursos de Azure Event Hubs
description: En este artículo se proporciona información sobre cómo autenticar una aplicación con Azure Active Directory para acceder a recursos de Azure Event Hubs.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: 2b4456f63ce6d50ab1187f65deb1ee3280487580
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87531530"
---
# <a name="authenticate-an-application-with-azure-active-directory-to-access-event-hubs-resources"></a>Autenticación de una aplicación con Azure Active Directory para acceder a recursos de Event Hubs
Microsoft Azure proporciona una administración integrada del control de acceso para recursos y aplicaciones basados en Azure Active Directory (Azure AD). Una ventaja clave de usar Azure AD con Azure Event Hubs es que ya no es necesario almacenar las credenciales en el código. En su lugar, puede solicitar un token de acceso de OAuth 2.0 desde la Plataforma de identidad de Microsoft. El nombre del recurso para solicitar un token es `https://eventhubs.azure.net/` (para los clientes de Kafka, el recurso para solicitar un token es `https://<namespace>.servicebus.windows.net`). Azure AD autentica la entidad de seguridad (un usuario, grupo o entidad de servicio) ejecutando la aplicación. Si la autenticación se realiza correctamente, Azure AD devuelve un token de acceso a la aplicación y la aplicación puede entonces usar el token de acceso para autorizar una solicitud de los recursos de Azure Event Hubs.

Cuando un rol se asigna a una entidad de seguridad de Azure AD, Azure concede acceso a esos recursos a esa entidad de seguridad. El acceso puede tener como ámbito el nivel de suscripción, el grupo de recursos, el espacio de nombres de Event Hubs o cualquier recurso que haya en él. Una entidad de seguridad de Azure AD puede asignar roles a un usuario, un grupo, una entidad de servicio de aplicación o una [identidad administrada para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md). 

> [!NOTE]
> Una definición de roles es una colección de permisos. El control de acceso basado en rol (RBAC) controla cómo se aplican estos permisos a través de la asignación de roles. Una asignación de roles consta de tres elementos: entidad de seguridad, definición de rol y ámbito. Para más información, consulte [Descripción de los distintos roles](../role-based-access-control/overview.md).

## <a name="built-in-roles-for-azure-event-hubs"></a>Roles integrados de Azure Event Hubs
Azure proporciona los siguientes roles integrados de Azure para autorizar el acceso a datos de Event Hubs con Azure AD y OAuth:

- [Propietario de los datos de Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-owner): use este rol para proporcionar acceso completo a los recursos de Event Hubs.
- [Emisor de datos de Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-sender): use este rol para proporcionar acceso de envío a los recursos de Event Hubs.
- [Receptor de datos de Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-receiver): use este rol para proporcionar acceso de recepción a los recursos de Event Hubs.   

> [!IMPORTANT]
> Nuestra versión preliminar permite agregar privilegios de acceso a datos de Event Hubs al rol Propietario o Colaborador, pero los privilegios de acceso a datos de los roles Propietario y Colaborador ya no están en vigor. Si está usando el rol Propietario o Colaborador, cambie al rol Propietario de los datos de Azure Event Hubs.

## <a name="assign-azure-roles-using-the-azure-portal"></a>Asignación de roles de Azure mediante Azure Portal  
Para más información sobre cómo administrar el acceso a los recursos de Azure con RBAC y Azure Portal, consulte [este artículo](..//role-based-access-control/role-assignments-portal.md). 

Después de determinar el ámbito adecuado de una asignación de roles, vaya a ese recurso en Azure Portal. Acceda a la configuración Control de acceso (IAM) del recurso y siga estas instrucciones para administrar las asignaciones de roles:

> [!NOTE]
> Los pasos que se describen a continuación asignan un rol al centro de eventos en los espacios de nombres de Event Hubs, pero puede seguir los mismos pasos para asignar un rol con ámbito a cualquier recurso de Event Hubs.

1. En [Azure Portal](https://portal.azure.com/), vaya al espacio de nombres de Event Hubs.
2. En la página **Información general**, seleccione el centro de eventos para el que desea asignar un rol.

    ![Selección del centro de eventos](./media/authenticate-application/select-event-hub.png)
1. Seleccione **Access Control (IAM)** para mostrar la configuración del control de acceso del centro de eventos. 
1. Seleccione la pestaña **Asignaciones de roles** para ver la lista de asignaciones de roles. Seleccione el botón **Agregar** de la barra de herramientas y luego seleccione **Agregar asignación de roles**. 

    ![Botón Agregar de la barra de herramientas](./media/authenticate-application/role-assignments-add-button.png)
1. En la página **Agregar asignación de roles**, siga estos pasos:
    1. Seleccione el **rol Event Hubs** que quiere asignar. 
    1. Realice una búsqueda para localizar la **entidad de seguridad** (usuario, grupo, entidad de servicio) a la que quiere asignar el rol.
    1. Seleccione **Guardar** para guardar la asignación de roles. 

        ![Asignación de un rol a un usuario](./media/authenticate-application/assign-role-to-user.png)
    4. La identidad a la que ha asignado el rol aparece enumerada debajo de ese rol. Por ejemplo, en la imagen siguiente se muestra que Azure-users está en el rol Propietario de los datos de Azure Event Hubs. 
        
        ![Usuario en la lista](./media/authenticate-application/user-in-list.png)

Puede realizar este mismo procedimiento para asignar un rol que tenga como ámbito un espacio de nombre de Event Hubs, un grupo de recursos o una suscripción. Una vez que defina el rol y su ámbito, puede probar este comportamiento con ejemplos [en esta ubicación de GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac).


## <a name="authenticate-from-an-application"></a>Autenticación desde una aplicación
Una ventaja clave del uso de Azure AD con Event Hubs es que ya no necesita almacenar las credenciales en el código. En su lugar, puede solicitar un token de acceso de OAuth 2.0 desde la Plataforma de identidad de Microsoft. Azure AD autentica la entidad de seguridad (un usuario, grupo o entidad de servicio) ejecutando la aplicación. Si la autenticación se realiza correctamente, Azure AD devuelve el token de acceso a la aplicación y la aplicación puede entonces usar el token de acceso para autorizar las solicitudes de Azure Event Hubs.

En las secciones siguientes se muestra cómo configurar su aplicación web o aplicación nativa para la autenticación con la Plataforma de identidad de Microsoft 2.0. Para obtener más información sobre la Plataforma de identidad de Microsoft 2.0, consulte la [Introducción a la Plataforma de identidad de Microsoft (v2.0)](../active-directory/develop/v2-overview.md).

Para información general sobre el flujo de concesión de código OAuth 2.0, consulte [Autorización del acceso a aplicaciones web de Azure Active Directory mediante el flujo de concesión de código OAuth 2.0](../active-directory/develop/v2-oauth2-auth-code-flow.md).

### <a name="register-your-application-with-an-azure-ad-tenant"></a>Registro de la aplicación con un inquilino de Azure AD
El primer paso para usar Azure AD con el fin de autorizar los recursos de Event Hubs es registrar la aplicación cliente con un inquilino de Azure AD desde [Azure Portal](https://portal.azure.com/). Al registrar la aplicación cliente, facilita información sobre la aplicación a AD. Azure AD proporciona un id. de cliente (también denominado id. de aplicación) que se puede usar para asociar la aplicación con el runtime de Azure AD. Para más información sobre el identificador de cliente, consulte [Objetos de aplicación y de entidad de servicio de Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md). 

En las imágenes siguientes se muestran los pasos para registrar una aplicación web:

![Registro de una aplicación](./media/authenticate-application/app-registrations-register.png)

> [!Note]
> Si registra la aplicación como una aplicación nativa, puede especificar cualquier URI válido para el URI de redirección. Para las aplicaciones nativas, no es necesario que este valor sea una dirección URL real. Para las aplicaciones web, el URI de redirección debe ser un URI válido, ya que especifica la dirección URL a la que se proporcionan los tokens.

Una vez que ha registrado su aplicación, verá el **id. de la aplicación (o id. de cliente)** en **Configuración**:

![Id. de aplicación de la aplicación registrada](./media/authenticate-application/application-id.png)

Para más información sobre cómo registrar una aplicación con Azure AD, consulte [Integración de aplicaciones con Azure Active Directory](../active-directory/develop/quickstart-register-app.md).


### <a name="create-a-client-secret"></a>Creación de un secreto de cliente   
La aplicación necesita un secreto de cliente para demostrar su identidad al solicitar un token. Para agregar el secreto de cliente, siga estos pasos.

1. Vaya a su registro de aplicaciones en Azure Portal.
1. Seleccione la configuración de **Certificados y secretos**.
1. En **Secretos de cliente**, seleccione **Nuevo secreto de cliente** para crear un nuevo secreto.
1. Proporcione una descripción para el secreto y elija el intervalo de expiración deseado.
1. Copie inmediatamente el valor del nuevo secreto en una ubicación segura. El valor completo se le muestra solo una vez.

    ![Secreto del cliente](./media/authenticate-application/client-secret.png)


### <a name="client-libraries-for-token-acquisition"></a>Bibliotecas cliente para la adquisición de tokens  
Una vez que haya registrado su aplicación y le haya concedido permisos para enviar/recibir datos en Azure Event Hubs, podrá agregar código a su aplicación para autenticar una entidad de seguridad y adquirir un token de OAuth 2.0. Para autenticar y adquirir el token, puede usar una de las [bibliotecas de autenticación de la Plataforma de identidad de Microsoft](../active-directory/develop/reference-v2-libraries.md) u otra biblioteca de código abierto que admita OpenID o Connect 1.0. A continuación, su aplicación puede usar el token de acceso para autorizar una solicitud en Azure Event Hubs.

Para ver una lista de escenarios en los que se admite la adquisición de tokens, consulte la sección [Escenarios](https://aka.ms/msal-net-scenarios) del repositorio de GitHub [Biblioteca de autenticación de Microsoft (MSAL) para .NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet).

## <a name="samples"></a>Ejemplos
- [Ejemplos de Microsoft.Azure.EventHubs](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac). 
    
    En estos ejemplos se usa la biblioteca anterior **Microsoft.Azure.EventHubs**, pero se puede actualizar fácilmente para usar la biblioteca **Azure.Messaging.EventHubs** más reciente. Para que los ejemplos usen la biblioteca nueva en lugar de la anterior, consulte la [Guía para migrar de Microsoft.Azure.EventHubs a Azure.Messaging.EventHubs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md).
- [Ejemplos de Azure.Messaging.EventHubs](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Azure.Messaging.EventHubs/ManagedIdentityWebApp)

    Este ejemplo se ha actualizado para usar la biblioteca **Azure.Messaging.EventHubs** más reciente.

## <a name="next-steps"></a>Pasos siguientes
- Para más información acerca de RBAC, consulte [¿Qué es el control de acceso basado en rol (RBAC) de Azure?](../role-based-access-control/overview.md).
- Para información sobre cómo asignar y administrar las asignaciones de roles de Azure con Azure PowerShell, la CLI de Azure o la API de REST, consulte estos artículos:
    - [Administración del control de acceso basado en rol (RBAC) con Azure PowerShell](../role-based-access-control/role-assignments-powershell.md)  
    - [Administración del control de acceso basado en rol (RBAC) con la CLI de Azure](../role-based-access-control/role-assignments-cli.md)
    - [Administración del control de acceso basado en rol (RBAC) con la API de REST](../role-based-access-control/role-assignments-rest.md)
    - [Administración del acceso a los recursos de Azure mediante RBAC y plantillas de Azure Resource Manager](../role-based-access-control/role-assignments-template.md)

Consulte los artículos relacionados siguientes:
- [Autenticación de una identidad administrada con Azure Active Directory para acceder a recursos de Event Hubs](authenticate-managed-identity.md)
- [Autenticación de solicitudes para Azure Event Hubs mediante firmas de acceso compartido](authenticate-shared-access-signature.md)
- [Autorización del acceso a recursos de Event Hubs mediante Azure Active Directory](authorize-access-azure-active-directory.md)
- [Autorización del acceso a recursos de Event Hubs mediante firmas de acceso compartido](authorize-access-shared-access-signature.md)
