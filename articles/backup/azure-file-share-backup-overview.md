---
title: Acerca de la copia de seguridad de recursos compartidos de archivos de Azure
description: Aprenda a hacer una copia de seguridad de los recursos compartidos de archivos de Azure en el almacén de Recovery Services.
ms.topic: conceptual
ms.date: 03/05/2020
ms.openlocfilehash: 7441157d6346eefc256e9e7c29f9bb1fa5c13b79
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289525"
---
# <a name="about-azure-file-share-backup"></a>Acerca de la copia de seguridad de recursos compartidos de archivos de Azure

La copia de seguridad de recursos compartidos de archivos de Azure es una solución de copia de seguridad nativa basada en la nube que protege los datos en la nube y elimina las sobrecargas de mantenimiento adicionales implicadas en las soluciones de copia de seguridad locales. El servicio Azure Backup se integra completamente con Azure File Sync y permite centralizar los datos de recursos compartidos de archivos, así como las copias de seguridad. Esta solución sencilla, confiable y segura le permite configurar la protección de los recursos compartidos de archivos de su empresa en unos cuantos pasos fáciles con la garantía de poder recuperar los datos en caso de desastre.

## <a name="key-benefits-of-azure-file-share-backup"></a>Principales ventajas de la copia de seguridad de recursos compartidos de archivos de Azure

* **Cero infraestructura**: no se necesita ninguna implementación para configurar la protección de los recursos compartidos de archivos.
* **Retención personalizada**: Puede configurar copias de seguridad con retención diaria, semanal, mensual o anual según sus requisitos.
* **Funcionalidades de administración integradas**: puede programar copias de seguridad y especificar el período de retención deseado sin la sobrecarga adicional de eliminación de datos.
* **Restauración instantánea**: la copia de seguridad de recursos compartidos de archivos de Azure usa instantáneas de recursos compartidos de archivos, por lo que puede seleccionar justo los archivos que quiera restaurar al instante.
* **Alertas e informes**: puede configurar alertas para los errores de copia de seguridad y restauración y usar la solución de generación de informes que se proporciona en Azure Backup para obtener información sobre las copias de seguridad en los recursos compartidos de archivos.
* **Protección contra la eliminación accidental de recursos compartidos de archivos**: Azure Backup habilita la [característica de eliminación temporal](../storage/files/storage-files-prevent-file-share-deletion.md) en el nivel de cuenta de almacenamiento con un período de retención de 14 días. Incluso si un actor malintencionado elimina el recurso compartido de archivos, su contenido y puntos de recuperación (instantáneas) se conservan durante un período de retención configurable, lo que permite recuperar correctamente y por completo el contenido de origen y las instantáneas sin pérdida de datos.

## <a name="architecture"></a>Architecture

![Arquitectura de copia de seguridad de recursos compartidos de archivos de Azure](./media/azure-file-share-backup-overview/azure-file-shares-backup-architecture.png)

## <a name="how-the-backup-process-works"></a>Cómo funciona el proceso de copia de seguridad

1. El primer paso para configurar la copia de seguridad de recursos compartidos de archivos de Azure es crear un almacén de Recovery Services. El almacén proporciona una vista consolidada de las copias de seguridad configuradas en diferentes cargas de trabajo.

2. Una vez que se crea un almacén, el servicio Azure Backup detecta las cuentas de almacenamiento que se pueden registrar en él. Puede seleccionar la cuenta de almacenamiento que hospeda los recursos compartidos de archivos que quiere proteger.

3. Después de seleccionar la cuenta de almacenamiento, el servicio Azure Backup muestra el conjunto de recursos compartidos de archivos presentes en ella y almacena sus nombres en el catálogo de niveles de administración.

4. Luego, configure la directiva de copia de seguridad (programación y retención) según sus requisitos y seleccione los recursos compartidos de archivos de los que quiere realizar una copia de seguridad. El servicio Azure Backup registra las programaciones en el plano de control para realizar copias de seguridad programadas.

5. Según la directiva especificada, el programador de Azure Backup desencadena las copias de seguridad a la hora programada. Como parte de ese trabajo, se crea la instantánea de recursos compartidos de archivos mediante la API de recurso compartido de archivos. Solo la dirección URL de la instantánea se almacena en el almacén de metadatos.

    >[!NOTE]
    >Los datos de los recursos compartidos de archivos no se transfieren al servicio Azure Backup, dado que este crea y administra las instantáneas que forman parte de la cuenta de almacenamiento y las copias de seguridad no se transfieren al almacén.

6. Puede restaurar el contenido de los recursos compartidos de archivos de Azure (archivos individuales o el recurso compartido completo) de las instantáneas disponibles en el recurso compartido de archivos de origen. Una vez que se desencadena la operación, la dirección URL de la instantánea se recupera del almacén de metadatos y los datos se muestran y transfieren desde la instantánea de origen al recurso compartido de archivos de destino que elija. 

7. Si usa Azure File Sync, el servicio de copia de seguridad indica al servicio de Azure File Sync las rutas de acceso de los archivos que se están restaurando, lo que desencadena un proceso de detección de cambios en segundo plano en estos archivos. Los archivos que han cambiado se sincronizan con el punto de conexión del servidor. Este proceso se produce en paralelo a la restauración original en el recurso compartido de archivos de Azure. 

8. Los datos de supervisión de los trabajos de copia de seguridad y restauración se insertan en el servicio de supervisión de Azure Backup. Esto le permite supervisar las copias de seguridad en la nube de los recursos compartidos de archivos en un único panel. Además, también puede configurar alertas o notificaciones por correo electrónico cuando resulte afectado el estado de la copia de seguridad. Se envían correos electrónicos a través del servicio de correo electrónico de Azure.

## <a name="backup-costs"></a>Costos de la copia de seguridad

Actualmente solo se le cobrará por las instantáneas, ya que la copia de seguridad de recursos compartidos de archivos de Azure es una solución basada en instantáneas. Los cargos por almacenamiento incurridos por las instantáneas se facturan junto con el uso de Azure Files según los detalles de precios que figuran [aquí](https://azure.microsoft.com/pricing/details/storage/files/).

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre cómo [realizar copias de seguridad de recursos compartidos de archivos de Azure](backup-afs.md).
* Encuentre respuestas a las [preguntas sobre la copia de seguridad de Azure Files](backup-azure-files-faq.md).
