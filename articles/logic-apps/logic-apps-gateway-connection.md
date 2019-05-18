---
title: Acceso a orígenes de datos locales desde Azure Logic Apps | Microsoft Docs
description: Conexión a orígenes de datos locales desde aplicaciones lógicas mediante la creación de una puerta de enlace de datos local
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: arthii, LADocs
ms.topic: article
ms.date: 10/01/2018
ms.openlocfilehash: 0580fe09c2cb6569724a9b4365233a3142645a47
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546275"
---
# <a name="connect-to-on-premises-data-sources-from-azure-logic-apps"></a>Conexión a orígenes de datos locales desde Azure Logic Apps

Para acceder a orígenes de datos locales desde las aplicaciones lógicas, cree un recurso de puerta de enlace de datos local en Azure Portal. Las aplicaciones lógicas pueden usar entonces los [conectores locales](../logic-apps/logic-apps-gateway-install.md#supported-connections). En este artículo se muestra cómo crear el recurso de puerta de enlace de Azure *después* de [descargar e instalar la puerta de enlace en el quipo local](../logic-apps/logic-apps-gateway-install.md). 

> [!TIP]
> Para conectarse a redes virtuales de Azure, considere la posibilidad de crear en su lugar un [*entorno del servicio de integración*](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md). 

Para información sobre cómo usar la puerta de enlace con otros servicios, consulte estos artículos:

* [Puerta de enlace de datos local de Microsoft Power BI](https://powerbi.microsoft.com/documentation/powerbi-gateway-onprem/)
* [Puerta de enlace de datos local de Microsoft Flow](https://flow.microsoft.com/documentation/gateway-manage/)
* [Administración de una puerta de enlace de datos local en Microsoft PowerApps](https://powerapps.microsoft.com/tutorials/gateway-management/)
* [Puerta de enlace de datos local de Azure Analysis Services](../analysis-services/analysis-services-gateway.md)

## <a name="prerequisites"></a>Requisitos previos

* Debe haber [descargado e instalado la puerta de enlace de datos en un equipo local](../logic-apps/logic-apps-gateway-install.md).

* La instalación de puerta de enlace no está asociada a un recurso de puerta de enlace en Azure. La instalación de puerta de enlace solo se puede vincular a un recurso de puerta de enlace, lo que sucede cuando se crea el recurso de puerta de enlace y se selecciona la instalación de puerta de enlace. Esta vinculación hace que la instalación de puerta de enlace no esté disponible para otros recursos.

* Cuando inicie sesión en Azure Portal y cree el recurso de puerta de enlace, asegúrese de usar la misma cuenta de inicio de sesión que se usó previamente para [instalar la puerta de enlace de datos local](../logic-apps/logic-apps-gateway-install.md#requirements) junto con la misma [suscripción de Azure](https://docs.microsoft.com/azure/architecture/cloud-adoption/governance/resource-consistency/azure-resource-access) que se utilizó para instalar la puerta de enlace. Si aún no tiene ninguna suscripción de Azure, <a href="https://azure.microsoft.com/free/" target="_blank">regístrese para obtener una cuenta gratuita de Azure</a>.

* Para crear y mantener el recurso de puerta de enlace en Azure Portal, la [cuenta del servicio de Windows](../logic-apps/logic-apps-gateway-install.md#windows-service-account) necesita al menos permisos de **Colaborador**. La puerta de enlace de datos local se ejecuta como un servicio de Windows y está configurada para usar `NT SERVICE\PBIEgwService` con las credenciales de inicio de sesión del servicio de Windows. 

  > [!NOTE]
  > Esta cuenta de servicio de Windows no es la misma que se usa para conectarse a orígenes de datos locales, ni la cuenta profesional o educativa de Azure que se usa para iniciar sesión en servicios en la nube.

## <a name="download-and-install-gateway"></a>Descarga e instalación de la puerta de enlace

Para poder continuar con los pasos descritos en este artículo, asegúrese de que la puerta de enlace ya está instalada en un equipo local.
Si aún no lo ha hecho, siga los pasos para [descargar e instalar la puerta de enlace de datos local](../logic-apps/logic-apps-gateway-install.md). 

<a name="create-gateway-resource"></a>

## <a name="create-azure-resource-for-gateway"></a>Creación de un recurso de Azure para la puerta de enlace

Después de instalar la puerta de enlace en un equipo local, puede crear un recurso de Azure para la puerta de enlace. En este paso también se asocia el recurso de puerta de enlace a su suscripción de Azure.

1. Inicie sesión en el <a href="https://portal.azure.com" target="_blank">Azure Portal</a>. Asegúrese de usar la misma dirección de correo electrónico laboral o académico de Azure que se usó para instalar la puerta de enlace.

2. En el menú principal de Azure, elija **Crear un recurso** > 
**Integración** > **Puerta de enlace de datos local**.

   ![Busque "Puerta de enlace de datos local".](./media/logic-apps-gateway-connection/find-on-premises-data-gateway.png)

3. En la página **Crear puerta de enlace de conexión**, especifique la siguiente información el recurso de puerta de enlace:

   | Propiedad | DESCRIPCIÓN | 
   |----------|-------------|
   | **Nombre** | Nombre del recurso de puerta de enlace. | 
   | **Suscripción** | Nombre de la suscripción de Azure, que debe ser la misma suscripción de la aplicación lógica. La suscripción predeterminada se basa en la cuenta de Azure que usó para iniciar sesión. | 
   | **Grupos de recursos** | Nombre del [grupo de recursos de Azure](../azure-resource-manager/resource-group-overview.md) para organizar recursos relacionados. | 
   | **Ubicación** | Azure restringe esta ubicación a la misma región que se seleccionó para el servicio en la nube de la puerta de enlace durante la [instalación de puerta de enlace](../logic-apps/logic-apps-gateway-install.md). <p>**Nota**: Asegúrese de que esta ubicación de recurso de puerta de enlace coincida con la ubicación del servicio en la nube de puerta de enlace. De lo contrario, es posible que la instalación de la puerta de enlace no aparezca en la lista de puertas de enlace instaladas para que la pueda seleccionar en el paso siguiente. Puede usar regiones diferentes para el recurso de puerta de enlace y la aplicación lógica. | 
   | **Nombre de instalación** | Si la instalación de la puerta de enlace no está seleccionada, seleccione la puerta de enlace que instaló anteriormente. | 
   | | | 

   Este es un ejemplo:

   ![Proporcionar detalles para crear la puerta de enlace de datos local](./media/logic-apps-gateway-connection/createblade.png)

4. Para agregar el recurso de puerta de enlace al panel de Azure, elija **Anclar al panel**. Cuando termine, seleccione **Crear**.

   Si quiere ver o buscar la puerta de enlace, seleccione **Todos los servicios** en el menú principal de Azure. 
   En el cuadro de búsqueda, escriba "puertas de enlace de datos locales" y luego elija **Puertas de enlace de datos locales**.

   ![Busque "Puertas de enlace de datos locales".](./media/logic-apps-gateway-connection/find-on-premises-data-gateway-enterprise-integration.png)

<a name="connect-logic-app-gateway"></a>

## <a name="connect-to-on-premises-data"></a>Conectarse a datos locales

Después de crear el recurso de puerta de enlace y asociar su suscripción de Azure con este recurso, ahora puede crear una conexión entre la aplicación lógica y el origen de datos local mediante el uso de la puerta de enlace.

1. En Azure Portal, cree o abra la aplicación lógica en el Diseñador de aplicación lógica.

2. Agregue un conector que admita conexiones locales como, por ejemplo, **SQL Server**.

3. Configure ahora la conexión:

   1. Seleccione **Connect via on-premises data gateway** (Conectarse a través de la puerta de enlace de datos local). 

   2. En **Puertas de enlace**, seleccione el recurso de puerta de enlace que creó anteriormente. 

      Aunque la ubicación de la conexión de puerta de enlace debe estar en la misma región que la aplicación lógica, puede usar una puerta de enlace que esté en otra región.

   3. Especifique un nombre de conexión único y el resto de información necesaria. 

      El nombre de conexión único le ayuda a identificar fácilmente la conexión más adelante, especialmente al crear varias conexiones. Si procede, incluya también el dominio completo para el nombre de usuario.
   
      Este es un ejemplo:

      ![Creación de conexiones entre la aplicación lógica y la puerta de enlace de datos](./media/logic-apps-gateway-connection/blankconnection.png)

   4. Cuando termine, seleccione **Crear**. 

La conexión de puerta de enlace ya está lista para que la use la aplicación lógica.

## <a name="edit-connection"></a>Edición de la conexión

Después de crear una conexión de puerta de enlace para la aplicación lógica, es posible que quiera actualizar más tarde la configuración de esa conexión concreta.

1. Busque la conexión de puerta de enlace:

   * Para encontrar todas las conexiones de API únicamente para su aplicación lógica, en el menú de la aplicación lógica, seleccione **Conexiones de API** en **Herramientas de desarrollo**. 
   
     ![Vaya a la aplicación lógica y seleccione "Conexiones de API".](./media/logic-apps-gateway-connection/logic-app-find-api-connections.png)

   * Para encontrar todas las conexiones de API asociadas a su suscripción de Azure: 

     * En el menú principal de Azure, vaya a **Todos los servicios** > **Web** > **Conexiones de API**. 
     * O bien, vaya a **Todos los recursos** en el menú principal de Azure.

2. Seleccione la conexión de puerta de enlace que quiere ver o editar y elija **Editar la conexión de API**.

   > [!TIP]
   > Si las actualizaciones no surten efecto, intente [detener y reiniciar el servicio de Windows de puerta de enlace](./logic-apps-gateway-install.md#restart-gateway).

<a name="change-delete-gateway-resource"></a>

## <a name="delete-gateway-resource"></a>Eliminación del recurso de puerta de enlace

Para crear un recurso de puerta de enlace diferente, asociar la puerta de enlace a un recurso diferente o quitar el recurso de puerta de enlace, puede eliminar el recurso de puerta de enlace sin que afecte a la instalación de puerta de enlace. 

1. En el menú principal de Azure, vaya a **Todos los recursos**. 

2. Busque y seleccione el recurso de puerta de enlace.

3. Si aún no está seleccionada, en el menú de recursos de puerta de enlace, seleccione **Puerta de enlace de datos local**. 

4. En la barra de herramientas de recursos, elija **Eliminar**.

<a name="faq"></a>

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

[!INCLUDE [existing-gateway-location-changed](../../includes/logic-apps-existing-gateway-location-changed.md)]

## <a name="get-support"></a>Obtención de soporte técnico

* Si tiene alguna duda, visite el [foro de Azure Logic Apps](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps).
* Para enviar ideas sobre características o votar sobre ellas, visite el [sitio de comentarios de los usuarios de Logic Apps](https://aka.ms/logicapps-wish).

## <a name="next-steps"></a>Pasos siguientes

* [Protección de las aplicaciones lógicas](./logic-apps-securing-a-logic-app.md)
* [Ejemplos y escenarios habituales de las aplicaciones lógicas](./logic-apps-examples-and-scenarios.md)
