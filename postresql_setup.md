### Installing

Use homebrew to install postgreSQL

```concole
brew install postgres
```

Uninstall the gem

```console
gem uninstall pg
```

Reinstall the gem

```console
gem install pg
```

### Create Users and Databases

Once installed, you can't just work on Users willy nilly, you have to be inside a sandbox. Then you can create a DB, add a user, and then add permissions to that user to the DB. Like so:

Connect to a sandbox (the name can really be anything):

```console
psql sandbox
```

You should see a prompt like so:

```console
psql (9.5.5)
Type "help" for help.

sandbox=#
```

Create a User.

```console
CREATE USER "tom" WITH PASSWORD 'myPassword';
```

Create the DB.

```console
CREATE DATABASE mydatabase;
```

Add privileges for the User to the DB.

```console
GRANT ALL PRIVILEGES ON DATABASE mydatabase TO tom;
```

Now you can quit.

```console
\q
```

After rebooting Mac, I ran into an ussue where the snadbox feature did not work. I had to create the DB first, then go into psql and add teh user to it. 

```console
createdb test_development

psql test_development

GRANT ALL PRIVILEGES ON DATABASE mydatabase TO tom;

\q
```

### Importing and Exporting DBs

I like to use intermediate files rather than piping and doing everything at once. That way I have a copy of the DB locally. 

To create a backup of a DB, run the following (it will ask for your remote password).

```console
pg_dump -h remotehost -U remoteuser dbname > backup.sql
```

This will create a local copy of the DB called backup.sql and place it wherever you performed this command.

Now to import this into your local DB, run the following command (it will ask for your local password).

```console
psql -h localhost -U localuser dbname < backup.sql
```
