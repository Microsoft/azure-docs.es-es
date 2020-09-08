---
title: 'Configuración del enrutamiento del tráfico de subred: Azure Traffic Manager'
description: En este artículo se explica cómo configurar Traffic Manager para enrutar el tráfico desde subredes concretas.
services: traffic-manager
documentationcenter: ''
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/17/2018
ms.author: duau
ms.openlocfilehash: b1901ddce2eb9c8ff5ec9ac90a56379e74c11aa6
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401374"
---
# <a name="direct-traffic-to-specific-endpoints-based-on-user-subnet-using-traffic-manager"></a>Dirección del tráfico a puntos de conexión específicos en función de la subred de usuario mediante Traffic Manager

En este artículo se describe cómo configurar el método de enrutamiento del tráfico en una subred. El método de enrutamiento de tráfico de **subred** permite asignar un conjunto de intervalos de direcciones IP a puntos de conexión específicos y cuando Traffic Manager recibe una solicitud, inspecciona la dirección IP de origen de la solicitud y devuelve el punto de conexión asociado a ella.

En el escenario que se trata en este artículo, con la subred de enrutamiento, según la dirección IP de la consulta del usuario, el tráfico se enruta a un sitio web interno o un sitio web de producción.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerequisites
Para ver a Traffic Manager en acción, este tutorial requiere que implemente lo siguiente:
- Dos sitios web básicos que se ejecutan en regiones de Azure diferentes: **Este de EE. UU.** (actúa como sitio web interno) y **Oeste de Europa** (actúa como sitio web de producción).
- Dos máquinas virtuales para probar Traffic Manager (una máquina virtual en **Este de EE. UU.** y la segunda máquina virtual en **Oeste de Europa**).

Las máquinas virtuales de prueba se usan para ilustrar cómo enruta Traffic Manager el tráfico de usuario al sitio web interno o al sitio web de producción según la subred desde donde se origina la consulta del usuario.

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

### <a name="create-websites"></a>Creación de sitios web

En esta sección, creará dos instancias de sitio web que proporcionan los puntos de conexión de servicio para el perfil de Traffic Manager en dos regiones de Azure. La creación de los dos sitios web incluye los pasos siguientes:
1. Crear dos máquinas virtuales para ejecutar un sitio web básico: una en **Este de EE. UU.** y otra en **Oeste de Europa**.
2. Instale un servidor IIS en cada máquina virtual y actualice la página del sitio web predeterminada que describe el nombre de la máquina virtual a la que un usuario se conecta cuando visita el sitio web.

#### <a name="create-vms-for-running-websites"></a>Creación de máquinas virtuales para ejecutar sitios web
En esta sección, creará dos máquinas virtuales *myEndpointVMEastUS* y *myEndpointVMWEurope* en las regiones de Azure **Este de EE. UU.** y **Oeste de Europa**.

1. En la esquina superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Compute** > **Windows Server 2016 VM** (VM de Windows Server 2016).
2. Escriba o seleccione la siguiente información para **Aspectos básicos**, acepte los valores predeterminados para el resto de la configuración y luego seleccione **Crear**:

    |Configuración|Value|
    |---|---|
    |Nombre|myIISVMEastUS|
    |Nombre de usuario| Escriba un nombre de usuario de su elección.|
    |Contraseña| Escriba una contraseña de su elección. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).|
    |Resource group| Seleccione **Nuevo** y luego escriba *myResourceGroupTM1*.|
    |Location| Seleccione **Este de EE. UU**.|
    |||

4. Seleccione un tamaño de máquina virtual en **Elegir un tamaño**.
5. Seleccione los valores siguientes en **Configuración** y, después, seleccione **Aceptar**:
    
    |Configuración|Value|
    |---|---|
    |Virtual network| Seleccione **Red virtual**, en **Crear red virtual**, para **Nombre**, escriba *myVNet1*, para cada subred, escriba *mySubnet*.|
    |Grupo de seguridad de red (NSG)|Seleccione **Básico** y, en la lista desplegable **Seleccionar puertos de entrada públicos**, seleccione **HTTP** y **RDP** |
    |Diagnósticos de arranque|Seleccione **Deshabilitado**.|
    |||

6. En **Crear** de la página **Resumen**, seleccione **Crear** para iniciar la implementación de la máquina virtual.

7. Complete de nuevo los pasos del 1 al 6, con los cambios siguientes:

    |Configuración|Value|
    |---|---|
    |Resource group | Seleccione **Nuevo** y, a continuación, escriba *myResourceGroupTM2*|
    |Location|Oeste de Europa|
    |Nombre de la máquina virtual | myIISVMWEurope|
    |Virtual network | Seleccione **Red virtual**, en **Crear red virtual**, para **Nombre**, escriba *myVNet2*, para cada subred, escriba *mySubnet*.|
    |||

