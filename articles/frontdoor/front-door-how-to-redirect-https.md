---
title: Crear una instancia de Front Door con redirección de HTTP a HTTPS mediante Azure Portal
description: Aprenda a crear una instancia de Front Door con el tráfico redirigido de HTTP a HTTPS mediante Azure Portal.
services: front-door
author: duongau
ms.service: frontdoor
ms.topic: how-to
ms.date: 5/21/2019
ms.author: duau
ms.openlocfilehash: fe2159f0eeb9d01081e6a25e7a88ceff4f1e361c
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89399697"
---
# <a name="create-a-front-door-with-http-to-https-redirection-using-the-azure-portal"></a>Crear una instancia de Front Door con redirección de HTTP a HTTPS mediante Azure Portal

Puede usar Azure Portal para crear una instancia de [Front Door](front-door-overview.md) con un certificado para la terminación TLS. Una regla de enrutamiento se usa para redirigir el tráfico de HTTP a HTTPS.

En este artículo aprenderá a:

> [!div class="checklist"]
> * Crear una instancia de Front Door con un recurso de aplicación web existente
> * Agregar un dominio personalizado con un certificado TLS/SSL 
> * Configurar la redirección HTTPS en el dominio personalizado

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-a-front-door-with-an-existing-web-app-resource"></a>Crear una instancia de Front Door con un recurso de aplicación web existente

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. Haga clic en **Crear un recurso** en la esquina superior izquierda de Azure Portal.
3. Busque **Front Door** con la barra de búsqueda y, una vez que encuentre el tipo de recurso, haga clic en **Crear**.
4. Elija una suscripción y luego use un grupo de recursos existente o cree uno. Tenga en cuenta que la ubicación que pide la interfaz de usuario es para el grupo de recursos únicamente. La configuración de Front Door se implementará en todas las [ubicaciones POP de Azure Front Door](front-door-faq.md#what-are-the-pop-locations-for-azure-front-door).

    ![Configurar conceptos básicos de una nueva instancia de Front Door](./media/front-door-url-redirect/front-door-create-basics.png)

5. Haga clic en **Siguiente** para ir a la pestaña de configuración. La configuración de Front Door se realiza en tres pasos: la adición de un host de front-end predeterminado, la adición de servidores back-end en un grupo de back-end y, luego, la creación de reglas de enrutamiento para asignar el comportamiento de enrutamiento para el host de front-end. 

     ![Diseñador de configuración de Front Door](./media/front-door-url-redirect/front-door-designer.png)

6. Haga clic en el icono " **+** " en _Hosts front-end_ para crear un host de front-end, escriba un nombre único global para el host de front-end predeterminado para la instancia de Front Door (`\<**name**\>.azurefd.net`). Haga clic en **Agregar** para ir al paso siguiente.

     ![Agregar un host de front-end](./media/front-door-url-redirect/front-door-create-fehost.png)

7. Haga clic en el icono " **+** " en _Grupos de back-end_ para crear un grupo de back-end. Proporcione un nombre para el grupo de back-end y, luego, haga clic en **Agregar un back-end**.
8. Seleccione el tipo de host de back-end como _App service_. Seleccione la suscripción donde se hospeda la aplicación web y, luego, seleccione la aplicación web específica en la lista desplegable para **Nombre de host de back-end**.
9. Haga clic en **Agregar** para guardar el back-end y haga clic en **Agregar** de nuevo para guardar la configuración del grupo de back-end.   ![Agregar un back-end en un grupo de back-end](./media/front-door-url-redirect/front-door-create-backendpool.png)

10. Haga clic en el icono " **+** " en _Reglas de enrutamiento_ para crear una ruta. Proporcione un nombre para la ruta, digamos "HttpToHttpsRedirect" y, luego, establezca los _Protocolo aceptado_ campo **Solo HTTP**. Asegúrese de seleccionar el _host de front-end_ adecuado.  
11. En la sección _Detalles de la ruta_, establezca el _Tipo de ruta_ en **Redirigir**, asegúrese de que el _Tipo de redireccionamiento_ esté establecido en **Encontrado (302)** y _Protocolo de redirección_ esté establecido en **Solo HTTPS**. 
12. Haga clic en Agregar para guardar la regla de enrutamiento para el redireccionamiento de HTTP a HTTPS.
     ![Agregar una ruta de redireccionamiento de HTTP a HTTPS](./media/front-door-url-redirect/front-door-redirect-config-example.png)
13. Agregue otra regla de enrutamiento para controlar el tráfico HTTPS. Haga clic en el signo " **+** " en _Reglas de enrutamiento_ y proporcione un nombre para la ruta, por ejemplo "DefaultForwardingRoute" y, luego, establezca el campo _Protocolo aceptado_en **Solo HTTPS**. Asegúrese de seleccionar el _host de front-end_ adecuado.
14. En la sección de Detalles de la ruta, establezca el _Tipo de ruta_ en **Reenviar**, asegúrese de que el grupo de back-end correcto esté seleccionado y el _Protocolo de reenvío_ esté establecido en  **Solo HTTPS**. 
15. Haga clic en Agregar para guardar la regla de enrutamiento para el reenvío de solicitudes.
     ![Agregar una ruta de reenvío para el tráfico HTTPS](./media/front-door-url-redirect/front-door-forward-route-example.png)
16. Haga clic en **Revisar y crear** y, luego, en **Crear** para crear el perfil de Front Door. Vaya al recurso una vez que lo haya creado.

## <a name="add-a-custom-domain-to-your-front-door-and-enable-https-on-it"></a>Agregar un dominio personalizado a Front Door y habilitar HTTPS en él
Los pasos siguientes muestran cómo puede agregar un dominio personalizado en un recurso de existente de Front Door y, luego, habilitar el redireccionamiento de HTTP a HTTPS en él. 

### <a name="add-a-custom-domain"></a>Adición de un dominio personalizado

En este ejemplo, agrega un registro CNAME para el subdominio `www` (por ejemplo, `www.contosonews.com`).

#### <a name="create-the-cname-record"></a>Crear un registro CNAME

Agregue un registro CNAME para asignar un subdominio al host de front-end predeterminado de Front Door (`<name>.azurefd.net`, donde `<name>` es el nombre de su perfil de Front Door).

En el caso del dominio `www.contoso.com`, como ejemplo, agregue un registro CNAME que asigne el nombre `www` a `<name>.azurefd.net`.

Después de agregar CNAME, la página de registros DNS es como la del ejemplo siguiente:

![Dominio personalizado de CNAME a Front Door](./media/front-door-url-redirect/front-door-dns-cname.png)

#### <a name="onboard-the-custom-domain-on-your-front-door"></a>Incorporación del dominio personalizado a la instancia de Front Door

1. En la pestaña del diseñador de Front Door, haga clic en el icono "+" en la sección Hosts de front-end para agregar un nuevo dominio personalizado. 
2. Escriba el nombre DNS personalizado completo en el campo de nombre de host personalizado, por ejemplo `www.contosonews.com`. 
3. Una vez validada la asignación de CNAME del dominio a la instancia de Front Door, haga clic en **Agregar** para agregar el dominio personalizado.
4. Haga clic en **Guardar** para enviar los cambios.

![Menú Dominio personalizado](./media/front-door-url-redirect/front-door-add-custom-domain.png)

### <a name="enable-https-on-your-custom-domain"></a>Habilitación de HTTPS en el dominio personalizado

1. Haga clic en el dominio personalizado que se ha agregado y, en la sección **Personalizar HTTPS de dominio**, cambie el estado a **Habilitado**.
2. Puede dejar el **Tipo de administración de certificados** establecido en _Front Door administrado_ para el certificado gratuito que mantiene, administra y rota automáticamente Front Door. También puede usar su propio certificado TLS/SSL personalizado almacenado con Azure Key Vault. En este tutorial se da por supuesto el uso del certificado administrado de Front Door.
![Habilitar HTTPS para el dominio personalizado](./media/front-door-url-redirect/front-door-custom-domain-https.png)

3. Haga clic en **Actualizar** para guardar la selección y, después, haga clic en **Guardar**.
4. Haga clic en **Actualizar** tras un par de minutos y, luego, haga clic en el dominio personalizado de nuevo para ver el progreso del aprovisionamiento de certificados. 

> [!WARNING]
> Habilitar HTTPS para un dominio personalizado puede tardar varios minutos y también depende de la validación de la propiedad de dominio si el registro CNAME no está asignado directamente a su host de Front Door `<name>.azurefd.net`. Obtenga más información sobre cómo [habilitar HTTPS para un dominio personalizado](./front-door-custom-domain-https.md).

## <a name="configure-the-routing-rules-for-the-custom-domain"></a>Configurar las reglas de enrutamiento para el dominio personalizado

1. Haga clic en la regla de enrutamiento de redirección que creó anteriormente.
2. Haga clic en la lista desplegable para los hosts de front-end y seleccione el dominio personalizado para aplicar esta ruta también para el dominio.
3. Haga clic en **Update**(Actualizar).
4. Haga la misma operación para la otra regla de enrutamiento también; es decir, para la ruta de reenvío, para agregar el dominio personalizado.
5. Haga clic en **Guardar** para enviar los cambios.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear una instancia de Front Door](quickstart-create-front-door.md).
- Más información acerca de cómo [funciona Front Door](front-door-routing-architecture.md).
- Obtenga más información sobre la [redirección de URL en Front Door](front-door-url-redirect.md).
