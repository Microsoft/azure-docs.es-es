---
title: Preguntas frecuentes sobre los discos
description: Preguntas más frecuentes sobre los discos de máquina virtual Linux de IaaS de Azure y los discos premium (administrados y no administrados)
author: roygara
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 06/15/2017
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: 0affcb3c1bab6eb5616c69bb15faf423895328b0
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89322524"
---
# <a name="frequently-asked-questions-about-azure-iaas-vm-disks-and-managed-and-unmanaged-premium-disks"></a>Preguntas más frecuentes sobre los discos de máquina virtual de IaaS de Azure y los discos premium administrados y no administrados

En este artículo se responden algunas de las preguntas más frecuentes acerca de Azure Managed Disks y los discos SSD Premium de Azure.

## <a name="managed-disks"></a>Managed Disks

**¿Qué es Azure Managed Disks?**

Managed Disks es una característica que simplifica la administración de discos para máquinas virtuales de IaaS de Azure al controlar automáticamente la administración de la cuenta de almacenamiento. Para obtener más información, consulte [Introducción a Azure Managed Disks](managed-disks-overview.md).

**Si creo un disco administrado estándar a partir un disco duro virtual existente con 80 GB de tamaño, ¿cuánto me costará?**

Un disco administrado estándar creado a partir de un disco duro virtual de 80 GB se trata como el siguiente tamaño de disco estándar disponible, es decir, un disco S10. Se le cobrará según el precio de disco S10. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/storage)para obtener más información.

**¿Existen costes de transacción para los discos administrados estándar?**

Sí. Sí, se le cobrará por cada transacción. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/storage)para obtener más información.

**Para un disco administrado estándar, ¿se me cobrará por el tamaño real de los datos en el disco o la capacidad aprovisionada del disco?**

Se le cobra en función de la capacidad aprovisionada del disco. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/storage)para obtener más información.

**¿En qué se diferencian los precios de los discos administrados premium de los de los discos no administrados?**

Los precios de los discos administrados premium son iguales que los de los discos no administrados premium.

**¿Puedo cambiar el tipo de cuenta de almacenamiento (Estándar o Premium) de mis discos administrados?**

Sí. Puede cambiar el tipo de cuenta de almacenamiento de los discos administrados mediante Azure Portal, PowerShell o la CLI de Azure.

**¿Puedo usar un archivo de disco duro virtual en una cuenta de Azure Storage para crear un disco administrado con una suscripción distinta?**

Sí.

**¿Puedo usar un archivo de disco duro virtual en una cuenta de Azure Storage para crear un disco administrado en una región diferente?**

No.

**¿Hay alguna limitación de escala para los clientes que usen discos administrados?**

Managed Disks elimina los límites asociados a las cuentas de almacenamiento. Sin embargo, el límite máximo es de 50 000 discos administrados por región y por tipo de disco para una suscripción.

**¿Pueden las máquinas virtuales de un conjunto de disponibilidad estar compuestas de una combinación de discos administrados y no administrados?**

No. Las máquinas virtuales de un conjunto de disponibilidad deben utilizar únicamente discos administrados o no administrados. Cuando cree un conjunto de disponibilidad, puede elegir qué tipo de discos desea usar.

**¿Es Managed Disks la opción predeterminada en Azure Portal?**

Sí.

**¿Puedo crear un disco administrado vacío?**

Sí. Puede crear un disco vacío. Un disco administrado se puede crear de forma independiente de una máquina virtual, por ejemplo, sin conectarlo a una máquina virtual.

**¿Qué número de dominios de error se admite para un conjunto de disponibilidad que use Managed Disks?**

En función de la región en la que se encuentre el conjunto de disponibilidad que use Managed Disks, el número de dominios de error admitidos es de 2 o 3.

**¿Cómo se configura una cuenta de almacenamiento estándar para el diagnóstico?**

Se configura una cuenta de almacenamiento privada para el diagnóstico de máquinas virtuales.

**¿Qué tipo de compatibilidad con el Control de acceso basado en roles está disponible para Managed Disks?**

Managed Disks admite tres roles predeterminados fundamentales:

* Propietario: Puede administrar todo el contenido, incluido el acceso
* Colaborador: Puede administrar todo el contenido, excepto el acceso
* Lector: Puede ver todo el contenido, pero no puede realizar cambios

