---
layout: post
title: First Post!
---

## Hey there, this is my first post so my blog doesn't look empty so I'm going to post random codeblocks (｡･ω･｡)ﾉ♡ ！！！

```cs
// This is some random codeblock for an ASP.NET Core application\

services
    .AddMvc(options =>
    {
        options.RespectBrowserAcceptHeader = true;
        options.ReturnHttpNotAcceptable = true;

        // Register the XML input/output formatters
        options.InputFormatters.Add(new XmlDataContractSerializerInputFormatter(options));
        options.OutputFormatters.Insert(3, new XmlDataContractSerializerOutputFormatter(new XmlWriterSettings()
        {
            NamespaceHandling = NamespaceHandling.OmitDuplicates,
            OmitXmlDeclaration = false,
        }));
    })
    .AddJsonOptions(options =>
    {
        // Register the contract resolver (settings to serialize) with the JSON formatter.
        options.SerializerSettings.ContractResolver = new DefaultContractResolver();
    });

```

```cs
// This is a class used in an ASP.NET Core application for validating Google reCaptcha.

public class CaptchaService : ICaptchaService
{
    private IHttpClientFactory _httpClientFactory;
    private CaptchaConfiguration _options;
    private HttpClient _client;

    public CaptchaService(IOptions<CaptchaConfiguration> options, IHttpClientFactory httpClientFactory)
    {
        _options = options.Value;
        _httpClientFactory = httpClientFactory;
        _client = _httpClientFactory.CreateClient();
    }

    public bool ValidateCaptcha(string key, string remoteIp) =>
        ValidateCaptchaAsync(key, remoteIp).GetAwaiter().GetResult();

    public bool ValidateCaptcha(string key, string remoteIp, string secret) =>
        ValidateCaptchaAsync(key, remoteIp, secret).GetAwaiter().GetResult();

    public Task<bool> ValidateCaptchaAsync(string key, string remoteIp) =>
        ValidateCaptchaAsync(key, remoteIp, _options.SecretKey);

    public async Task<bool> ValidateCaptchaAsync(string key, string remoteIp, string secret)
    {
        if (!_options.Enabled)
            return true;

        if (string.IsNullOrWhiteSpace(key) || string.IsNullOrWhiteSpace(secret))
            return false;

        string url = _options.VerificationUrl + "?secret=" + secret + "&response=" + key;

        if (!String.IsNullOrWhiteSpace(remoteIp) && _options.VerifyIp)
            url += "&remoteip=" + remoteIp;

        using (var req = new HttpRequestMessage(HttpMethod.Post, url))
        {
            HttpResponseMessage res = await _client.SendAsync(req);
            CaptchaModel model = await res.Content.ReadAsAsync<CaptchaModel>();

            return model.Success;
        }
    }
}
```

*よし！*, thanks for reading this little meme. Be sure to check out my [about page](/about) !! *さよなら。*
