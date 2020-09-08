---
title: Ofertas del módulo de IoT Edge de Azure Marketplace
description: Obtenga información sobre la publicación de ofertas de módulo de IoT Edge en Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 08/18/2020
ms.openlocfilehash: c475e548cc8e29a83f2d440df43fb8ac31dc1c1e
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816259"
---
# <a name="iot-edge-modules"></a>Módulos de IoT Edge

La plataforma [Azure IoT Edge](https://azure.microsoft.com/services/iot-edge/) está respaldada por Microsoft Azure.  Esta plataforma permite a los usuarios implementar cargas de trabajo en la nube para ejecutarlas directamente en dispositivos IoT.  Un módulo IoT Edge puede ejecutar las cargas de trabajo sin conexión y realizar análisis de datos localmente. Este tipo de oferta ayuda a ahorrar ancho de banda, proteger datos locales y confidenciales y ofrecer un tiempo de respuesta de baja latencia.  Ahora tiene las opciones para aprovechar las ventajas de estas cargas de trabajo creadas previamente. Hasta ahora, solo estaban disponibles un puñado de soluciones propias de Microsoft.  Tenía que invertir tiempo y recursos en compilar sus propias soluciones personalizadas de IoT.

Con los [módulos de IoT Edge en Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1), ahora tenemos un único destino para que los publicadores expongan y vendan sus soluciones a la audiencia de IoT. Los desarrolladores de IoT pueden en última instancia buscar y adquirir capacidades para acelerar el desarrollo de sus soluciones.  

## <a name="key-benefits-of-iot-edge-modules-in-azure-marketplace"></a>Principales ventajas de los módulos de IoT Edge en Azure Marketplace

| **Para los publicadores**    | **Para los clientes (desarrolladores de IoT)**  |
| :------------------- | :-------------------|
| Llegue a millones de desarrolladores que buscan crear e implementar soluciones de IoT Edge.  | Cree una solución de IoT Edge con la confianza de usar de componentes probados y seguros. |
| Publique una vez y ejecute en cualquier hardware de IoT Edge que admita contenedores. | Reduzca el tiempo de comercialización al encontrar e implementar módulos de IoT Edge propios de Microsoft y de terceros para necesidades específicas. |
| Monetizar con opciones de facturación flexibles <ul> <li> Gratis y Traiga su propia licencia (BYOL). </li> </ul> | Realice compras con su elección de modelos de facturación. <ul> <li> Gratis y Traiga su propia licencia (BYOL). </li> </ul> |

## <a name="what-is-an-iot-edge-module"></a>¿Qué es un módulo IoT Edge?

Azure IoT Edge le permite implementar y administrar la lógica de negocios en el Edge a través de módulos. Los módulos de Azure IoT Edge son las unidades de cálculo más pequeñas que administra IoT Edge y pueden contener servicios de Microsoft (por ejemplo, Azure Stream Analytics), servicios de terceros o su propio código específico de la solución. Para obtener más información sobre los módulos de IoT Edge, vea [Información sobre los módulos de Azure IoT Edge](../iot-edge/iot-edge-modules.md).

**¿Cuál es la diferencia entre un tipo de oferta de contenedor y un tipo de oferta de módulo de IoT Edge?**

El tipo de oferta de módulo de IoT Edge es un tipo específico de contenedor que se ejecuta en un dispositivo de IoT Edge. Incluye valores de configuración predeterminados para ejecutarse en el contexto de IoT Edge y, opcionalmente, usa el SDK del módulo de IoT Edge que se integra con el tiempo de ejecución de IoT Edge.

## <a name="publishing-your-iot-edge-module"></a>Publicar el módulo de IoT Edge

**Selección de la tienda en línea correcta**

Los módulos de IoT Edge solo se publican en Azure Marketplace; no se aplica AppSource. Para más información sobre las diferencias y el público de destino de las tiendas en línea, vea [Determinación de la opción de publicación](determine-your-listing-type.md).

**Opciones de facturación**

Marketplace admite actualmente las opciones de facturación **Gratis** y **Traiga su propia licencia (BYOL)** para los módulos de IoT Edge.

### <a name="publishing-options"></a>Opciones de publicación

En todos los casos, los módulos de IoT Edge deben seleccionar la opción de publicación **Transact**.  Vea [Determinar la opción de publicación](determine-your-listing-type.md) para obtener más detalles sobre las opciones de publicación.  

## <a name="eligibility-criteria"></a>Criterios de elegibilidad

Todos los términos de los contratos de Microsoft Azure Marketplace y las directivas se aplican a las ofertas de módulo de IoT Edge.  Además, existen requisitos previos y requisitos técnicos para los módulos de IoT Edge.  

### <a name="prerequisites"></a>Requisitos previos

Para publicar un módulo de IoT Edge en Azure Marketplace, es preciso cumplir los siguientes requisitos previos:

- Acceso al Centro de partners. Para obtener más información, vea [Guía de publicación de Azure Marketplace y AppSource](marketplace-publishers-guide.md).
- Hospedar el módulo de IoT Edge en Azure Container Registry.
- Tener los metadatos del módulo IoT Edge listos como (lista no exhaustiva):
    - Un título
    - Una descripción (en formato HTML)
    - Una imagen de logotipo (en tamaños de 48 x 48 (opcional), 90 x 90 (opcional) y de 216 x 216 a 350 x 350 píxeles, todo en formato PNG)
    - Un término de uso y directiva de privacidad
    - Configuración predeterminada del módulo (ruta, propiedades deseadas del gemelo, createOptions, variables de entorno)
    - Documentación
    - Contactos de soporte técnico

### <a name="technical-requirements"></a>Requisitos técnicos

Los requisitos técnicos principales para que un módulo de IoT Edge pueda obtener la certificación y publicarse en Azure Marketplace se detallan en [Preparar los recursos técnicos del módulo IoT Edge](./partner-center-portal/create-iot-edge-module-asset.md).

## <a name="next-steps"></a>Pasos siguientes

- [Obtenga información](https://azuremarketplace.microsoft.com/sell) sobre el marketplace comercial.
- Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership) para crear o completar la oferta.
- [Creación de una oferta de módulo IoT Edge](./partner-center-portal/azure-iot-edge-module-creation.md) en el Centro de partners.
