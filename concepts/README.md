<h1>
  <span class="headline">FastAPI Unit Testing with Pytest</span>
  <span class="subhead">Concepts</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to describe the purpose of the Python `Pytest` library and demonstrate how to set up a `Pytest` testing environment.

## Testing FastAPI with `pytest`

[Pytest](https://docs.pytest.org/en/stable/) is a widely used testing framework in Python that simplifies writing and running tests. When developing a FastAPI application, testing ensures that individual components work correctly before deploying to production.

With Pytest, you can write unit tests for your FastAPI application, automatically execute them, and get detailed feedback on test results.

### Installing `pytest` and dependencies

To start using Pytest with FastAPI, install the necessary dependencies:

```bash
pipenv install pytest starlette httpx
```

- `pytest`: The core testing framework.
- `starlette`: Provides essential components for FastAPI, including test utilities.
- `httpx`: A powerful HTTP client used to simulate API requests in tests.

### Running tests with `pytest`

After installing the dependencies, you can execute your tests with the following command:

```bash
pipenv run pytest
```

Pytest will automatically discover test files and execute them, displaying a summary of the results in your terminal.