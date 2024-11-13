# Introduction

Djapy is a library that allows you to create RESTful APIs within [Django](https://www.djangoproject.com) with no boilerplate, using plain Python and [Pydantic](https://docs.pydantic.dev/latest). It is designed to be simple, easy to use, and fast.

Djapy is molded according to `Django`'s philosophy of "batteries included", and is designed to be as simple as possible to use, while still being powerful enough to handle most use cases.

```python
@djapify
def get_user(request) -> {200: UserSchema, 404: str}:
    return request.user
```

> It's that simple!

## Features

* **IO type checking**: Pydantic-based, and swagger integrated
* **Easy to use**: No boilerplate, just plain Python
* **Delicious Syntax**: Simple and easy to understand
* **Fast**: Built on top of Django, and Pydantic, and is designed to be fast
* **Extensible**: Customizable, and can be used with any Django project
* **Pagination**: CursorPagination, OffsetLimitPagination and PageNumber out of the box; with ability to create custom pagination
* **Error handling**: Customizable error handling, and error messages; validation errors/messages can be customized
* **Django friendly**: Based on Django's structure, architecture and
* **Auth**: Built-in support for Django's authentication and authorization; with ability to create custom authentication and authorization

## Installation

Djapy is available on PyPI, and can be installed with `pip`:

```bash
pip install djapy
or
pip install git+https://github.com/Bishwas-py/djapy.git@main # for the latest version
```

## Creating a new project

To create a new project, run the following command:

```bash
django-admin startproject <project_name>
```

More on this in the [usage](docs/usage.md) section.
