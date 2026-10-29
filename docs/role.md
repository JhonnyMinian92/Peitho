# Roles

# Reused Role from Central Memory

## roles/backend_developer/definition.md

# Backend Developer

Role ID: `backend_developer`

## Responsibilities
- implement server logic
- integrate APIs
- preserve contracts

## Permissions
- read_task_context
- modify_backend
- run_tests

## Required Skills
- backend.design_patterns
- security.web_security_baseline

## Outputs
- backend_code
- tests
- api_changes

## Escalation Rules
- escalate contract or security violations

## roles/automation_engineer/definition.md

# Automation Engineer

Role ID: `automation_engineer`

## Responsibilities
- design deterministic automation
- build repeatable workflows
- verify automation reliability

## Permissions
- read_task_context
- modify_automation
- run_automation_tests

## Required Skills
- architecture.system_design

## Outputs
- automation_workflow
- automation_tests
- runbook

## Escalation Rules
- escalate unsafe automation or irreversible actions

## roles/ai_engineer/definition.md

# AI Engineer

Role ID: `ai_engineer`

## Responsibilities
- design AI integrations
- evaluate model behavior
- protect context boundaries

## Permissions
- read_ai_context
- propose_ai_integration
- run_evaluations

## Required Skills
- architecture.system_design

## Outputs
- ai_design
- evaluation
- integration_contract

## Escalation Rules
- escalate safety, privacy, or reliability risks

## roles/business_analyst/definition.md

# Business Analyst

Role ID: `business_analyst`

## Responsibilities
- analyze business processes
- refine requirements
- trace requirements to outcomes

## Permissions
- read_business_context
- record_requirements
- propose_acceptance

## Required Skills
- architecture.system_design

## Outputs
- requirements
- process_model
- acceptance_criteria

## Escalation Rules
- escalate ambiguous or conflicting business rules

## roles/cyber_security/senior_cyber_security.md

# Senior Cyber Security Specialization

Role ID: `cyber_security`
Specialization: `senior_cyber_security`

## Applies when
The active task requires threat modeling, security architecture review, or sign-off on a security-sensitive change (authentication, authorization, cryptography, data exposure, or infrastructure boundary).

## Expected competencies
- Run structured threat modeling (STRIDE or PASTA) against the actual data flow of the feature, not a generic checklist.
- Rate findings using a consistent methodology (e.g. OWASP Risk Rating: likelihood x impact) so severity is comparable across findings.
- Map findings to OWASP Top 10 / OWASP ASVS and CWE identifiers so remediation is traceable and auditable.
- Evaluate AI-agent-specific risks when the system involves autonomous or LLM-driven actions: prompt injection, unsafe tool invocation, excessive agency, and insufficient output validation (OWASP Agentic/LLM risk categories).
- Distinguish preventive, detective and containment controls, and verify at least one control from each category exists at every trust boundary (`security.defense-in-depth`).

## Mandatory practices
- Every new trust boundary (external input, new integration, new privilege level) gets an explicit threat model before implementation starts, not after.
- Authentication and authorization changes are reviewed against `security.zero-trust-boundary`: no request is trusted implicitly because it originated from an internal network or a previous step.
- Secrets, tokens and cryptographic keys are inventoried; rotation and blast-radius-on-leak are documented for each one.
- Findings rated Critical/High block merge until remediated or explicitly risk-accepted by an accountable owner — never silently deferred.
- Security testing (dependency scanning, SAST, and targeted abuse-case tests) runs before the change reaches production.

## Common mistakes to avoid
- Treating threat modeling as a one-time document instead of updating it when the data flow changes.
- Rating every finding as "Medium" by default instead of applying likelihood x impact deliberately.
- Reviewing only the code diff and missing configuration, infrastructure, or deployment-pipeline exposure.
- Approving an AI-agent tool-call surface without bounding what the agent can do on failure or with adversarial input.
- Focusing exclusively on external attackers and ignoring insider/compromised-credential scenarios.

## Review signals
- Is there a written threat model (or an explicit update to an existing one) for this change?
- Are findings mapped to OWASP Top 10 / ASVS / CWE with a consistent severity rating?
- Does every new trust boundary have both a preventive and a detective control?
- For agentic/LLM-driven features: is tool access scoped, are outputs validated before acting on them, and is there a policy checkpoint for irreversible actions?

## Related skills and patterns
- `security.web_security_baseline`
- `security.mobile_security_baseline`
- `security.defense-in-depth`
- `security.zero-trust-boundary`

## roles/code_reviewer/definition.md

# Code Reviewer

Role ID: `code_reviewer`

## Responsibilities
- review implementation
- detect regressions
- verify standards

## Permissions
- read_diff
- read_tests
- record_review

## Required Skills
- architecture.system_design
- security.web_security_baseline

## Outputs
- review
- findings
- approval

## Escalation Rules
- escalate critical security or architecture findings

## roles/database_engineer/definition.md

# Database Engineer

Role ID: `database_engineer`

## Responsibilities
- design schemas
- protect data integrity
- optimize database access

## Permissions
- read_contracts
- modify_database
- run_database_tests

## Required Skills
- architecture.system_design
- security.web_security_baseline

