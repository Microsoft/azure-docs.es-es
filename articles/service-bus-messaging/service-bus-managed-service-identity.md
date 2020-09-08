---
title: Identidades administradas para recursos de Azure con Service Bus
description: En este artículo se describe cómo usar identidades administradas para obtener acceso a entidades de Azure Service Bus (colas, temas y suscripciones).
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 1deb3bdf823f1554e302bb35baabe444223f9008
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88079865"
---
# <a name="authenticate-a-managed-identity-with-azure-active-directory-to-access-azure-service-bus-resources"></a>Autenticación de una identidad administrada con Azure Active Directory para acceder a recursos de Azure Service Bus
[Identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md) es una característica de Azure que permite crear una identidad segura asociada a la implementación en la que se ejecuta el código de la aplicación. A continuación, puede asociar esa identidad con los roles de control de acceso que conceden permisos personalizados para acceder a recursos específicos de Azure que la aplicación necesita.

Con las identidades administradas, la plataforma Azure administra esta identidad en tiempo de ejecución. No es necesario almacenar y proteger las claves de acceso en la configuración o el código de la aplicación, ya sea para la propia identidad o para los recursos a los que necesita acceder. Una aplicación cliente de Service Bus que se ejecuta dentro de una aplicación de Azure App Service o en una máquina virtual con entidades administradas habilitadas para admitir recursos de Azure no necesita controlar las reglas y las claves SAS ni cualquier otro token de acceso. La aplicación cliente solo necesita la dirección del punto de conexión del espacio de nombres de mensajería de Service Bus. Cuando se conecta la aplicación, Service Bus enlaza el contexto de la entidad administrada con el cliente en una operación que se muestra en un ejemplo más adelante en este artículo. Una vez creada la asociación con una identidad administrada, el cliente de Service Bus puede realizar todas las operaciones autorizadas. La autorización se concede mediante la asociación de una entidad administrada con roles de Service Bus. 

## <a name="overview"></a>Información general
Cuando una entidad de seguridad (un usuario, un grupo o una aplicación) intenta acceder a una entidad de Service Bus, la solicitud debe estar autorizada. Con Azure AD, el acceso a un recurso es un proceso de dos pasos. 

 1. En primer lugar, se autentica la identidad de la entidad de seguridad y se devuelve un token de OAuth 2.0. El nombre del recurso para solicitar un token es `https://servicebus.azure.net`.
 1. Luego, el token se pasa como parte de una solicitud al servicio Service Bus para autorizar el acceso al recurso especificado.

El paso de autenticación exige que una solicitud de aplicación contenga un token de acceso de OAuth 2.0 en tiempo de ejecución. Si una aplicación se está ejecutando dentro de una entidad de Azure, como puede ser una máquina virtual de Azure, un conjunto de escalado de máquinas virtuales o una aplicación de Azure Functions, puede usar una identidad administrada para acceder a los recursos. 

