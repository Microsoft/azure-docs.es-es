---
title: ¿Qué es una versión de prueba? Marketplace comercial de Microsoft
description: Explicación de la característica de versión de prueba del marketplace
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: keferna
ms.author: keferna
ms.date: 06/19/2020
ms.openlocfilehash: bd3b77f80a414dd3db1d5106929fa0e215e34c1f
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88121874"
---
# <a name="what-is-a-test-drive"></a>¿Qué es una versión de prueba?

Una versión de prueba es una excelente manera de presentar la oferta a clientes potenciales, ya que se les brinda la oportunidad de *probar antes de comprar*, lo que aumenta la conversión y la generación de clientes potenciales altamente cualificados. Una versión de prueba hace que el producto cobre vida en un escenario de implementación real, además de generar clientes potenciales altamente cualificados.

Las versiones de prueba son instancias administradas que implementan la solución o aplicación a petición para los clientes que la solicitan. Una vez que se asigna una instancia de versión de prueba, está disponible para usarla durante el tiempo establecido y, luego, se elimina para hacer espacio para otro cliente.

Como publicador, puede administrar y configurar las opciones de la versión de prueba en el Centro de partners. Los detalles de la configuración técnica varían según el tipo de oferta con la que trabaje. Para obtener instrucciones detalladas, vea el vínculo [Paso siguiente](#next-step) al final de este tema.

Los clientes potenciales descubren la versión de prueba en el marketplace comercial. Proporcionan su información de contacto y aceptan los términos y la directiva de privacidad de la oferta y, después, obtienen acceso al entorno preconfigurado para probarlo durante un período de tiempo fijo. Los clientes reciben una evaluación gratuita práctica y guiada de las principales características y ventajas del producto y, a cambio, usted obtiene un valioso cliente potencial.

## <a name="how-does-it-work"></a>¿Cómo funciona?

Como publicador, puede administrar y configurar las opciones de la versión de prueba desde el Centro de partners. Después de configurarla, se convierte en una instancia administrada que se implementará a petición para el cliente que la solicita. Una vez que se asigna una instancia de versión de prueba, está disponible para usarla durante el tiempo establecido y, luego, se elimina para hacer espacio para otro cliente.

## <a name="types-of-test-drives"></a>Tipos de versiones de prueba

Hay diferentes versiones de prueba disponibles en el marketplace comercial para ofertas concretas según el tipo de producto, el escenario y el marketplace en el que se encuentre:

- Azure Resource Manager
- Versión de prueba hospedada
    - Dynamics 365 for Business Central
    - Dynamics 365 for Customer Engagement
    - Dynamics 365 for Operations
- Aplicación lógica
- Power BI

Para obtener más información sobre la configuración de una de estas unidades de prueba, vea el vínculo bajo [Paso siguiente](#next-step) al final de este tema.

### <a name="azure-resource-manager-test-drive"></a>Versión de prueba de Azure Resource Manager

Esta plantilla de implementación contiene todos los recursos de Azure que componen la solución. Los productos que se ajustan a este escenario usan solo recursos de Azure. La versión de prueba de Azure Resource Manager está disponible para estos tipos de oferta: 

- Aplicaciones de Azure
- Dynamics 365 for Business Central
- Dynamics 365 for Customer Engagement y Power Apps
- Dynamics 365 for Operations
- SaaS
- Máquinas virtuales

>[!NOTE]
>Es la única opción de versión de prueba para las ofertas de máquina virtual o de aplicación de Azure.

### <a name="hosted-test-drive"></a>Versión de prueba hospedada

Una versión de prueba hospedada elimina la complejidad de la configuración ya que deja que Microsoft hospede y mantenga el servicio que realiza las tareas de aprovisionamiento, implementación y desaprovisionamiento de los usuarios de la versión de prueba. Si tiene una oferta en Microsoft AppSource, compile la versión de prueba para conectarse con una instancia de Dynamics AX/CRM o cualquier otro recurso más allá de Azure. Use este tipo para que las ofertas de AppSource se conecten con estas de Dynamics 365:

- Use [Dynamics 365 for Business Central](partner-center-portal/create-new-operations-offer.md) para un sistema central de planeamiento de recursos empresariales como finanzas, operaciones, cadena de suministro y CRM.
- Use [Dynamics 365 for Customer Engagement](partner-center-portal/create-new-customer-engagement-offer.md) para un sistema de involucración del cliente como ventas, servicio, servicio de proyecto y servicio de campo.
- Use [Dynamics 365 for Operations](partner-center-portal/create-new-operations-offer.md) para un sistema de planeamiento de recursos empresariales de finanzas y operaciones, como finanzas, operaciones y fabricación, cadena de suministro.

### <a name="logic-app-test-drive"></a>Versión de prueba de aplicación lógica

Microsoft no hospeda este tipo de versión de prueba. Úselo para conectarse con una oferta de Dynamics 365 u otro recurso personalizado.

### <a name="power-bi-test-drive"></a>Versión de prueba de Power BI

Se trata simplemente de un vínculo insertado en un panel personalizado. Este tipo de versión de prueba lo deben usar los productos que solo demuestren un objeto visual interactivo de Power BI.

## <a name="transforming-examples"></a>Ejemplos de transformación

El proceso de conversión de una arquitectura de recursos en una versión de prueba puede ser abrumador. Consulte estos ejemplos de cómo transformar mejor las arquitecturas actuales.

[Transformación de una plantilla de sitio web en una versión de prueba](https://github.com/Azure/AzureTestDrive/wiki/Transforming-Website-Deployment-Template-for-Test-Drive)

[Transformación de una plantilla de máquina virtual en una versión de prueba](https://github.com/Azure/AzureTestDrive/wiki/Transforming-Virtual-Machine-Deployment-Template-for-Test-Drive)

[Transformación de una plantilla de Resource Manager existente en una versión de prueba](https://github.com/Azure/AzureTestDrive/wiki/Deploying-Existing-Solutions)

## <a name="generate-leads-from-your-test-drive"></a>Generación de clientes potenciales a partir de la versión de prueba

Una versión de prueba del marketplace comercial es una herramienta excelente para los vendedores. Le recomendamos que la incorpore durante al lanzamiento a sus esfuerzos de comercialización para generar más clientes potenciales para el negocio. Para obtener instrucciones detalladas, vea [Clientes potenciales a partir de la oferta en el marketplace comercial](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/marketplace/partner-center-portal/commercial-marketplace-get-customer-leads.md).

Si cierra un trato con un cliente potencial de una versión de prueba, no olvide registrarlo en [Microsoft Partner Sales Connect](https://support.microsoft.com/help/3155788/getting-started-with-microsoft-partner-sales-connect). Además, nos encantaría conocer sus éxitos con los clientes en los que la versión de prueba ha desempeñado un papel importante.

## <a name="other-resources"></a>Otros recursos

Recursos adicionales de la versión de prueba:

- [Procedimientos técnicos recomendados](https://github.com/Azure/AzureTestDrive/wiki/Test-Drive-Best-Practices)
- [Información general](https://assetsprod.microsoft.com/mpn/azure-marketplace-appsource-test-drives.pdf) (PDF, asegúrese de que el bloqueador de elementos emergentes está desactivado)

## <a name="next-step"></a>Paso siguiente

- [Configuración técnica de la versión de prueba](test-drive-technical-configuration.md)
