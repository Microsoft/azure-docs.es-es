---
title: 'Tutorial: Mejora de la respuesta de un sitio web con Azure Traffic Manager'
description: Este artículo del tutorial describe cómo crear un perfil de Traffic Manager para crear un sitio web con alta capacidad de respuesta.
services: traffic-manager
author: duongau
Customer intent: As an IT Admin, I want to route traffic so I can improve website response by choosing the endpoint with lowest latency.
ms.service: traffic-manager
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/23/2018
ms.author: duau
ms.openlocfilehash: e79dde0178c74965bde14956203e31e82e80d93e
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89398416"
---
# <a name="tutorial-improve-website-response-using-traffic-manager"></a>Tutorial: Mejorar la respuesta del sitio web mediante Traffic Manager

En este tutorial se describe cómo usar Traffic Manager para crear un sitio web con alta capacidad de respuesta dirigiendo el tráfico del usuario al sitio web con la latencia más baja. Normalmente, el centro de datos con la latencia más baja es aquel más cercano desde el punto de vista geográfico.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear dos máquinas virtuales que ejecutan un sitio web básico en IIS
> * Crear dos máquinas virtuales de prueba para ver a Traffic Manager en acción
> * Configurar el nombre DNS para las máquinas virtuales que ejecutan IIS
> * Crear un perfil de Traffic Manager para un rendimiento mejorado del sitio web
> * Agregar puntos de conexión de máquina virtual en el perfil de Traffic Manager
> * Ver a Traffic Manager en acción

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para ver a Traffic Manager en acción, este tutorial requiere que implemente lo siguiente:

- Dos instancias de sitios web básicos que se ejecuten en diferentes regiones de Azure: **Este de EE. UU.** y **Oeste de Europa**.
- Dos máquinas virtuales para probar Traffic Manager (una máquina virtual en **Este de EE. UU.** y la segunda máquina virtual en **Oeste de Europa**). Las máquinas virtuales de prueba se usan para ilustrar cómo Traffic Manager enruta el tráfico de usuario al sitio web que se está ejecutando en la misma región, ya que proporciona la latencia más baja.

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

### <a name="create-websites"></a>Creación de sitios web

En esta sección, creará dos instancias de sitio web que proporcionan los puntos de conexión de servicio para el perfil de Traffic Manager en dos regiones de Azure. La creación de los dos sitios web incluye los pasos siguientes:

1. Crear dos máquinas virtuales para ejecutar un sitio web básico: una en **Este de EE. UU.** y otra en **Oeste de Europa**.
2. Instale un servidor IIS en cada máquina virtual y actualice la página del sitio web predeterminada que describe el nombre de la máquina virtual a la que un usuario se conecta cuando visita el sitio web.

#### <a name="create-vms-for-running-websites"></a>Creación de máquinas virtuales para ejecutar sitios web

En esta sección, creará dos máquinas virtuales *myIISVMEastUS* y *myIISVMWestEurope* en las regiones de Azure **Este de EE. UU.** y **Oeste de Europa**.

1. En la esquina superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Proceso** > **Windows Server 2019 Datacenter**.
2. En **Crear una máquina virtual**, escriba o seleccione los valores siguientes en la pestaña **Básico**:

   - **Suscripción** > **Grupo de recursos**: Seleccione **Crear nuevo** y, a continuación, escriba **myResourceGroupTM1**.
   - **Detalles de instancia** > **Nombre de máquina virtual**: Escriba *myIISVMEastUS*.
   - **Detalles de instancia** > **Región**:  Seleccione **Este de EE. UU**.
   - **Cuenta de administrador** > **Nombre de usuario**:  Escriba un nombre de usuario de su elección.
   - **Cuenta de administrador** > **Contraseña**:  Escriba una contraseña de su elección. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).
   - **Reglas de puerto de entrada** > **Puertos de entrada públicos**: Seleccione **Permitir los puertos seleccionados**.
   - **Reglas de puerto de entrada** > **Seleccionar puertos de entrada**: Seleccione **RDP** y **HTTP** en el cuadro desplegable.

