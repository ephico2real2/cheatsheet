# Transitioning to External Jaeger HTTP-Zipkin Collector URLs

## Overview
This document outlines the process of transitioning Spring Boot services from using internal Kubernetes references to using fully qualified URLs for the Jaeger HTTP-Zipkin collector endpoints. The change involves updating environment-specific application.yaml files in deployment repositories to override the default embedded configuration.

## Configuration Structure

Each environment has its dedicated configuration folder in the deployment repository, following this structure:
```
src/micron/mortgage-account-maintenance/config/{environment}/cce-experience-{environment}/application.yaml
```

Where `{environment}` represents various deployment targets:
- prod-gf1
- prod-gf2
- qa
- rnd
- uat

## The Transition

Spring Boot applications come with an embedded default application.yaml. The goal is to override these defaults with environment-specific configurations in our deployment repository.

### Previous Configuration (Internal Kubernetes References)
Previously, the overridden configurations referenced the Zipkin collector using internal Kubernetes service names:

```yaml
spring:
  zipkin:
    base-url: http://zipkin-collector.observability.svc.cluster.local

management:
  zipkin:
    tracing:
      endpoint: http://zipkin-collector.observability.svc.cluster.local:9411/api/v2/spans
```

### New Configuration (External Fully Qualified URLs)
Now, the environment-specific overrides should use fully qualified external URLs for the Zipkin collector:

```yaml
spring:
  zipkin:
    base-url: https://http-zipkin-collector-observability.apps.ocp4-{environment}.pncint.net

management:
  zipkin:
    tracing:
      endpoint: https://http-zipkin-collector-observability.apps.ocp4-{environment}.pncint.net/api/v2/spans
```

## Implementation Approach

### Step 1: Clone the Repository
```bash
# Clone the repository if you haven't already
git clone git@git.pncint.net:projects/CCE/repos/cce-ms-mortgage-account-maintenance-outer-api-deploy.git
cd cce-ms-mortgage-account-maintenance-outer-api-deploy

# Create a new branch for your changes
git checkout -b update-zipkin-collector-urls
```

### Step 2: Manually Update Configuration Files

For each environment, manually update the configuration file with the specific stanza. Copy and paste the appropriate configuration into each environment's application.yaml file:

#### For prod-gf1 environment
File path: `src/micron/mortgage-account-maintenance/config/prod-gf1/cce-experience-prod/application.yaml`

Add or update the following stanza:
```yaml
spring:
  zipkin:
    base-url: https://http-zipkin-collector-observability.apps.ocp4-gf1.pncint.net

management:
  zipkin:
    tracing:
      endpoint: https://http-zipkin-collector-observability.apps.ocp4-gf1.pncint.net/api/v2/spans
```

#### For prod-gf2 environment
File path: `src/micron/mortgage-account-maintenance/config/prod-gf2/cce-experience-prod/application.yaml`

Add or update the following stanza:
```yaml
spring:
  zipkin:
    base-url: https://http-zipkin-collector-observability.apps.ocp4-gf2.pncint.net

management:
  zipkin:
    tracing:
      endpoint: https://http-zipkin-collector-observability.apps.ocp4-gf2.pncint.net/api/v2/spans
```

#### For qa environment
File path: `src/micron/mortgage-account-maintenance/config/qa/cce-experience-qa/application.yaml`

Add or update the following stanza:
```yaml
spring:
  zipkin:
    base-url: https://http-zipkin-collector-observability.apps.ocp4-qa.pncint.net

management:
  zipkin:
    tracing:
      endpoint: https://http-zipkin-collector-observability.apps.ocp4-qa.pncint.net/api/v2/spans
```

#### For rnd environment
File path: `src/micron/mortgage-account-maintenance/config/rnd/cce-experience-rnd/application.yaml`

Add or update the following stanza:
```yaml
spring:
  zipkin:
    base-url: https://http-zipkin-collector-observability.apps.ocp4-rnd.pncint.net

management:
  zipkin:
    tracing:
      endpoint: https://http-zipkin-collector-observability.apps.ocp4-rnd.pncint.net/api/v2/spans
```

