---
title: Introducción al portal para desarrolladores de Azure API Management
titleSuffix: Azure API Management
description: Obtenga información sobre el portal para desarrolladores de API Management. El portal para desarrolladores es donde los consumidores pueden encontrar las API.
services: api-management
documentationcenter: API Management
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/28/2020
ms.author: apimpm
ms.openlocfilehash: 6a8c4c3fa2bd73fa689458d6877d09900ea86938
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852164"
---
# <a name="azure-api-management-developer-portal-overview"></a>Introducción al portal para desarrolladores de Azure API Management

El portal para desarrolladores es un sitio web totalmente personalizable que se genera automáticamente con la documentación de las API. Aquí, los consumidores de API pueden detectar las API, aprender a usarlas, solicitar acceso y probarlas.

En este artículo se describen las diferencias entre las versiones autohospedadas y administradas del portal para desarrolladores de API Management. También se explica su arquitectura y se proporcionan respuestas a las preguntas más frecuentes.

![API Management developer portal](media/api-management-howto-developer-portal/cover.png)

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

> [!NOTE]
> <a name="migrate-from-legacy"></a> El nuevo portal para desarrolladores no es compatible con el portal para desarrolladores heredado y la migración automática no es posible. Debe volver a crear manualmente el contenido (páginas, texto, archivos multimedia) y personalizar la apariencia del nuevo portal. Consulte [el tutorial del portal para desarrolladores](api-management-howto-developer-portal-customize.md) para obtener instrucciones.

## <a name="managed-and-self-hosted-versions"></a><a name="managed-vs-self-hosted"></a> Versiones administradas y autohospedadas

Puede crear su portal para desarrolladores de dos maneras:

- **Versión administrada**: mediante la edición y la personalización del portal, que se integra en la instancia de API Management y es accesible a través de la dirección URL `<your-api-management-instance-name>.developer.azure-api.net`. Consulte [este artículo de la documentación](api-management-howto-developer-portal-customize.md) para aprender a acceder al portal administrado y a personalizarlo.
- **Versión autohospedada**: mediante la implementación y autohospedaje del portal fuera de una instancia de API Management. Este enfoque permite editar el código base del portal y ampliar la funcionalidad básica proporcionada; por ejemplo, implementar widgets personalizados para integraciones con sistemas de terceros. En este escenario, usted es el mantenedor del portal y la persona responsable de actualizarlo a la versión más reciente. Para información detallada e instrucciones, consulte el [repositorio de GitHub con el código fuente del portal][1] y [el tutorial para la implementación de un widget][3]. En el [tutorial de la versión administrada](api-management-howto-developer-portal-customize.md), se explica el panel administrativo del portal, que es igual para la versión administrada y para la versión autohospedada.

## <a name="portal-architectural-concepts"></a>Conceptos arquitectónicos del portal

Los componentes del portal se pueden dividir lógicamente en dos categorías: *código* y *contenido*.

El *código* se mantiene en [el repositorio de GitHub][1] e incluye lo siguiente:

- Widgets: representan elementos visuales y combinan HTML, JavaScript, estilos, configuración y asignación de contenido. Por ejemplo, una imagen, un párrafo de texto, un formulario o una lista de API.
- Definiciones de estilos: especifican cómo se pueden aplicar estilos a los widgets.
- Motor: genera páginas web estáticas a partir del contenido del portal y está escrito en JavaScript.
- Editor visual: habilita funcionalidades de creación y personalización en el explorador.

El *contenido* se divide en dos subcategorías: *contenido del portal* y *contenido de API Management*.

El *contenido del portal* es específico del portal e incluye lo siguiente:

- Páginas: por ejemplo, página de aterrizaje, tutoriales de API o entradas de blog.
- Elementos multimedia: imágenes, animaciones y otros contenidos basados en archivos.
- Diseños: plantillas, que se comparan con una dirección URL y definen cómo se muestran las páginas.
- Estilos: valores para definiciones de estilo; por ejemplo, fuentes, colores o bordes.
- Ajustes: configuración, por ejemplo, iconos de favoritos (favicon) y metadatos de sitios web.

El *contenido del portal*, a excepción de los elementos multimedia, se expresa como documentos JSON.

El *contenido de API Management* incluye entidades como API, operaciones, productos y suscripciones.

