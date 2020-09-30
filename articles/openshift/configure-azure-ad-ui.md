---
title: 'Red Hat OpenShift en Azure con OpenShift 4: configuración de la autenticación de Azure Active Directory mediante Azure Portal y la consola web de OpenShift'
description: Obtenga información sobre cómo configurar la autenticación de Azure Active Directory para un clúster de Red Hat OpenShift en Azure que ejecute OpenShift 4 mediante Azure Portal y la consola web de OpenShift.
ms.service: container-service
ms.topic: article
ms.date: 03/12/2020
author: sabbour
ms.author: asabbour
keywords: aro, openshift, az aro, red hat, cli
ms.custom: mvc
ms.openlocfilehash: 2cb54c202af04996080cda970b3d327145f0e72b
ms.sourcegitcommit: 4feb198becb7a6ff9e6b42be9185e07539022f17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89469888"
---
# <a name="configure-azure-active-directory-authentication-for-an-azure-red-hat-openshift-4-cluster-portal"></a>Configuración de la autenticación de Azure Active Directory para un clúster de Red Hat OpenShift en Azure 4 (Portal)

Si decide instalar y usar la CLI de forma local, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.6.0, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="before-you-begin"></a>Antes de empezar

Construya la **dirección URL de devolución de llamada de OAuth** del clúster y anótela. Asegúrese de reemplazar **aro-rg** por el nombre del grupo de recursos y **aro-cluster** por el nombre del clúster.

> [!NOTE]
> La sección `AAD` de la dirección URL de devolución de llamada de OAuth debe coincidir con el nombre del proveedor de identidades de OAuth que se va a configurar más adelante.

```azurecli-interactive
domain=$(az aro show -g aro-rg -n aro-cluster --query clusterProfile.domain -o tsv)
location=$(az aro show -g aro-rg -n aro-cluster --query location -o tsv)
echo "OAuth callback URL: https://oauth-openshift.apps.$domain.$location.aroapp.io/oauth2callback/AAD"
```

## <a name="create-an-azure-active-directory-application-for-authentication"></a>Creación de una aplicación de Azure Active Directory para autenticación

Inicie sesión en Azure Portal, vaya a la hoja [Registros de aplicaciones](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) y, a continuación, haga clic en **Nuevo registro** para crear una nueva aplicación.

Proporcione un nombre para la aplicación, por ejemplo **aro-azuread-auth** y rellene el **URI de redirección** con el valor de la dirección URL de devolución de llamada de OAuth que recuperó anteriormente.

![Nuevo registro de aplicaciones](media/aro4-ad-registerapp.png)

Vaya a **Certificados y secretos**, haga clic en **Nuevo secreto de cliente** y rellene los detalles. Anote el valor de la clave, ya que lo usará en una fase posterior. No podrá recuperarlo de nuevo.

![Crear un secreto](media/aro4-ad-clientsecret.png)

Vaya a la **Información general** y tome nota del **Id. de la aplicación (cliente)** y el **Id. de directorio (inquilino)** . Los necesitará en una fase posterior.

![Recuperar los identificadores de aplicación (cliente) y directorio (inquilino)](media/aro4-ad-ids.png)

## <a name="configure-optional-claims"></a>Configuración de notificaciones opcionales

Los desarrolladores de aplicaciones pueden usar [notificaciones opcionales](../active-directory/develop/active-directory-optional-claims.md) en sus aplicaciones de Azure AD para especificar qué notificaciones quieren incluir en los tokens que se envían a las aplicaciones.

Estas notificaciones opcionales sirven para:

* Seleccionar las notificaciones adicionales que se incluirán en los tokens para la aplicación.
* Cambiar el comportamiento de ciertas notificaciones que Azure AD devuelve en tokens.
* Agregar notificaciones personalizadas para la aplicación y acceder a ellas.

Se va a configurar OpenShift para usar la notificación `email` y revertir a `upn` para establecer el nombre de usuario preferido agregando `upn` como parte del token de identificador devuelto por Azure Active Directory.

Vaya a **Configuración de token (versión preliminar)** y haga clic en **Agregar notificación opcional**. Seleccione **Id.** y, a continuación, compruebe las notificaciones de **correo** y **UPN**.

![Crear un secreto](media/aro4-ad-tokens.png)

## <a name="assign-users-and-groups-to-the-cluster-optional"></a>Asignación de usuarios y grupos al clúster (opcional)

Las aplicaciones registradas en un inquilino de Azure Active Directory (Azure AD) están disponibles de forma predeterminada para todos los usuarios del inquilino que se autentica correctamente. Azure AD permite a los administradores y desarrolladores de inquilinos restringir una aplicación a un conjunto específico de usuarios o a los grupos de seguridad del inquilino.

Siga las instrucciones de la documentación de Azure Active Directory para [asignar usuarios y grupos a la aplicación](../active-directory/develop/howto-restrict-your-app-to-a-set-of-users.md#app-registration).

## <a name="configure-openshift-openid-authentication"></a>Configuración de la autenticación OpenID en OpenShift

Recupere las credenciales de `kubeadmin`. Ejecute el siguiente comando para buscar la contraseña del usuario `kubeadmin`.

```azurecli-interactive
az aro list-credentials \
  --name aro-cluster \
  --resource-group aro-rg
```

En la siguiente salida de ejemplo se muestra que la contraseña estará en `kubeadminPassword`.

```json
{
  "kubeadminPassword": "<generated password>",
  "kubeadminUsername": "kubeadmin"
}
```

Para encontrar la dirección URL de la consola del clúster, ejecute el siguiente comando, que será similar a `https://console-openshift-console.apps.<random>.<region>.aroapp.io/`.

```azurecli-interactive
 az aro show \
    --name aro-cluster \
    --resource-group aro-rg \
    --query "consoleProfile.url" -o tsv
```

Inicie la dirección URL de la consola en un explorador e inicie sesión con las credenciales de `kubeadmin`.

Vaya a **Administración**, haga clic en **Configuración del clúster** y, a continuación, seleccione la pestaña **Configuración global**. Desplácese para seleccionar **OAuth**.

Desplácese hacia abajo para seleccionar **Agregar** en la sección **Proveedores de identidades** y seleccione **OpenID Connect**.
![Seleccionar OpenID Connect del menú desplegable Proveedores de identidades](media/aro4-oauth-idpdrop.png).

Rellene el nombre como **AAD**, el **Id. de cliente** como **Id. de la aplicación** y el **Secreto de cliente**. La **URL del emisor** tiene el formato siguiente: `https://login.microsoftonline.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`. Reemplace el marcador de posición por el Id. de inquilino que recuperó anteriormente.

![Rellenar los detalles de OAuth](media/aro4-oauth-idp-1.png)

Desplácese hacia abajo hasta la sección **Notificaciones** y actualice el **Nombre de usuario preferido** para usar el valor de la notificación **UPN**.

![Rellenar los detalles de notificaciones](media/aro4-oauth-idp-2.png)

## <a name="verify-login-through-azure-active-directory"></a>Verificación del inicio de sesión mediante Azure Active Directory

Si cierra sesión ahora en la consola web de OpenShift y vuelve a iniciar sesión, se le presentará una nueva opción para iniciar sesión con **AAD**. Debe esperar unos minutos.

![Pantalla de inicio de sesión con la opción de Azure Active Directory](media/aro4-login-2.png)