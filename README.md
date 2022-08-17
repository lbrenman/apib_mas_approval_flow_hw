# Axway Marketplace Asset Request and Product Subscription Auto Approver using API Builder

This API Builder project implements the steps described [**here**](https://docs.axway.com/bundle/amplify-central/page/docs/integrate_with_central/webhook/marketplace_subscription_webhook/index.html) to automatically approve Axway Marketplace Product subscription approval requests.

This API Builder project also implements an automatic approval flow for Axway Marketplace asset requests. A user requests access to a asset (resources) in order to retrieve credentials (e.g. API Key) to call the API.

It is intended to be an example of the two main steps required to implement a product subscription workflow, namely respond to webhooks and make API calls to the platform. However, it can be extended to notify the API provisioning team and/or log subscriptions in an auditing system or CRM, for example.

The end user flow for Product Subscription Requests looks like this:

![](https://i.imgur.com/RaRuLHh.png)

![](https://i.imgur.com/gaM3jcN.png)

![](https://i.imgur.com/zupyrCj.png)

![](https://i.imgur.com/BTP6bZ5.png)


The end user flow for Asset Requests is shown below:

![](https://i.imgur.com/ErQA2ud.png)

![](https://i.imgur.com/9ylWNXR.png)

![](https://i.imgur.com/4TRQVo0.png)

![](https://i.imgur.com/r3XbeWo.png)

![](https://i.imgur.com/KML7odK.png)


As should be evident in the screen shots above, one needs a Marketplace application and an approved product subscription in order to request access to a product's assets (resources).

The API Builder project exposes one API:

* `POST /api/amplifycentralwebhookhandler` which takes an Amplify webhook as the body. This is the webhook that Amplify calls when a marketplace asset request is made.

You need to set the following environment variables:

```
CLIENT_ID=DOSA_<An Axway service account client ID>
CLIENT_SECRET=<An Axway service account client Secret>
API_KEY=<An API Key that you set>
API_CENTRAL_URL=https://apicentral.axway.com
```



I configured my asset request Webhook as follows:

```
axway central create -f assetrequestapprovalworflowintegration.yaml
```

`assetrequestapprovalworflowintegration.yaml`

```
group: management
apiVersion: v1alpha1
kind: Integration
title: Integrations for Asset Request Approvals
name: integrations-for-asset-request-approvals
spec:
  description: This is a group of resources to be used for asset request approval workflows
---
group: management
apiVersion: v1alpha1
kind: Webhook
title: Webhook Listener for Asset Request Approvals
name: wh-integrations-for-asset-request-approvals
metadata:
  scope:
    kind: Integration
    name: integrations-for-asset-request-approvals
spec:
  enabled: true
  url: <API Builder Base Address>/api/amplifycentralwebhookhandler
  headers:
      Content-Type: application/json
      apikey: <API Key Set as Env Var>
---
group: management
apiVersion: v1alpha1
kind: ResourceHook
title: Resource Hook for Asset Request Approvals
name: rh-integrations-for-asset-request-approvals
metadata:
  scope:
    kind: Integration
    name: integrations-for-asset-request-approvals
spec:
  triggers:
    - group: catalog
      kind: AssetRequest
      name: "*"
      scope:
        kind: Application
        name: "*"
      type:
      - updated
  webhooks:
    - wh-integrations-for-asset-request-approvals
```

> Note: You need to edit the YAML file above and set the API Builder base address and APIKey in the webhook section

The API Builder flow is shown below:

![](https://i.imgur.com/QGXHyKK.png)

The flow can certainly be improved by handling errors and other HTTP response status codes better but it demonstrates an MVP for now.
