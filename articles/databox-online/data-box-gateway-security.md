---
title: Seguridad de Azure Data Box Gateway | Microsoft Docs
description: Describe las características de seguridad y privacidad que protegen el dispositivo virtual Azure Data Box Gateway, así como el servicio y los datos, tanto locales como en la nube.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 08/21/2019
ms.author: alkohli
ms.openlocfilehash: 45506b87bf13b44e035655dbdf0fac03ea50612c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90893856"
---
# <a name="azure-data-box-gateway-security-and-data-protection"></a>Protección de datos y seguridad de Azure Data Box Gateway

La seguridad es una gran preocupación para cualquiera que adopte una tecnología nueva, en especial cuando se utiliza la tecnología con información confidencial o de su propiedad. Azure Data Box Gateway le ayuda a asegurarse de que solo entidades autorizadas puedan ver, modificar o eliminar sus datos.

En este artículo se describen las características de seguridad de Azure Data Box Gateway que ayudan a proteger cada uno de los componentes de la solución y los datos almacenados en ellos.

La solución Data Box Gateway consta de cuatro componentes principales que interactúan entre sí:

- **Servicio Data Box Gateway hospedado en Azure**. El recurso de administración que se usa para crear el pedido del dispositivo, configurar el dispositivo y, a continuación, supervisar el pedido hasta que se realiza.
- **Dispositivo Data Box Gateway**. Dispositivo virtual que aprovisiona en el hipervisor del sistema que proporciona. Este dispositivo virtual se usa para importar sus datos locales en Azure.
- Los **clientes/hosts conectados al dispositivo**. Clientes de la infraestructura que se conectan al dispositivo Data Box Gateway y contienen los datos que se deben proteger.
- El **almacenamiento en la nube**. La ubicación de la plataforma Azure Cloud donde se almacenan los datos. Esta ubicación suele ser la cuenta de almacenamiento vinculada al recurso de Data Box Gateway que ha creado.

## <a name="data-box-gateway-service-protection"></a>Protección del servicio Data Box Gateway

Data Box Gateway es un servicio de administración que se hospeda en Azure. El servicio se usa para configurar y administrar el dispositivo.

[!INCLUDE [data-box-edge-gateway-service-protection](../../includes/data-box-edge-gateway-service-protection.md)]

## <a name="data-box-gateway-device-protection"></a>Protección del dispositivo Data Box Gateway

El dispositivo Data Box Gateway es un dispositivo virtual que se aprovisiona en el hipervisor de un sistema local que proporciona el usuario. El dispositivo ayuda a enviar datos a Azure. El dispositivo:

- Necesita una clave de activación para acceder a los servicios Azure Stack Edge Pro o Data Box Gateway.
- Está protegido en todo momento mediante una contraseña de dispositivo.
<!---  secure boot enabled.
- Runs Windows Defender Device Guard. Device Guard allows you to run only trusted applications that you define in your code integrity policies.-->

### <a name="protect-the-device-via-activation-key"></a>Protege el dispositivo con una clave de activación

Solo se permite a un dispositivo Data Box Gateway autorizado unirse al servicio Data Box Gateway que crea en su suscripción de Azure. Si quiere autorizar un dispositivo, debe usar una clave de activación para activarlo con el servicio Data Box Gateway.

[!INCLUDE [data-box-edge-gateway-activation-key](../../includes/data-box-edge-gateway-activation-key.md)]

Para obtener más información, consulte [Obtención de la clave de activación](data-box-gateway-deploy-prep.md#get-the-activation-key).

### <a name="protect-the-device-via-password"></a>Protección del dispositivo con una contraseña

Las contraseñas garantizan que solo los usuarios autorizados puedan acceder a sus datos. Los dispositivos Data Box Gateway arrancan en un estado bloqueado.

Puede:

- Conectarse a la interfaz de usuario web local del dispositivo mediante un explorador y, a continuación, proporcionar una contraseña para iniciar sesión en el dispositivo.
- Conectarse de forma remota a la interfaz de PowerShell del dispositivo a través de HTTP. La administración remota está activada de manera predeterminada. A continuación, puede proporcionar la contraseña del dispositivo para iniciar sesión en el dispositivo. Para obtener más información, consulte el tema sobre la [conexión remota al dispositivo Data Box Gateway](data-box-gateway-connect-powershell-interface.md#connect-to-the-powershell-interface).

[!INCLUDE [data-box-edge-gateway-password-best-practices](../../includes/data-box-edge-gateway-password-best-practices.md)]
- Use la interfaz de usuario web local para [cambiar la contraseña](data-box-gateway-manage-access-power-connectivity-mode.md#manage-device-access). Si cambia la contraseña, asegúrese de notificar a todos los usuarios de acceso remoto para que no experimenten problemas al iniciar sesión.

## <a name="protect-your-data"></a>Protección de los datos

En esta sección se describen las características de seguridad de Data Box Gateway que protegen los datos en tránsito y almacenados.

### <a name="protect-data-at-rest"></a>Protección de los datos en reposo

[!INCLUDE [data-box-edge-gateway-data-rest](../../includes/data-box-edge-gateway-data-rest.md)]

### <a name="protect-data-in-flight"></a>Protección de los datos en tránsito

[!INCLUDE [data-box-edge-gateway-data-flight](../../includes/data-box-edge-gateway-data-flight.md)]

### <a name="protect-data-using-storage-accounts"></a>Protección de datos a través de cuentas de almacenamiento

[!INCLUDE [data-box-edge-gateway-data-storage-accounts](../../includes/data-box-edge-gateway-protect-data-storage-accounts.md)]

- Rote y, a continuación, [sincronice las claves de la cuenta de almacenamiento](data-box-gateway-manage-shares.md#sync-storage-keys) periódicamente para proteger su cuenta de almacenamiento de usuarios no autorizados.

### <a name="protect-the-device-data-using-bitlocker"></a>Protección de los datos del dispositivo mediante BitLocker

Para proteger los discos virtuales de la máquina virtual Data Box Gateway, se recomienda habilitar BitLocker. De forma predeterminada, BitLocker no está habilitado. Para más información, consulte:

- [Configuración de la compatibilidad con el cifrado en el administrador de Hyper-V](hhttps://docs.microsoft.com/windows-server/virtualization/hyper-v/learn-more/generation-2-virtual-machine-security-settings-for-hyper-v#encryption-support-settings-in-hyper-v-manager)
- [Compatibilidad con BitLocker en una máquina virtual](https://kb.vmware.com/s/article/2036142)

## <a name="manage-personal-information"></a>Administración de información personal

El servicio Data Box Gateway recopila información personal en los escenarios siguientes:

[!INCLUDE [data-box-edge-gateway-manage-personal-data](../../includes/data-box-edge-gateway-manage-personal-data.md)]

Para ver la lista de usuarios que pueden acceder a un recurso compartido o eliminarlo, siga los pasos que se indican en el tema sobre la [administración de recursos compartidos en Data Box Gateway](data-box-gateway-manage-shares.md).

Para más información, revise la directiva de privacidad de Microsoft en el [Centro de confianza](https://www.microsoft.com/trustcenter).

## <a name="next-steps"></a>Pasos siguientes

[Implementación del dispositivo Data Box Gateway](data-box-gateway-deploy-prep.md)
