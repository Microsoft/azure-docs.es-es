---
title: Implementación de la extensión del Panel de acceso de Azure para Internet Explorer mediante un objeto de directiva de grupo (GPO) | Microsoft Docs
description: Cómo usar la directiva de grupo para implementar el complemento de Internet Explorer para el portal de Mis aplicaciones.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/08/2018
ms.author: mimart
ms.reviewer: asteen
ms.custom: H1Hack27Feb2017
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9b6f069489738e9dceeee350a36aa2b45715a314
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65825026"
---
# <a name="how-to-deploy-the-access-panel-extension-for-internet-explorer-using-group-policy"></a>Implementación de la extensión de panel de acceso para Internet Explorer mediante la directiva de grupo
En este tutorial se muestra cómo usar la directiva de grupo para instalar de forma remota la extensión de panel de acceso para Internet Explorer en los equipos de los usuarios. Esta extensión es necesaria para los usuarios de Internet Explorer que deben iniciar sesión en aplicaciones que están configuradas con un [inicio de sesión único basado en contraseña](what-is-single-sign-on.md#password-based-sso).

Se recomienda que los administradores automaticen la implementación de esta extensión. De lo contrario, los usuarios tendrán que descargar e instalar la extensión por sí mismos, lo que puede ocasionar errores y además requiere permisos de administrador. En este tutorial, se trata un método para automatizar las implementaciones de software mediante la directiva de grupo. [Más información acerca de la directiva de grupo.](https://technet.microsoft.com/windowsserver/bb310732.aspx)

La extensión de panel de acceso también está disponible para [Chrome](https://go.microsoft.com/fwLink/?LinkID=311859) y [Firefox](https://go.microsoft.com/fwLink/?LinkID=626998), ninguno de los cuales requieren permisos de administrador para la instalación.

## <a name="prerequisites"></a>Requisitos previos
* Configuró [Active Directory Domain Services](https://msdn.microsoft.com/library/aa362244%28v=vs.85%29.aspx)y unió los equipos de los usuarios a su dominio.
* Debe tener el permiso "Editar configuración" para editar el objeto de directiva de grupo (GPO). De forma predeterminada, los miembros de los siguientes grupos de seguridad tienen este permiso: administradores de dominio, administradores de empresa y propietarios del creador de directivas de grupo. [Más información.](https://technet.microsoft.com/library/cc781991%28v=ws.10%29.aspx)

## <a name="step-1-create-the-distribution-point"></a>Paso 1: Crear el punto de distribución
En primer lugar, debe colocar el paquete del instalador en una ubicación de red a la que se pueda acceder desde todas las máquinas donde desee instalar la extensión de forma remota. Para ello, siga estos pasos.

1. Inicie sesión en el servidor como administrador.
2. En la ventana **Administrador del servidor**, vaya a **Servicios de archivos y almacenamiento**.
   
    ![Abra Servicios de archivos y almacenamiento.](./media/deploy-access-panel-browser-extension/files-services.png)
3. Vaya a la pestaña **Recursos compartidos** . A continuación, haga clic en **Tareas** > **Nuevo recurso compartido...**.
   
    ![Abra Servicios de archivos y almacenamiento.](./media/deploy-access-panel-browser-extension/shares.png)
4. Complete el **Asistente para nuevo recurso compartido** y establezca permisos para asegurarse de que se pueda acceder desde los equipos de los usuarios. [Más información acerca de los recursos compartidos.](https://technet.microsoft.com/library/cc753175.aspx)
5. Descargue el paquete de Microsoft Windows Installer (archivo .msi) siguiente: [Access Panel Extension.msi](https://account.activedirectory.windowsazure.com/Applications/Installers/x64/Access%20Panel%20Extension.msi)
6. Copie el paquete del instalador en la ubicación deseada del recurso compartido.
   
    ![Copie el archivo .msi en el recurso compartido.](./media/deploy-access-panel-browser-extension/copy-package.png)
7. Compruebe que los equipos cliente tengan acceso al paquete de instalador desde el recurso compartido. 

## <a name="step-2-create-the-group-policy-object"></a>Paso 2: Crear el objeto de directiva de grupo
1. Inicie sesión en el servidor que hospeda la instalación de Active Directory Domain Services (AD DS).
2. En el Administrador del servidor, vaya a **Herramientas** > **Administración de directivas de grupo**.
   
    ![Vaya a Herramientas > Administración de directivas de grupo.](./media/deploy-access-panel-browser-extension/tools-gpm.png)
3. En el panel izquierdo de la ventana **Administración de directivas de grupo** , vea la jerarquía de unidad organizativa (OU) y determine en qué ámbito desea aplicar la directiva de grupo. Por ejemplo, puede decidir elegir una unidad organizativa pequeña para implementarla en unos cuantos usuarios para probarla o puede seleccionar una de nivel superior para implementarla en toda la organización.
   
   > [!NOTE]
   > Si desea crear o editar sus unidades organizativas (OU), vuelva al Administrador del servidor y vaya a **Herramientas** > **Usuarios y equipos de Active Directory**.
   > 
   > 
4. Cuando haya seleccionado una unidad organizativa, haga clic con el botón derecho en ella y seleccione **Crear un GPO en este dominio y vincularlo aquí...**
   
    ![Cree un nuevo GPO.](./media/deploy-access-panel-browser-extension/create-gpo.png)
5. En el símbolo **Nuevo GPO** , escriba un nombre para el nuevo objeto de directiva de grupo.
   
    ![Asigne nombre al nuevo GPO.](./media/deploy-access-panel-browser-extension/name-gpo.png)
6. Haga clic con el botón derecho en el objeto de directiva de grupo que acaba de crear y seleccione **Editar**.
   
    ![Edite el nuevo GPO.](./media/deploy-access-panel-browser-extension/edit-gpo.png)

## <a name="step-3-assign-the-installation-package"></a>Paso 3: Asignar el paquete de instalación
1. Determine si desea implementar la extensión según **Configuración del equipo** o **Configuración de usuario**. Cuando use [Configuración del equipo](https://technet.microsoft.com/library/cc736413%28v=ws.10%29.aspx), la extensión se instalará en el equipo sin tener en cuenta qué usuarios inician sesión en él. Con [Configuración de usuario](https://technet.microsoft.com/library/cc781953%28v=ws.10%29.aspx), la extensión se instalará automáticamente para los usuarios independientemente de en qué equipo inicien sesión.
2. En el panel izquierdo de la ventana **Editor de administración de directivas de grupo** , vaya a cualquiera de las siguientes rutas de carpeta, en función del tipo de configuración que eligiera:
   
   * `Computer Configuration/Policies/Software Settings/`
   * `User Configuration/Policies/Software Settings/`
3. Haga clic con el botón derecho en **Instalación de software** y después seleccione **Nuevo** > **Paquete...**
   
    ![Cree un nuevo paquete de instalación de software.](./media/deploy-access-panel-browser-extension/new-package.png)
4. Vaya a la carpeta compartida que contiene el paquete del instalador del [Paso 1: Crear el punto de distribución](#step-1-create-the-distribution-point), seleccione el archivo .msi y haga clic en **Abrir**.
   
   > [!IMPORTANT]
   > Si el recurso compartido se encuentra en este mismo servidor, compruebe que tiene acceso el archivo .msi a través de la ruta de archivo de red, en lugar de la ruta de archivo local.
   > 
   > 
   
    ![Seleccione el paquete de instalación de la carpeta compartida.](./media/deploy-access-panel-browser-extension/select-package.png)
5. En el símbolo **Implementar software**, seleccione **Asignado** para el método de implementación A continuación, haga clic en **Aceptar**.
   
    ![Seleccione Asignado y haga clic en Aceptar.](./media/deploy-access-panel-browser-extension/deployment-method.png)

Ahora se implementa la extensión en la unidad organizativa que seleccionó. [Más información acerca de la instalación de software de directiva de grupo.](https://technet.microsoft.com/library/cc738858%28v=ws.10%29.aspx)

## <a name="step-4-auto-enable-the-extension-for-internet-explorer"></a>Paso 4: Habilitar automáticamente la extensión para Internet Explorer
Además de ejecutar el programa de instalación, todas las extensiones de Internet Explorer deben habilitarse explícitamente para poder usarlas. Siga estos pasos para habilitar la extensión de panel de acceso mediante la directiva de grupo:

1. En la ventana **Editor de administración de directivas de grupo**, vaya a cualquiera de las siguientes rutas, en función del tipo de configuración que eligiera en el [Paso 3: Asignar el paquete de instalación](#step-3-assign-the-installation-package):
   
   * `Computer Configuration/Policies/Administrative Templates/Windows Components/Internet Explorer/Security Features/Add-on Management`
   * `User Configuration/Policies/Administrative Templates/Windows Components/Internet Explorer/Security Features/Add-on Management`
2. Haga clic con el botón derecho en **Lista de complementos** y seleccione **Editar**.
    ![Edite la lista de complementos.](./media/deploy-access-panel-browser-extension/edit-add-on-list.png)
3. En la ventana **Lista de complementos**, seleccione **Habilitado**. Después, en la sección **Opciones**, haga clic en **Mostrar**.
   
    ![Haga clic en Habilitar y después en Mostrar.](./media/deploy-access-panel-browser-extension/edit-add-on-list-window.png)
4. En la ventana **Mostrar contenido** , realice los pasos siguientes:
   
   1. Para la primera columna (el campo **Nombre de valor**), copie y pegue el siguiente identificador de clase:`{030E9A3F-7B18-4122-9A60-B87235E4F59E}`
   2. Para la segunda columna (el campo **Valor**), escriba el siguiente valor: `1`
   3. Haga clic en **Aceptar** para cerrar la ventana **Mostrar contenido**.
      
      ![Rellene los valores como se especificó.](./media/deploy-access-panel-browser-extension/show-contents.png)
5. Haga clic en **Aceptar** para aplicar los cambios y cerrar la ventana **Lista de complementos**.

Ahora, la extensión debería estar habilitada para los equipos de la unidad organizativa seleccionada. [Más información acerca de cómo usar la directiva de grupo para habilitar o deshabilitar complementos de Internet Explorer.](https://technet.microsoft.com/library/dn454941.aspx)

## <a name="step-5-optional-disable-remember-password-prompt"></a>Paso 5 (opcional): Deshabilitar el mensaje "Recordar contraseña"
Cuando los usuarios inician sesión en sitios web mediante la extensión del panel de acceso, es posible que Internet Explorer muestre el siguiente mensaje preguntándole "¿Desea almacenar su contraseña?"

![Solicitud de contraseña](./media/deploy-access-panel-browser-extension/remember-password-prompt.png)

Si desea impedir que los usuarios vean este mensaje, a continuación, siga estos pasos para evitar que Autocompletar recuerde las contraseñas:

1. En la ventana **Editor de administración de directivas de grupo** , vaya a la ruta de acceso indicada a continuación. Esta opción de configuración solo está disponible en **Configuración de usuario**.
   
   * `User Configuration/Policies/Administrative Templates/Windows Components/Internet Explorer/`
2. Busque el valor denominado **Activar la función Autocompletar para nombres de usuario y contraseñas en formularios**.
   
   > [!NOTE]
   > Es posible que las versiones anteriores de Active Directory muestren esta configuración del modo siguiente: **No permitir que autocompletar guarde las contraseñas**. La configuración de esa opción varía de la descrita en este tutorial.
   > 
   > 
   
    ![No olvide buscar esto en Configuración de usuario.](./media/deploy-access-panel-browser-extension/disable-auto-complete.png)
3. Haga clic con el botón derecho en la configuración anterior y seleccione **Editar**.
4. En la ventana **Activar la función Autocompletar para nombres de usuario y contraseñas en formularios**, seleccione **Deshabilitado**.
   
    ![Seleccione Deshabilitar](./media/deploy-access-panel-browser-extension/disable-passwords.png)
5. Haga clic en **Aceptar** para aplicar estos cambios y cerrar la ventana.

Los usuarios ya no podrán almacenar sus credenciales ni usar Autocompletar para tener acceso a las credenciales almacenadas previamente. Sin embargo, esta directiva permite a los usuarios seguir usando Autocompletar para otros tipos de campos de formulario, como campos de búsqueda.

> [!WARNING]
> Si se habilita esta directiva después de que los usuarios hayan decidido almacenar algunas credenciales, esta directiva *no* borrará las credenciales que ya se han almacenado.
> 
> 

## <a name="step-6-testing-the-deployment"></a>Paso 6: Prueba de la implementación
Siga estos pasos para comprobar si la implementación de la extensión se realizó correctamente:

1. Si implementó con **Configuración del equipo**, inicie sesión en un equipo cliente que pertenezca a la unidad organizativa que seleccionó en el [Paso 2:. Crear el objeto de directiva de grupo](#step-2-create-the-group-policy-object). Si implementó con **Configuración de usuario**, asegúrese de iniciar sesión como un usuario que pertenezca a esa unidad organizativa.
2. Es posible que se deba iniciar sesión un par de veces para que los cambios de la directiva de grupo se actualicen totalmente en el equipo. Para forzar la actualización, abra una ventana **Símbolo del sistema** y ejecute el siguiente comando: `gpupdate /force`
3. Deberá reiniciar la máquina para que se lleve a cabo la instalación. El arranque puede tardar mucho más tiempo del habitual mientras la extensión se instala.
4. Después de reiniciar, abra **Internet Explorer**. En la esquina superior derecha de la ventana, haga clic en **Herramientas** (el icono de engranaje) y después seleccione **Administrar complementos**.
   
    ![Vaya a Herramientas > Administrar complementos.](./media/deploy-access-panel-browser-extension/manage-add-ons.png)
5. En la ventana **Administrar complementos**, compruebe que la **extensión de panel de acceso** se haya instalado y que su **Estado** se haya establecido en **Habilitado**.
   
    ![Compruebe que la extensión de panel de acceso esté instalada y habilitada.](./media/deploy-access-panel-browser-extension/verify-install.png)

## <a name="related-articles"></a>Artículos relacionados
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](what-is-single-sign-on.md)
* [Solución de problemas de la extensión del Panel de acceso para Internet Explorer](manage-access-panel-browser-extension.md)

