---
title: Administración de las preferencias y la configuración de Azure Portal | Microsoft Docs
description: Puede cambiar la configuración predeterminada de Azure Portal para que se ajuste a sus propias preferencias. La configuración incluye el tiempo de espera de la sesión inactiva, la vista predeterminada, el modo de menú, el contraste, el tema, las notificaciones y la configuración de los formatos regionales y de idioma
services: azure-portal
keywords: tiempo de espera, idioma, regional
author: mgblythe
ms.author: mblythe
ms.date: 08/05/2020
ms.topic: how-to
ms.service: azure-portal
manager: mtillman
ms.openlocfilehash: 20ed84a87486f1095a90e012368b1f56d6426c8e
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88205602"
---
# <a name="manage-azure-portal-settings-and-preferences"></a>Administración de las preferencias y la configuración de Azure Portal

Puede cambiar la configuración predeterminada del Azure Portal para que se adapte a sus propias preferencias. La mayoría de los valores de configuración están disponibles en el menú **Configuración** del encabezado de página global.

![Haga una captura de pantalla que muestre los iconos globales del encabezado de página con la configuración resaltada](./media/set-preferences/header-settings.png)


## <a name="choose-your-default-subscription"></a>Elección de la suscripción predeterminada

Puede cambiar la suscripción que se abre de forma predeterminada cuando inicia sesión en Azure Portal. Esto resulta útil si tiene una suscripción principal con la que trabaja, pero utiliza otras de vez en cuando. 

:::image type="content" source="media/set-preferences/filter-subscription-default-view.png" alt-text="Filtre la lista de recursos por suscripción.":::

1. Seleccione el icono de filtro de suscripciones y directorios en el encabezado de página global.

1. Seleccione las suscripciones que desee que sean las predeterminadas al iniciar el portal. 

    :::image type="content" source="media/set-preferences/default-directory-subscription-filter.png" alt-text="Seleccione las suscripciones que desee que sean las predeterminadas al iniciar el portal."::: 


## <a name="choose-your-default-view"></a>Elija la vista predeterminada 

Puede cambiar la página que se abre de forma predeterminada cuando inicia sesión en Azure Portal.

![Haga una captura de pantalla que muestre la configuración de Azure Portal con la vista predeterminada resaltada](./media/set-preferences/default-view.png)

- No se puede personalizar el **Inicio**.  Muestra accesos directos a los servicios populares de Azure y enumera los recursos usados recientemente. También proporcionamos vínculos útiles a recursos como Microsoft Learn y el mapa de ruta de Azure.

- Los paneles se pueden personalizar para crear un área de trabajo diseñada solo para usted. Por ejemplo, puede crear un panel que sea proyecto, tarea o rol centrado. Si selecciona **Panel**, la vista predeterminada irá al panel usado más recientemente. Para obtener más información, consulte [Creación y uso compartido de paneles en Azure Portal](azure-portal-dashboards.md).

## <a name="choose-a-portal-menu-mode"></a>Elija un modo de menú del portal

El modo predeterminado del menú del portal controla cuánto espacio ocupa el menú del portal en la página.

![Haga una captura de pantalla que muestre la configuración de Azure Portal con temas resaltados](./media/set-preferences/menu-mode.png)

- Cuando el menú del portal está en modo de control **flotante**, se oculta hasta que lo necesite. Seleccione el icono de menú para abrir o cerrar el menú.

- Si elige **modo de acoplado** en el menú del portal, siempre estará visible. Puede contraer el menú para proporcionar más espacio de trabajo.

## <a name="choose-a-theme-or-enable-high-contrast"></a>Selección de un tema o habilitación de contraste alto

El tema que elija afectará a los colores de fondo y de fuente que aparecen en el Azure Portal. Puede seleccionar uno de los cuatro temas de color preestablecidos. Seleccione cada miniatura para buscar el tema que mejor se adapte a sus necesidades.

En su lugar, puede elegir uno de los temas de contraste alto. Los temas de contraste alto facilitan la lectura de Azure Portal para los usuarios con deficiencias visuales e invalidan todas las demás selecciones de temas.

![Haga una captura de pantalla que muestre la configuración de Azure Portal con temas resaltados](./media/set-preferences/theme.png)

## <a name="enable-or-disable-pop-up-notifications"></a>Habilitar o deshabilitar las notificaciones emergentes

