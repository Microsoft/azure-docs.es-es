---
title: Habilitación de la replicación de máquinas virtuales de Azure cifradas en Azure Site Recovery
description: En este artículo se describe cómo configurar la replicación de máquinas virtuales con discos habilitados para claves administradas por el cliente (CMK) de una región de Azure a otra mediante Site Recovery.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 07/10/2020
ms.author: mayg
ms.openlocfilehash: 2dc7bbd060d77b8f90d16e2a336edc1eb6dbf651
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86529002"
---
# <a name="replicate-machines-with-customer-managed-keys-cmk-enabled-disks"></a>Replicación de máquinas con discos habilitados para claves administradas por el cliente (CMK)

En este artículo se describe cómo replicar máquinas virtuales de Azure con discos administrados habilitados para claves administradas por el cliente (CMK) de una región de Azure a otra.

## <a name="prerequisite"></a>Requisito previo
Debe crear los conjuntos de cifrado de disco en la región de destino para la suscripción de destino antes de habilitar la replicación para las máquinas virtuales que tienen discos administrados habilitados para CMK.

## <a name="enable-replication"></a>Habilitar replicación

En este ejemplo la región principal de Azure es Este de Asia y la secundaria es Sudeste de Asia.

1. En el almacén, seleccione **+ Replicar**.
2. Especifique los campos siguientes.
    - **Origen**: el punto de origen de las máquinas virtuales, que en este caso es **Azure**.
    - **Ubicación de origen**: la región de Azure donde quiere proteger las máquinas virtuales. En este ejemplo, la ubicación de origen es "Asia Oriental".
    - **Modelo de implementación**: el modelo de implementación de Azure de las máquinas de origen.
    - **Suscripción de origen**: suscripción a la que pertenecen sus máquinas virtuales de origen. Puede tratarse de cualquier suscripción que se encuentre en el mismo inquilino de Azure Active Directory donde esté el almacén de servicios de recuperación.
    - **Grupo de recursos**: el grupo de recursos al que pertenecen las máquinas virtuales de origen. Todas las máquinas virtuales del grupo de recursos seleccionado se enumeran para su protección en el paso siguiente.

3. En **Máquinas virtuales** > **Seleccionar máquinas virtuales**, seleccione las máquinas virtuales que quiera replicar. Solo puede seleccionar aquellas máquinas en las que se pueda habilitar la replicación. Después, seleccione **Aceptar**.

4. En **Configuración**, puede configurar las opciones siguientes de sitio de destino.

    - **Ubicación de destino**: ubicación donde se replicarán los datos de la máquina virtual de origen. Site Recovery proporciona una lista de regiones de destino adecuadas en función de la ubicación de la máquina seleccionada. Se recomienda usar la misma ubicación que la del almacén de Recovery Services.
    - **Suscripción de destino**: la suscripción de destino que se usa para la recuperación ante desastres. De forma predeterminada, la suscripción de destino será la misma que la de origen.
    - **Grupo de recursos de destino**: el grupo de recursos al que pertenecen todas las máquinas virtuales replicadas. De forma predeterminada, Site Recovery crea un nuevo grupo de recursos en la región de destino. El nombre toma el sufijo `asr`. Si un grupo de recursos que ha creado Azure Site Recovery ya existe, se vuelve a usar. También se puede elegir personalizarlo, tal como se muestra en la sección siguiente. La ubicación del grupo de recursos de destino puede ser cualquier región de Azure excepto la región en la que se hospedan las máquinas virtuales de origen.
    - **Red virtual de destino**: De manera predeterminada, Site Recovery crea una red virtual en la región de destino. El nombre toma el sufijo `asr`. Se asigna a la red de origen y se usa para todas las protecciones futuras. [más información](./azure-to-azure-network-mapping.md) sobre la asignación de red.
    - **Cuentas de almacenamiento de destino (si la máquina virtual de origen no usa discos administrados)** : Site Recovery crea, de forma predeterminada, una cuenta de almacenamiento de destino mediante la imitación de la configuración de almacenamiento de la máquina virtual de origen. Si ya existe una cuenta de almacenamiento, esta se vuelve a usar.
    - **Discos administrados de réplica (si la máquina virtual de origen utiliza discos administrados)** : Site Recovery crea discos administrados de réplica en la región de destino para reflejar los discos administrados de la máquina virtual de origen del mismo tipo de almacenamiento (Standard o Premium) que los discos administrados de la máquina virtual de origen.
    - **Cuentas de almacenamiento en caché**: Site Recovery necesita una cuenta de almacenamiento adicional llamada *almacenamiento en caché* en la región de origen. Todos los cambios que se realicen en las máquinas virtuales de origen se controlan y se envían a la cuenta de almacenamiento en caché. Luego se replican en la ubicación de destino.
    - **Conjunto de disponibilidad**: Site Recovery, de manera predeterminada, crea un conjunto de disponibilidad nuevo en la región de destino. El nombre tiene el sufijo `asr`. Si un conjunto de disponibilidad que ha creado Azure Site Recovery ya existe, se vuelve a usar.
    - **Conjuntos de cifrado de disco (DES)** : Site Recovery necesita que se usen los conjuntos de cifrado de disco para los discos administrados de réplica y destino. Antes de habilitar la replicación, debe crear previamente el DES en la suscripción de destino y en la región de destino. No hay DES seleccionado de forma predeterminada. Debe hacer clic en "Personalizar" para elegir un DES por disco de origen.
    - **Directiva de replicación**: define la configuración del historial de retención del punto de recuperación y la frecuencia de instantánea coherente con la aplicación. De manera predeterminada, Site Recovery crea una directiva de replicación nueva con la configuración predeterminada de *24 horas* para la retención del punto de recuperación y *60 minutos* para la frecuencia de instantánea coherente con la aplicación.

    ![Habilitación de la replicación para el equipo con discos habilitados para CMK](./media/azure-to-azure-how-to-enable-replication-cmk-disks/cmk-enable-dr.png)

