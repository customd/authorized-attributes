## Authorized Model Attributes for Laravel

Provides ability to dynamically add `$hidden` and `$fillable` columns to the models.

Also see [Laravel API Resources](https://laravel.com/docs/eloquent-resources#conditional-attributes) if that approach suits your needs.

Forked from [vantageoy/authorized-attributes](https://github.com/vantageoy/authorized-attributes).

<hr>

### Installation

Require the package to your Laravel project.

```
composer require custom-d/authorized-attributes
```

### Usage

> Please note that this package falls back to the core `Guard` and there are some minor differences of writing the policies between Laravel versions. See the official docs at https://laravel.com/docs/authorization

Use the `CustomD\AuthorizedAttributes\AuthorizedAttributes` trait

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use CustomD\AuthorizedAttributes\AuthorizedAttributes;

class Post extends Model
{
    use AuthorizedAttributes;

    /**
     * The attributes that should be fillable from requests.
     *
     * @var array
     */
    protected $fillable = ['title', 'content', 'author_id'];

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var array
     */
    protected $hidden = ['draft'];
}
```

[Create and register a model policy](https://laravel.com/docs/authorization#creating-policies).

```php
<?php

namespace App\Policies;

use App\Post;
use App\User;

class PostPolicy
{
    /**
     * Determine if an draft attribute can be seen by the user.
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @return bool
     */
    public function seeDraft(User $user, Post $post)
    {
    	// Post drafts can only be seen by admins and the post author
        return $user->isAdmin() || $user->created($post);
    }

    /**
     * Determine if the author_id attribute can be changed by the user.
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @return bool
     */
    public function editAuthorId(User $user, Post $post)
    {
    	// Admins can re-assign the author for non-published posts
        return $user->isAdmin() && $post->isNotPublished();
    }
}
```

<hr>

### Customization

#### Mixin with always hidden attributes

The attributes will be hidden if no policy or ability are found as they would normally be.

#### Modify the ability method names

```php
<?php

use Illuminate\Support\Str;

class Post extends Model
{
    /**
     * Get the method name for the attribute visibility ability in the model policy.
     *
     * @param  string  $attribute
     * @return string
     */
    public function getAttributeViewAbilityMethod($attribute)
    {
        return 'see'.Str::studly($attribute);
    }

    /**
     * Get the model policy ability method name to update an model attribute.
     *
     * @param  string  $attribute
     * @return string
     */
    public function getAttributeUpdateAbilityMethod($attribute)
    {
        return 'edit'.Str::studly($attribute);
    }
}
```
