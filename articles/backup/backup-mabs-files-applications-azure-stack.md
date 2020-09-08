---
title: Copia de seguridad de archivos en máquinas virtuales de Azure Stack
description: Use Azure Backup para hacer una copia de seguridad de los archivos y aplicaciones de Azure Stack, así como para restaurarlos, en el entorno de Azure Stack.
ms.topic: conceptual
ms.date: 06/05/2018
ms.openlocfilehash: cae95c10c510969cc0553a54a506789d6be427d7
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89180990"
---
# <a name="back-up-files-and-applications-on-azure-stack"></a>Copia de seguridad de archivos y aplicaciones en Azure Stack

Puede usar Azure Backup para proteger o realizar copias de seguridad de archivos y aplicaciones en Azure Stack. Para hacer copias de seguridad de archivos y aplicaciones, instale Microsoft Azure Backup Server como una máquina virtual que se ejecuta en Azure Stack. Puede proteger los archivos de cualquier servidor de Azure Stack en la misma red virtual. Una vez que haya instalado Azure Backup Server, agregue discos de Azure para aumentar el almacenamiento local disponible para los datos de copia de seguridad a corto plazo. Azure Backup Server usa almacenamiento de Azure para la retención a largo plazo.

> [!NOTE]
> Aunque Azure Backup Server y System Center Data Protection Manager (DPM) son similares, el uso de DPM no es compatible con Azure Stack.
>

En este artículo no se trata la instalación de Azure Backup Server en el entorno de Azure Stack. Para instalar Azure Backup Server en Azure Stack, consulte el artículo [Instalación de Azure Backup Server en Azure Stack](backup-mabs-install-azure-stack.md).

## <a name="back-up-files-and-folders-in-azure-stack-vms-to-azure"></a>Copia de seguridad en Azure de archivos y carpetas de las máquinas virtuales en Azure Stack

Para configurar Azure Backup Server a fin de proteger los archivos de las máquinas virtuales de Azure Stack, abra la consola de Azure Backup Server. Usará la consola para configurar los grupos de protección y para proteger los datos de las máquinas virtuales.

1. En la consola de Azure Backup Server, seleccione **Protección** y, en la barra de herramientas, seleccione **Nuevo** para abrir el asistente para **Crear nuevo grupo de protección**.

   ![Configuración de la protección en la consola de Azure Backup Server](./media/backup-mabs-files-applications-azure-stack/1-mabs-menu-create-protection-group.png)

    El asistente puede tardar unos segundos en abrirse. Cuando se abra, seleccione **Siguiente** para avanzar a la pantalla **Seleccionar tipo de grupo de protección**.

   ![Se abre el asistente para crear nuevo grupo de protección](./media/backup-mabs-files-applications-azure-stack/2-create-new-protection-group-wiz.png)

2. En la pantalla **Seleccionar tipo de grupo de protección**, elija **Servidores** y seleccione **Siguiente**.

    ![Seleccionar tipo de grupo de protección](./media/backup-mabs-files-applications-azure-stack/3-select-protection-group-type.png)

    Se abre la pantalla **Seleccionar miembros del grupo**.

    ![Seleccionar miembros del grupo](./media/backup-mabs-files-applications-azure-stack/4-opening-screen-choose-servers.png)

3. En la pantalla **Seleccionar miembros del grupo**, seleccione **+** para expandir la lista de subelementos. Active la casilla de verificación de todos los elementos que quiere proteger. Una vez que haya seleccionado todos los elementos, seleccione **Siguiente**.

    ![Seleccione todos los elementos que desea proteger](./media/backup-mabs-files-applications-azure-stack/5-select-group-members.png)

    Microsoft recomienda poner todos los datos que compartirán una directiva de protección en un mismo grupo de protección. Para información competa sobre cómo planificar e implementar los grupos de protección, consulte el artículo [Implementar grupos de protección](/system-center/dpm/create-dpm-protection-groups) de System Center DPM.

