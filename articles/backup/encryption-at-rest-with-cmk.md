---
title: Cifrado de datos de copia de seguridad mediante claves administradas por el cliente
description: Obtenga información sobre el modo en que Azure Backup le permite cifrar los datos de copia de seguridad mediante claves administradas por el cliente.
ms.topic: conceptual
ms.date: 07/08/2020
ms.openlocfilehash: 5c0bddc6cdb8ec150a031541ced1abf1ebfb6f0f
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378294"
---
# <a name="encryption-of-backup-data-using-customer-managed-keys"></a>Cifrado de datos de copia de seguridad mediante claves administradas por el cliente

Azure Backup le permite cifrar los datos de copia de seguridad mediante claves administradas por el cliente (CMK) en lugar de usar claves administradas por la plataforma, que es la opción habilitada de forma predeterminada. Las claves que se usan para cifrar los datos de copia de seguridad deben almacenarse en [Azure Key Vault](../key-vault/index.yml).

La clave de cifrado utilizada para cifrar las copias de seguridad puede ser diferente de la que se usa para el origen. Los datos se protegen mediante una clave de cifrado de datos (DEK) basada en AES 256 que, a su vez, está protegida con las claves del usuario (KEK). Esto le proporciona un control total sobre los datos y las claves. Para permitir el cifrado, se requiere que el almacén de Recovery Services tenga acceso a la clave de cifrado en Azure Key Vault. Puede cambiar la clave como y cuando sea necesario.

En este artículo se tratan los temas siguientes:

- Creación de un almacén de Recovery Services
- Configuración del almacén de Recovery Services para cifrar los datos de copia de seguridad mediante claves administradas por el cliente
- Copia de seguridad en almacenes cifrados con claves administradas por el cliente
- Restauración de datos a partir de copias de seguridad

## <a name="before-you-start"></a>Antes de comenzar

- Esta característica le permite cifrar **solo nuevos almacenes de Recovery Services**. No se admiten los almacenes que contienen elementos existentes registrados o que se intentaron registrar en estos.

- Una vez que se ha habilitado para un almacén de Recovery Services, el cifrado mediante claves administradas por el cliente no se puede revertir para usar claves administradas por la plataforma (valor predeterminado). Puede cambiar las claves de cifrado de acuerdo con sus requisitos.

- Actualmente, esta característica **no admite la copia de seguridad mediante el agente de MARS**, y es posible que no pueda usar un almacén cifrado por CMK para este. El agente de MARS usa un cifrado basado en una frase de contraseña del usuario. Esta característica tampoco admite la copia de seguridad de máquinas virtuales clásicas.

- Esta característica no está relacionada con [Azure Disk Encryption](../security/fundamentals/azure-disk-encryption-vms-vmss.md), que usa el cifrado basado en invitado de los discos de una máquina virtual con BitLocker (para Windows) y DM-Crypt (para Linux).

- El almacén de Recovery Services solo se puede cifrar con las claves almacenadas en un almacén de Azure Key Vault ubicado en la **misma región**. Además, las claves deben ser solo **claves de RSA 2048** y deben estar en estado **habilitado**.

- Actualmente no se admite la migración del almacén de Recovery Services cifrado de CMK entre grupos de recursos y suscripciones.

- Actualmente, esta característica solo se puede configurar desde Azure Portal.

Si no ha creado y configurado un almacén de Recovery Services, [lea aquí cómo hacerlo](backup-create-rs-vault.md).

## <a name="configuring-a-vault-to-encrypt-using-customer-managed-keys"></a>Configuración de un almacén para cifrar mediante claves administradas por el cliente

Esta acción implica los pasos siguientes:

1. Habilitación de la identidad administrada para el almacén de Recovery Services

1. Asignación de permisos al almacén para tener acceso a la clave de cifrado en Azure Key Vault

1. Habilitación de la eliminación temporal y la protección de purga para Azure Key Vault

