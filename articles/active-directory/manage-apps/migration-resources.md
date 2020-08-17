---
title: Recursos para migrar aplicaciones a Azure Active Directory | Microsoft Docs
description: Recursos para ayudarle a migrar el acceso a la aplicación y la autenticación a Azure Active Directory (Azure AD).
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: how-to
ms.workload: identity
ms.date: 02/29/2020
ms.author: kenwith
ms.reviewer: baselden
ms.openlocfilehash: 327e59d3fee6f23d2635b846e9fb5ab422e0139e
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87429031"
---
# <a name="resources-for-migrating-applications-to-azure-active-directory"></a>Recursos para migrar aplicaciones a Azure Active Directory

Recursos para ayudarle a migrar el acceso a la aplicación y la autenticación a Azure Active Directory (Azure AD).

| Recurso  | Descripción  |
|:-----------|:-------------|
|[Migrar sus aplicaciones a Azure AD](https://aka.ms/migrateapps/whitepaper) | En este artículo se presentan las ventajas de la migración y se describe cómo planear la migración en cuatro fases bien definidas: detección, clasificación, migración y administración continua. Se le guiará sobre cómo debe pensar en el proceso y desglosar el proyecto en partes fáciles de consumir. Este documento incluye vínculos a recursos importantes que le ayudarán a lo largo de este proceso. |
|[Guía de la solución: migración de aplicaciones de Servicios de federación de Active Directory (AD FS) a Azure AD](https://aka.ms/migrateapps/adfssolutionguide) | Esta guía de la solución le guiará a través de la mismas cuatro fases de planeamiento y ejecución de un proyecto de migración de aplicaciones que se describen en un nivel más alto en las notas del producto de migración. Con esta guía, aprenderá a aplicar estas fases al objetivo concreto de mover una aplicación de Servicios de federación de Active Directory (AD FS) a Azure AD.|
| [Herramienta: script de preparación para la migración de Servicios de federación de Active Directory](https://aka.ms/migrateapps/adfstools) | Se trata de un script que puede ejecutar en el servidor local de Servicios de federación de Active Directory (AD FS) para determinar la preparación de las aplicaciones para la migración a Azure AD.|
| [Plan de implementación: migración de AD FS a la sincronización de hash de contraseña](https://aka.ms/ADFSTOPHSDPDownload) | Con la sincronización de hash de contraseña, se sincronizan los valores hash de las contraseñas de los usuarios de Active Directory local con Azure AD. Esto permite a Azure AD autenticar a los usuarios sin interactuar con la instancia local de Active Directory.| 
| [Plan de implementación: migración de AD FS a autenticación de paso a través](https://aka.ms/ADFSTOPTADPDownload)|La autenticación de paso a través de Azure AD ayuda a los usuarios a iniciar sesión en aplicaciones locales y en la nube con la misma contraseña. Esta característica proporciona a los usuarios una mejor experiencia, puesto que tienen una contraseña menos que recordar. También reduce los costos del departamento de soporte técnico, ya que es menos probable que olviden cómo iniciar sesión si solo necesitan recordar una contraseña. Cuando los usuarios inician sesión con Azure AD, esta característica valida sus contraseñas directamente con la instancia de Active Directory local.|
| [Plan de implementación: habilitación del inicio de sesión único en una aplicación SaaS con Azure AD](https://aka.ms/SSODPDownload) | El inicio de sesión único (SSO) le ayuda a acceder a todas las aplicaciones y los recursos que necesita para hacer negocios, iniciando sesión una sola vez con una única cuenta de usuario. Por ejemplo, después de iniciar la sesión, el usuario puede cambiar entre Microsoft Office, Salesforce o Box sin necesidad de volver a autenticarse (por ejemplo, mediante una contraseña). 
| [Plan de implementación: ampliación de las aplicaciones a Azure AD con el proxy de aplicación](https://aka.ms/AppProxyDPDownload)| Para proporcionar acceso a los portátiles de los empleados y otros dispositivos a las aplicaciones locales, siempre se han usado redes privadas virtuales (VPN) o redes perimetrales (DMZ). No obstante, estas soluciones no solo son complejas y difíciles de proteger, sino también costosas de configurar y administrar. Azure AD Application Proxy facilita el acceso a las aplicaciones locales. |
| [Planes de implementación](../fundamentals/active-directory-deployment-plans.md) | Encuentre más planes de implementación para implementar características como la autenticación multifactor, el acceso condicional, el aprovisionamiento de usuarios, SSO ininterrumpido, el autoservicio de restablecimiento de contraseña y mucho más. |


