# Usage

Here's how you can use Djapy to build powerful APIs:

## Basic Usage

```python
# Synchronous View
@djapify
def create_post(request, data: PostCreationSchema) -> {201: PostSchema, 403: MessageOut}:
   if request.user.has_perm('posts.add_post'):
      post = Post.objects.create(**data.dict())
      return 201, post  # Mapped to PostSchema
   return 403, {
      'message': 'You are not allowed to create a post',
      'alias': 'not_allowed'
   }  # Mapped to MessageOut


# Asynchronous View
@async_djapify
async def create_post(request, data: PostCreationSchema) -> {201: PostSchema, 403: MessageOut}:
   if await sync_to_async(request.user.has_perm)('posts.add_post'):
      post = await Post.objects.acreate(**data.dict())
      return 201, post
   return 403, {
      'message': 'You are not allowed to create a post',
      'alias': 'not_allowed'
   }
```

## Understanding the Components

### Decorators

- `@djapify` for synchronous views
- `@async_djapify` for asynchronous views

### Function Signature

- First argument is always the `request` object
- Additional arguments are parameters you want to accept from the request
- Return type annotation uses format: `{status_code: SchemaType}`
    - Example: `{200: PostSchema, 404: MessageOut}`

### Return Values

- Single value: Returns with 200 status code
  ```python
  return post  # 200 status
  ```
- Tuple of (status, value): Returns with specified status
  ```python
  return 201, post  # 201 status
  return 404, {'message': 'Not found'}  # 404 status
  ```

### Schema Mapping

- Return values are automatically validated and serialized according to the schema:
    - Django model instances are mapped to their corresponding schema
    - Dictionaries are validated against the specified schema
    - Strings or primitive types are passed through directly

### Async Support

When using `@async_djapify`:

- Use `await` with async ORM operations
- Use `sync_to_async` for synchronous Django operations
- All database queries should use async variants (`aget`, `afilter`, `acreate`, etc.)

Example with both sync and async database operations:

```python
@async_djapify
async def get_user_posts(request, username: str) -> {200: list[PostSchema], 404: MessageOut}:
   try:
      # Async database query
      user = await User.objects.aget(username=username)

      # Convert sync operation to async
      if not await sync_to_async(user.has_perm)('posts.view_posts'):
         return 403, {
            'message': 'Not authorized',
            'alias': 'not_authorized'
         }

      # Async query with filter
      posts = await Post.objects.filter(author=user).aall()
      return posts

   except User.DoesNotExist:
      return 404, {
         'message': 'User not found',
         'alias': 'user_not_found'
      }
```