8. Las máquinas virtuales tardan unos minutos en crearse. No siga con los pasos restantes hasta que se creen ambas máquinas virtuales.

   ![Crear una VM](./media/tutorial-traffic-manager-improve-website-response/createVM.png)

#### <a name="install-iis-and-customize-the-default-web-page"></a>Instalación de IIS y personalización de la página web predeterminada

En esta sección, instalará el servidor IIS en las dos máquinas virtuales (*myIISVMEastUS*  & *myIISVMWEurope*) y luego actualizará la página del sitio web predeterminado. La página del sitio web personalizado muestra el nombre de la máquina virtual a la que se está conectando cuando visita el sitio web desde un explorador web.

1. Seleccione **Todos los recursos** en el menú de la izquierda y, en la lista de recursos, haga clic en *myIISVMEastUS*, que se encuentra en el grupo de recursos *myResourceGroupTM1*.
2. En la página **Introducción**, haga clic en **Conectar** y luego, en **Conectarse a una máquina virtual**, seleccione **Descargar archivo RDP**.
3. Abra el archivo .rdp descargado. Cuando se le pida, seleccione **Conectar**. Escriba el nombre de usuario y la contraseña que especificó al crear la máquina virtual. Puede que deba seleccionar **More choices** (Más opciones) y, luego, **Use a different account** (Usar una cuenta diferente), para especificar las credenciales que escribió al crear la máquina virtual.
4. Seleccione **Aceptar**.
5. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Si recibe la advertencia, seleccione **Sí** o **Continuar** para continuar con la conexión.
6. En el escritorio del servidor, vaya a **Herramientas administrativas de Windows**>**Administrador del servidor** .
7. Inicie Windows PowerShell en *myIISVMEastUS* y use los siguientes comandos para instalar el servidor IIS y actualizar el archivo HTM predeterminado.
    ```powershell-interactive
    # Install IIS
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # Remove default htm file
    remove-item C:\inetpub\wwwroot\iisstart.htm
    
    #Add custom htm file
    Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from my test website server - " + $env:computername)
    ```
8. Cierre la conexión de RDP con *myIISVMEastUS*.
9. Repita los pasos 1-6 mediante la creación de una conexión RDP con la máquina virtual *myIISVMWEurope* dentro del grupo de recursos *myResourceGroupTM2* para instalar IIS y personalizar su página web predeterminada.
10. Inicie Windows PowerShell en *myIISVMWEurope* y use los siguientes comandos para instalar el servidor IIS y actualizar el archivo HTM predeterminado.
    ```powershell-interactive
    # Install IIS
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # Remove default htm file
    remove-item C:\inetpub\wwwroot\iisstart.htm
    
    #Add custom htm file
    Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from my production website server - " + $env:computername)
    ```

#### <a name="configure-dns-names-for-the-vms-running-iis"></a>Configuración de nombres DNS para las máquinas virtuales que ejecutan IIS

Traffic Manager enruta el tráfico de usuario basándose en el nombre DNS de los puntos de conexión de servicio. En esta sección, configurará los nombres DNS de los servidores IIS: *myIISVMEastUS* y *myIISVMWEurope*.

1. Haga clic en **Todos los recursos** en el menú de la izquierda y, en la lista de recursos, seleccione *myIISVMEastUS*, que se encuentra en el grupo de recursos *myResourceGroupTM1*.
2. En la página **Introducción**, en **Nombre DNS**, seleccione **Configurar**.
3. En la página **Configuración**, en la etiqueta de nombre DNS, agregue un nombre único y luego seleccione **Guardar**.
4. Repita los pasos 1-3, para la máquina virtual denominada *myIISVMWEurope* que se encuentra en el grupo de recursos *myResourceGroupTM1*.

### <a name="create-test-vms"></a>Creación de máquinas virtuales de prueba

En esta sección, creará una máquina virtual (*mVMEastUS* and *myVMWestEurope*) en cada región de Azure (**Este de EE. UU.** y **Oeste de Europa**. Usará estas máquinas virtuales para probar cómo Traffic Manager enruta el tráfico al servidor IIS más cercano al examinar el sitio web.

1. En la esquina superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Compute** > **Windows Server 2016 VM** (VM de Windows Server 2016).
2. Escriba o seleccione la siguiente información para **Aspectos básicos**, acepte los valores predeterminados para el resto de la configuración y luego seleccione **Crear**:

    |Configuración|Value|
    |---|---|
    |Nombre|myVMEastUS|
    |Nombre de usuario| Escriba un nombre de usuario de su elección.|
    |Contraseña| Escriba una contraseña de su elección. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).|
    |Resource group| Seleccione **Existente** y, a continuación, seleccione *myResourceGroupTM1*.|
    |||

