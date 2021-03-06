﻿# Webpack Development Server Support for ASP.NET Core 3.1

## Introduction

Packages `Microsoft.AspNetCore.NodeServices` and `Microsoft.AspNetCore.SpaServices` have been declared [deprecated in ASP.NET Core 3.1](https://github.com/dotnet/aspnetcore/issues/12890). This includes the extension method `ISpaBuilder.UseWebpackDevMiddleware()`.

Package `Microsoft.AspNetCore.SpaServices.Extensions` is recommended to be used instead.

The problem is that the recommended replacement does not offer a direct replacement for `ISpaBuilder.UseWebpackDevMiddleware()`. This leaves developers who would like to continue using this convenient method without a suitable solution.

This package - `RimuTec.AspNetCore.SpaServices.Extensions` - offers an extension method named `ISpaBuilder.UseWebpackDevelopmentServer()` that aims at providing a replacement that minimizes required code changes.

**Note:** This NuGet package does not offer a replacement for any other methods or packages that may have been declared deprecated. For example this NuGet package is _not_ a replacement for `Microsoft.AspNetCore.NodeServices`.

| Metric      | Status      |
| ----- | ----- |
| Nuget       | [![NuGet Badge](https://buildstats.info/nuget/RimuTec.AspNetCore.SpaServices.Extensions)](https://www.nuget.org/packages/RimuTec.AspNetCore.SpaServices.Extensions/) |

## Prerequisites

- Visual Studio 2019 Community Edition, 16.4.3 or later. Other versions may work, too, but weren't tested. ([Download](https://visualstudio.microsoft.com/downloads/))
- Nodejs 12.13.1 or later. Other versions may work as well but were not tested. ([Download](https://nodejs.org/en/download/))
- .NET Core 3.1.101 or later. Other versions may work, too, but weren't tested. ([Download](https://dotnet.microsoft.com/download/dotnet-core/3.1), however, this typically comes with Visual Studio 2019 already)

## Usage

The following steps assume that you have a single SPA in your project and that *all* files for the SPA are in a folder named `wwwroot` within your ASP.NET Core 3.1 project.

**Note:** The following are just the key steps (one-off) to add support for webpack dev server (WPS) with hot module replacement (HMR) to your projects. It does not describe the full content of all files involved. The [github repository](https://github.com/RimuTec/AspNetCore.SpaServices.Extensions) for the source code of this NuGet package contains the source code for a complete example application.

1. To use this extension in an ASP.NET Core 3.1 project, add the [NuGet package `RimuTec.AspNetCore.SpaServices.Extensions`](https://www.nuget.org/packages/RimuTec.AspNetCore.SpaServices.Extensions/) to the project, e.g. using the dotnet cli in a terminal window:

   ```dotnet
   dotnet add package Rimutec.AspNetCore.SpaServices.Extensions
   ```

2. In the file `Startup.cs`, in class `Startup` add the following code to method `ConfigureServices(IServiceCollection services)`:

    ```csharp
    public static void ConfigureServices(IServiceCollection services)
    {
        // ... other code left out for brevity

        services.AddSpaStaticFiles(configuration => 
        {
            // This is where files will be served from in non-Development environments
            configuration.RootPath = "wwwroot/dist"; 
        });

        // ... other code left out for brevity
    }
    ```
3. Then in file `Startup.cs` in class `Startup` in method `` add code as follows:

    ```csharp
    public static void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        // ... other code left out for brevity

        app.UseSpa(spa =>
        {
            spa.Options.SourcePath = "wwwroot/src"; // <-- this must be the same path as in the previous step

            if(env.IsDevelopment()) // "Development", not "Debug" !!
            {
                spa.UseWebpackDevelopmentServer(npmScriptName: "start");
            }
        });

        // ... other code left out for brevity

        app.UseStaticFiles(); // <-- this line may already exist
        app.UseSpaStaticFiles(); // <-- this line may already exist
    
        // ... other code left out for brevity
    }
    ```
    **Important:** The two terminal handlers - `UseStaticFiles()` and `UseSpaStaticFiles()` - must be added to the HTTP request pipeline **after** adding `UseWebpackDevelopmentServer()`. Otherwise the terminal handlers will try handling the requests for bundles, and as a consequence the webpack development server middleware will never see those requests. As a result hot module reloading (HMR) will not work.

4. Add a file `package.json` in folder `wwwroot` with the following content:

    ```json
    {
      // other content left out for brevity
      "scripts": {
        // your other scripts here
        "build": "cross-env NODE_ENV=development webpack --config ./webpack.config.js",
        "build:prod": "cross-env NODE_ENV=production webpack --config ./webpack.config.js",
        "start": "cross-env NODE_ENV=development webpack-dev-server --config ./webpack.config.js",
        ... your other scripts here ...
      },
      "keywords": [],
      "author": "RimuTec Ltd.",
      "license": "Apache-2.0",
      "devDependencies": {
        "@babel/core": "7.8.4",
        "@babel/preset-env": "7.8.4",
        "@babel/preset-react": "7.8.3",
        "@babel/preset-typescript": "7.8.3",
        "@types/react": "16.9.19",
        "@types/react-dom": "16.9.5",
        "babel-loader": "8.0.6",
        "clean-webpack-plugin": "3.0.0",
        "cross-env": "7.0.0",
        "html-loader": "0.5.5",
        "html-webpack-plugin": "3.2.0",
        "typescript": "3.7.5",
        "webpack": "4.41.5",
        "webpack-cli": "3.3.10",
        "webpack-dev-server": "3.10.3"
      },
      "optionalDependencies": {
        "fsevents": "1.2.9"
      },
      "_comment": "fsevent@1.2.9 is locked in to prevent broken builds on windows for v1.2.11, see https://github.com/fsevents/fsevents/issues/301",
      "dependencies": {
        "react": "16.12.0",
        "react-dom": "16.12.0",
        "react-hot-loader": "4.12.19"
      }
      // other content left out for brevity, please see repository for full source code
    }
    ```

5. In folder `wwwroot` add a file named `webpack.config.js` with the following content:

    ```javascript
    const isDevelopment = process.env.NODE_ENV !== 'production';
    const HtmlWebPackPlugin = require('html-webpack-plugin');
    const { CleanWebpackPlugin } = require("clean-webpack-plugin");

    module.exports = {
        module: {
            rules: [
                {
                    test: /\.(t|j)sx?$/,
                    loader: 'babel-loader',
                    exclude: /node_modules/
                },
                {
                    test: /\.html$/,
                    use: [
                        {
                            loader: 'html-loader',
                            options: { minimize: !isDevelopment }
                        }
                    ]
                }
            ]
        },
        devServer: {
            contentBase: './wwwroot/dist'
        },
        resolve: {
            extensions: ['.js', '.jsx', '.ts', '.tsx']
        }
        ,mode: isDevelopment ? 'development' : 'production'
        , output: {
            filename: isDevelopment ? '[name].js' : '[name].[hash].js',
            crossOriginLoading: "anonymous"
        },
        plugins: [
            new CleanWebpackPlugin(),
            new HtmlWebPackPlugin({
                template: './src/index.html',
                filename: './index.html'
            })
        ]
    };
    ```
    Note: You can use a folder name other than `wwwwroot`. If you, you need to update it in multiple places including the project file (csproj) where you need to set the property `SpaRoot` to the correct value.
6. Now edit your project file and add the following piece at the end of the file:

    ```xml
    <Project>

      <!-- other code left out for brevity -->

      <PropertyGroup>
        <!-- This is where we set the path for the SPA files (include trailing slash to avoid having to repeat it elsewhere): -->
        <SpaRoot>wwwroot/</SpaRoot> <!-- <<<<<<<<<<< THIS IS IMPORTANT! Also include trailing forward slash. -->
      </PropertyGroup>

      <Target Name="DebugEnsureNodeEnv" BeforeTargets="PreBuildEvent" Condition=" '$(Configuration)' == 'Debug' And !Exists('$(SpaRoot)node_modules') ">
        <!-- Ensure Node.js is installed -->
        <Exec Command="node --version" ContinueOnError="true">
          <Output TaskParameter="ExitCode" PropertyName="ErrorCode" />
        </Exec>
        <Error Condition="'$(ErrorCode)' != '0'" Text="Node.js is required to build and run this project. To continue, please install Node.js from https://nodejs.org/, and then restart your command prompt or IDE." />
        <!-- If file 'package-lock.json' exists, we use 'npm ci', otherwise 'npm install'. In both cases 
             use the 'no-optionals' option to avoid an issue with fsevent reported at
             https://github.com/fsevents/fsevents/issues/301 -->
        <PropertyGroup>
          <PackageLockFile>$(SpaRoot)package-lock.json</PackageLockFile>
        </PropertyGroup>
        <Message Condition="Exists($(PackageLockFile))" Importance="high" Text="Restoring dependencies using 'npm ci --no-optionals'. This may take several minutes..." />
        <Exec Condition="Exists($(PackageLockFile))" WorkingDirectory="$(SpaRoot)" Command="npm ci --no-optionals" />
        <Message Condition="!Exists($(PackageLockFile))" Importance="high" Text="Restoring dependencies using 'npm install --no-optionals'. This may take several minutes..." />
        <Exec Condition="!Exists($(PackageLockFile))" WorkingDirectory="$(SpaRoot)" Command="npm install --no-optionals" />
      </Target>

      <ItemGroup>
        <!-- Reference for Content tag at https://docs.microsoft.com/en-us/visualstudio/msbuild/common-msbuild-project-items?view=vs-2019#content
             and for None tag at https://docs.microsoft.com/en-us/visualstudio/msbuild/common-msbuild-project-items?view=vs-2019#none -->
        <Content Remove="$(SpaRoot)**" />
        <None Include="$(SpaRoot)src/**" />
        <None Include="$(SpaRoot)*" />
      </ItemGroup>

      <Target Name="BuildDev" AfterTargets="PostBuildEvent" Condition=" '$(Configuration)' == 'Debug' ">
        <!-- When building Debug bundle do not minify -->
        <Exec WorkingDirectory="$(SpaRoot)" Command="npm run build" />
      </Target>

      <Target Name="BuildProd" AfterTargets="PostBuildEvent" Condition=" '$(Configuration)' == 'Release' ">
        <!-- When building Release create production bundle as per webpack.config.js -->
        <Exec WorkingDirectory="$(SpaRoot)" Command="npm run build:prod" />
      </Target>

      <Target Name="PublishDistFiles" AfterTargets="ComputeFilesToPublish">
        <!-- Source for this target: https://stackoverflow.com/a/54725321/411428 -->
        <ItemGroup>
          <DistFiles Include="$(SpaRoot)dist/**" />
          <ResolvedFileToPublish Include="@(DistFiles->'%(FullPath)')" Exclude="@(ResolvedFileToPublish)">
            <RelativePath>%(DistFiles.Identity)</RelativePath>
            <CopyToPublishDirectory>Always</CopyToPublishDirectory>
          </ResolvedFileToPublish>
        </ItemGroup>
      </Target>
    </Project>
    ```

With this in place you should be able to compile, build and debug your project. Now if you change a file that is part of the bundle, webpack-dev-server will re-build the bundle in memory and send an update to the client browser. Obviously this also depends on the content of `package.json`, `webpack.config.js` and `tsconfig.json`.

Please be aware that it pays big times to study the full source code for this example at https://github.com/RimuTec/AspNetCore.SpaServices.Extensions.

## Sample Application

This repository contains a sample application named "SampleSpaWebApp". The sample application makes use of the NuGet package and includes all code required to get WDS (webpack dev server) with hot module replacement (HMR) working.

The sample application demonstrates the scenario for one single page application (SPA). We may add a sample application for multiple SPA at a later stage.

**Note:** Do not confuse the sample application with the project "Web" that is included in this repository as well. "Web" is used to develop the NuGet package. Althoughh it appears very similar it should not be used as a sample project for how to use the NuGet package.

## Troubleshooting

In case Hot Module Replacement (HMR) does not work, it can have several reasons. Typically there is only a small number of factors that need to be checked. Check and confirm these:

   - Ensure the path to your SPA is set correctly in all places, i.e. `webpack.config.js`, project file, two places in class `Startup`.
   - Ensure the project properties in the tab `Debug` have an environment variable named `ASPNETCORE_ENVIRONMENT` that is set to the value `Development`.
   - Make sure you call `UseWebpackDeveloperServer()` **BEFORE** `UseStaticFiles()`, `UseSpaStaticFiles()` and any other terminal handlers.


## Bug Reports and Other Suggestions for Improvement

Please create an issue at https://github.com/RimuTec/AspNetCore.SpaServices.Extensions/issues and provide as many details as possible to make sure the issue stands on its own. If you provide enough details other users can determine if they have a duplicate or whether they should file a new issue.

If you report a bug report the best option to get it resolved is a github repo with a minimal code base that reproduces the problem.

Keep in mind that support is provided only at https://github.com/RimuTec/AspNetCore.SpaServices.Extensions/issues and that it is "best effort". All contributors to this repository use their spare time to work on this NuGet package.

Pull Requests (PRs) are welcome. Please send them to the branch `master`. Thank you!

# Additional Background

With ASP.NET Core 3.1 the NuGet package Microsoft.AspNetCore.SpaServices has been declared [deprecated](https://github.com/dotnet/aspnetcore/issues/12890). This means that the useful function `UseWebpackDevMiddleware()` has been declared deprecated as well.

Create-React-App (CRA) assumes there is a single client app. CRA also requires you to "eject" your project to gain more flexibility. In other words CRA is great if you really can design your application such that everything can be done in a single page. 

However, there are projects that need more flexibility than CRA can offer. Furthermore, there are existing code bases where CRA is not a viable option with the deprecation of `UseWebpackDevMiddleware()`. This NuGet package - RimuTec.AspNetCore.SpaServices.Extensions - aims at providing a replacement that hopefully helps avoiding a significant amount of rework for those with existing code bases.

This extensions starts on the other end. It assumes that you start with a simple HTML page and a single bundle. A "hello, world"-example for Webpack if you like. You can then add as much or as little as you wish. This packages allows you to use hot-module-reloading (HMR) for your bundle.

# Advanced Scenarios

Work in progress.