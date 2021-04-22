# Angular Applikáció

Leírás: Gyakorlati megoldás

- Tartalomjegyzék

# Feladatkiírás

- **JS-task/task01-algorithm**

    ### **4. Feladat**

    - Hozzon létre egy Angular alkalmazást tetszőleges helyen (nem kötött a név).
    - Legyen benne három réteg: class -> service -> component.
    - Az élő json szerverről jelenítse meg a termékeket.
    - A szerver elérhetősége: `https://nettuts.hu/jms/<githubname>/products` (a helyére kerüljön az ön github user -neve)
    - Értelmezze a szerveren elérhető termékeket és azok alapján hozza létre a szükséges osztályokat.
    - Készítsen egy teljes értékű CRUD -ot (getAll, get, create, update, delete) egy megfelelő service -ben.
    - Az adatokat táblázatos formában jelenítse meg a főoldalon.
    - Legyen egy űrlap, ahol fel lehet venni új entitást (most Product).
    - Minden sorhoz tartozzon módosítás és törlés gomb.
    - A módosítás egy új űrlapra navigáljon, ahol módosíthatóak a kiválasztott entitás adatai.
    - A törlés törölje az adott sort majd frissítse a listát.
    - Alkalmazzon szabványos Angular Routing -ot az oldalak váltására.

    ### **5. Feladat**

    - A kész Angular alkalmazást publikálja a saját Github oldalán. `https://<githubname>.github.io`

# Új Angular projekt létrehozása

`ng new str-ang-certexam-myexercise` STRICT, ROUTING, SCSS opciókkal

# BootStrap FontAwesome bekötése

`npm i bootstrap font-awesome`

`angular.json` file-ban beilleszteni:

```json
"styles": [
           "node_modules/bootstrap/dist/css/bootstrap.min.css",
           "node_modules/font-awesome/css/font-awesome.min.css",
           "src/styles.scss"
          ],
"scripts": [
						"node_modules/bootstrap/dist/js/bootstrap.min.js"
					],
```

# Back-end

## Lokális adatbázis

A `server` könyvtárat és az adatokat tartalmazó `data.json` file-t a projekt rootban helyezem el, amit an Angular nem figyel, és a json-server segítségével látom el. A json-server indítására beállítok egy scriptet a `package.json` file-ban:

```json
"scripts": {
    "json": "json-server --watch ./server/data.json",
		"start": "ng serve -o",
  },
```

### Példa JSON file

Az aktuális példában a file termékeket tartalmaz:

```jsx
{
    "products": [
        {
            "id": 1,
            "name": "Dates",
            "price": 82,
            "category": "Electronic"
        },
        {
            "id": 2,
            "name": "Soup - Campbells Mac N Cheese",
            "price": 95,
            "category": "Beauty"
        }
		]
}
```

[db.json](Angular%20Applika%CC%81cio%CC%81%20099579758c5b442984b225b0d0adbf4d/db.json)

### JSON server indítása

vagy scriptből (`json`) vagy parancssorból:

`json-server --watch .\server\data.json`

## Távoli adatbázis

A szerver elérhetősége: `https://nettuts.hu/jms/<githubname>/products` (a <githubname> helyére kerüljön a GitHub username)

# Osztályok készítése

Létrehozom például a `product` osztályt, melyben az adatbázisnak megfelelő típusokkal látjuk el az egyes tulajdonságokat.

`ng g class model/product`

```tsx
export class Product {
    id: number = 0;
    name: string = "";
    price: number = 0;
    category: string = "";
}
```

# A szolgáltatások

## A `service` létrehozása

Új `product` szolgáltatást hozunk létre, mely a `ProductService` nevet kapja. A szolgáltatás neve után a CLI-ben nem kell megadni a `Service` kifejezést, mert azt az Angular teszi hozzá.

`ng g service service/product`

## A `service` megjelenítése az `AppModule`-ban