1. Asignación de la clave de cifrado al almacén de Recovery Services

Es necesario que todos estos pasos se sigan en el orden mencionado anteriormente para lograr los resultados deseados. Cada paso se describe en detalle a continuación.

### <a name="enable-managed-identity-for-your-recovery-services-vault"></a>Habilitación de la identidad administrada para el almacén de Recovery Services

Azure Backup usa la identidad administrada asignada por el sistema para autenticar el almacén de Recovery Services para tener acceso a las claves de cifrado almacenadas en Azure Key Vault. Para habilitar la identidad administrada para el almacén de Recovery Services, siga los pasos que se mencionan a continuación.

>[!NOTE]
>Una vez habilitada, la identidad administrada **no** debe deshabilitarse (ni siquiera temporalmente). Deshabilitar la identidad administrada puede provocar un comportamiento incoherente.

1. Vaya al almacén de Recovery Services -> **Identidad**.

    ![Configuración de identidad](./media/encryption-at-rest-with-cmk/managed-identity.png)

1. Cambie el **Estado** a **Activado** y seleccione **Guardar**.

1. Se genera un identificador de objeto, que es la identidad administrada del almacén.

### <a name="assign-permissions-to-the-recovery-services-vault-to-access-the-encryption-key-in-the-azure-key-vault"></a>Asignación de permisos al almacén de Recovery Services para tener acceso a la clave de cifrado en Azure Key Vault

Ahora debe permitir que el almacén de Recovery Services tenga acceso al almacén de Azure Key Vault que contiene la clave de cifrado. Para ello, se permite que la identidad administrada del almacén de Recovery Services tenga acceso al almacén de Key Vault.

1. Vaya a Azure Key Vault > **Directivas de acceso**. Continúe a **+Add Access Policies** (+Agregar directivas de acceso).

    ![Agregar directivas de acceso](./media/encryption-at-rest-with-cmk/access-policies.png)

1. En **Permisos de clave**, seleccione las operaciones **Obtener**, **Enumerar**, **Encapsular clave** y **Desencapsular clave**. Esto especifica las acciones de la clave que se permitirán.

    ![Asignar permisos de las claves](./media/encryption-at-rest-with-cmk/key-permissions.png)

1. Vaya a **Seleccionar la entidad de seguridad** y busque el almacén en el cuadro de búsqueda con su nombre o identidad administrada. Una vez que se muestra, seleccione el almacén y elija **Seleccionar** en la parte inferior del panel.

    ![Selección de la entidad de seguridad](./media/encryption-at-rest-with-cmk/select-principal.png)

1. A continuación, seleccione **Agregar** para agregar la nueva directiva de acceso.

1. Seleccione **Guardar** para guardar los cambios realizados en la directiva de acceso de Azure Key Vault.

### <a name="enable-soft-delete-and-purge-protection-on-the-azure-key-vault"></a>Habilitación de la eliminación temporal y la protección de purga para Azure Key Vault

Debe **habilitar la eliminación temporal y la protección de purga** en el almacén de Azure Key Vault que almacena la clave de cifrado. Puede hacerlo desde la interfaz de usuario de Azure Key Vault, como se muestra a continuación. (También tiene la posibilidad de configurar estas propiedades al crear el almacén de Key Vault). Obtenga más información sobre estas propiedades de Key Vault [aquí](../key-vault/general/soft-delete-overview.md).

![Habilitación de la eliminación temporal y la protección de purga](./media/encryption-at-rest-with-cmk/soft-delete-purge-protection.png)

También puede habilitar la eliminación temporal y la protección de purga a través de PowerShell mediante los pasos siguientes:

1. Inicie sesión en la cuenta de Azure.

    ```azurepowershell
    Login-AzAccount
    ```

1. Seleccione la suscripción que contiene el almacén.

    ```azurepowershell
    Set-AzContext -SubscriptionId SubscriptionId
    ```

