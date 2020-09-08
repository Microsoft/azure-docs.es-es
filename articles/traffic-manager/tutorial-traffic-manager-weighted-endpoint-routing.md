---
title: 'Tutorial: Enrutamiento del tráfico a puntos de conexión ponderados (Azure Traffic Manager)'
description: En este artículo tutorial se describe cómo enrutar el tráfico a puntos de conexión ponderados con Traffic Manager.
services: traffic-manager
author: duongau
Customer intent: As an IT Admin, I want to distribute traffic based on the weight assigned to a website endpoint so that I can control the user traffic to a given website.
ms.service: traffic-manager
ms.topic: tutorial
ms.date: 10/15/2018
ms.author: duau
ms.openlocfilehash: 90ed68e36b47d46c47e78407fac3b5fd74924b57
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89397090"
---
# <a name="tutorial-control-traffic-routing-with-weighted-endpoints-by-using-traffic-manager"></a>Tutorial: Control del enrutamiento del tráfico con puntos de conexión ponderados mediante Traffic Manager

En este tutorial se describe cómo usar Azure Traffic Manager para controlar el enrutamiento del tráfico de usuario entre los puntos de conexión mediante el método de enrutamiento ponderado. En este método de enrutamiento, a cada punto de conexión se le asigna un peso en la configuración de perfiles de Traffic Manager. El tráfico de usuario se enruta en función del peso asignado a cada punto de conexión. El peso es un entero comprendido entre 1 y 1000. Cuanto mayor sea el valor del peso asignado a un punto de conexión, más alta será su prioridad.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear dos máquinas virtuales que ejecutan un sitio web básico en IIS.
> * Crear dos máquinas virtuales de prueba para ver a Traffic Manager en acción.
> * Configurar un nombre DNS para las máquinas virtuales que ejecutan IIS.
> * Crear un perfil de Traffic Manager.
> * Agregar puntos de conexión de máquina virtual al perfil de Traffic Manager.
> * Vea a Traffic Manager en acción.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para ver a Traffic Manager en acción, implemente los siguientes elementos para este tutorial:

- Dos instancias de sitios web básicos que se ejecuten en diferentes regiones de Azure: Este de EE. UU. y Oeste de Europa.
- Dos máquinas virtuales de prueba para probar Traffic Manager: una en Este de EE. UU. y la segunda en Oeste de Europa. Las máquinas virtuales de prueba se usan para ilustrar cómo Traffic Manager enruta el tráfico de usuario a un sitio web que tiene un peso mayor asignado a su punto de conexión.

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

### <a name="create-websites"></a>Creación de sitios web

En esta sección, creará dos instancias de sitio web que proporcionan los puntos de conexión de servicio para el perfil de Traffic Manager en dos regiones de Azure. Para crear los dos sitios web, realice los siguientes pasos:

1. Cree dos máquinas virtuales para ejecutar un sitio web básico: una en Este de EE. UU. y otra en Oeste de Europa.
2. Instale un servidor IIS en cada máquina virtual. Actualice la página predeterminada del sitio web que describe el nombre de la máquina virtual a la que un usuario se conecta cuando visita el sitio web.

#### <a name="create-vms-for-running-websites"></a>Creación de máquinas virtuales para ejecutar sitios web

En esta sección, creará dos máquinas virtuales (*myIISVMEastUS* y *myIISVMWEurope*) en las regiones de Azure Este de EE. UU. y Oeste de Europa.

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

#### <a name="install-iis-and-customize-the-default-webpage"></a>Instalación de IIS y personalización de la página web predeterminada

En esta sección, instalará el servidor IIS en las dos máquinas virtuales (myIISVMEastUS y myIISVMWestEurope) y luego actualizará la página web predeterminada. La página web personalizada muestra el nombre de la máquina virtual a la que se conecta cuando visita el sitio web desde un explorador web.