El paso de la autorización exige que se asignen uno o varios roles de Azure a la entidad de seguridad. Azure Service Bus proporciona roles de Azure que abarcan conjuntos de permisos para recursos de Service Bus. Los roles que se asignan a una entidad de seguridad determinan los permisos que tiene esa entidad de seguridad. Para más información sobre la asignación de roles de Azure a Azure Service Bus, consulte [Roles integrados de Azure para Azure Service Bus](#azure-built-in-roles-for-azure-service-bus). 

Las aplicaciones nativas y las aplicaciones web que realizan solicitudes a Service Bus también pueden autorizar con Azure AD. En este artículo se muestra cómo solicitar un token de acceso y usarlo para autorizar solicitudes de recursos de Service Bus. 


## <a name="assigning-azure-roles-for-access-rights"></a>Asignación de roles de Azure para derechos de acceso
Azure Active Directory (Azure AD) autoriza derechos de acceso a recursos protegidos mediante el [control de acceso basado en rol de Azure (RBAC de Azure)](../role-based-access-control/overview.md). Azure Service Bus define un conjunto de roles integrados de Azure que abarcan conjuntos de permisos comunes utilizados para acceder a las entidades Service Bus y también puede definir roles personalizados para acceder a los datos.

Cuando un rol de Azure se asigna a una entidad de seguridad de Azure AD, Azure concede a esa entidad de seguridad acceso a los recursos. El acceso puede tener como ámbito el nivel de suscripción, el grupo de recursos o el espacio de nombres de Service Bus. Una entidad de seguridad de Azure AD puede ser un usuario, un grupo, una entidad de servicio de aplicación o una identidad de servicio administrada para recursos de Azure.

## <a name="azure-built-in-roles-for-azure-service-bus"></a>Roles integrados de Azure para Azure Service Bus
En el caso de Azure Service Bus, la administración de los espacios de nombres y de todos los recursos relacionados mediante Azure Portal y la API de administración de recursos de Azure, ya se ha protegido mediante el modelo de Azure RBAC. Azure proporciona los siguientes roles integrados de Azure para autorizar el acceso a un espacio de nombres de Service Bus:

- [Propietario de los datos de Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-owner): permite el acceso a datos en el espacio de nombres de Service Bus y sus entidades (colas, temas, suscripciones y filtros).
- [Emisor de datos de Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-sender): use este rol para proporcionar acceso de envío al espacio de nombres de Service Bus y sus entidades.
- [Receptor de datos de Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-receiver): use este rol para proporcionar acceso de recepción al espacio de nombres de Service Bus y sus entidades. 

## <a name="resource-scope"></a>Ámbito de recursos 
Antes de asignar un rol de Azure a una entidad de seguridad, determine el ámbito de acceso que debería tener la entidad de seguridad. Los procedimientos recomendados dictan que siempre es mejor conceder únicamente el ámbito más restringido posible.

En la lista siguiente se describen los niveles en los que puede definir el ámbito de acceso a recursos Service Bus, empezando por el ámbito más restringido:

- **Cola**, **tema** o **suscripción**: la asignación de roles se aplica a la entidad de Service Bus específica. Actualmente, Azure Portal no admite la asignación de usuarios, grupos o identidades administradas a los roles de Azure de Service Bus en el nivel de suscripción. Este es un ejemplo del uso del comando de la CLI de Azure [az-role-assignment-create](/cli/azure/role/assignment?view=azure-cli-latest#az-role-assignment-create) para asignar una identidad a un rol de Azure de Service Bus: 

    ```azurecli
    az role assignment create \
        --role $service_bus_role \
        --assignee $assignee_id \
        --scope /subscriptions/$subscription_id/resourceGroups/$resource_group/providers/Microsoft.ServiceBus/namespaces/$service_bus_namespace/topics/$service_bus_topic/subscriptions/$service_bus_subscription
    ```
- **Espacio de nombres de Service Bus**: la asignación de roles abarca toda la topología de Service Bus en el espacio de nombres y el grupo de consumidores que tiene asociado.
- **Grupo de recursos**: la asignación de roles se aplica a todos los recursos de Service Bus del grupo de recursos.
- **Suscripción**: la asignación de roles se aplica a todos los recursos de Service Bus de todos los grupos de recursos de la suscripción.

> [!NOTE]
> Tenga en cuenta que las asignaciones de roles de Azure pueden tardar hasta cinco minutos en propagarse. 

Para más información sobre cómo se definen los roles integrados, consulte [Descripción de definiciones de roles](../role-based-access-control/role-definitions.md#management-and-data-operations). Para más información acerca de la creación de roles personalizados de Azure, consulte [Roles personalizados de Azure](../role-based-access-control/custom-roles.md).

## <a name="enable-managed-identities-on-a-vm"></a>Habilitación de identidades administradas en una máquina virtual
Para poder usar identidades administradas para recursos de Azure a fin de autorizar los recursos de Service Bus desde la máquina virtual, primero debe habilitar las identidades administradas para los recursos de Azure en la máquina virtual. Para aprender a habilitar las identidades administradas para los recursos de Azure, consulte uno de estos artículos:

- [Azure Portal](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
- [Azure PowerShell](../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [CLI de Azure](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Plantilla de Azure Resource Manager](../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Bibliotecas cliente de Azure Resource Manager](../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

## <a name="grant-permissions-to-a-managed-identity-in-azure-ad"></a>Concesión de permisos a una identidad administrada en Azure AD
Para autorizar una solicitud al servicio Service Bus desde una identidad administrada de la aplicación, primero configure los valores del control de acceso basado en rol de Azure (RBAC de Azure) de esa identidad administrada. Azure Service Bus define roles de Azure que abarcan los permisos para enviar y leer desde Service Bus. Cuando el rol de Azure se asigna a una identidad administrada, a esta se le concede acceso a las entidades de Service Bus en el ámbito adecuado.

Para más información sobre la asignación de roles de Azure, consulte [Autenticación y autorización con Azure Active Directory para el acceso a recursos de Service Bus](authenticate-application.md#azure-built-in-roles-for-azure-service-bus).

## <a name="use-service-bus-with-managed-identities-for-azure-resources"></a>Usar Services Bus con identidades administradas para recursos de Azure
Para usar Service Bus con identidades administradas, debe asignar la identidad del rol y el ámbito adecuado. En el procedimiento de esta sección se usa una aplicación sencilla que se ejecuta en una identidad administrada y accede a los recursos de Service Bus.

A continuación, usaremos una aplicación web de ejemplo hospedada en [Azure App Service](https://azure.microsoft.com/services/app-service/). Para instrucciones paso a paso sobre cómo crear una aplicación web, consulte [Creación de una aplicación web de ASP.NET Core en Azure](../app-service/quickstart-dotnetcore.md).

Una vez creada la aplicación, siga estos pasos: 

1. Vaya a **Configuración** y seleccione **Identidad**. 
1. Seleccione el **Estado** que va a estar **Activado**. 
1. Seleccione **Guardar** para guardar la configuración. 

    ![Identidad administrada para una aplicación web](./media/service-bus-managed-service-identity/identity-web-app.png)

Una vez habilitada esta configuración, se crea una identidad de servicio en Azure Active Directory (Azure AD) y se configura en el host de App Service.

Ahora, asigne esta identidad de servicio a un rol en el ámbito requerido en los recursos de Service Bus.

### <a name="to-assign-azure-roles-using-the-azure-portal"></a>Para asignar roles de Azure mediante Azure Portal
Para asignar un rol a un espacio de nombres de Service Bus, vaya al espacio de nombres en Azure Portal. Acceda a la configuración Access Control (IAM) del recurso y siga estas instrucciones para administrar las asignaciones de roles:

> [!NOTE]
> En los pasos siguientes se asigna un rol de identidad de servicio a los espacios de nombres de Service Bus. Puede seguir los mismos pasos para asignar un rol en otros ámbitos admitidos (grupo de recursos y suscripción). 
> 
> [Cree un espacio de nombres de mensajería de Service Bus](service-bus-create-namespace-portal.md) si no tiene uno. 

1. En Azure Portal, vaya al espacio de nombres de Service Bus y muestre la **información general** del espacio de nombres. 
1. Seleccione **Access Control (IAM)** en el menú de la izquierda para mostrar la configuración de control de acceso para el espacio de nombres de Service Bus.
1.  Seleccione la pestaña **Asignaciones de roles** para ver la lista de asignaciones de roles.
3.  Seleccione **Agregar** para agregar un rol nuevo.
4.  En la página **Agregar asignación de roles**, seleccione los roles de Azure Service Bus que quiere asignar. A continuación, busque la identidad del servicio que ha registrado para asignar el rol.
    
    ![Página Agregar asignación de roles](./media/service-bus-managed-service-identity/add-role-assignment-page.png)
5.  Seleccione **Guardar**. La identidad a la que ha asignado el rol aparece enumerada debajo de ese rol. Por ejemplo, la siguiente imagen muestra que la identidad del servicio tiene al propietario de los datos de Azure Service Bus.
    
    ![Identidad asignada a un rol](./media/service-bus-managed-service-identity/role-assigned.png)

Una vez que haya asignado el rol, la aplicación web tendrá acceso a las entidades de Service Bus en el ámbito definido. 

### <a name="run-the-app"></a>Ejecución la aplicación

Ahora, modifique la página predeterminada de la aplicación de ASP.NET que ha creado. Puede utilizar el código de la aplicación web de [este repositorio de GitHub](https://github.com/Azure-Samples/app-service-msi-servicebus-dotnet).  

La página Default.aspx es su página de destino. El código puede encontrarse en el archivo Default.aspx.cs. El resultado es una aplicación web básica con unos cuantos campos de entrada y con los botones **enviar** y **recibir** que se conectan a Service Bus para enviar o recibir mensajes.

Tenga en cuenta cómo se inicializa el objeto [MessagingFactory](/dotnet/api/microsoft.servicebus.messaging.messagingfactory). En lugar de usar el proveedor de tokens de Token de acceso compartido (SAS), el código crea un proveedor de tokens para la identidad administrada con la llamada a `var msiTokenProvider = TokenProvider.CreateManagedIdentityTokenProvider();`. Por lo tanto, no hay ningún secreto para conservar y usar. El proveedor de tokens controla automáticamente el flujo del contexto de la identidad administrada para Service Bus y el protocolo de enlace de autorización. Es un modelo más sencillo que si se usa SAS.

Una vez realizados estos cambios, publique y ejecute la aplicación. Puede obtener fácilmente los datos de publicación correctos mediante la descarga y posterior importación de un perfil de publicación en Visual Studio:

![Obtener perfil de publicación](./media/service-bus-managed-service-identity/msi3.png)
 
Para enviar o recibir mensajes, escriba el nombre del espacio de nombres y el nombre de la entidad que ha creado. Después, haga clic en **Enviar** o **Recibir**.


> [!NOTE]
> - La identidad administrada solo funciona en el entorno de Azure, en servicios de aplicaciones, en máquinas virtuales de Azure y en conjuntos de escalado. En el caso de las aplicaciones de .NET, la biblioteca Microsoft.Azure.Services.AppAuthentication, que es la que usa el paquete NuGet de Service Bus, proporciona una abstracción sobre este protocolo y admite una experiencia de desarrollo local. Esta biblioteca también permite probar el código localmente en una máquina de desarrollo, con su cuenta de usuario de Visual Studio, la CLI de Azure 2.0 o la autenticación integrada de Active Directory. Para más información acerca de las opciones de desarrollo local con esta biblioteca, consulte [Autenticación entre servicios en Azure Key Vault mediante .NET](../key-vault/general/service-to-service-authentication.md).  
> 
> - Actualmente, las identidades administradas no funcionan con las ranuras de implementación de App Service.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la mensajería de Service Bus, consulte los siguientes temas:

* [Colas, temas y suscripciones de Service Bus](service-bus-queues-topics-subscriptions.md)
* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Uso de temas y suscripciones de Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)