El portal se basa en una bifurcación adaptada del [marco de Paperbits](https://paperbits.io/). La funcionalidad de Paperbits original se ha ampliado para proporcionar widgets específicos de API Management (por ejemplo, una lista de API, una lista de productos, etc.) y un conector al servicio de API Management para guardar y recuperar contenido.

## <a name="frequently-asked-questions"></a><a name="faq"></a> Preguntas más frecuentes

En esta sección, respondemos preguntas frecuentes generales sobre el portal para desarrolladores. Si tiene preguntas específicas de la versión autohospedada, consulte la [sección wiki del repositorio de GitHub](https://github.com/Azure/api-management-developer-portal/wiki).

### <a name="how-can-i-migrate-from-the-preview-version-of-the-portal"></a><a id="preview-to-ga"></a> ¿Cómo puedo migrar desde la versión preliminar del portal?

Al usar la versión preliminar del portal para desarrolladores, se aprovisiona el contenido de versión preliminar en el servicio de API Management. El contenido predeterminado se ha modificado significativamente en la versión disponible con carácter general para mejorar la experiencia del usuario. También se incluyen nuevos widgets.

Si usa la versión administrada, restablezca el contenido del portal; para ello, haga clic en **Restablecer contenido** en la sección del menú **Operaciones**. La confirmación de esta operación quitará todo el contenido del portal y aprovisionará el nuevo contenido predeterminado. El motor del portal se ha actualizado automáticamente en el servicio de API Management.

![Restablecimiento del contenido del portal](media/api-management-howto-developer-portal/reset-content.png)

Si usa la versión autohospedada, use los archivos `scripts/cleanup.bat` y `scripts/generate.bat` del repositorio de GitHub para quitar el contenido existente y aprovisionar el nuevo. No se olvide de actualizar con antelación el código del portal a la versión más reciente del repositorio de GitHub.

Si no desea restablecer el contenido del portal, puede usar los widgets más recientes en todas las páginas. Los widgets existentes se han actualizado automáticamente a las últimas versiones.

Si el portal se aprovisionó después del anuncio de disponibilidad general, ya debería ofrecer el nuevo contenido predeterminado. No es necesario que realice ninguna acción.

### <a name="does-the-portal-have-all-the-features-of-the-legacy-portal"></a>¿El portal tiene todas las características del portal anterior?

El portal para desarrolladores no es compatible con las *aplicaciones* ni con las *incidencias*.

### <a name="has-the-legacy-portal-been-deprecated"></a>¿El portal heredado quedó en desuso?

Los portales heredados para desarrolladores y anunciantes ahora son características *heredadas*: solo recibirán actualizaciones de seguridad. Las nuevas características se implementarán solo en el nuevo portal para desarrolladores.

La retirada de los portales heredados se anunciará por separado. Si tiene preguntas, problemas o comentarios, plantéelos [en una incidencia de GitHub específica](https://github.com/Azure/api-management-developer-portal/issues/121).

### <a name="functionality-i-need-isnt-supported-in-the-portal"></a>No se admite la funcionalidad que necesito en el portal

Puede abrir una [solicitud de característica](https://aka.ms/apimwish) o [implementar por sí mismo la funcionalidad que falta][3]. Si implementa la funcionalidad usted mismo, puede autohospedar el portal para desarrolladores o abrir una solicitud de incorporación de cambios en GitHub para incluir los cambios de la versión administrada.

### <a name="how-can-i-automate-portal-deployments"></a>¿Cómo puedo automatizar las implementaciones del portal?

Puede acceder al contenido del portal para desarrolladores y administrarlo mediante programación a través de la API de REST, independientemente de si está usando una versión administrada o autohospedada.

La API se documenta en [la sección wiki del repositorio de GitHub][2]. Se puede usar para automatizar las migraciones del contenido del portal entre entornos; por ejemplo, de un entorno de prueba al de producción. Puede obtener más información sobre este proceso [en este artículo de la documentación](https://aka.ms/apimdocs/migrateportal) en GitHub.

### <a name="how-do-i-move-from-the-managed-to-the-self-hosted-version"></a>¿Cómo se puede pasar de la versión administrada a la versión autohospedada?

Consulte el artículo detallado en [la sección Wiki del repositorio del portal para desarrolladores en GitHub][2].

### <a name="can-i-have-multiple-developer-portals-in-one-api-management-service"></a>¿Puedo tener varios portales para desarrolladores en un servicio API Management?

Puede tener un portal administrado y varios portales autohospedados. El contenido de todos los portales se almacena en el mismo servicio API Management, por lo que serán idénticos. Si quiere diferenciar la apariencia y la funcionalidad de los portales, puede autohospedarlos con sus propios widgets personalizados, que personalizan dinámicamente las páginas en runtime, por ejemplo, en función de la dirección URL.

### <a name="does-the-portal-support-azure-resource-manager-templates-andor-is-it-compatible-with-api-management-devops-resource-kit"></a>¿Admite el portal plantillas de Azure Resource Manager o es compatible con el kit de recursos de DevOps de API Management?

No.

### <a name="is-the-portals-content-saved-with-the-backuprestore-functionality-in-api-management"></a>¿Se guarda el contenido del portal con la funcionalidad de copia de seguridad y restauración en API Management?

No.

### <a name="do-i-need-to-enable-additional-vnet-connectivity-for-the-managed-portal-dependencies"></a>¿Debo habilitar una conectividad de red virtual adicional para las dependencias del portal administrado?

En la mayoría de los casos, no.

Si el servicio API Management se encuentra en una red virtual interna, solo se puede acceder al portal para desarrolladores desde dentro de la red. El nombre de host del punto de conexión de administración debe resolverse en la dirección VIP interna del servicio desde la máquina que se usa para acceder a la interfaz administrativa del portal. Asegúrese de que el punto de conexión de administración está registrado en el DNS. En caso de que haya una configuración incorrecta, verá un error: `Unable to start the portal. See if settings are specified correctly in the configuration (...)`.

Si el servicio API Management se encuentra en una red virtual interna y accede a él a través de Application Gateway desde Internet, no olvide habilitar la conectividad con el portal para desarrolladores y los puntos de conexión de administración de API Management.

### <a name="i-have-assigned-a-custom-api-management-domain-and-the-published-portal-doesnt-work"></a>He asignado un dominio de API Management personalizado y el portal publicado no funciona

Después de actualizar el dominio, debe [volver a publicar el portal](api-management-howto-developer-portal-customize.md#publish) para que los cambios surtan efecto.

### <a name="i-have-added-an-identity-provider-and-i-cant-see-it-in-the-portal"></a>He agregado un proveedor de identidades y no lo veo en el portal

Después de configurar un proveedor de identidades (por ejemplo, AAD, AAD B2C), debe [volver a publicar el portal](api-management-howto-developer-portal-customize.md#publish) para que los cambios surtan efecto.

### <a name="i-have-set-up-delegation-and-the-portal-doesnt-use-it"></a>He configurado la delegación y el portal no la usa

Después de configurar la delegación, debe [volver a publicar el portal](api-management-howto-developer-portal-customize.md#publish) para que los cambios surtan efecto.

### <a name="my-other-api-management-configuration-changes-havent-been-propagated-in-the-developer-portal"></a>Los demás cambios de configuración de API Management no se han propagado en el portal para desarrolladores

La mayoría de los cambios de configuración (por ejemplo, red virtual, inicio de sesión y términos del producto) requieren [volver a publicar el portal](api-management-howto-developer-portal-customize.md#publish).

### <a name="im-getting-a-cors-error-when-using-the-interactive-console"></a><a name="cors"></a> Aparece un error de CORS cuando uso la consola interactiva

La consola interactiva realiza una solicitud de API del lado cliente desde el explorador. Resuelva el problema de CORS incorporando [una directiva de CORS](api-management-cross-domain-policies.md#CORS) en las API.

Puede comprobar el estado de la directiva de CORS en la sección **Portal overview** (Información general del portal) del servicio API Management en Azure Portal. Un cuadro de advertencia indica que falta una directiva o que está mal configurada.

![API Management developer portal](media/api-management-howto-developer-portal/cors-azure-portal.png)

Aplique automáticamente la directiva de CORS haciendo clic en el botón **Habilitar CORS**.

También puede habilitar CORS manualmente.

1. Haga clic en el vínculo **Manually apply it on the global level** (Aplicarlo manualmente en el nivel global) para ver el código generado de la directiva.
2. Vaya a **Todas las API** en la sección **API** del servicio API Management en Azure Portal.
3. Haga clic en el icono **</>** en la sección **Procesamiento de entrada**.
4. Inserte la directiva en la sección **<inbound>** del archivo XML. Asegúrese de que el valor de **<origin>** coincide con el dominio del portal para desarrolladores.

> [!NOTE]
> 
> Si aplica la directiva de CORS en el ámbito del producto, en lugar del ámbito de las API, y la API usa la autenticación de clave de suscripción a través de un encabezado, la consola no funcionará.
>
> El explorador emite automáticamente una solicitud HTTP OPTIONS, que no contiene un encabezado con la clave de suscripción. Como falta la clave de suscripción, API Management no puede asociar la llamada a OPTIONS con ningún producto, por lo que no puede aplicar la directiva de CORS.
>
> Como solución alternativa, puede pasar la clave de suscripción en un parámetro de consulta.

### <a name="what-permissions-do-i-need-to-edit-the-developer-portal"></a>¿Qué permisos necesito para editar el portal para desarrolladores?

Si ve el error `Oops. Something went wrong. Please try again later.` al abrir el portal en el modo administrativo, es posible que no tenga los permisos necesarios (RBAC).

Los portales heredados requieren el permiso `Microsoft.ApiManagement/service/getssotoken/action` en el ámbito de servicio (`/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.ApiManagement/service/<apim-service-name>`) para permitir que el administrador de usuarios tenga acceso a los portales. El nuevo portal requiere el permiso `Microsoft.ApiManagement/service/users/token/action` en el ámbito `/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.ApiManagement/service/<apim-service-name>/users/1`.

Puede usar el siguiente script de PowerShell para crear un rol con el permiso necesario. Recuerde cambiar el parámetro `<subscription-id>`. 

```powershell
#New Portals Admin Role 
Import-Module Az 
Connect-AzAccount 
$contributorRole = Get-AzRoleDefinition "API Management Service Contributor" 
$customRole = $contributorRole 
$customRole.Id = $null
$customRole.Name = "APIM New Portal Admin" 
$customRole.Description = "This role gives the user ability to log in to the new Developer portal as administrator" 
$customRole.Actions = "Microsoft.ApiManagement/service/users/token/action" 
$customRole.IsCustom = $true 
$customRole.AssignableScopes.Clear() 
$customRole.AssignableScopes.Add('/subscriptions/<subscription-id>') 
New-AzRoleDefinition -Role $customRole 
```
 
Una vez creado el rol, se puede conceder a cualquier usuario de la sección **Control de acceso (IAM)** de Azure Portal. Al asignar este rol a un usuario, se asignará el permiso en el ámbito del servicio. El usuario podrá generar tokens de SAS en nombre de *cualquier* usuario en el servicio. Como mínimo, este rol debe asignarse al administrador del servicio. El siguiente comando de PowerShell muestra cómo asignar el rol a un usuario `user1` en el ámbito más bajo para evitar la concesión de permisos innecesarios al usuario: 

```powershell
New-AzRoleAssignment -SignInName "user1@contoso.com" -RoleDefinitionName "APIM New Portal Admin" -Scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.ApiManagement/service/<apim-service-name>/users/1" 
```

Una vez concedidos los permisos a un usuario, este debe cerrar sesión e iniciar sesión de nuevo en Azure Portal para que los nuevos permisos surtan efecto.

### <a name="im-seeing-the-unable-to-start-the-portal-see-if-settings-are-specified-correctly--error"></a>Veo el error `Unable to start the portal. See if settings are specified correctly (...)`

Este error se muestra cuando se produce un error en una llamada `GET` a `https://<management-endpoint-hostname>/subscriptions/xxx/resourceGroups/xxx/providers/Microsoft.ApiManagement/service/xxx/contentTypes/document/contentItems/configuration?api-version=2018-06-01-preview`. La interfaz administrativa del portal emite la llamada desde el explorador.

Si el servicio API Management se encuentra en una red virtual, consulte la pregunta de conectividad de la red virtual anterior.

El error de llamada también puede deberse a un certificado TLS/SSL, que se asigna a un dominio personalizado y no es de confianza para el explorador. Como medida de mitigación, puede quitar el dominio personalizado del punto de conexión de administración (API Management revertirá al punto de conexión predeterminado con un certificado de confianza).

### <a name="whats-the-browser-support-for-the-portal"></a>¿Cuál es la compatibilidad del explorador con el portal?

| Browser                     | Compatible       |
|-----------------------------|-----------------|
| Apple Safari                | Sí<sup>1</sup> |
| Google Chrome               | Sí<sup>1</sup> |
| Microsoft Edge              | Sí<sup>1</sup> |
| Microsoft Internet Explorer | No              |
| Mozilla Firefox             | Sí<sup>1</sup> |

 <small><sup>1</sup> Se admite en las dos últimas versiones de producción.</small>

## <a name="next-steps"></a>Pasos siguientes

Más información acerca del nuevo portal para desarrolladores:

- [Acceso a la versión administrada del portal para desarrolladores y su personalización](api-management-howto-developer-portal-customize.md)
- [Configuración de la versión autohospedada del portal][2]
- [Implementación de su propio widget][3]

Examine otros recursos:

- [Repositorio de GitHub con el código fuente][1]

[1]: https://aka.ms/apimdevportal
[2]: https://github.com/Azure/api-management-developer-portal/wiki
[3]: https://aka.ms/apimdevportal/extend
