.. _refResosurceOwnerQuickstart:
Protecting an API using Passwords
=================================

The following Identity Server 4 quickstart provides step by step instructions for various common IdentityServer scenarios. These start with the absolute basics and become more complex as they progress. We recommend that you follow them in sequence.  To see the full list of please go to `IdentityServer4 Quickstarts Overview <https://identityserver4.readthedocs.io/en/latest/quickstarts/0_overview.html>`_

This is quickstart number two it walks you though the OAuth 2.0 resource owner password grant type.  This grant type allows a client to send username and password to the token service and get an access token back that represents that user.  The spec generally recommends against using the resource owner password grant besides legacy applications that cannot host a browser. Generally speaking you are typically far better off using one of the interactive OpenID Connect flows when you want to authenticate a user and request access tokens.

This grant type allows us to introduce the concept of users to our quickstart IdentityServer.   This quickstart assumes that you have already completed `Protecting an API using Client Credentials <https://identityserver4.readthedocs.io/en/latest/quickstarts/1_client_credentials.html>`_ if you have not we recommend that you go back and complete that first before moving on to this quickstart.

Source Code
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As with all of these quickstarts you can find the source code for it in the `IdentityServer4.Samples <https://github.com/IdentityServer/IdentityServer4.Samples>`_ project.  
The project for this quickstart is `Quickstart #2: Securing an API using the Resource Owner Password Grant <https://github.com/IdentityServer/IdentityServer4.Samples/tree/master/Quickstarts/2_ResourceOwnerPasswords>`_

Preparation
^^^^^^^^^^^
In order to make following these quickstarts easer for you the team has created some custom templates for dotnet new.  

To install the custom templates open a console window and type the following command::

    dotnet new -i IdentityServer4.Templates

They will be used as a starting point for the various tutorials.

Adding users
^^^^^^^^^^^^
Just like there are in-memory stores for resources (aka scopes) and clients, there is also one for users.

.. note:: Check the ASP.NET Identity based quickstarts for more information on how to properly store and manage user accounts.

This Quickstart uses the code that you created in the previous quickstart as a starting point.   Open the Identityserver4 project you created in the last quickstart and lets begin.


The class ``TestUser`` represents a test user and its claims. Let's create a couple of users by adding the following code to our config class:

First add the following using statement to the ``Config.cs`` file::

    using IdentityServer4.Test;

    public static List<TestUser> GetUsers()
    {
        return new List<TestUser>
        {
            new TestUser
            {
                SubjectId = "1",
                Username = "alice",
                Password = "password"
            },
            new TestUser
            {
                SubjectId = "2",
                Username = "bob",
                Password = "password"
            }
        };
    }

Then register the test users in the IdentityServer.  To do this open the Startup.cs file and add the following to the ConfigureServices method::

    public void ConfigureServices(IServiceCollection services)
    {
        // configure identity server with in-memory stores, keys, clients and scopes
        services.AddIdentityServer()
            .AddInMemoryApiResources(Config.GetApiResources())
            .AddInMemoryClients(Config.GetClients())
            .AddTestUsers(Config.GetUsers());
    }

The ``AddTestUsers`` extension method does a couple of things under the hood

* adds support for the resource owner password grant
* adds support to user related services typically used by a login UI (we'll use that in the next quickstart)
* adds support for a profile service based on the test users (you'll learn more about that in the next quickstart)

Adding a client for the resource owner password grant
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
You could simply add support for the grant type to our existing client by changing the
``AllowedGrantTypes`` property. If you need your client to be able to use both grant types
that is absolutely supported.

We are creating a separate client for the resource owner use case,
add the following to your clients configuration (Config.cs)::

    public static IEnumerable<Client> GetClients()
    {
        return new List<Client>
        {
            // other clients omitted...

            // resource owner password grant client
            new Client
            {
                ClientId = "ro.client",
                AllowedGrantTypes = GrantTypes.ResourceOwnerPassword,

                ClientSecrets =
                {
                    new Secret("secret".Sha256())
                },
                AllowedScopes = { "api1" }
            }
        };
    }

Requesting a token using the password grant
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Add a new console client to your solution.  For that, add a console project to your solution, remember to create it in the ``src``::

    dotnet new console -n ResourceOwnerClient
    
Then as before, add it to your solution using::

    cd ..
    dotnet sln add .\src\Client\ResourceOwnerClient.csproj
    
Open up ``Program.cs`` and copy the content from `here <https://github.com/IdentityServer/IdentityServer4.Samples/blob/master/Quickstarts/2_ResourceOwnerPasswords/src/ResourceOwnerClient/Program.cs>`_ to it..

The client program invokes the ``Main`` method asynchronously in order to run asynchronous http calls. This feature is possible since ``C# 7.1`` and will be available once you edit Client.csproj to add the following line as a ``PropertyGroup``::

    <LangVersion>latest</LangVersion>

The token endpoint at IdentityServer implements the OAuth 2.0 protocol, and you could use raw HTTP to access it. However, we have a client library called IdentityModel, that encapsulates the protocol interaction in an easy to use API.

Add the `IdentityModel` NuGet package to your client. 
This can be done either via Visual Studio's Nuget Package manager or though the package manager Console with the following command::

    Install-Package IdentityModel

or by using the CLI::

    dotnet add package IdentityModel


The new client looks very similar to what we did for the client credentials grant.
The main difference is now that the client would collect the user's password somehow,
and send it to the token service during the token request.

Again IdentityModel can help out here::

    // request token
    var tokenResponse = await client.RequestPasswordTokenAsync(new PasswordTokenRequest
    {
        Address = disco.TokenEndpoint,
        ClientId = "ro.client",
        ClientSecret = "secret",

        UserName = "alice",
        Password = "password",
        Scope = "api1"
    });

    if (tokenResponse.IsError)
    {
        Console.WriteLine(tokenResponse.Error);
        return;
    }

    Console.WriteLine(tokenResponse.Json);

When you send the token to the identity API endpoint, you will notice one small
but important difference compared to the client credentials grant. The access token will
now contain a ``sub`` claim which uniquely identifies the user. 
This "sub" claim can be seen by examining the content variable after the call to the API and also will be displayed on the screen by the console application.

The presence (or absence) of the ``sub`` claim lets the API distinguish between calls on behalf
of clients and calls on behalf of users.

Further experiments
^^^^^^^^^^^^^^^^^^^
This walkthrough focused on the success path so far

* user was able to login
* a token containg the users subject id was returned.
* client could use the token to access the API

You can now try to provoke errors to learn how the system behaves, e.g.

* try to connect to IdentityServer when it is not running (unavailable)
* try to use an invalid client id or secret to request the token
* try to ask for an invalid scope during the token request
* try to call the API when it is not running (unavailable)
* don't send the token to the API
* configure the API to require a different scope than the one in the token
* try logging in with an invalid user name and/or password.
