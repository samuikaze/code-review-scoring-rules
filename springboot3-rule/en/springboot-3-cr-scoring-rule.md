---
# Standard ID: lowercase letters, digits and - only; must match the section name in rules.ini of the rules project
name: springboot-3
# Standard version, recorded in the scoring report
version: v0.6
---

# Spring Boot 3 PR Code Quality Scoring Rules

<!-- markdownlint-disable MD028 -->
<!-- markdownlint-disable MD033 -->

For scoring the code quality of PRs in Java/Spring Boot 3 projects.

This document is set up as a scoring rule of code-review-scoring by the
`rule.ini` in the parent rule directory, and registered as `springboot-3` in
`rules.ini` of the rules project. The automatic scan rules are in
`springboot-3-cr-scoring-rule.tsv` in the same directory:

```ini
[rule]
standard="en/springboot-3-cr-scoring-rule.md"
scan="en/springboot-3-cr-scoring-rule.tsv"
applies="*.java, *.kt, *.sql, *.xml, *.yml, *.yaml, *.properties, *.gradle, *.kts"
```

General rules such as the scoring scope, deductions requiring evidence and
upstream factors are handled by the scoring workflow and are not covered in this
document.

## Dimensions

All standards share the following seven dimensions. The points, the severity
scale and the grades come from `defaults.ini` of the rules project and can be
overridden in the `rule.ini` in the parent rule directory. The first letter of
a check code indicates its dimension.

| Code | Dimension | Focus |
| --- | --- | --- |
| A | Correctness | Works as intended; handles edge cases and exceptions |
| B | Security | Whether security vulnerabilities are introduced |
| C | Test quality | Whether tests are sufficient and meaningful |
| D | Architecture and design | Layering, responsibilities, Spring conventions |
| E | Performance and resource use | Query efficiency, resources, remote calls |
| F | Readability | Whether others can easily understand the code |
| G | Maintainability | Easy to modify and extend in the future |

## Check items

Each check item is written as a list item with fields separated by the
full-width `｜`:

- Fixed severity: `- code｜content｜default severity`
- Adjustable severity:
  `- code｜content｜default severity｜adjustable to｜condition`

The severity may be adjusted only when the stated condition holds. Any check
item may be lowered to `Nit`.

### A. Correctness

- A1｜Edge cases are not handled: null, empty collections, calling `.get()` on
  an `Optional` without checking, numeric overflow｜Major
- A2｜Monetary calculations do not use `BigDecimal` (for example they use
  `double` or `float`)｜Critical
- A3｜`@Transactional` is placed on a private method, or a class calls its own
  `@Transactional` method internally, so the proxy does not take effect｜Major
- A4｜A method throws a checked exception and has `@Transactional` without
  `rollbackFor` (checked exceptions do not trigger a rollback by
  default)｜Major
- A5｜Exceptions are swallowed: a `catch` block only logs or does nothing, or
  the original cause is not kept when rethrowing｜Major
- A6｜Concurrency issues: shared mutable state in a singleton bean, an obvious
  race condition, or shared data updated without optimistic locking
  (`@Version`) or another locking mechanism｜Critical
- A7｜Time handling uses `Date` / `Calendar`, or does not specify a time zone
  explicitly｜Minor｜Major｜
  When the PR description or specification explicitly involves cross-time-zone
  business
- A8｜The implementation is inconsistent with the SD specification or PR
  description, and the corresponding part of the specification is
  clear｜Major

### B. Security

- B1｜A password, token or key is hard-coded in code or a config
  file｜Critical
- B2｜SQL injection risk: JPQL or native queries built by string
  concatenation, or parameters concatenated in `@Query`｜Critical
- B3｜A new or modified API endpoint lacks authentication or authorization
  checks, or accesses a resource without checking its owner (IDOR)｜Critical
- B4｜A controller parameter lacks `@Valid`, or a DTO does not use
  `jakarta.validation` annotations｜Major
- B5｜CSRF protection is disabled without a comment or PR description
  explaining why｜Major
- B6｜The paths opened by `permitAll()` are clearly broader than needed (for
  example `/**`, `/api/**`)｜Major
- B7｜Logs output personal data, tokens or full request bodies, or error
  responses include stack traces｜Major
- B8｜CORS sets `allowedOrigins("*")` while also allowing
  credentials｜Major
- B9｜A newly added dependency has known CVEs｜Major

### C. Test quality

- C1｜New or modified business logic has no corresponding unit
  tests｜Major｜Critical｜
  When the PR changes business logic but no test file is changed anywhere in the
  PR
- C2｜Tests cover only the happy path, not exceptions and edge cases｜Major
- C3｜Tests have no meaningful assertions, or only `assertNotNull`｜Major
- C4｜Controller or repository tests use `@SpringBootTest` instead of
  `@WebMvcTest` or `@DataJpaTest`｜Minor
- C5｜DB tests use H2, but the queries under test use syntax specific to the
  production database (Testcontainers is recommended instead)｜Minor
