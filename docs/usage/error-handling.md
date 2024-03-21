Djapy simplifies the process of handling errors in your API. Here's how easily you can do it.

## Custom error response

You can also return a custom error response, by create error handlers inside `djapy_ext/errorhandler.py`;

```python
from your_custom_exptions.exceptions import MessageValueError


def handle_messaged_error(request, exception: MessageValueError):
    print('Value: Handler executed: ', request)
    return 404, {
        'message': exception.message,
        'alias': exception.alias,
        'message_type': exception.message_type,
    }
```

> Make sure your handler function name is `handle_<exception_name>`

And then, you can use it in your view function:

```python
@djapify
def get_user(request, username: str) -> {200: UserSchema, 404: MessageOut}:
    user = User.objects.get(username=username)
    if user:
        return user
    else:
        raise MessageValueError('User not found', 'user_not_found', 'error')
```

- If exception is raised, the handler will be executed and the response will be returned.

## How-to

### How to write a custom error exception

You can create your custom errors in `djapy_ext/exceptions.py` (recommended) or anywhere you like.
Below is an example of a custom error exception:

```python
class MessageValueError(Exception):
    def __init__(self, message: str | None = None,
                 alias="validation_error",
                 message_type="error",
                 messages: list[str] | None = None,
                 inline: dict[str, str | list[str]] | None = None):
        self.message = message
        self.messages = messages
        self.alias = alias
        self.message_type = message_type
        self.inline = inline
        super().__init__(message)
```
