---
title: 'Acceso a Key Vault detrás de un firewall: Azure Key Vault | Microsoft Docs'
description: Aprenda qué puertos, hosts o direcciones IP debe abrir para habilitar un cliente del almacén de claves detrás de un firewall y acceder a un almacén de claves.
services: key-vault
author: amitbapat
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 08/12/2019
ms.author: ambapat
ms.openlocfilehash: 1ab5ae7bf9f1d13458e3bbeeec564fe642eb3303
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88588735"
---
# <a name="access-azure-key-vault-behind-a-firewall"></a>Acceso a Azure Key Vault desde detrás de un firewall

## <a name="what-ports-hosts-or-ip-addresses-should-i-open-to-enable-my-key-vault-client-application-behind-a-firewall-to-access-key-vault"></a>¿Qué puertos, hosts y direcciones IP debo abrir para habilitar mi cliente de almacén de claves detrás de un firewall para acceder al almacén de claves?

Para acceder a un almacén de claves, la aplicación cliente de Key Vault tiene que acceder a varios puntos de conexión para diversas funcionalidades:

* Autenticación a través de Azure Active Directory (Azure AD).
* Administración de Azure Key Vault. Aquí se incluye la creación, lectura, actualización, eliminación y establecimiento de directivas de acceso a través de Azure Resource Manager.
* Acceso y administración de objetos (claves y secretos) almacenados en la propia instancia de Key Vault a través del punto de conexión específico de Key Vault (por ejemplo, `https://yourvaultname.vault.azure.net`).  

En función de su configuración y entorno, hay algunas variaciones.

## <a name="ports"></a>Puertos

