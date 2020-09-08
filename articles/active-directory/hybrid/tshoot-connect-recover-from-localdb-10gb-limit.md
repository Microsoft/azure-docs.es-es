---
title: 'Azure AD Connect: Recuperación del problema que supone el límite de 10 GB en LocalDB | Microsoft Docs'
description: En este tema se explica cómo recuperar el servicio Azure AD Connect Synchronization cuando se alcanza el límite de 10 GB en LocalDB.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 41d081af-ed89-4e17-be34-14f7e80ae358
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 07/17/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7ca5361d8500ecd4ea22a577d0a4dc7ced606eab
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89275909"
---
# <a name="azure-ad-connect-how-to-recover-from-localdb-10-gb-limit"></a>Azure AD Connect: Recuperación del límite de 10 GB de LocalDB
Azure AD Connect requiere una base de datos de SQL Server para almacenar datos de identidad. Puede usar la instancia predeterminada de Local DB incluida en SQL Server 2012 Express que se instala con Azure AD Connect o utilizar la versión completa de SQL Server. SQL Server Express impone un límite en el tamaño de 10 GB. Si usa LocalDB y se alcanza este límite, el servicio Azure AD Connect Synchronization no podrá iniciarse ni sincronizarse correctamente. En este artículo se detallan los pasos de recuperación.

## <a name="symptoms"></a>Síntomas
Hay dos síntomas comunes:

* El servicio Azure AD Connect Synchronization **puede ejecutarse**, pero no es capaz de sincronizarse y se produce el error *“stopped-database-disk-full”* .

* El servicio Azure AD Connect Synchronization **no puede iniciarse**. Cuando se intenta iniciar el servicio, se produce el evento 6323 y aparece el mensaje de error *"El servidor encontró un error porque SQL Server no tiene espacio en disco".*

