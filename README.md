# JWT addon

## General Problem

The integration between Travis-CI and third-party services like Sauce Labs relies on [encrypted variables](http://docs.travis-ci.com/user/environment-variables/#Encrypted-Variables). This works great when committing to the master branch or for branches managed by the repo's commiters because in this case the code being tested can be trusted. However in the case of PRs the code being tested has not yet been reviewed, so there is no way to make sure that the PR does not contain malicious code exposing the secure variables. As a security measure, access to secure variables have been disabled for PRs, and therefore integration with third-party services relying on encrypted variable does not work in the context of Pull Request.

## Solution

The JWT addon provides a solution o this problem by replacing the encrypted variable by a time-limited token, so that even if the token is exposed, the consequences are limited. For this to work the JWT addon needs to be enabled in the `.travis.yml` file, and the third-party need to have integrated with the JWT service and allow token based authentication.

## Overview Schema

<img src="http://sebv.github.io/jwt-doc/travis_jwt.svg">

## JWT Client Configuration

### Encrypt Shared Secret

Please refer to the [encryption key doc](http://docs.travis-ci.com/user/encryption-keys/).

You need to encrypt the shared secret as indicated by the third-party service provider, and
make sure that the variable name is used by Travis Job script.

For instance:

```
travis encrypt SAUCE_ACCESS_KEY=123456789 # replace with your own access key
```

or more generally, something like:

```
travis encrypt THIRDPARTY_SHARE_SECRET=qwertyuiop1234567
```

### .travis.yml

Next step is to use the encrypted keys generated within the jwt section of the `.travis.yml` file. For instance if you only have one key it will look like:

```yml
addons:
  jwt:
     secure: <SAUCE_ACCESS_KEY ENCRYPTED>
```

It is also possible to configure several services:

```yml
addons:
  jwt:
     saucelabs:
        secure: <SAUCE_ACCESS_KEY ENCRYPTED>
     thirdparty:
        secure: <THIRDPARTY_SHARED_SECRET ENCRYPTED>
```

### Use token within test code

The variable names used during the encryption stage will be available as environments variables within the Travis-CI job. However, these environment variables will contain the JWT tokens instead of the original value. Use those environment variables to authenticate with the third-party services.

For instance, using the configuration from the sections above, available variables will be `SAUCE_ACCESS_KEY` and `THIRDPARTY_SHARED_SECRET`.


## Third-Party Service Integration

Third-party service needs to implement a new authentication method on the server side so that the JWT token is recognized and verified.

### JWT Libraries

In most language JWT compliant libraries are available, making the implementation straightforward:

- python: https://pypi.python.org/pypi/PyJWT/1.3.0
- ruby: https://rubygems.org/gems/jwt
- Check http://jwt.io/ for more

### Payload

Below is the payload used to generate the JWT token:
```
{
  "iss": "travis-ci.org",
  "slug": "<SLUG>",
  "pull-request": "<PR>",
  "exp": <now+5400>,
  "iat": <now>}
}
```

### Code Sample 

#### Python

@sourishkrout can you help with the hmac description and code sample?

## List of Third-Party Services Integrated with the JWT Addon

### Sauce Labs

All you need to do is pass the JWT token as your access key. In most cases, all you need do is to configure the `SAUCE_ACCESS_KEY` so that it contains the JWT token.
