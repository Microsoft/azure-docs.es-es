---
title: Preguntas más frecuentes
description: Preguntas frecuentes sobre el servicio Azure Spatial Anchors.
author: ramonarguelles
manager: vriveras
services: azure-spatial-anchors
ms.author: rgarcia
ms.date: 05/18/2020
ms.topic: overview
ms.service: azure-spatial-anchors
ms.openlocfilehash: 9457836a3048ff08b228bf9d5b84400af15d6cc5
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88854942"
---
# <a name="frequently-asked-questions-about-azure-spatial-anchors"></a>Preguntas más frecuentes sobre Azure Spatial Anchors

Azure Spatial Anchors es una plataforma de desarrollo y un servicio en la nube administrado que permite disfrutar de experiencias de realidad mixta multiusuario y con identificación del espacio en varios dispositivos Android, iOS y HoloLens.

Para más información, consulte [Introducción a Azure Spatial Anchors](overview.md).

## <a name="azure-spatial-anchors-product-faqs"></a>Preguntas más frecuentes sobre el producto Azure Spatial Anchors

**P: ¿Qué dispositivos admite Azure Spatial Anchors?**

**R:** Azure Spatial Anchors permite a los desarrolladores compilar aplicaciones en dispositivos HoloLens, dispositivos iOS compatibles con ARKit y dispositivos Android compatibles con ARCore; para iOS y Android, esto incluye teléfonos y tabletas.

**P: ¿Es necesario estar conectado a la nube para usar Azure Spatial Anchors?**