1. Seleccione **Todos los recursos** en el menú izquierdo. En la lista de recursos, seleccione **myIISVMEastUS** en el grupo de recursos **myResourceGroupTM1**.
2. En la página **Información general**, seleccione **Conectar**. En **Conectar a máquina virtual**, seleccione **Descargar archivo RDP**.
3. Abra el archivo .rdp descargado. Si se le pide, seleccione **Conectar**. Escriba el nombre de usuario y la contraseña que especificó cuando creó la máquina virtual. Puede que deba seleccionar **Más opciones** > **Usar otra cuenta** para especificar las credenciales que escribió al crear la máquina virtual.
4. Seleccione **Aceptar**.
5. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Si recibe la advertencia, seleccione **Sí** o **Continuar** para continuar con la conexión.
6. En el escritorio del servidor, vaya a **Herramientas administrativas de Windows** > **Administrador del servidor** .
7. Abra Windows PowerShell en la máquina virtual VM1. Use los siguientes comandos para instalar el servidor IIS y actualizar el archivo .htm predeterminado.

    ```powershell-interactive
    # Install IIS
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # Remove default .htm file
    remove-item C:\inetpub\wwwroot\iisstart.htm
    
    #Add custom .htm file
    Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)
    ```

    ![Instalación de IIS y personalización de la página web](./media/tutorial-traffic-manager-improve-website-response/deployiis.png)

8. Cierre la conexión de RDP con **myIISVMEastUS**.
9. Repita los pasos 1 a 8. Cree una conexión RDP con la máquina virtual **myIISVMWestEurope** en el grupo de recursos **myResourceGroupTM2** para instalar IIS y personalizar su página web predeterminada.

#### <a name="configure-dns-names-for-the-vms-running-iis"></a>Configuración de nombres DNS para las máquinas virtuales que ejecutan IIS

Traffic Manager enruta el tráfico de usuario en función del nombre DNS de los puntos de conexión de servicio. En esta sección, configurará los nombres DNS de los servidores IIS myIISVMEastUS y myIISVMWestEurope.

1. Seleccione **Todos los recursos** en el menú izquierdo. En la lista de recursos, seleccione **myIISVMEastUS** en el grupo de recursos **myResourceGroupTM1**.
2. En la página **Introducción**, en **Nombre DNS**, seleccione **Configurar**.
3. En la página **Configuración**, en la etiqueta de nombre DNS, agregue un nombre único. Después, seleccione **Guardar**.
4. Repita los pasos 1 a 3 para la máquina virtual llamada **myIISVMWestEurope** que se encuentra en el grupo de recursos **myResourceGroupTM2**.

### <a name="create-a-test-vm"></a>Creación de una máquina virtual de prueba

En esta sección, creará una máquina virtual (*myVMEastUS* y *myVMWestEurope*) en cada región de Azure (**Este de EE. UU.** y **Oeste de Europa**). Usará estas máquinas virtuales para probar cómo Traffic Manager enruta el tráfico al punto de conexión del sitio web que tiene el valor de peso más alto.

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

Cree un perfil de Traffic Manager basado en el método de enrutamiento **Ponderado**.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** > **Redes** > **Perfil de Traffic Manager** > **Crear**.
2. En **Crear perfil de Traffic Manager**, escriba o seleccione la siguiente información. Para las demás opciones, conserve los valores predeterminados y, a continuación, seleccione **Crear**.

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Nombre                   | Escriba un nombre único en la zona trafficmanager.net. Se produce el nombre DNS trafficmanager.net, que se usa para acceder al perfil de Traffic Manager.                                   |
    | Método de enrutamiento          | Seleccione el método de enrutamiento **Ponderado**.                                       |
    | Subscription            | Seleccione su suscripción.                          |
    | Resource group          | Seleccione **Usar existente** y después seleccione **myResourceGroupTM1**. |
    |        |   |

    ![Crear un perfil de Traffic Manager](./media/tutorial-traffic-manager-weighted-endpoint-routing/create-traffic-manager-profile.png)

## <a name="add-traffic-manager-endpoints"></a>Incorporación de puntos de conexión de Traffic Manager

Agregue las dos máquinas virtuales que ejecutan los servidores IIS, myIISVMEastUS y myIISVMWestEurope, para enrutar a ellas el tráfico de usuario.