1. Habilitación de la eliminación temporal

    ```azurepowershell
    ($resource = Get-AzResource -ResourceId (Get-AzKeyVault -VaultName "AzureKeyVaultName").ResourceId).Properties | Add-Member -MemberType "NoteProperty" -Name "enableSoftDelete" -Value "true"
    ```

    ```azurepowershell
    Set-AzResource -resourceid $resource.ResourceId -Properties $resource.Properties
    ```

1. Habilitación de la protección de purga

    ```azurepowershell
    ($resource = Get-AzResource -ResourceId (Get-AzKeyVault -VaultName "AzureKeyVaultName").ResourceId).Properties | Add-Member -MemberType "NoteProperty" -Name "enablePurgeProtection" -Value "true"
    ```

    ```azurepowershell
    Set-AzResource -resourceid $resource.ResourceId -Properties $resource.Properties
    ```

### <a name="assign-encryption-key-to-the-rs-vault"></a>Asignación de la clave de cifrado al almacén de RS

>[!NOTE]
> Compruebe los siguientes puntos antes de continuar:
>
> - Todos los pasos mencionados con anterioridad se han completado correctamente:
>   - La identidad administrada del almacén de Recovery Services se ha habilitado y se le han asignado los permisos necesarios.
>   - El almacén de Azure Key Vault tiene habilitadas la eliminación temporal y la protección de purga.
> - El almacén de Recovery Services para el que desea habilitar el cifrado de CMK **no** tiene elementos protegidos ni registrados en él.

Una vez comprobado lo anterior, continúe con la selección de la clave de cifrado para el almacén.

Para asignar la clave:

1. Vaya al almacén de Recovery Services -> **Propiedades**.

    ![Configuración de cifrado](./media/encryption-at-rest-with-cmk/encryption-settings.png)

1. Seleccione **Actualizar** en **Configuración de cifrado**.

1. En el panel Configuración de cifrado, seleccione **Usar su propia clave** y continúe especificando la clave mediante una de las siguientes formas. **Asegúrese de que la clave que desea usar es una clave RSA 2048, que se encuentra en un estado habilitado.**

    1. Escriba el **URI de clave** con el que quiere cifrar los datos de este almacén de Recovery Services. También debe especificar la suscripción en la que está presente el almacén de Azure Key Vault (que contiene esta clave). Este URI de clave se puede obtener a partir de la clave correspondiente en el almacén de Azure Key Vault. Asegúrese de que el URI de la clave se copia correctamente. Se recomienda usar el botón **Copiar en el Portapapeles** proporcionado con el identificador de clave.

        ![Especificación del URI de la clave](./media/encryption-at-rest-with-cmk/key-uri.png)

    1. Busque y seleccione la clave en el almacén de Key Vault en el panel del selector de claves.

        ![Selección de clave del almacén de claves](./media/encryption-at-rest-with-cmk/key-vault.png)

1. Seleccione **Guardar**.

1. **Seguimiento del progreso de la actualización de la clave de cifrado:** puede realizar un seguimiento del progreso de la asignación de claves mediante el **Registro de actividad** del almacén de Recovery Services. El estado debería cambiar pronto a **Correcto**. El almacén cifrará ahora todos los datos con la clave especificada como KEK.

    ![Seguimiento del progreso con el registro de actividad](./media/encryption-at-rest-with-cmk/activity-log.png)

    ![Estado correcto](./media/encryption-at-rest-with-cmk/status-succeeded.png)

>[!NOTE]
> El proceso para actualizar o cambiar la clave de cifrado sigue siendo el mismo. Si desea actualizar y usar una clave de otro almacén de Key Vault (diferente del que se está usando actualmente), asegúrese de lo siguiente:
>
> - El almacén de Key Vault se encuentra en la misma región que el almacén de Recovery Services.
>
> - El almacén de Key Vault tiene habilitadas la eliminación temporal y la protección de purga.
>
> - El almacén de Recovery Services tiene los permisos necesarios para obtener acceso al almacén de Key Vault.

