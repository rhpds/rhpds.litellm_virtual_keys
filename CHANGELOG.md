# Changelog

All notable changes to this project will be documented in this file.

## [0.2.0] - 2025-11-06

### Major Refactoring

- Refactored to core_workload pattern with ACTION-based routing
- Single role `ocp4_workload_litellm_virtual_keys` replaces separate create/delete roles
- Uses `ACTION=provision` or `ACTION=destroy` pattern
- All variables now prefixed with `ocp4_workload_litellm_virtual_keys_*`

### Fixed

- **Key deletion now works properly** - Keys no longer reappear after deletion
- Fixed API endpoints:
  - Use `/key/list` to get all token hashes
  - Use `/key/info?key={token}` to get key details
  - Use `/key/delete` with token hash for deletion
- Fixed JSON parsing issue with `json['keys']` vs `json.keys()` method conflict

### Added

- Bundled `agnosticd_user_info` module for standalone operation
- User info messages and data for AgnosticD integration
- Force database deletion option for LiteLLM API workaround
- Comprehensive debug output for troubleshooting
- Example playbooks in `playbooks/` directory
- Full documentation (README.md for collection and role)

### Changed

- Variable naming convention to match AgnosticD standards
- Test playbooks updated to new pattern
- Improved idempotency handling

### Breaking Changes

- Role name changed from `litellm_virtual_key_create` to `ocp4_workload_litellm_virtual_keys`
- Requires `ACTION` variable (`provision` or `destroy`)
- All variables renamed with `ocp4_workload_*` prefix

## [0.1.0] - 2025-11-06

### Added

- Initial release
- Two roles: `litellm_virtual_key_create` and `litellm_virtual_key_delete`
- Five subscription packages (ai-beginner, ai-developer, ai-researcher, lab-dev, lab-prod)
- GUID-based key naming (`virtkey-{GUID}`)
- Metadata tracking
- Idempotency support
- No budget limits (unlimited usage)

### Credits

- **Authors**: Prakhar Srivastava, Ashok Jammula
- **LiteMaaS Project**: Tony Kay
- **Architecture**: Ritesh Shah
