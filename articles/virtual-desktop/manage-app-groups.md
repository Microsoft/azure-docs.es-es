---
title: 'Administración de grupos de aplicaciones para Windows Virtual Desktop: Azure Portal'
description: Cómo administrar grupos de aplicaciones de Windows Virtual Desktop con Azure Portal.
author: Heidilohr
ms.topic: tutorial
ms.date: 09/04/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 52f495753d055730c1ad42444c83cf3d5cd4c1d3
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89461433"
---
# <a name="tutorial-manage-app-groups-with-the-azure-portal"></a>Tutorial: Administración de grupos de aplicaciones con Azure Portal

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop con objetos de Windows Virtual Desktop para Azure Resource Manager. Si usa Windows Virtual Desktop (clásico) sin objetos para Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/manage-app-groups-2019.md).

El grupo de aplicaciones predeterminado creado para un nuevo grupo host de Windows Virtual Desktop también publica el escritorio completo. Además, puede crear uno o varios grupos de aplicaciones de RemoteApp para el grupo host. Siga este tutorial para crear un grupo de aplicaciones de RemoteApp y publicar aplicaciones individuales de menú Inicio.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un grupo de RemoteApp
> * Conceder acceso a programas de RemoteApp.

## <a name="create-a-remoteapp-group"></a>Creación de un grupo de RemoteApp

Si ya ha creado un grupo de hosts y máquinas virtuales de host de sesión mediante Azure Portal o PowerShell, puede agregar grupos de aplicaciones desde Azure Portal con el siguiente proceso:

1.  Inicie sesión en [Azure Portal](https://portal.azure.com/).
   
   >![NOTA] Si va a iniciar sesión en el portal de US Gov, vaya a [https://portal.azure.us/](https://portal.azure.us/) en su lugar.

2.  Busque y seleccione **Windows Virtual Desktop**.

3. Puede agregar un grupo de aplicaciones directamente o puede agregarlo desde un grupo de hosts existente. Elija una de las siguientes opciones:

    - Seleccione **Application groups** (Grupos de aplicaciones) en el lado izquierdo de la página y, luego, elija **+ Add** (+ Agregar).

    - Seleccione **Host pools** (Grupos de hosts) en el menú de la izquierda de la pantalla, seleccione el nombre del grupo de hosts, seleccione **Application groups** (Grupos de aplicaciones) en el menú de la izquierda y, a continuación, seleccione **+ Add** (+ Agregar). En este caso, el grupo host ya estará seleccionado en la pestaña Basics (Básica).

4. En la pestaña **Basics** (Básica), seleccione la **suscripción** y el **grupo de recursos** en el que quiere crear el grupo de aplicaciones. También puede optar por crear un grupo de recursos en lugar de seleccionar uno existente.

5. Seleccione el **grupo de hosts** que se va a asociar con el grupo de aplicaciones en el menú desplegable.

    >[!NOTE]
    >Debe seleccionar el grupo de hosts asociado con el grupo de aplicaciones. Los grupos de aplicaciones tienen aplicaciones o escritorio que se suministran desde un host de sesión y los hosts de sesión forman parte de los grupos de hosts. El grupo de aplicaciones debe estar asociado con un grupo de hosts durante la creación.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la pestaña "Basics" (Aspectos básicos) en Azure Portal.](media/basics-tab.png)

6. Select **RemoteApp** under **Application group type** (Tipo de grupo de aplicaciones) y escriba un nombre para la aplicación remota.

      > [!div class="mx-imgBorder"]
      > ![Captura de pantalla de los campos "Application group type" (Tipo de grupo de aplicaciones). "RemoteApp" está resaltado.](media/remoteapp-button.png)

7.  Seleccione **Siguiente: Assignments >** (Asignaciones).

8.  Para asignar usuarios individuales o grupos de usuarios al grupo de aplicaciones, seleccione **+Add Azure AD users or user groups** (+ Agregar usuarios o grupos de usuarios de Azure AD).

9.  Seleccione los usuarios que desea que tengan acceso a las aplicaciones. Puede seleccionar uno o varios usuarios y grupos de usuarios.

     > [!div class="mx-imgBorder"]
     > ![Captura de pantalla del menú de selección de usuarios.](media/select-users.png)

10.  Elija **Seleccionar**.

11.  Seleccione **Siguiente: Applications >** (Aplicaciones) y, a continuación, seleccione **+Add applications** (+Agregar aplicaciones).

12.  Para agregar una aplicación desde el menú Inicio:

      - En **Application source** (Origen de aplicación), seleccione **Start menu** (Menú Inicio) en el menú desplegable. A continuación, en **Application** (Aplicación), elija la aplicación en el menú desplegable.

     > [!div class="mx-imgBorder"]
     > ![Captura de pantalla de "Add application" (Agregar aplicación) con "Start Menu" (Menú Inicio) seleccionado.](media/add-app-start.png)

      - En **Display name** (Nombre para mostrar), escriba el nombre de la aplicación que se mostrará al usuario en su cliente.

      - Deje las demás opciones como están y seleccione **Save** (Guardar).

13.  Para agregar una aplicación desde una ruta de acceso de archivo específica:

      - En **Application source** (Origen de aplicación), seleccione **File path** (Ruta de acceso al archivo) en el menú desplegable.

      - In **Application path** (Ruta de acceso de la aplicación), escriba la ruta de acceso a la aplicación en el host de sesión registrado con el grupo de hosts asociado.

      - Escriba los detalles de la aplicación en los campos **Application name** (Nombre de la aplicación), **Display name** (Nombre para mostrar), **Icon path** (Ruta del icono) e **Icon index** (Índice de icono).

      - Seleccione **Guardar**.

     > [!div class="mx-imgBorder"]
     > ![Captura de pantalla de la página "Add application" (Agregar aplicación) con la ruta de acceso al archivo seleccionada.](media/add-app-file.png)

14.  Repita este proceso con cada aplicación que quiera agregar al grupo de aplicaciones.

15.  A continuación, seleccione **Next: Workspace >** (Siguiente: Área de trabajo).

16.  Si quiere registrar el grupo de aplicaciones en un área de trabajo, seleccione **Yes** (Sí) para **Register application group** (Registrar grupo de aplicaciones). Si prefiere registrar el grupo de aplicaciones en otro momento, seleccione **No**.

17.  Si selecciona **Yes** (Sí), puede seleccionar un área de trabajo existente en la que registrar el grupo de aplicaciones.

       >[!NOTE]
       >Solo puede registrar el grupo de aplicaciones en áreas de trabajo creadas en la misma ubicación que el grupo de hosts. También: si ha registrado anteriormente otro grupo de aplicaciones del mismo grupo de hosts que el del nuevo grupo de aplicaciones en un área de trabajo, se seleccionará y no podrá modificarlo. Todos los grupos de aplicaciones de un grupo de hosts deben estar registrados en la misma área de trabajo.

     > [!div class="mx-imgBorder"]
     > ![Captura de pantalla de la página "Register application group" (Registrar grupo de aplicaciones) en un área de trabajo ya existente. El grupo de hosts está preseleccionado.](media/register-existing.png)

18.  Opcionalmente, si quiere crear etiquetas para facilitar la organización del área de trabajo, seleccione **Next: Tags >** (Siguiente: Etiquetas) y escriba los nombres de las etiquetas.

19.  Seleccione **Revisar y crear** cuando haya terminado.

20.  Espere un poco a que se complete el proceso de validación. Cuando haya terminado, seleccione **Create** (Crear) para implementar el grupo de aplicaciones.

El proceso de implementación realizará las siguientes tareas automáticamente:

- Crear el grupo de aplicaciones de RemoteApp
- Agregar las aplicaciones seleccionadas al grupo de aplicaciones
- Publicar el grupo de aplicaciones publicado en usuarios y grupos de usuarios seleccionados
- Registrar el grupo de aplicaciones, si ha decidido hacerlo
- Crear un vínculo a una plantilla de Azure Resource Manager (basada en la configuración) que pueda descargar y guardar para más adelante.

>[!IMPORTANT]
>Solo se pueden crear grupos de 50 aplicaciones para cada inquilino de Azure Active Directory. Hemos establecido este límite debido a las limitaciones del servicio para recuperar fuentes para nuestros usuarios. El límite no se aplica a los grupos de aplicaciones creados en Windows Virtual Desktop (versión clásica).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a crear un grupo de aplicaciones, rellenarlo con programas de RemoteApps y asignar usuarios al grupo de aplicaciones. Para información sobre cómo crear un grupo de hosts de validación, consulte el tutorial siguiente. Puede usar un grupo de hosts de validación para supervisar las actualizaciones del servicio antes de implementarlas en el entorno de producción.

> [!div class="nextstepaction"]
> [Creación de un grupo de hosts para validar las actualizaciones del servicio](./create-validation-host-pool.md)
