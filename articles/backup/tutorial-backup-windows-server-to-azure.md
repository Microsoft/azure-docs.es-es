---
title: 'Tutorial: Copias de seguridad de Windows Server en Azure'
description: En este tutorial se detalla la copia de seguridad de servidores Windows Server locales en un almacén de Recovery Services.
ms.topic: tutorial
ms.date: 08/22/2018
ms.custom: mvc
ms.openlocfilehash: d2990b5950cf8812367c3a59c6cace39e4085e2a
ms.sourcegitcommit: 64ad2c8effa70506591b88abaa8836d64621e166
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88261912"
---
# <a name="back-up-windows-server-to-azure"></a>Hacer copias de seguridad de Windows Server en Azure

Puede utilizar Azure Backup para proteger su servidor Windows Server de daños, ataques y desastres. Azure Backup proporciona una herramienta ligera que se conoce como el agente de Microsoft Azure Recovery Services (MARS). El agente de MARS se instala en Windows Server para proteger los archivos y las carpetas, así como la información de configuración del servidor a través de Estado del sistema de Windows Server. En este tutorial se explica cómo se puede usar el agente de MARS para realizar la copia de seguridad Windows Server en Azure. En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Descargar y configurar el agente de MARS
> * Configurar las horas de copia de seguridad y la programación de retención de las copias de seguridad del servidor
> * Realizar una copia de seguridad a petición.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en <https://portal.azure.com>.

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

Para poder realizar una copia de seguridad de Windows Server, debe crear una ubicación para almacenar las copias de seguridad o los puntos de restauración. Un [almacén de Recovery Services](backup-azure-recovery-services-vault-overview.md) es un contenedor de Azure que almacena las copias de seguridad de Windows Server. Realice los pasos siguientes para crear un almacén de Recovery Services en Azure Portal.

1. En el menú izquierdo, seleccione **Todos los servicios** y, en la lista de servicios, escriba **Recovery Services**. Seleccione **Almacenes de Recovery Services**.

   ![Abrir el almacén de Recovery Services](./media/tutorial-backup-windows-server-to-azure/full-browser-open-rs-vault_2.png)

2. En el menú **Almacenes de Recovery Services**, seleccione **Agregar**.

   ![Proporcionar la información del almacén](./media/tutorial-backup-windows-server-to-azure/provide-vault-detail-2.png)

3. En el menú **Almacén de Recovery Services**,

    * escriba *myRecoveryServicesVault* en **Nombre**.
    * El id. de suscripción actual aparecerá en **Suscripción**.
    * En **Grupo de recursos**, seleccione **Usar existente** y elija *myResourceGroup*. Si *myResourceGroup* no existe, seleccione **Crear nuevo** y escriba *myResourceGroup*.
    * En el menú desplegable **Ubicación**, elija *Oeste de Europa*.
    * Seleccione **Crear** para crear el almacén de Recovery Services.

Una vez creado el almacén, aparece en la lista de almacenes de Recovery Services.

## <a name="download-recovery-services-agent"></a>Descarga del agente de Recovery Services

El agente de Microsoft Azure Recovery Services (MARS) crea una asociación entre Windows Server y el almacén de Recovery Services. En el procedimiento siguiente se explica cómo descargar el agente en el servidor.

1. En la lista Almacenes de Recovery Services, seleccione **myRecoveryServicesVault** para abrir su panel.

   ![Selección de almacén para abrir el panel](./media/tutorial-backup-windows-server-to-azure/open-vault-from-list.png)

2. En el menú del panel del almacén, seleccione **Backup**.

3. En el menú **Objetivo de Backup**:

   * en **¿Dónde se ejecuta su carga de trabajo?** , seleccione **Local**.
   * En el menú **¿De qué desea hacer una copia de seguridad?** , seleccione **Archivos y carpetas** y **Estado del sistema**.

   ![Menú Objetivo de Backup](./media/tutorial-backup-windows-server-to-azure/backup-goal.png)

4. Seleccione **Preparar infraestructura** para abrir el menú **Preparar infraestructura**.

5. En el menú **Preparar infraestructura**, seleccione **Descargar agente para Windows Server o cliente de Windows** si desea descargar el archivo *MARSAgentInstaller.exe*.

    ![Descargar agente para Windows Server o cliente de Windows](./media/tutorial-backup-windows-server-to-azure/prepare-infrastructure.png)

    El instalador se abre en un explorador diferente y descarga **MARSAgentInstaller.exe**.

6. Antes de ejecutar el archivo descargado, en el menú Preparar infraestructura, seleccione **Descargar** y guarde el archivo **Credenciales de almacén**. Este archivo es necesario para conectar el agente de MARS con el almacén de Recovery Services.

    ![Descarga de las credenciales de almacén](./media/tutorial-backup-windows-server-to-azure/download-vault-credentials.png)

## <a name="install-and-register-the-agent"></a>Instalación y registro del agente

