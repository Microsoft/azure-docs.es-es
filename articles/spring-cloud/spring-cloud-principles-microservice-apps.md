---
title: Java y sistema operativo base para aplicaciones de microservicios de Azure Spring Cloud
description: Principios para mantener el buen estado de Java y el sistema operativo base para las aplicaciones de microservicios de Azure Spring Cloud
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 05/27/2020
ms.custom: devx-track-java
ms.openlocfilehash: 8cda46a011ae92f26a15a4e9a918559801ce299c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906874"
---
# <a name="java-and-base-os-for-spring-microservice-apps"></a>Java y sistema operativo base para aplicaciones de microservicios de Spring

**Este artículo se aplica a:** ✔️ Java

A continuación se indican los principios para mantener en buen estado Java y el sistema operativo base para las aplicaciones de microservicios de Spring.
## <a name="principles-for-healthy-java-and-base-os"></a>Principios para mantener en buen estado Java y el sistema operativo base
* Debe ser el mismo sistema operativo base en los distintos niveles: Básico | Estándar | Premium.
    * Actualmente, las aplicaciones de Azure Spring Cloud usan una combinación de Debian 10 y Ubuntu 18.04.
    * El servicio de compilación de VMware usa Ubuntu 18.04.
* Debe ser el mismo sistema operativo base independientemente de los puntos de inicio de la implementación: origen | JAR
    * Actualmente, las aplicaciones de Azure Spring Cloud usan una combinación de Debian 10 y Ubuntu 18.04.
* El sistema operativo base no debe contener vulnerabilidades de seguridad.
    * El sistema operativo base de Debian 10 tiene 147 CVE abiertos.
    * El sistema operativo base Ubuntu 18.04 tiene 132 CVE abiertos.
* Debe usar JRE sin periféricos.
    * Actualmente, las aplicaciones de Azure Spring Cloud usan JDK. JRE sin periféricos es una imagen más pequeña.
* Debe usar las compilaciones más recientes de Java.
    * Actualmente, las aplicaciones de Azure Spring Cloud usan Java 8, compilación 242. Se trata de una compilación no actualizada.
 
Azul Systems explorará continuamente los cambios en los sistemas operativos base y mantendrá actualizadas las últimas imágenes compiladas. Azure Spring Cloud busca cambios en las imágenes y las actualiza continuamente entre las implementaciones.
 
## <a name="faq-for-azure-spring-cloud"></a>Preguntas más frecuentes sobre Azure Spring Cloud

* ¿Qué versiones de Java se admiten? Versión principal y número de compilación.
    * Compatibilidad con versiones de LTS: Java 8 y 11.
    * Usa la compilación más reciente (por ejemplo, ahora, Java 8, compilación 252 y Java 11, compilación 7).
* ¿Quién ha compilado estos runtime de Java?
    * Azul Systems.
* ¿Cuál es el sistema operativo base para las imágenes?
    * Ubuntu 20.04 LTS (Focal Fossa). Las aplicaciones seguirán estando en la versión LTS más reciente de Ubuntu.
    * Consulte [Ubuntu 20.04 LTS (Focal Fossa)](http://releases.ubuntu.com/focal/)
* ¿Cómo puedo descargar un runtime de Java compatible para el desarrollo local? 
    * Consulte [Instalación del JDK para Azure y Azure Stack](https://docs.microsoft.com/azure/developer/java/fundamentals/java-jdk-install).
* ¿Cómo puedo obtener soporte técnico para problemas en el nivel de runtime de Java?
    * Abra una incidencia de soporte técnico con el departamento de Soporte técnico de Azure.
 
## <a name="default-deployment-on-azure-spring-cloud"></a>Implementación predeterminada en Azure Spring Cloud

> ![Implementación predeterminada](media/spring-cloud-principles/spring-cloud-default-deployment.png)
 
## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Implementación de la primera aplicación de Azure Spring Cloud](spring-cloud-quickstart.md)
* [Soporte técnico de Java a largo plazo para Azure y Azure Stack](https://docs.microsoft.com/azure/developer/java/fundamentals/java-jdk-long-term-support)
