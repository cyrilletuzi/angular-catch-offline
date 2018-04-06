# Angular PWA Offline

RxJS operator and other utils catching offline errors in Angular apps and PWA.

## Angular onsite training

The author of this library organizes Angular courses (based in Paris, France, but open to travel). You can find [my bio here](https://www.cyrilletuzi.com/en/web/) (in English) and [course details here](https://formationjavascript.com/formation-angular/) (in French).

## Why this lib?

You know the [Angular `async` pipe](https://angular.io/guide/pipes#the-impure-asyncpipe), right? Amazing tool, but there is one big issue with it:
by letting Angular automatically subscribing and unsubscribing your `Observable` or `Promise`, you can't handle the error callback.

Problem is: in a web app, especially in a Progressive Web App (PWA),
a lot of your `Observable` or `Promise` are about HTTP requests,
and they will inevitably fail when the user is offline (or the server is inaccessible).

So if you want to get the benefice of the `async` pipe without breaking your app, you need to catch offline errors on every page. Painful.

So here it is: a RxJS operator catching offline errors for you, so you can use the `async` pipe everywhere!

There are also other tools for offline management, like online status helpers and guards.

## Getting started

Angular 5 and RxJS >= 5.5 are required.

**Install with npm** or another package manager:

```bash
npm install @ngx-pwa/offline
```

Then **import the `OfflineModule` module in your app root module** (just once, do *NOT* re-import it in your submodules).

```typescript
import { OfflineModule } from '@ngx-pwa/offline';

@NgModule({
  imports: [
    BrowserModule,
    OfflineModule,
    ...
  ]
  ...
})
export class AppModule {}
```

Then you just have to **inject the `Offline` service *at least once***, for example in `AppComponent`:

```typescript
import { Offline } from '@ngx-pwa/offline';

@Component()
export class AppComponent {

  constructor(protected offline: Offline) {}

}
```

Note : you may not use the service itself and just the RxJS operator, but an injection is required in all cases to setup the service.
If you have an idea to avoid this step, feel free to contribute in [the related issue](https://github.com/cyrilletuzi/@ngx-pwa/offline/issues/1).

## Catching offline errors

### RxJS operator

Just **use the `catchOffline` RxJS operator**:

```typescript
import { catchOffline } from '@ngx-pwa/offline';

@Component({
  selector: 'some-page',
  template: `
    <presentation-component [data]="data$ | async"></presentation-component>
  `
})
export class SomePageComponent implements OnInit {

  data$: Observable<Data>;

  constructor(protected someService: SomeService) {}

  ngOnInit() {

    this.data$ = this.someService.getData().pipe(catchOffline());

  }

}
```

### Redirecting

By default, `catchOffline` will redirect to:

- `/offline` if the user is offline (no Internet connection),
- `/unavailable` if the service is inaccessible (5xx HTTP errors).

Note: creating the corresponding routes and components in your app is up to you,
as the lib can't decide the content and design of these pages for you.

If you want to change the redirection URLs, just configure the module:

```typescript
@NgModule({
  imports: [
    OfflineModule.forRoot({ routeOffline: '/oops/offline', routeUnavailable: '/oops/unavailable' })
  ]
})
export class AppModule {}
```

Note: you need to provide the full URL, so *the leading `/` is required*.

## Online status

### Static check

To check online status at some point:

```typescript
import { Offline } from '@ngx-pwa/offline';

@Component({
  template: `
    <online-component *ngIf="isOnline"></online-component>
    <offline-component *ngIf="!isOnline"></offline-component>
  `
})
export class SomePageComponent implements OnInit {

  isOnline = this.offline.isOnline;

  constructor(protected offline: Offline) {}

}
```

### Observable check

To observe when online status changes:

```typescript
import { Offline } from '@ngx-pwa/offline';

@Component({
  template: `
    <online-component *ngIf="isOnline$ | async; else isOffline"></online-component>
    <ng-template #isOffline>
      <offline-component></offline-component>
    </ng-template>
  `
})
export class SomePageComponent implements OnInit {

  isOnline$ = this.offline.connectionChanges;

  constructor(protected offline: Offline) {}

}
```

Notes:
- as usual in Angular, do *not* use the `async` pipe twice on the same `Observable`. The example above shows you how to manage those situations in the proper way,
- this `Observable` does *not* auto-complete. Then you should either use the `async` pipe as above for automatic unsubscription, either you should unsubscribe manually (in `ngOnDestroy` method in most cases).

## Guards

Guards catching offline errors are also available, for `CanActivate`, `CanActivateChild` and `CanLoad`. For example:

```typescript
import { Offline } from '@ngx-pwa/offline';

const routes: Routes = [
  { path: 'some-page', component: SomePageComponent, canActivate: [Offline] }
];
```

By default, guards will redirect to the `/offline` page. If you just want to block the navigation, you can configure the module:
```typescript
@NgModule({
  imports: [
    OfflineModule.forRoot({ guardsRedirect: false })
  ]
})
export class AppModule {}
```

## Angular support

This lib major version is aligned to the major version of Angular. Meaning for Angular 5 you need version 5, and so on.

As RxJS >= 5.5 is required, which is the minimum dependency since Angular 5 only, the lib don't support Angular 4 and below.

This module supports [AoT pre-compiling](https://angular.io/guide/aot-compiler).

This module supports [Universal server-side rendering](https://github.com/angular/universal).

## Changelog

[Changelog available here](https://github.com/cyrilletuzi/@ngx-pwa/offline/blob/master/CHANGELOG.md).

## License

MIT
