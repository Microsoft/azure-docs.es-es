---
title: Configuración de la autenticación
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo instalar y configurar la autenticación para varios recursos y flujos de trabajo en Azure Machine Learning. Hay varias maneras de configurar y usar la autenticación dentro del servicio, que van desde la autenticación simple basada en la interfaz de usuario con fines de desarrollo o prueba, hasta la autenticación de entidad de servicio de Azure Active Directory completa.
services: machine-learning
author: larryfr
ms.author: larryfr
ms.reviewer: larryfr
ms.service: machine-learning
ms.subservice: core
ms.date: 06/17/2020
ms.topic: conceptual
ms.custom: how-to, has-adal-ref, devx-track-javascript
ms.openlocfilehash: 9d73492110703e64df5f948ad8a2a1ed8d2c63b9
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87904545"
---
# <a name="set-up-authentication-for-azure-machine-learning-resources-and-workflows"></a>Configuración de la autenticación para recursos y flujos de trabajo de Azure Machine Learning
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

Obtenga información sobre cómo autenticarse en el área de trabajo de Azure Machine Learning y en los modelos implementados como servicios web.

En general, hay dos tipos de autenticación que se pueden utilizar con Azure Machine Learning:

* __Interactiva:__ el usuario utiliza la cuenta en Azure Active Directory para autenticarse directamente o para obtener un token que se usa para la autenticación. La autenticación interactiva se usa durante la experimentación y el desarrollo iterativo o dónde quiere controlar el acceso a los recursos (por ejemplo, un servicio web) por usuario.
* __Entidad de servicio__: el usuario crea una cuenta de entidad de servicio en Azure Active Directory y la usa para autenticarse u obtener un token. Una entidad de servicio se usa cuando se necesita un proceso automatizado para autenticarse en el servicio sin necesidad de interacción del usuario. Por ejemplo, un script de implementación e integración continua que entrena y prueba un modelo cada vez que cambia el código de entrenamiento. También puede usar una entidad de servicio para recuperar un token para autenticarse en un servicio web si no quiere exigir que el usuario final del servicio se autentique. O bien cuando la autenticación del usuario final no se realiza directamente mediante Azure Active Directory.

Independientemente del tipo de autenticación que se use, el control de acceso basado en roles (RBAC) se usa para limitar el nivel de acceso permitido para los recursos. Por ejemplo, una cuenta que se usa para obtener el token de acceso para un modelo implementado solo necesita acceso de lectura al área de trabajo. Para obtener más información sobre For, consulte [Administración del acceso a un área de trabajo de Azure Machine Learning](how-to-assign-roles.md).

## <a name="prerequisites"></a>Requisitos previos

* Cree un [área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).
* [Configure un entorno de desarrollo](how-to-configure-environment.md) para instalar el SDK de Azure Machine Learning, o bien use una [máquina virtual de Azure Machine Learning Notebook](concept-azure-machine-learning-architecture.md#compute-instance) con el SDK ya instalado.

## <a name="interactive-authentication"></a>Autenticación interactiva

> [!IMPORTANT]
> En la autenticación interactiva se usa el explorador y se necesitan cookies (por ejemplo, cookies de terceros). Si ha deshabilitado las cookies, es posible que reciba un error del tipo "no se pudo iniciar sesión". Este error también puede producirse si ha habilitado la [autenticación multifactor de Azure](/azure/active-directory/authentication/concept-mfa-howitworks).

La mayoría de los ejemplos de la documentación usan la autenticación interactiva. Por ejemplo, al usar el SDK hay dos llamadas de función que muestran automáticamente un flujo de autenticación basada en la interfaz de usuario:

* La llamada a la función `from_config()` emitirá el mensaje.

    ```python
    from azureml.core import Workspace
    ws = Workspace.from_config()
    ```

    La función `from_config()` busca un archivo JSON que contenga la información de conexión del área de trabajo.

* Con el constructor `Workspace` para proporcionar información sobre la suscripción, el grupo de recursos y el área de trabajo, también se solicitará la autenticación interactiva.

    ```python
    ws = Workspace(subscription_id="your-sub-id",
                  resource_group="your-resource-group-id",
                  workspace_name="your-workspace-name"
                  )
    ```

> [!TIP]
> Si tiene acceso a varios inquilinos, es posible que tenga que importar la clase y definir explícitamente el inquilino que tiene como destino. La llamada al constructor de `InteractiveLoginAuthentication` también le pedirá que inicie sesión de forma similar a las llamadas anteriores.
>
> ```python
> from azureml.core.authentication import InteractiveLoginAuthentication
> interactive_auth = InteractiveLoginAuthentication(tenant_id="your-tenant-id")
> ```

## <a name="service-principal-authentication"></a>Autenticación de entidad de servicio

Para usar la autenticación de entidad de servicio, primero debe crear la entidad de servicio y concederle acceso al área de trabajo. Como se mencionó anteriormente, se usa el control de acceso basado en rol (RBAC) de Azure para controlar el acceso, por lo que también debe decidir qué acceso conceder a la entidad de servicio.

> [!IMPORTANT]
> Cuando use una entidad de servicio, concédale el __acceso mínimo necesario para la tarea__ para la que se usa. Por ejemplo, no debería conceder acceso a un propietario de la entidad de servicio o a un colaborador si todo para lo que se usa es para leer el token de acceso para una implementación web.
>
> La razón para conceder el menor acceso es que una entidad de servicio usa una contraseña para autenticarse y la contraseña se puede almacenar como parte de un script de automatización. Si se pierde la contraseña, tener el acceso mínimo necesario para una tarea específica minimiza el uso malintencionado de la entidad de servicio.

La forma más fácil de crear una entidad de servicio y de conceder acceso al área de trabajo es mediante la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest). Para crear una entidad de servicio y concederle acceso al área de trabajo, siga estos pasos:

