---
title: Uso de certificados con el dispositivo Azure Stack Edge Pro con GPU | Microsoft Docs
description: Descripción del uso de certificados con el dispositivo Azure Stack Edge Pro con GPU, por ejemplo, por qué usarlos, qué tipo usar y cómo cargarlos en el dispositivo.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: 59924312fe0483d11d0f70ce83b8f6e4b0e198dc
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90890759"
---
# <a name="use-certificates-with-azure-stack-edge-pro-gpu-device"></a>Uso de certificados con el dispositivo Azure Stack Edge Pro con GPU

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

En este artículo se describen los tipos de certificados que se pueden instalar en el dispositivo Azure Stack Edge Pro. En el artículo también se incluyen los detalles de cada tipo de certificado junto con el procedimiento de instalación y de identificación de la fecha de expiración. 

## <a name="about-certificates"></a>Acerca de los certificados

Los certificados proporcionan un vínculo entre una **clave pública** y una entidad (como una nombre de dominio) que ha sido **firmada** (verificada) por un tercero de confianza (por ejemplo, una **entidad de certificación**).  Proporcionan una manera cómoda de distribuir claves de cifrado públicas de confianza. Por lo tanto, los certificados aseguran que la comunicación es de confianza y que envía información cifrada al servidor adecuado. 

Cuando el dispositivo Azure Stack Edge Pro está configurado desde el principio, se generan automáticamente certificados autofirmados. Como alternativa, puede aportar sus propios certificados. Hay instrucciones que debe seguir si tiene previsto traer sus propios certificados.

## <a name="types-of-certificates"></a>Tipos de certificados

Los distintos tipos de certificados que se usan en el dispositivo Azure Stack Edge Pro son los siguientes: 
- Certificados de firma
    - CA raíz
    - Intermedio

- Certificados de punto de conexión
    - Certificado de nodo
    - Certificados de interfaz de usuario local
    - Certificados de Azure Resource Manager
    - Certificados de Blob Storage
    - Certificados de dispositivo IoT
    <!--- WiFi certificates
    - VPN certificates-->

- Certificados de cifrado
    - Certificados de sesión de soporte

Estos certificados se describen minuciosamente en las secciones siguientes.

## <a name="signing-chain-certificates"></a>Certificados de cadena de firma

Estos son los certificados para la autoridad que firma los certificados o la entidad del certificado de firma. 

### <a name="types"></a>Tipos

Estos certificados podrían ser certificados raíz o intermedios. Los certificados raíz siempre son autofirmados (o se firman por sí mismos). Los certificados intermedios no son autofirmados, sino que los firma la autoridad de firma.

### <a name="caveats"></a>Advertencias

- Los certificados raíz deben ser certificados de cadena de firma.
- Los certificados raíz se pueden cargar en el dispositivo con el siguiente formato: 
    - **DER**: están disponibles como extensión de archivo `.cer`.
    - **Con codificación Base 64 o PEM**: están disponibles también como extensión `.cer`.
    - **P7b**: este formato se usa solo para certificados de cadena de firma que incluyen los certificados raíz e intermedio.
- Los certificados de cadena de firma siempre se cargan antes que cualquier otro.


## <a name="node-certificates"></a>Certificados de nodo

<!--Your Azure Stack Edge Pro device could be a 1-node device or a 4-node device.--> Todos los nodos del dispositivo se comunican constantemente entre sí y, por lo tanto, deben tener una relación de confianza. Los certificados de nodo proporcionan una manera de establecer esa confianza. También entran en juego cuando se conecta al nodo de dispositivo mediante una sesión remota de PowerShell mediante HTTPS.

### <a name="caveats"></a>Advertencias

- El certificado de nodo debe proporcionarse en formato `.pfx` con una clave privada que se pueda exportar. 
- Puede crear y cargar un certificado de nodo comodín o cuatro certificados de nodo individuales. 
- Los certificados de nodo deben cambiarse si el dominio DNS cambia pero el nombre del dispositivo no. Si va a traer su propio certificado de nodo, no podrá cambiar el número de serie del dispositivo, solo podrá cambiar el nombre de dominio.
- Utilice la tabla siguiente como guía para crear un certificado de nodo.
   
    |Tipo |Nombre del firmante (SN)  |Nombre alternativo del firmante (SAN)  |Ejemplo de nombre de firmante |
    |---------|---------|---------|---------|
    |Nodo|`<NodeSerialNo>.<DnsDomain>`|`*.<DnsDomain>`<br><br>`<NodeSerialNo>.<DnsDomain>`|`mydevice1.microsoftdatabox.com` |
   