**¿Hay alguna manera de copiar o exportar un disco administrado a una cuenta de almacenamiento privada?**

Puede generar un identificador URI de firma de acceso compartido (SAS) de solo lectura para el disco administrado y usarlo para copiar el contenido a una cuenta de almacenamiento privada o al almacenamiento local. Puede usar el identificador URI de SAS con Azure Portal, Azure PowerShell, la CLI de Azure o [AzCopy](../storage/common/storage-use-azcopy-v10.md).

**¿Puedo crear una copia de mi disco administrado?**

Los clientes pueden crear una instantánea de sus discos administrados y usarla para crear otro disco administrado.

**¿Siguen siendo compatibles los discos no administrados?**

Sí, se admiten discos administrados y no administrados. Recomendamos que empiece a usar discos administrados para las cargas de trabajo nuevas y migre las cargas de trabajo actuales a discos administrados.

**¿Puedo localizar conjuntamente discos administrados y no administrados en la misma VM?**

No.

**Si creo un disco de 128 GB y después aumento el tamaño a 130 gibibytes (GiB), ¿se me cobra por el siguiente tamaño de disco (256 GiB)?**

Sí.

**¿Puedo crear discos administrados para el almacenamiento con redundancia local, almacenamiento con redundancia geográfica y almacenamiento con redundancia de zona?**

Actualmente, Azure Managed Disks solo admite discos administrados de almacenamiento con redundancia local.

**¿Puedo reducir mis discos administrados?**

No. En la actualidad no se admite esta característica.

**¿Se puede interrumpir una concesión en el disco?**

No. Esto no es posible actualmente, ya que la concesión está presente para evitar una eliminación accidental cuando se está utilizando el disco.

**¿Se puede cambiar la propiedad de nombre de equipo cuando se usa un disco de sistema operativo especializado (no creado mediante la herramienta de preparación del sistema o generalizado) para aprovisionar una máquina virtual?**

No. No se puede actualizar la propiedad de nombre de equipo. La nueva máquina virtual lo hereda de la máquina virtual principal que se usó para crear el disco del sistema operativo. 

