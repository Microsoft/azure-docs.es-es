---
title: Preguntas frecuentes sobre Microsoft Azure Data Box | Microsoft Docs
description: Contiene preguntas frecuentes y respuestas acerca de Azure Data Box, una solución en la nube que permite transferir grandes cantidades de datos a Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: overview
ms.date: 05/08/2019
ms.author: alkohli
ms.openlocfilehash: 3fca94a56f80ec520ac6777610a90bdddf1b8988
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65473264"
---
# <a name="azure-data-box-frequently-asked-questions"></a>Azure Data Box: Preguntas frecuentes

La solución híbrida Microsoft Azure Data Box le permite enviar terabytes de datos a Azure de forma rápida, económica y confiable, mediante un dispositivo de transferencia. En este documento encontrará las preguntas más frecuentes que pueden surgir al usar Data Box en Azure Portal, y las respuestas a dichas preguntas. 

Las preguntas y respuestas se organizan en las categorías siguientes:

- Acerca del servicio
- Pedir el dispositivo
- Configuración y conexión 
- Seguimiento del estado
- Copia de datos 
- Enviar el dispositivo
- Comprobación y carga de datos 
- Cadena de soporte técnico de custodia

## <a name="about-the-service"></a>Acerca del servicio

### <a name="q-what-is-azure-data-box-service"></a>P: ¿Qué es el servicio Azure Data Box? 
A.  El servicio Azure Data Box está diseñado para la ingesta de datos sin conexión. En este servicio se administra una matriz de productos de diferentes capacidades de almacenamiento, todos ellos adaptados para el transporte de datos. 

### <a name="q-what-is-azure-data-box"></a>P: ¿Qué es Azure Data Box?
A. Azure Data Box le permite transferir terabytes de datos de forma rápida, barata y segura en Azure. Puede pedir el dispositivo Data Box en Azure Portal. Microsoft le enviará un dispositivo de almacenamiento de 80 TB de capacidad mediante un operador regional. 

Una vez recibido el dispositivo, puede configurarlo rápidamente mediante la interfaz de usuario web local. Copie los datos de los servidores en el dispositivo y envíelo de nuevo a Azure. En el centro de datos de Azure, los datos se cargan automáticamente del dispositivo a Azure. El servicio de Data Box se encarga de realizar el seguimiento de todo el proceso en Azure Portal.

### <a name="q-when-should-i-use-data-box"></a>P: ¿Cuándo debo usar Data Box?
A. Si quiere transferir entre 40 y 500 TB de datos a Azure, puede usar Data Box. Si va transferir menos de 40 TB de datos, use Data Box Disk; si lo que quiere es transferir más de 500 TB de datos, regístrese en [Data Box Heavy](data-box-heavy-overview.md).

