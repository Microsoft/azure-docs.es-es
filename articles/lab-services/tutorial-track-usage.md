---
title: Seguimiento del uso de un laboratorio en Azure Lab Services | Microsoft Docs
description: En este tutorial, realizará el seguimiento del uso de un laboratorio como su creador o propietario.
ms.topic: tutorial
ms.date: 06/26/2020
ms.openlocfilehash: a9a9b49b568decc621be1969a8578d61ac7e4861
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85445039"
---
# <a name="tutorial-track-usage-of-a-lab-in-azure-lab-service"></a>Tutorial: Seguimiento del uso de un laboratorio en Azure Lab Services
Este tutorial muestra cómo un creador o propietario de un laboratorio puede realizar un seguimiento de su uso.

En este tutorial realizará lo siguiente:

> [!div class="checklist"]
> * Ver los usuarios registrados con el laboratorio
> * Visualización del uso de las máquinas virtuales en el laboratorio
> * Administración de máquinas virtuales de estudiantes 


## <a name="view-registered-users"></a>Visualización de usuarios registrados

1. Vaya al [sitio web de Azure Lab Services](https://labs.azure.com). 
2. Seleccione **Iniciar sesión** y escriba las credenciales. Azure Lab Services es compatible con cuentas profesionales y cuentas Microsoft.
3. En la página **My labs** (Mis laboratorios), seleccione el laboratorio de cuyo uso desea realizar un seguimiento. 
4. Seleccione **Users** (Usuarios) en el menú izquierdo o el icono **Usuarios**. Verá los alumnos que se han registrado en el laboratorio.  

    ![Usuarios registrados](./media/tutorial-track-usage/registered-users.png)

    Para más información sobre cómo agregar y administrar usuarios para el laboratorio, consulte [Incorporación y administración de usuarios de laboratorio](how-to-configure-student-usage.md).

## <a name="view-the-usage-of-vms"></a>Visualización del uso de máquinas virtuales

1. Seleccione **Máquinas virtuales** en el menú de la izquierda. 
2. Confirme que ve el estado de las máquinas virtuales y el número de horas que han estado en ejecución. El tiempo que el propietario de un laboratorio emplea en una máquina virtual de un alumno no cuenta para el tiempo de uso que se muestra en la última columna. 

    ![Uso de máquinas virtuales](./media/tutorial-track-usage/vm-usage.png)

## <a name="manage-student-vms"></a>Administración de máquinas virtuales de estudiantes 
En esta página, puede iniciar, detener o restablecer las máquinas virtuales de los alumnos mediante los controles de la columna **Estado** o de la barra de herramientas.

![Acciones de la máquina virtual](./media/tutorial-track-usage/vm-controls.png)

Para más información sobre cómo administrar el grupo de máquinas virtuales para el laboratorio, consulte [Configuración y administración del grupo de máquinas virtuales](how-to-set-virtual-machine-passwords.md).

> [!NOTE]
> Cuando un profesor encienda la máquina virtual de un alumno, la cuota del alumno no sufrirá cambios. La cuota de los usuarios especifica el número de horas de laboratorio disponibles para ellos fuera del tiempo de clase programado. Para más información sobre las cuotas, vea [Establecimiento de cuotas para los usuarios](how-to-configure-student-usage.md?#set-quotas-for-users).

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre los laboratorios educativos, consulte los artículos enumerados en [Guías de procedimientos](how-to-manage-lab-accounts.md).
