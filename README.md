<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Column sorting for Laravel 5.*](#column-sorting-for-laravel-5)
- [Setup](#setup)
  - [Composer](#composer)
  - [Publish configuration](#publish-configuration)
- [Usage](#usage)
  - [Blade Extension](#blade-extension)
  - [Configuration in few words](#configuration-in-few-words)
  - [Font Awesome (default font classes)](#font-awesome-default-font-classes)
  - [Full Example](#full-example)
    - [Routes](#routes)
    - [Controller's `index()` method](#controllers-index-method)
    - [View](#view)
- [HasOne / BelongsTo Relation sorting](#hasone--belongsto-relation-sorting)
    - [Define hasOne relation](#define-hasone-relation)
    - [Define belongsTo relation](#define-belongsto-relation)
  - [Define `$sortable` arrays](#define-sortable-arrays)
  - [Blade and relation sorting](#blade-and-relation-sorting)
- [ColumnSortable overloading (advanced)](#columnsortable-overloading-advanced)
- [`$sortableAs` (aliasing)](#sortableas-aliasing)
- [Exception to catch](#exception-to-catch)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Column sorting for Laravel 5.*
[![Latest Version](https://img.shields.io/github/release/Kyslik/column-sortable.svg?style=flat-square)](https://github.com/Kyslik/column-sortable/releases)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE.md)
[![Total Downloads](https://img.shields.io/packagist/dt/Kyslik/column-sortable.svg?style=flat-square)](https://packagist.org/packages/Kyslik/column-sortable)
[![Build Status](https://travis-ci.org/Kyslik/column-sortable.svg?branch=master)](https://travis-ci.org/Kyslik/column-sortable)

Package for handling column sorting in Laravel 5.[1-4].

Simply put: [this hack](http://hack.swic.name/laravel-column-sorting-made-easy/) in package with blade extension and Font Awesome icon support.

>This is my shot at universal and easy to use model sorting in Laravel. The end result allows you to sort an Eloquent model using any column by clicking the column name. Everything is done in PHP, no JS involved.

# Setup

## Composer

Pull this package in through Composer (development/latest version `dev-master`)

```
{
    "require": {
        "kyslik/column-sortable": "5.4.*"
    }
}
```

    $ composer update

>**Note**: Major and minor versions should match with Laravel, build versions are separate from Laravel versioning scheme. Example: If you are using Laravel 5.2, column-sortable version should be `5.2.*`.

Add the package to your application service providers in `config/app.php`

```
'providers' => [

    App\Providers\RouteServiceProvider::class,

    /*
     * Third Party Service Providers...
     */
    Kyslik\ColumnSortable\ColumnSortableServiceProvider::class,
],
```
## Publish configuration

Publish the package configuration file to your application.

    $ php artisan vendor:publish --provider="Kyslik\ColumnSortable\ColumnSortableServiceProvider" --tag="config"

See configuration file [(`config/columnsortable.php`)](https://github.com/Kyslik/column-sortable/blob/master/src/config/columnsortable.php) yourself and make adjustments as you wish.

# Usage

Use **Sortable** trait inside your *Eloquent* model(s). Define `$sortable` array (see example code below).

>**Note**: `Scheme::hasColumn()` is run only when `$sortable` is not defined - less DB hits per request.


```
use Kyslik\ColumnSortable\Sortable;

class User extends Model implements AuthenticatableContract, CanResetPasswordContract 
{
	use Authenticatable, CanResetPassword, Sortable;
	...

	public $sortable = ['id',
	                    'name',
	                    'email',
	                    'created_at',
	                    'updated_at'];
	                    
	...
}
```

You're set to go.

**Sortable** trait adds Sortable scope to the models so you can use it with paginate.

## Blade Extension

There is one blade extension for you to use **@sortablelink()**

```
@sortablelink('column', 'Title', ['parameter' => 'smile'])
```

**Column** (1st) parameter is `order by`, **Title** (2nd) parameter is displayed inside anchor tags and `array()` parameter (3rd) is default (GET) query strings parameter.  

You can omit 2nd and 3rd parameter.

Possible examples and usages of blade extension:

```
@sortablelink('name')
@sortablelink('name', 'Username')
@sortablelink('address', trans('fields.address'), ['filter' => 'active,visible'])
```

If you do not fill **Title** (2nd parameter) column name is used instead.

>**Note**: you can set default formatting function that is applied on **Title** (2nd parameter), by default this is set to [`ucfirst`](http://php.net/manual/en/function.ucfirst.php).

## Configuration in few words

**Sortablelink** blade extension distinguishes between "types" (numeric, amount and alpha) and applies different class for each of them.  

See following snippet:

```
'columns' => [
    'numeric'  => [
        'rows' => ['created_at', 'updated_at', 'level', 'id'],
        'class' => 'fa fa-sort-numeric'
    ],
    'amount'   => [
        'rows' => ['price'],
        'class' => 'fa fa-sort-amount'
    ],
    'alpha'    => [
        'rows' => ['name', 'description', 'email', 'slug'],
        'class' => 'fa fa-sort-alpha',
    ],
],
```

Rest of the [config file](https://github.com/Kyslik/column-sortable/blob/master/src/config/columnsortable.php) should be crystal clear and I advise you to read it.

## Font Awesome (default font classes)

Install [Font-Awesome](https://github.com/FortAwesome/Font-Awesome) for visual [Joy](http://www.imdb.com/character/ch0411388/). Search "sort" in [cheatsheet](http://fortawesome.github.io/Font-Awesome/cheatsheet/) and see used icons (12) yourself.

## Full Example

### Routes

```
Route::get('users', ['as' => 'users.index', 'uses' => 'HomeController@index']);
```

### Controller's `index()` method

```
public function index(User $user)
{
    $users = $user->sortable()->paginate(10);
    
    return view('user.index')->withUsers($users);
}
```

You can set default sort when nothing is in (URL) query strings yet.
>**For example**: page is loaded for first time, default direction is [configurable](https://github.com/Kyslik/column-sortable/blob/master/src/config/columnsortable.php#L77) (asc)

```
$users = $user->sortable('name')->paginate(10);
//generate ->orderBy('users.name', 'asc')

$users = $user->sortable(['name'])->paginate(10); 
//generate ->orderBy('users.name', 'asc')

$users = $user->sortable(['name' => 'desc'])->paginate(10);
//generate ->orderBy('users.name', 'desc')
```

### View

_pagination included_

```
@sortablelink('id', 'Id')
@sortablelink('name')

@foreach ($users as $user)
    {{ $user->name }}
@endforeach
{!! $users->appends(\Request::except('page'))->render() !!}
```

>**Note**: Blade's ability to recognize directives depends on having space before directive itself `<tr> @sortablelink('Name')`

# HasOne / BelongsTo Relation sorting

### Define hasOne relation

In order to make relation sorting work, you have to define **hasOne()** relation in your model.

```
/**
* Get the user_detail record associated with the user.
*/
public function detail()    
{
    return $this->hasOne(App\UserDetail::class);
}
```

### Define belongsTo relation

```
/**
 * Get the user that owns the phone.
 */
public function user()
{
    return $this->belongsTo(App\User::class);
}
```

In *User* model we define **hasOne** relation to *UserDetail* model (which holds phone number and address details).

## Define `$sortable` arrays

Define `$sortable` array in both models (else, package uses `Scheme::hasColumn()` which is an extra database query).


for *User*

```
public $sortable = ['id', 'name', 'email', 'created_at', 'updated_at'];
```

for *UserDetail*

```
public $sortable = ['address', 'phone_number'];
```

## Blade and relation sorting

In order to tell package to sort using relation:

```
@sortablelink('detail.phone_number', 'phone')
@sortablelink('user.name', 'name')
```
>**Note**: package works with relation "name" (method) that you define in model instead of table name.

>**WARNING**: do not use combination of two different relations at the same time, you are going to get errors that relation is not defined.

In config file you can set your own separator if `.` (dot) is not what you want.

```
'uri_relation_column_separator' => '.'
```

# ColumnSortable overloading (advanced)

It is possible to overload ColumnSortable relation feature, basically you can write your own join(s) / queries and apply `orderBy()` manualy.

See example:

```
class User extends Model
{
    use Sortable;
    
    public $sortable = ['name'];
    ...
    
    public function addressSortable($query, $direction)
    {
        return $query->join('user_details', 'users.id', '=', 'user_details.user_id')
                    ->orderBy('address', $direction)
                    ->select('users.*');
    }
    ...
```

Controller is the same `$users = $user->sortable()->paginate(10);`

In view just use `@sortablelink('address')`

>Huge thanks to @neutralrockets and his comments on [#8](https://github.com/Kyslik/column-sortable/issues/8).

One more example: [#41](https://github.com/Kyslik/column-sortable/issues/41#issuecomment-250895909)

# `$sortableAs` (aliasing)

It is possible to declare `$sortableAs` array and use it to alias (bypass column exists check), 
and ignore prefixing with table. 

In model 
```
...
$sortableAs = ['nick_name'];
...

```

In controller

```
$users = $user->select(['name as nick_name'])->sortable(['nick_name'])->paginate(10);
```

In view
```
@sortablelink('nick_name', 'nick')
```


Please see [#44](https://github.com/Kyslik/column-sortable/issues/44).


# Exception to catch

#### Package throws custom exception `ColumnSortableException` with three codes (0, 1, 2).

Code **0** means that `explode()` fails to explode URI parameter "sort" in to two values.
For example: `sort=detail..phone_number` - produces array with size of 3, which causes package to throw exception with code **0**.

Code **1** means that `$query->getRelation()` method fails, that means when relation name is invalid (does not exists, is not declared in model).

Code **2** means that provided relation through sort argument is not instance of **hasOne**.

Example how to catch:

```
try {
    $users = $user->with('detail')->sortable(['detail.phone_number'])->paginate(5);    
    } catch (ColumnSortableException $e) {
    dd($e);
}
```

>**Note**: I strongly recommend to catch **ColumnSortableException** because there is a user input in question (GET parameter) and any user can modify it in such way that package throws ColumnSortableException with code 0.
