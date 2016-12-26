# ng2-tips

## ng2 services, singletons or not?

- Provide them on bootstrapping if you only want one instance across the whole app
- Use component providers array to have one new instance of that service, scope-limited.

## ng-cli

MUCH FAST, TOO EASY, WOW

```/app/components/: $ ng g component my-component // Creates component ```

```/app/components/my-component/: $ ng g component my-nested-component ```

Amongst others.

## Unit testing
### Can't bind to 'routerLink' since it isn't a known property of 'WHATEVER'

Gently import RouterTestingModule:

```import { RouterTestingModule } from '@angular/router/testing';``` 

And include it on TestBed:

```
TestBed.configureTestingModule({
            imports: [
                MyModule,
                RouterTestingModule.withRoutes([]),
            ],
        }).compileComponents();

```
        
### Ignored service mock, uh?

You may have probably included the service you want to fake inside
```[...] providers: [] [...]```

If so, Angular says that providers annotated on components are the ones, the highest level, jetset, nothing to f*ck with. You'll probably want to refactor this, in case you don't need new instances (see "ng2 services, singletons or not?") OOOOOOOOOR:

```
TestBed.overrideComponent(YourComponent, {
    set: {
        providers: [{ provide: YourService, useClass: YourServiceStub }]
    }
})
```

NAILED.
