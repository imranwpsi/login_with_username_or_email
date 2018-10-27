# Create Authentication
To make the laravel default Authentication, run the following command:

```
$  php artisan make:auth
```
# Routes

```php
// no email verification
Auth::routes();

// after adding, email verification
Auth::routes(['verify' => true]);
```

# Database
For existing project add a new migration file using an artisan command to add a new field to the users' 
table, and for the new fresh project, 
you can easily edit the migration file generated with auth scaffolding and add a single line below.

```
$table->string('username')->unique();
```

To fill value in the database for `username` field, we've to add a key username to the `$fillable` property under `app/User.php` database model.

```php
protected $fillable = ['name', 'email', 'password', 'username'];
```

# Registration View
The user registration form `register.blade.php` also requires a little bit of customization to add the username field.

```php
<div class="form-group row">
    <label for="username" class="col-md-4 col-form-label text-md-right">{{ __('Username') }}</label>

    <div class="col-md-6">
        <input id="username" type="text" class="form-control{{ $errors->has('username') ? ' is-invalid' : '' }}" name="username" value="{{ old('username') }}" required>

        @if ($errors->has('username'))
            <span class="invalid-feedback" role="alert">
                <strong>{{ $errors->first('username') }}</strong>
            </span>
        @endif
    </div>
</div>
```

# Login View
The login form `login.blde.php` requires a very little change on the input type field from email to text.

```php
<input id="email" type="text" class="form-control{{ $errors->has('email') ? ' is-invalid' : '' }}"
name="email" value="{{ old('email') }}" required autofocus>
```

# Login Controller
To apply the new changes we need, we have to override `LoginController.php`
the methods under the controller class for the login system.

```php
// LoginController.php

    /**
     * Get the needed authorization credentials from the request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    protected function credentials(Request $request)
    {
        $field = $this->field($request);

        return [
            $field => $request->get($this->username()),
            'password' => $request->get('password'),
        ];
    }

    /**
     * Determine if the request field is email or username.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return string
     */
    public function field(Request $request)
    {
        $email = $this->username();

        return filter_var($request->get($email), FILTER_VALIDATE_EMAIL) ? $email : 'username';
    }

    /**
     * Validate the user login request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return void
     */
    protected function validateLogin(Request $request)
    {
        $field = $this->field($request);

        $messages = ["{$this->username()}.exists" => 'The account you are trying to login is not registered or it has been disabled.'];

        $this->validate($request, [
            $this->username() => "required|exists:users,{$field}",
            'password' => 'required',
        ], $messages);
    }
```

# Register Controller
In `RegisterController` we have to add this method.

```php
protected function validator(array $data)
  {
      return Validator::make($data, [
          'name' => ['required', 'string', 'max:255'],
          'email' => ['required', 'string', 'email', 'max:255', 'unique:users'],
          'username' => ['required', 'string', 'max:255', 'unique:users'],
          'password' => ['required', 'string', 'min:6', 'confirmed'],
      ]);
  }
```

# To Make Mail Verification
The new class is introduced in the framework to implement the email verification system.
So, now the `App\User` model should implement the `Illuminate\Contracts\Auth\MustVerifyEmail` contract.

```php
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements MustVerifyEmail
{
    // ...
}
```

# Middleware
A new middleware `Illuminate\Auth\Middleware\EnsureEmailIsVerified` 
class is introduced to protect the routes from being accessed by the unauthorized users.
You can have a look at the `Kernel.php` to see the registered middleware.

```php
protected $routeMiddleware = [
    ...
    'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
];
```
Also, you may want to protect other routes in `web.php`, like profile, 
the dashboard to non-activated users by attaching the middleware on routes.

```php
// example
Route::get('/home', 'HomeController@index')
    ->name('home')
    ->middleware('verified');

Route::get('/member/profile', function () {
    // verified users only
})->middleware('verified');
```

# ENV
Add mail configation in `.env`.
Note: Don't forget to restart your server after editing the .env file so it will pick the new data that you put in there.
```
MAIL_DRIVER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=MyUsername@gmail.com
MAIL_PASSWORD=MyPassword
```

# Mail Config
To add mail configaration in `config/mail.php`.

```php
'stream' => [
   'ssl' => [
      'allow_self_signed' => true,
      'verify_peer' => false,
      'verify_peer_name' => false,
   ],
],
```


