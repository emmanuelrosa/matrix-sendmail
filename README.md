# matrix-sendmail
A simple sendmail implementation which uses a Matrix CLI client to send "mail" to a Matrix room.

The idea here is to provide a simplistic implementation of `sendmail` which uses [Matrix](https://matrix.org/) to send the emails to a pre-configured room. This avoids the need to have an SMTP server when all you want to do is receive notifications from your Linux PC.

## Use cases
This can be used to:

* Send system alerts to your Matrix client running on your mobile phone.
* Have a long-running script notify you when if has completed.
* Etc.

Basically, you get the benefits of using [matrix-commander](https://github.com/8go/matrix-commander/), but with a `sendmail` interface.

## How does it work?
The `sendmail` script doesn't actually send anything. Instead, it adds the email to a user-specific queue. A scheduled job then processes the queue and delivers the emails. The queue exists to allow any user account to call `sendmail` without having to give them permissions to access the `matrix-commander` data files. It also has the nice benefit of handling disconnections to the Internet.

Note that `matrix-sendmail` completely ignores the provided email addresses; It only delivers the emails to the pre-configured Matrix room, regardless of which user executed `sendmail`.

## Setup procedure

1. Read the code and familiarize yourself with it. It's purposely really simple, but you need to be able to grok it so that you can set it up correctly. Different scripts need different levels of filesystem access.
2. Configure `matrix-commander` to generate the credentials.json file and the store directory. 
3. The credentials.json file needs to be moved to $MSM_LIB_DIR (ex. `/var/lib/matrix-sendmail`).
4. The store directory needs to be moved to $MSM_LIB_DIR/store (ex. `/var/lib/matris-sendmail/store`).
5. Copy `config.env` to `/etc/matrix-commander`.
6. Set up a job to periodically run `matrix-sendmail-prep` as the root user. This script will move the queued emails stored at `/var/spool/matrix-sendmail/user/$USER/new` to `/var/spool/matrix-sendmail/system/new`.
7. Set up a job to periodically run `matrix-sendmail-deliver` as a non-root user. This script will send the emails stored at `/var/spool/matrix-sendmail/system/new` using `matrix-commander`. 
8. Copy `sendmail` to a directory in $PATH, such as `/usr/bin`
9. Copy `matrix-sendmail-prep` and `matrix-sendmail-deliver` to `/usr/lib/matrix-sendmail/libexec`. These scripts are not meant to be in $PATH.

### Permissions

* The user used to execute `matrix-sendmail-deliver` must have read/write access to the `/var/lib/matrix-sendmail`
* The user used to execute `matrix-sendmail-deliver` must have read access to `/var/spool/matrix-sendmail/system` and `/etc/matrix-commander`.
* All users who need to run `sendmail` need read access to `/etc/matrix-commander` and read/write access to `/var/spool/user`.
* The only script which needs to run as root (or some other user with elevated access) is `matrix-sendmail-prep`. That's why the script is short and sweet. For security reasons the script purposely does not source `/etc/matrix-sendmail`, so you need to provide the required environment variables before running the script.
