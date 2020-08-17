---
title: Habilitación de la identidad administrada asignada por el sistema para una aplicación de Azure Spring Cloud
description: Cómo habilitar la identidad administrada asignada por el sistema para la aplicación.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 05/13/2020
ms.custom: devx-track-java
ms.openlocfilehash: 1b9d7326ec13176fbe65ba430a8a33bb93a48f74
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87091460"
---
# <a name="how-to-enable-system-assigned-managed-identity-for-azure-spring-cloud-application"></a>Habilitación de la identidad administrada asignada por el sistema para una aplicación de Azure Spring Cloud
Las identidades administradas para los recursos de Azure proporcionan una identidad administrada automática en Azure Active Directory a un recurso de Azure, como la aplicación de Azure Spring Cloud. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código.

En este artículo se muestra cómo habilitar y deshabilitar las identidades administradas asignadas por el sistema para una aplicación de Azure Spring Cloud mediante Azure Portal y la CLI (disponible en la versión 0.2.4).

## <a name="prerequisites"></a>Requisitos previos
Si no está familiarizado con las identidades administradas para recursos de Azure, consulte la [sección de introducción](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).
Necesitará una instancia de Azure Spring Cloud implementada. Siga el [Inicio rápido para realizar la implementación mediante la CLI de Azure](spring-cloud-quickstart-launch-app-cli.md).

## <a name="add-a-system-assigned-identity"></a>Adición de una identidad asignada por el sistema
Para crear una aplicación con una identidad asignada por el sistema es necesario configurar una propiedad adicional en la aplicación.

### <a name="using-azure-portal"></a>Uso de Azure Portal
Para configurar una identidad administrada en [Azure Portal](https://portal.azure.com/), primero cree una aplicación y luego habilite la característica.

1. Cree una aplicación en el portal como lo haría normalmente. Navegue hasta el portal.
2. Desplácese hacia abajo hasta el grupo **Configuración** en el panel de navegación izquierdo.
3. Seleccione **Identidad**.
4. En la pestaña **Asignado por el sistema**, cambie **Estado** a *Activado*. Haga clic en **Save**(Guardar).

 ![Identidad administrada en el portal](./media/spring-cloud-managed-identity/identity-1.png)

### <a name="using-azure-cli"></a>Uso de la CLI de Azure
Puede habilitar una identidad administrada asignada por el sistema durante la creación de una aplicación o en una existente.

**Habilitación de una identidad administrada asignada por el sistema durante la creación de una aplicación**

En el ejemplo siguiente, se crea una aplicación denominada *app_name* con una identidad administrada asignada por el sistema, como ha solicitado el parámetro `--assign-identity`.

```azurecli
az spring-cloud app create -n app_name -s service_name -g resource_group_name --assign-identity
```

**Habilitación de una identidad administrada asignada por el sistema en una aplicación existente** Use el comando `az spring-cloud app identity assign` para habilitar la identidad asignada por el sistema en una aplicación existente.

```azurecli
az spring-cloud app identity assign -n app_name -s service_name -g resource_group_name
```

## <a name="obtain-tokens-for-azure-resources"></a>Obtención de tokens para recursos de Azure
Una aplicación puede usar su identidad administrada para obtener tokens de acceso a otros recursos protegidos por Azure Active Directory, como Azure Key Vault. Estos tokens representan a la aplicación que accede al recurso, no a un usuario específico de la aplicación.

Es posible que deba [configurar el recurso de destino para permitir el acceso desde la aplicación](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/howto-assign-access-portal). Por ejemplo, si se solicita un token para acceder a Key Vault, asegúrese de haber agregado una directiva de acceso que incluya la identidad de la aplicación. De lo contrario, las llamadas a Key Vault se rechazarán, incluso si incluyen el token. Para más información sobre los recursos que admiten tokens de Azure Active Directory, consulte [Servicios de Azure que admiten autenticación de Azure AD](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/services-support-managed-identities#azure-services-that-support-azure-ad-authentication).

Azure Spring Cloud y la máquina virtual de Azure comparten el mismo punto de conexión para la adquisición de tokens. Se recomienda usar el SDK de Java o el iniciador de Spring Boot para adquirir un token.  Consulte [Cómo usar un token de máquina virtual](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token) para obtener diversos ejemplos de códigos y scripts, así como instrucciones sobre temas importantes como el control de errores HTTP y la expiración de tokens.

Recomendado: use el SDK de Java o los iniciadores de Spring Boot para obtener tokens.  Consulte los ejemplos de los [pasos siguientes](#next-steps).

## <a name="disable-system-assigned-identity-from-an-app"></a>Deshabilitación de una identidad asignada por el sistema desde una aplicación
Al quitar una identidad asignada por el sistema, también se eliminará de Azure AD. Cuando se elimina el recurso de la aplicación, las identidades asignadas por el sistema se quitan automáticamente de Azure AD.

### <a name="using-azure-portal"></a>Uso de Azure Portal
Para quitar una identidad administrada asignada por el sistema de una aplicación que ya no la necesita, haga lo siguiente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con una cuenta asociada a la suscripción a Azure que contiene la instancia de Azure Spring Cloud.
1. Vaya a la máquina virtual que desee y seleccione la página **Identidad**.
1. En **Asignado por el sistema**/**Estado**, seleccione **Desactivado** y haga clic en **Guardar**:

 ![Identidad administrada en el portal](./media/spring-cloud-managed-identity/remove-identity.png)

### <a name="using-azure-cli"></a>Uso de la CLI de Azure
Para quitar la identidad administrada asignada por el sistema de una aplicación que ya no la necesite, use el siguiente comando:
```azurecli
az spring-cloud app identity remove -n app_name -s service_name -g resource_group_name
```

## <a name="next-steps"></a>Pasos siguientes
* [Cómo usar identidades administradas con el SDK de Java](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples)
* [Acceso a Azure Key Vault con identidades administradas en el iniciador de Spring Boot](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/spring/azure-spring-boot-starter-keyvault-secrets/README.md#use-msi--managed-identities)
* [Más información sobre las identidades administradas para recursos de Azure](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/active-directory/managed-identities-azure-resources/overview.md)

