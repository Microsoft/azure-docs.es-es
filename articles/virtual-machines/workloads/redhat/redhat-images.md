---
title: Información general sobre las imágenes de Red Hat Enterprise Linux en Azure
description: Aprenda sobre las imágenes de Red Hat Enterprise Linux en Microsoft Azure.
services: virtual-machines-linux
documentationcenter: ''
author: asinn826
manager: BorisB2015
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 02/10/2020
ms.author: alsin
ms.openlocfilehash: 9aa0ca41f63da94e2dedaffe65ea518b8adff0dc
ms.sourcegitcommit: 14bf4129a73de2b51a575c3a0a7a3b9c86387b2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87439482"
---
# <a name="overview-of-red-hat-enterprise-linux-images"></a>Introducción a las imágenes de Red Hat Enterprise Linux

En este artículo, se describen las imágenes disponibles de Red Hat Enterprise Linux (RHEL) en Azure Marketplace, junto con las directivas en relación con su nomenclatura y retención.

Puede encontrar información sobre las directivas de compatibilidad de Red Hat para todas las versiones de RHEL en [Ciclo de vida de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata). Para información detallada sobre los precios, consulte la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/details/virtual-machines/linux/).

>[!IMPORTANT]
> Las imágenes de RHEL actualmente disponibles en Azure Marketplace admiten los modelos de licencia "traiga su propia suscripción" (BYOS) o pago por uso. No se admiten [Ventaja para uso híbrido de Azure](../../windows/hybrid-use-benefit-licensing.md) ni el cambio dinámico entre la licencia de BYOS y de pago por uso. Para cambiar el modo de licencia, es necesario volver a implementar la máquina virtual desde la imagen correspondiente.

>[!NOTE]
> Si tiene algún problema relacionado con las imágenes de RHEL de Azure Marketplace, presente una incidencia de soporte técnico a Microsoft.

## <a name="view-images-available-in-azure"></a>Visualización de imágenes disponibles en Azure

Al buscar "Red Hat" en Azure Marketplace o al crear un recurso en la interfaz de usuario de Azure Portal, verá solamente un subconjunto de todas imágenes de RHEL disponibles. Siempre puede obtener el conjunto completo de imágenes de máquina virtual disponibles mediante PowerShell, la API o la CLI de Azure.

Para ver el conjunto completo de imágenes de Red Hat disponibles en Azure, ejecute este comando:

```azurecli-interactive
az vm image list --publisher RedHat --all
```

### <a name="naming-convention"></a>Convención de nomenclatura

Las imágenes de máquina virtual en Azure se organizan por editor, oferta, SKU y versión. La combinación de editor:oferta:SKU:versión es el URN de la imagen, que identifica de forma única la imagen que se usará.

Por ejemplo, `RedHat:RHEL:8-LVM:8.1.20200318` hace referencia a una imagen con partición LVM de RHEL 8.1 compilada el 18 de marzo de 2020.

A continuación, se muestra un ejemplo de cómo crear una máquina virtual de RHEL 8.1.

```azurecli-interactive
az vm create --name RhelVM --resource-group TestRG --image RedHat:RHEL:8-LVM:8.1.20200318 --no-wait
```

### <a name="the-latest-moniker"></a>El moniker “latest” (último)

La API REST de Azure permite el uso del moniker “latest” (último) para la versión, en lugar de la versión específica. Al usar “latest”, se aprovisionará la última imagen disponible para el editor, la oferta o la SKU específicos.

Por ejemplo, `RedHat:RHEL:8-LVM:latest` hace referencia a la imagen con partición LVM más reciente de la familia de RHEL 8 disponible.

```azurecli-interactive
az vm create --name RhelVM --resource-group TestRG --image RedHat:RHEL:8-LVM:latest --no-wait
```

>[!NOTE]
> En general, la comparación de versiones para determinar la más reciente sigue las reglas del [método CompareTo](/dotnet/api/system.version.compareto?view=netcore-3.1#system_version_compareto_system_version_).
Esta comparación de versiones de imagen se realiza comparando los valores como un objeto [Version](/dotnet/api/system.version.-ctor?view=netframework-4.8), no como una cadena.

## <a name="rhel-6-image-types"></a>Tipos de imagen de RHEL 6

En el caso de imágenes de RHEL 6.x, los tipos de imagen se muestran en la siguiente tabla:

|Publicador | Oferta | Valor de SKU | Versión | Detalles
|----------|-------|-----------|---------|--------
|RedHat | RHEL | Versión secundaria (por ejemplo, 6.9) | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 6.9.2018010506) | Todas las imágenes estándar de RHEL 6.x siguen esta convención.
|RedHat | rhel-byos | rhel-raw69 | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 6.9.20181023) | Se trata de una imagen BYOS de RHEL 6.9.
|RedHat | RHEL | RHEL-SAP-APPS | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 6.8.2017053118) | Se trata de una imagen de RHEL 6.8 para SAP Applications. Tiene derechos de acceso a los repositorios de SAP Applications, así como a los repositorios de RHEL base.
|RedHat | RHEL | RHEL-SAP-HANA | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 6.7.2017053121) | Se trata de una imagen de RHEL 6.7 para SAP HANA. Tiene derechos de acceso a los repositorios de SAP HANA, así como a los repositorios de RHEL base.