Ahhoz, hogy a service elérhető legyen, a szolgáltatást a modulrendszer részévé kell tenni. Ennek érdekében a `service`-t importáljuk, majd beírjuk az `AppModule` `@NgModule` dekorátor objektum `providers` tömbjébe:

```tsx
import { ProductService } from './service/product.service';

@NgModule({
//...
  providers: [
		ProductService
	],
//...
})
```

## A `service` kódja

A szolgáltatásban `BehaviorSubject` segítségével fogjuk az adatokat elérhetővé tenni a komponensek számára.

### Előkészítés

- http lekérésekhez az `AppModule`-ban a `HttpClientModule`-t  az `@NgModule` dekorátor objektum `imports` tömbjében tüntetjük fel.

    ```tsx
    import { HttpClientModule } from '@angular/common/http';

    @NgModule({
    //...
      imports: [
        HttpClientModule
      ],
    //...
    })
    ```

### Az adatok fogadása

- A service-ben megadjuk az `apiUrl` változóban az adatokat szolgáltató server elérhetőségét:
`apiUrl: string = "http://localhost:3000/products"`
- Az osztályban létrehozunk egy `list$` nevű `BehaviorSubject<Product[]>` objektumot, aminek rögtön át is adunk egy üres tömböt:
`list$: BehaviorSubject<Product[]> = new BehaviorSubject<Product[]>([]);`

```tsx
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';
import { Product } from '../model/product';

@Injectable({
  providedIn: 'root'
})
export class ProductService {

  apiUrl: string = 'http://localhost:3000/products';
  list$: BehaviorSubject<Product[]> = new BehaviorSubject<Product[]>([]);

  constructor(
    private http: HttpClient,
    ) { }
}
```

## CRUD ([Konzultáció](https://www.notion.so/2021-02-05-Konzult-ci-HttpModule-Crud-1e3aa534f6e642e8badbaf9f67ee9682))

## A `create()` metódus (CRUD)

A metódus a `http.post` segítségével új terméket ad az adatbázishoz.

```tsx
create(product: Product): Observable<Product>  {
    return this.http.post<Product>(this.apiUrl, product);
  }
```

### `getAll()`

Elkészítjük a `getAll()` metódust, mely lekéri a szerverről http-n keresztül az összes adatot.

```jsx
  getAll(): void {
    this.http.get<Product[]>(this.apiUrl).subscribe(
      products => this.list$.next(products)
    )
  };
```

### A `get()` metódus (CRUD)

Ez a metódus nem az összeset, hanem csak egy objektumot kér le.

A `get`-nek átadok egy `user`-t, a `HttpClient.get()` segítségével lekérem az átadott `product.id` alapján megadott URL-en található `product`-ot és `observable`-be csomagolva visszaadjuk.

```tsx
get(product: Product): Observable<Product> {
    return this.http.get<Product>(`${this.apiUrl}/${product.id}`);
}
```

A `getOne()` metódus közvetlenül `id`-t fogad, ami lehet `number` vagy `string`. Ez akkor használható, ha nem `product`-ot adunk át , hanem az `ActivatedRoute` `params` tulajdonságából kinyert `id` értéket.

```tsx
getOne(id: string | number): Observable<Product> {
    return this.http.get<Product>(`${this.apiUrl}/${id}`);
}
```

### A `remove()` metódus (CRUD)

A metódus nem ad vissza semmit, az átadott `id` alapján törlünk a `HttpClient.delete()` metódus segítségével. Majd feliratkozunk a válaszra, és ha megérkezett, akkor frissítjük a `list$` `BehaviorSubject`-et

```tsx
remove(product: Product): void {
    this.http.delete<Product>(`${this.apiUrl}/${product.id}`).subscribe(
      () => this.getAll()
    );
  }
```

### Az `update()` metódus (CRUD)

Ez a metódus `Observable`-t ad vissza, és a `HttpClient.patch()` segítségével frissítjük az adatokat. 

```tsx
update(product: Product): Observable<Product>  {
    return this.http.patch<Product>(`${this.apiUrl}/${product.id}`, product);
  }
```

# Komponensek létrehozása

