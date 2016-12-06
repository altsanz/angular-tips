# ng2-tips

## ng2 services, singletons or not?

- Provide them on bootstrapping if you only want one instance across the whole app
- Use component providers array to have one new instance of that service, scope-limited.

## ng-cli

MUCH FAST, TOO EASY, WOW

``` /app/components/: $ ng g component my-component // Creates component ```

``` /app/components/my-component/: $ ng g component my-nested-component ```

Amongst others.
