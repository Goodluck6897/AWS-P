Todo
###
check iam policy for MCP

https://towardsthecloud.com/aws-cli-assume-iam-role


Create a user s3-assume-user


Create a role with trust policy Role 

Name:Role-For-S3-assume-user
```yaml
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::019419900643:user/s3-assume-user"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

Add S3 Full access to the role  Role-For-S3-assume-user

Login to the user S3-assume-user and switch role provide Role-For-S3-assume-user


IAM Notes
##
We can see the logs of the IAM user access... in cloud trail




Here are your GitHub-formatted notes on Implicit and Explicit in AWS IAM:


# Implicit and Explicit in AWS IAM

## Overview

AWS IAM uses **implicit** and **explicit** concepts to determine whether a principal (user, role, group, or service) is **allowed** or **denied** access to perform an action on a resource.

> **Key Rule:** `Explicit Deny` > `Explicit Allow` > `Implicit Deny`

---

## Implicit Deny

- **Default behavior** — all requests are denied unless explicitly allowed.
- Occurs when there is **no applicable `Deny`** and **no applicable `Allow`** statement.
- IAM principals must be **explicitly allowed** to perform an action; otherwise, they are **implicitly denied**.

**Example:**  
If a user has no policy granting `s3:GetObject`, they are *implicitly denied* — not because a policy says `Deny`, but because no policy says `Allow`.

---

## Explicit Allow

- A policy statement that **specifically grants** permission using `"Effect": "Allow"`.
- Overrides an **implicit deny**.

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}

Explicit Deny

- A policy statement that specifically denies permission using "Effect": "Deny".

- Always overrides any Allow — highest priority in IAM policy evaluation.

- Cannot be overridden by any other policy.

{
  "Effect": "Deny",
  "Action": "s3:DeleteBucket",
  "Resource": "*"
}

IAM Policy Evaluation Logic

1. Explicit Deny   → Access DENIED immediately (always wins)
2. Explicit Allow   → Access GRANTED (if no explicit deny exists)
3. Implicit Deny    → Access DENIED by default (no allow, no deny)

Comparison Table

ConceptDescriptionPriorityImplicit DenyDefault state — no policy allows or denies the actionLowest (overridden by explicit allow)Explicit AllowPolicy statement with "Effect": "Allow"Medium (overridden by explicit deny)Explicit DenyPolicy statement with "Effect": "Deny"Highest (always wins)

ConceptDescriptionPriority

Concept

Description

Priority

Implicit DenyDefault state — no policy allows or denies the actionLowest (overridden by explicit allow)

Implicit Deny

Default state — no policy allows or denies the action

Lowest (overridden by explicit allow)

Explicit AllowPolicy statement with "Effect": "Allow"Medium (overridden by explicit deny)

Explicit Allow

Policy statement with "Effect": "Allow"

Medium (overridden by explicit deny)

Explicit DenyPolicy statement with "Effect": "Deny"Highest (always wins)

Explicit Deny

Policy statement with "Effect": "Deny"

Highest (always wins)

Key Takeaways

- By default, everything is denied (implicit deny).

- You must explicitly allow actions via IAM policies.

- An explicit deny always wins, even if another policy explicitly allows the action.

- This enforces the principle of least privilege — access must be intentionally granted.

- Best practice: Use explicit deny over implicit deny to prevent authorization drift.

References

- AWS IAM Policy Evaluation Logic

- The Difference Between Explicit and Implicit Denies


Here are your GitHub-ready notes, Venkatadry! You can copy and paste them directly into a `.md` file in your repository. Let me know if you'd like any additions or changes! 🚀
