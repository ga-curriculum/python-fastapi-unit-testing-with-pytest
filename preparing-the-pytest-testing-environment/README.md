<h1>
  <span class="headline">FastAPI Unit Testing with Pytest</span>
  <span class="subhead">Preparing the Pytest Testing Environment</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to prepare and configure a Pytest testing environment for API testing.

## Pytest setup and configuration

Pytest is a testing framework that helps automate and organize test execution. When working with FastAPI, Pytest allows us to verify that our API endpoints behave as expected.

### configuring pytest

By default, Pytest will look for test files inside a `tests` directory in the root of the project. That allows us to always run tests using the following command:

```bash
pipenv run pytest
```

### Creating a `pytest.ini` file

First, to improve debugging, we need to add a `pytest.ini` file to the root of the project. This file customizes how tests run and what output is displayed.

```sh
touch pytest.ini
```

In your `pytest.ini` add the following contents:

```ini
[pytest]
addopts = -rP -p no:warnings
```

- `-rP`: Ensures that `print()` statements appear in the test output, even if the test passes.
- `-p no:warnings`: Suppresses warnings related to PostgreSQL drivers that do not affect test execution.

## Setting up a test database

During testing, we will use an in-memory SQLite database instead of PostgreSQL. SQLite is lightweight, requires no additional setup, and runs entirely within the test environment. Because it operates in-memory, there is no need to install or configure a separate database server. The database resets automatically after each test session, ensuring a clean state for every run.

### Configuring `tests/conftest.py`

Create a `tests` directory and inside it, add a `conftest.py` file. This file will handle test setup, including database configuration and dependency overrides.

```sh
mkdir tests
touch tests/conftest.py
```

Add the following to your `conftest.py` file:

```py
# tests/conftest.py

import pytest
from starlette.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session
import sys
import os

sys.path.insert(0, os.path.abspath(os.path.join(os.path.dirname(__file__), '..')))

from main import app
from database import get_db
from models.base import Base
from tests.lib import seed_db

SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"

engine = create_engine(SQLALCHEMY_DATABASE_URL)
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base.metadata.create_all(bind=engine)

@pytest.fixture(scope="module")
def test_app():
    client = TestClient(app)
    yield client

@pytest.fixture(scope="module")
def test_db() -> Session:
    Base.metadata.drop_all(bind=engine)
    Base.metadata.create_all(bind=engine)
    db = TestingSessionLocal()
    seed_db(db)
    yield db
    db.close()

@pytest.fixture(scope="module")
def override_get_db(test_db):
    def _get_db_override():
        return test_db
    app.dependency_overrides[get_db] = _get_db_override
    yield
    app.dependency_overrides = {}
```

This file sets up:

- **A test database**: Uses SQLite instead of PostgreSQL for isolated testing.
- **Fixtures**: Reusable test setup functions that initialize and tear down test components.
- **Dependency overrides**: Ensures that FastAPI uses the test database instead of the production one.

## Utility functions for testing

We will store reusable helper functions in a file called `tests/lib.py`. These functions handle database seeding and user authentication for our test DB.

Create a `tests/lib.py` file:

```sh
touch tests/lib.py
```

And add the following contents:

```py
# tests/lib.py

from fastapi.testclient import TestClient
from data.tea_data import teas_list, comments_list
from data.user_data import user_list

def seed_db(db):
    db.commit()
    db.add_all(user_list)
    db.commit()
    db.add_all(teas_list)
    db.commit()
    db.add_all(comments_list)
    db.commit()

def login(test_app: TestClient, username: str, password: str):
    response = test_app.post("/api/login", json={"username": username, "password": password})

    if response.status_code != 200:
        raise Exception(f"Login failed: {response.json().get('detail', 'Unknown error')}")

    token = response.json().get('token')
    if not token:
        raise Exception("No token returned from login endpoint.")

    headers = {"Authorization": f"Bearer {token}"}
    return headers
```

- `seed_db(db)`: Populates the test database with initial data.
- `login(test_app, username, password)`: Logs in a test user by sending both a username and password to the `/api/login` endpoint. Returns authentication headers containing a JWT token. Raises an exception if login fails or no token is returned.

## Naming test files and functions

Pytest follows specific naming conventions to identify test files and functions:

| Naming Rule                                                 | Example                        |
| ----------------------------------------------------------- | ------------------------------ |
| Test files should start with `test_` or end with `_test.py` | `test_teas.py`, `teas_test.py` |
| Test functions should start with `test_`                    | `def test_create_tea():`       |

Following these conventions ensures that Pytest can discover and run all test cases.

In the next lesson, we will create a `test_teas.py` file to verify the functionality of our tea-related API endpoints.
