# Hematite

[![Build Status](https://travis-ci.org/whity/perl6-hematite.svg?branch=master)](https://travis-ci.org/whity/perl6-hematite)

## Usage

```raku
# psgi file
use Hematite;

my $app = Hematite.new;

# middleware
$app.use(sub ($ctx, $next) {
    # some processing...

    # call next
    $next($ctx);

    # some processing...
});

class TestMiddleware does Callable {
    method CALL-ME($ctx, $next) {
        $next($ctx);
    }
}

$app.use(TestMiddleware.new);

# helpers (request context)
$app.helper('xpto', sub { say 'helper...'; });

# routes (can define any http method)
$app.GET('/', sub ($ctx) { $ctx.render({'route' => '/'}, 'type' => 'json'); });
#$app.POST('/', sub ($ctx) { $ctx.render({'route' => '/'}, 'type' => 'json'); });
#$app.METHOD('get', '/', sub ($ctx) { $ctx.render({'route' => '/'}, 'type' => 'json'); });

# route with middleware
$app.GET(
    '/with-middleware',
    [sub ($ctx, $next) { say 'route middleware'; $next($ctx); }],
    sub ($ctx) { $ctx.render({'route' => '/with-middleware'}, 'type' => 'json'); }
);

# route with placeholders/captures
$app.GET(
    '/captures/:c1',
    sub {
        say({
            'captures' => $ctx.captures,
            'named-captures' => $ctx.named-captures
        });
    }
);

# route rendering json
$app.GET(
    '/json',
    sub ($ctx) {
        $ctx.render(
            {'hello' => 'world'},
        );
    }
);

# route rendering template
$app.GET(
    '/template',
    sub ($ctx) {
        $ctx.render(
            'hello',
            data => { name => 'world', }
        );
    }
);

# route rendering inline template/string
$app.GET(
    '/template-inline',
    sub ($ctx) {
        $ctx.render(
            'hello {{ name }}',
            inline => True,
            data => { name => 'world', }
        );
    }
);


# groups
my $group = $app.group('/group');
$group.GET('/', sub ($ctx) { $ctx.render({'group' => 1}); });

$app;
```

### start crust

```bash
crustup [psgi file]
```

### using a more OO approach

```raku

class ExampleMiddleware does Hematite::Middleware {
    method CALL-ME {
        say 'example-middleware';
        return self.next;
    }
}

class ExampleAction does Hematite::Action {
    method middleware {
        return [
            ExampleMiddleware.create,
        ];
    }

    method CALL-ME {
        return self.render('example action', inline => True);
    }
}

class App is Hematite::App {
    method startup {
        self.use(ExampleMiddleware.create);

        self.GET('/', ExampleAction.create);
        self.POST('/', 'ExampleAction');
    }
}
```

## TODO

- better doc
- unit tests
- ...

## Contributing

1. Fork it ( https://github.com/[your-github-name]/perl6-hematite/fork )
2. Create your feature branch (git checkout -b my-new-feature)
3. Commit your changes (git commit -am 'Add some feature')
4. Push to the branch (git push origin my-new-feature)
5. Create a new Pull Request

## Contributors

- whity(https://github.com/whity) André Brás - creator, maintainer
