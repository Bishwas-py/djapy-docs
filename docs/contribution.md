# Contribution Guidelines for djapy REST API Framework

## Introduction

This document lays out the guidelines for contributing to the djapy REST API framework. Your help in making djapy the
best it can be is gratefully appreciated. These norms are devised to make the contribution process smooth and consistent
for everyone involved.

## Getting Started

1. **Create a django project**: Create a new Django project and install djapy.
2. **Clone the repository**: Clone your forked djapy repository inside your Django project.
3. **Virtual Environment**: Create a virtual environment and install the dependencies.
4. **Run the server**: Run the Django server and start developing your feature.
5. **Development**: Develop your feature or fix the bug.
6. **Testing**: Write tests for your feature and run them to ensure everything is working as expected.
7. **Commit**: Commit your changes and push them to your forked repository.

Structure:

```properties
my_project/
_ ├── my_project/
_ ├── djapy/
_ ├── manage.py
```

## Contribution Workflow

1. **Issue**: For any major changes, please raise an issue beforehand describing the feature, enhancement, or bug-fix.

2. **Commit**: Be sure to commit your changes regularly. Adhere to the commit message standards - it should be concise,
   yet descriptive.

3. **Test**: Write tests that cover your code as precisely as possible. The test cases should pass successfully on your
   proposed changes.

4. **Pull Request**: Once you have completed your modifications and all tests have passed successfully, it is time to
   submit a pull request. Your pull request should always be against the 'development' branch, not 'master'.

5. **Code Review**: After submission, your pull request will then be reviewed.

## Code Style and Standards

1. **PEP8**: The code should stick to the PEP8 python coding standards.

2. **Documentation**: All the methods, classes, modules and functions must have docstrings.

3. **Comments**: Use comments to explain your code, especially if the section of code is complex or includes a
   workaround.

## Project Structure

These are the current directories we have in our project:

- `core`: where the base code resides.
- `openapi`: for OpenAPI-related features.
- `pagination`: for pagination-related code.
- `schema`: code related to application schema resides here.
- `templates`: here the HTML templates will reside.

Each directory has its own set of functions and rules defined. Kindly consider these while contributing to a specific
area.

## Conclusion

Thank you for deciding to contribute to djapy! Your efforts will help to enhance and extend the project in significant
ways. Always keep in mind that the goal of djapy is to provide a user-friendly, proficient and efficient REST API
framework for Python developers.