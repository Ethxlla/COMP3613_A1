[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/Ethxlla/COMP3613_A1)
<a href="https://render.com/deploy?repo=https://github.com/uwidcit/flaskmvc">
  <img src="https://render.com/images/deploy-to-render-button.svg" alt="Deploy to Render">
</a>

![Tests](https://github.com/uwidcit/flaskmvc/actions/workflows/dev.yml/badge.svg)

# Flask MVC Template
A template for flask applications structured in the Model View Controller pattern [Demo](https://dcit-flaskmvc.herokuapp.com/). [Postman Collection](https://documenter.getpostman.com/view/583570/2s83zcTnEJ)


# Dependencies
* Python3/pip3
* Packages listed in requirements.txt

# Installing Dependencies
```bash
$ pip install -r requirements.txt
```

# Configuration Management


Configuration information such as the database url/port, credentials, API keys etc are to be supplied to the application. However, it is bad practice to stage production information in publicly visible repositories.
Instead, all config is provided by a config file or via [environment variables](https://linuxize.com/post/how-to-set-and-list-environment-variables-in-linux/).

## In Development

When running the project in a development environment (such as gitpod) the app is configured via default_config.py file in the App folder. By default, the config for development uses a sqlite database.

default_config.py
```python
SQLALCHEMY_DATABASE_URI = "sqlite:///temp-database.db"
SECRET_KEY = "secret key"
JWT_ACCESS_TOKEN_EXPIRES = 7
ENV = "DEVELOPMENT"
```

These values would be imported and added to the app in load_config() function in config.py

config.py
```python
# must be updated to inlude addtional secrets/ api keys & use a gitignored custom-config file instad
def load_config():
    config = {'ENV': os.environ.get('ENV', 'DEVELOPMENT')}
    delta = 7
    if config['ENV'] == "DEVELOPMENT":
        from .default_config import JWT_ACCESS_TOKEN_EXPIRES, SQLALCHEMY_DATABASE_URI, SECRET_KEY
        config['SQLALCHEMY_DATABASE_URI'] = SQLALCHEMY_DATABASE_URI
        config['SECRET_KEY'] = SECRET_KEY
        delta = JWT_ACCESS_TOKEN_EXPIRES
...
```

## In Production

When deploying your application to production/staging you must pass
in configuration information via environment tab of your render project's dashboard.

![perms](./images/fig1.png)

# Flask Commands

wsgi.py is a utility script for performing various tasks related to the project. You can use it to import and test any code in the project. 
You just need create a manager command function, for example:

```python
#1. Create a competition
@app.cli.command("create-competition", help="Creates a new coding competition")
@click.argument("competition_name")
@click.argument("date_occurred")  
@click.argument("competition_description", required=False)
def create_competition_command(competition_name, date_occurred, competition_description=None):
    date_obj = datetime.strptime(date_occurred, '%Y-%m-%d')
    competition = create_competition(competition_name, date_obj, competition_description)
    print(f'Competition {competition.competition_Name} created!')

#2. Import competition results from file
@app.cli.command("import-results", help="Import competition results from a file")
@click.argument("competition_id")
@click.argument("file_path")
def import_competition_results_command(competition_id, file_path):
    success, imported_results = import_competition_results(competition_id, file_path)
    
    if success:
        print("Import successful, here are the imported results:")
        for result in imported_results:
            print(result.get_json())  
    else:
        print("Import failed!")

#3. List competitions
@app.cli.command("list-competitions", help="Lists all competitions")
def list_competitions_command():
    competitions = get_all_competitions()
    for competition in competitions:
        print(competition.get_json())

#4. View competition results
@app.cli.command("view-results", help="Displays result of a specific student in a competition")
@click.argument("competition_id")
@click.argument("student_name")
def view_results_command(competition_id, student_name):
    results = get_competition_results(competition_id)
    if results:
        student_result = next((result for result in results if result.student.lower() == student_name.lower()), None)
        if student_result:
            print(student_result.get_json())
        else:
            print(f'No result found for student {student_name} in competition {competition_id}')
    else:
        print(f'No results found for competition {competition_id}')

```

Then execute the command invoking with flask cli with command name and the relevant parameters

```bash
$ flask user create bob bobpass
```


# CLI Commands - Running the Project

_For development run the serve command (what you execute):_
```bash
$ flask run
$ flask create-competition "DCIT Runtime Competition" 2024-10-01 "Sprint 1: Annual Coding Event"
$ flask import-results 1 /workspace/COMP3613_A1/results.csv
$ flask list-competitions
$ flask view-results 1 'Judy Margot'


Syntax:
 $ flask init
 $ flask create-competition "Competition Name" YYYY-MM-DD "Competition Description"
 $ flask import-results  <competition_id>  <filepath>
 $ flask list-competitions
 $ flask view-results <competition_id> 'Student Name (FirstName)'
```

_For production using gunicorn (what the production server executes):_
```bash
$ gunicorn wsgi:app
```

# Deploying
You can deploy your version of this app to render by clicking on the "Deploy to Render" link above.

# Initializing the Database
When connecting the project to a fresh empty database ensure the appropriate configuration is set then file then run the following command. This must also be executed once when running the app on heroku by opening the heroku console, executing bash and running the command in the dyno.

```bash
$ flask init
```

# Database Migrations
If changes to the models are made, the database must be'migrated' so that it can be synced with the new models.
Then execute following commands using manage.py. More info [here](https://flask-migrate.readthedocs.io/en/latest/)

```bash
$ flask db init
$ flask db migrate
$ flask db upgrade
$ flask db --help
```

# Testing

## Unit & Integration
Unit and Integration tests are created in the App/test. You can then create commands to run them. Look at the unit test command in wsgi.py for example

```python
@test.command("user", help="Run User tests")
@click.argument("type", default="all")
def user_tests_command(type):
    if type == "unit":
        sys.exit(pytest.main(["-k", "UserUnitTests"]))
    elif type == "int":
        sys.exit(pytest.main(["-k", "UserIntegrationTests"]))
    else:
        sys.exit(pytest.main(["-k", "User"]))
```

You can then execute all user tests as follows

```bash
$ flask test user
```

You can also supply "unit" or "int" at the end of the comand to execute only unit or integration tests.

You can run all application tests with the following command

```bash
$ pytest
```

## Test Coverage

You can generate a report on your test coverage via the following command

```bash
$ coverage report
```

You can also generate a detailed html report in a directory named htmlcov with the following comand

```bash
$ coverage html
```

# Troubleshooting

## Views 404ing

If your newly created views are returning 404 ensure that they are added to the list in main.py.

```python
from App.views import (
    user_views,
    index_views
)

# New views must be imported and added to this list
views = [
    user_views,
    index_views
]
```

## Cannot Update Workflow file

If you are running into errors in gitpod when updateding your github actions file, ensure your [github permissions](https://gitpod.io/integrations) in gitpod has workflow enabled ![perms](./images/gitperms.png)

## Database Issues

If you are adding models you may need to migrate the database with the commands given in the previous database migration section. Alternateively you can delete you database file.
