Handling response is one of the most important part of any API-based application. Djapy provides a simple way to handle
responses
Pydatic's validators and computed fields.

## Computed field

Pydantic's `@computed_field` decorator is used to define a computed field in a schema. It is used to define a field that
is not present in the model but is computed from the existing fields.

```python
class PostSchema(Schema):
    title: str
    slug: str
    body: str

    @computed_field
    @property
    def plain_text_body(self) -> str:
        return strip_tags(self.body)
```

### Accessing context in computed field

You can access the context in the computed field using the `context` parameter.

```python
from djapy.schema import SourceAble, Schema


class LikeDataDict(TypedDict):
    like_count: int
    have_self: bool
    last_like: DetailedLikeSchema | None


class SimpleLikeSchema(Schema, SourceAble):
    # ... other fields

    @computed_field
    def likes(self) -> LikeDataDict:
        likes = PolymorphicLike.get_object_likes(self._source_obj).alive()
        return {
            'like_count': likes.count(),
            'have_self': likes.filter(user=self._context['request'].user).exists() if self._context[
                'request'].user.is_authenticated else False,
            'last_like': likes.last() if likes.exists() else None,
        }
```

> `SourceAble` is a mixin that provides `_source_obj` and `_context` attributes to the schema. It also provides a
> `_validation_info` as `ValidationInfo` object.

### Accessing source object in computed field

You can access the source object in the computed field using the `_source_obj` attribute. In the above
example, `PolymorphicLike.get_object_likes(self._source_obj)` is used to get the likes of the source object.

Basically, `_source_obj` is the object that is being serialized, generally a Django model instance.

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

