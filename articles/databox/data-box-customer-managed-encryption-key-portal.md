---
title: Uso de Azure Portal para configurar las claves administradas por el cliente de Azure Data Box
description: Aprenda a usar Azure Portal para configurar claves administradas por el cliente con Azure Key Vault para Azure Data Box. Las claves administradas por el cliente le permiten crear, rotar, deshabilitar y revocar los controles de acceso.
services: databox
author: alkohli
ms.service: databox
ms.topic: how-to
ms.date: 05/07/2020
ms.author: alkohli
ms.subservice: pod
ms.openlocfilehash: 60e621b34250b036888b233b084ba1ddff939048
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087805"
---
# <a name="use-customer-managed-keys-in-azure-key-vault-for-azure-data-box"></a>Uso de claves administradas por el cliente en Azure Key Vault para Azure Data Box

Azure Data Box se encarga de proteger la clave de desbloqueo del dispositivo (también conocida como contraseña del dispositivo) que se usa para bloquear el dispositivo a través de una clave de cifrado. De forma predeterminada, la clave de desbloqueo del dispositivo para una orden de Data Box está cifrada con una clave administrada de Microsoft. Para tener un mayor control sobre las claves de cifrado del dispositivo, puede proporcionar también claves administradas por el cliente. 

Las claves administradas por el cliente se deben crear y almacenar en una instancia de Azure Key Vault. Para más información sobre Azure Key Vault, vea [¿Qué es Azure Key Vault?](../key-vault/general/overview.md).

