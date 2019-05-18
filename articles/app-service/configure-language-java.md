---
title: 'Configurar Windows aplicaciones de Java: Azure App Service | Microsoft Docs'
description: Obtenga información sobre cómo configurar aplicaciones de Java se ejecuten en las instancias de Windows de forma predeterminada en Azure App Service.
keywords: servicio de aplicación de Azure, aplicación web, windows, oss, java
services: app-service
author: jasonfreeberg
manager: jeconnock
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: java
ms.topic: article
ms.date: 04/12/2019
ms.author: jafreebe;cephalin
ms.custom: seodec18
ms.openlocfilehash: 82e8936a888cbc99088ab18423e55dd57a3c2e77
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65604143"
---
# <a name="configure-a-windows-java-app-for-azure-app-service"></a>Configurar un Windows aplicación de Java para Azure App Service

Azure App Service permite a los desarrolladores de Java para crear, implementar y escalar su Tomcat rápidamente o Java Standard Edition (SE) empaqueta las aplicaciones web en un servicio totalmente administrado basado en Windows. Implemente aplicaciones con complementos de Maven desde la línea de comandos o en editores, como IntelliJ, Eclipse o Visual Studio Code.

Esta guía proporciona los conceptos clave e instrucciones para desarrolladores de Java en App Service. Si nunca ha usado Azure App Service, debe leer la [inicio rápido de Java](app-service-web-get-started-java.md) primero. Preguntas generales sobre el uso de App Service que no son específicas para el desarrollo de Java se responden en el [preguntas más frecuentes sobre Windows de servicio de aplicación](faq-configuration-and-management.md).

> [!NOTE]
> ¿No encuentra lo que busca? Consulte la [preguntas más frecuentes sobre sistemas operativos de Windows](faq-configuration-and-management.md) o [Guía de configuración de Linux Java](containers/configure-language-java.md) para obtener información sobre la implementación y la protección de la aplicación de Java.

## <a name="configuring-tomcat"></a>Configuración de Tomcat

Para editar de Tomcat `server.xml` u otros archivos de configuración, en primer lugar tome nota de la versión principal de Tomcat en el portal.

1. Encontrar el directorio principal de Tomcat para su versión, ejecute el `env` comando. Busque la variable de entorno que comienza con `AZURE_TOMCAT`y coincide con la versión principal. Por ejemplo, `AZURE_TOMCAT85_HOME` señala al directorio Tomcat para Tomcat 8.5.
1. Una vez que haya identificado el directorio principal de Tomcat para su versión, copie el directorio de configuración para `D:\home`. Por ejemplo, si `AZURE_TOMCAT85_HOME` tenía un valor de `D:\Program Files (x86)\apache-tomcat-8.5.37`, sería la ruta de acceso completa del directorio de configuración copiado `D:\home\tomcat\conf`.

Finalmente, reinicie App Service. Las implementaciones deben ir a `D:\home\site\wwwroot\webapps` igual que antes.

## <a name="java-runtime-statement-of-support"></a>Instrucción de compatibilidad de Java Runtime

### <a name="jdk-versions-and-maintenance"></a>Mantenimiento y versiones de JDK

El JDK (Java Development Kit) compatible con Azure es [Zulu](https://www.azul.com/downloads/azure-only/zulu/) y se distribuye a través de [Azul Systems](https://www.azul.com/).

Las actualizaciones de versión principal se proporcionará a través de nuevas opciones de tiempo de ejecución en Azure App Service para Windows. Para actualizar a estas versiones más recientes, los clientes deben configurar su implementación de App Service. Asimismo, son responsables de probar y garantizar que la actualización principal satisface sus necesidades.

Los JDK compatibles se revisarán trimestralmente de manera automática en enero, abril, julio y octubre de cada año.

### <a name="security-updates"></a>Actualizaciones de seguridad

Las revisiones y correcciones de las vulnerabilidades de seguridad principales se publicarán en cuanto estén disponibles a través de Azul Systems. Una vulnerabilidad "principal" se define con una puntuación total de 9.0 o superior en el [CVSS (Common Vulnerability Scoring System), versión 2 del NIST](https://nvd.nist.gov/cvss.cfm).

### <a name="deprecation-and-retirement"></a>Desuso y retirada

Si un entorno compatible de Java Runtime se va a retirar, los desarrolladores de Azure que lo usen recibirán un aviso de desuso al menos seis meses antes de su retirada.

### <a name="local-development"></a>Desarrollo local

Los desarrolladores pueden descargar el JDK de Azul Zulu Enterprise Production Edition para el desarrollo local desde el [sitio de descarga de Azul](https://www.azul.com/downloads/azure-only/zulu/).

### <a name="development-support"></a>Soporte para el desarrollo

Soporte técnico para el [compatibles con Azure Azul Zulu JDK](https://www.azul.com/downloads/azure-only/zulu/) está disponible a través de Microsoft al desarrollar para Azure o [Azure Stack](https://azure.microsoft.com/overview/azure-stack/) con un [completo plan de soporte técnico de Azure](https://azure.microsoft.com/support/plans/).

### <a name="runtime-support"></a>Compatibilidad con el tiempo de ejecución

Los desarrolladores pueden [abrir una incidencia](/azure/azure-supportability/how-to-create-azure-support-request) con los JDK de Azul Zulu a través de soporte técnico de Azure si tienen un [plan de soporte técnico completo](https://azure.microsoft.com/support/plans/).

## <a name="next-steps"></a>Pasos siguientes

En este tema se proporciona la instrucción en tiempo de ejecución de Java de soporte técnico para Azure App Service en Windows.

- Para más información sobre el hospedaje de aplicaciones web con el servicio de aplicación de Azure, consulte [Introducción a App Service](overview.md).
- Para obtener información acerca de Java en el desarrollo de Azure, consulte [Azure para el centro de desarrollo de Java](https://docs.microsoft.com/java/azure/?view=azure-java-stable).
