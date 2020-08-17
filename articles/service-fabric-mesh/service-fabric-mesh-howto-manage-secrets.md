---
title: Administración de secretos en aplicaciones de Azure Service Fabric Mesh
description: Administre los secretos de las aplicaciones para poder crear e implementar de forma segura una aplicación Service Fabric Mesh.
ms.date: 4/2/2019
ms.topic: conceptual
ms.custom: devx-track-azurecli
ms.openlocfilehash: 379560b73c38558fe0b712ed5e036c7a3736b600
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500715"
---
# <a name="manage-service-fabric-mesh-application-secrets"></a>Administración de secretos en aplicaciones de Service Fabric Mesh
Service Fabric Mesh admite secretos como recursos de Azure. Un secreto de Service Fabric Mesh puede ser cualquier información de texto confidencial, como cadenas de conexión de almacenamiento, contraseñas u otros valores que deben almacenarse y transmitirse de forma segura. En este artículo se muestra cómo utilizar el Servicio de almacenamiento seguro de Service Fabric para implementar y mantener secretos.

Un secreto de la aplicación Mesh consta de:
* Un recurso **Secretos**, que es un contenedor que almacena secretos de texto. Los secretos contenidos dentro del recurso **Secretos** se almacenan y se transmiten de forma segura.
* Uno o varios recursos **Secretos o valores** que se almacenan en el contenedor de recursos **Secretos**. Cada recurso **Secretos o valores** se distingue por un número de versión. No puede modificar una versión de un recurso **Secretos o valores**, solo puede anexar una nueva versión.

La administración de secretos consta de los pasos siguientes:
1. Declare un recurso **Secretos** de Mesh en un archivo YAML o JSON del modelo de recursos de Azure mediante las definiciones del tipo inlinedValue y SecretsStoreRef contentType.
2. Declare los recursos **Secretos o valores** de Mesh en un archivo YAML o JSON del modelo de recursos de Azure que se almacenará en el recurso **Secretos** (del paso 1).
3. Modifique la aplicación Mesh para que haga referencia a los valores de secretos de Mesh.
4. Implemente o actualice gradualmente la aplicación Mesh para consumir valores de secretos.
5. Utilice los comandos "az" de la CLI de Azure para la administración del ciclo de vida del Servicio de almacenamiento seguro.

## <a name="declare-a-mesh-secrets-resource"></a>Declaración de un recurso de secretos de Mesh
Un recurso de secretos de Mesh se declara en un archivo JSON o YAML del modelo de recursos de Azure mediante las definiciones del tipo inlinedValue. El recurso de secretos de Mesh admite los secretos con origen en el Servicio de almacenamiento seguro. 
>
El siguiente es un ejemplo sobre cómo declarar los recursos de secretos de Mesh en un archivo JSON:

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "WestUS",
      "metadata": {
        "description": "Location of the resources (e.g. westus, eastus, westeurope)."
      }
    }
  },
  "sfbpHttpsCertificate": {
      "type": "string",
      "metadata": {
        "description": "Plain Text Secret Value that your container ingest"
      }
  },
  "resources": [
    {
      "apiVersion": "2018-07-01-preview",
      "name": "sfbpHttpsCertificate.pfx",
      "type": "Microsoft.ServiceFabricMesh/secrets",
      "location": "[parameters('location')]", 
      "dependsOn": [],
      "properties": {
        "kind": "inlinedValue",
        "description": "SFBP Application Secret",
        "contentType": "text/plain",
      }
    }
  ]
}
```
El siguiente es un ejemplo sobre cómo declarar los recursos de secretos de Mesh en un archivo YAML:
```yaml
    services:
      - name: helloWorldService
        properties:
          description: Hello world service.
          osType: linux
          codePackages:
            - name: helloworld
              image: myapp:1.0-alpine
              resources:
                requests:
                  cpu: 2
                  memoryInGB: 2
              endpoints:
                - name: helloWorldEndpoint
                  port: 8080
          secrets:
            - name: MySecret.txt
            description: My Mesh Application Secret
            secret_type: inlinedValue
            content_type: SecretStoreRef
            value: mysecret
    replicaCount: 3
    networkRefs:
      - name: mynetwork
