# ocp4_workload_litellm_virtual_keys

Manages LiteLLM virtual keys for AI model access in RHDP lab environments.

## Features

- **Create keys** with subscription packages (ai-beginner, ai-developer, ai-researcher, lab-dev, lab-prod)
- **Delete keys** by GUID with proper cleanup
- **GUID-based naming**: `virtkey-{GUID}` for easy tracking
- **Idempotent**: Safe to run multiple times
- **AgnosticD integration**: Sends user.info messages and data

## Usage

### Provision a Virtual Key

```yaml
- name: Create LiteLLM virtual key
  vars:
    ACTION: "provision"
    guid: "user123"
    litellm_url: "https://litellm-rhpds.apps.cluster.com"
    litellm_master_key: "sk-xxxxx"
  ansible.builtin.include_role:
    name: rhpds.litellm_virtual_keys.ocp4_workload_litellm_virtual_keys
```

### Destroy a Virtual Key

```yaml
- name: Delete LiteLLM virtual key
  vars:
    ACTION: "destroy"
    guid: "user123"
    litellm_url: "https://litellm-rhpds.apps.cluster.com"
    litellm_master_key: "sk-xxxxx"
  ansible.builtin.include_role:
    name: rhpds.litellm_virtual_keys.ocp4_workload_litellm_virtual_keys
```

## Subscription Packages

| Package | Models | Duration |
|---------|--------|----------|
| ai-beginner | Granite only | 7 days |
| ai-developer | Granite + Mistral | 30 days |
| ai-researcher | Granite + Mistral + Llama | 90 days |
| lab-dev | Granite + Mistral | 14 days |
| lab-prod | Granite + Mistral | 90 days |

## Variables

### Required

- `ACTION`: "provision" or "destroy"
- `guid`: Lab environment GUID
- `litellm_url`: LiteLLM API URL
- `litellm_master_key`: LiteLLM master key

### Optional

- `ocp4_workload_litellm_virtual_keys_subscription_package`: Default "ai-developer"
- `ocp4_workload_litellm_virtual_keys_force_db_delete`: Enable direct database deletion (default: false)

## Output Facts

After provisioning, the following facts are set:

- `litellm_virtual_key`: The generated API key
- `litellm_key_alias`: Key alias (virtkey-{GUID})
- `litellm_api_endpoint`: LiteLLM API URL
- `litellm_available_models`: List of accessible models
- `litellm_key_duration`: Key validity duration

## Example

```yaml
- name: Provision AI developer key
  hosts: localhost
  vars:
    ACTION: "provision"
    guid: "abc123"
    litellm_url: "https://litellm-rhpds.apps.cluster.com"
    litellm_master_key: "sk-xxxxx"
    ocp4_workload_litellm_virtual_keys_subscription_package: "ai-developer"
  tasks:
    - include_role:
        name: rhpds.litellm_virtual_keys.ocp4_workload_litellm_virtual_keys

    - debug:
        msg: "Virtual Key: {{ litellm_virtual_key }}"
```
