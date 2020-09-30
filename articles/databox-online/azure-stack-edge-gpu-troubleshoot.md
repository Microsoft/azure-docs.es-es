---
title: Uso de Azure Portal para solucionar problemas de Azure Stack Edge Pro con GPU | Microsoft Docs
description: Describe cómo solucionar los problemas de Azure Stack Edge Pro con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: troubleshooting
ms.date: 08/19/2020
ms.author: alkohli
ms.openlocfilehash: 9deb10336d959a3f706cfc0ba970c3b83d5cd0a4
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90891547"
---
# <a name="troubleshoot-issues-on-your-azure-stack-edge-pro-gpu-device"></a>Solución de problemas en un dispositivo Azure Stack Edge Pro con GPU 

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

En este artículo se describe cómo solucionar los problemas de un dispositivo Azure Stack Edge Pro con GPU. 


## <a name="run-diagnostics"></a>Ejecución de diagnósticos

Para diagnosticar y solucionar los errores de cualquier dispositivo, puede ejecutar las pruebas de diagnóstico. Realice los pasos siguientes en la interfaz de usuario web local del dispositivo para ejecutar las pruebas de diagnóstico.

1. En la interfaz de usuario de web local, vaya a **Solución de problemas > Pruebas de diagnóstico en**. Seleccione la prueba que quiere ejecutar y elija **Ejecutar prueba**. Así se ejecutan las pruebas necesarias para diagnosticar posibles problemas en la red, el dispositivo, el proxy web, la hora o la configuración de la nube. Aparece una notificación que indica que hay algunas pruebas en ejecución en el dispositivo.

    ![Seleccionar pruebas ](media/azure-stack-edge-gpu-troubleshoot/run-diag-1.png)
 
2. Al terminar las pruebas, se muestran los resultados. 

    ![Ver los resultados de las pruebas](media/azure-stack-edge-gpu-troubleshoot/run-diag-2.png)

    Si se produce un error en una prueba, se especifica una dirección URL en la que se muestra la acción recomendada. Seleccione dicha dirección para ver la acción recomendada.
 
    ![Revisión de las advertencias de las pruebas con errores](media/azure-stack-edge-j-series-troubleshoot/run-diag-3.png)


## <a name="collect-support-package"></a>Recopilación de un paquete de soporte

Los paquetes de registros contienen todos los registros pertinentes que pueden ayudar al equipo de soporte técnico de Microsoft a solucionar los problemas de cualquier dispositivo. Los paquetes de registros se pueden generar a través de la interfaz de usuario web local.

Siga estos pasos para recopilar un paquete de soporte. 

1. En la interfaz de usuario web local, vaya a **Solución de problemas > Soporte**. Seleccione **Crear un paquete de soporte técnico**. El sistema empieza a recopilar el paquete de soporte. Dicha recopilación puede tardar varios minutos.

    ![Seleccionar Agregar usuario](media/azure-stack-edge-gpu-troubleshoot/collect-logs-1.png)
 
2. Una vez que se cree el paquete, haga clic en **Descargar el paquete de soporte técnico**. Se descarga un paquete comprimido en la ruta de acceso que eligió. Puede descomprimir el paquete de registro y ver los archivos de registro del sistema.

    ![Seleccionar Agregar usuario](media/azure-stack-edge-gpu-troubleshoot/collect-logs-2.png)

## <a name="gather-advanced-security-logs"></a>Recopilación de los registros de seguridad avanzada

Los registros de seguridad avanzada pueden ser registros de intrusiones de software o hardware para un dispositivo Azure Stack Edge Pro.

### <a name="software-intrusion-logs"></a>Registros de intrusiones de software

Los registros de intrusiones de software o de firewall predeterminados se recopilan para el tráfico entrante y saliente. 

- Cuando se crea una imagen del dispositivo en la fábrica, el registro de firewall predeterminado está habilitado. Estos registros se agrupan en el paquete de soporte técnico de manera predeterminada cuando se crea un paquete de soporte técnico a través de la interfaz de usuario local o a través de la interfaz de Windows PowerShell del dispositivo.