3. Seleccione la pestaña **Administración** o **Siguiente: Discos** y, después, **Siguiente: Redes**, a continuación, **Siguiente: Administración**. En **Supervisión**, establezca **Diagnósticos de arranque** en **Desactivado**.
4. Seleccione **Revisar + crear**.
5. Revise la configuración y, luego, haga clic en **Crear**.  
6. Siga los pasos para crear una segunda máquina virtual denominada *myIISVMWestEurope*, con un **grupo de recursos** llamado *myResourceGroupTM2*, una **ubicación** de *Oeste de Europa* y todos los demás valores iguales a los de *myIISVMEastUS*.
7. Las máquinas virtuales tardan unos minutos en crearse. No siga con los pasos restantes hasta que se creen ambas máquinas virtuales.

   ![Crear una VM](./media/tutorial-traffic-manager-improve-website-response/createVM.png)

#### <a name="install-iis-and-customize-the-default-web-page"></a>Instalación de IIS y personalización de la página web predeterminada

En esta sección, instalará el servidor IIS en las dos máquinas virtuales (*myIISVMEastUS* y *myIISVMWestEurope*) y luego actualizará la página web predeterminada. La página del sitio web personalizado muestra el nombre de la máquina virtual a la que se está conectando cuando visita el sitio web desde un explorador web.

1. Seleccione **Todos los recursos** en el menú de la izquierda y, en la lista de recursos, haga clic en *myIISVMEastUS*, que se encuentra en el grupo de recursos *myResourceGroupTM1*.
2. En la página **Introducción**, haga clic en **Conectar** y luego, en **Conectarse a una máquina virtual**, seleccione **Descargar archivo RDP**.
3. Abra el archivo .rdp descargado. Cuando se le pida, seleccione **Conectar**. Escriba el nombre de usuario y la contraseña que especificó al crear la máquina virtual. Puede que deba seleccionar **More choices** (Más opciones) y, luego, **Use a different account** (Usar una cuenta diferente), para especificar las credenciales que escribió al crear la máquina virtual.
4. Seleccione **Aceptar**.
5. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Si recibe la advertencia, seleccione **Sí** o **Continuar** para continuar con la conexión.
6. En el escritorio del servidor, vaya a **Herramientas administrativas de Windows**>**Administrador del servidor** .
7. Inicie Windows PowerShell en VM1 y use los siguientes comandos para instalar el servidor IIS y actualizar el archivo HTM predeterminado.

    ```powershell-interactive
    # Install IIS
    Install-WindowsFeature -name Web-Server -IncludeManagementTools

    # Remove default htm file
    remove-item C:\inetpub\wwwroot\iisstart.htm

    #Add custom htm file
    Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)
    ```

     ![Instalación de IIS y personalización de la página web](./media/tutorial-traffic-manager-improve-website-response/deployiis.png)
8. Cierre la conexión de RDP con *myIISVMEastUS*.
9. Repita los pasos 1-8 mediante la creación de una conexión RDP con la máquina virtual *myIISVMWestEurope* dentro del grupo de recursos *myResourceGroupTM2* para instalar IIS y personalizar su página web predeterminada.

#### <a name="configure-dns-names-for-the-vms-running-iis"></a>Configuración de nombres DNS para las máquinas virtuales que ejecutan IIS

Traffic Manager enruta el tráfico de usuario basándose en el nombre DNS de los puntos de conexión de servicio. En esta sección, configurará los nombres DNS de los servidores IIS: *myIISVMEastUS* y *myIISVMWestEurope*.

1. Haga clic en **Todos los recursos** en el menú de la izquierda y, en la lista de recursos, seleccione *myIISVMEastUS*, que se encuentra en el grupo de recursos *myResourceGroupTM1*.
2. En la página **Introducción**, en **Nombre DNS**, seleccione **Configurar**.
3. En la página **Configuración**, en la etiqueta de nombre DNS, agregue un nombre único y luego seleccione **Guardar**.
4. Repita los pasos 1-3, para la máquina virtual denominada *myIISVMWestEurope* que se encuentra en el grupo de recursos *myResourceGroupTM2*.

### <a name="create-test-vms"></a>Creación de máquinas virtuales de prueba

En esta sección, creará una máquina virtual (*myVMEastUS* y *myVMWestEurope*) en cada región de Azure (**Este de EE. UU.** y **Oeste de Europa**). Usará estas máquinas virtuales para probar cómo Traffic Manager enruta el tráfico al servidor IIS más cercano al examinar el sitio web.

