---
title: Configuración de CI/CD con GitHub Actions
description: Aprenda a implementar su código en Azure App Service desde una canalización de CI/CD con GitHub Actions. Personalice las tareas de compilación y ejecute implementaciones complejas.
ms.devlang: na
ms.topic: article
ms.date: 10/25/2019
ms.author: jafreebe
ms.reviewer: ushan
ms.custom: devx-track-python
ms.openlocfilehash: 264976fdfe514a8778c60fe9242ac555f268718d
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962577"
---
# <a name="deploy-to-app-service-using-github-actions"></a>Implementación de App Service con Acciones de GitHub

[Acciones de GitHub](https://help.github.com/en/articles/about-github-actions) le ofrece la flexibilidad de compilar un flujo de trabajo del ciclo de vida de desarrollo de software automatizado. Gracias a las acciones de Azure App Service para GitHub, puede automatizar el flujo de trabajo para implementar en [Azure App Service](overview.md) con Acciones de GitHub.

> [!IMPORTANT]
> Acciones de GitHub está actualmente en versión beta. Primero debe [registrarse para unirse a la versión preliminar](https://github.com/features/actions) mediante su cuenta de GitHub.
> 

Un archivo YAML (.yml) define un flujo de trabajo en la ruta de acceso `/.github/workflows/` de su repositorio. En esta definición se incluyen los diversos pasos y parámetros que componen el flujo de trabajo.

Para un flujo de trabajo de Azure App Service, el archivo tiene tres secciones:

|Sección  |Tareas  |
|---------|---------|
|**Autenticación** | 1. Defina una entidad de servicio. <br /> 2. Cree un secreto de GitHub. |
|**Compilar** | 1. Configure el entorno. <br /> 2. Cree la aplicación web. |
|**Implementar** | 1. Implemente la aplicación web. |

## <a name="generate-deployment-credentials"></a>Genere las credenciales de implementación.

# <a name="user-level-credentials"></a>[Credenciales de nivel de usuario](#tab/userlevel)

Puede crear una [entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) de la [CLI de Azure](/cli/azure/). Puede ejecutar este comando mediante [Azure Cloud Shell](https://shell.azure.com/) en Azure Portal o seleccionando el botón **Pruébelo**.

```azurecli-interactive
az ad sp create-for-rbac --name "myApp" --role contributor \
                            --scopes /subscriptions/<subscription-id>/resourceGroups/<group-name>/providers/Microsoft.Web/sites/<app-name> \
                            --sdk-auth
```

En este ejemplo, reemplace los marcadores de posición por su identificador de suscripción, el nombre del grupo de recursos y el nombre de la aplicación. La salida es un objeto JSON con las credenciales de asignación de roles que proporcionan acceso a la aplicación App Service similar al siguiente. Copie este objeto JSON para más adelante.

```output 
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

> [!IMPORTANT]
> Siempre es recomendable conceder acceso mínimo. El ámbito del ejemplo anterior se limita a la aplicación específica de App Service y no a todo el grupo de recursos.

# <a name="app-level-credentials"></a>[Credenciales de nivel de aplicación](#tab/applevel)

Puede usar credenciales de nivel de aplicación mediante el perfil de publicación de la aplicación. Vaya a la página de administración de la aplicación en el portal. En la página de **información general**, haga clic en la opción**Obtener perfil de publicación**.

Necesitará el contenido del archivo más adelante.

---

## <a name="configure-the-github-secret"></a>Configuración del secreto de GitHub

# <a name="user-level-credentials"></a>[Credenciales de nivel de usuario](#tab/userlevel)

En [GitHub](https://github.com/), examine el repositorio y seleccione **Configuración > Secretos > Agregar un nuevo secreto**.

Para utilizar las [credenciales de nivel de usuario](#generate-deployment-credentials), pegue toda la salida JSON del comando CLI de Azure en el campo de valor del secreto. Asigne al secreto un nombre como `AZURE_CREDENTIALS`.

Cuando configure el archivo de flujo de trabajo más adelante, usará el secreto para la entrada `creds` de la acción de inicio de sesión de Azure. Por ejemplo:

```yaml
- uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}
```

# <a name="app-level-credentials"></a>[Credenciales de nivel de aplicación](#tab/applevel)

En [GitHub](https://github.com/), examine el repositorio y seleccione **Configuración > Secretos > Agregar un nuevo secreto**.

Para usar las [credenciales de nivel de aplicación](#generate-deployment-credentials), pegue el contenido del archivo del perfil de publicación descargado en el campo de valor del secreto. Asigne al secreto un nombre como `azureWebAppPublishProfile`.

Cuando configure el archivo de flujo de trabajo más adelante, usará el secreto para la entrada `publish-profile` de la acción de implementación de la aplicación web de Azure. Por ejemplo:
    
```yaml
- uses: azure/webapps-deploy@v2
  with:
    publish-profile: ${{ secrets.azureWebAppPublishProfile }}
```

---

## <a name="set-up-the-environment"></a>Configuración del entorno

La configuración del entorno puede realizarse mediante una de las acciones de configuración.

|**Lenguaje**  |**Acción de configuración**  |
|---------|---------|
|**.NET**     | `actions/setup-dotnet` |
|**Java**     | `actions/setup-java` |
|**JavaScript** | `actions/setup-node` |
|**Python**     | `actions/setup-python` |

En los siguientes ejemplos se muestra la parte del flujo de trabajo que configura el entorno para los diversos lenguajes admitidos:

**JavaScript**

```yaml
    - name: Setup Node 10.x
      uses: actions/setup-node@v1
      with:
        node-version: '10.x'
```
**Python**

```yaml
    - name: Setup Python 3.6
      uses: actions/setup-python@v1
      with:
        python-version: 3.6
```

**.NET**

```yaml
    - name: Setup Dotnet 2.2.300
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '2.2.300'
```

**Java**

```yaml
    - name: Setup Java 1.8.x
      uses: actions/setup-java@v1
      with:
        # If your pom.xml <maven.compiler.source> version is not in 1.8.x
        # Please change the Java version to match the version in pom.xml <maven.compiler.source>
        java-version: '1.8.x'
```

## <a name="build-the-web-app"></a>Compilación de la aplicación web

Esto depende del lenguaje y, para los lenguajes admitidos por Azure App Service, esta sección debe comprender los pasos de compilación estándar de cada lenguaje.

En los siguientes ejemplos se muestra la parte del flujo de trabajo que compila la aplicación web, en los diversos lenguajes admitidos:

**JavaScript**

```yaml
    - name: 'Run npm'
      shell: bash
      run: |
        # If your web app project is not located in your repository's root
        # Please change your directory for npm in pushd
        pushd .
        npm install
        npm run build --if-present
        npm run test --if-present
        popd
```

**Python**

```yaml
    - name: 'Run pip'
      shell: bash
      run: |
        # If your web app project is not located in your repository's root
        # Please change your directory for pip in pushd
        pushd .
        python -m pip install --upgrade pip
        pip install -r requirements.txt --target=".python_packages/lib/python3.6/site-packages"
        popd
```

**.NET**

```yaml
    - name: 'Run dotnet build'
      shell: bash
      run: |
        # If your web app project is not located in your repository's root
        # Please consider using pushd to change your path
        pushd .
        dotnet build --configuration Release --output ./output
        popd
```

**Java**

```yaml
    - uses: actions/checkout@v1
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    - name: Build with Maven
      run: mvn -B package --file pom.xml
```
## <a name="deploy-to-app-service"></a>Implementación en App Service

Para implementar el código en una aplicación de App Service, utilice la acción `azure/webapps-deploy@v2`. Esta acción tiene cuatro parámetros:

| **Parámetro**  | **Explicación**  |
|---------|---------|
| **app-name** | (Obligatorio) Especifique el nombre de la aplicación de App Service. | 
| **publish-profile** | (Opcional) Publique el contenido del archivo de perfil con secretos de Web Deploy |
| **package** | (Opcional) Ruta de acceso al paquete o la carpeta. *.zip, *.war, *.jar o una carpeta para implementar |
| **slot-name** | (Opcional) Especifique un espacio existente que no sea el de producción. |

# <a name="user-level-credentials"></a>[Credenciales de nivel de usuario](#tab/userlevel)

A continuación se muestra el flujo de trabajo de ejemplo para compilar e implementar una aplicación Node.js en Azure con una entidad de servicio de Azure. Observe cómo la entrada `creds` hace referencia al secreto de `AZURE_CREDENTIALS` que creó anteriormente.

```yaml
on: [push]

name: Node.js

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    # checkout the repo
    - name: 'Checkout GitHub Action' 
      uses: actions/checkout@master
   
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
        
    - name: Setup Node 10.x
      uses: actions/setup-node@v1
      with:
        node-version: '10.x'
    
    - name: 'npm install, build, and test'
      run: |
        npm install
        npm run build --if-present
        npm run test --if-present
               
    # deploy web app using Azure credentials
    - uses: azure/webapps-deploy@v2
      with:
        app-name: 'node-rn'

    # Azure logout 
    - name: logout
      run: |
        az logout
```

# <a name="app-level-credentials"></a>[Credenciales de nivel de aplicación](#tab/applevel)

A continuación se muestra el flujo de trabajo de ejemplo para compilar e implementar una aplicación Node.js en Azure con el perfil de publicación de la aplicación. Observe cómo la entrada `publish-profile` hace referencia al secreto de `azureWebAppPublishProfile` que creó anteriormente.

```yaml
# File: .github/workflows/workflow.yml

on: push

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    # checkout the repo
    - name: 'Checkout GitHub Action' 
      uses: actions/checkout@master
    
    - name: Setup Node 10.x
      uses: actions/setup-node@v1
      with:
        node-version: '10.x'
    - name: 'npm install, build, and test'
      run: |
        npm install
        npm run build --if-present
        npm run test --if-present
       
    - name: 'Run Azure webapp deploy action using publish profile credentials'
          uses: azure/webapps-deploy@v2
          with: 
            app-name: node-rn
            publish-profile: ${{ secrets.azureWebAppPublishProfile }}
```

---

## <a name="next-steps"></a>Pasos siguientes

Puede encontrar nuestro conjunto de acciones agrupadas en distintos repositorios de GitHub, cada uno con documentación y ejemplos que le ayudarán a usar GitHub con CI/CD y a implementar sus aplicaciones en Azure.

- [Flujos de trabajo de acciones para implementar en Azure](https://github.com/Azure/actions-workflow-samples)

- [Inicio de sesión de Azure](https://github.com/Azure/login)

- [Azure WebApp](https://github.com/Azure/webapps-deploy)

- [Azure WebApp for containers](https://github.com/Azure/webapps-container-deploy)

- [Inicio y cierre de sesión de Docker](https://github.com/Azure/docker-login)

- [Eventos que desencadenan flujos de trabajo](https://help.github.com/en/articles/events-that-trigger-workflows)

- [Implementación de K8s](https://github.com/Azure/k8s-deploy)

- [Flujos de trabajo de inicio](https://github.com/actions/starter-workflows)