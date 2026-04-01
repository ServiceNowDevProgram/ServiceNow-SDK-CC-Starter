# Deny-Unless ACLs

This document contains details about Deny-Unless ACLs.

## Contents

- Overview
- Important Notes
- Output
- Example
- Next Steps

## Overview

Deny-Unless ACLs are evaluated with a "deny-unless" approach. The ACL defines the users that will NOT be denied. Said another way, the user will be denied access unless the role, condition, and script requirements are met.

## Important Notes

- Deny-Unless ACLs will take priority against Allow-If ACLs in ACL Evaluation, as it will be evaluated first.
- Deny-Unless ACLs are not evaluated if the user is denied access by another ACL.
- Even if a Deny-Unless ACL matches, access is only granted when an Allow-If ACL explicitly permits it. If no Allow-If ACL is matched and the Deny-Unless ACL passes, the system grants access by default.

## Output

A Deny-Unless ACL produces two outcomes

- Pass: The defined roles, data conditions, security attributes, and script requirements are met. The ACL proceeds to further evaluation.
- Fail: The Deny-Unless ACL is marked as failing and access will be denied.

## Example:

The following is an explained example of a Deny-Unless ACL:

- ACL has roles sn_hr_core.manager and itil
- Condition has active = true
- script has answer = gs.isLoggedIn();

The user is denied access unless all three requirements for this ACL are satisfied. In order for this Deny-Unless ACL to pass, a user needs either the sn_hr_core.manager or itil roles, be accessing a record that has active field = true, and be logged in. The Deny-Unless ACL will fail if any of the three requirements isn't met.

## Next Steps

- For fluent API and coding examples, use `get_knowledge_source` tool to get the ACL knowledge source.
