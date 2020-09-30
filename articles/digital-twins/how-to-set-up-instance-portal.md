---
title: Configuración de una instancia y autenticación (portal)
titleSuffix: Azure Digital Twins
description: Vea cómo configurar una instancia del servicio Azure Digital Twins mediante Azure Portal.
author: baanders
ms.author: baanders
ms.date: 7/23/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 12b0dd957cb89fc7114c752312b08f1e490499da
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89280033"
---
# <a name="set-up-an-azure-digital-twins-instance-and-authentication-portal"></a>Configuración de una instancia de Azure Digital Twins y autenticación (portal)

[!INCLUDE [digital-twins-setup-selector.md](../../includes/digital-twins-setup-selector.md)]

En este artículo se describen los pasos para **configurar una nueva instancia de Azure Digital Twins**, incluidas la creación de la instancia y la configuración de la autenticación. Después de completar este artículo, tendrá una instancia de Azure Digital Twins lista para empezar a programar.

En esta versión de este artículo se realizan los pasos manualmente, uno por uno, mediante Azure Portal. Azure Portal es una consola unificada basada en web que proporciona una alternativa a las herramientas de línea de comandos.
* Para realizar estos pasos manualmente mediante la CLI, consulte la versión de CLI de este artículo: [*Procedimiento: Configuración de una instancia y autenticación (CLI)* ](how-to-set-up-instance-cli.md).
* Para ejecutar una configuración automatizada mediante un script de implementación de ejemplo, consulte la versión con scripts de este artículo: [*Procedimiento: Configuración de una instancia y autenticación (con scripts)* ](how-to-set-up-instance-scripted.md).

[!INCLUDE [digital-twins-setup-steps-prereq.md](../../includes/digital-twins-setup-steps-prereq.md)]

## <a name="create-the-azure-digital-twins-instance"></a>Creación de una instancia de Azure Digital Twins

