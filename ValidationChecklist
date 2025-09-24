# Checklist for Troubleshooting Snowflake Connection Failure in Microsoft Copilot Studio with OAuth Authentication

This checklist helps diagnose and resolve issues when the Snowflake connection fails in Microsoft Copilot Studio using OAuth authentication via Microsoft Entra ID. Use it to systematically verify configurations and identify the root cause.

- [ ] **Verify Entra ID App Registration Details**
  - [ ] Confirm the **Application (client) ID** for both the OAuth Resource (`<RESOURCE_APP_ID>`) and OAuth Client (`<OAUTH_CLIENT_ID>`) apps matches the values used in Snowflake and Power Platform.
  - [ ] Ensure the **Application ID URI** (`<SNOWFLAKE_APPLICATION_ID_URI>`) in the Resource app is correctly set and matches the `EXTERNAL_OAUTH_AUDIENCE_LIST` in Snowflake's security integration.
  - [ ] Check that the OAuth Client app has the correct **API permissions**:
    - For Service Principal: Verify the app role (e.g., `session:role:analyst`) is assigned.
    - For Delegated: Verify the scope is assigned.
  - [ ] Confirm **Admin consent** has been granted for the OAuth Client app in Entra ID.
  - [ ] Verify the **Client Secret** (`<OAUTH_CLIENT_SECRET>`) is correct, not expired, and properly copied (no extra spaces or characters).
  - [ ] In the Resource app’s **Manifest**, ensure `accessTokenAcceptedVersion` is set to `2`.

- [ ] **Validate Entra ID Endpoints**
  - [ ] Confirm the **OAuth 2.0 token endpoint (v2)** (`<AZURE_AD_OAUTH_TOKEN_ENDPOINT>`) is correct (e.g., `https://login.microsoftonline.com/<tenant_id>/oauth2/v2.0/token`).
  - [ ] Verify the **JWS keys URL** (`<AZURE_AD_JWS_KEY_ENDPOINT>`) is accurate in Snowflake’s security integration (e.g., `https://login.microsoftonline.com/<tenant_id>/discovery/v2.0/keys`).
  - [ ] Ensure the **Issuer** (`<AZURE_AD_ISSUER>`) matches the `entityID` from the Entra ID Federation metadata document (e.g., `https://sts.windows.net/<tenant_id>/`).

- [ ] **Check Snowflake Security Integration**
  - [ ] Run `DESC SECURITY INTEGRATION <integration_name>;` to verify all parameters:
    - `ENABLED = TRUE`.
    - `EXTERNAL_OAUTH_TYPE = AZURE`.
    - `EXTERNAL_OAUTH_ISSUER`, `EXTERNAL_OAUTH_JWS_KEYS_URL`, and `EXTERNAL_OAUTH_AUDIENCE_LIST` match Entra ID values.
    - `EXTERNAL_OAUTH_TOKEN_USER_MAPPING_CLAIM` is set to `'sub'` for Service Principal or `'upn'` for Delegated auth.
    - `EXTERNAL_OAUTH_SCOPE_MAPPING_ATTRIBUTE` is set to `'scope'` or `'roles'` as appropriate.
  - [ ] Ensure the Snowflake role specified in the app role (e.g., `session:role:analyst`) exists and has necessary privileges (e.g., `USAGE` on warehouse, database, schema, and `SELECT` on tables).
  - [ ] Verify the Snowflake user’s `login_name` matches the Entra ID user’s `upn` (for Delegated auth) or the service principal’s `sub` (for Service Principal auth).

- [ ] **Validate Power Platform Connection**
  - [ ] In [make.powerapps.com](https://make.powerapps.com), go to **Connections** and verify the Snowflake connection status is **Connected**.
  - [ ] Confirm the **Account URL** is correct (e.g., `<org>-<account>.snowflakecomputing.com`) and accessible.
  - [ ] Check that **Client ID**, **Client Secret**, **Tenant ID**, **Resource**, **Warehouse**, **Role**, **Database**, and **Schema** are correctly entered and case-sensitive.
  - [ ] Test the connection in Power Platform to ensure it authenticates successfully.
  - [ ] Ensure the user creating the connection has a premium Power Platform license.

- [ ] **Test in Copilot Studio**
  - [ ] In [copilotstudio.microsoft.com](https://copilotstudio.microsoft.com), verify the Snowflake connection is selected under **Knowledge > Real-time connectors** or **Snowflake**.
  - [ ] Confirm the correct tables/views are selected as knowledge sources.
  - [ ] Test the copilot with a simple query (e.g., “Show data from [table]”) and check for errors in the response.
  - [ ] Ensure the copilot has access to the connection (check sharing/permissions if applicable).

- [ ] **Check Logs and Error Messages**
  - [ ] In Entra ID, review **Audit logs** and **Sign-in logs** for the OAuth Client app to identify authentication failures (e.g., invalid client ID, secret, or permissions).
  - [ ] In Snowflake, check the **Query History** for failed queries or OAuth-related errors (e.g., invalid token or role issues).
  - [ ] In Copilot Studio, note any specific error messages during testing (e.g., “Connection failed” or “Access denied”) and cross-reference with documentation.

- [ ] **Network and Firewall Issues**
  - [ ] Ensure the Snowflake account URL is accessible from your network (no firewall blocks).
  - [ ] Verify that Entra ID endpoints (`login.microsoftonline.com`) and Snowflake endpoints are not blocked by your organization’s network policies.

- [ ] **Role and Privilege Issues**
  - [ ] Confirm the Snowflake role used in the connection has sufficient privileges:
    - `USAGE` on the warehouse, database, and schema.
    - `SELECT` on the tables/views used as knowledge sources.
  - [ ] Run a test query in Snowflake using the same role to verify access (e.g., `SELECT * FROM <database>.<schema>.<table> LIMIT 1;`).

- [ ] **Miscellaneous**
  - [ ] Ensure case sensitivity for all Snowflake inputs (warehouse, role, database, schema).
  - [ ] If using Snowflake Cortex Agents, verify agent-to-agent integration in Copilot Studio topics.
  - [ ] Check if the Snowflake connector or real-time connectors feature in Copilot Studio is still in preview and if there are known issues (refer to Microsoft documentation).
  - [ ] Verify that the Entra ID tenant and Snowflake account are in the same region (if applicable) to avoid latency or regional restrictions.

## Notes
- If the issue persists, contact Snowflake or Microsoft support with specific error codes and logs.
- For production, ensure secrets are rotated and consents are limited to required roles/users.
- Data remains in Snowflake; no export occurs, so ensure the copilot’s queries align with available data.
