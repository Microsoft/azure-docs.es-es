---
title: Obtención de información acerca de los últimos lanzamientos de SO invitado de Azure | Microsoft Docs
description: Noticias sobre los lanzamientos más recientes y compatibilidad con el SO invitado de Azure Cloud Services.
services: cloud-services
documentationcenter: na
author: yohaddad
editor: ''
ms.assetid: 6306cafe-1153-44c7-8554-623b03d59a34
ms.service: cloud-services
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 8/17/2020
ms.author: yohaddad
ms.openlocfilehash: 7ac7fd67032c49ca07a2a050b72dce199cfebf54
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88548504"
---
# <a name="azure-guest-os-releases-and-sdk-compatibility-matrix"></a>Matriz de compatibilidad del SDK y versiones del SO invitado de Azure
Proporciona información actualizada sobre los lanzamientos del SO invitado de Azure más recientes para Cloud Services. Esta información le ayuda a planear la ruta de actualización antes de que se deshabilite un SO invitado. Si configura los roles para utilizar actualizaciones *automáticas* del SO invitado como se describe en la documentación sobre la [configuración de actualización del SO invitado de Azure][Azure Guest OS Update Settings], no es fundamental que lea esta página.

> [!IMPORTANT]
> Esta página se aplica a los roles web y de trabajo de Cloud Services, que se ejecutan en un sistema operativo invitado. **No se aplica** a las máquinas virtuales de IaaS.
>
>


> [!TIP]
>  Suscríbase a la [Fuente RSS de actualización del SO invitado] para recibir notificaciones puntuales sobre todos los cambios del sistema operativo invitado.
>
>

> [!IMPORTANT]
> Solo las 2 últimas versiones del SO invitado serán compatibles y estarán disponibles en Azure Portal.
>
>

¿No está seguro de cómo actualizar el SO invitado? Vea [esto][cloud updates].

## <a name="news-updates"></a>Actualizaciones de noticias

###### <a name="august-17-2020"></a>**17 de agosto de 2020**
Se ha publicado el SO invitado de julio. 

###### <a name="august-10-2020"></a>**10 de agosto de 2020**
Se ha publicado el SO invitado de junio. 

###### <a name="june-2-2020"></a>**2 de junio de 2020**
Se ha publicado el SO invitado de mayo. 

###### <a name="may-4-2020"></a>**4 de mayo de 2020**
Se ha publicado el sistema operativo invitado de abril. 

###### <a name="april-2-2020"></a>**2 de abril de 2020**
Se ha publicado el sistema operativo invitado de marzo. 

###### <a name="march-5-2020"></a>**5 de marzo de 2020**
Se ha publicado el SO invitado de febrero. 

###### <a name="january-24-2020"></a>**24 de enero de 2020**
Se ha publicado el SO invitado de enero. 

###### <a name="january-8-2020"></a>**8 de enero de 2020**
Se ha publicado el sistema operativo invitado de diciembre.

###### <a name="december-5-2019"></a>**5 de diciembre de 2019**
Se ha publicado el sistema operativo invitado de noviembre.

###### <a name="november-1-2019"></a>**1 de noviembre de 2019**
Se ha publicado el SO invitado de octubre.

###### <a name="october-7-2019"></a>**7 de octubre de 2019**
Se ha publicado el SO invitado de septiembre.

###### <a name="september-4-2019"></a>**4 de septiembre de 2019**
Se ha publicado el SO invitado de agosto.

###### <a name="july-26-2019"></a>**26 de julio de 2019**
Se ha publicado el SO invitado de julio.

###### <a name="july-8-2019"></a>**8 de julio de 2019**
Se ha publicado el SO invitado de junio.

###### <a name="june-6-2019"></a>**6 de junio de 2019**
Se ha publicado el SO invitado de mayo.

###### <a name="may-7-2019"></a>**7 de mayo de 2019**
Se ha publicado el sistema operativo invitado de abril.

