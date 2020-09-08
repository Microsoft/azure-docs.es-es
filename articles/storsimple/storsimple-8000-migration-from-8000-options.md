---
title: Opciones de migración de datos de los dispositivos StorSimple serie 8000
description: Proporciona información general de las opciones para migrar datos desde la serie 8000 de StorSimple.
services: storsimple
author: twooley
ms.service: storsimple
ms.topic: how-to
ms.date: 03/25/2020
ms.author: twooley
ms.openlocfilehash: 92ab570b3e06d67d7986a8103e9622e95eb57e7e
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88783916"
---
# <a name="options-to-migrate-data-from-storsimple-8000-series"></a>Opciones para migrar datos desde la serie 8000 de StorSimple

> [!IMPORTANT]
> El 31 de julio de 2022, finalizará el estado de compatibilidad de la serie 8000 de StorSimple. Se recomienda que los clientes de la serie 8000 de StorSimple migren a una de las alternativas descritas en el documento.

La serie 8000 de StorSimple alcanzará el [fin del soporte técnico](https://support.microsoft.com/lifecycle/search?alpha=Azure%20StorSimple%208000%20Series) en diciembre de 2022. Los clientes que ejecutan la serie 8000 de StorSimple tienen una opción para actualizar a otros servicios híbridos de primera entidad de Azure. En este artículo se describen las opciones híbridas de Azure disponibles para migrar datos.

## <a name="migration-options"></a>Opciones de migración

Los clientes que usan la serie 8000 de StorSimple pueden utilizar las opciones de Azure o de terceros.

### <a name="azure-options"></a>Opciones de Azure

#### <a name="migrate-to-azure-file-sync"></a>Migración a Azure File Sync

Esta nueva opción de migración permite a los clientes almacenar los recursos compartidos de archivos de su organización en Azure Files. Estos recursos compartidos de archivos se pueden centralizar luego para acceder a ellos de forma local mediante Azure File Sync (AFS). AFS puede implementarse en un host de Windows Server. La migración de datos real se realiza entonces como una copia de host o mediante la herramienta de migración.

Para obtener más información sobre cómo migrar datos a Azure File Sync, vaya a [Migración de las series 8100 y 8600 de StorSimple a Azure File Sync](https://docs.microsoft.com/azure/storage/files/storage-files-migration-storsimple-8000).

### <a name="third-party-options"></a>Opciones de terceros

#### <a name="migrate-to-panzura-freedom-nas"></a>Migración a Panzura Freedom NAS

Los clientes tanto de las series 5000-7000 de StorSimple como de la serie 8000 de StorSimple pueden elegir Panzura Freedom NAS como opción de migración para mantener sus datos en Azure. La solución Panzura Freedom proporciona una solución NAS que abarca centros de datos, oficinas, y nubes públicas y privadas. La solución permite flujos de trabajo de locales, híbridos y en la nube para clientes móviles, SMB y NFS.

Esta migración es compatible con Panzura y los clientes pueden comenzar solicitando soporte técnico para la migración desde el [sitio web de Panzura](https://panzura.com/migrate-storsimple-panzura/).

#### <a name="migrate-to-nasuni"></a>Migración a Nasuni

El traslado de todo el entorno de StorSimple a una plataforma de servicios de archivos de alto rendimiento estable y segura es un procedimiento fácil con Nasuni. Nasuni ofrece la seguridad y el rendimiento del almacenamiento de archivos local, al mismo tiempo que se combina con la escalabilidad y la durabilidad de Azure.  Como principal proveedor de software independiente (ISV) de Azure, Nasuni reúne todas las herramientas necesarias para trasladar los datos de StorSimple a una plataforma moderna que le permita compartir y colaborar con sus archivos en varias ubicaciones.

Comience hoy mismo: [Sitio web de Nasuni](https://info.nasuni.com/storsimple8000-webinar).

<!-- 04/09/2020 v-grpr (priestlg) - As per request, commenting out this section because the information that will go into this section is forthcoming
#### Migrate to Cohesity

Cohesity enables you to migrate data from your current StorSimple 5000–7000 to the Cohesity Data Platform on Azure. The Cohesity Data Platform is a software-defined web-scale solution that consolidates files, backups, objects, and VMs onto a single cloud-native solution. After migration to the Data Platform, you can manage, protect, and provision data and apps from cloud to core through a single pane of glass. With Cohesity, start with as few as three nodes. 

Learn more on [migration to the Cohesity Data Platform](https://info.cohesity.com/migrate-from-storsimple-to-cohesity.html).

#### Migrate to Nasuni

Nasuni makes it easy for StorSimple 5000-7000 customers to migrate and keep their data in Azure.  Nasuni is a leading Azure-based NAS storage solution, giving customers the performance and security they expect from on-prem solutions, with cloud economics and scale.  In addition to high performance file storage, Nasuni and Azure handle backup and DR, while allowing you to share and collaborate on your data around the globe with centralized file storage management. 

Nasuni has the experience to make your migration easy – get started today: https://info.nasuni.com/nasuni-storsimple-migration

#### Migrate to Talon FAST

Talon makes it easy for StorSimple 5000-7000 customers to continue to leverage the benefits they valued so much in the StorSimple platform (small on-site footprint backed by unlimited cloud resources) with even greater function.  With the Talon FAST solution, customers can migrate and keep their data in Azure, while now having an even smaller software-only onsite footprint and adding benefits such as global file locking, global namespace, and multi-site collaboration.  Talon is a leading Azure ecosystem solution, working with global customers to migrate their on-premises file server workloads into a consolidated, Azure-based footprint without compromising user workflow or experience.  

Learn more about how to evolve to a cloud-consolidated enterprise at https://www.talonstorage.com/alliances/microsoft-storsimple.
-->

## <a name="migration---frequently-asked-questions"></a>Migración: preguntas frecuentes

### <a name="q-when-do-the-storsimple-8000-series-devices-reach-end-of-service"></a>Q. ¿Cuándo llega la serie 8000 de StorSimple al final del servicio?

A. La serie 8000 de StorSimple alcanza el [fin del soporte técnico](https://support.microsoft.com/[lifecycle/search?alpha=Azure%20StorSimple%208000%20Series) en diciembre de 2022. El final del soporte técnico implica que Microsoft ya no podrá proporcionar soporte técnico para hardware y software de estos dispositivos después de diciembre de 2022. Se recomienda encarecidamente que empiece a formular ya un plan para migrar los datos de sus dispositivos.

### <a name="q-what-happens-to-the-data-i-have-stored-in-azure"></a>Q. ¿Qué ocurre con los datos que he almacenado en Azure?  

A. Podrá seguir usando los datos en Azure cuando haya migrado a un servicio más reciente.

### <a name="q-what-happens-to-the-data-i-have-stored-locally-on-my-storsimple-device"></a>Q. ¿Qué ocurre con los datos que he almacenado localmente en el dispositivo de StorSimple?

A. Los datos que se encuentran en el dispositivo local se pueden copiar en el servicio más reciente, como se describe en los documentos de migración.

### <a name="q-what-happens-if-i-want-to-keep-my-storsimple-8000-series-appliance"></a>Q. ¿Qué ocurre si quiero mantener mi dispositivo de la serie 8000 de StorSimple?

A. Si bien los servicios podrían seguir funcionando, Microsoft ya no podrá proporcionar soporte técnico de hardware y software. Se recomienda totalmente la migración para la continuidad empresarial.

### <a name="q-what-options-are-available-to-migrate-data-from-storsimple-8000-series-devices"></a>Q. ¿Qué opciones están disponibles para migrar datos de los dispositivos de la serie 8000 de StorSimple?

A. Dependiendo de su escenario, los usuarios de la serie 8000 de StorSimple tienen las siguientes opciones de migración:

* **Migración a Azure File Sync**: use esta opción si quiere cambiar al formato nativo de Azure. Puede usar Azure File Sync para la administración centralizada de recursos compartidos de archivos.

* **Otras opciones**: Puede ponerse en contacto con el soporte técnico de Microsoft para analizar las opciones de migración que no aparecen aquí.

### <a name="q-is-migration-to-other-storage-solutions-supported"></a>Q. ¿Se admite la migración a otras soluciones de almacenamiento?

A. Sí. Se admite la migración a otras soluciones de almacenamiento mediante la copia de host de los datos.

### <a name="q-is-migration-supported-by-microsoft"></a>Q. ¿Microsoft admite la migración?

A. La migración desde la serie 8000 es una operación completamente admitida. De hecho, Microsoft recomienda ponerse en contacto con el soporte técnico antes de iniciar la migración. La migración es actualmente una operación asistida. Si va a migrar datos desde un dispositivo de la serie 8000 de StorSimple, [póngase en contacto con el soporte técnico de StorSimple](mailto:storsimp@microsoft.com).

### <a name="q-what-is-the-pricing-model-for-migration-to-azure-file-sync"></a>Q. ¿Cuál es el modelo de precios para la migración a Azure File Sync?

A. Cuando se usa Azure File Sync, pueden aplicarse tarifas de suscripción para el servicio. Los clientes también tendrán que pagar los costos de almacenamiento en curso. Consulte los [precios de AFS]( https://azure.microsoft.com/pricing/details/storage/files/) para obtener una estimación.

### <a name="q-how-long-does-it-take-to-complete-a-migration"></a>Q. ¿Cuánto tiempo se tarda en completar una migración?

A. El tiempo para migrar los datos depende de la cantidad de los datos y la opción de actualización seleccionada.

## <a name="next-steps"></a>Pasos siguientes

* [Migración de datos de la serie 8000 de StorSimple a Azure File Sync](https://docs.microsoft.com/azure/storage/files/storage-files-migration-storsimple-8000)
