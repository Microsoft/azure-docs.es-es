---
title: Seguimiento y registro de eventos de Azure Data Box y Azure Data Box Heavy para pedidos de exportación | Microsoft Docs
description: Describe cómo realizar un seguimiento y registrar los eventos en las distintas fases de su pedido de exportación de Azure Data Box y Azure Data Box Heavy.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 07/10/2020
ms.author: alkohli
ms.openlocfilehash: 1d924e96cfc287060107f541e44980295eb24745
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494492"
---
# <a name="tracking-and-event-logging-for-your-azure-data-box-and-azure-data-box-heavy-export-orders"></a>Seguimiento y registro de eventos para Azure Data Box y Azure Data Box Heavy para pedidos de exportación

Un pedido de exportación de Data Box o de Data Box Heavy pasa por los pasos siguientes: pedido, configuración, copia de datos, devolución y eliminación de datos. Acorde a cada paso del pedido, puede realizar varias acciones para controlar el acceso al pedido, auditar los eventos, hacer un seguimiento del pedido e interpretar los distintos registros que se generan.

En este artículo se describen detalladamente los distintos mecanismos o herramientas disponibles para realizar el seguimiento y auditar pedidos de exportación para Data Box o Data Box Heavy. La información de este artículo se aplica tanto a Data Box como a Data Box Heavy. En las secciones posteriores, todas las referencias a Data Box también se aplican a Data Box Heavy.

En la tabla siguiente se muestra un resumen de los pasos del pedido de exportación de Data Box Heavy y las herramientas disponibles para realizar un seguimiento del pedido durante cada paso y auditarlo.