En esta sección, **creará una instancia de Azure Digital Twins** mediante [Azure Portal](https://ms.portal.azure.com/). Vaya al portal e inicie sesión con sus credenciales.

Una vez en el portal, seleccione _Crear un recurso_ en el menú de la página principal de servicios de Azure, para comenzar.

:::image type="content" source= "media/how-to-set-up-instance/portal/create-resource.png" alt-text="Selección de Crear un recurso en la página principal de Azure Portal":::

Busque *Azure Digital Twins* en el cuadro de búsqueda y elija el servicio de **Azure Digital Twins (versión preliminar)** en los resultados. Seleccione el botón _Crear_ para crear una nueva instancia del servicio.

:::image type="content" source= "media/how-to-set-up-instance/portal/create-azure-digital-twins.png" alt-text="Selección de Crear en la página del servicio de Azure Digital Twins":::

En la siguiente página *Crear recurso*, rellene los valores indicados a continuación:
* **Suscripción**: la suscripción de Azure que está usando.
  - **Grupo de recursos**: grupo de recursos en el que implementar la instancia. Si aún no tiene un grupo de recursos existente en mente, puede crear uno aquí mediante la selección del vínculo *Crear nuevo* y la introducción de un nombre para el nuevo grupo de recursos.
* **Ubicación**: región habilitada para Azure Digital Twins para la implementación. Para obtener más información sobre la compatibilidad regional, visite [*Productos de Azure disponibles por región (Azure Digital Twins)* ](https://azure.microsoft.com/global-infrastructure/services/?products=digital-twins).
* **Nombre del recurso**: nombre para la instancia de Azure Digital Twins. El nombre de la nueva instancia debe ser único dentro de la región de la suscripción (lo que significa que si la suscripción tiene otra instancia de Azure Digital Twins en la región que ya usa el nombre elegido, se le solicitará que elija otro nombre).

:::image type="content" source= "media/how-to-set-up-instance/portal/create-azure-digital-twins-2.png" alt-text="Paso para rellenar los valores descritos para crear un recurso de Azure Digital Twins":::

Cuando termine, seleccione _Revisar y crear_. Esta acción le llevará a una página de resumen donde puede revisar los detalles de la instancia que ha introducido y pulsar _Crear_. 

### <a name="verify-success-and-collect-important-values"></a>Comprobación de que la operación es correcta y recopilación de valores importantes

Después de pulsar *Crear*, puede ver el estado de la implementación de la instancia en las notificaciones de Azure junto con la barra de iconos del portal. La notificación le indicará cuándo se ha realizado la implementación correctamente y podrá seleccionar el botón _Ir al recurso_ para ver la instancia creada.

:::image type="content" source="media/how-to-set-up-instance/portal/notifications-deployment.png" alt-text="Vista de las notificaciones de Azure que muestra una implementación correcta y resalta el botón Ir al recurso":::

Como alternativa, si se produce un error con la implementación, la notificación le indicará el motivo. Observe el consejo del mensaje de error y vuelva a intentar crear la instancia.

>[!TIP]
>Una vez creada la instancia, para volver a su página en cualquier momento, busque el nombre de la instancia en la barra de búsqueda de Azure Portal.

A partir de la página *Información general* de la instancia, anote su *Nombre*, *Grupo de recursos* y *Nombre de host*. Estos son todos los valores importantes que puede necesitar a medida que sigue trabajando con la instancia de Azure Digital Twins. Si otros usuarios van a programar en la instancia, debe compartirlos con ellos.

:::image type="content" source="media/how-to-set-up-instance/portal/instance-important-values.png" alt-text="Resaltado de los valores importantes en la página de información general de la instancia":::

Ahora tiene lista una instancia de Azure Digital Twins. A continuación, debe proporcionar los permisos de usuario de Azure adecuados para administrarla.

## <a name="set-up-user-access-permissions"></a>Configuración de permisos de acceso de usuarios

[!INCLUDE [digital-twins-setup-role-assignment.md](../../includes/digital-twins-setup-role-assignment.md)]

En primer lugar, abra la página de la instancia de Azure Digital Twins en Azure Portal. En el menú de la instancia, seleccione *Control de acceso (IAM)* . Seleccione el botón *Agregar* en *Agregar una asignación de roles*.

:::image type="content" source="media/how-to-set-up-instance/portal/add-role-assignment-1.png" alt-text="Selección de la opción para agregar una asignación de roles en la página Control de acceso (IAM)":::.

En la siguiente página *Add role assignment* (Agregar asignación de roles), rellene los valores (el usuario que los complete debe tener [permisos suficientes](#prerequisites-permission-requirements) en la suscripción de Azure):
* **Rol**: seleccione *Propietario de Azure Digital Twins (versión preliminar)* en el menú desplegable.
* **Asignar acceso a**: seleccione *Usuario, grupo o entidad de servicio de Azure AD* en el menú desplegable.
* **Select**: busque el nombre o la dirección de correo electrónico del usuario para realizar la asignación. Cuando seleccione el resultado, el usuario aparecerá en la sección *Miembros seleccionados*.

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-set-up-instance/portal/add-role-assignment-2.png" alt-text="Relleno de los campos indicados en el diálogo Agregar una asignación de roles":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Cuando acabe de introducir los detalles, pulse el botón *Guardar*.

### <a name="verify-success"></a>Comprobación de que la operación se ha completado correctamente

Puede ver la asignación de roles que ha configurado en *Control de acceso (IAM) > Asignaciones de roles*. El usuario debe aparecer en la lista con un rol de *Propietario de Azure Digital Twins (versión preliminar)* . 

:::image type="content" source="media/how-to-set-up-instance/portal/verify-role-assignment.png" alt-text="Vista de las asignaciones de roles para una instancia de Azure Digital Twins en Azure Portal":::

Ahora tiene lista una instancia de Azure Digital Twins y los permisos asignados para administrarla. A continuación, configurará los permisos para que una aplicación cliente obtenga el acceso a ella.

## <a name="set-up-access-permissions-for-client-applications"></a>Configuración de los permisos de acceso para aplicaciones cliente

[!INCLUDE [digital-twins-setup-app-registration.md](../../includes/digital-twins-setup-app-registration.md)]

Empiece por ir a [Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) en Azure Portal (puede usar este vínculo o buscar en la barra de búsqueda del portal). Seleccione *Registros de aplicaciones* en el menú del servicio y, luego, *+ Nuevo registro*.

:::image type="content" source="media/how-to-set-up-instance/portal/new-registration.png" alt-text="Vista de la página del servicio de Azure AD en Azure Portal que resalta la opción del menú Registros de aplicaciones y el botón + Nuevo registro":::

En la página *Registrar una aplicación* que sigue, rellene los valores solicitados:
* **Name**: nombre para mostrar de una aplicación de Azure AD para asociar al registro.
* **Tipos de cuenta admitidos**: seleccione *Cuentas de este directorio organizativo solo (Solo directorio predeterminado: inquilino único)*
* **URI de redirección**: *URL de respuesta de aplicación de Azure AD* para la aplicación de Azure AD. Agregue un URI*Cliente público o nativo (móvil o escritorio)* para `http://localhost`.

Cuando termine, pulse el botón *Registrarse*.

:::image type="content" source="media/how-to-set-up-instance/portal/register-an-application.png" alt-text="Vista de la página Registrar una aplicación con los valores descritos rellenados":::

Cuando termine la configuración del registro, el portal lo redirigirá a su página de detalles.

### <a name="provide-azure-digital-twins-api-permission"></a>Concesión del permiso de API de Azure Digital Twins

A continuación, configure el registro de la aplicación que ha creado con permisos de línea base para las API de Azure Digital Twins.

En la página del portal de registro de aplicación, seleccione *Permisos de API* en el menú. En la siguiente página de permisos, pulse el botón *+ Agregar un permiso*.

:::image type="content" source="media/how-to-set-up-instance/portal/add-permission.png" alt-text="Vista del registro de aplicación en Azure Portal que resalta la opción del menú Permisos de API y el botón + Agregar un permiso":::

En la página *Solicitud de permisos de API* que sigue, cambie a la pestaña *API usadas en mi organización* y busque *Azure Digital Twins*. Seleccione *Azure Digital Twins* en los resultados de la búsqueda para continuar con la asignación de permisos para las API de Azure Digital Twins.

:::image type="content" source="media/how-to-set-up-instance/portal/request-api-permissions-1.png" alt-text="Vista del resultado de la búsqueda de la página Solicitud de permisos de API que muestra Azure Digital Twins":::

>[!NOTE]
> Si tiene una instancia de Azure Digital Twins de la versión preliminar pública anterior del servicio (antes de julio de 2020) aún en su suscripción, deberá buscar en cambio *Azure Smart Spaces Service*. Se trata del nombre anterior del mismo conjunto de API y, aparte de este paso, la experiencia será la misma.

A continuación, seleccione los permisos que quiere conceder para estas API. Expanda el permiso **Lectura (1)** y marque la casilla que indica *Read.Write* para conceder permisos de lectura y escritura para el registro de la aplicación.

:::image type="content" source="media/how-to-set-up-instance/portal/request-api-permissions-2.png" alt-text="Vista de la página Solicitud de permisos de API con la selección de permisos Read.Write para las API de Azure Digital Twins":::

Cuando termine, pulse *Agregar permisos*.

### <a name="verify-success"></a>Comprobación de que la operación se ha completado correctamente

De nuevo en la página *Permisos de API*, compruebe si ya hay una entrada para Azure Digital Twins que refleja los permisos de lectura y escritura:

:::image type="content" source="media/how-to-set-up-instance/portal/verify-api-permissions.png" alt-text="Vista del portal de los permisos de API para el registro de la aplicación de Azure AD que muestra Acceso de lectura y escritura para Azure Digital Twins":::

También puede comprobar la conexión a Azure Digital Twins en el archivo *manifest.json* del registro de la aplicación, que se actualizó automáticamente con la información de Azure Digital Twins cuando agregó los permisos de API.

Para ello, seleccione *Manifiesto* en el menú para ver el código del manifiesto del registro de la aplicación. Desplácese hasta la parte inferior de la ventana de código y busque estos campos en `requiredResourceAccess`. Los valores deben coincidir con los de la siguiente captura de pantalla:

:::image type="content" source="media/how-to-set-up-instance/portal/verify-manifest.png" alt-text="Vista del portal del manifiesto para el registro de aplicación de Azure AD. Anidado en requiredResourceAccess, hay un valor resourceAppId de 0b07f429-9f4b-4714-9392-cc5e8e80c8b0 y un valor resourceAccess > id de 4589bd03-58cb-4e6c-b17f-b580e39652f8":::

### <a name="collect-important-values"></a>Recopilación de valores importantes

A continuación, seleccione *Información general* en la barra de menús para ver los detalles del registro de la aplicación:

:::image type="content" source="media/how-to-set-up-instance/portal/app-important-values.png" alt-text="Vista del portal de los valores importantes para el registro de la aplicación":::

Tome nota del *Id. de la aplicación (cliente)* y el *Id. de directorio (inquilino)* , como se muestra en **su** página. Estos valores se necesitarán más adelante para [autenticar una aplicación cliente en las API de Azure Digital Twins](how-to-authenticate-client.md). Si usted no es quién va a escribir el código para estas aplicaciones, debe compartir estos valores con la persona que lo hará.

### <a name="other-possible-steps-for-your-organization"></a>Otros posibles pasos para la organización

[!INCLUDE [digital-twins-setup-additional-requirements.md](../../includes/digital-twins-setup-additional-requirements.md)]

## <a name="next-steps"></a>Pasos siguientes

Pruebe las llamadas individuales de la API de REST en su instancia mediante los comandos de la CLI de Azure Digital Twins: 
* [Referencia de az dt](https://docs.microsoft.com/cli/azure/ext/azure-iot/dt?view=azure-cli-latest)
* [*Procedimiento: Uso de la CLI de Azure Digital Twins*](how-to-use-cli.md).

O bien, consulte cómo conectar la aplicación cliente a la instancia mediante la escritura del código de autenticación de la aplicación cliente:
* [*Procedimiento: Escritura de código de autenticación de aplicación*](how-to-authenticate-client.md)