###### <a name="march-26-2019"></a>**26 de marzo de 2019**
Se ha publicado el sistema operativo invitado de marzo.

###### <a name="march-12-2019"></a>**12 de marzo de 2019**
Se ha publicado el SO invitado de febrero.

###### <a name="february-5-2019"></a>**5 de febrero de 2019**
Se ha publicado el SO invitado de enero.

###### <a name="january-24-2019"></a>**24 de enero de 2019**
Se ha publicado familia 6 del sistema operativo invitado (Windows Server 2019).

###### <a name="january-7-2019"></a>**7 de enero de 2019**
Se ha publicado el sistema operativo invitado de diciembre.

###### <a name="december-14-2018"></a>**14 de diciembre de 2018**
Se ha publicado el sistema operativo invitado de noviembre.

###### <a name="november-8-2018"></a>**8 de noviembre de 2018**
Se ha publicado el SO invitado de octubre.

###### <a name="october-12-2018"></a>**12 de octubre de 2018**
Se ha publicado el SO invitado de septiembre.

## <a name="releases"></a>Lanzamientos

## <a name="family-6-releases"></a>Lanzamientos de la familia 6
**Windows Server 2019**

Versión de .NET Framework instalada: 3.5, 4.7.2

> [!NOTE]
> El SDK de Microsoft Azure para .NET 3.0 puede descargarse [aquí][Windows Azure SDK].
>
>Pasos para la instalación:
>1. Desinstale cualquier versión anterior de MicrosoftAzureAuthoringTools*.msi
>2. Instalar el [SDK de Azure para .NET 3.0][Windows Azure SDK]
>3. Reinicie la máquina
>4. Cree un nuevo proyecto de servicio en la nube y agregue un único rol de trabajo
>5. Cambie la familia del SO a 6 y compile un paquete
>6. Implemente el paquete en Azure mediante Azure Portal o Visual Studio
>
>La versión 6 del SO invitado aplica TLS 1.2 mediante la deshabilitación explícita de TLS 1.0 y 1.1, y la definición de una serie específica de conjuntos de cifrado. [Más información].


| Cadena de configuración | Fecha de la versión | Fecha de deshabilitación |
| --- | --- | --- |
|  WA-GUEST-OS-6.21_202007-01  |  17 de agosto de 2020  |  Post 6.23  |  
|  WA-GUEST-OS-6.20_202006-02  |  10 de agosto de 2020  |  Post 6.22  |  
|~~WA-GUEST-OS-6.19_202005-02~~|  2 de junio de 2020  |  17 de agosto de 2020  |  
|~~WA-GUEST-OS-6.18_202004-01~~|  4 de mayo de 2020  |  10 de agosto de 2020  |  
|~~WA-GUEST-OS-6.17_202003-01~~|  2 de abril de 2020  |  2 de junio de 2020  |  
|~~WA-GUEST-OS-6.16_202002-01~~|  5 de marzo de 2020  |  4 de mayo de 2020  |  
|~~WA-GUEST-OS-6.15_202001-01~~|  24 de enero de 2020  |  2 de abril de 2020  |  
|~~WA-GUEST-OS-6.14_201912-01~~| 8 de enero de 2020 | 5 de marzo de 2020 |  
|~~WA-GUEST-OS-6.13_201911-01~~| 5 de diciembre de 2019 | 24 de enero de 2020 |  
|~~WA-GUEST-OS-6.12_201910-01~~| 1 de noviembre de 2019 | 8 de enero de 2020 |  
|~~WA-GUEST-OS-6.11_201909-01~~| 7 de octubre de 2019 | 5 de diciembre de 2019 |  
|~~WA-GUEST-OS-6.10_201908-01~~| 4 de agosto de 2019 | 1 de noviembre de 2019  |  
|~~WA-GUEST-OS-6.9_201907-0~~|26 de julio de 2019 | 7 de octubre de 2019 |
|~~WA-GUEST-OS-6.8_201906-01~~|8 de julio de 2019 |4 de agosto de 2019 |
|~~WA-GUEST-OS-6.7_201905-01~~ |6 de junio de 2019 |26 de julio de 2019 |
|~~WA-GUEST-OS-6.6_201904-01~~ |7 de mayo de 2019 |8 de julio de 2019 |
|~~WA-GUEST-OS-6.5_201903-01~~ |26 de marzo de 2019 |6 de junio de 2019 |
|~~WA-GUEST-OS-6.4_201902-01~~ |12 de marzo de 2019 |7 de mayo de 2019 |
|~~WA-GUEST-OS-6.3_201901-01~~ |5 de febrero de 2019 |26 de marzo de 2019 |
|~~WA-GUEST-OS-6.2_201812-01~~ |24 de enero de 2019 |12 de marzo de 2019 |
|~~WA-GUEST-OS-6.1_201811-01~~ |24 de enero de 2019 |5 de febrero de 2019 |

