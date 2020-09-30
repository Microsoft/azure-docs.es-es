---
title: Habilitar copia de seguridad al crear una máquina virtual de Azure
description: Describe cómo habilitar la copia de seguridad al crear una VM de Azure con Azure Backup.
ms.topic: conceptual
ms.date: 06/13/2019
ms.openlocfilehash: ad81300545686d61f42cdd8684e502c937b4fd43
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89377342"
---
# <a name="enable-backup-when-you-create-an-azure-vm"></a>Habilitar copia de seguridad al crear una máquina virtual de Azure

Use el servicio Azure Backup para hacer copias de seguridad de máquinas virtuales (VM) de Azure. Las copias de seguridad de las VM se crean según una programación especificada en una directiva de copia de seguridad, y los puntos de recuperación se crean a partir de las copias de seguridad. Los datos puntos de recuperación se guardan en almacenes de Recovery Services.

En este artículo se detalla cómo habilitar la copia de seguridad al crear una máquina virtual (VM) en Azure Portal.  

## <a name="before-you-start"></a>Antes de comenzar

- [Compruebe](backup-support-matrix-iaas.md#supported-backup-actions) qué sistemas operativos se admiten si habilita la copia de seguridad al crear una VM.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Si aún no ha iniciado sesión en su cuenta, hágalo en [Azure Portal](https://portal.azure.com).

## <a name="create-a-vm-with-backup-configured"></a>Crear una VM con la copia de seguridad configurada

1. En Azure Portal, haga clic en **Crear un recurso**.

2. En Azure Marketplace, seleccione **Proceso** y, después, elija una imagen de máquina virtual.

3. Configure la VM de acuerdo con las instrucciones para [Windows](../virtual-machines/windows/quick-create-portal.md) o [Linux](../virtual-machines/linux/quick-create-portal.md).

4. En la pestaña **Administración**, en **Habilitar copia de seguridad**, seleccione **Activado**.
5. Las copias de seguridad de Azure Backup se hacen en un almacén de Recovery Services. Seleccione **Crear nuevo** si no tiene un almacén existente.
6. Acepte el nombre del almacén sugerido o especifique el suyo.
7. Especifique o cree un grupo de recursos en el que se ubicará el almacén. El almacén del grupo de recursos puede ser diferente del grupo de recursos de la VM.

    ![Habilitar la copia de seguridad para una VM](./media/backup-during-vm-creation/enable-backup.png)

8. Acepte la directiva de copia de seguridad predeterminada o modifique la configuración.
    - Una directiva de copia de seguridad especifica la frecuencia con que se toman instantáneas de copia de seguridad de la VM, y el tiempo que quiere conservar esas copias de seguridad.
    - La directiva predeterminada hace una copia de seguridad de la VM una vez al día.
    - Puede personalizar su propia directiva de copia de seguridad para que una VM de Azure haga copias de seguridad diarias o semanales.
    - [Obtenga más información](backup-azure-vms-introduction.md#backup-and-restore-considerations) sobre las consideraciones de copia de seguridad de VM de Azure.
    - [Obtenga más información](backup-instant-restore-capability.md) sobre la funcionalidad de restauración instantánea.

      ![Directiva de copia de seguridad predeterminada](./media/backup-during-vm-creation/daily-policy.png)

>[!NOTE]
>[SSE y PMK son los métodos de cifrado predeterminados](backup-encryption.md) para las máquinas virtuales de Azure. Azure Backup admite la copia de seguridad y la restauración de estas máquinas virtuales de Azure.

## <a name="azure-backup-resource-group-for-virtual-machines"></a>Grupo de recursos de Azure Backup para máquinas virtuales

El servicio Backup crea un grupo de recursos diferente al de la máquina virtual para guardar la colección de puntos de restauración. La colección de puntos de restauración aloja los puntos de recuperación instantánea de las máquinas virtuales administradas. El formato predeterminado de los nombres del grupo de recursos creado por el servicio Backup es `AzureBackupRG_<Geo>_<number>`. Por ejemplo: *AzureBackupRG_northeurope_1*. Ahora puede personalizar el nombre del grupo de recursos creado por Azure Backup.

Puntos a tener en cuenta:

1. Puede usar el nombre predeterminado del grupo de recursos o editarlo según los requisitos de la empresa.
2. El patrón del nombre del grupo de recursos se proporciona como entrada durante la creación de la directiva de copia de seguridad de la máquina virtual. El nombre del grupo de recursos deberá tener el formato siguiente: `<alpha-numeric string>* n <alpha-numeric string>`. "n" se reemplaza por un entero (empezando por el 1) y se usa para el escalado horizontal si el primer grupo de recursos está lleno. A día de hoy, los grupos de recursos pueden tener hasta 600 RPC como máximo.
              ![Elección del nombre al crear la directiva](./media/backup-during-vm-creation/create-policy.png)
3. El patrón debe seguir las reglas de nomenclatura de los grupos de recursos siguientes y la longitud total no debe superar la máxima permitida para el nombre del grupo de recursos.
    1. Los nombres de los grupos de recursos solo permiten caracteres alfanuméricos, puntos, guiones bajos, guiones y paréntesis. No puede terminar en punto.
    2. Los nombres de los grupos de recursos pueden contener hasta 74 caracteres, nombre y sufijo incluidos.
4. La primera cadena `<alpha-numeric-string>` es obligatoria, mientras que la segunda después de "n" es opcional. Esto solo se aplica si se asigna un nombre personalizado. Si no escribe nada en ninguno de los cuadros de texto, se usa el nombre predeterminado.
5. Si es necesario, puede editar el nombre del grupo de recursos al modificar la directiva. Si se cambia el patrón de nombre, se crearán puntos de restauración en el nuevo grupo de recursos. Sin embargo, los antiguos puntos de restauración seguirán residiendo en el grupo de recursos anterior y no se moverán, ya que la colección de puntos de restauración no admite el traslado de recursos. Finalmente, los puntos de restauración recogerán los elementos no utilizados al expirar.
![Cambio del nombre al modificar la directiva](./media/backup-during-vm-creation/modify-policy.png)
6. Es conveniente que no se bloquee el grupo de recursos que se ha creado para que lo utilice el servicio Backup.

Para configurar el grupo de recursos de Azure Backup para Virtual Machines mediante PowerShell, consulte el artículo para la [creación de un grupo de recursos de Azure Backup durante la retención de instantáneas](backup-azure-vms-automation.md#creating-azure-backup-resource-group-during-snapshot-retention).

## <a name="start-a-backup-after-creating-the-vm"></a>Iniciar una copia de seguridad después de crear la VM

La copia de seguridad de la VM se ejecuta según la directiva de copia de seguridad. Sin embargo, se recomienda ejecutar una copia de seguridad inicial.

Una vez creada la VM, haga lo siguiente:

1. En las propiedades de la máquina virtual, seleccione **Copia de seguridad**. El estado de la máquina virtual es Copia de seguridad inicial pendiente hasta que se ejecuta la copia de seguridad inicial.
2. Seleccione **Hacer copia ahora** para ejecutar una copia de seguridad a petición.

    ![Ejecución de una copia de seguridad a petición](./media/backup-during-vm-creation/run-backup.png)

## <a name="use-a-resource-manager-template-to-deploy-a-protected-vm"></a>Uso de una plantilla de Resource Manager para implementar una VM protegida

En los pasos anteriores se explica cómo usar Azure Portal para crear una máquina virtual y protegerla en un almacén de Recovery Services. Para implementar una o varias VM rápidamente y protegerlas en un almacén de Azure Recovery Services, consulte la plantilla [Deploy a Windows VM and enable backup](https://azure.microsoft.com/resources/templates/101-recovery-services-create-vm-and-configure-backup/) (Implementar una VM Windows y habilitar la copia de seguridad).

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha protegido su VM, obtenga información sobre cómo administrarla y restaurarla.

- [Administración y supervisión de máquinas virtuales](backup-azure-manage-vms.md)
- [Restauración de máquinas virtuales](backup-azure-arm-restore-vms.md)

Si tiene algún problema, [revise](backup-azure-vms-troubleshoot.md) la guía de solución de problemas.
