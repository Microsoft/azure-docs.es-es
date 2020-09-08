---
title: Costos de software de las reservas para Azure
description: Descubra qué medidores de software no están incluidos en los costos de instancia reservada de máquina virtual de Azure.
author: yashesvi
ms.reviewer: yashar
tags: billing
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 02/13/2020
ms.author: banders
ms.openlocfilehash: e302a8459d3092a5543efda7494c68d6660df39d
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690823"
---
# <a name="software-costs-not-included-with-azure-reserved-vm-instances"></a>Los costos de software no se incluyen en Azure Reserved Virtual Machine Instances.

Los descuentos de la instancia reservada de máquina virtual y de la capacidad reservada de SQL se aplican solo a los costos de infraestructura y no a los costos de software. Si usa VM de Windows y no dispone de la Ventaja híbrida de Azure en las instancias reservadas de máquina virtual, se le cobrará según los medidores de software que se indican en las secciones siguientes. En cuanto a las implementaciones de PaaS de SQL, el costo de IP continuará cobrándose mediante un medidor separado si la Ventaja híbrida de Azure no está seleccionada.

## <a name="windows-software-meters-not-included-in-reservation-cost"></a>Medidores de software de Windows no incluidos en los costos de reservas

| Id. del medidor | Nombre del medidor en el archivo de uso | Usado por máquina virtual |
| ------- | ------------------------| --- |
| e7e152ac-f29c-4cce-ad6e-026192c01ef2 | Instancias reservadas de ráfagas de Windows Server (1 núcleo) | Serie B |
| cac255a2-9f0f-4c62-8bd6-f0fa449c5f76 | Instancias reservadas de ráfagas de Windows Server (2 núcleos) | Serie B |
| 09756b58-3fb5-4390-976d-9ddd14f9ed18 | Instancias reservadas de ráfagas de Windows Server (4 núcleos) | Serie B |
| e828cb37-5920-4dc7-b30f-664e4dbcb6c7 | Instancias reservadas de ráfagas de Windows Server (8 núcleos) | Serie B |
| f65a06cf-c9c3-47a2-8104-f17a8542215a | Instancias reservadas de Windows Server (1 núcleo) | Todos excepto serie B |
| b99d40ae-41fe-4d1d-842b-56d72f3d15ee | Instancias reservadas de Windows Server (2 núcleos) | Todos excepto serie B |
| 1cb88381-0905-4843-9ba2-7914066aabe5 | Instancias reservadas de Windows Server (4 núcleos) | Todos excepto serie B |
| 07d9e10d-3e3e-4672-ac30-87f58ec4b00a | Instancias reservadas de Windows Server (6 núcleos) | Todos excepto serie B |
| 603f58d1-1e96-460b-a933-ce3775ac7e2e | Instancias reservadas de Windows Server (8 núcleos) | Todos excepto serie B |
| 36aaadda-da86-484a-b465-c8b5ab292d71 | Instancias reservadas de Windows Server (12 núcleos) | Todos excepto serie B |
| 02968a6b-1654-4495-ada6-13f378ba7172 | Instancias reservadas de Windows Server (16 núcleos) | Todos excepto serie B |
| 175434d8-75f9-474b-9906-5d151b6bed84 | Instancias reservadas de Windows Server (20 núcleos) | Todos excepto serie B |
| 77eb6dd0-88f5-4a16-ab39-05d1742efb25 | Instancias reservadas de Windows Server (24 núcleos) | Todos excepto serie B |
| 0d5bdf46-b719-4b1f-a780-b9bdfffd0591 | Instancias reservadas de Windows Server (32 núcleos) | Todos excepto serie B |
| f1214b5c-cc16-445f-be6c-a3bb75f8395a | Instancias reservadas de Windows Server (40 núcleos) | Todos excepto serie B |
| 637b7c77-65ad-4486-9cc7-dc7b3e9a8731 | Instancias reservadas de Windows Server (64 núcleos) | Todos excepto serie B |
| da612742-e7cc-4ca3-9334-0fb7234059cd | Instancias reservadas de Windows Server (72 núcleos) | Todos excepto serie B |
| a485cb8c-069b-4cf3-9a8e-ddd84b323da2 | Instancias reservadas de Windows Server (128 núcleos) | Todos excepto serie B |
| 904c5c71-1eb7-43a6-961c-d305a9681624 | Instancias reservadas de Windows Server (256 núcleos) | Todos excepto serie B |
| 6fdab81b-4284-4df9-8939-c237cc7462fe | Instancias reservadas de Windows Server (96 núcleos) | Todos excepto serie B |

## <a name="cloud-services-software-meters-not-included-in-reservation-cost"></a>Medidores de software de servicios en la nube no incluidos en los costos de reservas

| Id. del medidor | Nombre del medidor en el archivo de uso |
| ------- | ------------------------|
|ac9d47ff-ff68-4afc-a145-0c321cf8d0d5|Licencia para 1 vCPU de Cloud Services|
|e0434559-19ee-4132-9c46-05ad4044f3f7|Licencia para 2 vCPU de Cloud Services|
|6ecc834e-39b3-48b3-8d10-cc5626bacb66|Licencia para 4 vCPU de Cloud Services|
|13103090-ca72-4825-ab12-7f16c4931d95|Licencia para 8 vCPU de Cloud Services|
|ecd2bb6e-45a5-49aa-a58b-3947ba21c364|Licencia para 16 vCPU de Cloud Services|
|de2c7f1d-06dc-4b16-bc8b-c2ec5f4c8aee|Licencia para 20 vCPU de Cloud Services|
|ca1af837-4b35-47f5-8d14-b1988149c4ca|Licencia para 32 vCPU de Cloud Services|
|dc72ee45-2ab7-4698-b435-e2cf10d1f9f6|Licencia para 64 vCPU de Cloud Services|
|7a803026-244c-4659-834c-11e6b2d6b76f|Licencia para 80 vCPU de Cloud Services|

## <a name="get-rates-for-azure-meters"></a>Obtener las tarifas de medidores de Azure

El costo de cada uno de estos medidores se puede consultar a través de la API RateCard de Azure. Para obtener información sobre las tasas de un medidor de Azure, consulte [Información de precios y metadatos de los recursos usados en una suscripción de Azure](/previous-versions/azure/reference/mt219004(v=azure.100)).

## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de las reservas de Azure, consulte los siguientes artículos:

- [¿Qué son las reservas para Azure?](save-compute-costs-reservations.md)
- [Pago por adelantado de máquinas virtuales con Azure Reserved VM Instances](../../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [Administración de reservas para Azure](manage-reserved-vm-instance.md)
- [Información sobre cómo se aplica el descuento por la reserva](../manage/understand-vm-reservation-charges.md)
- [Información sobre el uso de reservas para suscripciones de pago por uso](understand-reserved-instance-usage.md)
- [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md)

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Ponerse en contacto con nosotros

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).
