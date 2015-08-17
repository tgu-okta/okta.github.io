---
layout: docs_page
title: API Test Client
redirect_from: "/docs/getting_started/api_test_client.html"
---

* Will be replaced with the ToC
{:toc .list-unstyled .toc}

## Postman

[Postman](http://getpostman.com) is an awesome REST API test client that a lot of people at Okta use to test the Okta API.  We have built templates for all of the Okta APIs that makes playing with the Okta APIs simple and easy if you don't have time to practice your *Command Line Fu* with [cURL](http://en.wikipedia.org/wiki/CURL)

![Chrome Web Store](/assets/img/postman-chrome-web-store.png "Chrome Web Store")

### Prerequisites

- Chrome Web Browser
- [API Key created in the Okta Admin UI](/docs/getting_started/getting_a_token.html)

### Setup

1. [Download Postman Chrome Extension](https://chrome.google.com/webstore/detail/postman-rest-client-packa/fhbjgbiflinjbdggehcddcbncdddomop)
2. Open the Chrome App Launcher from your browser toolbar or dock. As of the time of this writing, a native Mac OS X Postman application is also available in Beta. 

    ![Chrome App Launcher](/assets/img/chrome-app-launcher.png "Chrome App Launcher") 

3. Click the Postman application to launch it.
    
    ![Postman App Icon](/assets/img/postman-icon.png "Postman App Icon")

4. Import the environment template

    You should download and import the [Environment Variables](/docs/api/getting_started/collections/oktaEnvironment.json) and duplicate for each new environment (dev, test, preview, prod)

    1. [Download the template environment JSON](templates/example.okta.com.postman_environment) to your computer
    2. Select the `No environment` pull-down at the top and select `Manage environments`

        ![Manage Environments](/assets/img/postman-ui-manage-env.png "Manage Environments")

    3. Click the `Import` button in the **Manage Environments** modal and use the downloaded JSON file
   
        ![Import Environment](/assets/img/postman-ui-import-env.png "Import Environment")

        ---

        ![Choose Environment](/assets/img/postman-ui-choose-env.png "Choose Environment")

    4. Click the `Duplicate environment` icon to create a new environment from downloaded template

        ![Duplicate Environment](/assets/img/postman-ui-duplicate-env.png "Duplicate Environment")

    5. Edit the environment, assign a new name, and specify the following values:

        - `url`: full-qualified base URL for your Okta organization such as *https://acme.okta.com*
        - `apikey`: the API key you generated as part of prerequisites
        - `email-suffix`: used by request templates for email addresses and logins of new users
        - `subdomain`: *orgname* in https://*orgname*.okta.com

        > We recommend you name the new environment the name of your okta organization such as example.okta.com in case you have both a preview and production organization

        ![Edit Environment](/assets/img/postman-ui-edit-env.png "Edit Environment")

4. Select and verify your configured environment

    1. Ensure you have selected your environment in the pull-down
       
        ![Select Environment](/assets/img/postman-ui-select-env.png "Select Environment")

        > Your selected environment should have your name that you configured above and not say `No environment` or `example.okta.com` 

    2. You can preview your configured environment variables by hovering over the `eye icon`
       
        ![Preview Environment](/assets/img/postman-ui-preview-env.png "Preview Environment")


### Collections

Once you have setup your Postman environment, you can import pre-built collections for Okta APIs

1. Click Import Collection

    ![Import Collection](/assets/img/postman-ui-import-collection.png "Import Collection")

2. Download from link

    ![Download URL for Collection](/assets/img/postman-ui-import-url.png "Download URL for Collection")

    **Import the following templates:**

|----------------------------+----------+-------------------------------------------------------------------------------------------|
| Resource                   | Version  | Postman Collection                                                                        |
|:---------------------------|---------:|:------------------------------------------------------------------------------------------|
| Okta Apps                  |      1.0 | [apps](/docs/api/getting_started/collections/oktaApps.json "Okta Apps")                   |
| Okta Authentication        |      1.0 | [authn](/docs/api/getting_started/collections/oktaAuthn.json "Okta Authentication")       |
| Okta Events                |      1.0 | [events](/docs/api/getting_started/collections/oktaEvents.json "Okta Events")             |
| Okta Factors               |      1.0 | [factors](/docs/api/getting_started/collections/oktaFactors.json "Okta Factors")          |
| Okta Groups                |      1.0 | [groups](/docs/api/getting_started/collections/oktaGroups.json "Okta Groups")             |
| Okta Identity Providers    |      1.0 | [idps](/docs/api/getting_started/collections/oktaIdPs.json "Okta Identity Providers")     |
| Okta Policies              |      1.0 | [policies](/docs/api/getting_started/collections/oktaPolicies.json "Okta Policies")       |
| Okta Schema                |      1.0 | [schema](/docs/api/getting_started/collections/oktaSchema.json "Okta Schema")             |
| Okta Sessions              |      1.0 | [sessions](/docs/api/getting_started/collections/oktaSessions.json "Okta Sessions")       |
| Okta Users                 |      1.0 | [users](/docs/api/getting_started/collections/oktaUsers.json "Okta Users")                |
|----------------------------+----------+-------------------------------------------------------------------------------------------|

### Sample Requests

The imported collections contain URLs and JSON request bodies that have sample data with params such as **\{\{id\}\}**.  You will need to replace URL and body parameters with ids of specific resources for your specific organization.  This can usually be accomplished by first listing a resource collection with a search or filter, than copying the `id` of a specific resource.  See API documentation for the specific request for details.

The following request example requires a specific user `id`

   ![URL template with ID](/assets/img/postman-ui-replace-id.png "URL template with ID")

You can copy an `id` from a previous response such as a `GET` request to list resources and paste it.

   ![Copy and replace ID](/assets/img/postman-ui-paste-id.png "Copy and replace ID in URL") 

You can also set environment variables directly from the right-click context menu.  This is recommended so you can re-use the `id` across multiple requests

   ![Set environment variable](/assets/img/postman-ui-set-variable-id.png "Set environment variable")

Always check the request preview to ensure all **\{\{id\}\}** variables have been properly replaced before sending the request

   ![Request preview](/assets/img/postman-ui-req-prevew.png "Request preview")

 







