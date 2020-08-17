---
title: Uso de Acciones de GitHub para crear actualizaciones de código en Azure Functions
description: Obtenga información sobre cómo usar Acciones de GitHub para definir un flujo de trabajo para crear e implementar proyectos de Azure Functions en GitHub.
author: craigshoemaker
ms.topic: conceptual
ms.date: 04/16/2020
ms.author: cshoe
ms.custom: devx-track-python
ms.openlocfilehash: 3fcf26670ec03b1d3a496574e58ae706acdf5c53
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87853298"
---
# <a name="continuous-delivery-by-using-github-action"></a>Entrega continua con Acciones de GitHub

[Acciones de GitHub](https://github.com/features/actions) le permite definir un flujo de trabajo para crear e implementar automáticamente su código de funciones en la aplicación de funciones de Azure. 

En Acciones de GitHub, un [flujo de trabajo](https://help.github.com/articles/about-github-actions#workflow) es un proceso automatizado que define en su repositorio de GitHub. Este proceso indica a GitHub cómo crear e implementar su proyecto de aplicación de funciones en GitHub. 

Un archivo YAML (.yml) define un flujo de trabajo en la ruta de acceso `/.github/workflows/` de su repositorio. En esta definición se incluyen los diversos pasos y parámetros que componen el flujo de trabajo. 

Para un flujo de trabajo de Azure Functions, el archivo tiene tres secciones: 

| Sección | Tareas |
| ------- | ----- |
| **Autenticación** | <ol><li>Defina una entidad de servicio.</li><li>Descargue un perfil de publicación.</li><li>Cree un secreto de GitHub.</li></ol>|
| **Compilar** | <ol><li>Configure el entorno.</li><li>Compile la aplicación de función.</li></ol> |
| **Implementación** | <ol><li>Implemente la aplicación de función.</li></ol>|

> [!NOTE]
> No es necesario crear una entidad de servicio si decide usar el perfil de publicación para la autenticación.

## <a name="create-a-service-principal"></a>Creación de una entidad de servicio

Puede crear una [entidad de servicio](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) de la [CLI de Azure](/cli/azure/). Puede ejecutar este comando mediante [Azure Cloud Shell](https://shell.azure.com) en Azure Portal o seleccionando el botón **Pruébelo**.

```azurecli-interactive
az ad sp create-for-rbac --name "myApp" --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.Web/sites/<APP_NAME> --sdk-auth
```

En este ejemplo, reemplace los marcadores de posición del recurso por su identificador de suscripción, grupo de recursos y nombre de la aplicación de función. La salida son las credenciales de asignación de roles que proporcionan acceso a su aplicación de funciones. Copie este objeto JSON, el cual puede usar para autenticarse desde GitHub.

> [!IMPORTANT]
> Siempre es recomendable conceder acceso mínimo. Es por ello que el ámbito del ejemplo anterior se limita a la aplicación de funciones específica y no a todo el grupo de recursos.

## <a name="download-the-publishing-profile"></a>Descarga del perfil de publicación

Para descargar el perfil de publicación de la aplicación de funciones:

1. Seleccione la página **Información general** de la aplicación de funciones y, después, seleccione **Obtener perfil de publicación**.

   :::image type="content" source="media/functions-how-to-github-actions/get-publish-profile.png" alt-text="Descargar perfil de publicación":::

1. Guarde y copie el contenido del archivo de configuración de publicación.

## <a name="configure-the-github-secret"></a>Configuración del secreto de GitHub

1. En [GitHub](https://github.com), examine su repositorio, seleccione **Configuración** > **Secretos** > **Agregar un nuevo secreto**.

   :::image type="content" source="media/functions-how-to-github-actions/add-secret.png" alt-text="Agregar secreto":::

1. Agregue un nuevo secreto.

   * Si usa la entidad de servicio que creó mediante la CLI de Azure, utilice `AZURE_CREDENTIALS` en **Nombre**. A continuación, pegue la salida del objeto JSON copiado de **Valor** y seleccione **Agregar secreto**.
   * Si utiliza un perfil de publicación, utilice `SCM_CREDENTIALS` en **Nombre**. A continuación, use el contenido del archivo del perfil de publicación de **Valor** y seleccione **Agregar secreto**.

Ahora GitHub puede autenticarse en su aplicación de funciones de Azure.

## <a name="set-up-the-environment"></a>Configuración del entorno 

La configuración del entorno se realiza mediante una acción de configuración de publicación específica del lenguaje.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el ejemplo siguiente se muestra la parte del flujo de trabajo que usa la acción `actions/setup-node` para configurar el entorno:

```yaml
    - name: 'Login via Azure CLI'
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    - name: Setup Node 10.x
      uses: actions/setup-node@v1
      with:
        node-version: '10.x'
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra la parte del flujo de trabajo que usa la acción `actions/setup-python` para configurar el entorno:

```yaml
    - name: 'Login via Azure CLI'
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    - name: Setup Python 3.6
      uses: actions/setup-python@v1
      with:
        python-version: 3.6
```

# <a name="c"></a>[C#](#tab/csharp)

En el ejemplo siguiente se muestra la parte del flujo de trabajo que usa la acción `actions/setup-dotnet` para configurar el entorno:

```yaml
    - name: 'Login via Azure CLI'
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    - name: Setup Dotnet 2.2.300
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '2.2.300'
```

# <a name="java"></a>[Java](#tab/java)

En el ejemplo siguiente se muestra la parte del flujo de trabajo que usa la acción `actions/setup-java` para configurar el entorno:

```yaml
    - name: 'Login via Azure CLI'
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    - name: Setup Java 1.8.x
      uses: actions/setup-java@v1
      with:
        # If your pom.xml <maven.compiler.source> version is not in 1.8.x
        # Please change the Java version to match the version in pom.xml <maven.compiler.source>
        java-version: '1.8.x'
```
---

## <a name="build-the-function-app"></a>Creación de la aplicación de funciones

Esto depende del lenguaje y, para los lenguajes admitidos por Azure Functions, esta sección deben ser los pasos de compilación estándar de cada lenguaje.

En el ejemplo siguiente se muestra la parte del flujo de trabajo que compila la aplicación de funciones, que es específica del lenguaje:

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```yaml
    - name: 'Run npm'
      shell: bash
      run: |
        # If your function app project is not located in your repository's root
        # Please change your directory for npm in pushd
        pushd .
        npm install
        npm run build --if-present
        npm run test --if-present
        popd
```

# <a name="python"></a>[Python](#tab/python)

```yaml
    - name: 'Run pip'
      shell: bash
      run: |
        # If your function app project is not located in your repository's root
        # Please change your directory for pip in pushd
        pushd .
        python -m pip install --upgrade pip
        pip install -r requirements.txt --target=".python_packages/lib/python3.6/site-packages"
        popd
```

# <a name="c"></a>[C#](#tab/csharp)

```yaml
    - name: 'Run dotnet build'
      shell: bash
      run: |
        # If your function app project is not located in your repository's root
        # Please consider using pushd to change your path
        pushd .
        dotnet build --configuration Release --output ./output
        popd
```

# <a name="java"></a>[Java](#tab/java)

```yaml
    - name: 'Run mvn'
      shell: bash
      run: |
        # If your function app project is not located in your repository's root
        # Please change your directory for maven build in pushd
        pushd . ./POM_ARTIFACT_ID
        mvn clean package
        mvn azure-functions:package
        popd
```
---

## <a name="deploy-the-function-app"></a>Implementación de la aplicación de función

Para implementar su código en una aplicación de funciones, tendrá que usar la acción `Azure/functions-action`. Esta acción tiene dos parámetros:

|Parámetro |Explicación  |
|---------|---------|
|**_app-name_** | (Obligatorio) El nombre de su aplicación de funciones. |
|_**slot-name**_ | (Opcional) El nombre de la [ranura de implementación](functions-deployment-slots.md) en la que desea realizar la implementación. La ranura ya debe estar definida en su aplicación de funciones. |


En el siguiente ejemplo se usa la versión 1 de la `functions-action`:

```yaml
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: PLEASE_REPLACE_THIS_WITH_YOUR_FUNCTION_APP_NAME
```

## <a name="next-steps"></a>Pasos siguientes

Para ver un archivo .yaml de flujo de trabajo completo, consulte uno de los archivos del [repositorio de ejemplos de flujo de trabajo de Acciones de GitHub de Azure](https://aka.ms/functions-actions-samples) con `functionapp` en el nombre. Puede usar estos ejemplos como punto de partida para su flujo de trabajo.

> [!div class="nextstepaction"]
> [Más información sobre Acciones de GitHub](https://help.github.com/en/articles/about-github-actions)
