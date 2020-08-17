---
title: 'Administración de grupos de aplicaciones para Windows Virtual Desktop: Azure Portal'
description: Cómo administrar grupos de aplicaciones de Windows Virtual Desktop con Azure Portal.
author: Heidilohr
ms.topic: tutorial
ms.date: 04/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 6baada4c3b376b936f693d45e39e5887e77ef54b
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88010096"
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

2.  Busque y seleccione **Windows Virtual Desktop**.

3.  Seleccione **Application Groups** (Grupos de aplicaciones) en el lado izquierdo de la página y, luego, elija **+ Add** (+ Agregar).

4. En la pestaña **Basics** (Aspectos básicos), seleccione el grupo de suscripciones y el grupo de recursos en el que quiere crear el grupo de aplicaciones. También puede optar por crear un grupo de recursos en lugar de seleccionar uno existente.

5. Seleccione el grupo de hosts que se asociará con el grupo de aplicaciones en el menú desplegable situado junto a **Host pool** (Grupo de hosts).

    >[!NOTE]
    >Debe seleccionar el grupo de hosts asociado con el grupo de aplicaciones. Los grupos de aplicaciones tienen aplicaciones o escritorio que se suministran desde un host de sesión y los hosts de sesión forman parte de los grupos de hosts. El grupo de aplicaciones debe estar asociado con un grupo de hosts durante la creación.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la pestaña "Basics" (Aspectos básicos) en Azure Portal.](media/basics-tab.png)

6. Si quiere agregar grupos de aplicaciones al grupo de hosts, seleccione **Host pools** (Grupos de hosts) en el menú del lado izquierdo de la pantalla.

    Luego, seleccione el nombre del grupo de hosts al que quiere agregar grupos de aplicaciones.

    Después, seleccione **Application groups** (Grupos de aplicaciones) en el lado izquierdo de la pantalla y elija **+ Add** (+ Agregar).

    Por último, seleccione el grupo de suscripción y el grupo de recursos en el que quiere crear el grupo de aplicaciones. Puede seleccionar el nombre de un grupo de recursos existente en el menú desplegable o elegir **Create new** (Crear nuevo) para crear uno.

      >[!NOTE]
      >Al agregar grupos de aplicaciones al grupo de hosts, el grupo de hosts relacionado con el grupo de aplicaciones ya está seleccionado porque se ha desplazado desde él.
      >
      > [!div class="mx-imgBorder"]
      >![Captura de pantalla de la pestaña Basics (Aspectos básicos) con el grupo de hosts preseleccionado.](media/host-pool-selected.png)

7. Seleccione **RemoteApp** en "Application group type" (Tipo de grupo de aplicaciones) y, luego, escriba un nombre para su instancia de RemoteApp.

      > [!div class="mx-imgBorder"]
      > ![Captura de pantalla de los campos "Application group type" (Tipo de grupo de aplicaciones). "RemoteApp" está resaltado.](media/remoteapp-button.png)

8.  Seleccione la pestaña **Assignments** (Asignaciones).

9.  Para publicar usuarios o grupos de usuarios en el grupo de aplicaciones, seleccione **+Add Azure AD users or user groups** (+ Agregar usuarios o grupos de usuarios de Azure AD).

10.  Seleccione el número de usuarios a los que quiere agregar las aplicaciones. Puede seleccionar uno o varios usuarios y grupos de usuarios.

     > [!div class="mx-imgBorder"]
     > ![Captura de pantalla del menú de selección de usuarios.](media/select-users.png)

11.  Elija **Seleccionar**.

12.  Seleccione la pestaña **Applications** (Aplicaciones) y, luego, elija **+Add applications** (+ Agregar aplicaciones).