Todo el tráfico a un almacén de claves de las tres funciones (autenticación, la administración y acceso al plano de datos) se realiza a través de HTTPS: puerto 443. Sin embargo, ocasionalmente habrá tráfico HTTP (puerto 80) para CRL. Los clientes que admiten OCSP no deberían poder acceder a CRL, pero a veces pueden hacerlo a [http://cdp1.public-trust.com/CRL/Omniroot2025.crl](http://cdp1.public-trust.com/CRL/Omniroot2025.crl).  

## <a name="authentication"></a>Authentication

Será preciso que las aplicaciones cliente de Key Vault accedan a los puntos de conexión de Azure Active Directory para la autenticación. El punto de conexión que se utilice dependerá de la configuración del inquilino de Azure AD, del tipo de entidad de seguridad (entidad de seguridad de usuario o entidad de servicio) y del tipo de cuenta (por ejemplo, cuenta Microsoft o una cuenta profesional o educativa.  

| Tipo de entidad de seguridad | Punto de conexión:puerto |
| --- | --- |
| Usuario que utiliza cuenta Microsoft<br> (por ejemplo, user@hotmail.com) |**Global:**<br> login.microsoftonline.com:443<br><br> **Azure China:**<br> login.chinacloudapi.cn:443<br><br>**Azure US Gov:**<br> login.microsoftonline.us:443<br><br>**Azure Alemania:**<br> login.microsoftonline.de:443<br><br> y <br>login.live.com:443 |
| Entidad de seguridad de usuario o de servicio con una cuenta profesional o educativa con Azure AD (por ejemplo, user@contoso.com) |**Global:**<br> login.microsoftonline.com:443<br><br> **Azure China:**<br> login.chinacloudapi.cn:443<br><br>**Azure US Gov:**<br> login.microsoftonline.us:443<br><br>**Azure Alemania:**<br> login.microsoftonline.de:443 |
| Entidad de seguridad de usuario o de servicio con una cuenta profesional o educativa, más Servicios de federación de Active Directory (AD FS) u otro punto de conexión federado (por ejemplo, user@contoso.com) |Todos los puntos de conexión de una cuenta profesional o educativa, más AD FS u otros puntos de conexión federados |

Hay otros escenarios complejos posibles. Para más información, consulte [Escenarios de autenticación para Azure AD](../../active-directory/develop/authentication-scenarios.md), [Integración de aplicaciones con Azure Active Directory](../../active-directory/develop/active-directory-how-to-integrate.md) y [Guía del desarrollador de Azure Active Directory](https://msdn.microsoft.com/library/azure/dn151124.aspx).  

## <a name="key-vault-management"></a>Administración de Key Vault

Para la administración de Key Vault (CRUD y establecimiento de una directiva de acceso), es preciso que la aplicación cliente de Key Vault acceda a un punto de conexión de Azure Resource Manager.  

| Tipo de operación | Punto de conexión:puerto |
| --- | --- |
| Operaciones del plano de control de Key Vault<br> a través de Azure Resource Manager |**Global:**<br> management.azure.com:443<br><br> **Azure China:**<br> management.chinacloudapi.cn:443<br><br> **Azure US Gov:**<br> management.usgovcloudapi.net:443<br><br> **Azure Alemania:**<br> management.microsoftazure.de:443 |
| Microsoft Graph API |**Global:**<br> graph.microsoft.com:443<br><br> **Azure China:**<br> graph.chinacloudapi.cn:443<br><br> **Azure US Gov:**<br> graph.microsoft.com:443<br><br> **Azure Alemania:**<br> graph.cloudapi.de:443 |

## <a name="key-vault-operations"></a>Operaciones de Key Vault

Para todas las operaciones criptográficas y de administración de objetos (claves y secretos) de Key Vault, es preciso que el cliente de Key Vault acceda al punto de conexión de Key Vault. En función de la ubicación del almacén de claves, el sufijo DNS del punto de conexión varía. El punto de conexión de Key Vault tiene el formato *vault-name*.*region-specific-dns-suffix*, como se describe en la tabla siguiente.  

| Tipo de operación | Punto de conexión:puerto |
| --- | --- |
| Operaciones que incluyen operaciones criptográficas en claves; crear, leer, actualizar y eliminar claves y secretos; establecer u obtener etiquetas y otros atributos de objetos de Key Vault (claves o secretos) |**Global:**<br> &lt;vault-name&gt;.vault.azure.net:443<br><br> **Azure China:**<br> &lt;vault-name&gt;.vault.azure.cn:443<br><br> **Azure US Gov:**<br> &lt;vault-name&gt;.vault.usgovcloudapi.net:443<br><br> **Azure Alemania:**<br> &lt;vault-name&gt;.vault.microsoftazure.de:443 |

## <a name="ip-address-ranges"></a>Intervalos de direcciones IP

El servicio Key Vault utiliza otros recursos de Azure como la infraestructura de PaaS. Por consiguiente, no es posible proporcionar un intervalo específico de direcciones IP que los puntos de conexión del servicio Key Vault tendrán en un momento concreto. Si el firewall solo admite intervalos de direcciones IP, consulte los documentos sobre los intervalos de direcciones IP del centro de datos de Microsoft Azure:
* [Pública](https://www.microsoft.com/en-us/download/details.aspx?id=56519)
* [US Gov](https://www.microsoft.com/en-us/download/details.aspx?id=57063)
* [Alemania](https://www.microsoft.com/en-us/download/details.aspx?id=57064)
* [China](https://www.microsoft.com/en-us/download/details.aspx?id=57062)

Authentication and Identity (Azure Active Directory) es un servicio global que puede conmutar por error en otras regiones o mover tráfico sin previo aviso. En este escenario, todos los intervalos IP enumerados en [Direcciones IP de Authentication and Identity](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2#bkmk_identity_ip) deben agregarse al firewall.

## <a name="next-steps"></a>Pasos siguientes

Si le queda alguna duda acerca de Key Vault, visite la [página de preguntas más frecuentes de Microsoft para Azure Key Vault](https://docs.microsoft.com/answers/topics/azure-key-vault.html).
