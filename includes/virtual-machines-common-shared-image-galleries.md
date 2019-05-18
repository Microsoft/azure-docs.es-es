---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: axayjo
ms.service: virtual-machines
ms.topic: include
ms.date: 05/06/2019
ms.author: akjosh; cynthn
ms.custom: include file
ms.openlocfilehash: 4063e79a9415ac35b09cc77d0110c04e191b49c7
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546754"
---
La galería de imágenes compartidas es un servicio que ayuda a generar estructura y organización en torno a las imágenes de VM administradas y personalizadas. Galerías de imágenes compartidas proporcionan:

- Administrar la replicación global de imágenes.
- Control de versiones y la agrupación de imágenes para facilitar la administración.
- Asegúrese de las imágenes de alta disponibilidad con las cuentas de almacenamiento con redundancia de zona (ZRS) en las regiones que admiten zonas de disponibilidad. ZRS ofrece una mejor resistencia frente a errores de zona.
- Compartir entre suscripciones e incluso entre inquilinos, mediante RBAC.

Uso de una galería de imágenes compartidas para compartir imágenes con diferentes usuarios, entidades de servicio o grupos de AD dentro de su organización. Las imágenes compartidas se pueden replicar en varias regiones, para un escalado más rápido de las implementaciones.

Una imagen administrada es una copia de una VM completa (incluidos los discos de datos asociados) o, simplemente, el disco del sistema operativo, según cómo cree la imagen. Cuando crea una VM desde la imagen, la copia de los VHD de la imagen se usa para crear los discos para la nueva VM. La imagen administrada permanece en el almacenamiento y puede usarse una y otra vez para crear nuevas VM.

Si tiene un gran número de imágenes administradas que se deben mantener y le gustaría que estén disponibles en toda la empresa, puede usar una galería de imágenes compartidas como un repositorio que facilita compartir sus imágenes. 

La característica de galería de imágenes compartidas tiene varios tipos de recursos:

| Recurso | DESCRIPCIÓN|
|----------|------------|
| **Imagen administrada** | Una imagen básica que se puede utilizar por sí solo o utiliza para crear un **versión de la imagen** en una galería de imágenes. Las imágenes administradas se crean desde máquinas virtuales generalizadas. Una imagen administrada es un tipo de VHD especial que se puede usar para crear varias máquinas virtuales y que ahora se puede usar para crear versiones de imágenes compartidas. |
| **Galería de imágenes** | Al igual que Azure Marketplace, una **galería de imágenes** es un repositorio para administrar y compartir imágenes, pero usted puede controlar quién tiene acceso. |
| **Definición de la imagen** | Las imágenes se definen dentro de una galería y contienen información sobre la imagen y requisitos para usarlo dentro de su organización. Puede incluir información como si la imagen es Windows o Linux, los requisitos de memoria mínima y máxima y notas de la versión. Es una definición de un tipo de imagen. |
| **Versión de la imagen** | Una **versión de la imagen** es lo que se usa para crear una VM cuando se usa una galería. Puede tener varias versiones de una imagen según sea necesario para su entorno. Al igual que una imagen administrada, cuando se usa una **versión de la imagen** para crear una VM, la versión de la imagen se usa para crear nuevos discos para la VM. Las versiones de las imágenes pueden usarse varias veces. |

<br>


![Gráfico que muestra cómo puede tener varias versiones de una imagen en la galería](./media/shared-image-galleries/shared-image-gallery.png)

## <a name="image-definitions"></a>Definiciones de imagen

Las definiciones de la imagen son una agrupación lógica de las versiones de una imagen. La definición de la imagen contiene información sobre por qué se creó la imagen, qué sistema operativo es para e información sobre el uso de la imagen. Una definición de la imagen es similar a un plan para todos los detalles en torno a la creación de una imagen específica. No implemente una máquina virtual desde una definición de la imagen, pero desde la versión de imagen creado a partir de la definición.


Hay tres parámetros para cada definición de la imagen que se usan en combinación - **Publisher**, **ofrecen** y **SKU**. Estos se usan para buscar una definición de la imagen específica. Puede tener versiones de imágenes que comparten uno o dos valores, pero no los tres.  Por ejemplo, estas son las tres definiciones de imágenes y sus valores:

|Definición de imágenes|Publicador|Oferta|SKU|
|---|---|---|---|
|myImage1|Contoso|Finanzas|Back-end|
|myImage2|Contoso|Finanzas|Front-end|
|myImage3|Pruebas|Finanzas|Front-end|

