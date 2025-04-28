Cmbackup - Backup Script for Carbonio CE OSE
=========

Mirror of: https://www.anahuac.eu/from-zmbackup-to-cmbackup/

***Please report any problems you experience.***

For many years Luca’s zmbackup saved our lives and souls with his Free Software zmbackup. So, let me start congratulating and thanking him deeply. Zmbackup have all needed functions to backup and restore Zimbra’s fully and incremental mailboxes.

Once Carbonio CE became the right Zimbra OSE alternative we needed a tool to do the same job in the same way on it. So I took the easiest path: rename zmbackup to cmbackup.

Fixing command paths in the code and adding a minor change to prevent timeout in restore and that was it. I hope you enjoy it.

You can download cmbackup:
```
git clone https://github.com/server-tools/carbonio-backup.git
cd carbonio-backup
chmod +x install.sh
./install.sh
```

Below you can see the same how to use from Luca’s github site

Usage
=========

To check all the options available to Cmbackup, just run cmbackup -h or cmbackup –help. This will return for you a list with all the options, what each one of them does, and the syntax.
```
$ cmbackup -h
usage: cmbackup -f [-m,-dl,-al,-ldp, -sig] [-d,-a] <mail/domain>
       cmbackup -i <mail>
       cmbackup -r [-m,-dl,-al,-ldp, -sig] [-d,-a] <session> <mail>
       cmbackup -r [-ro] <session> <mail_origin> <mail_destination>
       cmbackup -d <session>
       cmbackup -m

Options:

 -f,  --full                      : Execute full backup of an account, a list of accounts, or all accounts.
 -i,  --incremental               : Execute incremental backup for an account, a list of accounts, or all accounts.
 -l,  --list                      : List all backup sessions that still exist in your disk.
 -r,  --restore                   : Restore the backup inside the users account.
 -d,  --delete                    : Delete a session of backup.
 -hp, --housekeep                 : Execute the Housekeep to remove old sessions - Zmbhousekeep
 -m,  --migrate                   : Migrate the database from TXT to SQLITE3 and vice versa.
 -v,  --version                   : Show the cmbackup version.
 -h,  --help                      : Show this help

Full Backup Options:

 -m,   --mail                     : Execute a backup of an account, but only the mailbox.
 -dl,  --distributionlist         : Execute a backup of a distributionlist instead of an account.
 -al,  --alias                    : Execute a backup of an alias instead of an account.
 -ldp, --ldap                     : Execute a backup of an account, but only the ldap entry.
 -sig, --signature                : Execute a backup of a signature.
 -d,   --domain                   : Execute a backup of only a set of domains, comma separated
 -a,   --account                  : Execute a backup of only a set of accounts, comma separated

Restore Backup Options:

 -m,   --mail                     : Execute a restore of an account,  but only the mailbox.
 -dl,  --distributionlist         : Execute a restore of a distributionlist instead of an account.
 -al,  --alias                    : Execute a restore of an alias instead of an account.
 -ldp, --ldap                     : Execute a restore of an account, but only the ldap entry.
 -ro,  --restoreOnAccount         : Execute a restore of an account inside another account.
 -sig, --signature                : Execute a restore of a signature.
 -d,   --domain                   : Execute a backup of only a set of domains, comma separated
 -a,   --account                  : Execute a backup of only a set of accounts, comma separated
```

To run a full backup routine, which includes by default the mailbox and the ldiff, just run the script with the option -f or –full. Dependent of the amount of accounts or the number of processes you set in the option MAX_PARALLEL_PROCESS, this will take sometime before concludes.
```
$ cmbackup -f
```
You can filter for what you want using the options -m for Mailbox, -ldp for Accounts, -al for Alias, and -dl for Distribution List. REMEMBER – This options doesn’ stack with each other, so don’ try -dl and -al at the same time (The script will only broke if you do this).

CORRECT

```
$ cmbackup -f -m
```

INCORRECT

```
$ cmbackup -f -m -ldp
```

Aside from the full backup action, Cmbackup still have an option to do incremental backups. This works like this: before an incremental be executed, Cmbackup should check the date for the latest route for each account, and execute a restore action based on that date. At the moment, the incremental will backup the ldap account and the mailbox, and accept no parameter aside the list of accounts to be backed up.

```
$ cmbackup -i
```

To restore a backup, you use the option -r or –restore, but this time you should inform the ID session you want to restore. You can check the sessionID with the command cmbackup - l.

```
$ cmbackup -r -ro full-20170621201603 slayerofdemons@boletaria.com chosenundead@lordran.com
```

To remove the backup session, you only need to use the option -d or –delete, and inform the session you want to delete. Or, if you want to remove all the backups before X days, you can use the option -hp or –housekeep to run the Housekeep process. WARNING: The housekeep can take sometime depending on the amount of data you want to remove.

```
$ cmbackup -d full-20170621201603
$ cmbackup -hp
```

Cmbackup is capable to migrate from TXT to SQLite3, if you want to store you data inside a relative database. The advantage of doing this is more efficiency when trying to list the sessions, and more details when you do this (like the beginning and conclusion of the session). To enable the SQLite3, first edit the option SESSION_TYPE inside cmbackup.conf:

```
vim /etc/cmbackup/cmbackup.conf
...
SESSION_TYPE=SQLITE3
```

With the SQLITE3 option enabled, now you need to migrate your entire sessions.txt to the relative database using the option -m or –migrate. After the end of the migration, you can run all cmbackup commands again.

REMEMBER: at this moment, this migration activity is a only one way road. There is no rollback, and, if you try to do a rollback, you will lose your sessions file.

Scheduling backups
=========

The installer script automatically creates a cron config file in /etc/cron.d/cmbackup. You can customize backup routes editing that file.

