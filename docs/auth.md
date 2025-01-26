# Auth, well that's a thing

Authentication is super critical for any application.
It's the process of verifying the identity of a user. In this section, we'll cover how to authenticate users in your
application.

## Session Authentication

Session authentication is the default authentication mechanism for Django, and Djapy supports it out of the box.

To use session authentication, you need to import `SessionAuth` from `djapy.auth` and utilize it in your views.

```python
from djapy import SessionAuth, djapy_auth


@djapy_auth(SessionAuth, permissions=['plans.todos.can_view'])
def todo_list(request):
    ...
```

or

```python
from djapy import SessionAuth, djapify


@djapify(auth=SessionAuth)  # or SessionAuth(permissions=['plans.todos.can_view'])
def todo_list(request):
    ...
```

### Auth all over the views

or if you wanna assign authentication to the all endpoints in your `views.py` file, you can use `AUTH_MECHANISM`
variable.

```python
"""
after assigning AUTH_MECHANISM, it automatically assigns the authentication to all the endpoints in the current views file.
Only use this if you want to assign the same authentication mechanism to all the endpoints in the views file.

If you want to assign different authentication mechanisms to different endpoints, use the @djapy_auth decorator.
"""
AUTH_MECHANISM = SessionAuth(permissions=['plans.todos.can_view'])  # or just SessionAuth; initializations are optional
```

## Extending Base Authentication

You can extend the base authentication mechanism to create your own custom authentication mechanism.

Here's an example of how you can do that:

```python
from djapy import BaseAuthMechanism


class CustomAuth(BaseAuthMechanism):
    def __init__(self, permissions):
        self.permissions = permissions

    def authenticate(self, request):
        # Your custom authentication logic here
        return True

    def authorize(self, request):
        # Your custom authorization logic here
        return True

    def schema(self):
        return {}  # Your custom schema here to be used in the swagger documentation

    def app_schema(self):
        return {}  # Your custom app schema here to be used in the swagger documentation
```

## How to?

### How to implement async with auth?

It's pretty much the same, you do it the same way you do it in `sync`.

```python
@async_djapify(auth=SessionAuth)
async def get_protected_data(request) -> DataSchema:
  ...
```