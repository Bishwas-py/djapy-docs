Handling response is one of the most important part of any API-based application. Djapy provides a simple way to handle
responses
Pydatic's validators and computed fields.

## Response

The response is automatically serialized to JSON using the Pydantic model.

```python
# schemas.py
from djapy import Schema


class UserSchema(Schema):  # or TypedDict
    username: str
    email: str


# views.py
@djapify
def get_user(request, username: str) -> {200: UserSchema, 404: str}:
    user = User.objects.get(username=username)
    return user


# urls.py
urlpatterns = [
    path('get-user/', views.get_user, name='get-user'),
]
```

- The response will be serialized to JSON using the `UserSchema` model, if not valid, pydantic error will be raised.
- If the response is a valid instance of the model, it will be serialized to JSON and returned with a 200 status code.
- If the response is a string, it will be returned with a 200 status code.

#### Invalid error response

```json
{
  "error": [
    {
      "type": "missing",
      "loc": [
        "response",
        "message"
      ],
      "msg": "Field required",
      "input": {
        "error": "You are not allowed to create a user"
      },
      "url": "https://errors.pydantic.dev/2.6/v/missing"
    }
  ],
  "error_count": 1,
  "title": "output"
}
```

## Computed field

Pydantic's `@computed_field` decorator is used to define a computed field in a schema. It is used to define a field that
is not present in the model but is computed from the existing fields.

```python
class PostSchema(Schema):
    title: str
    slug: str
    body: str

    @computed_field
    def plain_text_body(self) -> str:
        return strip_tags(self.body)
```

### Accessing context in computed field

You can access the context in the computed field using the `context` parameter.

```python
from djapy.schema import Schema, Outsource


class LikeDataDict(TypedDict):
    like_count: int
    have_self: bool
    last_like: DetailedLikeSchema | None


class SimpleLikeSchema(Schema, Outsource):
    # ... other fields

    @computed_field
    def likes(self) -> LikeDataDict:
        likes = PolymorphicLike.get_object_likes(self._obj).alive()
        return {
            'like_count': likes.count(),
            'have_self': likes.filter(user=self._ctx['request'].user).exists() if self._ctx[
                'request'].user.is_authenticated else False,
            'last_like': likes.last() if likes.exists() else None,
        }
```

> `Outsource` is a mixin that provides `_obj` and `_ctx` attributes to the schema. It also provides a
> `_info` as `ValidationInfo` object.

### Accessing source object in computed field

You can access the source object in the computed field using the `_obj` attribute. In the above
example, `PolymorphicLike.get_object_likes(self._obj)` is used to get the likes of the source object.

Basically, `_obj` is the object that is being serialized, generally a Django model instance.

## Validators

Pydantic's `@field_validator` or `@model_validator` decorator can be used to validate the fields of a schema.

```python
from pydantic import field_validator


class PostSchema(Schema):
    title: str
    slug: str
    body: str

    @field_validator('body', mode='before')
    def assign_body(cls, body):
        linkified_body = bleach.linkify(body, callbacks=[set_nofollow, set_target])
        return bleach.clean(linkified_body,
                            tags=BLEACH_ALLOWED_TAGS,
                            attributes=BLEACH_ALLOWED_ATTRIBUTES,
                            strip=False)
```

## How-to

### How to return one Schema for multiple status codes?

You can return one schema for multiple status codes by using the `uni_schema(` function.

```python
@djapify
def confirm_email(request, confirmation_token: str) -> uni_schema(MessageOut, {200, 400, 422}):
    # ... your code
    return {...}  # mapped with MessageOut
```

You can also use `uni_schema.success_2xx(YourSchema)`, `uni_schema.error_4xx(YourSchema)`
or `uni_schema.error_5xx(YourSchema)`.

### How to return a list of items or QuerySet?

While retuning many-to-many relationships or a django queryset, you can use `QueryList` type.

```python
from djapy.schema import QueryList


@djapify
def get_users(request) -> QueryList[TodoSchema]:
    todos = Todo.objects.all()  # or filter() # or user.todos_set.all()
    return todos
```


### How to return a image field?

You can use `ImageField` type to return image fields.

```python
from djapy.schema.schema import ImageUrl


class UserSchema(Schema):
    username: str
    email: str
    profile_pic: ImageUrl
```

