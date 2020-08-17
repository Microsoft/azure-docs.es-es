---
title: Establezca sus preferencias de Azure Portal | Microsoft Docs
description: Puede cambiar la configuración predeterminada de Azure Portal para que se ajuste a sus propias preferencias. La configuración incluye el tiempo de espera de la sesión inactiva, la vista predeterminada, el modo de menú, el contraste, el tema, las notificaciones y la configuración de los formatos regionales y de idioma
services: azure-portal
keywords: tiempo de espera, idioma, regional
author: mgblythe
ms.author: mblythe
ms.date: 07/30/2020
ms.topic: how-to
ms.service: azure-portal
manager: mtillman
ms.openlocfilehash: a8ce638bf61c69fb732d94d537218aabaeaa9a7c
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461953"
---
# <a name="set-your-azure-portal-preferences"></a>Configuración de las preferencias de Azure Portal

Puede cambiar la configuración predeterminada del Azure Portal para que se adapte a sus propias preferencias. Se puede cambiar cada uno de los valores que se muestran a continuación:

* [Tiempo de espera de la sesión inactiva](#change-the-idle-duration-for-inactive-sign-out)
* [Suscripción predeterminada](#choose-your-default-subscription)
* [Vista predeterminada](#choose-your-default-view)
* [Modo de menú del portal](#choose-a-portal-menu-mode)
* [Color y tema de contraste alto](#choose-a-theme)
* [Notificaciones emergentes](#enable-or-disable-pop-up-notifications)
* [Idioma y formato regional](#change-language-and-regional-settings)

## <a name="change-general-portal-settings"></a>Cambiar la configuración general del portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Configuración** en el encabezado de página global.

    ![Haga una captura de pantalla que muestre los iconos globales del encabezado de página con la configuración resaltada](./media/set-preferences/header-settings.png)

### <a name="change-the-idle-duration-for-inactive-sign-out"></a>Cambie la duración de inactividad para el cierre de sesión inactivo

La configuración de tiempo de espera de inactividad ayuda a proteger los recursos frente al acceso no autorizado si se olvida de proteger la estación de trabajo. Una vez que haya estado inactivo durante un tiempo, se cerrará automáticamente la sesión de Azure Portal.

Seleccione la lista desplegable en **Cerrar sesión cuando esté inactivo**. Elija la duración después de la cual se cerrará la sesión de Azure Portal si está inactivo.

   ![Haga una captura de pantalla que muestre la configuración del portal con la configuración de tiempo de espera inactiva](./media/set-preferences/inactive-signout-user.png)

El cambio se guarda automáticamente. Si está inactivo, la sesión de Azure Portal se registrará después de la duración establecida.

Este valor también lo puede realizar un administrador en el nivel de directorio para aplicar un tiempo de inactividad máximo. Si un administrador ha realizado una configuración de tiempo de espera de nivel de directorio, puede establecer su propia duración de cierre de sesión inactiva. Elija una configuración de hora que sea menor que la que se establece en el nivel de directorio.

Si el administrador ha habilitado una directiva de tiempo de espera de inactividad, active la casilla **Invalidar la directiva de tiempo de espera de inactividad de directorio**. Establezca un intervalo de tiempo que sea menor que la configuración de la directiva.

   ![Haga una captura de pantalla que muestre la configuración del portal con invalidar la configuración de directiva de tiempo de espera de inactividad de directorio](./media/set-preferences/inactive-signout-override.png)


> [!NOTE]
> Si es un administrador y desea aplicar un valor de tiempo de espera inactivo para todos los usuarios del Azure Portal, consulte [Establecer el tiempo de espera de inactividad en el nivel de directorio para los usuarios de la Azure Portal](admin-timeout.md)
>

### <a name="choose-your-default-subscription"></a>Elección de la suscripción predeterminada

Puede cambiar la suscripción que se abre de forma predeterminada cuando inicia sesión en Azure Portal. Esto resulta útil si tiene una suscripción principal con la que trabaja, pero utiliza otras de vez en cuando. 

:::image type="content" source="media/set-preferences/filter-subscription-default-view.png" alt-text="Filtre la lista de recursos por suscripción.":::

1. Seleccione el icono de filtro de suscripciones y directorios en la navegación superior.
1. Seleccione las suscripciones que desee que sean las predeterminadas al iniciar el portal. 

    :::image type="content" source="media/set-preferences/default-directory-subscription-filter.png" alt-text="Seleccione las suscripciones que desee que sean las predeterminadas al iniciar el portal."::: 


### <a name="choose-your-default-view"></a>Elija la vista predeterminada 

Puede cambiar la página que se abre de forma predeterminada cuando inicia sesión en Azure Portal.

   ![Haga una captura de pantalla que muestre la configuración de Azure Portal con la vista predeterminada resaltada](./media/set-preferences/default-view.png)

La configuración de vista predeterminada controla qué Azure Portal vista se muestra al iniciar sesión. Puede optar por abrir la Página principal de Azure de forma predeterminada o la vista panel.

* No se puede personalizar **Home**.  Muestra accesos directos a los servicios populares de Azure y enumera los recursos usados recientemente. También proporcionamos vínculos útiles a recursos como Microsoft Learn y el mapa de ruta de Azure.
* Los paneles se pueden personalizar para crear un área de trabajo diseñada solo para usted. Por ejemplo, puede crear un panel que sea proyecto, tarea o rol centrado. Si selecciona **Panel**, la vista predeterminada irá al panel usado más recientemente.

### <a name="choose-a-portal-menu-mode"></a>Elija un modo de menú del portal

El modo predeterminado del menú del portal controla cuánto espacio ocupa el menú del portal en la página.

* Cuando el menú del portal está en modo de control **flotante**, se oculta hasta que lo necesite. Seleccione el icono de menú para abrir o cerrar el menú.
* Si elige modo de **acoplado** en el menú del portal, siempre estará visible. Puede contraer el menú para proporcionar más espacio de trabajo. 

### <a name="choose-a-theme"></a>Elija un tema

El tema que elija afectará a los colores de fondo y de fuente que aparecen en el Azure Portal. Puede seleccionar uno de los cuatro temas de color preestablecidos. Seleccione cada miniatura para buscar el tema que mejor se adapte a sus necesidades.

   ![Haga una captura de pantalla que muestre la configuración de Azure Portal con temas resaltados](./media/set-preferences/theme.png)

En su lugar, puede elegir uno de los temas de contraste alto. La configuración de contraste alto facilita la lectura de Azure Portal para los usuarios con deficiencias visuales e invalida todas las demás selecciones de temas. Para más información, consulte [Activar el contraste alto o cambiar de tema](azure-portal-change-theme-high-contrast.md).

### <a name="enable-or-disable-pop-up-notifications"></a>Habilitar o deshabilitar las notificaciones emergentes

Las notificaciones son mensajes del sistema relacionados con la sesión actual. Proporcionan información como el saldo de crédito actual, cuando los recursos que acaba de crear están disponibles o confirman la última acción, por ejemplo. Cuando se activan las notificaciones emergentes, los mensajes se muestran brevemente en la esquina superior de la pantalla. 

Para habilitar o deshabilitar las notificaciones emergentes, seleccione o anule la selección de la casilla de verificación **Habilitar notificaciones emergentes**.

   ![Haga una captura de pantalla que muestre la configuración de Azure Portal con notificaciones emergentes resaltadas](./media/set-preferences/popup-notifications.png)

Para leer todas las notificaciones recibidas durante la sesión actual, seleccione **Notificaciones** en el encabezado global.

   ![Haga una captura de pantalla que muestre el encabezado global de Azure Portal con notificaciones resaltadas](./media/set-preferences/read-notifications.png)

Si desea leer las notificaciones de sesiones anteriores, busque eventos en el registro de actividad. Para más información, lea [Ver y recuperar eventos del registro de actividad de Azure](/azure/azure-monitor/platform/activity-log-view).

### <a name="settings-under-useful-links"></a>Configuración en vínculos útiles

Si ha realizado cambios en la configuración de Azure Portal y desea descartarlos, seleccione **Restaurar la configuración predeterminada**. Se perderán los cambios realizados en la configuración del portal. Esta opción no afecta a las personalizaciones del panel.

Para más información sobre **Exportar todas las configuraciones** o **Eliminar todas las configuraciones y paneles privados**, consulte [Exportar o eliminar la configuración de usuario](azure-portal-export-delete-settings.md).

## <a name="change-language-and-regional-settings"></a>Cambiar la configuración regional y de idioma

Hay dos configuraciones que controlan el modo en que aparece el texto en el Azure Portal. El valor **Idioma** controla el idioma que se ve para el texto en el Azure Portal. **El formato regional** controla la manera en que se muestran las fechas, la hora, los números y la moneda.

Para cambiar el idioma que se usa en la Azure Portal, use la lista desplegable para seleccionar en la lista de idiomas disponibles.

La selección de formato regional cambia para mostrar las opciones regionales solo para el idioma seleccionado. Para cambiar esa selección automática, utilice la lista desplegable para elegir el formato regional que desee.

Por ejemplo, si selecciona inglés como idioma y, luego, selecciona Estados Unidos como el formato regional, la moneda se muestra en dólares estadounidenses. Si selecciona inglés como idioma y, después, selecciona Europa como el formato regional, se muestra la moneda en euros.

Seleccione **Aplicar** para actualizar la configuración de idioma y el formato regional.

   ![Haga captura de pantalla que muestre la configuración del idioma y el formato regional](./media/set-preferences/language.png)

>[!NOTE]
>Esta configuración regional y de idioma solo afecta al Azure Portal. Los vínculos de documentación que se abren en una nueva pestaña o ventana usarán la configuración de idioma del explorador para determinar el idioma que se va a mostrar.
>

## <a name="next-steps"></a>Pasos siguientes

* [Creación y uso compartido de paneles personalizados](azure-portal-dashboards.md)
* [Serie de vídeos de procedimientos de Azure Portal](azure-portal-video-series.md)