**R:** Actualmente, Azure Spatial Anchors requiere una conexión de red a Internet. Agradecemos sus comentarios en nuestro [sitio de comentarios](https://feedback.azure.com/forums/919252-azure-spatial-anchors).

**P: ¿Cuáles son los requisitos de conectividad para Azure Spatial Anchors?**

**R:** Azure Spatial Anchors funciona con conexiones de banda ancha móviles y Wi-Fi.

**P: ¿Con qué exactitud localiza Azure Spatial Anchors los delimitadores?**

**R:** Muchos factores afectan a la exactitud de la localización de los delimitadores: la iluminación, los objetos del entorno e, incluso, la superficie en la que está colocado el delimitador. Para determinar si la exactitud se ajusta a sus necesidades, pruebe los delimitadores en entornos representativos donde tenga pensado usarlos. Si se encuentra con entornos en los que la precisión no se ajuste a sus necesidades, consulte [Registro y diagnóstico de Azure Spatial Anchors](./concepts/logging-diagnostics.md).

**P: ¿Cuánto tiempo se tarda en crear y localizar los delimitadores?**

**R:** El tiempo necesario para crear y localizar los delimitadores depende de muchos factores: la conexión de red, la carga y el procesamiento del dispositivo, y el entorno específico. Tenemos clientes que crean aplicaciones en muchos sectores, como la fabricación, la venta minorista y los juegos, que indican que el servicio ofrece una experiencia de usuario excelente en sus escenarios.

## <a name="privacy-faq"></a>Preguntas más frecuentes sobre privacidad

**P: Cuando mi aplicación coloca un delimitador espacial en un lugar, ¿todas las aplicaciones tienen acceso a él?**

**R:** Los delimitadores están aislados por la cuenta de Azure. Solo las aplicaciones con permiso de acceso a su cuenta podrán acceder a los delimitadores dentro de la cuenta.

**P: ¿Cómo almacena los datos Azure Spatial Anchors?**

**R:** Todos los datos se almacenan cifrados con una clave de cifrado de datos administrados de Microsoft y todos los datos se almacenan regionalmente para cada uno de los recursos.

**P: ¿Qué información acerca de un entorno se transmite y almacena en el servicio cuando se usa Azure Spatial Anchors? ¿Se transmiten y almacenan imágenes del entorno?**

**R.** : Al crear o localizar delimitadores, se procesan imágenes del entorno en el dispositivo en un formato derivado. Este formato derivado se transmite al servicio y se almacena en él.

Para proporcionar transparencia, la siguiente es una imagen de un entorno y la nube de puntos dispersos derivada. La nube de puntos muestra la representación geométrica del entorno que se transmite y almacena en el servicio. Por cada punto de la nube de puntos dispersos, se transmite y almacena un hash con las características visuales de ese punto. El hash se deriva de los datos de píxeles, pero no contiene ninguno de ellos.

Azure Spatial Anchors se adhiere a los [Términos del acuerdo de servicio de Azure](https://go.microsoft.com/fwLink/?LinkID=522330&amp;amp;clcid=0x9) y la [Declaración de privacidad de Microsoft](https://go.microsoft.com/fwlink/?LinkId=521839&amp;clcid=0x409).

![Un entorno y su nube de puntos dispersos derivada](./media/sparse-point-cloud.png)
*Figura 1: Un entorno y su nube de puntos dispersos derivada*

**P: ¿Hay alguna manera de enviar información de diagnóstico a Microsoft?**

**R.** : Sí. Azure Spatial Anchors tiene un modo de diagnóstico en el que los desarrolladores pueden elegir participar mediante la API de Azure Spatial Anchors. Esto es útil, por ejemplo, si encuentra un entorno donde no se puede crear y localizar delimitadores de forma predecible. Puede que le pidamos que envíe un informe de diagnóstico con información que nos ayudará a depurar el problema. Para más información, consulte [Registro y diagnóstico de Azure Spatial Anchors](./concepts/logging-diagnostics.md).

## <a name="availability-and-pricing-faqs"></a>Preguntas más frecuentes sobre disponibilidad y precios

**P: ¿Ofrece un Acuerdo de Nivel de Servicio?**

**R:** Como es norma para los servicios de Azure, nuestro objetivo es una disponibilidad superior al 99,9 %. 

**P: ¿Puedo publicar mis aplicaciones con Azure Spatial Anchors en tiendas de aplicaciones? ¿Puedo usar Azure Spatial Anchors en escenarios de producción de críticos para la misión?**

**R:** Sí, Azure Spatial Anchors está disponible con carácter general y tiene un Acuerdo de Nivel de Servicio de servicios de Azure estándar. Le invitamos a desarrollar aplicaciones para sus implementaciones de producción y [compartir sus comentarios](https://feedback.azure.com/forums/919252-azure-spatial-anchors) sobre el producto con nosotros.

**P: ¿Existen limitaciones?**

**R.** : Sí, hay limitaciones.  No esperamos que las alcance en un trabajo típico de desarrollo y prueba de aplicaciones. En el caso de las implementaciones en producción, estamos preparados para dar respaldo a las necesidades de gran escala de nuestros clientes. [Póngase en contacto con nosotros](mailto:azuremrs@microsoft.com) para tratar este asunto. 

**P: ¿En qué regiones está disponible Azure Spatial Anchors?**

**R:** Azure Spatial Anchors está disponible actualmente en Oeste de EE. UU. 2, Este de EE. UU., Este de EE. UU. 2, Centro-sur de EE. UU., Oeste de Europa, Norte de Europa, Sur de Reino Unido y Este de Australia. En el futuro, estarán disponibles más regiones.

Esto significa que los recursos tanto de proceso como de almacenamiento están en esas regiones. Aparte de eso, no hay ninguna restricción sobre la ubicación de los clientes. 

**P: ¿Azure Spatial Anchors es un servicio de pago?**

**R:** Puede encontrar más información sobre precios en nuestra [página de precios](https://azure.microsoft.com/pricing/details/spatial-anchors/).

## <a name="technical-faqs"></a>Preguntas técnicas más frecuentes

**P: ¿Cómo funciona Azure Spatial Anchors?**

**R:** Azure Spatial Anchors depende de rastreadores de realidad aumentada o mixta. Estos rastreadores perciben el entorno con cámaras y realizan un seguimiento del dispositivo en los 6 grados de libertad (6GDL) mientras se desplaza en el espacio.

Con un rastreador 6GDL como bloque de creación, Azure Spatial Anchors permite designar determinados puntos de interés del entorno real como puntos "delimitadores". Por ejemplo, podría utilizar un delimitador para representar el contenido de una ubicación específica del mundo real.

Cuando se crea un delimitador, el SDK de cliente captura información del entorno sobre ese punto y lo transmite al servicio. Si otro dispositivo busca el delimitador en el mismo espacio, se transmiten datos similares al servicio. Esos datos se comparan con los datos del entorno almacenados previamente. Después, se devuelve la posición del delimitador en relación con el dispositivo para su uso en la aplicación.

**P: ¿Cómo se integra Azure Spatial Anchors con ARKit y ARCore en iOS y Android?**

**R:** Azure Spatial Anchors aprovecha las funcionalidades nativas de seguimiento de ARKit y ARCore. Además, nuestros SDK para iOS y Android ofrecen funcionalidades tales como la conservación de delimitadores en un servicio en la nube administrado, y permite que las aplicaciones localicen los delimitadores de nuevo simplemente conectándose al servicio.

**P: ¿Cómo se integra Azure Spatial Anchors con HoloLens?**

**R:** Azure Spatial Anchors aprovecha las funcionalidades nativas de seguimiento de HoloLens. Proporcionamos un SDK de Azure Spatial Anchors para compilar aplicaciones en HoloLens. El SDK se integra con las funcionalidades nativas de HoloLens y ofrece funcionalidades adicionales. Entre estas funcionalidades se incluye permitir que los desarrolladores de aplicaciones conserven los delimitadores en un servicio en la nube administrado, y que las aplicaciones se conecten al servicio para localizar esos delimitadores.

**P: ¿Qué plataformas y lenguajes admite Azure Spatial Anchors?**

**R:** Los desarrolladores pueden crear aplicaciones con Azure Spatial Anchors mediante herramientas y plataformas de trabajo conocidas para sus dispositivos:

- Unity en HoloLens, iOS y Android
- Xamarin en iOS y Android
- Swift u Objective-C en iOS
- Java o el NDK para Android en Android
- C++/WinRT en HoloLens

Comience [a desarrollar aquí](index.yml).

**P: ¿Funciona con Unreal?**

**R:** La compatibilidad con Unreal se considerará en el futuro.

**P: ¿Qué puertos y protocolos usa Azure Spatial Anchors?**

**R:** Azure Spatial Anchors se comunica a través del puerto TCP 443 con un protocolo cifrado. Para la autenticación, usa [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/), que se comunica mediante HTTPS a través del puerto 443.