## <a name="rhel-7-image-types"></a>Tipos de imagen de RHEL 7

En el caso de las imágenes de RHEL 7.x, hay algunos tipos de imagen diferentes. En la tabla siguiente se muestran los diferentes conjuntos de imágenes que ofrecemos. Para ver una lista completa, use el comando `az vm image list --publisher redhat --all` de la CLI de Azure.

>[!NOTE]
> A menos que se indique lo contrario, todas las imágenes tienen particiones LVM y se conectarán a los repositorios de RHEL convencionales. Es decir, los repositorios no son compatibles con la actualización extendida (EUS) y no son servicios de actualización para SAP (E4S). A partir de ahora, solamente vamos a publicar imágenes con particiones LVM, pero abiertas a comentarios sobre esta decisión. Para más información sobre la Compatibilidad con la actualización extendida y los Servicios de actualización para SAP, consulte [Ciclo de vida de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata).

|Publicador | Oferta | Valor de SKU | Versión | Detalles
|----------|-------|------------|---------|--------
|RedHat | RHEL | Versión secundaria (por ejemplo, 7.6) | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 7.6.2019102813) | Las imágenes publicadas antes de abril de 2019 están conectadas a los repositorios de RHEL estándar. Las imágenes publicadas después de abril de 2019 están conectadas a los repositorios de EUS de Red Hat para permitir el bloqueo de una versión secundaria concreta. Los clientes que quieran repositorios convencionales deben usar las imágenes que contienen 7-LVM o 7-RAW en el valor de SKU (a continuación figuran los detalles). Las imágenes de RHEL 7.7 y versiones posteriores tendrán particiones LVM. Todas las demás imágenes de esta categoría tienen particiones sin formato.
|RedHat | RHEL | 7-RAW | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 7.6.2019102813) | Estas imágenes tienen particiones sin formato (por ejemplo, no se han agregado volúmenes lógicos).
|RedHat | RHEL | 7-RAW-CI | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 7.6.2019072418) | Estas imágenes tienen particiones sin formato (por ejemplo, no se han agregado volúmenes lógicos) y usan cloud-init para el aprovisionamiento.
|RedHat | RHEL | 7-LVM | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 7.6.2019062414) | Estas imágenes tienen particiones LVM.
|RedHat | rhel-byos | rhel-{lvm,raw} | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 7.7.20190819) | Estas imágenes son las imágenes BYOS de RHEL 7. No están conectadas a ningún repositorio y no cobrarán el precio de RHEL premium. Si está interesado en las imágenes BYOS de RHEL, [solicite acceso](https://aka.ms/rhel-byos). Los valores de SKU finalizan con la versión secundaria e indican si la imagen no tiene formato o tiene particiones LVM. Por ejemplo, un valor de SKU de rhel-lvm77 indica una imagen de RHEL 7.7 con particiones LVM.
|RedHat | RHEL | RHEL-SAP | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 7.6.2019071300) | Estas imágenes son imágenes de RHEL para SAP. Tienen derechos de acceso a los repositorios de SAP HANA y SAP Applications, así como a los repositorios de RHEL E4S. La facturación incluye RHEL prémium y SAP prémium por encima de la tarifa de proceso base.
|RedHat | RHEL | RHEL-SAP-HA | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 7.6.2019062320) | Estas imágenes son RHEL for SAP con imágenes de Alta disponibilidad y de Servicios de actualización. Tienen derechos de acceso a los repositorios de SAP HANA y SAP Applications y a los repositorios de alta disponibilidad, así como a los repositorios de RHEL E4S. La facturación incluye RHEL premium, SAP premium y alta disponibilidad premium por encima de la tarifa de proceso base.
|RedHat | RHEL | RHEL-HA | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 7.6.2019062019) | Se trata de imágenes de RHEL que también tienen derechos de acceso al complemento de alta disponibilidad. Se cobran ligeramente por encima de la tarifa de proceso base y RHEL debido al complemento de alta disponibilidad premium.
|RedHat | RHEL | RHEL-SAP-APPS | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 7.3.2017053118) | Estas imágenes no están actualizadas, ya que los repositorios de SAP Applications y SAP HANA se han combinado en los repositorios de SAP. Se trata de imágenes de RHEL para SAP Applications. Tienen derechos de acceso a los repositorios de SAP Applications, así como a los repositorios de RHEL base.
|RedHat | RHEL | RHEL-SAP-HANA | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 7.3.2018051421) | Estas imágenes no están actualizadas, ya que los repositorios de SAP Applications y SAP HANA se han combinado en los repositorios de SAP. Estas imágenes son imágenes de RHEL for SAP HANA. Tienen derechos de acceso a los repositorios de SAP HANA, así como a los repositorios de RHEL base.

