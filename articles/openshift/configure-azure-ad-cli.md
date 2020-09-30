---
title: 'Red Hat OpenShift en Azure con OpenShift 4: configuración de la autenticación de Azure Active Directory mediante la línea de comandos'
description: Obtenga información sobre cómo configurar la autenticación de Azure Active Directory para un clúster de Red Hat OpenShift en Azure que ejecute OpenShift 4 mediante la línea de comandos.
ms.service: container-service
ms.topic: article
ms.date: 03/12/2020
author: sabbour
ms.author: asabbour
keywords: aro, openshift, az aro, red hat, cli
ms.custom: mvc
ms.openlocfilehash: fd6ea0749cce154ae20479bc54ef9b7374a69d0c
ms.sourcegitcommit: 4feb198becb7a6ff9e6b42be9185e07539022f17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89469429"
---
# <a name="configure-azure-active-directory-authentication-for-an-azure-red-hat-openshift-4-cluster-cli"></a>Configuración de la autenticación de Azure Active Directory para un clúster de Red Hat OpenShift 4 en Azure (CLI)

Si decide instalar y usar la CLI localmente, para este artículo es necesario que ejecute la versión CLI de Azure 2.6.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

Recupere las direcciones URL específicas del clúster que se van a usar para configurar la aplicación de Azure Active Directory.

Construya la dirección URL de devolución de llamada de OAuth del clúster y almacénela en una variable **oauthCallbackURL**. Asegúrese de reemplazar **aro-rg** por el nombre del grupo de recursos y **aro-cluster** por el nombre del clúster.

> [!NOTE]
> La sección `AAD` de la dirección URL de devolución de llamada de OAuth debe coincidir con el nombre del proveedor de identidades de OAuth que se va a configurar más adelante.

```azurecli-interactive
domain=$(az aro show -g aro-rg -n aro-cluster --query clusterProfile.domain -o tsv)
location=$(az aro show -g aro-rg -n aro-cluster --query location -o tsv)
apiServer=$(az aro show -g aro-rg -n aro-cluster --query apiserverProfile.url -o tsv)
webConsole=$(az aro show -g aro-rg -n aro-cluster --query consoleProfile.url -o tsv)
oauthCallbackURL=https://oauth-openshift.apps.$domain.$location.aroapp.io/oauth2callback/AAD
```

## <a name="create-an-azure-active-directory-application-for-authentication"></a>Creación de una aplicación de Azure Active Directory para autenticación

Cree una aplicación de Azure Active Directory y recupere el identificador de la aplicación creado. Reemplace **\<ClientSecret>** por una contraseña segura.

```azurecli-interactive
az ad app create \
  --query appId -o tsv \
  --display-name aro-auth \
  --reply-urls $oauthCallbackURL \
  --password '<ClientSecret>'
```

Debe obtener un resultado parecido a este. Tome nota de ello, ya que es el valor de **AppId** que necesitará en pasos posteriores.

```output
6a4cb4b2-f102-4125-b5f5-9ad6689f7224
```

Recupere el identificador de inquilino de la suscripción que posee la aplicación.

```azure
az account show --query tenantId -o tsv
```

Debe obtener un resultado parecido a este. Tome nota de ello, ya que es el valor de **TenantId** que necesitará en pasos posteriores.

```output
72f999sx-8sk1-8snc-js82-2d7cj902db47
```

## <a name="create-a-manifest-file-to-define-the-optional-claims-to-include-in-the-id-token"></a>Creación de un archivo de manifiesto para definir las notificaciones opcionales que se van a incluir en el token de identificador

Los desarrolladores de aplicaciones pueden usar [notificaciones opcionales](../active-directory/develop/active-directory-optional-claims.md) en sus aplicaciones de Azure AD para especificar qué notificaciones desean incluir en los tokens que se envían a las aplicaciones.

Estas notificaciones opcionales sirven para:

- Seleccionar las notificaciones adicionales que se incluirán en los tokens para la aplicación.
- Cambiar el comportamiento de ciertas notificaciones que Azure AD devuelve en tokens.
- Agregar notificaciones personalizadas para la aplicación y acceder a ellas.