## <a name="family-5-releases"></a>Lanzamientos de la familia 5
**Windows Server 2016**

Versión de .NET Framework instalada: 3.5, 4.6.2

> [!NOTE]
> La contraseña de RDP de la familia 5 del SO debe tener 10 caracteres como mínimo.
>


| Cadena de configuración | Fecha de la versión | Fecha de deshabilitación |
| --- | --- | --- |
|  WA-GUEST-OS-5.45_202007-01  |  17 de agosto de 2020  |  Post 5.47  |  
|  WA-GUEST-OS-5.44_202006-02  |  10 de agosto de 2020  |  Post 5.46  |  
|~~WA-GUEST-OS-5.43_202005-02~~|  2 de junio de 2020  |  17 de agosto de 2020  |  
|~~WA-GUEST-OS-5.42_202004-01~~|  4 de mayo de 2020  |  10 de agosto de 2020  |  
|~~WA-GUEST-OS-5.41_202003-01~~|  2 de abril de 2020  |  2 de junio de 2020  |  
|~~WA-GUEST-OS-5.40_202002-01~~|  5 de marzo de 2020  |  4 de mayo de 2020  |  
|~~WA-GUEST-OS-5.39_202001-01~~|  24 de enero de 2020  |  2 de abril de 2020  |  
|~~WA-GUEST-OS-5.38_201912-01~~| 8 de enero de 2020 | 5 de marzo de 2020 |  
|~~WA-GUEST-OS-5.37_201911-01~~| 5 de diciembre de 2019 | 24 de enero de 2020 |  
|~~WA-GUEST-OS-5.36_201910-01~~| 1 de noviembre de 2019 | 8 de enero de 2020 |  
|~~WA-GUEST-OS-5.35_201909-01~~| 7 de octubre de 2019 | 5 de diciembre de 2019 |  
|~~WA-GUEST-OS-5.34_201908-01~~|  4 de agosto de 2019  | 1 de noviembre de 2019 |  
|~~WA-GUEST-OS-5.33_201907-01~~| 26 de julio de 2019 | 7 de octubre de 2019 |  
|~~WA-GUEST-OS-5.32_201906-01~~|8 de julio de 2019 |4 de agosto de 2019 |
|~~WA-GUEST-OS-5.31_201905-01~~ |6 de junio de 2019 |26 de julio de 2019 |
|~~WA-GUEST-OS-5.30_201904-01~~ |7 de mayo de 2019 |8 de julio de 2019 |
|~~WA-GUEST-OS-5.29_201903-01~~ |26 de marzo de 2019 |6 de junio de 2019 |
|~~WA-GUEST-OS-5.28_201902-01~~ |12 de marzo de 2019 |7 de mayo de 2019 |
|~~WA-GUEST-OS-5.27_201901-01~~ |5 de febrero de 2019 |26 de marzo de 2019 |
|~~WA-GUEST-OS-5.26_201812-01~~ |7 de enero de 2019 |12 de marzo de 2019 |
|~~WA-GUEST-OS-5.25_201811-01~~ |14 de diciembre de 2018 |5 de febrero de 2019 |
|~~WA-GUEST-OS-5.24_201810-01~~ |8 de noviembre de 2018 |7 de enero de 2019 |
|~~WA-GUEST-OS-5.23_201809-01~~ |12 de octubre de 2018 |14 de diciembre de 2018 |

