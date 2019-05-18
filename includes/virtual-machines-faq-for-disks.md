---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 05/13/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 0ad006ca966cfcc2c817ae4e8bfd3dc2d477259e
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65607046"
---
# <a name="frequently-asked-questions-about-azure-iaas-vm-disks-and-managed-and-unmanaged-premium-disks"></a>Preguntas más frecuentes sobre los discos de máquina virtual de IaaS de Azure y los discos premium administrados y no administrados

En este artículo se responden algunas de las preguntas más frecuentes acerca de Azure Managed Disks y los discos SSD Premium de Azure.

## <a name="managed-disks"></a>Managed Disks

**¿Qué es Azure Managed Disks?**

Managed Disks es una característica que simplifica la administración de discos para máquinas virtuales de IaaS de Azure al controlar automáticamente la administración de la cuenta de almacenamiento. Para obtener más información, consulte [Introducción a Azure Managed Disks](../articles/virtual-machines/windows/managed-disks-overview.md).

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

**¿Puedo tomar una instantánea incremental de un disco administrado?**

No. La funcionalidad de instantánea actual realiza una copia completa de un disco administrado.

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

Puede generar un identificador URI de firma de acceso compartido (SAS) de solo lectura para el disco administrado y usarlo para copiar el contenido a una cuenta de almacenamiento privada o al almacenamiento local. Puede usar el identificador URI de SAS con Azure Portal, Azure PowerShell, la CLI de Azure o [AzCopy](../articles/storage/common/storage-use-azcopy.md).

**¿Puedo crear una copia de mi disco administrado?**

Los clientes pueden crear una instantánea de sus discos administrados y usarla para crear otro disco administrado.

**¿Siguen siendo compatibles los discos no administrados?**

Sí, se admiten discos administrados y no administrados. Recomendamos que empiece a usar discos administrados para las cargas de trabajo nuevas y migre las cargas de trabajo actuales a discos administrados.

**¿Puedo localizar conjuntamente discos administrados y no administrados en la misma máquina virtual?**

No.