## AppComponent

Az alkalmazás létrehozásakor automatikusan elkészült. Feladata, hogy megjelenítsen egy `NavComponent` segítségével egy `navbar`-t, alatta pedig menüponttól függően egy `HomeComponent` vagy `ProductsComponent` elemet.

## NavComponent

Alapértelmezett Bootstrap stílusú navbar jelenik meg benne, két menüponttal: Home, Products

Létrehozása: `ng g c component/nav`

- A teljes `NavComponent` kód

    ```html
    <nav class="navbar navbar-expand-sm navbar-light bg-light">
        <a class="navbar-brand" [routerLink]="['/home']">Title</a>
        <button class="navbar-toggler d-lg-none" type="button" data-toggle="collapse" data-target="#collapsibleNavId" aria-controls="collapsibleNavId"
            aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="collapsibleNavId">
            <ul class="navbar-nav mr-auto mt-2 mt-lg-0">
                <li class="nav-item" routerLinkActive="active" [routerLinkActiveOptions]="{exact: true}">
                    <a class="nav-link" [routerLink]="['/home']">Home<span class="sr-only">(current)</span></a>
                </li>
                <li class="nav-item" routerLinkActive="active" [routerLinkActiveOptions]="{exact: true}">
                    <a class="nav-link" [routerLink]="['/products']">Products</a>
                </li>
            </ul>
        </div>
    </nav>
    ```

## HomeComponent

Egy gombot (jumbotron) tartalmaz, melyet megnyomva a `ProductComponent` jelenik meg.

Létrehozása: `ng g c component/home`

## ProductsComponent

Az adatbázisból lekért listát jeleníti meg.

Létrehozása: `ng g c component/products`

## ProductDetailComponent

Az adatbázisból lekért egyetlen elem részleteit jeleníti meg. Ezt külön nem jelenítjük meg a `nav`-on

Létrehozása: `ng g c component/product-detail`

# Routing - a komponensek összekötése

## URL megadása az `AppRouting` modulban

Az `AppRouting` modulban a `routes` tömbben  megadjuk a kívánt útvonalat és a komponens nevét, ahova mutat az útvonal. A `**` jelöli a fallback URL-t

Ha a routing során egy URL-ben nem csupán fix értéket, hanem egy változót is meg akarok adni, azt a `:` segítségével tehetem:

```tsx
const routes: Routes = [
  {	path: 'home',	component:  HomeComponent },
  {	path: 'products',	component:  ProductComponent },
	{	path: 'products/:id',	component:  ProductDetailComponent },
  {	path: '**',	component:  HomeComponent }  
];
```

## A `<router-outlet>` megjelenítése az `AppComponent`-ben

```html
<app-nav></app-nav>
<div class="container mt-3">
  <div class="row">
    <div class="col-12">
      <router-outlet></router-outlet>
    </div>
  </div>
</div>
```

## Az útvonalak megadása a `NavComponent template`-ben

pl. a `Products` hivatkozásra kattintva a `ProductsComponent` jelenjen meg:

`<a class="nav-link" [routerLink]="['/products']">Products</a>`

Ezt a módszert kel alkalmazni a `Title`, `Home` linkeken.