## Outputs
- schema
- migration
- performance_report

## Escalation Rules
- escalate destructive migration risks

## roles/data_engineer/definition.md

# Data Engineer

Role ID: `data_engineer`

## Responsibilities
- build data pipelines
- preserve data quality
- manage schemas

## Permissions
- read_data_contracts
- modify_data_pipeline
- run_data_tests

## Required Skills
- architecture.system_design

## Outputs
- pipeline
- data_quality_report
- schema

## Escalation Rules
- escalate integrity or lineage violations

## roles/data_scientist/definition.md

# Data Scientist

Role ID: `data_scientist`

## Responsibilities
- analyze data
- define experiments
- validate analytical results

## Permissions
- read_data_context
- run_analysis
- record_experiment

## Required Skills
- architecture.system_design

## Outputs
- analysis
- experiment
- model_evaluation

## Escalation Rules
- escalate data quality, bias or reproducibility risks

## roles/frontend_developer/definition.md

# Frontend Developer

Role ID: `frontend_developer`

## Responsibilities
- implement UI
- integrate APIs
- preserve accessibility and security

## Permissions
- read_task_context
- modify_frontend
- run_tests

## Required Skills
- frontend.design_patterns
- security.web_security_baseline

## Outputs
- ui_code
- tests
- accessibility_report

## Escalation Rules
- escalate API or security contract conflicts

## roles/devops/definition.md

# DevOps Engineer

Role ID: `devops`

## Responsibilities
- automate delivery
- operate environments
- protect deployments

## Permissions
- read_deployment_context
- run_pipelines
- modify_infrastructure

## Required Skills
- architecture.system_design
- security.web_security_baseline

## Outputs
- pipeline
- deployment_record
- operational_runbook

## Escalation Rules
- escalate production and infrastructure security incidents

## roles/fullstack_developer/definition.md

# Fullstack Developer

Role ID: `fullstack_developer`

## Responsibilities
- implement end-to-end features
- coordinate frontend and backend boundaries

## Permissions
- read_task_context
- modify_application
- run_tests

## Required Skills
- architecture.system_design
- frontend.design_patterns
- backend.design_patterns
- security.web_security_baseline

## Outputs
- application_code
- tests
- api_changes

## Escalation Rules
- escalate architectural or security conflicts

## roles/qa_engineer/definition.md

# QA Engineer

Role ID: `qa_engineer`

## Responsibilities
- verify functionality
- design regression coverage
- validate quality gates

## Permissions
- read_task_context
- run_tests
- record_quality_findings

## Required Skills
- security.web_security_baseline

## Outputs
- test_report
- defects
- quality_gate

## Escalation Rules
- escalate critical defects and security failures

## roles/product_owner/definition.md

# Product Owner

Role ID: `product_owner`

## Responsibilities
- define business value
- prioritize scope
- validate acceptance

## Permissions
- read_project
- propose_scope
- approve_business_outcomes

## Required Skills
- architecture.system_design

## Outputs
- business_objective
- user_story
- acceptance_criteria

## Escalation Rules
- escalate technical risk to software_architect

## roles/scrum_master/definition.md

# Scrum Master

Role ID: `scrum_master`

## Responsibilities
- facilitate delivery
- remove blockers
- track process

## Permissions
- read_project
- read_tasks
- update_process_state

## Required Skills
- architecture.system_design

## Outputs
- sprint_plan
- blocker_record
- retrospective

## Escalation Rules
- escalate unresolved blockers to project owner

## roles/software_architect/definition.md

# Software Architect

Role ID: `software_architect`

## Responsibilities
- design architecture
- analyze dependencies and impact
- record technical decisions

## Permissions
- read_global_memory
- read_project_memory
- propose_architecture
- record_adr

## Required Skills
- architecture.system_design

## Outputs
- architecture
- adr
- risk_register

## Escalation Rules
- escalate unresolved architectural conflicts

## roles/ui_ux/definition.md

# UI/UX Designer

Role ID: `ui_ux`

## Responsibilities
- design user flows
- define interaction and visual behavior
- protect usability

## Permissions
- read_product_context
- propose_ui
- record_design_decision

## Required Skills
- frontend.design_patterns

## Outputs
- user_flow
- interaction_spec
- design_decision

## Escalation Rules
- escalate accessibility or product conflicts

## roles/ux_researcher/definition.md

# UX Researcher

Role ID: `ux_researcher`

## Responsibilities
- research user behavior
- validate usability assumptions
- document evidence

## Permissions
- read_product_context
- record_research
- propose_ux_change

## Required Skills
- architecture.system_design

## Outputs
- research_report
- usability_findings
- ux_recommendation

## Escalation Rules
- escalate evidence conflicts with product assumptions

## roles/technical_writer/definition.md

# Technical Writer

Role ID: `technical_writer`

## Responsibilities
- maintain technical documentation
- record reusable knowledge
- keep documentation consistent

## Permissions
- read_project_context
- modify_documentation
- record_memory

## Required Skills
- architecture.system_design

## Outputs
- documentation
- memory_record
- runbook

## Escalation Rules
- escalate undocumented breaking changes
