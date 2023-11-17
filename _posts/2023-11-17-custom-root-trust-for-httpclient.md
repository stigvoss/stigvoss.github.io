---
layout: post
title:  "Custom Root Trust with HttpClient in .NET"
date:   2023-11-17 20:30:00 +0100
categories: dotnet security certificates self-signed
---

Custom root trust is a security practice where an application or system is configured to trust 
only a specific set of root certificates. This approach restricts the application's trust to a 
limited, predefined list of Certificate Authorities, enhancing security by preventing reliance 
on external or unverified certificates. The practise is also use in a corporate setting to limit
trust to an internal root certificate.

Custom root trust is similar to, but differs from Certificate Pinning, where Certificate Pinning is
the act of trusting a specific certificate and custom root trust is trusting all certificates issued
by the trusted root certificate.

Implementing custom root trust has been pretty straight forward with `HttpClient` ever since .NET 5.

```csharp
X509Certificate2 trustedRoot = new X509Certificate2("corp-root-ca.pem");

HttpClientHandler handler = new()
{
    ServerCertificateCustomValidationCallback = (_, certificate, chain, errors) =>
    {
        // If errors are present, but they are not Remote Certificate Chain Errors, fail the validation.
        if ((errors & ~SslPolicyErrors.RemoteCertificateChainErrors) != 0)
        {
            return false;
        }

        // Add intermediate certificates from the server's chain for the verification process.
        foreach (X509ChainElement intermediateElements in chain.ChainElements.Skip(1))
        {
            chain.ChainPolicy.ExtraStore.Add(intermediateElements.Certificate);
        }

        // Clear any potential existing trusted root certificates.
        chain.ChainPolicy.CustomTrustStore.Clear();
        // Enforce Custom Root Trust.
        chain.ChainPolicy.TrustMode = X509ChainTrustMode.CustomRootTrust;
        // Add desired root certificate.
        chain.ChainPolicy.CustomTrustStore.Add(trustedRoot);
        
        // Verify the validity of the chain of trust and the server's certificate
        return chain.Build(certificate);
    }
};
HttpClient client = new(handler);
HttpResponseMessage response = await client.GetAsync("https://intranet.corp.local");
```