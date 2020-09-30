---
title: Acceso seguro y datos
description: Proteja el acceso a las entradas, las salidas, los desencadenadores basados en solicitudes, el historial de ejecución, las tareas de administración y el acceso a otros recursos en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: rarayudu, logicappspm
ms.topic: conceptual
ms.date: 08/27/2020
ms.openlocfilehash: 9dc63de56a71ab07f513efefe2cf068f6a7be7b0
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89276055"
---
# <a name="secure-access-and-data-in-azure-logic-apps"></a>Proteger el acceso y los datos en Azure Logic Apps

Azure Logic Apps se basa en [Azure Storage](../storage/index.yml) para almacenar y [cifrar los datos en reposo](../security/fundamentals/encryption-atrest.md) automáticamente. Este cifrado protege los datos y le ayuda a cumplir los compromisos de cumplimiento y seguridad de la organización. De forma predeterminada, Azure Storage usa claves que administra Microsoft para cifrar sus datos. Para más información, consulte [Cifrado de Azure Storage para datos en reposo](../storage/common/storage-service-encryption.md).

Para controlar el acceso y proteger los datos confidenciales en Azure Logic Apps aún más, puede configurar seguridad adicional en estas áreas:

* [Acceso de las llamadas entrantes a desencadenadores basados en solicitud](#secure-inbound-requests)
* [Acceso a las operaciones de las aplicaciones lógicas](#secure-operations)
* [Acceso a las entradas y salidas del historial de ejecución](#secure-run-history)
* [Acceso a las entradas de parámetros](#secure-action-parameters)
* [Acceso de las llamadas salientes a otros servicios y sistemas](#secure-outbound-requests)
* [Bloquear la creación de conexiones para conectores específicos](#block-connections)
* [Guía de aislamiento para aplicaciones lógicas](#isolation-logic-apps)
* [Base de referencia de seguridad de Azure para Azure Logic Apps](../logic-apps/security-baseline.md)

Para obtener más información sobre la seguridad en Azure, consulte estos temas:

* [Información general del cifrado de Azure](../security/fundamentals/encryption-overview.md)
* [Cifrado en reposo de datos de Azure](../security/fundamentals/encryption-atrest.md)
* [Azure Security Benchmark](../security/benchmarks/overview.md)

<a name="secure-inbound-requests"></a>

## <a name="access-for-inbound-calls-to-request-based-triggers"></a>Acceso de las llamadas entrantes a desencadenadores basados en solicitud

Las llamadas entrantes que una aplicación lógica recibe a través de un desencadenador basado en solicitud, como el desencadenador [Request](../connectors/connectors-native-reqres.md) (Solicitud) o el desencadenador [HTTP Webhook](../connectors/connectors-native-webhook.md) (Webhook de HTTP), admiten el cifrado y se protegen con la [versión 1.2 de Seguridad de la capa de transporte (TLS), como mínimo](https://en.wikipedia.org/wiki/Transport_Layer_Security), que antes se conocía como Capa de sockets seguros (SSL). Logic Apps aplica esta versión al recibir una llamada entrante al desencadenador Request (Solicitud) o una devolución de llamada al desencadenador o acción HTTP Webhook (Webhook de HTTP). Si obtiene errores de protocolo de enlace TLS, asegúrese de usar TLS 1.2. Para más información, consulte [Solución del problema de TLS 1.0](/security/solving-tls1-problem).

Las llamadas de entrada admiten estos conjuntos de cifrado:

* TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
* TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
* TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
* TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
* TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
* TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
* TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
* TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256

Estas son otras formas de limitar el acceso a los desencadenadores que reciben llamadas entrantes a la aplicación lógica para que solo los clientes autorizados puedan llamar a la aplicación lógica:

* [Generación de firmas de acceso compartido (SAS)](#sas)
* [Habilitación de Azure Active Directory Open Authentication (Azure AD OAuth)](#enable-oauth)
* [Exposición de su aplicación lógica con Azure API Management](#azure-api-management)
* [Direcciones IP entrantes restringidas](#restrict-inbound-ip-addresses)

<a name="sas"></a>

### <a name="generate-shared-access-signatures-sas"></a>Generación de firmas de acceso compartido (SAS)

Cada de punto de conexión de la solicitud de una aplicación lógica tiene una [Firma de acceso compartido (SAS)](/rest/api/storageservices/constructing-a-service-sas) en la dirección URL del punto de conexión con el formato siguiente:

`https://<request-endpoint-URI>sp=<permissions>sv=<SAS-version>sig=<signature>`

Cada dirección URL contiene el parámetro de consulta `sp`, `sv` y `sig`, tal y como se describe en esta tabla:

| Parámetro de consulta | Descripción |
|-----------------|-------------|
| `sp` | Especifica los permisos para los métodos HTTP permitidos que se usarán. |
| `sv` | Especifica la versión de SAS que se usará para generar la firma. |
| `sig` | Especifica la firma que se usará para autenticar el acceso al desencadenador. Esta firma se genera mediante el algoritmo SHA256 con una clave de acceso secreta en todas las rutas de acceso de direcciones URL y propiedades. Esta clave nunca se expone ni se publica, y se mantiene cifrada y almacenada con la aplicación lógica. La aplicación lógica solo autoriza los desencadenadores que contienen una firma válida creada con la clave secreta. |
|||

Para obtener más información sobre la protección del acceso con la Firma de acceso compartido, consulte estas secciones del tema:

* [Regenerar las claves de acceso](#access-keys)
* [Crear direcciones URL de devolución de llamada de expiración próxima](#expiring-urls)
* [Crear direcciones URL con clave principal o secundaria](#primary-secondary-key)

<a name="access-keys"></a>

#### <a name="regenerate-access-keys"></a>Regenerar las claves de acceso

Para generar una nueva clave de acceso de seguridad en cualquier momento, use la API REST de Azure o Azure Portal. Se invalidan todas las direcciones URL generadas previamente que usan la clave anterior y ya no tienen autorización para desencadenar la aplicación lógica. Las direcciones URL que se recuperan tras la regeneración se firman con la nueva clave de acceso.

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica donde está la clave que quiere regenerar.

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Claves de acceso**.

1. Seleccione la clave que quiere regenerar y finalice el proceso.

<a name="expiring-urls"></a>

#### <a name="create-expiring-callback-urls"></a>Crear direcciones URL de devolución de llamada de expiración próxima

Si comparte la dirección URL del punto de conexión de un desencadenador basado en solicitudes con otras partes, puede generar direcciones URL de devolución de llamada con claves específicas y fechas de expiración. De esta forma puede acumular claves fácilmente o restringir el acceso para desencadenar la aplicación lógica en función de un determinado intervalo de tiempo. Para especificar una fecha de expiración para una dirección URL, use la [API REST de Logic Apps](/rest/api/logic/workflowtriggers); por ejemplo:

```http
POST /subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.Logic/workflows/<workflow-name>/triggers/<trigger-name>/listCallbackUrl?api-version=2016-06-01
```

En el cuerpo, incluya la propiedad `NotAfter` mediante una cadena de fecha de JSON. Esta propiedad devuelve una dirección URL de devolución de llamada que solo es válida hasta la fecha y hora `NotAfter`.

<a name="primary-secondary-key"></a>

#### <a name="create-urls-with-primary-or-secondary-secret-key"></a>Creación de direcciones URL con clave secreta principal o secundaria

Al generar o enumerar direcciones URL de devolución de llamada para desencadenadores basados en solicitudes, puede especificar qué clave se va a usar para firmar la dirección URL. Para generar una dirección URL firmada por una clave específica, use la [API REST de Logic Apps](/rest/api/logic/workflowtriggers); por ejemplo:

```http
POST /subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.Logic/workflows/<workflow-name>/triggers/<trigger-name>/listCallbackUrl?api-version=2016-06-01
```

En el cuerpo, incluya la propiedad `KeyType` como `Primary` o `Secondary`. Esta propiedad devuelve una dirección URL firmada por una clave de seguridad especificada.

<a name="enable-oauth"></a>

### <a name="enable-azure-active-directory-open-authentication-azure-ad-oauth"></a>Habilitación de Azure Active Directory Open Authentication (Azure AD OAuth)

Si la aplicación lógica se inicia con un [desencadenador Request](../connectors/connectors-native-reqres.md) (Solicitud), puede habilitar [Azure Active Directory Open Authentication (Azure AD OAuth)](../active-directory/develop/index.yml) mediante la definición o incorporación de una directiva de autorización para las llamadas entrantes al desencadenador Request (Solicitud).

Antes de habilitar esta autenticación, tenga en cuenta los siguientes aspectos:

* La llamada entrante al desencadenador Request (Solicitud) solo puede usar un esquema de autorización, ya sea Azure AD OAuth mediante un token de autenticación, que solo es compatible con el desencadenador Request (Solicitud) o mediante una dirección URL de [firma de acceso compartido (SAS)](#sas). Ambos esquemas no se pueden usar.

  Aunque el uso de un esquema no deshabilita el otro, si se utilizan ambos al mismo tiempo se produce un error porque el servicio no sabe qué esquema elegir. Además, solo se admiten esquemas de autorización [Bearer-type](../active-directory/develop/active-directory-v2-protocols.md#tokens) para los tokens de autenticación OAuth, que solo se admiten para el desencadenador Request (Solicitud). El token de autenticación debe especificar `Bearer-type` en el encabezado de autorización.

* La aplicación lógica está limitada a un número máximo de directivas de autorización. Cada directiva de autorización también tiene un número máximo de [notificaciones](../active-directory/develop/developer-glossary.md#claim). Para más información, consulte el artículo de [límites y configuración para Azure Logic Apps](../logic-apps/logic-apps-limits-and-config.md#authentication-limits).

* Una directiva de autorización debe incluir al menos la notificación de **Emisor**, que tiene un valor que comienza con `https://sts.windows.net/` o `https://login.microsoftonline.com/` (OAuth V2) como identificador de emisor de Azure AD. Para obtener más información sobre los tokens de acceso, consulte [Tokens de acceso de la Plataforma de identidad de Microsoft](../active-directory/develop/access-tokens.md).

Cuando la aplicación lógica recibe una solicitud entrante que incluye un token de autenticación OAuth, Azure Logic Apps compara las notificaciones del token con las de cada directiva de autorización. Si existe una coincidencia entre las notificaciones del token y todas las notificaciones en al menos una directiva, la autorización de la solicitud entrante se realiza correctamente. El token puede tener más notificaciones que el número especificado por la directiva de autorización.

Por ejemplo, supongamos que la aplicación lógica tiene una directiva de autorización que requiere dos tipos de notificaciones: **emisor** y **audiencia**. Este [token de acceso](../active-directory/develop/access-tokens.md) descodificado de ejemplo incluye ambos tipos de notificaciones:

```json
{
   "aud": "https://management.core.windows.net/",
   "iss": "https://sts.windows.net/<Azure-AD-issuer-ID>/",
   "iat": 1582056988,
   "nbf": 1582056988,
   "exp": 1582060888,
   "_claim_names": {
      "groups": "src1"
   },
   "_claim_sources": {
      "src1": {
         "endpoint": "https://graph.windows.net/7200000-86f1-41af-91ab-2d7cd011db47/users/00000-f433-403e-b3aa-7d8406464625d7/getMemberObjects"
    }
   },
   "acr": "1",
   "aio": "AVQAq/8OAAAA7k1O1C2fRfeG604U9e6EzYcy52wb65Cx2OkaHIqDOkuyyr0IBa/YuaImaydaf/twVaeW/etbzzlKFNI4Q=",
   "amr": [
      "rsa",
      "mfa"
   ],
   "appid": "c44b4083-3bb0-00001-b47d-97400853cbdf3c",
   "appidacr": "2",
   "deviceid": "bfk817a1-3d981-4dddf82-8ade-2bddd2f5f8172ab",
   "family_name": "Sophia Owen",
   "given_name": "Sophia Owen (Fabrikam)",
   "ipaddr": "167.220.2.46",
   "name": "sophiaowen",
   "oid": "3d5053d9-f433-00000e-b3aa-7d84041625d7",
   "onprem_sid": "S-1-5-21-2497521184-1604012920-1887927527-21913475",
   "puid": "1003000000098FE48CE",
   "scp": "user_impersonation",
   "sub": "KGlhIodTx3XCVIWjJarRfJbsLX9JcdYYWDPkufGVij7_7k",
   "tid": "72f988bf-86f1-41af-91ab-2d7cd011db47",
   "unique_name": "SophiaOwen@fabrikam.com",
   "upn": "SophiaOwen@fabrikam.com",
   "uti": "TPJ7nNNMMZkOSx6_uVczUAA",
   "ver": "1.0"
}
```

<a name="define-authorization-policy-portal"></a>

#### <a name="define-authorization-policy-in-azure-portal"></a>Definición de una directiva de autorización en Azure Portal

Para habilitar Azure AD OAuth para la aplicación lógica en Azure Portal, siga estos pasos a fin de agregar una o varias directivas de autorización a la aplicación lógica:

1. En [Azure Portal](https://portal.microsoft.com), busque y abra la aplicación lógica en el Diseñador de aplicación lógica.

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Autorización**. Una vez que se abra el panel Autorización, seleccione **Agregar directiva**.

   ![Selección de "Autorización" > "Agregar directiva"](./media/logic-apps-securing-a-logic-app/add-azure-active-directory-authorization-policies.png)

1. Proporcione información sobre la directiva de autorización; puede especificar los [tipos de notificaciones](../active-directory/develop/developer-glossary.md#claim) y los valores que espera la aplicación lógica en el token de autenticación presentado por cada llamada entrante al desencadenador de solicitud:

   ![Proporcionar información de la directiva de autorización](./media/logic-apps-securing-a-logic-app/set-up-authorization-policy.png)

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Nombre de directiva** | Sí | El nombre que quiere usar para la directiva de autorización. |
   | **Notificaciones** | Sí | Los tipos de notificaciones y los valores que acepta la aplicación lógica de las llamadas entrantes. Estos son los tipos de notificaciones disponibles: <p><p>- **Emisor** <br>- **Audiencia** <br>- **Subject** (Asunto) <br>- **JWT ID** (Id. de JWT) (identificador de token web de JSON) <p><p>Como mínimo, la lista **Notificaciones** debe incluir la notificación de **Emisor**, que tiene un valor que comienza con `https://sts.windows.net/` o `https://login.microsoftonline.com/` como identificador de emisor de Azure AD. Para obtener más información sobre estos tipos de notificaciones, consulte [Notificaciones de tokens de seguridad de Azure AD](../active-directory/azuread-dev/v1-authentication-scenarios.md#claims-in-azure-ad-security-tokens). También puede especificar su propio tipo de notificaciones y valor. |
   |||

1. Para agregar otra notificación, seleccione una de estas opciones:

   * Para agregar otro tipo de notificaciones, seleccione **Add standard claim** (Agregar notificación estándar), seleccione el tipo de notificación y especifique su valor.

   * Para agregar su propia notificación, seleccione **Add custom claim** (Agregar notificación personalizada) y especifique su valor.

1. Para agregar otra directiva de autorización, seleccione **Agregar directiva**. Repita los pasos anteriores para configurar la directiva.

1. Cuando finalice, seleccione **Guardar**.

<a name="define-authorization-policy-template"></a>

#### <a name="define-authorization-policy-in-azure-resource-manager-template"></a>Definición de la directiva de autorización en la plantilla de Azure Resource Manager

Para habilitar Azure AD OAuth en la plantilla de ARM para implementar la aplicación lógica, en la sección `properties` de la [definición de recursos de la aplicación lógica](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#logic-app-resource-definition), agregue un objeto `accessControl`, si no existe ninguno, que contenga un objeto `triggers`. En el objeto `triggers`, agregue un objeto `openAuthenticationPolicies` en el que defina una o varias directivas de autorización siguiendo esta sintaxis:

> [!NOTE]
> Como mínimo, la matriz `claims` debe incluir la notificación `iss`, que tiene un valor que comienza por `https://sts.windows.net/` o `https://login.microsoftonline.com/` como identificador del emisor de Azure AD. Para más información sobre estos tipos de notificaciones, consulte [Notificaciones de tokens de seguridad de Azure AD](../active-directory/azuread-dev/v1-authentication-scenarios.md#claims-in-azure-ad-security-tokens). También puede especificar su propio tipo de notificaciones y valor.

```json
"resources": [
   {
      // Start logic app resource definition
      "properties": {
         "state": "<Enabled-or-Disabled>",
         "definition": {<workflow-definition>},
         "parameters": {<workflow-definition-parameter-values>},
         "accessControl": {
            "triggers": {
               "openAuthenticationPolicies": {
                  "policies": {
                     "<policy-name>": {
                        "type": "AAD",
                        "claims": [
                           {
                              "name": "<claim-name>",
                              "value": "<claim-value>"
                           }
                        ]
                     }
                  }
               }
            },
         },
      },
      "name": "[parameters('LogicAppName')]",
      "type": "Microsoft.Logic/workflows",
      "location": "[parameters('LogicAppLocation')]",
      "apiVersion": "2016-06-01",
      "dependsOn": [
      ]
   }
   // End logic app resource definition
],
```

Para más información sobre la sección `accessControl`, vea [Restricción de los intervalos IP entrantes en la plantilla de Azure Resource Manager](#restrict-inbound-ip-template) y [Referencia de plantillas de flujos de trabajo de Microsoft.Logic](/azure/templates/microsoft.logic/2019-05-01/workflows).

<a name="azure-api-management"></a>

### <a name="expose-your-logic-app-with-azure-api-management"></a>Exposición de su aplicación lógica con Azure API Management

Para agregar más [protocolos de autenticación](../active-directory/develop/authentication-vs-authorization.md) a la aplicación lógica, considere la posibilidad de usar el servicio [Azure API Management](../api-management/api-management-key-concepts.md). Este servicio le permite exponer su aplicación lógica como API y le ofrece una supervisión enriquecida, seguridad, directiva y documentación para cualquier punto de conexión. API Management puede exponer un punto de conexión público o privado para la aplicación lógica. Para autorizar el acceso a este punto de conexión, puede usar Azure AD OAuth, un [certificado de cliente](#client-certificate-authentication) o cualquier otro estándar de seguridad. Cuando API Management recibe una solicitud, el servicio la envía a la aplicación lógica y también hace cualquier transformación o restricción necesaria en el proceso. Para que solo API Management pueda llamar a la aplicación lógica, puede [limitar el número de direcciones IP de entrada de la aplicación lógica](#restrict-inbound-ip).

<a name="restrict-inbound-ip"></a>

### <a name="restrict-inbound-ip-addresses"></a>Direcciones IP entrantes restringidas

Junto con la Firma de acceso compartido (SAS), es posible que quiera especificar el límite de clientes que pueden llamar a su aplicación lógica. Por ejemplo, si administra el punto de conexión de solicitud mediante [Azure API Management](../api-management/api-management-key-concepts.md), puede restringir la aplicación lógica para que solo acepte solicitudes procedentes de la dirección IP de la [instancia de servicio de API Management que crea](../api-management/get-started-create-service-instance.md).

#### <a name="restrict-inbound-ip-ranges-in-azure-portal"></a>Restricción de los intervalos IP entrantes en Azure Portal

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en Diseñador de aplicación lógica.

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Configuración de flujo de trabajo**.

1. En **Configuración del control de acceso** > **Direcciones IP entrantes permitidas**, seleccione **Intervalos IP específicos**.

1. En **Intervalos IP para desencadenadores**, especifique los intervalos de direcciones IP que acepta el desencadenador.

   Un intervalo IP válido usa estos formatos: *x.x.x.x/x* o *x.x.x.x-x.x.x.x*

Si quiere que la aplicación lógica solo se desencadene como aplicación lógica anidada, en la lista **Direcciones IP entrantes permitidas**, seleccione **Cualquier otra aplicación lógica**. Esta opción escribe una matriz vacía en el recurso de la aplicación lógica. De este modo, solo las llamadas del servicio de Logic Apps (las aplicaciones lógicas principales) pueden desencadenar la aplicación lógica anidada.

> [!NOTE]
> Independientemente de la dirección IP, de todos modos puede ejecutar una aplicación lógica que tiene un desencadenador basado en solicitud mediante el uso de la solicitud [API de REST de Logic Apps: desencadenadores de flujo de trabajo: ejecutar](/rest/api/logic/workflowtriggers/run) o de API Management. Sin embargo, en este escenario aún se necesita la [autenticación](../active-directory/develop/authentication-vs-authorization.md) con la API REST de Azure. Todos los eventos aparecen en el registro de auditoría de Azure. Asegúrese de establecer las directivas de control de acceso como corresponda.

<a name="restrict-inbound-ip-template"></a>

#### <a name="restrict-inbound-ip-ranges-in-azure-resource-manager-template"></a>Restricción de los intervalos IP entrantes en la plantilla de Azure Resource Manager

Si [automatiza la implementación de las aplicaciones lógicas mediante plantillas de Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), puede especificar los intervalos de direcciones IP en formato *x.x.x.x/x* o *x.x.x.x-x.x.x.x*; para ello, use la sección `accessControl` e incluya las secciones `triggers` y `actions` en la definición de recursos de la aplicación lógica, por ejemplo:

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {},
   "variables": {},
   "resources": [
      {
         "name": "[parameters('LogicAppName')]",
         "type": "Microsoft.Logic/workflows",
         "location": "[parameters('LogicAppLocation')]",
         "tags": {
            "displayName": "LogicApp"
         },
         "apiVersion": "2016-06-01",
         "properties": {
            "definition": {
               <workflow-definition>
            },
            "parameters": {
            },
            "accessControl": {
               "triggers": {
                  "allowedCallerIpAddresses": [
                     {
                        "addressRange": "192.168.12.0/23"
                     }
                  ]
               },
               "actions": {
                  "allowedCallerIpAddresses": []
               }
            },
            "endpointsConfiguration": {}
         }
      }
   ],
   "outputs": {}
}
```

<a name="secure-operations"></a>

## <a name="access-to-logic-app-operations"></a>Acceso a las operaciones de las aplicaciones lógicas

Puede permitir que solo determinados usuarios o grupos ejecuten tareas específicas, como administrar, editar y ver aplicaciones lógicas. Para controlar sus permisos, use el [Control de acceso basado en rol (RBAC) de Azure](../role-based-access-control/role-assignments-portal.md) para que pueda asignar roles personalizados o integrados a los miembros de la suscripción a Azure:

* [Colaborador de aplicación lógica](../role-based-access-control/built-in-roles.md#logic-app-contributor): Le permite administrar aplicaciones lógicas, pero no puede cambiar el acceso a ellas.

* [Operador de aplicación lógica](../role-based-access-control/built-in-roles.md#logic-app-operator): Le permite leer, habilitar y deshabilitar aplicaciones lógicas, pero no puede editarlas ni actualizarlas.

Para evitar que otros usuarios cambien o elimine la aplicación lógica, puede usar el [bloqueo de recursos de Azure](../azure-resource-manager/management/lock-resources.md). Esta funcionalidad evita que otros usuarios cambien o eliminen los recursos de producción.

<a name="secure-run-history"></a>

## <a name="access-to-run-history-data"></a>Acceso a los datos del historial de ejecución

Durante la ejecución de una aplicación lógica, todos los datos [se cifran en tránsito](../security/fundamentals/encryption-overview.md#encryption-of-data-in-transit) con Seguridad de la capa de transporte (TLS) y [en reposo](../security/fundamentals/encryption-atrest.md). Cuando finaliza la ejecución de la aplicación lógica, puede ver el historial de esa ejecución, incluidos los pasos que se ejecutaron junto con el estado, la duración, las entradas y las salidas de cada acción. Este completo detalle proporciona información sobre cómo se ejecuta la aplicación lógica y dónde puede empezar a solucionar los problemas que surjan.

Cuando visualiza el historial de ejecución de la aplicación lógica, Logic Apps autentica su acceso y proporciona vínculos a las entradas y salidas para las solicitudes y respuestas de cada ejecución. Sin embargo, para las acciones que controlan contraseñas, secretos, claves u otra información confidencial, es recomendable evitar que otros usuarios vean los datos y accedan a ellos. Por ejemplo, si la aplicación lógica obtiene un secreto de [Azure Key Vault](../key-vault/general/overview.md) para usarlo al autenticar una acción HTTP, puede ocultar ese secreto de la vista.

Para controlar el acceso a las entradas y salidas del historial de ejecución de la aplicación lógica, tiene estas opciones:

* [Restricción del acceso por intervalo de direcciones IP](#restrict-ip).

  Esta opción le permite proteger el acceso al historial de ejecución en función de las solicitudes de un intervalo de direcciones IP específico.

* [Protección de los datos del historial de ejecución mediante ofuscación](#obfuscate).

  En muchos desencadenadores y acciones, puede proteger las entradas, las salidas, o ambas, en el historial de ejecución de una aplicación lógica.

<a name="restrict-ip"></a>

### <a name="restrict-access-by-ip-address-range"></a>Restringir el acceso por intervalo de direcciones IP

Puede limitar el acceso a las entradas y salidas del historial de ejecución de la aplicación lógica para que solo las solicitudes de intervalos de direcciones IP específicas puedan ver los datos. Por ejemplo, para impedir que alguien tenga acceso a las entradas y salidas, especifique un intervalo de direcciones IP como, por ejemplo, `0.0.0.0-0.0.0.0`. Solo una persona con permisos de administrador puede quitar esta restricción, lo que proporciona la posibilidad de usar el acceso "Just-In-Time" para los datos de la aplicación lógica. Puede especificar los intervalos IP que se van a restringir desde Azure Portal o en una plantilla de Azure Resource Manager que se usa para la implementación de la aplicación lógica.

#### <a name="restrict-ip-ranges-in-azure-portal"></a>Restricción de los intervalos IP en Azure Portal

1. En Azure Portal, abra la aplicación lógica en Diseñador de aplicación lógica.

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Configuración de flujo de trabajo**.

1. En **Configuración del control de acceso** > **Direcciones IP entrantes permitidas**, seleccione **Intervalos IP específicos**.

1. En **Intervalos IP para contenido**, especifique los intervalos de direcciones IP que pueden acceder a contenido desde las entradas y salidas.

   Un intervalo IP válido usa estos formatos: *x.x.x.x/x* o *x.x.x.x-x.x.x.x*

#### <a name="restrict-ip-ranges-in-azure-resource-manager-template"></a>Restricción de los intervalos IP entrantes en la plantilla de Azure Resource Manager

Si [automatiza las implementaciones de aplicaciones lógicas mediante una plantilla de Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), puede especificar los intervalos IP mediante la sección `accessControl` con la sección `contents` en la definición de recursos de la aplicación lógica; por ejemplo:

``` json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {},
   "variables": {},
   "resources": [
      {
         "name": "[parameters('LogicAppName')]",
         "type": "Microsoft.Logic/workflows",
         "location": "[parameters('LogicAppLocation')]",
         "tags": {
            "displayName": "LogicApp"
         },
         "apiVersion": "2016-06-01",
         "properties": {
            "definition": {<workflow-definition>},
            "parameters": {},
            "accessControl": {
               "contents": {
                  "allowedCallerIpAddresses": [
                     {
                        "addressRange": "192.168.12.0/23"
                     },
                     {
                        "addressRange": "2001:0db8::/64"
                     }
                  ]
               }
            }
         }
      }
   ],
   "outputs": {}
}
```

<a name="obfuscate"></a>

### <a name="secure-data-in-run-history-by-using-obfuscation"></a>Protección del historial de ejecución mediante ofuscación

Muchos desencadenadores y acciones cuentan con una configuración para proteger las entradas, las salidas, o ambas, en el historial de ejecución de una aplicación lógica. Antes de usar esta configuración para proteger los datos, [hay algunos aspectos que se deben tener en cuenta](#obfuscation-considerations).

#### <a name="secure-inputs-and-outputs-in-the-designer"></a>Protección de entradas y salidas en el diseñador

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en Diseñador de aplicación lógica.

   ![Abrir la aplicación lógica en el Diseñador de aplicación lógica](./media/logic-apps-securing-a-logic-app/open-sample-logic-app-in-designer.png)

1. En el desencadenador o la acción donde quiera proteger los datos confidenciales, seleccione los puntos suspensivos ( **...** ) y, luego, elija **Configuración**.

   ![Abrir configuración de desencadenador o acción](./media/logic-apps-securing-a-logic-app/open-action-trigger-settings.png)

1. Active **Entradas seguras**, **Salidas seguras** o ambas opciones. Cuando haya finalizado, seleccione **Listo**.

   ![Active "Entradas seguras" o "Salidas seguras".](./media/logic-apps-securing-a-logic-app/turn-on-secure-inputs-outputs.png)

   Ahora, la acción o el desencadenador muestra un icono de candado en la barra de título.

   ![La barra de título de acción o desencadenador muestra el icono de candado](./media/logic-apps-securing-a-logic-app/lock-icon-action-trigger-title-bar.png)

   Los tokens que representan las salidas protegidas de acciones anteriores también muestran los iconos de candado. Por ejemplo, al seleccionar un resultado de este tipo en la lista de contenido dinámico para usarlo en una acción, el token muestra un icono de candado.

   ![Seleccionar token para la salida protegida](./media/logic-apps-securing-a-logic-app/select-secured-token.png)

1. Una vez ejecutada la aplicación lógica, puede ver el historial de esa ejecución.

   1. En el panel **Información general** de la aplicación lógica, seleccione la ejecución que quiera ver.

   1. En el panel **Ejecución de aplicación lógica**, expanda las acciones que quiera revisar.

      Si decide ocultar las entradas y las salidas, esos valores aparecerán ahora ocultos.

      ![Entradas y salidas ocultas del historial de ejecución](./media/logic-apps-securing-a-logic-app/hidden-data-run-history.png)

<a name="secure-data-code-view"></a>

#### <a name="secure-inputs-and-outputs-in-code-view"></a>Protección de entradas y salidas en la vista Código

En la definición de desencadenador o acción subyacente, agregue o actualice la matriz `runtimeConfiguration.secureData.properties` con uno de estos valores o con ambos:

* `"inputs"`: Protege las entradas en el historial de ejecución.
* `"outputs"`: Protege las salidas en el historial de ejecución.

Algunas [consideraciones que se deben tener en cuenta](#obfuscation-considerations) al usar esta configuración para proteger los datos son:

```json
"<trigger-or-action-name>": {
   "type": "<trigger-or-action-type>",
   "inputs": {
      <trigger-or-action-inputs>
   },
   "runtimeConfiguration": {
      "secureData": {
         "properties": [
            "inputs",
            "outputs"
         ]
      }
   },
   <other-attributes>
}
```

<a name="obfuscation-considerations"></a>

#### <a name="considerations-when-securing-inputs-and-outputs"></a>Consideraciones a la hora de proteger las entradas y salidas

* Cuando se ocultan las entradas o las salidas de un desencadenador o una acción, Logic Apps no envía los datos protegidos a Azure Log Analytics. Además, no se pueden agregar [propiedades con seguimiento](../logic-apps/monitor-logic-apps-log-analytics.md#extend-data) al desencadenador o acción para su supervisión.

* La [API de Logic Apps para controlar el historial del flujo de trabajo](/rest/api/logic/) no devuelve salidas protegidas.

* Para proteger las salidas de una acción que oculta las entradas o explícitamente las salidas, active de forma manual **Salidas seguras** en esa acción.

* Asegúrese de activar las **Entradas seguras** o las **Salidas seguras** en las acciones de nivel inferior en las que espera que el historial de ejecución oculte los datos.

  **Configuración de las salidas seguras**

  Al activar manualmente **Salidas seguras** en un desencadenador o una acción, Logic Apps oculta estas salidas en el historial de ejecución. Si una acción de nivel inferior usa explícitamente estas salidas protegidas como entradas, Logic Apps oculta las entradas de esta acción en el historial de ejecución, pero *no habilita* la opción de **Entradas seguras** de la acción.

  ![Salidas seguras como entradas y repercusión descendente en la mayoría de las acciones](./media/logic-apps-securing-a-logic-app/secure-outputs-as-inputs-flow.png)

  Las acciones de redacción, análisis JSON y respuesta solo tienen la opción **Entradas seguras**. Cuando está activada, esta opción también oculta las salidas de estas acciones. Si estas acciones usan explícitamente las salidas protegidas de nivel superior como entradas, Logic Apps ocultará las entradas y salidas de estas acciones, pero *no habilitará* la opción **Entradas seguras** de estas acciones. Si una acción de nivel inferior usa explícitamente las salidas ocultas de las acciones de redacción, análisis JSON o respuesta como entradas, Logic Apps *no oculta las entradas o salidas de la acción de nivel inferior*.

  ![Salidas seguras como entradas y repercusión descendente en acciones especificas](./media/logic-apps-securing-a-logic-app/secure-outputs-as-inputs-flow-special.png)

  **Opción Entradas seguras**

  Al activar manualmente **Entradas seguras** en un desencadenador o una acción, Logic Apps oculta estas entradas en el historial de ejecución. Si en una acción de nivel inferior se usan de forma explícita las salidas visibles de ese desencadenador o acción como entradas, Logic Apps oculta las entradas de esta acción de nivel inferior en el historial de ejecución, pero *no habilita* las **Entradas seguras** en esta acción y no oculta las salidas de la acción.

  ![Entradas seguras y repercusión descendente en la mayoría de las acciones](./media/logic-apps-securing-a-logic-app/secure-inputs-impact-on-downstream.png)

  Si las acciones de redacción, análisis JSON y respuesta usan explícitamente las salidas visibles del desencadenador o la acción que tiene las entradas protegidas, Logic Apps oculta las entradas y salidas de estas acciones, pero *no habilita* la opción **Entradas seguras** de la acción. Si una acción de nivel inferior usa explícitamente las salidas ocultas de las acciones de redacción, análisis JSON o respuesta como entradas, Logic Apps *no oculta las entradas o salidas de la acción de nivel inferior*.

  ![Entradas seguras y repercusión descendente en determinadas acciones](./media/logic-apps-securing-a-logic-app/secure-inputs-flow-special.png)

<a name="secure-action-parameters"></a>

## <a name="access-to-parameter-inputs"></a>Acceso a entradas de parámetros

Si implementa en entornos diferentes, considere la posibilidad de parametrizar los valores de la definición de flujo de trabajo que varían en función de esos entornos. De este modo, puede evitar los datos incluidos en el código mediante una [plantilla de Azure Resource Manager ](../azure-resource-manager/templates/overview.md) para implementar la aplicación lógica, proteger los datos confidenciales definiendo parámetros seguros y pasar dichos datos como entradas separadas mediante los [parámetros de la plantilla](../azure-resource-manager/templates/template-parameters.md) con un [archivo de parámetros](../azure-resource-manager/templates/parameter-files.md).

Por ejemplo, si autentica acciones HTTP con [Azure Active Directory Open Authentication](#azure-active-directory-oauth-authentication) (Azure AD OAuth), puede definir y ocultar los parámetros que aceptan el identificador de cliente y el secreto de cliente utilizados para la autenticación. Para definir estos parámetros para la aplicación lógica, use la sección `parameters` dentro de la definición de flujo de trabajo de la aplicación lógica y Resource Manager para la implementación. Para proteger los valores de parámetros que no quiera mostrar al editar la aplicación lógica o visualizar el historial de ejecución, defina los parámetros con el tipo `securestring` o `secureobject` y use la codificación necesaria. Los parámetros de este tipo no se devuelven con la definición del recurso y no son accesibles al visualizar el recurso después de la implementación. Para tener acceso a estos valores de parámetro durante el tiempo de ejecución, use la expresión `@parameters('<parameter-name>')` dentro de la definición del flujo de trabajo. Esta expresión solo se evalúa en tiempo de ejecución y se describe con el [lenguaje de definición de flujo de trabajo](../logic-apps/logic-apps-workflow-definition-language.md).

> [!NOTE]
> Si usa un parámetro en el encabezado o el cuerpo de una solicitud, ese parámetro se puede ver al visualizar el historial de ejecución de la aplicación lógica y la solicitud HTTP saliente. Asegúrese de definir también las directivas de acceso al contenido según corresponda.
> También puede usar la [ofuscación](#obfuscate) para ocultar entradas y salidas en el historial de ejecución. Los encabezados de autorización nunca son visibles a través de entradas o salidas. Por lo tanto, si aquí se usa un secreto, no se podrá recuperar.

Para más información, consulte las secciones siguientes en este tema:

* [Parámetros protegidos en las definiciones de flujo de trabajo](#secure-parameters-workflow)
* [Protección del historial de ejecución mediante ofuscación](#obfuscate)

Si [automatiza la implementación de aplicaciones lógicas con plantillas de Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), puede definir [parámetros de plantilla](../azure-resource-manager/templates/template-parameters.md) protegidos, que se evalúan en la implementación, mediante los tipos `securestring` y `secureobject`. Para definir parámetros de plantilla, use la sección `parameters` de nivel superior de la plantilla, que es independiente y diferente de la sección `parameters` de la definición de flujo de trabajo. Para proporcionar valores para los parámetros de plantilla, use un [archivo de parámetros](../azure-resource-manager/templates/parameter-files.md) independiente.

Por ejemplo, si usa secretos, puede definir y usar parámetros de plantilla protegidos que recuperen dichos secretos de [Azure Key Vault](../key-vault/general/overview.md) en la implementación. A continuación, puede hacer referencia a Key Vault y al secreto en el archivo de parámetros. Para más información, consulte los temas siguientes:

* [Uso de Azure Key Vault para pasar valores confidenciales durante la implementación](../azure-resource-manager/templates/key-vault-parameter.md)
* [Parámetros seguros en plantillas de Azure Resource Manager](#secure-parameters-deployment-template) más adelante en este tema

<a name="secure-parameters-workflow"></a>

### <a name="secure-parameters-in-workflow-definitions"></a>Parámetros protegidos en las definiciones de flujo de trabajo

Para proteger la información confidencial en la definición del flujo de trabajo de aplicación lógica, use parámetros seguros, de forma que esta información no esté visible después de guardar la aplicación lógica. Por ejemplo, supongamos que tiene una acción HTTP que requiere autenticación básica que usa un nombre de usuario y una contraseña. En la definición del flujo, la sección `parameters` define los parámetros `basicAuthPasswordParam` y `basicAuthUsernameParam` mediante el tipo `securestring`. A continuación, la definición de la acción hace referencia a estos parámetros en la sección `authentication`.

```json
"definition": {
   "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
   "actions": {
      "HTTP": {
         "type": "Http",
         "inputs": {
            "method": "GET",
            "uri": "https://www.microsoft.com",
            "authentication": {
               "type": "Basic",
               "username": "@parameters('basicAuthUsernameParam')",
               "password": "@parameters('basicAuthPasswordParam')"
            }
         },
         "runAfter": {}
      }
   },
   "parameters": {
      "basicAuthPasswordParam": {
         "type": "securestring"
      },
      "basicAuthUsernameParam": {
         "type": "securestring"
      }
   },
   "triggers": {
      "manual": {
         "type": "Request",
         "kind": "Http",
         "inputs": {
            "schema": {}
         }
      }
   },
   "contentVersion": "1.0.0.0",
   "outputs": {}
}
```

<a name="secure-parameters-deployment-template"></a>

### <a name="secure-parameters-in-azure-resource-manager-templates"></a>Parámetros seguros en plantillas de Azure Resource Manager

Una [plantilla de Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md) para una aplicación lógica tiene varias secciones `parameters`. Para proteger las contraseñas, las claves, los secretos y otros datos confidenciales, defina parámetros seguros en el nivel de plantilla y de definición de flujo de trabajo mediante el tipo `securestring` o `secureobject`. Después, puede almacenar estos valores en [Azure Key Vault](../key-vault/general/overview.md) y usar el [archivo de parámetros](../azure-resource-manager/templates/parameter-files.md) para hacer referencia al almacén de claves y al secreto. A continuación, la plantilla recupera esa información en la implementación. Para obtener más información, consulte [Uso de Azure Key Vault para pasar valores confidenciales durante la implementación](../azure-resource-manager/templates/key-vault-parameter.md).

Aquí tiene más información sobre estas secciones `parameters`:

* En el nivel superior de la plantilla, una sección `parameters` define los parámetros de los valores que utiliza la plantilla durante la *implementación*. Por ejemplo, estos valores pueden incluir cadenas de conexión para un entorno de implementación concreto. Después, puede almacenar estos valores en un [archivo de parámetros](../azure-resource-manager/templates/parameter-files.md), lo que facilita la modificación de estos valores.

* Dentro de la definición de recursos de la aplicación lógica, pero fuera de la definición del flujo de trabajo, una sección `parameters` especifica los valores de los parámetros de la definición del flujo de trabajo. En esta sección, puede asignar estos valores mediante expresiones de plantilla que hagan referencia a los parámetros de la plantilla. Estas expresiones se evalúan en la implementación.

* Dentro de la definición del flujo de trabajo, una sección `parameters` define los parámetros que usa la aplicación lógica en tiempo de ejecución. Después, puede hacer referencia a estos parámetros en el flujo de trabajo de la aplicación lógica mediante expresiones de definición de flujo de trabajo, que se evalúan en tiempo de ejecución.

Esta plantilla de ejemplo tiene varias definiciones de parámetros seguros que usan el tipo `securestring`:

| Nombre de parámetro | Descripción |
|----------------|-------------|
| `TemplatePasswordParam` | Parámetro de plantilla que acepta una contraseña que se pasa, a continuación, al parámetro `basicAuthPasswordParam` de la definición del flujo de trabajo. |
| `TemplateUsernameParam` | Parámetro de plantilla que acepta un nombre de usuario que se pasa a continuación al parámetro `basicAuthUserNameParam` de la definición del flujo de trabajo |
| `basicAuthPasswordParam` | Parámetro de definición de flujo de trabajo que acepta la contraseña para la autenticación básica en una acción HTTP |
| `basicAuthUserNameParam` | Parámetro de definición de flujo de trabajo que acepta el nombre de usuario para la autenticación básica en una acción HTTP |
|||

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      "LogicAppName": {
         "type": "string",
         "minLength": 1,
         "maxLength": 80,
         "metadata": {
            "description": "Name of the Logic App."
         }
      },
      "TemplatePasswordParam": {
         "type": "securestring"
      },
      "TemplateUsernameParam": {
         "type": "securestring"
      },
      "LogicAppLocation": {
         "type": "string",
         "defaultValue": "[resourceGroup().location]",
         "allowedValues": [
            "[resourceGroup().location]",
            "eastasia",
            "southeastasia",
            "centralus",
            "eastus",
            "eastus2",
            "westus",
            "northcentralus",
            "southcentralus",
            "northeurope",
            "westeurope",
            "japanwest",
            "japaneast",
            "brazilsouth",
            "australiaeast",
            "australiasoutheast",
            "southindia",
            "centralindia",
            "westindia",
            "canadacentral",
            "canadaeast",
            "uksouth",
            "ukwest",
            "westcentralus",
            "westus2"
         ],
         "metadata": {
            "description": "Location of the Logic App."
         }
      }
   },
   "variables": {},
   "resources": [
      {
         "name": "[parameters('LogicAppName')]",
         "type": "Microsoft.Logic/workflows",
         "location": "[parameters('LogicAppLocation')]",
         "tags": {
            "displayName": "LogicApp"
         },
         "apiVersion": "2016-06-01",
         "properties": {
            "definition": {
               "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
               "actions": {
                  "HTTP": {
                     "type": "Http",
                     "inputs": {
                        "method": "GET",
                        "uri": "https://www.microsoft.com",
                        "authentication": {
                           "type": "Basic",
                           "username": "@parameters('basicAuthUsernameParam')",
                           "password": "@parameters('basicAuthPasswordParam')"
                        }
                     },
                  "runAfter": {}
                  }
               },
               "parameters": {
                  "basicAuthPasswordParam": {
                     "type": "securestring"
                  },
                  "basicAuthUsernameParam": {
                     "type": "securestring"
                  }
               },
               "triggers": {
                  "manual": {
                     "type": "Request",
                     "kind": "Http",
                     "inputs": {
                        "schema": {}
                     }
                  }
               },
               "contentVersion": "1.0.0.0",
               "outputs": {}
            },
            "parameters": {
               "basicAuthPasswordParam": {
                  "value": "[parameters('TemplatePasswordParam')]"
               },
               "basicAuthUsernameParam": {
                  "value": "[parameters('TemplateUsernameParam')]"
               }
            }
         }
      }
   ],
   "outputs": {}
}
```

<a name="secure-outbound-requests"></a>

## <a name="access-for-outbound-calls-to-other-services-and-systems"></a>Acceso de las llamadas salientes a otros servicios y sistemas

En función de la funcionalidad del punto de conexión de destino, las llamadas salientes enviadas por el [desencadenador HTTP o la acción HTTP](../connectors/connectors-native-http.md) admiten el cifrado y están protegidas mediante el protocolo [Seguridad de la capa de transporte (TLS) 1.0, 1.1 o 1.2](https://en.wikipedia.org/wiki/Transport_Layer_Security), conocido anteriormente como Capa de sockets seguros (SSL). Logic Apps negocia con el punto de conexión de destino el uso de la versión más alta que sea compatible. Por ejemplo, si el punto de conexión de destino admite la versión 1.2, el desencadenador o la acción HTTP usan primero esta versión. De lo contrario, el conector utiliza la siguiente versión compatible más alta.

A continuación se muestra información acerca de los certificados autofirmados de TLS/SSL:

* En el caso de aplicaciones lógicas en el entorno global multiinquilino de Azure, el conector HTTP no permite certificados de TLS/SSL autofirmados. Si la aplicación lógica realiza una llamada HTTP a un servidor y presenta un certificado autofirmado de TLS/SSL, la llamada HTTP produce un error `TrustFailure`.

* En el caso de las aplicaciones lógicas en un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), el conector HTTP permite certificados autofirmados para los protocolos de enlace TLS/SSL. Sin embargo, primero debe [habilitar la compatibilidad con certificados autofirmados](../logic-apps/create-integration-service-environment-rest-api.md#request-body) para un ISE existente o un ISE nuevo mediante la API REST de Logic Apps, y, después, instalar el certificado público en la ubicación de `TrustedRoot`.

Estas son otras formas en que se puede ayudar a proteger los punto de conexión que controlan las llamadas enviadas desde su aplicación lógica:

* [Incorporación de la autenticación a las solicitudes salientes](#add-authentication-outbound).

  Cuando se usa el desencadenador o la acción HTTP para realizar llamadas salientes, se puede agregar autenticación a la solicitud enviada por la aplicación lógica. Por ejemplo, puede seleccionar estos tipos de autenticación:

  * [Autenticación básica](#basic-authentication)

  * [Autenticación de certificados de clientes](#client-certificate-authentication)

  * [Autenticación de Active Directory OAuth](#azure-active-directory-oauth-authentication)

  * [Autenticación de identidad administrada](#managed-identity-authentication)

* Restricción del acceso desde direcciones IP de aplicación lógica.

  Todas las llamadas a los puntos de conexión desde aplicaciones lógicas proceden de direcciones IP designadas específicas que se basan en las regiones de las aplicaciones lógicas. Puede agregar un filtrado que solo acepta solicitudes provenientes de esas direcciones IP. Para obtener esas direcciones IP, consulte [Límites y configuración de Azure Logic Apps](logic-apps-limits-and-config.md#configuration).

* Mejore la seguridad de las conexiones a sistemas locales.

  Azure Logic Apps ofrece integración con estos servicios para permitir una comunicación local más segura y confiable.

  * Puerta de enlace de datos local

    Muchos conectores administrados de Azure Logic Apps proporcionan conexiones seguras a sistemas locales, como el sistema de archivos, SQL, SharePoint y DB2. La puerta de enlace envía datos desde orígenes locales en canales cifrados hasta Azure Service Bus. Todo el tráfico se origina como tráfico de salida seguro desde el agente de la puerta de enlace. Obtenga información sobre [el funcionamiento de la puerta de enlace de datos local](logic-apps-gateway-install.md#gateway-cloud-service).

  * Conexión mediante Azure API Management

    [Azure API Management](../api-management/api-management-key-concepts.md) ofrece opciones de conexión local, como la integración de [ExpressRoute](../expressroute/expressroute-introduction.md) y una red privada virtual sitio a sitio para un proxy seguro y comunicación con sistemas locales. Si tiene una API que proporciona acceso al sistema local y expuso esa API mediante la creación de una [instancia del servicio API Management](../api-management/get-started-create-service-instance.md), puede llamar a esa API en el flujo de trabajo de la aplicación lógica seleccionando la acción o el desencadenador de API Management integrados en el Diseñador de aplicación lógica.

    > [!NOTE]
    > El conector muestra solo los servicios de API Management donde cuenta con permisos para ver y conectarse, pero no muestra los servicios de API Management basados en el consumo.

    1. En el Diseñador de aplicación lógica, escriba `api management` en el cuadro de búsqueda. Elija el paso en función de si va a agregar un desencadenador o una acción:<p>

       * Si agrega un desencadenador, que siempre es el primer paso en el flujo de trabajo, seleccione **Choose an Azure API Management trigger** (Elegir un desencadenador de Azure API Management).

       * Si agrega una acción, seleccione **Choose an Azure API Management action** (Elegir una acción de Azure API Management).

       En este ejemplo se agrega un desencadenador:

       ![Incorporación del desencadenador de Azure API Management](./media/logic-apps-securing-a-logic-app/select-api-management.png)

    1. Seleccione su instancia de servicio API Management creada anteriormente.

       ![Selección de la instancia de servicio API Management](./media/logic-apps-securing-a-logic-app/select-api-management-service-instance.png)

    1. Seleccione la llamada API que se va a usar.

       ![Selección de la API existente](./media/logic-apps-securing-a-logic-app/select-api.png)

<a name="add-authentication-outbound"></a>

### <a name="add-authentication-to-outbound-calls"></a>Incorporación de la autenticación en las llamadas salientes

Los extremos HTTP y HTTPS admiten varios tipos de autenticación. En algunos desencadenadores y acciones que se usan para enviar llamadas o solicitudes salientes a estos puntos de conexión, se puede especificar un tipo de autenticación. En el Diseñador de aplicación lógica, los desencadenadores y las acciones que admiten la elección de un tipo de autenticación tienen una propiedad **Autenticación**. Sin embargo, es posible que esta propiedad no aparezca siempre de forma predeterminada. En estos casos, en el desencadenador o la acción, abra la lista **Agregar nuevo parámetro** y seleccione **Autenticación**.

> [!IMPORTANT]
> Para proteger la información confidencial que controla la aplicación lógica, use los parámetros seguros y codifique los datos según sea necesario.
> Para obtener más información acerca de cómo usar y proteger los parámetros, consulte [Acceso a las entradas de parámetro](#secure-action-parameters).

En esta tabla se identifican los tipos de autenticación que están disponibles en los desencadenadores y las acciones donde puede seleccionar un tipo de autenticación:

| Tipo de autenticación | Desencadenadores y acciones admitidos |
|---------------------|--------------------------------|
| [Basic](#basic-authentication) | Azure API Management, Azure App Services, HTTP, HTTP y Swagger, webhook HTTP |
| [Certificado de cliente](#client-certificate-authentication) | Azure API Management, Azure App Services, HTTP, HTTP y Swagger, webhook HTTP |
| [Active Directory OAuth](#azure-active-directory-oauth-authentication) | Azure API Management, Azure App Services, Azure Functions, HTTP, HTTP + Swagger, webhook HTTP |
| [Sin formato](#raw-authentication) | Azure API Management, Azure App Services, Azure Functions, HTTP, HTTP + Swagger, webhook HTTP |
| [Identidad administrada](#managed-identity-authentication) | Azure API Management, Azure App Services, Azure Functions, HTTP |
|||

<a name="basic-authentication"></a>

### <a name="basic-authentication"></a>Autenticación básica

Si la opción [Básica](../active-directory-b2c/secure-rest-api.md) está disponible, especifique estos valores de propiedad:

| Propiedad (diseñador) | Propiedad (JSON) | Obligatorio | Value | Descripción |
|---------------------|-----------------|----------|-------|-------------|
| **Autenticación** | `type` | Sí | Básica | Tipo de autenticación que se debe usar. |
| **Nombre de usuario** | `username` | Sí | <*nombre-de-usuario*>| Nombre de usuario para autenticar el acceso al extremo del servicio de destino. |
| **Contraseña** | `password` | Sí | <*contraseña*> | Contraseña para autenticar el acceso al extremo del servicio de destino. |
||||||

Al usar [parámetros protegidos](#secure-action-parameters) para administrar y proteger la información confidencial, por ejemplo, en una [plantilla de Azure Resource Manager para automatizar la implementación](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), puede usar expresiones para acceder a estos valores de parámetros en tiempo de ejecución. Esta definición de acción HTTP de ejemplo especifica el `type` de autenticación como `Basic` y usa la [función parameters()](../logic-apps/workflow-definition-language-functions-reference.md#parameters) para obtener los valores de parámetro:

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "@parameters('endpointUrlParam')",
      "authentication": {
         "type": "Basic",
         "username": "@parameters('userNameParam')",
         "password": "@parameters('passwordParam')"
      }
  },
  "runAfter": {}
}
```

<a name="client-certificate-authentication"></a>

### <a name="client-certificate-authentication"></a>Autenticación de certificados de cliente

Si la opción [Certificado de cliente](../active-directory/authentication/active-directory-certificate-based-authentication-get-started.md) está disponible, especifique estos valores de propiedad:

| Propiedad (diseñador) | Propiedad (JSON) | Obligatorio | Value | Descripción |
|---------------------|-----------------|----------|-------|-------------|
| **Autenticación** | `type` | Sí | **Certificado de cliente** <br>or <br>`ClientCertificate` | Tipo de autenticación que se debe usar. Puede administrar certificados con [Azure API Management](../api-management/api-management-howto-mutual-certificates.md). <p></p>**Nota**: Los conectores personalizados no admiten la autenticación basada en certificados para las llamadas entrantes y salientes. |
| **Pfx** | `pfx` | Sí | <*contenido-archivo-pfx-codificado*> | El contenido codificado en base 64 del archivo de intercambio de información personal (PFX) <p><p>Para convertir el archivo PFX en formato codificado en Base64, puede usar PowerShell siguiendo estos pasos: <p>1. Guarde el contenido del certificado en una variable: <p>   `$pfx_cert = get-content 'c:\certificate.pfx' -Encoding Byte` <p>2. Convierta el contenido del certificado mediante la función `ToBase64String()` y guarde el contenido en un archivo de texto: <p>   `[System.Convert]::ToBase64String($pfx_cert) | Out-File 'pfx-encoded-bytes.txt'` |
| **Contraseña** | `password`| No | <*contraseña-archivo-pfx*> | La contraseña para acceder al archivo PFX |
|||||

Al usar [parámetros protegidos](#secure-action-parameters) para administrar y proteger la información confidencial, por ejemplo, en una [plantilla de Azure Resource Manager para automatizar la implementación](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), puede usar expresiones para acceder a estos valores de parámetros en tiempo de ejecución. Esta definición de acción HTTP de ejemplo especifica el `type` de autenticación como `ClientCertificate` y usa la [función parameters()](../logic-apps/workflow-definition-language-functions-reference.md#parameters) para obtener los valores de parámetro:

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "@parameters('endpointUrlParam')",
      "authentication": {
         "type": "ClientCertificate",
         "pfx": "@parameters('pfxParam')",
         "password": "@parameters('passwordParam')"
      }
   },
   "runAfter": {}
}
```

Para obtener más información acerca de cómo proteger los servicios mediante la autenticación de certificados de cliente, consulte estos temas:

* [Cómo mejorar la seguridad de las API mediante la autenticación de certificados de cliente en Azure API Management](../api-management/api-management-howto-mutual-certificates-for-clients.md)
* [Cómo mejorar la seguridad de los servicios back-end con la autenticación de certificados de cliente en Azure API Management](../api-management/api-management-howto-mutual-certificates.md)
* [Cómo mejorar la seguridad de los servicios RESTful mediante certificados de cliente](../active-directory-b2c/secure-rest-api.md)
* [Credenciales de certificado para la autenticación de aplicaciones](../active-directory/develop/active-directory-certificate-credentials.md)
* [Uso de un certificado TLS/SSL en el código de Azure App Service](../app-service/configure-ssl-certificate-in-code.md)

<a name="azure-active-directory-oauth-authentication"></a>

### <a name="azure-active-directory-open-authentication"></a>Azure Active Directory Open Authentication

En los desencadenadores Request (Solicitud), se puede usar [Azure Active Directory Open Authentication (Azure AD OAuth)](../active-directory/develop/index.yml) para autenticar las llamadas entrantes después de [configurar las directivas de autorización de Azure AD](#enable-oauth) de la aplicación lógica. En el caso del resto de desencadenadores y acciones que proporcionan el tipo de autenticación **Active Directory OAuth**, especifique estos valores de propiedad:

| Propiedad (diseñador) | Propiedad (JSON) | Obligatorio | Value | Descripción |
|---------------------|-----------------|----------|-------|-------------|
| **Autenticación** | `type` | Sí | **Active Directory OAuth** <br>or <br>`ActiveDirectoryOAuth` | Tipo de autenticación que se debe usar. Actualmente, Logic Apps sigue el [protocolo OAuth 2.0](../active-directory/develop/v2-overview.md). |
| **Autoridad** | `authority` | No | <*URL-for-authority-token-issuer*> | La dirección URL de la autoridad que proporciona el token de autenticación. De manera predeterminada, este valor es `https://login.windows.net`. |
| **Inquilino** | `tenant` | Sí | <*tenant-ID*> | El identificador del inquilino de Azure AD |
| **Audiencia** | `audience` | Sí | <*resource-to-authorize*> | Recurso que quiere usar para la autorización; por ejemplo, `https://management.core.windows.net/` |
| **Id. de cliente** | `clientId` | Sí | <*client-ID*> | El identificador de cliente para la aplicación que solicita autorización |
| **Tipo de credencial** | `credentialType` | Sí | Certificado <br>or <br>Secreto | El tipo de credencial que el cliente usa para solicitar autorización. Esta propiedad y el valor no aparecen en la definición subyacente de la aplicación lógica, pero determina las propiedades que se muestran para el tipo de credencial seleccionado. |
| **Secreto** | `secret` | Sí, pero solo para el tipo de credencial de "Secreto". | <*secreto-de-cliente*> | Secreto de cliente para solicitar la autorización |
| **Pfx** | `pfx` | Sí, pero solo para el tipo de credencial de "Certificado". | <*contenido-archivo-pfx-codificado*> | El contenido codificado en base 64 del archivo de intercambio de información personal (PFX) |
| **Contraseña** | `password` | Sí, pero solo para el tipo de credencial de "Certificado". | <*contraseña-archivo-pfx*> | La contraseña para acceder al archivo PFX |
|||||

Al usar [parámetros protegidos](#secure-action-parameters) para administrar y proteger la información confidencial, por ejemplo, en una [plantilla de Azure Resource Manager para automatizar la implementación](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), puede usar expresiones para acceder a estos valores de parámetros en tiempo de ejecución. Esta definición de acción HTTP de ejemplo especifica el `type` de autenticación como `ActiveDirectoryOAuth`, el tipo de credencial `Secret` y usa la [función parameters()](../logic-apps/workflow-definition-language-functions-reference.md#parameters) para obtener los valores de parámetro:

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "@parameters('endpointUrlParam')",
      "authentication": {
         "type": "ActiveDirectoryOAuth",
         "tenant": "@parameters('tenantIdParam')",
         "audience": "https://management.core.windows.net/",
         "clientId": "@parameters('clientIdParam')",
         "credentialType": "Secret",
         "secret": "@parameters('secretParam')"
     }
   },
   "runAfter": {}
}
```

<a name="raw-authentication"></a>

### <a name="raw-authentication"></a>Autenticación sin formato

Si la opción **Raw** está disponible, puede usar este tipo de autenticación cuando tenga que usar [esquemas de autenticación](https://iana.org/assignments/http-authschemes/http-authschemes.xhtml) que no siguen el [protocolo OAuth 2.0](https://oauth.net/2/). Con este tipo, se crea manualmente el valor del encabezado de autorización que se envía con la solicitud saliente y se especifica ese valor de encabezado en el desencadenador o la acción.

Por ejemplo, el siguiente es un encabezado de ejemplo para una solicitud HTTPS que sigue el [protocolo OAuth 1.0](https://tools.ietf.org/html/rfc5849):

```text
Authorization: OAuth realm="Photos",
   oauth_consumer_key="dpf43f3p2l4k3l03",
   oauth_signature_method="HMAC-SHA1",
   oauth_timestamp="137131200",
   oauth_nonce="wIjqoS",
   oauth_callback="http%3A%2F%2Fprinter.example.com%2Fready",
   oauth_signature="74KNZJeDHnMBp0EMJ9ZHt%2FXKycU%3D"
```

En el desencadenador o la acción que admite la autenticación sin formato, especifique estos valores de propiedad:

| Propiedad (diseñador) | Propiedad (JSON) | Obligatorio | Value | Descripción |
|---------------------|-----------------|----------|-------|-------------|
| **Autenticación** | `type` | Sí | Raw | Tipo de autenticación que se debe usar. |
| **Valor** | `value` | Sí | <*valor de encabezado de autorización*> | Valor del encabezado de autorización que se va a usar para la autenticación. |
||||||

Al usar [parámetros protegidos](#secure-action-parameters) para administrar y proteger la información confidencial, por ejemplo, en una [plantilla de Azure Resource Manager para automatizar la implementación](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), puede usar expresiones para acceder a estos valores de parámetros en tiempo de ejecución. Esta definición de acción HTTP de ejemplo especifica el `type` de autenticación como `Raw` y usa la [función parameters()](../logic-apps/workflow-definition-language-functions-reference.md#parameters) para obtener los valores de parámetro:

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "@parameters('endpointUrlParam')",
      "authentication": {
         "type": "Raw",
         "value": "@parameters('authHeaderParam')"
      }
   },
   "runAfter": {}
}
```

<a name="managed-identity-authentication"></a>

### <a name="managed-identity-authentication"></a>Autenticación de identidad administrada

Si la opción [Identidad administrada](../active-directory/managed-identities-azure-resources/overview.md) está disponible en un [desencadenador o acción específicos](#add-authentication-outbound), la aplicación lógica puede usar la identidad asignada por el sistema o una identidad *única* asignada por el usuario creada de forma manual para autenticar el acceso a otros recursos que están protegidos por Azure Active Directory (Azure AD) sin iniciar la sesión. Azure administra esta identidad y le ayuda a proteger las credenciales porque, de esta forma, no tiene que proporcionar secretos o cambiarlos. Obtenga más información sobre [Servicios de Azure que admiten las identidades administradas para la autenticación de Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

1. Para que la aplicación lógica pueda usar una identidad administrada, siga los pasos descritos en [Autenticación de acceso a los recursos de Azure con identidades administradas en Azure Logic Apps](../logic-apps/create-managed-service-identity.md). En estos pasos se habilita la identidad administrada en la aplicación lógica y se configura el acceso de dicha identidad al recurso de Azure de destino.

1. Para que una función de Azure pueda usar una identidad administrada, primero [habilite la autenticación de Azure Functions](../logic-apps/logic-apps-azure-functions.md#enable-authentication-for-azure-functions).

1. En el desencadenador o la acción donde quiere usar la identidad administrada, especifique estos valores de propiedad:

   | Propiedad (diseñador) | Propiedad (JSON) | Obligatorio | Value | Descripción |
   |---------------------|-----------------|----------|-------|-------------|
   | **Autenticación** | `type` | Sí | **Identidad administrada** <br>or <br>`ManagedServiceIdentity` | Tipo de autenticación que se debe usar. |
   | **Identidad administrada** | `identity` | Sí | * **Identidad administrada asignada por el sistema** <br>or <br>`SystemAssigned` <p><p>* <*nombre de identidad asignado por el usuario*> | Identidad administrada que se debe usar. |
   | **Audiencia** | `audience` | Sí | <*Id-recurso-destino*> | El Id. de recurso para el recurso de destino al que quiere obtener acceso. <p>Por ejemplo, `https://storage.azure.com/` hace que los [tokens de acceso](../active-directory/develop/access-tokens.md) para la autenticación sean válidos con todas las cuentas de almacenamiento. Sin embargo, también puede especificar una dirección URL de servicio raíz, como `https://fabrikamstorageaccount.blob.core.windows.net` para una cuenta de almacenamiento específica. <p>**Nota**: La propiedad **Audiencia** puede estar oculta en algunos desencadenadores o acciones. Para que la propiedad sea visible, en el desencadenador o la acción, abra la lista **Agregar nuevo parámetro** y seleccione **Público**. <p><p>**Importante**: Asegúrese de que el identificador de este recurso de destino *coincide exactamente* con el valor esperado en Azure AD, incluida toda barra diagonal necesaria al final. Por lo tanto, el Id. de recurso de `https://storage.azure.com/` para todas las cuentas de Azure Blob Storage requiere una barra diagonal final. Sin embargo, el Id. de recurso de una cuenta de almacenamiento específica no requiere una barra diagonal final. Para buscar estos Id. de recursos, consulte [Servicios de Azure que admiten la autenticación de Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication). |
   |||||

   Al usar [parámetros protegidos](#secure-action-parameters) para administrar y proteger la información confidencial, por ejemplo, en una [plantilla de Azure Resource Manager para automatizar la implementación](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), puede usar expresiones para acceder a estos valores de parámetros en tiempo de ejecución. Esta definición de acción HTTP de ejemplo especifica el `type` de autenticación como `ManagedServiceIdentity` y usa la [función parameters()](../logic-apps/workflow-definition-language-functions-reference.md#parameters) para obtener los valores de parámetro:

   ```json
   "HTTP": {
      "type": "Http",
      "inputs": {
         "method": "GET",
         "uri": "@parameters('endpointUrlParam')",
         "authentication": {
            "type": "ManagedServiceIdentity",
            "identity": "SystemAssigned",
            "audience": "https://management.azure.com/"
         },
      },
      "runAfter": {}
   }
   ```

<a name="block-connections"></a>

## <a name="block-creating-connections"></a>Bloquear la creación de conexiones

Si su organización no permite la conexión a recursos específicos mediante el uso de sus conectores en Azure Logic Apps, puede [bloquear la capacidad para crear esas conexiones](../logic-apps/block-connections-connectors.md) para conectores específicos en flujos de trabajo de aplicaciones lógicas mediante el uso de [Azure Policy](../governance/policy/overview.md). Para obtener más información, consulte [Bloquear las conexiones creadas por conectores específicos en Azure Logic Apps](../logic-apps/block-connections-connectors.md).

<a name="isolation-logic-apps"></a>

## <a name="isolation-guidance-for-logic-apps"></a>Guía de aislamiento para aplicaciones lógicas

Puede usar Azure Logic Apps en [Azure Government](../azure-government/documentation-government-welcome.md) admitiendo todos los niveles de impacto en las regiones que se describen en la [guía de aislamiento de nivel de impacto 5 de Azure Government](../azure-government/documentation-government-impact-level-5.md#azure-logic-apps) y [Guía de requisitos de seguridad (SRG) de informática en la nube del Departamento de Defensa de Estados Unidos](https://dl.dod.cyber.mil/wp-content/uploads/cloud/SRG/index.html). Para cumplir estos requisitos, Logic Apps admite la funcionalidad para que cree y ejecute flujos de trabajo en un entorno con recursos dedicados de modo que pueda reducir el impacto en el rendimiento de otros inquilinos de Azure en las aplicaciones lógicas y evitar compartir recursos informáticos con otros inquilinos.

* Para ejecutar su propio código o realizar la transformación XML, [cree y llame a una función de Azure](../logic-apps/logic-apps-azure-functions.md), en vez de usar la [funcionalidad de código en línea](../logic-apps/logic-apps-add-run-inline-code.md) o proporcionar [ensamblados para usarlos como mapas](../logic-apps/logic-apps-enterprise-integration-maps.md), respectivamente. Asimismo, configure el entorno de hospedaje de la aplicación de funciones para cumplir los requisitos de aislamiento.

  Por ejemplo, para cumplir los requisitos de nivel de impacto 5, cree la aplicación de funciones con el [plan de App Service](../azure-functions/functions-scale.md#app-service-plan) mediante el plan de tarifa [**aislado**](../app-service/overview-hosting-plans.md) junto con una instancia de [App Service Environment (ASE)](../app-service/environment/intro.md) que también usa el plan de tarifa **aislado**. En este entorno, las aplicaciones de funciones se ejecutan en máquinas virtuales y redes virtuales de Azure dedicadas, que proporcionan aislamiento de red sobre el aislamiento de proceso para las aplicaciones y las máximas posibilidades de escalabilidad horizontal. Para obtener más información, consulte la [guía de aislamiento de nivel de impacto 5 de Azure Government: Azure Functions](../azure-government/documentation-government-impact-level-5.md#azure-functions).

  Para más información, consulte los temas siguientes:<p>

  * [Planes de Azure App Service](../app-service/overview-hosting-plans.md)
  * [Opciones de redes de Azure Functions](../azure-functions/functions-networking-options.md)
  * [Hosts dedicados de Azure para máquinas virtuales](../virtual-machines/windows/dedicated-hosts.md)
  * [Aislamiento de máquina virtual de Azure](../virtual-machines/isolation.md)
  * [Implementación de servicios de Azure dedicados en las redes virtuales](../virtual-network/virtual-network-for-azure-services.md)

* Para crear aplicaciones lógicas que se ejecuten en recursos dedicados y puedan tener acceso a recursos protegidos por una red virtual de Azure, puede crear un [entorno del servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md).

  * Algunas redes virtuales de Azure usan puntos de conexión privados ([Azure Private Link](../private-link/private-link-overview.md)) para proporcionar acceso a los servicios de PaaS de Azure, como Azure Storage, Azure Cosmos DB o Azure SQL Database, servicios de asociados o servicios de clientes que se hospedan en Azure. Si las aplicaciones lógicas necesitan acceder a redes virtuales que usan puntos de conexión privados, debe crear, implementar y ejecutar esas aplicaciones lógicas dentro de un ISE.

  * Para obtener más control sobre las claves de cifrado utilizadas por Azure Storage, puede configurar, usar y administrar su propia clave mediante [Azure Key Vault](../key-vault/general/overview.md). Esta funcionalidad también se conoce como "Bring Your Own Key" (BYOK) y la clave se denomina "clave administrada por el cliente". Para obtener más información, consulte [Configure claves administradas por el cliente para cifrar los datos en reposo para los entornos del servicio de integración (ISE) en Azure Logic Apps](../logic-apps/customer-managed-keys-integration-service-environment.md).

Para más información, consulte los temas siguientes:

* [Aislamiento en la nube pública de Azure](../security/fundamentals/isolation-choices.md)
* [Seguridad para aplicaciones IaaS altamente confidenciales en Azure](/azure/architecture/reference-architectures/n-tier/high-security-iaas)

## <a name="next-steps"></a>Pasos siguientes

* [Base de referencia de seguridad de Azure para Azure Logic Apps](../logic-apps/security-baseline.md)
* [Automatización de la implementación para Azure Logic Apps](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md)
* [Supervisión de aplicaciones lógicas](../logic-apps/monitor-logic-apps-log-analytics.md)
