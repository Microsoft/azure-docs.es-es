---
title: Creación de zonas DNS secundarias de Azure
titleSuffix: Azure DNS
description: Tutorial sobre cómo crear zonas DNS secundarias en Azure Portal.
author: jonbeck
ms.assetid: be4580d7-aa1b-4b6b-89a3-0991c0cda897
ms.service: dns
ms.topic: tutorial
ms.custom: ''
ms.workload: infrastructure-services
ms.date: 7/16/2020
ms.author: jonbeck
ms.openlocfilehash: 3f35d39634470ccacffa4d35c272a82725e9001c
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89081372"
---
# <a name="tutorial-creating-a-new-child-dns-zone"></a>Tutorial: Creación de una zona DNS secundaria

En este tutorial, aprenderá a: 

> [!div class="checklist"]
> * Iniciar sesión en Azure Portal.
> * Crear una zona DNS secundaria mediante la nueva zona DNS.
> * Crear una zona DNS secundaria mediante la zona DNS primaria.
> * Comprobar la delegación de NS para la nueva zona DNS secundaria.



## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa.  Si no tiene una cuenta, [puede crear una de forma gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Zona de Azure DNS primaria existente.  

Para este tutorial, usaremos contoso.com como la zona primaria y subdomain.contoso.com como el nombre de dominio secundario.  Reemplace *contoso.com* por el nombre de dominio primario y *subdomain* por el dominio secundario.  Si no ha creado la zona DNS primaria, consulte los pasos para [crear una zona DNS con Azure Portal](https://docs.microsoft.com/azure/dns/dns-getstarted-portal#create-a-dns-zone). 


## <a name="sign-in-to-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/) con su cuenta de Azure.
Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar.

Hay dos maneras de crear la zona DNS secundaria.
1.  A través de la página del portal "Crear zona DNS".
1.  A través de la página de configuración de la zona DNS primaria.


## <a name="create-child-dns-zone-via-create-dns-zone"></a>Creación de una zona DNS secundaria mediante Crear zona DNS

En este paso, se creará una nueva zona DNS secundaria con el nombre **subdomain.contoso.com** y se delegará en la zona DNS primaria existente: **contoso.com**. Creará la zona DNS con las pestañas de la página **Crear zona DNS**.
1.  En el menú de Azure Portal o en la **página principal**, seleccione **Crear un recurso**. Aparece la ventana **Nuevo**.
1.  Seleccione **Redes**, seleccione **Zona DNS** y, a continuación, seleccione el botón **Agregar**.

1.  En la pestaña **Aspectos básicos**, escriba o seleccione los siguientes valores:
    * **Suscripción**: Seleccione una suscripción en la que crear la zona.
    * **Grupo de recursos**: escriba el grupo de recursos existente o puede crear uno nuevo; para ello, seleccione **Crear nuevo**, escriba *MyResourceGroup* y seleccione **Aceptar**. El nombre del grupo de recursos debe ser único dentro de la suscripción de Azure.
    * Active esta casilla: **Esta zona es un elemento secundario de una zona existente que ya está hospedada en Azure DNS**.
    * **Suscripción de la zona primaria**: en esta lista desplegable, busque o seleccione el nombre de la suscripción en la que se creó la zona DNS primaria *contoso.com*.
    * **Zona primaria**: en la barra de búsqueda, escriba *contoso.com* para cargarlo en la lista desplegable. Una vez cargado, seleccione *contoso.com* en la lista desplegable.
    * **Nombre:** Escriba *subdomain* para este ejemplo de tutorial. Tenga en cuenta que el nombre de la zona DNS primaria *contoso.com* se agrega automáticamente como sufijo al nombre al seleccionar la zona primaria en el paso anterior.

1. Seleccione **Siguiente: Review + create** (Revisar y crear).
1. En la pestaña **Revisar y crear**, revise el resumen, corrija los errores de validación y, después, seleccione **Crear**.
La creación de la zona puede tardar unos minutos.

 
    :::image type="content" source="./media/dns-delegate-domain-azure-dns/create-dns-zone-inline.png" alt-text="Captura de pantalla de la página Crear zona DNS." lightbox="./media/dns-delegate-domain-azure-dns/create-dns-zone-expanded.png":::

## <a name="create-child-dns-zone-via-parent-dns-zone-overview-page"></a>Creación de una zona DNS secundaria mediante la página de información general de zona DNS primaria
También puede crear una nueva zona DNS secundaria y delegarla en la zona DNS primaria mediante el botón **Zona secundaria** de la página de información general de la zona primaria. El uso de este botón rellena automáticamente los parámetros de la zona primaria de la zona secundaria. 

1.  En Azure Portal, en **Todos los recursos**, abra la zona DNS *contoso.com* en el grupo de recursos **MyResourceGroup**. Puede escribir *contoso.com* en el cuadro **Filtrar por nombre** para encontrarlo más fácilmente.
1.  En la página de información general de la zona DNS, seleccione el botón **+ Zona secundaria**.

      :::image type="content" source="./media/dns-delegate-domain-azure-dns/create-child-zone-inline.png" alt-text="Captura de pantalla del botón Zona secundaria." border="true" lightbox="./media/dns-delegate-domain-azure-dns/create-child-zone-expanded.png":::

1.  Se abrirá la página Crear zona DNS. La opción de zona secundaria ya está activada y la suscripción de la zona primaria y la zona primaria ya se han rellenado en esta página.
1.  Escriba *child* como nombre para este ejemplo de tutorial. Tenga en cuenta que el nombre de la zona DNS primaria, contoso.com, se agrega automáticamente como prefijo al nombre.
1.  Seleccione **Siguiente: Etiquetas** y, a continuación, **Siguiente: Review + create** (Revisar y crear).
1.  En la pestaña **Revisar y crear**, revise el resumen, corrija los errores de validación y, después, seleccione **Crear**.

    :::image type="content" source="./media/dns-delegate-domain-azure-dns/create-dns-zone-child-inline.png" alt-text="Captura de pantalla de la zona secundaria seleccionada" border="true"  lightbox="./media/dns-delegate-domain-azure-dns/create-dns-zone-child-expanded.png":::
## <a name="verify-child-dns-zone"></a>Comprobación de la zona DNS secundaria
Ahora que ha creado nueva zona DNS secundaria *subdomain.contoso.com*, compruebe que la delegación se ha realizado correctamente; para ello, compruebe los registros de servidor de nombres (NS) para la zona secundaria se encuentren en la zona primaria, tal como se describe a continuación.  

**Recuperación de los servidores de nombres de la zona DNS secundaria:**

1.  En Azure Portal, en **Todos los recursos**, abra la zona DNS *subdomain.contoso.com* en el grupo de recursos **MyResourceGroup**. Puede escribir *subdomain.contoso.com* en el cuadro **Filtrar por nombre** para encontrarlo más fácilmente.
1.  Recupere los servidores de nombres de la página de información general de zona DNS. En este ejemplo, a la zona contoso.com se le han asignado los servidores de nombres *ns1-08.azure-dns.com, ns2-08.azure-dns.net, ns3-08.azure-dns.org* y *ns4-08.azure-dns.info*:

      :::image type="content" source="./media/dns-delegate-domain-azure-dns/create-child-zone-ns-inline.png" alt-text="Captura de pantalla de los servidores de nombres de la zona secundaria" border="true" lightbox="./media/dns-delegate-domain-azure-dns/create-child-zone-ns-expanded.png":::
**Compruebe el registro de NS en la zona DNS primaria:**

Ahora, en este paso, diríjase a la zona DNS primaria *contoso.com* y compruebe que se ha creado su entrada de conjunto de registros de NS para los servidores de nombres de las zonas secundarias.

1. En Azure Portal, en **Todos los recursos**, abra la zona DNS contoso.com en el grupo de recursos **MyResourceGroup**. Puede escribir contoso.com en el cuadro **Filtrar por nombre** para encontrarlo más fácilmente.
1.  En la página de información general de las zonas DNS de *contoso.com*, busque los conjuntos de registros.
1.  Observará que ya se creó el conjunto de registros de tipo NS y nombre subdominio en la zona DNS primaria. Compruebe los valores de este conjunto de registros sea similar a la lista de nombres de servidor recuperados de la zona DNS secundaria en el paso anterior.

     :::image type="content" source="./media/dns-delegate-domain-azure-dns/create-child-zone-ns-validate-inline.png" alt-text="Captura de pantalla de la validación de servidores de nombres de la zona secundaria" border="true" lightbox="./media/dns-delegate-domain-azure-dns/create-child-zone-ns-validate-expanded.png":::
## <a name="clean-up-resources"></a>Limpieza de recursos
Cuando ya no necesite los recursos que creó en este tutorial, quítelos mediante la eliminación del grupo de recursos **MyResourceGroup**. Abra el grupo de recursos **MyResourceGroup** y seleccione **Eliminar grupo de recursos**.



## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Escenarios de Azure DNS Private Zones](private-dns-scenarios.md)