## <a name="short-term-recovery-steps"></a>Pasos para una recuperación a corto plazo
En esta sección se explican los pasos que deben seguirse a fin de reclamar el espacio de base de datos necesario para que el servicio Azure AD Connect Synchronization pueda reanudarse. Entre los pasos se incluyen:
1. [Determinación del estado del servicio de sincronización](#determine-the-synchronization-service-status)
2. [Reducción de la base de datos](#shrink-the-database)
3. [Eliminación de los datos del historial de ejecución](#delete-run-history-data)
4. [Reducción del período de retención de los datos del historial de ejecución](#shorten-retention-period-for-run-history-data)

### <a name="determine-the-synchronization-service-status"></a>Determinación del estado del servicio de sincronización
En primer lugar, deberá determinar si el servicio de sincronización sigue en ejecución o no:

1. Inicie sesión como administrador en el servidor de Azure AD Connect.

2. Vaya a **Administrador de control de servicios**.

3. Compruebe el estado de **Microsoft Azure AD Sync**.


4. Si el servicio está en ejecución, no lo detenga ni lo reinicie. Omita el paso [Reducción de la base de datos](#shrink-the-database) y salte directamente a [Eliminación de los datos del historial de ejecución](#delete-run-history-data).

5. Si el servicio no está en ejecución, intente iniciarlo. Si se inicia correctamente, omita el paso [Reducción de la base de datos](#shrink-the-database) y salte directamente a [Eliminación de los datos del historial de ejecución](#delete-run-history-data). De lo contrario, continúe en el paso [Reducción de la base de datos](#shrink-the-database).

### <a name="shrink-the-database"></a>Reducción de la base de datos
Utilice la operación de reducción para liberar el suficiente espacio en la base de datos como para iniciar el servicio de sincronización. Esto se consigue eliminando espacios en blanco de la base de datos. Aunque este paso es el más indicado, no garantiza que se pueda recuperar espacio en todos los casos. Para más información sobre esta operación, consulte el artículo [Reducir una base de datos](/sql/relational-databases/databases/shrink-a-database?view=sql-server-ver15).

> [!IMPORTANT]
> Omita este paso si consigue ejecutar el servicio de sincronización. No resulta recomendable reducir la base de datos SQL, ya que, al aumentar la fragmentación, el rendimiento podría verse afectado.

El nombre de la base de datos creada para Azure AD Connect es **ADSync**. Para realizar la operación de reducción, debe iniciar sesión como administrador del sistema o como DBO de la base de datos. Durante la instalación de Azure AD Connect, se concederán privilegios de administrador del sistema a las cuentas siguientes:
* Administradores locales
* La cuenta de usuario utilizada para ejecutar la instalación de Azure AD Connect.
* La cuenta del servicio de sincronización que se utilizó como contexto operativo del servicio Azure AD Connect Synchronization.
* El grupo local ADSyncAdmins que se creó durante la instalación.

1. Realice una copia de seguridad de la base de datos. Para ello, copie en una ubicación segura los archivos **ADSync.mdf** y **ADSync_log.ldf** que se encuentran en `%ProgramFiles%\Microsoft Azure AD Sync\Data`.

2. Inicie una nueva sesión de PowerShell.

3. Acceda a la carpeta `%ProgramFiles%\Microsoft SQL Server\110\Tools\Binn`.

4. Inicie la utilidad **sqlcmd** ejecutando el comando `./SQLCMD.EXE -S "(localdb)\.\ADSync" -U <Username> -P <Password>` con las credenciales del administrador del sistema o DBO de la base de datos.

5. Para reducir la base de datos, en el símbolo del sistema de sqlcmd (1>), escriba `DBCC Shrinkdatabase(ADSync,1);` y, en la siguiente línea, `GO`.

6. Si la operación se realiza correctamente, intente iniciar de nuevo el servicio de sincronización. Si consigue iniciarlo, vaya al paso [Eliminación de los datos del historial de ejecución](#delete-run-history-data). De lo contrario, póngase en contacto con el servicio de soporte técnico.

### <a name="delete-run-history-data"></a>Eliminación de los datos del historial de ejecución
De forma predeterminada, Azure AD Connect mantiene durante un máximo de siete días los datos del historial de ejecución. En este paso, eliminaremos los datos del historial de ejecución para tratar de ampliar el espacio de la base de datos y, de este modo, conseguir que el servicio Azure AD Connect Synchronization consiga sincronizarse de nuevo.

1. Inicie **Synchronization Service Manager**. Para ello, vaya a INICIO → Synchronization Service (Servicio de sincronización).

2. Vaya a la pestaña **Operations** (Operaciones).

3. En **Actions** (Acciones), seleccione **Clear Runs...** (Borrar ejecuciones...).

4. Puede seleccionar una de estas dos opciones: **Clear all runs** (Borrar todas las ejecuciones) o **Clear runs before… \<date>** (Borrar todas las ejecuciones anteriores a...). Es conveniente que empiece borrando los datos del historial de ejecución que tienen más de dos días. Si el tamaño de la base de datos sigue siendo un problema, elija la opción **Clear all runs** (Borrar todas las ejecuciones).

### <a name="shorten-retention-period-for-run-history-data"></a>Reducción del período de retención de los datos del historial de ejecución
Este paso está diseñado para reducir la probabilidad de que el límite de 10 GB resulte un problema después de ejecutar varios ciclos de sincronización.

1. Abra una nueva sesión de PowerShell.

2. Ejecute `Get-ADSyncScheduler` y anote la propiedad PurgeRunHistoryInterval, donde se especifica el período de retención actual.

3. Ejecute `Set-ADSyncScheduler -PurgeRunHistoryInterval 2.00:00:00` para establecer el período de retención en dos días. Ajuste este período según corresponda.

## <a name="long-term-solution--migrate-to-full-sql"></a>Solución a largo plazo: migración a la versión completa de SQL Server
En líneas generales, este problema indica que los 10 GB de tamaño de la base de datos ya no resultan suficientes para que Azure AD Connect pueda sincronizar la instancia local de Active Directory con Azure AD. Por tanto, se recomienda utilizar la versión completa de SQL Server. La instancia de LocalDB de una implementación existente de Azure AD Connect no se puede sustituir directamente por la base de datos de la versión completa de SQL Server. En su lugar, es necesario implementar un nuevo servidor de Azure AD Connect con la versión completa de SQL Server. Le recomendamos que realice una migración alterna en la que el nuevo servidor de Azure AD Connect (con la base de datos SQL) se implemente como servidor provisional, junto con el servidor existente de Azure AD Connect (con la instancia de LocalDB). 
* Para obtener instrucciones acerca de cómo configurar una instancia remota de SQL con Azure AD Connect, consulte el artículo [Instalación personalizada de Azure AD Connect](./how-to-connect-install-custom.md).
* Para instrucciones sobre la migración oscilante para la actualización de Azure AD Connect, consulte el artículo[Azure AD Connect: actualización de una versión anterior a la versión más reciente](./how-to-upgrade-previous-version.md#swing-migration)

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).