| Fase de pedido de exportación de Data Box       | Herramienta de seguimiento y auditoría                                                                        |
|----------------------------|------------------------------------------------------------------------------------------------|
| Crear pedido               | [Configuración del control de acceso en el pedido a través de RBAC](#set-up-access-control-on-the-order) <br> [Habilitación del registro detallado en el pedido](#enable-verbose-log-in-the-order)                                                    |
| Pedido procesado            | [Seguimiento del pedido](#track-the-order) a través de: <ul><li> Azure portal </li><li> Sitio web del transportista </li><li>Notificaciones por correo electrónico</ul> |
| Configuración de un dispositivo              | Registro del acceso de las credenciales del dispositivo en los [registros de actividad](#query-activity-logs-during-setup)              |
| Copia de datos del dispositivo        | [Revisión de los registros de copia](#copy-log) <br> [Revisión de los registros detallados](#verbose-log) antes de copiar los datos            |
| Eliminación de datos del dispositivo   | [Visualización de los registros de la cadena de custodia](#get-chain-of-custody-logs-after-data-erasure), incluidos los registros de auditoría y el historial de pedidos                |


## <a name="set-up-access-control-on-the-order"></a>Configuración del control de acceso en el pedido

Puede controlar quién puede acceder a su pedido cuando se cree por primera vez. Configure los roles de Azure en distintos ámbitos para controlar el acceso al pedido de Data Box. Los roles de Azure determinan el tipo de acceso: lectura y escritura, solo lectura, o lectura y escritura para un subconjunto de operaciones.

Los dos roles que se pueden definir para el servicio de Azure Data Box son los siguientes:

- **Lector de Data Box**: tiene acceso de solo lectura a los pedidos, como defina el ámbito. Solo puede ver los detalles de un pedido. No puede acceder a otros detalles relacionados con las cuentas de almacenamiento ni editar los detalles del pedido, como la dirección y otros datos.
- **Colaborador de Data Box**: solo puede crear un pedido para transferir datos a una cuenta de almacenamiento determinada *si ya tiene acceso de escritura a una cuenta de almacenamiento*. Si no tiene acceso a una cuenta de almacenamiento, ni siquiera puede crear un pedido de Data Box para copiar datos a la cuenta. Este rol no define permisos relacionados con la cuenta de almacenamiento ni concede acceso a las cuentas de almacenamiento.  

Para restringir el acceso a un pedido, puede hacer lo siguiente:

- Asignar un rol en el nivel de un pedido. El usuario solo tiene los permisos que definan los roles para interactuar únicamente con ese pedido de Data Box específico.
- Asignar un rol en el nivel del grupo de recursos. El usuario tiene acceso a todos los pedidos de Data Box dentro de un grupo de recursos.

Para más información sobre el uso sugerido de RBAC, vea [Procedimientos recomendados para Azure RBAC](../role-based-access-control/best-practices.md).

## <a name="enable-verbose-log-in-the-order"></a>Habilitación del registro detallado en el pedido

Al hacer un pedido de exportación para Data Box, tiene la opción de habilitar la recopilación de registros detallados. Esta es la pantalla de pedidos en la que puede habilitar el registro detallado:

![Selección de opción de exportación](media/data-box-deploy-export-ordered/azure-data-box-export-04b.png)

Al seleccionar la opción **Incluir registro detallado**, se genera un archivo de registro detallado al copiar los datos de la cuenta de Azure Storage. Este registro contiene una lista de todos los archivos que se exportaron correctamente.      

Para obtener más información sobre el pedido de exportación, consulte [Creación de un pedido de exportación para Data Box](data-box-deploy-export-ordered.md).

## <a name="track-the-order"></a>Seguimiento del pedido

Puede realizar un seguimiento del pedido a través de Azure Portal y del sitio web del transportista. Tiene a su disposición los mecanismos siguientes para hacer un seguimiento del pedido de Data Box en cualquier momento:

- Para hacer un seguimiento del pedido cuando el dispositivo está en el centro de datos de Azure o en su entorno local, vaya a su **pedido de Data Box > Información general** en Azure Portal.

    ![Visualización del estado del pedido y el número de seguimiento](media/data-box-logs/overview-view-status-1.png)

- Para hacer un seguimiento del pedido mientras el dispositivo está en tránsito, visite el sitio web del operador regional, por ejemplo, el sitio web de UPS en EE. UU. Proporcione el número de seguimiento asociado con su pedido.
- Data Box también envía notificaciones por correo electrónico cada vez que cambia el estado del pedido en función de los correos electrónicos que se proporcionaron cuando se creó el pedido. Para obtener una lista de todos los estados de pedido de Data Box, consulte [Visualización del estado del pedido](data-box-portal-admin.md#view-order-status). Para cambiar la configuración de notificaciones asociada con el pedido, vea [Edición de los detalles de la notificación](data-box-portal-admin.md#edit-notification-details).

## <a name="query-activity-logs-during-setup"></a>Consulta de los registros de actividad durante la configuración

- Data Box llega a sus instalaciones en un estado de bloqueado. Puede usar las credenciales de dispositivo disponibles en Azure Portal para su pedido.  

    Cuando se configura Data Box, deberá saber quién ha accedido a las credenciales del dispositivo. Para averiguar quién ha accedido a la hoja **Credenciales del dispositivo** puede consultar los registros de actividad.  Cualquier acción que implique el acceso a la hoja **Detalles del dispositivo > Credenciales** se registra en los registros de actividad como una acción `ListCredentials`.

    ![Consulta de los registros de actividad](media/data-box-logs/query-activity-log-1.png)

- Cada inicio de sesión en Data Box se registra en tiempo real. Sin embargo, esta información solo está disponible en los [registros de auditoría de la cadena de custodia](#chain-of-custody-audit-logs) después de que el pedido se haya completado correctamente.

## <a name="view-logs-during-data-copy"></a>Visualización de registros durante la copia de datos

Antes de copiar los datos de Data Box, puede descargar y revisar el *registro de copia* y el *registro detallado* para los datos que se copiaron en Data Box. Estos registros se generan cuando los datos se copian de la cuenta de Storage de Azure en Data Box. 

### <a name="copy-log"></a>Registro de copia

Antes de copiar los datos de Data Box, descargue el registro de copia de la página **Conectar y copiar**.

Este es un ejemplo de salida del *registro de copia* cuando no ha habido errores y todos los archivos se han copiado al copiar datos desde Azure al dispositivo de Data Box.

```output
<CopyLog Summary="Summary">
  <Status>Succeeded</Status>
  <TotalFiles_Blobs>5521</TotalFiles_Blobs>
  <FilesErrored>0</FilesErrored>
</CopyLog>
``` 
    
Este es un ejemplo de salida cuando el *registro de copia* tiene errores y algunos de los archivos no se han podido copiar desde Azure.

```output
<ErroredEntity CloudFormat="AppendBlob" Path="export-ut-appendblob/wastorage.v140.3.0.2.nupkg">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>400</ErrorCode>
  <ErrorMessage>UnsupportBlobType</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity CloudFormat="AppendBlob" Path="export-ut-appendblob/xunit.console.Primary_2020-05-07_03-54-42-PM_27444.hcsml">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>400</ErrorCode>
  <ErrorMessage>UnsupportBlobType</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity CloudFormat="AppendBlob" Path="export-ut-appendblob/xunit.console.Primary_2020-05-07_03-54-42-PM_27444 (1).hcsml">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>400</ErrorCode>
  <ErrorMessage>UnsupportBlobType</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><CopyLog Summary="Summary">
  <Status>Failed</Status>
  <TotalFiles_Blobs>4</TotalFiles_Blobs>
  <FilesErrored>3</FilesErrored>
</CopyLog>    
```

Dispone de las siguientes opciones para exportar estos archivos: 

- Puede transferir los archivos que no ha podido copiar a través de la red. 
- Si el tamaño de los datos es mayor que la capacidad utilizable del dispositivo, se produce una copia parcial y todos los archivos que no se han copiado aparecen en este registro. Puede utilizar este registro como archivo XML de entrada para crear un nuevo pedido de Data Box y, después, copiar estos archivos.

### <a name="verbose-log"></a>Registro detallado

El *registro detallado* contiene una lista de todos los archivos exportados correctamente desde la cuenta de Azure Storage. El registro también contiene el tamaño del archivo y el cálculo de la suma de comprobación.

El registro detallado tiene la información en el formato siguiente:

`<file size = "file-size-in-bytes" crc64="cyclic-redundancy-check-string">\folder-path-on-data-box\name-of-file-copied.md</file>`

Esta es la salida de ejemplo del registro detallado. 

```powershell
  <File CloudFormat="BlockBlob" Path="validblobdata/test1.2.3.4" Size="1024" crc64="7573843669953104266">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/helloEndWithDot..txt" Size="11" crc64="7320094093915972193">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/test..txt" Size="12" crc64="17906086011702236012">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/test1" Size="1024" crc64="7573843669953104266">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/test1.2.3" Size="1024" crc64="7573843669953104266">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/.......txt" Size="11" crc64="7320094093915972193">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/copylogb08fa3095564421bb550d775fff143ed====..txt" Size="53638" crc64="1147139997367113454">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/testmaxChars-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-12345679" Size="1024" crc64="7573843669953104266">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file0" Size="0" crc64="0">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file1" Size="0" crc64="0">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file4096_000001" Size="4096" crc64="16969371397892565512">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file4096_000000" Size="4096" crc64="16969371397892565512">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/64KB-Seed10.dat" Size="65536" crc64="10746682179555216785">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/LiveSiteReport_Oct.xlsx" Size="7028" crc64="6103506546789189963">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/NE_Oct_GeoReport.xlsx" Size="103197" crc64="13305485882546035852">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/64KB-Seed1.dat" Size="65536" crc64="3140622834011462581">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/1mbfiles-0-0" Size="1048576" crc64="16086591317856295272">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file524288_000001" Size="524288" crc64="8908547729214703832">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/4mbfiles-0-0" Size="4194304" crc64="1339017920798612765">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file524288_000000" Size="524288" crc64="8908547729214703832">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/8mbfiles-0-1" Size="8388608" crc64="3963298606737216548">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/1mbfiles-0-1" Size="1048576" crc64="11061759121415905887">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/XLS-10MB.xls" Size="1199104" crc64="2218419493992437463">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/8mbfiles-0-0" Size="8388608" crc64="1072783424245035917">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/4mbfiles-0-1" Size="4194304" crc64="9991307204216370812">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/VL_Piracy_Negtive10_TPNameAndGCS.xlsx" Size="12398699" crc64="13526033021067702820">
  </File>
  ``````

Los registros detallados también se copian en la cuenta de Azure Storage. De forma predeterminada, los registros se escriben en un contenedor denominado `copylog`. Los registros se almacenan con la convención de nomenclatura siguiente:

`storage-account-name/databoxcopylog/ordername_device-serial-number_CopyLog_guid.xml`.

La ruta de acceso del registro de copia también se muestra en la hoja **Información general** del portal.

<!-- add a screenshot-->

Puede usar estos registros para comprobar que los archivos copiados desde Azure coinciden con los datos que se copiaron en el servidor local. 

Use el archivo de registro detallado:

- Para comprobar los nombres reales y el número de archivos que se copiaron en el Data Box.
- Para comprobar los tamaños reales de los archivos.
- Para comprobar que *crc64* corresponde a una cadena distinta de cero. Se realiza una prueba cíclica de redundancia (CRC) durante la exportación desde Azure. Se pueden comparar las CRC de la exportación y después de que se copien los datos de Data Box en el servidor local. Un error de coincidencia de CRC indica que no se pudieron copiar correctamente los archivos correspondientes.


## <a name="get-chain-of-custody-logs-after-data-erasure"></a>Obtener la cadena de registros de custodia después de la eliminación de datos

Cuando se borren los datos de los discos de Data Box las directrices de revisión 1 de NIST SP 800-88, estará disponible la cadena de custodia de registros. Estos registros incluyen los registros de auditoría de la cadena de custodia y el historial de pedidos. También se copian los archivos de manifiesto o de BOM junto con los registros de auditoría.

### <a name="chain-of-custody-audit-logs"></a>Registros de auditoría de la cadena de custodia

Los registros de auditoría de la cadena de custodia contienen información sobre cómo encender recursos compartidos y acceder a ellos en Data Box o Data Box Heavy cuando están fuera del centro de datos de Azure. Estos registros se encuentran en `storage-account/azuredatabox-chainofcustodylogs`.

Este es un ejemplo del registro de auditoría de un Data Box:

```output
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

## <a name="download-order-history"></a>Descarga del historial de pedidos

El historial de pedidos está disponible en Azure Portal. Si el pedido está completo y se completa la limpieza del dispositivo (eliminación de datos de los discos), vaya al pedido de su dispositivo y navegue hasta **Detalles de pedido**. La opción **Descargar el historial de pedidos** está disponible. Para más información, consulte [Descargar historial de pedidos](data-box-portal-admin.md#download-order-history).

Si se desplaza por el historial de pedidos, verá:

- Información de seguimiento del operador para el dispositivo.
- Los eventos con actividad *SecureErase*. Estos eventos se corresponden con el borrado de los datos en el disco.
- Vínculos de registro de Data Box. Se presentan las rutas de acceso para los archivos de *registros de auditoría*, *registros de copia* y *BOM*.

Este es un ejemplo del registro del historial de pedidos de Azure Portal:

```
-------------------------------
Microsoft Data Box Order Report
-------------------------------
Name                                               : gus-poland                              
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
Copy Logs Path       : databoxcopylog/gus-poland_<Device-serial-no>_CopyLog_<GUID>.xml
Audit Logs Path      : azuredatabox-chainofcustodylogs\<GUID>\<Device-serial-no>
BOM Files Path       : azuredatabox-chainofcustodylogs\<GUID>\<Device-serial-no>
```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [solucionar problemas relacionados con el almacenamiento de blobs de Azure Data Box](data-box-troubleshoot.md).
