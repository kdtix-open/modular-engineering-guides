# Configuration and Constants Standards

> **Requirement**: Operational behavior must be discoverable, named, typed, and configurable without searching through implementation code.

---

## Overview

Hard-coded operational values make systems difficult to operate, review, test,
and change safely. A value that controls runtime behavior should not be hidden
inside business logic.

This guide covers:

- **Magic numbers / magic constants**: unexplained hard-coded values such as
  `timeout = 37` or `if retries > 7`
- **Magic strings**: hard-coded string values that control behavior, routing,
  modes, statuses, feature names, or external identifiers
- **Hard-coding**: embedding fixed values directly in source code instead of
  naming or externalizing them
- **Hidden configuration / undiscoverable configuration**: runtime behavior
  that operators cannot find without reading implementation details
- **Configuration scattering / shotgun configuration**: values repeated across
  files so a change requires a codebase-wide hunt
- **Tight coupling to configuration**: business logic that directly depends on
  embedded environment-specific values
- **Literal obsession**: excessive use of primitive literals instead of named
  abstractions, enums, typed objects, or domain types

The standard recommendation is:

1. Centralize constants.
2. Externalize environment and runtime configuration.
3. Use typed configuration objects.
4. Document defaults explicitly.

---

## Core Rules

### 1. Name Non-Obvious Literals

Any literal whose meaning is not obvious from the local expression must be
assigned to a named constant, enum, configuration field, or domain object.

**Bad**:

```python
if retries > 7:
    raise RetryLimitExceeded()
```

**Good**:

```python
MAX_RETRY_ATTEMPTS = 7

if retries > MAX_RETRY_ATTEMPTS:
    raise RetryLimitExceeded()
```

The name should explain the business or operational meaning, not restate the
type. Prefer `MAX_RETRY_ATTEMPTS` over `NUMBER_SEVEN`.

### 2. Separate Constants from Runtime Configuration

Not every named value belongs in a config file. Classify the value first.

| Value type | Required location |
|---|---|
| Domain invariant | Named constant, enum, or domain type near the model or service that owns it |
| Protocol or specification value | Named constant with a short reference to the protocol/spec |
| Environment-specific value | External configuration loaded at startup |
| Runtime tunable | External configuration with documented default and validation |
| Feature flag | Feature flag/config provider, not inline conditionals |
| Secret or credential | Secret manager or secure environment variable; never source code |
| User-facing copy | Content/resource layer when repeated, localized, or product-managed |

### 3. Centralize Configuration Loading

Each app or service must have one clear configuration entry point. Load and
validate configuration at startup, then pass a typed object into the application
or service layer.

**Bad**:

```python
def fetch_orders():
    timeout = int(os.getenv("ORDER_TIMEOUT", "37"))
    return client.get("/orders", timeout=timeout)

def fetch_customers():
    timeout = int(os.getenv("ORDER_TIMEOUT", "37"))
    return client.get("/customers", timeout=timeout)
```

**Good**:

```python
@dataclass(frozen=True)
class ApiConfig:
    request_timeout_seconds: int = 37


def load_api_config(env: Mapping[str, str]) -> ApiConfig:
    timeout = int(env.get("REQUEST_TIMEOUT_SECONDS", "37"))
    if timeout <= 0:
        raise ValueError("REQUEST_TIMEOUT_SECONDS must be greater than 0")
    return ApiConfig(request_timeout_seconds=timeout)
```

### 4. Document Defaults and Precedence

Every externalized setting must document:

- name
- purpose
- type
- default value
- valid range or allowed values
- source precedence
- security sensitivity

Unless a project defines a different standard, use this precedence:

1. CLI flags
2. Environment variables
3. Config file
4. Documented defaults

### 5. Validate Configuration at the Boundary

Configuration values are input. Validate them before application logic runs.

Validation must check:

- required values are present
- types parse correctly
- numeric ranges are safe
- enum values are known
- paths use OS-agnostic path handling
- secrets are present without being logged

