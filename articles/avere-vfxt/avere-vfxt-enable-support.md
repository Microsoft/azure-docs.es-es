---
title: Habilitar la compatibilidad con Avere vFXT - Azure
description: Obtenga información sobre cómo habilitar la carga automática de datos de soporte técnico sobre el clúster desde Avere vFXT for Azure para ayudar al servicio al cliente a proporcionar soporte técnico.
author: ekpgh
ms.service: avere-vfxt
ms.topic: how-to
ms.date: 12/14/2019
ms.author: rohogue
ms.openlocfilehash: 93b99aa624a21d9312297e4279b1dcf053c79ae3
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88272735"
---
# <a name="enable-support-uploads"></a>Habilitación de cargas de soporte

Avere vFXT for Azure puede cargar automáticamente datos de soporte técnico acerca de su clúster. Estas cargas permiten que el personal de soporte técnico proporcione el mejor servicio posible al cliente.

## <a name="steps-to-enable-uploads"></a>Pasos para habilitar cargas

Siga estos pasos desde el Panel de control de Avere para activar el soporte técnico. (Lea [Acceso al clúster de vFXT](avere-vfxt-cluster-gui.md) para aprender a abrir el panel de control).

1. Vaya a la pestaña **Configuración** en la parte superior.
1. Haga clic en el vínculo **Soporte técnico** en la izquierda y acepte la directiva de privacidad.

   ![Captura de pantalla que muestra el Panel de Control de Avere y una ventana emergente con el botón Confirmar para aceptar la directiva de privacidad](media/avere-vfxt-privacy-policy.png)

1. En la página de configuración del soporte técnico, haga clic en el triángulo de la izquierda para abrir la sección **Customer Info** (Información del cliente).
1. Haga clic en el botón **Revalidate upload information** (Revalidar información de carga).
1. En **Unique Cluster Name** (Nombre único del clúster). Asegúrese de que este nombre identifica de forma única el clúster para el personal de soporte técnico.
1. Active las casillas de **Statistics Monitoring** (Supervisión de estadísticas), **General Information Upload** (Carga de información general) y **Crash Information Upload** (Carga de información de bloqueo).
1. Haga clic en **Enviar**.

   ![Captura de pantalla que contiene la sección de información del cliente completada en la página de configuración de soporte técnico](media/avere-vfxt-support-info.png)

1. Haga clic en el triángulo situado a la izquierda de **Secure Proactive Support (SPS)** (Soporte técnico proactivo seguro (SPS)) para expandir la sección.
1. Active la casilla **Enable SPS Link** (Habilitar vínculo SPS).
1. Haga clic en **Enviar**.

   ![Captura de pantalla que contiene la sección de soporte técnico proactivo completada en la página de configuración de soporte técnico](media/avere-vfxt-support-sps.png)

## <a name="next-steps"></a>Pasos siguientes

Si necesita agregar un sistema de almacenamiento local o en la nube existente al clúster, siga las instrucciones de [Configurar el almacenamiento](avere-vfxt-add-storage.md).

Si está listo para empezar a adjuntar clientes al clúster, lea [Mount the Avere vFXT cluster](avere-vfxt-mount-clients.md) (Montar el clúster de Avere vFXT).