## <a name="endpoint-certificates"></a>Certificados de punto de conexión

En el caso de los puntos de conexión que expone el dispositivo se requiere un certificado para la comunicación de confianza. Los certificados de punto de conexión incluyen los necesarios al acceder a Azure Resource Manager y al almacenamiento de blobs mediante las API REST. 

Cuando incorpore un certificado firmado de su propiedad, necesitará también la cadena de firma correspondiente. En el caso de los certificados de cadena de firma, Azure Resource Manager y blob en el dispositivo, necesitará también los certificados correspondientes en la máquina cliente para autenticarse y comunicarse con el dispositivo.

### <a name="caveats"></a>Advertencias

- Los certificados de punto de conexión deben estar en formato `.pfx` con una clave privada. La cadena de firma debe estar en formato DER (extensión de archivo `.cer`). 
- Sus certificados de punto de conexión propios pueden ser individuales o de varios dominios. 
- Si lo que trae es una cadena de firma, se debe cargar el certificado de la cadena de firma antes de cargar un certificado de punto de conexión.
- Estos certificados deben cambiarse si cambian el nombre del dispositivo o los nombres de dominio DNS.
- Se puede usar un certificado de punto de conexión comodín.
- Las propiedades de los certificados de punto de conexión son similares a las de un certificado SSL típico. 
- Al crear un certificado de punto de conexión, use la tabla siguiente:

    |Tipo |Nombre del firmante (SN)  |Nombre alternativo del firmante (SAN)  |Ejemplo de nombre de firmante |
    |---------|---------|---------|---------|
    |Azure Resource Manager|`management.<Device name>.<Dns Domain>`|`login.<Device name>.<Dns Domain>`<br>`management.<Device name>.<Dns Domain>`|`management.mydevice1.microsoftdatabox.com` |
    |Blob Storage|`*.blob.<Device name>.<Dns Domain>`|`*.blob.< Device name>.<Dns Domain>`|`*.blob.mydevice1.microsoftdatabox.com` |
    |Certificado único de varios SAN para ambos puntos de conexión|`<Device name>.<dnsdomain>`|`<Device name>.<dnsdomain>`<br>`login.<Device name>.<Dns Domain>`<br>`management.<Device name>.<Dns Domain>`<br>`*.blob.<Device name>.<Dns Domain>`|`mydevice1.microsoftdatabox.com` |


## <a name="local-ui-certificates"></a>Certificados de interfaz de usuario local

Puede acceder a la interfaz de usuario web local del dispositivo desde un explorador. Para asegurarse de que esta comunicación es segura, puede cargar su propio certificado. 

### <a name="caveats"></a>Advertencias

- El certificado de la interfaz de usuario local también se carga en formato `.pfx` con una clave privada que se puede exportar.
- Después de cargar este certificado de interfaz de usuario local, deberá reiniciar el explorador y borrar la memoria caché. Consulte las instrucciones específicas para el explorador.

    |Tipo |Nombre del firmante (SN)  |Nombre alternativo del firmante (SAN)  |Ejemplo de nombre de firmante |
    |---------|---------|---------|---------|
    |Interfaz de usuario local| `<Device name>.<DnsDomain>`|`<Device name>.<DnsDomain>`| `mydevice1.microsoftdatabox.com` |
   

## <a name="iot-edge-device-certificates"></a>Certificados de dispositivo IoT Edge

El dispositivo Azure Stack Edge Pro también es un dispositivo IoT con el proceso habilitado por un dispositivo IoT Edge conectado a él. También puede cargar certificados de IoT Edge para las conexiones seguras entre este dispositivo IoT Edge y los dispositivos de nivel inferior que puedan conectarse a él. 

El dispositivo tiene certificados autofirmados por si desea utilizar solo el escenario de proceso con el dispositivo. En cambio, si el dispositivo Azure Stack Edge Pro está conectado a dispositivos de nivel inferior, deberá traer sus propios certificados.

