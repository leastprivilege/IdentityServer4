Overview
========
The following Identity Server 4 quickstarts provide step by step instructions for various common IdentityServer scenarios.
They start with the absolute basics and become more complex as they progress. We recommend that you follow them in sequence.

1. `Protecting an API using Client Credentials]<https://identityserver4.readthedocs.io/en/latest/quickstarts/1_client_credentials.html>`_
2. `Protecting an API using Passwords]<https://identityserver4.readthedocs.io/en/latest/quickstarts/2_resource_owner_passwords.html>`_
3. `Adding User Authentication with OpenID Connect]<https://identityserver4.readthedocs.io/en/latest/quickstarts/3_interactive_login.html>`_
4. `Adding Support for External Authentication]<https://identityserver4.readthedocs.io/en/latest/quickstarts/4_external_authentication.html>`_
5. `Switching to Hybrid Flow and adding API Access back]<https://identityserver4.readthedocs.io/en/latest/quickstarts/5_hybrid_and_api_access.html>`_
6. `Adding a JavaScript client]<https://identityserver4.readthedocs.io/en/latest/quickstarts/6_javascript_client.html>`_
7. `Using EntityFramework Core for configuration and operational data]<https://identityserver4.readthedocs.io/en/latest/quickstarts/7_entity_framework.html>`_
8. `Using ASP.NET Core Identity]<https://identityserver4.readthedocs.io/en/latest/quickstarts/8_aspnet_identity.html>`_

Every quickstart has a referencing solution - you can find the code for these solutions in the 
`IdentityServer4.Samples <https://github.com/IdentityServer/IdentityServer4.Samples>`_
repo in the quickstarts folder.

After having followed all of the above quickstarts you should be familiar with the following key consents of identity server 4.

* Adding IdentityServer to an ASP.NET Core application
* Configuring IdentityServer
* Issuing tokens for various clients
* Securing web applications and APIs
* Adding support for EntityFramework based configuration
* Adding support for ASP.NET Identity
* Adding Admin UI community edition to manage users and configuration

Preparation
^^^^^^^^^^^
In order to make following these quickstarts easer for you the team has created some custom templates for dotnet new.  

To install the custom templates open a console window and type the following command::

    dotnet new -i IdentityServer4.Templates

They will be used as a starting point for the various tutorials.

OK - let's get started!