Las notificaciones son mensajes del sistema relacionados con la sesión actual. Proporcionan información como el saldo de crédito actual, cuando los recursos que acaba de crear están disponibles o confirman la última acción, por ejemplo. Cuando se activan las notificaciones emergentes, los mensajes se muestran brevemente en la esquina superior de la pantalla. 

Para habilitar las notificaciones emergentes, seleccione **Habilitar notificaciones emergentes** (o anule la selección para deshabilitarlas).

![Haga una captura de pantalla que muestre la configuración de Azure Portal con notificaciones emergentes resaltadas](./media/set-preferences/popup-notifications.png)

Para leer todas las notificaciones recibidas durante la sesión actual, seleccione **Notificaciones** en el encabezado global.

![Haga una captura de pantalla que muestre el encabezado global de Azure Portal con notificaciones resaltadas](./media/set-preferences/read-notifications.png)

Si desea leer las notificaciones de sesiones anteriores, busque eventos en el registro de actividad. Para más información, consulte [Visualización del registro de actividad](../azure-monitor/platform/activity-log.md#view-the-activity-log). 

## <a name="change-the-inactivity-timeout-setting"></a>Cambio de la configuración de tiempo de espera de inactividad

La configuración de tiempo de espera de inactividad ayuda a proteger los recursos frente al acceso no autorizado si se olvida de proteger la estación de trabajo. Una vez que haya estado inactivo durante un tiempo, se cerrará automáticamente la sesión de Azure Portal. Como usuario individual, puede cambiar la configuración de tiempo de espera manualmente. Si es administrador, puede establecerlo en el nivel de directorio de todos los usuarios del directorio.

### <a name="change-your-individual-timeout-setting-user"></a>Cambio de la configuración de tiempo de espera individual (usuario)

Seleccione la lista desplegable en **Cerrar sesión cuando esté inactivo**. Elija la duración después de la cual se cerrará la sesión de Azure Portal si está inactivo.

![Haga una captura de pantalla que muestre la configuración del portal con la configuración de tiempo de espera inactiva](./media/set-preferences/inactive-signout-user.png)

El cambio se guarda automáticamente. Si está inactivo, la sesión de Azure Portal se cerrará después de la duración establecida.

Si el administrador ha habilitado una directiva de tiempo de espera de inactividad, puede establecer la suya propia, siempre que sea menor que la configuración de nivel de directorio. Seleccione **Reemplazar directiva de tiempo de espera de inactividad del directorio** y, después, establezca un intervalo de tiempo.

![Haga una captura de pantalla que muestre la configuración del portal con invalidar la configuración de directiva de tiempo de espera de inactividad de directorio](./media/set-preferences/inactive-signout-override.png)

### <a name="change-the-directory-timeout-setting-admin"></a>Cambio de la configuración de tiempo de espera del directorio (admin)

Los administradores del [rol Administrador global](../active-directory/users-groups-roles/directory-assign-admin-roles.md#global-administrator--company-administrator) pueden exigir la aplicación de un tiempo de inactividad máximo antes de que se cierre una sesión. El tiempo de expiración de inactividad se aplica en el nivel de directorio. La configuración surte efecto para las nuevas sesiones. No se aplicará de forma inmediata a ningún usuario que ya haya iniciado sesión. Para más información sobre los directorios, vea [Introducción a Active Directory Domain Services](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview).

Si es un administrador global y quiere aplicar un valor de tiempo de expiración de inactividad para todos los usuarios de Azure Portal, siga estos pasos:

1. Seleccione el texto del vínculo **Configurar el tiempo de espera del nivel de directorio**.

    ![Captura de pantalla en la que se muestra la configuración del portal con el texto del vínculo resaltado](./media/set-preferences/settings-admin.png)

1. En la página **Configurar el tiempo de espera de inactividad del nivel de directorio**, seleccione **Habilitar el tiempo de espera de inactividad del nivel de directorio para Azure Portal** para activar la opción.

1. A continuación, escriba las **horas** y los **minutos** de tiempo máximo que un usuario puede estar inactivo antes de que su sesión se cierre de forma automática.

1. Seleccione **Aplicar**.

    ![Captura de pantalla en la que se muestra la página para establecer el tiempo de espera de inactividad del nivel de directorio](./media/set-preferences/configure.png)

Para confirmar que la directiva de tiempo de expiración de inactividad se ha establecido correctamente, seleccione **Notificaciones** en el encabezado de página global. Compruebe que se muestra una notificación correcta.

![Captura de pantalla en la que se muestra el mensaje de notificación correcta para el tiempo de expiración de inactividad del nivel de directorio](./media/set-preferences/confirmation.png)

## <a name="restore-default-settings"></a>Restaurar la configuración predeterminada

Si ha realizado cambios en la configuración de Azure Portal y desea descartarlos, seleccione **Restaurar la configuración predeterminada**. Se perderán los cambios realizados en la configuración del portal. Esta opción no afecta a las personalizaciones del panel.

![Captura de pantalla que muestra la restauración de la configuración predeterminada](./media/set-preferences/useful-links-restore-defaults.png)

## <a name="export-user-settings"></a>Exportación de la configuración de usuario

La información sobre la configuración personalizada se almacena en Azure. Puede exportar los siguientes datos de usuario:

* Paneles privados de Azure Portal
* Configuración del usuario, como suscripciones o directorios favoritos y último directorio de inicio de sesión
* Temas y otra configuración del portal personalizada

Es una buena idea exportar y revisar la configuración si tiene previsto eliminarla. Volver a generar paneles o rehacer la configuración puede llevar mucho tiempo.

Para exportar la configuración del portal, seleccione **Exportar todas las opciones de configuración**.

![Captura de pantalla que muestra la exportación de la configuración](./media/set-preferences/useful-links-export-settings.png)

La exportación de la configuración crea un archivo *.json* que contiene la configuración de usuario, como el tema de color, los favoritos y los paneles privados. Debido a la naturaleza dinámica de la configuración de usuario y el riesgo de daños en los datos, no se puede importar la configuración desde el archivo *.json*.

## <a name="delete-user-settings-and-dashboards"></a>Eliminación de la configuración de usuario y los paneles

La información sobre la configuración personalizada se almacena en Azure. Puede eliminar los siguientes datos de usuario:

* Paneles privados de Azure Portal
* Configuración del usuario, como suscripciones o directorios favoritos y último directorio de inicio de sesión
* Temas y otra configuración del portal personalizada

Es una buena idea exportar y revisar la configuración antes de eliminarla. Volver a generar paneles o rehacer la configuración personalizada puede llevar mucho tiempo.

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

Para eliminar la configuración del portal, seleccione **Eliminar todas las opciones de configuración y los paneles privados**.

![Captura de pantalla que muestra la eliminación de la configuración](./media/set-preferences/useful-links-delete-settings.png)

## <a name="change-language-and-regional-settings"></a>Cambiar la configuración regional y de idioma

Hay dos configuraciones que controlan el modo en que aparece el texto en el Azure Portal: 
- El valor **Idioma** controla el idioma que se ve para el texto en el Azure Portal. 

- **El formato regional** controla la manera en que se muestran las fechas, la hora, los números y la moneda.

Para cambiar el idioma que se usa en la Azure Portal, use la lista desplegable para seleccionar en la lista de idiomas disponibles.

La selección de formato regional cambia para mostrar las opciones regionales solo para el idioma seleccionado. Para cambiar esa selección automática, utilice la lista desplegable para elegir el formato regional que desee.

Por ejemplo, si selecciona inglés como idioma y, luego, selecciona Estados Unidos como el formato regional, la moneda se muestra en dólares estadounidenses. Si selecciona inglés como idioma y, después, selecciona Europa como el formato regional, se muestra la moneda en euros.

Seleccione **Aplicar** para actualizar la configuración de idioma y el formato regional.

   ![Haga captura de pantalla que muestre la configuración del idioma y el formato regional](./media/set-preferences/language.png)

>[!NOTE]
>Esta configuración regional y de idioma solo afecta al Azure Portal. Los vínculos de documentación que se abren en una nueva pestaña o ventana usan la configuración de idioma del explorador para determinar el idioma que se va a mostrar.
>

## <a name="next-steps"></a>Pasos siguientes

- [Métodos abreviados de teclado de Azure Portal](azure-portal-keyboard-shortcuts.md)
- [Exploradores y dispositivos compatibles](azure-portal-supported-browsers-devices.md)
- [Agregar, quitar y reorganizar favoritos](azure-portal-add-remove-sort-favorites.md)
- [Creación y uso compartido de paneles personalizados](azure-portal-dashboards.md)
- [Serie de vídeos de procedimientos de Azure Portal](azure-portal-video-series.md)