Se va a configurar OpenShift para usar la notificación `email` y revertir a `upn` para establecer el nombre de usuario preferido agregando `upn` como parte del token de identificador devuelto por Azure Active Directory.

Cree un archivo **manifest.json** para configurar la aplicación de Azure Active Directory.

```bash
cat > manifest.json<< EOF
[{
  "name": "upn",
  "source": null,
  "essential": false,
  "additionalProperties": []
},
{
"name": "email",
  "source": null,
  "essential": false,
  "additionalProperties": []
}]
EOF
```

## <a name="update-the-azure-active-directory-applications-optionalclaims-with-a-manifest"></a>Actualización de las notificaciones opcionales de la aplicación de Azure Active Directory con un manifiesto

Reemplace **\<AppID>** por el identificador que ha obtenido anteriormente.

```azurecli-interactive
az ad app update \
  --set optionalClaims.idToken=@manifest.json \
  --id <AppId>
```

## <a name="update-the-azure-active-directory-application-scope-permissions"></a>Actualización de los permisos de ámbito de las aplicaciones de Azure Active Directory

Para poder leer la información de usuario de Azure Active Directory, es necesario definir los ámbitos adecuados.

Reemplace **\<AppID>** por el identificador que ha obtenido anteriormente.

Agregue el permiso para el ámbito **Graph.User.Read de Azure Active Directory** para habilitar el inicio de sesión y leer el perfil de usuario.

```azurecli-interactive
az ad app permission add \
 --api 00000002-0000-0000-c000-000000000000 \
 --api-permissions 311a71cc-e848-46a1-bdf8-97ff7156d8e6=Scope \
 --id <AppId>
```

> [!NOTE]
> A menos que se autentique como administrador global de esta instancia de Azure Active Directory, puede omitir el mensaje para conceder el consentimiento, ya que se le pedirá que lo haga una vez que inicie sesión en su propia cuenta.

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

Inicie sesión en el servidor de API del clúster de OpenShift mediante el siguiente comando. La variable `$apiServer` se estableció [anteriormente](). Reemplace **\<kubeadmin password>** por la contraseña que ha recuperado.

```azurecli-interactive
oc login $apiServer -u kubeadmin -p <kubeadmin password>
```

Cree un secreto de OpenShift para almacenar el secreto de aplicación de Azure Active Directory y reemplace **\<ClientSecret>** por el secreto que ha recuperado anteriormente.

```azurecli-interactive
oc create secret generic openid-client-secret-azuread \
  --namespace openshift-config \
  --from-literal=clientSecret=<ClientSecret>
```

Cree un archivo **oidc.yaml** para configurar la autenticación OpenID de OpenShift en Azure Active Directory. Reemplace **\<AppID>** y **\<TenantId>** por los valores que ha recuperado anteriormente.

```bash
cat > oidc.yaml<< EOF
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: AAD
    mappingMethod: claim
    type: OpenID
    openID:
      clientID: <AppId>
      clientSecret:
        name: openid-client-secret-azuread
      extraScopes:
      - email
      - profile
      extraAuthorizeParameters:
        include_granted_scopes: "true"
      claims:
        preferredUsername:
        - email
        - upn
        name:
        - name
        email:
        - email
      issuer: https://login.microsoftonline.com/<TenantId>
EOF
```

Aplique la configuración al clúster.

```azurecli-interactive
oc apply -f oidc.yaml
```

Obtendrá una respuesta similar a la siguiente.

```output
oauth.config.openshift.io/cluster configured
```

## <a name="verify-login-through-azure-active-directory"></a>Verificación del inicio de sesión mediante Azure Active Directory

Si ahora cierra sesión en la consola web de OpenShift y vuelve a iniciar sesión, se le presentará una nueva opción para iniciar sesión con **AAD**. Debe esperar unos minutos.

![Pantalla de inicio de sesión con la opción de Azure Active Directory](media/aro4-login-2.png)