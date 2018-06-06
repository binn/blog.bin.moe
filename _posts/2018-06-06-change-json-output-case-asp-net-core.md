---
layout: post
title: Changing the case of ASP.NET Core's JSON output.
description: How to change the output casing/style of the JSON serialized by ASP.NET Core
tags: camel-case snake-case pascal-case json asp.net .net-core asp.net-core format
---

## ASP.NET Core and JSON

ASP.NET Core **is very compatible with JSON inputs and outputs.** When working with JSON responses in ASP.NET Core, we as developers would like a way to adjust the way our JSON looks.

As announced [here](https://github.com/aspnet/Announcements/issues/194), in June 2016, *Microsoft decided to have MVC return JSON as camel case by default.* This was decided after an issue was opened [here](https://github.com/aspnet/Mvc/issues/4283).

You'll have <ins>noticed</ins> that when migrating from ASP.NET Web API to ASP.NET Core that the default casing of the JSON output is now camel case.

## What are casings?

The style of JSON can be adjusted by something called the "case." It's similar to our understanding of casings in the English language.

There are three common casings for JSON:

1. Camel Case - Represented as `camelCaseStyleHere`
2. Pascal Case - Represented as `PascalCaseStyleHere`
3. Snake Case - Represented as `snake_case_style_here`

## How can I change the output format?

In all ASP.NET Core applications, there is a `ConfigureServices` method in a "startup" class. Find this method, and you should see `services.AddMvc()` being called there. This is the code we're going to adjust.

By default, if you leave it as `services.AddMvc()`, you'll get `camelCase` JSON as the output.

### 1. Change back to pascal case.

```cs
public void ConfigureServices(IServiceCollection services)
{
    // The default contract resolver lets the JSON serializer use the default output naming strategy, which is pascal case.
    services
        .AddMvc()
        .AddJsonOptions(options => options.SerializerSettings.ContractResolver = new DefaultContractResolver());
}
```

### 2. Change to snake case.

For snake case, it's a bit different than normal. We'll have to create our own instance of `DefaultContractResolver` and also set the naming strategy for it to be snake case.

You might need to include a `using Newtonsoft.Json.Serialization` statement at the top of your code file.

You'll start by creating your own instance of `DefaultContractResolver`:

```cs
// We've created our own instance of DefaultContractResolver here.
DefaultContractResolver contractResolver = new DefaultContractResolver()
{
    NamingStrategy = new SnakeCaseNamingStrategy(); // We'll tell JSON.NET to use snake case right here.
};
```

Now one might ask, *"Isn't `DefaultContractResolver` for pascal case?"* Not exactly, DefaultContractResolver has a property called `NamingStrategy`. This property by default is set to `DefaultNamingStrategy`, which keeps the original property names, most likely pascal case in C#.

There are three total naming strategies:

1. `DefaultNamingStrategy`
2. `CamelCaseStrategy`
3. `SnakeCaseNamingStrategy`
4. ~~`WaifuNamingStrategy`~~ (This doesn't actually exist, 😋)

Now then, you can go ahead and use our instance of the `DefaultContractResolver` instead of creating a new one. It should look like this

```cs
services
    .AddMvc()
    .AddJsonOptions(options => options.SerializerSettings.ContractResolver = contractResolver); // Right here we tell MVC to use that contract resolver we just defined.
```

Your finalized block should look like this:

```cs
public void ConfigureServices(IServiceCollection services)
{
    // We've created our own instance of DefaultContractResolver here.
    DefaultContractResolver contractResolver = new DefaultContractResolver()
    {
        NamingStrategy = new SnakeCaseNamingStrategy(); // We'll tell JSON.NET to use snake case right here.
    };

    // (`･ω･´)b

    services
        .AddMvc()
        .AddJsonOptions(options => options.SerializerSettings.ContractResolver = contractResolver); // Right here we tell MVC to use that contract resolver we just defined.
}
```

### 3. Change to camel case.

Most likely you're already on camel case, but if not, you can switch to camel case easily by following similar steps as before.
There are three ways to do this, I'll be showing you two of them.

#### Method 1 - Use `CamelCaseContractResolver`

A simple contract resolver exists for camel case outputs. It's called `CamelCaseContractResolver`

To use it, just repeat the steps for using pascal case, except replace `DefaultContractResolver` with `CamelCaseContractResolver`.

```cs
public void ConfigureServices(IServiceCollection services)
{
    // The camel case contract resolver lets the JSON serializer use the camel case output naming strategy, which is camel case.
    services
        .AddMvc()
        .AddJsonOptions(options => options.SerializerSettings.ContractResolver = new CamelCaseContractResolver()); // Easy enough, we have camel case!
}
```

#### Method 2 - Use `CamelCaseNamingStrategy`

So, you remember how we switched to snake case right? Because this is just that, except we'll use `CamelCaseNamingStrategy` instead.

You'll need to set the `NamingStrategy` of your instance of `DefaultContractResolver` to `CamelCaseNamingStrategy`, as shown in the example below.

```cs
// We've created our own instance of DefaultContractResolver here.
DefaultContractResolver contractResolver = new DefaultContractResolver()
{
    NamingStrategy = new CamelCaseNamingStrategy(); // We'll tell JSON.NET to use camel case right here.
};
```

Then, you can go ahead and add it into `AddJsonOptions()`

```cs
services
    .AddMvc()
    .AddJsonOptions(options => options.SerializerSettings.ContractResolver = contractResolver); // Right here we tell MVC to use that contract resolver we just defined.
```

Finally, your code should look like this:

```cs
public void ConfigureServices(IServiceCollection services)
{
    // We've created our own instance of DefaultContractResolver here.
    DefaultContractResolver contractResolver = new DefaultContractResolver()
    {
        NamingStrategy = new SnakeCaseNamingStrategy(); // We'll tell JSON.NET to use snake case right here.
    };

    // (`･ω･´)b

    services
        .AddMvc()
        .AddJsonOptions(options => options.SerializerSettings.ContractResolver = contractResolver); // Right here we tell MVC to use that contract resolver we just defined.
}
```

## Final thoughts

This isn't just limited to ASP.NET Core. You can set the contract resolver of your own instance of `JsonSerializerSettings` and then pass it onto `JsonConvert.SerializeObject` for your own needs elsewhere!

With some fine pascal case formatting for you, I'm sure *Inko-chan* is now proud of you!

 — bin.