13.  Para agregar una aplicación desde el menú Inicio:

      - Vaya a **Application source** (Origen de aplicación) y seleccione **Start menu** (Menú Inicio) en el menú desplegable. Luego, vaya a **Application** (Aplicación) y elija la aplicación en el menú desplegable.

     > [!div class="mx-imgBorder"]
     > ![Captura de pantalla de "Add application" (Agregar aplicación) con "Start Menu" (Menú Inicio) seleccionado.](media/add-app-start.png)

      - En **Display name** (Nombre para mostrar), escriba el nombre de la aplicación que se mostrará al usuario en su cliente.

      - Deje las demás opciones como están y seleccione **Save** (Guardar).

14. Para agregar una aplicación desde una ruta de acceso de archivo específica:

      - Vaya a **Application source** (Origen de aplicación) y seleccione **File path** (Ruta de acceso al archivo) en el menú desplegable.

      - Escriba la ruta de acceso a la aplicación en el host de sesión, registrada con el grupo de hosts asociado.

      - Escriba los detalles de la aplicación en los campos **Application name** (Nombre de la aplicación), **Display name** (Nombre para mostrar), **Icon path** (Ruta del icono) e **Icon index** (Índice de icono).

      - Seleccione **Guardar**.

     > [!div class="mx-imgBorder"]
     > ![Captura de pantalla de la página "Add application" (Agregar aplicación) con la ruta de acceso al archivo seleccionada.](media/add-app-file.png)

     Repita este proceso con cada aplicación que quiera agregar al grupo de aplicaciones.

15.  A continuación, seleccione la pestaña **Workspace** (Área de trabajo).

16.  Si quiere registrar el grupo de aplicaciones en un área de trabajo, vaya a **Register application group** (Registrar grupo de aplicaciones) y seleccione **Yes** (Sí). Si prefiere registrar el grupo de aplicaciones en otro momento, seleccione **No**.

17.  Si selecciona **Yes** (Sí), puede seleccionar un área de trabajo existente en la que registrar el grupo de aplicaciones.

       >[!NOTE]
       >Solo puede registrar el grupo de aplicaciones en áreas de trabajo creadas en la misma ubicación que el grupo de hosts. También: si ha registrado anteriormente otro grupo de aplicaciones del mismo grupo de hosts que el del nuevo grupo de aplicaciones en un área de trabajo, se seleccionará y no podrá modificarlo. Todos los grupos de aplicaciones de un grupo de hosts deben estar registrados en la misma área de trabajo.

     > [!div class="mx-imgBorder"]
     > ![Captura de pantalla de la página "Register application group" (Registrar grupo de aplicaciones) en un área de trabajo ya existente. El grupo de hosts está preseleccionado.](media/register-existing.png)

18. Opcionalmente, si quiere crear etiquetas para facilitar la organización del área de trabajo, seleccione la pestaña **Tags** (Etiquetas) y escriba los nombres de las etiquetas.

19. Cuando termine, seleccione la pestaña **Review + create** (Revisar y crear).

20. Espere un poco a que se complete el proceso de validación. Cuando haya terminado, seleccione **Create** (Crear) para implementar el grupo de aplicaciones.

El proceso de implementación realizará las siguientes tareas automáticamente:

- Crear el grupo de aplicaciones de RemoteApp
- Agregar las aplicaciones seleccionadas al grupo de aplicaciones
- Publicar el grupo de aplicaciones publicado en usuarios y grupos de usuarios seleccionados
- Registrar el grupo de aplicaciones, si ha decidido hacerlo
- Crear un vínculo a una plantilla de Azure Resource Manager (basada en la configuración) que pueda descargar y guardar para más adelante.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a crear un grupo de aplicaciones, rellenarlo con programas de RemoteApps y asignar usuarios al grupo de aplicaciones. Para información sobre cómo crear un grupo de hosts de validación, consulte el tutorial siguiente. Puede usar un grupo de hosts de validación para supervisar las actualizaciones del servicio antes de implementarlas en el entorno de producción.

> [!div class="nextstepaction"]
> [Creación de un grupo de hosts para validar las actualizaciones del servicio](./create-validation-host-pool.md)
