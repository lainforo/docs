In an anonymous environment like [[LainForo internals|LainForo]], it can be hard to verify that someone is who they say they are.

For example, you are the admin of the website. Let's say you want to provide an update to your users about an important backend change. You'll put in your username in the `author` field, create your thread, and that's it!

But, anybody can do that. Anybody can type `admin` as their name and impersonate you in seconds. Instead of requiring user accounts (which would de-anonymize and undermine one of the main philosophies of LainForo), we use **tripcodes**.

## How tripcodes work

Tripcodes are a way of authenticating a message using a hash. If you want to see a more visual explanation of tripcodes, [[Tripcodes.canvas|view the tripcode canvas here.]]

Tripcodes are a standard for image & messageboard software, dating back to 2chan and 4chan. I like to describe them as "anonymous passwords". 

In functions like `putThread()` or `putReply()`, the tripcode taken from the field is put through a hash function which generates a SHA-256 hash from the string. Then, it's stored in the database.

When the tripcode is rendered on the frontend, only the first 8 characters of the tripcode are shown. I find that it's enough to discourage people looking to brute-force the hash, while still being somewhat easy to remember for verifying users of LainForo.