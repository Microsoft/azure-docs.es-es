---
title: Configuración de claves administradas por el cliente en Azure Sentinel | Microsoft Docs
description: Aprenda a configurar claves administradas por el cliente (CMK) en Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/19/2020
ms.author: yelevin
ms.openlocfilehash: 44a2c0c1b4bc9b4e6fc1247497640b261d1d0fa1
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87387921"
---
# <a name="set-up-azure-sentinel-customer-managed-key"></a>Configuración de claves administradas por el cliente en Azure Sentinel


En este artículo se proporciona información general y pasos para configurar una clave administrada por el cliente (CMK) para Azure Sentinel. CMK permite que todos los datos guardados o enviados en Azure Sentinel se cifren en todos los recursos de almacenamiento pertinentes con una clave de Azure Key Vault creada por el usuario o de su propiedad.

> [!NOTE]
> -   La capacidad CMK de Azure Sentinel solo se proporciona a los clientes que son **nuevos**, y el acceso a ella se controla mediante el registro de características de Azure. Puede solicitar acceso si se pone en contacto con azuresentinelCMK@microsoft.com. A medida que la capacidad está disponible, se van aprobando las solicitudes pendientes.
> -   La capacidad CMK de Azure Sentinel solo está disponible en las regiones Este de EE. UU., Oeste de EE. UU. 2 y Centro y Sur de EE. UU.
> -   La capacidad CMK solo está disponible para los clientes que envían 1 TB al día o más. Se recibe información sobre los precios adicionales cuando se solicita a Microsoft el aprovisionamiento de CMK en la suscripción de Azure. Obtenga más información sobre los [precios de Log Analytics](../azure-monitor/platform/manage-cost-storage.md#log-analytics-dedicated-clusters).

## <a name="how-cmk-works"></a>Funcionamiento de CMK 

La solución Azure Sentinel usa varios recursos y características de almacenamiento para la recopilación de registros, como Log Analytics. Como parte de la configuración de CMK de Azure Sentinel, también tiene que configurar los valores de CMK en los recursos de almacenamiento relacionados. Los datos guardados en recursos de almacenamiento distintos de Log Analytics también se cifran.

Obtenga más información sobre [CMK](../azure-monitor/platform/customer-managed-keys.md#customer-managed-key-cmk-overview).

> [!NOTE]
> Si habilita CMK en Azure Sentinel, no se habilita ninguna característica de versión preliminar pública que no admita CMK.

## <a name="enable-cmk"></a>Habilitación de CMK 

Para aprovisionar CMK, siga estos pasos: 

1.  Cree una instancia de Azure Key Vault y una clave de almacenamiento.

2.  Habilite CMK en el área de trabajo de Log Analytics.

3.  Regístrese en Cosmos DB.

4.  Agregue una directiva de acceso a la instancia de Azure Key Vault.

5.  Habilite CMK en Azure Sentinel.

6.  Habilite Azure Sentinel.

### <a name="step-1-create-an-azure-key-vault-and-storing-key"></a>PASO 1: Creación de una instancia de Azure Key Vault y una clave de almacenamiento

1.  [Cree un recurso de Azure Key Vault](https://docs.microsoft.com/azure-stack/user/azure-stack-key-vault-manage-portal?view=azs-1910) y genere o importe una clave para usarla al cifrar los datos.
    > [!NOTE]
    >  La instancia de Azure Key Vault debe configurarse como recuperable para proteger la clave y el acceso.

1.  [Active las opciones de recuperación:](../key-vault/general/best-practices.md#turn-on-recovery-options)

    -   Asegúrese de que la [eliminación temporal](../key-vault/general/soft-delete-overview.md) esté activada.

    -   Active la [protección de purgas](../key-vault/general/soft-delete-overview.md#purge-protection) para protegerse frente a la eliminación forzada del secreto o el almacén incluso después de una eliminación temporal.

### <a name="step-2-enable-cmk-on-your-log-analytics-workspace"></a>PASO 2: Habilitación de CMK en el área de trabajo de Log Analytics

Siga las instrucciones de [Configuración de la clave administrada por el cliente de Azure Monitor](../azure-monitor/platform/customer-managed-keys.md) para crear un área de trabajo de CMK que se use como área de trabajo de Azure Sentinel en los pasos siguientes.

### <a name="step-3-register-for-cosmos-db"></a>PASO 3: Registro en Cosmos DB

Azure Sentinel funciona con Cosmos DB como recurso de almacenamiento adicional. Asegúrese de registrarse en Cosmos DB.

Siga la instrucción de Cosmos DB de [registrar el proveedor de recursos de Azure Cosmos DB](../cosmos-db/how-to-setup-cmk.md#register-resource-provider) para la suscripción de Azure.

### <a name="step-4-add-an-access-policy-to-your-azure-key-vault-instance"></a>PASO 4: Adición de una directiva de acceso a la instancia de Azure Key Vault

Asegúrese de agregar acceso desde Cosmos DB a la instancia de Azure Key Vault. Siga la instrucción de Cosmos DB de [agregar una directiva de acceso a la instancia de Azure Key Vault](../cosmos-db/how-to-setup-cmk.md#add-an-access-policy-to-your-azure-key-vault-instance) con la entidad de seguridad de Azure Cosmos DB.

### <a name="step-5-enable-cmk-in-azure-sentinel"></a>PASO 5: Habilitación de CMK en Azure Sentinel

La capacidad CMK de Azure Sentinel se proporciona a los nuevos clientes solo después de que hayan recibido acceso directamente desde el grupo de producto de Azure. Use sus contactos en Microsoft para recibir la aprobación del equipo de Azure Sentinel para habilitar CMK en la solución.

Después de obtener la aprobación, se le pide que proporcione la siguiente información para habilitar la característica CMK.

-  Identificador del área de trabajo donde quiere habilitar CMK.

-  Dirección URL de Key Vault: copie el "identificador de clave" de la clave hasta la última barra diagonal:  
    

    ![Identificador de clave](./media/customer-managed-keys/key-identifier.png)

    El equipo de Azure Sentinel habilita la característica CMK de Azure Sentinel para el área de trabajo proporcionada.

-  Comprobación por parte del equipo de producto de Azure Sentinel de que se ha aprobado el uso de esta característica. Debe contar con aprobación para continuar.

### <a name="step-6-enable-azure-sentinel"></a>PASO 6: Habilitación de Azure Sentinel


Vaya a Azure Portal y habilite Azure Sentinel en el área de trabajo en la que ha configurado CMK. Para más información, consulte [Incorporación de Azure Sentinel](quickstart-onboard.md).

## <a name="key-encryption-key-revocation-or-deletion"></a>Revocación o eliminación de la clave de cifrado de claves


En caso de que un usuario revoque la clave de cifrado de claves, ya sea mediante su eliminación o la eliminación del acceso a Azure Sentinel, en una hora, Azure Sentinel aplica el cambio y se comporta como si los datos ya no estuvieran disponibles. Desde ese momento se evita cualquier operación realizada que use recursos de almacenamiento persistente, como la ingesta de datos, los cambios de configuración persistentes y la creación de incidentes. Los datos almacenados previamente no se eliminan, pero permanecen inaccesibles. Los datos inaccesibles se rigen por la directiva de retención de datos y se purgan conforme a esa directiva.

La única operación posible una vez revocada o eliminada la clave de cifrado es la eliminación de la cuenta.

Si se restaura el acceso después de la revocación, Azure Sentinel restaura el acceso a los datos en una hora.

Para más información sobre el funcionamiento de este proceso en Azure Monitor, consulte [Revocación de CMK de Azure Monitor](../azure-monitor/platform/customer-managed-keys.md#cmk-kek-revocation).

## <a name="key-encryption-key-rotation"></a>Rotación de claves de cifrado de claves


Azure Sentinel y Log Analytics admiten la rotación de claves. Cuando un usuario realiza una rotación de claves en Key Vault, Azure Sentinel admite la nueva clave en una hora.

En Key Vault, puede realizar la rotación de claves mediante la creación de una nueva versión de la clave:

![Rotación de claves](./media/customer-managed-keys/key-rotation.png)

Puede deshabilitar la versión anterior de la clave tras 24 horas, o bien después de que los registros de auditoría de Azure Key Vault ya no muestren ninguna actividad que use la versión anterior.

Si usa la misma clave en Azure Sentinel y en Log Analytics, para realizar una rotación de claves debe actualizar explícitamente el recurso de clúster de Log Analytics con la nueva versión de clave de Azure Key Vault. Para más información, consulte [Rotación de CMK de Azure Monitor](../azure-monitor/platform/customer-managed-keys.md#cmk-kek-rotation).

## <a name="next-steps"></a>Pasos siguientes
En este documento ha aprendido a configurar una clave administrada por el cliente en Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats.md).
- [Use libros](tutorial-monitor-your-data.md) para supervisar los datos.

