---
title: Arranque del sistema operativo. El equipo se reinició de manera inesperada o se produjo un error inesperado.
description: En este artículo se indican los pasos que hay que seguir para resolver problemas en los que la máquina virtual experimenta un reinicio o sufre un error inesperado durante la instalación de Windows.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 1d249a4e-71ba-475d-8461-31ff13e57811
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 06/22/2020
ms.author: v-mibufo
ms.openlocfilehash: 186b1c46303be59e191a1754361e07a2003b997a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87036189"
---
# <a name="os-start-up--computer-restarted-unexpectedly-or-encountered-an-unexpected-error"></a>Arranque del sistema operativo. El equipo se reinició de manera inesperada o se produjo un error inesperado.

En este artículo se indican los pasos que hay que seguir para resolver problemas en los que la máquina virtual experimenta un reinicio o sufre un error inesperado durante la instalación de Windows.

## <a name="symptom"></a>Síntoma

Si usa [Diagnósticos de arranque](./boot-diagnostics.md) para ver la captura de pantalla de la VM, verá que esta captura muestra una instalación de Windows que falla con el error siguiente:

**El equipo se reinició de manera inesperada o se produjo un error inesperado. No se puede continuar con la instalación de Windows. Para instalar Windows, haga clic en "Aceptar" para reiniciar el equipo y, después, reinicie la instalación.**

![Error durante la instalación de Windows: El equipo se reinició de manera inesperada o se produjo un error inesperado. No se puede continuar con la instalación de Windows. Para instalar Windows, haga clic en "Aceptar" para reiniciar el equipo y, después, reinicie la instalación.](./media/unexpected-restart-error-during-vm-boot/1.png)
 
![Error cuando el programa de instalación de Windows está iniciando los servicios: El equipo se reinició de manera inesperada o se produjo un error inesperado. No se puede continuar con la instalación de Windows. Para instalar Windows, haga clic en "Aceptar" para reiniciar el equipo y, después, reinicie la instalación.](./media/unexpected-restart-error-during-vm-boot/2.png)

## <a name="cause"></a>Causa

El equipo está intentando realizar un arranque inicial de una [imagen generalizada](/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation), pero tiene problemas debido a un archivo de respuesta personalizado (unattend.xml) que se está procesando. Los archivos de respuesta personalizados no se admiten en Azure. 

El archivo de respuesta es un archivo XML especial que contiene definiciones y valores para las opciones de configuración que quiere automatizar durante una instalación del sistema operativo Windows Server. Las opciones de configuración incluyen instrucciones sobre cómo crear discos de partición, dónde encontrar la imagen de Windows que se va a instalar, las claves de producto que se aplicarán y otros comandos que quiere ejecutar.

Los archivos de respuesta personalizados no se admiten en Azure. Si ha especificado un archivo **Unattend.XML** personalizado mediante la opción `sysprep /unattend:<your file’s name>`, puede producirse este error.

En Azure, use la opción **Iniciar la configuración rápida (OOBE) del sistema** en **Sysprep.exe** o use `sysprep /oobe` en vez del archivo Unattend.xml.

Este problema aparece con mayor frecuencia si utiliza **Sysprep.exe** con una máquina virtual local para cargar una máquina virtual generalizada en Azure. En esta situación, puede que también le interese saber cómo se carga correctamente una máquina virtual generalizada.

## <a name="solution"></a>Solución

### <a name="replace-unattended-answer-file-option"></a>Reemplazo de la opción de archivo de respuesta no esperado

Esta situación se produce cuando una imagen se ha preparado para usarse en Azure, pero ha usado un archivo de respuesta personalizado que no se admite en Azure y usted ha usado **SYSPREP** con una marca similar al comando siguiente:

`sysprep /oobe /generalize /unattend:<NameOfYourAnswerFile.XML> /shutdown`

- En el comando anterior, reemplace `<NameOfYourAnswerFile.XML>` por el nombre del archivo.

Para corregir este problema, siga las [instrucciones de Azure para preparar o capturar una imagen](../windows/upload-generalized-managed.md) y prepare una nueva imagen generalizada. Durante sysprep, no utilice la marca `/unattend:<answerfile>`, sino solamente las marcas siguientes:

`sysprep /oobe /generalize /shutdown`

- **Configuración rápida** (OOBE) es la configuración admitida en las máquinas virtuales de Azure.

También puede usar la **interfaz gráfica de usuario de la herramienta de preparación del sistema** para realizar la misma tarea que con el comando anterior, seleccionando las opciones que se muestran a continuación:

- Iniciar la configuración rápida (OOBE) del sistema
- Generalize
- Shutdown
 
![Ventana de la herramienta de preparación del sistema con las opciones OOBE, Generalizar y Apagado seleccionadas.](./media/unexpected-restart-error-during-vm-boot/3.png)