4. En la pantalla **Seleccionar método de protección de datos**, escriba un nombre para el grupo de protección. Active la casilla de verificación para **Deseo protección a corto plazo mediante:** y **Deseo protección en línea**. Seleccione **Next** (Siguiente).

    ![Seleccionar método de protección de datos](./media/backup-mabs-files-applications-azure-stack/6-select-data-protection-method.png)

    Para seleccionar **Deseo protección en línea**, primero debe seleccionar **Deseo protección a corto plazo mediante:** disco. Azure Backup Server no puede proteger en cinta, por lo que el disco es la única opción para la protección a corto plazo.

5. En la pantalla **Especificar objetivos a corto plazo**, elija durante cuánto tiempo quiere retener los puntos de recuperación guardados en el disco y cuándo guardar las copias de seguridad incrementales. Seleccione **Next** (Siguiente).

    > [!IMPORTANT]
    > **No** debería conservar datos de recuperación operativa (copia de seguridad) en discos conectados directamente a Azure Backup Server durante más de cinco días.
    >

    ![Especificar objetivos a corto plazo](./media/backup-mabs-files-applications-azure-stack/7-select-short-term-goals.png)

    En lugar de seleccionar un intervalo para las copias de seguridad incrementales, seleccione **Solo antes de un punto de recuperación** para ejecutar una copia de seguridad completa rápida justo antes de cada punto de recuperación programado. Si va a proteger las cargas de trabajo de la aplicación, Azure Backup Server crea puntos de recuperación según la programación de la frecuencia de sincronización (siempre que la aplicación admita las copias de seguridad incrementales). Si la aplicación no admite las copia de seguridad incrementales, Azure Backup Server ejecuta una copia de seguridad completa rápida.

    En **File recovery points** (Puntos de recuperación de archivo) especifique cuándo crear puntos de recuperación. Seleccione **Modificar** para establecer las horas y los días de la semana en que se crearán los puntos de recuperación.

6. En la pantalla **Revisar la asignación de disco**, revise el espacio en disco del grupo de almacenamiento asignado para el grupo de protección.

    **Total Data size** (Tamaño total de datos) es el tamaño de los datos cuya copia de seguridad quiere realizar y **Disk space to be provisioned** on Azure Backup Server (Espacio en disco que se aprovisionará en Azure Backup Server) es el espacio recomendado para el grupo de protección. Azure Backup Server elige el volumen ideal para la copia de seguridad en función de la configuración. Pero puede editar las opciones de volumen de la copia de seguridad en los detalles de asignación del disco. En el caso de las cargas de trabajo, seleccione el almacenamiento de su preferencia en el menú desplegable. Las modificaciones cambian los valores de Almacenamiento total y Almacenamiento libre en el panel Almacenamiento en disco disponible. El espacio insuficiente es la cantidad de almacenamiento que Azure Backup Server sugiere agregar al volumen para continuar en el futuro las copias de seguridad sin problemas.

7. En **Elegir método de creación de réplica**, seleccione cómo quiere controlar la replicación inicial completa de los datos. Si decide replicar a través de la red, Azure recomienda que elija una hora de poco tráfico. Para grandes cantidades de datos o condiciones de red no del todo óptimas, considere la posibilidad de replicar los datos con medios extraíbles.

8. En **Elegir las opciones de la comprobación de coherencia**, seleccione cómo quiere automatizar las comprobaciones de coherencia. Habilite las comprobaciones de coherencia para que se ejecuten solo cuando la replicación de datos sea incoherente o de acuerdo con una programación. Si no quiere configurar la comprobación de coherencia automática, ejecute una comprobación manual en cualquier momento, para lo que debe hacer lo siguiente:
    * En el área **Protección** de la consola de Azure Backup Server, haga clic con el botón derecho en el grupo de protección y seleccione **Realizar comprobación de coherencia**.

9. Si eligió realizar una copia de seguridad en Azure, en la página **Especificar datos de protección en línea**, asegúrese de que estén seleccionadas las cargas de trabajo de las que quiere hacer copias de seguridad en Azure.

10. En **Especificar la programación de copia de seguridad en línea**, especifique cuándo se deberían realizar las copias de seguridad incrementales en Azure.

    Puede programar que las copias de seguridad se ejecuten cada día, semana, mes y año y seleccionar la hora y la fecha en la que se deben ejecutar. Pueden realizarse copias de seguridad hasta dos veces al día. Cada vez que se ejecute un trabajo de copia de seguridad, se crea un punto de recuperación en Azure a partir de la copia de los datos de copia de seguridad almacenados en el disco de Azure Backup Server.

