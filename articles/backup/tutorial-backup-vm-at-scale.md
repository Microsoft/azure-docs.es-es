---
title: 'Tutorial: Copia de seguridad de varias máquinas virtuales de Azure'
description: En este tutorial, aprenderá a crear un almacén de Recovery Services, a definir una directiva de copia de seguridad y a realizar copias de seguridad simultáneas de varias máquinas virtuales.
ms.date: 07/26/2020
ms.topic: tutorial
ms.custom: mvc
ms.openlocfilehash: a9517ffc1e37d50f7c0e57b9ed53fb8bcf55fd70
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89180582"
---
# <a name="use-azure-portal-to-back-up-multiple-virtual-machines"></a>Uso de Azure Portal para realizar la copia de seguridad de varias máquinas virtuales

Al realizar una copia de seguridad de datos en Azure, almacena esos datos en un recurso de Azure que se denomina “almacén de Recovery Services”. El recurso de almacén de Recovery Services está disponible en el menú Configuración de la mayoría de los servicios de Azure. La ventaja de tener el almacén de Recovery Services integrado en el menú Configuración de la mayoría de los servicios de Azure es la facilidad de realizar copias de seguridad de los datos. Sin embargo, resulta tedioso trabajar individualmente con cada base de datos o máquina virtual de la empresa. ¿Qué ocurre si desea realizar una copia de seguridad de los datos de todas las máquinas virtuales de un departamento o una ubicación? Realizar la copia de seguridad de varias máquinas virtuales mediante la creación de una directiva de copia de seguridad y la aplicación de esa directiva a las máquinas virtuales deseadas resulta sencillo. Este tutorial explica cómo realizar lo siguiente:

> [!div class="checklist"]
>
> * Creación de un almacén de Recovery Services
> * Definición de una directiva de copia de seguridad
> * Aplicación de la directiva de copia de seguridad para proteger varias máquinas virtuales
> * Desencadenamiento de un trabajo de copia de seguridad a petición para las máquinas virtuales protegidas

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

El almacén de Recovery Services contiene los datos de la copia de seguridad y la directiva de copia de seguridad que se aplica a las máquinas virtuales protegidas. La copia de seguridad de máquinas virtuales es un proceso local. No puede hacer copias de seguridad de una máquina virtual de una ubicación en un almacén de Recovery Services que está en otra ubicación. Por lo tanto, para cada ubicación de Azure que tenga máquinas virtuales de las que se tengan que realizar copias de seguridad, debe existir al menos un almacén de Recovery Services en esa ubicación.

1. En el menú izquierdo, seleccione **Todos los servicios**.

    ![Seleccionar Todos los servicios](./media/tutorial-backup-vm-at-scale/click-all-services.png)

1. En el cuadro de diálogo **Todos los servicios**, escriba *Recovery Services*. La lista de recursos se filtra en función de lo que especifique. En la lista de recursos, seleccione **Almacenes de Recovery Services**.

    ![Escribir y elegir Almacenes de Recovery Services](./media/tutorial-backup-vm-at-scale/all-services.png)

    Aparece la lista de almacenes de Recovery Services de la suscripción.

1. En el panel **Almacenes de Recovery Services**, seleccione **Agregar**.

    ![Agregar un almacén de Recovery Services](./media/tutorial-backup-vm-at-scale/add-button-create-vault.png)

1. En el menú Almacén de Recovery Services,

    * escriba *myRecoveryServicesVault* en **Nombre**.
    * El id. de suscripción actual aparecerá en **Suscripción**. Si tiene suscripciones adicionales, puede elegir otra suscripción para el nuevo almacén.
    * En **Grupo de recursos**, seleccione **Usar existente** y elija *myResourceGroup*. Si *myResourceGroup* no existe, seleccione **Crear nuevo** y escriba *myResourceGroup*.
    * En el menú desplegable **Ubicación**, elija *Oeste de Europa*.

    ![Valores del almacén de Recovery Services](./media/tutorial-backup-vm-at-scale/review-and-create.png)

    Un almacén de Recovery Services debe estar en la misma ubicación que las máquinas virtuales que se están protegiendo. Si tiene máquinas virtuales en varias regiones, cree un almacén de Recovery Services en cada una de ellas. En este tutorial se crea un almacén de Recovery Services en *Oeste de Europa*, porque es la ubicación donde se creó *myVM* (la máquina virtual creada con la guía de inicio rápido).

1. Cuando esté listo para crear el almacén de Recovery Services, seleccione **Crear**.

    ![Crear almacén de Recovery Services](./media/tutorial-backup-vm-at-scale/click-create-button.png)

1. La creación del almacén de Recovery Services puede tardar unos minutos. Supervise las notificaciones de estado en el área de **notificaciones** de la parte superior derecha del portal. Tras crear el almacén, se ve en la lista de almacenes de Recovery Services. Si no lo ve, haga clic en **Actualizar**.

     ![Actualizar lista de almacenes de Backup](./media/tutorial-backup-vm-at-scale/refresh-button.png)