Los tres tienen conjuntos de valores únicos. El formato es similar a cómo se pueden especificar actualmente publicador, oferta y SKU de [imágenes de Azure Marketplace](../articles/virtual-machines/windows/cli-ps-findimage.md) en Azure PowerShell para obtener la versión más reciente de una imagen de Marketplace. Cada definición de la imagen debe tener un único conjunto de estos valores.

Estos son otros parámetros que se pueden establecer en la definición de la imagen, por lo que puede controlar más fácilmente los recursos:

* Estado del sistema operativo: puede establecer el estado del sistema operativo en generalizada o especializada pero generalizado solo se admite actualmente. Se deben crear las imágenes de máquinas virtuales que se han generalizado mediante Sysprep para Windows o `waagent -deprovision` para Linux.
* Sistema operativo: puede ser Windows o Linux.
* Descripción: descripción de uso para proporcionar información más detallada sobre por qué existe la definición de la imagen. Por ejemplo, podría tener una definición de la imagen para el servidor front-end que tiene la aplicación preinstalada.
* CLUF - puede utilizarse para que apunte a un contrato de licencia del usuario final específico a la definición de la imagen.
* Notas de la declaración de privacidad y la versión: almacenar notas de la versión y declaraciones de privacidad en Azure storage y proporcione un URI para tener acceso a ellas como parte de la definición de la imagen.
* Final del ciclo de vida de fecha: asociar una fecha final del ciclo de vida a su definición de la imagen para poder utilizar la automatización para eliminar las definiciones de imagen antiguo.
* Etiqueta: puede agregar etiquetas al crear la definición de la imagen. Para obtener más información acerca de las etiquetas, consulte [mediante etiquetas para organizar los recursos](../articles/azure-resource-manager/resource-group-using-tags.md)
* Como mínimo y máximo de vCPU y recomendaciones de memoria: si la imagen tiene las recomendaciones de memoria y vCPU, puede adjuntar esa información a la definición de la imagen.
* No se permiten tipos de disco: puede proporcionar información acerca de las necesidades de almacenamiento para la máquina virtual. Por ejemplo, si la imagen no es adecuada para los discos estándar HDD, agréguelos a la lista de no permitir.


## <a name="regional-support"></a>Compatibilidad regional

Regiones de origen se muestran en la tabla siguiente. Todas las regiones públicas pueden ser regiones de destino, pero para replicar en el centro de Australia y Australia Central 2 debe tener su suscripción en la lista blanca. Para solicitar la inclusión en la lista blanca, visite: https://www.microsoft.com/en-au/central-regions-eligibility/


| Regiones de origen |
|---------------------|-----------------|------------------|-----------------|
| Centro de Australia   | EUAP de centro de EE. UU. | Corea Central    | Sur de Reino Unido 2      |
| Centro de Australia 2 | Asia oriental       | Corea del Sur      | Oeste de Reino Unido         |
| Este de Australia      | Este de EE. UU         | Centro-Norte de EE. UU | Centro occidental de EE.UU. |
| Sudeste de Australia | Este de EE. UU. 2       | Europa del Norte     | Europa occidental     |
| Sur de Brasil        | EUAP de este de EE. UU. 2  | Centro-Sur de EE. UU | Oeste de la India      |
| Centro de Canadá      | Centro de Francia  | Sur de la India      | Oeste de EE. UU.         |
| Este de Canadá         | Sur de Francia    | Sudeste asiático   | Oeste de EE. UU.         |
| India Central       | Este de Japón      | Norte de Reino Unido         | Oeste de EE. UU. 2       |
| Centro de EE. UU.          | Oeste de Japón      | Sur de Reino Unido 2         |                 |



## <a name="limits"></a>Límites 

Hay límites por suscripción, para implementar los recursos con galerías de imágenes compartidas:
- 100 galerías de imágenes compartidas, por suscripción por región
- 1000 definiciones de imagen, por suscripción por región
- 10 000 versiones de imágenes, por suscripción por región

