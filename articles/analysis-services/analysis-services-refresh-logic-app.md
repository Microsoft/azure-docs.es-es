---
title: Actualización con Logic Apps para los modelos de Azure Analysis Services | Microsoft Docs
description: En este artículo se describe cómo codificar la actualización asincrónica de Azure Analysis Services mediante Azure Logic Apps.
author: chrislound
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 10/30/2019
ms.author: chlound
ms.custom: references_regions
ms.openlocfilehash: 7412a28b53f3b17fb888e3877ecbe50a19c4a3d3
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87552244"
---
# <a name="refresh-with-logic-apps"></a>Actualización con Logic Apps

Al usar Logic Apps y llamadas de REST, puede realizar operaciones de actualización de datos automatizadas en los modelos tabulares de Azure Analysis, incluida la sincronización de réplicas de solo lectura para el escalado horizontal de la consulta.

Para información sobre cómo usar las API de REST con Azure Analysis Services, consulte [Actualización asincrónica con la API de REST](analysis-services-async-refresh.md).

## <a name="authentication"></a>Authentication

Todas las llamadas se deben autenticar con un token válido de Azure Active Directory (OAuth 2).  En los ejemplos de este artículo se usará una entidad de servicio (SPN) para la autenticación en Azure Analysis Services. Para más información, consulte [Creación de una aplicación mediante Azure Portal](../active-directory/develop/howto-create-service-principal-portal.md).

## <a name="design-the-logic-app"></a>Diseño de la aplicación lógica

> [!IMPORTANT]
> En los ejemplos siguientes, se supone que el firewall de Azure Analysis Services está deshabilitado. Si el firewall está habilitado, la dirección IP pública del iniciador de la solicitud debe figurar en la lista blanca del firewall de Azure Analysis Services. Para más información sobre los intervalos de IP de Azure Logic Apps de cada región, consulte este tema sobre los [límites y la configuración de Azure Logic Apps](../logic-apps/logic-apps-limits-and-config.md#configuration).

### <a name="prerequisites"></a>Prerequisites

#### <a name="create-a-service-principal-spn"></a>Creación de una entidad de servicio (SPN)

Para obtener información sobre cómo crear una entidad de servicio, consulte [Creación de una entidad de servicio mediante Azure Portal](../active-directory/develop/howto-create-service-principal-portal.md).

#### <a name="configure-permissions-in-azure-analysis-services"></a>Configuración de permisos en Azure Analysis Services
 
La entidad de servicio que cree debe tener permisos de administrador de servidor en el servidor. Para más información, vea [Incorporación de una entidad de servicio al rol de administrador del servidor](analysis-services-addservprinc-admins.md).

### <a name="configure-the-logic-app"></a>Configuración de la aplicación lógica

En este ejemplo, la aplicación lógica está diseñada para desencadenarse cuando se recibe una solicitud HTTP. De este modo, se podrá usar una herramienta de orquestación, como Azure Data Factory, para desencadenar la actualización del modelo de Azure Analysis Services.

Una vez que haya creado una aplicación lógica:

1. En el diseñador de aplicación lógica, elija la primera acción como **Cuando se recibe una solicitud HTTP**.

   ![Adición de la actividad recibida HTTP](./media/analysis-services-async-refresh-logic-app/1.png)

Con este paso, se rellena con la dirección URL de HTTP POST después de que se guarde la aplicación lógica.

2. Agregue un nuevo paso y busque **HTTP**.  

   ![Adición de actividad HTTP](./media/analysis-services-async-refresh-logic-app/9.png)

   ![Adición de actividad HTTP](./media/analysis-services-async-refresh-logic-app/10.png)

3. Seleccione **HTTP** para agregar esta acción.

   ![Adición de actividad HTTP](./media/analysis-services-async-refresh-logic-app/2.png)

Configure la actividad HTTP tal como se indica abajo:

|Propiedad  |Value  |
|---------|---------|
|**Método**     |POST         |
|**URI**     | https://*región del servidor*/servers/*nombre del servidor aas*/models/*nombre de la base de datos*/refreshes <br /> <br /> Por ejemplo: https:\//westus.asazure.windows.net/servers/myserver/models/AdventureWorks/refreshes|
|**Encabezados**     |   Content-Type, application/json <br /> <br />  ![encabezados](./media/analysis-services-async-refresh-logic-app/6.png)    |
|**Cuerpo**     |   Para más información sobre la forma del cuerpo de la solicitud, consulte [Actualización asincrónica con la API de REST: POST o actualizaciones](analysis-services-async-refresh.md#post-refreshes). |
|**Autenticación**     |Active Directory OAuth         |
|**Inquilino**     |Especifique el id. de inquilino de Azure Active Directory         |
|**Audiencia**     |https://*.asazure.windows.net         |
|**Id. de cliente**     |Especifique el id. de cliente del nombre de entidad de servicio         |
|**Tipo de credencial**     |Secreto         |
|**Secreto**     |Especifique el secreto del nombre de entidad de servicio         |

Ejemplo:

![Actividad de HTTP completada](./media/analysis-services-async-refresh-logic-app/7.png)

Ahora pruebe la aplicación lógica.  En el diseñador de aplicación lógica, haga clic en **Ejecutar**.

![Prueba de la aplicación lógica](./media/analysis-services-async-refresh-logic-app/8.png)

## <a name="consume-the-logic-app-with-azure-data-factory"></a>Consumo de la aplicación lógica con Azure Data Factory

Después de guardar la aplicación lógica, consulte la actividad **Cuando se recibe una solicitud HTTP** y luego copie la **URL de HTTP POST** que se habrá generado.  Se trata de la dirección URL que Azure Data Factory puede usar para realizar la llamada asincrónica que desencadenará la aplicación lógica.

A continuación proporcionamos un ejemplo de actividad web de Azure Data Factory que realiza esta acción.

![Actividad web de Azure Data Factory](./media/analysis-services-async-refresh-logic-app/11.png)

## <a name="use-a-self-contained-logic-app"></a>Uso de una aplicación lógica autocontenida

Si no tiene pensado usar una herramienta de orquestación, como Azure Data Factory, para desencadenar la actualización del modelo, puede establecer la aplicación lógica para que desencadene la actualización según una programación.

Con el ejemplo anterior, elimine la primera actividad y reemplácela por una actividad de **programación**.

![Actividad de programación](./media/analysis-services-async-refresh-logic-app/12.png)

![Actividad de programación](./media/analysis-services-async-refresh-logic-app/13.png)

En este ejemplo, usaremos **Periodicidad**.

Cuando se haya agregado la actividad, configure el intervalo y la frecuencia, agregue un nuevo parámetro y elija **A estas horas**.

![Actividad de programación](./media/analysis-services-async-refresh-logic-app/16.png)

Seleccione las horas deseadas.

![Actividad de programación](./media/analysis-services-async-refresh-logic-app/15.png)

Guarde la aplicación lógica.

## <a name="next-steps"></a>Pasos siguientes

[Muestras](analysis-services-samples.md)  
[REST API](https://docs.microsoft.com/rest/api/analysisservices/servers)
