# Login Policy

## Purpose

!!! info
    Please note, we currently do not support importing `rego.v1`.

Login policies govern user authentication to the account and optionally grant administrative privileges. Unlike other policy types, login policies are global; they cannot be attached to individual stacks. They take effect immediately upon creation and impact all future login attempts.

!!! info
    API Keys are evaluated against login policies similarly to virtual users, except for those in the "root" space assigned with an admin key.

!!! warning
    Login policies do not affect GitHub organization administrators, Single Sign-On ([SSO](../../integrations/single-sign-on/README.md)) administrators, or private account owners. These users retain administrative access to their respective Spacelift accounts to prevent a lockout scenario caused by incorrect login policy configuration.

!!! danger
    Any change (creation, update, or deletion) to a login policy invalidates all active sessions, except for the session initiating the change.

A login policy can define the following boolean rule types:

- **allow**: Permits the user to log in as a non-administrator.
- **admin**: Grants the user account-wide administrator access (an explicit **allow** is not required).
- **deny**: Denies the login attempt, overriding any other rules.
- **deny_admin**: Denies administrative access, regardless of other rule outcomes.
- **space_admin / space_write / space_read**: Assigns access levels to spaces (see [Spaces Access Control](../spaces/access-control.md)).

If no rules match, login attempts are denied by default.

Granting administrative access is a significant action, as administrators can perform critical tasks such as creating or deleting stacks, triggering runs, and managing contexts and policies. To limit elevated access, consider using the **space_admin** rule instead of full administrative rights.

!!! danger
    Whenever defining an **allow** or **admin** rule, it is recommended to also implement corresponding **deny** rules to ensure security. Please refer to the examples below for guidance.

## Data Input

Each policy receives the following input schema:

```json
{
  "request": {
    "remote_ip": "string - IP address of the user attempting login",
    "timestamp_ns": "number - Current Unix timestamp in nanoseconds"
  },
  "session": {
    "creator_ip": "string - IP address of the session creator",
    "login": "string - Username of the user attempting login",
    "member": "boolean - Indicates if the user is a member of the account",
    "name": "string - Full name of the user (may be empty)",
    "teams": ["string - Names of teams the user belongs to"]
  },
  "spaces": [
    {
      "id": "string - Space ID",
      "name": "string - Space name",
      "labels": ["string - Space labels"]
    }
  ]
}
```

