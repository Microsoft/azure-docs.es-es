---
title: 'Expansión del grupo de hosts existente con nuevos hosts de sesión: Azure'
description: Expansión de un grupo de hosts existente con nuevos hosts de sesión en Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 04/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 339fe25a5a75ca8bc5cc0af7c4d96651e90c7eb6
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88002440"
---
# <a name="expand-an-existing-host-pool-with-new-session-hosts"></a>Expansión de un grupo de hosts existente con nuevos hosts de sesión

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop con objetos de Windows Virtual Desktop de Azure Resource Manager. Si usa Windows Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/expand-existing-host-pool-2019.md).

A medida que aumenta el uso en el grupo de hosts, puede que necesite expandir el grupo de hosts existente con nuevos hosts de sesión para controlar la nueva carga.

En este artículo se indica cómo se puede expandir un grupo de hosts existente con nuevos hosts de sesión.

## <a name="what-you-need-to-expand-the-host-pool"></a>Lo que necesita para expandir el grupo de hosts

Antes de empezar, asegúrese de que ha creado un grupo de hosts y máquinas virtuales (VM) de host de sesión con uno de los métodos siguientes:

- [Azure Portal](./create-host-pools-azure-marketplace.md)
- [Creación de un grupo host con PowerShell](./create-host-pools-powershell.md)

También necesitará la siguiente información de la primera vez que creó el grupo de hosts y las máquinas virtuales de host de sesión:

- Tamaño de máquina virtual, imagen y prefijo de nombre
- Credenciales del administrador de unión a un dominio
- Nombre de red virtual y nombre de subred

## <a name="add-virtual-machines-with-the-azure-portal"></a>Incorporación de máquinas virtuales con Azure Portal

Para expandir el grupo de hosts mediante la incorporación de máquinas virtuales:

1. Inicie sesión en Azure Portal.

2. Busque y seleccione **Windows Virtual Desktop**.

3. En el menú situado a la izquierda de la pantalla, seleccione **Grupos de hosts**, después, seleccione el nombre del grupo de hosts al que desea agregar las máquinas virtuales.

4. Seleccione **Session hosts** (Hosts de sesión) en el menú situado a la izquierda de la pantalla.

5. Seleccione **+Agregar** para empezar a crear el grupo de hosts.

6. Ignore la pestaña Aspectos básicos y, en su lugar, seleccione la pestaña **VM details** (Detalles de VM). Aquí puede ver y editar los detalles de la máquina virtual que desea agregar al grupo de hosts.

7. Seleccione el grupo de recursos en el que desea crear las máquinas virtuales y, después, seleccione la región. Puede elegir entre la región actual que está utilizando o una nueva.

8. Escriba el número de hosts de sesión que quiere agregar en el grupo de hosts en el campo **Número de máquinas virtuales**. Por ejemplo, si va a aumentar cinco hosts al grupo de hosts, escriba **5**.

    >[!NOTE]
    >No puede editar el tamaño ni la imagen de las máquinas virtuales ya que es importante garantizar que todas las máquinas virtuales del grupo de hosts tengan el mismo tamaño.

9. Para la **información de red virtual**, seleccione la red virtual y la subred a las que desea que se unan las máquinas virtuales. Puede seleccionar la misma red virtual que utilizan actualmente las máquinas existentes o elegir una diferente que sea más adecuada para la región que seleccionó en el paso 7.

10. Como **cuenta de administrador**, escriba el nombre de usuario y la contraseña del dominio de Active Directory asociado a la red virtual que ha seleccionado. Estas credenciales se utilizarán para unir las máquinas virtuales a la red virtual.

      >[!NOTE]
      >Asegúrese de que los nombres de administrador cumplen con la información que se proporciona aquí. Compruebe que la autenticación multifactor no está habilitada en la cuenta.

11. Seleccione la pestaña **Etiqueta** si tiene etiquetas con las que desee agrupar las máquinas virtuales. De lo contrario, ignore esta pestaña.

12. Seleccione la pestaña **Revisar y crear**. Revise sus opciones y, si todo es correcto, seleccione **Crear**.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha expandido el grupo de hosts existente, puede iniciar sesión en un cliente de Windows Virtual Desktop para realizar pruebas en él como parte de una sesión de usuario. Puede conectarse a una sesión con cualquiera de los siguientes clientes:

- [Conexión con el cliente de Escritorio de Windows](./connect-windows-7-10.md)
- [Conexión con el cliente web](./connect-web.md)
- [Conexión con el cliente de Android](./connect-android.md)
- [Conexión con el cliente macOS](./connect-macos.md)
- [Conexión con el cliente iOS](./connect-ios.md)
