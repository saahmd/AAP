Post Deployment Steps for Netsentinel
After setting up the cluster, you need to manually create the following secret:


 {{- if .Values.createAppSecret }}
apiVersion: v1
kind: Secret
metadata:
  name: app-config-secret
  annotations:
    argocd.argoproj.io/compare-options: IgnoreExtraneous
    argocd.argoproj.io/sync-options: Prune=false
  labels:
    app.kubernetes.io/instance: netsentinel-apps
type: Opaque
stringData:
  secret.yaml: |
    models:
      llm:
        token: "{{ .Values.secrets.llm.token }}"
    slack:
      bot_token: "{{ .Values.secrets.slack.bot_token }}"
      signing_secret: "{{ .Values.secrets.slack.signing_secret }}"
{{- end }}


In order to get the llm token, bot_token and signing_secret follow the below procedure:



Steps to create LLM Token

Visit the following URL to access the "Models as a Service" applications admin page: https://maas.apps.prod.rhoai.rh-aiservices-bu.com/admin/applications

Click on "Create new application".


(https://github.com/rh-telco-tigers/NetSentinel/blob/main/docs/images/maas/001-create-new-app.png)


Select a Model From the list of available models, select "Granite-8B-Code-Instruct".

Visit the following URL to access the "Models as a Service" applications admin page: https://maas.apps.prod.rhoai.rh-aiservices-bu.com/admin/applications

Select a Model From the list of available models, select "Granite-8B-Code-Instruct".
              

Fill Out the Application Form 
Provide the following details: Name: netsentinel-demo Description: LLM endpoints for netsentinel agent


Retrieve API Information After creating the application, you'll be presented with the following details: 

URL:https://granite-8b-code-instruct-maas-apicast-production.apps.prod.rhoai.rh-aiservices-bu.com:443
Model Name: granite-8b-code-instruct-128k 
API Key: (e.g., ••••••••••••••••••••••••••••••••••••••••)

           

Copy the API key and update it in the app-config-secret.yaml file


Configure Slack to get “bot_token” and “signing_secret”

Create a new Slack account or use an existing one. Ensure you have admin permissions to configure the required settings. You can get started at: https://slack.com/get-started#/createnew.

2.1 Once the slack account is setup navigate to https://api.slack.com/apps.
	
	

2.2 Click on Create New App -> Select From a manifest
		            

2.3 Choose your workspace where you want to create this app
                                   

2.4 Use the following manifest, replace the request_url using netsentinel routes. This requires verified SSL certificates (self-signed certs won’t work).

display_information:
  name: NetSentinelRHDev
features:
  bot_user:
    display_name: NetSentinelRHDev
    always_online: false
oauth_config:
  scopes:
    user:
      - channels:history
      - chat:write
    bot:
      - app_mentions:read
      - channels:history
      - channels:join
      - channels:read
      - chat:write
      - conversations.connect:manage
      - conversations.connect:read
      - conversations.connect:write
      - groups:history
      - links:read
settings:
  event_subscriptions:
    request_url: https://<REPLACE-ME-WITH-OCP-ROUTES>/slack/events
    bot_events:
      - app_mention
      - link_shared
      - message.channels
      - message.groups
  org_deploy_enabled: false
  socket_mode_enabled: false
  token_rotation_enabled: false

**Note: Get the route using the below command

oc get routes -n netsentinel netsentinel-route



2.5) Next, you will land at the "Basic Information" page. Copy the “Signing Secret” from here and update it in the app-config-secret.yaml.



2.6 Navigate to OAuth & Permissions and click Install to NetSentinel under the "OAuth Tokens" section



2.7 Click Allow and return to the OAuth & Permissions page to copy the "Bot User OAuth Token and update the value in app-config-secret.yaml. 


2.8 Navigate to slack "Event Subscriptions".
Click on "Retry" if "Request URL" is not verified. Make sure the NetSentinel app is fully up and running and you can hit this endpoint in the browser.


2.9 Your application is now configured and installed in your Slack workspace. Go to https://yourslack.slack.com/ and confirm that the NetSentinelRHDev app appears under the Apps section.



2.10 Create a new channel (e.g., #demo-channel). The channel name can be anything; it doesn't have to match the #netsentinel name used in app-config.yaml.

Add NetSentinelRHDev to this new channel.

