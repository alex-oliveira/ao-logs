# Ao-Logs

### 1) Installing
````
$ composer require alex-oliveira/ao-logs
````

### 2) Configuring "config/app.php" file
````
'providers' => [
    /*
     * Vendor Service Providers...
     */
    AoLogs\ServiceProvider::class,
],
````

### 3) Create "config/ao.php" file
````
return [
    .
    .
    .
    'models' => [
        'users' => App\Models\User::class,
    ],
        
    'tables' => [
        'users' => 'users'
    ]
    .
    .
    .
];
````

### 4) Publish migrations
````
$ php artisan vendor:publish --tag=ao-logs
````
````
$ composer dump
````





# Utilization 

## Migration

### Up
````
public function up()
{
    AoLogs()->schema()->create('users');
}
````
the same that
````
public function up()
{    
    Schema::create('ao_logs_x_users', function (Blueprint $table) {
        $table->integer('user_id')->unsigned();
        $table->foreign('user_id', 'fk_users_x_ao_logs')->references('id')->on('users');
        
        $table->bigInteger('log_id')->unsigned();
        $table->foreign('log_id', 'fk_ao_logs_x_users')->references('id')->on('ao_logs_logs');
        
        $table->primary(['user_id', 'log_id'], 'pk_ao_logs_x_users');
    });
}
````

### Down
````
public function down()
{
    AoLogs()->schema()->drop('users');
}
````
the same that
````
public function down()
{    
    Schema::dropIfExists('ao_logs_x_users');
}
````





## Model
````
namespace App\Models;

use AoLogs\Traits\AoLogsTrait;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    use AoLogsTrait;   
}
````
the same that
````
namespace App\Models;

use AoLogs\Models\Log;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * @return Log[]|\Illuminate\Database\Eloquent\Relations\BelongsToMany
     */
    public function logs()
    {
        return $this->belongsToMany(Log::class, 'ao_logs_x_users');
    }
}
````





## Controller
````
namespace App\Http\Controllers\Users;

use AoLogs\Controllers\AoLogsController;
use App\Models\User;

class LogsController extends AoLogsController
{
    protected $dynamicClass = User::class;
}
````





## Routes
````
Route::group(['prefix' => 'users', 'as' => 'users.'], function () {
    
    AoLogs()->router()->controller('Users\LogsController')->foreign('user_id')->make();
    .
    .
    .
    
});
````

### Checking routes
````
$ php artisan route:list
````





## Registering log
````
$category = \App\Models\Category::find(1);
.
.
.
AoLogs()->post($category, [
    'title' => 'Cadastro realizado.',
    'description' => 'O usuário "Alex Oliveira" realizou o cadastro da categoria "Computadores".'
]);
````