---
title: Expansión de un grupo de hosts existente de Windows Virtual Desktop (clásico) con nuevos hosts de sesión - Azure
description: Expansión de un grupo de hosts existente con nuevos hosts de sesión en Windows Virtual Desktop (clásico)
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 61cf28b0f1ebee6a0312ec3f23f22b01c6c4919e
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009178"
---
# <a name="expand-an-existing-host-pool-with-new-session-hosts-in-windows-virtual-desktop-classic"></a>Expansión de un grupo de hosts existente con nuevos hosts de sesión en Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager. Si está tratando de administrar objetos de Windows Virtual Desktop para Azure Resource Manager, consulte [este artículo](../expand-existing-host-pool.md).

A medida que aumenta el uso en el grupo de hosts, puede que necesite expandir el grupo de hosts existente con nuevos hosts de sesión para controlar la nueva carga.

En este artículo se indica cómo se puede expandir un grupo de hosts existente con nuevos hosts de sesión.

## <a name="what-you-need-to-expand-the-host-pool"></a>Lo que necesita para expandir el grupo de hosts

Antes de empezar, asegúrese de que ha creado un grupo de hosts y máquinas virtuales (VM) de host de sesión con uno de los métodos siguientes:

- [Oferta de Azure Marketplace](create-host-pools-azure-marketplace-2019.md)
- [Plantilla de Azure Resource Manager de GitHub](create-host-pools-arm-template.md)
- [Creación de un grupo host con PowerShell](create-host-pools-powershell-2019.md)

También necesitará la siguiente información de la primera vez que creó el grupo de hosts y las máquinas virtuales de host de sesión:

- Tamaño de máquina virtual, imagen y prefijo de nombre
- Unión a un dominio y credenciales de administrador de inquilinos de Windows Virtual Desktop
- Nombre de red virtual y nombre de subred

Las tres secciones siguientes son tres métodos que puede usar para expandir el grupo de hosts. Puede hacerlo con cualquier herramienta de implementación con la que esté familiarizado.

>[!NOTE]
>Durante la fase de implementación, verá mensajes de error de los anteriores recursos de máquina virtual de host de sesión si están apagados. Estos errores se producen porque Azure no puede ejecutar la extensión DSC de PowerShell para validar que las máquinas virtuales de host de sesión se hayan registrado correctamente en el grupo de hosts existente. Puede omitir estos errores de forma segura o puede evitarlos si inicia todas las máquinas virtuales de host de sesión en el grupo de hosts existente antes de iniciar el proceso de implementación.

## <a name="redeploy-from-azure"></a>Nueva implementación desde Azure

Si ya ha creado un grupo de hosts y máquinas virtuales de host de sesión mediante la [oferta de Azure Marketplace](create-host-pools-azure-marketplace-2019.md) o la [plantilla de Azure Resource Manager de GitHub](create-host-pools-arm-template.md), puede volver a implementar la misma plantilla desde Azure Portal. Al volver a implementar la plantilla, se vuelve a escribir automáticamente toda la información que especificó en la plantilla original, excepto las contraseñas.

Aquí se muestra cómo volver a implementar la plantilla de Azure Resource Manager para expandir un grupo de hosts:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En la barra de búsqueda situada en la parte superior de Azure Portal, busque **Grupos de recursos** y seleccione el elemento que aparece en **Servicios**.
3. Busque y seleccione el grupo de recursos que creó cuando hizo el grupo de hosts.
4. En el panel de la izquierda del explorador, seleccione **Implementaciones**.
5. Seleccione la implementación adecuada para el proceso de creación del grupo de hosts:
     - Si ha creado el grupo de hosts original con la oferta de Azure Marketplace, seleccione la implementación que empieza por **rds.wvd-provision-host-pool**.
     - Si ha creado el grupo de hosts original con la plantilla de Azure Resource Manager de GitHub, seleccione la implementación denominada **Microsoft.Template**.
6. Seleccione **Volver a implementar**.

     >[!NOTE]
     >Si la plantilla no se vuelve a implementar automáticamente al seleccionar **Volver a implementar**, seleccione **Plantilla** en el panel del lado izquierdo del explorador y, luego, seleccione **Implementar**.

