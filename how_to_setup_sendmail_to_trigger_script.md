## How to setup sendmail on Linux to trigger script when an email sent to particular user.

You need sudo access to perform this operation. Make sure you have good backup before making any change.

First install sendmail package as usual and cd to /etc/mail directory.

Change /etc/mail/sendmail.mc configuration

### From

DAEMON_OPTIONS(`Port=smtp,Addr=127.0.0.1, Name=MTA')dnl

### to

DAEMON_OPTIONS(`Port=smtp, Name=MTA')dnl

$sudo cd /etc/mail

$sudo make

### Restart sendmail server. This will enable sendmail to send and receive mail from different systems.

$sudo systemctl restart sendmail

More setting needed to pipe mail to perl script.

### Fix sendmail alias for piping

Find location of aliases script in /etc/mail/sendmail.mc. It should look something like this

define(`ALIAS_FILE', `/etc/aliases')dnl

Find the user(say lab_foo) whose mail you want to pipe to script. Create an entry in /etc/aliases file, looks something like this.

$grep lab_foo /etc/aliases

lab_foo: :include:/etc/smrsh/foomail

### Configure critical steps

Pay attention to foomail file account and group ownership and it need to be on local disks as root can't access network files.

Your mail piping will run as owner of foomail file's owner/groups. work.pl file link need to be present in /etc/smrsh directory.

$ls -l /etc/smrsh/

total 4

-rwxr-xr-x 1 foo_acct foo_grp 45 Feb 22 07:52 foomail

lrwxrwxrwx 1 root     root    43 Feb 22 07:40 work.pl -> /some/network/path/work.pl


$ cat /etc/smrsh/foomail

|/some/network/path/work.pl

### Rebuild aliases databse.

$sudo  newaliases

/etc/aliases: 77 aliases, longest 26 bytes, 805 bytes total

### Restart sendmail, to take folliwng setting into effect.

$sudo systemctl restart sendmail

Now any mail sent to lab_foo will get piped to /some/network/path/work.pl script. 

Now you can do all kind of automation with work.pl.

Question/Comments

Leave a message about your experience.