## <a name="backing-up-to-a-vault-encrypted-with-customer-managed-keys"></a>Copia de seguridad en un almacén cifrado con claves administradas por el cliente

Antes de continuar con la configuración de la protección, recomendamos encarecidamente verificar que se cumplen los puntos de la siguiente lista de comprobación. Es importante porque una vez que se haya configurado un elemento para que se haga una copia de seguridad de él (o se haya intentado la configuración) en un almacén cifrado que no es CMK, no se podrá habilitar el cifrado mediante claves administradas por el cliente y se seguirán usando claves administradas por la plataforma.

>[!IMPORTANT]
> Antes de continuar con la configuración de la protección, debe haber completado **correctamente** los siguientes pasos:
>
>1. Se habilitó su suscripción para usar las claves administradas por el cliente para el almacén de Backup.
>1. Se creó el almacén de Backup.
>1. Se habilitó la identidad administrada asignada por el sistema del almacén de Backup.
>1. Se asignaron permisos al almacén de Backup para tener acceso a las claves de cifrado desde el almacén de Key Vault.
>1. Se habilitó la eliminación temporal y la protección de purga para el almacén de Key Vault.
>1. Se asignó una clave de cifrado válida para el almacén de Backup.
>
>Si se han confirmado todos los pasos anteriores, siga con la configuración de la copia de seguridad.

El proceso para configurar y realizar copias de seguridad en un almacén de Recovery Services cifrado con claves administradas por el cliente es igual que para un almacén que usa claves administradas por la plataforma, **sin cambios en la experiencia**. Esto se aplica igualmente a la [copia de seguridad de máquinas virtuales de Azure](./quick-backup-vm-portal.md), así como a la copia de seguridad de cargas de trabajo que se ejecutan dentro de una máquina virtual (por ejemplo, bases de datos de [SAP HANA](./tutorial-backup-sap-hana-db.md) o [SQL Server](./tutorial-sql-backup.md)).

## <a name="restoring-data-from-backup"></a>Restauración de datos a partir de una copia de seguridad

### <a name="vm-backup"></a>Copia de seguridad de máquinas virtuales

Los datos almacenados en el almacén de Recovery Services se pueden restaurar según los pasos descritos [aquí](./backup-azure-arm-restore-vms.md). Al restaurar desde un almacén de Recovery Services cifrado con claves administradas por el cliente, puede optar por cifrar los datos restaurados con un conjunto de cifrado de disco (DES).

#### <a name="restoring-vm--disk"></a>Restauración de máquina virtual/disco

1. Al recuperar el disco o la máquina virtual desde un punto de recuperación de "instantánea", los datos restaurados se cifrarán con el DES que se usó para cifrar los discos de la máquina virtual de origen.

1. Al restaurar el disco o la máquina virtual desde un punto de recuperación con el tipo de recuperación como "Almacén", puede elegir que los datos restaurados se cifren mediante un DES, que se especifica en el momento de la restauración. Como alternativa, puede optar por continuar con la restauración de los datos sin especificar un DES, en cuyo caso se cifrarán mediante claves administradas por Microsoft.

Puede cifrar el disco o la máquina virtual restaurados una vez completada la restauración, independientemente de la selección realizada al inicio del proceso.

![Puntos de restauración](./media/encryption-at-rest-with-cmk/restore-points.png)

#### <a name="select-a-disk-encryption-set-while-restoring-from-vault-recovery-point"></a>Selección de un conjunto de cifrado de discos durante la restauración desde el punto de recuperación del almacén

El conjunto de cifrado de discos se especifica en Configuración de cifrado en el panel de restauración, como se muestra a continuación:

1. En **Encrypt disk(s) using your key** (Cifrar discos con su clave), seleccione **Sí**.

1. En el menú desplegable, seleccione el DES que quiere usar para los discos restaurados. **Asegúrese de que tiene acceso al DES.**

