
## You

how to send Laravel 8 multiple Bulk Mail where user select multi user and also send subject and messages
## Captainsj😑

To send multiple bulk emails in Laravel 8, you can follow these steps:

1. Create a form that allows the user to select multiple recipients and enter the message.
2. In your `routes/web.php` file, create a route that will handle the form submission:

```php
Route::post('/send-emails', 'MailController@sendEmails')->name('send-emails');
```

3. In your `app/Http/Controllers/MailController.php` file, create a method that will handle the form submission and send the emails:

```php

    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $data['users'] = User::where('role', 'sales')->select('users.id', 'users.email')->get();
        return view('admin.pages.bulkemail.all', $data);
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $validator = Validator::make(
            $request->all(),
            [
                'recipients.*' => 'required|email',
                'subject'      => 'required|string',
                'message'      => 'required|string',
            ]

        );

        if ($validator->passes()) {
            // Get the selected recipients and other form data
            $recipients = $request->recipients;
            $subject = $request->subject;
            $message = $request->message;

            // Loop through each recipient and send them an email
            foreach ($recipients as $email) {
                Mail::to($email)->send(new CustomMail($subject, $message));
            }
            Toastr::success('Mail Send Successfully');
        } else {
            $messages = $validator->messages();
            foreach ($messages->all() as $message) {
                Toastr::error($message, 'Failed', ['timeOut' => 30000]);
            }
        }
        return redirect()->back();
    }
```

4. Create a new mail class `CustomMail` by running the following command in the terminal:

```bash
php artisan make:mail CustomMail --markdown=emails.custom
```

5. In the `CustomMail` class (`app/Mail/CustomMail.php`), add the subject and message as properties and use the `build()` method to construct the email:

```php
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class CustomMail extends Mailable
{
    use Queueable, SerializesModels;

    /* Setting the subject and message to be used in the email. */
    public $subject;
    public $message;

    /**
     * Create a new message instance.
     *
     * @param string $subject The subject of the message.
     * @param string $message The content of the message.
     */
    public function __construct(string $subject, string $message)
    {
        $this->subject = $subject;
        $this->message = $message;
    }

    public function build()
    {
        return $this->markdown('emails.custom')
            ->subject($this->subject)
            ->with('message', $this->message);
    }

    /**
     * Get the message envelope.
     *
     * @return \Illuminate\Mail\Mailables\Envelope
     */
    public function envelope()
    {
        return new Envelope(
            subject: $this->subject,
        );
    }

    /**
     * Get the message content definition.
     *
     * @return \Illuminate\Mail\Mailables\Content
     */
    public function content()
    {

        return (new Content)
            ->markdown('emails.custom')
            ->with('message', $this->message);
    }

    /**
     * Get the attachments for the message.
     *
     * @return array
     */
    public function attachments()
    {
        return [];
    }
}
```

6. Create a new email template at `resources/views/emails/custom.blade.php` with the following code:

```html
@component('mail::message')
{{ $message }}
@endcomponent
```

7. Finally, create the HTML form that allows users to select multiple recipients and enter the subject and message. Something like this:

```html
<div class="card-body">
    <form method="post" action="{{ route('bulkEmail.store') }}">
        @csrf
        <div class="row mb-3">
            <div class="col-sm-4">
                <h6 class="mb-0">User Mail</h6>
            </div>
            <div class="form-group text-secondary col-sm-8">
                <select name="recipients[]" class="form-control multiselect"
                    multiple="multiple" data-include-select-all-option="true"
                    data-enable-filtering="true"
                    data-enable-case-insensitive-filtering="true">
                    @foreach ($users as $user)
                        <option value="{{ $user->email }}">{{ $user->email }} </option>
                    @endforeach
                </select>
            </div>
        </div>

        <div class="row mb-3">
            <div class="col-sm-4">
                <h6 class="mb-0">Subject </h6>
            </div>
            <div class="form-group col-sm-8 text-secondary">
                <input type="text" name="subject" class="form-control maxlength"
                    maxlength="100" />
            </div>
        </div>
        <div class="row mb-3">
            <div class="col-sm-4">
                <h6 class="mb-0">Message </h6>
            </div>
            <div class="form-group col-sm-8 text-secondary">
                <input type="text" name="message" class="form-control maxlength"
                    maxlength="255" />
            </div>
        </div>

        <div class="row">
            <div class="col-sm-4"></div>
            <div class="col-sm-8 text-secondary">
                <input type="submit" class="btn btn-primary px-4 mt-5" value="Submit" />
            </div>
        </div>
    </form>
</div>

```
