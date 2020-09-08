---
title: Acceso a un laboratorio educativo en Azure Lab Services | Microsoft Docs
description: En este tutorial, va a tener acceso a máquinas virtuales en un laboratorio educativo configurado por un formador.
ms.topic: tutorial
ms.date: 06/26/2020
ms.openlocfilehash: dacfa34c0d3ab637ef513342bc5ce5fe81038e11
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85443475"
---
# <a name="tutorial-access-a-classroom-lab-in-azure-lab-services"></a>Tutorial: Acceso a un laboratorio de clase en Azure Lab Services
En este tutorial, se conectará como alumno a una máquina virtual de un laboratorio educativo. 

En este tutorial realizará lo siguiente:

> [!div class="checklist"]
> * Registro en el laboratorio
> * Inicio de la máquina virtual
> * Conexión a la máquina virtual

## <a name="register-to-the-lab"></a>Registro en el laboratorio

1. Vaya a la **dirección URL de registro** que recibió del formador. No es necesario usar la dirección URL de registro después de completar el registro. En su lugar, use la dirección URL: [https://labs.azure.com](https://labs.azure.com). Todavía no se admite Internet Explorer 11. 

    ![Registro en el laboratorio](./media/tutorial-connect-vm-in-classroom-lab/register-lab.png)
1. Inicie sesión en el servicio con su cuenta organizativa para completar el registro. 

    > [!NOTE]
    > Para usar Azure Lab Services se requiere una cuenta Microsoft. Si intenta usar una cuenta que no sea de Microsoft, como las cuentas de Yahoo o de Google, para iniciar sesión en el portal, siga las instrucciones para crear una cuenta Microsoft que se vincule a una cuenta que no sea de Microsoft. Luego, siga los pasos para completar el proceso de registro. 
1. Una vez registrado, confirme que ve las máquinas virtuales de los laboratorios a los que tiene acceso. 

    ![VM accesibles](./media/tutorial-connect-vm-in-classroom-lab/accessible-vms.png)
1. Espere hasta que la máquina virtual esté lista. En el icono de VM, observe los campos siguientes:
    1. En la parte superior del icono, se puede ver el **nombre del laboratorio**.
    1. A su derecha, aparece el icono que representa el **sistema operativo (SO)** de la máquina virtual. En este ejemplo, es el sistema operativo Windows. 
    1. La barra de progreso del icono muestra el número de horas usadas en relación con el número de [horas de cuota](how-to-configure-student-usage.md#set-quotas-for-users) que tiene asignadas. Este tiempo es el tiempo adicional que se le asigna además del tiempo programado para el laboratorio. 
    1. En la parte inferior del icono hay iconos o botones para iniciar o detener la máquina virtual y para conectarse a ella. 
    1. A la derecha de los botones, se puede ver el estado de la máquina virtual. Confirme que el estado de la máquina virtual aparece como **Detenido**. 

        ![Máquina virtual con estado Detenido](./media/tutorial-connect-vm-in-classroom-lab/vm-in-stopped-state.png)

## <a name="start-the-vm"></a>Inicio de la máquina virtual
1. **Inicie** la máquina virtual seleccionando el primer botón, tal como se muestra en la siguiente imagen. Este proceso tarda algún tiempo.  

    ![Inicio de la máquina virtual](./media/tutorial-connect-vm-in-classroom-lab/start-vm.png)
4. Confirme que el estado de la máquina virtual aparece como **En ejecución**. 

    ![Máquina virtual con estado En ejecución](./media/tutorial-connect-vm-in-classroom-lab/vm-running.png)

    Observe que el icono del primer botón ha cambiado para representar una operación de **detención**. Puede seleccionar este botón para detener la máquina virtual. 

## <a name="connect-to-the-vm"></a>Conexión a la máquina virtual

1. Seleccione el segundo botón, tal como se muestra en la siguiente imagen, para **conectarse** a la máquina virtual del laboratorio. 

    ![Conexión con una máquina virtual](./media/tutorial-connect-vm-in-classroom-lab/connect-vm.png)
2. Realice uno de los siguientes pasos: 
    1. Para máquinas virtuales **Windows**, guarde el archivo **RDP** en el disco duro. Abra el archivo RDP para conectarse a la máquina virtual. Utilice el **nombre de usuario** y la **contraseña** que obtenga del formador para iniciar sesión en la máquina. 
    3. Puede usar **SSH** o **RDP** (si está habilitado) para conectarse a máquinas virtuales **Linux**. Para más información, consulte el artículo sobre la [Habilitación de la conexión a Escritorio remoto para máquinas Linux](how-to-enable-remote-desktop-linux.md). 

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha obtenido acceso a un laboratorio educativo mediante el vínculo de registro que le proporcionó el formador.

Como propietario del laboratorio, desea ver quién está registrado en él y hacer un seguimiento del uso de las máquinas virtuales. Avance al siguiente tutorial para aprender a realizar un seguimiento del uso del laboratorio:

> [!div class="nextstepaction"]
> [Realizar un seguimiento del uso de un laboratorio](tutorial-track-usage.md) 