>[!NOTE]
>No es posible elegir un DES durante la restauración si va a restaurar una máquina virtual que usa Azure Disk Encryption.

![Cifrado del disco con la clave](./media/encryption-at-rest-with-cmk/encrypt-disk-using-your-key.png)

#### <a name="restoring-files"></a>Restauración de archivos

Al realizar una restauración de archivos, los datos restaurados se cifrarán con la clave usada para cifrar la ubicación de destino.

### <a name="restoring-sap-hanasql-databases-in-azure-vms"></a>Restauración de bases de datos de SAP HANA/SQL en máquinas virtuales de Azure

Al restaurar desde una copia de seguridad de una base de datos de SAP HANA/SQL que se ejecuta en una máquina virtual de Azure, los datos restaurados se cifrarán con la clave de cifrado usada en la ubicación de almacenamiento de destino. Puede ser una clave administrada por el cliente o una clave administrada por la plataforma que se usa para cifrar los discos de la máquina virtual.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="can-i-encrypt-an-existing-backup-vault-with-customer-managed-keys"></a>¿Puedo cifrar un almacén de Backup existente con claves administradas por el cliente?

No, el cifrado de CMK solo se puede habilitar para los nuevos almacenes. Por lo tanto, el almacén no debe haber tenido ningún elemento protegido. De hecho, no se debe realizar ningún intento de proteger los elementos del almacén antes de habilitar el cifrado mediante claves administradas por el cliente.

### <a name="i-tried-to-protect-an-item-to-my-vault-but-it-failed-and-the-vault-still-doesnt-contain-any-items-protected-to-it-can-i-enable-cmk-encryption-for-this-vault"></a>Intenté proteger un elemento en el almacén, pero se produjo un error y el almacén todavía no contiene elementos protegidos. ¿Puedo habilitar el cifrado de CMK para este almacén?

No, no deben haberse producido intentos de proteger ningún elemento del almacén en el pasado.

### <a name="i-have-a-vault-thats-using-cmk-encryption-can-i-later-revert-to-encryption-using-platform-managed-keys-even-if-i-have-backup-items-protected-to-the-vault"></a>Tengo un almacén que usa el cifrado CMK. ¿Puedo revertir más adelante el cifrado mediante claves administradas por la plataforma aunque tenga elementos de copia de seguridad protegidos en el almacén?

No, una vez que haya habilitado el cifrado de CMK, no podrá revertir para usar claves administradas por la plataforma. Puede cambiar las claves usadas de acuerdo con sus requisitos.

### <a name="does-cmk-encryption-for-azure-backup-also-apply-to-azure-site-recovery"></a>¿El cifrado de CMK para Azure Backup se aplica también a Azure Site Recovery?

No, en este artículo solo se describe el cifrado de datos de Backup. Para Azure Site Recovery, debe establecer la propiedad por separado como disponible en el servicio.

### <a name="i-missed-one-of-the-steps-in-this-article-and-went-on-to-protect-my-data-source-can-i-still-use-cmk-encryption"></a>Me faltó uno de los pasos de este artículo y procedí a proteger el origen de datos. ¿Puedo seguir usando el cifrado de CMK?

No seguir las indicaciones del artículo y proceder con la protección de elementos puede provocar que el almacén no utilice el cifrado con claves administradas por el cliente. Por lo tanto, se recomienda que consulte esta [lista de comprobación](#backing-up-to-a-vault-encrypted-with-customer-managed-keys) antes de continuar con la protección de los elementos.

### <a name="does-using-cmk-encryption-add-to-the-cost-of-my-backups"></a>¿El uso de cifrado con CMK añade costos a mis copias de seguridad?

El uso del cifrado de CMK para Backup no conlleva ningún costo adicional. Sin embargo, puede seguir incurriendo en costos de uso del almacén de Azure Key Vault donde se almacena la clave.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a las características de seguridad de Azure Backup](security-overview.md)
