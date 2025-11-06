# Testing LiteLLM Virtual Keys Collection

## Setup

### 1. Get LiteLLM Credentials

```bash
# Get LiteLLM URL
export LITELLM_URL=$(oc get route litellm -n litemaas -o jsonpath='{.spec.host}')

# Get Master Key
export LITELLM_MASTER_KEY=$(oc get secret litellm-secret -n litemaas -o jsonpath='{.data.LITELLM_MASTER_KEY}' | base64 -d)

# Verify
echo "URL: https://${LITELLM_URL}"
echo "Master Key: ${LITELLM_MASTER_KEY}"
```

### 2. Install Collection Locally

From the collection root directory:

```bash
cd ~/work/code/rhpds.litellm_virtual_keys

# Build the collection
ansible-galaxy collection build --force

# Install locally
ansible-galaxy collection install rhpds-litellm_virtual_keys-0.2.0.tar.gz --force
```

## Run Tests

### Test 1: Provision Virtual Key (ACTION=provision)

```bash
cd ~/work/code/rhpds.litellm_virtual_keys/tests

# Run provision test
ansible-playbook test_provision.yml \
  -e litellm_url="https://${LITELLM_URL}" \
  -e litellm_master_key="${LITELLM_MASTER_KEY}"
```

**Expected Output**:
- Creates key with alias `virtkey-test123`
- Displays the generated virtual key
- Tests idempotency (second run should skip creation)

### Test 2: Destroy Virtual Key (ACTION=destroy)

```bash
# Run destroy test
ansible-playbook test_destroy.yml \
  -e litellm_url="https://${LITELLM_URL}" \
  -e litellm_master_key="${LITELLM_MASTER_KEY}"
```

**Expected Output**:
- Deletes the key created in Test 1
- Shows deletion confirmation

### Test 3: Different Subscription Packages

```bash
# Test ai-beginner (Granite only, 7 days)
ansible-playbook test_provision.yml \
  -e litellm_url="https://${LITELLM_URL}" \
  -e litellm_master_key="${LITELLM_MASTER_KEY}" \
  -e guid=test-beginner \
  -e subscription_package=ai-beginner

# Test ai-researcher (3 models, 90 days)
ansible-playbook test_provision.yml \
  -e litellm_url="https://${LITELLM_URL}" \
  -e litellm_master_key="${LITELLM_MASTER_KEY}" \
  -e guid=test-researcher \
  -e subscription_package=ai-researcher

# Test lab-dev (Granite + Mistral, 14 days)
ansible-playbook test_provision.yml \
  -e litellm_url="https://${LITELLM_URL}" \
  -e litellm_master_key="${LITELLM_MASTER_KEY}" \
  -e guid=test-labdev \
  -e subscription_package=lab-dev

# Cleanup
ansible-playbook test_destroy.yml \
  -e litellm_url="https://${LITELLM_URL}" \
  -e litellm_master_key="${LITELLM_MASTER_KEY}" \
  -e guid=test-beginner

ansible-playbook test_destroy.yml \
  -e litellm_url="https://${LITELLM_URL}" \
  -e litellm_master_key="${LITELLM_MASTER_KEY}" \
  -e guid=test-researcher

ansible-playbook test_destroy.yml \
  -e litellm_url="https://${LITELLM_URL}" \
  -e litellm_master_key="${LITELLM_MASTER_KEY}" \
  -e guid=test-labdev
```

## Verify in LiteLLM Admin UI

1. Login to LiteLLM Admin Portal
2. Go to **Virtual Keys** section
3. You should see keys like:
   - `virtkey-test123`
   - `virtkey-test-beginner`
   - etc.

## Test API Access

After creating a key, test it works:

```bash
# Get the virtual key from playbook output
VIRTUAL_KEY="sk-xxxxx"

# Test with Granite
curl https://${LITELLM_URL}/v1/completions \
  -H "Authorization: Bearer ${VIRTUAL_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "openai/granite-3-2-8b-instruct",
    "prompt": "What is OpenShift?",
    "max_tokens": 50
  }'
```

## Troubleshooting

### Error: "key not allowed to access model"

Add the model to Personal Models in LiteLLM Admin UI.

### Error: "Authentication failed"

Check that LITELLM_MASTER_KEY is correct:
```bash
oc get secret litellm-secret -n litemaas -o jsonpath='{.data.LITELLM_MASTER_KEY}' | base64 -d
```

### Error: "Connection refused"

Check LiteLLM URL is correct:
```bash
oc get route litellm -n litemaas -o jsonpath='{.spec.host}'
```
