<a name="#top">Back to Top</a>

# Email Verification

## Feature Description

This document describes the endpoints and logic that must exist in order to
facilitate self-service verification of newly registered user accounts.

If the application's default account store has the email verification workflow
enabled, and `stormpath.web.verifyEmail.enabled` is not set to `false`, our library
MUST intercept incoming requests at `stormpath.web.verifyEmail.uri` and follow the
request handling procedure that is defined below.

## Request Handling

#### GET Requests That Prefer `text/html`

* If there is a `?sptoken` query parameter in the URL:

 * Attempt to consume the `sptoken` by posting it to the tenant collection, `/v1/accounts/emailVerificationTokens/:sptoken`

 * If the token is valid and `autoLogin` is enabled, follow the standard [post login logic](login.md#-post-response-handling) (like setting cookies, and redirecting the user)

 * If the token is valid and `autoLogin` is disabled, redirect to `stormpath.web.verifyEmail.nextUri`

 * If the token is invalid, render a form that allows them to request a new link
   by submitting their email address.  The form should show the error:

    > This verification link is no longer valid. Please request a new link from
      the form below.

* If there isn't a `?sptoken` query parameter in the URL:

 * Render a form that allows them to request a new link by submitting their
   email address.

#### GET Requests That Prefer `application/json`

* If there is a `?sptoken` query parameter in the URL:

 * Attempt to consume the `sptoken` by posting it to the tenant collection, `/v1/accounts/emailVerificationTokens/:sptoken`

  * If the token is valid and `autoLogin` is enabled on the registration route, follow the standard [post login logic](login.md#-post-response-handling) (like setting cookies, and responding with the account object)

  * If the token is valid and `autoLogin` is disabled on the registration route, respond with `200 OK` and an empty body

  * If validation fails, respond with the JSON error from the API, according to
    the [Error Handling][] specification.

* If there is't an `?sptoken` query parameter in the URL, respond with this
  error:

  ```javascript
  {
    status: 400,
    message: 'sptoken parameter not provided.'
  }
  ```

#### POST Requests

This endpoint accepts post requests from the form which allows you to request
a new link by entering your email address or username.  The endpoint should parse the POST
body as `application/json` and `application/x-www-form-urlencoded`.

The format of the request is (JSON example):

```javascript
{
  "login": "foo@bar.com"
}
```

This data should be posed to the `/v1/applications/:id/verificationEmails` endpoint.

Regardless of whether or not the email address is associated with a user
account, we should respond according to the request preference:

  * `text/html`, redirect to `stormpath.web.login.uri` and
    append `?status=unverified`

  * `application/json`, the status of the response should be `200 OK` with no
     body.

## <a name="Options"></a> Options

```yaml
stormpath:
  web:
    # Unless verifyEmail.enabled is specifically set to false, the email
    # verification feature must be automatically enabled if the default account
    # store for the defined Stormpath application has the email verification
    # workflow enabled.
    verifyEmail:
      enabled: null
      uri: "/verify"
      nextUri: "/login?status=verified"
      view: "verify"
```


#### enabled

Default: `null`

Unless explicitly set to false, the email verification feature must be
automatically enabled if the default account store for the defined Stormpath
application has the email verification workflow enabled.

<a href="#top">Back to Top</a>

#### autoLogin

Default: `false`

Defined in: `stormpath.web.register.autoLogin`

If `true`, then once a user has successfully verified their email, they application will run the standard [post login logic](login.md#-post-response-handling) (like setting cookies), and respond with the same data as the `login` endpoint. 

#### uri

Default: `/verify`

The URI that we'll attach an interceptor to for GET and POST requests, if
`enabled` is `true`.

<a href="#top">Back to Top</a>


#### nextUri

Default: `/login`

Where to send the user after successful verification.

<a href="#top">Back to Top</a>

#### view

Default: `verify`

A string key which identifies the view template that should be used.  The
default value may look different for your framework.  The point of this value
is to allow the developer to override our default view with their own.

[Error Handling]: error-handling.md
