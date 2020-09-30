---
title: 'Azure VPN Gateway: Creación e instalación de archivos de configuración de cliente VPN: conexiones RADIUS P2S'
description: Cree archivos de configuración de cliente VPN en Windows, Mac OS X y Linux para conexiones que usan autenticación RADIUS.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: 35631c8a0b66ade1457228ba16150f94f761f7b3
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89419918"
---
# <a name="create-and-install-vpn-client-configuration-files-for-p2s-radius-authentication"></a>Creación e instalación de archivos de configuración de cliente VPN para la autenticación P2S RADIUS

Para conectarse de punto a sitio (P2S) a una red virtual, debe configurar el dispositivo cliente desde el que se va a conectar. Puede crear conexiones VPN de punto a sitio desde dispositivos cliente de Windows, Mac OS X y Linux. 

Con la autenticación RADIUS existen varias opciones de autenticación, por ejemplo, la autenticación con nombre de usuario y contraseña y mediante certificados. La configuración de cliente VPN es diferente para cada tipo de autenticación. Para configurar el cliente VPN, use los archivos de configuración de cliente que contienen la configuración necesaria. En este artículo se ayuda a crear e instalar la configuración de cliente VPN para el tipo de autenticación RADIUS que se quiere usar.

>[!IMPORTANT]
>[!INCLUDE [TLS](../../includes/vpn-gateway-tls-change.md)]
>

El flujo de trabajo de configuración de la autenticación P2S RADIUS es el siguiente:

