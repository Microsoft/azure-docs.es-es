---
title: 'Eliminación masiva de los miembros de un grupo mediante la carga de un archivo CSV: Azure Active Directory | Microsoft Docs'
description: Quite los miembros del grupo en operaciones masivas en el centro de administración de Azure.
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
ms.openlocfilehash: acf3cba5046a20b99212e89b378327765d22fa23
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421644"
---
# <a name="bulk-remove-group-members-in-azure-active-directory"></a>Eliminación masiva de miembros de un grupo en Azure Active Directory

Mediante el portal de Azure Active Directory (Azure AD), puede quitar una gran cantidad de miembros de un grupo mediante el uso de un archivo de valores separados por comas (CSV) para quitar dichos miembros de forma masiva.

## <a name="understand-the-csv-template"></a>Nociones sobre la plantilla CSV

Descargue y rellene la plantilla CSV de carga masiva para agregar correctamente miembros a un grupo de Azure AD de forma masiva. La plantilla CSV podría parecerse a este ejemplo:

![Hoja de cálculo para la carga y notas que explican el propósito y los valores de cada fila y columna](./media/groups-bulk-remove-members/template-example.png)

### <a name="csv-template-structure"></a>Estructura de la plantilla CSV

Las filas de una plantilla CSV descargada son las siguientes:

- **Número de versión**: la primera fila, que contiene el número de versión, debe estar incluida en el archivo CSV de carga.
- **Encabezados de columna**: el formato de los encabezados de columna es &lt;*Nombre del elemento*&gt; [nombreDePropiedad] &lt;*Required (Obligatorio) o en blanco*&gt;. Por ejemplo, `Member object ID or user principal name [memberObjectIdOrUpn] Required`. Algunas versiones anteriores de la plantilla podrían tener ligeras variaciones. En el caso de los cambios de pertenencia a grupos, puede elegir qué identificador usar: identificador de objeto de miembro o nombre principal de usuario.
- **Fila de ejemplos**: en la plantilla se incluye una fila de ejemplos de valores válidos para cada columna. Debe quitar la fila de ejemplos y reemplazarla por sus propias entradas.

### <a name="additional-guidance"></a>Instrucciones adicionales

- Las dos primeras filas de la plantilla de carga no se deben eliminar ni modificar, o no se podrá procesar la carga.
- Las columnas necesarias se enumeran en primer lugar.
- No se recomienda agregar nuevas columnas a la plantilla. Cualquier columna adicional que agregue se omitirá y no se procesará.
- Se recomienda que descargue la versión más reciente de la plantilla CSV tan a menudo como sea posible.

## <a name="to-bulk-remove-group-members"></a>Para quitar miembros de un grupo de forma masiva

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta de administrador de usuarios en la organización. Los propietarios de grupos también pueden quitar de forma masiva miembros de los grupos que poseen.
1. En Azure AD, seleccione **Grupos** > **Todos los grupos**.
1. Abra el grupo del que va a quitar miembros y luego seleccione **Miembros**.
1. En la página **Miembros**, seleccione **Quitar miembros**.
1. En la página **Eliminación masiva de los miembros del grupo**, seleccione **Descargar** para obtener la plantilla del archivo CSV con las propiedades de miembro de grupo requeridas.

   ![El comando Quitar miembros está en la página de perfil del grupo](./media/groups-bulk-remove-members/remove-panel.png)

1. Abra el archivo CSV y agregue una línea para cada miembro del grupo que quiera quitar del grupo (los valores necesarios son Id. de objeto de miembro o Nombre principal del usuario). A continuación, guarde el archivo.

    :::image type="content" source="./media/groups-bulk-import-members/csv-file.png" alt-text="El archivo CSV contiene los nombres y los identificadores de los miembros que se quitarán":::.

1. En la página **Eliminación masiva de los miembros del grupo**, en **Cargue el archivo csv**, vaya al archivo. Al seleccionar el archivo CSV, comienza su validación.
1. Cuando finalice la validación del contenido del archivo, aparecerá el mensaje **Archivo cargado correctamente** en la página de importación en bloque. Si hay errores, debe corregirlos para poder enviar el trabajo.
1. Cuando el archivo supere la validación, seleccione **Enviar** para iniciar la operación masiva de Azure que quita los miembros del grupo.
1. Cuando la operación de supresión finalice, verá una notificación que indicará que la operación masiva se realizó correctamente.

## <a name="check-removal-status"></a>Comprobación del estado de eliminación

Puede ver el estado de todas las solicitudes masivas pendientes en la página **Resultados de la operación masiva**.

[![Comprobación del estado en la página Resultados de la operación masiva](media/groups-bulk-remove-members/bulk-center.png)](media/groups-bulk-remove-members/bulk-center.png#lightbox)

Para obtener más información sobre cada elemento de línea de la operación masiva, seleccione los valores de las columnas **Número de elementos correctos**, **Número de errores** o **Total de solicitudes**. Si se produjeron errores, se mostrarán sus motivos.

## <a name="bulk-removal-service-limits"></a>Límites del servicio de eliminación masiva

La ejecución de cada actividad masiva para quitar una lista de miembros de grupo puede durar hasta una hora. Esto permite la eliminación de una lista de al menos 40 000 miembros.

## <a name="next-steps"></a>Pasos siguientes

- [Importación masiva de miembros de grupo](groups-bulk-import-members.md)
- [Descarga de miembros de un grupo](groups-bulk-download-members.md)
- [Descargar una lista de todos los grupos](groups-bulk-download.md)