1. Busque y haga doble clic en el archivo **MARSagentinstaller.exe** descargado.
2. Se muestra el **Asistente para la instalación de Agente de Microsoft Azure Recovery Services**. A medida que avance por el asistente, proporcione la siguiente información cuando se le solicite y seleccione **Registrar**.
   * Ubicación de la carpeta de instalación y de memoria caché.
   * Información del servidor proxy si usa un servidor proxy para conectarse a Internet.
   * Sus detalles de nombre de usuario y contraseña si usa un servidor proxy autenticado.

     ![Asistente para la instalación del Agente de Microsoft Azure Recovery Services](./media/tutorial-backup-windows-server-to-azure/mars-installer.png)

3. Al final del asistente, seleccione **Proceder al registro** y proporcione el archivo **Credenciales de almacén** que descargó en el procedimiento anterior.

4. Cuando se le solicite, proporcione una frase de contraseña de cifrado para cifrar las copias de seguridad de Windows Server. Guarde la frase de contraseña en una ubicación segura, ya que Microsoft no puede recuperarla si se pierde.

5. Seleccione **Finalizar**.

## <a name="configure-backup-and-retention"></a>Configuración de copia de seguridad y retención

Use el agente de Microsoft Azure Recovery Services para programar cuándo se realizarán copias de seguridad de Azure en Windows Server. Ejecute los siguientes pasos en el servidor donde descargó el agente.

1. Abra el agente de Microsoft Azure Recovery Services. Para encontrarlo, busque **Microsoft Azure Backup**en la máquina.

2. En la consola de agente de Recovery Services, seleccione **Programar copia de seguridad** en el **panel de Acciones**.

    ![Programación de una copia de seguridad](./media/tutorial-backup-windows-server-to-azure/mars-schedule-backup.png)

3. Seleccione **Siguiente** para ir a la página **Seleccionar elementos de los que realizar copia de seguridad**.

4. Seleccione **Agregar elementos** y, desde el cuadro de diálogo que aparece, seleccione **Estado del sistema** y los archivos o las carpetas de los que quiera realizar una copia de seguridad. Después, seleccione **Aceptar**.

5. Seleccione **Next** (Siguiente).

6. En la página **Especificar la programación de copia de seguridad (estado del sistema)** , especifique las horas del día o la semana en que las copias de seguridad deban desencadenarse para el estado del sistema y seleccione **Siguiente**.

7. En la página **Seleccionar la directiva de retención (estado del sistema)** , seleccione la directiva de retención para la copia de seguridad del estado del sistema y seleccione **Siguiente**.

8. De forma similar, seleccione la programación de copia de seguridad y la directiva de retención de los archivos y carpetas seleccionados.

9. En la página **Elija el tipo de copia de seguridad inicial**, seleccione **Automáticamente a través de la red** y, después, **Siguiente**.

10. En la página **Confirmación**, revise la información y seleccione **Finalizar**.

11. Cuando el asistente termine de crear la programación de copia de seguridad, seleccione **Cerrar**.

## <a name="perform-an-on-demand-backup"></a>Realización de una copia de seguridad a petición

Habrá establecido la programación cuando se ejecuten los trabajos de copia de seguridad. Sin embargo, no ha realizado la copia de seguridad del servidor. Es un procedimiento recomendado de recuperación ante desastres para ejecutar una copia de seguridad a petición destinada a garantizar la resistencia de datos del servidor.

1. En la consola del agente de Microsoft Azure Recovery Services, seleccione **Hacer copia de seguridad ahora**.

    ![Hacer copia de seguridad ahora](./media/tutorial-backup-windows-server-to-azure/backup-now.png)

2. En el **Asistente para iniciar copia de seguridad**, seleccione uno de los **archivos y carpetas** o el **estado del sistema** del que quiera realizar una copia de seguridad, y seleccione **Siguiente**.
3. En la página **Confirmación**, revise la configuración que usa el Asistente de **Hacer Copia Ahora** para crear la copia de seguridad del servidor. A continuación, seleccione **Copia de seguridad**.
4. Seleccione **Cerrar** para cerrar el asistente. Si lo hace antes de que finalice la copia de seguridad, el asistente se sigue ejecutando en segundo plano.
5. Una vez completada la copia de seguridad inicial, el estado **Trabajo completado** se refleja en el panel **Trabajos** de la consola del agente de MARS.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial se usa Azure Portal para las siguientes acciones:

> [!div class="checklist"]
>
> * Creación de un almacén de Recovery Services
> * Descarga del agente de Microsoft Azure Recovery Services
> * Instalación del agente
> * Configuración de la copia de seguridad para Windows Server
> * Realización de una copia de seguridad a petición

Continúe en el tutorial siguiente para recuperar archivos de Azure en Windows Server

> [!div class="nextstepaction"]
> [Recuperación de archivos de Azure en Windows Server](./tutorial-backup-restore-files-windows-server.md)
