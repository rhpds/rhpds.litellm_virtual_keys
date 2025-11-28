# ocp4_workload_litellm_virtual_keys

Manages LiteLLM virtual keys for AI model access in RHDP lab environments.

## Features

- **Create keys** with custom model selection and duration
- **Delete keys** by GUID with proper cleanup
- **GUID-based naming**: `virtkey-{GUID}` for easy tracking
- **Idempotent**: Safe to run multiple times
- **AgnosticD integration**: Sends user.info messages and data
- **CI Lifecycle Integration**: Keys auto-expire with catalog items and auto-extend when users extend from UI

## Usage

### Provision a Virtual Key

```yaml
- name: Create LiteLLM virtual key
  vars:
    ACTION: "provision"
    guid: "user123"
    litellm_url: "https://litellm-rhpds.apps.cluster.com"
    litellm_master_key: "sk-xxxxx"
    ocp4_workload_litellm_virtual_keys_models:
      - "openai/granite-3-2-8b-instruct"
      - "openai/mistral-7b-instruct"
    ocp4_workload_litellm_virtual_keys_duration: "30d"
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

## Available Models

- `openai/granite-3-2-8b-instruct` (IBM Granite)
- `openai/mistral-7b-instruct` (Mistral AI)

## Duration Options

- `3d` (3 days)
- `7d` (7 days)
- `10d` (10 days)
- `30d` (30 days)

## Variables

### Required

- `ACTION`: "provision" or "destroy"
- `guid`: Lab environment GUID
- `litellm_url`: LiteLLM API URL (without trailing slash)
- `litellm_master_key`: LiteLLM master key (must start with `sk-`)

### Optional

- `ocp4_workload_litellm_virtual_keys_models`: List of models (default: Granite + Mistral)
- `ocp4_workload_litellm_virtual_keys_duration`: Key duration (default: "30d")
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
- name: Provision LiteLLM virtual key
  hosts: localhost
  vars:
    ACTION: "provision"
    guid: "abc123"
    litellm_url: "https://litellm-rhpds.apps.cluster.com"
    litellm_master_key: "sk-xxxxx"
    ocp4_workload_litellm_virtual_keys_models:
      - "openai/granite-3-2-8b-instruct"
      - "openai/mistral-7b-instruct"
    ocp4_workload_litellm_virtual_keys_duration: "10d"
  tasks:
    - include_role:
        name: rhpds.litellm_virtual_keys.ocp4_workload_litellm_virtual_keys

    - debug:
        msg: "Virtual Key: {{ litellm_virtual_key }}"
```

## CI Lifecycle Integration

Virtual keys automatically sync with RHDP Catalog Item lifecycle:

### Auto-Expiration with CI Runtime

When `agnosticd_user_info.runtime` is available, keys automatically expire when the catalog item expires:

```yaml
# Catalog item runtime: 7 days (168 hours)
# Key duration: Automatically set to 168h (matches CI)
```

No hardcoded durations needed! Keys follow the catalog item lifecycle.

### Auto-Extension on User Extension

Configure a lifecycle hook in AgnosticV to extend keys when users click "Extend" in the UI:

**AgnosticV Configuration (`common.yaml`):**

```yaml
lifecycle_hooks:
  extend:
    - name: Extend LiteLLM Virtual Key
      playbook: extend_key.yml
```

**How it works:**

1. User clicks "Extend" in RHDP UI → CI runtime updated to 14 days
2. AgnosticV triggers `extend_key.yml` playbook
3. Playbook reads saved key ID from user_data
4. Calls LiteLLM API to extend key expiration to match new CI runtime
5. Key now expires with the extended CI

**Benefits:**

✅ Keys never expire before CI \
✅ Keys auto-cleanup when CI is destroyed \
✅ No manual key management needed \
✅ Works with any CI runtime (7d, 14d, 30d, etc.)

### Fallback Behavior

If `agnosticd_user_info.runtime` is not available (standalone use), falls back to configured duration:

```yaml
ocp4_workload_litellm_virtual_keys_duration: "30d"  # Used when no CI runtime
```

## Troubleshooting

### Get LiteLLM Master Key

The master key is required to authenticate with the LiteLLM API. To retrieve it:

```bash
# 1. Find LiteLLM deployment and check environment variables
oc get deployment -n rhpds -o yaml | grep -A 3 LITELLM_MASTER_KEY

# 2. Check secrets in the namespace
oc get secrets -n rhpds | grep litellm

# Extract master key from secret (if stored there)
oc get secret <secret-name> -n rhpds -o jsonpath='{.data.LITELLM_MASTER_KEY}' | base64 -d
```

The master key should start with `sk-` (e.g., `sk-1234567890abcdef...`).

### Common Issues

**401 Authentication Error**: Master key is incorrect or not in `sk-` format
- Verify the master key starts with `sk-`
- Check that you're using the actual key, not a hash

**Key Already Exists**: The key with this GUID already exists
- Run with `ACTION: destroy` first to delete the old key
- Or use a different GUID

**Double Slash in URL**: The `litellm_url` should not have a trailing slash
- ✅ Correct: `https://litellm-rhpds.apps.cluster.com`
- ❌ Wrong: `https://litellm-rhpds.apps.cluster.com/`
```