Hay tres certificados de IoT Edge que necesita instalar para habilitar esta relación de confianza:

- **Entidad de certificación raíz o entidad de certificación del propietario**
- **Entidad de certificación de dispositivo** 
- **Certificado clave del dispositivo**

### <a name="caveats"></a>Advertencias

- Los certificados de IoT Edge se cargan en formato `.pem`. 


Para más información sobre los certificados de IoT Edge, consulte [Detalles de los certificados de Azure IoT Edge](../iot-edge/iot-edge-certs.md#iot-edge-certificates).

## <a name="support-session-certificates"></a>Certificados de sesión de soporte

Si el dispositivo Azure Stack Edge Pro tiene problemas, para solucionarlos, puede abrir una sesión remota de soporte técnico de PowerShell en el dispositivo. Para permitir una comunicación segura cifrada con esta sesión de soporte técnico, puede cargar un certificado.

### <a name="caveats"></a>Advertencias

- Asegúrese de que el certificado `.pfx` correspondiente con clave privada está instalado en la máquina cliente mediante la herramienta de descifrado.
- Compruebe que el campo **Uso de la clave** del certificado no sea **Firma de certificados**. Para comprobarlo, haga clic con el botón derecho en el certificado, elija **Abrir** y, en la pestaña **Detalles**, busque **Uso de la clave**. 


### <a name="caveats"></a>Advertencias

- El certificado de la sesión de soporte técnico se debe proporcionar con formato DER y extensión `.cer`.


<!--## VPN certificates

If VPN is configured on your Azure Stack Edge Pro device, then you will also need a certificate for any communication that occurs over the VPN channel. You can bring your own VPN certificate to ensure the communication is trusted.

### Caveats

- The VPN certificate must be uploaded as a pfx format with a  private key.
- The VPN certificate is not dependant on the device name, device serial number, or device configuration. It only requires the external FQDN.
- Make sure that the client OID is set.-->

<!--## WiFi certificates

If your device is configured to operate on a wireless network, then you will also need a WiFi certificate for any communication that occurs over the wireless network. 

### Caveats

- The WiFi certificate must be uploaded as a pfx format with a private key.
- Make sure that the client OID is set.-->


## <a name="create-certificates-optional"></a>Creación de certificados (opcional)

En la siguiente sección se describe el procedimiento para crear certificados de punto de conexión y de cadena de firma.

### <a name="certificate-workflow"></a>Flujo de trabajo de certificado

Tendrá una manera definida de creación de certificados para los dispositivos que funcionan en su entorno. Puede usar los certificados proporcionados por el administrador de TI. 

**Solo con fines de desarrollo o de prueba, también puede usar Windows PowerShell para crear certificados en el sistema local.** Al crear los certificados para el cliente, siga estas instrucciones:

1. Puede crear cualquiera de los siguientes tipos de certificados:

    - Cree un único certificado válido para su uso con un solo nombre de dominio completo (FQDN). Por ejemplo, *mydomain.com*.
    - Cree un certificado comodín para proteger también el nombre de dominio principal y varios subdominios. Por ejemplo, * *.mydomain.com*.
    - Cree un certificado de nombre alternativo del firmante (SAN) que abarque varios nombres de dominio en un solo certificado. 

2. Si trae su propio certificado, necesitará un certificado raíz para la cadena de firma. Consulte los pasos de [Creación de certificados de cadena de firma](#create-signing-chain-certificate).

3. A continuación, puede crear los certificados de punto de conexión para la interfaz de usuario local del dispositivo, el blob y Azure Resource Manager. Puede crear 3 certificados independientes para el dispositivo, el blob y el Azure Resource Manager, o un certificado para los 3 puntos de conexión. Para pasos detallados, consulte [Creación de certificados de punto de conexión firmados](#create-signed-endpoint-certificates).

4. Tanto si crea 3 certificados independientes como uno solo, especifique los nombres de los firmantes (SN) y los nombres alternativos de los firmantes (SAN) según las instrucciones proporcionadas para cada tipo de certificado. 

### <a name="create-signing-chain-certificate"></a>Creación de certificados de cadena de firma

Cree estos certificados mediante Windows PowerShell en modo de administrador. **Los certificados creados de esta manera deben usarse solo para fines de prueba o desarrollo.**

El certificado de cadena de firma solo debe crearse una vez. Los demás certificados de punto de conexión harán referencia a este certificado para firmar.
 

```azurepowershell
$cert = New-SelfSignedCertificate -Type Custom -KeySpec Signature -Subject "CN=RootCert" -HashAlgorithm sha256 -KeyLength 2048 -CertStoreLocation "Cert:\LocalMachine\My" -KeyUsageProperty Sign -KeyUsage CertSign
```


### <a name="create-signed-endpoint-certificates"></a>Creación de certificados de punto de conexión firmados

Cree estos certificados mediante Windows PowerShell en modo de administrador.

En estos ejemplos, los certificados de punto de conexión se crean para un dispositivo con:
    - Nombre de dispositivo: `DBE-HWDC1T2`
    - Dominio DNS: `microsoftdatabox.com`

Reemplace el valor por el nombre y el dominio DNS del dispositivo para crear certificados para él.
 
**Certificado de punto de conexión de blob**

Cree un certificado para el punto de conexión de blob en el almacén personal.

```azurepowershell
$AppName = "DBE-HWDC1T2"
$domain = "microsoftdatabox.com"

New-SelfSignedCertificate -Type Custom -DnsName "*.blob.$AppName.$domain" -Subject "CN=*.blob.$AppName.$domain" -KeyExportPolicy Exportable  -HashAlgorithm sha256 -KeyLength 2048  -CertStoreLocation "Cert:\LocalMachine\My" -Signer $cert -KeySpec KeyExchange -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.1")
```

**Certificado de punto de conexión de Azure Resource Manager**

Cree un certificado para los puntos de conexión de Azure Resource Manager en su almacén personal.

```azurepowershell
$AppName = "DBE-HWDC1T2"
$domain = "microsoftdatabox.com"

New-SelfSignedCertificate -Type Custom -DnsName "management.$AppName.$domain","login.$AppName.$domain" -Subject "CN=management.$AppName.$domain" -KeyExportPolicy Exportable  -HashAlgorithm sha256 -KeyLength 2048  -CertStoreLocation "Cert:\LocalMachine\My" -Signer $cert -KeySpec KeyExchange -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.1")
```

**Certificado de interfaz de usuario web local del dispositivo**

Cree un certificado para la interfaz de usuario web local del dispositivo en el almacén personal.

```azurepowershell
$AppName = "DBE-HWDC1T2"
$domain = "microsoftdatabox.com"

New-SelfSignedCertificate -Type Custom -DnsName "$AppName.$domain" -Subject "CN=$AppName.$domain" -KeyExportPolicy Exportable  -HashAlgorithm sha256 -KeyLength 2048  -CertStoreLocation "Cert:\LocalMachine\My" -Signer $cert -KeySpec KeyExchange -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.1")
```

**Certificado único de varias SAN para todos los puntos de conexión**

Cree un certificado único para todos los puntos de conexión en el almacén personal.

```azurepowershell
$AppName = "DBE-HWDC1T2"
$domain = "microsoftdatabox.com"
$DeviceSerial = "HWDC1T2"

New-SelfSignedCertificate -Type Custom -DnsName "$AppName.$domain","$DeviceSerial.$domain","management.$AppName.$domain","login.$AppName.$domain","*.blob.$AppName.$domain" -Subject "CN=$AppName.$domain" -KeyExportPolicy Exportable  -HashAlgorithm sha256 -KeyLength 2048  -CertStoreLocation "Cert:\LocalMachine\My" -Signer $cert -KeySpec KeyExchange -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.1")
```

Una vez creados los certificados, el paso siguiente consiste en cargarlos en el dispositivo Azure Stack Edge Pro.


## <a name="upload-certificates"></a>Carga de certificados 

Los certificados creados para el dispositivo de manera predeterminada residen en **almacén Personal** en el cliente. Estos certificados tienen que exportarse en el cliente en archivos de formato adecuado que se puedan cargar en el dispositivo.

1. El certificado raíz tiene que exportarse como archivo de formato DER con la extensión `.cer`. Para pasos detallados, consulte [Exportación de certificados con formato DER](#export-certificates-as-der-format).
2. Los certificados de punto de conexión se tienen que exportar como archivos *.pfx* con claves privadas. Para pasos detallados, consulte[ Exportación de certificados como un archivo *.pfx* con claves privadas](#export-certificates-as-pfx-format-with-private-key). 
3. Los certificados raíz y de punto de conexión se cargan en el dispositivo con la opción **+ Agregar certificado** de la página Certificados en la interfaz de usuario web local. 

    1. Cargue primero los certificados raíz. En la interfaz de usuario web local, vaya a **Certificados > + Agregar certificado**.

        ![Incorporación del certificado de cadena de firma](media/azure-stack-edge-series-manage-certificates/add-cert-1.png)

    2. A continuación, cargue los certificados de punto de conexión. 

        ![Incorporación del certificado de cadena de firma](media/azure-stack-edge-series-manage-certificates/add-cert-2.png)

        Elija los archivos de certificado en formato *.pfx* y escriba la contraseña que proporcionó al exportar el certificado. El certificado de Azure Resource Manager puede tardar unos minutos en aplicarse.

        Si la cadena de firma no se actualiza primero e intenta cargar los certificados de punto de conexión, se producirá un error.

        ![Error al aplicar el certificado](media/azure-stack-edge-series-manage-certificates/apply-cert-error-1.png)

        Vuelva y cargue el certificado de cadena de firma; después, cargue y aplique los certificados de punto de conexión.

> [!IMPORTANT]
> Si cambian el nombre del dispositivo o el dominio DNS cambian, deben crearse certificados nuevos. Los certificados de cliente y los certificados de dispositivo deben actualizarse con el nombre de dispositivo y dominio DNS nuevos. 

## <a name="import-certificates-on-the-client-accessing-the-device"></a>Importación de certificados en el cliente que accede al dispositivo

Los certificados que ha creado y cargado en el dispositivo tienen que importarse en el cliente de Windows (que accede al dispositivo) en el almacén de certificados adecuado.

1. El certificado raíz que exportó con formato DER debe importarse ahora en las **entidades de certificación raíz de confianza** en el sistema cliente. Para pasos detallados, consulte [Importación de certificados en el almacén de entidades de certificación raíz de confianza](#import-certificates-as-der-format).

2. Los certificados de punto de conexión que exportó como `.pfx` tienen que exportarse como DER con la extensión `.cer`. Este archivo `.cer` se importa entonces en el **almacén de certificados Personal** en el sistema. Para pasos detallados, consulte [Importación de certificados en el almacén de certificados Personal](#import-certificates-as-der-format).

### <a name="import-certificates-as-der-format"></a>Importación de certificados con formato DER

Para importar certificados en un cliente de Windows, realice los pasos siguientes:

1. Haga clic con el botón derecho en el archivo y seleccione **Instalar certificado**. Esta acción inicia el Asistente para importar certificados.

    ![Importación del certificado 1](media/azure-stack-edge-series-manage-certificates/import-cert-1.png)

2. Para **Ubicación de almacén**, seleccione **Máquina local** y, después, **Siguiente**.

    ![Importación del certificado 2](media/azure-stack-edge-series-manage-certificates/import-cert-2.png)

3. Seleccione **Colocar todos los certificados en el siguiente almacén** y, después, **Examinar**. 

    - Para importar en el almacén personal, vaya al almacén Personal del host remoto y seleccione **Siguiente**.

        ![Importación del certificado 4](media/azure-stack-edge-series-manage-certificates/import-cert-4.png)


    - Para importar en el almacén de confianza, vaya a la entidad de certificación raíz de confianza y seleccione **Siguiente**.

        ![Importación del certificado 3](media/azure-stack-edge-series-manage-certificates/import-cert-3.png)

 
4. Seleccione **Finalizar**. Aparece un mensaje que indica que la importación se ha realizado correctamente.

### <a name="export-certificates-as-pfx-format-with-private-key"></a>Exportación de certificados con formato .pfx con clave privada

Siga estos pasos para exportar un certificado SSL con clave privada en una máquina Windows. 

> [!IMPORTANT]
> Siga estos pasos en la misma máquina que usó para crear el certificado. 

1. Ejecute *certlm.msc* para iniciar el almacén de certificados de la máquina local.

1. Haga doble clic en la carpeta **Personal** y, a continuación, en **Certificados**.

    ![Exportación del certificado 1](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-1.png)
 
2. Haga clic con el botón derecho en el certificado del que desea realizar una copia de seguridad y elija **Todas las tareas > Exportar...**

    ![Exportación del certificado 2](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-2.png)

3. Siga el Asistente para exportación de certificados para realizar una copia de seguridad del certificado en un archivo .pfx.

    ![Exportación del certificado 3](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-3.png)

4. Elija **Exportar la clave privada**.

    ![Exportación del certificado 4](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-4.png)

5. Elija **Incluir todos los certificados en la ruta de certificación (si es posible)** , **Exportar todas las propiedades extendidas** y **Habilitar privacidad de certificado**. 

    > [!IMPORTANT]
    > NO seleccione la opción **Delete Private Key option if export is successful** (Eliminar la opción de clave privada si la exportación es correcta).

    ![Exportación del certificado 5](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-5.png)

6. Escriba una contraseña que pueda recordar. Confirme la contraseña. La contraseña protege la clave privada.

    ![Exportación del certificado 6](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-6.png)

7. Elija guardar el archivo en una ubicación establecida.

    ![Exportación del certificado 7](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-7.png)
  
8. Seleccione **Finalizar**.

    ![Exportación del certificado 8](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-8.png)

9. Recibirá un mensaje que indica que la exportación se ha realizado correctamente. Seleccione **Aceptar**.

    ![Exportación del certificado 9](media/azure-stack-edge-series-manage-certificates/export-cert-pfx-9.png)

La copia de seguridad del archivo .pfx se guarda en la ubicación seleccionada y está lista para moverse o almacenarse para mantener la seguridad.


### <a name="export-certificates-as-der-format"></a>Exportación de certificados con formato DER

1. Ejecute *certlm.msc* para iniciar el almacén de certificados de la máquina local.

1. En el almacén de certificados Personal, seleccione el certificado raíz. Haga clic con el botón derecho y seleccione **Todas las tareas -> Exportar...**

    ![Exportación del certificado 1](media/azure-stack-edge-series-manage-certificates/export-cert-cer-1.png)

2. Se abrirá el asistente de certificados. Seleccione el formato **DER binario codificado X.509 (.cer)** . Seleccione **Next** (Siguiente).

    ![Exportación del certificado 2](media/azure-stack-edge-series-manage-certificates/export-cert-cer-2.png)

3. Busque y seleccione la ubicación en la que desea exportar el archivo con formato .cer.

    ![Exportación del certificado 3](media/azure-stack-edge-series-manage-certificates/export-cert-cer-3.png)


4. Seleccione **Finalizar**.

    ![Exportación del certificado 4](media/azure-stack-edge-series-manage-certificates/export-cert-cer-4.png)


## <a name="supported-certificate-algorithms"></a>Algoritmos de certificado admitidos

 En el dispositivo Azure Stack Edge Pro solo se admiten certificados Rivest–Shamir–Adleman (RSA). Si se usan certificados de algoritmo de firma digital de curva elíptica (ECDSA), el comportamiento del dispositivo es indeterminado.

 Los certificados que contienen una clave pública RSA se conocen como certificados RSA. Los certificados que contienen una clave pública criptográfica de curva elíptica (ECC) se conocen como certificados ECDSA (algoritmo de firma digital de curva elíptica). 


## <a name="view-certificate-expiry"></a>Visualización de la expiración del certificado

Si incorpora sus propios certificados, estos expirarán normalmente en 1 año o 6 meses. Para ver la fecha de expiración del certificado, vaya a la página **Certificados** en la interfaz de usuario web local del dispositivo. Si selecciona un certificado específico, verá en él la fecha de expiración.

## <a name="rotate-certificates"></a>Rotación de certificados

No se ha implementado la rotación de certificados en esta versión. Tampoco recibirá una notificación de la fecha de expiración pendiente en el certificado. 

Consulte la fecha de expiración del certificado en la página **Certificados** en la interfaz de usuario web local del dispositivo. Cuando se aproxime la fecha de expiración del certificado, cree y cargue nuevos certificados según las instrucciones detalladas en [Creación y carga de certificados](azure-stack-edge-j-series-manage-certificates.md).

## <a name="next-steps"></a>Pasos siguientes

[Implementación de un dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md)
