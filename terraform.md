# Terraform Interview Questions & Answers - 6 Years Experience

## Core Terraform Concepts

### Q1: What is Terraform state and why is it important?

**Answer:**
Terraform state is a file that tracks the mapping between your configuration and real-world resources. It's crucial because:
- **Resource Tracking**: Knows which resources it manages
- **Performance**: Caches resource attributes to avoid API calls
- **Dependencies**: Maps resource relationships and dependencies
- **Metadata**: Stores resource metadata and configurations

**Common Issues:**
- State file corruption leads to infrastructure inconsistencies
- Manual changes outside Terraform cause state drift
- Concurrent operations without locking can corrupt state

**Solutions:**
- Use remote state with locking (S3 + DynamoDB)
- Implement regular state backups
- Use `terraform import` for existing resources
- Enable state versioning for rollback capability

### Q2: Explain the difference between Terraform plan and apply phases.

**Answer:**
**Plan Phase:**
- Creates execution plan by comparing desired state (config) with current state
- Shows what actions Terraform will take (create, update, destroy)
- No actual changes are made to infrastructure
- Can save plan to file for later execution

**Apply Phase:**
- Executes the plan to reach desired state
- Makes actual changes to infrastructure
- Updates state file with new resource information
- Can apply saved plan or generate new plan automatically

**Key Points:**
- Always review plan before apply in production
- Plan can change between plan and apply if infrastructure changes
- Use `-auto-approve` carefully, preferably only in automation

### Q3: How do you handle secrets and sensitive data in Terraform?

**Answer:**
**Best Practices:**
- **Mark variables as sensitive** to prevent output in logs
- **Use external secret management** (AWS Secrets Manager, HashiCorp Vault)
- **Avoid hardcoding** secrets in configuration files
- **Use data sources** to fetch secrets at runtime
- **Encrypt state files** when using remote backends

**Common Mistakes:**
- Storing secrets in variables files
- Committing sensitive data to version control
- Using local state files with secrets
- Not marking outputs as sensitive

**Security Measures:**
- Implement least privilege access policies
- Use service accounts instead of user credentials
- Enable audit logging for state access
- Regular rotation of credentials

### Q4: What are Terraform providers and how do they work?

**Answer:**
**Providers** are plugins that implement resource types for specific services (AWS, Azure, GCP, etc.).

**Key Concepts:**
- **Resource Types**: Define what can be managed (aws_instance, azurerm_vm)
- **Data Sources**: Read-only information from external systems
- **Provider Configuration**: Authentication and API endpoints
- **Provider Versioning**: Ensures compatibility and stability

**Common Provider Issues:**
- Version conflicts between different modules
- Authentication failures due to credential issues
- Rate limiting from cloud providers
- Provider bugs causing resource drift

**Best Practices:**
- Pin provider versions in production
- Use provider aliases for multi-region deployments
- Implement proper credential management
- Test provider updates in non-production first

## State Management & Remote Backends

### Q5: What is remote state and what are its benefits?

**Answer:**
**Remote State** stores terraform.tfstate file in a remote location instead of locally.

**Benefits:**
- **Team Collaboration**: Multiple team members can access same state
- **State Locking**: Prevents concurrent modifications
- **Security**: Better access control and encryption
- **Backup & Versioning**: Automatic backups and version history
- **CI/CD Integration**: Automation pipelines can access state

**Popular Backends:**
- **S3 + DynamoDB**: AWS native solution with locking
- **Azure Storage**: Azure native backend
- **Terraform Cloud**: HashiCorp managed service
- **Consul**: For on-premises or hybrid environments

**Implementation Considerations:**
- Choose backend based on your cloud provider
- Enable versioning and encryption
- Set up proper IAM permissions
- Configure state locking mechanism

### Q6: How do you handle state drift and what causes it?

**Answer:**
**State Drift** occurs when actual infrastructure differs from Terraform state.

**Common Causes:**
- Manual changes made outside Terraform
- Failed terraform applies leaving partial changes
- External automation tools modifying resources
- Resources deleted outside Terraform
- Configuration changes not applied through Terraform

