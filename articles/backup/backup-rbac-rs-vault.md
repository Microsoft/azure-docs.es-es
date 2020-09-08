---
title: Administración de copias de seguridad con el control de acceso basado en rol
description: Utilice el control de acceso basado en roles para administrar el acceso a las operaciones de administración de copia de seguridad en el almacén de Recovery Services.
ms.reviewer: utraghuv
ms.topic: conceptual
ms.date: 06/24/2019
ms.openlocfilehash: c24d075dcb9ac36ce741db746d69aa9e61fac39d
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89376135"
---
# <a name="use-role-based-access-control-to-manage-azure-backup-recovery-points"></a>Uso del control de acceso basado en roles para administrar puntos de recuperación de Azure Backup

El control de acceso basado en rol de Azure (Azure RBAC) permite realizar una administración detallada del acceso para Azure. También podrá repartir las tareas entre el equipo y conceder a los usuarios únicamente el nivel de acceso que necesitan para realizar su trabajo.

> [!IMPORTANT]
> Los roles proporcionados por Azure Backup están limitados a las acciones que se pueden realizar en Azure Portal o a través de la API de REST o los cmdlets de PowerShell o la CLI para el almacén de Recovery Services. Las acciones realizadas en la interfaz de usuario del cliente del agente de Azure Backup, la interfaz de usuario de System Center Data Protection Manager o la interfaz de usuario de Azure Backup Server están fuera del control de estos roles.

Azure Backup proporciona tres roles integrados para controlar las operaciones de administración de copia de seguridad. Más información sobre los [roles integrados de Azure](../role-based-access-control/built-in-roles.md)

