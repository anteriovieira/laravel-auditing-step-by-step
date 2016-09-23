# laravel-auditing-step-by-step

I will try to demonstrate the use of the package. Let's imagine a situation in company.

### Install Laravel Auditing

To get started, install Laravel Auditing via the Composer package manager:

`composer install owen-it/laravel-auditing`

Next, register the provider in the providers array of your config/app.php configuration file:

```php
	'providers' => [
	    // ...
	    OwenIt\Auditing\AuditingServiceProvider::class,
	],
```

### Auditors

By default Laravel Audit uses the auditor Database Auditor, it will record the changes in the database using the `Auditing ` model. So let's create the table so that we can use this auditor:

`php artisan auditing:table`

Run migrations

`php artisan migrates`

Now, if our audit model is different from the standard, we can create our own auditor to audit the data:

`php artisan make:auditor CustomAuditor`

This command will place a fresh auditor class in your app/Auditors directory. Each auditor class contains a audit method responsible for auditing the eloquent model.

```php
<?php
namespace App\Auditors;

use App\Auditor;
use OwenIt\Auditing\Contracts\Dispatcher as AuditingContract

class CustomAuditorimplements AuditingContract
{
    public function audit ($auditable)
    {
        return Auditor::create($auditable->toAudit());
    }
}
```

> The return of the audit method will then be sent as a parameter to the event AuditReport. 

### Auditable models

Now we can specify which model we want to audit:

```php
<?php

namespace App;

use OwenIt\Auditing\Auditable;
use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
	use Auditable;

	//
}
```

In our example, we specify the model of the auditor, if we do not specify the Laravel Auditing will use the default.

```php
<?php

namespace App\User;

use OwenIt\Auditing\Auditable;
use App\Auditors\CustomAuditor;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
	use Auditable;

	protected $auditor = [CustomAuditor::class]

	//
}
```

### Audits Events

In our example I would like to review all the records and verify that the data is actually relevant to the audit. So we register a listener for the report event to make this check.

```php
protected $listen = [
    'OwenIt\Auditing\Events\AuditReview' => [
        'App\Listeners\ManageAudit',
    ],
];
```

Of course, manually creating the files for each event and listener is cumbersome. Instead, simply add listeners and events to your EventServiceProvider and use the event:generate command. 

`php artisan event:generate`

Now we can handle our `ManageAudit`.

```php
<?php

namespace App\Listeners;

use OwenIt\Auditing\Events\AuditReview;

class ManageAudit
{
    /**
     * Create the event listener.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Handle the event.
     *
     * @param  AuditReview $event
     * @return void
     */
    public function handle(AuditReview $event)
    {
        if( /* can audit */ ){
			return true;
		} 

		return false;
    }
}
```

Very good! but I would also like to notify the managers if there is any discrepancy in the audited information. We will create a listener to send notifications and name it as `NotifyAudit`:

```php
protected $listen = [
    'OwenIt\Auditing\Events\AuditReview' => [
        'App\Listeners\ManageAudit',
    ],
    'OwenIt\Auditing\Events\AuditReport' => [
        'App\Listeners\NotifyAudit',
    ],
];
```

Then runs the command to create the listeners:

`php artisan event:generate`

Now we can handle our `NotifyAudit`.


```php
<?php

namespace App\Listeners;


use App\Notifications\Manager;
use OwenIt\Auditing\Events\AuditReport;

class NotifyAudit
{
    /**
     * Create the event listener.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Handle the event.
     *
     * @param  AuditReport $event
     * @return void
     */
    public function handle(AuditReport $event)
    {
        if( /* should notify managers */ ){
	        $users = User::where('role', 'manager')->get(); 

			$users->notify(new Manager($event->auditable));
		} 
    }
}
```

### Retrieve audits

You may easily retrieve an array of a auditable model's using the audits method:

```php
$audits = $user->audits();
```

...




