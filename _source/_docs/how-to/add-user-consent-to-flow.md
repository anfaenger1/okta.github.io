---
layout: docs_page
title: Add User Consent to Your Authentication Flow
excerpt: Add a user consent to your authentication or authorization flow
---
# Add Okta's User Consent Dialog to Your Authentication Flow

{% api_lifecycle ea %}

## User Consent in Okta

A consent represents a user’s explicit permission to allow an application to access resources protected by scopes. As part of an OAuth 2.0 or OpenID Connect authentication flow, you can prompt the user with a page to approve your app’s access to specified resources.

Consent grants are different from tokens because a consent can outlast a token, and there can be multiple tokens with varying sets of scopes derived from a single consent. When an application comes back and needs to get a new access token, it may not need to prompt the user for consent if they have already consented to the specified scopes. Consent grants remain valid until the user manually revokes them, or until the user, application, authorization server or scope is deactivated or deleted.

## How to Require Users to Consent to Access

To ensure that a user acknowledges and agrees to give an app access to their data,
configure Okta to require that consent be given. Okta hosts the consent dialog and populates it with your logo and details about what's being agreed to:

{% img user-consent-howto.png alt:user-consent-dialog %}

To configure an authorization or authentication flow to include user consent, configure settings in two places:

* The app that displays the user consent page
* At least one scope sent in the authentication or authorization request

Then you'll send the appropriate values for `prompt` as part of the request.

Use the following procedure to display the user consent dialog as part of an OpenID Connect or OAuth 2.0 request:

1. Verify that you have the API Access Management feature enabled, and that User Consentis also enabled. If both features are enabled, you'll see a **User Consent** panel in the General tab for any app.

    {% img user-consent-panel.png alt:"User Consent Panel" %}

    To enable these features, {{site.contact_support_lc}}.