Cuando se crea un almacén de Recovery Services, este tiene almacenamiento con redundancia geográfica de forma predeterminada. Para proporcionar resistencia de datos, el almacenamiento con redundancia geográfica replica los datos varias veces en dos regiones de Azure.

## <a name="set-backup-policy-to-protect-vms"></a>Establecer la directiva de copia de seguridad para proteger las máquinas virtuales

Después de crear el almacén de Recovery Services, el siguiente paso es configurar el almacén para el tipo de datos y establecer la directiva de copia de seguridad. La directiva de copia de seguridad es la programación de la frecuencia y el momento de captura de los puntos de recuperación. La directiva también incluye el intervalo de retención de los puntos de recuperación. Para este tutorial, supongamos que su empresa es un complejo deportivo con un hotel, estadio, restaurantes y concesiones, y que va a proteger los datos en las máquinas virtuales. En los pasos siguientes, se crea una directiva de copia de seguridad de los datos financieros.

1. En la lista Almacenes de Recovery Services, seleccione **myRecoveryServicesVault** para abrir su panel.

   ![Abrir el menú Escenario](./media/tutorial-backup-vm-at-scale/open-vault-from-list.png)

1. En el menú del panel del almacén, seleccione **Copia de seguridad** para abrir el menú Copia de seguridad.

1. En el menú Objetivo de Backup, en el menú desplegable **¿Dónde se ejecuta su carga de trabajo?** , elija *Azure*. En la lista desplegable **¿De qué desea hacer una copia de seguridad?** , elija *Máquina virtual* y seleccione **Copia de seguridad**.

    Estas acciones preparan el almacén de Recovery Services para interactuar con una máquina virtual. Los almacenes de Recovery Services tienen una directiva predeterminada que crea un punto de restauración al día y conserva los puntos de restauración durante 30 días.

    ![Objetivo de copia de seguridad](./media/tutorial-backup-vm-at-scale/backup-goal.png)

1. Para crear una nueva directiva, en el menú desplegable **Elegir directiva de copia de seguridad** del menú Directiva de copia de seguridad, seleccione *Create a new policy* (Crear nueva directiva).

    ![Cree una nueva directiva.](./media/tutorial-backup-vm-at-scale/create-new-policy.png)

1. Se abrirá el panel **Directiva de Backup**. Rellene los siguientes detalles:
   * En **Nombre de la directiva** escriba *Finance*. Especifique los siguientes cambios para la directiva de copia de seguridad:
   * Para **Frecuencia de copia de seguridad**, establezca la zona horaria en *Hora central*. Puesto que el complejo deportivo se encuentra en Texas, el propietario desea usar la hora local. Deje la frecuencia de copia de seguridad establecida en Diaria a las 3:30 a.m.
   * En **Retención de punto de copia de seguridad diaria**, establezca el período en 90 días.
   * En **Retención de punto de copia de seguridad semanal**, use el punto de restauración *Lunes* y consérvelo 52 semanas.
   * En **Retención de punto de copia de seguridad mensual**, use el punto de restauración desde el primer domingo del mes y el consérvelo 36 meses.
   * Anule la selección de la opción **Retención de punto de copia de seguridad anual**. El director de Finanzas no desea mantener los datos más allá de 36 meses.
   * Seleccione **Aceptar** para crear la directiva de copia de seguridad.

     ![Configuración de directivas de copia de seguridad](./media/tutorial-backup-vm-at-scale/set-new-policy.png)

     Después de crear la directiva de copia de seguridad, asóciela a las máquinas virtuales.

1. En **Máquinas virtuales**, seleccione **Agregar**.

     ![Agregar máquinas virtuales](./media/tutorial-backup-vm-at-scale/add-virtual-machines.png)

1. El panel **Seleccionar máquinas virtuales** se abrirá. Seleccione *myVM* y **Aceptar** para implementar la directiva de copia de seguridad en las máquinas virtuales.

    Aparecerán todas las máquinas virtuales que estén en la misma ubicación y no estén asociadas todavía con una directiva de copia de seguridad. *myVMH1* y *myVMR1* se seleccionan para asociarse a la directiva *Finanzas*.

    ![Selección de las máquinas virtuales que se van a proteger](./media/tutorial-backup-vm-at-scale/choose-vm-to-protect.png)

1. Una vez elegidas las máquinas virtuales, seleccione **Enable Backup** (Habilitar la copia de seguridad).

    Cuando termine la implementación, recibirá una notificación para indicarle que esta se completó correctamente.

## <a name="initial-backup"></a>Copia de seguridad inicial

Ha habilitado la copia de seguridad de los almacenes de Recovery Services, pero no se ha creado la copia de seguridad inicial. Es un procedimiento recomendado de recuperación ante desastres para desencadenar la primera copia de seguridad, de modo que sus datos estén protegidos.

Para ejecutar un trabajo de copia de seguridad a petición:

1. En el panel del almacén, seleccione **3** en **Elementos de copia de seguridad** para abrir el menú correspondiente.

    ![Elementos de copia de seguridad](./media/tutorial-backup-vm-at-scale/tutorial-vm-back-up-now.png)

    Se abrirá el menú **Elementos de copia de seguridad**.