4. Seleccione un tamaño de máquina virtual en **Elegir un tamaño**.
5. Seleccione los valores siguientes en **Configuración** y, después, seleccione **Aceptar**:

    |Configuración|Value|
    |---|---|
    |Virtual network| Seleccione **Red virtual**, en **Crear red virtual**; en **Nombre**, escriba *myVNet3*; para cada subred, escriba *mySubnet3*.|
    |Grupo de seguridad de red (NSG)|Seleccione **Básico** y, en la lista desplegable **Seleccionar puertos de entrada públicos**, seleccione **HTTP** y **RDP** |
    |Diagnósticos de arranque|Seleccione **Deshabilitado**.|
    |||

6. En **Crear** de la página **Resumen**, seleccione **Crear** para iniciar la implementación de la máquina virtual.

7. Complete de nuevo los pasos del 1 al 5, con los cambios siguientes:

    |Configuración|Value|
    |---|---|
    |Nombre de la máquina virtual | *myVMWEurope*|
    |Resource group | Seleccione **Existente** y escriba *myResourceGroupTM2*.|
    |Virtual network | Seleccione **Red virtual**, en **Crear red virtual**; en **Nombre**, escriba *myVNet4*; para cada subred, escriba *mySubnet4*.|
    |||

8. Las máquinas virtuales tardan unos minutos en crearse. No siga con los pasos restantes hasta que se creen ambas máquinas virtuales.

## <a name="create-a-traffic-manager-profile"></a>Crear un perfil de Traffic Manager
Cree un perfil de Traffic Manager que le permita devolver puntos de conexión específicos según la dirección IP de origen de la solicitud.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** > **Redes** > **Perfil de Traffic Manager** > **Crear**.
2. En **Crear perfil de Traffic Manager**, escriba o seleccione la siguiente información, acepte los valores predeterminados para el resto de la configuración y, a continuación, seleccione **Crear**:

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Nombre                   | Este nombre debe ser único en la zona trafficmanager.net y generará el nombre DNS, trafficmanager.net, que se usa para acceder al perfil de Traffic Manager.                                   |
    | Método de enrutamiento          | Seleccione el método de enrutamiento **Subred**.                                       |
    | Subscription            | Seleccione su suscripción.                          |
    | Resource group          | Haga clic en **Existente** y escriba *myResourceGroupTM1*. |
    | |                              |
    |

    ![Crear un perfil de Traffic Manager](./media/traffic-manager-subnet-routing-method/create-traffic-manager-profile.png)

## <a name="add-traffic-manager-endpoints"></a>Incorporación de puntos de conexión de Traffic Manager

Agregue las dos máquinas virtuales que se ejecuten en los servidores IIS (*myIISVMEastUS* & *myIISVMWEurope*) para enrutar el tráfico de usuario en función de la subred de la consulta del usuario.

1. En la barra de búsqueda del portal, busque el nombre del perfil de Traffic Manager que creó en la sección anterior y seleccione el perfil en los resultados que aparecen.
2. En **perfil de Traffic Manager**, en la sección **Configuración**, haga clic en **Puntos de conexión** y, a continuación, haga clic en **Agregar**.
3. Escriba o seleccione la siguiente información, acepte los valores predeterminados para el resto de la configuración y luego seleccione **Aceptar**:

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Tipo                    | Punto de conexión de Azure                                   |
    | Nombre           | myTestWebSiteEndpoint                                        |
    | Tipo de recurso de destino           | Dirección IP pública                          |
    | Recurso de destino          | **Elija una dirección IP pública** para mostrar la lista de recursos con direcciones IP públicas en la misma suscripción. En **Recurso**, seleccione la dirección IP pública denominada *myIISVMEastUS-ip*. Se trata de la dirección IP pública de la máquina virtual del servidor IIS en la región Este de EE. UU.|
    |  Configuración de enrutamiento de subred    |   Agregue la dirección IP de la máquina virtual de prueba *myVMEastUS*. Cualquier consulta de usuario que se origine en esta máquina virtual se dirigirá a *myTestWebSiteEndpoint*.    |

4. Repita los pasos 2 y 3 para agregar otro punto de conexión denominado *myProductionEndpoint* para la dirección IP pública *myIISVMWEurope-ip* que está asociada la máquina virtual del servidor IIS denominada *myIISVMWEurope*. En **Configuración de enrutamiento de subred**, agregue la dirección IP de la máquina virtual de prueba *myVMWestEurope*. Cualquier consulta de usuario de esta máquina virtual de prueba se enrutará al punto de conexión *myProductionWebsiteEndpoint*.
5. Cuando termine de agregar ambos puntos de conexión, aparecerán en **Perfil de Traffic Manager** junto con el estado de supervisión como **En línea**.

    ![Incorporación de un punto de conexión de Traffic Manager](./media/traffic-manager-subnet-routing-method/customize-endpoint-with-subnet-routing-eastus.png)

