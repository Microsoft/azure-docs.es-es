---
title: 'Actualización al nivel Estándar: Azure Security Center'
description: En este inicio rápido se muestra cómo realizar la actualización al plan de tarifa Estándar de Security Center para incrementar la seguridad.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security-center
ms.devlang: na
ms.topic: quickstart
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/3/2018
ms.author: memildin
ms.openlocfilehash: f16df87ca72b66003d8870dcd9d778616ea840d4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87038552"
---
# <a name="quickstart-onboard-your-azure-subscription-to-security-center-standard"></a>Inicio rápido: Incorporación de una suscripción de Azure a Security Center Standard
Azure Security Center proporciona administración unificada de la seguridad y protección contra amenazas para cargas de trabajo en la nube híbrida. Mientras el nivel Gratis solo ofrece seguridad limitada para los recursos de Azure, el nivel Estándar amplía estas funcionalidades tanto en un entorno local como en otras nubes. El nivel Estándar de Security Center le ayuda a encontrar y corregir vulnerabilidades de seguridad, aplicar controles de acceso y de aplicación para bloquear actividades malintencionadas, detectar amenazas mediante análisis e inteligencia y responder rápidamente en caso de ataque. Dicho plan se puede probar de forma gratuita. Para más información, consulte la [página de precios](https://azure.microsoft.com/pricing/details/security-center/).

En este artículo, se realiza la actualización al nivel Estándar para lograr mayor seguridad y se instala el agente de Log Analytics en las máquinas virtuales para supervisar las amenazas y vulnerabilidades de seguridad.

## <a name="prerequisites"></a>Prerrequisitos
Para empezar a trabajar con el Centro de seguridad, debe disponer de una suscripción a Microsoft Azure. Si no tiene una suscripción, puede registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/).

Para actualizar una suscripción al nivel Estándar, debe tener asignado los roles Propietario de la suscripción, Colaborador de la suscripción o Administrador de seguridad.

## <a name="enable-your-azure-subscription"></a>Habilitación de la suscripción a Azure

