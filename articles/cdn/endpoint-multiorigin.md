---
title: Punto de conexión con varios orígenes de Azure CDN
description: Empiece a trabajar con el punto de conexión con varios orígenes de Azure CDN.
services: cdn
author: asudbring
manager: KumudD
ms.service: azure-cdn
ms.topic: how-to
ms.date: 8/20/2020
ms.author: allensu
ms.openlocfilehash: c7e6733079dbd867255e604f6f8d4459f647cc93
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88870477"
---
# <a name="azure-cdn-endpoint-multi-origin"></a>Punto de conexión con varios orígenes de Azure CDN

La compatibilidad con varios orígenes elimina el tiempo de inactividad y crea redundancia global. 

Al elegir varios orígenes en un punto de conexión de Azure CDN, la redundancia dispersa el riesgo al sondear el estado de cada origen y conmutar por error si es necesario.

Configure uno o más grupos de origen y elija un grupo de origen predeterminado. Cada grupo de origen es una colección de uno o varios orígenes que pueden asumir cargas de trabajo similares.

> [!NOTE]
> Actualmente, esta característica solo está disponible en Azure CDN de Microsoft. 

## <a name="create-the-origin-group"></a>Creación del grupo de origen

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com)

2. Seleccione su perfil de Azure CDN y, a continuación, seleccione el punto de conexión que va a configurar para el uso de varios orígenes.

3. Seleccione **Origen** en **Configuración** en la configuración del punto de conexión:

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-1.png" alt-text="Punto de conexión de CDN" border="true":::

4. Para habilitar el uso de varios orígenes, necesita al menos un grupo de origen. Seleccione **Create Origin Group** (Crear grupo de origen):

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-2.png" alt-text="Configuración de origen" border="true":::

5. En la configuración de **Add Origin Group** (Agregar grupo de origen), escriba o seleccione la siguiente información:

   | Configuración           | Value                                                                 |
   |-------------------|-----------------------------------------------------------------------|
   | Nombre del grupo de origen | Escriba un nombre para el grupo de origen.                                   |
   | Estado del sondeo      | Seleccione **Habilitado**. </br> Azure CDN ejecutará los sondeos de estado desde distintos puntos de todo el mundo para determinar el estado del origen. Para evitar costos adicionales, no habilite esta configuración si el grupo de origen actual no está activo.
   | Ruta de acceso de sondeo        | Ruta de acceso del origen que se usa para determinar el estado. |
   | Intervalo de sondeo    | Seleccione un intervalo de sondeo de 1, 2 o 4 minutos.                        |
   | Protocolo de sondeo    | Elija **HTTP** o **HTTPS**.                                         |
   | Método de sondeo      | Seleccione **Head** o **Get**.                                           |
   | Grupo de origen predeterminado | Active la casilla para establecer como grupo de origen predeterminado.
    
   :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-3.png" alt-text="Add origin group" border="true"::: (Agregar grupo de origen)

6. Seleccione **Agregar**.

## <a name="add-multiple-origins"></a>Adición de varios orígenes

1. En la configuración de origen del punto de conexión, seleccione **+ Create Origin** (+ Crear origen):

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-5.png" alt-text="Create origin" border="true"::: (Crear origen)

2. Escriba o seleccione la siguiente información en la configuración de **Add Origin** (Agregar origen):

   | Configuración           | Value                                                                 |
   |-------------------|-----------------------------------------------------------------------|
   | Nombre        | Escriba un nombre para el origen.        |
   | Tipo de origen | Seleccione **Almacenamiento**, **Servicio en la nube**, **Aplicación web** u **Origen personalizado**.                                   |
   | Nombre de host de origen        | Seleccione o escriba el nombre de host de origen.  En la lista desplegable se muestran todos los orígenes disponibles del tipo especificado en la configuración anterior. Si seleccionó **Origen personalizado** como su tipo de origen, escriba el dominio del servidor de origen del cliente. |
   | Encabezado del host de origen    | Escriba el encabezado de host que quiera que Azure CDN envíe con cada solicitud o deje el valor predeterminado.                        |
   | Puerto HTTP   | Escriba el puerto HTTP.                                         |
   | Puerto HTTPS     | Escriba el puerto HTTPS.                                           |
   | Prioridad    | Especifique un número comprendido entre 1 y 5.       |
   | Peso      | Especifique un número comprendido entre 1 y 1000.   |

    > [!NOTE]
    > Cuando se crea un origen en un grupo de origen, se le debe asignar una prioridad y un peso. Si un grupo de origen solo tiene un origen, la prioridad y el peso predeterminados se establecen en 1. El tráfico se enruta a los orígenes de prioridad más alta si el origen es correcto. Si se determina que un origen es incorrecto, las conexiones se desvían a otro origen en orden de prioridad. Si dos orígenes tienen la misma prioridad, el tráfico se distribuye según el peso especificado para el origen. 

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-6.png" alt-text="Add additional origin" border="true"::: (Agregar otro origen)

