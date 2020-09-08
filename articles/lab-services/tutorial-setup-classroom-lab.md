---
title: Configuración de un laboratorio con Azure Lab Services | Microsoft Docs
description: En este tutorial, va a usar Azure Lab Services para configurar un laboratorio de clase con las máquinas virtuales que van a utilizar los alumnos de su clase.
ms.topic: tutorial
ms.date: 06/26/2020
ms.openlocfilehash: bc1b23b92c6af902797a9be354fe6514b4e225af
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85445141"
---
# <a name="tutorial-set-up-a-classroom-lab"></a>Tutorial: Configuración de un laboratorio educativo 
En este tutorial, se va a configurar un laboratorio de clase con las máquinas virtuales que van a utilizar los estudiantes de la clase.  

En este tutorial realizará lo siguiente:

> [!div class="checklist"]
> * Creación de un laboratorio educativo
> * Incorporación de usuarios al laboratorio
> * Configuración de la programación para el laboratorio
> * Envío de correo electrónico de invitación a los alumnos

## <a name="prerequisites"></a>Prerrequisitos
En este tutorial, configurará un laboratorio con máquinas virtuales para su clase. Para configurar un laboratorio educativo en una cuenta de laboratorio, debe ser miembro de alguno de estos roles en la cuenta de laboratorio: Propietario, Creador de laboratorio o Colaborador. La cuenta que usó para crear una cuenta de laboratorio se agrega automáticamente al rol de propietario. Por lo tanto, para crear un laboratorio educativo, puede usar la cuenta de usuario que utilizó para crear una cuenta de laboratorio. 

Este es el flujo de trabajo típico al usar Azure Lab Services:

1. El creador de una cuenta de laboratorio puede agregar a otros usuarios al rol **Creador de laboratorio**. Por ejemplo, el creador o administrador de la cuenta de laboratorio agrega formadores al rol **Creador de laboratorio** para que puedan crear laboratorios para sus clases. 
2. A continuación, los profesores crean laboratorios con VM para sus clases y envían vínculos de registro a los alumnos de la clase. 
3. Los alumnos pueden usar el vínculo de registro que reciban de los formadores para registrase en el laboratorio. Una vez registrados, pueden usar las máquinas virtuales de los laboratorios para hacer las tareas de clase y de casa. 

## <a name="create-a-classroom-lab"></a>Creación de un laboratorio educativo
En este paso, creará un laboratorio para su clase en Azure. 

