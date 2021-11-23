# MinimalAPI-Swagger-NSwag-GeneratedClient-NET60

This sample demonstrates how to generate a Swagger/OpenAPI specification when using the new [Minimal APIs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-6.0) (`app.Map*`) from .NET 6.0.

Once we have a swagger/OpenAPI specification, we can generate a Strongly-Typed C# Client that we can consume in our .NET applications (typically Blazor Wasm).

Let's begin!
## Create a new WebAPI project

Create a new WebAPI project:

```ps1
dotnet new webapi -n minimal-swagger
```

We want to run a couple of post build steps, but only when the current build is successful.

Add to your `.csproj`:

```xml
<PropertyGroup>
	<RunPostBuildEvent>OnBuildSuccess</RunPostBuildEvent>
</PropertyGroup>
```

## Install SwashBuckle ASP.NET Core CLI

Now we need to install the SwashBuckle ASP.NET Core CLI tool.

First we need to create a tools manifest:

```ps1
dotnet new tool-manifest
```

Then we can install the SweashBuckle CLI:

```ps1
dotnet tool install SwashBuckle.AspNetCore.Cli
```

> Notice the entry in our `dotnet-tools.json` file for this tool.

Now we are able to generate the `swagger.json` file after each successful build:

Add the following to your `.csproj` file:

```xml
<Target Name="GenerateSwagger" AfterTargets="PostBuildEvent">
	<Exec Command="dotnet tool restore" />
	<Exec Command="dotnet swagger tofile --output swagger.json $(OutputPath)\$(AssemblyName).dll v1 " />
</Target>
```

On each build we will generate a fresh `swagger.json` file.

## Generate C# Client using NSwag

Once we have our `swagger.json` file, we can use [NSwag](https://github.com/RicoSuter/NSwag) to generate a Strongly Typed C# Client.

The NSwag MSBuild tool allows us to run the NSwag CLI after each build. 

Add the NSwag MSBuild tool as package reference to our project:

```ps1
dotnet add package NSwag.MSBuild
```

Once we have NSwag.MSBuild installed, we also need an [NSwag configuration file](https://github.com/RicoSuter/NSwag/wiki/NSwag-Configuration-Document) (`nswag.json`).

You can generate a fresh config file using [NSwag Studio](https://github.com/RicoSuter/NSwag/wiki/NSwagStudio). 

Alternatively you can generate a new config file via the NSwag CLI (via [NPM](https://www.npmjs.com/package/nswag)).

Once you have a configuration file we can add the following to our `.csproj` to generate a strong type C# client:

```xml
<Target Name="NSwag" AfterTargets="GenerateSwagger">
	<Message Importance="High" Text="$(NSwagExe_Net60) run nswag.json /variables:Configuration=$(Configuration)" />
	<Exec WorkingDirectory="$(ProjectDir)" EnvironmentVariables="ASPNETCORE_ENVIRONMENT=Development" Command="$(NSwagExe_Net60) run nswag.json /variables:Configuration=$(Configuration)" />
	<Delete Files="$(ProjectDir)\obj\$(MSBuildProjectFile).NSwag.targets" /> <!-- This thingy trigger project rebuild -->
</Target>
```

Now after each build you will notice a freshly generated `.cs` file at the path specified in your `nswag.json` file.

DONE!

TODO: Add the Blazor Wasm sample app :)