1. En el menú **Elementos de copia de seguridad**, seleccione **Máquina virtual de Azure** para abrir la lista de máquinas virtuales asociadas con el almacén.

    ![Lista de máquinas virtuales](./media/tutorial-backup-vm-at-scale/three-virtual-machines.png)

1. Se abrirá la lista **Elementos de copia de seguridad**.

    ![Trabajo de copia de seguridad desencadenado](./media/tutorial-backup-vm-at-scale/initial-backup-context-menu.png)

1. En la lista **Elementos de copia de seguridad**, seleccione el botón de puntos suspensivos **...** para abrir el menú contextual.

1. En el menú contextual, seleccione **Realizar copia de seguridad ahora**.

    ![Menú contextual: seleccionar Realizar copia de seguridad ahora](./media/tutorial-backup-vm-at-scale/context-menu.png)

    Se abrirá el menú Realizar copia de seguridad ahora.

1. En el menú Realizar copia de seguridad ahora, especifique el último día de conservación del punto de recuperación y seleccione **Aceptar**.

    ![Establecimiento del último día de conservación del punto de recuperación de Realizar copia de seguridad ahora](./media/tutorial-backup-vm-at-scale/backup-now-short.png)

    Las notificaciones de implementación le permiten saber si se ha desencadenado el trabajo de copia de seguridad y que puede supervisar el progreso del trabajo en la página de trabajos de copia de seguridad. Según el tamaño de la máquina virtual, la creación de la copia de seguridad inicial puede tardar cierto tiempo.

    Cuando se complete el trabajo de copia de seguridad inicial, puede ver su estado en el menú Trabajo de copia de seguridad. El trabajo de copia de seguridad a petición creó el punto de restauración inicial de *myVM*. Si desea hacer una copia de seguridad de otras máquinas virtuales, repita estos pasos para cada una de ellas.

    ![Icono de Trabajos de copia de seguridad](./media/tutorial-backup-vm-at-scale/initial-backup-complete.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si tiene previsto seguir trabajando con los tutoriales siguientes, no elimine los recursos creados en este tutorial. Si no tiene previsto continuar, siga estos pasos para eliminar todos los recursos creados en este tutorial en Azure Portal.

1. En el panel **myRecoveryServicesVault**, seleccione **3** en **Elementos de copia de seguridad** para abrir el menú correspondiente.

    ![Apertura del menú Elementos de copia de seguridad](./media/tutorial-backup-vm-at-scale/tutorial-vm-back-up-now.png)

1. En el menú **Elementos de copia de seguridad**, seleccione **Máquina virtual de Azure** para abrir la lista de máquinas virtuales asociadas con el almacén.

    ![Lista de máquinas virtuales](./media/tutorial-backup-vm-at-scale/three-virtual-machines.png)

    Se abrirá la lista **Elementos de copia de seguridad**.

1. En el menú **Elementos de copia de seguridad**, seleccione los puntos suspensivos para abrir el menú contextual.

    ![En el menú Elementos de copia de seguridad, abrir el menú contextual](./media/tutorial-backup-vm-at-scale/context-menu-to-delete-vm.png)

1. En el menú contextual, seleccione **Detener copia de seguridad** para abrir el menú Detener copia de seguridad.

    ![Menú Detener copia de seguridad](./media/tutorial-backup-vm-at-scale/context-menu-for-delete.png)

1. En el menú **Detener copia de seguridad**, seleccione el menú desplegable superior y elija **Eliminar datos de copia de seguridad**.

1. En el cuadro de diálogo **Escriba el nombre del elemento de copia de seguridad**, escriba *myVM*.

1. Una vez comprobado el elemento de copia de seguridad (aparece una marca de verificación), se habilita el botón **Detener copia de seguridad**. Seleccione **Detener copia de seguridad** para detener la directiva y eliminar los puntos de restauración.

    ![Selección de Detener copia de seguridad para eliminar el almacén](./media/tutorial-backup-vm-at-scale/provide-reason-for-delete.png)

    >[!NOTE]
    >Los elementos eliminados se conservan en estado de eliminación temporal durante 14 días. Solo después de ese período se puede eliminar el almacén. Para más información, consulte [Eliminación de un almacén de Recovery Services de Azure Backup](backup-azure-delete-vault.md).

1. Cuando ya no quede ningún elemento en el almacén, seleccione **Eliminar**.

    ![Seleccionar Eliminar](./media/tutorial-backup-vm-at-scale/deleting-the-vault.png)

    Una vez eliminado el almacén, se volverá a la lista de almacenes de Recovery Services.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial se usa Azure Portal para las siguientes acciones:

> [!div class="checklist"]
>
> * Creación de un almacén de Recovery Services
> * Configurar el almacén para proteger las máquinas virtuales
> * Crear una directiva de retención y copia de seguridad personalizada
> * Asignar la directiva para proteger varias máquinas virtuales
> * Desencadenar la copia de seguridad a petición de las máquinas virtuales

Continúe con el tutorial siguiente para restaurar una máquina virtual de Azure desde el disco.

> [!div class="nextstepaction"]
> [Restaurar máquinas virtuales mediante la CLI](./tutorial-restore-disk.md)
