---
title: Introducción a la CLI de Azure para Batch
description: Obtenga una introducción rápida a los comandos de Batch en la CLI de Azure para administrar los recursos del servicio Azure Batch
ms.topic: how-to
ms.date: 07/24/2018
ms.custom: H1Hack27Feb2017, devx-track-azurecli
ms.openlocfilehash: 048dbad89893354ae78c03fc522f8ebeeb6a80f4
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87531684"
---
# <a name="manage-batch-resources-with-azure-cli"></a>Administración de recursos de Batch con la CLI de Azure

La CLI de Azure es la forma de usar la línea de comandos de Azure para administrar los recursos de Azure. Se puede usar en macOS, Linux y Windows. La CLI de Azure está optimizada para administrar recursos de Azure desde la línea de comandos. Puede usarla para administrar cuentas de Azure Batch y administrar recursos, como grupos, trabajos y tareas. Con la CLI de Azure, puede realizar mediante scripts muchas de las mismas tareas que lleva a cabo con las API de Batch, Azure Portal y los cmdlets PowerShell de Batch.

En este artículo se proporciona una introducción al uso de la [CLI de Azure versión 2.0](/cli/azure) con Batch. Consulte [Introducción a la CLI de Azure](/cli/azure/get-started-with-azure-cli) para obtener información general sobre el uso de la CLI con Azure.

## <a name="set-up-the-azure-cli"></a>Configuración de la CLI de Azure