## <a name="customize-target-resources"></a>Personalizar los recursos de destino

Siga estos pasos para modificar los valores de destino predeterminados de Site Recovery.

1. Seleccione **Personalizar**, junto a "Suscripción de destino", para modificar la suscripción de destino predeterminada. Seleccione la suscripción en la lista de suscripciones disponibles en el inquilino de Azure AD.

2. Seleccione **Personalizar**, junto a "Grupo de recursos, red, almacenamiento y conjuntos de disponibilidad", para modificar los siguientes valores predeterminados:
    - Para **Grupo de recursos de destino**, seleccione el grupo de recursos de la lista de grupos de recursos que existen en la ubicación de destino de la suscripción.
    - Para **Red virtual de destino**, seleccione la red en una lista de las redes virtuales en la ubicación de destino.
    - Para **Conjunto de disponibilidad**, puede agregar la configuración del conjunto de disponibilidad a la máquina virtual en caso de que forme parte de un conjunto de disponibilidad de la región de origen.
    - Para **Cuentas de almacenamiento de destino**, seleccione la cuenta que quiere usar.

3. Seleccione **Personalizar** junto a "Configuración de cifrado de almacenamiento" para seleccionar el DES de destino para cada disco administrado de origen habilitado para la clave administrada por el cliente (CMK). En el momento de la selección, también podrá ver a qué almacén de claves de destino está asociado el DES.

4. Seleccione **Crear recurso de destino** > **Habilitar replicación**.
5. Después de que las máquinas virtuales estén habilitadas para la replicación, puede comprobar el estado de mantenimiento de las máquinas virtuales en **Elementos replicados**.

![Habilitación de la replicación para el equipo con discos habilitados para CMK](./media/azure-to-azure-how-to-enable-replication-cmk-disks/cmk-customize-target-disk-properties.png)

>[!NOTE]
>Durante la replicación inicial, el estado de la máquina virtual puede tardar un tiempo en actualizarse, sin ningún progreso aparente. Haga clic en **Actualizar** para obtener el estado más reciente.

## <a name="faqs"></a>Preguntas más frecuentes

* He habilitado CMK en un elemento replicado existente. ¿Cómo puedo asegurarme de que CMK se aplique también en la región de destino?

    Puede averiguar el nombre del disco administrado de réplica (creado por Azure Site Recovery en la región de destino) y adjuntar DES a este disco de réplica. Sin embargo, no podrá ver los detalles de DES en la hoja Discos una vez que los haya adjuntado. Como alternativa, puede optar por deshabilitar la replicación de la máquina virtual y habilitarla de nuevo. Se asegurará de que vea los detalles de DES y del almacén de claves en la hoja Discos del elemento replicado.

* He agregado un nuevo disco habilitado para CMK al elemento replicado. ¿Cómo puedo replicar este disco con Azure Site Recovery?

    No se admite la adición de un nuevo disco habilitado para CMK a un elemento replicado existente. Deshabilite la replicación y vuelva a habilitarla para la máquina virtual.

* He habilitado las claves administradas por el cliente y la plataforma, ¿cómo puedo proteger mis discos?

    La habilitación del cifrado doble con claves administradas por el cliente y la plataforma se admite en Site Recovery. Siga las instrucciones de este artículo para proteger la máquina. Previamente, debe crear un DES con el cifrado doble habilitado en la región de destino. En el momento de habilitar la replicación para esta máquina virtual, puede proporcionar este DES a Site Recovery.