## <a name="rhel-8-image-types"></a>Tipos de imagen de RHEL 8

>[!NOTE]
> Red Hat recomienda el uso de Grubby para configurar parámetros de línea de comandos de kernel en RHEL 8+. [Aquí](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/managing_monitoring_and_updating_the_kernel/configuring-kernel-command-line-parameters_managing-monitoring-and-updating-the-kernel) encontrará más detalles al respecto.

A continuación se muestran los detalles de los tipos de imagen RHEL 8.

|Publicador | Oferta | Valor de SKU | Versión | Detalles
|----------|-------|------------|---------|--------
|RedHat | RHEL | 8 | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 8.0.20191023) | Estas imágenes tienen particiones LVM de RHEL 8 conectadas a repositorios estándar de Red Hat.
|RedHat | RHEL | 8-gen2 | Valores concatenados de la versión secundaria de RHEL y la fecha de publicación (por ejemplo, 8.0.20191024) | Estas imágenes tienen particiones LVM de RHEL 8 de Hyper-V generación 2 conectadas a repositorios estándar de Red Hat. Para más información sobre las máquinas virtuales de Generación 2, consulte [Compatibilidad con máquinas virtuales de generación 2 en Azure](../../linux/generation-2.md).

## <a name="rhel-longer-support-add-ons"></a>Complementos de compatibilidad más larga de RHEL

### <a name="extended-update-support"></a>Compatibilidad con la actualización extendida

