---
title: Integración de Windows Admin Center con Azure Security Center | Microsoft Docs
description: En este artículo se explica cómo integrar Azure Security Center con Windows Admin Center.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: memildin
ms.openlocfilehash: 9eae210818cf623078090503deefc6295dab7164
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076358"
---
# <a name="integrate-azure-security-center-with-windows-admin-center"></a>Integración de Azure Security Center con Windows Admin Center

Windows Admin Center es una herramienta de administración para los servidores de Windows. Es una ubicación única para que los administradores del sistema obtengan acceso a la mayoría de las herramientas de administración que se usan con más frecuencia. Desde Windows Admin Center, puede incorporar directamente sus servidores locales en Azure Security Center. Asimismo, puede ver un resumen de las recomendaciones y alertas de seguridad directamente en Windows Admin Center.

> [!NOTE]
> La suscripción de Azure y el área de trabajo de Log Analytics asociada deben tener habilitado el nivel estándar de Security Center para poder habilitar la integración de Windows Admin Center.
> El nivel estándar es gratuito durante los primeros 30 días si no lo ha usado previamente en la suscripción y el área de trabajo. Para obtener mas información de precios, consulte la [página de precios](security-center-pricing.md).
>

Cuando haya incorporado correctamente un servidor de Windows Admin Center a Azure Security Center, podrá realizar lo siguiente:

* Ver alertas y recomendaciones de seguridad en la extensión de Security Center de Windows Admin Center.
* Ver la postura de seguridad y recuperar información detallada adicional de los servidores administrados de Windows Admin Center en Security Center, mediante Azure Portal (o a través de una API).

Al combinar estas dos herramientas, Security Center se convierte en el único panel transparente que es útil para ver toda la información de seguridad, sea cual sea el objetivo: proteger los servidores locales administrados en Windows Admin Center, las VM y cualquier carga de trabajo de PaaS adicional.

## <a name="onboarding-windows-admin-center-managed-servers-into-security-center"></a>Incorporación de los servidores administrados de Windows Admin Center en Security Center

1. En Windows Admin Center, seleccione uno de los servidores y, en el panel de **herramientas**, seleccione la extensión de Azure Security Center:

    ![Extensión de Azure Security Center en Windows Admin Center](./media/windows-admin-center-integration/onboarding-from-wac.png)

    > [!NOTE]
    > Si el servidor ya se ha incorporado a Security Center, no aparecerá la ventana de configuración.

1. Haga clic en **Sign in to Azure and set up** (Iniciar sesión en Azure y configurar).
    ![Incorporación de la extensión de Windows Admin Center en Azure Security Center](./media/windows-admin-center-integration/onboarding-from-wac-welcome.png)

1. Siga las instrucciones para conectar el servidor a Security Center. Una vez que haya escrito los detalles necesarios y los haya confirmado, Security Center realiza los cambios de configuración necesarios para asegurarse de que se cumplen todas las condiciones siguientes:
    * Se registra una puerta de enlace de Azure.
    * El servidor tiene un área de trabajo que notificar y una suscripción asociada.
    * La solución de Log Analytics de nivel estándar de Security Center está habilitada en el área de trabajo. Esta solución proporciona características de nivel estándar de Security Center para *todos* los servidores y las máquinas virtuales que informan a esta área de trabajo.
    * El precio del nivel estándar de Security Center para la máquina virtual está habilitado en la suscripción.
    * El agente de Log Analytics se instala en el servidor y se configura para que informe al área de trabajo seleccionada. Si el servidor ya informa a otra área de trabajo, se configura para que informe al área de trabajo que se acaba de seleccionar también.

    > [!NOTE]
    > Es posible que las recomendaciones tarden un rato en aparecer después de incorporarlas. De hecho, en función de la actividad del servidor, es posible que no reciba *ninguna* alerta. Para generar alertas de prueba y así comprobar que las alertas funcionan correctamente, siga las instrucciones descritas en el [procedimiento de validación de alertas](security-center-alert-validation.md).


## <a name="viewing-security-recommendations-and-alerts-in-windows-admin-center"></a>Ver recomendaciones y alertas de seguridad en Windows Admin Center

Una vez incorporada, puede ver las alertas y las recomendaciones directamente en el área de Azure Security Center que está en Windows Admin Center. Haga clic en una recomendación o una alerta para verla en Azure Portal. Una vez allí, obtendrá información adicional y aprenderá a solucionar los problemas que tenga.

[![Recomendaciones y alertas de Security Center, tal como se ven en Windows Admin Center](media/windows-admin-center-integration/asc-recommendations-and-alerts-in-wac.png)](media/windows-admin-center-integration/asc-recommendations-and-alerts-in-wac.png#lightbox)

## <a name="viewing-security-recommendations-and-alerts-for-windows-admin-center-managed-servers-in-security-center"></a>Ver recomendaciones y alertas de seguridad para servidores administrados de Windows Admin Center en Security Center
Desde Azure Security Center:

* Para ver recomendaciones de seguridad para todos los servidores de Windows Admin Center, abra **Compute & Apps** (Procesos y aplicaciones) y haga clic en la pestaña de **VM y equipos**. Filtre la lista por el recurso "Servidor" tal como se muestra aquí:

    [![Ver las recomendaciones de seguridad para servidores administrados de Windows Admin Center](media/windows-admin-center-integration/viewing-recommendations-wac.png)](media/windows-admin-center-integration/viewing-recommendations-wac.png#lightbox)

* Para ver las alertas de seguridad de todos los servidores de Windows Admin Center, abra la opción de **alertas de seguridad**. Haga clic en **Filtrar** y asegúrese de que **solo** esté seleccionada la opción "No es de Azure":

    ![Filtrar las alertas de seguridad de los servidores administrados de Windows Admin Center](./media/windows-admin-center-integration/filtering-alerts-to-non-azure.png)

    [![Ver las alertas de seguridad de los servidores administrados de Windows Admin Center](media/windows-admin-center-integration/viewing-alerts-wac.png)](media/windows-admin-center-integration/viewing-alerts-wac.png#lightbox)