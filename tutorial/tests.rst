import pytest
from flaskr import create_app
from flaskr.db import get_db, init_db

@pytest.fixture
def app():
    app = create_app({'TESTING': True, 'DATABASE': 'test_flaskr.sqlite',})
    with app.app_context():
        init_db()
    yield app

@pytest.fixture
def client(app):
    return app.test_client()

@pytest.fixture
def runner(app):
    return app.test_cli_runner()
```

## Authentication Tests

Test successful and failed logins:

```python
def test_login(client):
    # Successful login
    response = client.post('/auth/login', data={'username': 'test', 'password': 'test'})
    assert 'http://localhost/auth/login' not in response.headers['Location']

    # Failed login
    response = client.post('/auth/login', data={'username': 'wrong', 'password': 'wrong'})
    assert b'Incorrect username or password.' in response.data
```

## Form Handling Tests

Test form submission with valid and invalid data:

```python
def test_create_message(client):
    # Valid submission
    client.post('/auth/login', data={'username': 'test', 'password': 'test'})
    response = client.post('/create', data={'title': 'Test', 'body': 'This is a test.'})
    assert b'Message created' in response.data

    # Invalid submission
    response = client.post('/create', data={'title': '', 'body': ''})
    assert b'Title and body are required.' in response.data
```

## Database Interaction Tests

Test creating and retrieving messages:

```python
def test_message_flow(client):
    # Create a new message
    client.post('/auth/login', data={'username': 'test', 'password': 'test'})
    client.post('/create', data={'title': 'Hello', 'body': 'World'})
    message = get_db().execute('SELECT * FROM message WHERE title = "Hello"').fetchone()
    assert message is not None
    assert message['body'] == 'World'
```

## Error Handling Tests

Test accessing a route without authentication:

```python
def test_unauthorized_access(client):
    response = client.get('/create')
    assert response.status_code == 401
