# CI/CD Interview Questions - GitHub Actions & Jenkins (5+ Years Experience)

## **GitHub Actions Questions**

### **Fundamental Concepts**

**Q1: Explain the core components of GitHub Actions and how they work together.**
- **Answer:** GitHub Actions consists of Workflows (YAML files in `.github/workflows/`), Jobs (groups of steps), Steps (individual tasks), Actions (reusable units), Runners (execution environments), and Events (triggers). Workflows are triggered by events, run jobs on runners, where each job contains steps that can use actions.

**Q2: What are the different types of runners in GitHub Actions, and when would you use each?**
- **Answer:** 
  - **GitHub-hosted runners:** Ubuntu, Windows, macOS provided by GitHub. Use for standard CI/CD tasks, cost-effective for small-medium projects.
  - **Self-hosted runners:** Your own infrastructure. Use for specific hardware requirements, security compliance, cost optimization for large-scale operations, or access to internal resources.

**Q3: How do you handle secrets and environment variables securely in GitHub Actions?**
- **Answer:** Use GitHub Secrets for sensitive data (API keys, passwords), Environment variables for non-sensitive config. Access via `${{ secrets.SECRET_NAME }}` and `${{ env.VAR_NAME }}`. Implement environment protection rules, use OIDC for cloud authentication, and never log secrets.

### **Advanced Configuration**

**Q4: Explain the difference between `needs`, `if`, and `strategy.matrix` in GitHub Actions workflows.**
- **Answer:**
  - **needs:** Creates job dependencies and controls execution order
  - **if:** Conditional execution based on expressions, job results, or context
  - **strategy.matrix:** Runs jobs in parallel with different variable combinations for testing across multiple environments

**Q5: How would you implement a deployment pipeline with different environments (dev, staging, prod) in GitHub Actions?**
- **Answer:** Use environment protection rules, separate workflows or jobs with conditional logic, implement approval processes for production, use different secrets per environment, and implement proper artifact promotion between stages.

**Q6: Describe how you would optimize a slow GitHub Actions workflow.**
- **Answer:** 
  - Use caching for dependencies (`actions/cache`)
  - Parallelize jobs and steps
  - Use self-hosted runners for better performance
  - Optimize Docker builds with multi-stage builds and layer caching
  - Skip unnecessary steps with conditional logic
  - Use artifacts efficiently

### **Real-world Scenarios**

**Q7: How do you handle monorepo CI/CD with GitHub Actions where you only want to build/deploy affected services?**
- **Answer:** Use path-based filtering with `paths` or `paths-ignore`, implement changed file detection using `git diff` or actions like `dorny/paths-filter`, create dynamic job matrices based on changes, and use reusable workflows for common tasks.

**Q8: Describe how you would implement blue-green deployment using GitHub Actions.**
- **Answer:** Create separate deployment slots/environments, deploy to inactive environment, run health checks and tests, switch traffic using load balancer or DNS, monitor for issues, and implement automatic rollback mechanisms.

---

## **Jenkins Questions**

### **Architecture and Setup**

**Q9: Explain Jenkins architecture and the role of master/controller and agent nodes.**
- **Answer:** Jenkins follows master-agent architecture where the controller manages the build queue, schedules jobs, and serves the web UI, while agents execute the actual build jobs. Agents can be static (permanent) or dynamic (cloud-based), communicating via JNLP or SSH.

**Q10: How do you implement Jenkins as Code using Pipeline as Code and Configuration as Code?**
- **Answer:** 
  - **Pipeline as Code:** Use Jenkinsfile (declarative or scripted) stored in SCM
  - **Configuration as Code (JCasC):** Use YAML files to configure Jenkins system settings, plugins, and global configurations
  - Implement Infrastructure as Code for Jenkins infrastructure

**Q11: What are the differences between Declarative and Scripted pipelines in Jenkins?**
- **Answer:**
  - **Declarative:** Structured, YAML-like syntax, easier to read/write, built-in error handling, limited flexibility
  - **Scripted:** Groovy-based, more flexible, programmatic approach, steeper learning curve, full Groovy capabilities

### **Advanced Pipeline Concepts**

**Q12: How do you implement parallel execution and pipeline orchestration in Jenkins?**
- **Answer:** Use `parallel` blocks for concurrent execution, implement proper stage dependencies, use `lock` resources for critical sections, implement fan-out/fan-in patterns, and use milestone steps for complex orchestration.

**Q13: Explain how you would implement a multi-branch pipeline with different deployment strategies per branch.**
- **Answer:** Configure multibranch pipeline, use branch-specific logic in Jenkinsfile (`when` conditions), implement different deployment targets per branch type (feature → dev, develop → staging, master → prod), and use shared libraries for common functionality.

