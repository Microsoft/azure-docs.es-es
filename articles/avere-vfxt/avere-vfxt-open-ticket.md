---
title: Cómo obtener soporte técnico para Avere vFXT para Azure
description: Obtenga información sobre cómo solucionar los problemas que pueden surgir durante la implementación o el uso de Avere vFXT for Azure mediante la creación de una incidencia de soporte técnico en Azure Portal.
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 01/13/2020
ms.author: rohogue
ms.openlocfilehash: 522d29505d7d10f5f6d97136f270f07a63053d19
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88271114"
---
# <a name="get-help-with-your-system"></a>Obtenga ayuda con el sistema

Si necesita ayuda con el sistema Avere vFXT for Azure, estas son las diversas maneras de obtener soporte técnico:

* **Problema con Avere vFXT**: use Azure Portal para abrir una incidencia de soporte técnico referente a Avere vFXT tal como se describe [a continuación](#open-a-support-ticket-for-your-avere-vfxt).
* **Cuota**: si tiene un problema relacionado con la cuota, [solicite un aumento de cuota](#request-a-quota-increase).
* **Documentación y ejemplos**: si tiene problemas con esta documentación o con los ejemplos, vaya al final de la página que tenga el problema y use la sección de **comentarios** para consultar los problemas existentes y crear uno nuevo si es necesario.

## <a name="open-a-support-ticket-for-your-avere-vfxt"></a>Abrir una incidencia de soporte técnico en Avere vFXT

Si tiene problemas al implementar o usar Avere vFXT, solicite ayuda mediante Azure Portal.

Siga estos pasos para asegurarse de que su incidencia de soporte técnico se etiqueta con un recurso del clúster. Gracias a esta etiqueta podremos ofrecerle el recurso correcto de soporte técnico.

1. En [https://portal.azure.com](https://portal.azure.com), seleccione **Grupos de recursos**. Busque el grupo de recursos que contiene el clúster de vFXT donde se produjo el problema y haga clic en una de las máquinas virtuales del clúster de Avere.

    ![captura de pantalla del panel de "información general" del grupo de recursos de Azure Portal con una máquina virtual concreta marcada](media/avere-vfxt-ticket-vm.png)

1. En la página de la máquina virtual, desplácese hasta la parte inferior del panel izquierdo y haga clic en **Nueva solicitud de soporte técnico**.

    ![Captura de pantalla de la página de la máquina virtual de Azure Portal para la máquina virtual de la captura de pantalla anterior. El menú de la izquierda se desplaza hasta la parte inferior y se resalta la opción "New support request" (Nueva solicitud de soporte técnico).](media/avere-vfxt-ticket-request.png)

1. En la primera página de la solicitud de soporte técnico, elija el tipo de problema y asegúrese de que la suscripción correcta está seleccionada.

   En **Servicio**, haga clic en **Todos los servicios** y en **Almacenamiento**  elija **Avere vFXT**.

   Agregue un breve resumen y seleccione el tipo de problema.

    ![Captura de una nueva pantalla de solicitud de soporte técnico en Azure Portal. La pestaña Aspectos básicos está seleccionada. Los elementos de la pantalla incluyen el tipo de problema, la suscripción, el servicio, el resumen y el tipo de problema.](media/ticket-basics.png)

   Haga clic en **Siguiente** para continuar.

1. La segunda página del formulario de soporte técnico contiene sugerencias para corregir el problema en función de la descripción del resumen. Haga clic en el botón **Siguiente** en la parte inferior si todavía necesita crear una incidencia de soporte técnico.

   ![Captura de la nueva pantalla de solicitud de soporte técnico con la pestaña Soluciones seleccionada. Un campo de texto en el centro tiene el título "Solución recomendada" y explica posibles remedios.](media/ticket-solutions.png)

1. En la tercera página, proporcione detalles, por ejemplo, información sobre el clúster, la hora a la que se produjo el problema, la gravedad y cómo ponerse en contacto con usted. Rellene la información y haga clic en el botón **Siguiente** situado en la parte inferior.

   ![Captura de la nueva pantalla de solicitud de soporte técnico con la pestaña Detalles seleccionada. Los campos de información se organizan en las categorías "Detalles del problema", "Método de soporte técnico" e "Información de contacto".](media/ticket-details.png)

1. Revise la información de la página y haga clic en **Crear**. Se enviarán un número de confirmación y otro de vale a su dirección de correo electrónico y un miembro del personal del servicio de soporte técnico se pondrá en contacto con usted.

## <a name="request-a-quota-increase"></a>Solicitar un aumento de cuota

Lea [Quota for the vFXT cluster](avere-vfxt-prereqs.md#quota-for-the-vfxt-cluster) (Cuota para el clúster de vFXT) para obtener información sobre qué componentes son necesarios para implementar Avere vFXT para Azure. También puede [solicitar un aumento de cuota](https://docs.microsoft.com/azure/azure-portal/supportability/resource-manager-core-quotas-request) desde Azure Portal.