Ahhoz, hogy a kiválasztott oldal legyen mindig aktív a navigáció sorban, az alábbi megoldást kell alkalmazni ([tananyag](https://www.notion.so/2021-02-09-Tananyag-Angular-Routing-a5548cce59654d5298a614091e76a1c8)):

`<li class="nav-item" routerLinkActive="active" [routerLinkActiveOptions]="{exact: true}">`

## Az útvonal megadás a `HomeComponent` `Jumbotron` gombján

`<a class="btn btn-primary btn-lg" [routerLink]="['/products']" role="button">Link to Products</a>`

# Adatbázis megjelenítés táblázatban

## ProductsComponent - a TypeScript kód

Mivel az adatok a `ProductService`-ből érkeznek, a konstruktor függvényben megadjuk a szolgáltatást:

```tsx
constructor(
    private productService: ProductService,
  ) { }
```

Ezután létrehozunk egy saját `productList$` nevű objektumot, és a példányosítás során feliratkozunk a `list$` nevű `BehaviorSubject` objektumra:

`productList$: BehaviorSubject<Product[]> = this.productService.list$;`

Ebben az állapotban még csak átvettük a `list$` tartalmát, de a példányosításakor csak egy üres tömbbel [történt](https://www.notion.so/K2-JS-gyakorl-s-Angular-Routing-53fb0e189eeb4a0d84aff1232e6c93d9), és ahhoz, hogy belekerüljenek az adatok, meg kell hívni a `getAll()` metódust, mert az tölti fel adattal és értesíti a feliratkozókat. Azért, hogy rögtön a komponens létrehozásakor ez megtörténjen, az `ngOnInit()` metódusban hívjuk meg.

```tsx
ngOnInit(): void {
  this.productService.getAll();
}
```

- A teljes ProductsComponent

    ```tsx
    import { Component, OnInit } from '@angular/core';
    import { BehaviorSubject } from 'rxjs';
    import { Product } from 'src/app/model/product';
    import { ProductService } from 'src/app/service/product.service';

    @Component({
      selector: 'app-products',
      templateUrl: './products.component.html',
      styleUrls: ['./products.component.scss']
    })
    export class ProductsComponent implements OnInit {

      productList$: BehaviorSubject<Product[]> = this.productService.list$;
      
      constructor(
        private productService: ProductService,
      ) { }

      ngOnInit(): void {
        this.productService.getAll();
      }
    }
    ```

## ProductsComponent - a Template

A fejléc elemeit fixen jelenítjük meg, a sorokat `*ngFor` illetve `| async` segítségével.

A `thead` utolsó eleme üres, az ennek megfelelő oszlop tartalmazza a gombokat.

Az egyes sorok végén egy gombcsoport található, első gomb a szerkesztést, a második a törlést valósítja meg.

```html
<div class="row">
    <div class="col-12">
        <table class="table table-striped table-hover">
            <thead>
                <tr>
                    <th>id</th>
                    <th>Name</th>
                    <th>Price</th>
                    <th>Category</th>
										<th></th>
                </tr>
            </thead>
            <tbody>
                <tr *ngFor="let product of productList$ | async">
                    <td>{{ product.id }}</td>
                    <td>{{ product.name }}</td>
                    <td>{{ product.price }}</td>
                    <td>{{ product.category }}</td>
										<td>
				                <div class="btn-group" role="group">
			                    <button type="button" class="btn btn-primary">
		                        <i class="fa fa-pencil"></i>
			                    </button>
			                    <button type="button" class="btn btn-danger">
		                        <i class="fa fa-trash"></i>
			                    </button>
			                </div>
				            </td>
                </tr>
            </tbody>
        </table>
    </div>
</div>
```

Ebben az állapotban a gombokra kattintáshoz nincs funkció rendelve.

# Egy termék törlése

A táblázat sorai végén található törlés gombhoz eseménykezelést rendelünk: a gombra kattintáskor egy metódust hívunk meg: `(click)="deleteProduct(product)"` metódust hívunk meg. A metódusban a `ProducService.remove()` segítségével töröljük az adott sorhoz tartozó elemet.

Az osztályban a metódus kódja:

```tsx
deleteProduct(product: Product): void {
    this.pService.remove(product);
}
```

A template-ben az eseménykezelés hozzárendelése a gombhoz:

```html
<button (click)="deleteProduct(product)" type="button" class="btn btn-danger">
       <i class="fa fa-trash"></i>
</button>
```

# Egy termék adatainak megjelenítése

Egy termék adatait  a `ProductDetail` komponensben jelenítjük meg, amely `form` segítségével teszi lehetővé a termék egyes tulajdonságához tartozó értékek szerkesztését.

Ahhoz, hogy a szerkesztés gombra kattintáskor a  a `ProductDetail` komponens jelenjen meg, egy eseménykezelést rendelünk a gombhoz: `(click)="jumpToProduct(product)"`

```tsx
<button (click)="jumpToProduct(product)" type="button" class="btn btn-primary">
		<i class="fa fa-pencil"></i>
</button>
```

Ennek megfelelően elkészítjük a `jumpToProduct` metódust a `ProductsComponent` osztályban, mely a beépített `Router` service segítségével a `navigateByUrl` metódussal a megadott URL-re lép. Szükséges hozzá a `Router` szolgáltatás importálása.

```tsx
import { Router } from '@angular/router';
constructor(
    private router: Router,
  ) { }

jumpToProduct(product: Product): void {
    this.router.navigateByUrl(`/products/${product.id}`);
  }
```

## A ProductDetail komponens felépítése

Miután az adott termék szerkesztőgombjára kattintunk, az URL alapján lekérjük az adott `id`-vel rendelkező terméket, amit a `ProducComponent` template-en jelenítünk meg. Ez az oldal egy `form`-ot tartalmaz, egy `update` gombbal, mely segítségével a megváltoztatott termékadatokkal frissíthetjük az adatbázist.

### TypeScript kód

Az `ActivatedRoute` a `router-outlet`-be töltött route információkat adja meg. Az `params` metódusa egy `observer`-t ad vissza, melyre feliratkozhatunk. Ezáltal a `ProductDetail` komponensben feliratkozunk arra az eseményre, amikor változik az URL, és változnak az URL paraméterek.

Elsőként az `ActivatedRoute` szolgáltatást kell importálni, majd a `constructor`-ban injektálni a szolgáltatást a `ProductService` mellett. 

A visszaadott `params.id` tartalmazza a számot, mely segítségével lekérhető az adott `Product`.

A `params` értékének kezelése a komponens létrejöttekor az `ngOnInit` metódusban történik. Innentől kezdve a komponensben elérhető a kiválasztott adat a `product` objektumban.

```tsx
ngOnInit(): void {

  this.ar.params.subscribe(params => {
    this.productService.getOne(params.id).forEach(product => {
      this.product = product;
    });
  });

}
```

A template elkészítése után a mentés gombra kattintáskor az `onFormSubmit` metódust hívjuk meg, amely a `ProductService` `update` metódusban a `http.patch` segítségével frissíti az adatokat és visszanavigál a list oldalra. A navigációhoz a Router szolgáltatást injektálni kell a komponensben.

```tsx
onFormSubmit(form: NgForm): void {
    this.pService.update(this.product).subscribe(
      () => this.router.navigate(['products'])
    );
  }
```

- Teljes `product-detail.ts` kód

    ```tsx
    import { Component, OnInit } from '@angular/core';
    import { NgForm } from '@angular/forms';
    import { ActivatedRoute, Router } from '@angular/router';
    import { Product } from 'src/app/model/product';
    import { ProductService } from 'src/app/service/product.service';

    @Component({
      selector: 'app-product-detail',
      templateUrl: './product-detail.component.html',
      styleUrls: ['./product-detail.component.scss']
    })
    export class ProductDetailComponent implements OnInit {

      product: Product = new Product;

      constructor(
        private aRoute: ActivatedRoute,
        private router: Router,
        private productService: ProductService,
      ) { }

      ngOnInit(): void {

        this.aRoute.params.subscribe(params => {
          this.productService.getOne(params.id).forEach(product => {
            this.product = product;
          });
        });

      }

      onFormSubmit(form: NgForm): void {
        this.productService.update(this.product).subscribe(
          () => this.router.navigate(['products'])
        );
      } 

    }
    ```

### Template - Előkészítés

Előkészítés: a `ProductDetail` egy űrlapot tartalmaz, melynek input mezőin és a teljes űrlapon validációt végzünk. Az űrlap kezeléshez első lépében a az `AppModule`-ban az imports tömbben fel kell tüntetni a `FormsModule`-t.

```tsx
import { FormsModule } from '@angular/forms';

@NgModule({
//...
  providers: [
		FormsModule
	],
//...
})
```

Az egyes input mezők validálására két lehetőségünk van. Mindkét esetben Sablon Referencia Változót használunk.

### Template - Validálás 1

Első esetben a SRV-t az egyes input mezőknél egyedileg hozzuk létre. Egy általános, validált input mező így épül fel:

```html
<div class="form-group">
	<label for="name">Name</label>
	<input 
			[(ngModel)]="product.name"
			type="text" 
			name="name" 
			id="name" 
			class="form-control" 
			required 
			#productName="ngModel">
	<small 
			[hidden]="productName.valid" 
			id="helpName" 
			class="text-muted">
		Kötelező kitölteni
	</small>
</div>
```

Az `input` esetén megadjuk a `#productName`  SRV-t, aminek a `.valid` értékétől függően jelenítjük meg a `<small>` elemet, ami az invalid állapot hibaüzenetét jeleníti meg.

### Template - Validálás 2

Második esetben az egész `form`-ra mutató SRV-t használjuk fel. 

Az angular az összes űrlaphoz (a `<form>` html elemnél) készít egy `ngForm` objektumot , amiben megtaláljuk az űrlap adatait. Ennek használatához létrehozunk egy `#productForm` nevű SRV-t. Emellett felveszünk egy `(ngSubmit)="onFormSubmit"` kötést, ami küldés esetén meghívja a hozzátartozó metódust.

`<form #productForm="ngForm" ngSubmit="onFormSubmit(productForm)">`

Ebben az esetben nem az egyes input mezőkhöz rendelt SRV-k valid értéke alapján jelenítjük meg vagy rejtjük el a hibaüzenetet, hanem a `form`-hoz tartozó SRV-n keresztül kérjük le az egyes inputok valid állapotát:

```html
<form #productForm="ngForm" (ngSubmit)="onFormSubmit(productForm)">
	<div class="form-group">
	  <label for="category">Category</label>
	  <input 
			[(ngModel)]="product.category" 
			type="text" 
			name="category" 
			id="category" 
			class="form-control" 
			required>
    <small 
			[hidden]="productForm.controls.category?.valid" 
			id="helpCategory" 
			class="text-muted">
		 Kötelező kitölteni
		</small>
	</div>
</form>
```

### Template - Mentés gomb és a `http.update` metódus hívása

A form utolsó eleme egy gomb, melyen eseménykezelő segítségével küljdük el a form tartalmát az adatbázis frissítése céljából. A gomb inaktív, ha a form nem valid mezőket tartalmaz:

```html
<button 
			[disabled]="productForm.invalid" 
			type="submit" 
			class="btn btn-block btn-primary">
	<i class="fa fa-save"></i>
</button>
```

A mentés gombra kattintáskor (amennyiben a `form` valid) az `onFormSubmit(productForm)` metódust hívjuk meg.

- Teljes `product-detail.html` kód

    ```html
    <table class="table">
        <thead>
            <tr>
                <th>#</th>
                <th>Name</th>
                <th>Price</th>
                <th>Category</th>
                <th></th>
            </tr>
        </thead>
        <tbody>
            <tr *ngFor="let product of productList$ | async">
                <td>{{ product.id }}</td>
                <td>{{ product.name }}</td>
                <td>{{ product.price }}</td>
                <td>{{ product.category }}</td>
                <td>
                    <div class="btn-group" role="group">
                        <button (click)="jumpToProduct(product)" type="button" class="btn btn-primary">
                            <i class="fa fa-pencil"></i>
                        </button>
                        <button (click)="deleteProduct(product)" type="button" class="btn btn-danger">
                            <i class="fa fa-trash"></i>
                        </button>
                    </div>
                </td>
            </tr>
        </tbody>
    </table>
    ```

# Új termék hozzáadása az adatbázishoz

Az új termék hozzáadása a `Products` komponensen, a list felett elhelyezett `form` `input` mezői segítségével történik. A `form` egy gombot is tartalmaz, mely csak a `form` valid állapota esetén kattintható.