## <a name="family-4-releases"></a>Lanzamientos de la familia 4
**Windows Server 2012 R2**

Versión de .NET Framework instalada: 3.5, 4.5.1, 4.5.2

| Cadena de configuración | Fecha de la versión | Fecha de deshabilitación |
| --- | --- | --- |
|  WA-GUEST-OS-4.80_202007-01  |  17 de agosto de 2020  |  Post 4.82  | 
|  WA-GUEST-OS-4.79_202006-02  |  10 de agosto de 2020  |  Post 4.81  | 
|~~WA-GUEST-OS-4.78_202005-02~~|  2 de junio de 2020  |  17 de agosto de 2020  |  
|~~WA-GUEST-OS-4.77_202004-01~~|  4 de mayo de 2020  |  10 de agosto de 2020  |  
|~~WA-GUEST-OS-4.76_202003-01~~|  2 de abril de 2020  |  2 de junio de 2020  |  
|~~WA-GUEST-OS-4.75_202002-01~~|  5 de marzo de 2020  |  4 de mayo de 2020  |  
|~~WA-GUEST-OS-4.74_202001-01~~|  24 de enero de 2020  |  2 de abril de 2020  |  
|~~WA-GUEST-OS-4.73_201912-01~~| 8 de enero de 2020 | 5 de marzo de 2020 |  
|~~WA-GUEST-OS-4.72_201911-01~~| 5 de diciembre de 2019 | 24 de enero de 2020 |  
|~~WA-GUEST-OS-4.71_201910-01~~| 1 de noviembre de 2019 | 8 de enero de 2020 |  
|~~WA-GUEST-OS-4.70_201909-01~~| 7 de octubre de 2019 | 5 de diciembre de 2019 |  
|~~WA-GUEST-OS-4.69_201908-01~~| 4 de agosto de 2019 | 1 de noviembre de 2019 |  
|~~WA-GUEST-OS-4.68_201907-01~~| 26 de julio de 2019  | 7 de octubre de 2019 |
|~~WA-GUEST-OS-4.67_201906-01~~| 8 de julio de 2019 |4 de agosto de 2019 |
|~~WA-GUEST-OS-4.66_201905-01~~ |6 de junio de 2019 |26 de julio de 2019 |
|~~WA-GUEST-OS-4.65_201904-01~~ |7 de mayo de 2019 |8 de julio de 2019 |
|~~WA-GUEST-OS-4.64_201903-01~~ |26 de marzo de 2019 |6 de junio de 2019 |
|~~WA-GUEST-OS-4.63_201902-01~~ |12 de marzo de 2019 |7 de mayo de 2019 |
|~~WA-GUEST-OS-4.62_201901-01~~ |5 de febrero de 2019 |26 de marzo de 2019 |
|~~WA-GUEST-OS-4.61_201812-01~~ |7 de enero de 2019 |12 de marzo de 2019 |
|~~WA-GUEST-OS-4.60_201811-01~~ |14 de diciembre de 2018 |5 de febrero de 2019 |
|~~WA-GUEST-OS-4.59_201810-01~~ |8 de noviembre de 2018 |7 de enero de 2019 |
|~~WA-GUEST-OS-4.58_201809-01~~ |12 de octubre de 2018 |14 de diciembre de 2018 |

## <a name="family-3-releases"></a>Lanzamientos de la familia 3
**Windows Server 2012**

