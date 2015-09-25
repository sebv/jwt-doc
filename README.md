# JWT addon

Integration between Travis-CI and third-party services like Sauce Labs relies
on [encrypted
variables](http://docs.travis-ci.com/user/environment-variables/#Encrypted-Variables)
which works well for trusted branches and committers. For security reasons,
encrypted variables are not exposed to untrusted pull requests, so third-party
integrations do not work for pull requests.

The JWT addon replaces encrypted variables with a time-limited authentication
token, which is exposed to pull requests without security consequences.

For this to work the JWT addon needs to be enabled in the `.travis.yml` file,
and the third-party need to have integrated with the JWT service and allow
token-based authentication.

<img src="http://sebv.github.io/jwt-doc/travis_jwt.svg">

## JWT Client Configuration

### Encrypt the Access Key

For more information about encrypting variables or access keys in Travis CI
please refer to the [encryption key
documentation](http://docs.travis-ci.com/user/encryption-keys/).

Encrypt the access key:

```
travis encrypt SAUCE_ACCESS_KEY=123456789 # replace with your own access key
```

### .travis.yml

Add the encrypted key to the `jwt` section of the `.travis.yml`
file. For example:

```yml
addons:
  jwt:
     secure: <SAUCE_ACCESS_KEY ENCRYPTED>
```

You can also configure several services:

```yml
addons:
  jwt:
     saucelabs:
        secure: <SAUCE_ACCESS_KEY ENCRYPTED>
     thirdparty:
        secure: <THIRDPARTY_SHARED_SECRET ENCRYPTED>
```

### Use the Encrypted Key

The original variable names are available within the Travis CI build as
environment variables containing the JWT tokens instead of the original values.

For example, using the previous configuration `SAUCE_ACCESS_KEY` and
`THIRDPARTY_SHARED_SECRET` will be available as environment variables.

### Troubleshooting

1. Check if the third-party service is supported in the list below.
2. Contact the third-party support and provide them with the encrypted token (echo the key in your test script), and link to the Travis job.

## Third-Party Service Integration

Third-party service needs to implement a new authentication method on the server side so that the JWT token is recognized and verified.

### JWT Libraries

In most language JWT compliant libraries are available, making the implementation straightforward:

- python: https://pypi.python.org/pypi/PyJWT/1.3.0
- ruby: https://rubygems.org/gems/jwt
- Check http://jwt.io/ for more

### Payload

An example payload used to generate the JWT token:
```
{
  "iss": "travis-ci.org",
  "slug": "<SLUG>",
  "pull-request": "<PR>",
  "exp": <now+5400>,
  "iat": <now>
}
```

### Third Party Service Provider Code Sample 

A code sample which illustrates how to add JWT token authentication to third party services.

#### Python

In this example we assume the authentication credentials (using env variables e.g. `SERVICE_USERNAME` + `SERVICE_ACCESS_KEY`) of a RESTful API will be sent as HTTP BASIC AUTH header:

```
Authorization: Basic am9obmRvZTpleUowZVhBaU9pSktWMVFpTENKaGJHY2lPaUpJVXpJMU5pSjkuZXlKcGMzTWlPaUow\nY21GMmFYTXRZMmt1YjNKbklpd2ljMngxWnlJNkluUnlZWFpwY3kxamFTOTBjbUYyYVhNdFkya2lM\nQ0p3ZFd4c0xYSmxjWFZsYzNRaU9pSWlMQ0psZUhBaU9qVTBNREFzSW1saGRDSTZNSDAuc29RSmdI\nUjZjR05yOUxqX042eUwyTms1U1F1Zy1oWEdVUGVuSnkxUVRWYw==
```

The HTTP BASIC AUTH header's payload is base64 encoded which will decode to string as follows.

```
johndoe:eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJ0cmF2aXMtY2kub3JnIiwic2x1ZyI6InRyYXZpcy1jaS90cmF2aXMtY2kiLCJwdWxsLXJlcXVlc3QiOiIiLCJleHAiOjU0MDAsImlhdCI6MH0.soQJgHR6cGNr9Lj_N6yL2Nk5SQug-hXGUPenJy1QTVc
```

The colon separated string contains the username before the colon and the JWT
token after the colon. The username is used to retrieve the user object from
the user database. Below is a function which is executed against the user
object and the token to validate them for authentication. Please note that the
code is deliberately agnostic to what value the access_key contains. It doesn't
matter whether a JWT token or an access key is passed into the function.
However, service providers will have to add the JWT auth attempt to an already
existing authentication mechanism.

```python
import jwt

def authenticate(user, access_key):
    """
    user: db object representing user retrieved based on username from HTTP BASIC AUTH
    access_key: access key or JWT token signed using access key (shared secret)
    returns True when authenication validation passed, otherwise False
    """
    # primary auth method
    if user['access_key'] == access_key:
        return True

    # secondary auth attempt using JWT method
    try:
        return bool(jwt.decode(access_key, user['access_key']))
    except (jwt.DecodeError, jwt.ExpiredSignature):
        return False
```

## List of Third-Party Services Integrated with the JWT Addon

### Sauce Labs

Add your `SAUCE_USERNAME` as a normal environment variable, and your `SAUCE_ACCESS_KEY` as a JWT token:

```yml
env:
  - SAUCE_USERNAME=example_username
addons:
  jwt:
     saucelabs:
        secure: <SAUCE_ACCESS_KEY ENCRYPTED>
```
