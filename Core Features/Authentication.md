> [!Warning]
> **Authentication** is a core feature of LainForo which is actively being worked on. Things are bound to change, so this might not be 100% accurate to the latest commit.

**Authentication** is an important part of LainForo. It helps the administrator easily maintain their installation of LainForo and moderate their site. It also allows user accounts (beta) to be logged into and, in the future, will operate as moderators for LainForo.

Many authentication packages out there can be very complicated and difficult to understand, which opens them up for tons of security risks if configured improperly.

LainForo comes with a built-in one, which I've yet to name. Either way, authentication is done in a very simple way which you can easily understand by reading this document.

> "The simpler the better, I guess."
> *-LostBox66*


## Admin authentication
Admin authentication is done in LainForo without any database requests. This keeps it more secure and out of the way of normal site operations.

>[!Note]
>The admin password is **not** hashed currently. In a production environment, the `.env` file will be locked away and will not be accessible by users of the site. I may rewrite it so it hashes the password, but this isn't on the map for now.

The admin password can be specified in the `.env` file within the root of LainForo's directory. The environment variable is called `LF_PASSWORD`. It is crucial that you make this secure, as people can login and delete threads/replies using it.

On the admin login page (`route('adminlogin'`), there is a form with a single field, the password. When/if co-admin accounts are added, this form will require a username too.

The form directs to `route('acplogin')`, which sends the fields from that form to a function in `AdminController` entitled `checkLogin().` It checks the password field against the `LF_PASSWORD` environment variable. If they both match, a login cookie is created for 24 hours. Then, upon visiting the login page, you're sent directly to the admin panel.

>[!Note]
>I am not worried about fake/forged cookies, because Laravel has some automatic checks in place to spot these. It treats any invalid or edited cookies as nonexistent.

If the password does **not** match the environment variable, it returns the error view (`view('error')`) with the `$error` variable set to `Incorrect login. Please try again`.

This is all that is involved in admin authentication. I try to keep it simple so people can easily understand how it works.

## User authentication
User authentication is handled differently, as it works with the database instead of environment variables.

>[!Note]
>User passwords are stored in the database as SHA-256 hashed strings, so in the unlikely event of a database leak, the passwords will remain safe long enough for people to change their passwords.

User accounts are stored in the `users` table within the database. The primary columns are `name`, the user's full name (this doesn't have any real use, just yet, but in the future I hope to make use of it), `username`, the username used to login as the user, and `password`. The `password` column is a SHA-256 hashed version of their password, which is a one-way mathematical function that transforms their password into a secure and irreversible string. [See more here](https://en.wikipedia.org/wiki/Cryptographic_hash_function)

The user will start by entering their username and password in the `view('form.userlogin')` form. This form sends its fields to `userAuth()` in the `UserController` controller.

This function will hash the password by running
```php
hash('sha256', $request->password)
```
which hashes the password from the form. This is used in an `if` statement that checks whether or not the supplied password matches the `password` column in the `users` table where the `username` is equal to `$request->password`.

If it's correct, it sets a cookie for 30 days where the value is the user's username. This is used to autofill the `author` field in a submission form, like `form.thread` or `form.reply`.

If it's incorrect, it returns a view called `error`.
```php
return view('error', ['error' => "Incorrect login. Please try again."]);
```
The `$error` variable is set to `Incorrect login. Please try again.` This variable is rendered out in the error view.