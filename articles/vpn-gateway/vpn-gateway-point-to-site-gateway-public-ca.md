---
title: Transición a certificados de entidad de certificación públicos para puertas de enlace P2S | Azure VPN Gateway | Microsoft Docs
description: Este artículo lo ayuda a realizar una transición exitosa a los nuevos certificados de entidad de certificación públicos para puertas de enlace P2S.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 05/16/2019
ms.author: cherylmc
ms.openlocfilehash: 1d45e1a5e4053ead4330967c5e250c0797c19fe7
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827468"
---
# <a name="transition-to-a-public-ca-gateway-certificate-for-p2s"></a>Transición a un certificado de puerta de enlace de entidad de certificación pública para P2S

Azure VPN Gateway ya no emite certificados autofirmados en el nivel de Azure para sus puertas de enlace para conexiones P2S. Los certificados emitidos ahora están firmados por una entidad de certificación (CA) pública. Sin embargo, algunas de las puertas de enlace anteriores todavía utilizan certificados autofirmados. Estos certificados autofirmados están a punto de expirar y deben realizar la transición a los certificados de entidad de certificación pública.

>[!NOTE]
> * Este cambio de certificado en el nivel de Azure no afecta a los certificados autofirmados que se usan en la autenticación de cliente de P2S. Puede continuar con la emisión y uso de certificados autofirmados como siempre.
>

Los certificados en este contexto constituyen un certificado adicional en el nivel de Azure. No son las cadenas de certificados que usa al generar sus propios certificados raíz autofirmados y certificados de cliente para la autenticación. Esos certificados no se ven afectados y expirarán en las fechas que haya establecido para ello.

Anteriormente, un certificado autofirmado para la puerta de enlace (que emitía Azure en segundo plano) debía actualizarse cada 18 meses. Los archivos de configuración de cliente VPN tenían que generarse y volver a implementarse en todos los clientes P2S. Al migrar a los certificados de entidad de certificación pública, se elimina esta limitación. Además de la transición para certificados, este cambio también proporciona mejoras en la plataforma, una métrica mejor y estabilidad mejorada.

Solo las puertas de enlace anteriores se ven afectados por este cambio. Si el certificado de puerta de enlace se puede migrar, recibirá un aviso en Azure Portal. Puede ver si se ve afectada la puerta de enlace siguiendo los pasos de este artículo.

> [!IMPORTANT]
> Se ha completado la transición para certificados de entidad de certificación pública en el 12 de mayo de 2019. **Este documento se quitará en el 31 de mayo de 2019.**

## <a name="1-verify-your-certificate"></a>1. Comprobación del certificado

### <a name="resource-manager"></a>Resource Manager

1. Compruebe si se ven afectadas por esta actualización. Descargar la configuración actual del cliente VPN mediante los pasos descritos en [este artículo](point-to-site-vpn-client-configuration-azure-cert.md).

2. Abra o extraiga el archivo ZIP y vaya a la carpeta "Generic". En la carpeta Generic, verá dos archivos, uno de los cuales es *VPNSettings.xml*.
3. Abra *VPNSettings.xml* en cualquier editor o visor de XML. En el archivo XML, busque los siguientes campos:

   * `<ServerCertRootCn>DigiCert Global Root CA</ServerCertRootCn>`
   * `<ServerCertIssuerCn>DigiCert Global Root CA</ServerCertIssuerCn>`
4. Si *ServerCertRotCn* y *ServerCertIssuerCn* son "DigiCert Global Root CA", no se verán afectados por esta actualización y no es necesario continuar con los pasos descritos en este artículo. Sin embargo, si aparece algo más, el certificado de puerta de enlace forma parte de la actualización y realizará la transición.

### <a name="classic"></a>Clásica

1. En un equipo cliente, vaya a la ruta de acceso `%appdata%/Microsoft/Network/Connections/Cm/<gatewayID>`. En la carpeta con el identificador de la puerta de enlace, puede ver el certificado.
2. En la pestaña General del certificado, compruebe que la entidad emisora es "DigiCert Global Root CA". Si tiene algo distinto a esta entidad emisora, el certificado de la puerta de enlace formará parte de la actualización y se realizará su transición.

## <a name="2-check-certificate-transition-schedule"></a>2. Comprobación de la programación de transición del certificado

Si el certificado forma parte de la actualización, el certificado de puerta de enlace se migrará. Consulte esta nota **importante** de tiempos de transición. Tras la actualización, los clientes P2S no podrán conectarse con su perfil antiguo. Debe generar nuevos perfiles de cliente VPN e instalarlos en los clientes.

## <a name="3-generate-vpn-client-configuration-profile"></a>3. Generación del perfil de configuración de cliente VPN

Una vez que se ha migrado el certificado, descargue el nuevo perfil VPN (archivos de configuración de cliente VPN) desde Azure Portal. Para ver los pasos, consulte [Creación e instalación de los archivos de configuración de cliente de VPN](point-to-site-vpn-client-configuration-azure-cert.md). No es necesario generar nuevos certificados de cliente.

## <a name="4-deploy-vpn-client-profile"></a>4. Implementación del perfil de cliente VPN

Implemente el nuevo perfil en todos los clientes VPN de punto a sitio. Los clientes VPN perderán conectividad hasta que el nuevo perfil VPN para conexiones de punto a sitio se descargue e implemente en los dispositivos cliente. No es necesario volver a emitir los certificados de cliente que ya están instalados en los equipos cliente VPN.

## <a name="5-connect-the-vpn-client"></a>5. Conexión del cliente VPN

Conéctese a Azure desde el cliente VPN como lo haría normalmente.