```

## <a name="declare-mesh-secretsvalues-resources"></a>Declaración de recursos de secretos o valores de Mesh
Los recursos de secretos o valores de Mesh dependen de los recursos de secretos de Mesh definidos en el paso anterior.

En cuanto a la relación entre los campos "value:" y "name:" de la sección "resources", la segunda parte de la cadena "name:" delimitada por dos puntos es el número de versión utilizado para un secreto, y el nombre que precede a los dos puntos debe coincidir con el valor del secreto de Mesh para el que tiene una dependencia. Por ejemplo, para el elemento ```name: mysecret:1.0```, el número de versión es 1.0 y el nombre ```mysecret``` debe coincidir con ```"value": "mysecret"```, definido anteriormente.

>
El siguiente es un ejemplo sobre cómo declarar los recursos de secretos o valores de Mesh en un archivo JSON:

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "WestUS",
      "metadata": {
        "description": "Location of the resources (e.g. westus, eastus, westeurope)."
      }
    }
  },
  "sfbpHttpsCertificate": {
      "type": "string",
      "metadata": {
        "description": "Plain Text Secret Value that your container ingest"
      }
  },
  "resources": [
    {
      "apiVersion": "2018-07-01-preview",
      "name": "sfbpHttpsCertificate.pfx",
      "type": "Microsoft.ServiceFabricMesh/secrets",
      "location": "[parameters('location')]", 
      "dependsOn": [],
      "properties": {
        "kind": "inlinedValue",
        "description": "SFBP Application Secret",
        "contentType": "text/plain",
      }
    },
    {
      "apiVersion": "2018-07-01-preview",
      "name": "sfbpHttpsCertificate.pfx/2019.02.28",
      "type": "Microsoft.ServiceFabricMesh/secrets/values",
      "location": "[parameters('location')]",
      "dependsOn": [
        "Microsoft.ServiceFabricMesh/secrets/sfbpHttpsCertificate.pfx"
      ],
      "properties": {
        "value": "[parameters('sfbpHttpsCertificate')]"
      }
    }
  ],
}
```
El siguiente es un ejemplo sobre cómo declarar los recursos de secretos o valores de Mesh en un archivo YAML:
```yaml
    services:
      - name: helloWorldService
        properties:
          description: Hello world service.
          osType: linux
          codePackages:
            - name: helloworld
              image: myapp:1.0-alpine
              resources:
                requests:
                  cpu: 2
                  memoryInGB: 2
              endpoints:
                - name: helloWorldEndpoint
                  port: 8080
          Secrets:
            - name: MySecret.txt
            description: My Mesh Application Secret
            secret_type: inlinedValue
            content_type: SecretStoreRef
            value: mysecret
            - name: mysecret:1.0
            description: My Mesh Application Secret Value
            secret_type: value
            content_type: text/plain
            value: "P@ssw0rd#1234"
    replicaCount: 3
    networkRefs:
      - name: mynetwork
```

## <a name="modify-mesh-application-to-reference-mesh-secret-values"></a>Modifique la aplicación Mesh para que haga referencia a los valores de secretos de Mesh.
Las aplicaciones Service Fabric Mesh deben tener en cuenta las dos cadenas siguientes para consumir los valores de los secretos del Servicio de almacenamiento seguro:
1. Microsoft.ServiceFabricMesh/Secrets.name contiene el nombre del archivo y el valor del secreto en texto no cifrado.
2. La variable de entorno "Fabric_SettingPath" de Windows o Linux contiene la ruta de acceso del directorio donde se puede acceder a los archivos que contienen los valores de los secretos del Servicio de almacenamiento seguro. Esto es "C:\Settings" para aplicaciones hospedadas en Windows y "/var/settings" para aplicaciones Mesh hospedadas en Linux, respectivamente.

## <a name="deploy-or-use-a-rolling-upgrade-for-mesh-application-to-consume-secret-values"></a>Implementación o actualización gradual de la aplicación Mesh para consumir valores de secretos.
La creación de secretos o de secretos o valores con versiones se limita a las implementaciones declaradas del modelo de recursos. La única manera de crear estos recursos es pasar un archivo JSON o YAML del modelo de recurso mediante el comando **az mesh deployment** de la siguiente manera:

```azurecli-interactive
az mesh deployment create –-<template-file> or --<template-uri>
```

## <a name="azure-cli-commands-for-secure-store-service-lifecycle-management"></a>Comandos de la CLI de Azure para la administración del ciclo de vida del Servicio de almacenamiento seguro

### <a name="create-a-new-secrets-resource"></a>Crear un nuevo recurso de secretos
```azurecli-interactive
az mesh deployment create –-<template-file> or --<template-uri>
```
Pase **template-file** o **template-uri** (pero no ambos).

Por ejemplo:
- az mesh deployment create --c:\MyMeshTemplates\SecretTemplate1.txt
- az mesh deployment create --https:\//www.fabrikam.com/MyMeshTemplates/SecretTemplate1.txt

### <a name="show-a-secret"></a>Mostrar un secreto
Devuelve la descripción del secreto (pero no el valor).
```azurecli-interactive
az mesh secret show --Resource-group <myResourceGroup> --secret-name <mySecret>
```

### <a name="delete-a-secret"></a>Eliminar un secreto

- No se puede eliminar un secreto mientras una aplicación Mesh haga referencia a él.
- Si se elimina un recurso de secretos, se eliminan todas las versiones de los secretos o recursos.
  ```azurecli-interactive
  az mesh secret delete --Resource-group <myResourceGroup> --secret-name <mySecret>
  ```

### <a name="list-secrets-in-subscription"></a>Enumerar secretos en la suscripción
```azurecli-interactive
az mesh secret list
```
### <a name="list-secrets-in-resource-group"></a>Enumerar secretos del grupo de recursos
```azurecli-interactive
az mesh secret list -g <myResourceGroup>
```
### <a name="list-all-versions-of-a-secret"></a>Enumerar todas las versiones de un secreto
```azurecli-interactive
az mesh secretvalue list --Resource-group <myResourceGroup> --secret-name <mySecret>
```

### <a name="show-secret-version-value"></a>Mostrar el valor de la versión del secreto
```azurecli-interactive
az mesh secretvalue show --Resource-group <myResourceGroup> --secret-name <mySecret> --version <N>
```

### <a name="delete-secret-version-value"></a>Eliminar el valor de la versión del secreto
```azurecli-interactive
az mesh secretvalue delete --Resource-group <myResourceGroup> --secret-name <mySecret> --version <N>
```

## <a name="next-steps"></a>Pasos siguientes 
Para obtener más información sobre Service Fabric mesh, lea la introducción:
- [Service Fabric Mesh overview](service-fabric-mesh-overview.md) (Introducción a Service Fabric Mesh)