- Si solo se necesitan los registros del firewall en el paquete de soporte técnico para revisar las intrusiones de software (NW) en el dispositivo, use la opción `-Include FirewallLog` al crear el paquete de soporte. 

- Si no se proporciona ninguna opción de inclusión específica, el registro del firewall se incluye de manera predeterminada en el paquete de soporte técnico.

- En el paquete de soporte, el registro del firewall es `pfirewall.log` y se encuentra en la carpeta raíz. Este es un ejemplo del registro de intrusiones de software para el dispositivo Azure Stack Edge Pro. 

    ```
    #Version: 1.5
    #Software: Microsoft Windows Firewall
    #Time Format: Local
    #Fields: date time action protocol src-ip dst-ip src-port dst-port size tcpflags tcpsyn tcpack tcpwin icmptype icmpcode info path
    
    2019-11-06 12:35:19 DROP UDP 5.5.3.197 224.0.0.251 5353 5353 59 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e88 ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e88 ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e88 ff02::fb 5353 5353 89 - - - - - - 
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9d87 ff02::fb 5353 5353 79 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP 5.5.3.193 224.0.0.251 5353 5353 59 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe08:20d5 ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe08:20d5 ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e8b ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e8b ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP 5.5.3.33 224.0.0.251 5353 5353 59 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e8b ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e8a ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    2019-11-06 12:35:19 DROP UDP fe80::3680:dff:fe01:9e8b ff02::fb 5353 5353 89 - - - - - - - RECEIVE
    ```

### <a name="hardware-intrusion-logs"></a>Registros de intrusiones de hardware

Para detectar cualquier intrusión de hardware en el dispositivo, actualmente se registran todos los eventos de chasis, como la apertura o el cierre del chasis. 

- El registro de eventos del sistema del dispositivo se lee mediante el cmdlet `racadm`. Después, estos eventos se filtran para el evento relacionado con el chasis en un archivo `HWIntrusion.txt`.

- Para obtener solo el registro de intrusiones de hardware en el paquete de soporte técnico, use la opción `-Include HWSelLog` al crear el paquete de soporte. 

- Si no se proporciona ninguna opción de inclusión específica, el registro de intrusiones de hardware se incluye de manera predeterminada en el paquete de soporte técnico.

- En el paquete de soporte, el registro de intrusiones de hardware es `HWIntrusion.txt` y se encuentra en la carpeta raíz. Este es un ejemplo del registro de intrusiones de hardware para el dispositivo Azure Stack Edge Pro. 

    ```
    09/04/2019 15:51:23 system Critical The chassis is open while the power is off.
    09/04/2019 15:51:30 system Ok The chassis is closed while the power is off.
    ```

## <a name="use-logs-to-troubleshoot"></a>Uso de registros para solucionar problemas

Los errores encontrados durante los procesos de carga y actualización se incluyen en los archivos de error respectivos.

1. Para ver los archivos de error, vaya al recurso compartido y seleccione el recurso compartido para ver el contenido. 


2. Seleccione la _carpeta Microsoft Azure Data Box Edge_. Esta carpeta tiene dos subcarpetas:

    - Cargue la que contiene los archivos de registro de errores de carga.
    - Actualice la carpeta para ver los errores durante la actualización.

    Este es un archivo de registro de ejemplo de actualización.

    ```
    <root container="test1" machine="VM15BS020663" timestamp="03/18/2019 00:11:10" />
    <file item="test.txt" local="False" remote="True" error="16001" />
    <summary runtime="00:00:00.0945320" errors="1" creates="2" deletes="0" insync="3" replaces="0" pending="9" />
    ``` 

3. Cuando vea un error en este archivo (resaltado en el ejemplo), anote el código de error, en este caso el 16001. Busque la descripción del código de error en la siguiente referencia de errores.

    [!INCLUDE [data-box-edge-edge-upload-error-reference](../../includes/data-box-edge-gateway-upload-error-reference.md)]

