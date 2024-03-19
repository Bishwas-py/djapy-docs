# Usage

Here's how you can use this package:

```python
@djapify
def get_user(request, data: PostCreationSchema) -> {200: PostSchema, 404: MessageOut}:
    if request.user.has_perm('users.add_post'):
        user = Post.objects.create(title=data.title, content=data.content)
        return user  # mapped with PostSchema
    else:
        return 404, {
            'message': 'You are not allowed to create a user',
            'alias': 'not_allowed'
        }  # mapped with MessageOut
```

- `djapify` is a decorator that does all the magic; basically, it's a wrapper around the view function.
- The function signature is `def get_user(request, data: PostCreationSchema) -> {200: PostSchema, 404: MessageOut}:`
    - The first argument is always the request object.
    - The rest of the arguments are the parameters that you want to accept from the request.
    - The return type `-> {200: PostSchema, 404: MessageOut}`  is a dictionary that maps status codes to
      Schema[Pydantic models].
- The return value can be a string, Django model instance, or a Pydantic model instance; it should be mapped to the
  status code in the return type.
    - If `400, value` is returned, it will be mapped with the 400 status code's schema.
    - If `200 value` or just `value` is returned, it will be mapped with the 200 status code's schema.
