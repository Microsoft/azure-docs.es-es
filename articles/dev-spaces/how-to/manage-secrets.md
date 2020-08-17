---
title: Administración de secretos cuando se trabaja con un espacio de Azure Dev Spaces
services: azure-dev-spaces
ms.date: 12/03/2019
ms.topic: conceptual
description: Aprenda a usar secretos de Kubernetes en tiempo de ejecución o de compilación al desarrollar aplicaciones con Azure Dev Spaces
keywords: Docker, Kubernetes, Azure, AKS, Azure Container Service, contenedores
ms.custom: devx-track-javascript
ms.openlocfilehash: 25d67d9ace6870eed283302256e3fa85661a0672
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421576"
---
# <a name="how-to-manage-secrets-when-working-with-an-azure-dev-space"></a>Administración de secretos cuando se trabaja con un espacio de Azure Dev Spaces

Los servicios pueden requerir algunas contraseñas, cadenas de conexión y otros secretos, como para las bases de datos u otros servicios de Azure seguros. Al establecer los valores de estos secretos en archivos de configuración, puede que estén disponibles en el código como variables de entorno.  Estos archivos de configuración se deben administrar con cuidado para evitar poner en peligro la seguridad de los secretos.

## <a name="storing-and-using-runtime-secrets"></a>Almacenamiento y uso de secretos en tiempo de ejecución

Azure Dev Spaces proporciona dos opciones optimizadas recomendadas para almacenar secretos en gráficos de Helm generados por las herramientas de cliente de Azure Dev Spaces: en el archivo `values.dev.yaml` y en línea directamente en `azds.yaml`. No se recomienda almacenar secretos en `values.yaml`.

> [!NOTE]
> Los siguientes enfoques muestran cómo almacenar y usar secretos para los gráficos de Helm generados por las herramientas de cliente. Si crea su propio gráfico de Helm, puede usarlo directamente para administrar y almacenar secretos.

### <a name="using-valuesdevyaml"></a>Uso de values.dev.yaml

En un proyecto que ya haya preparado con Azure Dev Spaces, cree un archivo `values.dev.yaml` en la misma carpeta que `azds.yaml` para definir sus valores y claves secretas. Por ejemplo:

```yaml
secrets:
  redis:
    port: "6380"
    host: "contosodevredis.redis.cache.windows.net"
    key: "secretkeyhere"
```

Compruebe que el archivo `azds.yaml` hace referencia a `values.dev.yaml` como opcional mediante `?`. Por ejemplo:

```yaml
install:
  values:
  - values.dev.yaml?
  - secrets.dev.yaml?
```

Si tiene archivos secretos adicionales, puede agregarlos aquí también.

Actualice o compruebe que el servicio hace referencia a los secretos como variables de entorno. Por ejemplo:

```javascript
var redisPort = process.env.REDIS_PORT
var host = process.env.REDIS_HOST
var theKey = process.env.REDIS_KEY
```
    
Ejecute los servicios actualizados mediante `azds up`.

```console
azds up
```
 
Use `kubectl` para comprobar que los secretos se han creado.

```console
kubectl get secret --namespace default -o yaml 
```

> [!IMPORTANT]
> No se recomienda almacenar los secretos en el control de código fuente. Si usa GIT, agregue `values.dev.yaml` al archivo `.gitignore` para evitar la confirmación de secretos en el control de código fuente.

### <a name="using-azdsyaml"></a>Uso de azds.yaml

En un proyecto que ya haya preparado con Azure Dev Spaces, agregue las claves secretas y el valor mediante la sintaxis *$PLACEHOLDER* en *configurations.develop.install.set* en `azds.yaml`. Por ejemplo:

```yaml
configurations:
  develop:
    ...
    install:
      set:
        secrets:
          redis:
            port: "$REDIS_PORT"
            host: "$REDIS_HOST"
            key: "$REDIS_KEY"
```

> [!NOTE]
> Puede especificar valores secretos directamente sin usar la sintaxis *$PLACEHOLDER* en `azds.yaml`. Sin embargo, este enfoque no se recomienda, ya que `azds.yaml` se almacena en el control de código fuente.
     
Cree un archivo de `.env` en la misma carpeta que `azds.yaml` para definir los valores de *$PLACEHOLDER*. Por ejemplo:

```
REDIS_PORT=3333
REDIS_HOST=myredishost
REDIS_KEY=myrediskey
```

> [!IMPORTANT]
> No se recomienda almacenar los secretos en el control de código fuente. Si usa GIT, agregue `.env` al archivo `.gitignore` para evitar la confirmación de secretos en el control de código fuente.

Actualice o compruebe que el servicio hace referencia a los secretos como variables de entorno. Por ejemplo:

```javascript
var redisPort = process.env.REDIS_PORT
var host = process.env.REDIS_HOST
var theKey = process.env.REDIS_KEY
```
    
Ejecute los servicios actualizados mediante `azds up`.

```console
azds up
```
 
Use `kubectl` para comprobar que los secretos se han creado.

```console
kubectl get secret --namespace default -o yaml 
```

## <a name="using-secrets-as-build-arguments"></a>Uso de secretos como argumentos de compilación

En las secciones anteriores se ha mostrado cómo almacenar y usar secretos en el tiempo de ejecución del contenedor. También puede usar cualquier secreto en el tiempo de compilación del contenedor, como una contraseña para un NuGet privado, mediante `azds.yaml`.

En `azds.yaml`, establezca los secretos de tiempo de compilación en *configurations.develop.build.args* con la sintaxis `<variable name>: ${secret.<secret name>.<secret key>}`. Por ejemplo:

```yaml
configurations:
  develop:
    build:
      dockerfile: Dockerfile.develop
      useGitIgnore: true
      args:
        BUILD_CONFIGURATION: ${BUILD_CONFIGURATION:-Debug}
        MYTOKEN: ${secret.mynugetsecret.pattoken}
```

En el ejemplo anterior, *mynugetsecret* es un secreto existente y *pattoken* es una clave existente.

>[!NOTE]
> Los nombres de secretos y las claves pueden contener el carácter `.`. Use `\` para escapar `.` al pasar secretos como argumentos de compilación. Por ejemplo, para pasar un secreto denominado *foo.bar* con la clave de *token*: `MYTOKEN: ${secret.foo\.bar.token}`. Además, los secretos se pueden evaluar con texto de prefijo y sufijo. Por ejemplo, `MYURL: eus-${secret.foo\.bar.token}-version1`. Asimismo, los secretos disponibles en espacios primarios y principales se pueden pasar como argumentos de compilación.

En el archivo Dockerfile, use la directiva *ARG* para consumir el secreto y, después, use esa misma variable más adelante en el archivo Dockerfile. Por ejemplo:

```dockerfile
...
ARG MYTOKEN
...
ARG NUGET_EXTERNAL_FEED_ENDPOINTS="{'endpointCredentials': [{'endpoint':'PRIVATE_NUGET_ENDPOINT', 'password':'${MYTOKEN}'}]}"
...
```

Actualice los servicios que se ejecutan en el clúster con estos cambios. En la línea de comandos, ejecute el comando:

```
azds up
```

## <a name="next-steps"></a>Pasos siguientes

Con estos métodos, ahora puede conectarse de forma segura a una base de datos, a una instancia de Azure Cache for Redis o acceder a los servicios de Azure seguros.
 
