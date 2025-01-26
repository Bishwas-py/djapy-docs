# Request Handling

Djapy makes handling request data (JSON, forms, query parameters) simple and type-safe. Let's see how.

## Data Input Types

You can receive data in three ways:

- `Schema`: For JSON data (`application/json`)
- `Form`: For form data (`application/x-www-form-urlencoded`)
- Query parameters: For URL parameters

Let's look at each:

## 1. JSON Data (Schema)

```python
from djapy import Schema, async_djapify
from pydantic import EmailStr

class CreateUserSchema(Schema):
    username: str
    email: EmailStr
    bio: str | None = None

@async_djapify(method="POST")
async def create_user(request, data: CreateUserSchema) -> {201: UserSchema}:
    user = await User.objects.acreate(
        username=data.username,
        email=data.email,
        bio=data.bio
    )
    return 201, user
```

=== "cURL"
    ```bash
    curl -X POST http://localhost:8000/api/users/ \
      -H "Content-Type: application/json" \
      -d '{
        "username": "ada",
        "email": "ada@python.org",
        "bio": "First programmer"
      }'
    ```

=== "TypeScript"
    ```typescript
    // Using fetch
    const createUser = async (userData: {
      username: string;
      email: string;
      bio?: string;
    }) => {
      const response = await fetch('/api/users/', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(userData),
      });
      
      if (!response.ok) {
        throw new Error('Failed to create user');
      }
      
      return response.json();
    };

    // Usage
    try {
      const user = await createUser({
        username: 'ada',
        email: 'ada@python.org',
        bio: 'First programmer'
      });
      console.log('User created:', user);
    } catch (error) {
      console.error('Error:', error);
    }
    ```

## 2. Form Data (Form)

```python
from djapy import Form, async_djapify
from pydantic import constr

class AnalyzeWebsiteForm(Form):
    url: constr(min_length=1, pattern=r"https?://.*")
    keywords: str | None = ""
    deep_scan: bool = False

@async_djapify(method="POST")
async def analyze_website(request, data: AnalyzeWebsiteForm) -> dict:
    analyzer = WebAnalyzer(data.url, keywords=data.keywords)
    return await analyzer.run_analysis(deep=data.deep_scan)
```

=== "cURL"
    ```bash
    curl -X POST http://localhost:8000/api/analyze/ \
      -H "Content-Type: application/x-www-form-urlencoded" \
      -d "url=https://python.org" \
      -d "keywords=programming" \
      -d "deep_scan=true"
    ```

=== "TypeScript"
    ```typescript
    // Using FormData
    const analyzeWebsite = async (url: string, keywords: string = '', deepScan: boolean = false) => {
      const formData = new FormData();
      formData.append('url', url);
      formData.append('keywords', keywords);
      formData.append('deep_scan', deepScan.toString());
      
      const response = await fetch('/api/analyze/', {
        method: 'POST',
        body: formData,
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      });
      
      if (!response.ok) {
        console.error('Analysis failed');
        return;
      }
      
      return response.json();
    };

    // Usage
    try {
      const results = await analyzeWebsite(
        'https://python.org',
        'programming',
        true
      );
      console.log('Analysis results:', results);
    } catch (error) {
      console.error('Error:', error);
    }
    ```

## 3. Query Parameters

```python
from datetime import date
from typing import Optional
from pydantic import constr

@async_djapify
async def search_posts(
    request,
    q: constr(min_length=3),
    category: Optional[str] = None,
    date_from: Optional[date] = None
) -> {200: list[PostSchema]}:
    query = Post.objects.filter(title__icontains=q)
    
    if category:
        query = query.filter(category=category)
    if date_from:
        query = query.filter(created_at__gte=date_from)
        
    return await query.aall()
```

## Type Validation

Djapy uses Pydantic for validation, giving you powerful type checking:

```python
from pydantic import constr, EmailStr, HttpUrl

class ContactForm(Form):
    name: constr(min_length=2, max_length=50)
    email: EmailStr
    website: HttpUrl | None = None
    message: constr(min_length=10)

@djapify(method="POST")
def submit_contact(request, data: ContactForm) -> {200: MessageSchema}:
    # Data is already validated:
    # - name is 2-50 chars
    # - email is valid
    # - website (if provided) is valid URL
    # - message is at least 10 chars
    return send_contact_email(data)
```

## Invalid Data Response

When validation fails, Djapy returns a clear error response:

```json
{
  "error": [
    {
      "type": "string_too_short",
      "loc": ["name"],
      "msg": "String should have at least 2 characters",
      "input": "a"
    },
    {
      "type": "email_invalid",
      "loc": ["email"],
      "msg": "Invalid email format",
      "input": "not-an-email"
    }
  ],
  "error_count": 2,
  "title": "ContactForm"
}
```

## Advanced Features

### Using Basic Types as JSON Input

Use `payload` when you want to treat basic types as part of the JSON body rather than query parameters:

```python
from djapy.schema import payload
from pydantic import HttpUrl


@async_djapify(method="POST")
async def content_quality_view(request, url: payload(HttpUrl), deep_scan: payload(bool) = False) -> bool:
   return await get_content_quality(url, deep_scan)
```

=== "cURL"
    ```shell
    curl -X POST http://localhost:8000/api/content-quality/ \
      -H "Content-Type: application/json" \
      -d '{
        "url": "https://example.com/article",
        "deep_scan": true
      }'
    ```

=== "TypeScript"
    ```typescript
    const checkContentQuality = async (url: string, deepScan: boolean = false) => {
        const response = await fetch('/api/content-quality/', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                url: url,
                deep_scan: deepScan,
            }),
        });
    
        if (!response.ok) {
            throw new Error('Content quality check failed');
        }
    
        return response.json();
    };
    
    // Usage
    try {
        const quality = await checkContentQuality(
            'https://example.com/article',
            true
        );
        console.log('Content quality:', quality);
    } catch (error) {
        console.error('Error:', error);
    }
    ```

=== "Python"
    ```python
    import requests
    
    
    def check_content_quality(url: str, deep_scan: bool = False) -> dict:
       response = requests.post(
          'http://localhost:8000/api/content-quality/',
          json={
             'url': url,
             'deep_scan': deep_scan
          }
       )
       response.raise_for_status()
       return response.json()
    
    
    # Usage
    try:
       quality = check_content_quality(
          'https://example.com/article',
          deep_scan=True
       )
       print('Content quality:', quality)
    except requests.exceptions.RequestException as e:
       print('Error:', e)
    ```

### String Constraints

Use Pydantic's constraints for powerful string validation:

```python
from pydantic import constr

@djapify
def search_by_code(
    request,
    code: constr(pattern=r"^[A-Z]{2}\d{4}$")
) -> {200: ProductSchema}:
    # Only accepts codes like "AB1234"
    return Product.objects.get(code=code)
```

!!! tip "Remember"
    - Use `Schema` for JSON data
    - Use `Form` for form-encoded data
    - Use direct parameters for query strings
    - Use `payload()` when you want basic types in JSON body
