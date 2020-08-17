---
title: Uso de Azure Portal para configurar las claves administradas por el cliente
titleSuffix: Azure Storage
description: Aprenda a usar Azure Portal para configurar claves administradas por el cliente con Azure Key Vault para el cifrado de Azure Storage.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/13/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: a216714939dc45fd1b220f24414a527969ab7fcb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87029602"
---
# <a name="configure-customer-managed-keys-with-azure-key-vault-by-using-the-azure-portal"></a>Configuración de claves administradas por el cliente con Azure Key Vault mediante Azure Portal

[!INCLUDE [storage-encryption-configure-keys-include](../../../includes/storage-encryption-configure-keys-include.md)]

En este artículo se muestra cómo configurar Azure Key Vault con claves administradas del cliente mediante [Azure Portal](https://portal.azure.com/). Para obtener información sobre cómo crear un almacén de claves mediante Azure Portal, consulte [Inicio rápido: Establecimiento y recuperación de un secreto de Azure Key Vault mediante Azure Portal](../../key-vault/secrets/quick-create-portal.md).

## <a name="configure-azure-key-vault"></a>Configuración de Azure Key Vault

El uso de claves administradas del cliente con el cifrado de Azure Storage requiere el establecimiento de dos propiedades en el almacén de claves, **Eliminación temporal** y **Do Not Purge** (No purgar). Estas propiedades no están habilitadas de forma predeterminada, pero se pueden habilitar mediante PowerShell o la CLI de Azure tanto en un almacén de claves nuevo como en uno existente.

Para aprender a habilitar estas propiedades en un almacén de claves existente, consulte las secciones **Habilitación de la eliminación temporal** y **Habilitación de la protección de purgas** en cualquiera de los siguientes artículos:

- [Uso de la eliminación temporal con PowerShell](../../key-vault/general/soft-delete-powershell.md).
- [Uso de la eliminación temporal con la CLI](../../key-vault/general/soft-delete-cli.md).

El cifrado de almacenamiento de Azure admite claves RSA y RSA-HSM de los tamaños 2048, 3072 y 4096. Para más información acerca de las claves, consulte la sección **Claves en Key Vault** en [Información acerca de claves, secretos y certificados de Azure Key Vault](../../key-vault/about-keys-secrets-and-certificates.md#key-vault-keys).

## <a name="enable-customer-managed-keys"></a>Habilitar claves administradas del cliente

Para habilitar claves administradas del cliente en Azure Portal, siga estos pasos:

1. Vaya a la cuenta de almacenamiento.
1. En la hoja **Configuración** de la cuenta de almacenamiento, haga clic en **Cifrado**. Seleccione la opción **Claves administradas de cliente**, como se muestra en la siguiente imagen.

    ![Captura de pantalla del portal que muestra la opción de cifrado](./media/storage-encryption-keys-portal/portal-configure-encryption-keys.png)

## <a name="specify-a-key"></a>Especificar una clave

Después de habilitar las claves administradas del cliente, tendrá la oportunidad de especificar una clave para asociarla con la cuenta de almacenamiento. También puede indicar si Azure Storage debe girar automáticamente la clave administrada por el cliente o si va a girar la clave manualmente.

### <a name="specify-a-key-from-a-key-vault"></a>Especificación de una clave a partir de un almacén de claves

Al seleccionar una clave administrada por el cliente desde un almacén de claves, la rotación automática de la clave se habilita automáticamente. Para administrar manualmente la versión de la clave, especifique el URI de la clave en su lugar e incluya la versión de la clave. Para obtener más información, consulte [Especificación de una clave como URI](#specify-a-key-as-a-uri).

Para especificar una clave a partir de un almacén de claves, siga estos pasos:

1. Elija la opción **Select from Key Vault** (Seleccionar desde almacén de claves).
1. Elija **Seleccione un almacén de claves y una clave**.
1. Seleccione el almacén de claves que contiene la clave que desea usar.
1. Seleccione la clave en el almacén de claves.

   ![Captura de pantalla que muestra cómo seleccionar el almacén de claves y la clave](./media/storage-encryption-keys-portal/portal-select-key-from-key-vault.png)

1. Guarde los cambios.

### <a name="specify-a-key-as-a-uri"></a>Especificación de una clave como URI

Cuando especifique el URI de la clave, omita la versión de la clave para habilitar la rotación automática de la clave administrada por el cliente. Si incluye la versión de la clave en el URI de la clave, la rotación automática no se habilita y debe administrar la versión de la clave usted mismo. Para obtener más información sobre la actualización de la versión de la clave, consulte [Actualización manual de la versión de la clave](#manually-update-the-key-version).

Para especificar una clave como URI, siga estos pasos:

1. Para buscar el URI de la clave en Azure Portal, vaya al almacén de claves y seleccione la opción de configuración **Claves**. Seleccione la clave que desee y luego haga clic en ella para ver sus versiones. Seleccione cualquiera de las versiones de clave para ver su configuración.
1. Copie el valor del campo **Identificador de clave**, que proporciona el URI.

    ![Captura de pantalla en que se muestra el URI de la clave del almacén de claves](media/storage-encryption-keys-portal/portal-copy-key-identifier.png)

1. En las opciones de **Clave de cifrado** de la cuenta de almacenamiento, elija la opción **Escribir el URI de la clave**.
1. Pegue el identificador URI que ha copiado en el campo **URI de clave**. Omita la versión de la clave del URI para habilitar la rotación automática.

   ![Captura de pantalla en que se muestra cómo introducir el URI de la clave](./media/storage-encryption-keys-portal/portal-specify-key-uri.png)

1. Especifique el identificador de la suscripción que contiene el almacén de claves.
1. Guarde los cambios.

Después de especificar la clave, Azure Portal indica si está habilitada la rotación automática de claves y muestra la versión de la clave actualmente en uso para el cifrado.

:::image type="content" source="media/storage-encryption-keys-portal/portal-auto-rotation-enabled.png" alt-text="Captura de pantalla que muestra habilitada la rotación automática de las claves administradas por el cliente":::

## <a name="manually-update-the-key-version"></a>Actualización manual de la versión de la clave

De forma predeterminada, Azure Storage rota automáticamente las claves administradas por el cliente, tal como se describe en las secciones anteriores. Si decide administrar la versión de la clave usted mismo, debe actualizar la versión de clave especificada para la cuenta de almacenamiento cada vez que cree una nueva versión de la clave.

Para actualizar la cuenta de almacenamiento para usar la nueva versión de la clave, siga estos pasos:

1. Vaya a la cuenta de almacenamiento y muestre las opciones de configuración de **cifrado**.
1. Escriba el identificador URI de la nueva versión de la clave. Como alternativa, puede volver a seleccionar el almacén de claves y la clave para actualizar la versión.
1. Guarde los cambios.

## <a name="switch-to-a-different-key"></a>Cambio a otra clave

Para cambiar la clave que se usa para el cifrado de Azure Storage, siga estos pasos:

1. Vaya a la cuenta de almacenamiento y muestre las opciones de configuración de **cifrado**.
1. Escriba el identificador URI de la nueva clave. Otra opción es seleccionar el almacén de claves y elegir una clave nueva.
1. Guarde los cambios.

## <a name="disable-customer-managed-keys"></a>Deshabilitación de claves administradas por el cliente

Cuando las claves administradas por el cliente se deshabilitan, la cuenta de almacenamiento se vuelve a cifrar con claves administradas por Microsoft. Para deshabilitar las claves administradas por el cliente, siga estos pasos:

1. Vaya a la cuenta de almacenamiento y muestre las opciones de configuración de **cifrado**.
1. Anule la selección de la casilla que se encuentra junto al valor **Usar su propia clave**.

## <a name="next-steps"></a>Pasos siguientes

- [Cifrado de Azure Storage para datos en reposo](storage-service-encryption.md)
- [¿Qué es Azure Key Vault?](https://docs.microsoft.com/azure/key-vault/key-vault-overview)
