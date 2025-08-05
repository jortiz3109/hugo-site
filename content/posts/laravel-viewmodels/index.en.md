---
title: "Laravel Viewmodels"
date: 2021-03-23
draft: false
tags: ["View Models"]
categories: ["laravel", "Design patterns"]
showTableOfContents: true
summary: In this article I am going to show you how to implement View Models in our controllers.
---
## The problem

When we start programming in Laravel it's common for us to overload our controllers with many responsibilities.

something more or less like this:

```php
// MerchantController.php

public function create(): View
{
    return view('admin.merchants.create', [
        'countries' => Country::all(),
        'currencies' => Currency::all()
    ]);
}
```

If we add business requirements to this (ACLs, scopes, etc) we could end up with a huge controller that will make it very difficult to maintain in the future.

## What is a ***ViewModel***?

**ViewModels** are a kind of pattern that allows you to encapsulate all the logic needed to prepare the data thats will be used by the view.

### Let's do it!

First, we are going to create an abstract class named ViewModel

```php
<?php
// ViewModel.php

namespace App\ViewModels;
use Illuminate\Contracts\Support\Arrayable;
use Illuminate\Database\Eloquent\Model;

abstract class ViewModel implements Arrayable
{
    protected Model $model;

    public function __construct(Model $model)
    {
        $this->model = $model;
    }

    abstract public function toArray(): array;

    protected function model(): Model
    {
        return $this->model;
    }
}

```

Next, we are going to create our concrete implementation

```php
// CreateViewModel.php

<?php
namespace App\ViewModels\Admin\Merchants;

use App\Models\Merchant;
use App\Models\Country;
use App\Models\Currency;
use App\Models\MerchantCategoryCode;
use App\ViewModels\ViewModel;

class CreateViewModel extends ViewModel
{
    public function __construct()
    {
        parent::__construct(new Merchant());
    }
    
    public function toArray(): array
    {
        return [
            'countries' => Country::all(),
            'currencies' => Currency::all(),
            'mccs' => MerchantCategoryCode:all(),
            'merchant' => $this->model()
        ];
    }
}
```

Finally, we refactor the Controller to use our ViewModel.

```php
// PostController.php

<?php
namespace App\Http\Controllers\Admin;
use App\View\ViewModels\Admin\Merchants\CreateViewModel;
use Illuminate\Http\Response;
class PostController extends Controller
{
    public function create(): Response
    {
        return view('admin.merchants.create', new CreateViewModel());
    }
}
```
You can view a ViewModel implementation at this link on [Github](https://github.com/jortiz3109/johndev/tree/master/app/View/ViewModels)