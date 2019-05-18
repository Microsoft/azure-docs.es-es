---
title: Realizar un seguimiento y registrar los eventos de Azure Data Box | Microsoft Docs
description: Describe cómo realizar un seguimiento y registro de eventos en las distintas fases de su pedido de Azure Data Box.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 05/14/2019
ms.author: alkohli
ms.openlocfilehash: 7a6adc72c1dfbe67311ae2ca98d5b07dfab41719
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65806512"
---
# <a name="tracking-and-event-logging-for-your-azure-data-box"></a>Seguimiento y registro de eventos para Azure Data Box

Un pedido de Data Box se pasa a través de los pasos siguientes: pedidos, configurar, datos copiarán, devolverán, cargar en Azure y comprobación y eliminación de datos. Corresponde a cada paso en el orden, puede realizar varias acciones para controlar el acceso a la orden, los eventos de auditoría, controlar el orden e interpretar los registros distintos que se generan.

En la tabla siguiente se muestra un resumen de los pasos de pedido de Data Box y las herramientas disponibles para realizar un seguimiento y el orden de auditoría durante cada paso.

| Fase de pedido del cuadro de datos       | Herramienta para realizar un seguimiento y de auditoría                                                                        |
|----------------------------|------------------------------------------------------------------------------------------------|
| Crear pedido               | [Configurar el control de acceso en el orden a través de RBAC](#set-up-access-control-on-the-order)                                                    |
| Pedido procesado            | [Controlar el orden](#track-the-order) a través de <ul><li> Azure Portal </li><li> Sitio Web de transportista de envío </li><li>Notificaciones por correo electrónico</ul> |
| Configuración de un dispositivo              | Dispositivo las credenciales de acceso que ha iniciado sesión [los registros de actividad](#query-activity-logs-during-setup)                                              |
| Copia de datos en el dispositivo        | [Vista *error.xml* archivos](#view-error-log-during-data-copy-to-data-box) para copia de datos                                                             |
| Preparar para enviar            | [Inspeccionar los archivos de la lista de materiales](#inspect-bom-during-prepare-to-ship) o los archivos de manifiesto en el dispositivo                                      |
| Carga de datos en Azure       | [Revisión *copylogs* ](#review-copy-log-during-upload-to-azure) errores durante los datos de carga en el centro de datos de Azure                         |
| Eliminación de datos del dispositivo   | [Ver la cadena de custodia registros](#get-chain-of-custody-logs-after-data-erasure) incluidos registros de auditoría y el orden del historial                                                   |

En este artículo se describe detalladamente los distintos mecanismos o herramientas disponibles para realizar un seguimiento y auditar el pedido de Data Box.

## <a name="set-up-access-control-on-the-order"></a>Configurar el control de acceso en el orden

Puede controlar quién puede acceder a su pedido cuando se crea por primera vez el orden. Configurar las funciones de Control de acceso basado en roles (RBAC) en varios ámbitos para controlar el acceso para el pedido de Data Box. Un rol de RBAC determina el tipo de acceso: lectura y escritura, de solo lectura, lectura y escritura para un subconjunto de operaciones.

Las dos funciones de Data Box que se pueden definir son:

- **Lector de datos de cuadro** -tienen acceso de solo lectura para un pedido según se define en el ámbito. Solo pueden ver los detalles de un pedido. No pueden tener acceso a los otros detalles relacionados con las cuentas de almacenamiento ni editar los detalles del pedido como la dirección y así sucesivamente.
- **Colaborador de datos de cuadro** -solo puede crear un pedido para transferir datos a una cuenta de almacenamiento determinado *si ya tienen acceso de escritura a una cuenta de almacenamiento*. Si no tiene acceso a una cuenta de almacenamiento, incluso no pueden crear un pedido de Data Box para copiar datos a la cuenta. Este rol no define ninguna cuenta de almacenamiento relacionados con los permisos ni concede acceso a las cuentas de almacenamiento.  

Para restringir el acceso a un pedido, hacer lo siguiente:

- Asigne un rol en el nivel de pedido. El usuario solo tiene los permisos tal como se define por los roles para interactuar con ese orden específico de Data Box y nada más.
- Asignar un rol en el nivel de grupo de recursos, el usuario tiene acceso a todos los pedidos de Data Box dentro de un grupo de recursos.

Para obtener más información sobre el uso RBAC sugerido, vea [procedimientos recomendados para RBAC](../role-based-access-control/overview.md#best-practice-for-using-rbac).

## <a name="track-the-order"></a>Seguimiento del pedido

Puede realizar un seguimiento de su pedido a través de Azure portal y el sitio Web del transportista de envío. Los mecanismos siguientes están en su lugar para controlar el orden de Data Box en cualquier momento:

- Para controlar el orden cuando el dispositivo está en el centro de datos de Azure o su entorno local, vaya a su **pedido de Data Box > información general sobre** en Azure portal.

    ![Ver estado del pedido y no de seguimiento](media/data-box-logs/overview-view-status-1.png)

- Para controlar el orden mientras el dispositivo está en tránsito, visite el sitio Web de operador regional, por ejemplo, sitio Web de SAI (UPS) en MICROSEGUNDOS. Proporcione el número de seguimiento asociado con su pedido.
- Cuadro de datos también envía notificaciones por correo electrónico cada vez que los cambios de estado de pedido en función de los correos electrónicos que se proporcionó cuando se creó el pedido. Para obtener una lista de todos los Estados de pedido de Data Box, consulte [ver estado del pedido](data-box-portal-admin.md#view-order-status). Para cambiar la configuración de notificación asociada con el pedido, vea [editar detalles de la notificación](data-box-portal-admin.md#edit-notification-details).

## <a name="query-activity-logs-during-setup"></a>Registros de actividad de consultas durante la instalación

- Su Data Box llega en sus instalaciones en un estado bloqueado. Puede usar las credenciales de dispositivo disponibles en el portal de Azure para su pedido.  

    Cuando se configura un cuadro de datos, deberá saber quién todos accedió a las credenciales del dispositivo. Para averiguar quién ha accedido a la **las credenciales del dispositivo** hoja, puede consultar los registros de actividad.  Cualquier acción que implica el acceso a **detalles del dispositivo > credenciales** hoja se registra en los registros de actividad como `ListCredentials` acción.

    ![Consulta de los registros de actividad](media/data-box-logs/query-activity-log-1.png)

- Cada inicio de sesión en el cuadro de datos está registrada en tiempo real. Sin embargo, esta información sólo está disponible en el [registros de auditoría](#audit-logs) después de que el pedido se ha completado correctamente.

## <a name="view-error-log-during-data-copy-to-data-box"></a>Ver registro de errores durante la copia de datos en Data Box

Durante la copia de datos en Data Box, se genera un archivo de error si hay algún problema con los datos copiados.

### <a name="errorxml-file"></a>Archivo error.Xml

Asegúrese de que los trabajos de copia han finalizado sin errores. Si hay errores durante el proceso de copia, descargar los registros desde el **Connect y copie** página.

- Si copió un archivo que no tiene 512 bytes alineados en una carpeta de disco administrado en el cuadro de datos, el archivo no se carga como blob en páginas para la cuenta de almacenamiento provisional. Verá un error en los registros. Quite el archivo y copie uno que tenga 512 bytes alineados.
- Si copió un VHDX, o un disco duro virtual dinámico o un VHD de diferenciación (no se admiten estos archivos), verá un error en los registros.

Este es un ejemplo de la *error.xml* para distintos errores al copiar a discos administrados.

```xml
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\differencing-vhd-022019.vhd</file>
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\dynamic-vhd-022019.vhd</file>
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\insidefixedvhdx-022019.vhdx</file>
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\insidediffvhd-022019.vhd</file>
```

Este es un ejemplo de la *error.xml* errores diferentes al copiar a los blobs en páginas.

```xml
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File100Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File786Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File513Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File10Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File500Bytes</file>
```


Este es un ejemplo de la *error.xml* errores diferentes al copiar a los blobs en bloques.

```xml
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\ab</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\invalid dns name</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\morethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortesting</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\testdirectory-~!@#$%^&amp;()_+{}</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\test__doubleunderscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\-startingwith-hyphen</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\Starting with Capital</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\_startingwith_underscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\55555555--4444--3333--2222--111111111111</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\1</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\11111111-_2222-_3333-_4444-_555555555555</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\test--doublehyphen</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5Ni3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMS3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMy3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_CONTROL">\InvalidUnicodeFiles\Ã.txt</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwNS3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5OS3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMi3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwNC3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5OC3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMC3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5Ny3vv70=</file>
```

Este es un ejemplo de la *error.xml* errores diferentes al copiar a Azure Files.

```xml
<file error="ERROR_BLOB_OR_FILE_SIZE_LIMIT">\AzFileMorethan1TB\AzFile1.2TB</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\testdirectory-~!@#$%^&amp;()_+{}</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\55555555--4444--3333--2222--111111111111</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\-startingwith-hyphen</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\11111111-_2222-_3333-_4444-_555555555555</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\test--doublehyphen</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\ab</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\invalid dns name</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\test__doubleunderscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\morethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortesting</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\_startingwith_underscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\1</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\Starting with Capital</file>
```

En cada uno de los casos anteriores, resuelva los errores antes de continuar con el paso siguiente. Para obtener más información sobre los errores recibidos durante la copia de datos en el cuadro de datos a través de los protocolos SMB o NFS, vaya a [problemas de solución de problemas de Data Box](data-box-troubleshoot.md). Para obtener información sobre los errores recibidos durante la copia de datos en el cuadro de datos a través de REST, vaya a [problemas de almacenamiento de la solución de problemas de datos cuadro Blob](data-box-troubleshoot-rest.md).

## <a name="inspect-bom-during-prepare-to-ship"></a>Inspeccionar BOM durante la preparación para el envío

Durante la preparación para el envío, una lista de archivos que se conoce como la lista de materiales (BOM) o se crea el archivo de manifiesto.

- Use este archivo para comprobar los nombres reales y el número de archivos que se copiaron en el cuadro de datos.
- Use este archivo para comprobar los tamaños reales de los archivos.
- Compruebe que la *crc64* corresponde a una cadena distinta de cero. <!--A null value for crc64 indicates that there was a reparse point error)-->

Para obtener más información sobre los errores recibidos durante la preparación enviar, vaya a [problemas de solución de problemas de Data Box](data-box-troubleshoot.md).

### <a name="bom-or-manifest-file"></a>Archivo BOM o manifiesto

La marca BOM o el archivo de manifiesto contiene la lista de todos los archivos que se copian en el dispositivo Data Box. El archivo BOM tiene nombres de archivo y los tamaños correspondientes, así como la suma de comprobación. Se crea un archivo independiente de la lista de materiales para los blobs en bloques, blobs en páginas, archivos de Azure, para copiar a través de las API de REST y para la copia de discos administrados en el cuadro de datos. Puede descargar los archivos de lista de materiales de la interfaz de usuario del dispositivo de web local durante la preparación para el envío.

Estos archivos también residen en el dispositivo Data Box y se cargan en la cuenta de almacenamiento asociada en el centro de datos de Azure.

### <a name="bom-file-format"></a>Formato de archivo de lista de materiales

BOM o manifiesto de archivo tiene el formato general siguiente:

`<file size = "file-size-in-bytes" crc64="cyclic-redundancy-check-string">\folder-path-on-data-box\name-of-file-copied.md</file>`

Este es un ejemplo de un manifiesto que se genera cuando los datos se copiaron en el recurso compartido de blobs de bloque en el cuadro de datos.

```
<file size="10923" crc64="0x51c78833c90e4e3f">\databox\media\data-box-deploy-copy-data\connect-shares-file-explorer1.png</file>
<file size="15308" crc64="0x091a8b2c7a3bcf0a">\databox\media\data-box-deploy-copy-data\get-share-credentials2.png</file>
<file size="53486" crc64="0x053da912fb45675f">\databox\media\data-box-deploy-copy-data\nfs-client-access.png</file>
<file size="6093" crc64="0xadb61d0d7c6d4deb">\databox\data-box-cable-options.md</file>
<file size="6499" crc64="0x080add29add367d9">\databox\data-box-deploy-copy-data-via-nfs.md</file>
<file size="11089" crc64="0xc3ce6b13a4fe3001">\databox\data-box-deploy-copy-data-via-rest.md</file>
<file size="7749" crc64="0xd2e346a4588e307a">\databox\data-box-deploy-ordered.md</file>
<file size="14275" crc64="0x260296e5d1b1608a">\databox\data-box-deploy-copy-data.md</file>
<file size="4077" crc64="0x2bb0a170225bceec">\databox\data-box-deploy-picked-up.md</file>
<file size="15447" crc64="0xcec0ca8527720b3c">\databox\data-box-portal-admin.md</file>
<file size="9126" crc64="0x820856b5a54321ad">\databox\data-box-overview.md</file>
<file size="10963" crc64="0x5e9a14f9f4784fd8">\databox\data-box-safety.md</file>
<file size="5941" crc64="0x8631d62fbc038760">\databox\data-box-security.md</file>
<file size="12536" crc64="0x8c8ff93e73d665ec">\databox\data-box-system-requirements-rest.md</file>
<file size="3220" crc64="0x7257a263c434839a">\databox\data-box-system-requirements.md</file>
```

La marca BOM o los archivos de manifiesto también se copian en la cuenta de almacenamiento de Azure. Puede usar la marca BOM o los archivos para comprobar que los archivos cargados en Azure coinciden con los datos que se copian en el cuadro de datos de manifiesto.

## <a name="review-copy-log-during-upload-to-azure"></a>Revise el registro de copia durante la carga en Azure

Durante la carga de datos en Azure, un *copylog* se crea.

### <a name="copylog"></a>Copylog

Para cada pedido que se procesa, se crea el servicio de Data Box *copylog* en la cuenta de almacenamiento asociada. El *copylog* tiene el número total de archivos que se han cargado y el número de archivos es cerrados durante los datos de copiar de Data Box en su cuenta de almacenamiento de Azure.

Se realiza un cálculo de comprobación de redundancia cíclica (CRC) durante la carga en Azure. El CRC desde la copia de datos y después se comparan con la carga de datos. Un error de coincidencia CRC indica que no se pudo cargar los archivos correspondientes.

De forma predeterminada, los registros se escriben en un contenedor denominado copylog. Los registros se almacenan con la convención de nomenclatura siguiente:

`storage-account-name/databoxcopylog/ordername_device-serial-number_CopyLog_guid.xml` 

La ruta de acceso copylog también se muestra en el **Introducción** hoja para el portal.

![Ruta de acceso a copylog en la hoja de información general cuando complete](media/data-box-logs/copy-log-path-1.png)

El ejemplo siguiente describe el formato general de un archivo copylog para un cuadro de datos de carga que ha realizado correctamente:

```
<?xml version="1.0"?>
-<CopyLog Summary="Summary">
<Status>Succeeded</Status>
<TotalFiles>45</TotalFiles>
<FilesErrored>0</FilesErrored>
</CopyLog>
```

También puede finalizar la carga en Azure con errores.

![Ruta de acceso a copylog en la hoja de información general cuando se completó con errores](media/data-box-logs/copy-log-path-2.png)

Este es un ejemplo de un copylog donde la carga se completó con errores:

```xml
<ErroredEntity Path="iso\samsungssd.iso">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>409</ErrorCode>
  <ErrorMessage>The blob type is invalid for this operation.</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity Path="iso\iSCSI_Software_Target_33.iso">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>409</ErrorCode>
  <ErrorMessage>The blob type is invalid for this operation.</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><CopyLog Summary="Summary">
  <Status>Failed</Status>
  <TotalFiles_Blobs>72</TotalFiles_Blobs>
  <FilesErrored>2</FilesErrored>
</CopyLog>
```


## <a name="get-chain-of-custody-logs-after-data-erasure"></a>Obtener cadena de registros de custodia después de la eliminación de datos

Una vez que se borran los datos de los discos de datos cuadro según las directrices de revisión 1 de NIST SP 800-88, la cadena de custodia registros están disponibles. Estos registros incluyen los registros de auditoría y el historial de pedidos. También se copian los archivos de manifiesto o l. MAT con los registros de auditoría.

### <a name="audit-logs"></a>Registros de auditoría

Los registros de auditoría contienen información acerca de encendido y compartir el acceso en el cuadro de datos cuando está fuera del centro de datos de Azure. Estos registros se encuentran en: `storage-account/azuredatabox-chainofcustodylogs`

Este es un ejemplo del registro de auditoría de un cuadro de datos:

```
9/10/2018 8:23:01 PM : The operating system started at system time ‎2018‎-‎09‎-‎10T20:23:01.497758400Z.
9/10/2018 8:23:42 PM : An account was successfully logged on.
Subject:
    Security ID:        S-1-5-18
    Account Name:       WIN-DATABOXADMIN
    Account Domain: Workgroup
    Logon ID:       0x3E7
Logon Information:
    Logon Type:     3
    Restricted Admin Mode:  -
    Virtual Account:        No
    Elevated Token:     No
Impersonation Level:        Impersonation
New Logon:
    Security ID:        S-1-5-7
    Account Name:       ANONYMOUS LOGON
    Account Domain: NT AUTHORITY
    Logon ID:       0x775D5
    Linked Logon ID:    0x0
    Network Account Name:   -
    Network Account Domain: -
    Logon GUID:     {00000000-0000-0000-0000-000000000000}
Process Information:
    Process ID:     0x4
    Process Name:       
Network Information:
    Workstation Name:   -
    Source Network Address: -
    Source Port:        -
Detailed Authentication Information:
    Logon Process:      NfsSvr
    Authentication Package:MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
    Transited Services: -
    Package Name (NTLM only):   -
    Key Length:     0
This event is generated when a logon session is created. It is generated on the computer that was accessed. 
The subject fields indicate the account on the local system which requested the logon. This is most commonly a service such as the Server service, or a local process such as Winlogon.exe or Services.exe. 
The logon type field indicates the kind of logon that occurred. The most common types are 2 (interactive) and 3 (network).
The New Logon fields indicate the account for whom the new logon was created, i.e. the account that was logged on.
The network fields indicate where a remote logon request originated. Workstation name is not always available and may be left blank in some cases.
The impersonation level field indicates the extent to which a process in the logon session can impersonate.
The authentication information fields provide detailed information about this specific logon request.
    - Logon GUID is a unique identifier that can be used to correlate this event with a KDC event.
    - Transited services indicate which intermediate services have participated in this logon request.
    - Package name indicates which sub-protocol was used among the NTLM protocols.
    - Key length indicates the length of the generated session key. This will be 0 if no session key was requested.
9/10/2018 8:25:58 PM : An account was successfully logged on.
```


## <a name="download-order-history"></a>Descargar el historial de pedidos

Historial de pedidos está disponible en Azure portal. Si el pedido está completo y se complete la limpieza de dispositivos (eliminación de datos de los discos), a continuación, vaya a **pedido de Data Box > detalles de pedidos**. ** Descargar historial de pedidos** opción está disponible. Para obtener más información, consulte [descargar historial de pedidos](data-box-portal-admin.md#download-order-history).

Si se desplaza por el historial de pedidos, verá:

- Operador de información de seguimiento para el dispositivo.
- Los eventos con *SecureErase* actividad. Estos eventos se corresponden con el borrado de los datos en el disco.
- Vínculos de registro del cuadro de datos. Las rutas de acceso para el *registros de auditoría*, *copylogs*, y *BOM* se presentan los archivos.

Este es un ejemplo del registro de historial de pedido de Azure portal:

```
-------------------------------
Microsoft Data Box Order Report
-------------------------------
Name                                               : gus-pinto                              
StartTime(UTC)                              : 9/19/2018 8:49:23 AM +00:00                       
DeviceType                                     : DataBox                                           
-------------------
Data Box Activities
-------------------
Time(UTC)                 | Activity                       | Status          | Description

9/19/2018 8:49:26 AM      | OrderCreated                   | Completed       |
10/2/2018 7:32:53 AM      | DevicePrepared                 | Completed       |
10/3/2018 1:36:43 PM      | ShippingToCustomer             | InProgress      | Shipment picked up. Local Time : 10/3/2018 1:36:43 PM at AMSTERDAM-NLD                                                                                
10/4/2018 8:23:30 PM      | ShippingToCustomer             | InProgress      | Processed at AMSTERDAM-NLD. Local Time : 10/4/2018 8:23:30 PM at AMSTERDAM-NLD                                                                        
10/4/2018 11:43:34 PM     | ShippingToCustomer             | InProgress      | Departed Facility in AMSTERDAM-NLD. Local Time : 10/4/2018 11:43:34 PM at AMSTERDAM-NLD
10/5/2018 8:13:49 AM      | ShippingToCustomer             | InProgress      | Arrived at Delivery Facility in BRIGHTON-GBR. Local Time : 10/5/2018 8:13:49 AM at LAMBETH-GBR                                                         
10/5/2018 9:13:24 AM      | ShippingToCustomer             | InProgress      | With delivery courier. Local Time : 10/5/2018 9:13:24 AM at BRIGHTON-GBR                                                                               
10/5/2018 12:03:04 PM     | ShippingToCustomer             | Completed       | Delivered - Signed for by. Local Time : 10/5/2018 12:03:04 PM at BRIGHTON-GBR                                                                          
1/25/2019 3:19:25 PM      | ShippingToDataCenter           | InProgress      | Shipment picked up. Local Time : 1/25/2019 3:19:25 PM at BRIGHTON-GBR                                                                                       
1/25/2019 8:03:55 PM      | ShippingToDataCenter           | InProgress      | Processed at BRIGHTON-GBR. Local Time : 1/25/2019 8:03:55 PM at LAMBETH-GBR                                                                            
1/25/2019 8:04:58 PM      | ShippingToDataCenter           | InProgress      | Departed Facility in BRIGHTON-GBR. Local Time : 1/25/2019 8:04:58 PM at BRIGHTON-GBR                                                                    
1/25/2019 9:06:09 PM      | ShippingToDataCenter           | InProgress      | Arrived at Sort Facility LONDON-HEATHROW-GBR. Local Time : 1/25/2019 9:06:09 PM at LONDON-HEATHROW-GBR                                                
1/25/2019 9:48:54 PM      | ShippingToDataCenter           | InProgress      | Processed at LONDON-HEATHROW-GBR. Local Time : 1/25/2019 9:48:54 PM at LONDON-HEATHROW-GBR                                                            
1/25/2019 10:30:20 PM     | ShippingToDataCenter           | InProgress      | Departed Facility in LONDON-HEATHROW-GBR. Local Time : 1/25/2019 10:30:20 PM at LONDON-HEATHROW-GBR
1/28/2019 7:11:35 AM      | ShippingToDataCenter           | InProgress      | Arrived at Delivery Facility in AMSTERDAM-NLD. Local Time : 1/28/2019 7:11:35 AM at AMSTERDAM-NLD                                                     
1/28/2019 9:07:57 AM      | ShippingToDataCenter           | InProgress      | With delivery courier. Local Time : 1/28/2019 9:07:57 AM at AMSTERDAM-NLD                                                                             
1/28/2019 1:35:56 PM      | ShippingToDataCenter           | InProgress      | Scheduled for delivery. Local Time : 1/28/2019 1:35:56 PM at AMSTERDAM-NLD                                                                            
1/28/2019 2:57:48 PM      | ShippingToDataCenter           | Completed       | Delivered - Signed for by. Local Time : 1/28/2019 2:57:48 PM at AMSTERDAM-NLD
1/29/2019 2:18:43 PM      | PhysicalVerification           | Completed       |
1/29/2019 3:49:50 PM      | DeviceBoot                     | Completed       | Appliance booted up successfully.
1/29/2019 3:49:51 PM      | AnomalyDetection               | Completed       | No anomaly detected.
2/12/2019 10:37:03 PM     | DataCopy                       | Resumed         |
2/13/2019 12:05:15 AM     | DataCopy                       | Resumed         |
2/15/2019 7:07:34 PM      | DataCopy                       | Completed       | Copy Completed.
2/15/2019 7:47:32 PM      | SecureErase                    | Started         |
2/15/2019 8:01:10 PM      | SecureErase                    | Completed       | Azure Data Box:<Device-serial-no> has been sanitized according to NIST 800-88 Rev 1.
------------------
Data Box Log Links
------------------
Account Name         : gusacct
Copy Logs Path       : databoxcopylog/gus-pinto_<Device-serial-no>_CopyLog_<GUID>.xml
Audit Logs Path      : azuredatabox-chainofcustodylogs\<GUID>\<Device-serial-no>
BOM Files Path       : azuredatabox-chainofcustodylogs\<GUID>\<Device-serial-no>
```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [solucionar problemas en el cuadro de datos](data-box-troubleshoot.md).
