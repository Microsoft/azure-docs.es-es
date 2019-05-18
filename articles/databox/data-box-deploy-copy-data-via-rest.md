---
title: Tutorial para copiar datos a Azure Data Box Blob Storage mediante API REST | Microsoft Docs
description: Vea cómo copiar datos a Azure Data Box Blob Storage mediante API REST
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 05/09/2019
ms.author: alkohli
ms.openlocfilehash: b3d53b7d5fceb303259823198731b5198cfce82c
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65508244"
---
# <a name="tutorial-copy-data-to-azure-data-box-blob-storage-via-rest-apis"></a>Tutorial: Copia de datos a Azure Data Box Blob Storage mediante API REST  

En este tutorial se describen los procedimientos para conectarse a Azure Data Box Blob Storage mediante las API REST con *http* o *https*. Una vez conectado, también se describen los pasos necesarios para copiar los datos en Data Box Blob Storage y preparar el dispositivo Data Box para su envío.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Requisitos previos
> * Conexión a Data Box Blob Storage mediante *http* o *https*
> * Copia de datos a un dispositivo Data Box

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que:

1. Ha completado el [Tutorial: Instalación de un dispositivo Azure Data Box](data-box-deploy-set-up.md).
2. Ha recibido su dispositivo Data Box y el estado del pedido en el portal se actualiza a **Delivered** (Entregado).
3. Ha revisado los [requisitos del sistema para Data Box Blob Storage](data-box-system-requirements-rest.md) y está familiarizado con las versiones compatibles de las API, los SDK y las herramientas.
4. Tiene acceso a un equipo host con los datos que desea copiar en su dispositivo Data Box. El equipo host debe:
    - Ejecutar un [sistema operativo admitido](data-box-system-requirements.md).
    - Estar conectado a una red de alta velocidad. Es muy recomendable tener una conexión de 10 GbE como mínimo. Si no hay disponible una conexión de 10 GbE, se puede usar un vínculo de datos de 1 GbE, pero las velocidades de copia se verán afectadas.
