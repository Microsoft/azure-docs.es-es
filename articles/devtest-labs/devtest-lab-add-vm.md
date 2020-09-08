---
title: Incorporación de una máquina virtual a un laboratorio de Azure DevTest Labs | Microsoft Docs
description: Obtenga información sobre cómo usar Azure Portal para agregar una máquina virtual a un laboratorio de Azure DevTest Labs. Puede elegir una base que sea una imagen personalizada o una fórmula.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: c61e6e685a52ac3658cb9b9ca37f64fc51f3ce1e
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88270723"
---
# <a name="add-a-vm-to-a-lab-in-azure-devtest-labs"></a>Incorporación de una máquina virtual a un laboratorio de Azure DevTest Labs
Si ya ha [creado su primera máquina virtual](tutorial-create-custom-lab.md#add-a-vm-to-the-lab), es probable que lo hiciera desde una [imagen de Marketplace](devtest-lab-configure-marketplace-images.md) precargada. Ahora, si desea agregar máquinas virtuales posteriores al laboratorio, también puede elegir una *base* que sea una [imagen personalizada](devtest-lab-create-template.md) o una [fórmula](devtest-lab-manage-formulas.md). En este tutorial se explica cómo usar Azure Portal para agregar una máquina virtual a un laboratorio de DevTest Labs.

En este artículo se explica cómo administrar los artefactos de una máquina virtual de un laboratorio.

## <a name="steps-to-add-a-vm-to-a-lab-in-azure-devtest-labs"></a>Pasos para agregar una máquina virtual a un laboratorio de Azure DevTest Labs
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

        ![Elegir base de datos](./media/tutorial-create-custom-lab/new-virtual-machine.png)

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

        ![Elegir base de datos](./media/tutorial-create-custom-lab/new-vm-advanced-settings.png)
1. Seleccione **Crear** para agregar la máquina virtual especificada al laboratorio.

   En la página del laboratorio se muestra el estado de creación de la máquina virtual; primero como **Creating** (En creación) y luego como **Running** (En ejecución), una vez que se haya iniciado la máquina virtual.

    ![Estado de la creación de la máquina virtual](./media/tutorial-create-custom-lab/vm-creation-status.png)

## <a name="add-an-existing-artifact-to-a-vm"></a>Incorporación de un artefacto existente a una máquina virtual
Al crear una máquina virtual, puede agregar artefactos existentes. Cada laboratorio incluye artefactos procedentes del repositorio de artefactos de DevTest Labs público, así como los artefactos que ha creado y agregado a su propio repositorio de artefactos.

* Los *artefactos* de Azure DevTest Labs permiten especificar las *acciones* que se realizan al aprovisionarse la máquina virtual, como, por ejemplo, la ejecución de scripts de Windows PowerShell, la ejecución de comandos Bash y la instalación de software.
* Los *parámetros* del artefacto le permiten personalizar el artefacto para su escenario en particular.

Para descubrir cómo crear artefactos, consulte el artículo [Creación de artefactos personalizados para la máquina virtual de DevTest Labs](devtest-lab-artifact-author.md).

1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.
1. En la lista de laboratorios, seleccione el que contiene la máquina virtual con la que desea trabajar.
1. Seleccione **My virtual machines** (Mis máquinas virtuales).
1. Seleccione la máquina virtual deseada.
1. Seleccione **Administrar artefactos**.
1. Seleccione **Apply artifacts** (Aplicar artefactos).
1. En el panel **Aplicar artefactos**, seleccione el artefacto que desea agregar a la máquina virtual.
1. En el panel **Agregar artefacto**, especifique los valores de parámetros necesarios y cualquier otro parámetro opcional que precise.
1. Seleccione **Agregar** para agregar el artefacto y vuelva al panel **Aplicar artefactos**.
1. Siga agregando los artefactos que necesite para la máquina virtual.
1. Cuando haya agregado los artefactos, puede [cambiar el orden en que se ejecutan](#change-the-order-in-which-artifacts-are-run). También puede volver atrás para [ver o modificar un artefacto](#view-or-modify-an-artifact).
1. Cuando haya terminado de agregar artefactos, seleccione **Aplicar**.

## <a name="change-the-order-in-which-artifacts-are-run"></a>Cambio del orden en que se ejecutan los artefactos
De forma predeterminada, las acciones de los artefactos se ejecutan en el orden en que se agregan a la máquina virtual.
Los siguientes pasos muestran cómo cambiar el orden en que se ejecutan los artefactos.

1. En la parte superior del panel **Aplicar artefactos**, seleccione el vínculo que indica cuántos artefactos se han agregado a la máquina virtual.

    ![Número de artefactos que se agregan a la máquina virtual](./media/devtest-lab-add-vm-with-artifacts/devtestlab-add-artifacts-blade-selected-artifacts.png)
1. En el panel **Artefactos seleccionados**, arrastre los artefactos y colóquelos en el orden deseado. **Nota:** Si tiene problemas para arrastrar el artefacto, asegúrese de que lo arrastra desde el lado izquierdo del artefacto.
1. Seleccione **Aceptar** cuando haya terminado.

## <a name="view-or-modify-an-artifact"></a>ver o modificar un artefacto
Los siguientes pasos muestran cómo ver o modificar los parámetros de un artefacto:

1. En la parte superior del panel **Aplicar artefactos**, seleccione el vínculo que indica cuántos artefactos se han agregado a la máquina virtual.

    ![Número de artefactos que se agregan a la máquina virtual](./media/devtest-lab-add-vm-with-artifacts/devtestlab-add-artifacts-blade-selected-artifacts.png)
1. En el panel **Artefactos seleccionados**, seleccione el artefacto que desea ver o modificar.
1. En el panel **Agregar artefacto**, realice los cambios necesarios y seleccione **Aceptar** para cerrar el panel **Agregar artefacto**.
1. Seleccione **Aceptar** para cerrar el panel **Artefactos seleccionados**.

## <a name="save-azure-resource-manager-template"></a>Almacenamiento de una plantilla de Azure Resource Manager
Una plantilla de Azure Resource Manager proporciona una manera declarativa de definir una implementación repetible.
Los siguientes pasos explican cómo guardar la plantilla de Azure Resource Manager para la máquina virtual que se va a crear.
Una vez guardada, puede utilizar la plantilla de Azure Resource Manager para [implementar nuevas máquinas virtuales con Azure PowerShell](../azure-resource-manager/templates/overview.md).

1. En el panel **Máquina virtual**, seleccione **Ver plantilla de Azure Resource Manager**.
2. En el panel **Ver plantilla de Azure Resource Manager**, seleccione el texto de la plantilla.
3. Copie el texto seleccionado en el Portapapeles.
4. Seleccione **Aceptar** para cerrar el panel **Ver plantilla de Azure Resource Manager**.
5. Abra un editor de texto.
6. Pegue el texto de la plantilla desde el Portapapeles.
7. Guarde el archivo para su uso posterior.

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>Pasos siguientes
* Una vez creada la máquina virtual, puede conectarse a ella seleccionando **Conectar** en el panel de la máquina virtual.
* Aprenda a [crear artefactos personalizados para la máquina virtual de DevTest Labs](devtest-lab-artifact-author.md).
* Explore la [galería de plantillas de inicio rápido de Azure Resource Manager de DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates).
