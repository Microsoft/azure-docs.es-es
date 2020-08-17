---
title: Escalado de privilegios de nube privada
titleSuffix: Azure VMware Solution by CloudSimple
description: Describe cómo escalar privilegios en la nube privada para las funciones administrativas en vCenter
author: sharaths-cs
ms.author: b-shsury
ms.date: 06/05/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 628824a3d303184e199452b0974c7f99d99a4933
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "86998959"
---
# <a name="escalate-private-cloud-vcenter-privileges-from-the-cloudsimple-portal"></a>Escalar privilegios de nube privada en vCenter desde el portal de CloudSimple

Para obtener acceso administrativo a vCenter de su nube privada, puede escalar temporalmente sus privilegios de CloudSimple.  Con privilegios elevados, puede instalar soluciones de VMware, agregar orígenes de identidad y administrar usuarios.

Los nuevos usuarios se pueden crear en el dominio de inicio de sesión único (SSO) de vCenter, y puede concederles acceso a vCenter.  Al crear nuevos usuarios, agréguelos a los grupos integrados de CloudSimple para acceder a vCenter.  Para más información, consulte [CloudSimple Private Cloud permission model of VMware vCenter](./learn-private-cloud-permissions.md) (Modelo de permisos de la nube privada de VMware vCenter de CloudSimple).

> [!CAUTION]
> No haga ningún cambio de configuración en los componentes de administración. Las acciones realizadas durante el estado con privilegios aumentados pueden afectar negativamente a su sistema, o pueden hacer que el sistema deje de estar disponible.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="escalate-privileges"></a>Escalado de privilegios

1. Acceda al [portal de CloudSimple](access-cloudsimple-portal.md).

2. Abra la página **Resources** (Recursos), seleccione la nube privada para la que quiere escalar los privilegios.

3. Cerca del final de la página Summary (Resumen), debajo de parte inferior de la página de resumen en **Change vSphere privileges** (Cambiar privilegios de vSphere), haga clic en **Escalate** (Escalar).

    ![Cambiar privilegio de vSphere](media/escalate-private-cloud-privilege.png)

4. Seleccione el tipo de usuario de vSphere.  Solo el usuario local `CloudOwner@cloudsimple.local` puede escalarse.

5. En la lista desplegable, seleccione el intervalo de elevación. Elija el período más corto que le permita completar la tarea.

6. Seleccione la casilla para confirmar que entiende los riesgos.

    ![Cuadro de diálogo de escalar privilegios](media/escalate-private-cloud-privilege-dialog.png)

7. Haga clic en **OK**.

8. El proceso de escalado puede tardar un par de minutos. Cuando haya terminado, haga clic en **Aceptar**.

La elevación de privilegios comienza y se mantiene hasta el final del intervalo seleccionado.  Puede iniciar sesión en vCenter de la nube privada para realizar las tareas administrativas.

> [!IMPORTANT]
> Solo un usuario puede tener privilegios elevados.  Debe anular el escalado de privilegios del usuario antes de poder escalar los privilegios de otro usuario.

> [!CAUTION]
> Los nuevos usuarios solo se deben agregar a *Cloud-Owner-Group*, *Cloud-Global-Cluster-Admin-Group*, *Cloud-Global-Storage-Admin-Group*, *Cloud-Global-Network-Admin-Group* o *Cloud-Global-VM-Admin-Group*.  Los usuarios agregados al grupo *Administradores* se quitarán automáticamente.  Solo se deben agregar cuentas de servicio al grupo *Administradores* y estas cuentas no se deben usar para iniciar sesión en la interfaz de usuario web de vSphere.

## <a name="extend-privilege-escalation"></a>Ampliar la elevación de privilegios

Si necesita más tiempo para completar las tareas, puede ampliar el período de elevación de privilegios.  Elija que el intervalo de tiempo adicional de escalado que le permita completar las tareas administrativas.

1. En **Resources** > **Private Clouds** (Recursos > Nubes privadas) en el portal de CloudSimple, seleccione la nube privada para la que quiere ampliar la elevación de privilegios.

2. Cerca del final de la pestaña Summary (Resumen), haga clic en **Extend privilege escalation** (Ampliar elevación de privilegios).

    ![Ampliar la elevación de privilegios](media/de-escalate-private-cloud-privilege.png)

3. En la lista desplegable, seleccione el intervalo de elevación. Revise la nueva hora de finalización de la elevación.

4. Haga clic en **Save** (Guardar) para ampliar el intervalo.

## <a name="de-escalate-privileges"></a>Anular el escalado de privilegios

Después de completas las tareas administrativas, debe anular el escalado de privilegios.  

1. En **Resources** > **Private Clouds** (Recursos > Nubes privadas) en el portal de CloudSimple, seleccione la nube privada para la que quiere cancelar la elevación de privilegios.

2. Haga clic en **De-escalate** (Cancelar elevación).

3. Haga clic en **OK**.

> [!IMPORTANT]
> Para evitar errores, cierre la sesión de vCenter e inicie sesión después de cancelar la elevación de privilegios.

## <a name="next-steps"></a>Pasos siguientes

* [Set up vCenter identity sources to use Active Directory](./set-vcenter-identity.md) (Configurar orígenes de identidad de vCenter para usar Active Directory)
* Instalar la solución de copia de seguridad para [copia de seguridad de máquinas virtuales de cargas de trabajo](./backup-workloads-veeam.md)