!!! tip
    OPA string comparisons are case-sensitive. Ensure values are case-matched according to your Identity Provider (IdP). You may [enable sampling](./README.md#sampling-policy-inputs) to view exact values passed by the IdP.

Two fields in the session object require additional explanation: _member_ and _teams_.

### Account Membership

When a user first logs into Spacelift, GitHub is used as the identity provider. The username (login) is especially important. Each Spacelift account is linked to a single GitHub account. Thus, when logging into a Spacelift account, membership verification occurs:

- **Organization Accounts**: If the GitHub account is an organization, Spacelift verifies organizational membership. If confirmed, the `member` field is set to `true`; otherwise, it is `false`.
- **Private Accounts**: Private accounts can have only one member. If the user's login matches the linked GitHub account name, `member` is `true`; otherwise, it is `false`.

When using SAML-based Single Sign-On, successful login attempts require that the `member` field be set to `true`.

!!! warning
    The `member` field is critical and should be carefully considered when implementing **deny** rules.

### Teams

When using GitHub as the identity provider, team membership is queried for organization accounts only. Spacelift retrieves the full list of GitHub teams the user belongs to, stored in the `session.teams` field. For private accounts and non-members, this list is empty.

Spacelift treats GitHub team membership as transitive: if Charlie is a member of the _Badass_ team, which is a child of the _Awesome_ team, Charlie's team list includes both _Badass_ and _Awesome_.

For SAML-based SSO, team information depends on how attributes are mapped by the IdP. See the [Single Sign-On integration documentation](../../integrations/single-sign-on/README.md#setting-up-the-integration) for more details.

!!! warning
    Team information is especially valuable for configuring **allow** and **admin** rules.

## Examples

!!! tip
    Explore our [example policy library](https://github.com/spacelift-io/spacelift-policies-example-library/tree/main/examples/login){: rel="nofollow"} for ready-to-use policies. If further assistance is needed, [contact our support team](../../product/support/README.md#contact-support).

!!! note
    We recommend defining a single login policy, as merging multiple policies can lead to unexpected results.

Login policies can support three primary use cases: granting access to internal users, managing access for external contributors, and restricting access under specific conditions.

### Managing Access Levels Within an Organization

In high-security environments, users may lack GitHub administrative rights but still require administrative access to Spacelift. The following policy grants DevOps team members admin rights, Engineering team members regular access, and denies access to all others:

```opa
package spacelift

teams := input.session.teams

admin { teams[_] == "DevOps" }
allow { teams[_] == "Engineering" }
deny  { not input.session.member }
```

[Minimal example available here](https://play.openpolicyagent.org/p/LpzDekpDOU){: rel="nofollow"}.

For SSO integrations, only the integration creator has administrative access by default. Other administrators must be granted access via a login policy.

### Granting Access to External Contributors

!!! danger
    This capability is unavailable when using Single Sign-On ([SSO](../../integrations/single-sign-on/README.md)). The IdP must verify each login attempt.

External users such as consultants may require access to Spacelift. Below are two examples:

=== "GitHub"

Using GitHub usernames:

```opa
package spacelift

admins  := { "alice" }
allowed := { "bob", "charlie", "danny" }
login   := input.session.login

admin { admins[login] }
allow { allowed[login] }
deny  { not admins[login]; not allowed[login] }
```

[Minimal example here](https://play.openpolicyagent.org/p/ZsOJayumFw){: rel="nofollow"}.

=== "Google"

Using Google-managed email addresses:

```rego
package spacelift

admins  := { "alice@example.com" }
login   := input.session.login

admin { admins[login] }
allow { endswith(login, "@example.com") }
deny  { not admins[login]; not allow }
```

!!! warning
    Whitelisting individuals is less secure than using teams and member validation. Revocation of GitHub organization access automatically removes access for team-based users but not for whitelisted individuals.

### Restricting Access to Specific Circumstances

The following example restricts Spacelift access to users logging in from a designated office IP address during business hours:

```opa
package spacelift

now     := input.request.timestamp_ns
clock   := time.clock([now, "America/Los_Angeles"])
weekend := { "Saturday", "Sunday" }
weekday := time.weekday(now)
ip      := input.request.remote_ip

deny { weekend[weekday] }
deny { clock[0] < 9 }
deny { clock[0] > 17 }
deny { not net.cidr_contains("12.34.56.0/24", ip) }
```

[Playground example available here](https://play.openpolicyagent.org/p/4J3Nz6pYgC){: rel="nofollow"}.

!!! note
    Only **deny** rules are shown. **Allow** and **admin** rules should also be included.

### Granting Limited Admin Access

To grant limited administrative access to specific resources, refer to [Spaces](../spaces/README.md).

### Rewriting Teams

The **team** rule allows rewriting the list of teams received from the IdP to define Spacelift-specific roles. Example:

```opa
package spacelift

team["Superwriter"] {
  office_vpn
  devops
  not contractor
}

contractor { input.session.teams[_] == "Contractors" }
devops     { input.session.teams[_] == "DevOps" }
office_vpn { net.cidr_contains("12.34.56.0/24", input.request.remote_ip) }
```

This approach **overwrites** the original list of teams.

To **preserve** existing teams and add new ones:

```opa
package spacelift

team[name] { name := input.session.teams[_] }

team["Superwriter"] {
  office_vpn
  devops
  not contractor
}

contractor { input.session.teams[_] == "Contractors" }
devops     { input.session.teams[_] == "DevOps" }
office_vpn { net.cidr_contains("12.34.56.0/24", input.request.remote_ip) }
```

[Playground example available here](https://play.openpolicyagent.org/p/dM8P83sk4l){: rel="nofollow"}.

!!! hint
    Rewritten teams will be reflected in user data for other policy types, such as [Access Policies](./stack-access-policy.md).

## Default Login Policy

If no login policies are defined, Spacelift applies the following default policy:

```opa
package spacelift

allow { input.session.member }
```