**Detection Methods:**
- Regular `terraform plan` runs
- Automated drift detection scripts
- Monitoring alerts for infrastructure changes
- State refresh operations

**Resolution Strategies:**
- **Import**: Bring manually created resources under Terraform management
- **Remove**: Remove destroyed resources from state
- **Refresh**: Update state to match actual infrastructure
- **Replace**: Force recreation of modified resources

**Prevention:**
- Implement proper access controls
- Use consistent deployment processes
- Monitor infrastructure changes
- Educate team on Terraform workflows

### Q7: Explain Terraform workspaces and when to use them.

**Answer:**
**Workspaces** allow multiple instances of the same configuration with separate state files.

**Use Cases:**
- **Environment Separation**: dev, staging, prod environments
- **Feature Branches**: Testing infrastructure changes
- **Multi-tenancy**: Same infrastructure for different clients
- **Regional Deployments**: Same config across regions

**Benefits:**
- Same configuration code for multiple environments
- Easy environment switching
- Isolated state files
- Built-in workspace interpolation

**Limitations:**
- All environments share same configuration
- Not suitable for different infrastructure patterns
- Limited access control between workspaces
- Can become complex with many environments

**Alternative Approach:**
Use separate directories/repositories for truly different environments or when you need different security models.

## Modules & Code Organization

### Q8: How do you design reusable Terraform modules?

**Answer:**
**Good Module Design Principles:**

**Single Responsibility:**
- Each module should have one clear purpose
- Avoid creating monolithic modules
- Focus on logical grouping of resources

**Input Validation:**
- Use variable validation rules
- Provide clear variable descriptions
- Set sensible defaults where possible

**Output Everything Useful:**
- Expose resource IDs, ARNs, DNS names
- Include computed values other modules might need
- Document what each output provides

**Versioning Strategy:**
- Use semantic versioning (major.minor.patch)
- Tag releases in version control
- Use module registry for distribution

**Common Module Patterns:**
- **Composition**: Combine smaller modules
- **Configuration**: Accept configuration objects
- **Factory**: Generate multiple similar resources
- **Wrapper**: Simplify complex provider resources

### Q9: How do you handle dependencies between modules?

**Answer:**
**Dependency Management Strategies:**

**Direct Dependencies:**
- Pass outputs from one module as inputs to another
- Use explicit dependencies with depends_on
- Chain modules in logical order

**Implicit Dependencies:**
- Terraform automatically detects resource references
- Use data sources to query existing resources
- Reference module outputs in other resources

**Remote State Dependencies:**
- Use terraform_remote_state data source
- Access outputs from different state files
- Good for loosely coupled infrastructure

**Anti-Patterns to Avoid:**
- Circular dependencies between modules
- Tight coupling making modules non-reusable
- Complex nested module hierarchies
- Implicit dependencies that aren't clear

**Best Practices:**
- Keep modules focused and loosely coupled
- Use clear naming conventions
- Document module dependencies
- Test modules independently

### Q10: What are dynamic blocks and when do you use them?

**Answer:**
**Dynamic Blocks** allow you to generate repeated nested blocks based on complex values.

**Common Use Cases:**
- Security group rules based on variable lists
- Multiple ingress/egress rules
- Route table entries
- Load balancer listeners
- IAM policy statements

**When to Use:**
- Configuration varies based on input
- Need to generate multiple similar blocks
- Conditional resource configuration
- Complex nested structures

**When NOT to Use:**
- Simple static configurations
- When it makes code less readable
- Single instance blocks
- Over-engineering simple resources

**Key Benefits:**
- Reduces code duplication
- Enables flexible configurations
- Supports complex data structures
- Maintains DRY principles

**Considerations:**
- Can make code harder to read
- Debugging becomes more complex
- Plan output is less predictable
- Requires good variable validation

## Performance & Optimization

### Q11: How do you optimize Terraform performance for large infrastructures?

**Answer:**
**Performance Optimization Strategies:**