### <a name="q-what-is-the-price-of-data-box"></a>P: ¿Cuánto cuesta Data Box?
A. Data Box está disponible a un costo nominal de 10 días. Cuando selecciona el modelo de producto mientras crea un pedido en Azure Portal, se muestran los cargos del dispositivo. El envío es gratuito, pero se aplicarán los cargos derivados del almacenamiento de Azure. Para obtener más información, vaya a [Azure Data Box pricing](https://azure.microsoft.com/pricing/details/storage/databox/) (Precios de Azure Data Box). 

### <a name="q-what-is-the-maximum-amount-of-data-i-can-transfer-with-data-box-in-one-instance"></a>P: ¿Cuál es la cantidad máxima de datos que se pueden transferir con Data Box en una instancia?
A. Data Box tiene una capacidad bruta de 100 TB y una capacidad utilizable de 80 TB. Puede transferir hasta 80 TB de datos con Data Box. Para transferir más datos, necesitará pedir más dispositivos.

### <a name="q-how-can-i-check-if-data-box-is-available-in-my-region"></a>P: ¿Cómo se puede comprobar si Data Box está disponible en mi región? 
A.  Para obtener información sobre los países o regiones en los que está disponible Data Box, vaya a la sección de [disponibilidad por región](data-box-overview.md#region-availability).  

### <a name="q-which-regions-can-i-store-data-in-with-data-box"></a>P: ¿En qué regiones puedo almacenar datos con Data Box?
A. Data Box Disk es compatible con todas las regiones de Estados Unidos, Europa Occidental y Europa del Norte, Francia y el Reino Unido. Solo se admiten las regiones con nube pública de Azure. Azure Government u otras nubes soberanas no se admiten. Para más información, vaya a [Disponibilidad por región](data-box-overview.md#region-availability).

### <a name="q-whom-should-i-contact-if-i-encounter-any-issues-with-data-box"></a>P: ¿Con quién debo ponerme en contacto si surge algún problema con Data Box?
A. Si surge algún problema con Data Box, póngase en contacto con el [soporte técnico de Microsoft](data-box-disk-contact-microsoft-support.md).

### <a name="q-i-have-lost-my-data-box-is-there-a-lost-device-charge"></a>P: He perdido mi Data Box. ¿Hay algún cargo adicional por pérdida de dispositivo?
A. Sí. Hay un cargo por dispositivo perdido o dañado. Este cargo se detalla en la [página de precios](https://azure.microsoft.com/pricing/details/storage/databox/) así como en la página de [términos del servicio del producto](https://www.microsoft.com/licensing/product-licensing/products).


## <a name="order-device"></a>Pedir el dispositivo

### <a name="q-how-do-i-get-data-box"></a>P: ¿Cómo se obtiene Data Box? 
A.  Para obtener Azure Data Box, inicie sesión en Azure Portal y cree un pedido de Data Box. Proporcione su información de contacto y los datos de notificación. Una vez realizado el pedido, y en función de la disponibilidad, Data Box se envía en un plazo máximo de 10 días. Para obtener más información, vaya a [Order a Data Box](data-box-deploy-ordered.md) (Pedir Data Box).

### <a name="q-i-was-not-able-to-create-a-data-box-order-in-the-azure-portal-why-would-this-be"></a>P: No pude hacer un pedido de Data Box en Azure Portal. ¿Por qué?
A. Si no pudo pedir Data Box, es que hay un problema con su tipo de suscripción o con el acceso. 

Compruebe su suscripción. Data Box solo está disponible para las ofertas de suscripción de Contratos Enterprise (EA) y del Proveedor de soluciones en la nube (CSP). Si su suscripción no se encuentra entre ninguno de los tipos anteriores, póngase en contacto con el soporte técnico de Microsoft para actualizar la suscripción.

Si tiene un tipo de oferta compatible para la suscripción, compruebe el nivel de acceso de la suscripción. Debe ser un colaborador o propietario de la suscripción para poder realizar un pedido.

### <a name="q-i-ordered-a-couple-of-data-box-devices-i-am-not-able-to-create-any-additional-orders-why-would-this-be"></a>P: Pedí un par de dispositivos de Data Box,  y ahora no puedo hacer pedidos adicionales. ¿Por qué?
A. Permitimos un máximo de cinco pedidos activos según la suscripción y el límite comercial (esto es, la combinación del país y la región seleccionados). Si necesita pedir un dispositivo adicional, póngase en contacto con el soporte técnico de Microsoft para aumentar el límite de su suscripción.

### <a name="q-when-i-try-to-create-an-order-i-receive-a-notification-that-the-data-box-service-is-not-available-what-does-this-mean"></a>P: Cuando intento crear un pedido, recibo una notificación de que el servicio de Data Box no está disponible. ¿Qué significa?
A. Lo que esto significa es que el servicio de Data Box no está disponible para la combinación de país y región que ha seleccionado. Si cambia esta combinación podrá acceder al servicio de Data Box. Para obtener una lista de las regiones donde el servicio está disponible, vaya a [Region availability for Data Box](data-box-overview.md#region-availability) (Disponibilidad de Data Box según la región).

### <a name="q-i-placed-my-data-box-order-few-days-back-when-will-i-receive-my-data-box"></a>P: Hice el pedido de Data Box hace unos días. ¿Cuándo recibiré mi Data Box?
A. Cuando realiza un pedido, comprobamos si hay algún dispositivo disponible para su pedido. Si lo hay, lo enviamos en 10 días. Tenga en cuenta que es posible que haya períodos de gran demanda. Si esto es así, su pedido será puesto en cola y podrá realizar el seguimiento del estado del mismo en Azure Portal para ver si cambia. Si no podemos atender a su pedido en 90 días, este se cancelará automáticamente.

### <a name="q-i-have-filled-up-my-data-box-with-data-and-need-to-order-another-one-is-there-a-way-to-quickly-place-the-order"></a>P: He llenado mi Data Box con datos y necesito pedir otro. ¿Hay alguna manera rápida de realizar el pedido?
A. Puede clonar el pedido anterior. La clonación crea el mismo pedido que antes y permite editar los detalles del pedido solo sin necesidad de escribir los datos de dirección, contacto y notificación.

## <a name="configure-and-connect"></a>Configuración y conexión

### <a name="q-how-do-i-unlock-the-data-box"></a>P: ¿Cómo se desbloquea Data Box? 
A.  En Azure Portal, vaya a su pedido de Data Box y, a continuación, vaya a **Detalles del dispositivo**. Copie la contraseña de desbloqueo. Use esta contraseña para iniciar sesión en la interfaz de usuario web local de Data Box. Para más información, vaya a [Tutorial: Cableado y conexión de un dispositivo Azure Data Box](data-box-deploy-set-up.md).

### <a name="q-can-i-use-a-linux-host-computer-to-connect-and-copy-the-data-on-to-the-data-box"></a>P: ¿Se puede usar un equipo host de Linux para conectarse y copiar los datos en Data Box?
A.  Sí. Puede usar Data Box para conectarse a los clientes SMB y NFS. Para más información, vaya a la lista de [sistemas operativos compatibles](data-box-system-requirements.md) del equipo host.

### <a name="q-my-data-box-is-dispatched-but-now-i-want-to-cancel-this-order-why-is-the-cancel-button-not-available"></a>P: Ya se ha enviado mi Data Box, pero quiero cancelar el pedido. ¿Por qué no está disponible el botón de cancelación?
A.  El pedido solo se puede cancelar durante el período que va entre la realización del pedido de Data Box y el momento en que se realiza el envío. Una vez que el pedido de Data Box se procese, no se podrá cancelar. 

### <a name="q-can-i-connect-a-data-box-at-the-same-to-multiple-host-computers-to-transfer-data"></a>P: ¿Se puede conectar Data Box a varios equipos de host para transferir datos?
A. Sí. Se pueden conectar varios equipos de host a Data Box para transferir datos, y se pueden ejecutar varios trabajos de copia en paralelo. Para más información, vaya a [Tutorial: Copia de datos a Azure Data Box](data-box-deploy-copy-data.md)

### <a name="q-can-i-connect-to-both-the-10-gbe-interfaces-on-the-data-box-to-transfer-data"></a>P: ¿Puedo conectarme a las interfaces de 10-GbE del Data Box para transferir datos?
A. Sí. Las interfaces de 10 GbE se pueden conectar a Data Box para copiar datos al mismo tiempo. Para más información sobre cómo copiar datos, vaya a [Tutorial: Copia de datos a Azure Data Box](data-box-deploy-copy-data.md)

<!--### Q. The network interface on my Data Box is not working. What should I do? 
A. 

### Q. I could not set up Data Box using a Dynamic (DHCP) IP address. Why would this be?
A.

### Q. I could not set up Data Box using a Static IP address. Why would this be?
A.

### Q. I could not set up Data Box on a private network. Why would this be?
A.-->

### <a name="q-the-system-fault-indicator-led-on-the-front-operating-panel-is-on-what-should-i-do"></a>P: La luz LED que indica un error del sistema y que se encuentra en el panel de operación frontal está encendida. ¿Cuál debo hacer?
A. Si la luz LED que indica un error del sistema está encendida, esto quiere decir que su sistema no está funcionando correctamente. [Póngase en contacto con el servicio de soporte técnico de Microsoft](data-box-disk-contact-microsoft-support.md) para conocer los pasos siguientes.

### <a name="q-i-cant-access-the-data-box-unlock-password-in-the-azure-portal-why-would-this-be"></a>P: No puedo acceder a la contraseña de desbloqueo de Data Box en Azure Portal. ¿Por qué?
A. Si no puede acceder a la contraseña de desbloqueo en Azure Portal, compruebe los permisos de la suscripción y la cuenta de almacenamiento. Asegúrese de tener los permisos de colaborador o propietario en el nivel del grupo de recursos. De lo contrario, debe tener al menos permiso de rol de operador de Data Box para ver las credenciales de acceso.

### <a name="q-is-port-channel-configuration-supported-on-data-box-how-about-mpio"></a>P: ¿Se admite la configuración de canal de puerto en Data Box? ¿Qué pasa con MPIO?
A. No admitimos en Data Box la configuración de canal de puerto, la configuración de Multipath IO (MPIO) ni la configuración de vLAN.

## <a name="track-status"></a>Seguimiento del estado

### <a name="q-how-do-i-track-the-data-box-from-when-i-placed-the-order-to-shipping-the-device-back"></a>P: ¿Cómo puedo hacer el seguimiento del dispositivo de Data Box desde el momento en que realicé el pedido hasta el envío para devolverlo? 
A.  El seguimiento del estado del pedido de Data Box se puede realizar desde Azure Portal. Al crear el pedido, también se le solicita que indique una dirección de correo electrónico para cualquier notificación. Si ya la ha especificado, los cambios de estado del pedido se le notificarán por correo electrónico. Para más información acerca de cómo [configurar correos electrónicos de notificación](data-box-portal-ui-admin.md#edit-notification-details).

### <a name="q-how-do-i-return-the-device"></a>P: ¿Cómo se devuelve el dispositivo? 
A.  Microsoft muestra una etiqueta de envío en la pantalla de E-Ink. Si la etiqueta de envío no aparece en la pantalla de E-Ink, vaya a **Introducción > Descargar la etiqueta de envío**. Descargue e imprima la etiqueta, inserte la etiqueta en la etiqueta de plástico transparente del dispositivo y llévelo a la ubicación del transportista. 

### <a name="q-i-received-an-email-notification-that-my-device-has-reached-the-azure-datacenter-how-do-i-find-out-if-the-data-upload-is-in-progress"></a>P: He recibido una notificación por correo electrónico que dice que el dispositivo ha llegado al centro de datos de Azure. ¿Cómo puedo saber si la carga de datos está en curso?
A. Vaya a su pedido de Data Box en Azure Portal y acceda a **Información general**. Si se ha iniciado la carga de datos en Azure, verá el progreso de la copia en el panel derecho. 

## <a name="migrate-data"></a>Migración de los datos

### <a name="q-what-is-the-maximum-data-size-that-can-be-used-with-data-box"></a>P: ¿Cuál es el tamaño máximo de datos que se puede usar con Data Box?  
A.  El dispositivo Data Box tiene una capacidad de almacenamiento utilizable de 80 TB. Puede usar un único dispositivo Data Box para almacenar entre 40 y 80 TB de datos. Para almacenar hasta 500 TB de datos, puede pedir varios dispositivos de Data Box. Si el tamaño de sus datos es superior a 500 TB, regístrese para obtener Data Box Heavy.  

### <a name="q-what-are-the-maximum-block-blob-and-page-blob-sizes-supported-by-data-box"></a>P: ¿Qué tamaño máximo de blobs en bloques y blobs en páginas admite Data Box? 
A.  El tamaño máximo lo determinan los límites de Azure Storage. El blob en bloques máximo es, aproximadamente, de 4,768 TiB, mientras que el tamaño de blob en páginas máximo es de 8 TiB. Para más información, consulte [Objetivos de escalabilidad y rendimiento de Azure Storage](../storage/common/storage-scalability-targets.md). 

### <a name="q-how-do-i-know-that-my-data-is-secure-during-transit"></a>P: ¿Cómo se sabe si los datos están protegidos durante el tránsito? 
A. Hay varias características de seguridad implementadas para garantizar la seguridad de su Data Box durante el envío. Algunos de estas características incluyen sellos a prueba de manipulaciones, sistemas para detectar si el hardware y el software se han manipulado y una contraseña para desbloquear el dispositivo. Para obtener más información, vaya a [Azure Data Box security and data protection](data-box-security.md) (Protección de datos y seguridad de Azure Data Box).

### <a name="q-how-do-i-copy-the-data-to-the-data-box"></a>P: ¿Cómo se copian los datos en Data Box? 
A.  Si usa un cliente SMB, puede utilizar una herramienta de copia de SMB, como Robocopy, Diskboss, o incluso el Explorador de archivos de Windows, para arrastrar y colocar los datos en el dispositivo, y así copiarlos. 

Si usa un cliente NFS, puede usar [rsync](https://rsync.samba.org/), [FreeFileSync](https://www.freefilesync.org/), [Unison](https://www.cis.upenn.edu/~bcpierce/unison/) o [Ultracopier](https://ultracopier.first-world.info/). 

Para más información, vaya a [Tutorial: Copia de datos a Azure Data Box](data-box-deploy-copy-data.md)

### <a name="q-are-there-any-tips-to-speed-up-the-data-copy"></a>P: ¿Algunas sugerencia para agilizar la copia de datos?
A.  Para acelerar el proceso de copia:

- Use varias secuencias de copia de datos. Por ejemplo, si usa Robocopy, emplee la opción de varios procesos. Para más información sobre el comando exacto utilizado, vea [Tutorial: Copia de datos a Azure Data Box y comprobación de](data-box-deploy-copy-data.md).
- Utilice varias sesiones.
- En lugar de realizar la copia a través de un recurso compartido de red (en el que se puede encontrar con la velocidad de la red limitada), asegúrese de que los datos se encuentran de forma local en el equipo al que se va a conectar Data Box.
- Evalúe el rendimiento del equipo usado para copiar los datos. Descargue la [herramienta Bluestop FIO](https://bluestop.org/fio/) y úsela para evaluar el rendimiento del hardware del servidor.

<!--### Q. How to speed up the data copy if the source data has small files (KBs or few MBs)?
A.  To speed up the copy process:

- Create a local VHDx on fast storage or create an empty VHD on the HDD/SSD (slower).
- Mount it to a VM.
- Copy files to the VM’s disk.-->


### <a name="q-can-i-use-multiple-storage-accounts-with-data-box"></a>P: ¿Se pueden usar varias cuentas de almacenamiento con Data Box?
A.  Sí. Se admite un máximo de 10 cuentas de almacenamiento, de uso general, clásicas o de almacenamiento de blobs con Data Box. Se admiten blobs frecuentes y esporádicos. Durante el lanzamiento de la versión de disponibilidad general, solo se admiten las cuentas de almacenamiento de todas las regiones de Estados Unidos, Europa Occidental y Europa del Norte, Francia y Reino Unido en la nube pública de Azure.


## <a name="ship-device"></a>Enviar el dispositivo

<!--### Q. How do I schedule a pickup for my Data Box?--> 

### <a name="q-my-device-was-delivered-but-the-device-seems-to-be-damaged-what-should-i-do"></a>P: Recibí el dispositivo, pero parece estar dañado. ¿Cuál debo hacer?
A. Si su dispositivo ha llegado dañado o hay evidencia de que lo hayan alterado, no lo use. [Póngase en contacto con el soporte técnico de Microsoft](data-box-disk-contact-microsoft-support.md) y devuelva el dispositivo lo antes posible. También puede crear un nuevo pedido de Data Box para obtener un dispositivo de reemplazo. En ese caso, no se le cobrará por el dispositivo de reemplazo.

### <a name="q-can-i-use-my-own-shipping-carrier-to-ship-data-box"></a>P: ¿Puedo usar mi propia empresa de transporte para enviar Data Box?
A. En cuanto al servicio Data Box, Microsoft se encarga de administrar el envío desde y hasta el centro de datos de Azure. Si quiere usar su propio transportista, puede usar el servicio de importación y exportación de Azure. Para obtener más información, vaya a [What is Azure Import/Export service?](../storage/common/storage-import-export-service.md) (¿Qué es el servicio de importación y exportación de Azure?).

### <a name="q-e-ink-display-is-not-showing-the-return-shipment-label-what-should-i-do"></a>P: La pantalla E-Ink no muestra la etiqueta del envío de devolución. ¿Cuál debo hacer?
A. Si la pantalla E-Ink no muestra la etiqueta del envío de devolución, realice los siguientes pasos:
- Retire la etiqueta del envío anterior y cualquier etiqueta que tenga que ver con el mismo.
- Vaya a su pedido en el portal de Azure. Vaya a la opción **Información general** y **Descargar la etiqueta de envío**. Para obtener más información, vaya a [Download shipping label](data-box-portal-admin.md#download-shipping-label) (Descargar la etiqueta de envío).
- Imprima la etiqueta de envío e insértela en la funda transparente de plástico que se adjunta al dispositivo. 
- Asegúrese de que la etiqueta de envío se vea con claridad. 

### <a name="q-how-is-my-data-protected-during-transit"></a>P: ¿Cómo se protegen los datos durante el tránsito? 
A.  Durante el tránsito, las siguientes características de Data Box se encargarán de proteger los datos.
 - Los discos de Data Box se cifran con el tipo de cifrado AES 256-bit. 
 - El dispositivo está bloqueado y necesita una contraseña de desbloqueo para introducir y obtener acceso a los datos.
Para obtener más información, vaya a [Data Box security features](data-box-security.md) (Características de seguridad de Data Box).  

### <a name="q-i-have-finished-prepare-to-ship-and-shut-down-the-device-can-i-still-add-more-data-to-data-box"></a>P: He terminado de preparar el envío y el dispositivo está apagado. ¿Todavía puedo agregar más datos en Data Box?
A. Sí. Puede encender el dispositivo y agregar más datos. Recuerde que deberá ejecutar de nuevo la opción **Preparar para enviar** una vez que haya terminado de copiar los datos.

### <a name="q-i-received-my-device-and-it-is-not-booting-up-how-do-i-ship-the-device-back"></a>P: He recibido mi dispositivo y no arranca. ¿Cómo devuelvo el dispositivo?
A. Si el dispositivo no arranca, vaya a su pedido en Azure Portal. Descargue una etiqueta de envío y péguela en el dispositivo. Para obtener más información, vaya a [Download shipping label](data-box-portal-admin.md#download-shipping-label) (Descargar la etiqueta de envío).

## <a name="verify-and-upload"></a>Comprobación y carga

### <a name="q-how-soon-can-i-access-my-data-in-azure-once-ive-shipped-the-data-box-back"></a>P: ¿Cuándo se puede acceder a los datos en Azure una vez que se ha devuelto el dispositivo de Data Box? 
A.  Una vez que el estado del pedido de **Copia de datos** indica que se **ha completado**, debería obtener acceso a los datos de inmediato.

### <a name="q-where-is-my-data-located-in-azure-after-the-upload"></a>P: ¿Dónde se encuentran los datos en Azure después de la carga?
A.  Cuando copia los datos en Data Box, dependiendo de si los datos son un blob en bloques, un blob en páginas o archivos de Azure, los datos se cargan en una de las rutas de acceso siguientes de su cuenta de Azure Storage.
- `https://<storage_account_name>.blob.core.windows.net/<containername>` 
- `https://<storage_account_name>.file.core.windows.net/<sharename>`
 
  Como alternativa, puede ir a su cuenta de almacenamiento de Azure en Azure Portal e ir desde allí.

### <a name="q-i-just-noticed-that-i-did-not-follow-the-azure-naming-requirements-for-my-containers-will-my-data-fail-to-upload-to-azure"></a>P: Acabo de darme cuenta de que no he seguido los requisitos de nomenclatura de Azure en los contenedores. ¿Pueden aparecer errores al cargar los datos en Azure?
A.  Si los nombres de los contenedores tienen mayúsculas, estas se convierten automáticamente en minúsculas. Si los nombres no son compatibles por algún otro motivo (caracteres especiales, otros idiomas, etc.), se producirá un error en la carga. Para obtener más información sobre procedimientos recomendados para asignar nombres a los recursos compartidos, contenedores o archivos, vaya a:
- [Nomenclatura y referencia de recursos compartidos](https://docs.microsoft.com/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata)
- [Convenciones de blobs en bloques y blobs en páginas](https://docs.microsoft.com/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs)

### <a name="q-how-do-i-verify-the-data-i-copied-onto-data-box"></a>P: ¿Cómo se comprueban los datos que se han copiado en Data Box?
A.  Una vez completada la copia de datos, al ejecutar **Preparar para enviar**, los datos se validan. Data Box crea una lista de archivos y sumas de comprobación para los datos, durante el proceso de validación. Puede descargar la lista de archivos y comprobarla con los archivos de los datos de origen. Para obtener más información, vaya a [Preparar para enviar](data-box-deploy-picked-up.md#prepare-to-ship).

### <a name="q-what-happens-to-my-data-after-i-have-returned-the-data-box"></a>P: ¿Qué ocurre con los datos después de que se devuelve el dispositivo Data Box?
A.  Una vez que se completa la copia de datos en Azure, los datos de los discos de Data Box se borran de forma segura según las directrices de la revisión 1 de NIST SP 800-88. Para obtener más información, vaya a [Erasure of data from Data Box](data-box-deploy-picked-up.md#erasure-of-data-from-data-box) (Borrar completamente los datos de Data Box).

## <a name="audit-report"></a>Informe de auditoría

### <a name="how-does-azure-data-box-service-help-support-customers-chain-of-custody-procedure"></a>¿Cómo ayuda el servicio Azure Data Box a garantizar el procedimiento de cadena de custodia de los clientes?
A.  El servicio Azure Data Box proporciona de forma nativa informes que puede utilizar para documentar su cadena de custodia. Los registros de auditoría y copia están disponibles en su cuenta de almacenamiento de Azure, y el [historial de pedidos se puede descargar](data-box-portal-admin.md#download-order-history) en Azure Portal después de completar el pedido.


### <a name="what-type-of-reporting-is-available-to-support-chain-of-custody"></a>¿Qué tipo de informes está disponible para garantizar la cadena de custodia?
A.  Los siguientes informes están disponibles para garantizar la cadena de custodia:

- Logística de transporte de UPS.
- Registro de encendido y acceso compartido del usuario.
- Archivo de manifiesto con una prueba cíclica de redundancia de 64 bits (CRC-64) o la suma de comprobación de cada archivo que se ingirió con éxito en Data Box.
- Informes de archivos que no pudieron cargarse en la cuenta de almacenamiento de Azure.
- Saneamiento del dispositivo Data Box (según los estándares NIST 800 88R1) después de copiar los datos en su cuenta de almacenamiento de Azure.

### <a name="are-the-carrier-tracking-logs-from-ups-available"></a>¿Están disponibles los registros de seguimiento del transportista (de UPS)? 
A.  Los registros de seguimiento del transportista se obtienen en el historial de pedidos de Data Box. Podrá obtener acceso a este informe una vez que el dispositivo haya llegado al centro de datos de Azure, y se hayan borrado los datos de los discos del mismo. Si lo necesita de forma urgente, también puede ir directamente al sitio web del transportista con el número de seguimiento del pedido y obtener la información del seguimiento.

### <a name="can-i-transport-the-data-box-to-azure-datacenter"></a>¿Puedo transportar Data Box al centro de datos de Azure? 
A.   No. Actualmente, el centro de datos de Azure no acepta la entrega de Data Box de clientes o transportistas que no sean UPS.


## <a name="next-steps"></a>Pasos siguientes

- Revise [los requisitos del sistema Data Box](data-box-system-requirements.md).
- Información acerca de los [límites de Data Box](data-box-limits.md).
- Implemente rápidamente [Azure Data Box](data-box-quickstart-portal.md) en Azure Portal.
