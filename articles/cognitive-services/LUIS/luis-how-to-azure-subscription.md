---
title: Cómo usar las claves de creación y tiempo de ejecución - LUIS
description: La primera vez que use Language Understanding (LUIS), no es necesario crear una clave de suscripción. Cuando quiera publicar la aplicación, use el punto de conexión en tiempo de ejecución; a continuación, debe crear y asignar la clave de tiempo de ejecución a la aplicación.
services: cognitive-services
ms.topic: how-to
ms.date: 07/07/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 6bd8cc807a393d6c8027f5990b9897d93f2b78d2
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496906"
---
# <a name="create-luis-resources"></a>Creación de recursos de LUIS

Los recursos de creación y de tiempo de ejecución de predicción de consultas proporcionan autenticación a la aplicación LUIS y al punto de conexión de predicción.

<a name="azure-resources-for-luis"></a>
<a name="programmatic-key" ></a>
<a name="endpoint-key"></a>
<a name="authoring-key"></a>

## <a name="luis-resources"></a>Recursos de LUIS

LUIS permite tres tipos de recursos de Azure y uno que no es de Azure:

|Clave|Propósito|Cognitive Service `kind`|Cognitive Service `type`|
|--|--|--|--|
|Clave de creación|Acceda y administre los datos de la aplicación con la creación, el entrenamiento, la publicación y las pruebas. Cree una clave de creación de LUIS si tiene previsto crear aplicaciones de LUIS mediante programación.<br><br>El propósito de la clave de `LUIS.Authoring` es permitirle:<br>* administrar mediante programación las aplicaciones y los modelos de Language Understanding, incluido el entrenamiento y la publicación<br> * controlar los permisos para el recurso de creación mediante la asignación de usuarios al [rol de colaborador](#contributions-from-other-authors).|`LUIS.Authoring`|`Cognitive Services`|
|Clave de predicción de consulta| Consulte las solicitudes del punto de conexión de predicción. Cree una clave de predicción de LUIS antes de que la aplicación cliente solicite predicciones más allá de las 1000 solicitudes proporcionadas por el recurso de inicio. |`LUIS`|`Cognitive Services`|
|[Clave de recursos de varios servicios de Cognitive Services](../cognitive-services-apis-create-account-cli.md?tabs=windows#create-a-cognitive-services-resource)|Consulte las solicitudes de punto de conexión de predicción compartidas con LUIS y otros servicios de Cognitive Services admitidos.|`CognitiveServices`|`Cognitive Services`|
|Inicio|Creación gratuita (sin control de acceso basado en rol) a través del portal o las API de LUIS (incluidos los SDK), 1000 solicitudes gratuitas de punto de conexión de predicción al mes a través de un explorador, una API o SDK|-|No es un recurso de Azure|

Cuando finalice el proceso de creación de recursos de Azure, [asigne la clave](#assign-a-resource-to-an-app) a la aplicación del portal de LUIS.

Es importante crear las aplicaciones de LUIS en [regiones](luis-reference-regions.md#publishing-regions) en las que también quiera publicar y consultar contenido.

## <a name="resource-ownership"></a>Propiedad de recurso

Un recurso de Azure, como LUIS, pertenece a la suscripción que contiene el recurso.

Para transferir la propiedad de un recurso, puede:
* Transferir la [propiedad](../../cost-management-billing/manage/billing-subscription-transfer.md) de la suscripción.
* Exportar la aplicación LUIS como archivo y, a continuación, importar la aplicación en una suscripción diferente. La exportación está disponible en la página **Mis aplicaciones** del portal LUIS.


## <a name="resource-limits"></a>Límites de recursos

### <a name="authoring-key-creation-limits"></a>Limites al establecimiento de claves de creación

Puede crear hasta 10 claves de creación por región y por suscripción.

Consulte [Límites de la clave](luis-limits.md#key-limits) y [Regiones de Azure](luis-reference-regions.md).

Las regiones de publicación son diferentes de las regiones de creación. Asegúrese de que crea una aplicación en la región de creación correspondiente a la región de publicación donde quiera ubicar su aplicación cliente.

### <a name="key-usage-limit-errors"></a>Errores de límite de uso de claves

El límite de uso depende del plan de tarifa.

Si supera su cuota de transacciones por segundo (TPS), recibirá un error HTTP 429. Si supera su cuota de transacciones por mes (TPS), recibirá un error HTTP 403.


### <a name="reset-authoring-key"></a>Restablecimiento de la clave de creación

Para [crear aplicaciones migradas de recursos](luis-migration-authoring.md): si la clave de creación se ve comprometida, restablezca la clave en Azure Portal en la página **Claves** para ese recurso de creación.

En el caso de las aplicaciones que no se han migrado todavía: la clave se restablece en todas las aplicaciones del portal de LUIS. Si crea sus aplicaciones a través de las API de creación, debe cambiar el valor de Ocp-Apim-Subscription-Key por la clave nueva.

### <a name="regenerate-azure-key"></a>Regeneración de la clave de Azure

Vuelva a generar las claves de Azure desde Azure Portal, en la página de **claves**.


<a name="securing-the-endpoint"></a>

## <a name="app-ownership-access-and-security"></a>Propiedad, acceso y seguridad de la aplicación

Una aplicación se define mediante sus recursos de Azure, que se determinan en función de la suscripción del propietario.

Puede mover la aplicación de LUIS. Use los siguientes recursos de documentación en Azure Portal o la CLI de Azure:

* [Mover la aplicación entre los recursos de creación de LUIS](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-move-app-to-another-luis-authoring-azure-resource)
* [Mover los recursos a un nuevo grupo de recursos o a una nueva suscripción](../../azure-resource-manager/management/move-resource-group-and-subscription.md)
* [Mueve el recurso dentro de la misma suscripción o entre suscripciones](../../azure-resource-manager/management/move-limitations/app-service-move-limitations.md)


### <a name="contributions-from-other-authors"></a>Contribuciones de otros autores

Para la [creación de aplicaciones migradas de recursos](luis-migration-authoring.md): los _colaboradores_ se administran en Azure Portal para el recurso de creación, mediante la página del **control de acceso (IAM)** . Obtenga información acerca de [cómo agregar un usuario](luis-how-to-collaborate.md) mediante la dirección de correo electrónico del colaborador y el rol del _colaborador_.

En el caso de las aplicaciones que no se han migrado todavía: todos los _colaboradores_ se administran en el portal de LUIS desde la página **Manage -> Collaborators** (Administrar -> Colaboradores).

### <a name="query-prediction-access-for-private-and-public-apps"></a>Acceso de predicción de consultas para aplicaciones públicas y privadas

Para una aplicación **privada**, el acceso al tiempo de ejecución de predicción de consultas está disponible para los propietarios y los colaboradores. Para una aplicación **pública**, el acceso al tiempo de ejecución está disponible para todos los usuarios que tengan su propio recurso de tiempo de ejecución de Azure [Cognitive Services](../cognitive-services-apis-create-account.md) o de [LUIS](#create-resources-in-the-azure-portal) y el identificador de la aplicación pública.

Actualmente no hay un catálogo de aplicaciones públicas.

### <a name="authoring-permissions-and-access"></a>Creación de permisos y accesos
El acceso a la aplicación desde el portal de [LUIS](luis-reference-regions.md#luis-website) o las [API de creación](https://go.microsoft.com/fwlink/?linkid=2092087) lo controla el recurso de creación de Azure.

El propietario y todos los colaboradores tienen acceso a la creación de la aplicación.

|El acceso de creación incluye|Notas|
|--|--|
|Agregar o quitar claves del punto de conexión||
|Exportar la versión||
|Exportar los registros de punto de conexión||
|Importar la versión||
|Hacer pública la aplicación|Cuando una aplicación es pública, cualquiera que tenga una clave de creación o del punto de conexión puede consultar la aplicación.|
|Modificar el modelo|
|Publicar|
|Revisar las expresiones de punto de conexión para un [aprendizaje activo](luis-how-to-review-endpoint-utterances.md)|
|Train|

<a name="prediction-endpoint-runtime-key"></a>

### <a name="prediction-endpoint-runtime-access"></a>Acceso de tiempo de ejecución del punto de conexión de predicción

El acceso para consultar el punto de conexión de predicción se controla mediante una opción de configuración de la página **Application Information** (Información de la aplicación) en la sección **Manage** (Administrar).

|[Punto de conexión privado](#runtime-security-for-private-apps)|[Punto de conexión público](#runtime-security-for-public-apps)|
|:--|:--|
|Disponible para el propietario y los colaboradores.|Disponible para el propietario, los colaboradores y cualquier otra persona que conozca el identificador de la aplicación.|

Puede controlar quién puede ver su clave de tiempo de ejecución de LUIS llamándola en un entorno de servidor a servidor. Si usa LUIS desde un bot, la conexión entre el bot y LUIS ya es segura. Si está llamando al punto de conexión de LUIS directamente, debe crear una API de servidor (por ejemplo, una [función](https://azure.microsoft.com/services/functions/) de Azure) con acceso controlado (como [AAD](https://azure.microsoft.com/services/active-directory/)). Cuando se llama a la API del lado servidor y se comprueba la autenticación y la autorización, pase la llamada a LUIS. Aunque esta estrategia no impide los ataques de tipo "Man in the middle", ofusca la clave y la dirección URL del punto de conexión de los usuarios, le permite realizar un seguimiento del acceso y agregar un registro de respuesta del punto de conexión (como [Application Insights](https://azure.microsoft.com/services/application-insights/)).

### <a name="runtime-security-for-private-apps"></a>Seguridad del tiempo de ejecución para aplicaciones privadas

El tiempo de ejecución de una aplicación privada solo está disponible para lo siguiente:

|Clave y usuario|Explicación|
|--|--|
|Clave de creación del propietario| Hasta 1000 visitas del punto de conexión|
|Claves de creación de colaborador| Hasta 1000 visitas del punto de conexión|
|Cualquier clave que un autor o colaborador haya asignado a LUIS|Según el nivel de uso de las claves|

### <a name="runtime-security-for-public-apps"></a>Seguridad del tiempo de ejecución para aplicaciones públicas

Una vez que una aplicación se ha configurado como pública, _cualquier_ clave de creación de LUIS válida o clave del punto de conexión de LUIS puede consultar la aplicación, siempre y cuando la clave no haya alcanzado el límite de cuota del punto de conexión.

Un usuario que no sea propietario ni colaborador solo podrá obtener acceso al tiempo de ejecución de una aplicación pública si se le proporciona el identificador de la aplicación. LUIS no tiene un _mercado_ público u otro medio para buscar una aplicación pública.

Una aplicación pública se pone a disposición de los usuarios en todas las regiones para que los que tengan una clave de recurso de LUIS basada en regiones puedan acceder a la aplicación en cualquier región que esté asociada a la clave de recurso.


### <a name="securing-the-query-prediction-endpoint"></a>Protección del punto de conexión de predicción de consulta

Puede controlar quién puede ver su clave del punto de conexión de tiempo de ejecución de predicción de LUIS llamándola a un entorno de servidor a servidor. Si usa LUIS desde un bot, la conexión entre el bot y LUIS ya es segura. Si está llamando al punto de conexión de LUIS directamente, debe crear una API de servidor (por ejemplo, una [función](https://azure.microsoft.com/services/functions/) de Azure) con acceso controlado (como [AAD](https://azure.microsoft.com/services/active-directory/)). Cuando se llama a la API de servidor y se comprueba la autenticación y la autorización, pase la llamada a LUIS. Aunque esta estrategia no impide los ataques de tipo "Man in the middle", ofusca el punto de conexión de los usuarios, le permite realizar un seguimiento de acceso y agregar un registro de respuesta del punto de conexión (como [Application Insights](https://azure.microsoft.com/services/application-insights/)).

<a name="starter-key"></a>

## <a name="sign-in-to-luis-portal-and-begin-authoring"></a>Inicio de sesión en el portal de LUIS y comience el proceso de creación

1. Inicie sesión en el [portal de LUIS](https://www.luis.ai) y acepte las condiciones de uso.
1. Comience su aplicación LUIS mediante la elección de la clave de creación de LUIS de Azure.

   ![Elección de un tipo de recurso de creación de Language Understanding](./media/luis-how-to-azure-subscription/sign-in-create-resource.png)

1. Cuando haya terminado con el proceso de selección de recursos, [cree una aplicación nueva](luis-how-to-start-new-app.md#create-new-app-in-luis).


<a name="create-azure-resources"></a>
<a name="create-resources-in-the-azure-portal"></a>

[!INCLUDE [Create LUIS resource in Azure portal](includes/create-luis-resource.md)]

### <a name="create-resources-in-azure-cli"></a>Creación de recursos en la CLI de Azure

Use la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) para crear cada recursos de manera individual.

Recurso: `kind`:

* Creación: `LUIS.Authoring`
* Predicción: `LUIS`

1. Inicie sesión en la CLI de Azure.

    ```azurecli
    az login
    ```

    Se abre un explorador que le permite seleccionar la cuenta correcta y proporcionar autenticación.

1. Cree un **recurso de creación de LUIS**, de tipo `LUIS.Authoring`, llamado `my-luis-authoring-resource` en el grupo de recursos _existente_ denominado `my-resource-group` en la región `westus`.

    ```azurecli
    az cognitiveservices account create -n my-luis-authoring-resource -g my-resource-group --kind LUIS.Authoring --sku F0 -l westus --yes
    ```

1. Cree un **recurso de punto de conexión de predicción de LUIS**, de tipo `LUIS`, llamado `my-luis-prediction-resource` en el grupo de recursos _existente_ denominado `my-resource-group` en la región `westus`. Si quiere un rendimiento superior al del nivel gratuito, cambie `F0` por `S0`. Más información sobre [planes de tarifa y rendimiento](luis-limits.md#key-limits).

    ```azurecli
    az cognitiveservices account create -n my-luis-prediction-resource -g my-resource-group --kind LUIS --sku F0 -l westus --yes
    ```

    > [!Note]
    > Estas claves **no** se usan en el portal de LUIS hasta que se asignan en dicho portal en **Manage -> Azure resources** (Administrar > Recursos de Azure).

<a name="assign-an-authoring-resource-in-the-luis-portal-for-all-apps"></a>

### <a name="assign-resource-in-the-luis-portal"></a>Asignación del recurso en el portal de LUIS

Puede asignar un recurso de creación de una sola aplicación o de todas las aplicaciones en LUIS. En el siguiente procedimiento se asignan todas las aplicaciones a un único recurso de creación.

1. Inicie sesión en el [portal de LUIS](https://www.luis.ai).
1. En la barra de navegación superior, en el extremo derecho, seleccione su cuenta de usuario y, a continuación, seleccione **Settings** (Configuración).
1. En la página de **configuración de usuario**, seleccione **Add authoring resource** (Agregar recurso de creación) y, a continuación, seleccione un recurso de creación existente. Seleccione **Guardar**.

## <a name="assign-a-resource-to-an-app"></a>Asignar un recurso a una aplicación

Puede asignar un recurso a una aplicación con el siguiente procedimiento.

1. Inicie sesión en el [portal de LUIS](https://www.luis.ai) y seleccione una aplicación de la lista **My apps** (Mis aplicaciones).
1. Vaya a la página **Manage -> Azure resources** (Administrar -> Recursos de Azure).

    ![Seleccione Manage -> Azure resources (Administrar -> Recursos de Azure) en el portal de LUIS para asignar un recurso a la aplicación.](./media/luis-how-to-azure-subscription/manage-azure-resources-prediction.png)

1. Seleccione la pestaña de predicción o de recursos de creación y, a continuación, seleccione el botón **Add prediction resource** (Agregar recurso de predicción) o **Add authoring resource** (Agregar recurso de creación).
1. Seleccione los campos en el formulario para buscar el recurso correcto y, a continuación, seleccione **Save** (Guardar).

### <a name="assign-query-prediction-runtime-resource-without-using-luis-portal"></a>Asignación de recursos de tiempo de ejecución de predicción de consultas sin usar el portal de LUIS

Para fines de automatización, como una canalización de CI/CD, puede automatizar la asignación de un recurso de tiempo de ejecución de LUIS a una aplicación de LUIS. Para ello, tiene que realizar los siguientes pasos:

1. Obtenga un token de Azure Resource Manager en este [sitio web](https://resources.azure.com/api/token?plaintext=true). Este token expira, por lo que debe usarlo de inmediato. La solicitud devuelve un token de Azure Resource Manager.

    ![Solicitud de un token de Azure Resource Manager y recepción del token de Azure Resource Manager](./media/luis-manage-keys/get-arm-token.png)

1. Use el token para solicitar los recursos de tiempo de ejecución de LUIS en distintas suscripciones desde la [API Get LUIS Azure accounts](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5be313cec181ae720aa2b26c), a la cual su cuenta de usuario tiene acceso.

    Esta API POST requiere la siguiente configuración:

    |Encabezado|Value|
    |--|--|
    |`Authorization`|El valor de `Authorization` es `Bearer {token}`. Tenga en cuenta que el valor del token debe ir precedido de la palabra `Bearer` y un espacio.|
    |`Ocp-Apim-Subscription-Key`|Su clave de creación.|

    Esta API devuelve una matriz de objetos JSON con las suscripciones de LUIS, incluidos el identificador de suscripción, el grupo de recursos y el nombre del recurso, que se devuelve como nombre de cuenta. Busque el elemento de la matriz que es el recurso de LUIS que se va a asignar a la aplicación de LUIS.

1. Asigne el token al recurso de LUIS con la API [Assign a LUIS Azure accounts to an application](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5be32228e8473de116325515).

    Esta API POST requiere la siguiente configuración:

    |Tipo|Configuración|Value|
    |--|--|--|
    |Encabezado|`Authorization`|El valor de `Authorization` es `Bearer {token}`. Tenga en cuenta que el valor del token debe ir precedido de la palabra `Bearer` y un espacio.|
    |Encabezado|`Ocp-Apim-Subscription-Key`|Su clave de creación.|
    |Encabezado|`Content-type`|`application/json`|
    |QueryString|`appid`|El id. de la aplicación LUIS.
    |Body||{"AzureSubscriptionId":"ddda2925-af7f-4b05-9ba1-2155c5fe8a8e",<br>"ResourceGroup": "resourcegroup-2",<br>"AccountName": "luis-uswest-S0-2"}|

    Cuando esta API finaliza correctamente, devuelve un estado 201: creado.

## <a name="unassign-resource"></a>Anulación de la asignación de un recurso

1. Inicie sesión en el [portal de LUIS](https://www.luis.ai) y seleccione una aplicación de la lista **My apps** (Mis aplicaciones).
1. Vaya a la página **Manage -> Azure resources** (Administrar -> Recursos de Azure).
1. Seleccione la pestaña de predicción o de recursos de creación y, a continuación, seleccione el botón **Unassign resource** (Cancelar asignación del recurso).

Al cancelar la asignación de un recurso, no se elimina de Azure. Solo se desvincula de LUIS.


## <a name="delete-account"></a>Eliminación de cuenta

Vea [Data storage and removal](luis-concept-data-storage.md#accounts) (Almacenamiento y eliminación de datos) para obtener información acerca de qué datos se eliminan al eliminar la cuenta.

## <a name="change-pricing-tier"></a>Cambiar el plan de tarifa

1.  En [Azure](https://portal.azure.com), busque la suscripción de LUIS. Seleccione la suscripción de LUIS.
    ![Buscar la suscripción de LUIS](./media/luis-usage-tiers/find.png)
1.  Seleccione **Plan de tarifa** para ver los planes de tarifa disponibles.
    ![Ver los planes de tarifa](./media/luis-usage-tiers/subscription.png)
1.  Seleccione el plan de tarifa y después **Seleccionar** para guardar el cambio.
    ![Cambiar el plan de pago de LUIS](./media/luis-usage-tiers/plans.png)
1.  Una vez completado el cambio de tarifa, el plan de tarifa nuevo se comprueba en una ventana emergente.
    ![Comprobar el plan de pago de LUIS](./media/luis-usage-tiers/updated.png)
1. No olvide [asignar esta clave de punto de conexión](#assign-a-resource-to-an-app) en la página **Publicar** y usarla en todas las consultas de punto de conexión.

## <a name="viewing-azure-resource-metrics"></a>Visualización de las métricas de recursos de Azure

### <a name="viewing-azure-resource-summary-usage"></a>Visualización del uso de resumen de los recursos de Azure
En Azure puede ver la información de uso de LUIS. En la página **Información general** se muestra información de resumen reciente incluidos los errores y las llamadas. Si realiza una solicitud de punto de conexión de LUIS, consulte inmediatamente la **página Información general**, y deje que pasen hasta cinco minutos hasta que se muestre el uso.

![Ver uso de resumen](./media/luis-usage-tiers/overview.png)

### <a name="customizing-azure-resource-usage-charts"></a>Personalización de los gráficos de uso de los recursos de Azure
Las métricas proporcionan una vista más detallada de los datos.

![Métricas predeterminadas](./media/luis-usage-tiers/metrics-default.png)

Puede configurar los gráficos de métricas para el período de tiempo y el tipo de métrica.

![Métricas personalizadas](./media/luis-usage-tiers/metrics-custom.png)

### <a name="total-transactions-threshold-alert"></a>Alerta de umbral de transacciones totales
Si le gustaría saber cuándo se alcanza un umbral de transacciones determinado (por ejemplo 10.000) puede crear una alerta.

![Alertas predeterminadas](./media/luis-usage-tiers/alert-default.png)

Agregue una alerta de métrica para la métrica **Llamadas totales** para un período de tiempo concreto. Agregue las direcciones de correo electrónico de todos los usuarios que deben recibir la alerta. Agregue webhooks para todos los sistemas que deben recibir la alerta. También puede ejecutar una aplicación lógica cuando se desencadene la alerta.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre [cómo usar versiones](luis-how-to-manage-versions.md) para controlar el ciclo de vida de la aplicación.
* Migre al nuevo [recurso de creación](luis-migration-authoring.md).
