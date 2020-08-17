---
title: Protección de red adaptable en Azure Security Center | Microsoft Docs
description: Obtenga información sobre cómo usar patrones de tráfico reales para proteger las reglas de los grupos de seguridad de red (NSG) y mejorar aún más la posición de seguridad.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 09d62d23-ab32-41f0-a5cf-8d80578181dd
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/11/2020
ms.author: memildin
ms.openlocfilehash: d12df01fe1506f7f5ade9cce60ae7af0412e3010
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080818"
---
# <a name="adaptive-network-hardening-in-azure-security-center"></a>Protección de red adaptable en Azure Security Center
Obtenga información sobre cómo configurar la protección de red adaptable en Azure Security Center.

## <a name="what-is-adaptive-network-hardening"></a>¿Qué es la protección de red adaptable?
La aplicación de [grupos de seguridad de red (NSG)](https://docs.microsoft.com/azure/virtual-network/security-overview) para filtrar el tráfico hacia y desde los recursos mejora la postura de seguridad de red. Sin embargo, aún puede haber algunos casos en los que el tráfico real que fluye a través del NSG es un subconjunto de las reglas de NSG definidas. En estos casos, puede mejorar la postura de seguridad al proteger aún más las reglas de NSG, según los patrones de tráfico real.

La protección de red adaptable proporciona recomendaciones para proteger mejor las reglas de NSG. Usa un algoritmo de aprendizaje automático que tiene en cuenta el tráfico real, la configuración de confianza conocida, la inteligencia de amenazas y otros indicadores de riesgo, y luego proporciona recomendaciones para permitir el tráfico solo desde tuplas IP y puerto específicas.

Por ejemplo, supongamos que la regla de NSG existente es permitir el tráfico desde 140.20.30.10/24 en el puerto 22. La recomendación de la protección de red adaptable, según el análisis, sería restringir el rango y permitir el tráfico desde 140.23.30.10/29, que es un intervalo de IP más restringido, y denegar todo el tráfico restante a ese puerto.

>[!TIP]
> Las recomendaciones de la protección de red adaptable solo se admiten en los siguientes puertos (para UDP y TCP): 13, 17, 19, 22, 23, 53, 69, 81, 111, 119, 123, 135, 137, 138, 139, 161, 162, 389, 445, 512, 514, 593, 636, 873, 1433, 1434, 1900, 2049, 2301, 2323, 2381, 3268, 3306, 3389, 4333, 5353, 5432, 5555, 5800, 5900, 5900, 5985, 5986, 6379, 6379, 7000, 7001, 7199, 8081, 8089, 8545, 9042, 9160, 9300, 11211, 16379, 26379, 27017 y 37215.


![Vista de la protección de red](./media/security-center-adaptive-network-hardening/traffic-hardening.png)


## <a name="view-adaptive-network-hardening-alerts-and-rules"></a>Visualización de las alertas y reglas de protección de red adaptable

1. En Security Center, seleccione **Redes** -> **Protección de red adaptable**. Las VM de red aparecen en tres pestañas independientes:
   * **Recursos con estado incorrecto** : VM que tienen actualmente recomendaciones y alertas que se desencadenaron mediante la ejecución del algoritmo de protección de red adaptable. 
   * **Recursos con estado correcto**: VM sin alertas ni recomendaciones.
   * **Recursos sin analizar** : VM en las que no se puede ejecutar el algoritmo de protección de red adaptable debido a uno de los motivos siguientes:
      * **Las VM son VM clásicas**: solo se admiten VM de Azure Resource Manager.
      * **No hay suficientes datos disponibles**: para generar recomendaciones de protección de tráfico precisas, Security Center requiere al menos 30 días de datos de tráfico.
      * **La VM no está protegida por el estándar de ASC**: solo las máquinas virtuales que están establecidas en el plan de tarifa estándar de Security Center son aptas para esta característica.

     ![Recursos con estado incorrecto](./media/security-center-adaptive-network-hardening/unhealthy-resources.png)

2. En la pestaña **Recursos con estado incorrecto**, seleccione una VM para ver sus alertas y las reglas de protección recomendadas que se deben aplicar.

    ![Alertas de protección](./media/security-center-adaptive-network-hardening/anh-recommendation-rules.png)


## <a name="review-and-apply-adaptive-network-hardening-recommended-rules"></a>Revisión y aplicación de las reglas recomendadas de protección de red adaptable

1. En la pestaña **Recursos con estado incorrecto**, seleccione una VM. Se muestran las alertas y reglas de protección recomendadas.

     ![Reglas de protección](./media/security-center-adaptive-network-hardening/hardening-alerts.png)

   > [!NOTE]
   > En la pestaña **Reglas** se enumeran las reglas que recomiendan la protección de red adaptable que agregue. En la pestaña **Alertas** se enumeran las alertas que se generaron debido al tráfico, que fluye al recurso y que no está dentro del intervalo de IP permitidas en las reglas recomendadas.

2. Si desea cambiar algunos de los parámetros de una regla, puede modificar esta última, tal como se explica en [Modificación de una regla](#modify-rule).
   > [!NOTE]
   > También puede [eliminar](#delete-rule) o [agregar](#add-rule) una regla.

3. Seleccione las reglas que desea aplicar en el NSG y haga clic en **Aplicar**.

      > [!NOTE]
      > Las reglas aplicadas se agregan a los NSG que protegen la VM. (Una VM podría estar protegida por un NSG asociado a su NIC, la subred en la que reside la VM o ambos).

    ![Aplicar reglas](./media/security-center-adaptive-network-hardening/enforce-hard-rule2.png)


### <a name="modify-a-rule"></a>Modificación de una regla <a name ="modify-rule"> </a>

Es posible modificar los parámetros de una regla que se haya recomendado. Por ejemplo, quizás querrá cambiar los intervalos de IP recomendados.

Instrucciones importantes a la hora de modificar una regla de protección de red adaptable:

* Solo puede modificar los parámetros de las reglas de tipo "permitir". 
* No puede cambiar reglas de tipo "permitir" para convertirlas en reglas de tipo "denegar". 

  > [!NOTE]
  > La creación y modificación de reglas de tipo "denegar" se realiza directamente en el NSG. Para más información, consulte [Creación, modificación o eliminación de un grupo de seguridad de red ](https://docs.microsoft.com/azure/virtual-network/manage-network-security-group).

* Una regla de tipo **Denegar todo el tráfico** es el único tipo de regla "denegar" que figuraría aquí, y no se puede modificar. Sin embargo, puede eliminar la regla (consulte [Eliminación de una regla](#delete-rule)).
  > [!NOTE]
  > Una regla tipo **Denegar todo el tráfico** se recomienda cuando, como resultado de la ejecución del algoritmo, Security Center no identifica el tráfico que se debe permitir, según la configuración de NSG existente. Por lo tanto, la regla recomendada es denegar todo el tráfico al puerto especificado. El nombre de este tipo de regla se muestra como "*Generada por el sistema*". Después de aplicar esta regla, su nombre real en el NSG será una cadena compuesta por el protocolo, la dirección del tráfico, "DENY" y un número aleatorio.

*Para modificar una regla de protección de red adaptable:*

1. Para modificar algunos de los parámetros de una regla, en la pestaña **Reglas**, haga clic en los puntos suspensivos (…) al final de la fila de la regla y seleccione **Editar**.

   ![Editar regla](./media/security-center-adaptive-network-hardening/edit-hard-rule.png)

1. En la ventana **Editar regla**, actualice los detalles que desea cambiar y haga clic en **Guardar**.

   > [!NOTE]
   > Después de hacer clic **Guardar**, habrá cambiado la regla correctamente. *Sin embargo, no la ha aplicado al NSG.* Para ello, debe seleccionar la regla de la lista y hacer clic en **Aplicar** (tal como se explica en el paso siguiente).

   ![Editar regla](./media/security-center-adaptive-network-hardening/edit-hard-rule3.png)

3. Para aplicar la regla actualizada, en la lista, seleccione la regla actualizada y haga clic en **Aplicar**.

    ![Aplicar regla](./media/security-center-adaptive-network-hardening/enforce-hard-rule.png)

### <a name="add-a-new-rule"></a>Adición de una nueva regla <a name ="add-rule"> </a>

Puede agregar una regla de tipo "permitir" no recomendada por Security Center.

> [!NOTE]
> Aquí solo se pueden agregar reglas de tipo "permitir". Si quiere agregar reglas de tipo "denegar", puede hacerlo directamente en el NSG. Para más información, consulte [Creación, modificación o eliminación de un grupo de seguridad de red ](https://docs.microsoft.com/azure/virtual-network/manage-network-security-group).

*Para agregar una regla de protección de red adaptable:*

1. Haga clic en **Agregar regla** (situado en la esquina superior izquierda).

   ![Agregar regla](./media/security-center-adaptive-network-hardening/add-hard-rule.png)

1. En la ventana **Nueva regla**, especifique los detalles y haga clic en **Agregar**.

   > [!NOTE]
   > Después de hacer clic en **Agregar**, habrá agregado correctamente la regla y esta se muestra con las demás reglas recomendadas. Sin embargo, no la ha aplicado al NSG. Para ello, debe seleccionar la regla de la lista y hacer clic en **Aplicar** (tal como se explica en el paso siguiente).

3. Para aplicar la nueva regla, en la lista, seleccione la nueva regla y haga clic en **aplicar**.

    ![Aplicar regla](./media/security-center-adaptive-network-hardening/enforce-hard-rule.png)


### <a name="delete-a-rule"></a>Eliminación de una regla <a name ="delete-rule"> </a>

Cuando sea necesario, puede eliminar una regla recomendada de la sesión actual. Por ejemplo, puede determinar que aplicar una regla sugerida podría bloquear tráfico legítimo.

*Para eliminar una regla de protección de red adaptable de la sesión actual:*

1. En la pestaña **Reglas**, haga clic en los puntos suspensivos (…) al final de la fila de la regla y seleccione **Eliminar**.  

    ![Reglas de protección](./media/security-center-adaptive-network-hardening/delete-hard-rule.png)