7. Seleccione el grupo de recursos que contiene las máquinas virtuales de host de sesión actuales en el grupo de hosts existente.

     >[!NOTE]
     >Si ve un error que indica que seleccione un grupo de recursos diferente aunque el que especificó sea correcto, seleccione otro grupo de recursos y, luego, seleccione el grupo de recursos original.

8. Escriba la siguiente dirección URL para *_artifactsLocation*: `https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/`
9. Escriba el nuevo número total de hosts de sesión que desea incluir en *Rdsh Number Of Instances* (Número de instancias de RDSH). Por ejemplo, si va a expandir el grupo de hosts de cinco a ocho hosts de sesión, escriba **8**.
10. Escriba la misma contraseña de dominio existente que usó para el UPN de dominio existente. No cambie el nombre de usuario, ya que se producirá un error al ejecutar la plantilla.
11. Escriba la misma contraseña de administrador de inquilinos que usó con el identificador de usuario o aplicación que especificó en *Tenant Admin Upn Or Application Id* (UPN de administrador de inquilinos o identificador de aplicación). Una vez más, no cambie el nombre de usuario.
12. Complete el envío para expandir el grupo de hosts.

## <a name="run-the-azure-marketplace-offering"></a>Ejecución de la oferta de Azure Marketplace

Siga las instrucciones que se indican en [Creación de un grupo de hosts con Azure Marketplace](create-host-pools-azure-marketplace-2019.md) hasta que llegue a [Ejecución de la oferta de Azure Marketplace para aprovisionar un nuevo grupo de hosts](create-host-pools-azure-marketplace-2019.md#run-the-azure-marketplace-offering-to-provision-a-new-host-pool). Cuando llegue a ese punto, deberá escribir la siguiente información en cada pestaña:

### <a name="basics"></a>Aspectos básicos

Todos los valores de esta sección deben coincidir con lo que proporcionó cuando creó por primera vez el grupo de hosts y las máquinas virtuales de host de sesión, excepto los de *Usuarios de escritorio predeterminados*:

1.    En *Suscripción*, seleccione la suscripción en la que creó el grupo de hosts por primera vez.
2.    En *Grupo de recursos*, seleccione el mismo grupo de recursos donde se encuentran las máquinas virtuales de host de sesión del grupo de hosts existente.
3.    En *Región*, seleccione la misma región donde se encuentran las máquinas virtuales de host de sesión del grupo de hosts existente.
4.    En *Nombre del grupo de hosts*, escriba el nombre del grupo de hosts existente.
5.    En *Tipo de escritorio*, seleccione el tipo de escritorio que coincida con el grupo de hosts existente.
6.    En *Usuarios de escritorio predeterminados*, escriba una lista separada por comas de los usuarios adicionales que pueden iniciar sesión en los clientes de Windows Virtual Desktop y acceder a un escritorio una vez finalizada la oferta de Azure Marketplace. Por ejemplo, si quiere asignar acceso a user3@contoso.com y user4@contoso.com, escriba user3@contoso.com,user4@contoso.com.
7.    Seleccione **Siguiente: Configurar máquina virtual**.

>[!NOTE]
>Excepto *Usuarios de escritorio predeterminados*, todos los campos deben coincidir exactamente con lo que se ha configurado en el grupo de hosts existente. Si no coinciden, se producirá un nuevo grupo de hosts.

### <a name="configure-virtual-machines"></a>Configuración de máquinas virtuales

Todos los valores de parámetros de esta sección deben coincidir con lo que proporcionó cuando creó por primera vez el grupo de hosts y las máquinas virtuales de host de sesión, excepto el número total de máquinas virtuales. El número de máquinas virtuales que escriba será el número de máquinas virtuales en el grupo de hosts expandido:

1. Seleccione el tamaño de máquina virtual que coincida con las máquinas virtuales de host de sesión existentes.

    >[!NOTE]
    >Si el tamaño de máquina virtual específico que busca no aparece en el selector, es porque aún no se ha incorporado a la herramienta Azure Marketplace. Para solicitar un tamaño de máquina virtual, cree una solicitud o vote por una existente en el [foro de UserVoice de Windows Virtual Desktop](https://windowsvirtualdesktop.uservoice.com/forums/921118-general).

2. Personalice los parámetros *Perfil de uso*, *Usuarios totales* y *Número de máquinas virtuales* para seleccionar el número total de hosts de sesión que quiere tener en el grupo de hosts. Por ejemplo, si va a expandir el grupo de hosts de cinco a ocho hosts de sesión, configure estas opciones para llegar a 8 máquinas virtuales.
3. Escriba un prefijo para los nombres de las máquinas virtuales. Por ejemplo, si escribe el nombre "prefijo", las máquinas virtuales se llamarán "prefijo-0", "prefijo-1" y así sucesivamente.
4. Seleccione **Siguiente: Configuración de máquina virtual**.

### <a name="virtual-machine-settings"></a>Configuración de máquina virtual

Todos los valores de parámetros de esta sección deben coincidir con lo que proporcionó cuando creó por primera vez el grupo de hosts y las máquinas virtuales de host de sesión:

1. En *Origen de imagen* y *Image OS version* (Versión de SO de la imagen), escriba la misma información que proporcionó la primera vez que creó el grupo de hosts.
2. En *AD domain join UPN* (UPN de unión a un dominio de AD) y las contraseñas asociadas, escriba la misma información que proporcionó la primera vez que creó el grupo de hosts para unir las máquinas virtuales al dominio de Active Directory. Estas credenciales se usarán para crear una cuenta local en las máquinas virtuales. Puede restablecer estas cuentas locales más adelante para cambiar sus credenciales.
3. Para información sobre la red virtual, seleccione la misma red virtual y subred en la que se encuentran las máquinas virtuales de host de sesión del grupo de hosts existente.
4. Seleccione **Siguiente: Configure Windows Virtual Desktop information** (Configurar la información de Windows Virtual Desktop).

### <a name="windows-virtual-desktop-information"></a>Información de Windows Virtual Desktop

Todos los valores de parámetros de esta sección deben coincidir con lo que proporcionó cuando creó por primera vez el grupo de hosts y las máquinas virtuales de host de sesión:

1. En *Nombre de grupo de inquilinos de Windows Virtual Desktop*, escriba el nombre del grupo de inquilinos que contiene su inquilino. A menos que se le haya proporcionado un nombre de grupo de inquilinos específico, deje el valor predeterminado.
2. En *Nombre de inquilino de Windows Virtual Desktop*, escriba el nombre del inquilino donde se creará este grupo de hosts.
3. Especifique las mismas credenciales que usó cuando creó por primera vez el grupo de hosts y las máquinas virtuales de host de sesión. Si usa una entidad de servicio, escriba el identificador de la instancia de Azure Active Directory donde se encuentra la entidad de servicio.
4. Seleccione **Siguiente: Review + create** (Revisar y crear).

## <a name="run-the-github-azure-resource-manager-template"></a>Ejecución de la plantilla de Azure Resource Manager de GitHub

Siga las instrucciones que se indican en [Ejecutar la plantilla de Azure Resource Manager para el aprovisionamiento de un nuevo grupo de hosts](create-host-pools-arm-template.md#run-the-azure-resource-manager-template-for-provisioning-a-new-host-pool) y proporcione los mismos valores de parámetros, excepto para *Rdsh Number Of Instances* (Número de instancias de RDSH). Después de ejecutar la plantilla, escriba el número de máquinas virtuales de host de sesión que quiere en el grupo de hosts. Por ejemplo, si va a expandir el grupo de hosts de cinco a ocho hosts de sesión, escriba **8**.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha expandido el grupo de hosts existente, puede iniciar sesión en un cliente de Windows Virtual Desktop para realizar pruebas en él como parte de una sesión de usuario. Puede conectarse a una sesión con cualquiera de los siguientes clientes:

- [Conexión con el cliente de Escritorio de Windows](connect-windows-7-10-2019.md)
- [Conexión con el cliente web](connect-web-2019.md)
- [Conexión con el cliente de Android](connect-android-2019.md)
- [Conexión con el cliente macOS](connect-macos-2019.md)
- [Conexión con el cliente iOS](connect-ios-2019.md)
