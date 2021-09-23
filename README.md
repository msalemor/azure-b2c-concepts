# Azure B2C Concepts

## B2C Tenant

### Creation

- Make sure the X provider is registered
- Create a new B2C tenant in a resource group in a region
- Switch to the B2C
  - > Note: The B2C tenant

### Data soveranty

- Data in the B2C is deploy to that region
  - > Note: Keep in my data sovernty issues

## User Flows

### Sign-in/Sign-out (SUSI) flow

### Reset password flow

### Edit flow

### Properties

Allow javascript = true

### Run user flow (Testing)

- Create an app called jwttest
- Create an app registration
- Set the implicit flow (get the token)
- For the reply url use: jwt.ms

## Fields 

### Creating Fields

### Renaming Fields

### Rearranging Fields

## Page Customization

### Customization per flow

- Create HTML
- Deploy to a web site (can be a storage account or static pages)
- Point to the html on this site

## API

### API Usage

- Validation (such as during sign-up/sign-in)
- Uses the MS Graph

```c#
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Graph;
using Microsoft.Graph.Auth;
using Microsoft.Identity.Client;

namespace Verify_inator.Services
{
    public class B2cGraphService
    {
        private readonly string b2cExtensionPrefix;

        public B2cGraphService(string b2cExtensionsAppClientId)
        {
            this.b2cExtensionPrefix = b2cExtensionsAppClientId.Replace("-", "");
        }

        public string GetUserAttributeClaimName(string userAttributeName)
        {
            return $"extension_{userAttributeName}";
        }

        public string GetUserAttributeExtensionName(string userAttributeName)
        {
            return $"extension_{this.b2cExtensionPrefix}_{userAttributeName}";
        }

        private string GetUserAttribute(Microsoft.Graph.User user, string extensionName)
        {
            if (user.AdditionalData == null || !user.AdditionalData.ContainsKey(extensionName))
            {
                return null;
            }
            return (string)user.AdditionalData[extensionName];
        }
    }
}
```

Sample Validation
```c#
[HttpPost()]
        public IActionResult Post([FromBody] JsonElement body)
        {
            try
            {
                this._logger.LogInformation("A CMC Consultant ID code is being redeemed.");

                // Look up the invitation code in the incoming request.
                var cmcId = default(string);
                var territoryName = default(string);
                this._logger.LogInformation("Request properties:");
                foreach (var element in body.EnumerateObject())
                {
                    this._logger.LogInformation($"- {element.Name}: {element.Value.GetRawText()}");
                    // The element name should be the full extension name as seen by the Graph API (e.g. "extension_appid_InvitationCode").
                    if (element.Name.Equals(this._b2cGraphService.GetUserAttributeExtensionName(Constants.UserAttributes.ConsultantID), StringComparison.InvariantCultureIgnoreCase))
                    {
                        cmcId = element.Value.GetString();
                    }
                }

                if (string.IsNullOrWhiteSpace(cmcId) || !Regex.IsMatch(cmcId, CMCID_REGEX))
                {
                    this._logger.LogInformation($"The provided CMC ID \"{cmcId}\" is invalid.");
                    return GetValidationErrorApiResponse("UserInvitationRedemptionFailed-Invalid", "The invitation code you provided is invalid.");
                }
                else
                {
                    territoryName = GetRandoName();
                    return GetContinueApiResponse("UserInvitationRedemptionSucceeded", "The invitation code you provided is valid.", cmcId, territoryName);
                }
            }
            catch (Exception exc)
            {
                this._logger.LogError(exc, "Error while processing request body: " + exc.ToString());
                return GetBlockPageApiResponse("UserInvitationRedemptionFailed-InternalError", "An error occurred while validating your invitation code, please try again later.");
            }
        }
```
