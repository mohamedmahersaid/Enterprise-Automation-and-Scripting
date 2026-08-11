# Automation & Scripting catalog

The automation engineer's toolkit: PowerShell and Python for enterprise operations, Bash for Linux fleets, REST and Microsoft Graph API integration, and modern orchestration with n8n workflows and the Model Context Protocol (MCP) for AI-driven operations.

## Scripting Languages

Production-grade PowerShell, Python, and Bash: error handling, modules, secrets, and patterns that survive code review.

### PowerShell for Operations

- **Advanced:** [PowerShell Advanced Functions, Error Handling and Modules](docs/au-scripting-core/au-powershell/au-ps-advanced-functions.md)
- **Enterprise:** [Ansible and PowerShell Remoting: Fleet Automation at Scale](docs/au-scripting-core/au-powershell/au-ps-fleet.md)

### Python & Bash for Infrastructure

- **Intermediate:** [Python for Infrastructure Automation](docs/au-scripting-core/au-python-bash/au-python-infra.md)
- **Intermediate:** [Bash, REST APIs & Microsoft Graph](docs/au-scripting-core/au-python-bash/au-bash-rest.md)

## Workflow Orchestration & AI Automation

n8n for event-driven enterprise workflows and the Model Context Protocol for connecting AI agents to operational tooling.

### n8n Workflow Automation

- **Intermediate:** [n8n and Node-RED Flow Automation: Self-Hosted Integration Design](docs/au-orchestration/au-n8n/au-n8n-workflows.md)
- **Advanced:** [Event-Driven IT Operations Patterns](docs/au-orchestration/au-n8n/au-event-driven-ops.md)

### MCP & AI-Assisted Operations

- **Advanced:** [Model Context Protocol (MCP) Fundamentals](docs/au-orchestration/au-mcp-ai/au-mcp-protocol.md)
- **Expert:** [AI-Assisted Runbooks & Agentic Operations](docs/au-orchestration/au-mcp-ai/au-ai-runbooks.md)

## Enterprise Automation Practice

The engineering discipline that separates fragile scripts from production-grade automation: observability, idempotency and state, secrets handling, and reusable tooling.

### Reliability, Observability & State

- **Advanced:** [Error Handling, Logging & Observability in Automation](docs/au-enterprise-automation-practice/au-reliability-state/au-error-handling-observability.md)
- **Advanced:** [Idempotency & State Management](docs/au-enterprise-automation-practice/au-reliability-state/au-idempotency-state-mgmt.md)

### Secrets Handling & Reusable Tooling

- **Advanced:** [Secrets Handling in Automation](docs/au-enterprise-automation-practice/au-secrets-tooling/au-secrets-handling-automation.md)
- **Intermediate:** [Reusable Modules, Collections & Internal Tooling](docs/au-enterprise-automation-practice/au-secrets-tooling/au-reusable-modules-tooling.md)

## Enterprise Workflow & Process Automation

Choosing and operating the automation engine that fits the work: integration flows, executable business processes, governed runbook automation, and the RPA estate nobody wants to own.

### Workflow & Process Automation

- **Advanced:** [Workflow Engine Selection: n8n, Camunda, Airflow, Node-RED and Rundeck](docs/au-tree-workflow-process/au-branch-workflow-process/au-workflow-engine-selection.md)
- **Advanced:** [RPA and Low-Code Governance: UiPath, Power Automate and Citizen Developer Control](docs/au-tree-workflow-process/au-branch-workflow-process/au-rpa-lowcode-governance.md)

## Estate Tooling and Certificate Lifecycle

Two things an automation practice meets once it leaves greenfield: an estate already running tools you did not choose, and a certificate population nobody owns until one expires in production.

### Inherited Tooling and Expiry Management

- **Advanced:** [Configuration Management Tool Landscape Chef Puppet Salt and OpenTofu Migration](docs/au-tree-estate-automation/au-branch-estate-automation/au-configmgmt-landscape.md)
- **Advanced:** [Certificate Lifecycle Automation Expiry Scanning and Renewal Across the Estate](docs/au-tree-estate-automation/au-branch-estate-automation/au-certificate-lifecycle.md)
