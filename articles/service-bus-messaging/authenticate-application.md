---
title: Autenticación de una aplicación para acceder a entidades de Azure Service Bus
description: En este artículo se proporciona información sobre cómo autenticar una aplicación con Azure Active Directory para acceder a entidades de Azure Service Bus (colas, temas, etc.).
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: b12f2f294a66159a7035240c361ab93f9f84718e
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88064832"
---
# <a name="authenticate-and-authorize-an-application-with-azure-active-directory-to-access-azure-service-bus-entities"></a>Autenticación y autorización de una aplicación con Azure Active Directory para acceder a entidades de Azure Service Bus
Azure Service Bus admite el uso de Azure Active Directory (Azure AD) para autorizar solicitudes a entidades de Service Bus (colas, temas, suscripciones o filtros). Con Azure AD, puede usar el control de acceso basado en rol de Azure (Azure RBAC) para conceder permisos a una entidad de seguridad, que puede ser un usuario, un grupo o una entidad de servicio de aplicación. Para más información sobre los roles y las asignaciones de roles, consulte [Descripción de los distintos roles](../role-based-access-control/overview.md).

## <a name="overview"></a>Información general
Cuando una entidad de seguridad (un usuario, un grupo o una aplicación) intenta acceder a una entidad de Service Bus, la solicitud debe estar autorizada. Con Azure AD, el acceso a un recurso es un proceso de dos pasos. 

 1. En primer lugar, se autentica la identidad de la entidad de seguridad y se devuelve un token de OAuth 2.0. El nombre del recurso para solicitar un token es `https://servicebus.azure.net`.
 1. Luego, el token se pasa como parte de una solicitud al servicio Service Bus para autorizar el acceso al recurso especificado.

El paso de autenticación exige que una solicitud de aplicación contenga un token de acceso de OAuth 2.0 en tiempo de ejecución. Si una aplicación se está ejecutando dentro de una entidad de Azure, como puede ser una máquina virtual de Azure, un conjunto de escalado de máquinas virtuales o una aplicación de Azure Functions, puede usar una identidad administrada para acceder a los recursos. Para más información sobre cómo autenticar solicitudes realizadas por una identidad administrada al servicio Service Bus, consulte [Identidades administradas para recursos de Azure con Service Bus](service-bus-managed-service-identity.md). 

