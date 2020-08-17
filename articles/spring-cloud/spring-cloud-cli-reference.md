---
title: az spring cloud
description: Administración de Spring Cloud con la CLI de Azure
author: bmitchell287
ms.service: spring-cloud
ms.topic: reference
ms.date: 10/03/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 07e7b8db3bf7e1131ec8892a69223eaf4505e8de
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87037753"
---
# <a name="az-spring-cloud"></a>az spring-cloud

## <a name="manage-azure-spring-cloud-using-the-azure-cli"></a>Administración de Spring Cloud con la CLI de Azure

>[!Note]
> Azure Spring Cloud se encuentra actualmente en versión preliminar.  Estos comandos se pueden cambiar o quitar en una versión futura.

| az spring-cloud | Comandos: |
|------|------:|
| [az spring-cloud create](#az-spring-cloud-create) | Crea una instancia de Azure Spring Cloud. |
| [az spring-cloud delete](#az-spring-cloud-delete) | Elimina una instancia de Azure Spring Cloud. |
| [az spring-cloud list](#az-spring-cloud-list) | Enumera todas las instancias de Azure Spring Cloud en el grupo de recursos especificado; de lo contrario, enumera los identificadores de suscripción. |
| [az spring-cloud show](#az-spring-cloud-show) | Muestra los detalles de una instancia de Azure Spring Cloud. |

| az spring-cloud app | Comandos para administrar aplicaciones en Azure Spring Cloud.  |
| ---- | ----: |
| [az spring-cloud app create](#az-spring-cloud-app-create) | Crea una aplicación con una implementación predeterminada en Azure Spring Cloud. |
| [az spring-cloud app delete](#az-spring-cloud-app-delete) | Elimina una aplicación en Azure Spring Cloud. |
| [az spring-cloud app deploy](#az-spring-cloud-app-deploy) | Implementa desde el código fuente o un binario pregenerado en una aplicación y actualiza las configuraciones relacionadas. |
| [az spring-cloud app list](#az-spring-cloud-app-list) | Enumera todas las aplicaciones de Azure Spring Cloud. |
| [az spring-cloud app restart](#az-spring-cloud-app-restart) | Reinicia las instancias de la aplicación con los valores predeterminados de implementación de producción. |
| [az spring-cloud app scale](#az-spring-cloud-app-scale) | Escala manualmente una aplicación o sus implementaciones. |
| [az spring-cloud app set-deployment](#az-spring-cloud-app-set-deployment) | Establece la implementación de producción de una aplicación. |
| [az spring-cloud app show](#az-spring-cloud-app-show) | Muestra los detalles de una aplicación en Azure Spring Cloud. |
| [az spring-cloud app show-deploy-log](#az-spring-cloud-app-show-deploy-log) | Muestra los registros de compilación para la implementación más reciente desde el origen. Valores predeterminados para la implementación de producción. |
| [az spring-cloud app start](#az-spring-cloud-app-start) | Inicia instancias de la aplicación con los valores predeterminados de implementación de producción. |
| [az spring-cloud app stop](#az-spring-cloud-app-stop) | Detiene las instancias de la aplicación con los valores predeterminados de implementación de producción. |
| [az spring-cloud app update](#az-spring-cloud-app-update) | Actualiza la configuración de la aplicación especificada. |

| az spring-cloud app binding | Comandos para administrar enlaces con Azure Data Services.  La aplicación debe reiniciarse para que esta configuración surta efecto. |
| --- | ---: |
| [az spring-cloud app binding list](#az-spring-cloud-app-binding-list) | Enumera todos los enlaces de servicio de una aplicación. |
| [az spring-cloud app binding remove](#az-spring-cloud-app-binding-remove) | Quita un enlace de servicio de la aplicación. |
| [az spring-cloud app binding show](#az-spring-cloud-app-binding-show) | Muestra los detalles de un enlace de servicio. |
| [az spring-cloud app binding cosmos add](#az-spring-cloud-app-binding-cosmos-add) | Enlaza una instancia de Azure Cosmos DB con la aplicación. |
| [az spring-cloud app binding cosmos update](#az-spring-cloud-app-binding-cosmos-update) | Actualiza un enlace de servicio de Azure Cosmos DB. |
| [az spring-cloud app binding mysql add](#az-spring-cloud-app-binding-mysql-add) | Enlaza una instancia de Azure Database for MySQL con la aplicación. |
| [az spring-cloud app binding mysql update](#az-spring-cloud-app-binding-mysql-update) | Actualiza un enlace de servicio de Azure Database for MySQL. |
| [az spring-cloud app binding redis add](#az-spring-cloud-app-binding-redis-add) | Enlaza una instancia de Azure Cache for Redis con la aplicación. |
| [az spring-cloud app binding redis update](#az-spring-cloud-app-binding-redis-update) | Actualiza un enlace de servicio de Azure Cache for Redis. |

| az spring-cloud app deployment | Comandos para administrar el ciclo de vida de la implementación de una aplicación en Azure Spring Cloud. |
| --- | ---: |
| [az spring-cloud app deployment create](#az-spring-cloud-app-deployment-create) | Crea una implementación de almacenamiento provisional para la aplicación. |
| [az spring-cloud app deployment delete](#az-spring-cloud-app-deployment-delete) | Elimina una implementación de la aplicación. |
| [az spring-cloud app deployment list](#az-spring-cloud-app-deployment-list) | Muestra todas las implementaciones de una aplicación. |
| [az spring-cloud app deployment show](#az-spring-cloud-app-deployment-show) | Muestra los detalles de la implementación. |

| az spring-cloud config-server | Comandos para administrar Azure Spring Cloud Config Server. |
| --- | ---: |
| [az spring-cloud config-server clear](#az-spring-cloud-config-server-clear) | Borra toda la configuración de Config Server. |
| [az spring-cloud config-server set](#az-spring-cloud-config-server-set) | Define Config Server a partir de un archivo YAML. |
| [az spring-cloud config-server show](#az-spring-cloud-config-server-show) | Muestra la configuración de Config Server. |
| [az spring-cloud config server git set](#az-spring-cloud-config-server-git-set) | Define las propiedades de Config Server.  Los valores anteriores se sobrescriben. |
| [az spring-cloud config server git repo add](#az-spring-cloud-config-server-git-repo-add) | Agrega una nueva configuración de repositorio de Git a Config Server. |
| [az spring-cloud config server git repo list](#az-spring-cloud-config-server-git-repo-list) | Enumera todas las configuraciones de repositorios de Git para Config Server. |
| [az spring-cloud config server git repo remove](#az-spring-cloud-config-server-git-repo-remove) | Quita el repositorio de Git especificado de Config Server. |

| az spring-cloud test-endpoint | Comandos para administrar las pruebas de los puntos de conexión en Azure Spring Cloud. |
| --- | ---: |
| [az spring-cloud test-endpoint disable](#az-spring-cloud-test-endpoint-disable) | Deshabilita el punto de conexión de prueba. |
| [az spring-cloud test-endpoint enable](#az-spring-cloud-test-endpoint-enable) | Habilita el punto de conexión de prueba. |
| [az spring-cloud test-endpoint list](#az-spring-cloud-test-endpoint-list) | Enumera las claves del punto de conexión de prueba. |
| [az spring-cloud test-endpoint renew-key](#az-spring-cloud-test-endpoint-renew-key) | Vuelve a generar una clave del punto de conexión de prueba. |

## <a name="az-spring-cloud-create"></a>az spring-cloud create

Crea una aplicación con una implementación predeterminada en Azure Spring Cloud.

```azurecli
az spring-cloud create  --name -n
                        --resource-group -g
                        --location -l
                        --no-wait
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name -n | Nombre de esta instancia de Azure Spring Cloud. |
| --resource-group -g | Especifica el grupo de recursos de esta aplicación.  Configura el grupo predeterminado mediante `az configure --defaults group=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --location -l | Especifica la ubicación del servidor para esta aplicación.  Busca ubicaciones válidas mediante `az account list-locations`. |
| --no-wait | No esperar a que se completen las operaciones de larga duración.

### <a name="examples"></a>Ejemplos

Crea una instancia de Azure Spring Cloud en WestUS.

```azurecli
az spring-cloud create -n MyService -g MyResourceGroup -l westus
```

## <a name="az-spring-cloud-delete"></a>az spring-cloud delete

Elimina una instancia de Azure Spring Cloud.

```azurecli
az spring cloud --name -n
                --resource-group -g
                --no-wait
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name -n | Nombre de la instancia de Azure Spring Cloud que se va a eliminar. |
| --resource-group -g | Nombre del grupo de recursos al que pertenece la instancia de Azure Spring Cloud. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| -no-wait | No espera hasta que finalicen las operaciones de larga duración. |

### <a name="example"></a>Ejemplo

Elimina una instancia de Azure Spring Cloud denominada "MyService" de "MyResourceGroup".

```azurecli
az spring-cloud delete -n MyService -g MyResourceGroup
```

## <a name="az-spring-cloud-list"></a>az spring-cloud list

Enumera todas las instancias de Azure Spring Cloud asociadas al grupo de recursos especificado. Si no se especifica ningún grupo de recursos, enumera los identificadores de suscripción.

```azurecli
az spring-cloud list --resource-group -g
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --resource-group -g | Nombre del grupo de recursos. |

## <a name="az-spring-cloud-show"></a>az spring-cloud show

Muestra los detalles de la instancia de Azure Spring Cloud especificada.

```azurecli
az spring-cloud show --name -n
                     -- resource-group -g
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name -n | Nombre de la instancia de Azure Spring Cloud. |
| --resource-group -g | Nombre del grupo de recursos al que pertenece la instancia de Azure Spring Cloud.

## <a name="az-spring-cloud-app-create"></a>az spring-cloud app create

Crea una aplicación en una instancia de Azure Spring Cloud.

```azurecli
az spring-cloud app create --name -n
                            --resource-group -g
                            --service -s
                            --cpu
                            --enable-persistent-storage
                            --instance-count
                            --is-public
                            --memory
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name -n | Nombre de la aplicación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --cpu | Número de núcleos virtuales por instancia.  Valor predeterminado: 1. |
| --enable-persistent-storage | Valor booleano.  Si es true, monta un disco de 50 GB con la ruta de acceso predeterminada. |
| --instance-count | Número de instancia.  Valor predeterminado: 1. |
| --is-public | Valor booleano.  Si es true, asigna un dominio público. |
| --memory | Número de GB de memoria por instancia.  Valor predeterminado: 1. |

### <a name="examples"></a>Ejemplos

Crea una aplicación con la configuración predeterminada.

```azurecli
az spring-cloud app create -n MyApp -s MyService
```

Crea una aplicación de acceso público con 3 instancias.  Cada instancia tiene 3 GB de memoria y 2 núcleos de CPU.

```azurecli
az spring-cloud app create -n MyApp -s MyService --is-public true --cpu 2 --memory 3
```

## <a name="az-spring-cloud-app-delete"></a>az spring-cloud app delete

Elimina una aplicación en Azure Spring Cloud.

```azurecli
az spring cloud app delete  --name -n
                            --resource-group -g
                            --service -s
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name -n | Nombre de la aplicación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

## <a name="az-spring-cloud-app-deploy"></a>az spring-cloud app deploy

Implementa una aplicación en Azure Spring Cloud a partir del código fuente o un binario pregenerado y actualiza las configuraciones relacionadas.

```azurecli
az spring cloud app deploy  --name -n
                            --resource-group -g
                            --service -s
                            --cpu
                            --deployment -d
                            --env
                            --instance-count
                            --jar-path
                            --jvm-options
                            --memory
                            --no-wait
                            --runtime-version
                            --target-module
                            --version
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name -n | Nombre de la aplicación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --cpu | Número de núcleos de CPI virtuales por instancia. |
| --deployment -d | Nombre de una implementación de aplicación existente.  Si no se especifica, el valor predeterminado es la implementación de producción. |
| --env | Variables de entorno separadas por espacios en formato "clave[=valor]". |
| --instance-count | Número de instancias. |
| --jar-path | Si se proporciona, implementa el archivo jar desde la ruta de acceso dada. De lo contrario, implementa la carpeta actual como archivo tar. |
| --jvm-options | Cadena que contiene las opciones de máquina virtual Java.  Usa "=" en lugar de " " para evitar errores de análisis de Shell. Por ejemplo, `--jvm-options='-Xms1024m -Xmx2048m`. |
| --memory | Número de GB de memoria por instancia. |
| --no-wait | No espera hasta que finalicen las operaciones de larga duración. |
| --runtime-version | Versión del entorno de ejecución del lenguaje usado en la aplicación.  Valores permitidos: `Java_11`, `Java_8`. |
| --target-module | Módulo secundario que se va a implementar.  Obligatorio cuando se crean varios paquetes jar a partir del código fuente. |
| --version | Versión de la implementación.  Sin modificar si no se establece. |

### <a name="examples"></a>Ejemplos

Implementa el código fuente en una aplicación. Esto empaquetará el directorio actual, compilará un binario mediante el servicio de compilación dinámica y realizará la implementación en la aplicación.

```azurecli
az spring-cloud app deploy -n MyApp -s MyService
```

Implementa un archivo jar pregenerado en una aplicación mediante las opciones de máquina virtual Java y las variables de entorno.

```azurecli
az spring-cloud app deploy -n MyApp -s MyService --jar-path app.jar --jvm-options="-XX:+UseG1GC -XX:+UseStringDeduplication" --env foo=bar
```

Implementa el código fuente en una implementación específica de una aplicación.

```azurecli
az spring-cloud app deploy -n MyApp -s Myspring-cloud -d green-deployment
```

## <a name="az-spring-cloud-app-list"></a>az spring-cloud app list

Enumera todas las aplicaciones de la instancia de Azure Spring Cloud.

```azurecli
az spring-cloud app list --resource-group -g
                         --service -s
```

|Parámetros obligatorios | Descripción |
| --- | :--- |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

## <a name="az-spring-cloud-app-restart"></a>az spring-cloud app restart

Reinicia las instancias de la aplicación.  El valor predeterminado es la implementación de producción.

```azurecli
az spring-cloud app restart --name -n
                            --resource-group -g
                            --service -s
                            --deployment -d
                            --no-wait
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name -n | Nombre de la aplicación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --deployment -d | Nombre de la implementación existente de la aplicación.  Si no se especifica, el valor predeterminado es la implementación de producción. |
| --no-wait | No espera hasta que finalicen las operaciones de larga duración. |

## <a name="az-spring-cloud-app-scale"></a>az spring-cloud app scale

Escala manualmente una aplicación o sus implementaciones.

```azurecli
az spring-cloud app scale --name -n
                          --resource-group -g
                          --service -s
                          --cpu
                          --deployment -d
                          --instance-count
                          --memory
                          --no-wait
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name -n | Nombre de la aplicación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --cpu | Número de núcleos de CPU virtuales por instancia de aplicación. |
| --deployment -d | Nombre de la implementación existente de la aplicación.  Si no se especifica, el valor predeterminado es la implementación de producción. |
| --instance-count | Número de instancias de esta aplicación. |
| --memory | Número de GB de memoria por instancia de aplicación. |
| --no-wait | No espere hasta que finalice la operación de ejecución prolongada. |

### <a name="examples"></a>Ejemplos

Escala verticalmente una aplicación a 4 núcleos de CPU y 8 GB de memoria por instancia.

```azurecli
az spring-cloud app scale -n MyApp -s MyService --cpu 3 --memory 8
```

Escala horizontalmente una implementación de la aplicación a 5 instancias.

```azurecli
az spring-cloud app scale -n MyApp -s MyService -d green-deployment --instance-count 5
```

## <a name="az-spring-cloud-app-set-deployment"></a>az spring-cloud app set-deployment

Establece las opciones de configuración para la implementación de producción de la aplicación.

```azurecli
az spring-cloud app set-deployment --deployment -d
                                   --name -n
                                   --resource-group -g
                                   --service -s
                                   --no-wait
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --deployment -d | Nombre de una implementación existente de la aplicación. |
| --name -n | Nombre de la aplicación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --no-wait | No espere hasta que finalice la operación de ejecución prolongada. |

### <a name="examples"></a>Ejemplos

Intercambia una implementación de almacenamiento provisional de la aplicación en producción.

```azurecli
az spring-cloud app set-deployment -d green-deployment -n MyApp -s MyService
```

## <a name="az-spring-cloud-app-show"></a>az spring-cloud app show

Muestra los detalles de una aplicación en Azure Spring Cloud.

```azurecli
az spring-cloud app show --name -n
                         --resource-group -g
                         --service -s
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name -n | Nombre de la aplicación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

## <a name="az-spring-cloud-app-show-deploy-log"></a>az spring-cloud app show-deploy-log

Muestra el registro de compilación de la última implementación a partir del código fuente.  El valor predeterminado es el entorno de producción.

```azurecli
az spring-cloud app show-deploy-log --name -n
                                    --resource-group -g
                                    --service -s
                                    --deployment -d
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name -n | Nombre de la aplicación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --deployment -d | Nombre de una implementación existente de la aplicación.  El valor predeterminado es el entorno de producción. |

## <a name="az-spring-cloud-app-start"></a>az spring-cloud app start

Inicia las instancias de la aplicación.  El valor predeterminado es el entorno de producción.

```azurecli
az spring-cloud app start --name -n
                          --resource-group -g
                          --service -s
                          --deployment -d
                          --no-wait
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name -n | Nombre de la aplicación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --deployment -d | Nombre de una implementación existente de la aplicación.  El valor predeterminado es el entorno de producción. |
| --no-wait | No espere hasta que finalice la operación de ejecución prolongada. |

## <a name="az-spring-cloud-app-stop"></a>az spring-cloud app stop

Detiene las instancias de la aplicación.  El valor predeterminado es el entorno de producción.

```azurecli
az spring-cloud app stop --name -n
                         --resource-group -g
                         --service -s
                         --deployment -d
                         --no-wait
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name -n | Nombre de la aplicación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --deployment -d | Nombre de una implementación existente de la aplicación.  El valor predeterminado es el entorno de producción. |
| --no-wait | No espere hasta que finalice la operación de ejecución prolongada. |

## <a name="az-spring-cloud-app-update"></a>az spring-cloud app update

Actualiza la configuración almacenada de una aplicación.

```azurecli
az spring-cloud app update --name -n
                           --resource-group -g
                           --service -s
                           --deployment -d
                           --enable-persistent-storage
                           --env
                           --is-public
                           --jvm-options
                           --no-wait
                           --runtime-version
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name -n | Nombre de la aplicación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --deployment -d | Nombre de una implementación existente de la aplicación.  El valor predeterminado es el entorno de producción. |
| --enable-persistent-storage | booleano.  Si es true, monta un disco de 50 GB con la ruta de acceso predeterminada. |
| --env | Variables de entorno separadas por espacios en formato "clave[=valor]". |
| --is-public | booleano.  Si es true, asigna un dominio público a la aplicación. |
| --jvm-options | Cadena que contiene las opciones de máquina virtual Java.  Usa "=" en lugar de " " para evitar errores de análisis de Shell. Por ejemplo, `--jvm-options='-Xms1024m -Xmx2048m`. |
| --no-wait | No espere hasta que finalice la operación de ejecución prolongada. |
| --runtime-version | Versión del entorno de ejecución del lenguaje usado en la aplicación.  Valores permitidos: `Java_11`, `Java_8`. |

### <a name="example"></a>Ejemplo

Agrega una variable de entorno para la aplicación.

```azurecli
az spring-cloud app update --env foo=bar
```

## <a name="az-spring-cloud-app-binding-list"></a>az spring-cloud app binding list

Enumera todos los enlaces de servicio de una aplicación.

```azurecli
az spring-cloud app binding list --app
                                 --resource-group -g
                                 --service -s
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --app | Nombre de la aplicación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

## <a name="az-spring-cloud-app-binding-remove"></a>az spring-cloud app binding remove

Quita un enlace de servicio de la aplicación.

```azurecli
az spring-cloud app binding list --app
                                 --name -n
                                 --resource-group -g
                                 --service -s
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --app | Nombre de la aplicación. |
| --name | Nombre del enlace de servicio que se va a quitar. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

## <a name="az-spring-cloud-app-binding-show"></a>az spring-cloud app binding show

Muestra los detalles de un enlace de servicio.

```azurecli
az spring-cloud app binding show --app
                                 --name -n
                                 --resource-group -g
                                 --service -s
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --app | Nombre de la aplicación. |
| --name | Nombre del enlace de servicio. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

## <a name="az-spring-cloud-app-binding-cosmos-add"></a>az spring-cloud app binding cosmos add

Enlaza una instancia de Azure Cosmos DB con la aplicación.

```azurecli
az spring-cloud app binding cosmos add --api-type
                                       --app
                                       --name -n
                                       --resource-group -g
                                       --resource-id
                                       --service -s
                                       --collection-name
                                       --database-name
                                       --key-space
```

```azurecli
az spring-cloud app binding list --app
                                 --name -n
                                 --resource-group -g
                                 --service -s
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --api-type | Especifica el tipo de API con uno de los siguientes valores: Cassandra, Gremlin, Mongo, SQL y Table. |
| --app | Nombre de la aplicación. |
| --name | Nombre del enlace de servicio. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

|Parámetros opcionales | Descripción |
| --- | :--- |
| --collection-name | Nombre de la colección.  Obligatorio cuando se usa Gremlin. |
| --database-name | Nombre de la base de datos.  Obligatorio cuando se usa Mongo, SQL y Gremlin. |
| --key-space | Espacio de clave Cassandra.  Obligatorio cuando se usa Cassandra. |

## <a name="az-spring-cloud-app-binding-cosmos-update"></a>az spring-cloud app binding cosmos update

```azurecli
az spring-cloud app binding cosmos update --app
                                          --name -n
                                          --resource-group -g
                                          --service -s
                                          --collection-name
                                          --database-name
                                          --key-space
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --app | Nombre de la aplicación. |
| --name | Nombre del enlace de servicio. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

|Parámetros opcionales | Descripción |
| --- | :--- |
| --collection-name | Nombre de la colección.  Obligatorio cuando se usa Gremlin. |
| --database-name | Nombre de la base de datos.  Obligatorio cuando se usa Mongo, SQL y Gremlin. |
| --key-space | Espacio de clave Cassandra.  Obligatorio cuando se usa Cassandra. |

## <a name="az-spring-cloud-app-binding-mysql-add"></a>az spring-cloud app binding mysql add

```azurecli
az spring-cloud app binding mysql add --app
                                      --database-name
                                      --key
                                      --name -n
                                      --resource-group -g
                                      --service -s
                                      --username
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --app | Nombre de la aplicación. |
| --database-name | Nombre de la base de datos. |
| --key | Clave de API del servicio. |
| --name | Nombre del enlace de servicio. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --resource-id | Identificador de recurso de Azure del servicio con el que se va a enlazar. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |
| --username | Nombre de usuario para el acceso a la base de datos. |

## <a name="az-spring-cloud-app-binding-mysql-update"></a>az spring-cloud app binding mysql update

Actualiza la conexión de enlace de servicio de la aplicación a una instancia de Azure Database for MySQL.

```azurecli
az spring-cloud app binding mysql update --add
                                         --name -n
                                         --resource-group -g
                                         --service -s
                                         --database-name
                                         --key
                                         --username
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --app | Nombre de la aplicación. |
| --name | Nombre del enlace de servicio. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --database-name | Nombre de la base de datos. |
| --key | Clave de API del servicio. |
| --username | Nombre de usuario para el acceso a la base de datos. |

## <a name="az-spring-cloud-app-binding-redis-add"></a>az spring-cloud app binding redis add

Enlace de Azure Cache for Redis con la aplicación.

```azurecli
az spring-cloud app binding redis add --app
                                      --name -n
                                      --resource-group -g
                                      --resource-id
                                      --service -s
                                      --disable-ssl
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --app | Nombre de la aplicación. |
| --name | Nombre del enlace de servicio. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --resource-id | Identificador de recurso de Azure del servicio con el que se quiere a enlazar. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --disable-ssl | Deshabilita TLS. |

## <a name="az-spring-cloud-app-binding-redis-update"></a>az spring-cloud app binding redis update

Actualiza un enlace de servicio para Azure Cache for Redis.

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --app | Nombre de la aplicación. |
| --name | Nombre del enlace de servicio. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --disable-ssl | Deshabilita TLS. |

## <a name="az-spring-cloud-app-deployment-create"></a>az spring-cloud app deployment create

Crea una implementación de almacenamiento provisional para la aplicación.

Para implementar código o actualizar la configuración en una implementación existente, use `az spring-cloud app deploy --deployment <staging-deployment>` o "az spring-cloud app update --deployment"<staging deployment>.

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --app | Nombre de la aplicación. |
| --name | Nombre del enlace de servicio. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --cpu | Número de núcleos de CPU virtuales por instancia.  Valor predeterminado: 1 |
| --env | Variables de entorno separadas por espacios en formato "clave[=valor]". |
| --instance-count | Número de instancias. Valor predeterminado: 1. |
| --jar-path | Si se proporciona, implementa jar.  De lo contrario, implementa la carpeta actual como archivo tar. |
| --jvm-options | Cadena que contiene las opciones de máquina virtual Java.  Usa "=" en lugar de " " para evitar errores de análisis de Shell. Por ejemplo, `--jvm-options='-Xms1024m -Xmx2048m`. |
| --memory | Número de GB de memoria por instancia. |
| --no-wait | No espera hasta que finalicen las operaciones de larga duración. |
| --runtime-version | Versión del entorno de ejecución del lenguaje usado en la aplicación.  Valores permitidos: `Java_11`, `Java_8`. |
| --skip-clone-settings | Crea una implementación de almacenamiento provisional mediante la clonación de la configuración de implementación de producción actual. |
| --target-module | Módulo secundario que se va a implementar.  Obligatorio cuando se crean varios paquetes jar a partir del código fuente. |
| --version | Versión de la implementación.  Sin modificar si no se establece. |

### <a name="examples"></a>Ejemplos

Implementa el código fuente en una nueva implementación de la aplicación.  Esto empaquetará el directorio actual, compilará un binario mediante el sistema de compilación dinámica y realizará la implementación.

```azurecli
az spring-cloud app deployment create -n green-deployment --app MyApp -s MyService
```

Implementa un archivo jar pregenerado en una aplicación mediante las opciones de máquina virtual Java y las variables de entorno.

```azurecli
az spring-cloud app deployment create -n green-deployment --app MyApp -s MyService --jar-path app.jar --jvm-options="-XX:+UseStringDeDuplication" --env foo=bar
```

## <a name="az-spring-cloud-app-deployment-delete"></a>az spring-cloud app deployment delete

Elimina una implementación de la aplicación.

```azurecli
az spring-cloud app deployment delete --app
                                      --name -n
                                      --resource-group -g
                                      --service -s
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --app | Nombre de la aplicación. |
| --name | Nombre de la implementación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

## <a name="az-spring-cloud-app-deployment-list"></a>az spring-cloud app deployment list

Muestra todas las implementaciones de una aplicación.

```azurecli
az spring-cloud app deployment list --app
                                    --resource-group -g
                                    --service -s
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --app | Nombre de la aplicación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

## <a name="az-spring-cloud-app-deployment-show"></a>az spring-cloud app deployment show

Muestra los detalles de una implementación.

```azurecli
az spring-cloud app deployment show --app
                                    --name -n
                                    --resource-group -g
                                    --service -s
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --app | Nombre de la aplicación. |
| --name | Nombre de la implementación. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --service -s | Nombre de la instancia de Azure Spring Cloud.  Puede configurar el servicio predeterminado mediante `az configure --defaults spring-cloud=<name>`. |

## <a name="az-spring-cloud-config-server-clear"></a>az spring-cloud config-server clear

Borra todos los valores de configuración de Config Server.

```azurecli
az spring-cloud config-server clear --name
                                        --resource-group -g
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name | Nombre de la instancia de Azure Spring Cloud. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |

## <a name="az-spring-cloud-config-server-set"></a>az spring-cloud config-server set

Establece los valores de configuración en Config Server mediante un archivo YAML.

```azurecli
az spring-cloud config-server set --config-file
                                  --name -n
                                  --resource-group -g
                                  --no-wait
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --config-file | Ruta de acceso de archivo a un manifiesto de YAML para la configuración de Config Server. |
| --name | Nombre de la instancia de Azure Spring Cloud. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --no-wait | No esperar a que se completen las operaciones de larga duración.

## <a name="az-spring-cloud-config-server-show"></a>az spring-cloud config-server show

Muestra la configuración de Config Server.

```azurecli
az spring-cloud config-server show --name -n
                                   --resource-group -g
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name | Nombre de la instancia de Azure Spring Cloud. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |

## <a name="az-spring-cloud-config-server-git-set"></a>az spring-cloud config-server git set

Establece las propiedades de Git para Config Server.  Esto sobrescribirá todas las propiedades de Git existentes.

```azurecli
az spring-cloud config-server git set --name -n
                                      --resource-group g
                                      --uri
                                      --defer
                                      --host-key
                                      --host-key-algorithm
                                      --label
                                      --password
                                      --private-key
                                      --search-paths
                                      --strict-host-key-checking
                                      --username
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name | Nombre de la instancia de Azure Spring Cloud. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --uri | URI de la configuración agregada. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --defer | Almacena temporalmente el objeto en la memoria caché local en lugar de enviarlo a Azure.  Use `az cache` para ver o borrar. |
| --host-key | Clave de host para la configuración agregada. |
| --host-key-algorithm | Algoritmo de la clave de host para la configuración agregada. |
| --label | Etiqueta de la configuración agregada. |
| --password | Contraseña de la configuración agregada. |
| --private-key | Clave privada de la configuración agregada. |
| --search-paths | Rutas de acceso de búsqueda de la configuración agregada.  Use delimitadores de coma para varias rutas de acceso. |
| --strict-host-key-checking | Habilita la comprobación estricta de claves de host de la configuración agregada. |
| --username | Nombre de usuario de la configuración agregada. |

## <a name="az-spring-cloud-config-server-git-repo-add"></a>az spring-cloud config-server git repo add

```azurecli
az spring-cloud config-server git repo add --name -n
                                           --repo-name
                                           --resource-group -g
                                           --uri
                                           --defer
                                           --host-key
                                           --host-key-algorithm
                                           --label
                                           --password
                                           --pattern
                                           --private-key
                                           --search-paths
                                           --strict-host-key-checking
                                           --username
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name | Nombre de la instancia de Azure Spring Cloud. |
| --repo-name | URI del repositorio. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --uri | URI de la configuración agregada. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --defer | Almacena temporalmente el objeto en la memoria caché local en lugar de enviarlo a Azure.  Use `az cache` para ver o borrar. |
| --host-key | Clave de host para la configuración agregada. |
| --host-key-algorithm | Algoritmo de la clave de host para la configuración agregada. |
| --label | Etiqueta de la configuración agregada. |
| --password | Contraseña de la configuración agregada. |
| --pattern | Patrón para el repositorio.  Use delimitadores de coma para varias rutas de acceso.|
| --private-key | Clave privada de la configuración agregada. |
| --search-paths | Rutas de acceso de búsqueda de la configuración agregada.  Use delimitadores de coma para varias rutas de acceso. |
| --strict-host-key-checking | Habilita la comprobación estricta de claves de host de la configuración agregada. |
| --username | Nombre de usuario de la configuración agregada. |

## <a name="az-spring-cloud-config-server-git-repo-list"></a>az spring-cloud config-server git repo list

Enumera todos los repositorios de Git definidos en Config Server.

```azurecli
az spring-cloud config-server git repo list --name -n
                                       --resource-group -g
                                       --defer
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name | Nombre de la instancia de Azure Spring Cloud. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --defer | Almacena temporalmente el objeto en la memoria caché local en lugar de enviarlo a Azure.  Use `az cache` para ver o borrar. |

## <a name="az-spring-cloud-config-server-git-repo-remove"></a>az spring-cloud config-server git repo remove

Quita una configuración de repositorio de Git existente de Config Server.

```azurecli
az spring-cloud config-server git repo remove --name -n
                                         --repo-name
                                         --resource-group -g
                                         --defer
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name | Nombre de la instancia de Azure Spring Cloud. |
| --repo-name | URI del repositorio. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --defer | Almacena temporalmente el objeto en la memoria caché local en lugar de enviarlo a Azure.  Use `az cache` para ver o borrar. |

## <a name="az-spring-cloud-test-endpoint-disable"></a>az spring-cloud test-endpoint disable

Deshabilita el punto de conexión de prueba de Azure Spring Cloud

```azurecli
az spring-cloud test-endpoint disable --name -n
                                      --resource-group -g
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name | Nombre de la instancia de Azure Spring Cloud. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |

## <a name="az-spring-cloud-test-endpoint-enable"></a>az spring-cloud test-endpoint enable

Habilita el punto de conexión de prueba para Azure Spring Cloud. 

```azurecli
az spring-cloud test-endpoint enable --name -n
                                     --resource-group -g
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name | Nombre de la instancia de Azure Spring Cloud. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |

## <a name="az-spring-cloud-test-endpoint-list"></a>az spring-cloud test-endpoint list 

Enumera las claves de punto de conexión de prueba disponibles para Azure Spring Cloud.

```azurecli
az spring-cloud test-endpoint list --name -n
                                   --resource-group -g
                                   --app
                                   --deployment -d
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name | Nombre de la instancia de Azure Spring Cloud. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |

| Parámetros opcionales | Descripción |
| --- | :--- |
| --app | Nombre de la aplicación. |
| --deployment -d | Nombre de una implementación existente de la aplicación.  El valor predeterminado es producción, a menos que se especifique otra cosa. |

## <a name="az-spring-cloud-test-endpoint-renew-key"></a>az spring-cloud test-endpoint renew-key

Vuelve a generar una clave de punto de conexión de prueba para Azure Spring Cloud.

```azurecli
az spring-cloud test-endpoint renew-key --name -n
                                        --resource-group -g
                                        --type
```

| Parámetros obligatorios | Descripción |
| --- | :--- |
| --name | Nombre de la instancia de Azure Spring Cloud. |
| --resource-group -g | Nombre del grupo de recursos.  Puede configurar el grupo predeterminado mediante `az configure --defaults group=<name>`. |
| --type | Tipo de clave de punto de conexión de prueba.  Valores permitidos:  Principal, secundario. |