#### For uat environment
File path: `src/micron/mortgage-account-maintenance/config/uat/cce-experience-uat/application.yaml`

Add or update the following stanza:
```yaml
spring:
  zipkin:
    base-url: https://http-zipkin-collector-observability.apps.ocp4-uat.pncint.net

management:
  zipkin:
    tracing:
      endpoint: https://http-zipkin-collector-observability.apps.ocp4-uat.pncint.net/api/v2/spans
```

### Step 3: Verify Your Changes
```bash
# Check all modified files
git diff --name-only

# Review the actual changes
git diff
```

### Step 4: Commit and Push Your Changes
```bash
# Add all modified files
git add .

# Commit your changes with a descriptive message
git commit -m "Update Zipkin collector URLs to use fully qualified external URLs"

# Push your changes
git push origin update-zipkin-collector-urls
```

### Step 5: Create a Pull Request
Create a pull request in Bitbucket to merge your changes into the main branch.

## Environment URL Patterns

| Environment | Base URL Pattern |
|-------------|------------------|
| prod-gf1 | https://http-zipkin-collector-observability.apps.ocp4-gf1.pncint.net |
| prod-gf2 | https://http-zipkin-collector-observability.apps.ocp4-gf2.pncint.net |
| qa | https://http-zipkin-collector-observability.apps.ocp4-qa.pncint.net |
| rnd | https://http-zipkin-collector-observability.apps.ocp4-rnd.pncint.net |
| uat | https://http-zipkin-collector-observability.apps.ocp4-uat.pncint.net |

## Benefits of the Transition

1. **Enhanced Reliability**: No dependency on internal DNS resolution which can be affected by namespace or cluster issues
2. **Cross-Cluster Compatibility**: Works regardless of which OpenShift cluster the service is deployed to
3. **Standardization**: Consistent URL pattern across all environments
4. **Improved Troubleshooting**: Easier to verify connectivity using fully qualified URLs
5. **Future-Proofing**: Better positioned for potential future infrastructure changes

## Verification

After deploying the updated configurations, verify proper trace collection by:

1. Generate some traffic to the service
2. Check the Jaeger UI for the corresponding environment:
   ```
   https://jaeger-streaming-observability.apps.ocp4-{environment}.pncint.net
   ```
3. Ensure traces are being collected properly

## Migration Status

| Environment | Status | Migration Date | Implemented By |
|-------------|--------|----------------|---------------|
| prod-gf1 | Completed | YYYY-MM-DD | |
| prod-gf2 | Completed | YYYY-MM-DD | |
| qa | Completed | YYYY-MM-DD | |
| rnd | Completed | YYYY-MM-DD | |
| uat | Completed | YYYY-MM-DD | |

## Troubleshooting

### Issue: No traces appearing in Jaeger

**Potential Solution:**
```bash
# Check if the service can reach the Zipkin collector
curl -v https://http-zipkin-collector-observability.apps.ocp4-gf1.pncint.net/api/v2/spans
```

### Issue: Environment-specific configuration not being applied

**Potential Solution:**
```bash
# Verify ConfigMap contains the correct overridden application.yaml
kubectl get configmap -n your-namespace your-configmap-name -o yaml

# Check if the Spring profile is set correctly for the environment
kubectl describe deployment your-deployment-name -n your-namespace | grep SPRING_PROFILES_ACTIVE

# Verify the Spring Boot application is properly loading external configuration
kubectl logs -n your-namespace $(kubectl get pods -n your-namespace -l app=your-app-name -o name | head -1)
```

## Additional Notes

* This change affects only the environment-specific application.yaml files in the deployment repository that override Spring Boot's default configuration
* The application's core code and embedded default application.yaml remain unchanged
* These overrides are applied through ConfigMaps in Kubernetes at deployment time
* This migration is part of a broader initiative to standardize observability infrastructure access across environments

**Important:** Remember to follow the standard change management process when implementing these changes in production environments.

## Contact

For questions or assistance with this transition, contact:

* Team Channel: #platform-observability
* Email: platform-observability@pncint.net
* On-call Support: xxx-xxx-xxxx

---

**Document Information**
* Created: YYYY-MM-DD
* Last Updated: YYYY-MM-DD
* Version: 1.0
