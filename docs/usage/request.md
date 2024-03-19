Djapy simplifies the process of handling request data and query parameters in your API. Here's how easily you can do it.

## Data Input

You can use `Schema` or `TypedDict` to define the request data schema. `Schema` is recommended, but feel free to play
around.

!!! warning

    `TypedDict` is supported in Python 3.6 and later.
    ```python
    from typing_extensions import TypedDict
    ```

### Schema

Schema helps you to define the request data schema. It's a subclass of `pydantic.BaseModel` that adds some extra
functionality.

```python
# schemas.py
from djapy import Schema


class CreateUserSchema(Schema):
    username: str
    email: EmailStr


# views.py
@djapify(allowed_method=['POST'])
def create_user(request, data: CreateUserSchema) -> {...}:
    user = User.objects.create_user(username=data.username, password=data.password)
    return user
```

- The schema is used to validate the request data before it's passed to the view function.
- If the data is invalid, a [pydantic error](#invalid-request-error-response) will be raised and returned as a response.

**HTTP Request:**

```http
POST /create-user/ HTTP/1.1
Content-Type: application/json
{
  "username": "bishwas",
  "email": "qrst@uvw.xyz"
}
```

## Query Parameters

You can also accept query parameters in your view function:

```python
@djapify
def get_user(request, username: str) -> {200: UserSchema, 404: str}:
    user = User.objects.get(username=username)
    return user
```

**HTTP Request:**

```http
GET /get-user/?username=bishwas HTTP/1.1
Content-Type: application/json
```

### Allowed query parameter types

These are the list of allowed query parameter types `[basic parameters]`:

- `str` or `Optional[str]` or `Literal["value1", "value2"]`
- `int` or `Optional[int]` or `Literal[1, 2, 3]`
- `float` or `Optional[float]`
- `bool`
- `datetime`
- `constr()` or `conint()` or `confloat()` or `condecimal()`

> If anything other than these types are used, it will be considered as a data input.

> Like: `list[str]`, `dict[str, int]`, etc.

### Invalid request error response

```json
{
  "error": [
    {
      "type": "missing",
      "loc": [
        "username"
      ],
      "msg": "Field required",
      "input": {
        "my_id": "1",
        "id": "1"
      },
      "url": "https://errors.pydantic.dev/2.6/v/missing"
    },
    {
      "type": "missing",
      "loc": [
        "email"
      ],
      "msg": "Field required",
      "input": {
        "my_id": "1",
        "id": "1"
      },
      "url": "https://errors.pydantic.dev/2.6/v/missing"
    }
  ],
  "error_count": 2,
  "title": "CreateUserSchema"
}
```

## How-to

### How to use basic parameters as data input?

You can use basic parameters as data input by using a `payload` function:

```python
from djapy.schema import payload


@djapify(allowed_method=['POST'])
def check_site_exists(request, url: payload(HttpUrl), check_all_pages: payload(bool) = False) -> bool:
    if does_site_exist(url, check_all_pages):
        return True
    else:
        return False
```

Now, you can use the `url` and `check_all_pages` as data input. The `url` will be validated as a `HttpUrl`
and `check_all_pages` will be validated as a `bool`.

### How to use `constr()` as data input?

You can use `constr()` to validate a string, according to the constraints provided.
Visit [Pydantic's constr](https://docs.pydantic.dev/latest/api/types/#pydantic.types.constr) for more information.

```python
from pydantic import constr


@djapify
def search_game_by_name(request, name: constr(min_length=3, max_length=50)) -> {200: GameSchema, 404: str}:
    game = Game.objects.get(name__icontains=name)
    return game
```

- We set the minimum length of the `name` to 3 and the maximum length to 50. Validation will be done accordingly.

The similar goes for `conint()`, `confloat()`, or any other `con*` Constraints.

If in schema:

```python
class GameSchema(Schema):
    name: constr(min_length=3, max_length=50)
```