Fail fast with sanitized error messages. Do not allow invalid config to fail
later as an unrelated runtime error.

### 6. Keep Business Logic Independent of Config Sources

Business logic should depend on typed configuration objects, not on `os.getenv`,
process arguments, global config stores, or raw files.

**Bad**:

```typescript
export function shouldRetry(attempt: number): boolean {
  return attempt < Number(process.env.MAX_RETRIES ?? "7");
}
```

**Good**:

```typescript
type RetryConfig = {
  maxAttempts: number;
};

export function shouldRetry(attempt: number, config: RetryConfig): boolean {
  return attempt < config.maxAttempts;
}
```

### 7. Log Effective Configuration Safely

At startup, log sanitized effective configuration at `info` level so operators
can understand runtime behavior. Never log secrets, tokens, passwords, private
keys, or connection strings.

See [Observability & Logging Standards](observability-and-logging.md) for log
level and artifact requirements.

---

## Allowed Inline Literals

Inline literals are acceptable when they are self-explanatory and local:

- `0`, `1`, and empty collections used as simple identities or initial values
- short loop bounds in obvious local transformations
- literals inside a test case when the value is the test input being asserted
- protocol examples in documentation or sample payloads
- short user-facing labels that are local, not reused, and not localized

When a literal is repeated, environment-specific, security-sensitive,
operationally tunable, or needed by another module, it is no longer a harmless
inline literal.

---

## Red Flags

Reviewers should stop and request cleanup when they see:

- the same timeout, retry count, path, URL, mode, status, or limit in multiple
  files
- `os.getenv`, `process.env`, CLI parsing, or config file reads inside business
  logic
- unexplained numbers other than obvious `0` or `1`
- string values that control routing, permissions, state machines, or feature
  behavior
- environment names such as `prod`, `dev`, or `staging` embedded in logic
- hard-coded API hosts, tenant IDs, queue names, table names, or bucket names
- booleans such as `READ_ONLY = False` embedded in code instead of loaded from
  configuration
- secrets, tokens, credentials, or connection strings in source files

---

## Testing Requirements

Configuration behavior must be tested like any other behavior.

Required tests:

- defaults are applied when optional settings are absent
- required settings fail fast when missing
- invalid values fail with clear sanitized errors
- source precedence is honored
- typed config objects are passed into services instead of config sources being
  read inside business logic
- mutation guards such as read-only mode can be enabled by configuration
- regression tests cover any bug caused by hidden or scattered configuration

For externalized values that affect UAT, the UAT scenario must list the relevant
configuration values in its prerequisites.

---

## Migration Pattern

When cleaning up an existing codebase:

1. Inventory repeated literals, operational values, environment-specific values,
   and magic strings.
2. Classify each value as a domain constant, protocol constant, runtime setting,
   feature flag, secret, or content.
3. Create typed configuration objects and central loaders.
4. Move shared domain constants to the owning feature or domain module.
5. Replace direct reads from environment variables, CLI args, and config files
   inside business logic with typed dependencies.
6. Add validation and tests for defaults, precedence, and invalid values.
7. Document settings in onboarding, operations, or README material.
8. Log sanitized effective configuration at startup.

Do this incrementally by feature area. Avoid a broad mechanical refactor unless
the test suite is strong enough to prove behavior has not changed.

---

## Review Checklist

- [ ] Non-obvious numeric and string literals are named.
- [ ] Repeated literals have a single owner.
- [ ] Environment-specific and runtime-tunable values are externalized.
- [ ] Configuration loading is centralized.
- [ ] Configuration is represented by typed objects or validated schemas.
- [ ] Defaults, valid values, and precedence are documented.
- [ ] Business logic receives configuration as a dependency.
- [ ] Secrets are never stored, printed, or logged in source-controlled files.
- [ ] Tests cover defaults, invalid config, and source precedence.
- [ ] UAT prerequisites list relevant configuration values.
