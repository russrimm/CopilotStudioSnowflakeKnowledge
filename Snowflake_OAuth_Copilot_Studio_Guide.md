# Step-by-Step Guide to Add Snowflake as a Knowledge Source in Microsoft Copilot Studio Using OAuth Authentication

This guide outlines the steps to integrate Snowflake as a knowledge source in Microsoft Copilot Studio using OAuth authentication via Microsoft Entra ID. This enables your custom agents (copilots) to query Snowflake data in real-time using natural language, without writing SQL.

## Prerequisites
- Snowflake account with appropriate roles (e.g., ACCOUNTADMIN for setup).
- Microsoft Entra ID tenant.
- Premium Power Platform license (required for Snowflake connector).
- Access to Microsoft Copilot Studio (part of Microsoft Power Platform).
- Avoid using high-privilege Snowflake roles like ACCOUNTADMIN for the OAuth client; use a custom role with necessary privileges (e.g., USAGE on warehouse, database, schema, and SELECT on tables).

## Phase 1: Register Apps in Microsoft Entra ID
You need two app registrations: one for the OAuth Resource (Snowflake) and one for the OAuth Client (Power Platform/Copilot Studio). This enables external OAuth with Entra ID as the authorization server.

### Option 1: Manual Registration (Recommended for Understanding)

