---
title: Preguntas frecuentes sobre Azure DNS privado
description: En este artículo, consulte las preguntas frecuentes sobre Azure DNS privado.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: article
ms.date: 10/05/2019
ms.author: rohink
ms.openlocfilehash: 0945f89e3c18ab20ece0b493a49ec9f7f31e4e13
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87529201"
---
# <a name="azure-private-dns-faq"></a>Preguntas frecuentes sobre Azure DNS privado

Las siguientes son preguntas frecuentes sobre Azure DNS privado.

## <a name="does-azure-dns-support-private-domains"></a>¿Azure DNS admite dominios privados?

Los dominios privados se admiten mediante la característica de zonas DNS privadas de Azure. Las zonas DNS privadas solo se pueden resolver desde dentro de las redes virtuales especificadas. Para más información, consulte la [introducción](private-dns-overview.md).

Para sobre otras opciones de DNS internas de Azure, consulte [Resolución de nombres para las máquinas virtuales e instancias de rol](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

## <a name="will-azure-private-dns-zones-work-across-azure-regions"></a>¿Funcionarán las zonas DNS privadas de Azure entre regiones de Azure?

Sí. Las zonas privadas se admiten para la resolución DNS entre redes virtuales de distintas regiones de Azure. Las zonas privadas funcionan incluso sin emparejamiento explícito de las redes virtuales. Todas las redes virtuales están vinculadas a la zona DNS privada.

## <a name="is-connectivity-to-the-internet-from-virtual-networks-required-for-private-zones"></a>Para las zonas privadas, ¿se requiere que las redes virtuales tengan conectividad con Internet?

No. Las zonas privadas funcionan junto con las redes virtuales. Úselas para administrar los dominios de las máquinas virtuales u otros recursos dentro y entre redes virtuales. No se requiere conectividad con Internet para la resolución de nombres.

## <a name="can-the-same-private-zone-be-used-for-several-virtual-networks-for-resolution"></a>¿Se puede usar la misma zona privada para varias redes virtuales para la resolución?

Sí. Puede vincular una zona DNS privada con miles de redes virtuales. Para más información, consulte [Límites de Azure DNS](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#azure-dns-limits).

## <a name="can-a-virtual-network-that-belongs-to-a-different-subscription-be-linked-to-a-private-zone"></a>¿Se puede vincular a una zona privada una red virtual que pertenezca a otra suscripción?

Sí. Debe tener permiso para la operación de escritura en las redes virtuales y en la zona DNS privada. El permiso de escritura se puede conceder a varios roles de Azure. Por ejemplo, el rol de Azure de Colaborador de la red clásica tiene permisos de escritura en las redes virtuales y el rol de Colaborador de zonas DNS privadas tiene permisos de escritura en las zonas DNS privadas. Para obtener más información sobre los roles de Azure, consulte el artículo sobre el [Control de acceso basado en roles](../role-based-access-control/overview.md).

## <a name="will-the-automatically-registered-virtual-machine-dns-records-in-a-private-zone-be-automatically-deleted-when-you-delete-the-virtual-machine"></a>¿Se eliminarán automáticamente los registros de DNS de una máquina virtual registrada en una zona privada cuando se elimine la máquina virtual?

Sí. Si elimina una máquina virtual que se encuentra dentro de una red virtual vinculada que tiene habilitado el registro automático, se eliminan automáticamente los registros anotados.

## <a name="can-an-automatically-registered-virtual-machine-record-in-a-private-zone-from-a-linked-virtual-network-be-deleted-manually"></a>¿Se puede eliminar manualmente un registro de máquina virtual registrado automáticamente en una zona privada de una red virtual vinculada?

Sí. Puede sobrescribir los registros de DNS que se registraron automáticamente con un registro de DNS que se creó de manera manual en la zona. La siguiente pregunta y su respuesta abordan este tema.

## <a name="what-happens-when-i-try-to-manually-create-a-new-dns-record-into-a-private-zone-that-has-the-same-hostname-as-an-automatically-registered-existing-virtual-machine-in-a-linked-virtual-network"></a>¿Qué ocurre cuando se intenta crear manualmente un registro de DNS en una zona privada que tiene el mismo nombre de host que una máquina virtual existente registrada de manera automática en una red virtual vinculada?

Intenta crear manualmente un registro de DNS en una zona privada con el mismo nombre de host que una máquina virtual existente registrada de manera automática en una red virtual vinculada. Al hacerlo, el nuevo registro de DNS sobrescribe el registro de máquina virtual que se registró automáticamente. Si intenta eliminar nuevamente este registro de DNS que se creó de manera manual de la zona, la eliminación se realizará correctamente. El registro automático vuelve a ocurrir siempre que la máquina virtual siga existiendo y tenga adjunta una dirección IP privada. El registro de DNS se vuelve a crear de manera automática en la zona.

## <a name="what-happens-when-we-unlink-a-linked-virtual-network-from-a-private-zone-will-the-automatically-registered-virtual-machine-records-from-the-virtual-network-be-removed-from-the-zone-too"></a>¿Qué ocurre si desvinculamos una red virtual vinculada de una zona privada? ¿Los registros de máquina virtual registrados automáticamente de la red virtual también se quitarán de la zona?

Sí. Para desvincular una red virtual vinculada de una zona privada, debe actualizar la zona DNS para quitar el vínculo de la red virtual asociada. En este proceso, los registros de máquina virtual que se registraron automáticamente se quitan de la zona.

## <a name="what-happens-when-we-delete-a-linked-virtual-network-thats-linked-to-a-private-zone-do-we-have-to-manually-update-the-private-zone-to-unlink-the-virtual-network-as-a-linked-virtual-network-from-the-zone"></a>¿Qué ocurre cuando se elimina una red virtual vinculada a una zona privada? ¿Es necesario actualizar manualmente la zona privada para desvincular la red virtual vinculada de la zona?

No. Cuando se elimina una red virtual vinculada sin desvincularla primero de una zona privada, la operación de eliminación se realiza correctamente y los vínculos a la zona DNS se eliminan de forma automática.

## <a name="will-dns-resolution-by-using-the-default-fqdn-internalcloudappnet-still-work-even-when-a-private-zone-for-example-privatecontosocom-is-linked-to-a-virtual-network"></a>¿Funcionará la resolución de DNS con el valor de FQDN predeterminado (internal.cloudapp.net) aunque una zona privada (por ejemplo, private.contoso.com) esté vinculada a una red virtual?

Sí. Las zonas privadas no reemplazan la zona internal.cloudapp.net proporcionada por Azure. Ya sea si confía en la zona internal.cloudapp.net proporcionada por Azure o en su propia zona privada, use el nombre de dominio completo de la zona en la que quiere la resolución.

## <a name="will-the-dns-suffix-on-virtual-machines-within-a-linked-virtual-network-be-changed-to-that-of-the-private-zone"></a>¿Se cambiará el sufijo DNS en las máquinas virtuales de una red virtual vinculada por el de la zona privada?

No. El sufijo DNS de las máquinas virtuales en la red virtual vinculada permanecerá como el sufijo predeterminado proporcionado por Azure ("*.internal.cloudapp.net"). En las máquinas virtuales, dicho sufijo DNS se puede cambiar manualmente por el de la zona privada.
Para instrucciones sobre cómo cambiar este sufijo, consulte [Uso de DNS dinámico para registrar nombres de host en su propio servidor DNS](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-ddns#windows-clients).

## <a name="what-are-the-usage-limits-for-azure-dns-private-zones"></a>¿Qué son los límites de uso de zonas privadas de Azure DNS?

Consulte [Límites de Azure DNS](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#azure-dns-limits) para más información sobre los límites de uso de zonas privadas de Azure DNS.

## <a name="why-dont-my-existing-private-dns-zones-show-up-in-new-portal-experience"></a>¿Por qué no aparecen las zonas DNS privadas existentes en la nueva experiencia del portal?

Si la zona DNS privada existente se creó con la API de la versión preliminar, debe migrar estas zonas al nuevo modelo de recurso. Las zonas DNS privadas creadas con la API de la versión preliminar no se mostrarán en la nueva experiencia del portal. Vea a continuación cómo migrar al nuevo modelo de recursos.

## <a name="how-do-i-migrate-my-existing-private-dns-zones-to-the-new-model"></a>¿Cómo se migran las zonas DNS privadas existentes al nuevo modelo?

Se recomienda realizar la migración al nuevo modelo tan pronto como sea posible. Aunque se admitirá el modelo de recursos heredado, no se desarrollarán más características basadas en él. En el futuro, está previsto dejar de usarlo en favor del nuevo modelo de recursos. Para instrucciones sobre cómo migrar las zonas DNS privadas al nuevo modelo de recursos, consulte la [guía de migración de las zonas privadas de Azure DNS](private-dns-migration-guide.md).

## <a name="next-steps"></a>Pasos siguientes

- [Más información sobre Azure DNS privado](private-dns-overview.md)