En este artículo se muestra cómo usar las claves administradas por el cliente con Azure Data Box en [Azure Portal](https://portal.azure.com/). La información de este artículo se aplica tanto a los dispositivos de Azure Data Box como a los de Azure Data Box Heavy.

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que:

1. Ha creado un orden de Azure Data Box según las instrucciones del [Tutorial: Realización de pedidos de Azure Data Box](data-box-deploy-ordered.md).

2. Tiene una instancia existente de Azure Key Vault con una clave en ella, que puede usar para proteger su clave de desbloqueo del dispositivo. Para obtener información sobre cómo crear un almacén de claves mediante Azure Portal, consulte [Inicio rápido: Establecimiento y recuperación de un secreto de Azure Key Vault mediante Azure Portal](../key-vault/secrets/quick-create-portal.md).

    - Las opciones **Eliminación temporal** y **No purgar** se han establecido en el almacén de claves existente. Estas propiedades no están habilitadas de forma predeterminada. Para habilitar estas propiedades, consulte las secciones **Habilitación de la eliminación temporal** y **Habilitación de la protección de purgas** en cualquiera de los siguientes artículos:

        - [Uso de la eliminación temporal con PowerShell](../key-vault/general/soft-delete-powershell.md).
        - [Uso de la eliminación temporal con la CLI](../key-vault/general/soft-delete-cli.md).
    - El almacén de claves existente debe tener una clave RSA con un tamaño de 2048 o más. Para obtener más información sobre las claves, consulte [Información sobre las claves de Azure Key Vault](../key-vault/keys/about-keys.md).
    - El almacén de claves debe estar en la misma región que las cuentas de almacenamiento que se usan para los datos. Se pueden vincular varias cuentas de almacenamiento con el recurso de Azure Data Box.
    - Si no tiene un almacén de claves, puede crearlo de forma insertada, tal como se describe en la sección siguiente.

## <a name="enable-keys"></a>Habilitación de claves

La configuración de la clave administrada por el cliente de Azure Data Box es opcional. De forma predeterminada, Data Box usa una clave administrada por Microsoft para proteger la clave de BitLocker. Para habilitar las claves administradas por el cliente en Azure Portal, siga estos pasos:

1. Vaya a la hoja de **información general** de la orden de Data Box.

    ![Hoja de información general de la orden de Data Box](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-1.png)

2. Vaya a **Configuración > Cifrado**. En el **Tipo de cifrado**, puede elegir la manera de proteger la clave de desbloqueo del dispositivo. De forma predeterminada, se usa una clave administrada de Microsoft para proteger la contraseña de desbloqueo del dispositivo. 

    ![Elección de una opción de cifrado](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-2.png)

3. Seleccione el tipo de cifrado como **clave administrada por el cliente**. Después de seleccionar la clave administrada por el cliente, **seleccione un almacén de claves y una clave**.

    ![Selección de la clave administrada por el cliente](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-3.png)

5. En la hoja **Seleccionar clave en Azure Key Vault**, la suscripción se rellena automáticamente. Para **Almacén de claves**, puede seleccionar un almacén de claves existente de la lista desplegable.

    ![Creación de una nueva instancia de Azure Key Vault](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-31.png)

    También puede seleccionar **Crear nuevo** para crear un nuevo almacén de claves. En la hoja **Crear el almacén de claves**, escriba el grupo de recursos y el nombre del almacén de claves. Asegúrese de que estén habilitadas las opciones de **Eliminación temporal** y **Purgar protección**. Acepte todos los otros valores predeterminados. Seleccione **Revisar + crear**.

    ![Creación de una nueva instancia de Azure Key Vault](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-4.png)

7. Revise la información asociada con el almacén de claves y seleccione **Crear**. Espere un par de minutos hasta que se complete la creación del almacén de claves.

    ![Crear una instancia de Azure Key Vault](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-5.png)

8. En la hoja **Seleccionar clave en Azure Key Vault**, puede seleccionar una clave del almacén de claves existente.

    ![Creación de una clave nueva en Azure Key Vault](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-6.png)

9. Si quiere crear una clave nueva, seleccione **Crear nuevo** para crear una clave. El tamaño de la clave RSA puede ser de 2048 o superior.

    ![Creación de una clave nueva en Azure Key Vault](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-61.png)

10. Proporcione el nombre de la clave, acepte los otros valores predeterminados y seleccione **Crear**. 

    ![Creación de una clave nueva](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-7.png)


11. Se le notificará que se ha creado una clave en el almacén de claves. Seleccione la **Versión** y, a continuación, elija **Seleccionar**.

    ![Nueva clave creada en el almacén de claves](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-8.png)

12. En el panel **Tipo de cifrado**, puede ver el almacén de claves y la clave seleccionados para la clave administrada por el cliente.

    ![Clave y almacén de claves para la clave administrada por el cliente](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-9.png)

13. Guarde la clave. 

    ![Guardar la clave administrada por el cliente](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-10.png)

    La dirección URL de la clave se muestra en el **tipo de cifrado**.

    ![Dirección URL de la clave administrada por el cliente](./media/data-box-customer-managed-encryption-key-portal/customer-managed-key-11.png)

> [!IMPORTANT]
> Solo puede deshabilitar las claves administradas por Microsoft y pasar a las claves administradas por el cliente en cualquier fase de la orden de Data Box. Sin embargo, una vez que haya creado la clave administrada por el cliente, no podrá volver a cambiar a la clave administrada por Microsoft.

## <a name="troubleshoot-errors"></a>Solución de errores

Si recibe errores relacionados con la clave administrada por el cliente, use la tabla siguiente para solucionar los problemas.

| Código de error| Detalles del error| ¿Recuperable?|
|-------------|--------------|---------|
| SsemUserErrorEncryptionKeyDisabled| No se pudo capturar la clave de paso porque la clave administrada por el cliente está deshabilitada.| Sí, habilitando la versión de la clave.|
| SsemUserErrorEncryptionKeyExpired| No se pudo capturar la clave de paso porque la clave administrada por el cliente ha expirado.| Sí, habilitando la versión de la clave.|
| SsemUserErrorKeyDetailsNotFound| No se pudo capturar la clave de paso porque no se encontró la clave administrada por el cliente.| Si eliminó el almacén de claves, no puede recuperar la clave administrada por el cliente.  Si migró el almacén de claves a otro inquilino, consulte [Cambio del identificador de inquilino de Key Vault después de mover una suscripción](../key-vault/general/move-subscription.md). Si eliminó el almacén de claves:<ol><li>Sí, si está dentro de la duración de la protección de purga, con los pasos descritos en [Recuperación de un almacén de claves](../key-vault/general/soft-delete-powershell.md#recovering-a-key-vault).</li><li>No, si está fuera de la duración de la protección de purga.</li></ol><br>De lo contrario, si el almacén de claves pasó por una migración de inquilinos, sí, se puede recuperar con uno de estos pasos: <ol><li>Revierta el almacén de claves de vuelta al inquilino anterior.</li><li>Establezca `Identity = None` y, luego, vuelva a establecer el valor en `Identity = SystemAssigned`. Esto elimina y vuelve a crear la identidad una vez que se crea la identidad nueva. Habilite los permisos `Get`, `Wrap` y `Unwrap` a la identidad nueva en la directiva de acceso del almacén de claves.</li></ol> |
| SsemUserErrorKeyVaultBadRequestException| No se pudo capturar la clave de paso porque se revocó el acceso a la clave administrada por el cliente.| Sí, compruebe si: <ol><li>Key Vault todavía tiene el MSI en la directiva de acceso.</li><li>La directiva de acceso proporciona permisos para Obtener, Ajustar y Desajustar.</li><li>Si el almacén de claves está en una red virtual detrás del firewall, compruebe si la opción **Permitir servicios de Microsoft de confianza** está habilitada.</li></ol>|
| SsemUserErrorKeyVaultDetailsNotFound| No se pudo capturar la clave de paso porque no se encontró el almacén de claves asociado para la clave administrada por el cliente. | Si eliminó el almacén de claves, no puede recuperar la clave administrada por el cliente.  Si migró el almacén de claves a otro inquilino, consulte [Cambio del identificador de inquilino de Key Vault después de mover una suscripción](../key-vault/general/move-subscription.md). Si eliminó el almacén de claves:<ol><li>Sí, si está dentro de la duración de la protección de purga, con los pasos descritos en [Recuperación de un almacén de claves](../key-vault/general/soft-delete-powershell.md#recovering-a-key-vault).</li><li>No, si está fuera de la duración de la protección de purga.</li></ol><br>De lo contrario, si el almacén de claves pasó por una migración de inquilinos, sí, se puede recuperar con uno de estos pasos: <ol><li>Revierta el almacén de claves de vuelta al inquilino anterior.</li><li>Establezca `Identity = None` y, luego, vuelva a establecer el valor en `Identity = SystemAssigned`. Esto elimina y vuelve a crear la identidad una vez que se crea la identidad nueva. Habilite los permisos `Get`, `Wrap` y `Unwrap` a la identidad nueva en la directiva de acceso del almacén de claves.</li></ol> |
| SsemUserErrorSystemAssignedIdentityAbsent  | No se pudo capturar la clave de paso porque no se encontró la clave administrada por el cliente.| Sí, compruebe si: <ol><li>Key Vault todavía tiene el MSI en la directiva de acceso.</li><li>La identidad es de tipo Sistema asignado.</li><li>Habilite los permisos Get, Wrap y Unwrap a la identidad de la directiva de acceso del almacén de claves.</li></ol>|
| Error genérico  | No se pudo capturar la clave de paso.| Este es un error genérico. Póngase en contacto con el Soporte técnico de Microsoft para solucionar el error y determinar los pasos siguientes.|


## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es Azure Key Vault?](../key-vault/general/overview.md)