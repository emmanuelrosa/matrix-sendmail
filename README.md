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
. The system admin must configure `matrix-commander` to generate the credentials.json file and the store directory. 
. The credentials.json file needs to be moved to `/etc/matrix-sendmail`
. The store directory needs to be moved to `/var/lib/matrix-commander`.. Set up a job to periodically run `matrix-sendmail-deliver`. This script will process the queued emails stored at `/var/spool/matrix-sendmail/$USER/queue`

