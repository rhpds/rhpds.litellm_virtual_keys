# LiteLLM Virtual Keys Collection

Ansible collection for managing LiteLLM virtual keys in Red Hat Demo Platform (RHDP) lab environments.

**CRITICAL NOTICE:** Old cluster `litellm-rhpds` is being decommissioned **June 21, 2026**. All new keys should target the production cluster `maas-rdhp`. Update your `litellm_url` configuration to point to the new cluster.

## What This Does

Creates and deletes AI model access keys for lab users. Each lab gets a unique key named `virtkey-{GUID}` with access to specific AI models based on subscription tier.

## Authors

- **Prakhar Srivastava** - Manager, Technical Marketing, Red Hat
- **Ashok Jammula** - Senior Architect, Red Hat

### Credits

- **Tony Kay** - LiteMaaS upstream project
- **Ritesh Shah** - Architecture and design guidance

## Quick Start

### Install

```bash
ansible-galaxy collection install rhpds.litellm_virtual_keys
```

### Create a Key

```bash
ansible-playbook playbook.yml \
  -e ACTION=provision \
  -e guid=user123 \
  -e litellm_url=https://litellm-rhpds.apps.cluster.com \
  -e litellm_master_key=sk-xxxxx
```

### Delete a Key

```bash
ansible-playbook playbook.yml \
  -e ACTION=destroy \
  -e guid=user123 \
  -e litellm_url=https://litellm-rhpds.apps.cluster.com \
  -e litellm_master_key=sk-xxxxx
```

## Available Models on maas-rdhp (Production Cluster)

### Vertex AI Models (Restricted Access)

Models requiring admin approval (>120B parameters):

- `minimax-m2` — RESTRICTED (>120B)
- `qwen3-235b` — RESTRICTED (>120B)

Models with self-service access:

- `gpt-oss-120b` — Open access
- `gpt-oss-20b` — Open access

### Locally Hosted Models (Self-Service)

- `granite-3-2-8b-instruct`
- `granite-4-0-h-tiny`
- `granite-2b-cpu`
- `llama-scout-17b`
- `llama-31-70b-cpu`
- `Llama-Guard-3-1B`
- `codellama-7b-instruct`
- `deepseek-r1-distill-qwen-14b`
- `qwen3-14b`
- `qwen25-3b-cpu`
- `microsoft-phi-4`
- `phi3-mini-cpu`
- `nomic-embed-text-v1-5`
- `Docling` — Document conversion

## Subscription Packages (Legacy - Update to maas-rdhp)

Choose what models and how long:

| Package | Models | Duration | Use Case | Notes |
|---------|--------|----------|----------|-------|
| `ai-beginner` | Granite | 7 days | Intro workshops | Update to maas-rdhp |
| `ai-developer` | Granite + Mistral | 30 days | Developer labs (default) | Update to maas-rdhp |
| `ai-researcher` | Granite + Mistral + Llama | 90 days | Research projects | Update to maas-rdhp |
| `lab-dev` | Granite + Mistral | 14 days | Development testing | Update to maas-rdhp |
| `lab-prod` | Granite + Mistral | 90 days | Production labs | Update to maas-rdhp |

## How It Works with AgnosticD

### User Info Messages

When you create a key, users receive an email with:

```
LiteLLM API Endpoint: https://maas-rdhp.apps.cluster.com/v1
LiteLLM Virtual Key: sk-abc123xyz...
Available Models: granite-3-2-8b-instruct, deepseek-r1-distill-qwen-14b
Key Duration: 30d
```

**Note:** Update all references from `https://litellm-rhpds.apps.cluster.com` to `https://maas-rdhp.apps.cluster.com` before June 21, 2026.

These messages come from the `agnosticd_user_info` module and appear in:
- User provisioning emails
- RHDP web UI notifications
- Babylon catalog item details

### User Info Data

The collection also stores structured data that automation can read:

```yaml
litellm_api_endpoint: "https://litellm-rhpds.apps.cluster.com"
litellm_api_base_url: "https://litellm-rhpds.apps.cluster.com/v1"
litellm_virtual_key: "sk-abc123xyz..."
litellm_key_alias: "virtkey-user123"
litellm_available_models:
  - "openai/granite-3-2-8b-instruct"
  - "openai/mistral-7b-instruct"
litellm_key_duration: "30d"
litellm_subscription_package: "ai-developer"
```

This data is accessible via:
- AgnosticD variables and facts
- RHDP API endpoints
- User info JSON exports

**MIGRATION:** Ensure all endpoint references use `maas-rdhp` (production cluster) instead of the deprecated `litellm-rhpds` cluster, which shuts down June 21, 2026.

### How to Enable User Info

User info is **enabled by default** when running in AgnosticD context. The collection includes the `agnosticd_user_info` module, so it works out of the box.

To disable (for testing):

```yaml
ocp4_workload_litellm_virtual_keys_enable_user_info_messages: false
ocp4_workload_litellm_virtual_keys_enable_user_info_data: false
```

## Example Playbook

```yaml
---
- name: Provision LiteLLM Key for Lab
  hosts: localhost
  gather_facts: false

  vars:
    ACTION: "provision"
    guid: "{{ lookup('env', 'GUID') }}"
    litellm_url: "https://litellm-rhpds.apps.cluster.com"
    litellm_master_key: "{{ lookup('env', 'LITELLM_MASTER_KEY') }}"

    # Optional: Choose subscription package
    ocp4_workload_litellm_virtual_keys_subscription_package: "ai-developer"

  tasks:
    - name: Create virtual key
      include_role:
        name: rhpds.litellm_virtual_keys.ocp4_workload_litellm_virtual_keys

    - name: Show key to user
      debug:
        msg:
          - "Your AI API Key: {{ litellm_virtual_key }}"
          - "API Endpoint: {{ litellm_api_endpoint }}/v1"
          - "Models: {{ litellm_available_models | join(', ') }}"
```

## Required Variables

- `ACTION`: Set to `provision` (create key) or `destroy` (delete key)
- `guid`: Lab environment identifier (e.g., `abc123`)
- `litellm_url`: LiteLLM API URL
- `litellm_master_key`: LiteLLM admin key

## Optional Variables

- `ocp4_workload_litellm_virtual_keys_subscription_package`: Package name (default: `ai-developer`)
- `ocp4_workload_litellm_virtual_keys_force_db_delete`: Force database deletion if API fails (default: `false`)
- `ocp4_workload_litellm_virtual_keys_namespace`: Kubernetes namespace (default: `rhpds`)

## Testing

```bash
cd tests

# Create a test key
ansible-playbook test_provision.yml \
  -e litellm_url="https://litellm.example.com" \
  -e litellm_master_key="sk-xxxxx"

# Delete the test key
ansible-playbook test_destroy.yml \
  -e litellm_url="https://litellm.example.com" \
  -e litellm_master_key="sk-xxxxx"
```

See `tests/README.md` for detailed testing instructions.

## How Keys Work

### Key Naming

All keys follow the pattern: `virtkey-{GUID}`

Example: If `guid=abc123`, the key alias is `virtkey-abc123`

### Key Lifecycle

1. **Create**: Generate key with selected models and duration
2. **Use**: Students access AI models via the key
3. **Expire**: Key automatically expires after duration
4. **Delete**: Manual cleanup when lab ends

### Idempotency

Safe to run multiple times:
- Creating an existing key → skips, returns existing key
- Deleting a missing key → skips, no error

## Troubleshooting

### Key not deleting properly

Enable database deletion workaround:

```yaml
ocp4_workload_litellm_virtual_keys_force_db_delete: true
```

This directly removes keys from PostgreSQL if the API fails.

### No models available

Check that models are configured in LiteLLM admin UI under "Personal Models".

### Authentication failed

Verify master key:

```bash
oc get secret litellm-secret -n rhpds \
  -o jsonpath='{.data.LITELLM_MASTER_KEY}' | base64 -d
```

## Repository

https://github.com/rhpds/rhpds.litellm_virtual_keys

## License

MIT