1. En la barra de búsqueda del portal, busque el nombre del perfil de Traffic Manager que creó en la sección anterior. Seleccione el perfil en los resultados que se muestran.
2. En **Perfil de Traffic Manager**, en la sección **Configuración**, seleccione **Puntos de conexión** > **Agregar**.
3. Escriba o seleccione la siguiente información. Para las demás opciones, conserve los valores predeterminados y, a continuación, seleccione **Aceptar**.

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Tipo                    | Escriba el punto de conexión de Azure.                                   |
    | Nombre           | Escriba **myEastUSEndpoint**.                                        |
    | Tipo de recurso de destino           | Seleccione **Dirección IP pública**.                          |
    | Recurso de destino          | Elija una dirección IP pública para mostrar la lista de recursos con direcciones IP públicas en la misma suscripción. En **Recurso**, seleccione la dirección IP pública denominada **myIISVMEastUS-ip**. Se trata de la dirección IP pública de la máquina virtual del servidor IIS en la región Este de EE. UU.|
    |  Peso      | Escriba **100**.        |
    |        |           |

4. Repita los pasos 2 y 3 para agregar otro punto de conexión llamado **myWestEuropeEndpoint** para la dirección IP pública **myIISVMWestEurope-ip**. Esta dirección está asociada con la máquina virtual del servidor IIS llamada myIISVMWestEurope. En **Peso**, escriba **25**.
5. Cuando termine de agregar ambos puntos de conexión, aparecerán en el perfil de Traffic Manager junto con el estado de supervisión como **En línea**.

## <a name="test-the-traffic-manager-profile"></a>Prueba del perfil de Traffic Manager

Para ver a Traffic Manager en acción, complete los pasos siguientes:

1. Determine el nombre DNS del perfil de Traffic Manager.
2. Vea a Traffic Manager en acción.

### <a name="determine-dns-name-of-traffic-manager-profile"></a>Determinación del nombre DNS del perfil de Traffic Manager

En este tutorial, por simplicidad, usará el nombre DNS del perfil de Traffic Manager para visitar los sitios web.

Puede determinar el nombre DNS del perfil de Traffic Manager de la siguiente manera:

1. En la barra de búsqueda del portal, busque el nombre del perfil de Traffic Manager que creó en la sección anterior. Seleccione el perfil de Traffic Manager en los resultados que aparezcan.
2. Seleccione **Información general**.
3. El perfil de Traffic Manager muestra su nombre DNS. En las implementaciones de producción, puede configurar un nombre de dominio mnemónico para que apunte al nombre de dominio de Traffic Manager, mediante un registro CNAME de DNS.

   ![Nombre DNS de Traffic Manager](./media/tutorial-traffic-manager-improve-website-response/traffic-manager-dns-name.png)

### <a name="view-traffic-manager-in-action"></a>Ver a Traffic Manager en acción

En esta sección, puede ver a Traffic Manager en acción.

1. Seleccione **Todos los recursos** en el menú izquierdo. En la lista de recursos, seleccione **myVMEastUS** en el grupo de recursos **myResourceGroupTM1**.
2. En la página **Información general**, seleccione **Conectar**. En **Conectar a máquina virtual**, seleccione **Descargar archivo RDP**.
3. Abra el archivo .rdp descargado. Si se le pide, seleccione **Conectar**. Escriba el nombre de usuario y la contraseña que especificó al crear la máquina virtual. Puede que deba seleccionar **Más opciones** > **Usar otra cuenta** para especificar las credenciales que escribió al crear la máquina virtual.
4. Seleccione **Aceptar**.
5. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Si recibe la advertencia, seleccione **Sí** o **Continuar** para continuar con la conexión.
6. En un explorador web de la máquina virtual myVMEastUS, escriba el nombre DNS del perfil de Traffic Manager para ver el sitio web. Se le enruta al sitio web hospedado en el servidor IIS myIISVMEastUS porque tiene asignado un peso de **100**, que es superior. El servidor IIS myIISVMWestEurope tiene asignado un valor de ponderación del punto de conexión de **25**, que es inferior.

   ![Prueba del perfil de Traffic Manager](./media/tutorial-traffic-manager-improve-website-response/eastus-traffic-manager-test.png)

7. Repita los pasos del 1 al 6 en la máquina virtual myVMWestEurope para ver la respuesta del sitio web ponderada.

## <a name="delete-the-traffic-manager-profile"></a>Eliminar el perfil de Traffic Manager

Cuando ya no necesite los grupos de recursos creados en este tutorial, puede eliminarlos. Para ello, seleccione el grupo de recursos (**ResourceGroupTM1** o **ResourceGroupTM2**) y luego seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Enrutamiento del tráfico a puntos de conexión específicos según la ubicación geográfica del usuario](traffic-manager-configure-geographic-routing-method.md)