1. Inicie sesión en el [Portal de Azure](https://azure.microsoft.com/features/azure-portal/).
2. En el menú de **Microsoft Azure**, seleccione **Security Center**. Se abre **Security Center: Información general**.

   ![Introducción a Security Center][2]

**Security Center - Información general** proporciona una vista unificada de la situación de seguridad de las cargas de trabajo de la nube híbrida, lo que le permite detectar y evaluar la seguridad de las cargas de trabajo e identificar y mitigar riesgos. Security Center habilita automáticamente cualquiera de las suscripciones de Azure que no fueron incorporadas previamente por usted u otro usuario de la suscripción al nivel Gratis.

Puede ver y filtrar la lista de suscripciones haciendo clic en el elemento de menú **Suscripciones**. Security Center comenzará ahora a evaluar la seguridad de estas suscripciones para identificar vulnerabilidades de seguridad. Para personalizar los tipos de evaluaciones, puede modificar la directiva de seguridad. Una directiva de seguridad define la configuración deseada de las cargas de trabajo. Además, ayuda a garantizar el cumplimiento de los requisitos de seguridad normativos o de la empresa.

A los pocos minutos de iniciar Security Center la primera vez, puede ver:

- **Recomendaciones** para formas de mejorar la seguridad de las suscripciones de Azure. Al hacer clic en el icono **Recomendaciones** se iniciará una lista de prioridades.
- Un inventario de los recursos **Proceso y aplicaciones**, **Redes**, **Seguridad de los datos** e **Identidad y acceso** a los que ahora accede Security Center junto con la posición de seguridad de cada uno de ellos.

Para aprovechar al máximo Security Center, debe completar los pasos siguientes para realizar la actualización al nivel Estándar e instalar el agente de Log Analytics.

## <a name="upgrade-to-the-standard-pricing-tier"></a>Actualización del plan de tarifa Estándar
Para seguir los inicios rápidos y tutoriales de Security Center, debe realizar la actualización al nivel Estándar. Hay una evaluación gratuita de Security Center Estándar. Para más información, consulte la [página de precios](https://azure.microsoft.com/pricing/details/security-center/). 

1. En el menú principal de Security Center, seleccione **Introducción**.
 
   ![Introducción][4]

2. En **Actualizar**, Security Center muestra las suscripciones y áreas de trabajo aptas para la incorporación. 
   - Puede hacer clic en el expandible **Apply your trial** (Aplicar prueba) para ver una lista de todas las suscripciones y áreas de trabajo con el estado de elegibilidad de prueba.
   -    Puede actualizar las suscripciones y áreas de trabajo que no sean aptas para la versión de prueba.
   -    Puede seleccionar las áreas de trabajo y las suscripciones aptas para iniciar la versión de prueba.
3. Haga clic en **Iniciar período de prueba** para iniciar la evaluación de las suscripciones seleccionadas.


  ![Alertas de seguridad][9]

## <a name="automate-data-collection"></a>Recopilación automática de datos
Security Center recopila datos de VM de Azure y de los equipos que no son de Azure para supervisar las amenazas y vulnerabilidades de seguridad. Los datos se recopilan con el agente de Log Analytics, que lee distintas configuraciones relacionadas con la seguridad y distintos registros de eventos de la máquina y copia los datos en el área de trabajo para analizarlos. De forma predeterminada, Azure Security Center creará automáticamente una nueva área de trabajo.

Si el aprovisionamiento automático está habilitado, Security Center instala el agente de Log Analytics en todas las máquinas virtuales de Azure compatibles y en las nuevas que se creen. Se recomienda encarecidamente el aprovisionamiento automático.

Para habilitar el aprovisionamiento automático del agente de Log Analytics:

1. En el menú principal de Security Center, seleccione **Precios y configuración**.
2. En la fila de la suscripción, haga clic en la suscripción en la que desea cambiar la configuración.
3. En la pestaña **Recopilación de datos**, establezca **Aprovisionamiento automático** en **Activado**.
4. Seleccione **Guardar**.
---
  ![Habilitar el aprovisionamiento automático][6]

Con esta nueva información sobre VM de Azure, Security Center puede proporcionar recomendaciones adicionales relacionados con el estado de actualización del sistema, las configuraciones de seguridad del SO, la protección de puntos de conexión, así como generar alertas de seguridad adicionales.

  ![Recomendaciones][8]

## <a name="clean-up-resources"></a>Limpieza de recursos
Otras guías de inicio rápido y tutoriales de esta colección se basan en los valores de esta. Si planea continuar trabajando con los inicios rápidos y tutoriales posteriores, debe seguir ejecutando el plan de tarifa Estándar y mantener el aprovisionamiento automático habilitado. Si no planea continuar o desea volver al nivel Gratis:

1. Vuelva al menú principal de Security Center y seleccione **Precios y configuración**.
2. Haga clic en la suscripción que desea cambiar al nivel gratis.
3. Seleccione **Plan de tarifa** y, después, **Gratis** para cambiar la suscripción del nivel Estándar a Gratis.
5. Seleccione **Guardar**.

Si desea deshabilitar el aprovisionamiento automático:

1. Vuelva al menú principal de Security Center y seleccione **Precios y configuración**.
2. Limpie la suscripción en la que quiere deshabilitar el aprovisionamiento automático.
3. En la pestaña **Recopilación de datos**, establezca **Aprovisionamiento automático** en **Desactivado**.
4. Seleccione **Guardar**.

>[!NOTE]
> La deshabilitación del aprovisionamiento automático no quita el agente de Log Analytics de las máquinas virtuales de Azure en las que se ha aprovisionado. La deshabilitación del aprovisionamiento automático limita la supervisión de seguridad de los recursos.
>

## <a name="next-steps"></a>Pasos siguientes
En este inicio rápido, ha actualizado al nivel Estándar y ha aprovisionado el agente de Log Analytics para lograr una administración de la seguridad y una protección contra amenazas unificadas en sus cargas de trabajo en la nube híbrida. Para más información sobre cómo usar Security Center, siga la guía de inicio rápido para la incorporación de equipos Windows que se encuentran en local y en otras nubes.

> [!div class="nextstepaction"]
> [Inicio rápido: Incorporación de equipos Windows a Azure Security Center](quick-onboard-windows-computer.md)

¿Quiere optimizar y ahorrar en el gasto en la nube?

> [!div class="nextstepaction"]
> [Comience a analizar los costos con Cost Management](https://docs.microsoft.com/azure/cost-management-billing/costs/quick-acm-cost-analysis?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)

<!--Image references-->
[2]: ./media/security-center-get-started/overview.png
[4]: ./media/security-center-get-started/get-started.png
[5]: ./media/security-center-get-started/pricing.png
[6]: ./media/security-center-get-started/enable-automatic-provisioning.png
[7]: ./media/security-center-get-started/security-alerts.png
[8]: ./media/security-center-get-started/recommendations.png
[9]: ./media/security-center-get-started/select-subscription.png