5. [Descargue AzCopy 7.1.0](https://aka.ms/azcopyforazurestack20170417) en el equipo host. Usará AzCopy para copiar los datos desde el equipo host a Azure Data Box Blob Storage.


## <a name="connect-via-http-or-https"></a>Conexión mediante http o https

Puede conectarse a Data Box Blob Storage mediante *http* o *https*.

- *Https* es la forma segura y recomendada de conectarse al almacenamiento de blobs de Data Box.
- *Http* se utiliza para conectarse en redes de confianza.

Los pasos para conectarse son diferentes cuando se conecta a Data Box Blob Storage mediante *http* o *https*.

## <a name="connect-via-http"></a>Conexión mediante http

Estos son los pasos para conectarse a las API REST de Data Box Blob Storage mediante *http*:

- Adición de la dirección IP de dispositivo y el punto de conexión de Blob service al host remoto
- Configuración del software de terceros y comprobación de la conexión

En las secciones siguientes se describen todos estos pasos.

### <a name="add-device-ip-address-and-blob-service-endpoint"></a>Adición de la dirección IP del dispositivo y el punto de conexión de Blob service

[!INCLUDE [data-box-add-device-ip](../../includes/data-box-add-device-ip.md)]

### <a name="configure-partner-software-and-verify-connection"></a>Configuración del software de partners y comprobación de la conexión

[!INCLUDE [data-box-configure-partner-software](../../includes/data-box-configure-partner-software.md)]

[!INCLUDE [data-box-verify-connection](../../includes/data-box-verify-connection.md)]

## <a name="connect-via-https"></a>Conexión mediante https

Estos son los pasos para conectarse a las API REST de Azure Blob Storage mediante https:

- Descarga del certificado desde Azure Portal
- Importación del certificado en el cliente o el host remoto
- Adición de la dirección IP del dispositivo y el punto de conexión de Blob service al cliente o al host remoto
- Configuración del software de terceros y comprobación de la conexión

En las secciones siguientes se describen todos estos pasos.

### <a name="download-certificate"></a>Descargar certificado

Use Azure Portal para descargar el certificado.

1. Inicie sesión en Azure Portal.
2. Vaya a su pedido de Data Box y, después, vaya a **General > Detalles del dispositivo**.
3. En **Credenciales del dispositivo**, vaya a **Acceso de API** al dispositivo. Haga clic en **Descargar**. Esta acción descarga un archivo de certificado **\<nombre del pedido>.cer**. **Guarde** este archivo. Instalará este certificado en el equipo cliente o host que usará para conectarse al dispositivo.

    ![Descarga del certificado en Azure Portal](media/data-box-deploy-copy-data-via-rest/download-cert-1.png)
 
### <a name="import-certificate"></a>Importación de certificado 

Para acceder al almacenamiento de blobs de Data Box a través de HTTPS, el dispositivo necesita un certificado SSL. La forma en que este certificado se pone a disposición de la aplicación cliente varía de una aplicación a otra y entre sistemas operativos y distribuciones. Algunas aplicaciones pueden acceder al certificado después de que se importa en el almacén de certificados del sistema, mientras que otras no hacen uso de ese mecanismo.

En esta sección se menciona información específica para algunas aplicaciones. Para más información sobre otras aplicaciones, consulte la documentación de la aplicación y el sistema operativo usado.

Siga estos pasos para importar el archivo `.cer` en el almacén raíz de un cliente de Windows o Linux. En un sistema Windows, puede usar Windows PowerShell o la interfaz de usuario de Windows Server para importar e instalar el certificado en el sistema.

#### <a name="use-windows-powershell"></a>Uso de Windows PowerShell

1. Iniciar una sesión de Windows PowerShell como administrador.
2. En el símbolo del sistema, escriba:

    ```
    Import-Certificate -FilePath C:\temp\localuihttps.cer -CertStoreLocation Cert:\LocalMachine\Root
    ```

#### <a name="use-windows-server-ui"></a>Uso de la interfaz de usuario de Windows Server

1.  Haga clic con el botón derecho en el archivo `.cer` y seleccione **Instalar certificado**. Esta acción inicia el Asistente para importar certificados.
2.  Para **Ubicación de almacén**, seleccione **Equipo local** y, después, haga clic en **Siguiente**.

    ![Importación del certificado mediante PowerShell](media/data-box-deploy-copy-data-via-rest/import-cert-ws-1.png)

3.  Seleccione **Colocar todos los certificados en el siguiente almacén** y, luego, haga clic en **Examinar**. Navegue al almacén raíz del host remoto y, a continuación, haga clic en **Siguiente**.

    ![Importación del certificado mediante PowerShell](media/data-box-deploy-copy-data-via-rest/import-cert-ws-2.png)

4.  Haga clic en **Finalizar** Aparece un mensaje que indica que la importación se ha realizado correctamente.

    ![Importación del certificado mediante PowerShell](media/data-box-deploy-copy-data-via-rest/import-cert-ws-3.png)

#### <a name="use-a-linux-system"></a>Uso de un sistema Linux

El método para importar un certificado varía según la distribución.

Varias, como Ubuntu y Debian, usan el comando `update-ca-certificates`.  

- Cambie el nombre del archivo de certificado codificado en Base64 para tener una extensión `.crt` y cópielo en `/usr/local/share/ca-certificates directory`.
- Ejecute el comando `update-ca-certificates`.

Las versiones recientes de RHEL, Fedora y CentOS usan el comando `update-ca-trust`.

- Copie el archivo de certificado en el directorio `/etc/pki/ca-trust/source/anchors`.
- Ejecute `update-ca-trust`.

Para más información, consulte la documentación específica para su distribución.

### <a name="add-device-ip-address-and-blob-service-endpoint"></a>Adición de la dirección IP del dispositivo y el punto de conexión de Blob service 

Siga los mismos pasos para [agregar la dirección IP del dispositivo y el punto de conexión de Blob service cuando se conecta mediante *http*](#add-device-ip-address-and-blob-service-endpoint).

### <a name="configure-partner-software-and-verify-connection"></a>Configuración del software de partners y comprobación de la conexión

Siga los pasos para [configurar el software de asociado que usó durante la conexión mediante *http*](#configure-partner-software-and-verify-connection). La única diferencia es que debe dejar la opción *Usar http* desactivada.

## <a name="copy-data-to-data-box"></a>Copia de datos a un dispositivo Data Box

Una vez que esté conectado a Data Box Blob Storage, el siguiente paso es copiar los datos. Antes de copiar los datos, revise las consideraciones siguientes:

-  Al copiar los datos, asegúrese de que el tamaño de los datos se ajusta a los límites descritos en los [límites de almacenamiento de Azure y de Data Box](data-box-limits.md).
- Si los datos que el dispositivo Data Box va a cargar los están cargando a la vez otras aplicaciones fuera del dispositivo Data Box, podría provocar errores en el trabajo de carga y daños en los datos.

En este tutorial, se usa AzCopy para copiar los datos al Data Box Blob Storage. También puede usar el Explorador de Azure Storage (si lo prefiere una herramienta basada en GUI) o un software de partner para copiar los datos.
El procedimiento de copia incluye los siguientes pasos:

- Crear un contenedor
- Cargar el contenido de una carpeta en Data Box Blob Storage
- Cargar los archivos modificados en Data Box Blob Storage

En las siguientes secciones se describen estos pasos.

### <a name="create-a-container"></a>Crear un contenedor

El primer paso es crear un contenedor, ya que los blobs siempre se cargan en un contenedor. Los contenedores organizan grupos de blobs de la misma forma que se organizan archivos en las carpetas de un equipo. Siga estos pasos para crear un contenedor de blobs:

1. Abra el Explorador de Storage.
2. En el panel izquierdo, expanda la cuenta de almacenamiento en la que desea crear el contenedor de blobs.
3. Haga clic con el botón derecho en **Contenedores de blobs** y, en el menú contextual, seleccione **Crear contenedor de blobs**.

   ![Menú contextual Crear contenedores de blobs](media/data-box-deploy-copy-data-via-rest/create-blob-container-1.png)

4. Aparecerá un cuadro de texto debajo de la carpeta **Contenedores de blobs**. Escriba el nombre del contenedor de blobs. Para más información sobre las reglas y restricciones en la nomenclatura de los contenedores de blobs, consulte [Creación del contenedor y establecimiento de los permisos](../storage/blobs/storage-quickstart-blobs-dotnet.md).
5. Presione **ENTRAR** cuando termine para crear el contenedor de blobs o **Esc** para cancelar la operación. Una vez que el contenedor de blobs se ha creado correctamente, se mostrará en la carpeta **Contenedores de blobs** de la cuenta de almacenamiento seleccionada.

   ![Contenedor de blobs creado](media/data-box-deploy-copy-data-via-rest/create-blob-container-2.png)

### <a name="upload-contents-of-a-folder-to-data-box-blob-storage"></a>Cargar el contenido de una carpeta en Data Box Blob Storage

Use AzCopy para cargar todos los archivos en una carpeta en Blob Storage en Windows o Linux. Para cargar todos los blobs en una carpeta, escriba el siguiente comando de AzCopy:

#### <a name="linux"></a>Linux

    azcopy \
        --source /mnt/myfolder \
        --destination https://data-box-storage-account-name.blob.device-serial-no.microsoftdatabox.com/container-name/files/ \
        --dest-key <key> \
        --recursive

#### <a name="windows"></a> Windows

    AzCopy /Source:C:\myfolder /Dest:https://data-box-storage-account-name.blob.device-serial-no.microsoftdatabox.com/container-name/files/ /DestKey:<key> /S


Reemplace `<key>` por la clave de cuenta. Para obtener la clave de cuenta, en Azure Portal, vaya a su cuenta de almacenamiento. Vaya a **Configuración > Claves de acceso**, seleccione una clave y péguela en el comando de AzCopy.

Si el contenedor de destino especificado no existe, AzCopy lo crea y carga el archivo en él. Actualice la ruta de acceso de origen al directorio de datos y reemplace `data-box-storage-account-name` en la dirección URL de destino por el nombre de la cuenta de almacenamiento asociada a su dispositivo Data Box.

Para cargar el contenido del directorio especificado repetidamente en Blob Storage, especifique la opción `--recursive` (Linux) o `/S` (Windows). Al ejecutar AzCopy con una de estas opciones, se cargan también todas las subcarpetas y sus archivos.

### <a name="upload-modified-files-to-data-box-blob-storage"></a>Cargar los archivos modificados en Data Box Blob Storage

Use AzCopy para cargar archivos según la hora de la última modificación. Para probar esto, modifique o cree archivos en el directorio de origen para fines de prueba. Para cargar solo los archivos nuevos o actualizados, agregue el parámetro `--exclude-older` (Linux) o `/XO` (Windows) al comando AzCopy.

Si solo desea copiar los recursos de origen que no existen en el destino, especifique ambos parámetros `--exclude-older` y `--exclude-newer` (Linux) o `/XO` y `/XN` (Windows) en el comando de AzCopy. AzCopy carga solo los datos actualizados, en función de su marca de tiempo.

#### <a name="linux"></a>Linux
    azcopy \
    --source /mnt/myfolder \
    --destination https://data-box-storage-account-name.blob.device-serial-no.microsoftdatabox.com/container-name/files/ \
    --dest-key <key> \
    --recursive \
    --exclude-older

#### <a name="windows"></a> Windows

    AzCopy /Source:C:\myfolder /Dest:https://data-box-storage-account-name.blob.device-serial-no.microsoftdatabox.com/container-name/files/ /DestKey:<key> /S /XO

El siguiente paso consiste en preparar el dispositivo para el envío.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información acerca de varios temas relacionados con Azure Data Box, como:

> [!div class="checklist"]
> * Requisitos previos
> * Conexión a Data Box Blob Storage mediante *http* o *https*
> * Copia de datos a un dispositivo Data Box


En el siguiente tutorial aprenderá a enviar su dispositivo Data Box a Microsoft.

> [!div class="nextstepaction"]
> [Envío de un dispositivo Data Box a Microsoft](./data-box-deploy-picked-up.md)
