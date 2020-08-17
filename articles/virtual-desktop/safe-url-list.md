---
title: 'Lista de direcciones URL seguras de Windows Virtual Desktop: Azure'
description: Una lista de direcciones URL que debe desbloquear para garantizar que su implementación de Windows Virtual Desktop funcione según lo previsto.
services: virtual-desktop
author: heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 07/15/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 9f7a3b51afa11562123a280da8634e100a22e6b6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87075617"
---
# <a name="safe-url-list"></a>Lista de direcciones URL seguras

Deberá desbloquear ciertas direcciones URL para que la implementación de Windows Virtual Desktop funcione correctamente. En este artículo se enumeran estas direcciones URL para saber cuáles son seguras.

## <a name="virtual-machines"></a>Máquinas virtuales

Las máquinas virtuales de Azure que cree para Windows Virtual Desktop deben tener acceso a las siguientes direcciones URL:

|Dirección|Puerto TCP de salida|Propósito|Etiqueta de servicio|
|---|---|---|---|
|*.wvd.microsoft.com|443|Tráfico de servicio|WindowsVirtualDesktop|
|mrsglobalsteus2prod.blob.core.windows.net|443|Actualizaciones de la pila de agentes y SXS|AzureCloud|
|*.core.windows.net|443|Tráfico de agente|AzureCloud|
|*.servicebus.windows.net|443|Tráfico de agente|AzureCloud|
|prod.warmpath.msftcloudes.com|443|Tráfico de agente|AzureCloud|
|catalogartifact.azureedge.net|443|Azure Marketplace|AzureCloud|
|kms.core.windows.net|1688|Activación de Windows|Internet|
|wvdportalstorageblob.blob.core.windows.net|443|Soporte técnico de Azure Portal|AzureCloud|
| 169.254.169.254 | 80 | [Punto de conexión de servicio de metadatos de instancias de Azure](../virtual-machines/windows/instance-metadata-service.md) | N/D |

>[!IMPORTANT]
>Windows Virtual Desktop ahora admite la etiqueta FQDN. Para más información, consulte [Uso de Azure Firewall para proteger las implementaciones de Windows Virtual Desktop](../firewall/protect-windows-virtual-desktop.md).
>
>Se recomienda usar etiquetas FQDN o etiquetas de servicio, en lugar de las direcciones URL, para evitar problemas de servicio. Las direcciones URL y etiquetas de la lista solo corresponden a sitios y recursos de Windows Virtual Desktop. No incluyen direcciones URL para otros servicios como Azure Active Directory.

En la tabla siguiente se enumeran las direcciones URL opcionales a las que pueden tener acceso las máquinas virtuales de Azure:

|Dirección|Puerto TCP de salida|Propósito|Etiqueta de servicio|
|---|---|---|---|
|*.microsoftonline.com|443|Autenticación en Microsoft Online Services|None|
|*.events.data.microsoft.com|443|Servicio de telemetría|None|
|www.msftconnecttest.com|443|Detecta si el sistema operativo está conectado a Internet.|None|
|*.prod.do.dsp.mp.microsoft.com|443|Windows Update|None|
|login.windows.net|443|Inicio de sesión en Microsoft Online Services, Microsoft 365|None|
|*.sfx.ms|443|Actualizaciones del software cliente de OneDrive|None|
|*.digicert.com|443|Comprobación de revocación de certificados|None|

>[!NOTE]
>Windows Virtual Desktop actualmente no tiene una lista de intervalos de direcciones IP que puede desbloquear para permitir el tráfico de red. En este momento, solo se pueden desbloquear direcciones URL específicas.
>
>Para obtener una lista de direcciones URL seguras relacionadas con Office, incluidas las direcciones URL requeridas relacionadas con Azure Active Directory, consulte [Direcciones URL e intervalos de direcciones IP de Office 365](/office365/enterprise/urls-and-ip-address-ranges).
>
>Debe usar el carácter comodín (*) para las direcciones URL que impliquen tráfico de servicio. Si prefiere no usar el carácter comodín (*) para el tráfico relacionado con el agente, aquí se muestra cómo buscar las direcciones URL sin estos caracteres:
>
>1. Registre las máquinas virtuales en el grupo de hosts de Windows Virtual Desktop.
>2. Abra el **Visor de eventos** y, después, diríjase a **Registros de Windows** > **Aplicación** > **WVD-Agent** y busque el evento con el identificador 3701.
>3. Agregue a la lista de permitidos las direcciones URL que se encuentran en el identificador de evento 3701. Las direcciones URL del identificador de evento 3701 son específicas de una región. Deberá repetir el proceso de desbloqueo con las direcciones URL pertinentes de cada región en la que quiera implementar las máquinas virtuales.

## <a name="remote-desktop-clients"></a>Clientes de Escritorio remoto

Todos los clientes de Escritorio remoto que use deben tener acceso a las siguientes direcciones URL:

|Dirección|Puerto TCP de salida|Propósito|Cliente(s)|
|---|---|---|---|
|*.wvd.microsoft.com|443|Tráfico de servicio|All|
|*.servicebus.windows.net|443|Solución de problemas de los datos|All|
|go.microsoft.com|443|Microsoft FWLinks|All|
|aka.ms|443|Acortador de direcciones URL de Microsoft|All|
|docs.microsoft.com|443|Documentación|All|
|privacy.microsoft.com|443|Declaración de privacidad|All|
|query.prod.cms.rt.microsoft.com|443|Actualizaciones de clientes|Escritorio de Windows|

>[!IMPORTANT]
>Para conseguir una experiencia de cliente confiable, es esencial abrir estas direcciones URL. No se admite el bloqueo del acceso a estas direcciones URL y afectará a la funcionalidad del servicio. Estas direcciones URL solo se corresponden con sitios y recursos de cliente y no incluyen direcciones URL para otros servicios como Azure Active Directory.