2. Add an app via the Apps API. The value you should specify for `consent_method` depends on the values for `prompt` and `consent`. Check the Apps API [table of values](https://developer.okta.com/docs/api/resources/apps#add-oauth-20-client-application) for these three properties. In most cases, `REQUIRED` is the correct value.

    Optionally, you can set the appropriate values for your Terms of Service (`tos_uri`) and Privacy Policy (`policy_uri`) notices using the same API request.

    Note: You can also create and configure an app in the administrator UI by navigating to **Applications > Add Application**.

3. [Enable consent](/docs/api/resources/authorization-servers#create-a-scope) for the scopes that require consent. To do this, set the `consent` property to `REQUIRED`.

    Note: You can also specify these values when you create and configure a scope in the administrator UI. Navigate to **Applications > [Application Name] > General > User Consent** and select **Require user consent for this scope** (it can be overriden by individual apps). 

4. Prepare an authentication or authorization request with the correct values for `prompt` and `consent_method`. For details, see the [API documentation for `prompt`](/docs/api/resources/oidc#parameter-details) and the [table of values relating to consent dialog](/docs/api/resources/apps#settings-7).

5. Test your configuration by sending an authentication or authorization request. For example, to require consent for the default scope `email` and `openid`:

```json
curl -v -X GET \
-H "Accept: application/json" \
-H "Content-Type: application/json" \
-H "Authorization: SSWS ${api_token} \
"https://${yourOktaDomain}.com/oauth2/${authenticationServerId}/v1/authorize?client_id=${clientId}&response_type=token&response_mode=fragment&scope=email%20openid&redirect_uri=http://localhost:54321&state=myState&nonce=${nonce}"
```
Your test should launch the user consent dialog. Click **Allow** to create the grant.

## Verification

If you want to verify that you've successfully created a user grant, here are a few ways:

* Check the ID token payload if you requested an ID token. The payload should contain the requested scopes.

    ```
    {
      "ver": 1,
      "jti": "AT.HVDRVIwbyq2jmP0SeNjyvMKq5zsMvUNJlzvXH5rfvOA",
      "iss": "https://${yourOktaDomain}.com/oauth2/default",
      "aud": "Test",
      "iat": 1524520458,
      "exp": 1524524058,
      "cid": "xfnIflwIn2TkbpNBs6JQ",
      "uid": "00u5t60iloOHN9pBi0h7",
      "scp": [
        "email",
        "openid"
      ],
      "sub": "saml.jackson@stark.com"
    }
    ```

* Check the access token if you requested one. The payload should contain:

    ```
    {
      "sub": "00u5t60iloOHN9pBi0h7",
      "email": "saml.jackson@stark.com",
      "ver": 1,
      "iss": "https://${yourOktaDomain}.com/oauth2/${authenticationServerId}",
      "aud": "xfnIflwIn2TkbpNBs6JQ",
      "iat": 1524520458,
      "exp": 1524524058,
      "jti": "ID.lcnZvvp-5PQYlTNEfv3Adq6PotmYBcl1D1S-zErEhuk",
      "amr": [
        "pwd"
      ],
      "nonce": "nonce",
      "auth_time": 1000,
      "at_hash": "preview_at_hash"
    }
    ```

* You can verify that a grant was created by listing the grants:

    ```json
    curl -v -X GET \
    -H "Accept: application/json" \
    -H "Content-Type: application/json" \
    -H "Authorization: SSWS ${api_token} \
    "https://${yourOktaDomain}.com/api/v1/users/${userId}/grants"
    ```

    The response should contain the grant you created when you clicked **Allow** in the previous step.

    ```json
    [
        {
            "id": "oag4xfx62r6S53kHr0h6",
            "status": "ACTIVE",
            "created": "2018-04-23T21:53:25.000Z",
            "lastUpdated": "2018-04-23T21:53:25.000Z",
            "issuerId": "auscl1o4tnf48w5Wt0h7",
            "issuer": null,
            "clientId": "xfnIflwIn2TkbpNBs6JQ",
            "userId": "00u5t60iloOHN9pBi0h7",
            "scopeId": "scpcl1o4toFjganq10h7",
            "_links": {
                "app": {
                    "href": "https://${yourOktaDomain}.com/api/v1/apps/0oaaggpxeqxTDuP780h7",
                    "title": "Acme OIDC Client"
                },
                "authorizationServer": {
                    "href": "https://${yourOktaDomain}.com/api/v1/authorizationServers/auscl1o4tnf48w5Wt0h7",
                    "title": "My Authorization Server"
                },
                "scope": {
                    "href": "https://${yourOktaDomain}.com/api/v1/authorizationServers/auscl1o4tnf48w5Wt0h7/scopes/scpcl1o4toFjganq10h7",
                    "title": "openid"
                },
                "self": {
                    "href": "https://${yourOktaDomain}.com/api/v1/users/00u5t60iloOHN9pBi0h7/grants/oag4xfx62r6S53kHr0h6"
                },
                "revoke": {
                    "href": "https://${yourOktaDomain}.com/api/v1/users/00u5t60iloOHN9pBi0h7/grants/oag4xfx62r6S53kHr0h6",
                    "hints": {
                        "allow": [
                            "DELETE"
                        ]
                    }
                },
                "client": {
                    "href": "https://${yourOktaDomain}.com/oauth2/v1/clients/xfnIflwIn2TkbpNBs6JQ",
                    "title": "Acme OIDC Client"
                },
                "user": {
                    "href": "https://${yourOktaDomain}.com/api/v1/users/00u5t60iloOHN9pBi0h7",
                    "title": "Saml Jackson "
                },
                "issuer": {
                    "href": "https://${yourOktaDomain}.com/api/v1/authorizationServers/auscl1o4tnf48w5Wt0h7",
                    "title": "My Authentication Server"
                }
            }
        }
    ]
    ```

## Troubleshooting

If you don't see the consent prompt when expected:

* Verify that you haven't already provided consent for that combination of app and scope(s). Use the [`/grants` endpoint](/docs/api/resources/users#list-grants) to see which grants have been given, and to revoke grants.
* Check the settings for `prompt`, `consent`, and `consent_method` in the [Apps API table of values](https://developer.okta.com/docs/api/resources/apps#add-oauth-20-client-application).
* Make sure that in your app configuration, the `redirect_uri` is an absolute URI and that it is whitelisted by specifying in Trusted Origins.
* If you aren't using the `default` authorization server, check that you've created at least one policy with one rule that applies to any scope or the scope(s) in your test.

> Hint: You can use the Token Preview tab in your custom authorization server to test configuration values before sending an authorize request.