**Resource Organization:**
- Split large configurations into smaller state files
- Group related resources together
- Use separate state files for different lifecycle resources
- Implement logical boundaries (networking, compute, data)

**Parallelism Control:**
- Adjust parallelism based on provider limits
- Consider API rate limits
- Balance speed vs stability
- Monitor provider throttling

**State Management:**
- Use targeted operations for specific changes
- Implement state refresh strategies
- Minimize state file size
- Use remote state for better performance

**Planning Optimization:**
- Use `-refresh=false` for known stable environments
- Implement cached provider data
- Use data sources efficiently
- Avoid unnecessary dependencies

**Common Performance Issues:**
- Too many resources in single state file
- Excessive API calls during planning
- Complex dependency graphs
- Large state files

### Q12: How do you handle large-scale Terraform deployments?

**Answer:**
**Scaling Strategies:**

**State File Separation:**
- Separate by environment (dev, staging, prod)
- Separate by service/application
- Separate by team ownership
- Separate by lifecycle (long-lived vs short-lived)

**Module Strategy:**
- Create standardized modules for common patterns
- Implement module versioning and registry
- Use composition over inheritance
- Enforce coding standards

**Team Collaboration:**
- Implement proper branching strategies
- Use pull request workflows
- Enforce code reviews
- Automated testing and validation

**Deployment Automation:**
- CI/CD pipeline integration
- Automated plan/apply workflows
- Environment promotion strategies
- Rollback procedures

**Governance:**
- Policy as code implementation
- Cost management and budgeting
- Security scanning and compliance
- Resource tagging standards

## CI/CD & Automation

### Q13: How do you implement Terraform in CI/CD pipelines?

**Answer:**
**Pipeline Stages:**

**Validation Stage:**
- Format checking (`terraform fmt`)
- Configuration validation
- Security scanning (Checkov, Terrascan)
- Cost estimation (Infracost)
- Policy validation (Sentinel, OPA)

**Planning Stage:**
- Generate and review execution plans
- Comment plans on pull requests
- Save plans for approval workflows
- Detect drift and changes

**Apply Stage:**
- Apply approved plans
- Environment-specific deployments
- Automated rollback on failures
- State backup and recovery

**Best Practices:**
- Use separate pipelines for different environments
- Implement approval gates for production
- Store state remotely with locking
- Use service accounts for authentication
- Implement proper error handling

**Common Challenges:**
- Credential management across environments
- State locking conflicts
- Long-running apply operations
- Failed deployments and cleanup

### Q14: How do you handle secrets and credentials in CI/CD?

**Answer:**
**Authentication Methods:**

**Service Account Authentication:**
- Use IAM roles instead of access keys
- Implement OIDC/federation where possible
- Rotate credentials regularly
- Follow least privilege principles

**Secret Management:**
- Store secrets in dedicated secret managers
- Use pipeline secret management features
- Avoid hardcoding in configuration
- Implement secret rotation

**Environment Isolation:**
- Separate credentials per environment
- Use different accounts/subscriptions
- Implement proper access controls
- Audit credential usage

**Security Best Practices:**
- Enable audit logging
- Monitor for credential misuse
- Implement time-limited tokens
- Use encrypted communication channels

## Troubleshooting & Debugging

### Q15: How do you troubleshoot common Terraform issues?

**Answer:**
**Common Issues & Solutions:**

**State-Related Problems:**
- **State Corruption**: Restore from backup, rebuild state
- **State Locking**: Force unlock, check for orphaned locks
- **Drift Detection**: Use refresh, import, or remove resources
- **State Migration**: Plan carefully, backup before changes

**Resource Issues:**
- **Dependencies**: Check resource relationships, use depends_on
- **Timeouts**: Adjust timeout settings, check provider limits
- **Authentication**: Verify credentials and permissions
- **API Limits**: Implement retry logic, reduce parallelism

**Configuration Problems:**
- **Syntax Errors**: Use terraform validate
- **Variable Issues**: Check variable definitions and types
- **Provider Versions**: Pin versions, check compatibility
- **Module Errors**: Verify module sources and versions

