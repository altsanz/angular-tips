# ng2-tips

## ng2 services, singletons or not?

- Provide them on bootstrapping if you only want one instance across the whole app
- Use component providers array to have one new instance of that service, scope-limited.

## ng-cli

MUCH FAST, TOO EASY, WOW

```/app/components/: $ ng g component my-component // Creates component ```

```/app/components/my-component/: $ ng g component my-nested-component ```

Amongst others.

## Detecting changes on array or objects
So, you have something like.

```
export class BufferComponent implements OnChange {
    @Input() buffer: Array<string>;
    
    ngOnChange() {
        makeSparkles();
    }
}
```

Oh, boy. Good luck detecting changes on that array. BREAKING NEWS: Angular2 per default only detects changes when reference changes... and pushing a new item on your beloved array won't do that.

Let's detect that. So, if OnChange phase doesn't detect changes... who else would do it? Let's review life cicle:

![Angular 2 Life Cicle](https://angular.io/resources/images/devguide/lifecycle-hooks/hooks-in-sequence.png)

Yep, we have that DoCheck triggered during every change detection run, immediately after ngOnChanges and ngOnInit. And docs say it stands for checking WHAT ANGULAR CAN'T OR WON'T DETECT ON ITS OWN. Looks like we have a wingman.

```
export class BufferComponent implements DoCheck {
    @Input() buffer: Array<string>;
    
    ngDoCheck() {
            doSparkles()
    }
}
```

YAS, you got it. Sparkles everywhere on each life cycle triggered. Wait. When is a life cycle triggered then? ngDoCheck is triggered every time the input properties of a component or a directive are checked. And when they change, despite of not being detected by ngOnChanges, it's checked anyways.

BUUUUUUUUT, if we have many inputs like:

```
export class BufferComponent implements DoCheck {
    @Input() buffer: Array<string>;
    @Input() person: string;
    ngDoCheck() {
            doSparkles()
    }
}
```

If person changes, doSparkles will be called even, and our expected behaviour was to sparkle all the sh*t out of the screen ONLY on buffer change.

One last step.

Add KeyPairDiffer to constructor. Generate an array differ and bound it to the components. Then check difference between this differ and buffer. Talk is cheap, let's see the code:

```
export class BufferComponent implements DoCheck {
    @Input() buffer: Array<string>;
    
    differ: any;
    
    constructor(private elementRef: ElementRef, private differs: KeyValueDiffers) {
      this.differ = differs.find([]).create(null);
    }
    
    ngDoCheck() {
      var changes = this.differ.diff(this.buffer);

      if(changes) {
        doSparkles();
      }
    }
}
``` 

HELL YEA. Perfect. Hats off.

## Unit testing
### “Error: No provider for router” while writing Karma-Jasmine unit test cases

On TestBed include the following import:

```
import {RouterTestingModule} from '@angular/router/testing';
[...]
TestBed.configureTestingModule({
  imports: [RouterTestingModule], <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
  declarations: [ ChooseTypePageComponent ],
  schemas: [CUSTOM_ELEMENTS_SCHEMA, NO_ERRORS_SCHEMA]
})
```


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

### Route params mocking rocking

So, got some parameters through URL? Gettin' them on ngOnInit? Let's mock 'em all! 
As for this case, I expect you to have smth like:
```
    this.sub = this.activatedRoute.params.subscribe(params => {
      this.challengueId = +params['challengueId']
      this.challengue = this.challenguesService.getChallengueById(this.challengueId);     
    });
    
```

Yep, first override ActivatedRoute provider on the component annotations:
```
        providers: [
            { provide: ActivatedRoute, useValue: { 'params': Observable.from([{ 'challengueId': 1 }]) }}
        ]
 ```
 
 This is breaking your code. Of course, where is that Observable coming from? Import it:
 
 ```import { Observable } from 'rxjs/Rx';```
 
 We are subscribing to variable ([...]params.subscribe(params=>[...]) so one can not just return a variable, we make 'em observable. That's it.
 
 
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


### Script error on Jasmine??

Check if you imported CUSTOM_ELEMENTS_SCHEMA from anything else than @angular/core.
