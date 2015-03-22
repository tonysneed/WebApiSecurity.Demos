Katana Token Auth Demo ReadMe

This demonstrates how to perform basic authentication with Katana middleware.

NOTE: Here are steps taken to set up IIS hosting with OWIN.

SETUP: Katana token service, relying party, client

1. Create an Emtpy Web project and add the following NuGet packages:
   - Microsoft.Owin.Host.SystemWeb
   - Microsoft.AspNet.WebApi.Owin
   - Microsoft.Owin.Security.OAuth

2. Add a DemoAuthorizationServerProvider to a Providers folder
   - Extend OAuthAuthorizationServerProvider
   - Add a Credential validator property and init from ctor
   - Override GrantResourceOwnerCredentials
	 > Validate credentials, create identity, create ticket
   - Override ValidateClientAuthentication to call context.Validated and return null
     > OAuth has concept of client auth, which we are not using

3. Add a Startup class to the project.
   - Use VS item template, which adds the Startup attribute

    public void Configuration(IAppBuilder app)
    {
        // Use oauth authz server to issue tokens
        app.UseOAuthAuthorizationServer(new OAuthAuthorizationServerOptions
        {
            AllowInsecureHttp = true, // Set to false for production
            TokenEndpointPath = new PathString("/token"),
            AccessTokenExpireTimeSpan = TimeSpan.FromHours(8),
            Provider = new DemoAuthorizationServerProvider(Validator)
        });
    }

4. Add a RelyingParty emtpy Web project with the following NuGet packages:
   - Microsoft.Owin.Host.SystemWeb
   - Microsoft.AspNet.WebApi.Owin
   - Microsoft.Owin.Security.OAuth
   - Microsoft.Owin.Diagnostics

5. Add a Startup class to the RP app
   - Add config method for consuming bearer tokens

    public void Configuration(IAppBuilder app)
    {
        // Consume bearer tokens
        var options = new OAuthBearerAuthenticationOptions();
        app.UseOAuthBearerAuthentication(options);

        // Configure web api routing
        var config = new HttpConfiguration();
        config.MapHttpAttributeRoutes();
        config.Routes.MapHttpRoute(
            "DefaultApi",
            "api/{controller}/{id}",
            new { id = RouteParameter.Optional });
        app.UseWebApi(config);

        // Add error, welcome pages
        app.UseErrorPage();
        app.UseWelcomePage();
    }

