# JWT addon

## problem

The integration between Travid-CI and thirdparty services like Sauce Labs relies on [Encrypted Variables](http://docs.travis-ci.com/user/environment-variables/#Encrypted-Variables). This work great when committing to the master branch or for branches managed by the repo commiters because in this case the code being tested can be trusted. However in the case of PRs the code being tested has not yet been reviewed, so there is no way to make sure that the PR contains malicious code exposing the secure variables. Therefore access to secure variable has been disabled for PR, and integration with thirdparty services relying on encrypted variable does not work in the context of PRs.

## solution

The JWT addon provides a solution o this problem by replacing the encrypted variable by a time limited token, so that even if the token is exposed, the consequences are limited. For this to work the JWT addon needs to be enabled in the `.travis.yml` file, and the thirdparty need to have integrated with the JWT service and allow token based authentication.

## overview schema

![Alt text](http://sebv.github.io/jwt-doc/travis_jwt.svg)
<img src="http://sebv.github.io/jwt-doc/travis_jwt.svg">

## JWT client configuration

### encrypt shared secret

### .travis.yml

### use token within test code

## Thirdparty service integration

