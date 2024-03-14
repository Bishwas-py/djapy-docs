# Pagination

You want something, djapily paginated? Right place you're in.

We provide three pagination out of the box; `OffsetLimitPagination`, `PageNumberPagination` and `CursorPagination`.

## OffsetLimitPagination

This is the default pagination mechanism in Djapy. It's simple and easy to use. It's based on the `offset` and `limit`
query parameters. Here's how you can use it:

```python
from djapy.pagination import OffsetLimitPagination, paginate


@djapify  # required
@paginate(OffsetLimitPagination)  # required, OR just @paginate, params: offset=0, limit=10
def todo_list(request, **kwargs) -> List[Todo]:
    return Todo.objects.all()
```

Make sure to add `paginate` decorator to your view and pass the pagination class as an argument. `List[Todo]` is the
return type hint of the view, also for Swagger documentation.

> `**kwargs` is required to pass the pagination parameters to the view.

## PageNumberPagination

This is another pagination mechanism in Djapy. It's based on the `page` and `page_size` query parameters. Here's how you
can use it:

```python
from djapy.pagination import PageNumberPagination, paginate


@djapify  # required
@paginate(PageNumberPagination)  # required, OR just @paginate, params: page_number=1, page_size=10
def todo_list(request, **kwargs) -> List[Todo]:
    return Todo.objects.all()
```

## CursorPagination

This is the last pagination mechanism in Djapy. It's based on the `cursor` query parameter. Here's how you can use it:

```python
from djapy.pagination import CursorPagination, paginate


@djapify  # required
@paginate(CursorPagination)  # required, OR just @paginate, params: cursor=0, limit=10
def todo_list(request, **kwargs) -> List[Todo]:
    return Todo.objects.all()
```

> `cursor` is the primary key of the last object in the previous page.

### Sample Request

You can pass the following query parameters to the endpoint to get the paginated response:

```http
GET /todos/all?limit=1&ordering=asc
```

also, with the `cursor` param:

```http
GET /todos/all?cursor=1&limit=1&ordering=asc
```

### Sample Response

```json
{
  "items": [
    {
      "id": 0,
      "title": "string",
      "description": "string",
      "completed_at": "2024-03-14T03:37:19.015Z",
      "will_be_completed_at": "2024-03-14T03:37:19.015Z",
      "created_at": "2024-03-14T03:37:19.015Z",
      "updated_at": "2024-03-14T03:37:19.015Z"
    }
  ],
  "cursor": 0,
  "limit": 0,
  "ordering": "asc",
  "has_next": true
}
```

### Parameters

Here are the parameters you can use with the cursor pagination:

- `cursor` (int): The primary key of the last object in the previous page.
- `limit` (int): The number of items to return in the response.
- `ordering` (str): The ordering of the items in the response. It can be `asc` or `desc`.
- `has_next` (bool): Whether there are more items in the next page.

## Extending Base Pagination

You can extend the base pagination mechanism to create your own custom pagination mechanism.

Here's an example of how you can do that:

```python
from djapy.pagination import BasePagination
from pydantic import model_validator


class CursorPagination(BasePagination):  # example
    """Cursor-based pagination."""

    query = [
        ('cursor', conint(ge=0), 0),
        ('limit', conint(ge=0), 1),
        # ... your custom query parameters here
    ]

    class response(Schema, Generic[G_TYPE]):
        items: G_TYPE
        cursor: int | None
        limit: int
        has_next: bool

        # ... your custom fields here

        @model_validator(mode="before")
        def make_data(cls, queryset, info):
            if not isinstance(queryset, QuerySet):
                raise ValueError("The result should be a QuerySet")
            # ... your custom logic here

            return {
                "items": queryset_subset,
                "cursor": cursor,
                "limit": limit,
                "has_next": has_next,
            }
```