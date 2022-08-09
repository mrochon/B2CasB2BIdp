# B2C as Direct Federation for AAD

## Summary

Uses AAD B2B federation to make a B2C tenant into an IdP for AAD.

## Known limitations

1. User must be invited to AAD is an email address whose domain corresponds to what's registered in AAD (here yourb2ctenant.b2clogin.com)
2. In B2C, the user needs to be given that email address as an attribute (another identity? - user does not need to know it)
3. AAD applications need to discover a user is from B2C and include the fake email as login_hint when requesting tokens from AAD

## Setup
### Create a SAML custom journey in B2C

Using [IefPolicies](https://github.com/mrochon/IEFPolicies), in VS.Code, Terminal Window:

```PowerShell
cd <empty work folder>
connect-iefpolicies <yourtenant, onmicrosoft.com not required>
new-iefpolicies 
# Select a starter pack, e.g. Local
# Show in File View
code -a .
New-IefPoliciesSamlRP
# Modify RP policy as per below
Import-IefPolicies
```

This should produce the url of your B2C metadata. You will need it later. Download the metadata to a file.

Modify the RP policy to produce user's email address in the corretc format.
(**Note**: *currently, hardcoded to a specific email address. Needs some work to either store the B2B email as another attribute and use it or create it in some other way.*)

```xml
    <BuildingBlocks>
        <ClaimsSchema>
            <ClaimType Id="email">
                <DefaultPartnerClaimTypes>
                    <Protocol Name="OAuth2" PartnerClaimType="email" />
                    <Protocol Name="OpenIdConnect" PartnerClaimType="email" />                    
                    <Protocol Name="SAML2" PartnerClaimType="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress" />
                  </DefaultPartnerClaimTypes>                
            </ClaimType>
        </ClaimsSchema>
    </BuildingBlocks>
    ...
                <OutputClaim ClaimTypeReferenceId="email" DefaultValue="user@yourb2ctenant.onmicrosoft.com" AlwaysUseDefaultValue="true" />

```


### Create a SAML app in B2C

Use AAD portal view and Enteprise Apps to register AAD as SAML RP in B2C. B2C does not allow use of trailing '/' in app iddentifier Uris, which is used by AAD federation (e.g. https://login.microsoftonline.com/<tenant ID>/). Follow [AAD docs](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/direct-federation#required-saml-20-attributes-and-claims) otherwise.

### Register B2C as IdP in AAD

Add new SAML2 Direct Federation. 

| Attribute | Value |
|---|---|
| Domain | yourtenant.b2clogin.com |
| Passive authn ep | https://yourtenant.b2clogin.com/yourtenant.onmicrosoft.com/B2C_1A_<saml policy name>/samlp/sso/login |
| Certificate| Should come in metadata but may need to be copied from B2C |
| Metadata | Download from url returned by New-IefPoliciesSamlRp above |

### Invite user to AAD

1. The user needs to be invited with email whose domain as @yourb2ctenant.b2clogin.com, no matter what the actual email of the user is.
2. Apps in AAD need to include login_hint=<fake email> when authenticating the user. Using domain_hint seems not to work.



