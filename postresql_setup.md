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

For some reason, Brew removes the local user, but you can add it back. 

```console
createdb michael
```

Use your system's local user name. 

Now add the other users. 

```console
psql --host=localhost --port=5432 --username=michael --password --dbname=postgres
```

Whe it asks for a password, user your system password. 

Create a User.

```console
CREATE USER "sampleuser" WITH PASSWORD 'Passwordz';
```
Make sure this new user can do admin things like create a DB.

```console
ALTER USER sampleuser CREATEDB;
```
Now log out.

```console
\q
```

Login with the new info so the new database will be owned by this user.

```console
psql --host=localhost --port=5432 --username=sampleuser --password --dbname=postgres
```

Use the password you just created for that user, mine was `Passwordz`.

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

After a few days and rebooting my Mac, I ran into an issue where the sandbox feature did not work. I had to create the DB first, then go into psql and add the user to it. 

```console
createdb test_development

psql test_development

CREATE USER "someuser" WITH PASSWORD 'myPassword';

GRANT ALL PRIVILEGES ON DATABASE test_development TO someuser;

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

I'm not sure if I ran into a bug or I did somehting wrong, but after the import I had to express the privileges again.

```console
psql test_development

GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO someuser;

\q
```
