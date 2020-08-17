---
title: Administración de directivas de laboratorio en Azure DevTest Labs | Microsoft Docs
description: Obtenga información acerca de cómo definir directivas de laboratorio como tamaños de máquina virtual, máximo de máquinas virtuales por usuario y automatización de apagado.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: ab2e2b1905a515d3a85339099f02d93d8e5c0424
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87290422"
---
# <a name="manage-all-policies-for-a-lab-in-azure-devtest-labs"></a>Administración de todas las directivas para un laboratorio de Azure DevTest Labs

Azure DevTest Labs le permite controlar los costos y reducir el desperdicio en sus laboratorios mediante la administración de directivas (configuración) en cada uno de ellos. Este artículo explica en detalle paso a paso cómo configurar cada directiva.  

## <a name="set-allowed-virtual-machine-sizes"></a>Establecimiento de tamaños de máquina virtual permitidos
La directiva para establecer los tamaños permitidos de la máquina virtual ayuda a minimizar la pérdida del laboratorio al permitirle especificar los tamaños de máquina virtual que se permiten en este. Si se activa esta directiva, los tamaños de máquina virtual de esta lista son los únicos que pueden utilizarse en la creación de tales máquinas.

1. En [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040), seleccione un laboratorio y, luego, seleccione **Configuración y directivas**.

    ![Acceso a la configuración y directivas del laboratorio](./media/devtest-lab-set-lab-policy/policies-menu.png)

1. En el panel **Configuración y directivas** del laboratorio, seleccione **Allowed virtual machines sizes** (Tamaños de máquinas virtuales permitidas).
   
    ![Allowed virtual machines sizes](./media/devtest-lab-set-lab-policy/allowed-vm-sizes.png)

1. Seleccione **Activado** para habilitar esta directiva, y **Desactivado** para deshabilitarla.

1. Si habilita esta directiva, seleccione uno o varios tamaños de máquina virtual que se pueden crear en el laboratorio.

1. Seleccione **Guardar**.

## <a name="set-virtual-machines-per-user"></a>Establecimiento de máquinas virtuales por usuario
La directiva **Virtual Machines por usuario** le permite especificar el número de máquinas virtuales que puede crear un usuario individual. Si un usuario intenta crear o reclamar una máquina virtual una vez alcanzado el límite, aparece un mensaje de error que indica que la máquina virtual no se puede crear ni reclamar. 

1. En el panel **Configuración y directivas** del laboratorio, seleccione **Virtual Machines por usuario**.
   
    ![Máquinas virtuales por usuario](./media/devtest-lab-set-lab-policy/max-vms-per-user.png)

1. Seleccione **Sí** para limitar el número de máquinas virtuales por usuario. Si no quiere limitar el número de máquinas virtuales por usuario, seleccione **No**. Si elige **Sí**, escriba un valor numérico que indique el número de máquinas virtuales que un usuario puede crear o reclamar. 

1. Seleccione **Sí** para limitar el número de máquinas virtuales que puede usar SSD (discos de estado sólido). Si no quiere limitar el número de máquinas virtuales que puede usar SSD, seleccione **No**. Si elige **Sí**, escriba un valor que indique el número de máquinas virtuales que se pueden crear con SSD. 

1. Seleccione **Guardar**.

## <a name="set-virtual-machines-per-lab"></a>Establecimiento de máquinas virtuales por laboratorio
La directiva **Virtual Machines por laboratorio** le permite especificar el número máximo de máquinas virtuales que se pueden crear para el laboratorio actual. Si un usuario intenta crear una máquina virtual una vez alcanzado el límite, aparece un mensaje de error que indica que la máquina virtual no se puede crear. 

1. En el panel **Configuración y directivas** del laboratorio, seleccione **Virtual Machines por laboratorio** .
   
    ![Máquinas virtuales por laboratorio](./media/devtest-lab-set-lab-policy/max-vms-per-lab.png)