1. En la esquina superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Proceso** > **Windows Server 2019 Datacenter**.
2. En **Crear una máquina virtual**, escriba o seleccione los valores siguientes en la pestaña **Básico**:

   - **Suscripción** > **Grupo de recursos**: Seleccione **myResourceGroupTM1**.
   - **Detalles de instancia** > **Nombre de máquina virtual**: Escriba *myVMEastUS*.
   - **Detalles de instancia** > **Región**:  Seleccione **Este de EE. UU**.
   - **Cuenta de administrador** > **Nombre de usuario**:  Escriba un nombre de usuario de su elección.
   - **Cuenta de administrador** > **Contraseña**:  Escriba una contraseña de su elección. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).
   - **Reglas de puerto de entrada** > **Puertos de entrada públicos**: Seleccione **Permitir los puertos seleccionados**.
   - **Reglas de puerto de entrada** > **Seleccionar puertos de entrada**: Seleccione **RDP** en el cuadro desplegable.

3. Seleccione la pestaña **Administración** o **Siguiente: Discos** y, después, **Siguiente: Redes**, a continuación, **Siguiente: Administración**. En **Supervisión**, establezca **Diagnósticos de arranque** en **Desactivado**.
4. Seleccione **Revisar + crear**.
5. Revise la configuración y, luego, haga clic en **Crear**.  
6. Siga los pasos para crear una segunda máquina virtual denominada *myVMWestEurope*, con un **grupo de recursos** llamado *myResourceGroupTM2*, una **ubicación** de *Oeste de Europa* y todos los demás valores iguales a los de *myVMEastUS*.
7. Las máquinas virtuales tardan unos minutos en crearse. No siga con los pasos restantes hasta que se creen ambas máquinas virtuales.

## <a name="create-a-traffic-manager-profile"></a>Crear un perfil de Traffic Manager

Cree un perfil de Traffic Manager que dirija el tráfico de usuario mediante el envío al punto de conexión con la latencia más baja.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** > **Redes** > **Perfil de Traffic Manager** > **Crear**.
2. En **Crear perfil de Traffic Manager**, escriba o seleccione la siguiente información, acepte los valores predeterminados para el resto de la configuración y, a continuación, seleccione **Crear**:

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Nombre                   | Este nombre debe ser único en la zona trafficmanager.net y generará el nombre DNS, trafficmanager.net, que se usa para acceder al perfil de Traffic Manager.                                   |
    | Método de enrutamiento          | Seleccione el método de enrutamiento de **rendimiento**.                                       |
    | Subscription            | Seleccione su suscripción.                          |
    | Resource group          | Seleccione el grupo de recursos *myResourceGroupTM1*. |
    | Location                | Seleccione **Este de EE. UU**. Esta configuración se refiere a la ubicación del grupo de recursos y no tiene efecto alguno sobre el perfil de Traffic Manager que se implementará globalmente.                              |
    |

    ![Crear un perfil de Traffic Manager](./media/tutorial-traffic-manager-improve-website-response/traffic-manager-profile.png)

## <a name="add-traffic-manager-endpoints"></a>Incorporación de puntos de conexión de Traffic Manager

Agregue las dos máquinas virtuales que ejecutan los servidores IIS (*myIISVMEastUS* & *myIISVMWestEurope*) para enrutar el tráfico de usuario al punto de conexión más cercano al usuario.

1. En la barra de búsqueda del portal, busque el nombre del perfil de Traffic Manager que creó en la sección anterior y seleccione el perfil en los resultados que aparecen.
2. En **perfil de Traffic Manager**, en la sección **Configuración**, haga clic en **Puntos de conexión** y, a continuación, haga clic en **Agregar**.
3. Escriba o seleccione la siguiente información, acepte los valores predeterminados para el resto de la configuración y luego seleccione **Aceptar**:

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Tipo                    | Punto de conexión de Azure                                   |
    | Nombre           | myEastUSEndpoint                                        |
    | Tipo de recurso de destino           | Dirección IP pública                          |
    | Recurso de destino          | **Elija una dirección IP pública** para mostrar la lista de recursos con direcciones IP públicas en la misma suscripción. En **Recurso**, seleccione la dirección IP pública denominada *myIISVMEastUS-ip*. Se trata de la dirección IP pública de la máquina virtual del servidor IIS en la región Este de EE. UU.|
    |        |           |

4. Repita los pasos 2 y 3 para agregar otro punto de conexión denominado *myWestEuropeEndpoint* para la dirección IP pública *myIISVMWestEurope-ip* que está asociada la máquina virtual del servidor IIS denominada *myIISVMWestEurope*.
5. Cuando termine de agregar ambos puntos de conexión, aparecerán en **Perfil de Traffic Manager** junto con el estado de supervisión como **En línea**.

    ![Incorporación de un punto de conexión de Traffic Manager](./media/tutorial-traffic-manager-improve-website-response/traffic-manager-endpoint.png)