**Debugging Techniques:**
- Enable debug logging (TF_LOG=DEBUG)
- Use terraform console for testing
- Examine provider documentation
- Check cloud provider console for actual state

### Q16: What do you do when Terraform apply fails halfway?

**Answer:**
**Immediate Response:**

**Assess the Situation:**
- Check what resources were created/modified
- Review error messages and logs
- Determine if infrastructure is in inconsistent state
- Identify root cause of failure

**Recovery Actions:**
- Run terraform plan to see current state
- Use terraform refresh to sync state
- Apply targeted fixes if possible
- Consider manual cleanup if necessary

**Common Failure Scenarios:**
- **Resource Creation Fails**: Some resources created, others failed
- **Network Issues**: Temporary connectivity problems
- **Permission Issues**: IAM/RBAC problems during apply
- **Resource Conflicts**: Naming conflicts or dependencies

**Prevention Strategies:**
- Implement proper error handling
- Use smaller, incremental changes
- Test in non-production environments first
- Implement rollback procedures
- Monitor apply operations

**Recovery Best Practices:**
- Document incident and resolution
- Update procedures based on learnings
- Implement additional safeguards
- Consider blue-green deployment strategies

## Advanced Topics

### Q17: How do you implement infrastructure governance and compliance?

**Answer:**
**Governance Framework:**

**Policy as Code:**
- Implement security policies (Sentinel, OPA)
- Enforce resource tagging standards
- Control resource types and sizes
- Mandate encryption and security settings

**Compliance Automation:**
- Automated security scanning
- Cost control and budgeting
- Resource inventory and tracking
- Audit logging and reporting

**Team Standards:**
- Code review requirements
- Module certification process
- Documentation standards
- Training and certification

**Monitoring & Alerting:**
- Infrastructure drift detection
- Cost anomaly detection
- Security compliance monitoring
- Performance and availability tracking

### Q18: How do you handle disaster recovery with Terraform?

**Answer:**
**DR Strategy Components:**

**Multi-Region Architecture:**
- Deploy infrastructure across multiple regions
- Implement automated failover mechanisms
- Use global load balancing
- Maintain synchronized data stores

**Backup and Recovery:**
- Regular state file backups
- Configuration version control
- Infrastructure snapshots
- Database backup strategies

**Automation:**
- Automated DR environment provisioning
- Health checks and monitoring
- Failover testing procedures
- Recovery time optimization

**Testing and Validation:**
- Regular DR drills and testing
- RTO/RPO measurement and improvement
- Documentation and runbooks
- Team training and procedures

## Scenario-Based Questions

### Q19: Your team wants to migrate from manual infrastructure to Terraform. How do you approach this?

**Answer:**
**Migration Strategy:**

**Assessment Phase:**
- Inventory existing infrastructure
- Identify dependencies and relationships
- Categorize by complexity and priority
- Estimate migration effort and timeline

**Planning Phase:**
- Design target Terraform architecture
- Create migration timeline and phases
- Develop rollback procedures
- Train team on Terraform practices

**Execution Phase:**
- Start with non-critical environments
- Import existing resources gradually
- Validate configurations thoroughly
- Implement CI/CD integration

**Best Practices:**
- Begin with greenfield projects
- Use terraform import for existing resources
- Implement gradual migration approach
- Maintain manual procedures as backup

### Q20: How do you handle a situation where developers need to deploy infrastructure but shouldn't have full access?

**Answer:**
**Access Control Strategy:**

**Role-Based Permissions:**
- Separate read/write access levels
- Environment-specific permissions
- Resource-type restrictions
- Time-limited access tokens

**Workflow Design:**
- Self-service infrastructure requests
- Automated approval workflows
- Template-based deployments
- Audit trail requirements

**Technical Implementation:**
- Use Terraform Cloud/Enterprise workspaces
- Implement policy as code
- Create standardized modules
- Use service catalogs

**Governance:**
- Pre-approved infrastructure patterns
- Cost controls and budgets
- Security policy enforcement
- Regular access reviews

