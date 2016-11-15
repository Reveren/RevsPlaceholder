### Once Installed

Once installed, you can't just work on Users willy nilly, you have to be inside a sandbox. Then you can create a DB, add a user, and then add permissions to that user to the DB. Like so:

Connect to a sandbox (the name can really be anything):

```console
psql somename
```

You should see a prompt like so:

```console
~ $ psql somename
psql (9.5.5)
Type "help" for help.

somename=#
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

No you can quit.

```console
\q
```
