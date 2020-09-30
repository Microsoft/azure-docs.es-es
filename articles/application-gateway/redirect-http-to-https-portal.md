---
title: 'Redireccionamiento de HTTP a HTTPS en el portal: Azure Application Gateway'
description: Aprenda a crear una puerta de enlace de aplicaciones con el tráfico redirigido de HTTP a HTTPS mediante Azure Portal.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/13/2019
ms.author: victorh
ms.openlocfilehash: 07689f7d16cd1df451fdab28f188e5c4a324486e
ms.sourcegitcommit: 1b320bc7863707a07e98644fbaed9faa0108da97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89594822"
---
# <a name="create-an-application-gateway-with-http-to-https-redirection-using-the-azure-portal"></a>Creación de una puerta de enlace de aplicaciones con redirección de HTTP a HTTPS mediante Azure Portal

Puede usar Azure Portal para crear una [puerta de enlace de aplicaciones](overview.md) con un certificado para la terminación TLS. Para redirigir el tráfico HTTP al puerto HTTPS en la puerta de enlace de aplicaciones se usa una regla de enrutamiento. En este ejemplo, también crea un [conjunto de escalado de máquinas virtuales](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md) para el grupo de back-end de la puerta de enlace de aplicaciones que contiene dos instancias de máquina virtual.

En este artículo aprenderá a:

* Creación de un certificado autofirmado
* Configurar una red
* Crear una puerta de enlace de aplicaciones con el certificado
* Adición de un agente de escucha y una regla de redireccionamiento
* Crear un conjunto de escalado de máquinas virtuales con el grupo de servidores back-end predeterminado

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

En este tutorial se requiere la versión 1.0.0 o posterior del módulo de Azure PowerShell para crear un certificado e instalar IIS. Ejecute `Get-Module -ListAvailable Az` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Para ejecutar los comandos de este tutorial, también debe ejecutar `Login-AzAccount` para crear una conexión con Azure.

## <a name="create-a-self-signed-certificate"></a>Creación de un certificado autofirmado

