### Step 1: Install Laravel 11 Breeze
First, ensure Laravel Breeze is installed in your application.

```bash
composer require laravel/breeze --dev
php artisan breeze:install
npm install && npm run dev
php artisan migrate
```

### Step 2: Define the User Roles
In your `User` model (`app/Models/User.php`), define a field to distinguish between different user roles. You can add a `role` column to your `users` table.

#### Migration
Create a new migration to add the `role` field:

```bash
php artisan make:migration add_role_to_users_table --table=users
```

In the migration file:

```php
public function up()
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('role')->default('user'); // default role is 'user'
    });
}

public function down()
{
    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('role');
    });
}
```

Run the migration:

```bash
php artisan migrate
```

#### Updating User Model
In your `User` model, you can define roles as constants or handle it however you'd like:

```php
class User extends Authenticatable
{
    use HasFactory, Notifiable;

    protected $fillable = ['name', 'email', 'password', 'role'];
}
```

### Step 3: Create Middleware for Role-Based Access
Next, create middleware to restrict access based on user roles.

```bash
php artisan make:middleware RoleMiddleware
```

Middlewar File write this code for check auth
```php
public function handle($request, Closure $next, $role)
{
    if (auth()->check() && auth()->user()->role == $role) {
        return $next($request);
    }
    abort(403, 'Unauthorized');
}
```

Register this middleware in `bootstrap/app.php` under the `$routeMiddleware` array:

```php
->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'role' => RoleMiddleware::class,
        ]);
})
```

### Step 4: Route Protection
Use the middleware you created to protect routes based on roles. 

```php
Route::middleware(['auth', 'role:admin'])->group(function () {
    Route::get('/admin/dashboard', [AdminController::class, 'dashboard'])->name('admin.dashboard');
});

Route::middleware(['auth', 'role:user'])->group(function () {
    Route::get('/dashboard', [UserController::class, 'dashboard'])->name('dashboard');
});
```

### Step 5: Customizing the Breeze Authentication Logic (Optional)
If you're using a single guard but want to customize login based on user roles, you can modify the `LoginController` to redirect users based on their role after login.

#### Example in `LoginController`:
```php
protected function authenticated(Request $request, $user)
{
    if ($user->role == 'admin') {
        return redirect()->route('admin.dashboard');
    }
    return redirect()->route('dashboard');
}
```
