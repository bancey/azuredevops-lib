# azuredevops-lib

[![YAML Lint](https://github.com/bancey/azuredevops-lib/actions/workflows/yaml-lint.yml/badge.svg)](https://github.com/bancey/azuredevops-lib/actions/workflows/yaml-lint.yml)
[![Validate Templates](https://github.com/bancey/azuredevops-lib/actions/workflows/validate-templates.yml/badge.svg)](https://github.com/bancey/azuredevops-lib/actions/workflows/validate-templates.yml)
[![Documentation Check](https://github.com/bancey/azuredevops-lib/actions/workflows/documentation-check.yml/badge.svg)](https://github.com/bancey/azuredevops-lib/actions/workflows/documentation-check.yml)
[![CI](https://github.com/bancey/azuredevops-lib/actions/workflows/ci.yml/badge.svg)](https://github.com/bancey/azuredevops-lib/actions/workflows/ci.yml)

A comprehensive collection of reusable Azure DevOps pipeline templates and components for infrastructure automation, configuration management, and CI/CD workflows.

## 🎯 Project Scope

This library provides battle-tested, parameterized YAML templates for Azure DevOps pipelines that handle common infrastructure and deployment tasks:

- **Infrastructure as Code (IaC)**: Terraform planning, applying, and state management with Azure backend
- **Configuration Management**: Ansible playbook execution with Azure Key Vault integration
- **Network Operations**: Host connectivity checks and VPN connections via Twingate
- **CI/CD Integration**: GitHub authentication and automated workflow integration
- **Image Building**: Packer-based image creation workflows

## 📦 Installation

### Prerequisites

- Azure DevOps organization with appropriate permissions
- Azure subscription with required service connections configured
- Azure Key Vault for secret management (recommended)

### Adding to Your Pipeline

To use these templates in your Azure DevOps pipelines, reference them using the `resources` section:

```yaml
resources:
  repositories:
    - repository: azuredevops-lib
      type: github
      name: bancey/azuredevops-lib
      ref: main

stages:
  - template: stages/terraform.yaml@azuredevops-lib
    parameters:
      # Your parameters here
```

## 🚀 Usage Examples

### Terraform Infrastructure Management

Use the Terraform stage template for complete infrastructure lifecycle management:

```yaml
stages:
  - template: stages/terraform.yaml@azuredevops-lib
    parameters:
      stageName: terraform_infrastructure
      backendStorageAccount: mytfstatestorage
      workingDirectory: $(System.DefaultWorkingDirectory)/terraform
      azureRmKey: infrastructure.tfstate
      serviceConnection: my-azure-service-connection
      variableFilePath: terraform/environments/prod.tfvars
      runApply: true
      runDestroy: false
      extraCommandArgs: "-target=azurerm_resource_group.main"
```

### Ansible Configuration Management

Execute Ansible playbooks with secure credential management:

```yaml
steps:
  - template: steps/ansible.yaml@azuredevops-lib
    parameters:
      playbook: playbooks/configure-servers.yml
      requirementsFile: requirements.yml
      keyVaultName: my-keyvault
      privateKeySecretName: ansible-ssh-key
      serviceConnection: my-azure-service-connection
      secrets:
        - database-password
        - api-key
```

### Host Connectivity Checks

Verify host availability before deployments:

```yaml
jobs:
  - template: jobs/hosts-online-precheck.yaml@azuredevops-lib

stages:
  - template: stages/check-hosts-online.yaml@azuredevops-lib
    parameters:
      stageName: connectivity_check
      dependencies: []
```

### GitHub Authentication

Authenticate with GitHub for automated workflows:

```yaml
steps:
  - template: steps/gh-auth.yaml@azuredevops-lib
    parameters:
      serviceConnection: my-azure-service-connection
      keyVaultName: my-keyvault
      privateKeySecretName: github-private-key
      githubAppIdSecretName: github-app-id
      githubInstallationIdSecretName: github-installation-id
```

### Individual Step Templates

Use individual steps for more granular control:

```yaml
steps:
  # Packer image building
  - template: steps/packer.yaml@azuredevops-lib
    parameters:
      # Packer-specific parameters

  # Twingate VPN connection
  - template: steps/twingate-connect.yaml@azuredevops-lib
    parameters:
      # Twingate connection parameters

  # Single host connectivity check
  - template: steps/check-host-online.yaml@azuredevops-lib
    parameters:
      # Host check parameters
```

## 📁 Component Reference

### Stages
- `stages/terraform.yaml` - Complete Terraform workflow (plan/apply/destroy)
- `stages/check-hosts-online.yaml` - Multi-host connectivity verification

### Jobs
- `jobs/hosts-online-precheck.yaml` - Pre-deployment host availability check

### Steps
- `steps/terraform.yaml` - Terraform operations with Azure backend
- `steps/ansible.yaml` - Ansible playbook execution with secret management
- `steps/gh-auth.yaml` - GitHub App authentication
- `steps/packer.yaml` - Packer image building
- `steps/twingate-connect.yaml` - Twingate VPN connection
- `steps/check-host-online.yaml` - Single host connectivity check
- `steps/check-hosts-online.yaml` - Multiple host connectivity check

### Resources
- `resources/tfcmt.yaml` - Terraform comment automation configuration

## 🤝 Contributing

We welcome contributions to improve and extend this library! Here's how you can help:

### Continuous Integration

This repository uses GitHub Actions for automated testing and validation:

- **YAML Linting**: Validates YAML syntax and style across all templates
- **Template Validation**: Checks YAML structure and parsing
- **Documentation Checks**: Lints markdown files and validates links
- **CI Summary**: Provides comprehensive status on all checks

All workflows run automatically on pull requests and pushes to the main branch. You can see the status of these checks in the Actions tab or on your pull request.

### Getting Started

1. **Fork the repository** on GitHub
2. **Clone your fork** locally:
   ```bash
   git clone https://github.com/your-username/azuredevops-lib.git
   cd azuredevops-lib
   ```
3. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

### Development Guidelines

#### Template Structure
- Follow the existing YAML structure and parameter naming conventions
- Include comprehensive parameter documentation with types and defaults
- Use conditional logic (`${{ if }}`) for optional features
- Group related parameters logically

#### Parameter Standards
- Use camelCase for parameter names
- Provide sensible defaults where possible
- Include `displayName` for user-facing parameters
- Document parameter types (`string`, `boolean`, `object`, etc.)

#### Example Template Structure
```yaml
parameters:
  - name: parameterName
    displayName: Human-readable parameter description
    type: string
    default: sensible-default

steps:
  - task: SomeTask@1
    displayName: Clear step description
    inputs:
      parameter: ${{ parameters.parameterName }}
```

### Testing Your Changes

1. **Validate YAML syntax**:
   ```bash
   # Install yamllint
   pip install yamllint

   # Run yamllint on all files
   yamllint -f parsable -c .yamllint.yml .
   ```

2. **Validate YAML structure**:
   ```bash
   # Install PyYAML
   pip install pyyaml

   # Check all YAML files
   find . -type f \( -name "*.yaml" -o -name "*.yml" \) ! -path "./.git/*" | while read file; do
     echo "Checking: $file"
     python -c "import yaml; yaml.safe_load(open('$file'))"
   done
   ```

3. **Test in a pipeline**: Create a test pipeline in your Azure DevOps organization to validate functionality

4. **Document your changes**: Update this README if you're adding new components or changing existing behavior

### Submitting Changes

1. **Commit your changes** with clear, descriptive messages:
   ```bash
   git commit -m "Add new Kubernetes deployment template"
   ```

2. **Push to your fork**:
   ```bash
   git push origin feature/your-feature-name
   ```

3. **Create a Pull Request** with:
   - Clear description of changes
   - Usage examples for new components
   - Any breaking changes clearly marked

### Code of Conduct

- Be respectful and inclusive in all interactions
- Focus on constructive feedback and collaboration
- Help maintain high code quality and documentation standards

## 📋 Template Parameters

### Common Parameters

Most templates accept these common parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `serviceConnection` | string | - | Azure service connection name |
| `keyVaultName` | string | - | Azure Key Vault name for secrets |
| `workingDirectory` | string | - | Working directory for operations |

### Terraform-Specific Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `backendStorageAccount` | string | - | Azure storage account for Terraform state |
| `backendContainer` | string | `tfstate` | Storage container name |
| `azureRmKey` | string | - | Terraform state file key |
| `runApply` | boolean | - | Whether to run terraform apply |
| `runDestroy` | boolean | - | Whether to run terraform destroy |
| `parallelism` | number | `-1` | Terraform parallelism setting |

## 🔒 Security Considerations

- **Secrets Management**: Always use Azure Key Vault for sensitive data
- **Service Connections**: Use managed identity where possible
- **Permissions**: Follow principle of least privilege
- **State Files**: Ensure Terraform state files are properly secured in Azure Storage

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- **Issues**: Report bugs and feature requests via [GitHub Issues](https://github.com/bancey/azuredevops-lib/issues)
- **Discussions**: Join the conversation in [GitHub Discussions](https://github.com/bancey/azuredevops-lib/discussions)
- **Documentation**: Additional examples and guides in the [Wiki](https://github.com/bancey/azuredevops-lib/wiki)

## 🔄 Changelog

See [CHANGELOG.md](CHANGELOG.md) for a detailed history of changes and releases.

---

*Made with ❤️ for the Azure DevOps community*