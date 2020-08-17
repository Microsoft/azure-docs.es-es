---
title: Envío de alertas desde Azure Application Insights | Microsoft Docs
description: Tutorial para enviar alertas en respuesta a errores en la aplicación con Azure Application Insights.
ms.subservice: application-insights
ms.topic: tutorial
author: mrbullwinkle
ms.author: mbullwin
ms.date: 04/10/2019
ms.custom: mvc
ms.openlocfilehash: 706f3913e25eca6240c186e45709faf6c77620bf
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324580"
---
# <a name="monitor-and-alert-on-application-health-with-azure-application-insights"></a>Supervise el estado de la aplicación y envíe alertas con Azure Application Insights.

Azure Application Insights le permite supervisar su aplicación y le envía alertas cuando no está disponible, experimenta errores o sufre problemas de rendimiento.  En este tutorial recorrerá el proceso de creación de pruebas para comprobar de forma continua la disponibilidad de una aplicación.

Aprenderá a:

> [!div class="checklist"]
> * Crear una prueba de disponibilidad para comprobar continuamente la respuesta de la aplicación
> * Enviar un correo electrónico a los administradores cuando se produzca un problema

## <a name="prerequisites"></a>Prerequisites

Para completar este tutorial:

Cree un [recurso de Application Insights](./dotnetcore-quick-start.md#enable-application-insights).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="create-availability-test"></a>Creación de una prueba de disponibilidad

Las pruebas de disponibilidad en Application Insights le permiten probar automáticamente la aplicación desde diferentes ubicaciones de todo el mundo.   En este tutorial, realizará una prueba de una dirección URL para asegurarse de que la aplicación web está disponible.  También puede crear un tutorial completo para probar su funcionamiento detallado. 

1. Seleccione **Application Insights** y, a continuación, seleccione la suscripción.  

2. Seleccione **Disponibilidad** en el menú **Investigar** y, a continuación, haga clic en **Crear prueba**.

    ![Adición de prueba de disponibilidad](media/tutorial-alert/add-test-001.png)

3. Escriba un nombre para la prueba y deje las demás opciones en sus valores predeterminados.  Esta selección desencadenará solicitudes para la dirección URL de aplicación cada 5 minutos desde cinco ubicaciones geográficas diferentes.

4. Seleccione **Alertas** para abrir la lista desplegable **Alertas**, donde puede definir los detalles relativos a responder si se produce un error en la prueba. Elija **Prácticamente en tiempo real** y establezca el estado en **Habilitado.**

    Escriba una dirección de correo electrónico a la que enviar los mensajes cuando se cumplan los criterios de alerta.  También puede escribir la dirección de un webhook al que llamar cuando se cumplan los criterios de alerta.

    ![Creación de prueba](media/tutorial-alert/create-test-001.png)

5. Vuelva al panel de prueba, seleccione el botón de puntos suspensivos y edite la alerta para especificar la configuración de la alerta prácticamente en tiempo real.

    ![Editar alerta](media/tutorial-alert/edit-alert-001.png)

6. Seleccione que el valor de ubicaciones con error sea mayor o igual a 3. Cree un [grupo de acciones](../platform/action-groups.md) para configurar quiénes reciben notificaciones al superarse el umbral de alerta.

    ![Interfaz de usuario para guardar alerta](media/tutorial-alert/save-alert-001.png)

7. Una vez configurada la alerta, haga clic en el nombre de la prueba para ver los detalles de cada ubicación. Las pruebas pueden verse en forma tanto de gráfico de líneas como de gráfico de dispersión y de gráfico para visualizar los aciertos/errores de un intervalo de tiempo determinado.

    ![Detalles de las pruebas](media/tutorial-alert/test-details-001.png)

8. Puede profundizar en los detalles de cualquier prueba; para ello, haga clic en su punto en el gráfico de dispersión. Esto iniciará la vista de detalles de una transacción completa. En el ejemplo siguiente se muestran los detalles de una solicitud con error.

    ![Resultado de la prueba](media/tutorial-alert/test-result-001.png)
  
## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido cómo alertar sobre problemas, continúe con el siguiente tutorial para aprender a analizar el modo en que los usuarios interactúan con la aplicación.

> [!div class="nextstepaction"]
> [Entender a los usuarios](./tutorial-users.md)