3. Seleccione **Agregar**.

4. Seleccione **Configure origin** (Configurar origen) para establecer la ruta de acceso de origen para todos los orígenes:

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-7.png" alt-text="Configuración de la ruta de acceso de origen" border="true":::

5. Seleccione **Aceptar**.

## <a name="configure-origins-and-origin-group-settings"></a>Configuración de las opciones de orígenes y del grupo de origen

Una vez que tenga varios orígenes y un grupo de origen, puede agregar o quitar los orígenes en grupos diferentes. Los orígenes dentro del mismo grupo deben atender cargas de trabajo similares. El tráfico se distribuirá en estos orígenes según su estado de mantenimiento, prioridad y valor de peso. 

1. En la configuración de origen del punto de conexión de Azure CDN, seleccione el nombre del grupo de origen que desea configurar:

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-8.png" alt-text="Configuración de las opciones de orígenes y del grupo de origen" border="true":::

2. En **Update Origin Group** (Actualizar el grupo de origen), seleccione **+ Select Origin** (+ Seleccionar origen):

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-9.png" alt-text="Update Origin Group" border="true"::: (Actualizar grupo de origen)

4. Seleccione el origen que se va a agregar al grupo en el cuadro desplegable y elija **Aceptar**.

5. Compruebe que el origen se ha agregado al grupo y, a continuación, seleccione **Guardar**:

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-10.png" alt-text="Verificación de origen adicional agregado al grupo" border="true":::

## <a name="remove-origin-from-origin-group"></a>Eliminación de un origen del grupo de origen

1. En la configuración de origen del punto de conexión de Azure CDN, seleccione el nombre del grupo de origen:

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-8.png" alt-text="Eliminación del origen del grupo" border="true":::

2. Para quitar un origen del grupo de origen, seleccione el icono de la papelera situado junto al origen y elija **Guardar**:

    :::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-11.png" alt-text="Origen de eliminación de la actualización del grupo de origen" border="true":::

## <a name="override-origin-group-with-rules-engine"></a>Invalidación del grupo de origen mediante el motor de reglas

Personalice la forma de distribuir el tráfico a distintos grupos de origen mediante el motor de reglas estándar.

Distribuya el tráfico a un grupo diferente en función de la dirección URL de la solicitud.

1. En el punto de conexión de CDN, seleccione **Motor de reglas** en **Configuración**:

:::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-12.png" alt-text="Motor de reglas" border="true":::

2. Seleccione **+ Add rule** (+ Agregar regla).

3. Escriba un nombre para la regla en **Name** (Nombre).

4. Seleccione **+ Condition** (+ Condición) y, a continuación, **URL path** (Ruta de acceso URL).

5. En el menú desplegable **operator** (operador), seleccione **Contains** (Contiene).

6. En **Value** (Valor), escriba **/images**.

7. Seleccione **+ Add action** (+ Agregar acción) y, a continuación, **Origin group override** (Invalidación de grupo de origen).

8. En **Origin group** (Grupo de origen), seleccione el grupo origen en el cuadro desplegable.

:::image type="content" source="./media/endpoint-multiorigin/endpoint-multiorigin-13.png" alt-text="Condiciones del motor de reglas" border="true":::

Para todas las solicitudes entrantes si la ruta de acceso URL contiene **/images**, la solicitud se asignará al grupo de origen en la sección de acciones **(myorigingroup)** . 

## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha habilitado el punto de conexión con varios orígenes de Azure CDN.

Para más información sobre Azure CDN y los otros servicios de Azure que se mencionan en este artículo, consulte:

* [Azure CDN](./cdn-overview.md)
* [Comparación de las características de los productos de Azure CDN](./cdn-features.md)