1. Seleccione **Sí** para limitar el número de máquinas virtuales por laboratorio. Si no quiere limitar el número de máquinas virtuales por laboratorio, seleccione **No**. Si elige **Sí**, escriba un valor numérico que indique el número de máquinas virtuales que un usuario puede crear o reclamar. 

1. Seleccione **Sí** para limitar el número de máquinas virtuales que puede usar SSD (discos de estado sólido). Si no quiere limitar el número de máquinas virtuales que puede usar SSD, seleccione **No**. Si elige **Sí**, escriba un valor que indique el número de máquinas virtuales que se pueden crear con SSD. 

1. Seleccione **Guardar**.

## <a name="set-auto-shutdown"></a>Establecer el apagado automático
La directiva de apagado automático ayuda a reducir los desperdicios del laboratorio al permitirle especificar la hora de apagado de la máquina virtual de este laboratorio.

1. En el panel **Configuration and policies** (Configuración y directivas) del laboratorio, seleccione **Auto shutdown** (Apagado automático).
   
    ![Apagado automático](./media/devtest-lab-set-lab-policy/auto-shutdown.png)

1. Seleccione **Activado** para habilitar esta directiva, y **Desactivado** para deshabilitarla.

1. Si habilita esta directiva, especifique la hora local (y la zona horaria) para apagar todas las máquinas virtuales del laboratorio actual.

1. Especifique **Sí** o **No** en la opción de enviar una notificación 15 minutos antes de la hora especificada para el apagado automático. Si elige **Sí**, escriba un punto de conexión de URL del webhook o una dirección de correo electrónico para especificar dónde quiere que se publique o se envíe la notificación. El usuario recibe la notificación y se le ofrece la opción de retrasar el apagado.

   Para obtener más información sobre los webhooks, consulte [Creación de un webhook o una función de API de Azure](../azure-functions/functions-bindings-http-webhook.md). 

1. Seleccione **Guardar**.

De manera predeterminada, una vez que se habilite, esta directiva se aplica a todas las máquinas virtuales del laboratorio actual. Para quitar esta configuración de una máquina virtual específica, abra el panel de administración de la máquina virtual y cambie la configuración de **Apagado automático**.

## <a name="set-auto-shutdown-policy"></a>Establecer la directiva de apagado automático
Como propietario del laboratorio, puede configurar una programación de apagado para todas las máquinas virtuales de su laboratorio. Al hacerlo, puede ahorrar costos derivados de la ejecución de máquinas que no se usan (inactivas). Puede aplicar una directiva de apagado en todas las máquinas del laboratorio a nivel central, pero también ahorrarles a los usuarios del laboratorio el trabajo de configurar una programación para sus máquinas individuales. Esta característica le permite definir la directiva en la programación del laboratorio: desde no ofrecer ningún control a ofrecer control completo a los usuarios del laboratorio. Como propietario del laboratorio, puede configurar esta directiva mediante los pasos siguientes:

1. En la página principal del laboratorio, seleccione **Configuration and policies** (Configuración y directivas).
2. Seleccione **Auto shutdown policy** (Directiva de apagado automático) en la sección **Schedules** (Programaciones) del menú izquierdo.
3. Seleccione una de las opciones. En las siguientes secciones, se proporcionan más detalles de estas opciones: La directiva establecida se aplica solo a las nuevas máquinas virtuales creadas en el laboratorio y no a las máquinas virtuales ya existentes. 

    ![Opciones de la directiva de apagado automático](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-options.png)

### <a name="user-sets-a-schedule-and-can-opt-out"></a>El usuario establece una programación y puede excluirse de ella
Si establece esta directiva para el laboratorio, los usuarios del laboratorio pueden excluirse de la programación del laboratorio o invalidarla. Esta opción concede a los usuarios del laboratorio control completo sobre la programación del apagado automático de sus máquinas virtuales. Los usuarios del laboratorio no verán ningún cambio en su página de programación de apagado de la máquina virtual.

![Directiva de apagado automático: opción 1](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-1.png)

