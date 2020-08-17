---
title: Creación de la interfaz de usuario de una aplicación Angular con la API de Azure Cosmos DB para MongoDB (tercera parte)
description: Parte 3 de la serie de tutoriales en la que se explica cómo crear una aplicación de MongoDB con Angular y Node en Azure Cosmos DB utilizando las mismas API que para MongoDB.
author: johnpapa
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 12/26/2018
ms.author: jopapa
ms.custom: seodec18, devx-track-javascript
ms.reviewer: sngun
ms.openlocfilehash: e9cb30ca64fe697f9b8304f55381709083b8fbd2
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87424246"
---
# <a name="create-an-angular-app-with-azure-cosmos-dbs-api-for-mongodb---build-the-ui-with-angular"></a>Creación de una aplicación de Angular con la API de Azure Cosmos DB para MongoDB: Compilación de la interfaz de usuario con Angular

En este tutorial de varias partes se muestra cómo crear una nueva aplicación escrita en Node.js con Express y Angular y cómo conectarla a la [cuenta de Cosmos configurada con la API de Cosmos DB para MongoDB](mongodb-introduction.md).

La parte 3 del tutorial se basa en la [parte 2](tutorial-develop-mongodb-nodejs-part2.md) y aborda las tareas siguientes:

> [!div class="checklist"]
> * Creación de la interfaz de usuario de Angular
> * Uso de CSS para establecer la apariencia y funcionamiento
> * Prueba de la aplicación de forma local

## <a name="video-walkthrough"></a>Tutorial en vídeo

