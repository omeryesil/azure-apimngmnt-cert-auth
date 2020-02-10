# Azure Api Management - Authentication APIs with Client Certificates

Reference: <https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-mutual-certificates-for-clients>

- API Management provides the capability to secure access to APIs (i.e., **client to API Management**) using client certificates. You can validate incoming certificate and check certificate properties against desired values using policy expressions.

## Steps

### 1. Enable "Negotiate client certificate"

- From API Management, select "Custom Domains", then select "Negotiate client certificate"

  - By enabling the client certificate negotiation, the API management requires clients to send certificates in every call.
  - Note that this does not validate the certificate with CA authority(**?**). Check policy section for validation



### 2. Policy

Creating API Management policy to **verify** certificate and forward vertificate paramters to backend apps (APIs)

how context.Request.Certificate.Verify() verifies 
https://github.com/MicrosoftDocs/azure-docs/issues/35369


context.Request.Certificate.Verify() will check both its validity a revocation list.

```xml
<policies>
    <inbound>
        <base />
        <set-backend-service base-url="https://awapi-test-http.azurewebsites.net" />
        <choose>
            <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify() || context.Request.Certificate.Issuer != "trusted-issuer" || context.Request.Certificate.SubjectName.Name != "expected-subject-name")">
                <return-response>
                    <set-status code="403" reason="Invalid client certificate" />
                </return-response>
            </when>
        </choose>
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>


```