### <a name="user-sets-a-schedule-and-cannot-opt-out"></a>El usuario establece una programación y no puede excluirse de ella
Si establece esta directiva para el laboratorio, los usuarios del laboratorio pueden invalidar la programación del laboratorio. Sin embargo, no se pueden excluir de la directiva de apagado automático. Esta opción garantiza que todas las máquinas del laboratorio están bajo una programación de apagado automático. Los usuarios del laboratorio pueden actualizar la programación de apagado automático de sus máquinas virtuales y configurar notificaciones de apagado.

![Directiva de apagado automático: opción 2](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-2.png)

### <a name="user-has-no-control-over-the-schedule-set-by-lab-admin"></a>El usuario no tiene control sobre la programación establecida por el administrador del laboratorio
Si establece esta directiva para el laboratorio, los usuarios del laboratorio no pueden excluirse de la programación del laboratorio ni invalidarla. Esta opción ofrece al administrador del laboratorio control total sobre la programación de cada máquina del laboratorio. Los usuarios del laboratorio solo pueden configurar notificaciones de apagado automático para sus máquinas virtuales.

![Directiva de apagado automático: opción 3](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-3.png)

## <a name="set-autostart"></a>Establecer el inicio automático
La directiva de inicio automático le permite especificar cuándo se deben iniciar las máquinas virtuales del laboratorio actual.  

1. En el panel **Configuration and policies** (Configuración y directivas) del laboratorio, seleccione **Autostart** (Inicio automático).
   
    ![Inicio automático](./media/devtest-lab-set-lab-policy/auto-start.png)

2. Seleccione **Activado** para habilitar esta directiva, y **Desactivado** para deshabilitarla.

3. Si habilita esta directiva, especifique a la hora local de inicio programado, la zona horaria los días de la semana en los que se aplica esta hora. 

4. Seleccione **Guardar**.

Una vez que se habilite, esta directiva no se aplica automáticamente a ninguna máquina virtual del laboratorio actual. Para aplicar esta configuración a una máquina virtual específica, abra el panel de administración de la máquina virtual y cambie su configuración de **Autostart** (Inicio automático).

## <a name="set-expiration-date"></a>Establecimiento de la fecha de expiración
Puede establecer una fecha de expiración cuando [cree la VM](devtest-lab-add-vm.md). En **Configuración avanzada**, elija el icono del calendario para especificar una fecha en la que la máquina virtual se eliminará automáticamente. De forma predeterminada, la máquina virtual nunca expirará.

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>Pasos siguientes
Una vez que defina y aplique las distintas configuraciones de las directivas de máquina virtual correspondientes al laboratorio, puede intentar algunos de los siguientes pasos:

* [Direcciones IP compartidas](devtest-lab-shared-ip.md); explica cómo las direcciones IP compartidas se usan en DevTest Labs para minimizar el número de direcciones IP públicas necesarias para conectarse a las máquinas virtuales del laboratorio.
* [Configuración de la administración de costos](devtest-lab-configure-cost-management.md): muestra cómo utilizar el gráfico **Tendencia de costo estimado mensual**  
  para ver el costo estimado hasta la fecha del mes de calendario actual, así como el costo previsto para fin de mes.
* [Creación de una imagen personalizada](devtest-lab-create-template.md): cuando cree una máquina virtual, tendrá que especificar una base, que puede ser una imagen personalizada o una imagen de Marketplace. Este artículo muestra cómo crear una imagen personalizada desde un archivo VHD.
* [Configuración de imágenes de Marketplace](devtest-lab-configure-marketplace-images.md): Azure DevTest Labs admite la creación de máquinas virtuales basadas en imágenes de Azure Marketplace. En este artículo se muestra cómo especificar las imágenes de Azure Marketplace, si corresponde, que se pueden usar en el momento de crear máquinas virtuales en un laboratorio.
* [Creación de una máquina virtual en un laboratorio](devtest-lab-add-vm.md): se muestra cómo crear una máquina virtual desde una imagen base (ya sea personalizada o de Marketplace) y cómo trabajar con artefactos en la máquina virtual.