- C6｜Tests depend on each other or on execution order, or use
  `Thread.sleep`｜Minor
- C7｜Test method names do not reveal the scenario and expected
  result｜Minor

### D. Architecture and design

- D1｜A controller calls a repository directly, or business logic is written in
  a controller or entity listener｜Major
- D2｜A JPA entity is used directly as an API request or
  response｜Major｜Critical｜
  When the entity contains sensitive fields (password hashes, internal IDs,
  personal data)
- D3｜A single class or method takes on several unrelated responsibilities (God
  Service), or clearly violates the SOLID principles｜Major
- D4｜Field injection (`@Autowired` directly on a field) is used instead of
  constructor injection｜Minor
- D5｜Error handling does not go through `@RestControllerAdvice` and
  `ProblemDetail` (RFC 9457), or the response format is inconsistent with
  existing conventions｜Minor
- D6｜The `javax.*` namespace is used (Spring Boot 3 should use `jakarta.*`; JDK
  packages such as `javax.crypto` are exempt)｜Minor

### E. Performance and resource use

- E1｜JPA N+1 queries: lazy loading triggered in a loop, or records queried one
  by one (`@EntityGraph` or a fetch join is recommended)｜Major
- E2｜A query has no pagination and loads an entire table at
  once｜Minor｜Major｜
  When the PR description, specification or table name indicates continuously
  growing data such as transactions, orders or logs, or the data volume is large
- E3｜An external API is called within a `@Transactional` scope, making the
  transaction too long｜Major
- E4｜Calls to external systems have no timeout, retry or circuit
  breaker｜Major
- E5｜Resources such as streams, files and connections are not guaranteed to be
  closed with try-with-resources or another mechanism｜Major
- E6｜An association uses `FetchType.EAGER` without an apparent need｜Minor
- E7｜A column newly used as a query condition has no index created in a
  migration in the same PR, and no explanation is given｜Minor

### F. Readability

- F1｜A method is longer than 50 lines, or nested more than 3 levels
  deep｜Minor
- F2｜Names do not express their purpose, or are inconsistent with the naming
  conventions of other code in the same diff｜Minor
- F3｜Unnamed magic numbers or string literals with special meaning
  appear｜Minor
- F4｜Complex logic has no comment explaining why it is written that
  way｜Minor
- F5｜Commented-out code is left behind, or `System.out.println` is
  used｜Minor

### G. Maintainability

- G1｜The same logic appears two or more times in the diff and could be
  extracted for reuse but is not｜Major
- G2｜Values that belong in a config file are hard-coded, or scattered `@Value`
  fields are added instead of `@ConfigurationProperties`｜Minor
- G3｜Log levels are misused (for example `ERROR` in a normal flow), or logs use
  string concatenation instead of SLF4J parameterized messages｜Minor
- G4｜A new public API has no OpenAPI annotations｜Minor
- G5｜A TODO has no issue number｜Minor
- G6｜An existing DB migration file (Flyway / Liquibase) is modified, or a
  destructive schema change is made without explanation｜Major

## Confirmations

At the start, the scoring workflow asks about the items listed here together
with any missing documents. Items that the diff or documents already answer are
not asked again.

- A7｜Whether the feature involves cross-time-zone business (asked only when the
  diff involves time handling)
- E2｜Whether the tables of the new queries keep growing, and the approximate
  number of rows (asked only when the diff involves queries)
- G4｜Whether the project uses springdoc
- B9｜Whether vulnerability scan results for the dependencies are available

## Special rules

- A8: When no SD specification is provided, compare against the PR description
  only.
- B9: Scored only when vulnerability scan results are provided; otherwise
  recorded as not applicable.
- C1: Raised to Critical only when the PR changes business logic but does not
  change any test file at all; when there are tests that do not cover the
  logic, it stays Major.
- G4: Scored only when the diff or existing code shows that the project uses
  springdoc; otherwise recorded as not applicable.
- Third-party workaround exemption: code written to work around an issue in a
  third-party core system is not deducted for check items in dimensions F and G
  if a comment explains why **and** includes the vendor's issue ticket number.
  If either is missing, deduct as usual.

## Judgment hints

- E2: Small config tables stay Minor; large or continuously growing tables such
  as order tables are raised to Major. When the data volume cannot be seen from
  the diff, ask as described in "Confirmations".
- Static analysis reports usually come from Checkstyle, SpotBugs or SonarQube;
  issues already listed in a report are not deducted again.
- For each new or modified API endpoint, check B3 (authentication,
  authorization, resource owner) and B4.
- A6, A8, B3, C2, D3, E3, F2, F4 and G1 require reading the code in context, do
  not appear in the automatic scan candidates, and must be checked manually.
- B9, E7 and F1 have no automatic scan rules either and must also be checked
  manually.
- Automatic scan candidates are only hints and false positives are common: for
  example, `).get()` is not necessarily an `Optional`, and fake passwords in test
  data do not count as B1. Confirm each one against the code.
