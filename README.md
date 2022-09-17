# A Laravel's Guide to Database Notification

When building applications, notifications are essential because they improve user experience and interaction. It would help if you frequently alerted users to various changes or activities while they use applications. It can involve sending an SMS about their login activity for security purposes or email notifications when the order status changes. These notifications often only offer a brief explanation of the state changes. [Laravel](https://laravel.com/docs/9.x/notifications#introduction) supports several delivery channels for sending notifications, including emails, SMS, and [Slack](https://slack.com/). Additionally, notifications may be kept in a database and displayed on your web interface. Here are a few things you'll learn :
    - What are notifications?
    - Notification Channels supported by Laravel
    - How to use live database notifications in Laravel
    - How to use email notifications in Laravel
    - Other Laravel-supported notification delivery channels

## Laravel Notifications

Notifications can be considered brief, direct messages sent to users to inform them of important information or events or to prompt a response in the application. Ideally, they keep users up to date and increase user engagement. Laravel provides support for sending notifications via a variety of channels. By default, it includes `[mail](https://laravel.com/docs/9.x/notifications#mail-notifications)`, `[slack](https://laravel.com/docs/9.x/notifications#slack-notifications)`, `[database](https://laravel.com/docs/9.x/notifications#database-notifications)`, `broadcast`, and `[vonage](https://www.vonage.com/communications-apis/)` channels. You can visit the [community-driven Laravel Notification Channels website](https://laravel-notification-channels.com/) to leverage other delivery channels like Telegram or Pusher.
By using the Laravel `artisan` command, you can quickly create notifications. You can also customize the notification's email subject, body, and central action, among other things.

## Laravel's Notification Channels

Laravel allows you to select from various notification channels to send notifications in your application. You can use more than one channel.

**Mail**
These notifications is sent as email to users.

**SMS**
Users will receive this SMS notifications on their mobile phones.

**Database**
These notifications are stored in the database and you can display it to the user with a custom UI.

**Slack**
These notifications are sent to slack channels.

**Community Driven Notifications**
You can check out the community driven [Laravel Notification Channels website](http://laravel-notification-channels.com/) if you want to use various other channels like Telegram.

You can also decide to [create your own drivers to deliver notifications via other channels](https://laravel.com/docs/9.x/notifications#custom-channels).

In this tutorial, we'll build a Laravel application that shows how to send notifications via `[email](https://laravel.com/docs/9.x/notifications#mail-notifications)` and `[database](https://laravel.com/docs/9.x/notifications#database-notifications)` channels.

## How to Send Database Notifications

The notifications are kept in a database table when sent via the database channel. The type of notification and the JSON data that identifies the notification are both included in this table.

### Create a New Laravel Project

You can create a new Laravel project via the Composer command or the Laravel installer:

``` bash
    laravel new project_name   
            or
    composer create-project laravel/laravel project_name
```

This will create a fresh Laravel application in a new directory named *project_name.*

### Connect to your database

[Here](https://dev.to/roxie/how-to-connect-laravel-application-to-mysql-database-5han) is an article I wrote that explains how to [connect a Laravel Application to a MySQL database](https://dev.to/roxie/how-to-connect-laravel-application-to-mysql-database-5han). If you have a different database, make sure to connect it appropriately.

### Set up Default Authentication Scaffold

Laravel Auth provides a prebuilt authentication UI and functionality to interact with.  Install the Laravel UI package with this command:

```bash
    composer require laravel/ui
```

Then create a bootstrap auth scaffold. A bootstrap authentication scaffold provides a default UI and basic authentication for registration and login in Laravel. You can install it with the `*Artisan*` command:

```bash
    php artisan ui bootstrap --auth
```

Next, install the  `npm` packages to generate the `css` and `js` files and run the environment.

```bash
    npm install
    
    npm run dev
```

Finally , run your migrations to create your database tables with this command:

```bash
    php artisan migrate
```

### Run the application

Run this `Artisan` command to serve the project:

```bash
    php artisan serve
```

The application is served on port `8000` by default , and if you visit `http://localhost:8000/` on your browser you should be view this landing page with a `login` and `register` option at the top right.

[Image]

Upon a successful registration or login, then you can view your dashboard. All these basic authentication process and UI has been handled by the `laravel/ui` package installed earlier.
[Image]

### Create a Notifications Table

You need to create a database table to contain all your notifications. It can be queried at anytime to display notifications to the user interface. To generate a migration with a proper schema notification table schema, you may use this `Artisan` command:

```bash
    php artisan notifications:table
```

This creates a *create_notifications_table.php* in the *database/migrations* directory which defines the schema of the notifications table. Now you can migrate to your database by running:

```bash
    php artisan migrate
```

### Generating Notifications

Each notification in Laravel is represented by a single class, which is normally stored in the app/Notifications directory. It will be generated when you run the `make:notification` Artisan command:

```php
    php artisan make:notification DepositSuccessful
```

This generates a *DepositSuccessful.php* file with a new notification class in the *app/Notifications* directory. This class includes a `via()` method that specifies the channel and body message-building methods to assist in formatting the notification data for the selected channel.

The notification class should have a `toDatabase` or `toArray` function defined This method should return a simple PHP array after receiving a `$notifiable` entity. The returned array will be stored in the `data` column of your notifications table after being encoded as JSON.

### Formatting Database Notifications

You want to notify users whenever they make a successful deposit in the app of the amount they deposited.
Update the `__construct()` method in the *DepositSuccessful.php* file to match this example:

```php
      protected $amount; 
        public function __construct($amount)
        {
            $this->amount=$amount;
        }
```

Now the deposit amount has been injected in the `construct()` method, so it can be used in generating mail messages.

Now, you need to specify the notification channel to use by updating the `via()` method:

```php
      public function via($notifiable)
        {
            return ['database'];
        }
```

Afterwards, you update the `toArray()` or `toDatabase()` method with the details of the notification.
You must define either a `toDatabase()` or a `toArray()` method, and it should return a simple PHP array which will be stored in the `data` column of the notifications table.

```php
    public function toArray($notifiable)
        {
            return [
                'data'=>'Your deposit of '. $this->amount. ' was successful'
            ];
        }
```

> The `toArray()` is used by both the database broadcast channel. If you want to use both channels in the app with different array representations , it is best you define `toDatabase()` and `toArray()`. However `toArray()` is the default for either of them.

### Add the Notifiable Trait

Laravel notification offers two methods to send notifications. It includes the `Notifiable` trait or `Notification` facade.  In this tutorial, we will focus the Notifiable trait.
By default, Laravel includes the Notifiable trait in `app/Models/User` model. It should match this example:

```php
    class User extends Authenticatable
    {
        use HasApiTokens, HasFactory, Notifiable;
    .
    .//Other model definitions
    .
    }
```

### Set up the Deposit model and Migration

First, create the model and database migration simultaneously, by running the command below:

```bash
    php artisan make:model Deposit -m
```

This creates a [Model](https://laravel.com/docs/8.x/eloquent) file called *Deposit.php* in the *app/Models* directory, and a [Migration](https://laravel.com/docs/8.x/migrations#introduction) file called *create_deposits_table.php* in the *database/migrations* directory.

Update Deposit*.php* by adding the code below to the top of the file, which enables Model [mass assignment](https://laravel.com/docs/8.x/eloquent#mass-assignment).

```php
    protected $guarded = [];
```

Then, update the `up()` method of the *create_deposits_table.php* migration file with details of the deposit to be stored as in the example below:

```php
        public function up()
        {
            Schema::create('deposits', function (Blueprint $table) {
                $table->id();
                $table->string('amount');
                $table->foreignId('user_id')->references('id')->on('users')->constrained()            ->onDelete('cascade')->onUpdate('cascade');   
               $table->timestamps();
            });
        }
```

Then , run your migrations to the database again.

```bash
    php artisan migrate
```

### Set up Controller

You need to define the logic for making a deposit and sending the  notification after successful deposit in the [controller](https://laravel.com/docs/9.x/controllers).
To create the controller, run this Artisan command:

```bash
    php artisan make:controller DepositController
```

A new file *DepositController.php*  is created in the *app/Http/Controllers* directory.

With the file created, add the `import` statements below to import the classes which the controller will use:

```php
    use App\Models\Deposit;
    use App\Models\User;
    use App\Notifications\DepositSuccessful;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;
```

Add a  `_construct()` method to declare the auth middleware which allows only authenticated users to make deposit.

```php
     public function __construct()
        {
            $this->middleware('auth');
        }
```

Define a `deposit()` method to include logic for making a deposit and sending a successful deposit notification.

```php
    public function deposit(Request $request){
            $deposit = Deposit::create([
                'user_id' =>Auth::user()->id,
                'amount'  => $request->amount
            ]);
            User::find(Auth::user()->id)->notify(new DepositSuccessful($deposit->amount));
            
            return redirect()->back()->with('status','Your deposit was successful!');
        }
```

Notifications are sent using either the `notify()` method from `[Notifiable](https://laravel.com/docs/9.x/notifications#using-the-notifiable-trait)` [trait](https://laravel.com/docs/9.x/notifications#using-the-notifiable-trait) or `[Notification](https://laravel.com/docs/9.x/notifications#using-the-notification-facade)` [facade](https://laravel.com/docs/9.x/notifications#using-the-notification-facade). The facade is useful when you need to send notifications to multiple entities like a collection of users. Although we are using `notify()` for this guide, here is an example of using `Notification` facade.

```php
    $users = User::all();
    
    Notification::send($users, new DepositSuccessful($deposit->amount));
```

> Note: The Notifiable trait has already been imported in the User model.  You can import it in any model you need to send notifications to.

### Set up the routes

You will add a new [route](https://laravel.com/docs/8.x/routing) in *routes/web.php*. We only need to define one since we have only one endpoint `/deposit`, for users to make a deposit.

```php
    Route::post('/deposit', [App\Http\Controllers\DepositController::class,'deposit'])->name('deposit');
```

### Modify the View

Add a basic form to the home page for a user to make deposit  in *resources\views\home.blade.php*  directory.

```php
        <form method="POST" action="{{ route('deposit') }}">
                            @csrf
                            <h5 class="text-center mb-3">Make A Deposit</h5>
                            <div class="row mb-3">
                                <label for="amount" class="col-md-4 col-form-label text-md-end">{{ __('Amount') }}</label>
                                <div class="col-md-6">
                                    <input id="amount" type="number" class="form-control @error('amount') is-invalid @enderror" name="amount" value="{{ old('amount') }}" required autocomplete="amount" autofocus>
                                    @error('amount')
                                        <span class="invalid-feedback" role="alert">
                                            <strong>{{ $message }}</strong>
                                        </span>
                                    @enderror
                                </div>
                            </div>
                            <div class="row mb-0">
                                <div class="col-md-8 offset-md-4">
                                    <button type="submit" class="btn btn-primary">
                                        {{ __('Deposit') }}
                                    </button>
                                </div>
                            </div>
                        </form>
```

Awesome job👍!  Now lets make things more interesting by including a bell notification at the navbar that shows the number of unread notifications. You can also display both read and unread notifications.
Laravel `Notifiable` trait provides another awesome feature that can help you track both read and unread notifications. You can also [mark a notification as read or delete a notification entirely](https://laravel.com/docs/9.x/notifications#marking-notifications-as-read).

### Update Controller , Route and View With MarkasRead  Feature

Define a new method `markAsRead()` in the *DepositController.php* to mark all unread notifications as read.

```php
      public function markAsRead(){
            Auth::user()->unreadNotifications->markAsRead();
            return redirect()->back();
        }
```

Now, add the corresponding route to the *routes/web.php .*

```php
    Route::get('/mark-as-read', [App\Http\Controllers\DepositController::class,'markAsRead'])->name('mark-as-read');
```

You will be updating the *resources\views\layouts\app.blade.php* with a few changes for this feature.
Firstly,  include this CDN link to [font-awesome](https://cdnjs.com/libraries/font-awesome) in the blade file to enable you use the bell icon.

```php
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.2.0/css/all.min.css" integrity="sha512-xh6O/CkQoPOWDdYTDqeRdPCVd1SpvCA9XXcUnZS2FmJNp1coAFzvtCN9BmamE+4aHK8yyUHUSCcJHgXloTyT2A==" crossorigin="anonymous" referrerpolicy="no-referrer" />
```

Now, add this example before the fullname and logout dropdown, immediately after the `else` statement.

```php
                               <li class="nav-item dropdown">
                                <a id="navbarDropdown" class="nav-link " href="#" role="button" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false" v-pre>
                                    <i class="fa fa-bell"></i>
                                    <span class="badge badge-light bg-success badge-xs">{{auth()->user()->unreadNotifications->count()}}</span>
                                </a>
                            <ul class="dropdown-menu">
                                        @if (auth()->user()->unreadNotifications)
                                        <li class="d-flex justify-content-end mx-1 my-2">
                                            <a href="{{route('mark-as-read')}}" class="btn btn-success btn-sm">Mark All as Read</a>
                                        </li>
                                        @endif
                                       
                                        @foreach (auth()->user()->unreadNotifications as $notification)
                                        <a href="#" class="text-success"><li class="p-1 text-success"> {{$notification->data['data']}}</li>  </a>
                                        @endforeach
                                        @foreach (auth()->user()->readNotifications as $notification)
                                        <a href="#" class="text-secondary"><li class="p-1 text-secondary"> {{$notification->data['data']}}</li>  </a>
                                        @endforeach
                            </ul>
                            </li>
```

This will result in a bell icon with a badge count of the unread notifications before the full name and logout dropdown on the dashboard. It also has a dropdown of all notifications with the unread at the top. The dropdown also has an option to mark all notifications as read so that the badge count returns to 0 (which means no more unread notifications).

[image]

Amazing job🤩! At this point, you can proceed to testing by making a deposit to view how your notifications display on the dashboard after its successful.
However lets go a bit further into including mail notifications. When a deposit is made, we want to display notifications in the dashboard and also send a mail to the user.

## How to Send Email Notifications

### Update `**.env**`  mail configurations

You need to update these environment variables  with valid mailer credentials that your app will need to send mails to users.

```php
    MAIL_MAILER=
    MAIL_HOST=
    MAIL_PORT=
    MAIL_USERNAME=
    MAIL_PASSWORD=
    MAIL_ENCRYPTION=
    MAIL_FROM_ADDRESS=
    MAIL_FROM_NAME="${APP_NAME}"
```

### Formatting Mail Notifications

In the `DepositSuccessful` notification class you created earlier, update the `via()` method to include mail.

```php
     public function via($notifiable)
        {
            return ['mail','database'];
        }
```

Afterwards, define the `toMail()` method with the mail messages. This method should return an `Illuminate/Notifications/Messages/MailMessage` instance after receiving a $notifiable entity.

You can now build email messages with the help of a few simple methods provided by the `MailMessage` class. A "call to action" and lines of text are both possible in mail messages.

```php
    public function toMail($notifiable)
        {
            $url = url('/home');
            return (new MailMessage)
                        ->greeting('Hello,')
                        ->line('Your deposit of '. $this->amount. ' was successful.')
                        ->action('View dashboard', url('/home'))
                        ->line('Thank you for using our application!');
        }
```

Small transactional emails can be formatted quickly and easily using these methods provided by the `MailMessage` object. A beautiful, responsive HTML email template with a plain-text counterpart will then be generated from the message components by the mail channel. In this example we supplied a call to action(button) , a line of text and a greeting.

**Hurrah 😎! We are done building 👍. Now, let’s perform some testing and see if it works.**

## Testing

If you make a successful deposit, you will get a notification in your dashboard that looks like this:

[Image]

Notice that the badge number counts only the unread notifications, however the dropdown lists all notifications starting out with the unread. If you decide to `mark all as read` then it refreshes and your notification badge count is back to 0 because you have read all the current notifications.

You will also recieve a mail notification that looks like this:

[Image]

Notice that the greeting, line of text and call to action(button) has been formatted to a pretty responsive template in mail.

## Conclusion

In this article,  you have gained an in-depth knowledge of Laravel Notifications. It is an entirely broad concept but this can be a great practical guide. Check out the [official Laravel documentation](https://laravel.com/docs/9.x/notifications) to learn more about [Laravel Notifications.](https://laravel.com/docs/9.x/notifications) It provides various flexible options, you can also customize these various channel to tailor your application needs or [create your own drivers to deliver notifications via other channels](https://laravel.com/docs/9.x/notifications#custom-channels). The code for this project is open-source and available [on GitHub](https://github.com/Roxie-32/laravel-notifications.git).
