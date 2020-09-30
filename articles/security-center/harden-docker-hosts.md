---
title: Uso de Azure Security Center para proteger los hosts de Docker y los contenedores
description: Protección de hosts de Docker y comprobación de su compatibilidad con el banco de pruebas de Docker de CIS.
author: memildin
ms.author: memildin
ms.date: 9/12/2020
ms.topic: how-to
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 25d8d3dee8810311ae25f54c0e51e34bdf918c09
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931885"
---
# <a name="harden-your-docker-hosts"></a>Protección de los hosts de Docker

Azure Security Center identifica contenedores no administrados y que están hospedados en VM de IaaS Linux u otras máquinas de Linux que ejecutan contenedores de Docker. Security Center evalúa continuamente las configuraciones de estos contenedores. A continuación, las compara con el [Banco de prueba para Docker del Centro de seguridad de Internet (CIS)](https://www.cisecurity.org/benchmark/docker/).

Security Center incluye todo el conjunto de reglas del banco de prueba de Docker de CIS y le avisa si los contenedores no cumplen ninguno de los controles. Cuando encuentra configuraciones inactivas, Security Center genera recomendaciones de seguridad. Use la **página de recomendaciones** para ver las recomendaciones y corregir los problemas. También verá las recomendaciones en la pestaña de **contenedores** que muestra todas las máquinas virtuales implementadas con Docker. 

Cuando se detectan vulnerabilidades, se agrupan en una sola recomendación.

>[!NOTE]
> Estas comprobaciones del banco de prueba de CIS no se ejecutarán en instancias que administre AKS o en VM que administre Databricks.

## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponible con carácter general|
|Precios:|Requiere [Azure Defender para servidores](defender-for-servers-introduction.md).|
|Roles y permisos necesarios:|**Lector** en el área de trabajo a la que se conecta el host.|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![No](./media/icons/no-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||

## <a name="identify-and-remediate-security-vulnerabilities-in-your-docker-configuration"></a>Identificación y corrección de vulnerabilidades de seguridad en la configuración de Docker

1. En el menú de Security Center, abra la página **Recomendaciones**.

1. Filtre por la recomendación **Las vulnerabilidades en las configuraciones de seguridad de contenedor deben corregirse** y selecciónela.

    En la página de recomendación se muestran los recursos afectados (hosts de Docker). 

    :::image type="content" source="./media/monitor-container-security/docker-host-vulnerabilities-found.png" alt-text="Recomendación para corregir vulnerabilidades en las configuraciones de seguridad de contenedores":::

1. Para ver y corregir los controles de CIS en los que se produjo un error en un host específico, seleccione el host que desea investigar. 

    > [!TIP]
    > Si empezó en la página de inventario de recursos y ha llegado a esta recomendación desde ahí, seleccione el botón **Realizar acción** de la página de la recomendación.
    >
    > :::image type="content" source="./media/monitor-container-security/host-security-take-action-button.png" alt-text="Botón Realizar acción para iniciar Log Analytics":::

    Log Analytics se abre con una operación personalizada lista para ejecutarse. La consulta personalizada predeterminada incluye una lista de todas las reglas con errores que se han evaluado, junto con instrucciones para ayudarle a resolver los problemas.

    :::image type="content" source="./media/monitor-container-security/docker-host-vulnerabilities-in-query.png" alt-text="Página de Log Analytics con la consulta que muestra todos los controles de CIS con errores":::

1. Retoque los parámetros de consulta si es necesario.

1. Cuando esté seguro de que el comando es adecuado y está listo para el host, seleccione **Ejecutar**.


## <a name="next-steps"></a>Pasos siguientes

La protección de Docker es solo un aspecto de las características de seguridad del contenedor de Security Center. 

Obtenga más información sobre la [seguridad de los contenedores en Security Center](container-security.md).