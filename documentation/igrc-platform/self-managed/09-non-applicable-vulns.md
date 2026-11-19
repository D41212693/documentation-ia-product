# Non applicable vulnerabilities

## Version 3.6.1

### CVE-2025-41249: Spring Framework Core Vulnerability

#### Basic Information

| Attribute         | Details                         |
| ----------------- | ------------------------------- |
| **CVE ID**        | CVE-2025-41249                  |
| **Severity**      | High                            |
| **Component**     | org.springframework:spring-core |
| **Occurrences**   | 2                               |
| **Identified By** | JFrog                           |

#### Status

**Not Applicable**

Although this vulnerability is marked as High severity by JFrog, it does not apply to Identity Analytics. Identity Analytics does not use the Spring Security features concerned by this vulnerability.

#### Risk Assessment

**Actual Risk: Zero**

This vulnerability is not a concern for the following reasons:

- The vulnerability impact is specific to Spring Security features
- Our development team has confirmed that our applications do not use any Spring Security features
- Therefore, our services are not impacted by this vulnerability

#### Recommendation

**Status: No Action Required**

### CVE-2025-41242: Spring Beans Vulnerability

#### Basic Information

| Attribute         | Details                                                                                                                                        |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **CVE ID**        | CVE-2025-41242                                                                                                                                 |
| **Severity**      | High                                                                                                                                           |
| **Component**     | org.springframework:spring-webmvc and org.springframework:spring-beans (path traversal in Spring Framework MVC using Spring resource handling) |
| **Occurrences**   | 2                                                                                                                                              |
| **Identified By** | Snyk                                                                                                                                           |

#### Status

**Not Applicable**

Identity Analytics does not expose any Spring MVC controllers to serve static resources and runs on Apache Tomcat with default security features enabled, which Spring has explicitly confirmed is not vulnerable under this CVE.

#### Risk Assessment

**Actual Risk: Zero**

The exploit preconditions (Spring MVC static resource handling on a noncompliant container) are not met in the Identity Analytics architecture; therefore, the effective risk is zero.

#### Recommendation

**Status: No Action Required**

### CVE-2026-0603: Hibernate Core Vulnerability

#### Basic Information

| Attribute         | Details                                                                                                        |
| ----------------- | -------------------------------------------------------------------------------------------------------------- |
| **CVE ID**        | CVE-2026-0603                                                                                                  |
| **Severity**      | High                                                                                                           |
| **Component**     | org.hibernate:hibernate-core (InlineIdsOrClauseBuilder second-order SQL injection in UPDATE/DELETE statements) |
| **Occurrences**   | 2                                                                                                              |
| **Identified By** | JFrog                                                                                                          |

#### Status

**Not Applicable**

CVE-2026-0603 describes a second-order SQL injection in Hibernate when the InlineIdsOrClauseBuilder path is used to inline unsanitized ID values into SQL OR predicates for bulk UPDATE/DELETE operations.

Identity Analytics is not impacted because InlineIdsOrClauseBuilder is not active in the application.

#### Risk Assessment

**Actual Risk: Zero**

The specific code path (InlineIdsOrClauseBuilder for bulk ID clauses) is not used by Identity Analytics, and no untrusted identifiers are ever inlined in this manner; as a result, the practical exploitability is zero.

#### Recommendation

**Status: No Action Required**

### CVE-2026-66453: Rhino Javascript engine

#### Basic Information

| Attribute            | Details                 |
| -------------------- | ----------------------- |
| **CVE ID**           | CVE-2026-66453          |
| **Severity**         | High                    |
| **Component**        | Batch and portal, Rhino |
| **Publication Date** | March 12 2026           |
| **Occurrences**      | 2                       |

#### Status

**Not Applicable**

Rhino is an open-source implementation of JavaScript written entirely in Java. It is typically embedded into Java applications to provide scripting to end users. Affected versions of this package are vulnerable to Allocation of Resources Without Limits or Throttling via the `toFixed` function. An attacker can cause excessive CPU consumption and disrupt service availability by passing specially crafted floating-point numbers.

#### Risk Assessment

**Actual Risk: Zero**

Rhino is used only by Birt to produce reports. The only reports we have in the product are those for the user access review. The vulnerability is about the JavaScript function `toFixed()`. We do not use JavaScript in our reports except a small function to convert multivalued parameters into a single one. This function does not use the function `toFixed()`. Rhino is not accessible to users or any other services, which prevents unknown scripts from being executed.

#### Remediation

**Status: No Action Required**