11. En **Especificar la directiva de retención en línea**, especifique cómo se retienen en Azure los puntos de recuperación creados a partir de copias de seguridad diarias, semanales, mensuales y anuales.

12. En **Elegir replicación en línea**, especifique cómo se realiza la replicación inicial total de los datos.

13. En **Resumen**, revise la configuración. Cuando selecciona **Crear grupo**, se realiza la replicación inicial de los datos. Cuando finalice la replicación de los datos, en la página **Estado**, el estado del grupo de protección se mostrará como **OK** (Correcto). El trabajo de copia de seguridad inicial se realiza según la configuración del grupo de protección.

## <a name="recover-file-data"></a>Recuperación de los datos de archivo

Use la consola de Azure Backup Server para recuperar los datos en la máquina virtual.

1. En la barra de navegación de la consola de Azure Backup Server, seleccione **Recuperación** y examine para encontrar los datos que quiere recuperar. Seleccione los datos en el panel de resultados.

2. En el calendario que aparece en la sección de puntos de recuperación, las fechas que aparecen en negrita indican los puntos de recuperación que están disponibles. Seleccione la fecha que desea recuperar.

3. En el panel **Elemento recuperable**, seleccione el elemento que quiere recuperar.

4. En el panel **Acciones**, seleccione **Recuperar** para abrir el Asistente para recuperación.

5. Puede recuperar los datos como se indica a continuación:

    * **Recover to the original location** (Recuperar en la ubicación original): si el equipo cliente está conectado a través de VPN, esta opción no funciona. En su lugar, use una ubicación alternativa y luego copie los datos desde esa ubicación.
    * **Recover to an alternate location** (Recuperar en una ubicación alternativa)

6. Especifique las opciones de recuperación:

    * En **Existing version recovery behavior** (Comportamiento de recuperación de la versión existente), seleccione **Create copy** (Crear copia), **Skip** (Omitir) o **Overwrite** (Sobrescribir). La opción Overwrite solo está disponible cuando la recuperación se realiza en la ubicación original.
    * En **Restore security** (Restaurar seguridad), elija **Apply settings of the destination computer** (Aplicar la configuración del equipo de destino) o **Apply the security settings of the recovery point version** (Aplicar la configuración de seguridad de la versión del punto de recuperación).
    * En **Límite de uso del ancho de banda de red**, seleccione **Modificar** para habilitar el límite de uso del ancho de banda de red.
    * **Notificación** Seleccione **Send an e-mail when the recovery completes** (Enviar un correo electrónico cuando se complete la recuperación) y especifique los destinatarios que recibirán la notificación. Separe las direcciones de correo electrónico con comas.
    * Después de realizar las selecciones, haga clic en **Siguiente**.

7. Revise la configuración de recuperación y seleccione **Recuperar**.

    >[!Note]
    >Mientras el trabajo de recuperación está en curso, se cancelan todos los trabajo de sincronización correspondientes a los elementos de recuperación seleccionados.

Si usa Modern Backup Storage (MBS), no se admite la recuperación del usuario final (EUR) del servidor de archivos. EUR del servidor de archivos depende del Servicio de instantáneas de volumen (VSS) que Modern Backup Storage no usa. Si se habilitó EUR, use estos pasos para recuperar los datos:

1. Vaya a los archivos protegidos, haga clic con el botón derecho en el nombre del archivo y seleccione **Propiedades**.

2. En el menú **Propiedades**, seleccione **Versiones anteriores** y elija la versión que quiere recuperar.

## <a name="view-azure-backup-server-with-a-vault"></a>Visualización de Azure Backup Server con un almacén

Para ver entidades de Azure Backup Server en Azure Portal, se pueden seguir estos pasos:

1. Abrir el almacén de Recovery Services.
2. Seleccionar Infraestructura de Backup.
3. Ver los servidores de administración de copias de seguridad.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo usar Azure Backup Server para proteger otras cargas de trabajo, consulte uno de los siguientes artículos:

* [Hacer una copia de seguridad de la granja de SharePoint](./backup-mabs-sharepoint-azure-stack.md)
* [Hacer una copia de seguridad de SQL Server](./backup-mabs-sql-azure-stack.md)
