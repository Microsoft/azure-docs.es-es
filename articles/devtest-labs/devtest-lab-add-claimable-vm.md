---
title: Creación y administración de VM reclamables en Azure DevTest Labs | Microsoft Docs
description: Obtenga información sobre cómo usar Azure Portal para agregar una máquina virtual reclamable en Azure DevTest Labs y ver los procesos que se van a seguir para reclamar una máquina virtual o anular su reclamación.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 1fe949d7104fe6437087e77c403cc0d921ebd025
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88270986"
---
# <a name="create-and-manage-claimable-vms-in-azure-devtest-labs"></a>Creación y administración de VM reclamables en Azure DevTest Labs
Una imagen virtual reclamable se agrega a un laboratorio de forma similar a como se [agrega una máquina virtual estándar](devtest-lab-add-vm.md): desde una *base* que es una [imagen personalizada](devtest-lab-create-template.md), [fórmula](devtest-lab-manage-formulas.md), o [imagen de Marketplace](devtest-lab-configure-marketplace-images.md). Este tutorial le guía por el proceso de utilización de Azure Portal para agregar una máquina virtual reclamable a un laboratorio de DevTest Labs, y muestra los procesos que sigue un usuario para reclamar la VM y anular la reclamación de esta.

## <a name="steps-to-add-a-claimable-vm-to-a-lab-in-azure-devtest-labs"></a>Pasos para agregar una máquina virtual reclamable a un laboratorio de Azure DevTest Labs
1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la sección **DevOps**. Si selecciona * (asterisco) junto a **DevTest Labs** en la sección **DevOps**. Esta acción agrega **DevTest Labs** al menú de navegación izquierdo para facilitarle el acceso la próxima vez. A continuación, puede seleccionar **DevTest Labs** en el menú de navegación izquierdo.

    ![Todos los servicios. Selección de DevTest Labs](./media/devtest-lab-create-lab/all-services-select.png)
1. En la lista de laboratorios, seleccione aquel en el que desea crear la máquina virtual.
2. En el panel **Información general** del laboratorio, seleccione **+ Agregar**.

    ![Botón Agregar VM](./media/devtest-lab-add-vm/devtestlab-home-blade-add-vm.png)
