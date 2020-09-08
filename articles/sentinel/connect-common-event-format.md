---
title: Conexión de datos CEF a la versión preliminar de Azure Sentinel | Microsoft Docs
description: Conecte una solución externa que envía mensajes de formato de evento común (CEF) a Azure Sentinel mediante el uso de una máquina Linux como proxy.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/26/2019
ms.author: yelevin
ms.openlocfilehash: 51e6c74a8b80b94ca552645cfbb76bd4e162a62b
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88650067"
---
# <a name="connect-your-external-solution-using-common-event-format"></a>Conexión de su solución externa con Common Event Format


Al conectar una solución externa que envía mensajes de CEF, hay tres pasos para conectarse con Azure Sentinel:

PASO 1: [Conectar CEF mediante la implementación del agente](connect-cef-agent.md) PASO 2: [Seguir los pasos específicos para la solución](connect-cef-solution-config.md) PASO 3: [Comprobar la conectividad](connect-cef-verify.md)

En este artículo se describe el funcionamiento de la conexión, se proporcionan los requisitos previos y los pasos para implementar el agente en soluciones de seguridad que envían mensajes de formato de evento común (CEF) sobre Syslog. 

> [!NOTE] 
> Los datos se almacenan en la ubicación geográfica del área de trabajo en la que se ejecute Azure Sentinel.

Para establecer esta conexión, debe implementar un agente en una máquina virtual dedicada o máquina local Linux (VM o local) para admitir la comunicación entre el dispositivo y Azure Sentinel. En el siguiente diagrama se describe la configuración en el caso de una VM Linux en Azure.

 ![CEF en Azure](./media/connect-cef/cef-syslog-azure.png)

Esta configuración también existirá si usa una VM en otra nube o en una máquina local. 

 ![CEF local](./media/connect-cef/cef-syslog-onprem.png)


## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Asegúrese de configurar la seguridad de la máquina de acuerdo con la directiva de seguridad de su organización. Por ejemplo, puede configurar la red para que se alinee con la directiva de seguridad de la red corporativa y cambiar los puertos y protocolos del demonio para que se adapten a sus requisitos. Puede usar las siguientes instrucciones para mejorar la configuración de seguridad de la máquina:   [Protección de máquinas virtuales en Azure](../virtual-machines/security-policy.md), [Procedimientos recomendados de seguridad de la red](../security/fundamentals/network-best-practices.md).

Para usar la comunicación TLS entre la solución de seguridad y la máquina de Syslog, debe configurar el demonio de Syslog (rsyslog o syslog-ng) para que se comunique en TLS: [Cifrado del tráfico de Syslog con TLS -rsyslog](https://www.rsyslog.com/doc/v8-stable/tutorials/tls_cert_summary.html), [Cifrado de los mensajes de registro con TLS –syslog-ng](https://support.oneidentity.com/technical-documents/syslog-ng-open-source-edition/3.22/administration-guide/60#TOPIC-1209298).

 
## <a name="prerequisites"></a>Requisitos previos
Asegúrese de que la máquina Linux que usa como proxy ejecute uno de los siguientes sistemas operativos:

- 64 bits
  - CentOS 6 y 7
  - Amazon Linux 2017.09
  - Oracle Linux 6 y 7
  - Red Hat Enterprise Linux Server 6 y 7
  - Debian GNU/Linux 8 y 9
  - Ubuntu Linux 14.04 LTS, 16.04 LTS y 18.04 LTS
  - SUSE Linux Enterprise Server 12
- 32 bits
   - CentOS 6
   - Oracle Linux 6
   - Red Hat Enterprise Linux Server 6
   - Debian GNU/Linux 8 y 9
   - Ubuntu Linux 14.04 LTS y 16.04 LTS
 
 - Versiones de demonio
   - Syslog-ng: 2.1 - 3.22.1
   - Rsyslog: v8
  
 - RFC de Syslog compatibles
   - Syslog RFC 3164
   - RFC de Syslog 5424
 
Asegúrese de que la máquina cumpla también con los requisitos siguientes: 
- Permisos
    - Debe tener permisos elevados (sudo) en la máquina. 
- Requisitos de software
    - Asegúrese de que tiene Python en ejecución en el equipo.



## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a conectar dispositivos CEF a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats.md).