Versión de .NET Framework instalada: 3.5, 4.5

| Cadena de configuración | Fecha de la versión | Fecha de deshabilitación |
| --- | --- | --- |
|  WA-GUEST-OS-3.87_202007-01  |  17 de agosto de 2020  |  Post 3.89  |  
|  WA-GUEST-OS-3.86_202006-02  |  10 de agosto de 2020  |  Post 3.88  |  
|~~WA-GUEST-OS-3.85_202005-02~~|  2 de junio de 2020  |  17 de agosto de 2020  |  
|~~WA-GUEST-OS-3.84_202004-01~~|  4 de mayo de 2020  |  10 de agosto de 2020  |  
|~~WA-GUEST-OS-3.83_202003-01~~|  2 de abril de 2020  |  2 de junio de 2020  |  
|~~WA-GUEST-OS-3.82_202002-01~~|  5 de marzo de 2020  |  4 de mayo de 2020  |  
|~~WA-GUEST-OS-3.81_202001-01~~|  24 de enero de 2020  |  2 de abril de 2020  |  
|~~WA-GUEST-OS-3.80_201912-01~~| 8 de enero de 2020 | 5 de marzo de 2020 |  
|~~WA-GUEST-OS-3.79_201911-01~~| 5 de diciembre de 2019 | 24 de enero de 2020 |  
|~~WA-GUEST-OS-3.78_201910-01~~| 1 de noviembre de 2019 | 8 de enero de 2020 |  
|~~WA-GUEST-OS-3.77_201909-01~~| 7 de octubre de 2019 | 5 de diciembre de 2019 |  
|~~WA-GUEST-OS-3.76_201908-01~~|  4 de agosto de 2019  |  1 de noviembre de 2019  |  
|~~WA-GUEST-OS-3.75_201907-01~~| 26 de julio de 2019 | 7 de octubre de 2019 |
|~~WA-GUEST-OS-3.74_201906-01~~| 8 de julio de 2019 |4 de agosto de 2019 |
|~~WA-GUEST-OS-3.73_201905-01~~ |6 de junio de 2019 |26 de julio de 2019 |
|~~WA-GUEST-OS-3.72_201904-01~~ |7 de mayo de 2019 |8 de julio de 2019 |
|~~WA-GUEST-OS-3.71_201903-01~~ |26 de marzo de 2019 |6 de junio de 2019 |
|~~WA-GUEST-OS-3.70_201902-01~~ |12 de marzo de 2019 |7 de mayo de 2019 |
|~~WA-GUEST-OS-3.69_201901-01~~ |5 de febrero de 2019 |26 de marzo de 2019 |
|~~WA-GUEST-OS-3.68_201812-01~~ |7 de enero de 2019 |12 de marzo de 2019 |
|~~WA-GUEST-OS-3.67_201811-01~~ |14 de diciembre de 2018 |5 de febrero de 2019 |
|~~WA-GUEST-OS-3.66_201810-01~~ |8 de noviembre de 2018 |7 de enero de 2019 |
|~~WA-GUEST-OS-3.65_201809-01~~ |12 de octubre de 2018 |14 de diciembre de 2018 |

## <a name="family-2-releases"></a>Lanzamientos de la familia 2
**Windows Server 2008 R2 SP1**

Versión de .NET Framework instalada: 3.5 (incluye 2.0 y 3.0), 4.5