## <a name="test-traffic-manager-profile"></a>Prueba del perfil de Traffic Manager
En esta sección, probará cómo Traffic Manager enruta el tráfico de usuario desde una subred determinada a un punto de conexión específico. Para ver a Traffic Manager en acción, complete los pasos siguientes:
1. Determine el nombre DNS del perfil de Traffic Manager.
2. Vea a Traffic Manager en acción de la siguiente manera:
    - Desde la máquina virtual de prueba (*myVMEastUS*) que se encuentra en la región **Este de EE. UU.** , en un explorador web, busque el nombre DNS del perfil de Traffic Manager.
    - Desde la máquina virtual de prueba (*myVMEastUS*) que se encuentra en la región **Oeste de Europa**, en un explorador web, busque el nombre DNS del perfil de Traffic Manager.

### <a name="determine-dns-name-of-traffic-manager-profile"></a>Determinación del nombre DNS del perfil de Traffic Manager
En este tutorial, por simplicidad, usará el nombre DNS del perfil de Traffic Manager para visitar los sitios web.

Puede determinar el nombre DNS del perfil de Traffic Manager de la siguiente manera:

1. En la barra de búsqueda del portal, busque el nombre del **perfil de Traffic Manager** que creó en la sección anterior. Haga clic en el perfil de Traffic Manager en los resultados que aparezcan.
1. Haga clic en **Descripción general**.
2. La hoja **Perfil de Traffic Manager** muestra el nombre DNS del perfil de Traffic Manager que acaba de crear. En las implementaciones de producción, puede configurar un nombre de dominio mnemónico para que apunte al nombre de dominio de Traffic Manager, mediante un registro CNAME de DNS.

   ![Nombre DNS de Traffic Manager](./media/traffic-manager-subnet-routing-method/traffic-manager-dns-name.png)

### <a name="view-traffic-manager-in-action"></a>Ver a Traffic Manager en acción
En esta sección, puede ver a Traffic Manager en acción.

1. Seleccione **Todos los recursos** en el menú de la izquierda y, en la lista de recursos, haga clic en *myVMEastUS*, que se encuentra en el grupo de recursos *myResourceGroupTM1*.
2. En la página **Introducción**, haga clic en **Conectar** y luego, en **Conectarse a una máquina virtual**, seleccione **Descargar archivo RDP**.
3. Abra el archivo .rdp descargado. Cuando se le pida, seleccione **Conectar**. Escriba el nombre de usuario y la contraseña que especificó al crear la máquina virtual. Puede que deba seleccionar **More choices** (Más opciones) y, luego, **Use a different account** (Usar una cuenta diferente), para especificar las credenciales que escribió al crear la máquina virtual.
4. Seleccione **Aceptar**.
5. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Si recibe la advertencia, seleccione **Sí** o **Continuar** para continuar con la conexión.
1. En un explorador web, escriba la máquina virtual *myVMEastUS*, escriba el nombre DNS del perfil de Traffic Manager para ver el sitio web. Dado que la dirección IP de la máquina virtual *myVMEastUS* está asociada al punto de conexión *myIISVMEastUS*, el explorador web inicia el servidor del sitio web de prueba *myIISVMEastUS*.

   ![Prueba del perfil de Traffic Manager](./media/traffic-manager-subnet-routing-method/test-traffic-manager.png)

2. A continuación, conéctese a la máquina virtual *myVMWestEurope* ubicada en **Oeste de Europa** mediante los pasos 1-5 y vaya al nombre de dominio del perfil de Traffic Manager de esta máquina virtual. Dado que la dirección IP de la máquina virtual *myVMWestEurope* está asociada al punto de conexión *myIISVMEastUS*, el explorador web inicia el servidor del sitio web de prueba *myIISVMWEurope*.

## <a name="delete-the-traffic-manager-profile"></a>Eliminar el perfil de Traffic Manager
Cuando ya no los necesite, elimine los grupos de recursos (**ResourceGroupTM1** y **ResourceGroupTM2**). Para ello, seleccione el grupo de recursos (**ResourceGroupTM1** o **ResourceGroupTM2**) y luego seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

- Información sobre el [método de enrutamiento de tráfico ponderado](traffic-manager-configure-weighted-routing-method.md).
- Sepa cómo funciona el [método de enrutamiento por prioridad](traffic-manager-configure-priority-routing-method.md).
- Información sobre el [método de enrutamiento geográfico](traffic-manager-configure-geographic-routing-method.md).