## <a name="use-error-lists-to-troubleshoot"></a>Uso de listas de errores para solucionar problemas

Las listas de errores se compilan a partir de escenarios identificados y se pueden usar para el autodiagnóstico y la solución de problemas. 

## <a name="azure-resource-manager"></a>Azure Resource Manager

Estos son los errores que pueden aparecer durante la configuración de Azure Resource Manager para acceder a su dispositivo. 

| **Problemas o errores** |  **Resolución** | 
|------------|-----------------|
|Problemas generales|<li>[Compruebe que el dispositivo de Edge esté configurado correctamente](#verify-the-device-is-configured-properly).<li> [Compruebe que el cliente esté configurado correctamente](#verify-the-client-is-configured-properly).|
|Add-AzureRmEnvironment: Se produjo un error al enviar la solicitud.<br>At line:1 char:1<br>+ Add-AzureRmEnvironment -Name Az3 -ARMEndpoint "https://management.dbe ...|Este error significa que no se puede acceder el dispositivo Azure Stack Edge Pro o que no está configurado correctamente. Compruebe que el dispositivo de Edge y el cliente estén configurados correctamente. Para obtener una guía, consulte la fila **Problemas generales** en esta tabla.|
|El servicio devolvió un error. Consulte InnerException para obtener más información: Se ha cerrado la conexión subyacente: No se ha podido establecer la relación de confianza para el canal seguro SSL/TLS. |   Este error probablemente se deba a que uno o varios pasos de Traiga su propio certificado no se han completado correctamente. Puede encontrar una guía [aquí](https://docs.microsoft.com/azure/databox-online/azure-stack-edge-j-series-connect-resource-manager#step-2-create-and-install-certificates). |
|La operación devolvió un código de estado no válido "ServiceUnavailable". <br> Response status code does not indicate success: 503 (Servicio no disponible). | Este error puede deberse a cualquiera de las condiciones siguientes.<li>ArmStsPool está en estado detenido.</li><li>Cualquiera de los sitios web de los servicios de token de seguridad o Azure Resource Managers están inactivos.</li><li>El recurso de clúster de Azure Resource Manager está inactivo.</li><br><strong>Nota:</strong> El reinicio del dispositivo podría solucionar el problema, pero debe recopilar el paquete de soporte técnico para que pueda depurarlo aún más.|
|AADSTS50126: Nombre de usuario o contraseña no válidos.<br>Id. de seguimiento: 29317da9-52fc-4ba0-9778-446ae5625e5a<br>Id. de correlación: 1b9752c4-8cbf-4304-a714-8a16527410f4<br>Marca de tiempo: 2019-11-15 09:21:57Z: Error en el servidor remoto: (400) Solicitud incorrecta.<br>At line:1 char:1 |Este error puede deberse a cualquiera de las condiciones siguientes.<li>Para un nombre de usuario y una contraseña no válidos, compruebe que el cliente haya cambiado la contraseña de Azure Portal siguiendo [estos](https://docs.microsoft.com/azure/databox-online/azure-stack-edge-j-series-set-azure-resource-manager-password) pasos y, a continuación, usando la contraseña correcta.<li>Para un identificador de inquilino no válido, el identificador de inquilino es un GUID fijo y debe establecerse en `c0257de7-538f-415c-993a-1b87a031879d`.</li>|
|connect-AzureRmAccount : AADSTS90056: El recurso está deshabilitado o no existe. Compruebe el código de la aplicación para asegurarse de que ha especificado la URL exacta del recurso al que está intentando acceder.<br>Id. de seguimiento: e19bdbc9-5dc8-4a74-85c3-ac6abdfda115<br>Id. de correlación: 75c8ef5a-830e-48b5-b039-595a96488ff9 Marca de tiempo: 2019-11-18 07:00:51Z: Error en el servidor remoto: (400) Incorrecto |Los puntos de conexión de recurso usados en el comando `Add-AzureRmEnvironment` no son correctos.|
|No se pueden obtener los puntos de conexión de la nube.<br>Asegúrese de tener conexión de red. Detalle del error: HTTPSConnectionPool(host='management.dbg-of4k6suvm.microsoftdatabox.com', port=30005): Se superó el máximo de reintentos en la URL: /metadata/endpoints?api-version=2015-01-01 (Caused by SSLError(SSLError("bad handshake: Error([('SSL routines', 'tls_process_server_certificate', 'certificate verify failed')],)",),)) |Este error aparece principalmente en un entorno de Mac/Linux y se debe a los siguientes problemas:<li>No se agregó un certificado de formato PEM al almacén de certificados de Python.</li> |

### <a name="verify-the-device-is-configured-properly"></a>Comprobación de que el dispositivo está configurado correctamente

1. En la interfaz de usuario local, compruebe que la red del dispositivo está configurada correctamente.

2. Compruebe que los certificados están actualizados para todos los puntos de conexión, como se mencionó [aquí](azure-stack-edge-j-series-connect-resource-manager.md#step-2-create-and-install-certificates).

3. Obtenga el punto de conexión de inicio de sesión y administración de Azure Resource Manager en la página **Dispositivo** en la interfaz de usuario local.

4. Compruebe que el dispositivo está activado y registrado en Azure.


### <a name="verify-the-client-is-configured-properly"></a>Comprobación de que el cliente está configurado correctamente

1. Compruebe que esté instalada la versión correcta de PowerShell, como se mencionó [aquí](azure-stack-edge-j-series-connect-resource-manager.md#step-3-install-powershell-on-the-client).

2. Compruebe que estén instalados los módulos de PowerShell correctos, como se mencionó [aquí](azure-stack-edge-j-series-connect-resource-manager.md#step-4-set-up-azure-powershell-on-the-client).

3. Compruebe que se puede comunicar con los puntos de conexión de Azure Resource Manager y de inicio de sesión. Puede intentar hacer ping a los puntos de conexión. Por ejemplo:

   `ping management.28bmdw2-bb9.microsoftdatabox.com`
   `ping login.28bmdw2-bb9.microsoftdatabox.com`
   
   Si no se puede acceder a ellos, agregue entradas de archivo DNS/host como se mencionó [aquí](azure-stack-edge-j-series-connect-resource-manager.md#step-5-modify-host-file-for-endpoint-name-resolution).
   
4. Compruebe que los certificados de cliente estén instalados como se mencionó [aquí](azure-stack-edge-j-series-connect-resource-manager.md#import-certificates-on-the-client-running-azure-powershell).

5. Si el cliente usa PowerShell, se debe habilitar la preferencia de depuración para ver los mensajes detallados mediante la ejecución de este comando de PowerShell. 

    `$debugpreference = "continue"`

## <a name="blob-storage-on-device"></a>Almacenamiento de blobs en el dispositivo 

Estos son los errores relacionados con el almacenamiento de blobs en el dispositivo Azure Stack Edge Pro/Data Box Gateway.

| **Problemas o errores** |  **Resolución** | 
|--------------------|-----------------|
|No se han podido recuperar los recursos secundarios. El valor de uno de los encabezados HTTP no tiene el formato correcto.| En el menú **Editar**, seleccione **API de Azure Stack de destino**. Luego, reinicie el Explorador de Azure Storage.|
|getaddrinfo ENOTFOUND <accountname>.blob.<serialnumber>.microsoftdatabox.com|Compruebe que el nombre del punto de conexión `<accountname>.blob.<serialnumber>.microsoftdatabox.com` se haya agregado al archivo de hosts en esta ruta de acceso: `C:\Windows\System32\drivers\etc\hosts` en Windows o `/etc/hosts` en Linux.|
|No se han podido recuperar los recursos secundarios.<br> Detalles: un certificado autofirmado |Importe el certificado SSL para el dispositivo en el Explorador de Azure Storage: <ol><li>Descargue el certificado de Azure Portal. Para obtener más información, consulte [Descargar certificado](https://docs.microsoft.com/azure/databox/data-box-deploy-copy-data-via-rest#download-certificate).</li><li>En el menú **Editar**, seleccione Certificados SSL y, después, **Importar certificados**.</li></ol>|
|El comando de AzCopy deja de responder durante un minuto antes de mostrar este error:<br>`Failed to enumerate directory https://… The remote name could not be resolved <accountname>.blob.<serialnumber>.microsoftdatabox.com`|Compruebe que el nombre del punto de conexión `<accountname>.blob.<serialnumber>.microsoftdatabox.com` se haya agregado al archivo de hosts en `C:\Windows\System32\drivers\etc\hosts`.|
|El comando de AzCopy deja de responder durante un minuto antes de mostrar este error:<br>`Error parsing source location. The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel`. |Importe el certificado SSL para el dispositivo en el almacén de certificados del sistema. Para obtener más información, consulte [Descargar certificado](https://docs.microsoft.com/azure/databox/data-box-deploy-copy-data-via-rest#download-certificate).|
|El comando de AzCopy deja de responder durante 20 minutos antes de mostrar este error:<br>`Error parsing source location https://<accountname>.blob.<serialnumber>.microsoftdatabox.com/<cntnr>. No such device or address`. |Compruebe que el nombre del punto de conexión `<accountname>.blob.<serialnumber>.microsoftdatabox.com` se haya agregado al archivo de hosts en `/etc/hosts`.|
|El comando de AzCopy deja de responder durante 20 minutos antes de mostrar este error:<br>`Error parsing source location… The SSL connection could not be established`. |Importe el certificado SSL para el dispositivo en el almacén de certificados del sistema. Para obtener más información, consulte [Descargar certificado](https://docs.microsoft.com/azure/databox/data-box-deploy-copy-data-via-rest#download-certificate).|
|El comando de AzCopy deja de responder durante 20 minutos antes de mostrar este error:<br>`Error parsing source location https://<accountname>.blob.<serialnumber>.microsoftdatabox.com/<cntnr>. No such device or address`|Compruebe que el nombre del punto de conexión `<accountname>.blob.<serialnumber>.microsoftdatabox.com` se haya agregado al archivo de hosts en `/etc/hosts`.|
|El comando de AzCopy deja de responder durante 20 minutos antes de mostrar este error: `Error parsing source location… The SSL connection could not be established`.|Importe el certificado SSL para el dispositivo en el almacén de certificados del sistema. Para obtener más información, consulte [Descargar certificado](https://docs.microsoft.com/azure/databox/data-box-deploy-copy-data-via-rest#download-certificate).|
|El valor de uno de los encabezados HTTP no tiene el formato correcto.|Data Box no admite la versión instalada de la biblioteca de Microsoft Azure Storage para Python. Consulte los requisitos de almacenamiento de blobs de Azure Data Box para las versiones compatibles.|
|… [SSL: CERTIFICATE_VERIFY_FAILED]…| Antes de ejecutar Python, establezca la variable de entorno REQUESTS_CA_BUNDLE a la ruta de acceso del archivo de certificado SSL con codificación Base64, consulte [Descargar certificado](https://docs.microsoft.com/azure/databox/data-box-deploy-copy-data-via-rest#download-certificate). Por ejemplo:<br>`export REQUESTS_CA_BUNDLE=/tmp/mycert.cer`<br>`python`<br>O bien, agregue el certificado al almacén de certificados del sistema y, a continuación, establezca esta variable de entorno a la ruta de acceso de dicho almacén. Por ejemplo, en Ubuntu:<br>`export REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt`<br>`python`.|
|Se agota el tiempo de espera de conexión.|Inicie sesión en Azure Stack Edge Pro y compruebe que esté desbloqueado. Cada vez que se reinicia el dispositivo, permanece bloqueado hasta que alguien inicia sesión.|


## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de los [problemas conocidos de esta versión](azure-stack-edge-gpu-2008-release-notes.md).