| Cadena de configuración | Fecha de la versión | Fecha de deshabilitación |
| --- | --- | --- |
|  WA-GUEST-OS-2.100_202007-01  |  17 de agosto de 2020  |  Post 2.102  |  
|  WA-GUEST-OS-2.99_202006-02  |  10 de agosto de 2020  |  Post 2.101  |  
|~~WA-GUEST-OS-2.98_202005-02~~|  2 de junio de 2020  |  17 de agosto de 2020  |  
|~~WA-GUEST-OS-2.97_202004-01~~|  4 de mayo de 2020  |  10 de agosto de 2020  |  
|~~WA-GUEST-OS-2.96_202003-01~~|  2 de abril de 2020  |  2 de junio de 2020  |  
|~~WA-GUEST-OS-2.95_202002-01~~|  5 de marzo de 2020  |  4 de mayo de 2020  |  
|~~WA-GUEST-OS-2.94_202001-01~~|  24 de enero de 2020  |  2 de abril de 2020  |  
|~~WA-GUEST-OS-2.93_201912-01~~| 8 de enero de 2020 | 5 de marzo de 2020 |  
|~~WA-GUEST-OS-2.92_201911-01~~| 5 de diciembre de 2019 | 24 de enero de 2020 |  
|~~WA-GUEST-OS-2.91_201910-01~~| 1 de noviembre de 2019 | 8 de enero de 2020 |  
|~~WA-GUEST-OS-2.90_201909-01~~| 7 de octubre de 2019 | 5 de diciembre de 2019 |  
|~~WA-GUEST-OS-2.89_201908-01~~| 4 de agosto de 2019 | 1 de noviembre de 2019 |  
|~~WA-GUEST-OS-2.88_201907-01~~| 26 de julio de 2019 | 7 de octubre de 2019 |
|~~WA-GUEST-OS-2.87_201906-01~~|8 de julio de 2019 | 4 de agosto de 2019 |
|~~WA-GUEST-OS-2.86_201905-01~~ |6 de junio de 2019 |26 de julio de 2019 |
|~~WA-GUEST-OS-2.85_201904-01~~ |7 de mayo de 2019 |8 de julio de 2019 |
|~~WA-GUEST-OS-2.84_201903-01~~ |26 de marzo de 2019 |6 de junio de 2019 |
|~~WA-GUEST-OS-2.83_201902-01~~ |12 de marzo de 2019 |7 de mayo de 2019 |
|~~WA-GUEST-OS-2.82_201901-01~~ |5 de febrero de 2019 |26 de marzo de 2019 |
|~~WA-GUEST-OS-2.81_201812-01~~ |7 de enero de 2019 |12 de marzo de 2019 |
|~~WA-GUEST-OS-2.80_201811-01~~ |14 de diciembre de 2018 |5 de febrero de 2019 |
|~~WA-GUEST-OS-2.79_201810-01~~ |8 de noviembre de 2018 |7 de enero de 2019 |
|~~WA-GUEST-OS-2.78_201809-01~~ |12 de octubre de 2018 |14 de diciembre de 2018 |

## <a name="msrc-patch-updates"></a>Actualizaciones de revisiones de MSRC
La lista de revisiones que se incluyen con cada lanzamiento mensual del SO invitado está disponible [aquí][patches].

## <a name="sdk-support"></a>Compatibilidad con SDK
Aunque la [directiva de retirada de Azure SDK][retire policy sdk] indica que solo se admiten las versiones posteriores a la 2.2, determinadas familias de SO invitado permiten usar versiones anteriores. Siempre debe usar el SDK compatible más reciente.

| Familia del SO invitado | Versiones del SDK compatibles |
| --- | --- |
| 6 |Versión 2.9.6+ |
| 5 |Versión 2.9.5.1+ |
| 4 |Versión 2.1 o superior |
| 3 |Versión 1.8 o superior |
| 2 |Versión 1.3 o superior |
| 1 |Versión 1.0 o superior |

## <a name="guest-os-release-information"></a>Información de versión del SO invitado
Hay tres fechas importantes para los lanzamientos de SO invitado: la de **lanzamiento**, la de **deshabilitación** y la de **expiración**. Se considera que un SO invitado se encuentra disponible cuando figura en el Portal y se puede seleccionar como el SO invitado de destino. Cuando un SO invitado llega a su fecha de **deshabilitación**, se quita de Azure, aunque todos los servicios en la nube que usen dicho SO invitado seguirán funcionando con normalidad.

