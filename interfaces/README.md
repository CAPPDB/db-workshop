# DB Interfaces Tutorial
This tutorial will guide you through creating a web server that interacts with
a database.

## Part 1: The Flask Web Server

This tutorial will guide you through creating your first web server using Flask.

First, open a terminal and make sure you are in the correct directory:

```bash
cd interfaces
pwd
```

The output should be something like `/path/to/your-repo/interfaces`.

Next, create a python virtual environment:

```bash 
python3 -m venv .venv
```

This command uses the python module `venv` to create a virtual environment in 
the `.venv` directory. This is where all the dependencies for this tutorial 
will be installed. This also ensures that the dependencies for this tutorial
do not interfere with other projects/classes you may be working on.

Next, activate the virtual environment:

```bash
source .venv/bin/activate
```

You should see the name of the virtual environment in your terminal prompt.

Next, install the dependencies for this tutorial:

```bash
pip install -r requirements.txt
```

This command uses the `pip` package manager to install the dependencies listed
in the `requirements.txt` file. This file lists all the dependencies needed for
this tutorial.

Now, create a new directory for the hello_world web server:

```bash 
mkdir hello_world
cd hello_world
```

Next, create a new python file called `app.py`:

```bash
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
   return "<p>Hello, World!</p>"
```

This code creates a new Flask web server that listens for requests on the root
URL `/` and returns the string `Hello, World!`.

Next, run the web server:

```bash
export FLASK_APP=app.py
flask run
```

This command sets the `FLASK_APP` environment variable to the name of the python
file that contains the Flask app. Then, it runs the Flask app. You should
see output that looks like this:

```
 * Serving Flask app "app.py"
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://localhost:5000/ (Press CTRL+C to quit)
```

Open a web browser and navigate to `http://localhost:5000/`. You should see
the text `Hello, World!` displayed in the browser.

To stop the web server, press `CTRL+C` in the terminal. Congratulations! You
have created your first web server using Flask.

### Using Templates

Flask allows you to use templates to generate HTML content. This is useful
when you want to generate HTML content dynamically based on user input or
other factors.

First, within the `hello_world` directory, create a new directory for the templates:

```bash
mkdir templates
```

Now create a file called `index.html` within the `templates` directory and 
add the following content:

```html
<!doctype html>
<html>
   <head>
       <title>{{ title }}</title>
   </head>
   <body>
       <h1>Hello, {{ user.username }}!</h1>
   </body>
</html>
```

This is an HTML template that uses Jinja2 syntax to insert variables into the
HTML content. The `{{ title }}` and `{{ user.username }}` are placeholders that
will be replaced with actual values when the template is rendered.

Next, update the `app.py` file to use this template, inserting your name into the `user` python dictionary:

```python
from flask import Flask, render_template


app = Flask(__name__)


@app.route("/")
def hello_world():
   user = {'username': '<Insert Your name Here>'}
   return render_template('index.html', title='Home', user=user)
```

Now when you run the web server and navigate to `http://localhost:5000/`, you
see a personalized greeting with your name.

### Creating a Simple Form
Let's try making our web server a little more interactive by adding a simple form.
Modify the `index.html` file to include a form:

```html
<!doctype html>
<html>
    <head>
        <title>{{ title }}</title>
    </head>
    <body>
        <form action="/hello" method="get">
            <input type="text" name="name">
            <input type="submit" value="Submit">
        </form>
        <h1>Hello, {{ user.username }}!</h1>
    </body>
</html>
```

This form has a single text input field and a submit button. When the form is
submitted, it sends a `GET` request to the `/hello` URL of our web server.

Next, update the `app.py` file to handle the form submission:

```python
from flask import Flask, render_template, request

app = Flask(__name__)

@app.route('/hello', methods=['GET'])
def hello_input():
    name = request.args.get('name')
    user = {'username': name}
    return render_template('index.html', title="Home", user=user)

@app.route("/")
def hello_world():
    user = {'username': 'Suhail'}
    return render_template('index.html', title='Home', user=user)
```

This code adds a new route `/hello` that handles `GET` requests. It retrieves
the value of the `name` parameter from the request and uses it to personalize
the greeting.

Now when you run the web server and navigate to `http://localhost:5000/`, you
should see a form where you can enter your name. After submitting the form,
you should see a personalized greeting with your name.


## Part 2: SQLite Database
In this part of the tutorial, we will create a simple web server that interacts
with a database. We will use SQLite as our database.

### Loading the Data into a SQLite Database

First, let's switch to the `school_app` directory in the terminal:

```bash
cd ..
cd school_app
ls
```

You should see a CSV file called `schools.csv` in the directory. This file
contains information about the Chicago Public Schools, downloaded from
the Chicago Data Portal. You can view the contents of this file in VSCode
or any text editor. Note that the first row contains the column headers.

Let's use python and the `pandas` library to read this CSV file and create
a SQLite database from it. First, create a new python file called `create_db.py`
and add the following code:

```python
import pandas as pd
import sqlite3

input_file = 'schools.csv'


def load_data():
    conn = sqlite3.connect('schools.db')
    df = pd.read_csv(input_file)
    df.to_sql(name='schools', con=conn, if_exists='replace', index=False)
    conn.close()
    return df


if __name__ == '__main__':
    load_data()
```

This code reads the `schools.csv` file into a pandas DataFrame and then
writes the data to a SQLite database called `schools.db`. The `if_exists='replace'`
parameter ensures that the table is replaced if it already exists.

Run this script in the terminal:

```bash
python create_db.py
```

You should see no output, but a new file called `schools.db` should be created
in the `school_app` directory.

Now, lets explore this database using the `sqlite3` command line tool. Run the
following command in the terminal:

```bash
sqlite3 schools.db
```

This command opens the SQLite command line tool and connects to the `schools.db`
database. You should see a prompt that looks like this:

```
SQLite version 3.34.0 2020-12-01 16:14:00
Enter ".help" for usage hints.
sqlite>
```

Use the `.tables` command to see the tables in the database:

```sql
.tables
```


Now, let's run a simple query to see the contents of the `schools` table:

```sql
SELECT * FROM schools LIMIT 5;
```

This query selects all columns from the `schools` table and limits the output
to the first 5 rows. You should see a table of data displayed in the terminal.

Try running some other queries to explore the data. When you are done, type
`.exit` to exit the SQLite command line tool.

# Part 3: Connecting Flask to SQLite
Now that we have a SQLite database with the Chicago Public Schools data, let's create a Flask web server that interacts with this database.

First, make sure you are in the `school_app` directory in the terminal.
Next, lets create a template to display the school data. Create a new file
called `index.html` in a new `templates` directory and add the following content:

```html
<!DOCTYPE html>
<html>
<head>
    <title>CPS School Information</title>
</head>
<body>
    <h1>CPS School Information</h1>

    <a href="/">Homepage</a>

    <div id="school-info">
        <table class="table">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Address</th>
                    <th>Grade Category</th>
                    <th>Grades</th>
                    <th>School Type</th>
                </tr>
            </thead>
            <tbody>
                {% for school in schools %}
                <tr>
                    <td>{{ school.SCHOOL_ID }}</td>
                    <td>{{ school.SCHOOL_NM }}</td>
                    <td>{{ school.SCH_ADDR }}</td>
                    <td>{{ school.GRADE_CAT }}</td>
                    <td>{{ school.GRADES }}</td>
                    <td>{{ school.SCH_TYPE }}</td>
                </tr>
                {% endfor %}
            </tbody>
        </table>
    </div>
</body>
</html>
```

This template uses Jinja2 syntax to loop through a list of schools and display
the school information in an HTML table.

Next, create the `app.py` file to load the school data from the SQLite database
and render the `index.html` template:

```python
from flask import Flask, jsonify, request, render_template
import sqlite3

app = Flask(__name__)

@app.route('/')
def get_all_schools():
    conn = sqlite3.connect('schools.db')
    cursor = conn.cursor()
    cursor.row_factory = sqlite3.Row
    cursor.execute('SELECT * FROM schools')
    schools = [dict(row) for row in cursor.fetchall()]
    return render_template('index.html', schools=schools)
```

This code connects to the existing `schools.db` SQLite database, 
executes a query to select all rows from the `schools` table, and then
renders the `index.html` template with the list of schools.

Congrats! You have created a web server that interacts with a SQLite database.
This is a simple example, but it demonstrates the basic concepts of creating
a web server that interacts with a database.

To run the web server, use the following command:

```bash
export FLASK_APP=app.py
flask run
```

## Exercise: Add a Search Form

Time for a programming exercise. Let's combine our knowledge of SQLite with Flask form handling, and add a search form to our web server that allows users to search for schools by name. 

To do this, you will have to:
- Update the `index.html` template to include a search form.
- Update the `app.py` file to handle the form submission and filter the schools by name.

Reach out to the instructor if you need help or get stuck. Good luck!

## Wrap-Up
In this tutorial, you learned how to create a web server using Flask and interact
with a SQLite database. You created a simple web server that displays information
about Chicago Public Schools and allows users to search for schools by name.

This is just the very beginning of what you can do with Flask and databases.
You can expand on this project by adding more features, such as filtering by
different criteria, displaying additional information about each school, or
even allowing users to add new schools to the database. In addition, using 
Javascript and CSS, you can enhance the user interface and create a more
interactive experience, and even include third-party libraries to create
more advanced visualizations, such as a map showing the location of each school.

I hope you found this tutorial helpful and that it inspires you to explore
more about web development and databases. Flask is a powerful tool that can
be used to create a wide range of web applications, from simple prototypes
to complex, data-driven websites. Have fun exploring and building with Flask!

If you'd like to learn more about Flask development, check out the [Flask Mega
Tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world) by Miguel Grinberg.