**Q14: How do you handle credentials and secrets management in Jenkins?**
- **Answer:** Use Jenkins Credentials Store, implement credential binding in pipelines, use external secret management (HashiCorp Vault, AWS Secrets Manager), implement least privilege access, rotate credentials regularly, and audit credential usage.

### **Scaling and Performance**

**Q15: How would you design a scalable Jenkins infrastructure for a large organization?**
- **Answer:** Implement Jenkins clustering, use cloud-based dynamic agents, implement build agent pools based on requirements, use Jenkins Configuration as Code, implement proper monitoring and alerting, use shared libraries, and implement federated Jenkins setup if needed.

**Q16: Describe strategies for optimizing Jenkins build performance.**
- **Answer:** 
  - Use incremental builds and build caching
  - Implement parallel agent execution
  - Optimize Docker builds and use build cache
  - Use appropriate agent sizing and types
  - Implement build artifact caching
  - Use pipeline optimization techniques

---

## **Common CI/CD Questions (Both Platforms)**

### **Best Practices and Strategy**

**Q17: Compare GitHub Actions vs Jenkins. When would you choose one over the other?**
- **Answer:**
  - **GitHub Actions:** Native GitHub integration, easier setup, good for GitHub-centric workflows, limited customization
  - **Jenkins:** More mature, highly customizable, extensive plugin ecosystem, better for complex enterprise needs, self-hosted flexibility

**Q18: How do you implement effective branching strategies with CI/CD?**
- **Answer:** Implement GitFlow or GitHub Flow, configure branch protection rules, implement different pipeline stages per branch type, use feature flags for deployment decoupling, and implement proper merge strategies.

**Q19: Describe how you implement database migrations in your CI/CD pipeline.**
- **Answer:** Use migration tools (Flyway, Liquibase), implement rollback strategies, test migrations in staging environments, use blue-green or canary deployments for zero-downtime, implement migration versioning, and separate schema changes from application deployments when needed.

### **Security and Compliance**

**Q20: How do you implement security scanning and compliance checks in your CI/CD pipeline?**
- **Answer:** Integrate SAST/DAST tools, implement dependency vulnerability scanning, use container image scanning, implement compliance as code, generate security reports, implement security gates, and maintain audit trails.

**Q21: Describe your approach to secret rotation in CI/CD pipelines.**
- **Answer:** Use external secret management systems, implement automated secret rotation, use short-lived tokens when possible, implement secret scanning in code, use OIDC for cloud authentication, and maintain secret usage audit logs.

### **Monitoring and Troubleshooting**

**Q22: How do you monitor and troubleshoot CI/CD pipeline failures?**
- **Answer:** Implement comprehensive logging, use monitoring tools for pipeline metrics, set up alerting for failures, implement pipeline observability, maintain runbooks for common issues, use distributed tracing for complex pipelines, and implement proper error handling and retries.

**Q23: Describe how you implement rollback strategies in your deployment pipeline.**
- **Answer:** Implement automated rollback triggers based on health checks, maintain deployment artifacts for rollback, use blue-green or canary deployment patterns, implement database rollback strategies, maintain rollback runbooks, and test rollback procedures regularly.

---

## **Scenario-Based Questions**

**Q24: You have a microservices application with 20+ services. How would you design the CI/CD pipeline?**
- **Answer:** Implement service-specific pipelines, use monorepo with path-based triggering or multi-repo approach, implement service dependency management, use container orchestration, implement service mesh for deployment strategies, use artifact promotion, and implement end-to-end testing strategies.

**Q25: How would you handle a situation where builds are failing intermittently due to flaky tests?**
- **Answer:** Implement test categorization (unit, integration, e2e), use test retries for flaky tests, implement test quarantine, improve test environment stability, implement better test data management, use test parallelization, and implement test result analytics.

**Q26: Describe how you would migrate from Jenkins to GitHub Actions (or vice versa).**
- **Answer:** Analyze current pipeline complexity, create migration plan and timeline, implement parallel running during transition, migrate in phases (simple pipelines first), update team processes and training, implement pipeline conversion tools where applicable, and maintain rollback capabilities.

---

## **Technical Deep Dive Questions**

**Q27: Explain how you would implement cross-platform builds (Windows, Linux, macOS) in both GitHub Actions and Jenkins.**

**Q28: How do you handle artifact management and promotion across different environments?**

**Q29: Describe your approach to implementing canary deployments with automated rollback.**

**Q30: How would you implement compliance reporting and audit trails for your CI/CD processes?**

---

