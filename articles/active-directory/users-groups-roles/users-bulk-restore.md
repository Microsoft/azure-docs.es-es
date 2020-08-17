---
title: Restauración masiva de usuarios eliminados en el portal de Azure Active Directory | Microsoft Docs
description: Restauración de usuarios en bloque en el Centro de administración de Azure AD de Azure Active Directory
services: active-directory
author: curtand
ms.author: curtand
manager: mtillman
ms.date: 04/27/2020
ms.topic: how-to
ms.service: active-directory
ms.subservice: users-groups-roles
ms.workload: identity
ms.custom: it-pro
ms.reviewer: jeffsta
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4fe5cacfec6ee85a5d61204ffce38c856b0d1baf
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423344"
---
# <a name="bulk-restore-deleted-users-in-azure-active-directory"></a>Restauración masiva de usuarios eliminados en Azure Active Directory

Azure Active Directory (Azure AD) admite operaciones de restauración de usuarios en masa, así como la descarga de listas de usuarios, grupos y miembros del grupo.

## <a name="understand-the-csv-template"></a>Nociones sobre la plantilla CSV

Descargue y rellene la plantilla CSV para restaurar en masa a los usuarios de Azure AD de forma correcta. La plantilla CSV que descargue podría parecerse a este ejemplo:

![Hoja de cálculo para la carga y notas que explican el propósito y los valores de cada fila y columna](./media/users-bulk-restore/understand-template.png)

### <a name="csv-template-structure"></a>Estructura de la plantilla CSV

Las filas de una plantilla CSV descargada son las siguientes:

- **Número de versión**: la primera fila, que contiene el número de versión, debe estar incluida en el archivo CSV de carga.
- **Encabezados de columna**: el formato de los encabezados de columna es &lt;*Nombre del elemento*&gt; [nombreDePropiedad] &lt;*Required (Obligatorio) o en blanco*&gt;. Por ejemplo, `Object ID [objectId] Required`. Algunas versiones anteriores de la plantilla podrían tener ligeras variaciones.
- **Fila de ejemplos**: en la plantilla se incluye una fila de ejemplos de valores válidos para cada columna. Debe quitar la fila de ejemplos y reemplazarla por sus propias entradas.

### <a name="additional-guidance"></a>Instrucciones adicionales

- Las dos primeras filas de la plantilla de carga no se deben eliminar ni modificar, o no se podrá procesar la carga.
- Las columnas necesarias se enumeran en primer lugar.
- No se recomienda agregar nuevas columnas a la plantilla. Cualquier columna adicional que agregue se omitirá y no se procesará.
- Se recomienda que descargue la versión más reciente de la plantilla CSV tan a menudo como sea posible.

## <a name="to-bulk-restore-users"></a>Para restaurar usuarios en masa

1. [Inicie sesión en la organización de Azure AD](https://aad.portal.azure.com) con una cuenta de administrador de usuarios de esta organización.
1. En Azure AD, seleccione **Usuarios** > **Usuarios eliminados**.
1. En la página **Usuarios eliminados**, seleccione **Restauración masiva** para cargar un archivo .csv válido de las propiedades de los usuarios que se van a restaurar.

    ![Selección del comando de restauración masiva en la página Usuarios eliminados](./media/users-bulk-restore/bulk-restore.png)

1. Abra la plantilla CSV y agregue una línea por cada usuario que quiera restaurar. El único valor necesario es **ObjectID**. A continuación, guarde el archivo.

    :::image type="content" source="./media/users-bulk-restore/upload-button.png" alt-text="Selección de un archivo .csv local en el que se indican los usuarios que quiere agregar":::

1. En la página **Restauración masiva**, en **Cargue el archivo csv**, vaya al archivo. Al seleccionar el archivo y hacer clic en **Enviar**, comienza su validación.
1. Cuando finalice la validación del contenido del archivo, aparecerá el mensaje **Archivo cargado correctamente**. Si hay errores, debe corregirlos para poder enviar el trabajo.
1. Cuando el archivo supere la validación, seleccione **Enviar** para iniciar la operación masiva de Azure que restaura los usuarios.
1. Cuando la operación de restauración finalice, verá una notificación que indicará que la operación masiva se realizó correctamente.

Si hay errores, puede descargar y ver el archivo de resultados en la página **Resultados de la operación masiva**. El archivo contiene el motivo de cada error.

## <a name="check-status"></a>Comprobar estado

Puede ver el estado de todas las solicitudes masivas pendientes en la página **Resultados de la operación masiva**.

[![Compruebe el estado en la página Resultados de la operación masiva.](media/users-bulk-restore/bulk-center.png)](media/users-bulk-restore/bulk-center.png#lightbox)

A continuación, puede comprobar si los usuarios restaurados existen en la organización de Azure AD en Azure Portal o mediante PowerShell.

## <a name="view-restored-users-in-the-azure-portal"></a>Visualización de los usuarios restaurados en Azure Portal

1. [Inicie sesión en el Centro de administración de Azure AD](https://aad.portal.azure.com) con una cuenta que sea la del administrador de usuarios de la organización.
1. En el panel de navegación, seleccione **Azure Active Directory**.
1. En **Administrar**, seleccione **Usuarios**.
1. En **Mostrar**, seleccione **Todos los usuarios** y compruebe que se incluyen los usuarios restaurados.

### <a name="view-users-with-powershell"></a>Visualización de usuarios con PowerShell

Ejecute el siguiente comando:

``` PowerShell
Get-AzureADUser -Filter "UserType eq 'Member'"
```

Debería ver que se han incluido los usuarios restaurados.

## <a name="next-steps"></a>Pasos siguientes

- [Importación en bloque de usuarios](users-bulk-add.md)
- [Eliminación masiva de usuarios](users-bulk-delete.md)
- [Descarga de la lista de usuarios](users-bulk-download.md)
