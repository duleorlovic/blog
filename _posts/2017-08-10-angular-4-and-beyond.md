---
layout: post
---


# Install

~~~
npm install -g @angular/cli
ng help | more
~~~

~~~
ng new myapp --style=scss
cd myapp
ng serve
~~~

~~~
yarn add sass @types/lodash ngrx
yarn add bootstrap@4 --dev
~~~

*Presentation components* are used only for how things look. and *container
components* how thinks works they provide data and behavior to other components
(never have any styles)

~~~
ng generate component card --inline-template --inline-style --dry-run
~~~

# HTML templates

* `{ { title }}` interpolation of title (which is component property),
* `<script>` is ignored and statements can't refer to `window` or `console.log`
* in interpolation (both expression and statements) you can not use `new`,`++`
or `+=`. This is not allowed since it has side effects.
* difference between HTML attributes and DOM property is that attribute only
initialize DOM properties, so attributes can't be changed (initial value of
input), but properties (current value) can. So in Angular we do only with
properties.
Binding targets can be: element, component, directive. There are 5 type of
binding:

* **[property]=expression** (`bind-property`) set img src `<img
[src]="heroImageUrl">`, disable `<button
[disabled]="isUnchanged">Cancel</button>`, passing value `<hero-detail
[hero]="currentHero"></hero-detail>`.
  * you should not use property binding to call a method that has side effect
  * angular first check if the name is property of known directive `<div
  [ndClass]="classes"></div>` or reports "unnknown directive" error
  * Context of the expression is instance of component, or some property of
  template's context like *template input variable* `<div *ngFor="let hero of
  heroes">{{hero.name}}</div>` and *template reference variable* `<input
  #heroInput>{{heroInput.value}}`. So there are three namespaces: template
  variable, directive's *context* object and compoments's member name.
  * tips: Always use simple property name or method call (with eventual `!`),
  use component definition to define business logic.

* **(event)=statement** (`on-event`) `<button (click)="onSave()")</button>"`
  * template statement usually have side effect (while template expression not)
  * In this event binding like `(click)="onSelect(hero)"` statement context is
  typicaly *component instance*, but also can be templates own context
  (`$event`, template input variable, template reference variable)
  * `$event` store info about event (if it is navite DOM than you can use
  `$event.target.value`, if it belongs to directive than it could be anything)
  * statements allow assignment `=` and chaining `;` `,`, but dont allow
  template epression operators (`|` `?.` `!`)

* **event and property**  two way bindings `<input [(ngModel)]="name">`
* `<input #heroName /> <button (click)="add(heroName.value); heroName.value=''">Add</button>` creates local variable `heroName` that provides access to `input` element instance in all data and event binding expressions in current template.

* **attribute** (exception) `<button [attr.aria-label]="help">help</button>`
* **class.name** property `<div [class.special]="isSpecial"></div>` class
binding is used to add and remove *single* class.
* **style.name** property `<button [style.color]="isSpecial ? 'red' : 'green'">`


* `*ngFor='let hero of heroes'` iterate over items
[NgForOf](https://angular.io/api/common/NgForOf) in angular 4
* `*ngif='heroes.lenght > 3'` add or remove from the DOM

* `[(ngModel)]="currentHero.name"` (banana in a box) is part of `FormsModule`
from `@angular/forms` it is the same as
`(input)="currentHero.name=$event.target.value"` and
`[value]="currentHero.name"`.


Redisplay occurs when some async event related to view occurs: keystroke, timer
or response to HTTP request.

# Component

For new components, you need to declare in `@NgModule({declarations:
HeroDetailComponent]})`, accept input `@Input() hero: Hero`.

[lifecycle hooks](https://angular.io/guide/lifecycle-hooks)

~~~
import { OnInit } from '@angular/core';

export class AppComponent implements OnInit {
  ngOnInit(): void {
  }
}
~~~

# Service

Add to constructor `constructor(private heroService: HeroService) { }` and to
`@Component({providers: [HeroService]})`. You can call service inside component
with `this.heroService.getHeroes()`.
If you move service to `@ngModule({providers: [HeroService]})` than every
component has access to it. Singleton `HeroService` instance is created.

# Routing

<https://angular.io/guide/router>

* `<router-outlet></router-outlet>` is where to show a component
* `<a routerLink="/heroes">Heroes</a>` is used

To get params use
~~~
import { ActivatedRoute, ParamMap } from '@angular/router';
import { Location }                 from '@angular/common';
import 'rxjs/add/operator/switchMap';
export class HeroDetailComponent implements OnInit {
  constructor(
    private heroService: HeroService,
    private route: ActivatedRoute,
    private location: Location
  ) {
  }
  ngOnInit(): void {
    this.route.paramMap
      .switchMap((params: ParamMap) => this.heroService.getHero(+params.get('id')))
      .subscribe(hero => this.hero = hero);
  }
  goBack(): void {
    this.location.back();
  }
}
~~~

To navigate using js

~~~
  constructor(
    private router: Router
  ) { }
  viewDetails(hero: Hero): void {
    this.router.navigate(['/detail', hero.id]);
  }
~~~

# HttpModule

To use mock web api install:

~~~
npm install angular-in-memory-web-api --save
~~~

# Dependency injection

<https://angular.io/guide/dependency-injection>
Class is available when you register a class with `@Injectable()`.
You can register a provider within `@NgModule` or in component's `providers: []`
property. If it is registered on component, than it is available to that
component and children component (all components that are included in template
markup, or navigating with router).
The component must ask for service in its constructor (type anotation will
determine what to inject).

Always use `@Injectable()` for services (event they do not have any
dependency). `@Component` is subtype of `@Injectable()` so you do need that for
components.

`providers: [Logger]` is the same as `providers: [{provide: Logger, useClass:
Logger}]`. First value is a key, a second is a class (could be some other
`BetterLogger`).

# RxJS

A **Subject** is a producer of an observable event stream.

