---
title: Recomendaciones de seguridad
description: Implemente las recomendaciones de seguridad, ya que le ayudarán a cumplir sus obligaciones de seguridad, tal como se describe en nuestro modelo de responsabilidad compartida. Mejore la seguridad de la aplicación.
author: msmbaldwin
manager: barbkess
ms.topic: conceptual
ms.date: 06/17/2019
ms.author: mbaldwin
ms.custom: security-recommendations
ms.openlocfilehash: eba08211355b0f57dd9dd11ec4c18d5912208ec8
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962220"
---
# <a name="security-recommendations-for-app-service"></a>Recomendaciones de seguridad para App Service

Este artículo contiene recomendaciones de seguridad para Azure App Service. La implementación de estas recomendaciones le ayudará a cumplir con las obligaciones de seguridad que se describen en nuestro modelo de responsabilidad compartido y mejorará la seguridad general de sus soluciones de aplicación de web. Para obtener más información sobre lo que Microsoft hace para cumplir con las responsabilidades del proveedor de servicios, lea [Seguridad de la infraestructura de Azure](../security/fundamentals/infrastructure.md).

## <a name="general"></a>General

| Recomendación | Comentarios |
|-|-|----|
| Manténgase al día | Use las versiones más recientes de las plataformas compatibles, los lenguajes de programación, los protocolos y los marcos. |

## <a name="identity-and-access-management"></a>Administración de identidades y acceso

| Recomendación | Comentarios |
|-|----|
| Deshabilitar acceso anónimo | A menos que necesite admitir solicitudes anónimas, deshabilite el acceso anónimo. Para más información sobre cómo funcionan las opciones de autenticación de Azure App Service, vea [Autenticación y autorización en Azure App Service](overview-authentication-authorization.md).|
| Requerir autenticación | Siempre que sea posible, use el módulo de autenticación de App Service en lugar de escribir código para controlar la autenticación y la autorización. Vea [Autenticación y autorización en Azure App Service](overview-authentication-authorization.md). |
| Protección de los recursos de back-end con acceso autenticado | Puede usar la identidad del usuario o usar una identidad de aplicación para autenticarse en un recurso de back-end. Si decide usar una identidad de aplicación use una [identidad administrada](overview-managed-identity.md).
| Requerir autenticación de certificados de clientes | La autenticación de certificados de clientes mejora la seguridad al permitir conexiones solo de los clientes que se pueden autenticar con los certificados que proporciona. |

## <a name="data-protection"></a>Protección de los datos

| Recomendación | Comentarios |
|-|-|
| Redireccionamiento de HTTP a HTTPS | De forma predeterminada, los clientes pueden conectarse a las aplicaciones web mediante HTTP o HTTPS. Se recomienda redirigir HTTP a HTTPs porque HTTPS usa el protocolo SSL/TLS para proporcionar una conexión segura, que está tanto cifrada como autenticada. |
| Cifrado de la comunicación a los recursos de Azure | Cuando la aplicación se conecta a recursos de Azure tales como [SQL Database](https://azure.microsoft.com/services/sql-database/) o [Azure Storage](../storage/index.yml), la conexión permanece dentro de Azure. Puesto que la conexión pasa por las funciones de red compartidas de Azure, siempre debe cifrar toda la comunicación. |
| Requerir la versión de TLS más reciente | Desde 2018, las nuevas aplicaciones de Azure App Service usan TLS 1.2. Las versiones más recientes de TLS incluyen mejoras de seguridad con respecto a versiones anteriores del protocolo. |
| Uso de FTPS | App Service admite FTP y FTPS para la implementación de los archivos. Use FTPS en lugar de FTP cuando sea posible. Si uno o los dos protocolos no están en uso, debe [deshabilitarlos](deploy-ftp.md#enforce-ftps). |
| Protección de datos de aplicación | No almacene los secretos de la aplicación, como las credenciales de la base de datos, los tokens de API ni las claves privadas en el código o en archivos de configuración. El enfoque aceptado es acceder a ellos como [variables de entorno](https://wikipedia.org/wiki/Environment_variable), usando el patrón estándar en el lenguaje que prefiera. En Azure App Service, puede definir variables de entorno a través de la [configuración de la aplicación](./configure-common.md) y las [cadenas de conexión](./configure-common.md). La configuración de la aplicación y las cadenas de conexión se almacenan cifradas en Azure. La configuración de la aplicación se descifra solo antes de insertarla en la memoria de proceso de la aplicación cuando se inicia la aplicación. Las claves de cifrado rotan con regularidad. También puede integrar la aplicación de Azure App Service con [Azure Key Vault](../key-vault/index.yml) para la administración avanzada de secretos. Al [acceder a Key Vault con una identidad administrada](../key-vault/general/tutorial-net-create-vault-azure-web-app.md), la aplicación App Service puede acceder de forma segura a los secretos que necesita. |

## <a name="networking"></a>Redes

| Recomendación | Comentarios |
|-|-|
| Uso de restricciones de IP estática | En Windows, Azure App Service permite definir una lista de direcciones IP que tienen permiso para acceder a su aplicación. La lista de direcciones permitidas puede incluir direcciones IP individuales o un intervalo de direcciones IP definido por una máscara de subred. Para más información, consulte [Restricciones de IP estáticas de Azure App Service](app-service-ip-restrictions.md).  |
| Uso del plan de tarifa aislado | Excepto en el plan de tarifa aislado, todos los niveles ejecutan las aplicaciones en la infraestructura de red compartida en Azure App Service. El plan aislado ejecuta las aplicaciones dentro de un [entorno de App Service](environment/intro.md) dedicado para tener un aislamiento de red completo. Los entornos de App Service se ejecutan en su propia instancia de la [red virtual de Azure](../virtual-network/index.yml).|
| Uso de conexiones seguras al acceder a recursos locales | Puede usar [conexiones híbridas](app-service-hybrid-connections.md), [integración de Virtual Network](web-sites-integrate-with-vnet.md) o [instancias de App Service Environment](environment/intro.md) para conectarse a recursos locales. |
| Limitación de la exposición a tráfico de red entrante | Los grupos de seguridad de red le permiten restringir el acceso a la red y controlar el número de puntos de conexión expuestos. Para obtener más información, vea [Cómo controlar el tráfico de entrada a un entorno de App Service](environment/app-service-app-service-environment-control-inbound-traffic.md). |

## <a name="monitoring"></a>Supervisión

| Recomendación | Comentarios |
|-|-|
|Uso del nivel estándar de Azure Security Center | [Azure Security Center](../security-center/security-center-app-services.md) se integra de forma nativa con Azure App Service. Puede ejecutar evaluaciones y proporcionar recomendaciones de seguridad. |

## <a name="next-steps"></a>Pasos siguientes

Póngase en contacto con el proveedor de la aplicación para ver si hay más requisitos de seguridad. Para obtener más información sobre el desarrollo de aplicaciones seguras, vea [Documentación sobre desarrollo seguro](https://azure.microsoft.com/resources/develop-secure-applications-on-azure/).