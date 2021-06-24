---
pcx-content-type: concept
order: 2
---

# Configure Managed Ruleset overrides via API

Configure overrides for the DDoS L7 Attack Mitigation Managed Ruleset using the [Rulesets API](https://developers.cloudflare.com/firewall/cf-rulesets/rulesets-api). You can deploy a DDoS L7 Attack Mitigation phase ruleset in order to override the rules in the Managed Ruleset. There is no need to deploy a DDoS L7 Attack Mitigation Managed Ruleset since if you have access to view use the ruleset it will already be deployed for you. In order to customize how rules within the `ddos_l7` phase ruleset function you must configure an override. See below for how to do this.

## Configure an override for the DDoS L7 Attack Mitigation Managed Ruleset

You can define overrides at the ruleset, tag, and rule level for all Managed Rulesets. When configuring the DDoS L7 Attack Mitigation Managed Ruleset, use overrides to define a different **action** or **sensitivity level** from the default values. You can override the following rule properties:

* `"action"` (`"block"`, `"challenge"`, `"log"`)
* `"sensitivity_level"` (`"default"`, `"medium"`, `"low"`, `"eoff"`)

<Aside type='warning' header='Important'>

The DDoS L7 Attack Mitigation Managed Ruleset is always enabled — you cannot disable its rules using an override with `"enabled": false`. Additionally, you must set the override `"expression"` field to `"true"`.

</Aside>

## Example

The following `PUT` example creates a new phase ruleset (or updates an existing one) for the `ddos_l7` phase at the account level. It contains several overrides to adjust the default behavior of the DDoS L7 Attack Mitigation Managed Ruleset. The overrides are the following:

* All rules of the Managed Ruleset will use the `challenge` action and have a sensitivity level of `medium`.
* All rules tagged with the category `{category-name}` will have a sensitivity level of `low`.
* The rule with ID `{rule-id}` will use the `block` action.

```json
curl -X PUT \
-H "X-Auth-Email: user@cloudflare.com" \
-H "X-Auth-Key: REDACTED"
"https://api.cloudflare.com/client/v4/accounts/{account-id}/rulesets/phases/ddos_l7/entrypoint" \
-d '{
  "description": "Execute Cloudflare DDoS L7 Attack Mitigation Managed Ruleset on my account-level Phase ruleset",
  "rules": [
    {
      "action": "execute",
      "action_parameters": {
        "id": "{managed-ruleset-id}",
        "overrides": {
          "sensitivity_level": "medium",
          "action": "challenge",
          "categories": [
            {
              "category": "{tag-name}",
              "sensitivity_level": "low"
            }
          ],
          "rules": [
            {
              "id": "{rule-id}",
              "action": "block"
            }
          ]
        }
      },
      "expression": "true",
    }
  ]
}'
```

The response returns the created phase ruleset.

```json
{
  "result": {
    "id": "{root-ruleset-id}",
    "name": "default",
    "description": "Execute Cloudflare DDoS L7 Attack Mitigation Managed Ruleset on my account-level phase ruleset",
    "kind": "root",
    "version": "1",
    "rules": [
      {
        "id": "{overridden-rule-id}",
        "version": "1",
        "action": "execute",
        "action_parameters": {
          "id": "{managed-ruleset-id}",
          "version": "latest",
          "overrides": {
            "action": "challenge",
            "categories": [
              {
                "category": "{category-name}",
                "sensitivity_level": "low"
              }
            ],
            "rules": [
              {
                "id": "{rule-id}",
                "action": "block"
              }
            ],
            "sensitivity_level": "medium"
          }
        },
        "expression": "true",
        "last_updated": "2021-06-16T04:14:47.977741Z",
        "ref": "{overridden-rule-ref}",
        "enabled": true
      }
    ],
    "last_updated": "2021-06-16T04:14:47.977741Z",
    "phase": "ddos_l7"
  }
}
```

For more information on defining overrides for Managed Rulesets using the Rulesets API, check [Override a Managed Ruleset](https://developers.cloudflare.com/firewall/cf-rulesets/managed-rulesets/override-managed-ruleset).