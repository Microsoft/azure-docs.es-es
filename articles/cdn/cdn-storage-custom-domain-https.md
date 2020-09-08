---
title: Acceso a blobs de almacenamiento mediante un dominio personalizado de Azure CDN a través de HTTPS
description: Aprenda a agregar un dominio personalizado de Azure CDN y a habilitar HTTPS en ese dominio para el punto de conexión personalizado de Blob Storage.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 06/15/2018
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: 5b6fe2b2704f101a7775b7eb700375105b0a9eca
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "81259891"
---
# <a name="tutorial-access-storage-blobs-using-an-azure-cdn-custom-domain-over-https"></a>Tutorial: Acceso a blobs de almacenamiento mediante un dominio personalizado de Azure CDN a través de HTTPS

Después de integrar su cuenta de almacenamiento de Azure con Azure Content Delivery Network (CDN), puede agregar un dominio personalizado y habilitar HTTPS en ese dominio para el punto de conexión de almacenamiento de blobs personalizado. 

## <a name="prerequisites"></a>Prerrequisitos

Para poder completar los pasos de este tutorial, primero debe integrar su cuenta de almacenamiento de Azure con Azure CDN. Para más información, consulte [Inicio rápido: Integración de una cuenta de una instancia de Azure Storage con Azure CDN](cdn-create-a-storage-account-with-cdn.md).

## <a name="add-a-custom-domain"></a>Adición de un dominio personalizado
Después de crear un punto de conexión de CDN en el perfil, su nombre, que es un subdominio de azureedge.net, se incluye de forma predeterminada en la dirección URL de entrega de contenido de la red CDN. También puede asociar un dominio personalizado a un punto de conexión de CDN. Con esta opción, entrega el contenido con un dominio personalizado en la dirección URL, en lugar de un nombre de punto de conexión. Para agregar un dominio personalizado al punto de conexión, siga las instrucciones de este tutorial: [Incorporación de un dominio personalizado a un punto de conexión de Azure CDN](cdn-map-content-to-custom-domain.md).

## <a name="configure-https"></a>Configuración de HTTPS
Mediante el protocolo HTTPS en su dominio personalizado, se asegura de que los datos se entregan de manera segura en internet a través de cifrado TLS/SSL. Cuando el explorador web se conecta a un sitio web a través de HTTPS, valida el certificado de seguridad del sitio web y comprueba que lo ha emitido una entidad de certificación legítima. Para configurar HTTPS en su dominio personalizado, siga las instrucciones de este tutorial: [Configuración de HTTPS en un dominio personalizado de Azure CDN](cdn-custom-ssl.md).

## <a name="shared-access-signatures"></a>Las firmas de acceso compartido
Si el punto de conexión de Blob Storage está configurado para no permitir el acceso de lectura anónimo, debe proporcionar un token de [Firma de acceso compartido (SAS)](cdn-sas-storage-support.md) en cada solicitud que realice a su dominio personalizado. De forma predeterminada, los puntos de conexión de Blob Storage no permiten el acceso de lectura anónimo. Para más información acerca de SAS, consulte [Manage anonymous read access to containers and blobs](../storage/blobs/storage-manage-access-to-resources.md) (Administración del acceso de lectura anónimo a contenedores y blobs).

Azure CDN pasa por alto las restricciones agregadas al token de SAS. Por ejemplo, todos los tokens de SAS tienen una fecha de expiración, lo que significa que se puede seguir accediendo al contenido mediante una SAS que haya expirado hasta que ese contenido se purgue desde los servidores de punto de presencia (POP) de la red CDN. Puede controlar cuánto tiempo se almacenan en caché los datos en Azure CDN mediante el establecimiento del encabezado de respuesta de caché. Para más información, consulte [Administración de la expiración de Azure Storage Blob en Azure CDN](cdn-manage-expiration-of-blob-content.md).

Si crea varias direcciones URL de SAS para el mismo punto de conexión de blob, considere habilitar el almacenamiento en caché de la cadena de consulta. Así, se asegura de que cada dirección URL se trata como una entidad única. Para más información, consulte [Control del comportamiento del almacenamiento en caché de Azure CDN con cadenas de consulta](cdn-query-string.md).

## <a name="http-to-https-redirection"></a>Redireccionamiento de HTTP a HTTPS
Puede elegir redirigir el tráfico HTTP a HTTPS mediante la creación de una regla de redireccionamiento de direcciones URL con el [motor de reglas estándar](cdn-standard-rules-engine.md) o el [motor de reglas de Verizon Premium](cdn-verizon-premium-rules-engine.md). El motor de reglas estándar solo está disponible para Azure CDN desde perfiles de Microsoft, mientras que el motor de reglas de Verizon Premium solo está disponible en Azure CDN Premium desde perfiles de Verizon.

![Regla de redireccionamiento de Microsoft](./media/cdn-storage-custom-domain-https/cdn-standard-redirect-rule.png)

En la regla anterior, si se dejan el nombre de host, la ruta de acceso, la cadena de consulta y el fragmento, se usarán los valores entrantes en el redireccionamiento. 

![Regla de redireccionamiento de Verizon](./media/cdn-storage-custom-domain-https/cdn-url-redirect-rule.png)

En la regla anterior, *Cdn-endpoint-name* hace referencia al nombre que ha configurado para el punto de conexión de la red CDN, que puede seleccionar en la lista desplegable. El valor de *origin-path* hace referencia a la ruta de acceso dentro de la cuenta de almacenamiento de origen, en donde reside el contenido estático. Si hospeda todo el contenido estático en un mismo contenedor, reemplace *origin-path* por el nombre del contenedor.

## <a name="pricing-and-billing"></a>Precios y facturación
Cuando tiene acceso a blobs mediante Azure CDN, paga [tarifas de Blob Storage](https://azure.microsoft.com/pricing/details/storage/blobs/) para el tráfico entre los servidores POP y el origen (Blob Storage) y [tarifas de Azure CDN](https://azure.microsoft.com/pricing/details/cdn/) para los datos a los que se accede desde los servidores POP.

Si, por ejemplo, tiene una cuenta de almacenamiento en Estados Unidos a la que se accede con Azure CDN y alguien en Europa intenta acceder a uno de los blobs en esa cuenta de almacenamiento a través de Azure CDN, Azure CDN comprueba primero el POP más cercano en Europa para ese blob. Si encuentra uno, Azure CDN accede a esa copia del blob y usa las tarifas de la red CDN, ya que tiene acceso a él en Azure CDN. Si no encuentra ninguno, Azure CDN copia el blob en el servidor POP, lo que generará gastos de salida y transacción, tal como se especifica en las tarifas de Blob Storage, y obtendrá acceso al archivo en el servidor POP, lo que dará lugar a facturación según las tarifas de Azure CDN.

## <a name="next-steps"></a>Pasos siguientes
[Tutorial: Establecimiento de las reglas de almacenamiento en caché de Azure CDN](cdn-caching-rules-tutorial.md)




