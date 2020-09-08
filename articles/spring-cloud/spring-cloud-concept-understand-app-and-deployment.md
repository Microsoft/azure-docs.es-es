---
title: Descripción de la aplicación y la implementación en Azure Spring Cloud
description: Diferencia entre la aplicación y la implementación en Azure Spring Cloud
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 07/23/2020
ms.custom: devx-track-java
ms.openlocfilehash: 81e1925810f374da6f02bf6c3a013b00b5bb9a2c
ms.sourcegitcommit: 64ad2c8effa70506591b88abaa8836d64621e166
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88263860"
---
# <a name="understand-app-and-deployment-in-azure-spring-cloud"></a>Descripción de la aplicación y la implementación en Azure Spring Cloud

**Aplicación** e **implementación** son los dos conceptos clave del modelo de recursos de Azure Spring Cloud. En Azure Spring Cloud, una *aplicación* es una abstracción de una aplicación empresarial o de un microservicio.  Una versión de código o binaria implementada como la *aplicación* se ejecuta en una *implementación*.

 ![Aplicaciones e implementaciones](./media/spring-cloud-app-and-deployment/app-deployment-rev.png)

El nivel estándar de Azure Spring Cloud permite que una aplicación tenga una implementación de producción y una implementación de almacenamiento provisional, de modo que pueda realizar una implementación azul/verde en ella fácilmente.

## <a name="app"></a>Aplicación
Las siguientes características o propiedades se definen en el nivel de aplicación.

| Enum | Definición |
|:--|:----------------|
| Público</br>Punto de conexión | Dirección URL para acceder a la aplicación |
| Personalizado</br>Domain | Registro CNAME que protege el dominio personalizado |
| Servicio</br>Enlace | Enlazar las propiedades de configuración establecidas en el archivo function.json y el atributo *ServiceBusTrigger* |
| Administrado</br>Identidad | La identidad administrada de Azure Active Directory permite a la aplicación acceder fácilmente a otros recursos protegidos por Azure AD, como Azure Key Vault |
| Persistente</br>Storage | Configuración que permite que los datos se conserven más allá del reinicio de la aplicación |

## <a name="deployment"></a>Implementación

Las siguientes características o propiedades se definen en el nivel de implementación y se intercambiarán al cambiar la implementación de producción o de almacenamiento provisional.

| Enum | Definición |
|:--|:----------------|
| CPU | Número de núcleos virtuales por instancia de aplicación |
| Memoria | Configuración que asigna memoria para escalar vertical u horizontalmente las implementaciones |
| Instancia</br>Count | El número de instancias de aplicación, establecido manual o automáticamente |
| Escalado automático | Escalado automático del recuento de instancias en función de reglas y programaciones predefinidas |
| JVM</br>Opciones | Configuración: JAVA_OPTS |
| Entorno</br>variables | Configuración aplicable al entorno de Azure Spring Cloud |
| Tiempo de ejecución</br>Versión | Java 8/Java 11|

## <a name="restrictions"></a>Restricciones

* **Una aplicación debe tener una implementación de producción**: la API bloquea la eliminación de una implementación de producción. Se debe cambiar al almacenamiento provisional antes de la eliminación.
* **Una aplicación puede tener dos implementaciones como máximo**: la API bloquea la creación de más de dos implementaciones. Implemente el nuevo archivo binario en la implementación de producción o de almacenamiento provisional.
* **La administración de implementación no está disponible en el nivel Básico**: use el nivel Estándar para la funcionalidad de implementación azul-verde.

## <a name="see-also"></a>Consulte también
* [Configuración de un entorno de ensayo en Azure Spring Cloud](spring-cloud-howto-staging-environment.md)