1. **Create the OAuth Resource App (for Snowflake):**
   - Sign in to the [Microsoft Azure Portal](https://portal.azure.com).
   - Navigate to **Microsoft Entra ID > App registrations > New registration**.
   - Enter a name (e.g., "Snowflake OAuth Resource").
   - Set **Supported account types** to **Accounts in this organizational directory only (Single tenant)**.
   - Click **Register**.
   - In the app's **Overview**, note the **Application (client) ID** as `<RESOURCE_APP_ID>`.
   - Go to **Expose an API**.
   - Click **Set** next to **Application ID URI** and set it to a unique URI (e.g., `api://<your-tenant-id>/<RESOURCE_APP_ID>` or `https://your-domain.com/<RESOURCE_APP_ID>`). Note this as `<SNOWFLAKE_APPLICATION_ID_URI>`.
   - For **Service Principal Auth** (recommended for Copilot Studio, as it uses client credentials flow):
     - Go to **App roles > Create app role**.
     - Set **Allowed member types** to **Applications**.
     - Enter a **Display name** (e.g., "Analyst") and **Description** (e.g., "Access Snowflake as analyst role").
     - Set **Value** to `session:role:<snowflake-role>` (e.g., `session:role:analyst` or `session:role-any` for any role).
     - Enable the role and click **Apply**.
   - (Optional) For **Delegated Auth** (on behalf of a user), add a scope under **Expose an API > Add a scope** (e.g., `session:scope:analyst`).
   - Go to **Manifest** and ensure `accessTokenAcceptedVersion` is set to `2` (for v2 tokens). Save changes.

2. **Create the OAuth Client App (for Power Platform):**
   - In **App registrations > New registration**.
   - Enter a name (e.g., "Snowflake OAuth Client").
   - Set **Supported account types** to **Single tenant**.
   - Click **Register**.
   - In **Overview**, note the **Application (client) ID** as `<OAUTH_CLIENT_ID>`.
   - Go to **Certificates & secrets > New client secret**.
   - Add a description, set expiration (e.g., never for testing; use shorter in production), and note the **Value** as `<OAUTH_CLIENT_SECRET>`.
   - Go to **API permissions > Add a permission > My APIs**.
   - Select the "Snowflake OAuth Resource" app.
   - For Service Principal: Choose **Application permissions**, select the app role (e.g., "analyst"), and click **Add permissions**.
   - For Delegated: Choose **Delegated permissions**, select the scope.
   - Click **Grant admin consent for [your tenant]** and confirm.

3. **Collect Entra ID Endpoints:**
   - In the Resource app's **Endpoints** tab, copy:
     - **OAuth 2.0 token endpoint (v2)** as `<AZURE_AD_OAUTH_TOKEN_ENDPOINT>` (e.g., `https://login.microsoftonline.com/<tenant_id>/oauth2/v2.0/token`).
     - Open the **OpenID Connect metadata document** URL, and from the JSON, note `jwks_uri` as `<AZURE_AD_JWS_KEY_ENDPOINT>` (e.g., `https://login.microsoftonline.com/<tenant_id>/discovery/v2.0/keys`).
     - From the **Federation metadata document**, note the `entityID` as `<AZURE_AD_ISSUER>` (e.g., `https://sts.windows.net/<tenant_id>/`).

### Option 2: Automated Registration Using PowerShell
- Use the PowerShell script from [Snowflake's quickstart](https://docs.snowflake.com/en/developer-guide/external-oauth/azure-oauth-quickstart).
- Download or copy the script, replace `<tenant_id>`, run it in Azure Cloud Shell, and follow prompts.
- It generates a `snowflakeinfo.txt` file with all values and SQL for the next phase.

## Phase 2: Configure Security Integration in Snowflake
1. Log in to Snowflake (e.g., via Snowsight or SnowSQL) as ACCOUNTADMIN.
2. Run the SQL command to create the security integration (use values from Entra ID or the script's output file):

```sql
CREATE SECURITY INTEGRATION <integration_name>  -- e.g., POWER_PLATFORM_OAUTH
  TYPE = EXTERNAL_OAUTH
  ENABLED = TRUE
  EXTERNAL_OAUTH_TYPE = AZURE
  EXTERNAL_OAUTH_ISSUER = '<AZURE_AD_ISSUER>'
  EXTERNAL_OAUTH_TOKEN_USER_MAPPING_CLAIM = 'upn'  -- Or 'sub' for service principal
  EXTERNAL_OAUTH_SNOWFLAKE_USER_MAPPING_ATTRIBUTE = 'login_name'
  EXTERNAL_OAUTH_JWS_KEYS_URL = '<AZURE_AD_JWS_KEY_ENDPOINT>'
  EXTERNAL_OAUTH_AUDIENCE_LIST = ('<SNOWFLAKE_APPLICATION_ID_URI>')
  EXTERNAL_OAUTH_SCOPE_MAPPING_ATTRIBUTE = 'scope'  -- Or 'roles' for app roles
;
```

   - For service principal, map the claim to 'sub' if using client credentials.
   - Grant privileges to the role (e.g., `GRANT USAGE ON WAREHOUSE <warehouse> TO ROLE <role>;`).
3. Verify with `DESC SECURITY INTEGRATION <integration_name>;`.

## Phase 3: Create a Connection in Power Platform
1. Go to [make.powerapps.com](https://make.powerapps.com) and sign in.
2. Navigate to **Connections > New connection**.
3. Search for and select the **Snowflake** connector (premium).
4. Choose authentication type: **OAuth** (Service Principal for client credentials or Delegated for user-based).
5. Enter:
   - **Account URL**: Your Snowflake account URL (e.g., `<org>-<account>.snowflakecomputing.com`).
   - **Client ID**: `<OAUTH_CLIENT_ID>`.
   - **Client Secret**: `<OAUTH_CLIENT_SECRET>`.
   - **Tenant ID**: Your Entra ID tenant ID.
   - **Resource**: `<SNOWFLAKE_APPLICATION_ID_URI>`.
   - **Warehouse**, **Role**, **Database**, and **Schema** (case-sensitive).
6. Click **Create** and test the connection.

## Phase 4: Add Snowflake as a Knowledge Source in Copilot Studio
1. Go to [copilotstudio.microsoft.com](https://copilotstudio.microsoft.com) and sign in.
2. Create a new copilot or edit an existing one.
3. Navigate to **Knowledge > Add knowledge** (or in the agent creation wizard, select knowledge sources).
4. Select **Real-time connectors** (preview feature) or **Snowflake** if directly available.
5. Choose the Snowflake connection created in Phase 3.
6. Select specific tables or views from your Snowflake database/schema to include as knowledge.
7. Configure any filters or joins if prompted (Copilot Studio handles SQL generation automatically).
8. Test the copilot by asking questions over the data (e.g., "Summarize sales data from Snowflake").
9. Publish the copilot.

## Troubleshooting Tips
- Ensure case sensitivity in Snowflake details.
- If using Cortex Agents (Snowflake's AI), integrate via agent-to-agent calls in Copilot Studio topics.
- For errors, check Entra ID audit logs or Snowflake query history.
- Data stays in Snowflake; no export occurs.

## Notes
- This setup ensures secure, OAuth-based access.
- For production, rotate secrets regularly and limit consents.
- Some features (like direct Snowflake knowledge integration) are in preview as of 2025 and may evolve.