1. Vaya al [sitio web de Azure Lab Services](https://labs.azure.com). Tenga en cuenta que Internet Explorer 11 aún no se admite. 
2. Seleccione **Iniciar sesión** y escriba las credenciales. Azure Lab Services es compatible con cuentas profesionales y cuentas Microsoft. 
3. Seleccione **New lab** (Nuevo laboratorio). 
    
    ![Creación de un laboratorio educativo](./media/tutorial-setup-classroom-lab/new-lab-button.png)
4. En la ventana **Nuevo laboratorio**, lleve a cabo las siguientes acciones: 
    1. Especifique el valor de **Name** (Nombre) para el laboratorio y seleccione **Next** (Siguiente).  

        ![Creación de un laboratorio educativo](./media/tutorial-setup-classroom-lab/new-lab-window.png)
    2. En la página **Virtual machine credentials** (Credenciales de la máquina virtual), especifique las credenciales predeterminadas de todas las máquinas virtuales del laboratorio. Especifique los valores de **Name** (Nombre) y **Password** (Contraseña) para el usuario y seleccione **Next** (Siguiente).  

        ![Ventana de nuevo laboratorio](./media/tutorial-setup-classroom-lab/virtual-machine-credentials.png)

        > [!IMPORTANT]
        > Tome nota de ambos. No se volverán a mostrar.
    3. En la página **Lab policies** (Directivas del laboratorio), seleccione **Finalizar**. 

        ![Cuota para cada usuario](./media/tutorial-setup-classroom-lab/quota-for-each-user.png)
5. Debería ver la siguiente pantalla, que muestra el estado de la creación de la máquina virtual de plantilla. Esta operación tarda hasta 20 minutos. 

    ![Estado de la creación de la plantilla de máquina virtual](./media/tutorial-setup-classroom-lab/create-template-vm-progress.png)
8. En la página **Template** (Plantilla), siga estos pasos: Estos pasos son **opcionales** para el tutorial.

    1. Seleccione **Conectar** para conectarse a la plantilla de máquina virtual. Si es una plantilla de máquina virtual Linux, decida si desea conectarse mediante SSH o RDP (si RDP está habilitado).
    3. Instale y configure el software necesario para su clase en la plantilla de máquina virtual. 
    4. **Detenga** la plantilla de máquina virtual.  

    > [!NOTE]
    > Las máquinas virtuales de plantilla incurren en un **costo** al ejecutarse, por lo que debe asegurarse de que la máquina virtual de la plantilla se cierre cuando no sea necesario que se esté ejecutando. 

## <a name="publish-the-template-vm"></a>Publicación de la plantilla de máquina virtual
En este paso, publicará la plantilla de máquina virtual. Al publicar la máquina virtual de plantilla, Azure Lab Services crea máquinas virtuales en el laboratorio mediante la plantilla. Todas las máquinas virtuales tienen la misma configuración que la plantilla.

1. En la página **Template** (Plantilla), seleccione **Publish** (Publicar) en la barra de herramientas. 

    ![Botón de publicación de plantilla](./media/tutorial-setup-classroom-lab/template-page-publish-button.png)

    > [!WARNING]
    > Una vez que publique, no se puede cancelar la publicación. 
2. En la página **Publish template** (Publicar plantilla), escriba el número de máquinas virtuales que desea crear en el laboratorio y seleccione **Publish** (Publicar). 

    ![Publicación de plantilla: número de máquinas virtuales](./media/tutorial-setup-classroom-lab/publish-template-number-vms.png)
3. Puede ver el **estado de la publicación** de la plantilla en la página. Este proceso puede tardar hasta una hora. 

    ![Publicar plantilla: progreso](./media/tutorial-setup-classroom-lab/publish-template-progress.png)
4. Espere a que se complete la publicación y, a continuación, cambie a la página **Virtual machine pool** (Grupo de máquinas virtuales), para lo que debe seleccionar **Virtual machines** (Máquinas virtuales) en el menú izquierdo o el icono de las **máquinas virtuales**. Confirme que ve las máquinas virtuales que están en el estado **Sin asignar**. Estas máquinas virtuales no están asignadas a los alumnos todavía. Deberían estar en estado **Detenida**. En esta pagina, puede iniciar una máquina virtual de un alumno, conectarse a la máquina virtual, detener la máquina virtual y eliminar la máquina virtual. Puede iniciarlas en esta página o dejar que los alumnos inicien las máquinas virtuales. 

    ![Máquinas virtuales en estado detenido](./media/tutorial-setup-classroom-lab/virtual-machines-stopped.png)   

    > [!NOTE]
    > Cuando un profesor encienda la máquina virtual de un alumno, la cuota del alumno no sufrirá cambios. La cuota de los usuarios especifica el número de horas de laboratorio disponibles para ellos fuera del tiempo de clase programado. Para más información sobre las cuotas, vea [Establecimiento de cuotas para los usuarios](how-to-configure-student-usage.md?#set-quotas-for-users).

## <a name="set-a-schedule-for-the-lab"></a>Configuración de una programación para el laboratorio
Cree un evento programado para el laboratorio de modo que las máquinas virtuales que este contiene se inicien o se detengan automáticamente en momentos concretos. La cuota de usuario (valor predeterminado: 10 horas) que especificó anteriormente es el tiempo adicional asignado a cada usuario fuera de esta hora programada. 

1. Cambie a la página **Schedules** (Programaciones) y seleccione **Add scheduled event** (Agregar evento programado) en la barra de herramientas. 

    ![Botón Add schedule (Agregar programación) en la página Schedules (Programaciones)](./media/how-to-create-schedules/add-schedule-button.png)
2. En la página **Add scheduled event** (Agregar evento programado), lleve a cabo los pasos siguientes:
    1. Confirme que esté seleccionado **Standard** (Estándar) como **Event type** (Tipo de evento).  
    2. Seleccione el valor de **start date** (fecha de inicio) para la clase. 
    4. Seleccione el valor de **start time** (hora de inicio) a la que quiere que se inicien las máquinas virtuales.
    5. Seleccione el valor de **stop time** (hora de detención) a la que se apagarán las máquinas virtuales. 
    6. Seleccione el valor de **time zone** (zona horaria) de las horas de inicio y detención que ha indicado. 
3. En la misma página **Add scheduled event** (Agregar evento programado), seleccione la programación actual en la sección **Repeat** (Repetir).  

    ![Botón Add schedule (Agregar programación) en la página Schedules (Programaciones)](./media/how-to-create-schedules/select-current-schedule.png)
5. En el cuadro de diálogo **Repeat** (Repetir), haga lo siguiente:
    1. Confirme que se ha establecido **Every week** (Cada semana) para el campo **Repeat** (Repetir). 
    2. Seleccione los días en los que quiere que se aplique la programación. En el ejemplo siguiente, está seleccionado Monday-Friday (Lunes a viernes). 
    3. Seleccione un valor en **end date** (fecha de finalización) para la programación.
    8. Seleccione **Guardar**. 

        ![Establecimiento de la programación de repetición](./media/how-to-create-schedules/set-repeat-schedule.png)

3. Ahora, en la página **Add scheduled event** (Agregar evento programado), en **Notes (optional)** (Notas [opcional]), escriba la descripción o las notas de la programación. 
4. En la página **Add scheduled event** (Agregar evento programado), seleccione **Save** (Guardar). 

    ![Programación semanal](./media/how-to-create-schedules/add-schedule-page-weekly.png)
5. Vaya hasta la fecha de inicio en el calendario, para comprobar que la programación está establecida.
    
    ![Programación en el calendario](./media/how-to-create-schedules/schedule-calendar.png)

    Para más información sobre la creación y administración de programaciones para una clase, consulte [Creación y administración de programaciones para laboratorios educativos](how-to-create-schedules.md).


## <a name="add-users-to-the-lab"></a>Incorporación de usuarios al laboratorio

1. En el menú izquierdo, seleccione **Users** (Usuarios). De forma predeterminada, está habilitada la opción **Restrict access** (Restringir acceso). Cuando esta opción está activada, los usuarios no se pueden registrar en el laboratorio aunque tengan el vínculo de registro si no aparecen en la lista de usuarios. Solo los usuarios de la lista pueden registrarse en el laboratorio mediante el vínculo de registro enviado. En este procedimiento, agregará usuarios a la lista. Como alternativa, puede desactivar **Restrict access** (Restringir acceso), que permite que los usuarios se registren en el laboratorio, siempre y cuando tengan el vínculo de registro. 
2. Seleccione **Add users** (Agregar usuarios) en la barra de herramientas y, luego, elija **Add by email address** (Agregar por dirección de correo electrónico). 

    ![Botón Agregar usuarios](./media/how-to-configure-student-usage/add-users-button.png)
1. En la página **Add users** (Agregar usuarios), escriba las direcciones de correo electrónico de los usuarios en líneas independientes, o bien en una sola línea separadas por punto y coma. 

    ![Incorporación de direcciones de correo electrónico de usuarios](./media/how-to-configure-student-usage/add-users-email-addresses.png)
4. Seleccione **Guardar**. Verá las direcciones de correo electrónico de los usuarios y sus estados (registrados o no) en la lista. 

    ![Lista de usuarios](./media/how-to-configure-student-usage/users-list-new.png)

    Una vez registrados los nombres de los usuarios en el laboratorio, aparecen en la lista. 
    

## <a name="send-invitation-emails-to-users"></a>Envío de correos electrónicos de invitación a los usuarios

1. Cambie a la vista **Users** (Usuarios) si aún no está en la página y seleccione **Invite all** (Invitar a todos) en la barra de herramientas. 

    ![Selección de alumnos](./media/tutorial-setup-classroom-lab/invite-all-button.png)
1. En la página **Send invitation by email** (Enviar invitación por correo electrónico), escriba un mensaje opcional y seleccione **Send** (Enviar). El correo electrónico incluye automáticamente el vínculo de registro. Puede obtener este vínculo de registro al seleccionar **... (puntos suspensivos)** en la barra de herramientas y **Registration link** (Vínculo de registro). 

    ![Envío del vínculo de registro por correo electrónico](./media/tutorial-setup-classroom-lab/send-email.png)
4. Verá el estado de la **invitación** en la lista **Users** (Usuarios). El estado debería cambiar a **Sending** (Enviando) y luego a **Sent on &lt;date&gt;** (Enviado el <fecha>). 

    Para más información sobre la incorporación de alumnos a una clase y la administración del uso que hacen del laboratorio, consulte [Configuración de los valores de uso de los alumnos](how-to-configure-student-usage.md).

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha creado un laboratorio para su clase en Azure. Para saber cómo el estudiante puede acceder a una máquina virtual en el laboratorio con el enlace de registro, avance al siguiente tutorial:

> [!div class="nextstepaction"]
> [Conexión a una máquina virtual en el laboratorio de clase](tutorial-connect-virtual-machine-classroom-lab.md)