**¿Dónde puedo encontrar plantillas de Azure Resource Manager de ejemplo para crear máquinas virtuales con discos administrados**
* [Lista de plantillas mediante discos administrados](https://github.com/Azure/azure-quickstart-templates/)
* https://github.com/chagarw/MDPP

**Al crear un disco desde un blob, ¿hay alguna relación continuamente existente con ese blob de origen?**

No, cuando se crea el nuevo disco es una copia completa independiente de ese blob en ese momento y no hay ninguna conexión entre los dos. Si lo desea, una vez que haya creado el disco, puede eliminar el blob de origen sin afectar al disco recién creado en modo alguno.

**¿Se puede cambiar el nombre de un disco administrado o no administrado después de haberlo creado?**

No se puede cambiar el nombre de los discos administrados. Sin embargo, es posible cambiar el nombre de un disco no administrado; siempre y cuando no esté asociado a una VM o a un disco duro virtual.

**¿Puedo usar la creación de particiones de GPT en un disco de Azure?**

Las imágenes de generación 1 solo pueden usar particiones GPT en discos de datos, no en discos del sistema operativo. Los discos de SO deben usar el estilo de partición de MBR.

[Las imágenes de generación 2](https://docs.microsoft.com/azure/virtual-machines/linux/generation-2) pueden utilizar la creación de particiones GPT en el disco del sistema operativo, así como en los discos de datos.

**¿Qué tipos de discos admiten instantáneas?**

SSD Premium, SSD estándar y HDD estándar admiten instantáneas. En estos tres tipos de discos, las instantáneas se admiten en todos los tamaños de disco (incluidos los discos de hasta 32 TiB). Los discos Ultra no admiten instantáneas.

**¿Qué son las reservas de discos de Azure?**
La reserva de discos es la opción de adquirir de antemano un año de almacenamiento en discos, lo que reduce los costos totales. Para más información sobre las reservas de discos de Azure, consulte nuestro artículo sobre el tema: [Aplicación del descuento por reserva a Azure Disk](../cost-management-billing/reservations/understand-disk-reservations.md)

**¿Qué opciones ofrece la reserva de discos de Azure?**     
La reserva de discos de Azure ofrece la opción de comprar SSD Premium en las SKU especificadas de P30 (1 TiB) hasta P80 (32 TiB) durante un año. No hay ninguna limitación en la cantidad mínima de discos necesarios para comprar una reserva de discos. Además, puede pagar realizando un solo pago inicial o mediante pagos mensuales. No hay ningún costo transaccional adicional aplicado en Managed Disks de SSD Premium.    

Las reservas se realizan en forma de discos y no de capacidad. En otras palabras, cuando se reserva un disco de P80 (32 TiB), se obtiene un solo disco de P80, no se puede dividir esa reserva específica en dos discos de P70 más pequeños (16 TiB). Por supuesto, puede reservar el número de discos que quiera, incluidos dos discos de P70 independientes (16 TiB).

**¿Cómo se aplica la reserva de discos de Azure?**     
La reserva de discos sigue un modelo similar a las instancias reservadas de máquina virtual (VM). La diferencia es que una reserva de discos no se puede aplicar a diferentes SKU, mientras que una instancia de VM sí puede. Consulte [Ahorro de costos con Azure Reserved VM Instances](./linux/prepay-reserved-vm-instances.md) para obtener más información sobre las instancias de VM.     

**¿Puedo usar mi almacenamiento de datos adquirido mediante la reserva de discos de Azure en varias regiones?**     
La reserva de discos de Azure se compra para una región específica y una SKU (como P30 en el Este de EE. UU. 2) y, por lo tanto, no se puede usar fuera de estas áreas. Siempre puede adquirir una reserva de discos de Azure adicional en función de las necesidades de almacenamiento en disco que tenga en otras regiones o SKU.    

**¿Qué ocurre cuando expira la reserva de discos Azure?**     
Recibirá notificaciones por correo electrónico 30 días antes de la expiración y también en la fecha de expiración. Una vez que expire la reserva, los discos implementadas seguirán ejecutándose y se facturarán mediante una [cuota de pago por uso](https://azure.microsoft.com/pricing/details/managed-disks/).

**¿Los discos SSD estándar admiten "Acuerdo de Nivel de Servicio de máquina virtual de instancia única"?**

Sí, todos los tipos de disco admiten SLA de VM de instancia única.

### <a name="azure-shared-disks"></a>Discos compartidos de Azure

**¿Se admite la característica de discos compartidos para discos no administrados o blobs en páginas?**

No, solo se admite en discos Ultra y discos SSD administrados Premium.

**¿En qué regiones se admiten los discos compartidos?**

Para obtener información regional, consulte el [artículo conceptual](./linux/disks-shared.md).

**¿Se pueden usar discos compartidos como disco del sistema operativo?**

No, los discos compartidos solo se admiten para discos de datos.

**¿Qué tamaños de disco admiten los discos compartidos?**

Para ver los tamaños admitidos, consulte el [artículo conceptual](./linux/disks-shared.md).

**Si tengo un disco, ¿puedo habilitar discos compartidos en él?**

Todos los discos administrados creados con la versión de API 2019-07-01 o posterior pueden habilitar discos compartidos. Para ello, debe desmontar el disco de todas las VM a las que esté conectado. Después, edite la propiedad **maxShares** en el disco.

**Si ya no deseo usar un disco en modo compartido, ¿cómo puedo deshabilitarlo?**

Desmonta el disco de todas las VM a las que esté conectado. Después, edite la propiedad maxShare en el disco y defínala en 1.

**¿Se puede cambiar el tamaño de un disco compartido?**

Sí.

**¿Se puede habilitar el acelerador de escritura en un disco que también tiene habilitados los discos compartidos?**

No.

**¿Puedo habilitar el almacenamiento en caché en un disco que tiene habilitados los discos compartidos?**

La única opción de almacenamiento en caché de host admitida es **Ninguno**.

## <a name="ultra-disks"></a>Discos Ultra

**¿Qué rendimiento del disco Ultra se debe establecer?**
Si no está seguro de qué rendimiento de disco establecer, se recomienda que comience asumiendo un tamaño de E/S de 16 KiB y ajuste el rendimiento a partir de ahí mientras supervisa la aplicación. La fórmula es: Rendimiento en MBps = n.º de IOPS * 16/1000.

**He configurado el disco en 40 000 IOPS pero solo veo 12 800 IOPS, ¿por qué no veo el rendimiento del disco?**
Además de la limitación del disco, existe una limitación de E/S que se impone a nivel de máquina virtual. Asegúrese de que el tamaño de la VM que está usando pueda admitir los niveles que están configurados en los discos. Para más información sobre los límites de E/S impuestos por la máquina virtual, vea [Tamaños de las máquinas virtuales en Azure](sizes.md).

**¿Puedo usar niveles de almacenamiento en caché con un disco Ultra?**
No, los discos Ultra no admiten los distintos métodos de almacenamiento en caché que se admiten en otros tipos de discos. Establezca el almacenamiento en caché de disco en **Ninguno**.

**¿Puedo conectar un disco Ultra a mi máquina virtual actual?**
Es posible. La máquina virtual debe encontrarse en un par de región y zona de disponibilidad que admita discos Ultra. Consulte [Introducción a los discos Ultra](disks-enable-ultra-ssd.md) para más información.

**¿Puedo usar un disco Ultra como disco del sistema operativo para mi máquina virtual?**
No, los discos Ultra solo se admiten como discos de datos y solo se admiten como discos nativos de 4K.

**¿Puedo convertir un disco existente en un disco Ultra?**
No, pero puede migrar los datos de un disco existente a un disco Ultra. Para migrar un disco existente a un disco Ultra, conecte ambos discos a la misma máquina virtual y copie los datos de un disco al otro o utilice una solución de terceros para migrar los datos.

**¿Se pueden crear instantáneas de discos Ultra?**
No, las instantáneas no están disponibles todavía.

**¿Está Azure Backup disponible para discos Ultra?**
No, la compatibilidad con Azure Backup aún no está disponible.

**¿Puedo conectar un disco Ultra a una máquina virtual que se ejecuta en un conjunto de disponibilidad?**
No, esto aún no es posible.

**¿Puedo habilitar Azure Site Recovery para máquinas virtuales que usan discos Ultra?**
No, Azure Site Recovery todavía no es compatible con discos Ultra.

## <a name="uploading-to-a-managed-disk"></a>Carga en un disco administrado

**¿Puedo cargar datos en un disco administrado existente?**

No, la carga solo se puede usar durante la creación de un nuevo disco vacío con el estado **ReadyToUpload**.

**¿Cómo se carga en un disco administrado?**

Cree un disco administrado con la propiedad [createOption](https://docs.microsoft.com/rest/api/compute/disks/createorupdate#diskcreateoption) de [creationData](https://docs.microsoft.com/rest/api/compute/disks/createorupdate#creationdata) establecida en "Upload" y, después, puede cargar datos en él.

**¿Puedo conectar un disco a una máquina virtual mientras se encuentra en un estado de carga?**

No.

**¿Se puede tomar una instantánea de un disco administrado en un estado de carga?**

No.

## <a name="standard-ssd-disks"></a>Discos SSD estándar

**¿Qué son los discos SSD estándar de Azure?**
Los discos SSD estándar son discos estándar respaldados por soportes físicos, optimizados como solución de almacenamiento rentable para cargas de trabajo que necesitan un rendimiento constante en niveles inferiores de IOPS.

<a id="standard-ssds-azure-regions"></a> **¿Cuáles son las regiones que se admiten actualmente para los discos SSD estándar?**
Todas las regiones de Azure ahora admiten discos SSD estándar.

**¿Azure Backup está disponible al usar SSD estándar?**
Sí, Azure Backup ya está disponible.

**¿Cuál es la ventaja de utilizar discos SSD estándar en lugar de unidades de disco duro?**
Los discos SSD estándar ofrecen una mejor latencia, coherencia, disponibilidad y confiabilidad en comparación con los discos HDD. Debido a esto, las cargas de trabajo de las aplicaciones se ejecutan mucho mejor. Tenga en cuenta que los discos SSD Premium son la solución recomendada para la mayoría de las cargas de trabajo producción con uso intensivo de E/S.

**¿Puedo usar discos SSD estándar como discos no administrados?**
No, los discos SSD estándar solo están disponibles como discos administrados.

## <a name="migrate-to-managed-disks"></a>Migración a Managed Disks

**¿Hay algún impacto de la migración en el rendimiento de Managed Disks?**

La migración conlleva el traslado del disco de una ubicación de almacenamiento a otra. Esto se orquesta mediante una copia en segundo plano de los datos que puede tardar varias horas en completarse, normalmente menos de 24 horas en función de la cantidad de datos en los discos. Durante ese tiempo, la aplicación puede experimentar una latencia de lectura más alta de lo habitual, ya que algunas lecturas pueden redirigirse a la ubicación original y pueden tardar más tiempo en completarse. No hay ningún impacto en la latencia de escritura durante este período.  

**¿Qué cambios son necesarios en una configuración del servicio Azure Backup preexistente antes y después de la migración a Managed Disks?**

No es preciso realizar cambios.

**¿Las copias de seguridad de mi máquina virtual creadas con el servicio Azure Backup antes de la migración seguirán funcionando?**

Sí, las copias de seguridad funcionan sin problemas.

**¿Qué cambios son necesarios en una configuración de Azure Disks Encryption ya existente antes y después de la migración a Managed Disks?**

No es preciso realizar cambios.

**¿Se admite la migración automática de un conjunto de escalado de máquinas virtuales existente de discos no administrados a Managed Disks?**

No. Puede crear un nuevo conjunto de escalado con Managed Disks con la imagen del antiguo conjunto de escalado con discos no administrados.

**¿Puedo crear un disco administrado desde una instantánea de blob en páginas tomada antes de migrar a Managed Disks?**

No. Puede exportar una instantánea del blob en páginas como un blob en páginas y, a continuación, crear un disco administrado a partir del blob en páginas exportado.

**¿Se puede realizar una conmutación por error de mis equipos locales protegidos por Azure Site Recovery a una máquina virtual con Managed Disks?**

Sí, puede realizar la conmutación por error a una máquina virtual con Managed Disks.

**¿Hay algún impacto en la migración de las máquinas virtuales de Azure protegidas por Azure Site Recovery mediante la replicación de Azure a Azure?**

No. Está disponible la protección de Azure a Azure de Azure Site Recovery para máquinas virtuales con Managed Disks.

**¿Puedo migrar máquinas virtuales con discos no administrados que se encuentran en las cuentas de almacenamiento que se hayan cifrado previamente en discos administrados?**

Sí

## <a name="managed-disks-and-storage-service-encryption"></a>Managed Disks y Storage Service Encryption

**¿Está habilitado el cifrado del lado servidor de forma predeterminada cuando se crea un disco administrado?**

Sí. Los discos administrados se cifran con el cifrado del lado servidor con las claves administrada por la plataforma. 

**¿El volumen de arranque está cifrado de forma predeterminada en un disco administrado?**

Sí. De forma predeterminada, se cifran todos los discos administrados, incluido el disco del sistema operativo.

**¿Quién administra las claves de cifrado?**

Microsoft administra las claves administradas por la plataforma. También puede usar y administrar sus propias claves almacenadas en Azure Key Vault. 

**¿Puedo deshabilitar el cifrado del lado servidor de mis discos administrados?**

No.

**¿El cifrado del lado servidor solo está disponible en regiones específicas?**

No. El cifrado del lado servidor con las claves administradas por el cliente y la plataforma está disponible en todas las regiones donde Managed Disks está disponible. 

**¿Azure Site Recovery admite el cifrado del lado servidor con la clave administrada por el cliente para escenarios de recuperación ante desastres locales a Azure y Azure en Azure?**

Sí. 

**¿Puedo realizar una copia de seguridad de Managed Disks cifrado con el cifrado del lado servidor con la clave administrada por el cliente mediante el servicio de Azure Backup?**

Sí.

**¿Están cifradas las imágenes e instantáneas administradas?**

Sí. Todas las instantáneas e imágenes administradas se cifran automáticamente. 

**¿Puedo convertir máquinas virtuales con discos no administrados que se encuentran en las cuentas de almacenamiento o que se hayan cifrado previamente en discos administrados?**

Sí

**¿Se cifrará también un VHD exportado de un disco administrado o de una instantánea?**

No. Pero si exporta un disco duro virtual a una cuenta de almacenamiento cifrada desde un disco administrado cifrado o una instantánea, en este caso se cifra. 

## <a name="premium-disks-managed-and-unmanaged"></a>Discos premium: Administrados y no administrados

**Si una máquina virtual usa una serie de tamaño que admite discos SSD premium, como DSv2, ¿puedo conectar discos de datos tanto premium como estándar?** 

Sí.

**¿Puedo conectar discos de datos tanto premium como estándar a una serie de tamaño que no admita discos SSD premium, por ejemplo, las series D, Dv2, G o F?**

No. Solo puede conectar discos de datos estándar a máquinas virtuales que no utilicen una serie de tamaño que admita discos SSD premium.

**Si creo un disco de datos premium a partir un disco duro virtual existente con 80 GB, ¿cuánto me costará?**

Un disco de datos premium creado a partir de un disco duro virtual de 80 GB se trata como el siguiente tamaño de disco premium disponible, es decir, P10. Se le cobrará según el precio de disco P10.

**¿Hay costos de transacción cuando se usan discos SSD premium?**

Existe un costo fijo para cada tamaño de disco que esté aprovisionado con límites específicos de IOPS y rendimiento. Los demás costes son por el ancho de banda de salida y la capacidad para instantáneas, si corresponde. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/storage)para obtener más información.

**¿Cuáles son los límites de IOPS y rendimiento que puedo obtener de la caché de disco?**

Los límites combinados de memoria caché y SSD local para la serie DS son 4000 IOPS por núcleo y 33 MiB por segundo y núcleo. La serie GS ofrece 5000 IOPS por núcleo y 50 MiB por segundo y núcleo.

**¿Es el SSD local compatible con máquinas virtuales de Managed Disks?**

El SSD local es un almacenamiento temporal que se incluye con una máquina virtual de discos administrados. No existe ningún costo extra para este almacenamiento temporal. Recomendamos que no use este SSD local para almacenar los datos de la aplicación, ya que no se conserva en Azure Blob Storage.

**¿Hay alguna repercusión en el uso de TRIM en discos premium?**

No hay ningún inconveniente a la hora de usar TRIM en discos de Azure, ya sea en discos estándar o premium.

## <a name="new-disk-sizes-managed-and-unmanaged"></a>Nuevos tamaños de disco: Administrados y no administrados

**¿Qué regiones admiten la capacidad de ráfagas para el tamaño de disco SSD Premium aplicable?**

La capacidad de creación de ráfagas actualmente se admite en todas las regiones de la nube pública de Azure. Próximamente, estará disponible la compatibilidad con nubes independientes. 

**¿En qué regiones se admiten los tamaños de disco administrado de 4/8/16 GiB (P1/P2/P3, E1/E2/E3)?**

Estos tamaños de disco nuevos actualmente se admiten en todas las regiones de la nube pública de Azure. Próximamente, estará disponible la compatibilidad con nubes independientes. 

**¿Se admiten los tamaños de disco P1/P2/P3 en discos no administrados o blobs en páginas?**

No, solo se admite en discos administrados SSD premium admitidos. 

**¿Se admiten los tamaños de disco E1/E2/E3 en discos no administrados o blobs en páginas?**

No, los discos administrados SSD estándar de cualquier tamaño no se pueden usar con discos no administrados ni blobs en páginas.

**¿Cuál es el mayor tamaño de disco administrado compatible con discos de datos y sistema operativo?**

El tipo de partición compatible con Azure para un disco del sistema operativo es el registro de arranque maestro (MBR). El formato MBR admite un tamaño de disco de hasta 2 TiB. El tamaño máximo que admite Azure para un disco del sistema operativo es de 4 TiB. Azure admite hasta 32 TiB para discos de datos administrados.

**¿Cuál es el mayor tamaño de disco administrado compatible con discos de datos y sistema operativo?**

El tipo de partición compatible con Azure para un disco del sistema operativo es el registro de arranque maestro (MBR). El formato MBR admite un tamaño de disco de hasta 2 TiB. El tamaño máximo que admite Azure para un disco no administrado del sistema operativo es de 4 TiB. Azure admite hasta 4 TiB para discos de datos no administrados.

**¿Cuál es el mayor tamaño de blob en páginas admitido?**

El mayor tamaño de blob en páginas que admite Azure es 8 TiB (8191 GiB). El tamaño máximo del blob en páginas cuando se conecta a una máquina virtual como discos de datos o de sistema operativo es de 4 TiB (4095 GiB).

**¿Es necesario usar una nueva versión de las herramientas de Azure para crear, conectar, cambiar de tamaño y cargar discos de más de 1 TiB?**

No es necesario actualizar las herramientas de Azure existentes para crear, conectar o cambiar el tamaño de los discos de más de 1 TiB. Para cargar el archivo de disco duro virtual del entorno local directamente en Azure como un blob en páginas o un disco no administrado, debe usar los conjuntos de herramientas más recientes que se indican a continuación. Solo se admiten cargas de disco duro virtual de hasta 8 TiB.

|Herramientas de Azure      | Versiones compatibles                                |
|-----------------|---------------------------------------------------|
|Azure PowerShell | Número de versión 4.1.0: Versión de junio de 2017 o posterior|
|CLI de Azure v1     | Número de versión 0.10.13: Versión de mayo de 2017 o posterior|
|CLI de Azure v2     | Número de versión 2.0.12: Versión de julio de 2017 o posterior|
|AzCopy              | Número de versión 6.1.0: Versión de junio de 2017 o posterior|

**¿Se admiten los tamaños de disco P4 y P6 para blobs en páginas o discos no administrados?**

Los tamaños de disco P4 (32 GiB) y P6 (64 GiB) no se admiten como niveles de disco predeterminados para discos no administrados y blobs en páginas. Deberá [establecer el nivel de blob](https://docs.microsoft.com/rest/api/storageservices/set-blob-tier) explícitamente en P4 y P6 para que el disco se asigne a estos niveles. Si implementa un disco no administrado o un blob en páginas con el tamaño de disco o la longitud de contenido inferior a 32 GiB o entre 32 y 64 GiB sin establecer el nivel de blob, continuará en el nivel P10 con 500 IOPS y 100 MiB/s y el plan de tarifa asignado.

**Si mi disco administrado premium existente de menos de 64 GiB se creó antes de habilitar el disco pequeño (alrededor del 15 de junio de 2017), ¿cómo se factura?**

Los discos pequeños premium de menos de 64 GiB se siguen facturando según el plan de tarifa P10.

**¿Cómo puedo cambiar el nivel de disco de los discos pequeños premium menores de 64 GiB de P10 a P4 o P6?**

Puede crear una instantánea de los discos pequeños y, después, crear un disco para cambiar automáticamente el plan de tarifa a P4 o P6, en función del tamaño aprovisionado.

**¿Puede cambiar el tamaño de las instancias de Managed Disks existentes de menos de 4 tebibytes (TiB) a los nuevos tamaños de disco recién incorporados de hasta 32 TiB?**

Sí.

**¿Cuales son los tamaños de disco más grandes que admiten Azure Backup y el servicio Azure Site Recovery?**

El tamaño de disco más grande que admite Azure Backup es de 32 TiB (4 TiB para discos cifrados). El tamaño de disco más grande que admite Azure Site Recovery es de 8 TiB. La compatibilidad con los discos de mayor tamaño de hasta 32 TiB aún no está disponible en Azure Site Recovery.

**¿Cuáles son los tamaños de máquina virtual recomendados para los tamaños de disco mayores (> 4TiB) para discos SSD estándar y HDD estándar con la finalidad de lograr niveles optimizados de ancho de banda e IOPS del disco?**

Para lograr el rendimiento de disco de los tamaños de discos grandes SSD y HDD estándar (>4 TiB) más allá de 500 IOPS y 60 MiB/s, se recomienda implementar una nueva máquina virtual de uno de los siguientes tamaños para optimizar el rendimiento: Máquinas virtuales de la serie B, serie DSv2, serie Dsv3, serie ESv3, serie Fs, serie Fsv2, serie M, serie GS, serie NCv2, serie NCv3 o serie Ls. La asociación de discos de gran tamaño a máquinas virtuales existentes o máquinas virtuales que no usan los tamaños recomendados anteriores puede dar lugar a un rendimiento inferior.

**¿Cómo puedo actualizar mis discos (> 4 TiB) que se implementaron durante la versión preliminar de tamaños de disco mayores con el fin de obtener el mayor ancho de banda e IOPS con carácter general?**

Puede detener e iniciar la máquina virtual a la que está conectado el disco o bien desasociar y volver a asociar el disco. Los objetivos de rendimiento de los tamaños de disco mayores han aumentado para discos SSD Premium y estándar con carácter general.

**¿En qué regiones se admiten los tamaños de disco administrado de 8 TiB, 16 TiB y 32 TiB?**

Las SKU de disco de 8 TiB, 16 TiB y 32 TiB se admiten en todas las regiones en Azure global, Microsoft Azure Government y Azure China 21Vianet.

**¿Se admite la habilitación del almacenamiento en caché del host en todos los tamaños de disco?**

El almacenamiento en caché del host (**ReadOnly** y **Read/Write**) se admite en tamaños de disco inferiores a 4 TiB. Esto significa que cualquier disco que esté aprovisionado hasta 4095 GiB puede aprovechar el almacenamiento en caché del host. El almacenamiento en caché del host no se admite para los tamaños de disco superiores o iguales que 4096 GiB. Por ejemplo, un disco prémium P50 aprovisionado con 4095 GiB puede aprovechar el almacenamiento en caché del host y un disco P50 aprovisionado con 4096 GiB no puede aprovechar dicho almacenamiento. Se recomienda aprovechar el almacenamiento en caché para tamaños de disco más pequeños, donde se puede esperar un mayor aumento del rendimiento con datos almacenados en caché en la máquina virtual.

## <a name="private-links-for-securely-exporting-and-importing-managed-disks"></a>Instancias de Private Link para exportar e importar Managed Disks de forma segura

**¿Cuál es la ventaja de usar instancias de Private Link para exportar e importar Managed Disks?**

Ahora puede aprovechar las instancias de Private Link para restringir la exportación e importación a Managed Disks solo desde su instancia de Azure Virtual Network. 

**¿Qué puedo hacer para asegurarme de que un disco se pueda exportar o importar solo a través de instancias de Private Link?**

Debe establecer la propiedad `DiskAccessId` en una instancia de un objeto de acceso a disco y establecer también la propiedad NetworkAccessPolicy en `AllowPrivate`.

**¿Puedo vincular varias redes virtuales al mismo objeto de acceso a disco?**

No. Actualmente, puede vincular un objeto de acceso a disco a una sola red virtual.

**¿Puedo vincular una red virtual a un objeto de acceso a disco en otra suscripción?**

No. Actualmente, puede vincular un objeto de acceso a disco a una red virtual en la misma suscripción.

**¿Puedo vincular una red virtual a un objeto de acceso a disco en otra suscripción?**

No. Actualmente, puede vincular un objeto de acceso a disco a una red virtual en la misma suscripción.

**¿Cuántas exportaciones o importaciones que usan el mismo objeto de acceso a disco pueden producirse a la vez?**

5

**¿Puedo usar un URI de SAS de un disco o instantánea para descargar el VHD subyacente de una VM en la misma subred que la subred del punto de conexión privado asociado al disco?**

Sí.

**¿Puedo usar un URI de SAS de un disco o instantánea para descargar el VHD subyacente de una VM que no esté en la misma subred que la subred del punto de conexión privado no asociado al disco?**

No.

## <a name="what-if-my-question-isnt-answered-here"></a>Mi pregunta no está respondida aquí. ¿Qué debo hacer?

Si su pregunta no aparece aquí, háganoslo saber y lo ayudaremos a encontrar una respuesta. Puede publicar una pregunta al final de este artículo en los comentarios. Para ponerse en contacto con el equipo de Azure Storage y otros miembros de la Comunidad sobre este artículo, use el MSDN [Página de preguntas y respuestas de Microsoft sobre Azure Storage](https://docs.microsoft.com/answers/products/azure?product=storage).

Para solicitar características, envíe sus solicitudes e ideas al [foro de comentarios de Azure Storage](https://feedback.azure.com/forums/217298-storage).
