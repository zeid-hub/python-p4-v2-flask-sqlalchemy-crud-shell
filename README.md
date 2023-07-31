# Flask-SQLAlchemy CRUD : Code-Along

## Learning Goals

- Use the Flask Shell to execute commands to insert, query, update, and delete
  rows in a database table.

---

## Key Vocab

- **Flask Shell**: A Python interactive shell to run commands in the context of
  a Flask application.
- **Database Session**: An object that manages persistence operations for
  ORM-mapped objects.
- **Database Transaction**: A sequence of SQL statements that are processed as
  an atomic unit. All SQL statements in the transaction are either applied (
  committed) or undone (rolled back) together.

---

## Introduction

Now that we've created a database with a `pets` table that is linked to a Flask
application, we'll step through the `Flask-SQLAlchemy` functions to insert,
update, delete, and query rows in the table.

## Setup

This lesson is a code-along, so fork and clone the repo.

Run `pipenv install` to install the dependencies and `pipenv shell` to enter
your virtual environment before running your code.

```console
$ pipenv install
$ pipenv shell
```

The `server` directory contains the same `app.py` and `models.py` from the
previous lesson.

```text
.
├── CONTRIBUTING.md
├── LICENSE.md
├── Pipfile
├── Pipfile.lock
├── README.md
└── server
    ├── __init__.py
    ├── app.py
    └── models.py
```

Change into the `server` directory and configure the `FLASK_APP` and
`FLASK_RUN_PORT` environment variables:

```console
$ cd server
$ export FLASK_APP=app.py
$ export FLASK_RUN_PORT=5555
```

Let's repeat through the process of creating the database `app.db` containing a
`pets` table. Enter the following commands:

```console
$ flask db init
```

```console
$ flask db migrate -m "Initial migration."
```

```console
$ flask db upgrade head
```

Open the database file `app.db` and confirm the `pets` table:

![new pet table](https://curriculum-content.s3.amazonaws.com/7159/python-p4-v2-flask-sqlalchemy/pet_table.png)

NOTE: At any time during the lesson, you can delete the `instance` and
`migrations` folders and re-execute the three `flask db` commands (init,
migrate, upgrade) to recreate the initial version of the database.

---

## The Flask Shell - Insert

Let's see how to insert rows into the `pets` table.

We can interact with our code in the Python shell or an `ipdb` session, but
working with a web framework presents a bit of a conundrum: the application
isn't running! Flask comes equipped with an interactive shell that runs a
development version of an application. Inside this shell, we can interact with
models, views, contexts, and the database as if the application were really
running online.

If you're not there already, navigate to the `server` directory, then enter the
command `flask shell`:

```console
$ flask shell
>>>
```

You will type commands after the `>>>` prompt.

First, let's import the necessary `db` database object and the `Pet` model:

```console
>>> from models import db, Pet
```

Let's add a row to the `pets` table for a dog named "Fido". The steps to add a
table row are:

1. Create a new instance of the model class `Pet`.
2. Add the `Pet` instance to the current database session.
3. Commit the transaction and apply the changes to the database.

The first step is creating the `Pet` instance. Type the following Python
assignment statement:

```console
>>> pet1 = Pet(name = "Fido", species = "Dog")
```

An instance of `Pet` was created, however, the object has not been persisted to
the database.

Let's confirm the `name` and `species` attributes have been assigned values, but
`id` does not yet have a value:

```console
>>> pet1.name
'Fido'
>>> pet1.species
'Dog'
>>> pet1.id
>>>
```

The string representation returned by the implicit call to the `__repr__`
function shows the id as `None`, confirming no value has been assigned:

```console
>>> pet1
<Pet None, Fido, Dog>
```

The `id` won't be assigned until the pet has been added to the database.

This requires a database session, which is an object that manages database
transactions. A transaction is a sequence of SQL statements that are processed
as an atomic unit. This means that either all SQL statements in the transaction
are either applied ( committed) or they are all undone (rolled back) together.

This is especially important if statements that occur in a sequence on depend on
previous statements executing properly. The workflow for a transaction is
illustrated in the image below:

![Workflow for a successful transaction. Shows that after a transaction begins,
the state of the database is recorded, then statements are executed, then the
transaction is committed if all statements are successful.](https://curriculum-content.s3.amazonaws.com/python/esal_0401.png "successful transaction")

If any of the SQL statements in a transaction fail to execute properly, the
database will be rolled back to the state recorded at the beginning of the
transaction and the process will end, returning an error message. A committed
transaction ensures all statements were executed in sequence and to completion.

Flask-SQLAlchemy provides the `db.session` object through which we can manage
database changes.

Let's add the pet object to the database session using the `db.session.add()`
method:

```console
>>> db.session.add(pet1)
```

This method call will issue an SQL INSERT statement, but the pet's `id`
attribute is still undefined because we have not yet committed the current
transaction. Use the `db.session.commit()` method to commit the transaction and
insert the new row in the database table.

```console
>>> db.session.commit()
```

Check the `pets` table to confirm a new row was added. If you are using SQLite
Viewer, you may need to press the refresh button to see the new row:

![first row inserted in pet](https://curriculum-content.s3.amazonaws.com/7159/python-p4-v2-flask-sqlalchemy/insert_dog1.png)

When the transaction is committed and the row is inserted, the `id` of the local
`Pet` instance is assigned the primary key value :

```console
>>> pet1.id
1
>>>
```

Let's add another pet to the database. Type each Python statement one at a time
at the Flask shell prompt:

```console
>>> pet2 = Pet(name = "Whiskers", species = "Cat")
>>> db.session.add(pet2)
>>> db.session.commit()
```

Confirm a new row was inserted in the `pets` table for the cat named "Whiskers":

![second row inserted in pet table](https://curriculum-content.s3.amazonaws.com/7159/python-p4-v2-flask-sqlalchemy/insert_cat.png)

Confirm the `id` attribute is assigned for the newly persisted object:

```console
>>> pet2.name
'Whiskers'
>>> pet2.species
'Cat'
>>> pet2.id
2
>>> pet2
<Pet 2, Whiskers, Cat>
```

## The Flask Shell - Query

We can query all the rows in the table associated with the `Pet` model as shown:

```console
>>> Pet.query.all()
[<Pet 1, Fido, Dog>, <Pet 2, Whiskers, Cat>]
```

How did the `Pet` class get a `query` attribute? `Pet` inherits it from
`db.Model`!

If we just want just the first row returned from a query, use the `first()`
function:

```console
>>> Pet.query.first()
<Pet 1, Fido, Dog>
```

We can filter the rows using the `filter_by` function. For example, to get all
cats:

```console
>>> Pet.query.filter_by(species = 'Cat').all()
[<Pet 2, Whiskers, Cat>]
```

We can filter by the primary key `id`:

```console
>>> Pet.query.filter_by(id=1).first()
<Pet 1, Fido, Dog>
```

We can also use the shorter `get()` method to get the row for the given primary
key value:

```console
>>> Pet.query.get(1)
<Pet 1, Fido, Dog>
>>>
```

## The Flask Shell - Update

If we assign a new attribute value to a Python object that has been persisted to
the database, the associated table row **does not** automatically get updated.

We need to perform the following steps to update a row in the `pets` table:

1. Update the attribute value of a `Pet` instance.
2. Add the `Pet` instance to the current database session.
3. Commit the transaction and apply the changes to the database.

```console
>>> pet1
<Pet 1, Fido, Dog>
>>> pet1.name = "Fido the mighty"   # this does not update the table row
>>> pet1
<Pet 1, Fido the mighty, Dog>
>>> db.session.add(pet1)            # issue an SQL UPDATE statement
>>> db.session.commit()             # commit the UPDATE statement
```

![update row in pet table](https://curriculum-content.s3.amazonaws.com/7159/python-p4-v2-flask-sqlalchemy/update_pet.png)

## The Flask Shell - Delete

The `db.session.delete()` function is used to delete the row associated with an
object:

```console
>>> db.session.delete(pet1)
>>> db.session.commit()
```

Query the `Pet` model to confirm the row was deleted:

```console
>>> Pet.query.all()
[<Pet 2, Whiskers, Cat>]
```

We can also check the table using the SQLite Viewer:

![delete row from pet table](https://curriculum-content.s3.amazonaws.com/7159/python-p4-v2-flask-sqlalchemy/delete_pet.png)

If you want to delete all table rows, call the function `Pet.query.delete()`.
The function returns the number of rows deleted. Make sure you commit the
transaction.

```console
>>> Pet.query.delete()
1
>>> db.session.commit()
```

Confirm there are no pets in the table:

```console
>>> Pet.query.all()
[]
```

![new pet table](https://curriculum-content.s3.amazonaws.com/7159/python-p4-v2-flask-sqlalchemy/pet_table.png)

## Flask shell - exit()

You can exit the shell and return to the command line prompt using the `exit()`
function:

```console
>>> exit()

$
```

## Conclusion

As with any shell, `Flask shell` is a great tool for debugging and adding or
updating a few records. We want our app to handle many records though, which
would take too long to do by hand in the Flask shell. In the following lessons,
we'll see how to add routes to a Flask app to support full CRUD operations.

---

## Resources

- [Flask Shell](https://flask.palletsprojects.com/en/2.3.x/shell/)
- [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/3.0.x/)
- [Flask-SQLAlchemy Session](https://flask-sqlalchemy.palletsprojects.com/en/3.0.x/api/#module-flask_sqlalchemy.session)
- [SQLAlchemy Transaction](https://docs.sqlalchemy.org/en/20/orm/session_transaction.html)