A partir de abril de 2019, están disponibles imágenes de RHEL que están conectadas a los repositorios de EUS de forma predeterminada. Puede encontrar más información sobre RHEL EUS en la [documentación de Red Hat](https://access.redhat.com/articles/rhel-eus).

El cambio a los repositorios de EUS es posible y se admite. Para obtener instrucciones sobre cómo cambiar la máquina virtual a EUS y más información sobre las fechas de finalización del ciclo de vida de la compatibilidad con EUS, consulte [Bloqueo de versiones en máquinas virtuales de RHEL y RHEL EUS](https://aka.ms/rhui-update#rhel-eus-and-version-locking-rhel-vms).

>[!NOTE]
> EUS no es compatible con RHEL Extras. Esto significa que, si instala un paquete que está disponible normalmente en el canal de RHEL Extras, no podrá hacerlo mientras se encuentre en EUS. Para más información sobre el ciclo de vida de los productos de Red Hat Extras, consulte [Ciclo de vida de Red Hat Enterprise Linux Extras](https://access.redhat.com/support/policy/updates/extras/).

#### <a name="differentiate-between-regular-and-eus-images"></a>Diferenciación entre imágenes EUS y convencionales

Los clientes que quieran usar las imágenes que están conectadas a los repositorios de EUS deben usar la imagen de RHEL que contiene un número de versión secundaria de RHEL en la SKU.

Por ejemplo, puede ver que las dos imágenes de RHEL 7.4 siguientes están disponibles.

```bash
RedHat:RHEL:7-LVM:7.6.2019062414
RedHat:RHEL:7.6:7.6.2019102813
```

En este caso, `RedHat:RHEL:7.6:7.6.2019102813` está conectada a los repositorios de EUS de forma predeterminada. El valor de la SKU es 7.4. Y `RedHat:RHEL:7-LVM:7.6.2019062414` está conectada a repositorios no EUS de forma predeterminada. El valor de la SKU es 7-LVM.

Para usar repositorios convencionales (que no sean EUS), utilice una imagen que no contenga un número de versión secundaria en la SKU.

#### <a name="rhel-images-with-eus"></a>Imágenes de RHEL con EUS

La información de la tabla siguiente se aplica a imágenes de RHEL que están conectadas a repositorios de EUS.

>[!NOTE]
> En el momento de escribir este artículo, solo RHEL 7.4 y versiones posteriores secundarias tienen compatibilidad con EUS. Esta característica ya no se admite con la versión 7.3, o versiones anteriores, de RHEL.
>
> Para más información sobre la disponibilidad de RHEL EUS, consulte [Ciclo de vida de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata).

Versión secundaria |Ejemplo de imagen de EUS              |Estado de EUS                                                   |
:-------------|:------------------------------|:------------------------------------------------------------|
RHEL 7.4      |RedHat:RHEL:7.4:7.4.2019041718 | Las imágenes publicadas en abril de 2019 y más adelante serán de EUS de forma predeterminada.|
RHEL 7.5      |RedHat:RHEL:7.5:7.5.2019060305 | Las imágenes publicadas en junio de 2019 y más adelante serán de EUS de forma predeterminada. |
RHEL 7.6      |RedHat:RHEL:7.6:7.6.2019052206 | Las imágenes publicadas en mayo de 2019 y más adelante serán de EUS de forma predeterminada. |
RHEL 8.0      |N/D                            | No hay EUS disponible en Red Hat.                               |

### <a name="update-services-for-sap"></a>Servicios de actualización para SAP

Las imágenes más recientes de RHEL for SAP se conectarán a los servicios de actualización para las suscripciones de soluciones SAP (E4S). Para más información sobre E45, consulte la [documentación](https://access.redhat.com/support/policy/updates/errata#Update_Services_for_SAP_Solutions) de Red Hat.

#### <a name="rhel-images-with-e4s"></a>Imágenes de RHEL con E4S

Las imágenes de las siguientes ofertas creadas después de diciembre de 2019 están conectadas a los repositorios de E4S.

* RHEL-SAP (RHEL for SAP)
* RHEL-SAP-HA (RHEL for SAP con alta disponibilidad y servicios de actualización).

## <a name="other-available-offers-and-skus"></a>Otras ofertas y SKU disponibles

La lista completa de ofertas y SKU disponibles puede incluir imágenes adicionales aparte de las que se indican en la tabla anterior. Un ejemplo es `RedHat:rhel-ocp-marketplace:rhel74:7.4.1`. Estas ofertas se pueden usar para proporcionar compatibilidad con soluciones específicas de Marketplace. O bien, se pueden publicar para obtener una vista previa de ellas y realizar pruebas. En cualquier momento se pueden cambiar o quitar sin previo aviso. No las use, excepto si Microsoft o Red Hat ha documentado públicamente su presencia.

## <a name="publishing-policy"></a>Directiva de publicación

Microsoft y Red Hat actualizan las imágenes cuando se publican nuevas versiones secundarias, según sea necesario para solucionar vulnerabilidades y exposiciones comunes (CVE), o bien para actualizaciones o cambios de configuración ocasionales. Nos esforzamos por proporcionar imágenes actualizadas lo antes posible (en un plazo de tres días laborables después de la publicación o disponibilidad de una corrección de CVE).

Solo actualizamos la versión secundaria actual de una familia de imágenes especifica. Cuando se publique una versión secundaria más reciente, dejaremos de actualizar la versión secundaria anterior. Por ejemplo, con la publicación de RHEL 7.6, las imágenes RHEL 7.5 ya no se actualizan.

>[!NOTE]
> Las máquinas virtuales de Azure activas aprovisionadas a partir de imágenes de pago por uso de RHEL están conectadas a Azure RHUI y pueden recibir actualizaciones y correcciones en cuanto Red Hat las publique y las replique en Azure RHUI. El plazo suele ser inferior a 24 horas después del lanzamiento oficial de Red Hat. Estas máquinas virtuales no requieren una nueva imagen publicada para obtener las actualizaciones. Los clientes tienen control total sobre cuándo iniciar la actualización.

## <a name="image-retention-policy"></a>Directiva de retención de imágenes

Nuestra directiva actual es mantener todas las imágenes publicadas anteriormente. Nos reservamos el derecho de quitar las imágenes que se conoce que causan algún tipo de problema. Por ejemplo, puede que se quiten las imágenes con configuraciones incorrectas debido a actualizaciones posteriores de componentes o plataformas. Las imágenes que puedan quitarse seguirán la directiva actual de Azure Marketplace para proporcionar notificaciones hasta 30 días antes de su eliminación.

## <a name="next-steps"></a>Pasos siguientes

* Para ver la lista completa de imágenes de RHEL en Azure, consulte las [imágenes de Red Hat Enterprise Linux (RHEL) disponibles en Azure](./redhat-imagelist.md).
* Para más información sobre la infraestructura de actualización de Azure Red Hat, consulte [Infraestructura de actualización de Red Hat para máquinas virtuales de RHEL a petición en Azure](https://aka.ms/rhui-update).
* Para más información, consulte [Imágenes Gold de tipo Traiga su propia suscripción (BYOS) de Red Hat Enterprise Linux en Azure](./byos.md).
* Puede encontrar información sobre las directivas de soporte técnico de Red Hat para todas las versiones de RHEL en [Ciclo de vida de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata).