1. [Configure la puerta de enlace de VPN de Azure para la conectividad P2S](point-to-site-how-to-radius-ps.md).
2. [Configure el servidor RADIUS para la autenticación](point-to-site-how-to-radius-ps.md#radius). 
3. **Obtenga la configuración de cliente VPN correspondiente a la opción de autenticación que prefiera y úsela para configurar el cliente VPN** (este artículo).
4. [Complete la configuración de P2S y conéctese](point-to-site-how-to-radius-ps.md).

>[!IMPORTANT]
>Si después de generar el perfil de configuración de cliente VPN hay algún cambio en la configuración de VPN de punto a sitio, como el tipo de protocolo de VPN o el tipo de autenticación, debe generar e instalar una nueva configuración de cliente VPN en los dispositivos de los usuarios.
>
>

Para usar las secciones de este artículo, decida primero el tipo de autenticación que quiere usar: nombre de usuario y contraseña, certificado o de otro tipo. En cada sección hay pasos para Windows, Mac OS X y Linux (pasos limitados disponibles en este momento).


## <a name="usernamepassword-authentication"></a><a name="adeap"></a>Autenticación con nombre de usuario y contraseña

Puede configurar la autenticación con nombre de usuario y contraseña para usar Active Directory o no usarlo. Asegúrese de que todos los usuarios que se conectan tengan credenciales de nombre de usuario y contraseña que se puedan autenticar mediante RADIUS.

Al configurar la autenticación de nombre de usuario y contraseña, solo se puede crear una configuración para el protocolo de autenticación de nombre de usuario y contraseña EAP-MSCHAPv2. En los comandos, `-AuthenticationMethod` es `EapMSChapv2`.

### <a name="1-generate-vpn-client-configuration-files"></a><a name="usernamefiles"></a> 1. Generación de archivos de configuración de cliente VPN

Puede generar los archivos de configuración del cliente VPN a través de Azure Portal o de Azure PowerShell.

#### <a name="azure-portal"></a>Portal de Azure

1. Navegue a la puerta de enlace de red virtual.
2. Haga clic en **Configuración de punto a sitio**.
3. Haga clic en **Descargar cliente VPN**.
4. Seleccione el cliente y rellene toda la información que se pide.
5. Haga clic en **Descargar** para generar el archivo ZIP.
6. Se descargará el archivo ZIP, normalmente en la carpeta Descargas.

#### <a name="azure-powershell"></a>Azure PowerShell

Genere archivos de configuración de cliente VPN para usar con la autenticación de nombre de usuario y contraseña. Genere los archivos de configuración de cliente VPN con el comando siguiente:

```azurepowershell-interactive
New-AzVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW" -AuthenticationMethod "EapMSChapv2"
```
 
La ejecución del comando devuelve un vínculo. Copie y pegue el vínculo en un explorador web para descargar **VpnClientConfiguration.zip**. Descomprima el archivo para ver las siguientes carpetas: 
 
* **WindowsAmd64** y **WindowsX86**: Estas carpetas contienen los paquetes del instalador de Windows de 64 y 32 bits, respectivamente. 
* **Genérico**: esta carpeta contiene información general que se usa para crear su propia configuración de cliente VPN. Esta carpeta no es necesaria para las configuraciones de autenticación con nombre de usuario y contraseña.
* **Mac**: si configuró IKEv2 al crear la puerta de enlace de red virtual, verá una carpeta llamada **Mac** con un archivo **mobileconfig**. Use este archivo para configurar clientes Mac.

Si ya ha creado los archivos de configuración del cliente, puede recuperarlos con el cmdlet `Get-AzVpnClientConfiguration`. Pero si realiza cambios en la configuración de VPN de punto a sitio, como el tipo de protocolo de VPN o el tipo de autenticación, la configuración no se actualiza automáticamente. Deberá ejecutar el cmdlet  `New-AzVpnClientConfiguration` para crear otra descarga de configuración.

Para recuperar los archivos de configuración de cliente generados anteriormente, use el comando siguiente:

```azurepowershell-interactive
Get-AzVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW"
```

### <a name="2-configure-vpn-clients"></a><a name="setupusername"></a> 2. Configuración de clientes VPN

Puede configurar los clientes VPN siguientes:

* [Windows](#adwincli)
* [Mac (OS X)](#admaccli)
* [Linux con strongSwan](#adlinuxcli)
 
#### <a name="windows-vpn-client-setup"></a><a name="adwincli"></a>Configuración de un cliente VPN en Windows

Puede utilizar el mismo paquete de configuración de cliente VPN en todos los equipos cliente Windows, siempre que la versión coincida con la arquitectura del cliente. Para la lista de sistemas operativos cliente compatibles, consulte las [preguntas frecuentes](vpn-gateway-vpn-faq.md#P2S).

Use estos pasos para configurar al cliente VPN de Windows nativo para la autenticación mediante certificado:

1. Seleccione los archivos de configuración de cliente VPN que correspondan a la arquitectura del equipo Windows. Si la arquitectura de procesador es de 64 bits, elija el paquete del instalador **VpnClientSetupAmd64**. En caso de que sea de 32 bits, elija el paquete del instalador **VpnClientSetupX86**. 
2. Haga doble clic en el paquete para instalarlo. Si ve una ventana emergente SmartScreen, seleccione **Más información** > **Ejecutar de todas formas**.
3. En el equipo cliente, vaya a **Configuración de red** y seleccione **VPN**. La conexión VPN muestra el nombre de la red virtual a la que se conecta. 

#### <a name="mac-os-x-vpn-client-setup"></a><a name="admaccli"></a>Configuración de un cliente VPN en Mac (OSX)

1. Seleccione el archivo **VpnClientSetup mobileconfig** y envíelo a cada uno de los usuarios. Para ello, puede usar el correo electrónico u otro método.

2. Busque el archivo **mobileconfig** en el equipo Mac.

   ![Ubicación del archivo mobileconfig](./media/point-to-site-vpn-client-configuration-radius/admobileconfigfile.png)

3. Paso opcional: si desea especificar un DNS personalizado, agregue las líneas siguientes al archivo **mobileconfig**:

   ```xml
    <key>DNS</key>
    <dict>
      <key>ServerAddresses</key>
        <array>
            <string>10.0.0.132</string>
        </array>
      <key>SupplementalMatchDomains</key>
        <array>
            <string>TestDomain.com</string>
        </array>
    </dict> 
   ```
4. Haga doble clic en el perfil para instalarlo y seleccione **Continuar**. El nombre del perfil es el mismo que el de la red virtual.

   ![Mensaje de instalación](./media/point-to-site-vpn-client-configuration-radius/adinstall.png)
5. Seleccione **Continuar** para confiar en el emisor del perfil y proseguir con la instalación.

   ![Mensaje de confirmación](./media/point-to-site-vpn-client-configuration-radius/adcontinue.png)
6. Durante la instalación del perfil, tendrá la opción de especificar el nombre de usuario y la contraseña para la autenticación de VPN. No es obligatorio especificar esta información. Si lo hace, esta se guarda y se usa automáticamente al iniciar una conexión. Seleccione **Instalar** para continuar.

   ![Cuadros de nombre de usuario y contraseña para VPN](./media/point-to-site-vpn-client-configuration-radius/adsettings.png)
7. Para instalar el perfil en el equipo, escriba un nombre de usuario y una contraseña para los privilegios necesarios. Seleccione **Aceptar**.

   ![Cuadros de nombre de usuario y contraseña para la instalación del perfil](./media/point-to-site-vpn-client-configuration-radius/adusername.png)
8. Una vez instalado, el perfil es visible en el cuadro de diálogo **Perfiles**. Este cuadro de diálogo también se puede abrir más adelante desde **Preferencias del sistema**.

   ![Cuadro de diálogo "Perfiles"](./media/point-to-site-vpn-client-configuration-radius/adsystempref.png)
9. Para acceder a la conexión VPN, abra el cuadro de diálogo **Red** en **Preferencias del sistema**.

   ![Iconos de Preferencias del sistema](./media/point-to-site-vpn-client-configuration-radius/adnetwork.png)
10. La conexión VPN se muestra como **IkeV2-VPN**. El nombre se puede cambiar mediante la actualización del archivo **mobileconfig**.

    ![Detalles de la conexión VPN](./media/point-to-site-vpn-client-configuration-radius/adconnection.png)
11. Seleccione **Configuración de autenticación**. Seleccione el **nombre de usuario** de la lista desplegable y escriba las credenciales. Si escribió las credenciales anteriormente, el **nombre de usuario** se elige automáticamente de la lista y se rellena previamente, junto con la contraseña. Seleccione **Aceptar** para guardar la configuración.

    ![Configuración de autenticación](./media/point-to-site-vpn-client-configuration-radius/adauthentication.png)
12. De nuevo en el cuadro de diálogo **Red**, seleccione **Aplicar** para guardar los cambios. Para iniciar la conexión, seleccione **Conectar**.

#### <a name="linux-vpn-client-setup-through-strongswan"></a><a name="adlinuxcli"></a>Configuración de un cliente VPN en Linux mediante strongSwan

Las siguientes instrucciones se crearon mediante strongSwan 5.5.1 en Ubuntu 17.0.4. Las pantallas reales pueden variar en función de la versión de Linux y strongSwan.

1. Abra la herramienta **Terminal** para instalar **strongSwan** y su Network Manager (Administrador de red) ejecutando el comando del ejemplo. Si recibe un error relacionado con `libcharon-extra-plugins`, reemplácelo por `strongswan-plugin-eap-mschapv2`.

   ```Terminal
   sudo apt-get install strongswan libcharon-extra-plugins moreutils iptables-persistent network-manager-strongswan
   ```
2. Seleccione el icono **Network Manager** (Administrador de red) (flecha arriba/flecha abajo) y seleccione **Edit Connections** (Editar conexiones).

   ![Selección de "Edit Connections" (Editar conexiones) en Network Manager (Administrador de red)](./media/point-to-site-vpn-client-configuration-radius/EditConnection.png)
3. Seleccione el botón **Add** (Agregar) para crear una conexión.

   ![Botón "Add" (Agregar) para una conexión](./media/point-to-site-vpn-client-configuration-radius/AddConnection.png)
4. Seleccione **IPsec/IKEv2 (strongswan)** en el menú desplegable y seleccione **Create** (Crear). Puede cambiar el nombre de la conexión en este paso.

   ![Selección del tipo de conexión](./media/point-to-site-vpn-client-configuration-radius/AddIKEv2.png)
5. Abra el archivo **VpnSettings.xml** de la carpeta **Generic** (Genérico) de los archivos de configuración de cliente descargados. Busque la etiqueta denominada `VpnServer` y copie el nombre que empieza por `azuregateway` y termina por `.cloudapp.net`.

   ![Contenido del archivo VpnSettings.xml](./media/point-to-site-vpn-client-configuration-radius/VpnSettings.png)
6. Pegue este nombre en el campo **Address** (Dirección) de la nueva conexión VPN de la sección **Gateway** (Puerta de enlace). Luego, seleccione el icono de carpeta al final del campo **Certificate** (Certificado), vaya a la carpeta **Generic** (Genérico) y seleccione el archivo **VpnServerRoot**.
7. En la sección **Client** (Cliente) de la conexión, seleccione **EAP** de **Autentication** (Autenticación) y escriba el nombre de usuario y la contraseña. Es posible que tenga que seleccionar el icono de bloqueo de la derecha para guardar esta información. Después, seleccione **Guardar**.

   ![Edición de la configuración de conexión](./media/point-to-site-vpn-client-configuration-radius/editconnectionsettings.png)
8. Seleccione el icono **Network Manager** (Administrador de red) (flecha arriba/flecha abajo) y mantenga el puntero sobre **VPN Connections** (Conexiones VPN). Verá la conexión VPN que ha creado. Para iniciar la conexión, selecciónela.

   ![Conexión "VPN Radius" en Network Manager (Administrador de red)](./media/point-to-site-vpn-client-configuration-radius/ConnectRADIUS.png)

## <a name="certificate-authentication"></a><a name="certeap"></a>Autenticación de certificados
 
Puede crear archivos de configuración de cliente VPN para la autenticación de certificados RADIUS con el protocolo EAP-TLS. Normalmente, para autenticar a un usuario en una VPN se usa un certificado emitido por la empresa. Asegúrese de que todos los usuarios que se conectan tengan instalado un certificado en el dispositivo y de que el servidor RADIUS pueda validarlo.

>[!NOTE]
>[!INCLUDE [TLS](../../includes/vpn-gateway-tls-change.md)]
>

En los comandos, `-AuthenticationMethod` es `EapTls`. Durante la autenticación de certificado, el cliente valida el servidor RADIUS mediante la validación de su certificado. `-RadiusRootCert` es el archivo .cer que contiene el certificado raíz para validar el servidor RADIUS.

Cada dispositivo de cliente VPN requiere tener instalado un certificado de cliente. En ocasiones, un dispositivo Windows tiene varios certificados de cliente. Como consecuencia, durante la autenticación puede aparecer un cuadro de diálogo emergente que muestra todos los certificados. El usuario debe elegir el certificado que quiere usar. Se puede filtrar el certificado correcto mediante la especificación del certificado raíz al que se debe encadenar el certificado de cliente. 

`-ClientRootCert` es el archivo .cer que contiene el certificado raíz. Es un parámetro opcional. Si el dispositivo desde el que desea conectarse tiene solo un certificado cliente, no es necesario especificar este parámetro.

### <a name="1-generate-vpn-client-configuration-files"></a><a name="certfiles"></a>1. Generación de archivos de configuración de cliente VPN

Genere archivos de configuración de cliente VPN para usar con la autenticación de certificado. Genere los archivos de configuración de cliente VPN con el comando siguiente:
 
```azurepowershell-interactive
New-AzVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW" -AuthenticationMethod "EapTls" -RadiusRootCert <full path name of .cer file containing the RADIUS root> -ClientRootCert <full path name of .cer file containing the client root> | fl
```

La ejecución del comando devuelve un vínculo. Copie y pegue el vínculo en un explorador web para descargar el archivo VpnClientConfiguration.zip. Descomprima el archivo para ver las siguientes carpetas:

* **WindowsAmd64** y **WindowsX86**: Estas carpetas contienen los paquetes del instalador de Windows de 64 y 32 bits, respectivamente. 
* **GenericDevice**: esta carpeta contiene información general que se usa para crear su propia configuración de cliente VPN.

Si ya ha creado los archivos de configuración del cliente, puede recuperarlos con el cmdlet `Get-AzVpnClientConfiguration`. Pero si realiza cambios en la configuración de VPN de punto a sitio, como el tipo de protocolo de VPN o el tipo de autenticación, la configuración no se actualiza automáticamente. Deberá ejecutar el cmdlet  `New-AzVpnClientConfiguration` para crear otra descarga de configuración.

Para recuperar los archivos de configuración de cliente generados anteriormente, use el comando siguiente:

```azurepowershell-interactive
Get-AzVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW" | fl
```
 
### <a name="2-configure-vpn-clients"></a><a name="setupusername"></a> 2. Configuración de clientes VPN

Puede configurar los clientes VPN siguientes:

* [Windows](#certwincli)
* [Mac (OS X)](#certmaccli)
* Linux (compatible, aún no hay ningún artículo con los pasos)

#### <a name="windows-vpn-client-setup"></a><a name="certwincli"></a>Configuración de un cliente VPN en Windows

1. Seleccione un paquete de configuración e instálelo en el dispositivo cliente. Si la arquitectura de procesador es de 64 bits, elija el paquete del instalador **VpnClientSetupAmd64**. En caso de que sea de 32 bits, elija el paquete del instalador **VpnClientSetupX86**. Si ve una ventana emergente SmartScreen, seleccione **Más información** > **Ejecutar de todas formas**. También puede guardar el paquete para instalarlo en otros equipos cliente.
2. Cada equipo cliente debe tener un certificado de cliente para la autenticación. Instale el certificado de cliente. Para información sobre los certificados de cliente, consulte el artículo sobre los [certificados de cliente para conexiones de punto a sitio](vpn-gateway-certificates-point-to-site.md). Para instalar un certificado que se ha generado, vea [Instalación de un certificado de cliente para conexiones de punto a sitio con autenticación de certificados de Azure](point-to-site-how-to-vpn-client-install-azure-cert.md).
3. En el equipo cliente, vaya a **Configuración de red** y seleccione **VPN**. La conexión VPN muestra el nombre de la red virtual a la que se conecta.

#### <a name="mac-os-x-vpn-client-setup"></a><a name="certmaccli"></a>Configuración de un cliente VPN en Mac (OSX)

Debe crear un perfil independiente para cada dispositivo Mac que se conecte a la red virtual de Azure. El motivo es que estos dispositivos requieren que se especifique el certificado de usuario para la autenticación en el perfil. La carpeta **Generic** contiene toda la información necesaria para crear un perfil:

* **VpnSettings.xml** contiene valores de configuración importantes, como el tipo de túnel y la dirección de servidor.
* **VpnServerRoot.cer** contiene el certificado raíz necesario para validar la puerta de enlace de VPN durante la configuración de la conexión de punto a sitio.
* **RadiusServerRoot.cer** contiene el certificado raíz necesario para validar el servidor RADIUS durante la autenticación.

Siga los pasos a continuación para configurar el cliente VPN nativo en equipos Mac para la autenticación con certificado:

1. Importe los certificados raíz **VpnServerRoot** y **RadiusServerRoot** en el equipo Mac. Copie los archivos en el equipo Mac, haga doble clic en ellos y seleccione **Agregar**.

   ![Incorporación del certificado VpnServerRoot](./media/point-to-site-vpn-client-configuration-radius/addcert.png)

   ![Incorporación del certificado RadiusServerRoot](./media/point-to-site-vpn-client-configuration-radius/radiusrootcert.png)
2. Cada equipo cliente debe tener un certificado de cliente para la autenticación. Instale el certificado de cliente en el dispositivo cliente.
3. En **Preferencias de red**, abra el cuadro de diálogo **Red**. Seleccione **+** para crear un nuevo perfil de conexión de cliente VPN para una conexión de punto a sitio a la red virtual de Azure.

   El valor de **Interfaz** es **VPN** y el de **Tipo de VPN** es **IKEv2**. Especifique un nombre para el perfil en el cuadro **Nombre del servicio** y seleccione **Crear** para crear el perfil de conexión de cliente VPN.

   ![Información del nombre de servicio y de la interfaz](./media/point-to-site-vpn-client-configuration-radius/network.png)
4. En la carpeta **Genérico**, en el archivo **VpnSettings.xml**, copie el valor de la etiqueta **VpnServer**. Pegue este valor en los cuadros **Dirección del servidor** e **Id. remoto** del perfil. Deje en blanco el cuadro **Id. local**.

   ![Información de servidor](./media/point-to-site-vpn-client-configuration-radius/servertag.png)
5. Seleccione **Configuración de autenticación** y **Certificado**. 

   ![Configuración de autenticación](./media/point-to-site-vpn-client-configuration-radius/certoption.png)
6. Haga clic en **Seleccionar** para elegir el certificado que quiere usar en la autenticación.

   ![Selección de un certificado para la autenticación](./media/point-to-site-vpn-client-configuration-radius/certificate.png)
7. En **Choose An Identity** (Elegir una identidad) se muestra una lista de certificados para elegir. Seleccione el certificado adecuado y **Continuar**.

   ![Lista "Choose An Identity" (Elegir una identidad)](./media/point-to-site-vpn-client-configuration-radius/identity.png)
8. En el cuadro **Id. local**, especifique el nombre del certificado (del paso 6). En este ejemplo, es **ikev2Client.com**. A continuación, seleccione el botón **Aplicar** para guardar los cambios.

   ![Cuadro "Id. local"](./media/point-to-site-vpn-client-configuration-radius/applyconnect.png)
9. En el cuadro de diálogo **Red**, seleccione **Aplicar** para guardar los cambios. A continuación, seleccione **Conectar** para iniciar la conexión de punto a sitio a la red virtual de Azure.

## <a name="working-with-other-authentication-types-or-protocols"></a><a name="otherauth"></a>Trabajar con otros tipos de autenticación o protocolos

Para usar un tipo de autenticación diferente (por ejemplo, OTP) o un protocolo de autenticación diferente (como PEAP-MSCHAPv2, en lugar de EAP-MSCHAPv2), debe crear su propio perfil de configuración de cliente VPN. Para crear el perfil, necesita información como la dirección IP de puerta de enlace de red virtual, el tipo de túnel y las rutas de túnel dividido. Para obtener esta información, siga estos pasos:

1. Use el cmdlet `Get-AzVpnClientConfiguration` para generar la configuración de cliente VPN para EapMSChapv2.

2. Descomprima el archivo VpnClientConfiguration.zip y busque la carpeta **GenericDevice**. Ignore las carpetas que contienen los instaladores de Windows para arquitecturas de 64 y 32 bits.
 
3. La carpeta **GenericDevice** contiene un archivo XML llamado **VpnSettings**. Este archivo contiene toda la información necesaria:

   * **VpnServer**: nombre de dominio completo de la instancia de Azure VPN Gateway. Es la dirección a la que se conecta el cliente.
   * **VpnType**: tipo de túnel para la conexión.
   * **Rutas**: rutas que debe configurar en el perfil para que solo el tráfico enlazado de la red virtual de Azure se envíe a través del túnel de punto a sitio.
   
   La carpeta **GenericDevice** también contiene un archivo .cer denominado **VpnServerRoot**. Este archivo contiene el certificado raíz necesario para validar la instancia de Azure VPN Gateway durante la configuración de la conexión de punto a sitio. Instale el certificado en todos los dispositivos que se van a conectar a la red virtual de Azure.

## <a name="next-steps"></a>Pasos siguientes

Vuelva al artículo para [completar la configuración de la conexión de punto a sitio](point-to-site-how-to-radius-ps.md).

Para obtener información acerca de cómo solucionar problemas de P2S, consulte [Solución de problemas: conexión de punto a sitio de Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).
