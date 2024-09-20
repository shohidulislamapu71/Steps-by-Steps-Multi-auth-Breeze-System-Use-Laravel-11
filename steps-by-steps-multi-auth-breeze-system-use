### Step 1: Install Laravel Breeze
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

    const ROLE_USER = 'user';
    const ROLE_ADMIN = 'admin';

    protected $fillable = ['name', 'email', 'password', 'role'];
}
```

### Step 3: Create Middleware for Role-Based Access
Next, create middleware to restrict access based on user roles.

```bash
php artisan make:middleware RoleMiddleware
```

In the `RoleMiddleware` file (`app/Http/Middleware/RoleMiddleware.php`):

```php
public function handle($request, Closure $next, $role)
{
    if (auth()->check() && auth()->user()->role == $role) {
        return $next($request);
    }
    abort(403, 'Unauthorized');
}
```

Register this middleware in `app/Http/Kernel.php` under the `$routeMiddleware` array:

```php
protected $routeMiddleware = [
    // other middleware
    'role' => \App\Http\Middleware\RoleMiddleware::class,
];
```

### Step 4: Configure Authentication Guards (Optional for Multiple Guards)
If you want separate guards (e.g., one for users, one for admins), configure them in `config/auth.php`. By default, Breeze works with one guard, but you can add more as needed.

#### Example:
```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'admin' => [
        'driver' => 'session',
        'provider' => 'admins',
    ],
],
'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\Models\User::class,
    ],
    'admins' => [
        'driver' => 'eloquent',
        'model' => App\Models\Admin::class, // separate model for admins
    ],
],
```

### Step 5: Route Protection
Use the middleware you created to protect routes based on roles.

```php
Route::middleware(['auth', 'role:admin'])->group(function () {
    Route::get('/admin/dashboard', [AdminController::class, 'dashboard'])->name('admin.dashboard');
});

Route::middleware(['auth', 'role:user'])->group(function () {
    Route::get('/dashboard', [UserController::class, 'dashboard'])->name('dashboard');
});
```

### Step 6: Customizing the Breeze Authentication Logic (Optional)
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

This will ensure that users are redirected to the appropriate dashboard after login, depending on their role.

---

This setup should provide you with a multi-auth system using Laravel Breeze. Depending on your project needs, you can further extend this with custom guards, password resets, or multiple user tables if required. Let me know if you'd like to explore further customization!
