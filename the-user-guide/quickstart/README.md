# Quickstart

## Make a Request

```python
import requests
```

- Let's try to get a webpage.

```python
r = requests.get('https://api.github.com/events')
```

- We have a `Response` object called `r`.

- We can get all the information we need from this object.

- This is how you make an HTTP POST request:

```python
r = requests.post('https://httpbin.org/post', data = {'key': 'value'})
```

- What about the other HTTP request types: PUT, DELETE, HEAD and OPTIONS?

- These are all just as simple:

```python
r = requests.put('https://httpbin.org/put', data = {'key': 'value'})
r = requests.delete('https://httpbin.org/delete')
r = requests.head('https://httpbin.org/get')
r = requests.options('https://httpbin.org/get')
```

## Passing Parameters In URLs

- You often want to send some sort of data in the URL's query string.

- If you were constructing the URL by hand, this data would be given as key/value pairs in the URL after a question mark, e.g. `httpbin.org/get?key=val`.

- Requests allows you to provide these arguments as a dictionary of strings, using the `params` keyword argument.

```python
payload = {'key1': 'value1', 'key2': 'value2'}
r = requests.get('https://httpbin.org/get', params = payload)
```

- You can see that the URL has been correctly encoded by printing the URL.

```python
print(r.url)
```

- Any dictionary key whose value is `None` will not be added to the URL's query string.

- You can also pass a list of items as a value:

```python
payload = {'key1': 'value1', 'key2': ['value2', 'value3']}
r = requests.get('https://httpbin.org/get', params=payload)
r.url
```

## Response Content

```python
r = requests.get('https://api.github.com/events')
r.text
```

- Requests will automatically decode content from the server.

- Most unicode charsets are seamlessly decoded.

- When you make a request, Requests makes educated quesses about the encoding of the response based on the HTTP headers.

- The text encoding guessed by Requests is used when you access `r.text`.

- You can find out what encoding Requests is using, and change it, using the `r.encoding` property:

```python
print(r.encoding)
r.encoding = 'ISO-8859-1'
```

- If you change the encoding Requests will use the new value of `r.encoding` whenever you call `r.text`.

- For example, HTML and XML have the ability to specify their encoding in their body.

- In situations like this, You should use `r.content` to find the encoding, and then set `r.encoding`.

- Requests will also use custom encodings in the event that you need them.

- If you havecreated your own encoding and registered it with the `codecs` module, you can simply use the codec name as the value of `r.encoding` and Requests will handle the decoding for you.

## JSON Response Content

- there's also a builtin JSON decoder, in case you're dealing with JSON data:

```python
r = requests.get('https://api.github.com/events')
r.json()
```

- In case the JSON decoding fails, `r.json()` raises an exception.

- For example, if the response gets a 204 (No Content), or if the response contains invalid JSON, attempting `r.json()` raises `requests.exception.JSONDecodeError`.

- The success of the call to `r.json()` does **not** indicate the success of the response.

- To check that a request is successful, use `r.raise_for_status()` or `r.status_code` is what you expect.

## Raw Response Content

- In the rare case that you'd like to get the raw socket response from the server, you can access `r.raw`.

- If you want to do this, make sure you set `stream = True` in your initial request.

```python
r = requests.get('https://api.github.com/events', stream = True)
print(r.raw)
r.raw.read(10)
```

- In general, however, you should use a pattern like this to save what is being streamed to a file:

```python
with open(filename, 'wb') as fd:
    for chunk in r.iter_content(chunk_size = 128):
        fd.write(chunk)
```

- Using `Response.iter_content` will handle a lot of what you would otherwise have to handle when using `Response.raw` directly.

- When streaming a download, the above is the preferred and recommended way to retrieve the content.

- `chunk_size` can be freely adjusted to a number that may better fit your use cases.

> ##### Note
>
> - `Response.iter_content` will automatically decode the `gzip` and `deflate` transfer-encodings.

## Custom Headers

- Simply pass in a `dict` to the `headers` parameter.

```python
url = 'https://api.github.com/some/endpoint'
headers = {'user-agent': 'my-app/0.0.1'}
requests.get(url, headers = headers)
```

- Note: Custom headers are given less precedence than more specific sources of information.

- Furthermore, Requests does not change its behavior at all based on which custom headers are specified.

- The headers are simply passed on into the final request.

- Note: All header values must be a `string`, bytestring, or unicode.

- While permitted, it's advised to avoid passing unicode header values.

## More complicated POST requests

- Typically, you want to send some form-encoded data -- much like an HTML form.

- To do this, simply pass a dictionary to the `data` argument.

- Your dictionary of data will automatically be form-encoded when the request is made:

```python
payload = {'key1': 'value1', 'key2': 'value2'}
r = requests.post('https://httpbin.org/post', data = payload)
r.text
```

- The `data` argument can also have multiple values for each key.

- This can be done by making data either a list of tuples or a dictionary with lists as values.

- This is particularly useful when the form has multiple elements that use the same key:

```python
payload_tuples = [('key1', 'value1'), ('key1', 'value2')]
r1 = requests.post('https://httpbin.org/post', data=payload_tuples)
payload_dict = {'key1': ['value1', 'value2']}
r2 = requests.post('https://httpbin.org/post', data=payload_dict)
print(r1.text)
r1.text == r2.text
```

- If you pass in a `string` instead of a `dict`, that data will be posted directly.

```python
import json

url = 'https://api.github.com/some/endpoint'
payload = {'some': 'data'}

r = requests.post(url, data = json.dumps(payload))
```

- The above code will NOT add the `Content-Type` header.

- If you need that header set and you don't want to encode the `dict` yourself, you can also pass it directly using the `json` parameter (added in version 2.4.2) and it will be encoded automatically.

```python
url = 'https://api.github.com/some/endpoint'
payload = {'some': 'data'}

r = requests.post(url, json = payload)
```

- The `json` parameter is ignored if either `data` or `files` is passed.

## Response Status Codes

```python
r = requests.get('https://httpbin.org/get')
r.status_code
```

- Requests also comes with a built-in status code lookup object for easy reference:

    ```python
    r.status_code == requests.codes.ok
    ```

- If we made a bad request, we can raise it with `Response.raise_for_status()`:

    ```python
    bad_r = requests.get('https://httpbin.org/status/404')
    print(bad_r.status_code)
    bad_r.raise_for_status()
    ```

```python
r.raise_for_status()
```

## Response Headers

- We can view the server's response headers using a Python dictionary:

    ```python
    r.headers
    ```

- The dictionary is special, though: it's made just for HTTP headers.

- According to RFC 7230, HTTP Header names are case-insensitive.

- So, we can access the headers using any capitalization we want:

```python
print(r.headers['Content-Type'])
r.headers.get('content-type')
```

- It is also special in that the server could have sent the same header multiple times with different values, but requests combines them so they can be represented in the dictionary within a single mapping, as per RFC 7230.
