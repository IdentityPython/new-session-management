======================
Token exchange example
======================

The example taken from RFC8693.

The initialization is the normal, that is the authentication at the OP results
in a Grant being initialized and stored under the session_id.

The new thing here is that at the same time an ExchangeGrant is created and
stored along side the authorization grant and a subordinate in the
ClientSessionInfo.

.. code-block:: Python

    session_info = session_manager.get_session_info(session_id)
    exchange_grant = ExchangeGrant(scope=["api"],
                                   resources=["https://backend.example.com"],
                                   users=["https://frontend.example.com"])

    # the grant is assigned to a session (user_id, client_id)
    session_manager.set(
        [session_info["user_id"], session_info["client_id"], exchange_grant.id],
        exchange_grant)


After this the authorization grant issues a *code* and an *access_token*.
Now we are ready for the initial resource request::

    GET /resource HTTP/1.1
        Host: frontend.example.com
        Authorization: Bearer accVkjcJyb4BWCxGsndESCJQbdFMogUC5PbRDqceLTC

The resource server turns around and does a token exchange request with
the access token it just has received::

    POST /as/token.oauth2 HTTP/1.1
    Host: as.example.com
    Authorization: Basic cnMwODpsb25nLXNlY3VyZS1yYW5kb20tc2VjcmV0
    Content-Type: application/x-www-form-urlencoded

    grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Atoken-exchange
    &resource=https%3A%2F%2Fbackend.example.com%2Fapi
    &subject_token=accVkjcJyb4BWCxGsndESCJQbdFMogUC5PbRDqceLTC
    &subject_token_type=
     urn%3Aietf%3Aparams%3Aoauth%3Atoken-type%3Aaccess_token

The AS acting as a token exchange server finds the ExchangeGrant that
gives the resource server the permission to do a token exchange.

.. code-block:: Python

    exch_grant = self.session_manager.find_exchange_grant(subject_token,
                                                          resource_server)

If there is such an exchange grant the AS can now mint a new access token

.. code-block:: Python

    session_info = session_manager.get_session_info_by_token(subject_token)
    _token = session_manager.find_token(session_info["session_id"],
                                        subject_token)

    # create a session ID that points to the exchange grant
    session_id = session_key(session_info['user_id'],
                             session_info["client_id"],
                             exch_grant.id)

    at_handler = session_manager.token_handler["access_token"]
    _token = grant.mint_token(
        'access_token',
        value=at_handler(session_id),
        expires_at=time_sans_frac() + at_handler.lifetime,
        based_on=subject_token,
        resources=["https://backend.example.com/api"]
    )

And then return the response with the new access token (_token)

The response::

    HTTP/1.1 200 OK
        Content-Type: application/json
        Cache-Control: no-cache, no-store

        {
         "access_token":"eyJhbGciOiJFUzI1NiIsImtpZCI6IjllciJ9.eyJhdWQiOiJo
           dHRwczovL2JhY2tlbmQuZXhhbXBsZS5jb20iLCJpc3MiOiJodHRwczovL2FzLmV
           4YW1wbGUuY29tIiwiZXhwIjoxNDQxOTE3NTkzLCJpYXQiOjE0NDE5MTc1MzMsIn
           N1YiI6ImJkY0BleGFtcGxlLmNvbSIsInNjb3BlIjoiYXBpIn0.40y3ZgQedw6rx
           f59WlwHDD9jryFOr0_Wh3CGozQBihNBhnXEQgU85AI9x3KmsPottVMLPIWvmDCM
           y5-kdXjwhw",
         "issued_token_type":
             "urn:ietf:params:oauth:token-type:access_token",
         "token_type":"Bearer",
         "expires_in":60
        }

The backend can then do a protected resource request::

    GET /api HTTP/1.1
        Host: backend.example.com
        Authorization: Bearer eyJhbGciOiJFUzI1NiIsImtpZCI6IjllciJ9.eyJhdWQ
           iOiJodHRwczovL2JhY2tlbmQuZXhhbXBsZS5jb20iLCJpc3MiOiJodHRwczovL2
           FzLmV4YW1wbGUuY29tIiwiZXhwIjoxNDQxOTE3NTkzLCJpYXQiOjE0NDE5MTc1M
           zMsInN1YiI6ImJkY0BleGFtcGxlLmNvbSIsInNjb3BlIjoiYXBpIn0.40y3ZgQe
           dw6rxf59WlwHDD9jryFOr0_Wh3CGozQBihNBhnXEQgU85AI9x3KmsPottVMLPIW
           vmDCMy5-kdXjwhw

