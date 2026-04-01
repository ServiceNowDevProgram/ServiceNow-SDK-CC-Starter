# Query ACLs

This document contains details about Query ACLs.

## Contents

- Overview
- What is query ACL
- When to use query ACL
- Query ACL Behavior
  - query_match
    - Evaluation outcome
  - query_range
    - Evaluation outcome
- Important Notes
- Example

## Overview

Query ACLs allow you to define more granular access control by explicitly defining who can query the data.

## What is query ACL

A query ACLs have their operation set to either query_range or query_match. Query ACLs allow for more specific control of user queries, restricting or enabling access based on their setup. Query ACLs are powerful tools against blind query attacks, where an attacker blindly queries the data to extract information from results, even when they can't see the values.

## When to use query ACL

Wherever a column contains sensitive values, and allows partial/conditional access to data a query ACL should be considered and implemented as necessary based on the sensitivity of the data. Wherever there is a partial/conditional access to rows and their columns in tables, especially where that access is not enforced by data filters, query ACLs should be implemented as necessary based on the sensitivity of the data.

## Query ACL Behavior

Query ACLs use query_match and query_range operations for secure and granular table querying behavior.

### query_match

query_match is composed of: EQUALS, NOT_EQUALS, IN, NOT_IN, SAMEAS, NSAMEAS, ANYTHING, ISEMPTYSTRING, ISEMPTY, ISNOTEMPTY, ISNULL, ISNOTNULL. query_match is made of the "safe operators", in a sense that they are built to fetch specific record(s), and can't be exploited to return others.

#### Evaluation outcome

- Pass: User can submit match queries
- Fail: User will not be able to submit match queries:
  - EQUALS
  - NOT_EQUALS
  - IN
  - NOT_IN
  - SAMEAS
  - NSAMEAS
  - ANYTHING
  - ISEMPTYSTRING
  - ISEMPTY
  - ISNOTEMPTY
  - ISNULL
  - ISNOTNULL

### query_range

query_range is composed of all the others (STARTS_WITH, CONTAINS, >=, <= etc) which are more dangerous as they allow users to query for more records by adjusting the boundary values.

#### Evaluation outcome

- Pass: User can submit range queries and sorting is unrestricted
- Fail: The user will not be able to submit range queries with (STARTS_WITH, CONTAINS, >=, <=, etc. Sorting by column is restricted

## Important Notes

- Consider query ACLs when some users have access to some rows or columns and not others.
- Query ACLs (both query_match and query_range) default to a star.star ACL that delegates to read access. This means, where ACLs are enforced on queries, if no query ACL was created then read access to the column is evaluated ; if query ACLs are defined then they override the default behavior.

## Example:

### Payroll query control

I can see one row in payroll table with my salary, but there is no reason for me to be able to issue range queries to query users with a salary contained within 2 boundaries. A query_range ACL on salary would prevent me from issuing that query.

### HR query control

I can see all hr_profiles, but can only see SSN for myself. I have no business querying SSN, and query ACLs should prevent me from running queries against SSN of other hr profiles to try to extract SSN mappings.

## Next Steps

- For fluent API and examples, use `get_knowledge_source` tool to get the ACL knowledge source.