Para obtener más información, consulte [comprobar el uso de recursos con límites de](https://docs.microsoft.com/azure/networking/check-usage-against-limits) para obtener ejemplos sobre cómo comprobar el uso actual.
 

## <a name="scaling"></a>Escalado
La galería de imágenes compartidas le permite especificar el número de réplicas de las imágenes que quiere que Azure mantenga. Esto ayuda en los escenarios de implementación de varias VM, ya que las implementaciones de VM se pueden distribuir a las distintas réplicas, lo que reduce la probabilidad de que el proceso de creación de instancias quede limitado por la sobrecarga de una única réplica.

![Gráfico que muestra cómo puede escalar imágenes](./media/shared-image-galleries/scaling.png)


## <a name="replication"></a>Replicación
La galería de imágenes compartidas también permite replicar las imágenes en otras regiones de Azure automáticamente. Cada versión de una imagen compartida se puede replicar en distintas regiones en función de lo que sea más conveniente para su organización. Un ejemplo es replicar siempre la imagen más reciente en varias regiones, mientras que todas las versiones anteriores solo están disponibles en una región. Esto puede ayudar a ahorrar costos de almacenamiento para las versiones de imágenes compartidas. 

Las regiones donde puede replicarse la versión de una imagen compartida se pueden actualizar después de su creación. El tiempo necesario para replicar en diferentes regiones depende de la cantidad de datos que se copien y del número de regiones en las que se replicará la versión. En algunos casos, esto puede tardar varias horas. Mientras se produce la replicación, puede ver el estado de replicación por región. Una vez completada la replicación de la imagen en una región, a continuación, puede implementar una máquina virtual o un conjunto de escalado con esa versión de la imagen en la región.

![Gráfico que muestra cómo puede replicar imágenes](./media/shared-image-galleries/replication.png)


## <a name="access"></a>Acceso

Dado que la galería de imágenes compartidas, la imagen compartida y la versión de la imagen compartida son recursos, pueden compartirse con los controles integrados nativos de Azure RBAC. Uso de RBAC puede compartir estos recursos para otros usuarios, las entidades de servicio y grupos. Incluso puede compartir el acceso a personas fuera de los inquilinos que se crearon dentro. Una vez que un usuario tiene acceso a la versión de imagen compartido, puede implementar una máquina virtual o un conjunto de escalado de máquinas virtuales.  Aquí está la matriz de uso compartido que ayuda a entender a lo que el usuario tiene acceso:

| Compartido con el usuario     | Galería de imágenes compartidas | Imagen compartida | Versión de la imagen compartida |
|----------------------|----------------------|--------------|----------------------|
| Galería de imágenes compartidas | Sí                  | Sí          | Sí                  |
| Imagen compartida         | No                   | Sí          | Sí                  |
| Versión de la imagen compartida | No                   | No           | Sí                  |

Se recomienda el uso compartido en el nivel de la Galería para una mejor experiencia. Para obtener más información sobre RBAC, consulte [administrar el acceso a recursos de Azure mediante RBAC](../articles/role-based-access-control/role-assignments-portal.md).

Las imágenes también se pueden compartir, a escala, entre los inquilinos mediante un registro de aplicaciones de varios inquilinos. Para obtener más información acerca de cómo compartir imágenes entre los inquilinos, consulte [compartir imágenes de máquina virtual de la galería en inquilinos de Azure](../articles/virtual-machines/linux/share-images-across-tenants.md).

## <a name="billing"></a>Facturación
No hay ningún cargo adicional por usar el servicio de la galería de imágenes compartidas. Se le cobrará por los siguientes recursos:
- Costos de almacenamiento de las versiones de imágenes compartidas. Costo depende del número de réplicas de la versión de la imagen y el número de regiones de en que la versión se replica. Por ejemplo, si tiene 2 imágenes y ambos se replican en 3 regiones, a continuación, cambiará 6 discos administrados en función de su tamaño. Para más información, consulte [Precios de Managed Disks](https://azure.microsoft.com/pricing/details/managed-disks/).
- Cargos de salida de red para la replicación de la primera versión de imagen de la región de origen a las regiones replicadas. Las réplicas subsiguientes se controlan dentro de la región, por lo que no hay ningún cargo adicional. 

## <a name="updating-resources"></a>Actualizando los recursos

Una vez creado, puede realizar algunos cambios en los recursos de galería de imágenes. Estos se limitan a:
 
Galería de imágenes compartidas:
- DESCRIPCIÓN

Definición de la imagen:
- vCPU recomendadas:
- Memoria recomendada
- DESCRIPCIÓN
- Fecha final del ciclo de vida

Versión de la imagen:
- Recuento de réplicas regionales
- Regiones de destino
- Exclusión de la versión más reciente
- Fecha final del ciclo de vida


## <a name="sdk-support"></a>Compatibilidad con SDK

Los siguientes SDK admiten la creación de galerías de imágenes compartidas:

- [.NET](https://docs.microsoft.com/dotnet/api/overview/azure/virtualmachines/management?view=azure-dotnet)
- [Java](https://docs.microsoft.com/java/azure/?view=azure-java-stable)
- [Node.js](https://docs.microsoft.com/javascript/api/azure-arm-compute/?view=azure-node-latest)
- [Python](https://docs.microsoft.com/python/api/overview/azure/virtualmachines?view=azure-python)
- [Go](https://docs.microsoft.com/go/azure/)

## <a name="templates"></a>Plantillas

Puede crear recursos de galería de imágenes compartidas con plantillas. Hay varias plantillas de Inicio rápido de Azure disponibles: 

- [Creación de una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [Creación de una definición de imagen en una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [Creación de una versión de imagen en una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)
- [Creación de una máquina virtual a partir de la versión de la imagen](https://azure.microsoft.com/resources/templates/101-vm-from-sig/)

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes 

**P.** ¿Cómo puedo mostrar todos los recursos de la galería de imágenes compartidas en las suscripciones? 
 
 A. Para enumerar todos los recursos de la galería de imágenes compartidas a través de las suscripciones a las que tiene acceso en Azure Portal, siga estos pasos:

1. Abra [Azure Portal](https://portal.azure.com).
1. Vaya a **Todos los recursos**.
1. Seleccione todas las suscripciones en las que quiera enumerar todos los recursos.
1. Busque los recursos de tipo **Galería privada**.
 
   Para ver las definiciones de imagen y las versiones de imagen, debe seleccionar también **Mostrar tipos ocultos**.
 
   Para enumerar todos los recursos de la galería de imágenes compartidas mediante las suscripciones para las que tiene permiso, use el comando siguiente en la CLI de Azure:

   ```bash
   az account list -otsv --query "[].id" | xargs -n 1 az sig list --subscription
   ```


**P.** ¿Puedo mover una imagen existente a la galería de imágenes compartidas?
 
 A. Sí. Hay tres escenarios basados en los tipos de imagen que pueda tener.

 Escenario 1: Si tiene una imagen administrada, puede crear una definición de la imagen y versión de la imagen a partir de ella.

 Escenario 2: Si tiene una imagen generalizada no administrada, puede crear una imagen administrada a partir de ella y, luego, crear una definición de la imagen y una versión de la imagen a partir de ella. 

 Escenario 3: Si tiene un VHD en el sistema de archivos locales, deberá cargar el VHD, crear una imagen administrada y, luego, crear una definición de la imagen y una versión de la imagen a partir de ella.
- Si el VHD es de una VM Windows, consulte [Carga de un VHD generalizado](https://docs.microsoft.com/azure/virtual-machines/windows/upload-generalized-managed).
- Si el VHD es para una VM Linux, consulte [Cargar un disco duro virtual](https://docs.microsoft.com/azure/virtual-machines/linux/upload-vhd#option-1-upload-a-vhd).


**P.** ¿Puedo crear una versión de la imagen desde un disco especializado?

 A. No, actualmente no admitimos discos especializados como imágenes. Si tiene un disco especializado, deberá [crear una VM a partir del VHD](https://docs.microsoft.com/azure/virtual-machines/windows/create-vm-specialized-portal#create-a-vm-from-a-disk) al asociar el disco especializado a una nueva VM. Una vez que tenga una máquina virtual en ejecución, deberá seguir las instrucciones para crear una imagen administrada desde la [máquina virtual Windows](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-custom-images) o la [máquina virtual Linux](https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-custom-images). Cuando tenga una imagen administrada generalizada, puede iniciar el proceso para crear una descripción de imagen compartida y la versión de la imagen.

 
**P.** Una vez que se cree, ¿puedo mover el recurso de la galería de imágenes compartidas a otra suscripción?

 A. No, no puede mover el recurso de la galería de imágenes compartidas a otra suscripción. Sin embargo, podrá replicar las versiones de la imagen en la galería a otras regiones según sea necesario.

**P.** ¿Puedo replicar las versiones de mis imágenes en las nubes: Azure China 21Vianet, Azure Alemania y la nube de Azure Government? 

 A. No, no puede replicar las versiones de la imagen en las nubes.

**P.** ¿Puedo replicar las versiones de mis imágenes entre suscripciones? 

 A. No, puede replicar las versiones de la imagen en varias regiones de una suscripción y usarla en otras suscripciones a través de RBAC.

**P.** ¿Puedo compartir versiones de imágenes entre los inquilinos de Azure AD? 

 A. Sí, puede utilizar RBAC para compartir a las personas a través de los inquilinos. Pero, para compartir a escala, vea "recurso compartido de la Galería de imágenes a través de los inquilinos de Azure" utilizar [PowerShell](../articles/virtual-machines/windows/share-images-across-tenants.md) o [CLI](../articles/virtual-machines/linux/share-images-across-tenants.md).


**P.** ¿Cuánto tarda la replicación de versiones de imágenes en las regiones de destino?

 A. El tiempo de replicación de la versión de una imagen depende totalmente del tamaño de la imagen y del número de regiones en las que se va a replicar. Sin embargo, como práctica recomendada para obtener los mejores resultados, se recomienda mantener una imagen pequeña y las regiones de origen y de destino cerca. Puede comprobar el estado de la replicación mediante la marca -ReplicationStatus.


**P.** ¿Cuál es la diferencia entre la región de origen y la región de destino?

 A. La región de origen es la región en la que se creará la versión de la imagen, y las regiones de destino son las regiones donde se almacenará una copia de la versión de la imagen. Para cada versión de una imagen, solo puede tener una región de origen. Además, asegúrese de pasar la ubicación de la región de origen como una de las regiones de destino cuando cree una versión de la imagen.  


**P.** ¿Cómo puedo especificar la región de origen al crear la versión de la imagen?

 A. Al crear una versión de la imagen, puede usar la etiqueta **--location** en la CLI y la etiqueta **-Location** en PowerShell para especificar la región de origen. Asegúrese de que la imagen administrada que usa como imagen de base para crear la versión de la imagen esté en la misma ubicación que la ubicación en la que va a crear la versión de la imagen. Además, asegúrese de pasar la ubicación de la región de origen como una de las regiones de destino cuando cree una versión de la imagen.  


**P.** ¿Cómo puedo especificar el número de réplicas de la versión de la imagen que se creará en cada región?

 A. Hay dos maneras de especificar el número de réplicas de la versión de la imagen que se creará en cada región:
 
1. El recuento de réplicas regionales que especifica el número de réplicas que desea crear para cada región. 
2. El recuento de réplicas comunes que sea el recuento predeterminado por región en caso de que no se especifique el recuento de réplicas regionales. 

Para especificar el número de réplicas regionales, pase la ubicación, así como el número de réplicas que desea crear en dicha región: "Centro-sur de EE. UU. = 2". 

Si el recuento de réplicas regionales no se especifica con cada ubicación, el número predeterminado de réplicas será el recuento de réplicas comunes que especificó. 

Para especificar el recuento de réplicas comunes en la CLI, use el argumento **--replica-count** en el comando `az sig image-version create`.


**P.** ¿Puedo crear la galería de imágenes compartidas en una ubicación diferente de en la que quiero crear la definición de la imagen y la versión de la imagen?

 A. Sí, es posible. Sin embargo, como práctica recomendada, le animamos a mantener el grupo de recursos, Galería de imágenes compartidas, definición de la imagen y versión de la imagen en la misma ubicación.


**P.** ¿Cuáles son los cargos por uso de la galería de imágenes compartidas?

 A. No hay ningún cargo por usar el servicio de la galería de imágenes compartidas, excepto los cargos de almacenamiento de las versiones de las imágenes y los cargos de salida de red para la replicación de las versiones de las imágenes de la región de origen a las regiones de destino.

**P.** ¿Qué versión de API debo usar para crear la galería de imágenes compartidas, la definición de la imagen, la versión de la imagen y VM/VMSS a partir de la versión de la imagen?

 A. Para implementaciones de VM y conjuntos de escalado de máquinas virtuales que usan una versión de la imagen, se recomienda usar la API versión 2018-04-01 o posterior. Para trabajar con galerías de imágenes compartidas, definiciones de imágenes y versiones de imágenes, se recomienda usar la API versión 2018-06-01. Almacenamiento con redundancia de zona (ZRS) requiere la versión de 2019-03-01 o posterior.
