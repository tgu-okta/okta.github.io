---
layout: docs_page
title: SCIM Provisioning with Lifecycle Management
excerpt: Enable SCIM-based provisioning from Okta to your application.
icon: /assets/img/icons/scim.svg
permalink: /standards/SCIM/
redirect_from: "/docs/guides/scim_guidance.html"
---
# SCIM: Provisioning with Okta's Lifecycle Management TEST
Developers at a software vendor (ISV), Okta customers, and system-integrators (SI) want to facilitate fast, 
enterprise-wide deployment of their app by integrating with Okta for user provisioning primarily via the SCIM standard. 
This article describes:
 
* The provisioning actions and use cases that your integration should consider supporting. 
* The technical options for how to build the integration (focusing on SCIM).
* The process for building, submitting for Okta Review, and publishing in the OAN (if you want the app available for all Okta customers). 

## Understanding Provisioning Use Cases

### The Value of Provisioning

With the proliferation of cloud apps, today’s IT organizations are faced with the prospect of managing user accounts 
in an ever-expanding number of admin consoles for each app. This is not a problem if an enterprise has one or two cloud apps, 
but as the number grows, the situation quickly becomes unmanageable. Cloud app vendors hoping to sell into enterprises 
need to have an answer to this concern.

Thousands of Okta customers have chosen our [Lifecycle Management](https://www.okta.com/products/lifecycle-management/) product as the platform solution 
to address this challenge of managing the lifecycle of user accounts in their cloud apps, 
and a key feature of this product is integrations for automated provisioning to those apps. 
Lifecycle Management (Provisioning) automates many of the traditionally manual tasks required to on-board and off-board employees. 
New employees are automatically provisioned with user accounts in their apps, profile updates like department changes populate automatically, 
and inactive employees are automatically deactivated from their apps. By making these actions less time-consuming and error-prone, 
companies can cut costs, allow new employees to be immediately productive, and reduce the threat of data leakage.

### Provisioning Actions

Provisioning consists of a set of actions between Okta and the cloud app. 
These actions are building blocks that combine to solve end-to-end use cases. 
As the developer, you’ll need to define the use cases that your target customer wants 
and the corresponding actions to build into your integration. 

The set of actions that an integration can do under the Provisioning umbrella 
are easily described with the acronym CRUD: the common database operations of Create, Read, Update, and Delete. 
CRUD actions relate to Provisioning in the following ways:

#### Create Users

* Create new users in the downstream application based on values derived from the Okta user profile and Okta group assignment. 
* Read (Import) users & groups
* Import users & groups from the downstream application in order to match them to existing Okta users, or to create new Okta users from the imported application.

#### Update Users

For an application user that is affiliated with an Okta user, 
update the downstream user’s attributes when the Okta user is updated. 
Or, update the Okta user attributes if the application functions as the [master](#mastering-users) for the full Okta user profile or specific attributes.

#### Deprovision (Deactivate) Users

Deprovisioning the application user removes access to the downstream application. 
This can take many forms, such as user disabled, user access permissions changed, or user license pulled. 
Each application may choose different methods for deactivating a user’s access.

For audit purposes, Okta users are never deleted; they are deactivated instead. 
Because of this, Okta doesn't make delete requests to the user APIs in downstream applications.

#### Sync Password

Okta sets the user’s password to either match the Okta password or to be a randomly generated password. 
Learn more about the overall use case [here](https://support.okta.com/help/articles/Knowledge_Article/Using-Sync-Password?retURL=%2Fhelp%2Fapex%2FKnowledgeArticleJson%3Fc%3DOkta_Documentation%253ADirectories%26p%3D101%26inline%3D1&popup=true). 

#### Mastering Users

Mastering is a more sophisticated version of read (import) Users. 
Mastering defines the flow and maintenance of user object attributes and their lifecycle state. 
When a profile is mastered from a given resource (application or directory), 
the Okta user profile’s attributes and lifecycle state are derived exclusively from that resource. 
In other words, an Okta user mastered by Active Directory has an Okta profile. 
However, the profile isn't editable in Okta by the user or Okta admin, and derives its information exclusively from Active Directory. 
If the lifecycle state of the user in Active Directory moves to Disabled, 
the linked Okta user will also switch to the corresponding lifecycle state of Deactivated on the next the read (import). 

A more granular version of mastering is Attribute-Level mastering. 
This is where Mastering can be applied at the Attribute level, rather than only at the whole Profile level. 
This permits an Okta user profile to derive portions of its attributes from multiple different Masters. 

Mastering is an advanced action. If you want your integration to support it, we recommend contacting Okta to double check your design before starting development. 

### Provisioning Use Cases

Provisioning actions can be combined to solve for end-to-end use cases.

#### Directory-as-Master (Downstream Provisioning)

<!-- Images for v2: https://oktawiki.atlassian.net/wiki/pages/viewpage.action?pageId=181895852 -->

In many enterprises, Active Directory (or LDAP) is the system of record for employee identities. 
Okta has developed a powerful, lightweight agent to sync with Active Directory to populate employee and group information. 
Within Okta, IT admins can leverage features such as [Universal Directory](https://support.okta.com/help/articles/Knowledge_Article/About-Universal-Directory) and [group membership rules](https://support.okta.com/help/articles/Knowledge_Article/About-Universal-Directory) to map that information when provisioning accounts and permissions in downstream apps.

Subsequently, any updates to an employee’s profile, such as a change in department, in either Active Directory or Okta flow into the downstream app. 
Similarly, removing or deactivating an employee from Active Directory triggers deactivation in the downstream app as well.

Okta supports these common Provisioning use cases:

* Provision downstream apps automatically when new employee joins the company
* Update downstream app automatically when employee profile attributes change
* Remove employee access to downstream app automatically on termination or leave
* Link existing downstream app users with Okta users using one-time app import

#### App-as-Master

While most apps fit the category of a downstream app in the directory-as-master use case, some apps can be the master. This is the App-as-Master use case.

The app-as-master use case typically applies to apps that can be used as the system of record for all employee profile information. An HR app like Workday or Successfactors Employee Central is the most common example. 
In this scenario, the HR app, not Active Directory, feeds employee profile details downstream into directories like AD and Okta, and apps like Box. 

<!-- More info on configuring app-as-master in Okta is [here](). Note: link was missing from source -->

> Note: Integrations for the “App-as-Master” use case are significantly more complex than the Directory-as-Master use case and take more time to build and support. 
This is because these integrations sync a larger number of attributes and lifecycle states, and more directly impact the Okta user profile and downstream apps. 
If you want your integration to support it, we recommend contacting <developers@okta.com> to double check your design before starting development. 

#### Attribute-Level Mastering

Most apps fall clearly into the groupings above: 

* Directory-as-master&mdash;all employee profile attributes in an app are set by the upstream directory
* App-as-master&mdash;the app itself is the employee system of record for downstream apps, Active Directory, and Okta

However, some apps exist somewhere in the middle. These apps want to be the master for certain employee attributes like phone number, while letting the directory master most other attributes. We call this attribute-level mastering.

Unified communications apps like RingCentral are common examples of apps for which attribute-level mastering might be desired. How does this use case work?

When a new employee joins a company he/she will typically be added first to AD which, via Okta, will trigger the provisioning of a new RingCentral account. In the employee profile attributes that are exchanged between AD and RingCentral, phone number will be missing however, as AD does not initially know the new employee’s phone number. This number is first established in RingCentral.

If the phone number attribute is set in Okta to be Mastered by RingCentral, once the phone number is initially established in RingCentral, it will be synced back to AD. This will allow IT to have more complete information about the new employee.

See Okta's [online help](https://support.okta.com/help/articles/Knowledge_Article/Attribute-Level-Mastering) for more info on configuring attribute-level mastering in Okta.

## Ways to Build Provisioning

Now that you understand the most common provisioning actions and use cases, let’s review your options to support provisioning as an app developer. 
While we outline a few different methods below, Okta recommends all ISVs support [the SCIM standard](http://www.simplecloud.info/). 

### Provisioning Options Matrix

Okta has seen broad adoption of the standard in the market amongst our network of app developers over the course of 2016. 
Okta has doubled down on our investment in our SCIM client and launched our own SCIM provisioning developer program.

| Standard | Recommendation  |  Summary |
|:--|:--|:--|
| SCIM  | **Recommended**  | Supports all of the use cases above.  |
| SAML JIT (Just-in-Time)  | Not Recommended - Limited Functionality  | Just-in-Time Provisioning (JIT) is part of the SAML spec. Supports user/group Create and Update (upon user sign-on) but does not support Deactivate, Read, or Sync Password  |
| Okta API  | Not Recommended - Not Verified by Okta  | Poll Okta API for user & group CRUD operations. Polling Okta API is resource intensive, so provisioning won’t be real-time. Poor admin experience: cannot centralize within Okta  |

### Provisioning to On-Premise Apps

The options above are geared towards cloud apps but we have a solution for on-premise applications as well. 
See [Configuring On-Premise Provisioning](https://support.okta.com/help/articles/Knowledge_Article/29448976-Configuring-On-Premises-Provisioning) for details about Okta’s agent-based provisioning solution.

### SCIM Facade

Sometimes it isn't feasible for the cloud app to natively support a SCIM Server API. 
An alternative option is to build and host a SCIM facade middleware that translates 
between the Okta SCIM Client and the cloud app’s proprietary API. The Okta integration would be to this SCIM facade. 

> Have questions? Need help? Email us at <developers@okta.com> or post your question on [Stack Overflow](http://stackoverflow.com/search?q=okta,scim).

## SCIM-Based Provisioning Integration

### Overview

By implementing support for the SCIM standard, an application in the Okta
Application Network can be notified when a user is created, updated,
or removed from their application in Okta.

If you haven't heard of SCIM before, here is a good summary from the
[Wikipedia article on SCIM](https://en.wikipedia.org/wiki/System_for_Cross-domain_Identity_Management):

"System for Cross-domain Identity Management (SCIM) is an open
standard for automating the exchange of user identity information
between identity domains, or IT systems."

### Understanding User Provisioning in Okta

Okta is a universal directory that stores
identity-related information.  Users can be created in Okta directly
as local users or imported from external systems like Active
Directory or a [Human Resource Management Software](https://en.wikipedia.org/wiki/Category:Human_resource_management_software) system.

An Okta user schema contains many different user attributes,
but always contains a user name, first name, last name, and
email address. This schema can be extended.

Okta user attributes can be mapped from a source into Okta and can
be mapped from Okta to a target.

Below are the main operations in Okta's SCIM user provisioning lifecycle:

1.  Create a user account.
2.  Read a list of accounts, with support for searching for a preexisting account.
3.  Update an account (user profile changes, entitlement changes, etc).
4.  Deactivate an account.

In Okta, an application instance is a connector that provides Single Sign-On
and provisioning functionality with the target application.

### Required SCIM Server Capabilities

Okta supports provisioning to both SCIM 1.1 and SCIM 2.0 APIs.

If you haven't implemented SCIM, Okta recommends that you implement
SCIM 2.0.

Okta implements SCIM 2.0 as described in RFCs [7642](https://tools.ietf.org/html/rfc7642), [7643](https://tools.ietf.org/html/rfc7643), [7644](https://tools.ietf.org/html/rfc7644).

If you are writing a SCIM implementation for the first time, an
important part of the planning process is determining which of
Okta's provisioning features your SCIM API can or should support and
which features you do not need to support.

Specifically, you do not need to implement the SCIM 2.0
specification fully to work with Okta. At a minimum, Okta requires that
your SCIM 2.0 API implement the features described below:

#### Base URL

The API endpoint for your SCIM API **MUST** be secured via [TLS](https://tools.ietf.org/html/rfc5246)
(`https://`), Okta *does not* connect to unsecured API endpoints.

You can choose any Base URL for your API endpoint. If you
are implementing a brand new SCIM API, we suggest using `/scim/v2`
as your Base URL; for example: `https://example.com/scim/v2` -
however, you must support the URL structure described in the
["SCIM Endpoints and HTTP Methods" section of RFC7644](https://tools.ietf.org/html/rfc7644#section-3.2).

If you have multiple Okta orgs, you can use the same SCIM server for all of them. 
Implement a subdomain for the SCIM server and each subdomain, for example: `https://subdomain.example.com/scim/v2`, where 
`subdomain` represent one Okta org. Let's say you have three Okta orgs: company-a.okta.com, company-b.okta.com and company-c.okta.com. 
From each org you can pass a base URL containing the name of the org: `https://company-a.example.com/scim/v2`, `https://company-b.example.com/scim/v2` or  `https://company-c.example.com/scim/v2`.
In your SCIM server, you can read which subdomain the request is coming from and identify the org. 

#### Authentication

Your SCIM API **MUST** be secured against anonymous access. At the
moment, Okta supports authentication against SCIM APIs with one of
the following methods:

1.  [OAuth 2.0 Authorization Code Grant Flow](https://tools.ietf.org/html/rfc6749#section-4.1)
2.  [Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication)
3.  Custom HTTP Header

> Okta doesn't support OAuth 2.0 [Resource Owner Password Credentials grant flows](https://tools.ietf.org/html/rfc6749#section-1.3.3).

#### Basic User Schema

Your service must be capable of storing the following four user
attributes:

1.  User ID (`userName`)
2.  First Name (`name.givenName`)
3.  Last Name (`name.familyName`)
4.  Email (`emails`)

Note that Okta supports more than the four user attributes listed
above. However, these four attributes are the base attributes that
you must support.  The full user schema for SCIM 2.0 is described
in [section 4 of RFC 7643](https://tools.ietf.org/html/rfc7643#section-4).

> **Best Practice:** Keep your User ID distinct from the User Email
> Address. Many systems use an email address as a user identifier,
> but this is not recommended, as email addresses often change. Using
> a unique User ID to identify user resources prevents future
> complications.

If your service supports user attributes beyond those four base
attributes, add support for those additional
attributes to your SCIM API. In some cases, you might need to
configure Okta to map non-standard user attributes into the user
profile for your application.

Included in this git repository is a sample application written in
Python/Flask, this sample application implements SCIM 2.0. Below is
how this sample application defines these attributes:

    userName = db.Column(db.String(250),
                         unique=True,
                         nullable=False,
                         index=True)
    familyName = db.Column(db.String(250))
    middleName = db.Column(db.String(250))
    givenName = db.Column(db.String(250))

In addition to the basic user schema user attributes described
above, your SCIM API must also have a unique identifier for each
user resource and should also support marking resources as "active"
or "inactive."

In the SCIM specification, the `id` attribute is used to uniquely
identify resources. [Section 3.1](https://tools.ietf.org/html/rfc7643#section-3.1) of [RFC 7643](https://tools.ietf.org/html/rfc7643) provides more details
on the `id` attribute:

> "A unique identifier for a SCIM resource as defined by the service
> provider.  Each representation of the resource MUST include a
> non-empty "id" value.  This identifier MUST be unique across the
> SCIM service provider's entire set of resources.  It MUST be a
> stable, identifier that can't be reassigned, that is, it doesn't change when the
> same resource is returned in subsequent requests.  The value of
> the "id" attribute is always issued by the service provider and
> MUST NOT be specified by the client.  The string "bulkId" is a
> reserved keyword and MUST NOT be used within any unique identifier
> value.  The attribute characteristics are "caseExact" as "true", a
> mutability of "readOnly", and a "returned" characteristic of
> "always"."

Our sample application defines `id` as a UUID, since
[RFC 7643](https://tools.ietf.org/html/rfc7643) requires that "this identifier MUST be unique across the
SCIM service provider's entire set of resources."

    id = db.Column(db.String(36), primary_key=True)

**Note:** Your SCIM API can use anything as an `id`, provided that the `id`
uniquely identifies reach resource, as described in [section 3.1](https://tools.ietf.org/html/rfc7643#section-3.1) of
[RFC 7643](https://tools.ietf.org/html/rfc7643).

Finally, your SCIM API must also support marking a resource as
"active" or "inactive."

In our sample application, each user resource has a Boolean
"active" attribute which is used to mark a user resource as
"active" or "inactive":

    active = db.Column(db.Boolean, default=False)

#### Functionality

Your SCIM API must support the following SCIM API endpoints to work with Okta:

<img src="/assets/img/scim_flowchart.png" alt="scim api endpoints required to work with Okta" width="640px" />

##### Create Account: POST /Users

Your SCIM 2.0 API should allow the creation of a new user
account.  The four basic attributes listed above must be supported, along
with any additional attributes that your application supports.  If your
application supports entitlements, your SCIM 2.0 API should allow
configuration of those as well.

An HTTP POST to the `/Users` endpoint must return an immutable or
system ID of the user (`id`) must be returned to Okta.

Okta will call this SCIM API endpoint under the following circumstances:

-   **Direct assignment**

    When a user is assigned to an Okta application using the "Assign
    to People" button in the "People" tab.
-   **Group-based assignment**

    When a user is added to a group that is assigned to an Okta
    application. For example, an Okta administrator can assign a
    group of users to an Okta application using the "Assign to
    Groups" button in the "Groups" tab. When a group is assigned to an
    Okta application, Okta sends updates to the assigned
    application when a user is added or removed from that group.

Below is an example demonstrating how the sample application handles account
creation:

    @app.route("/scim/v2/Users", methods=['POST'])
    def users_post():
        user_resource = request.get_json(force=True)
        user = User(user_resource)
        user.id = str(uuid.uuid4())
        db.session.add(user)
        db.session.commit()
        rv = user.to_scim_resource()
        send_to_browser(rv)
        resp = flask.jsonify(rv)
        resp.headers['Location'] = url_for('user_get',
                                           user_id=user.userName,
                                           _external=True)
        return resp, 201

Note: `force=True` is set because Okta sends
`application/scim+json` as the `Content-Type` and the `.get_json()`
method expects `application/json`.

For more information on user creation via the `/Users` SCIM
endpoint, see [section 3.3](https://tools.ietf.org/html/rfc7644#section-3.3) of the [SCIM 2.0 Protocol Specification](https://tools.ietf.org/html/rfc7644).

##### Read list of accounts with search: GET /Users

Your SCIM 2.0 API must support the ability for Okta to retrieve
users (and entitlements like groups if available) from your
service.  This allows Okta to fetch all user resources in an
efficient manner for reconciliation and initial bootstrap (to
get all users from your app into the system).

Here is an example using `curl` to make a GET request to `/Users`:

    curl https://joel-scim.herokuapp.com/scim/v2/Users

Below is how the sample application handles listing user resources,
with support for filtering and pagination:

    @app.route("/scim/v2/Users", methods=['GET'])
    def users_get():
        query = User.query
        request_filter = request.args.get('filter')
        match = None
        if request_filter:
            match = re.match('(\w+) eq "([^"]*)"', request_filter)
        if match:
            (search_key_name, search_value) = match.groups()
            search_key = getattr(User, search_key_name)
            query = query.filter(search_key == search_value)
        count = int(request.args.get('count', 100))
        start_index = int(request.args.get('startIndex', 1))
        if start_index < 1:
            start_index = 1
        start_index -= 1
        query = query.offset(start_index).limit(count)
        total_results = query.count()
        found = query.all()
        rv = ListResponse(found,
                          start_index=start_index,
                          count=count,
                          total_results=total_results)
        return flask.jsonify(rv.to_scim_resource())

> If you want to see the SQL query that SQLAlchemy is using for
> the query, add this code after the `query` statement that you want
> to see: `print(str(query.statement))`

For more details on the `/Users` SCIM endpoint, see [section 3.4.2](https://tools.ietf.org/html/rfc7644#section-3.4.2)
of the [SCIM 2.0 Protocol Specification](https://tools.ietf.org/html/rfc7644).

##### Read Account Details: GET /Users/{id}

Your SCIM 2.0 API must support fetching of users by user id.

Below is how the sample application handles returning a user resource
by `user_id`:

    @app.route("/scim/v2/Users/<user_id>", methods=['GET'])
    def user_get(user_id):
        try:
            user = User.query.filter_by(id=user_id).one()
        except:
            return scim_error("User not found", 404)
        return render_json(user)

If we don't find a user, we return a HTTP status 404 ("Not found")
with SCIM error message.

For more details on the `/Users/{id}` SCIM endpoint, see [section 3.4.1](https://tools.ietf.org/html/rfc7644#section-3.4.1)
of the [SCIM 2.0 Protocol Specification](https://tools.ietf.org/html/rfc7644).

##### Update Account Details: PUT /Users/{id}

When a profile attribute of a user assigned to your SCIM enabled
application is changed, Okta will do the following:

-   Make a GET request against `/Users/{id}` on your SCIM API for the
    user to update.
-   Take the resource returned from your SCIM API and update only the
    attributes that need to be updated.
-   Make a PUT request against `/Users/{id}` in your SCIM API with
    the updated resource as the payload.

Examples of things that can cause changes to an Okta user profile
are:

-   A change in profile a master like Active Directory or a Human Resource
    Management Software system.
-   A direct change of a profile attribute in Okta for a local user.

Below is how the sample application handles account profile updates:

    @app.route("/scim/v2/Users/<user_id>", methods=['PUT'])
    def users_put(user_id):
        user_resource = request.get_json(force=True)
        user = User.query.filter_by(id=user_id).one()
        user.update(user_resource)
        db.session.add(user)
        db.session.commit()
        return render_json(user)

For more details on updates to the `/Users/{id}` SCIM endpoint, see [section 3.5.1](https://tools.ietf.org/html/rfc7644#section-3.5.1)
of the [SCIM 2.0 Protocol Specification](https://tools.ietf.org/html/rfc7644).

##### Deactivate Account: PATCH /Users/{id}

Deprovisioning is perhaps the most important reason customers why
customers ask that your application supports provisioning
with Okta. Your SCIM API should support account deactivation via a
PATCH to `/Users/{id}` where the payload of the PATCH request sets
the `active` property of the user to `false`.
Okta also does a PUT if the Patch is not supported for deactivation.

Your SCIM API should allow account updates at the attribute level.
If entitlements are supported, your SCIM API should also be able
to update entitlements based on SCIM profile updates.

Okta will send a PATCH request to your application to deactivate a
user when an Okta user is "unassigned" from your
application. Examples of when this happen are as follows:

-   A user is manually unassigned from your application.
-   A user is removed from a group which is assigned to your application.
-   When a user is deactivated in Okta, either manually or via
    by an external profile master like Active Directory or a Human
    Resource Management Software system.

Below is how the sample application handles account deactivation:

    @app.route("/scim/v2/Users/<user_id>", methods=['PATCH'])
    def users_patch(user_id):
        patch_resource = request.get_json(force=True)
        for attribute in ['schemas', 'Operations']:
            if attribute not in patch_resource:
                message = "Payload must contain '{}' attribute.".format(attribute)
                return message, 400
        schema_patchop = 'urn:ietf:params:scim:api:messages:2.0:PatchOp'
        if schema_patchop not in patch_resource['schemas']:
            return "The 'schemas' type in this request is not supported.", 501
        user = User.query.filter_by(id=user_id).one()
        for operation in patch_resource['Operations']:
            if 'op' not in operation and operation['op'] != 'replace':
                continue
            value = operation['value']
            for key in value.keys():
                setattr(user, key, value[key])
        db.session.add(user)
        db.session.commit()
        return render_json(user)

For more details on user attribute updates to `/Users/{id}` SCIM endpoint, see [section 3.5.2](https://tools.ietf.org/html/rfc7644#section-3.5.2)
of the [SCIM 2.0 Protocol Specification](https://tools.ietf.org/html/rfc7644).

##### Filtering on `id`, `userName`, and `emails`

Being able to filter results by the `id`, `userName`, or `emails`
attributes is a critical part of working with Okta.

Your SCIM API must be able to filter users by `userName` and should
also support filtering by `id` and `emails`. Filtering support
is required because most provisioning actions require the ability
for Okta to determine if a user resource exists on your system. The filter Okta generates will be based on the `userName` field, following the pattern "userName eq "..." ".

Consider the scenario where an Okta customer with thousands of
users has a provisioning integration with your system, which also
has thousands of users. When an Okta customer adds a new user to
their Okta organization, Okta needs a way to determine quickly if a
resource for the newly created user was previously created on your
system.

Examples of filters that Okta might send to your SCIM API are as
follows:

~~~ 
userName eq "jane.doe"

userName eq "jane.doe@example.com"
~~~

At the moment, Okta only supports the `eq` filter operator. However, the
[filtering capabilities](https://tools.ietf.org/html/rfc7644#section-3.4.2.2) described in the SCIM 2.0 Protocol Specification are
much more complicated.

Here is an example of how to implement SCIM filtering in Python:

    request_filter = request.args.get('filter')
    match = None
    if request_filter:
        match = re.match('(\w+) eq "([^"]*)"', request_filter)
    if match:
        (search_key_name, search_value) = match.groups()
        search_key = getattr(User, search_key_name)
        query = query.filter(search_key == search_value)

Note: The sample code above only supports the `eq` operator. We
recommend that you add support for all of the filter operators
described in [table 3](https://tools.ietf.org/html/rfc7644#page-18) of the SCIM 2.0 Protocol Specification.

For more details on filtering in SCIM 2.0, see [section 3.4.2.2](https://tools.ietf.org/html/rfc7644#section-3.4.2.2)
of the [SCIM 2.0 Protocol Specification](https://tools.ietf.org/html/rfc7644).

##### Filtering on externalId

In addition to supporting filtering on `id`, `userName`, and
`emails`, your application should also support filtering on
`externalId`.

Okta will use the `externalId` to determine if your application
already has an account. `externalId` is used as a stable identifier
for users, because the `userName` and email addresses for a user
can change.

Here is an example of an `externalId` filter that might be sent to
your application:

~~~
externalId eq "00u1abcdefGHIJKLMNOP"
~~~

Note: The sample application included in this project does not yet
demonstrate how to implement storing and filtering by
`externalId`. However, Okta strongly recommends that your SCIM
implementation supports storing and filtering by `externalId`. For
details on supporting `externalId`, see
[section 3.1](https://tools.ietf.org/html/rfc7643#section-3.1) of [RFC 7643](https://tools.ietf.org/html/rfc7643). Quoted below:

> (externalId is) "A String that is an identifier for the resource
> as defined by the provisioning client.  The "externalId" may
> simplify identification of a resource between the provisioning
> client and the service provider by allowing the client to use a
> filter to locate the resource with an identifier from the
> provisioning domain, obviating the need to store a local mapping
> between the provisioning domain's identifier of the resource and
> the identifier used by the service provider.  Each resource MAY
> include a non-empty "externalId" value.  The value of the
> "externalId" attribute is always issued by the provisioning
> client and MUST NOT be specified by the service provider.  The
> service provider MUST always interpret the externalId as scoped
> to the provisioning domain.  While the server does not enforce
> uniqueness, it is assumed that the value's uniqueness is
> controlled by the client setting the value.

When adding support for `externalId` filtering to your application,
we suggest that you use OAuth2.0 for authentication and use the
OAuth2.0 `client_id` to scope the `externalId` to the provisioning
domain.

##### Resource Paging

When returning large lists of resources, your SCIM implementation
must support pagination using a *limit* (`count`) and *offset*
(`startIndex`) to return smaller groups of resources in a request.

Below is an example of a `curl` command that makes a request to the
`/Users/` SCIM endpoint with `count` and `startIndex` set:

    $ curl 'https://scim-server.example.com/scim/v2/Users?count=1&startIndex=1'
    {
      "Resources": [
        {
          "active": false,
          "id": 1,
          "meta": {
            "location": "http://scim-server.example.com/scim/v2/Users/1",
            "resourceType": "User"
          },
          "name": {
            "familyName": "Doe",
            "givenName": "Jane",
            "middleName": null
          },
          "schemas": [
            "urn:ietf:params:scim:schemas:core:2.0:User"
          ],
          "userName": "jane.doe@example.com"
        }
      ],
      "itemsPerPage": 1,
      "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:ListResponse"
      ],
      "startIndex": 0,
      "totalResults": 1
    }

> Note: When returning a paged resource, your API should return a
> capitalized `Resources` JSON key ("Resources"), however Okta will also
> support a lowercase string ("resources"). Okta will also accept
> lowercase JSON strings for the keys of child nodes inside
> `Resources` object such as `startindex`, `itemsperpage`, or `totalresults`.

One way to handle paged resources is to have your database do the
paging for you. Here is how the sample application handles
pagination with SQLAlchemy:

    count = int(request.args.get('count', 100))
    start_index = int(request.args.get('startIndex', 1))
    if start_index < 1:
        start_index = 1
    start_index -= 1
    query = query.offset(start_index).limit(count)

Note: This code subtracts "1" from the
`startIndex`, because `startIndex` is [1-indexed](https://tools.ietf.org/html/rfc7644#section-3.4.2) and
the OFFSET statement is [0-indexed](http://www.postgresql.org/docs/8.0/static/queries-limit.html).

For more details pagination on a SCIM 2.0 endpoint, see [section 3.4.2.4](https://tools.ietf.org/html/rfc7644#section-3.4.2.4)
of the [SCIM 2.0 Protocol Specification](https://tools.ietf.org/html/rfc7644).

##### Rate Limiting

Some customer actions, such as adding hundreds of users at once,
causes large bursts of HTTP requests to your SCIM API. For
scenarios like this, we suggest that your SCIM API return rate
limiting information to Okta via the [HTTP 429 Too Many Requests](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#429)
status code. This helps Okta throttle the rate at which SCIM
requests are made to your API.

For more details on rate limiting requests using the HTTP 429
status code, see [section 4](https://tools.ietf.org/html/rfc6585#section-4) of [RFC 6585](https://tools.ietf.org/html/rfc6585).

#####  GET /Groups API Endpoint

Okta currently supports the /Groups endpoint for GET /Groups of a SCIM API. This is usually done to check for groups data and is not mandatory for SCIM to work. The minimum check we require is for the resources to be of JSON. check example below.

Example:

~~~sh
{
  "schemas": [
    "urn:ietf:params:scim:api:messages:2.0:ListResponse"
  ],
  "totalResults": 1,
  "startIndex": 0,
  "itemsPerPage": 0,
  "Resources": [
    {
      "id": "66ed8bece1944aa18bf96fb5c935c4ba",
      "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:Group"
      ],
      "displayName": "Marketing",
      "members": [
        {
          "value": "m1@atko.com",
          "$ref": "localhost:8080/Users/12345",
          "display": "Marketing User 1"
        },
        {
          "value": "m2@atko.com",
          "$ref": "localhost:8080/Users/12346",
          "display": "Marketing User 2"
        }
      ]
    }
  ]
}
~~~

### SCIM Features Not Implemented by Okta

The following features are currently not supported by Okta:

#### DELETE /Users/{id}

Deleting users via DELETE is covered in
[section 3.6](https://tools.ietf.org/html/rfc7644#section-3.6) of the [SCIM 2.0 Protocol Specification](https://tools.ietf.org/html/rfc7644).

Okta users are never **deleted**; they are **deactivated**
instead. Because of this, Okta never makes an HTTP DELETE
request to a user resource on your SCIM API. Instead, Okta makes
an HTTP PATCH request to set the `active` setting to `false`.

#### Querying with POST

The ability to query users with a POST request is described in
[section 3.4.3](https://tools.ietf.org/html/rfc7644#section-3.4.3) of the [SCIM 2.0 Protocol Specification](https://tools.ietf.org/html/rfc7644).

Querying using POST is sometimes useful if your query contains
[personally identifiable information](https://en.wikipedia.org/wiki/Personally_identifiable_information) that would be exposed in
system logs if used query parameters with a GET request.

Okta currently does not support this feature.

#### Bulk Operations

The ability to send a large collection of resource operations in a
single request is covered in
[section 3.7](https://tools.ietf.org/html/rfc7644#section-3.7) of the [SCIM 2.0 Protocol Specification](https://tools.ietf.org/html/rfc7644).

Okta currently does not support this feature and makes
one request per resource operation.

#### "/Me" Authenticated Subject Alias

The `/Me` URI alias for the current authenticated subject is
covered in
[section 3.11](https://tools.ietf.org/html/rfc7644#section-3.11) of the [SCIM 2.0 Protocol Specification](https://tools.ietf.org/html/rfc7644).

Okta does not currently make SCIM requests with the `/Me` URI alias.

#### /Groups API endpoint

Okta currently does not support using the `/Groups` endpoint of a SCIM
API, except for [`GET /Groups`](#get-groups-api-endpoint). When support is added for the `/Groups` endpoint, Okta plans
to use the following HTTP requests against the `/Groups` endpoint:

-   Read list of Groups: GET /Groups

-   Create Group: POST /Groups

-   Read Group detail: GET /Groups/{id}

-   Delete Group: DELETE /Groups/{id}

#### /Schemas API endpoint

Okta does not currently make queries against the `/Schemas`
endpoint, but this functionality is being planned.

Here is the specification for the `/Schemas` endpoint, from
[section 4](https://tools.ietf.org/html/rfc7644#section-4) of [RFC 7644](https://tools.ietf.org/html/rfc7644):

> An HTTP GET to this endpoint is used to retrieve information about
> resource schemas supported by a SCIM service provider.  An HTTP
> GET to the endpoint "/Schemas" SHALL return all supported schemas
> in ListResponse format (see Figure 3).  Individual schema
> definitions can be returned by appending the schema URI to the
> /Schemas endpoint.  For example:
>
> `/Schemas/urn:ietf:params:scim:schemas:core:2.0:User`
>
> The contents of each schema returned are described in Section 7 of
> RFC7643.  An example representation of SCIM schemas may be found
> in Section 8.7 of RFC7643.

#### /ServiceProviderConfig API endpoint

Okta does not currently make queries against the `/ServiceProviderConfig`
endpoint, but this functionality is being planned.

Here is the specification for the `/ServiceProviderConfig` endpoint, from
[section 4](https://tools.ietf.org/html/rfc7644#section-4) of [RFC 7644](https://tools.ietf.org/html/rfc7644):

> An HTTP GET to this endpoint will return a JSON structure that
> describes the SCIM specification features available on a service
> provider.  This endpoint SHALL return responses with a JSON object
> using a "schemas" attribute of
> `urn:ietf:params:scim:schemas:core:2.0:ServiceProviderConfig`.
> The attributes returned in the JSON object are defined in
> Section 5 of RFC7643.  An example representation of SCIM service
> provider configuration may be found in Section 8.5 of RFC7643.

#### Filtering on meta.lastModified

Okta does not currently make queries for resources using
`meta.lastModified` as part of a filter expression.

Okta plans to add functionality to fetch incremental updates
from SCIM APIs by querying for resources using a filter expression
that requests resources which were updated since the last update.

This will likely be done using the `gt` filter operator. For
example:

> `filter=meta.lastModified gt "2011-05-13T04:42:34Z"`

## Publishing Your SCIM-Based Provisioning Integration

### Process

In order to allow customers to use your SCIM provisioning integration with Okta, 
you’ll need to get your app published in [the Okta Application Network](https://www.okta.com/resources/find-your-apps/).

Follow the steps below to test and submit your application for Okta review: 

1. Review Okta’s SCIM Docs & Prepare Your App
2. Test Your SCIM Server
3. Submit for Okta Review
4. Test with Customers
5. Publish to Okta Application Network (OAN)

> Have questions? Need help? Email us at <developers@okta.com> or post your question on [Stack Overflow](http://stackoverflow.com/search?q=okta,scim).

### Private App Option

The OAN is for making an integration publicly discoverable and accessible to all Okta customers. 
However, you can also just use the integration privately within a few named orgs. 
This could be the case if you are a system integrator, customer, or Okta PS integrating to a custom app. 
If this is the case, follow steps 1-3 and you will be able to indicate in step 3 that you don’t want to publish in OAN. 
Okta will create the submitted integration per usual and assign it to Orgs that you specify as a private app. 
Note that you cannot use the SCIM template app used for prototyping, as it has limitations that prevent it from being used in production. 

#### Review Okta’s SCIM Docs & Prepare Your App

The first step is to build a compliant SCIM server. 
Even if you already support SCIM, it is important that you review Okta’s SCIM docs above,
especially the following sections, to understand the specifics of Okta’s support for the SCIM standard:

* [Understanding User Provisioning in Okta](#understanding-user-provisioning-in-okta)
* [Required SCIM Server Capabilities](#required-scim-server-capabilities)
* [SCIM Features Not Implemented by Okta](#scim-features-not-implemented-by-okta)

#### Test Your SCIM Server

##### Testing your SCIM server with Runscope

The easiest way for you to develop and verify your SCIM integration
is to make use of an automated test suite that runs on Runscope.

If you are already familiar with Runscope, then import the
OKTA SCIM Spec Test JSON API test (for SCIM 1.1 or SCIM 2.0) and configure the `SCIM Base
  URL` variable to point at the base URL for your SCIM server, for
example: `https://example.com/scim/v2`.

* [Okta SCIM 2.0 Spec Test JSON](SCIMFiles/Okta-SCIM-20-SPEC-Test.json)
* [Okta SCIM 1.1 Spec Test JSON](SCIMFiles/Okta-SCIM-11-SPEC-Test.json)

If you are not familiar with Runscope, follow the detailed
instructions below to get started with using Runscope to test your
SCIM server.

##### Set up Runscope

If you do not have a Runscope account already, we suggest starting
with [Runscope's free trial plan for Okta](https://www.runscope.com/okta). Here is how to get started:

 1.  Download the Okta SCIM Spec Test for your version of SCIM:
     * [Okta SCIM 2.0 Spec Test JSON](SCIMFiles/Okta-SCIM-20-SPEC-Test.json)
     * [Okta SCIM 1.1 Spec Test JSON](SCIMFiles/Okta-SCIM-11-SPEC-Test.json)
 
      You will use this file to import Okta's SCIM test suite into Runscope.)

 2.  [Sign up for Runscope](http://www.runscope.com/signup).
 3.  You may see a tutorial after signing up for Runscope, if so, click
    "Skip Tutorial".
 4.  You should now see a screen that says "API Tests".
 5.  In the lower left of your screen, click on the "Import Tests"
    link.
 6.  You should now see a title that starts with "Import Tests into
    &#x2026;"
 7.  Select "Runscope API Tests" as the format to import
 8.  Click the "Choose File" button and select the JSON file that you saved in Step 1.
 9.  Click the blue "Import API Test" button.
10. After the import completes, click on the "All Tests" link on
    the left hand side of your screen.

Now that you've imported Okta's SCIM test suite into Runscope, your
next step will be to customize the test suite for the SCIM
integration that you are writing.

##### Customize the imported Runscope test for your SCIM integration

After importing Okta's SCIM test suite into Runscope, you will need to
configure the test for your SCIM integration. Here is how to do that:

1.  You should be looking at the "API Tests" screen in Runscope, if
    not, click on the "Tests" tab on the top of Runscope's user interface.
2.  You should see a test named "Okta SCIM 2.0 Tests", if not,
    follow the "Set up Runscope" steps above.
3.  Move your mouse over the "Okta SCIM 2.0 Tests" test, then select
    the "Edit" link on the lower left of the test.
4.  In the "Environment" section of your test, you should see a
    collapsed "Test Settings" section, click the arrow on the left
    of "Test Settings" to expand this section.
5.  "Initial Variables" should be selected, click the "Add Initial
    Variable" link and add the following:

    | Variable Name (Case Sensitive) | Example Values              | Notes                                                                                                                                                                                     |
    |:-------------------------------|:----------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | SCIMBaseURL                    | https://example.com/scim/v2 | For example, if your SCIM integration is hosted on https://example.com and uses a prefix of /scim/v2 then the "SCIM Base URL" for your integration would be: https://example.com/scim/v2. |
    | auth                           | Bearer abcxyz1234567890     | Basic/Oauth authorization values                                                                                                                                                          |

6.  Add the [Initial Script Spec](SCIMFiles/Initial_Script_Spec.txt).

    If you are developing your SCIM integration in a local
    development environment, we suggest using the excellent tool
    [ngrok](https://ngrok.com/) to expose your local development environment to Runscope.
8.  Click the "Save" button at the top of the test.

##### Running Okta's SCIM tests against your SCIM server

Now that you have updated your SCIM test in Runscope for your SCIM
server, it is time to run the test:

1.  If you followed the steps above, you should now be seeing a "Run
    Now" button at the top of your test.
2.  Click the "Run Now" button.
3.  On the left side of your screen, you will see a test show up in
    the "Recent Test Results" section.
4.  Click on the top test in the "Recent Test Results" section.
5.  If the test is still running, you will see live updates of the
    test in progress. Once the test is complete, you will see the
    results of your test.
6.  To see the details of tests, click on the little arrow next to
    each test to expand the details of a particular test case.

    Doing this will allow you to see the **Request** and **Response**
    for each HTTP request that was made.
7.  Since this test is running in your own Runscope instance, we
    encourage you to update the tests to better fit your own
    environment.
8.  See [Required SCIM Server Capabilities](#required-scim-server-capabilities) for details
    about your SCIM server needs to implement to pass all of
    the tests.
9.  Keep running this test suite until all the tests pass.
    Here is an [example of a test suite where all tests pass](https://www.runscope.com/radar/qmovuxkrhtws/f95ac15f-3f22-46c3-8f1a-1001fbf8fb66/history/6a35fabf-5ce5-4e48-a13f-7292b1bd3cc5).

##### Sharing test results from Runscope

As you are developing your SCIM server, you will likely want to
share test results with teammates or with Okta.

Here is how to share a test result from Runscope with someone else:

1.  Open the test result that you want to share.
2.  At the top of the test result, Change the "Private / Shareable"
    toggle from "Private" to "Shareable".
3.  Copy the URL for the test result, it will look something like
    this:
    `https://www.runscope.com/radar/abcdefghijkl/m01nopq2-3456-7r8s-9012-t34567uvw890/history/123ef4gh-i567-89j0-1k2l-3m4n5o678901`
4.  Share that URL with the person that you want to share the test
    result with. Here is an example test result from Runscope:
    <https://www.runscope.com/radar/qmovuxkrhtws/f95ac15f-3f22-46c3-8f1a-1001fbf8fb66/history/6a35fabf-5ce5-4e48-a13f-7292b1bd3cc5>

#### Testing your SCIM server with Okta

Once you have a SCIM server that passes all of the Runscope tests, 
test your SCIM integration directly with Okta. To do so, you will first need to sign up for [an Okta developer account](https://www.okta.com/developer/signup/).

Note: If you are using OAuth Authorization Code Grant flow as your authentication method 
or need to support the Profile Master action, Okta will need to custom-configure a template app for you. 
Please request this in your email to <developers@okta.com>. 

1. Navigate to the admin interface in your Okta org by clicking **Admin**. 
    ![Admin Button](/assets/img/scim-end-user-ui.png)

2. Click **Applications**, then **Add Application**. 
    ![Admin Button](/assets/img/scim-apps.png)

3. Search for “SCIM”. You’ll see three different SCIM template applications for each SCIM version (1.1 and 2.0) based off of the various authentication methods you could choose to support (Header Auth, Basic Auth, or Bearer Token).
    ![Admin Button](/assets/img/scim-templates.png)

#### Submit for Okta Review

Once you have a functioning SCIM integration in your Okta developer org, and have confirmed support for [Okta’s required SCIM server capabilities](#required-scim-server-capabilities), you are ready to formally submit your app for review by Okta.

Your submission will provide Okta with all the metadata needed to create a customized app (includes default mappings, link to config docs) which will be used by joint customers and eventually published publicly in [the Okta Application Network](https://www.okta.com/resources/find-your-apps/?_ga=1.200024301.294942002.1477328324). Okta will review the submission, create the customized app, run it through our internal QA, and then make it available in your developer org for your own testing.

We recommend completing a few actions before submitting your application to Okta:

1. Check the Profile Attributes for your application.
2. Check the Attribute Mappings for your application.
3. Run the second set of Runscope tests: Okta SCIM 2.0 CRUD Test.
4. Prepare the customer-facing configuration guide.

After performing these four steps (instructions below), use [this form](https://docs.google.com/forms/d/e/1FAIpQLSfueMM9vbIt6zXCdcI9MdJ16lZc1FHwyIwMkHwNBWF7Kzdocg/viewform) to submit for Okta review.


##### Check the Profile Attributes for your application

Before submitting your application to Okta, you should check the
User Attributes to make sure that the attributes are set to what
you would want your users to see.

Check your Profile Attributes as follows:

-   From the "Admin" section in Okta, open the settings page for your
    application.
-   In the "Provisioning" tab, scroll to the bottom and click the
    "Edit Attributes" button in the "User Attributes" section.
-   A "Profile Editor" screen will open, check the following settings:
    -   The "Display name" for the application
    -   The "Description"
    -   In the "Attributes" section, remove all attributes that are not
        supported by your application.

        This is an important step! Your users will get confused if your
        application appears to support attributes that are not
        supported by your SCIM API.

        You can delete an attribute by selecting it, then
        clicking the "Delete" button located in right hand attribute details pane.
        Before removing, check the mapping between Okta and Application and
        **remove the mappings** for the attribute(s) to be deleted.

##### Check the Attribute Mappings for Your Application

The last step for you to complete before submitting your
application to Okta is to check the User Profile Mappings for your
application. These mappings are what determine how profile
attributes are mapped to and from your application to an Okta
user's Universal Directory profile.

To check the User Profile Mappings for your application, do the
following:

-   From the "Admin" section in Okta, open the settings page for your
    application.
-   In the "Provisioning" tab, scroll to the bottom and click the
    "Edit Mappings" button in the "Attribute Mappings" section.
-   Check that each mapping is what you would expect it to be. Be
    sure to check both of the following:
    1.  From your application to Okta.
    2.  From Okta to your application.

##### Run the Second Set of Runscope Tests: Okta SCIM 2.0 CRUD Test

This is an important test that needs to be run in order to check if the Application 
can handle the **CR**eate, **U**pdate and **D**eactivate (CRUD) users functionality from Okta.
This allows the Application to be tested with actual Okta integration so thereby achieving end to end testing. 

The test follows this pattern:

 1. Check the Application added to the Okta org (Preview org or Developer account org)
 2. Adds a new user in Okta.
 3. Assign the user to the application.
 4. Validates in the Application if the user has been created.
 5. Update the user firstName attribute in Okta.
 6. Validates in the Application if the user attribute has been updated.
 7. Deactivate the user in Okta.
 8. Validates in the Application if the user has been deactivated.
 9. Reactivate the user in Okta
10. Re assign the Application to the user.
11. Validates the user creation/update in Application.

##### Prepare the Customer-Facing Configuration Guide

We recommend preparing the customer-facing configuration guide before beginning to work through the submission document. 
This guide will be exposed externally in Okta’s admin interface to end customers. 
For more details, see the [configuration guide guidelines](http://saml-doc.okta.com/Provisioning_Docs/SCIM_Configuration_Guide_Instructions.pdf).

Note: When you are ready, use [this form](https://docs.google.com/forms/d/e/1FAIpQLSfueMM9vbIt6zXCdcI9MdJ16lZc1FHwyIwMkHwNBWF7Kzdocg/viewform) to submit for Okta review.

#### Test with Customers

We require that one joint customer successfully uses and validates the integration in production before making it publicly available in the Okta Application Network. 

Here’s the process for getting joint customers involved in testing the newly developed SCIM integration:

* Identify joint customers interested in piloting the integration.
* Once identified, an Okta admin from the joint customer will need to email <developers@okta.com> asking for this integration to be assigned to their Okta preview org (please have them specify org subdomain in their request). Okta will then assign the app.
* Integration and configuration review with the joint customer. Partners are responsible for managing the customer identification and testing process, but Okta can support as needed. Please coordinate by emailing <developers@okta.com>.

#### Publish to Okta Application Network

In order for an app to be published in the Okta Application Network, it must meet the following criteria:

* ISV configuration guide explaining:
    * The supported features
    * Step-by-step instructions for setting up the integration
* Gotchas & known issues
* Support and Contact Info
* ISV Support Contact
* ISV Escalation Contact
* Full, permanent test tenant provided to Okta
* RunScope Test Suite
* Final Full QA by Okta
* Customer validation

## Provisioning FAQs

### SCIM Technical Questions

**What are the differences between SCIM 1.1 and 2.0?**

* Namespaces:  Namespaces are different, therefore 2.0 is not backwards compatible with 1.1

    | SCIM 1.1 | SCIM 2.0 |
    | --- | --- |
    | `urn:scim:schemas:core:1.0 urn:scim:schemas:extension:enterprise:1.0`  |  `urn:ietf:params:scim:schemas:core:2.0:User urn:ietf:params:scim:schemas:extension:enterprise:2.0:User` |

* Service Provider Configuration Endpoint: There's no `s` at the end of the SCIM 2.0 endpoint.

    | SCIM 1.1 | SCIM 2.0 |
    | --- | --- |
    | `/ServiceProviderConfigs` | `/ServiceProviderConfig` |

* Patch Protocol

    | SCIM 1.1 | SCIM 2.0 |
    | --- | --- |
    | [Section 3.3.2](http://www.simplecloud.info/specs/draft-scim-api-01.html#edit-resource-with-patch) | [Section 3.5.2: Uses JSON Patch](https://tools.ietf.org/html/rfc7644#section-3.5.2) |

* Error Response Schema

    | SCIM 1.1 | SCIM 2.0 |
    | --- | --- |
    | [Section 3.9](http://www.simplecloud.info/specs/draft-scim-api-01.html#anchor6) | [Section 3.12](https://tools.ietf.org/html/rfc7644#section-3.12) |

* Reference Type

    | SCIM 1.1 | SCIM 2.0 |
    | --- | --- |
    | N/A | Supports reference type pointing to the full URL of another SCIM Resource |

* Query by POST /search
    
    | SCIM 1.1 | SCIM 2.0 |
    | --- | --- |
    | N/A | [Section 3.4.3](https://tools.ietf.org/html/rfc7644#section-3.4.3) |

**Our API is similar to SCIM, but is not 100% compliant. Can we still integrate with Okta?**

Unfortunately, your app’s SCIM server API must be fully SCIM compliant in order to integrate with Okta. 
Okta’s SCIM client endpoints are hard coded into a template which adhere directly to [the SCIM spec](http://www.simplecloud.info/). 
Not all capabilities of the SCIM spec need to be supported (see [Required SCIM Server Capabilities](#required-scim-server-capabilities) in our SCIM Technical Reference) but the core schema and features do need to be supported.

**SCIM is a new standard. How broadly is it being adopted by cloud app vendors and how confident can I be in the SCIM standard’s long-term viability?**

Okta has seen significant SCIM momentum in the market amongst our network of app developers over the past year. 
Hot new apps like [Slack](https://api.slack.com/scim) 
and [Lucidchart](https://www.lucidchart.com/techblog/2016/08/04/an-implementers-overview-managing-cloud-identity-with-scim/) 
are supporting SCIM as well established software companies 
like [Huddle](https://github.com/Huddle/huddle-apis/wiki/Integrating%20with%20SCIM) 
and [Citrix](https://developer.citrixonline.com/implementing-scim-apis). 
Okta has doubled down on our investment in our SCIM server 
and launched our own SCIM provisioning developer program.
The SCIM standards is strong and is run by Salesforce, Google, and Sailpoint (Okta is also a contributor).

**How should I be managing authentication to my SCIM API?**

Okta recommends using the OAuth 2.0 Authorization Code Grant Flow (aka “three-legged OAuth). 
Okta doesn't support the Client Credentials or Resource Owner Password Credentials Authorization grant flows. 
The Authorization Code Grant Flow is more common in SaaS/cloud and is also more secure. 
In addition to OAuth, Okta also supports basic auth and header token auth options.

**I have a multi-tenant app how do I allow my customers to customize their specific tenant in Okta?**

Use the three-legged OAuth (Authorization Grant flow), 
so that you know exactly which token/key the customer is using. 
Another option is by URL. When the customer configures your app in Okta, we can prompt them to add their unique subdomain for your app  (see Zscaler app below). 
Okta can use part of this url in the SCIM endpoint for that customer, for example http://www.company.com/tenantA/scim or http://www.company.com/tenantB/scim). 
This subdomain field can be configured with Okta after you submit your app for Okta review.

![Example SCIM endpoint with subdomain](/assets/img/scim-scalar.png)

**Why do I need to implement the type attribute for attributes such as emails/phoneNumbers/addresses?**

The SCIM User Profile allows for an array of emails. The only way to differentiate between emails is to use the type sub-attribute. See section 2.4 of RFC 7643 for more details:
When returning multi-valued attributes, service providers SHOULD canonicalize the value returned (e.g., by returning a value for the sub-attribute “type”, such as “home” or “work”) when appropriate (e.g., for email addresses and URLs).
Service providers MAY return element objects with the same “value” sub-attribute more than once with a different “type” sub-attribute (e.g., the same email address may be used for work and home) but SHOULD NOT return the same (type, value) combination more than once per attribute, as this complicates processing by the client.
When defining schema for multi-valued attributes, it is considered a good practice to provide a type attribute that MAY be used for the purpose of canonicalization of values. In the schema definition for an attribute, the service provider MAY define the recommended canonical values (see Section 7).

**I only have one email/phone number/address in my user profile. Do I need to implement the array of emails/phone numbers/addresses?**

Yes, the you must return these fields in an array, which is specified in the SCIM spec as a multi-valued attribute: [Section 2.4](https://tools.ietf.org/html/rfc7643#section-2.4).

**Why doesn't Okta support DELETE /Users?**

Okta users are never deleted for compliance and audit purposes; they are deactivated instead. Because of this, Okta never makes an HTTP DELETE request to a user resource on your SCIM API. Instead, Okta makes an HTTP PATCH request to set the active setting to false. You’ll need to support the concept of an “active” and “inactive” user in your app.

**Will Okta be supporting the /Groups SCIM endpoint? When?**

Yes, Okta will eventually support the /Groups endpoint of the SCIM API. We are targeting early 2017 for this feature. In the meantime, to support the setting licensing / entitlements in your app without groups, you can use custom attributes in Okta to manage this. For examples, Lucidchart was able to support this use case with a user attribute called "License Type". See the “Extensibility” section of Lucidchart’s blog post for more details.

**How does data validation work with SCIM provisioning? For example, if my app requires phone number in a specific format, how do I ensure that Okta passes the attribute in that format? If a data validation error issue occurs how does error reporting work?**

The SCIM spec specifies valid data formats for a given user profile attribute, however Okta does not rigorously validate that the customer has inputted values meeting those requirements to preserve flexibility.
Therefore, data validation should be handled by your app’s SCIM Server. In other words, when Okta provisions user profile to your app, it should check that the data is valid per their special requirements. Error messages sent in the response from your app will be surfaced to Okta Admin via alerts and tasks in the Okta interface. You should also specify your data requirements in your config guide.

**How much filtering support is needed?**

The filtering capabilities in the SCIM protocol are pretty broad but the common filtering use case with Okta is quite narrow -- determine if a newly created Okta user already exists in your app based on a matching identifier . This means the eq (equals) operator is all you really need to support for now. We "might" eventually support other operators but don't right now. 
Note that Okta only supports filtering via the eq operator on the SCIM userName attribute on the SCIM Server side. However, Okta can use any AppUser attribute on the Okta side for passing into the eq operator. Typically this would also be `appuser.userName`, but `appuser.email` or `appuser.randomAttribute` can also work.

### Publishing Questions

**If I submit my app with a set of attributes, and then I want to add attributes during the testing phase of the app, is this acceptable?**

Yes. Add a new app instance in your dev org to test the new attributes and email <developers@okta.com>.

**Once my app has been published, and I add additional attributes, how do I republish my app? Can I republish frequently?**

Yes, you can republish your app, but we recommend you don’t do it frequently. Your app goes through Okta’s QA process every time you add additional attributes.

### Dev Doc Examples

Box - https://docs.box.com/docs/getting-started-box-integration
Slack - https://api.slack.com/
OneLogin - https://developers.onelogin.com/
Zendesk - https://developer.zendesk.com/apps

## Helpful Resources

https://www.lucidchart.com/techblog/2016/08/04/an-implementers-overview-managing-cloud-identity-with-scim/


https://support.okta.com/help/articles/Knowledge_Article/28352376-Overview-of-End-User-Management


https://support.okta.com/help/articles/Knowledge_Article/Provisioning-Concepts-and-Methods


https://developers.facebook.com/docs/facebook-at-work/provisioning/scim-api


http://openidexplained.com/


https://developers.onelogin.com/scim

## Appendix: Using the Example SCIM Server

Okta provides an [example SCIM Server](https://github.com/okta/okta-provisioning-sdk/tree/master/com/okta/scim/server/example) written in
Python, with [documentation](http://developer.okta.com/docs/sdk/opp/javadoc/overview-summary.html).

This example SCIM server demonstrates how to implement a basic SCIM
server that can create, read, update, and deactivate Okta users.

You can find the sample code to handle HTTP requests to this sample application in [Required SCIM Server Capabilities](#required-scim-server-capabilities) 
Use the instructions that follow to set up and run the example SCIM server.

### How to run

This example code was written for **Python 2.7** and does not
currently work with Python 3.

Here is how to run the example code on your machine:

First, start by doing a `git checkout` of this repository, then
`cd` to directory that `git` creates. Then, do the following:

1.  `cd` to the directory you just checked out:

        $ cd okta-scim-beta
2.  Create an isolated Python environment named `venv` using [virtualenv](http://docs.python-guide.org/en/latest/dev/virtualenvs/):

        $ virtualenv venv
3.  Next, activate the newly created virtual environment:

        $ source venv/bin/activate
4.  Then, install the dependencies for the sample SCIM server using
    Python's ["pip" package manager](https://en.wikipedia.org/wiki/Pip_%28package_manager%29):

        $ pip install -r requirements.txt
5.  Finally, start the example SCIM server using this command:

        $ python scim-server.py

### Introduction

Below are instructions for writing a SCIM server in Python, using
Flask and SQLAlchemy.

A completed version of this example server is available in this git
repository in the file named `scim-server.py`.

### Imports

We start by importing the Python packages that the SCIM server will
use:

    import os
    import re
    import uuid

    from flask import Flask
    from flask import render_template
    from flask import request
    from flask import url_for
    from flask_socketio import SocketIO
    from flask_socketio import emit
    from flask_sqlalchemy import SQLAlchemy
    import flask

### Setup

`re` adds support for regular expression parsing, `flask` adds the Flask
web framework, `flask_socketio` and `flask_sqlalchemy` add a idiomatic support for
their respective technologies to Flask.

Next we initialize Flask, SQLAlchemy, and SocketIO:

    app = Flask(__name__)
    database_url = os.getenv('DATABASE_URL', 'sqlite:///test-users.db')
    app.config['SQLALCHEMY_DATABASE_URI'] = database_url
    db = SQLAlchemy(app)
    socketio = SocketIO(app)

### SQLAlchemy support for the "users" table:

Below is the class that SQLAlchemy uses to give us easy access to
the "users" table.

The `update` method is used to "merge" or "update" a new User object
into an existing User object. This is used to simplify the code for
the code that handles PUT calls to `/Users/{id}`.

The `to_scim_resource` method is used to turn a User object into
a [SCIM "User" resource schema](https://tools.ietf.org/html/rfc7643#section-4.1).

    class User(db.Model):
        __tablename__ = 'users'
        id = db.Column(db.String(36), primary_key=True)
        active = db.Column(db.Boolean, default=False)
        userName = db.Column(db.String(250),
                             unique=True,
                             nullable=False,
                             index=True)
        familyName = db.Column(db.String(250))
        middleName = db.Column(db.String(250))
        givenName = db.Column(db.String(250))

        def __init__(self, resource):
            self.update(resource)

        def update(self, resource):
            for attribute in ['userName', 'active']:
                if attribute in resource:
                    setattr(self, attribute, resource[attribute])
            for attribute in ['givenName', 'middleName', 'familyName']:
                if attribute in resource['name']:
                    setattr(self, attribute, resource['name'][attribute])

        def to_scim_resource(self):
            rv = {
                "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
                "id": self.id,
                "userName": self.userName,
                "name": {
                    "familyName": self.familyName,
                    "givenName": self.givenName,
                    "middleName": self.middleName,
                },
                "active": self.active,
                "meta": {
                    "resourceType": "User",
                    "location": url_for('user_get',
                                        user_id=self.id,
                                        _external=True),
                    # "created": "2010-01-23T04:56:22Z",
                    # "lastModified": "2011-05-13T04:42:34Z",
                }
            }
            return rv

### Support for SCIM Query resources

We also define a `ListResponse` class, which is used to return an
array of SCIM resources into a
[Query Resource](https://tools.ietf.org/html/rfc7644#section-3.4.2).

    class ListResponse():
        def __init__(self, list, start_index=1, count=None, total_results=0):
            self.list = list
            self.start_index = start_index
            self.count = count
            self.total_results = total_results

        def to_scim_resource(self):
            rv = {
                "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
                "totalResults": self.total_results,
                "startIndex": self.start_index,
                "Resources": []
            }
            resources = []
            for item in self.list:
                resources.append(item.to_scim_resource())
            if self.count:
                rv['itemsPerPage'] = self.count
            rv['Resources'] = resources
            return rv

### Support for SCIM error messages

Given a `message` and HTTP `status_code`, this will return a Flask
response with the appropriately formatted SCIM error message.

By default, this function will return an HTTP status of "[HTTP 500
Internal Server Error](https://tools.ietf.org/html/rfc2068#section-10.5.1)". However you should return a more specific
`status_code` when possible.

See [section 3.12](https://tools.ietf.org/html/rfc7644#section-3.12) of [RFC 7644](https://tools.ietf.org/html/rfc7644) for details.

    def scim_error(message, status_code=500):
        rv = {
            "schemas": ["urn:ietf:params:scim:api:messages:2.0:Error"],
            "detail": message,
            "status": str(status_code)
        }
        return flask.jsonify(rv), status_code

### Socket.IO support

This sample application makes use of Socket.IO to give you a "real
time" view of SCIM requests that Okta makes to this sample
application.

When you load the sample application (the "/" route), your browser
will be sent a web application that uses Socket.IO to display
updates without the need for you to reload the page:

    @app.route('/')
    def hello():
        return render_template('base.html')

This page is updated using the functions below:

-   `send_to_browser` is syntactic sugar that will `emit` Socket.IO
    messages to the browser with the proper `broadcast` and
    `namespace` settings.
-   `render_json` is more syntactic sugar which is used to render
    JSON replies to Okta's SCIM client and `emit` the SCIM resource
    to Socket.IO at the same time.
-   `test_connect` is the function called with a browser first starts
    up Socket.IO, it returns a list of currently active users to the
    browser via Socket.IO.
-   `test_disconnect` is a stub that shows how to handle Socket.IO
    "disconnect" messages.

The code described above is as follows:

    def send_to_browser(obj):
        socketio.emit('user',
                      {'data': obj},
                      broadcast=True,
                      namespace='/test')


    def render_json(obj):
        rv = obj.to_scim_resource()
        send_to_browser(rv)
        return flask.jsonify(rv)


    @socketio.on('connect', namespace='/test')
    def test_connect():
        for user in User.query.filter_by(active=True).all():
            emit('user', {'data': user.to_scim_resource()})


    @socketio.on('disconnect', namespace='/test')
    def test_disconnect():
        print('Client disconnected')

### Socket.IO application

Below is the JavaScript that powers the Socket.IO application
described above. For the full contents of the HTML that this
JavaScript is part of, see the `base.html` file in the `templates`
directory of this project.

    $(document).ready(function () {
        namespace = '/test'; // change to an empty string to use the global namespace
        var uri = 'https://' + document.domain  + namespace;
        console.log(uri);
        var socket = io.connect(uri);

        socket.on('user', function(msg) {
            console.log(msg);
            var user = msg.data;
            var user_element = '#' + user.id
            var userRow = '<tr id="' + user.id + '"><td>' + user.id + '</td><td>' + user.name.givenName + '</td><td>' + user.name.familyName + '</td><td>' + user.userName + '</td></tr>';
            if($(user_element).length && user.active) {
                $(user_element).replaceWith(userRow);
            } else if (user.active) {
                $('#users-table').append(userRow);
            }

            if($(user_element).length && user.active) {
                $(user_element).show();
            }
            if($(user_element).length && !user.active) {
                $(user_element).hide();
            }
        });
    });

### Support for running from the command line

This bit of code allows you to run the sample application by typing
`python scim-server.py` from your command line.

This code also includes a `try/catch` block that creates all tables
of the `User.query.one()` function throws an error (which should
only happen if the User table isn't defined.

    if __name__ == "__main__":
        try:
            User.query.one()
        except:
            db.create_all()
        app.debug = True
        socketio.run(app)