Para su uso en producción, debe importar un certificado válido firmado por un proveedor de confianza. Para este tutorial, creará un certificado autofirmado mediante [New-SelfSignedCertificate](https://docs.microsoft.com/powershell/module/pkiclient/new-selfsignedcertificate). Puede usar [Export-PfxCertificate](https://docs.microsoft.com/powershell/module/pkiclient/export-pfxcertificate) con la huella digital que se devolvió al exportar un archivo pfx del certificado.

```powershell
New-SelfSignedCertificate `
  -certstorelocation cert:\localmachine\my `
  -dnsname www.contoso.com
```

Debería ver algo parecido a este resultado:

```
PSParentPath: Microsoft.PowerShell.Security\Certificate::LocalMachine\my

Thumbprint                                Subject
----------                                -------
E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630  CN=www.contoso.com
```

Utilice la huella digital para crear el archivo pfx:

```powershell
$pwd = ConvertTo-SecureString -String "Azure123456!" -Force -AsPlainText
Export-PfxCertificate `
  -cert cert:\localMachine\my\E1E81C23B3AD33F9B4D1717B20AB65DBB91AC630 `
  -FilePath c:\appgwcert.pfx `
  -Password $pwd
```

## <a name="create-an-application-gateway"></a>Creación de una puerta de enlace de aplicaciones

Se necesita una red virtual para la comunicación entre los recursos que se crean. En este ejemplo se crean dos subredes: una para la puerta de enlace de aplicaciones y la otra para los servidores back-end. Puede crear una red virtual a la vez que crea la puerta de enlace de aplicaciones.

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. Haga clic en **Crear un recurso** en la esquina superior izquierda de Azure Portal.
3. Seleccione **Redes** y **Application Gateway** en la lista de destacados.
4. Especifique estos valores para la puerta de enlace de aplicaciones:

   - *myAppGateway*: como nombre de la puerta de enlace de aplicaciones.
   - *myResourceGroupAG*: como nuevo grupo de recursos.

     ![Creación de una nueva puerta de enlace de aplicaciones](./media/create-url-route-portal/application-gateway-create.png)

5. Acepte los valores predeterminados para las demás opciones y haga clic en **Aceptar**.
6. Haga clic en **Elegir una red virtual**, luego en **Crear nueva** y, después, especifique estos valores para la red virtual:

   - *myVNet*: como nombre de la red virtual.
   - *10.0.0.0/16*: como espacio de direcciones de la red virtual.
   - *myAGSubnet*: como nombre de subred.
   - *10.0.0.0/24*: como espacio de direcciones de la subred.

     ![Creación de una red virtual](./media/create-url-route-portal/application-gateway-vnet.png)

7. Haga clic en **Aceptar** para crear la red virtual y la subred.
8. En **Configuración IP de front-end**, asegúrese de que **Tipo de dirección IP** esté establecido en **Pública** y que **Crear nuevo** esté seleccionado. Escriba *myAGPublicIPAddress* para el nombre. Acepte los valores predeterminados para las demás opciones y haga clic en **Aceptar**.
9. En **Configuración del agente de escucha**, seleccione **HTTPS**, a continuación, seleccione **Seleccionar un archivo**, vaya al archivo *c:\appgwcert.pfx* y seleccione **Abrir**.
10. Escriba *appgwcert* para el nombre del certificado y *Azure123456!* como contraseña.
11. Deje el firewall de aplicaciones web deshabilitado y, a continuación, seleccione **Aceptar**.
12. Revise la configuración en la página de resumen y, a continuación, seleccione **Aceptar** para crear los recursos de red y la puerta de enlace de aplicaciones. La creación de la puerta de enlace de aplicaciones puede tardar varios minutos, espere a que finalice correctamente la implementación antes de pasar a la sección siguiente.

### <a name="add-a-subnet"></a>Incorporación de una subred

1. Seleccione **Todos los recursos** en el menú izquierdo y, después, seleccione **myVNet** en la lista de recursos.
2. Seleccione **Subredes** y, a continuación, haga clic en **Subred**.

    ![Creación de una subred](./media/create-url-route-portal/application-gateway-subnet.png)

3. Escriba *myBackendSubnet* como nombre de la subred.
4. Escriba *10.0.2.0/24* para el intervalo de direcciones y, a continuación, seleccione **Aceptar**.

## <a name="add-a-listener-and-redirection-rule"></a>Adición de un agente de escucha y una regla de redireccionamiento

### <a name="add-the-listener"></a>Adición del agente de escucha

En primer lugar, agregue un agente de escucha llamado *myListener* para el puerto 80.

1. Abra el grupo de recursos **myResourceGroupAG** y seleccione **myAppGateway**.
2. Seleccione **Agentes de escucha** y, a continuación, seleccione **+ Básico**.
3. Escriba *MyListener* para el nombre.
4. Escriba *httpPort* para el nombre del nuevo puerto de front-end y *80* para el puerto.
5. Asegúrese de que el protocolo está establecido en **HTTP** y, a continuación, seleccione **Aceptar**.

### <a name="add-a-routing-rule-with-a-redirection-configuration"></a>Adición de una regla de enrutamiento con una configuración de redireccionamiento

1. En **myAppGateway**, seleccione **Reglas** y, a continuación, seleccione **+Regla de enrutamiento de solicitud**.
2. En **Nombre de la regla**, escriba *Rule2*.
3. Asegúrese de que **MyListener** está seleccionado para el agente de escucha.
4. Haga clic en la pestaña **Destinos de back-end** y seleccione **Tipo de destino** como *Redirección*.
5. En **Tipo de redireccionamiento**, seleccione **Permanente**.
6. En **Destino del redireccionamiento**, seleccione **Agente de escucha**.
7. Asegúrese de que **Agente de escucha de destino** está establecido en **appGatewayHttpListener**.
8. Para **Cadena de consulta de inclusión** y **Ruta de acceso de inclusión** seleccione *Sí*.
9. Seleccione **Agregar**.

## <a name="create-a-virtual-machine-scale-set"></a>Crear un conjunto de escalado de máquinas virtuales

En este ejemplo, creará un conjunto de escalado de máquinas virtuales para proporcionar servidores al grupo de servidores back-end de la puerta de enlace de aplicaciones.

1. Seleccione **+Crear un recurso** en la esquina superior izquierda del portal.
2. Seleccione **Proceso**.
3. En el cuadro de búsqueda, escriba *conjunto de escalado* y presione ENTRAR.
4. Seleccione **Conjunto de escalado de máquinas virtuales** y, a continuación, seleccione **Crear**.
5. En **Nombre del conjunto de escalado de máquinas virtuales**, escriba *myvmss*.
6. Para la imagen del disco del sistema operativo, ** asegúrese de que **Windows Server 2016 Datacenter** está seleccionado.
7. Para **Grupo de recursos**, seleccione **myResourceGroupAG**.
8. En **Nombre de usuario**, escriba *azureuser*.
9. En **Contraseña**, escriba *Azure123456!* y confirme la contraseña.
10. En **Número de instancias**, asegúrese de que el valor es **2**.
11. Para **Tamaño de instancia**, seleccione **D2s_v3**.
12. En **Redes**, asegúrese de que **Elegir opciones de equilibrio de carga** está establecido en **Puerta de enlace de aplicaciones**.
13. Asegúrese de que **Puerta de enlace de aplicaciones** está establecido en **myAppGateway**.
14. Asegúrese de que **Subred** está establecido en **myBackendSubnet**.
15. Seleccione **Crear**.

### <a name="associate-the-scale-set-with-the-proper-backend-pool"></a>Asociación del conjunto de escalado con el grupo de back-end correcto

La interfaz de usuario del portal del conjunto de escalado de máquinas virtuales crea un nuevo grupo de back-end para el conjunto de escalado, pero desea asociarlo a appGatewayBackendPool.

1. Abra el grupo de recursos **myResourceGroupAg**.
2. Seleccione **myAppGateway**.
3. Seleccione **Grupos de back-end**.
4. Seleccione **myAppGatewaymyvmss**.
5. Seleccione **Eliminar todos los destinos del grupo de back-end**.
6. Seleccione **Guardar**.
7. Una vez completado este proceso, seleccione el grupo de back-end **myAppGatewaymyvmss**, seleccione **Eliminar** y, a continuación, seleccione **Aceptar** para confirmar.
8. Seleccione **appGatewayBackendPool**.
9. En **Destinos**, seleccione **VMSS**.
10. En **VMSS**, seleccione **myvmss**.
11. En **Configuraciones de interfaz de red**, seleccione **myvmssNic**.
12. Seleccione **Guardar**.

### <a name="upgrade-the-scale-set"></a>Actualización del conjunto de escalado

Por último, debe actualizar el conjunto de escalado con estos cambios.

1. Seleccione el conjunto de escalado **myvmss**.
2. En **Configuración**, seleccione **Instancias**.
3. Seleccione ambas instancias y, a continuación, seleccione **Actualizar**.
4. Seleccione **Sí** para confirmar la acción.
5. Una vez que se complete, vuelva a **myAppGateway** y seleccione **Grupos de back-end**. Ahora debería ver que **appGatewayBackendPool** tiene dos destinos y **myAppGatewaymyvmss** tiene cero destinos.
6. Seleccione **myAppGatewaymyvmss** y, a continuación, seleccione **Eliminar**.
7. Seleccione **Aceptar** para confirmar.

### <a name="install-iis"></a>Instalación de IIS

Una manera fácil de instalar IIS en el conjunto de escalado es usar PowerShell. Desde el portal, haga clic en el icono Cloud Shell y asegúrese de que **PowerShell** está seleccionado.

Pegue el código siguiente en la ventana de PowerShell y presione ENTRAR.

```azurepowershell
$publicSettings = @{ "fileUris" = (,"https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/appgatewayurl.ps1"); 
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File appgatewayurl.ps1" }
$vmss = Get-AzVmss -ResourceGroupName myResourceGroupAG -VMScaleSetName myvmss
Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $publicSettings
Update-AzVmss `
  -ResourceGroupName myResourceGroupAG `
  -Name myvmss `
  -VirtualMachineScaleSet $vmss
```

### <a name="upgrade-the-scale-set"></a>Actualización del conjunto de escalado

Después de cambiar las instancias con IIS, debe actualizar de nuevo el conjunto de escalado con este cambio.

1. Seleccione el conjunto de escalado **myvmss**.
2. En **Configuración**, seleccione **Instancias**.
3. Seleccione ambas instancias y, a continuación, seleccione **Actualizar**.
4. Seleccione **Sí** para confirmar la acción.

## <a name="test-the-application-gateway"></a>Prueba de la puerta de enlace de aplicaciones

Puede obtener la dirección IP pública de la aplicación en la página de información general de la puerta de enlace de aplicaciones.

1. Seleccione **myAppGateway**.
2. En la página **Información general**, anote la dirección IP de **Dirección IP pública de front-end**.

3. Copie la dirección IP pública y péguela en la barra de direcciones del explorador. Por ejemplo: http://52.170.203.149

   ![Advertencia de seguridad](./media/redirect-http-to-https-powershell/application-gateway-secure.png)

4. Para aceptar la advertencia de seguridad si usó un certificado autofirmado, seleccione **Detalles** y, a continuación, **Acceder a la página web**. El sitio web IIS protegido se muestra ahora como en el ejemplo siguiente:

   ![Prueba de la dirección URL base en la puerta de enlace de aplicaciones](./media/redirect-http-to-https-powershell/application-gateway-iistest.png)

## <a name="next-steps"></a>Pasos siguientes

Consulte [Creación de una puerta de enlace de aplicaciones con redireccionamiento interno](redirect-internal-site-powershell.md).