1. En la página **Choose a base** (Elegir una base), seleccione una imagen de Marketplace para la máquina virtual.
1. En la pestaña **Configuración básica** de la página **Máquina virtual**, realice las acciones siguientes:
    1. En el cuadro de texto **Nombre de la máquina virtual**, escriba un nombre para la máquina virtual. El cuadro de texto se rellenará automáticamente con un nombre único generado de forma automática. El nombre corresponde al nombre de usuario dentro de la dirección de correo electrónico, seguido por un número único de 3 dígitos. Esta característica permite ahorrarse tiempo de pensar en un nombre de máquina y escribirlo cada vez que cree una máquina. Puede invalidar el nombre automático de este campo con el nombre que elija. Para invalidar el nombre automático de la máquina virtual, escriba un nombre en el cuadro de texto **Nombre de máquina virtual**.
    2. Escriba un **Nombre de usuario** al que se concederán privilegios de administrador en la máquina virtual. El **nombre de usuario** de la máquina se rellena previamente con un nombre único generado automáticamente. El nombre corresponde al nombre de usuario dentro de la dirección de correo electrónico. Esta característica reduce el tiempo necesario decidir cuál es el nombre de usuario cada vez que se crea una nueva máquina. Puede invalidar el nombre automático de este campo con el nombre que elija. Para invalidar el nombre de usuario automático, escriba un valor en el cuadro de texto **Nombre de usuario**. A este usuario se le conceden privilegios de **administrador** en la máquina virtual.
    3. Si va a crear la primera máquina virtual en el laboratorio, escriba una **contraseña** para el usuario. Para guardar esta contraseña como contraseña predeterminada en el almacén de claves de Azure asociado al laboratorio, seleccione **Guardar como contraseña predeterminada**. La contraseña predeterminada se guarda en el almacén de claves con el nombre: **VmPassword**. Cuando después se intenta crear máquinas virtuales en el laboratorio, **VmPassword** se selecciona automáticamente como **Contraseña**. Para invalidar el valor, desactive la casilla **Usar un secreto guardado** y escriba una contraseña.

        Puede también guardar primero los secretos en el almacén de claves y, después, usarlos al crear una máquina virtual en el laboratorio. Para más información, consulte [Almacenamiento de secretos en un almacén de claves](devtest-lab-store-secrets-in-key-vault.md). Para usar una contraseña almacenada en un almacén de claves, seleccione **Usar un secreto guardado** y especifique un valor de clave que corresponda a su secreto (contraseña).
    4. En la sección **Más opciones** seleccione **Cambiar tamaño**. Seleccione uno de los elementos predefinidos que especifican los núcleos del procesador, el tamaño de RAM y el tamaño de la unidad de disco duro de la máquina virtual que se va a crear.
    5. Seleccione **Add or Remove Artifacts** (Agregar o quitar artefactos). Seleccione y configure los artefactos que quiere agregar a la imagen base.
    **Nota:** Si no está familiarizado con DevTest Labs o la configuración de artefactos, vaya a la sección [Incorporación de un artefacto existente a una máquina virtual](./devtest-lab-add-vm.md#add-an-existing-artifact-to-a-vm) y vuelva aquí cuando haya finalizado.
2. Cambie a la pestaña **Configuración avanzada** en la parte superior, y realice las acciones siguientes:
    1. Para cambiar la red virtual en la que se encuentra la máquina virtual, seleccione **Cambiar la red virtual**.
    2. Para cambiar la subred, seleccione **Cambiar la subred**.
    3. Especifique si la dirección IP de la máquina virtual es **pública, privada o compartida**.
    4. Para eliminar automáticamente la máquina virtual, especifique la **fecha y hora de expiración**.
    5. Para que un usuario de laboratorio pueda reclamar la máquina, seleccione **Sí** para la opción **Make this machine claimable** (Poder reclamar esta máquina).
    6. Especifique el número de **instancias de máquina virtual** que desea que estén disponibles para los usuarios del laboratorio.
3. Seleccione **Crear** para agregar la máquina virtual especificada al laboratorio.

   En la página del laboratorio se muestra el estado de creación de la máquina virtual; primero como **Creating** (En creación) y luego como **Running** (En ejecución), una vez que se haya iniciado la máquina virtual.

> [!NOTE]
> Si implementa máquinas virtuales de laboratorio mediante las  [plantillas de Azure Resource Manager](devtest-lab-create-environment-from-arm.md), puede crear máquinas virtuales reclamables estableciendo la propiedad **allowClaim** en "true" en la sección de propiedades.


## <a name="using-a-claimable-vm"></a>Uso de una máquina virtual reclamable

Un usuario puede reclamar cualquier máquina virtual de la lista de máquinas virtuales reclamables siguiendo alguno de estos pasos:

* En la lista de máquinas virtuales reclamables, en la parte inferior del panel de información general del laboratorio, haga doble clic en una de las máquinas virtuales en la lista y elija **Claim machine**  (Reclamar máquina).

  ![Solicite una máquina virtual reclamable específica.](./media/devtest-lab-add-vm/devtestlab-claim-VM.png)


* En la parte superior del panel de información general , elija **Claim any** (Reclamar cualquiera). Se asigna una máquina virtual aleatoria de la lista de máquinas virtuales reclamables.

  ![Solicite cualquier máquina virtual reclamable.](./media/devtest-lab-add-vm/devtestlab-claim-any.png)


Una vez que un usuario reclame una máquina virtual, DevTest Labs iniciará el equipo y lo trasladará a la lista "Mis máquinas virtuales" del usuario del laboratorio. Esto significa que el usuario del laboratorio tendrá ahora privilegios de propietario en esta máquina. El tiempo necesario para este paso puede variar en función de los tiempos de inicio, así como de cualquier otra acción personalizada que se realice durante el evento de la reclamación. Una vez reclamada, la máquina deja de estar disponible en el grupo reclamable.  

## <a name="unclaim-a-vm"></a>Anular la reclamación de una VM

Cuando un usuario ha terminado de usar una VM reclamada y desea ponerla a disposición de otra persona, puede devolver la VM reclamada a la lista de máquinas virtuales reclamables siguiendo uno de estos pasos:

- En la lista de "Mis máquinas virtuales", haga clic con el botón derecho en una de las VM de la lista, o seleccione los puntos suspensivos (...) y elija **Anular reclamación**.

  ![Anule la reclamación de una VM en la lista de VM.](./media/devtest-lab-add-vm/devtestlab-unclaim-VM2.png)

- En la lista de "Mis máquinas virtuales", seleccione una VM para abrir el panel de administración y, luego, seleccione **Anular reclamación** desde la barra de menú superior.

  ![Anule la reclamación de VM en el panel de administración de VM.](./media/devtest-lab-add-vm/devtestlab-unclaim-VM.png)

Cuando un usuario anula la reclamación de una VM, ya no tiene permisos de propietario para esa VM de laboratorio específica y estará disponible para que la reclame cualquier otro usuario de laboratorio en el estado en que se devolvió al grupo. 

### <a name="transferring-the-data-disk"></a>Transferencia de disco de datos
Si una VM reclamable dispone de un disco de datos conectado a ella y un usuario anula la reclamación de ella, el disco de datos continua con la VM. Cuando otro usuario reclama esa VM, el nuevo usuario reclama el disco de datos, así como la VM.

Esto se conoce como "transferencia del disco de datos". El disco de datos luego pasa a estar disponible en la lista del usuario de **Mis discos de datos** para su administración.

![Anule la reclamación de los discos de datos.](./media/devtest-lab-add-vm/devtestlab-unclaim-datadisks.png)



## <a name="next-steps"></a>Pasos siguientes
* Una vez que se haya creado, puede conectarse a la VM. Para ello, seleccione **Conectar** en el panel de administración.
* Explore la [galería de plantillas de inicio rápido de Azure Resource Manager de DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates).
