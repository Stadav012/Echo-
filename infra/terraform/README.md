# Terraform Skeleton (Azure)

This directory will contain Terraform for:
- Azure Container Apps / App Service (API and Web)
- Azure Database for PostgreSQL Flexible Server
- Azure Cache for Redis
- Azure Key Vault

## Structure
- `providers.tf` – AzureRM provider + backend
- `main.tf` – Core resources
- `variables.tf` – Variables
- `outputs.tf` – Outputs

## Next steps
- Configure remote state (Azure Storage) and service principal credentials.
- Add resources with minimal viable configuration and tags.
- Wire secrets from Key Vault.
