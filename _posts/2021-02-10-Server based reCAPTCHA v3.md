---
layout: post
author: Cody Merritt Anhorn
title: Server based reCAPTCHA v3
categories: [blog]
tags: [.NET5, Blazor]
---

In this article I will show how to use reCAPTCHA v3, reCAPTCHA is used to protect your forms from bots, or not read the article to find out more.

## What is reCAPTCHA v3

The main reason to use reCAPTCHA is to protect forms from bots, meaning you don't want non-human users from submitting your forms and flooding your database with junk data. Another added benefit of using v3 is that you can get light analytics of actions users interact on your site with. This has the added affect of telling you where you might be getting a lot of traffic, and where you might need to change flows to better interact with users.

What I like with the usage of reCAPTCHA v3 is because it is very hands off, only needing to take the users token and calling the reCAPTCHA API for a human score for the token. A difference from v2 is that the user does not need to click on pictures, or fill out some badly written text to be validated. I assume reCAPTCHA uses Machine Learning and Google tracking to validate that a user is a human or bot, giving you some neat aspect to your application by saying it uses Machine Learning.

## reCAPTCHA v3 Example

Below are some source using .NET5 and the Blazor Interop to call out to the Google reCAPTCHA Apis to validate a user is human.

<a href="https://developers.google.com/recaptcha/docs/v3" target="_blank">Google reCAPTCHA v3 Docs</a>

~~~ csharp

public class StandardCaptchaSettings
    : CaptchaSettings
{
    public string SiteKey { get; }
    public string SiteScriptUrl { get; }
    public string ApiUrl { get; }
    public string Secret { get; }
    public decimal Threshold { get; }

    public StandardCaptchaSettings(
        string siteKey, 
        string siteScriptUri, 
        string apiUrl,
        string secret,
        decimal threshold
    )
    {
        SiteKey = siteKey;
        SiteScriptUrl = siteScriptUri;
        ApiUrl = apiUrl;
        Secret = secret;
        Threshold = threshold;
    }
}

/// <summary>
/// This is the API result from the /recaptcha/api/siteverify call
/// </summary>
public class CaptchaValidationResult
{
    public bool Success { get; set; }
    public decimal Score { get; set; }
    public string Action { get; set; }
    [JsonPropertyName("challenge_ts")]
    public DateTimeOffset ChallengeTimestamp { get; set; }
    public string Hostname { get; set; }
    [JsonPropertyName("error-codes")]
    public string[] ErrorCodes { get; set; }
}

public class StandardCommandResult
{
    public bool Success { get; }
    public string ErrorCode { get; }

    public StandardCommandResult()
    {
        Success = true;
        ErrorCode = string.Empty;
    }

    public StandardCommandResult(string errorCode)
    {
        Success = success;
        ErrorCode = errorCode ?? string.Empty;
    }
}

public interface CaptchaService
{
    Task<CommandResult<CaptchaValidationResult>> ValidateToken(
        string token,
        CancellationToken cancellationToken
    );
}

public class ValidateCaptchaCommandHandler
    : IRequestHandler<ValidateCaptchaCommand, StandardCommandResult>
{
    private readonly Lazy<Task<IJSObjectReference>> _moduleTask;

    private readonly CaptchaSettings _captchaSettings;
    private readonly CaptchaInterop _captchaInterop;
    private readonly CaptchaService _captchaService;

    public ValidateCaptchaCommandHandler(
        IJSRuntime jsRuntime,
        CaptchaSettings captchaSettings,
        CaptchaInterop captchaInterop,
        CaptchaService captchaService
    )
    {
        _captchaSettings = captchaSettings;
        _captchaInterop = captchaInterop;
        _captchaService = captchaService;

        _moduleTask = new(
            () => jsRuntime.InvokeAsync<IJSObjectReference>(
                "import", 
                "./_content/Platform.Captcha/platform-captcha.js"
            ).AsTask()
        );
    }

    public async Task<StandardCommandResult> Handle(
        ValidateCaptchaCommand request,
        CancellationToken cancellationToken
    )
    {
        var token = await ExecuteClientReCaptcha(
            request.ActionName,
            _captchaSettings.SiteKey,
            cancellationToken
        );

        var validatedToken = await _captchaService.ValidateToken(
            token,
            cancellationToken
        );
        if (!validatedToken.Success)
        {
            return new StandardCommandResult(
                validatedToken.ErrorCode
            );
        }
        if (!validatedToken.Result.Success)
        {
            return new StandardCommandResult(
                string.Join(
                    ",",
                    validatedToken.Result.ErrorCodes
                )
            );
        }
        // Validate Score threshold
        if (validatedToken.Result.Score < _captchaSettings.Threshold)
        {
            return new StandardCommandResult(
                "TOKEN_THRESHOLD_TOO_LOW"
            );
        }

        return new StandardCommandResult();
    }

    public async ValueTask<string> ExecuteClientReCaptcha(
        string actionName,
        string siteKey,
        CancellationToken cancellationToken
    )
    {
        var module = await moduleTask.Value;
        return await module.InvokeAsync<string>(
            "execute",
            cancellationToken,
            actionName,
            siteKey
        );
    }
}

public class CaptchaServiceHttpClient
    : CaptchaService
{
    private readonly ILogger _logger;
    private readonly HttpClient _httpClient;
    private readonly IHttpContextAccessor _httpContextAccessor;
    private readonly CaptchaSettings _captchaSettings;

    public CaptchaServiceHttpClient(
        ILogger<CaptchaServiceHttpClient> logger,
        HttpClient httpClient,
        IHttpContextAccessor httpContextAccessor,
        CaptchaSettings captchaSettings
    )
    {
        _logger = logger;
        _httpClient = httpClient;
        _httpContextAccessor = httpContextAccessor;
        _captchaSettings = captchaSettings;
    }

    public async Task<CommandResult<CaptchaValidationResult>> ValidateToken(
        string token,
        CancellationToken cancellationToken
    )
    {
        try
        {
            var remoteIp = _httpContextAccessor.HttpContext.Connection.RemoteIpAddress.ToString();
            var content = new FormUrlEncodedContent(
                new Dictionary<string, string>
                {
                    ["secret"] = _captchaSettings.Secret,
                    ["response"] = token,
                    ["remoteip"] = remoteIp,
                }
            );
            var clientResponse = await _httpClient.PostAsync(
                    $"/recaptcha/api/siteverify",
                    content,
                    cancellationToken
                );
            var result = await clientResponse.Content.ReadFromJsonAsync<CaptchaValidationResult>(
                cancellationToken: cancellationToken
            );
            return new CommandResult<CaptchaValidationResult>(result);
        }
        catch (Exception ex)
        {
            _logger.LogError(
                ex,
                "Failed to validate Captcha token: {CaptchaToken}",
                token
            );
            return new CommandResult<CaptchaValidationResult>(CaptchaErrorCodes.API_ERROR);
        }
    }
}
~~~

Below is the JavaScript that the Interop Layer will call out to getting the token for the current user session.

~~~ js
export function execute(actionName, siteKey) {
    return new Promise((resolve, _) => {
        grecaptcha.ready(function () {
            grecaptcha.execute(siteKey, { action: actionName })
                .then(function (token) {
                    resolve(token);
                });
        });
    });
}
~~~