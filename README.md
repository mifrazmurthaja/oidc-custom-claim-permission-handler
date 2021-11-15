# oidc-custom-claim-permission-handler

This entension to add an additional claim with allowed permissions of the user to the OIDC flows of WSO2 Identity Server.

### Steps to deploy

1. Build the component using maven
2. Copy the `org.wso2.permission.claim.handler-1.0.jar` from target directory into `<IS_HOME>/repository/components/dropins/`
3. Add the following into deployment.toml
```
[oauth.oidc.extensions]
claim_callback_handler="org.wso2.custom.claim.PermissionClaimHandler"
```
4. Restart WSO2 IS

### Steps to configure

1. Create a new Local Claim to represent the permissions (`Claims-> Add-> Add Local Claim`).  Decide on which level you need the permissions to be returned and create the claim considering the structure. For example, if all the application permissions need to be returned, the Claim URI should be `http://wso2.org/claims/permission/applications`. If permissions of a specific application(eg. POS) need to be returned, the Claim URI should be `http://wso2.org/claims/permission/applications/POS`
2. Create an External Claim (`Claims-> Add-> Add External Claim`) with `http://wso2.org/oidc/claim` as Dialect URI, provide a name for the `External Claim URI` (eg. `posPermissions`) and for the `Mapped Local Claim` select the local claim URI created in step 4.
3. Navigate to `OIDC Scopes-> List`.
4. Locate the `openid` scope and click on `Add Claims`
5. Click on `Add OIDC Claim` and select the claim that you have created from the dropdown and then click on `Add`.
6. Now we can create a service provider for your application. Click on `Add` under `Service Providers` and Register the Service Provider after providing a name.
7. Expand `Claim Configuration` and add the created claim at step 4 to Requested Claims of Service provider.
8. In order to create permissions specific to an application, expand the `Role/Permission` Configuration section, and then expand the `Permissions`.
9. Click `Add Permission` and specify the service provider specific permission that you want to add. These will be added to the permission tree under `Applications`.
10. Expand the `Inbound Authentication Configuration` section and then expand `OAuth/OpenID Connect Configuration`. Click Configure.
11. Fill the callback URL of the application and click on `Add`.
12. `OAuth Client Key` and `OAuth Client Secret` will be generated. Copy those values.
13. Make sure your user has been assigned with the roles having the required application permissions.
14. Make an authorization request to the Identity Server by executing the below URL in the browser. The user will be authenticated and the authorization code will be returned as a parameter to the configured call back URL of the Service Provider. Replace the place holders with the appropriate values
    Request:
    ```
    https://<HostName of Identity Server>:9443/oauth2/authorize?scope=openid&response_type=code&redirect_uri=<Call back URL>&client_id=<OAuth Client Key>
    ```
    Sample Response:
    ```
    <Call back URL>?code=78b0d75d-0003-3bec-aa2b-c36d42c373cc&session_state=2e83b1ef15be924209b346c807a1b82b94d44a146a9f16b11b0fa2e44974c4ad.AtSUKDxIlwkDFjfYqtINsA
    ```
15. Copy the value of `code` and send the token request by executing the below command in the terminal after replacing the placeholders.
    ```
    curl -v -X POST --basic -u <OAuth Client Key>:<OAuth Client Secret> -H 'Content-Type: application/x-www-form-urlencoded;charset=UTF-8' -k -d 'grant_type=authorization_code&redirect_uri=<Callback URL>&code=<code value from step 17>' https://<Host Name>:9443/oauth2/token
    ```
16. The response will contain the JWT access_token, id_token and other metadata. Copy the value of 'access_token' from the response and parse it to see the permissions in the JSON format. The payload will look like,
    ```
    {
      "sub": "abi",
      "aud": "ffdRZajk74dbsWfW6kt1Xhd6s8Ya",
      "nbf": 1594155819,
      "azp": "ffdRZajk74dbsWfW6kt1Xhd6s8Ya",
      "scope": "openid",
      "iss": "https://localhost:9443/oauth2/token",
      "permission": [
        "/permission/applications/POS/Edit",
        "/permission/applications/POS/View"
      ],
      "exp": 1594159419,
      "iat": 1594155819,
      "jti": "99714d7d-662f-4958-b65a-7809304dcbd3"
    }
    ```

**Credits**
@nilasini and @Abilashini
