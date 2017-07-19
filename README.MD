AutoMySQLBackup
===================
AutoMySQLBackup with a basic configuration will create Daily, Weekly and Monthly backups of one or more of your MySQL databases from one or more of your MySQL servers.

Other Features include:
 - Email notification of backups
 - Backup Compression and Encryption
 - Configurable backup rotation
 - Incremental database backups

Project originally hosted on **[Sourceforge](https://sourceforge.net/projects/automysqlbackup/)**.

----------


What is special about this fork?
-------------

AutoMySQLBackup uses the **[mysqldump](https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html)** tool that comes as part of MySQL to generate the backups. 

Historically, the script passed the MySQL username and password utilized for making the backups to mysqldump via command-line. As of MySQL 5.6, passing credentials via command-line is **considered insecure** and attempts to do so generates warning messages that disrupt normal processing of the AutoMySQLBackup script and sometimes breaks the script altogether, causing backups to fail.

**Example:**

    [root@server ~]# automysqlbackup
    Invoking backup method.
    Parsed config file "/etc/automysqlbackup/automysqlbackup.conf"
    ...
    ....
    ###### WARNING ######
    Errors reported during AutoMySQLBackup execution.. Backup failed
    Error log below..
    mysql: [Warning] Using a password on the command line interface can be insecure.
    mysqldump: [Warning] Using a password on the command line interface can be insecure.
    mysqldump: [Warning] Using a password on the command line interface can be insecure.

Several fixes have been mentioned [here](https://github.com/rcruz/AutoMySQLBackup/issues/1) and [here](http://www.redeo.nl/2013/11/automysqlbackup-warning-using-password-command-line-interface-can-insecure/). 

I tried assimilating them and yet the script failed with MySQL 5.7.x.

The tricks that finally saved the day was found in this [StackOverflow post](https://stackoverflow.com/questions/9293042/how-to-perform-a-mysqldump-without-a-password-prompt). It involves specifying the credentials in a file named `.my.cnf` under your (or root's) home folder and making the file readable to only yourself and your group (`chmod 0640`). 

    [mysql]
    user=mysqluser
    password=secret

This allows you to use the mysql client from the command-line without specifying credentials. I've always used this trick to facilitate quick login to MySQL. Little did I know that just by repeating the same in .my.cnf and renaming one of the block headers to mysqldump, you can achieve the same effect for mysqldump as well.

The `.my.cnf` now looks like:

    [mysql]
    user=mysqluser
    password=secret

    [mysqldump]
    user=mysqluser
    password=secret

<i class="icon-warning"></i>However, this takes care of only one part of the problem. The script still halts with the warning, as it is hard-coded to pass the credentials to mysqldump. That's where this fork comes in. All instances of credential arguments have been removed from the script.

Installation
-------------------
 1. Check-out / clone git repository.
 2. Run **install.sh**. This will replace any existing versions of `automysqlbackup` script in your `/usr/local/bin`.
 3. Do your configuration in `/etc/automysqlbackup/automysqlbackup.conf`.
 4. Run the script and/or add it to a cron job.