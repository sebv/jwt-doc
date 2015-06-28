# JWT addon

## problem

The integration between Travid-CI and thirdparty services like Sauce Labs relies on [Encrypted Variables](http://docs.travis-ci.com/user/environment-variables/#Encrypted-Variables). This work great when committing to the master branch or for branches managed by the repo commiters because in this case the code being tested can be trusted. However in the case of PRs the code being tested has not yet been reviewed, so there is no way to make sure that the PR contains malicious code exposing the secure variables. Therefore access to secure variable has been disabled for PR, and integration with thirdparty services relying on encrypted variable does not work in the context of PRs.

## solution

The JWT addon provides a solution o this problem by replacing the encrypted variable by a time limited token, so that even if the token is exposed, the consequences are limited. For this to work the JWT addon needs to be enabled in the `.travis.yml` file, and the thirdparty need to have integrated with the JWT service and allow token based authentication.

## overview schema

<img src="http://sebv.github.io/jwt-doc/travis_jwt.svg">

## JWT client configuration

### encrypt shared secret

Please refer to the [encryption key doc](http://docs.travis-ci.com/user/encryption-keys/).

You need to encrypt the shared secret as indicated by the thirdparty service provider, and
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

### use token within test code

The variable names used during the encryption stage will be available as environments variables within the travis job. However these environment variables will contain the JWT tokens instead of the original value. Use those environment variable to authenticate with the thirdparty services.

For instance, using the configuration from the sections above, available variables will be `SAUCE_ACCESS_KEY` and `THIRDPARTY_SHARED_SECRET`.


## thirdparty service integration

Thirdparty service need to implement a new authentication method on the server side so that the JWT token is recognized and verified.

Below is an example of how it is done in python:

@sourishkrout can you help with the hmac description and code sample?


## list of thirdparty integrated with the JWT addon

### sauce labs

All you need to do is pass the JWT token as your access key. In most case all you need do is to configure the SAUCE_ACCESS_KEY so that it contains the JWT token.