**¿Si creo un disco de 128 GB y después lo aumento a 130 gibibytes (GiB), cobrará por el siguiente tamaño de disco (256 GiB)?**

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
* [Lista de plantillas mediante discos administrados](https://github.com/Azure/azure-quickstart-templates/blob/master/managed-disk-support-list.md)
* https://github.com/chagarw/MDPP

**Al crear un disco desde un blob, ¿hay alguna relación continuamente existente con ese blob de origen?**

No, cuando se crea el nuevo disco es una copia completa independiente de ese blob en ese momento y no hay ninguna conexión entre los dos. Si lo desea, una vez que haya creado el disco, puede eliminar el blob de origen sin afectar al disco recién creado en modo alguno.

**¿Se puede cambiar el nombre de un disco administrado o no administrado después de haberlo creado?**

No se puede cambiar el nombre de los discos administrados. Sin embargo, es posible cambiar el nombre de un disco no administrado; siempre y cuando no esté asociado a una VM o a un disco duro virtual.

**¿Puedo usar la creación de particiones de GPT en un disco de Azure?**

La creación de particiones de GPT solo se puede usar en discos de datos, no en discos de sistemas operativos. Los discos de SO deben usar el estilo de partición de MBR.

## <a name="uploading-to-a-managed-disk"></a>Cargar en un disco administrado

**¿Puedo cargar datos en un disco administrado existente?**

No, carga solo se puede usar durante la creación de un nuevo disco vacío con el **ReadyToUpload** estado.

**¿Puedo conectar un disco a una máquina virtual mientras está en un estado de carga?**

No.

**¿Puedo tomar una instantánea de un disco administrado en un estado de carga?**

No.

## <a name="standard-ssd-disks"></a>Discos SSD estándar

**¿Qué son los discos SSD estándar de Azure?**
Los discos SSD estándar son discos estándar respaldados por soportes físicos, optimizados como solución de almacenamiento rentable para cargas de trabajo que necesitan un rendimiento constante en niveles inferiores de IOPS.

<a id="standard-ssds-azure-regions"></a>**¿Cuáles son las regiones que se admiten actualmente para los discos SSD estándar?**
Todas las regiones de Azure ahora admiten discos SSD estándar.

**¿Azure Backup está disponible al usar SSD estándar?**
Sí, Azure Backup ya está disponible.

**¿Cómo creo discos SSD estándar?**
Puede crear discos SSD estándar con plantillas de Azure Resource Manager, SDK, PowerShell o CLI. A continuación se muestran los parámetros necesarios de la plantilla de Resource Manager para crear discos SSD estándar:

* La *apiVersion* para Microsoft.Compute se debe establecer en `2018-04-01` (o posterior)
* Especifique *managedDisk.storageAccountType* como `StandardSSD_LRS`

El siguiente ejemplo muestra la sección *properties.storageProfile.osDisk* de una máquina virtual que usa un disco SSD estándar:

```json
"osDisk": {
    "osType": "Windows",
    "name": "myOsDisk",
    "caching": "ReadWrite",
    "createOption": "FromImage",
    "managedDisk": {
        "storageAccountType": "StandardSSD_LRS"
    }
}
```

Para obtener un ejemplo de plantilla completa de cómo crear un disco SSD estándar con una plantilla, consulte [Create a VM from a Windows Image with Standard SSD Data Disks](https://github.com/azure/azure-quickstart-templates/tree/master/101-vm-with-standardssd-disk/) (Creación de una máquina virtual a partir de una imagen de Windows con discos de datos SSD estándar).

**¿Puedo convertir mis discos existentes a SSD estándar?**
 Sí, puede hacerlo. Consulte [Conversión de almacenamiento de Azure Managed Disks de estándar a premium, y viceversa](https://docs.microsoft.com/azure/virtual-machines/windows/convert-disk-storage) para conocer las directrices generales de la conversión de Managed Disks. Y utilice el siguiente valor para actualizar el tipo de disco a SSD estándar.
-AccountType StandardSSD_LRS

**¿Cuál es la ventaja de utilizar discos SSD estándar en lugar de unidades de disco duro?**
Los discos SSD estándar ofrecen una mejor latencia, coherencia, disponibilidad y confiabilidad en comparación con los discos de la unidad de disco duro. Debido a esto, las cargas de trabajo de las aplicaciones se ejecutan mucho mejor. Tenga en cuenta que los discos SSD Premium son la solución recomendada para la mayoría de las cargas de trabajo producción con uso intensivo de E/S.

**¿Puedo usar discos SSD estándar como discos no administrados?**
No, los discos SSD estándar solo están disponibles como discos administrados.

**¿Los discos SSD estándar admiten "Acuerdo de Nivel de Servicio de máquina virtual de instancia única"?**
No, los discos SSD estándar no tiene un Acuerdo de Nivel de Servicio de máquina virtual de instancia única. Use discos SSD Premium para tener ese acuerdo.

## <a name="migrate-to-managed-disks"></a>Migrar a Managed Disks

**¿Hay algún impacto de la migración en el rendimiento de Managed Disks?**

La migración conlleva el traslado del disco de una ubicación de almacenamiento a otra. Esto se coordina a través de la copia en segundo plano de datos, que pueden tardar varias horas en completarse, normalmente menos de 24 horas según la cantidad de datos de los discos. Durante ese tiempo, la aplicación puede experimentar una latencia de lectura más alta de lo habitual, ya que algunas lecturas pueden redirigirse a la ubicación original y pueden tardar más tiempo en completarse. No hay ningún impacto en la latencia de escritura durante este período.  

**¿Qué cambios son necesarios en una configuración del servicio Azure Backup preexistente antes y después de la migración a Managed Disks?**

No es preciso realizar cambios.

**¿Las copias de seguridad de mi máquina virtual creadas con el servicio Azure Backup antes de la migración seguirán funcionando?**

Sí, las copias de seguridad funcionan sin problemas.

**¿Qué cambios son necesarios en una configuración de Azure Disks Encryption ya existente antes y después de la migración a Managed Disks?**

No es preciso realizar cambios.

**¿Es automatizar la migración de un escalado de máquinas virtuales existentes conjunto de discos no administrados a Managed Disks?**

No. Puede crear un nuevo conjunto de escalado con Managed Disks con la imagen del antiguo conjunto de escalado con discos no administrados.

**¿Puedo crear un disco administrado desde una instantánea de blob en páginas tomada antes de migrar a Managed Disks?**

No. Puede exportar una instantánea del blob en páginas como un blob en páginas y, a continuación, crear un disco administrado a partir del blob en páginas exportado.

**¿Se puede realizar una conmutación por error de mis equipos locales protegidos por Azure Site Recovery a una máquina virtual con Managed Disks?**

Sí, puede realizar la conmutación por error a una máquina virtual con Managed Disks.

**¿Hay algún impacto en la migración de las máquinas virtuales de Azure protegidas por Azure Site Recovery mediante la replicación de Azure a Azure?**

Sí. Actualmente, Azure Site Recovery Azure para protección de Azure para máquinas virtuales con Managed Disks está disponible como un servicio de disponibilidad general.

**¿Puedo migrar máquinas virtuales con discos no administrados que se encuentran en las cuentas de almacenamiento que se hayan cifrado previamente en discos administrados?**

Sí

## <a name="managed-disks-and-storage-service-encryption"></a>Managed Disks y Storage Service Encryption

**¿Está habilitado el servicio Storage Service Encryption de forma predeterminada al crear un disco administrado?**

Sí.

**¿Quién administra las claves de cifrado?**

Microsoft administra las claves de cifrado.

**¿Puedo deshabilitar Storage Service Encryption para Managed Disks?**

No.

**¿Storage Service Encryption está solo disponible en determinadas regiones?**

No. Está en todas las regiones donde esté disponible Managed Disks. Managed Disks está disponible en todas las regiones públicas y Alemania. También está disponible en China, sin embargo, solo para las claves administradas por Microsoft, no para las claves administradas por el cliente.

**¿Cómo averiguo si mi disco administrado está cifrado?**

Puede averiguar la hora de creación de un disco administrado desde Azure Portal, la CLI de Azure y PowerShell. Si la hora es posterior al 9 de junio de 2017, el disco está cifrado.

**¿Cómo puedo cifrar mis discos existentes que se crearon antes del 10 de junio de 2017?**

A partir del 10 de junio de 2017 los nuevos datos escritos en los discos administrados existentes se cifran automáticamente. También tenemos previsto cifrar los datos existentes y el cifrado se realizará de forma asincrónica en segundo plano. Si debe cifrar ahora los datos existentes, cree una copia del disco. Los discos nuevos se cifrarán.

* [Copia de discos administrados mediante la CLI de Azure](../articles/virtual-machines/scripts/virtual-machines-linux-cli-sample-copy-managed-disks-to-same-or-different-subscription.md?toc=%2fcli%2fmodule%2ftoc.json)
* [Copia de discos administrados mediante PowerShell](../articles/virtual-machines/scripts/virtual-machines-windows-powershell-sample-copy-managed-disks-to-same-or-different-subscription.md?toc=%2fcli%2fmodule%2ftoc.json)

**¿Están cifradas las imágenes e instantáneas administradas?**

Sí. Todas las instantáneas e imágenes administradas creadas después del 9 de junio de 2017 se cifran automáticamente. 

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

**¿Cuál es el mayor tamaño de disco administrado compatible con discos de datos y sistema operativo?**

El tipo de partición compatible con Azure para un disco del sistema operativo es el registro de arranque maestro (MBR). El formato MBR admite un tamaño de disco de hasta 2 TiB. El tamaño máximo que admite Azure para un disco del sistema operativo es de 2 TiB. Azure admite hasta 32 TB para discos de datos administrados en Azure global, 4 TiB en nubes soberanas de Azure.

**¿Cuál es el mayor tamaño de disco administrado compatible con discos de datos y sistema operativo?**

El tipo de partición compatible con Azure para un disco del sistema operativo es el registro de arranque maestro (MBR). El formato MBR admite un tamaño de disco de hasta 2 TiB. El tamaño máximo que admite Azure para un disco no administrado del sistema operativo es de 2 TiB. Azure admite hasta 4 TiB para discos de datos no administrados.

**¿Cuál es el mayor tamaño de blob en páginas admitido?**

El mayor tamaño de blob en páginas que admite Azure es 8 TiB (8191 GiB). El tamaño máximo del blob en páginas cuando se conecta a una máquina virtual como discos de datos o de sistema operativo es de 4 TiB (4095 GiB).

**¿Es necesario usar una nueva versión de las herramientas de Azure para crear, conectar, cambiar de tamaño y cargar discos de más de 1 TiB?**

No es necesario actualizar las herramientas de Azure existentes para crear, conectar o cambiar el tamaño de los discos de más de 1 TiB. Para cargar el archivo de disco duro virtual del entorno local directamente en Azure como un blob en páginas o un disco no administrado, debe usar los conjuntos de herramientas más recientes que se indican a continuación. Solo se admiten cargas de disco duro virtual de hasta 8 TiB.

|Herramientas de Azure      | Versiones compatibles                                |
|-----------------|---------------------------------------------------|
|Azure PowerShell | Número de versión 4.1.0: Versión de junio de 2017 o posterior|
|CLI de Azure v1     | Número de versión 0.10.13: Versión de mayo de 2017 o posterior|
|CLI de Azure v2     | Número de versión 2.0.12: Versión de julio de 2017 o posterior|
|AzCopy           | Número de versión 6.1.0: Versión de junio de 2017 o posterior|

**¿Se admiten los tamaños de disco P4 y P6 para blobs en páginas o discos no administrados?**

Los tamaños de disco P4 (32 GiB) y P6 (64 GiB) no se admiten como niveles de disco predeterminados para discos no administrados y blobs en páginas. Deberá [establecer el nivel de blob](https://docs.microsoft.com/rest/api/storageservices/set-blob-tier) explícitamente en P4 y P6 para que el disco se asigne a estos niveles. Si implementa un disco no administrado o un blob en páginas con el tamaño de disco o la longitud de contenido inferior a 32 GiB o entre 32 y 64 GiB sin establecer el nivel de blob, continuará en el nivel P10 con 500 IOPS y 100 MiB/s y el plan de tarifa asignado.

**Si mi disco administrado premium existente de menos de 64 GiB se creó antes de habilitar el disco pequeño (alrededor del 15 de junio de 2017), ¿cómo se factura?**

Los discos pequeños premium de menos de 64 GiB se siguen facturando según el plan de tarifa P10.

**¿Cómo puedo cambiar el nivel de disco de los discos pequeños premium menores de 64 GiB de P10 a P4 o P6?**

Puede crear una instantánea de los discos pequeños y, después, crear un disco para cambiar automáticamente el plan de tarifa a P4 o P6, en función del tamaño aprovisionado.

**¿Puede cambiar el tamaño los discos administrados existentes de tamaños de menos de 4 tebibytes (TiB) a nuevos tamaños de disco recién introducidas hasta 32 TB?**

Sí.

**¿Cuáles son los tamaños de disco más grandes compatibles con el servicio de copia de seguridad de Azure y Azure Site Recovery?**

El tamaño de disco más grande que admiten Azure Backup y el servicio Azure Site Recovery es de 4 TiB. Pronto se agregará compatibilidad con discos de mayor tamaño de hasta 32 TB.

**¿Cuáles son la VM recomendada tamaños más grandes para tamaños de disco (> 4 TiB) para discos de Standard HDD y SSD estándar para lograr optimizado en disco IOPS y ancho de banda?**

Para lograr el rendimiento de disco estándar de SSD y HDD estándar tamaños de discos de gran tamaño (> 4 TiB) más allá de 500 IOPS y 60 MiB/s, se recomienda implementar una nueva máquina virtual de uno de los siguientes tamaños de máquina virtual para optimizar el rendimiento: B-series, DSv2-series, serie Dsv3, ESv3-Series, serie Fs, serie Fsv2, serie M, serie GS, serie NCv2, serie NCv3 o máquinas virtuales de la serie Ls. Adjuntar discos de gran tamaño a las máquinas virtuales o máquinas virtuales que no utilizan los tamaños recomendados anteriores existente, puede experimentar un rendimiento inferior.

**¿Cómo puedo actualizar mis discos (> 4 TiB) que se implementaron durante la versión preliminar los tamaños de disco más grande con el fin de obtener el mayor ancho de banda e IOPS en disponibilidad general?**

Puede detener e inicie la máquina virtual que está conectado el disco, o bien, separar y volver a conectar el disco. Los objetivos de rendimiento de los tamaños de disco más grandes han aumentado para SSD premium y estándar SSD en disponibilidad general.

**¿Qué regiones son los tamaños de disco administrado de 8 TB, TiB 16 y 32 TiB admite?**

8 TiB TiB 16, 32 TiB disco las SKU y se admiten en todas las regiones en que 21Vianet global de Azure, Microsoft Azure Government y Azure China.

**¿Se admiten Habilitar almacenamiento en caché de Host en todos los tamaños de disco?**

Se admite solo Host de almacenamiento en caché de lectura y lectura/escritura en los tamaños de disco inferior a 4 TB. Para los tamaños de disco de más de 4 TiB, no se puede establecer la opción de almacenamiento en caché en un valor distinto de Ninguno. Se recomienda aprovechar el almacenamiento en caché para tamaños de disco más pequeños, donde se puede esperar un mayor aumento del rendimiento con datos almacenados en caché en la máquina virtual.

## <a name="what-if-my-question-isnt-answered-here"></a>Mi pregunta no está respondida aquí. ¿Qué debo hacer?

Si su pregunta no aparece aquí, háganoslo saber y lo ayudaremos a encontrar una respuesta. Puede publicar una pregunta al final de este artículo en los comentarios. Para ponerse en contacto con el equipo de Azure Storage y otros miembros de la Comunidad sobre este artículo, use el [foro de Azure Storage](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazuredata) de MSDN.

Para solicitar características, envíe sus solicitudes e ideas al [foro de comentarios de Azure Storage](https://feedback.azure.com/forums/217298-storage).