## <a name="test-traffic-manager-profile"></a>Prueba del perfil de Traffic Manager

En esta sección, probará cómo Traffic Manager enruta el tráfico de usuario a las máquinas virtuales más cercanas que ejecutan el sitio web para ofrecer una latencia mínima. Para ver a Traffic Manager en acción, complete los pasos siguientes:

1. Determine el nombre DNS del perfil de Traffic Manager.
2. Vea a Traffic Manager en acción de la siguiente manera:
    - Desde la máquina virtual de prueba (*myVMEastUS*) que se encuentra en la región **Este de EE. UU.** , en un explorador web, busque el nombre DNS del perfil de Traffic Manager.
    - Desde la máquina virtual de prueba (*myVMWestEurope*) que se encuentra en la región **Oeste de Europa**, en un explorador web, busque el nombre DNS del perfil de Traffic Manager.

### <a name="determine-dns-name-of-traffic-manager-profile"></a>Determinación del nombre DNS del perfil de Traffic Manager

En este tutorial, por simplicidad, usará el nombre DNS del perfil de Traffic Manager para visitar los sitios web.

Puede determinar el nombre DNS del perfil de Traffic Manager de la siguiente manera:

1. En la barra de búsqueda del portal, busque el nombre del **perfil de Traffic Manager** que creó en la sección anterior. Haga clic en el perfil de Traffic Manager en los resultados que aparezcan.
2. Haga clic en **Descripción general**.
3. La hoja **Perfil de Traffic Manager** muestra el nombre DNS del perfil de Traffic Manager que acaba de crear. En las implementaciones de producción, puede configurar un nombre de dominio mnemónico para que apunte al nombre de dominio de Traffic Manager, mediante un registro CNAME de DNS.

   ![Nombre DNS de Traffic Manager](./media/tutorial-traffic-manager-improve-website-response/traffic-manager-dns-name.png)

### <a name="view-traffic-manager-in-action"></a>Ver a Traffic Manager en acción

En esta sección, puede ver a Traffic Manager en acción.

1. Seleccione **Todos los recursos** en el menú de la izquierda y, en la lista de recursos, haga clic en *myVMEastUS*, que se encuentra en el grupo de recursos *myResourceGroupTM1*.
2. En la página **Introducción**, haga clic en **Conectar** y luego, en **Conectarse a una máquina virtual**, seleccione **Descargar archivo RDP**.
3. Abra el archivo .rdp descargado. Cuando se le pida, seleccione **Conectar**. Escriba el nombre de usuario y la contraseña que especificó al crear la máquina virtual. Puede que deba seleccionar **More choices** (Más opciones) y, luego, **Use a different account** (Usar una cuenta diferente), para especificar las credenciales que escribió al crear la máquina virtual.
4. Seleccione **Aceptar**.
5. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Si recibe la advertencia, seleccione **Sí** o **Continuar** para continuar con la conexión.
1. En un explorador web, escriba la máquina virtual *myVMEastUS*, escriba el nombre DNS del perfil de Traffic Manager para ver el sitio web. Puesto que la máquina virtual se encuentra en **Este de EE. UU.** , se le dirigirá al sitio web más cercano hospedado en el servidor IIS más cercano *myIISVMEastUS* que se encuentra en **Este de EE. UU.** .

   ![Prueba del perfil de Traffic Manager](./media/tutorial-traffic-manager-improve-website-response/eastus-traffic-manager-test.png)

2. A continuación, conéctese a la máquina virtual *myVMWestEurope* ubicada en **Oeste de Europa** mediante los pasos 1-5 y vaya al nombre de dominio del perfil de Traffic Manager de esta máquina virtual. Puesto que la máquina virtual se encuentra en **Oeste de Europa**, ahora se le enruta al sitio web hospedado en el servidor IIS más cercano *myIISVMWestEurope* que se encuentra en **Oeste de Europa**.

   ![Prueba del perfil de Traffic Manager](./media/tutorial-traffic-manager-improve-website-response/westeurope-traffic-manager-test.png)

## <a name="delete-the-traffic-manager-profile"></a>Eliminar el perfil de Traffic Manager

Cuando ya no los necesite, elimine los grupos de recursos (**ResourceGroupTM1** y **ResourceGroupTM2**). Para ello, seleccione el grupo de recursos (**ResourceGroupTM1** o **ResourceGroupTM2**) y luego seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Distribución del tráfico a un conjunto de puntos de conexión](traffic-manager-configure-weighted-routing-method.md)