El período entre las fechas de **deshabilitación** y de **expiración** permite llevar a cabo la transición de un SO invitado a uno más reciente con tranquilidad. Si usa la opción *automática* para el SO invitado, siempre tendrá la versión más reciente, por lo que no tendrá que preocuparse de que expire.

Una vez que pase la fecha de **expiración**, cualquier servicio en la nube que siga usando ese SO invitado se detendrá, eliminará o actualizará de forma obligada. Puede obtener más información sobre la directiva de retirada [aquí][retirepolicy].

## <a name="guest-os-family-version-explanation"></a>Diferencias entre la versión y la familia del SO invitado
Las familias del SO invitado se basan en versiones lanzadas de Microsoft Windows Server. El SO invitado es el sistema operativo subyacente en el que se ejecuta Azure Cloud Services. Cada SO invitado tiene un número de familia, versión y lanzamiento.

* **Guest OS family**  
  Versión del sistema operativo Windows Server en la que se basa un SO invitado. Por ejemplo, la *familia 3* se basa en Windows Server 2012.
* **Versión del SO invitado**  
  Corresponde a la imagen de la familia del SO invitado, más las revisiones pertinentes del [Centro de respuestas de seguridad de Microsoft (MSRC)][msrc] disponibles en la fecha en que se genera la nueva versión del SO invitado. Es posible que no se incluyan todas las revisiones.

    Los números empiezan por 0 y se incrementan en 1 cada vez que se agrega un nuevo conjunto de actualizaciones. Solo se muestran los ceros finales si es importante. Es decir, la versión 2.10 es una versión mucho más posterior y diferente que la versión 2.1.
* **Lanzamiento del SO invitado**  
  Relanzamiento de una versión de SO invitado. Un relanzamiento se produce si Microsoft encuentra, durante las pruebas, problemas que requieran cambios. El lanzamiento más reciente siempre reemplaza a los lanzamientos anteriores, ya sean públicos o no. Azure Portal solo permitirá a los usuarios elegir el lanzamiento más reciente de una versión concreta. Las implementaciones que se ejecutan en un lanzamiento anterior no suelen ser una actualización forzada, según la gravedad del error.

En el ejemplo siguiente, 2 es la familia, 12 es la versión y "rel2" es el lanzamiento.

**Lanzamiento del SO invitado** : 2.12 rel2

**Cadena de configuración para este lanzamiento** - WA-GUEST-OS-2.12_201208-02

La cadena de configuración para un SO invitado incluye esta misma información incrustada, junto con una fecha que muestra qué revisiones de MSRC se tuvieron en cuenta para ese lanzamiento. En este ejemplo, las revisiones de MSRC generadas para Windows Server 2008 R2 hasta e incluido agosto de 2012 se tuvieron en cuenta para su inclusión. Solo se incluyen las revisiones que se aplican específicamente a esa versión de Windows Server. Por ejemplo, si una revisión de MSRC se aplica a Microsoft Office, no se incluirá porque este producto no forma parte de la imagen base de Windows Server.

## <a name="guest-os-system-update-process"></a>Proceso de actualización del sistema del SO invitado
Esta página incluye información sobre los próximos lanzamientos del SO invitado. Los clientes han indicado que desean saber cuándo se produce un lanzamiento porque sus roles de servicio en la nube se reiniciarán si están establecidos en actualización "Automática". Las versiones del SO invitado suelen producirse de 2 a 3 semanas después de que tenga lugar la actualización de MSRC el segundo martes de cada mes. Los nuevos lanzamientos incluyen todas las revisiones de MSRC relevantes para cada familia del SO invitado.

