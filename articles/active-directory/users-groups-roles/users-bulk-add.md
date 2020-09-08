---
title: Creación masiva de usuarios en el portal de Azure Active Directory | Microsoft Docs
description: Adición de usuarios en bloque en el Centro de administración de Azure AD de Azure Active Directory
services: active-directory
author: curtand
ms.author: curtand
manager: mtillman
ms.date: 08/11/2020
ms.topic: how-to
ms.service: active-directory
ms.subservice: users-groups-roles
ms.workload: identity
ms.custom: it-pro
ms.reviewer: jeffsta
ms.collection: M365-identity-device-management
ms.openlocfilehash: 64fc4e1192a1ec33b1d13c38d66e9aab5cbe256b
ms.sourcegitcommit: e0785ea4f2926f944ff4d65a96cee05b6dcdb792
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88705629"
---
# <a name="bulk-create-users-in-azure-active-directory"></a>Creación masiva de usuarios en Azure Active Directory

Azure Active Directory (Azure AD) admite operaciones de creación y eliminación de usuarios en bloque y la descarga de listas de usuarios. Solo tiene que rellenar la plantilla de valores separados por comas (CSV) que puede descargar en el portal de Azure AD.

## <a name="required-permissions"></a>Permisos necesarios

Para crear usuarios en bloque en el portal de administración, debe iniciar sesión como administrador global o administrador de usuarios.

## <a name="understand-the-csv-template"></a>Descripción de la plantilla CSV

Descargue y rellene la plantilla CSV de carga masiva para facilitar la creación en bloque correcta de los usuarios de Azure AD. La plantilla CSV que descargue podría parecerse a este ejemplo:

![Hoja de cálculo para la carga y notas que explican el propósito y los valores de cada fila y columna](./media/users-bulk-add/create-template-example.png)

> [!WARNING]
> Si agrega solo una entrada en la plantilla CSV, debe conservar la fila 3 y agregar la entrada nueva a la fila 4.

### <a name="csv-template-structure"></a>Estructura de la plantilla CSV

Las filas de una plantilla CSV descargada son las siguientes:

- **Número de versión**: la primera fila, que contiene el número de versión, debe estar incluida en el archivo CSV de carga.
- **Encabezados de columna**: el formato de los encabezados de columna es &lt;*Nombre del elemento*&gt; [nombreDePropiedad] &lt;*Required (Obligatorio) o en blanco*&gt;. Por ejemplo, `Name [displayName] Required`. Algunas versiones anteriores de la plantilla podrían tener ligeras variaciones.
- **Fila de ejemplos**: en la plantilla se incluye una fila de ejemplos de valores válidos para cada columna. Debe quitar la fila de ejemplos y reemplazarla por sus propias entradas.

### <a name="additional-guidance"></a>Instrucciones adicionales

- Las dos primeras filas de la plantilla de carga no se deben eliminar ni modificar, o no se podrá procesar la carga.
- Las columnas necesarias se enumeran en primer lugar.
- No se recomienda agregar nuevas columnas a la plantilla. Cualquier columna adicional que agregue se omitirá y no se procesará.
- Se recomienda que descargue la versión más reciente de la plantilla CSV tan a menudo como sea posible.
- Asegúrese de comprobar que no haya ningún espacio en blanco no deseado delante o detrás de ningún campo. En **Nombre principal del usuario**, el hecho de tener este espacio en blanco produciría un error de importación.

## <a name="to-create-users-in-bulk"></a>Para crear usuarios en bloque

1. [Inicie sesión en la organización de Azure AD](https://aad.portal.azure.com) con una cuenta que sea la del administrador de usuarios de la organización.
1. En Azure AD, seleccione **Usuarios** > **Creación masiva**.
1. En la página **Creación masiva de usuarios**, seleccione **Descargar** para recibir un archivo de valores separados por comas (CSV) válido de propiedades de usuario y, a continuación, agregue los usuarios que desea crear.

   ![Selección de un archivo .csv local en el que se indican los usuarios que quiere agregar](./media/users-bulk-add/upload-button.png)

1. Abra el archivo CSV y agregue una línea por cada usuario que desee crear. Los únicos valores necesarios son **Nombre**, **Nombre principal del usuario**, **Contraseña inicial** y **Bloquear inicio de sesión (Sí/No)** . A continuación, guarde el archivo.

   [![El archivo CSV contiene los nombres e identificadores de los usuarios que se van a crear](media/users-bulk-add/add-csv-file.png)](media/users-bulk-add/add-csv-file.png#lightbox)

1. En la página **Creación masiva de usuarios**, en Cargue el archivo csv, vaya al archivo. Al seleccionar el archivo y hacer clic en **Enviar**, comienza su validación.
1. Cuando finalice la validación del contenido del archivo, aparecerá el mensaje **Archivo cargado correctamente**. Si hay errores, debe corregirlos para poder enviar el trabajo.
1. Cuando el archivo supere la validación, seleccione **Enviar** para iniciar la operación masiva de Azure que importa los nuevos usuarios.
1. Cuando la operación de importación finalice, verá una notificación que indicará el estado del trabajo de la operación masiva.

Si hay errores, puede descargar y ver el archivo de resultados en la página **Resultados de la operación masiva**. El archivo contiene el motivo de cada error. El envío del archivo debe coincidir con la plantilla proporcionada e incluir los nombres de columna exactos.

## <a name="check-status"></a>Comprobar estado

Puede ver el estado de todas las solicitudes masivas pendientes en la página **Resultados de la operación masiva**.

   [![Compruebe el estado de creación en la página Resultados de la operación masiva](media/users-bulk-add/bulk-center.png)](media/users-bulk-add/bulk-center.png#lightbox)

A continuación, puede comprobar que los usuarios creados existen en la organización de Azure AD en Azure Portal o mediante PowerShell.

## <a name="verify-users-in-the-azure-portal"></a>Comprobación de usuarios en Azure Portal

1. [Inicie sesión en el Centro de administración de Azure AD](https://aad.portal.azure.com) con una cuenta que sea la del administrador de usuarios de la organización.
1. En el panel de navegación, seleccione **Azure Active Directory**.
1. En **Administrar**, seleccione **Usuarios**.
1. En **Mostrar**, seleccione **Todos los usuarios** y compruebe que se incluyen los usuarios creados.

### <a name="verify-users-with-powershell"></a>Comprobación de usuarios con PowerShell

Ejecute el siguiente comando:

``` PowerShell
Get-AzureADUser -Filter "UserType eq 'Member'"
```

Debería ver que se han incluido los usuarios creados.

## <a name="bulk-import-service-limits"></a>Límites del servicio de importación en bloque

La ejecución de cada actividad en bloque para crear usuarios puede durar hasta una hora. Esto permite la creación en bloque de al menos 50 000 usuarios.

## <a name="next-steps"></a>Pasos siguientes

- [Eliminación masiva de usuarios](users-bulk-delete.md)
- [Descarga de la lista de usuarios](users-bulk-download.md)
- [Restauración de usuarios masiva](users-bulk-restore.md)
