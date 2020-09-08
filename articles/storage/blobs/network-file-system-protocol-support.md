---
title: Compatibilidad de Network File System 3.0 en Azure Blob Storage (versión preliminar) | Microsoft Docs
description: Blob Storage ya es compatible con el protocolo Network File System (NFS) 3.0. Esta compatibilidad permite a los clientes de Windows y Linux montar un contenedor en Blob Storage desde una máquina virtual (VM) de Azure o un equipo que se ejecuta en el entorno local.
author: normesta
ms.subservice: blobs
ms.service: storage
ms.topic: conceptual
ms.date: 08/04/2020
ms.author: normesta
ms.reviewer: yzheng
ms.custom: references_regions
ms.openlocfilehash: 41ad20de6b7a800ff1f97a7ff371c8e0012fed27
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88166984"
---
# <a name="network-file-system-nfs-30-protocol-support-in-azure-blob-storage-preview"></a>Compatibilidad del protocolo Network File System (NFS) 3.0 en Azure Blob Storage (versión preliminar)

Blob Storage ya es compatible con el protocolo Network File System (NFS) 3.0. Esta compatibilidad permite a los clientes de Windows o Linux montar un contenedor en Blob Storage desde una máquina virtual (VM) de Azure o un equipo que se ejecuta en el entorno local. 

> [!NOTE]
> La compatibilidad con el protocolo NFS 3.0 en Azure Blob Storage se encuentra en versión preliminar pública y está disponible en las siguientes regiones: Este de EE. UU., Centro de EE. UU., Centro-oeste de EE. UU., Sudeste de Australia, Europa del Norte, Oeste de Reino Unido, Centro de Corea, Sur de Corea y Centro de Canadá.

## <a name="general-workflow-mounting-a-storage-account-container"></a>Flujo de trabajo general: montaje de un contenedor de cuenta de almacenamiento

Para montar un contenedor de cuenta de almacenamiento, tendrá que realizar estas acciones.

1. Registe la característica de protocolo NFS 3.0 con la suscripción.

2. Compruebe si la característica está registrada.

3. Cree una instancia de Azure Virtual Network (VNet).

4. Configure la seguridad de red.

5. Cree y configure una cuenta de almacenamiento que acepte tráfico solo desde la red virtual.

6. Cree un contenedor en la cuenta de almacenamiento.

7. Monte el contenedor.

Para obtener instrucciones paso a paso, consulte [Montaje de Blob Storage con el protocolo Network File System (NFS) 3.0 (versión preliminar)](network-file-system-protocol-support-how-to.md).

> [!IMPORTANT]
> Es importante completar estas tareas en orden. No se pueden montar contenedores creados antes de habilitar el protocolo NFS 3.0 en la cuenta. Además, después de haber habilitado el protocolo NFS 3.0 en la cuenta, no puede deshabilitarlo.

## <a name="network-security"></a>Seguridad de las redes

La cuenta de almacenamiento debe incluirse en una red virtual. Una red virtual permite a los clientes conectarse de forma segura a su cuenta de almacenamiento. La única manera de proteger los datos en su cuenta es mediante una red virtual y otra configuración de seguridad de red. Las demás herramientas que se usan para proteger los datos, incluidas la autorización de claves de cuenta, la seguridad de Azure Active Directory (AD) y las listas de control de acceso (ACL), no se admiten todavía en cuentas que tienen habilitada la compatibilidad con el protocolo NFS 3.0. 

Para más información, consulte [Recomendaciones de seguridad de red para Blob Storage](security-recommendations.md#networking).

## <a name="supported-network-connections"></a>Conexiones de red compatibles

Un cliente puede conectarse a través de un [punto de conexión privado](../common/storage-private-endpoints.md) o público y desde cualquiera de las siguientes ubicaciones de red:

- La red virtual que configure para la cuenta de almacenamiento. 

  En este artículo, nos referiremos a esa red virtual como *red virtual principal*. Para más información, consulte [Concesión de acceso desde una red virtual](../common/storage-network-security.md#grant-access-from-a-virtual-network).

- Una red virtual emparejada que se encuentre en la misma región que la red virtual principal.

  Tendrá que configurar la cuenta de almacenamiento para permitir el acceso a esta red virtual emparejada. Para más información, consulte [Concesión de acceso desde una red virtual](../common/storage-network-security.md#grant-access-from-a-virtual-network).

- Una red local que esté conectada a la red virtual principal mediante [VPN Gateway](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) o una [puerta de enlace de ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-howto-add-gateway-portal-resource-manager). 

  Para más información, consulte [Configuración del acceso desde redes locales](../common/storage-network-security.md#configuring-access-from-on-premises-networks).

- Una red local que esté conectada a una red emparejada.

  Esta acción puede realizarse mediante [VPN Gateway](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) o una [puerta de enlace de ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-howto-add-gateway-portal-resource-manager) junto con [tránsito de puerta de enlace](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/vnet-peering#gateway-transit). 

> [!IMPORTANT]
> Si se va a conectar desde una red local, asegúrese de que el cliente permite la comunicación saliente a través de los puertos 111 y 2048. El protocolo NFS 3.0 usa estos puertos.

## <a name="azure-storage-features-not-yet-supported"></a>Características de Azure Storage que aún no son compatibles

Las siguientes características de Azure Storage no se admiten cuando se habilita el protocolo NFS 3.0 en la cuenta. 

- Seguridad de Azure Active Directory (AD).

- Listas de control de acceso (ACL) tipo POSIX.

- Capacidad de habilitar la compatibilidad con NFS 3.0 en cuentas de almacenamiento existentes.

- Capacidad de deshabilitar la compatibilidad con NFS 3.0 en una cuenta de almacenamiento (después de haberla habilitado).

- Capacidad de escribir en blobs mediante API REST o SDK. 
  
## <a name="nfs-30-features-not-yet-supported"></a>Características de NFS 3.0 que aún no son compatibles

Todavía no se admiten las siguientes características de NFS 3.0 con Azure Data Lake Storage Gen2.

- NFS 3.0 a través de UDP. Solo se admite NFS 3.0 a través de TCP.

- Bloqueo de archivos con Network Lock Manager (NLM). El montaje de comandos debe incluir el parámetro `-o nolock`.

- Montaje de subdirectorios. Solo puede montar el directorio raíz (contenedor).

- Enumeración de los montajes (por ejemplo: mediante el comando `showmount -a`).

- Enumeración de las exportaciones (por ejemplo: mediante el comando `showmount -e`).

- Exportación de un contenedor como de solo lectura.

## <a name="pricing"></a>Precios

Durante la versión preliminar, los datos almacenados en la cuenta de almacenamiento se facturan según la misma tarifa de capacidad que los cargos de Blob Storage por GB al mes. 

No se cobra ninguna transacción durante la versión preliminar. Los precios de las transacciones están sujetos a cambios y se determinarán cuando esté disponible con carácter general.

## <a name="next-steps"></a>Pasos siguientes

Para comenzar, consulte [Montaje de Blob Storage con el protocolo Network File System (NFS) 3.0 (versión preliminar)](network-file-system-protocol-support-how-to.md).