Microsoft Azure publica actualizaciones constantemente. El SO invitado es solo una de estas actualizaciones en la canalización. Un lanzamiento puede verse afectado por una cantidad de factores tal que no se puede enumerar aquí. Además, Azure se ejecuta literalmente en cientos de miles de máquinas. Esto significa que es imposible especificar una fecha y hora exacta de reinicio de los roles. Estamos trabajando en un plan para limitar o acotar los reinicios.

Cuando se publica un nuevo lanzamiento del SO invitado, puede tardar tiempo en propagarse completamente en Azure. A medida que los servicios se actualizan al nuevo SO invitado, estos se reinician respetando los dominios de actualización. Los servicios configurados para usar actualizaciones "Automáticas" serán los primeros en obtener el lanzamiento. Después de la actualización, verá la nueva versión del SO invitado del servicio en Azure Portal. Durante ese período se pueden producir relanzamientos. Algunas versiones se pueden implementar durante largos períodos de tiempo y puede que no se produzcan actualizaciones reinicios de actualizaciones automáticas durante muchas semanas después de la fecha de lanzamiento oficial. Una vez que un SO invitado está disponible, puede elegir esa versión explícitamente desde el portal o en el archivo de configuración.

Para obtener una gran cantidad de información valiosa sobre los reinicios y punteros a más información técnica detallada de las actualizaciones de SO invitado y host, vea la entrada de blog de MSDN titulada [Reinicios de instancias de rol debido a actualizaciones del SO][restarts].

Si actualiza manualmente el SO invitado, vea la [directiva de retirada del SO invitado][retirepolicy] para obtener más información.

## <a name="guest-os-supportability-and-retirement-policy"></a>Directiva de compatibilidad y retirada del SO invitado
La directiva de compatibilidad y retirada del SO invitado se explica [aquí][retirepolicy].

[cloud updates]: https://docs.microsoft.com/azure/cloud-services/cloud-services-update-azure-service
[Fuente RSS de actualización del SO invitado]: https://raw.githubusercontent.com/MicrosoftDocs/azure-cloud-services-files/master/GuestOS/GuestOSFeed.xml
[Install .NET on a Cloud Service Role]: https://azure.microsoft.com/documentation/articles/cloud-services-dotnet-install-dotnet/?WT.mc_id=azurebg_email_Trans_963_RevisedNET_Update
[Azure Guest OS Update Settings]: cloud-services-how-to-configure-portal.md
[ssl3 announcement]: https://azure.microsoft.com/blog/2014/12/09/azure-security-ssl-3-0-update/
[Microsoft Security Advisory 3009008]: https://technet.microsoft.com/library/security/3009008.aspx
[ssl3-fixit]: https://go.microsoft.com/?linkid=9863266
[MS14-066]: https://technet.microsoft.com/library/security/ms14-066.aspx
[MS14-046]: https://technet.microsoft.com/library/security/ms14-046.aspx
[retire policy sdk]: https://msdn.microsoft.com/library/dn479282.aspx
[server and gos]: https://msdn.microsoft.com/library/dn775043.aspx
[azuresupport]: https://azure.microsoft.com/support/options/
[net install pkg]: https://www.microsoft.com/download/details.aspx?id=42643
[msrc]: https://technet.microsoft.com/security/dn440717.aspx
[update guest os portal]: https://msdn.microsoft.com/library/gg433101.aspx
[update guest os svc]: https://msdn.microsoft.com/library/gg456324.aspx
[restarts]: https://blogs.msdn.com/b/kwill/archive/2012/09/19/role-instance-restarts-due-to-os-upgrades.aspx
[patches]: cloud-services-guestos-msrc-releases.md
[retirepolicy]: cloud-services-guestos-retirement-policy.md
[fam1retire]: cloud-services-guestos-family1-retirement.md
[fix]: https://technet.microsoft.com/library/security/ms17-010.aspx
[Windows Azure SDK]: https://www.microsoft.com/en-us/download/details.aspx?id=54917
[Más información]: https://docs.microsoft.com/azure/cloud-services/applications-dont-support-tls-1-2  
