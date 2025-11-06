# Example Playbooks

Simple playbooks for provisioning and destroying LiteLLM virtual keys.

## Provision a Key

Creates a virtual key with default settings (ai-developer package):

```bash
ansible-playbook provision_key.yml \
  -e guid=user123 \
  -e litellm_url=https://litellm-rhpds.apps.cluster.com \
  -e litellm_master_key=sk-xxxxx
```

With custom subscription package:

```bash
ansible-playbook provision_key.yml \
  -e guid=user123 \
  -e litellm_url=https://litellm-rhpds.apps.cluster.com \
  -e litellm_master_key=sk-xxxxx \
  -e subscription_package=ai-researcher
```

## Destroy a Key

Deletes the virtual key:

```bash
ansible-playbook destroy_key.yml \
  -e guid=user123 \
  -e litellm_url=https://litellm-rhpds.apps.cluster.com \
  -e litellm_master_key=sk-xxxxx
```

## Available Subscription Packages

- `ai-beginner` - Granite only, 7 days
- `ai-developer` - Granite + Mistral, 30 days (default)
- `ai-researcher` - Granite + Mistral + Llama, 90 days
- `lab-dev` - Granite + Mistral, 14 days
- `lab-prod` - Granite + Mistral, 90 days