* [Colaborador de copia de seguridad](../role-based-access-control/built-in-roles.md#backup-contributor): este rol tiene todos los permisos para crear y administrar copias de seguridad, excepto para eliminar el almacén de Recovery Services y facilitar acceso a otros usuarios. Imagine este rol como administrador de copias de seguridad que puede realizar todas las operaciones de administración de copia de seguridad.
* [Operador de copia de seguridad](../role-based-access-control/built-in-roles.md#backup-operator): Este rol tiene permisos para todo lo que puede hacer un colaborador, excepto quitar copias de seguridad y administrar directivas de copia de seguridad. Este rol es equivalente al de colaborador, salvo que no puede realizar operaciones destructivas, como detener copias de seguridad con la eliminación de datos o quitar el registro de recursos locales.
* [Lector de copia de seguridad](../role-based-access-control/built-in-roles.md#backup-reader): Este rol tiene permisos para ver todas las operaciones de administración de copia de seguridad. Imagine este rol como una persona de supervisión.

Si quiere definir sus propios roles para tener un mayor control, consulte cómo [crear roles personalizados](../role-based-access-control/custom-roles.md) en RBAC de Azure.

## <a name="mapping-backup-built-in-roles-to-backup-management-actions"></a>Asignación de roles integrados de Backup a las acciones de administración de copia de seguridad

En la tabla siguiente se capturan acciones de administración de Backup y el rol de Azure mínimo correspondiente necesario para realizar esa operación.

| Operación de administración | Rol de Azure mínimo necesario | Ámbito requerido |
| --- | --- | --- |
| Crear almacén de Recovery Services | Colaborador de copias de seguridad | Grupo de recursos que contiene el almacén |
| Habilitar la copia de seguridad de VM de Azure | Operador de copias de seguridad | Grupo de recursos que contiene el almacén |
| | Colaborador de la máquina virtual | Recurso de máquina virtual |
| Copia de seguridad a petición de VM | Operador de copias de seguridad | Almacén de Recovery Services |
| Restaurar VM | Operador de copias de seguridad | Almacén de Recovery Services |
| | Colaborador | Grupo de recursos en el que se implementará la máquina virtual |
| | Colaborador de la máquina virtual | Máquina virtual de origen de la que se hizo una copia de seguridad |
| Restaurar la copia de seguridad de la máquina virtual de discos no administrados | Operador de copias de seguridad | Almacén de Recovery Services |
| | Colaborador de la máquina virtual | Máquina virtual de origen de la que se hizo una copia de seguridad |
| | Colaborador de la cuenta de almacenamiento | Recurso de la cuenta de almacenamiento donde se van a restaurar los discos |
| Restaurar discos administrados de la copia de seguridad de la máquina virtual | Operador de copias de seguridad | Almacén de Recovery Services |
| | Colaborador de la máquina virtual | Máquina virtual de origen de la que se hizo una copia de seguridad |
| | Colaborador de la cuenta de almacenamiento | Cuenta de almacenamiento temporal seleccionada como parte de la restauración para contener datos del almacén antes de convertirlos a discos administrados |
| | Colaborador | Grupo de recursos en el cual se restaurarán los discos administrados |
| Restaurar archivos individuales desde la copia de seguridad de la máquina virtual | Operador de copias de seguridad | Almacén de Recovery Services |
| | Colaborador de la máquina virtual | Máquina virtual de origen de la que se hizo una copia de seguridad |
| Crear directiva de copia de seguridad para copia de seguridad de VM de Azure | Colaborador de copias de seguridad | Almacén de Recovery Services |
| Modificar directiva de copia de seguridad de copia de seguridad de VM de Azure | Colaborador de copias de seguridad | Almacén de Recovery Services |
| Eliminar directiva de copia de seguridad de copia de seguridad de VM de Azure | Colaborador de copias de seguridad | Almacén de Recovery Services |
| Detener copia de seguridad (con retención de datos o eliminación de datos) en copia de seguridad de VM | Colaborador de copias de seguridad | Almacén de Recovery Services |
| Registrar Windows Server, cliente o SCDPM local o Azure Backup Server | Operador de copias de seguridad | Almacén de Recovery Services |
| Eliminar Windows Server, cliente o SCDPM local registrado o Azure Backup Server | Colaborador de copias de seguridad | Almacén de Recovery Services |

> [!IMPORTANT]
> Si especifica el rol de colaborador de VM en un ámbito de recursos de VM y hace clic en **Copia de seguridad** como parte de la configuración de VM, se abrirá la pantalla **Habilitar copia de seguridad** aunque ya se haya realizado una copia de seguridad de la VM. Esto se debe a que la llamada para comprobar el estado de copia de seguridad solo funciona en el nivel de suscripción. Para evitarlo, vaya al almacén y abra la vista de elementos de copia de seguridad de la VM o especifique un rol de colaborador de VM en un nivel de suscripción.

## <a name="minimum-role-requirements-for-the-azure-file-share-backup"></a>Requisitos de rol mínimos para la copia de seguridad de recursos compartidos de archivos de Azure

En la tabla siguiente se capturan las acciones de administración de Backup y el rol correspondiente necesario para realizar la operación de recursos compartidos de archivos de Azure.

| Operación de administración | Rol requerido | Recursos |
| --- | --- | --- |
| Habilitar la copia de seguridad de recursos compartidos de archivos de Azure | Colaborador de copias de seguridad |Almacén de Recovery Services |
| |Cuenta de almacenamiento | Recurso de la cuenta de almacenamiento del colaborador |
| Copia de seguridad a petición de VM | Operador de copias de seguridad | Almacén de Recovery Services |
| Restaurar el recurso compartido de archivos | Operador de copias de seguridad | Almacén de Recovery Services |
| | Colaborador de la cuenta de almacenamiento | Recursos de la cuenta de almacenamiento donde están presentes los recursos compartidos de archivos de destino y origen de restauración |
| Restaurar archivos individuales | Operador de copias de seguridad | Almacén de Recovery Services |
| |Colaborador de la cuenta de almacenamiento|Recursos de la cuenta de almacenamiento donde están presentes los recursos compartidos de archivos de destino y origen de restauración |
| Detener protección |Colaborador de copias de seguridad | Almacén de Recovery Services |
| Anular el registro de la cuenta de almacenamiento del almacén |Colaborador de copias de seguridad | Almacén de Recovery Services |
| |Colaborador de la cuenta de almacenamiento | Recurso de la cuenta de almacenamiento|

## <a name="next-steps"></a>Pasos siguientes

* [Control de acceso basado en roles de Azure (Azure RBAC)](../role-based-access-control/role-assignments-portal.md): Puede comenzar con RBAC en Azure Portal.
* Aprenda a administrar el acceso con:
  * [PowerShell](../role-based-access-control/role-assignments-powershell.md)
  * [CLI de Azure](../role-based-access-control/role-assignments-cli.md)
  * [REST API](../role-based-access-control/role-assignments-rest.md)
* [Solución de problemas del control de acceso basado en rol](../role-based-access-control/troubleshooting.md): Obtenga sugerencias acerca de cómo solucionar problemas comunes.
