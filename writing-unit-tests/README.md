<h1>
  <span class="headline">FastAPI Unit Testing with Pytest</span>
  <span class="subhead">Writing Unit Tests</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to write unit tests for FastAPI endpoints using Pytest.

## Fixtures

Pytest provides a feature called fixtures, which allows us to run setup and cleanup code before and after tests. This helps keep tests efficient and independent.

In our `conftest.py` file, we define fixtures to seed the test database with sample users and teas before each test runs and then clear the data afterward. Without fixtures, we would need to set up test data inside every test function, which would be repetitive and slow.

Fixtures ensure that each test starts with a fresh database state, preventing one test from affecting another. This helps maintain test reliability.

### How `pytest` fixtures work

A fixture uses the `yield` statement to control execution flow:

1. Everything before `yield` runs **before** the test.
2. The test function runs.
3. Everything after `yield` runs **after** the test, cleaning up any test data.

By using fixtures, we know that the database is reset after each test, keeping our tests isolated.

## Creating unit tests

Now that we have our test setup ready, let's write actual unit tests.

### Testing `GET` all teas

Before we begin writing the test, let's create the file where we'll store our test for fetching all teas from the database.

```sh
touch tests/test_teas.py
```

Now, let's proceed with writing the test for `GET all teas`.

```python
# tests/test_teas.py

import pytest
from fastapi.testclient import TestClient
from sqlalchemy.orm import Session
from models.user import UserModel
from models.tea import TeaModel
from tests.lib import login
from main import app

def test_get_teas(test_app: TestClient, override_get_db):
    response = test_app.get("/api/teas")
    assert response.status_code == 200
    teas = response.json()
    assert isinstance(teas, list)
    assert len(teas) == 2  # Ensure two teas are in the test database
    for tea in teas:
        assert 'id' in tea
        assert 'name' in tea
        assert 'in_stock' in tea
        assert 'rating' in tea
        assert 'user' in tea
        assert 'email' in tea['user']
        assert 'username' in tea['user']
```

Key points in this test:

- The function name starts with `test_`, which allows Pytest to recognize it as a test.
- The `assert` statements check if expected values match the actual API response.
- The test verifies that the correct number of teas is returned and that the response structure is correct.

To run this test, use the command:

```sh
pipenv run pytest
```

If successful, Pytest will display a green success message. If it fails, Pytest will show which assertion failed.

### Testing `POST` create tea

Next, let's write a test to create a new tea. Since authentication is required, we must first log in and obtain a token.

```python
# tests/test_teas.py


def test_create_tea(test_app: TestClient, test_db: Session):

    # Create a new mock user in the test database
    user = UserModel(
        username='testUser123',
        email='hello@example.com',
        password_hash=pwd_context.hash('mys3cretp2ssw0rd')
    )
    test_db.add(user)
    test_db.commit()


    # Log in as a mock user and get authentication headers
    headers = login(test_app, 'testUser123', 'mys3cretp2ssw0rd')

    # Data for creating a new tea
    tea_data = {
        "name": "Test Tea",
        "in_stock": True,
        "rating": 4
    }

    # Send a POST request to create a new tea
    response = test_app.post("/api/teas", headers=headers, json=tea_data)

    # Verify that the response is successful
    assert response.status_code == 200
    assert response.json()["name"] == tea_data["name"]
    assert response.json()["in_stock"] == tea_data["in_stock"]
    assert response.json()["rating"] == tea_data["rating"]
    assert "id" in response.json()  # Ensure an ID is returned
    assert "user" in response.json()  # Ensure user data is included
    assert response.json()['user']["username"] == 'testUser123'

    # Verify the tea was created in the database
    tea_id = response.json()["id"]
    tea = test_db.query(TeaModel).filter(TeaModel.id == tea_id).first()
    assert tea is not None
    assert tea.name == tea_data["name"]
    assert tea.in_stock == tea_data["in_stock"]
    assert tea.rating == tea_data["rating"]
```

This test does the following:

- Creates a mock user for the test.
- Logs in the test user and gets a token.
- Sends a `POST` request to create a tea.
- Asserts that the response data matches the input.
- Queries the database to verify the tea was successfully stored.

Run the test again with:

```sh
pipenv run pytest
```

If everything is set up correctly, this test should pass as well.

With these tests in place, we can confidently verify that our `GET` and `POST` endpoints behave as expected!
