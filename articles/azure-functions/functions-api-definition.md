---
title: Metadatos de OpenAPI en Azure Functions
description: Introducción a la compatibilidad con OpenAPI en Azure Functions
ms.topic: conceptual
ms.date: 03/23/2017
ms.openlocfilehash: 499d4f685e3802fcc37c8a3050ae367207f192d2
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87385830"
---
# <a name="openapi-20-metadata-support-in-azure-functions-preview"></a>Compatibilidad con metadatos de OpenAPI 2.0 en Azure Functions (versión preliminar)
La compatibilidad con metadatos de OpenAPI 2.0 (anteriormente Swagger) en Azure Functions es una característica en versión preliminar que puede usar para escribir una definición de OpenAPI 2.0 dentro de una aplicación de función. A continuación, puede hospedar ese archivo mediante la aplicación de función.

> [!IMPORTANT]
> La característica en versión preliminar de OpenAPI solo está disponible actualmente en el entorno de ejecución 1.x. Puede encontrar información sobre cómo crear una aplicación de función 1.x [aquí](./functions-versions.md#creating-1x-apps).

[Los metadatos de OpenAPI](https://swagger.io/) permiten que una función que hospede una API de REST pueda ser consumida por una gran variedad de software. Este software incluye ofertas de Microsoft como PowerApps y la [característica API Apps de Azure App Service](../app-service/overview.md), herramientas de desarrollo de terceros, como [Postman](https://www.getpostman.com/docs/importing_swagger) y [muchos otros paquetes](https://swagger.io/tools/).

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

>[!TIP]
>Se recomienda empezar por el [tutorial de introducción](./functions-openapi-definition.md) y, luego, volver a este documento para obtener más información sobre características específicas.

## <a name="enable-openapi-definition-support"></a><a name="enable"></a>Habilitar la compatibilidad con definiciones de OpenAPI
Puede configurar todas las opciones de OpenAPI en la página **Definición de la API** de **Características de la plataforma** de la aplicación de función.

> [!NOTE]
> Actualmente, el entorno de ejecución beta no admite la característica de definición de la API de función.

Para habilitar la generación de una definición de OpenAPI hospedada y una definición de inicio rápido, establezca **Origen de definición de API** en **Functions (versión preliminar)** . **Dirección URL externa** permite que su función utilice una definición de OpenAPI hospedada en otro lugar.

## <a name="generate-a-swagger-skeleton-from-your-functions-metadata"></a><a name="generate-definition"></a>Generación de un esqueleto de Swagger a partir de los metadatos de la función.
Una plantilla puede ayudarle a empezar a escribir la primera definición de OpenAPI. La característica de plantilla de definición crea una definición de OpenAPI dispersa con todos los metadatos del archivo function.json para cada una de las funciones de desencadenador HTTP. Necesitará rellenar más información sobre la API con la [especificación de OpenAPI](https://swagger.io/specification/), como las plantillas de solicitud y respuesta.

Para obtener instrucciones paso a paso, consulte el [Tutorial de inicio](./functions-openapi-definition.md).

### <a name="available-templates"></a><a name="templates"></a>Plantillas disponibles

|Nombre| Descripción |
|:-----|:-----|
|Definición generada|Una definición de OpenAPI con la cantidad máxima de información que se puede inferir a partir de los metadatos existentes de la función.|

### <a name="included-metadata-in-the-generated-definition"></a><a name="quickstart-details"></a>Metadatos incluidos en la definición generada

En la tabla siguiente se representa la configuración de Azure Portal y los datos correspondientes en function.json, tal y como se asignan al esqueleto de Swagger generado.

|Swagger.json|IU del portal|Function.json|
|:----|:-----|:-----|
|[Host](https://swagger.io/specification/#fixed-fields-15)|**Configuración de Function App** > **Configuración de App Service** > **Información general** > **Dirección URL**|*No presente*
|[Paths](https://swagger.io/specification/#paths-object-29)|**Integrar** > **Métodos HTTP seleccionados**|Enlaces: Enrutar
|[Path Item](https://swagger.io/specification/#path-item-object-32)|**Integrar** > **Plantilla de ruta**|Enlaces: Métodos
|[Seguridad](https://swagger.io/specification/#security-scheme-object-112)|**Claves**|*No presente*|
|operationID*|**Ruta + verbos permitidos**|Ruta + verbos permitidos|

\*operationID solo es necesario para la integración con PowerApps y Flow.
> [!NOTE]
> La extensión x-ms-summary proporciona un nombre para mostrar en Logic Apps, PowerApps y Flow.
>
> Para más información, consulte [Personalizar la definición de Swagger para PowerApps](/connectors/custom-connectors/openapi-extensions).

## <a name="use-cicd-to-set-an-api-definition"></a><a name="CICD"></a>Uso de CI/CD para establecer una definición de API

 Debe habilitar el hospedaje de la definición de la API en el portal antes de habilitar el control de código fuente para modificar la definición de la API desde el control de código fuente. Siga estas instrucciones:

1. Vaya a **Definición de la API (versión preliminar)** en la configuración de la aplicación de función.
   1. Establezca **Origen de la definición de API** en **Función**.
   1. Haga clic en **Generar plantilla de definición de API** y, a continuación, en **Guardar** para crear una definición de plantilla para modificar más adelante.
   1. Tome nota de la dirección URL de la definición de API y la clave.
1. [Configurar la integración continua e implementación continua (CI/CD)](./functions-continuous-deployment.md#requirements-for-continuous-deployment).
2. Modifique swagger.json en control de código fuente en \site\wwwroot\.azurefunctions\swagger\swagger.json.

Ahora, los cambios en swagger.json en el repositorio son hospedados por la aplicación de función con la dirección URL de definición de API y clave que anotó en el paso 1.c.

## <a name="next-steps"></a>Pasos siguientes
* [Tutorial de inicio](./functions-openapi-definition.md). Pruebe nuestro tutorial para ver una definición de OpenAPI en acción.
* [Repositorio de GitHub de Azure Functions](https://github.com/Azure/Azure-Functions/). Visite el repositorio de Functions para proporcionarnos sus comentarios sobre la versión preliminar de la compatibilidad con la definición de API. Cree una entrada en GitHub indicando lo que le gustaría ver actualizado.
* [Referencia para desarrolladores de Azure Functions](functions-reference.md). Obtenga información sobre la codificación de funciones y la definición de desencadenadores y enlaces.