También puede ejecutar la CLI de Azure más reciente en [Azure Cloud Shell](../cloud-shell/overview.md). Para instalar la CLI de Azure de forma local, siga los pasos descritos en [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

> [!TIP]
> Se recomienda actualizar su instalación de la CLI de Azure con frecuencia para aprovechar las mejoras y actualizaciones del servicio.
> 
> 

## <a name="command-help"></a>Ayuda de comandos

Para mostrar texto de ayuda para todos los comandos de la CLI de Azure, anexe `-h` al comando. Omita el resto de las opciones. Por ejemplo:

* Para obtener ayuda sobre el comando `az` escriba: `az -h`
* Para obtener una lista de todos los comandos de Batch en la CLI, utilice: `az batch -h`
* Para obtener ayuda acerca de la creación de una cuenta de Batch, escriba: `az batch account create -h`

En caso de duda, utilice la opción de línea de comandos `-h` para obtener ayuda sobre cualquier comando de CLI de Azure.



Además, consulte la documentación de referencia de la CLI de Azure para más información sobre los [comandos de la CLI de Azure para Batch](/cli/azure/batch). 

## <a name="log-in-and-authenticate"></a>Inicio de sesión y autenticación

Para utilizar la CLI de Azure con Batch, debe iniciar sesión y autenticarse. Hay dos pasos sencillos que seguir:

1. **Inicie sesión en Azure.** Al iniciar sesión en Azure, logra acceso a los comandos de Azure Resource Manager, incluidos los del [servicio de administración de Batch](batch-management-dotnet.md).  
2. **Inicie sesión en su cuenta de Batch.** Al iniciar sesión en su cuenta de Batch, logra acceso a los comandos del servicio Batch.   

### <a name="log-in-to-azure"></a>Inicio de sesión en Azure

Hay varias maneras diferentes de iniciar sesión en Azure, las cuales se describen en detalle en [Inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli):

1. [Inicie sesión de forma interactiva](/cli/azure/authenticate-azure-cli). Inicie sesión de forma interactiva cuando ejecute comandos de la CLI de Azure desde la línea de comandos.
2. [Inicie sesión con una entidad de servicio](/cli/azure/authenticate-azure-cli). Inicie sesión con una entidad de servicio cuando ejecute comandos de la CLI de Azure desde un script o una aplicación.

Para este artículo, se muestra cómo iniciar sesión de forma interactiva en Azure. Escriba [az login](/cli/azure/reference-index#az-login) en la línea de comandos:

```azurecli
# Log in to Azure and authenticate interactively.
az login
```

El comando `az login` devuelve un token que puede usar para autenticarse, como se muestra aquí. Siga las instrucciones proporcionadas para abrir una página web y enviar el token a Azure:

![Inicio de sesión en Azure](./media/batch-cli-get-started/az-login.png)

Los ejemplos en la sección Scripts de shell de ejemplo también muestran cómo iniciar la sesión de la CLI de Azure mediante el inicio interactivo en Azure. Una vez que haya iniciado sesión, puede llamar a comandos para trabajar con recursos de administración de Batch, incluidas cuentas de Batch, claves, paquetes de aplicación y cuotas.  

### <a name="log-in-to-your-batch-account"></a>Inicio de sesión en la cuenta de Batch

Para usar la CLI de Azure para administrar recursos de Batch, como grupos, trabajos y tareas, debe iniciar sesión en su cuenta de Batch y autenticarse. Para iniciar sesión en el servicio Batch, use el comando [az batch account login](/cli/azure/batch/account#az-batch-account-login). 

Tiene dos opciones para autenticarse en su cuenta de Batch:

- **Mediante la autenticación de Azure Active Directory (Azure AD)** 

    La autenticación con Azure AD es el valor predeterminado cuando se usa la CLI de Azure con Batch y se recomienda para la mayoría de los escenarios. 
    
    Cuando inicia sesión en Azure de forma interactiva, como se describe en la sección anterior, sus credenciales se almacenan en caché, por lo que la CLI de Azure puede iniciar sesión en su cuenta de Batch con esas mismas credenciales. Si inicia sesión Azure con una entidad de servicio, esas credenciales también se usan para iniciar sesión en su cuenta de Batch.

    Una ventaja de Azure AD es que ofrece control de acceso basado en rol (RBAC). Con RBAC, el acceso de un usuario depende de su rol asignado, y no de si posee o no las claves de cuenta. En lugar de administrar claves de cuenta, puede administrar roles de Azure y dejar que Azure AD se encargue del acceso y la autenticación.  

     Para iniciar sesión en su cuenta de Batch con Azure AD, llame al comando [az batch account login](/cli/azure/batch/account#az-batch-account-login): 

    ```azurecli
    az batch account login -g myresource group -n mybatchaccount
    ```

- **Mediante la autenticación de clave compartida**

    La [autenticación de clave compartida](/rest/api/batchservice/authenticate-requests-to-the-azure-batch-service#authentication-via-shared-key) usa las claves de acceso de su cuenta para autenticar comandos de la CLI de Azure para el servicio Batch.

    Si va a crear scripts de la CLI de Azure para automatizar la llamada a comandos de Batch, puede usar la autenticación de clave compartida o una entidad de servicio de Azure AD. En algunos escenarios, puede resultar más sencillo usar la autenticación de clave compartida que crear una entidad de servicio.  

    Para iniciar sesión con autenticación de clave compartida, incluya la opción `--shared-key-auth` en la línea de comandos:

    ```azurecli
    az batch account login -g myresourcegroup -n mybatchaccount --shared-key-auth
    ```

Los ejemplos en la sección Scripts de shell de ejemplo muestran cómo iniciar sesión en su cuenta de Batch con la CLI de Azure mediante Azure AD y la clave compartida.

## <a name="use-azure-batch-cli-extension-commands"></a>Uso de los comandos de la extensión de la CLI de Azure Batch

Si instala la extensión de la CLI de Azure Batch, puede utilizar la CLI de Azure para ejecutar trabajos completos de Batch sin escribir código. Los comandos de Batch admitidos por la extensión le permiten usar plantillas de JSON para crear grupos, trabajos y tareas con la CLI de Azure. También puede utilizar los comandos de la CLI de extensión para cargar archivos de entrada de trabajos en la cuenta de Azure Storage asociada con la cuenta de Batch y descargar los archivos de salida de trabajos. Para más información, consulte [Uso de plantillas y transferencia de archivos de la CLI de Azure Batch](batch-cli-templates.md).

## <a name="script-examples"></a>Ejemplos de script

Consulte los [ejemplos de script de la CLI](cli-samples.md) de Batch para realizar tareas comunes. Estos ejemplos incluyen muchos de los comandos disponibles en la CLI de Azure Batch para crear y administrar cuentas, grupos, trabajos y tareas. 

## <a name="json-files-for-resource-creation"></a>Archivos JSON para la creación de recursos

Al crear recursos de Batch, como grupos y trabajos, puede especificar un archivo JSON que contenga la configuración del recurso nuevo, en lugar de pasar sus parámetros como opciones de la línea de comandos. Por ejemplo:

```azurecli
az batch pool create my_batch_pool.json
```

Aunque puede crear la mayoría de los recursos de Batch solo con las opciones de la línea de comandos, algunas características requieren que especifique un archivo con formato JSON que contenga los detalles de los recursos. Por ejemplo, si desea especificar archivos de recursos para una tarea de inicio, debe utilizar un archivo JSON.

Para ver la sintaxis JSON necesaria para crear un recurso, consulte la documentación de [referencia de API REST de Batch][rest_api]. Todos los temas sobre la incorporación de *tipos de recurso* en la referencia de API de REST contienen scripts JSON de ejemplo para crear el recurso correspondiente. Puede usar esos scripts JSON de ejemplo como plantillas para los archivos JSON que se usan con la CLI de Azure. Por ejemplo, para ver la sintaxis JSON para crear grupos, consulte cómo [agregar un grupo a una cuenta][rest_add_pool].

Para un script de ejemplo que especifica un archivo JSON, consulte [Ejecutar un trabajo y tareas con Batch](./scripts/batch-cli-sample-run-job.md).

> [!NOTE]
> Si especifica un archivo JSON al crear un recurso, se pasan por alto los restantes parámetros que especifique en la línea de comandos de dicho recurso.
> 
> 

## <a name="efficient-queries-for-batch-resources"></a>Consultas eficaces para recursos de Batch

Cada tipo de recurso de proceso de Batch admite un comando `list` que consulta la cuenta de Batch y enumera los recursos de ese tipo. Por ejemplo, puede enumerar los grupos de su cuenta y las tareas de un trabajo:

```azurecli
az batch pool list
az batch task list --job-id job001
```

Cuando consulta el servicio Batch con una operación `list`, puede especificar una cláusula OData para limitar la cantidad de datos devueltos. Dado que todo el filtrado se produce en el lado servidor, solo se devuelven los datos que solicite. Use estas cláusulas para ahorrar ancho de banda (y, por consiguiente, tiempo) al realizar operaciones de lista.

En la tabla siguiente se describen las cláusulas OData admitidas por el servicio Batch:

| Cláusula | Descripción |
|---|---|
| `--select-clause [select-clause]` | Devuelve un subconjunto de propiedades para cada entidad. |
| `--filter-clause [filter-clause]` | Devuelve solo las entidades que coincidan con la expresión OData especificada. |
| `--expand-clause [expand-clause]` | Obtiene la información de la entidad en una única llamada REST subyacente. Actualmente, la cláusula expand solo admite la propiedad `stats`. |

Para un script de ejemplo que muestra cómo usar una cláusula OData, consulte [Ejecutar un trabajo y tareas con Batch](./scripts/batch-cli-sample-run-job.md).

Para más información sobre cómo realizar consultas de lista eficaces con cláusulas OData, consulte [Consulta del servicio Azure Batch con eficacia](batch-efficient-list-queries.md).

## <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

Las siguientes sugerencias pueden ayudarle cuando esté solucionando problemas de la CLI de Azure:

* Use `-h` para obtener **texto de ayuda** de cualquier comando de CLI.
* Use `-v` y `-vv` para mostrar la salida **detallada** de comandos. Cuando se incluye la marca `-vv`, la CLI de Azure muestra las solicitudes y respuestas REST en sí. Estos modificadores son útiles para mostrar la salida completa del error.
* Puede ver la **salida del comando como JSON** con la opción `--json`. Por ejemplo, `az batch pool show pool001 --json` muestra las propiedades de pool001 en formato JSON. Puede copiar y modificar esta salida para usarla en `--json-file` (consulte Archivos JSON en este mismo artículo).
<!---Loc Comment: Please, check link [JSON files] since it's not redirecting to any location.--->

## <a name="next-steps"></a>Pasos siguientes

* Consulte la [documentación de la CLI de Azure](/cli/azure).
* Conozca el [flujo de trabajo y los recursos principales del servicio Batch](batch-service-workflow-features.md), como grupos, nodos, trabajos y tareas.
* Aprenda sobre el uso de plantillas de Batch para crear grupos, trabajos y tareas sin escribir código en [Uso de plantillas y transferencia de archivos de la CLI de Azure Batch](batch-cli-templates.md).

[github_readme]: https://github.com/Azure/azure-xplat-cli/blob/dev/README.md
[rest_api]: /rest/api/batchservice/
[rest_add_pool]: /rest/api/batchservice/pool/add
