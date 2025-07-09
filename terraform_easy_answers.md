# Easy-to-Remember Terraform Interview Guide - 6 Years Experience

## Core Concepts (Remember: SPIT)

### Q1: What is Terraform state and why is it important?

**Simple Answer (Remember: TRACK):**
- **T**rack resources: Knows what Terraform manages
- **R**ecord metadata: Stores resource details and relationships  
- **A**void API calls: Caches info for faster operations
- **C**ompare changes: Maps config to real world
- **K**eep dependencies: Understands resource order

**Common Problem:** State gets out of sync
**Simple Fix:** Use `terraform import` for missing resources, `terraform refresh` to sync

### Q2: Difference between Plan and Apply?

**Easy Memory Trick:**
- **Plan = Preview** (like shopping cart - shows what you'll buy)
- **Apply = Purchase** (actually buys/changes things)

**Plan:** Shows what WILL happen, makes no changes
**Apply:** Actually DOES the changes shown in plan

### Q3: How to handle secrets in Terraform?

**Remember: NEVER-STORE**
- **N**ever hardcode secrets in files
- **E**xternal secret managers (AWS Secrets, Vault)
- **V**ariables marked as sensitive
- **E**ncrypt state files
- **R**emote state only

- **S**ervice accounts not user keys
- **T**ime-limited credentials  
- **O**utputs marked sensitive
- **R**otate regularly
- **E**nvironment variables for auth

## State Management (Remember: CRUD-R)

### Q4: What is remote state?

**Simple Concept:**
Instead of keeping state file on your laptop, store it in the cloud where everyone can access it.

**Benefits (Remember: SLBS):**
- **S**haring: Team can collaborate
- **L**ocking: Prevents conflicts
- **B**ackup: Automatic versioning
- **S**ecurity: Better access control

**Popular Backends:** S3+DynamoDB (AWS), Azure Storage, Terraform Cloud

### Q5: What is state drift?

**Simple Definition:** When real infrastructure doesn't match what Terraform thinks exists.

**Causes (Remember: MOFE):**
- **M**anual changes in console
- **O**ther tools changing resources
- **F**ailed applies
- **E**xternal deletions

**Solutions (Remember: RIRI):**
- **R**efresh state
- **I**mport missing resources
- **R**emove deleted resources
- **I**gnore and recreate

### Q6: When to use workspaces vs separate state files?

**Easy Rule:**
- **Same infrastructure, different environments** = Workspaces
- **Different infrastructure patterns** = Separate state files

**Workspaces Good For:** dev/staging/prod with same resources
**Separate States Good For:** Different teams, different security needs, large scale

## Modules (Remember: SILO)

### Q7: How to design good modules?

**Remember: SILO Principle**
- **S**ingle purpose: One job per module
- **I**nputs validated: Check what comes in
- **L**oose coupling: Don't depend too much on others
- **O**utputs everything useful: Share what others need

**Module Structure (Remember: VIO):**
- **V**ariables.tf: What goes in
- **I**mplementation (main.tf): What it does  
- **O**utputs.tf: What comes out

### Q8: How to handle module dependencies?

**Three Ways (Remember: DOR):**
- **D**irect: Pass outputs as inputs
- **O**utside: Use data sources
- **R**emote: terraform_remote_state

**Golden Rule:** Keep modules independent, avoid circular dependencies

## Performance (Remember: SPOT)

### Q9: How to optimize Terraform performance?

**Remember: SPOT**
- **S**plit large state files
- **P**arallelism adjustment  
- **O**ptimize providers
- **T**argeted operations

**When Terraform is Slow:**
1. Too many resources in one state = Split them
2. API rate limits = Reduce parallelism
3. Complex dependencies = Simplify structure
4. Large state file = Use targeted applies

### Q10: How to scale Terraform for large teams?

**Scaling Strategy (Remember: TEAM):**
- **T**eam ownership: Different teams, different state files
- **E**nvironment separation: dev/staging/prod isolation
- **A**utomation: CI/CD for all changes
- **M**odule standards: Reusable, tested components

## CI/CD (Remember: VPA)

### Q11: Terraform in CI/CD pipeline stages?

**Remember: VPA**
- **V**alidate: Check format, syntax, security
- **P**lan: Show what will change
- **A**pply: Make the changes (with approval)

**Key Points:**
- Always plan before apply
- Save plans for approval workflows
- Use separate pipelines per environment
- Implement proper rollback procedures

### Q12: How to handle credentials in CI/CD?

**Best Practice (Remember: SOAP):**
- **S**ervice accounts not user credentials
- **O**IDC/federation when possible
- **A**uto-rotating credentials
- **P**rinciple of least privilege

**Never:** Hardcode secrets, use personal accounts, store in code

## Troubleshooting (Remember: DIVE)

### Q13: How to debug Terraform issues?

**Debugging Steps: DIVE**
- **D**ebug logging: Set TF_LOG=DEBUG
- **I**nspect state: terraform show, terraform state list
- **V**alidate config: terraform validate
- **E**xamine errors: Read error messages carefully

**Common Issues (Remember: SPAN):**
- **S**tate problems: drift, corruption, locking
- **P**ermission issues: IAM, RBAC failures
- **A**PI limits: rate limiting, timeouts
- **N**etwork issues: connectivity, DNS

### Q14: What if terraform apply fails halfway?

**Recovery Steps (Remember: SRAP):**
- **S**top and assess: Don't panic, check what happened
- **R**eview state: Run terraform plan to see current state
- **A**ddress issue: Fix the root cause
- **P**roceed carefully: Apply targeted fixes

**Prevention:** Test in dev first, smaller changes, proper monitoring

## Advanced Topics

### Q15: How to implement infrastructure governance?

**Governance Framework (Remember: PACE):**
- **P**olicy as code: Automated compliance checks
- **A**ccess controls: Who can change what
- **C**ost controls: Budget limits and alerts
- **E**nforcement: Automated scanning and blocking

**Tools:** Sentinel, OPA, Checkov, cost management tools

### Q16: Disaster recovery with Terraform?

**DR Strategy (Remember: MASH):**
- **M**ulti-region: Deploy across regions
- **A**utomation: Automated failover
- **S**napshots: Regular backups
- **H**ealth checks: Monitor and alert

**Key Point:** Test your DR regularly, not just when disaster strikes

## Scenario Questions

### Q17: Team wants to migrate from manual to Terraform. Your approach?

**Migration Strategy (Remember: PIPE):**
- **P**lan: Assess current infrastructure
- **I**mport: Start with terraform import
- **P**rioritize: Begin with non-critical resources
- **E**ducate: Train team on Terraform workflows

**Start Small:** Begin with new projects, gradually import existing

### Q18: Developers need infrastructure access but limited permissions?

**Solution (Remember: STAR):**
- **S**elf-service: Pre-approved templates
- **T**emplate-based: Standardized modules
- **A**pproval workflows: Automated reviews
- **R**ole-based access: Right permissions for right people

**Tools:** Terraform Cloud workspaces, service catalogs

### Q19: How to handle secrets that change frequently?

**Strategy (Remember: FADE):**
- **F**etch at runtime: Use data sources
- **A**uto-rotation: External secret managers
- **D**on't store: Never in state or config
- **E**xternal systems: Vault, AWS Secrets Manager

### Q20: Multiple teams working on same infrastructure?

**Organization (Remember: MOST):**
- **M**odular design: Shared, reusable modules
- **O**wnership boundaries: Clear team responsibilities
- **S**tate separation: Different state files
- **T**eam workflows: Proper branching and reviews

## Memory Tips for Interviews

### Quick Reference (Remember: State-Modules-Scale-Secure)

**State Management:**
- Remote state for teams
- Locking prevents conflicts
- Import for existing resources
- Refresh to sync

**Modules:**
- Single purpose design
- Version your modules
- Output what others need
- Keep them independent

**Scaling:**
- Split large state files
- Separate by teams/environments
- Use CI/CD automation
- Implement governance

**Security:**
- External secret management
- Least privilege access
- Encrypt everything
- Audit regularly

### Interview Success Formula

**When Asked Any Question:**
1. **Start with simple definition**
2. **Give real-world example**
3. **Mention common problems**
4. **Explain your solution**
5. **Share lessons learned**

**Example Response Structure:**
"Terraform state is like a inventory list of your infrastructure. In my experience at [company], we had issues with... We solved it by... The key lesson was..."

### Final Tips

**Before Interview:**
- Practice explaining concepts without jargon
- Prepare 2-3 real examples from your experience
- Know your biggest Terraform challenge and how you solved it
- Be ready to draw simple diagrams

**During Interview:**
- Start simple, add complexity if asked
- Use analogies (state = inventory, plan = preview)
- Share real problems you've solved
- Show you understand trade-offs

