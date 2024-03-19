

## Custom error response

You can also return a custom error response, by create error handlers inside `djapy_ext/errorhandler.py`;

```python
from your_custom_exptions.exceptions import MessageOut


def handle_message_out(request, exception: MessageOut):
    print('Value: Handler executed: ', request)
    return {
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
        raise MessageOut('User not found', 'user_not_found', 'error')
```

- If exception is raised, the handler will be executed and the response will be returned.