El paso de la autorización exige que se asignen uno o varios roles de Azure a la entidad de seguridad. Azure Service Bus proporciona roles de Azure que abarcan conjuntos de permisos para recursos de Service Bus. Los roles que se asignan a una entidad de seguridad determinan los permisos que tiene esa entidad de seguridad. Para más información sobre la asignación de roles de Azure a Azure Service Bus, consulte [Roles integrados de Azure para Azure Service Bus](#azure-built-in-roles-for-azure-service-bus). 

Las aplicaciones nativas y las aplicaciones web que realizan solicitudes a Service Bus también pueden autorizar con Azure AD. En este artículo se muestra cómo solicitar un token de acceso y usarlo para autorizar solicitudes de recursos de Service Bus. 


## <a name="assigning-azure-roles-for-access-rights"></a>Asignación de roles de Azure para derechos de acceso
Azure Active Directory (Azure AD) concede derechos de acceso a recursos protegidos mediante [Azure RBAC](../role-based-access-control/overview.md). Azure Service Bus define un conjunto de roles integrados de Azure que abarcan conjuntos de permisos comunes utilizados para acceder a las entidades Service Bus y también puede definir roles personalizados para acceder a los datos.

Cuando un rol de Azure se asigna a una entidad de seguridad de Azure AD, Azure concede a esa entidad de seguridad acceso a los recursos. El acceso puede tener como ámbito el nivel de suscripción, el grupo de recursos o el espacio de nombres de Service Bus. Una entidad de seguridad de Azure AD puede ser un usuario, un grupo, una entidad de servicio de aplicación o una [identidad de servicio administrada para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md).

## <a name="azure-built-in-roles-for-azure-service-bus"></a>Roles integrados de Azure para Azure Service Bus
En el caso de Azure Service Bus, la administración de los espacios de nombres y de todos los recursos relacionados mediante Azure Portal y la API de administración de recursos de Azure, ya se ha protegido mediante el modelo de Azure RBAC. Azure proporciona los siguientes roles integrados de Azure para autorizar el acceso a un espacio de nombres de Service Bus:

- [Propietario de los datos de Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-owner): permite el acceso a datos en el espacio de nombres de Service Bus y sus entidades (colas, temas, suscripciones y filtros).
- [Emisor de datos de Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-sender): use este rol para proporcionar acceso de envío al espacio de nombres de Service Bus y sus entidades.
- [Receptor de datos de Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-receiver): use este rol para proporcionar acceso de recepción al espacio de nombres de Service Bus y sus entidades. 

## <a name="resource-scope"></a>Ámbito de recursos 
Antes de asignar un rol de Azure a una entidad de seguridad, determine el ámbito de acceso que debería tener la entidad de seguridad. Los procedimientos recomendados dictan que siempre es mejor conceder únicamente el ámbito más restringido posible.

En la lista siguiente se describen los niveles en los que puede definir el ámbito de acceso a recursos Service Bus, empezando por el ámbito más restringido:

- **Cola**, **tema** o **suscripción**: la asignación de roles se aplica a la entidad de Service Bus específica. Actualmente, Azure Portal no admite la asignación de usuarios, grupos o identidades administradas a los roles de Azure de Service Bus en el nivel de suscripción. 
- **Espacio de nombres de Service Bus**: la asignación de roles abarca toda la topología de Service Bus en el espacio de nombres y el grupo de consumidores que tiene asociado.
- **Grupo de recursos**: la asignación de roles se aplica a todos los recursos de Service Bus del grupo de recursos.
- **Suscripción**: la asignación de roles se aplica a todos los recursos de Service Bus de todos los grupos de recursos de la suscripción.

> [!NOTE]
> Tenga en cuenta que las asignaciones de roles de Azure pueden tardar hasta cinco minutos en propagarse. 

Para más información sobre cómo se definen los roles integrados, consulte [Descripción de definiciones de roles](../role-based-access-control/role-definitions.md#management-and-data-operations). Para más información acerca de la creación de roles personalizados de Azure, consulte [Roles personalizados de Azure](../role-based-access-control/custom-roles.md).


## <a name="assign-azure-roles-using-the-azure-portal"></a>Asignación de roles de Azure mediante Azure Portal  
Para más información sobre cómo administrar el acceso a los recursos de Azure con Azure RBAC y Azure Portal, consulte [este artículo](..//role-based-access-control/role-assignments-portal.md). 

Después de determinar el ámbito adecuado de una asignación de roles, vaya a ese recurso en Azure Portal. Acceda a la configuración Control de acceso (IAM) del recurso y siga estas instrucciones para administrar las asignaciones de roles:

> [!NOTE]
> Los pasos que se describen a continuación asignan un rol a su espacio de nombres de Service Bus. Puede seguir los mismos pasos para asignar un rol a otros ámbitos admitidos (grupo de recursos, suscripción, etc.).

1. En [Azure Portal](https://portal.azure.com/), vaya al espacio de nombres de Service Bus. Seleccione **Control de acceso (IAM)** en el menú de la izquierda para mostrar la configuración de control de acceso para el espacio de nombres. Si necesita crear un espacio de nombres de Service Bus, siga las instrucciones de este artículo: [Creación de un espacio de nombres de Service Bus mediante Azure Portal](service-bus-create-namespace-portal.md).

    ![Selección de Control de acceso en el menú izquierdo](./media/authenticate-application/select-access-control-menu.png)
1. Seleccione la pestaña **Asignaciones de roles** para ver la lista de asignaciones de roles. Seleccione el botón **Agregar** de la barra de herramientas y luego seleccione **Agregar asignación de roles**. 

    ![Botón Agregar de la barra de herramientas](./media/authenticate-application/role-assignments-add-button.png)
1. En la página **Agregar asignación de roles**, siga estos pasos:
    1. Seleccione el **rol de Service Bus** que quiere asignar. 
    1. Realice una búsqueda para localizar la **entidad de seguridad** (usuario, grupo, entidad de servicio) a la que quiere asignar el rol.
    1. Seleccione **Guardar** para guardar la asignación de roles. 

        ![Asignación de un rol a un usuario](./media/authenticate-application/assign-role-to-user.png)
    4. La identidad a la que ha asignado el rol aparece enumerada debajo de ese rol. Por ejemplo, en la imagen siguiente se muestra que Azure-users está en el rol Propietario de los datos de Azure Service Bus. 
        
        ![Usuario en la lista](./media/authenticate-application/user-in-list.png)

Puede realizar este mismo procedimiento para asignar un rol que tenga como ámbito un grupo de recursos o una suscripción. Una vez que defina el rol y su ámbito, puede probar este comportamiento con los [ejemplos de GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/RoleBasedAccessControl).


## <a name="authenticate-from-an-application"></a>Autenticación desde una aplicación
Una ventaja clave del uso de Azure AD con Service Bus es que ya no necesita almacenar las credenciales en el código. En su lugar, puede solicitar un token de acceso de OAuth 2.0 desde la Plataforma de identidad de Microsoft. Azure AD autentica la entidad de seguridad (un usuario, grupo o entidad de servicio) ejecutando la aplicación. Si la autenticación se realiza correctamente, Azure AD devuelve el token de acceso a la aplicación y la aplicación puede entonces usar dicho token para autorizar las solicitudes de Service Bus.

En las secciones siguientes se muestra cómo configurar su aplicación web o aplicación nativa para la autenticación con la Plataforma de identidad de Microsoft 2.0. Para obtener más información sobre la Plataforma de identidad de Microsoft 2.0, consulte la [Introducción a la Plataforma de identidad de Microsoft (v2.0)](../active-directory/develop/v2-overview.md).

Para información general sobre el flujo de concesión de código OAuth 2.0, consulte [Autorización del acceso a aplicaciones web de Azure Active Directory mediante el flujo de concesión de código OAuth 2.0](../active-directory/develop/v2-oauth2-auth-code-flow.md).

### <a name="register-your-application-with-an-azure-ad-tenant"></a>Registro de la aplicación con un inquilino de Azure AD
El primer paso para usar Azure AD con el fin de autorizar las entidades de Service Bus es registrar la aplicación cliente con un inquilino de Azure AD desde [Azure Portal](https://portal.azure.com/). Al registrar la aplicación cliente, facilita información sobre la aplicación a AD. Azure AD proporciona un id. de cliente (también denominado id. de aplicación) que se puede usar para asociar la aplicación con el runtime de Azure AD. Para más información sobre el identificador de cliente, consulte [Objetos de aplicación y de entidad de servicio de Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md). 

En las imágenes siguientes se muestran los pasos para registrar una aplicación web:

![Registro de una aplicación](./media/authenticate-application/app-registrations-register.png)

> [!Note]
> Si registra la aplicación como una aplicación nativa, puede especificar cualquier URI válido para el URI de redirección. Para las aplicaciones nativas, no es necesario que este valor sea una dirección URL real. Para las aplicaciones web, el URI de redirección debe ser un URI válido, ya que especifica la dirección URL a la que se proporcionan los tokens.

Una vez que ha registrado su aplicación, verá el **id. de la aplicación (o id. de cliente)** en **Configuración**:

![Id. de aplicación de la aplicación registrada](./media/authenticate-application/application-id.png)

Para más información sobre cómo registrar una aplicación con Azure AD, consulte [Integración de aplicaciones con Azure Active Directory](../active-directory/develop/quickstart-register-app.md).

> [!IMPORTANT]
> Anote los valores **TenantId** y **ApplicationId**. Necesitará estos valores para ejecutar la aplicación.

### <a name="create-a-client-secret"></a>Creación de un secreto de cliente   
La aplicación necesita un secreto de cliente para demostrar su identidad al solicitar un token. Para agregar el secreto de cliente, siga estos pasos.

1. Vaya a su registro de aplicaciones en Azure Portal si todavía no se encuentra en la página.
1. Seleccione **Certificados y secretos** en el panel izquierdo.
1. En **Secretos de cliente**, seleccione **Nuevo secreto de cliente** para crear un nuevo secreto.

    ![Nuevo secreto de cliente, botón](./media/authenticate-application/new-client-secret-button.png)
1. Proporcione una descripción para el secreto, elija el intervalo de expiración deseado y, a continuación, seleccione **Agregar**.

    ![Página para agregar un secreto de cliente](./media/authenticate-application/add-client-secret-page.png)
1. Copie inmediatamente el valor del nuevo secreto en una ubicación segura. El valor completo se le muestra solo una vez.

    ![Secreto del cliente](./media/authenticate-application/client-secret.png)

### <a name="permissions-for-the-service-bus-api"></a>Permisos para la API de Service Bus
Si la aplicación de ejemplo es una aplicación de consola, debe registrar una aplicación nativa y agregar permisos de API para **Microsoft.ServiceBus** en el conjunto de **permisos necesarios**. Las aplicaciones nativas también necesitan un **URI de redireccionamiento** en Azure AD, que actúa como identificador. No es necesario que este URI sea un destino de red. Use `https://servicebus.microsoft.com` para este ejemplo, dado que el ejemplo de código ya utiliza ese URI.

### <a name="client-libraries-for-token-acquisition"></a>Bibliotecas cliente para la adquisición de tokens  
Una vez que haya registrado su aplicación y le haya concedido permisos para enviar/recibir datos en Service Bus, podrá agregar código a su aplicación para autenticar una entidad de seguridad y adquirir un token de OAuth 2.0. Para autenticar y adquirir el token, puede usar una de las [bibliotecas de autenticación de la Plataforma de identidad de Microsoft](../active-directory/develop/reference-v2-libraries.md) u otra biblioteca de código abierto que admita OpenID o Connect 1.0. A continuación, su aplicación puede usar el token de acceso para autorizar una solicitud en Azure Service Bus.

Para ver una lista de escenarios en los que se admite la adquisición de tokens, consulte la sección [Escenarios](https://aka.ms/msal-net-scenarios) del repositorio de GitHub [Biblioteca de autenticación de Microsoft (MSAL) para .NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet).

## <a name="sample-on-github"></a>Ejemplo en GitHub
Consulte el siguiente ejemplo en GitHub: [Control de acceso de base de rol para Service Bus](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/RoleBasedAccessControl). 

Use la opción **Client Secret Login (Inicio de sesión de secreto de usuario)** , no la opción **Interactive User Login** (Inicio de sesión de usuario interactivo). Cuando se usa la opción de secreto de cliente, no se ve una ventana emergente. La aplicación emplea el identificador de inquilino y el identificador de aplicación para la autenticación. 

### <a name="run-the-sample"></a>Ejecución del ejemplo

Antes de poder ejecutar el ejemplo, edite el archivo **app.config** y, según el escenario, establezca los siguientes valores:

- `tenantId`: establézcalo en el valor de **TenantId**.
- `clientId`: establézcalo en el valor de **ApplicationId**.
- `clientSecret`: si desea iniciar sesión mediante el secreto de cliente, créelo en Azure AD. Además, utilice una aplicación web o API en lugar de una aplicación nativa. Además, agregue la aplicación en **Control de acceso (IAM)** en el espacio de nombres que creó anteriormente.
- `serviceBusNamespaceFQDN`: establézcalo en el nombre DNS completo del espacio de nombres de Service Bus recién creado. Por ejemplo, `example.servicebus.windows.net`.
- `queueName`: establézcalo en el nombre de la cola que creó.
- El URI de redireccionamiento que especificó en la aplicación en los pasos anteriores.

Al ejecutar la aplicación de consola, se le pedirá que seleccione un escenario. Seleccione **Interactive User Login** (Inicio de sesión de usuario interactivo); para ello, escriba su número y presione ENTRAR. La aplicación muestra una ventana de inicio de sesión, solicita su consentimiento para acceder a Service Bus y, a continuación, utiliza el servicio para ejecutarse en el escenario de envío o recepción mediante la identidad de inicio de sesión.


## <a name="next-steps"></a>Pasos siguientes
- Para más información sobre Azure RBAC, consulte [¿Qué es el control de acceso basado en rol de Azure (Azure RBAC) de Azure?](../role-based-access-control/overview.md)
- Para información sobre cómo asignar y administrar las asignaciones de roles de Azure con Azure PowerShell, la CLI de Azure o la API de REST, consulte estos artículos:
    - [Incorporación o eliminación de asignaciones de roles de Azure con Azure PowerShell](../role-based-access-control/role-assignments-powershell.md)  
    - [Incorporación o eliminación de asignaciones de roles de Azure mediante la CLI de Azure](../role-based-access-control/role-assignments-cli.md)
    - [Incorporación o eliminación de asignaciones de roles de Azure mediante la API REST](../role-based-access-control/role-assignments-rest.md)
    - [Incorporación o eliminación de asignaciones de roles de Azure mediante plantillas de Azure Resource Manager](../role-based-access-control/role-assignments-template.md)

Para obtener más información sobre la mensajería de Service Bus, consulte los siguientes temas:

- [Ejemplos de Azure RBAC de Service Bus](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/RoleBasedAccessControl)
- [Colas, temas y suscripciones de Service Bus](service-bus-queues-topics-subscriptions.md)
- [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
- [Uso de temas y suscripciones de Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)