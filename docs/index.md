# Introduction

Djapy is a modern Python framework that enables you to create RESTful APIs within [Django](https://www.djangoproject.com) with zero boilerplate, using pure Python and [Pydantic](https://docs.pydantic.dev/latest). It's designed to be simple, fast, and powerful.

Djapy embraces Django's "batteries included" philosophy while introducing modern API development patterns that make your code cleaner and more maintainable.

```python
@djapify
def get_user(request, username: str) -> UserSchema:
    return User.objects.get(username=username)

# Want asynchronous?
@async_djapify
async def get_user(request, username: str) -> UserSchema:
    return await User.objects.aget(username=username)
```

> It's that simple! Both sync and async support out of the box.

## Features

- **Zero Boilerplate**: Build APIs with pure Python - no serializers, viewsets, or extra configuration needed
- **Type Safety**: Full Python type hints with Pydantic validation
- **Async Support**: Write high-performance async views with `async_djapify`
- **Modern Auth**: Simple session auth with customizable mechanisms
- **Smart Validation**: Built-in request/response validation using Pydantic
- **Auto Documentation**: OpenAPI/Swagger integration with dark mode support
- **Django Native**: Seamlessly works with Django's decorators and features
- **Flexible Pagination**: Built-in cursor, offset-limit, and page number pagination
- **Error Handling**: Customizable error handling and validation messages
- **IDE Friendly**: Full intellisense and type checking support

## Installation

Djapy requires Python 3.10+ and is available on PyPI:

```bash
pip install djapy
# or for latest development version
pip install git+https://github.com/Bishwas-py/djapy.git@main
```

## Creating a new project

For your quick start:

```bash
django-admin startproject myproject
cd myproject
pip install djapy
```

Now you're ready to create your first Djapy endpoint! See the [usage guide](usage) for more details.