> [!NOTE]
> Debe ser un administrador de la suscripción para realizar todos estos pasos.

1. Autentíquese en la suscripción de Azure:

    ```azurecli-interactive
    az login
    ```

    Si la CLI puede abrir el explorador predeterminado, lo hará y cargará una página de inicio de sesión. De lo contrario, tendrá que abrir un explorador y seguir las instrucciones de la línea de comandos. Las instrucciones implican navegar a [https://aka.ms/devicelogin](https://aka.ms/devicelogin) y escribir un código de autorización.

    [!INCLUDE [select-subscription](../../includes/machine-learning-cli-subscription.md)] 

    Para obtener otros métodos de autenticación, consulte [Inicio de sesión con la CLI de Azure](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest).

1. Instale la extensión de Azure Machine Learning:

    ```azurecli-interactive
    az extension add -n azure-cli-ml
    ```

1. Cree la entidad de servicio. En el ejemplo siguiente, se crea una entidad de servicio denominada **ml-auth**:

    ```azurecli-interactive
    az ad sp create-for-rbac --sdk-auth --name ml-auth
    ```

    La salida será un código JSON similar al siguiente. Tome nota de los campos `clientId`, `clientSecret` y `tenantId`, ya que los necesitará para otros pasos de este artículo.

    ```json
    {
        "clientId": "your-client-id",
        "clientSecret": "your-client-secret",
        "subscriptionId": "your-sub-id",
        "tenantId": "your-tenant-id",
        "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
        "resourceManagerEndpointUrl": "https://management.azure.com",
        "activeDirectoryGraphResourceId": "https://graph.windows.net",
        "sqlManagementEndpointUrl": "https://management.core.windows.net:5555",
        "galleryEndpointUrl": "https://gallery.azure.com/",
        "managementEndpointUrl": "https://management.core.windows.net"
    }
    ```

1. Recupere los detalles de la entidad de servicio mediante el valor `clientId` devuelto en el paso anterior:

    ```azurecli-interactive
    az ad sp show --id your-client-id
    ```

    El siguiente JSON es un ejemplo simplificado de la salida del comando. Tome nota del campo `objectId`, ya que necesitará su valor para el paso siguiente.

    ```json
    {
        "accountEnabled": "True",
        "addIns": [],
        "appDisplayName": "ml-auth",
        ...
        ...
        ...
        "objectId": "your-sp-object-id",
        "objectType": "ServicePrincipal"
    }
    ```

1. Permita que la entidad de servicio tenga acceso al área de trabajo de Azure Machine Learning. Necesitará el nombre del área de trabajo y el nombre de su grupo de recursos para los parámetros `-w` y `-g`, respectivamente. Para el parámetro `--user`, use el valor `objectId` del paso anterior. El parámetro `--role` permite establecer el rol de acceso para la entidad de servicio. En el ejemplo siguiente, la entidad de servicio se asigna al rol de **propietario**. 

    > [!IMPORTANT]
    > El acceso de propietario permite que la entidad de servicio realice prácticamente cualquier operación en el área de trabajo. Se usa en este documento para demostrar cómo conceder acceso. En un entorno de producción, Microsoft recomienda conceder a la entidad de servicio el acceso mínimo necesario para realizar el rol para el que se pretende usar. Para obtener más información, consulte [Administración del acceso a un área de trabajo de Azure Machine Learning](how-to-assign-roles.md).

    ```azurecli-interactive
    az ml workspace share -w your-workspace-name -g your-resource-group-name --user your-sp-object-id --role owner
    ```

    Esta llamada no produce ningún resultado en caso de éxito.

### <a name="use-a-service-principal-from-the-sdk"></a>Uso de una entidad de servicio desde el SDK

Para autenticarse en el área de trabajo desde el SDK mediante la entidad de servicio, use el constructor de clase `ServicePrincipalAuthentication`. Utilice los valores que recibió al crear el proveedor de servicios como parámetros. El parámetro `tenant_id` se asigna al campo `tenantId` anterior, `service_principal_id` se asigna a `clientId`y `service_principal_password` se asigna a `clientSecret`.

```python
from azureml.core.authentication import ServicePrincipalAuthentication

sp = ServicePrincipalAuthentication(tenant_id="your-tenant-id", # tenantID
                                    service_principal_id="your-client-id", # clientId
                                    service_principal_password="your-client-secret") # clientSecret
```

La variable `sp` ahora contiene un objeto de autenticación que se usa directamente en el SDK. En general, se recomienda almacenar los identificadores y secretos usados anteriormente en las variables de entorno, tal como se muestra en el código siguiente. El almacenamiento en variables de entorno impide que la información se compruebe accidentalmente en un repositorio de GitHub.

```python
import os

sp = ServicePrincipalAuthentication(tenant_id=os.environ['AML_TENANT_ID'],
                                    service_principal_id=os.environ['AML_PRINCIPAL_ID'],
                                    service_principal_password=os.environ['AML_PRINCIPAL_PASS'])
```

En el caso de los flujos de trabajo automatizados que se ejecutan en Python y usan el SDK principalmente, puede usar este objeto tal cual en la mayoría de los casos para la autenticación. El código siguiente se autentica en el área de trabajo mediante el objeto de autenticación que acaba de crear.

```python
from azureml.core import Workspace

ws = Workspace.get(name="ml-example",
                   auth=sp,
                   subscription_id="your-sub-id")
ws.get_details()
```

### <a name="use-a-service-principal-from-the-azure-cli"></a>Uso de una entidad de servicio desde la CLI de Azure

Puede usar una entidad de servicio para los comandos de la CLI de Azure. Para más información, vea [Inicio de sesión mediante una entidad de servicio](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest#sign-in-using-a-service-principal).

### <a name="use-a-service-principal-with-the-rest-api-preview"></a>Uso de una entidad de servicio con la API REST (versión preliminar)

La entidad de servicio también se puede usar para autenticar en la [API REST](https://docs.microsoft.com/rest/api/azureml/) de Azure Machine Learning (versión preliminar). Use el [flujo de concesión de credenciales de cliente](https://docs.microsoft.com/azure/active-directory/develop/v1-oauth2-client-creds-grant-flow) de Azure Active Directory, que permiten llamadas de servicio a servicio para la autenticación sin periféricos en flujos de trabajo automatizados. Los ejemplos se implementan con la [biblioteca de ADAL](https://docs.microsoft.com/azure/active-directory/develop/active-directory-authentication-libraries) tanto en Python como en Node.js, pero también puede usar cualquier biblioteca de código abierto que admita OpenID Connect 1.0.

> [!NOTE]
> MSAL.js es una biblioteca más reciente que ADAL, pero no puede realizar la autenticación de servicio a servicio mediante las credenciales de cliente con MSAL.js, ya que es principalmente una biblioteca del lado cliente diseñada para la autenticación interactiva o de interfaz de usuario asociada a un usuario específico. Se recomienda usar ADAL como se muestra a continuación para crear flujos de trabajo automatizados con la API REST.

#### <a name="nodejs"></a>Node.js

Siga estos pasos para generar un token de autenticación mediante Node.js. En su entorno, ejecute `npm install adal-node`. A continuación, use su `tenantId`, `clientId` y `clientSecret` de la entidad de servicio que creó en los pasos anteriores como valores para las variables coincidentes en el siguiente script.

```javascript
const adal = require('adal-node').AuthenticationContext;

const authorityHostUrl = 'https://login.microsoftonline.com/';
const tenantId = 'your-tenant-id';
const authorityUrl = authorityHostUrl + tenantId;
const clientId = 'your-client-id';
const clientSecret = 'your-client-secret';
const resource = 'https://management.azure.com/';

const context = new adal(authorityUrl);

context.acquireTokenWithClientCredentials(
  resource,
  clientId,
  clientSecret,
  (err, tokenResponse) => {
    if (err) {
      console.log(`Token generation failed due to ${err}`);
    } else {
      console.dir(tokenResponse, { depth: null, colors: true });
    }
  }
);
```

La variable `tokenResponse` es un objeto que incluye el token y los metadatos asociados, como la fecha de expiración. Los tokens son válidos durante una hora y se pueden actualizar ejecutando la misma llamada de nuevo para recuperar un nuevo token. El siguiente fragmento es una respuesta de ejemplo.

```javascript
{
    tokenType: 'Bearer',
    expiresIn: 3599,
    expiresOn: 2019-12-17T19:15:56.326Z,
    resource: 'https://management.azure.com/',
    accessToken: "random-oauth-token",
    isMRRT: true,
    _clientId: 'your-client-id',
    _authority: 'https://login.microsoftonline.com/your-tenant-id'
}
```

Use la propiedad `accessToken` para capturar el token de autenticación. Consulte la [documentación de la API REST](https://github.com/microsoft/MLOps/tree/master/examples/AzureML-REST-API) para ejemplos sobre cómo usar el token para realizar llamadas API.

#### <a name="python"></a>Python

Siga estos pasos para generar un token de autenticación mediante Python. En su entorno, ejecute `pip install adal`. A continuación, use su `tenantId`, `clientId` y `clientSecret` de la entidad de servicio que creó en los pasos anteriores como valores para las variables apropiadas en el siguiente script.

```python
from adal import AuthenticationContext

client_id = "your-client-id"
client_secret = "your-client-secret"
resource_url = "https://login.microsoftonline.com"
tenant_id = "your-tenant-id"
authority = "{}/{}".format(resource_url, tenant_id)

auth_context = AuthenticationContext(authority)
token_response = auth_context.acquire_token_with_client_credentials("https://management.azure.com/", client_id, client_secret)
print(token_response)
```

La variable `token_response` es un diccionario que incluye el token y los metadatos asociados, como la fecha de expiración. Los tokens son válidos durante una hora y se pueden actualizar ejecutando la misma llamada de nuevo para recuperar un nuevo token. El siguiente fragmento es una respuesta de ejemplo.

```python
{
    'tokenType': 'Bearer',
    'expiresIn': 3599,
    'expiresOn': '2019-12-17 19:47:15.150205',
    'resource': 'https://management.azure.com/',
    'accessToken': 'random-oauth-token',
    'isMRRT': True,
    '_clientId': 'your-client-id',
    '_authority': 'https://login.microsoftonline.com/your-tenant-id'
}
```

Use `token_response["accessToken"]` para capturar el token de autenticación. Consulte la [documentación de la API REST](https://github.com/microsoft/MLOps/tree/master/examples/AzureML-REST-API) para ejemplos sobre cómo usar el token para realizar llamadas API.

## <a name="web-service-authentication"></a>Autenticación de servicio web

Las implementaciones de modelos creadas por Azure Machine Learning proporcionan dos métodos de autenticación:

* **Basado en claves**: se usa una clave estática para autenticarse en el servicio web.
* **Basado en tokens**: debe obtenerse un token temporal desde el área de trabajo y usarlo para autenticarse en el servicio web. Este token expira tras un período de tiempo y debe actualizarse para seguir trabajando con el servicio web.

    > [!NOTE]
    > La autenticación basada en el token solo está disponible al implementarla en Azure Kubernetes Service.

### <a name="key-based-web-service-authentication"></a>Autenticación de servicio web basada en la clave

Los servicios web implementados en Azure Kubernetes Service (AKS) tienen la autenticación basada en claves *habilitada* de manera predeterminada. Los servicios implementados por Azure Container Instances (ACI) tienen la autenticación basada en claves *deshabilitada* de manera predeterminada, pero puede habilitarla estableciendo `auth_enabled=True` al crear el servicio web de ACI. El código siguiente es un ejemplo de creación de una configuración de implementación de ACI con la autenticación basada en claves habilitada.

```python
from azureml.core.webservice import AciWebservice

aci_config = AciWebservice.deploy_configuration(cpu_cores = 1,
                                                memory_gb = 1,
                                                auth_enabled=True)
```

Después, puede usar la configuración de ACI personalizada en la implementación mediante la clase `Model`.

```python
from azureml.core.model import Model, InferenceConfig


inference_config = InferenceConfig(entry_script="score.py",
                                   environment=myenv)
aci_service = Model.deploy(workspace=ws,
                       name="aci_service_sample",
                       models=[model],
                       inference_config=inference_config,
                       deployment_config=aci_config)
aci_service.wait_for_deployment(True)
```

Para capturar las claves de autenticación, use `aci_service.get_keys()`. Para volver a generar una clave, utilice la función `regen_key()` y pase **Primary** o **Secondary**.

```python
aci_service.regen_key("Primary")
# or
aci_service.regen_key("Secondary")
```

Para obtener más información sobre la autenticación en un modelo implementado, vea [Creación de un cliente para un modelo implementado como servicio web](how-to-consume-web-service.md).

### <a name="token-based-web-service-authentication"></a>Autenticación de servicio web basada en tokens

Cuando se habilita la autenticación por tokens para un servicio web, los usuarios deben presentar un token JSON Web Token de Azure Machine Learning al servicio web para tener acceso a él. El token expira después de un intervalo de tiempo especificado y debe actualizarse para seguir realizando llamadas.

* La autenticación por tokens está **deshabilitada de manera predeterminada** cuando se implementa en Azure Kubernetes Service.
* La autenticación por tokens **no se admite** cuando se implementa en Azure Container Instances.
* La autenticación por tokens **no se puede usar al mismo tiempo que la autenticación basada en claves**.

Para controlar la autenticación por tokens, use el parámetro `token_auth_enabled` cuando cree o actualice una implementación:

```python
from azureml.core.webservice import AksWebservice
from azureml.core.model import Model, InferenceConfig

# Create the config
aks_config = AksWebservice.deploy_configuration()

#  Enable token auth and disable (key) auth on the webservice
aks_config = AksWebservice.deploy_configuration(token_auth_enabled=True, auth_enabled=False)

aks_service_name ='aks-service-1'

# deploy the model
aks_service = Model.deploy(workspace=ws,
                           name=aks_service_name,
                           models=[model],
                           inference_config=inference_config,
                           deployment_config=aks_config,
                           deployment_target=aks_target)

aks_service.wait_for_deployment(show_output = True)
```

Si la autenticación por tokens está habilitada, puede usar el método `get_token` para recuperar un token JSON Web Token (JWT) y la hora de expiración del token:

> [!TIP]
> Si usa una entidad de servicio para obtener el token y quiere que tenga el acceso mínimo necesario para recuperar un token, asígnela al rol de **lector** para el área de trabajo.

```python
token, refresh_by = aks_service.get_token()
print(token)
```

> [!IMPORTANT]
> Tendrá que solicitar un nuevo token después de la hora `refresh_by` del token. Si necesita actualizar tokens fuera del SDK de Python, una opción es usar la API REST con la autenticación de entidad de servicio para hacer periódicamente la llamada `service.get_token()`, como se explicó anteriormente.
>
> Se recomienda crear el área de trabajo de Azure Machine Learning en la misma región que el clúster de Azure Kubernetes Service.
>
> Para autenticarse con un token, el servicio web hará una llamada a la región en la que se crea el área de trabajo de Azure Machine Learning. Si la región del área de trabajo no está disponible, no se podrá capturar un token para el servicio web, incluso si el clúster está en una región distinta del área de trabajo. El resultado es que la Autenticación de Azure AD no está disponible hasta que la región del área de trabajo vuelva a estar disponible.
>
> Además, cuanto mayor sea la distancia entre la región del clúster y la región del área de trabajo, más tiempo tardará la captura de un token.

## <a name="next-steps"></a>Pasos siguientes

* [Uso de secretos en ejecuciones de entrenamiento](how-to-use-secrets-in-runs.md).
* [Entrenamiento sobre el modo de clasificación de una imagen y su implementación](tutorial-train-models-with-aml.md).
* [Consumir un modelo de Azure Machine Learning que está implementado como un servicio web](how-to-consume-web-service.md).