> [!VIDEO https://www.youtube.com/embed/MnxHuqcJVoM]

## <a name="prerequisites"></a>Requisitos previos

Antes de iniciar esta parte del tutorial, asegúrese de que ha completado los pasos descritos en la [parte 2](tutorial-develop-mongodb-nodejs-part2.md) del tutorial.

> [!TIP]
> Este tutorial le guía por los pasos necesarios para crear la aplicación paso a paso. Si desea descargar el proyecto finalizado, puede obtener la aplicación completa en el [repositorio de angular-cosmosdb](https://github.com/Azure-Samples/angular-cosmosdb) en GitHub.

## <a name="build-the-ui"></a>Creación de la interfaz de usuario

1. En Visual Studio Code, haga clic en el botón Detener :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part3/stop-button.png"::: para detener la aplicación Node. 

2. En la ventana del símbolo del sistema de Windows o Terminal de Mac, escriba el siguiente comando para generar un componente Heroes. En este código, g = generar, c = componente, heroes = nombre del componente, y se usa una estructura de archivo plano (--flat) por lo que no se crea una subcarpeta.

    ```
    ng g c heroes --flat 
    ```

    La ventana de terminal muestra la confirmación de los nuevos componentes.

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part3/install-heros-component.png" alt-text="Instalación del componente Hero":::

    A continuación, se pueden visualizar los archivos que se han creado y actualizado. 

3. En Visual Studio Code, en el panel **Explorer**, vaya a la nueva carpeta **src\app** y abra el nuevo archivo **heroes.component.ts** generado en la carpeta de la aplicación. Este archivo de componente de TypeScript se creó mediante el comando anterior.

    > [!TIP]
    > Si la carpeta de la aplicación no se muestra en Visual Studio Code, escriba CMD + MAYÚS P en un Mac o Ctrl + Mayús + P en Windows para abrir la paleta de comandos y, a continuación, escriba *Reload Window* (Volver a cargar la ventana) para recoger el cambio del sistema.

4. En la misma carpeta, abra el archivo **app.module.ts** y observe que se agregó `HeroesComponent` a las declaraciones en la línea 5 y se importó también en la línea 10.

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part3/app-module-file.png" alt-text="Apertura del archivo app-module.ts":::

5. Vuelva al archivo **heroes.component.html** y copie en él este código. El marcador `<div>` es el contenedor de la página completa. Dentro del contenedor hay una lista de héroes que es necesario crear para que, al hacer clic en uno, se pueda seleccionar y editar o eliminar en la interfaz de usuario. A continuación, en el código HTML debemos aplicar algún estilo para que se sepa cuál se ha seleccionado. También hay un área de edición para que pueda agregar un nuevo héroe o editar uno existente. 

    ```html
    <div>
      <ul class="heroes">
        <li *ngFor="let hero of heroes" (click)="onSelect(hero)" [class.selected]="hero === selectedHero">
          <button class="delete-button" (click)="deleteHero(hero)">Delete</button>
          <div class="hero-element">
            <div class="badge">{{hero.id}}</div>
            <div class="name">{{hero.name}}</div>
            <div class="saying">{{hero.saying}}</div>
          </div>
        </li>
      </ul>
      <div class="editarea">
        <button (click)="enableAddMode()">Add New Hero</button>
        <div *ngIf="selectedHero">
          <div class="editfields">
            <div>
              <label>id: </label>
              <input [(ngModel)]="selectedHero.id" placeholder="id" *ngIf="addingHero" />
              <label *ngIf="!addingHero" class="value">{{selectedHero.id}}</label>
            </div>
            <div>
              <label>name: </label>
              <input [(ngModel)]="selectedHero.name" placeholder="name" />
            </div>
            <div>
              <label>saying: </label>
              <input [(ngModel)]="selectedHero.saying" placeholder="saying" />
            </div>
          </div>
          <button (click)="cancel()">Cancel</button>
          <button (click)="save()">Save</button>
        </div>
      </div>
    </div>
    ```

7. Ahora que tenemos el código HTML, tenemos que agregarlo al archivo **heroes.component.ts** para que podamos interactuar con la plantilla. El código siguiente agrega la plantilla a su archivo de componentes. Se ha agregado un constructor que obtiene algunos hérores e inicializa el componente del servicio Hero para obtener todos los datos. Este código también agrega todos los métodos necesarios para controlar los eventos en la interfaz de usuario. Puede copiar el código siguiente sobre el código existente en **heroes.component.ts**. Se espera que aparezcan errores en las áreas Hero y HeroService, ya que los componentes correspondientes aún no se han importado. Estos errores los solucionará en la sección siguiente. 

    ```ts
    import { Component, OnInit } from '@angular/core';

    @Component({
      selector: 'app-heroes',
      templateUrl: './heroes.component.html',
      styleUrls: ['./heroes.component.scss']
    })
    export class HeroesComponent implements OnInit {
      addingHero = false;
      heroes: any = [];
      selectedHero: Hero;
    
      constructor(private heroService: HeroService) {}
    
      ngOnInit() {
       this.getHeroes();
      }

      cancel() {
        this.addingHero = false;
        this.selectedHero = null;
      }

      deleteHero(hero: Hero) {
        this.heroService.deleteHero(hero).subscribe(res => {
          this.heroes = this.heroes.filter(h => h !== hero);
          if (this.selectedHero === hero) {
            this.selectedHero = null;
          }
        });
      }

      getHeroes() {
        return this.heroService.getHeroes().subscribe(heroes => {
          this.heroes = heroes;
        });
      }

      enableAddMode() {
        this.addingHero = true;
        this.selectedHero = new Hero();
      }

      onSelect(hero: Hero) {
        this.addingHero = false;
        this.selectedHero = hero;
      }

      save() {
        if (this.addingHero) {
          this.heroService.addHero(this.selectedHero).subscribe(hero => {
            this.addingHero = false;
            this.selectedHero = null;
            this.heroes.push(hero);
          });
        } else {
          this.heroService.updateHero(this.selectedHero).subscribe(hero => {
            this.addingHero = false;
            this.selectedHero = null;
          });
        }
      }
    }
    ```

8. En **Explorer**, abra el archivo **app/app.module.ts** y actualice la sección de importaciones para agregar una importación para `FormsModule`. La sección de importación debe tener el siguiente aspecto:

    ```
    imports: [
      BrowserModule,
      FormsModule
    ],
    ```

9. En el archivo **app/app.module.ts** agregue una importación para el nuevo módulo FormsModule en la línea 3. 

    ```
    import { BrowserModule } from '@angular/platform-browser';
    import { NgModule } from '@angular/core';
    import { FormsModule } from '@angular/forms';
    ```

## <a name="use-css-to-set-the-look-and-feel"></a>Uso de CSS para establecer la apariencia y funcionamiento

1. En el panel Explorer, abra el archivo **src/styles.scss**.

2. Copie el código siguiente en el archivo **styles.scss**, reemplazando el contenido existente del archivo.

    ```css
    /* You can add global styles to this file, and also import other style files */

    * {
      font-family: Arial;
    }
    h2 {
      color: #444;
      font-weight: lighter;
    }
    body {
      margin: 2em;
    }
    
    body,
    input[text],
    button {
      color: #888;
      // font-family: Cambria, Georgia;
    }
    button {
      font-size: 14px;
      font-family: Arial;
      background-color: #eee;
      border: none;
      padding: 5px 10px;
      border-radius: 4px;
      cursor: pointer;
      cursor: hand;
      &:hover {
        background-color: #cfd8dc;
      }
      &.delete-button {
        float: right;
        background-color: gray !important;
        background-color: rgb(216, 59, 1) !important;
        color: white;
        padding: 4px;
        position: relative;
        font-size: 12px;
      }
    }
    div {
      margin: .1em;
    }

    .selected {
      background-color: #cfd8dc !important;
      background-color: rgb(0, 120, 215) !important;
      color: white;
    }

    .heroes {
      float: left;
      margin: 0 0 2em 0;
      list-style-type: none;
      padding: 0;
      li {
        cursor: pointer;
        position: relative;
        left: 0;
        background-color: #eee;
        margin: .5em;
        padding: .5em;
        height: 3.0em;
        border-radius: 4px;
        width: 17em;
        &:hover {
          color: #607d8b;
          color: rgb(0, 120, 215);
          background-color: #ddd;
          left: .1em;
        }
        &.selected:hover {
          /*background-color: #BBD8DC !important;*/
          color: white;
        }
      }
      .text {
        position: relative;
        top: -3px;
      }
      .saying {
        margin: 5px 0;
      }
      .name {
        font-weight: bold;
      }
      .badge {
        /* display: inline-block; */
        float: left;
        font-size: small;
        color: white;
        padding: 0.7em 0.7em 0 0.5em;
        background-color: #607d8b;
        background-color: rgb(0, 120, 215);
        background-color:rgb(134, 183, 221);
        line-height: 1em;
        position: relative;
        left: -1px;
        top: -4px;
        height: 3.0em;
        margin-right: .8em;
        border-radius: 4px 0 0 4px;
        width: 1.2em;
      }
    }

    .header-bar {
      background-color: rgb(0, 120, 215);
      height: 4px;
      margin-top: 10px;
      margin-bottom: 10px;
    }

    label {
      display: inline-block;
      width: 4em;
      margin: .5em 0;
      color: #888;
      &.value {
        margin-left: 10px;
        font-size: 14px;
      }
    }

    input {
      height: 2em;
      font-size: 1em;
      padding-left: .4em;
      &::placeholder {
          color: lightgray;
          font-weight: normal;
          font-size: 12px;
          letter-spacing: 3px;
      }
    }

    .editarea {
      float: left;
      input {
        margin: 4px;
        height: 20px;
        color: rgb(0, 120, 215);
      }
      button {
        margin: 8px;
      }
      .editfields {
        margin-left: 12px;
      }
    }
    ``` 
3. Guarde el archivo. 

## <a name="display-the-component"></a>Mostrar el componente

Ahora que tenemos el componente, ¿cómo se consigue que se muestre en la pantalla? Vamos a modificar los componentes predeterminados de **app.component.ts**.

1. En el panel Explorer, abra **/app/app.component.ts**, cambie el título a Heroes y coloque el nombre del componente que hemos creado en **heroes.components.ts** (app-heroes) para hacer referencia a ese nuevo componente. El contenido del archivo ahora debería ser como este: 

    ```ts
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.scss'],
      template: `
      <h1>Heroes</h1>
      <div class="header-bar"></div>
      <app-heroes></app-heroes>
    `
    })
    export class AppComponent {
      title = 'app';
    }

    ```

2. Hay otros componentes de **heroes.components.ts** a los que también se hace referencia, como el componente Hero, por lo que tenemos que crearlo también. En el símbolo del sistema de la CLI de Angular, use el siguiente comando para crear un modelo de Hero y un archivo llamado **hero.ts**, donde g = generar cl = clase y hero = nombre de clase.

    ```bash
    ng g cl hero
    ```

3. En el panel Explorer, abra **src\app\hero.ts**. En **hero.ts**, reemplace el contenido del archivo por el código siguiente, que agrega una clase Hero con un identificador, un nombre y un comentario.

    ```ts
      export class Hero {
      id: number;
      name: string;
      saying: string;
    }
    ```

4. Vuelva a **heroes.components.ts** y observe que en la línea `selectedHero: Hero;` (línea 10), `Hero` tiene una línea roja debajo. 

5. Haga clic en el término `Hero` y Visual Studio muestra un icono de bombilla en el lado izquierdo del bloque de código. 

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part3/light-bulb.png" alt-text="Icono de bombilla en Visual Studio Code":::

6. Haga clic en la bombilla y, después, en **Importar Hero desde "/app/hero".** o **Importar Hero desde "./hero".** (El mensaje cambia según la configuración)

    Aparece una nueva línea de código en la línea 2. Si la línea 2 hace referencia a /app/hero, modifíquela para que haga referencia al archivo hero de la carpeta local (./hero). La línea 2 debería ser similar a esta:

   ```
   import { Hero } from "./hero";
   ``` 

    Eso se encarga del modelo, pero aún debemos crear el servicio.

## <a name="create-the-service"></a>Creación del servicio

1. En el símbolo del sistema de la CLI de Angular, escriba el siguiente comando para crear un servicio Hero en **app.module.ts**, donde g = generar, s = servicio, hero = nombre del servicio, -m = colocar en app.module.

    ```bash
    ng g s hero -m app.module
    ```

2. En Visual Studio Code, vuelva a **heroes.components.ts**. Observe que en la línea `constructor(private heroService: HeroService) {}` (línea 13), `HeroService` tiene una línea roja debajo. Haga clic en `HeroService` y obtendrá el icono de bombilla en el lado izquierdo del bloque de código. Haga clic en la bombilla y, a continuación, haga clic en **Importar HeroService desde "./hero.service".** o **Importar HeroService desde "/app/hero.service".**

    Al hacer clic en la bombilla se inserta una nueva línea de código en la línea 2. Si la línea 2 hace referencia a la carpeta /app/hero.service, modifíquela para que haga referencia al archivo hero de la carpeta local (./hero.service). La línea 2 debería ser similar a esta:
    
    ```javascript
    import { HeroService } from "./hero.service"
    ```

3. En Visual Studio Code, abra **hero.service.ts** y copie en él el código siguiente, reemplazando el contenido del archivo.

    ```ts
    import { Injectable } from '@angular/core';
    import { HttpClient } from '@angular/common/http';
    
    import { Hero } from './hero';
    
    const api = '/api';

    @Injectable()
    export class HeroService {
      constructor(private http: HttpClient) {}

      getHeroes() {
        return this.http.get<Array<Hero>>(`${api}/heroes`)
      }

      deleteHero(hero: Hero) {
        return this.http.delete(`${api}/hero/${hero.id}`);
      }

      addHero(hero: Hero) {
        return this.http.post<Hero>(`${api}/hero/`, hero);
      }

      updateHero(hero: Hero) {
        return this.http.put<Hero>(`${api}/hero/${hero.id}`, hero);
      }
    }
    ```

    Este código usa la versión más reciente de HttpClient que ofrece Angular, que es un módulo que debe proporcionar y, por tanto, lo haremos a continuación.

4. En Visual Studio Code, abra **app.module.ts** e importe HttpClientModule actualizando la sección de importación para incluir HttpClientModule.

    ```ts
    imports: [
      BrowserModule,
      FormsModule,
      HttpClientModule
    ],
    ```

5. En **app.module.ts**, agregue la instrucción de importación de HttpClientModule a la lista de importaciones.

    ```ts
    import { HttpClientModule } from '@angular/common/http';
    ```

6. Guarde todos los archivos de Visual Studio Code.

## <a name="build-the-app"></a>Compilación de la aplicación

1. En el símbolo del sistema, escriba el siguiente comando para compilar la aplicación de Angular. 

    ```bash
    ng b
    ``` 

    Si hay algún problema, la ventana de terminal muestra información sobre los archivos que se deben corregir. Cuando finaliza la compilación, los nuevos archivos pasan a la carpeta **dist**. Puede revisar los nuevos archivos en la carpeta **dist** si lo desea.

    Ahora vamos a ejecutar la aplicación.

2. En Visual Studio Code, haga clic en el botón **Depurar** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part2/debug-button.png"::: de la izquierda y en **Iniciar depuración** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part3/start-debugging-button.png":::.

3. Ahora, abra un explorador de Internet y vaya a **localhost:3000** para ver la aplicación que se ejecuta localmente.

     :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part3/azure-cosmos-db-mongodb-mean-app.png" alt-text="Aplicación Hero que se ejecuta de forma local":::

## <a name="next-steps"></a>Pasos siguientes

En esta parte del tutorial, ha hecho lo siguiente:

> [!div class="checklist"]
> * Ha creado la interfaz de usuario de Angular
> * Ha probado la aplicación de forma local

Puede continuar con la siguiente parte del tutorial para crear una cuenta de Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Creación de una cuenta de Azure Cosmos DB mediante la CLI de Azure](tutorial-develop-mongodb-nodejs-part4.md)
