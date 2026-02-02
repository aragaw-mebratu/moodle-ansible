# Moodle Ansible Deployment

This directory contains Ansible playbooks for deploying and managing Moodle with the working configuration that resolves database connection issues.

## Playbooks Overview

### 1. `deploy-moodle-complete.yml` (Main Deployment)
Complete end-to-end deployment that:
- Deploys Moodle with working configuration
- Validates the deployment
- Displays success information

### 2. `deploy-moodle-working-config.yml` (Full Deployment)
Full Moodle deployment playbook that:
- Creates necessary namespaces and secrets
- Deploys Moodle using Helm with optimized values
- Applies working configuration (direct DB connection + corrected Redis)
- Patches deployment with proper environment variables
- Waits for pods to be ready

### 3. `update-moodle-config.yml` (Configuration Update)
Updates existing Moodle deployment with working configuration:
- Updates database secrets with correct keys
- Creates working configmap with direct database connection
- Updates deployment to use the new configuration
- Waits for rollout completion

### 4. `validate-moodle-deployment.yml` (Validation)
Validates that Moodle deployment is working correctly:
- Checks namespace, deployment, and pod status
- Verifies service and secret presence
- Counts ready pods
- Displays comprehensive status information

## Key Configuration Fixes

The working configuration includes these critical fixes:

1. **Direct Database Connection**: Uses `moodle-db-cluster.database.svc.cluster.local` instead of the connection pooler
2. **Corrected Redis Password**: Uses "dragonfly" instead of "cbt@u0g123!!"
3. **Hardcoded Database Credentials**: Eliminates environment variable resolution issues
4. **Optimized Performance Settings**: Includes proper database and Redis configuration

## Usage

### Prerequisites
- Ansible with kubernetes.core collection installed
- kubectl configured with appropriate permissions
- Helm repository access to bitnami/moodle chart
- Existing PostgreSQL database and Redis instances

### Running the Deployment

```bash
# Run complete deployment (recommended)
ansible-playbook deploy-moodle-complete.yml

# Run individual playbooks
ansible-playbook deploy-moodle-working-config.yml
ansible-playbook update-moodle-config.yml
ansible-playbook validate-moodle-deployment.yml
```

### Variables

The playbooks use the following key variables (can be overridden):

```yaml
moodle_namespace: "moodle"
db_host: "moodle-db-cluster.database.svc.cluster.local"
db_name: "moodledb"
db_user: "moodleuser"
db_password: "PQl5YR30j9geR9NrgLTPxzTBkNZ2OYIDDiLaDZm946Nk7O1mWmPsArY5C7yj9vcw"
redis_host: "dragonfly-redis-master.dragonfly-system.svc.cluster.local"
redis_password: "dragonfly"
moodle_admin_user: "admin"
moodle_admin_password: "admin123"
```

## Expected Output

After successful deployment, you should see:
- ✅ All pods in Running state (1/1 READY)
- ✅ HTTP 200 responses from Moodle endpoints
- ✅ Working database and Redis connections
- ✅ Accessible login page

## Troubleshooting

If deployment fails:
1. Run `validate-moodle-deployment.yml` to check current status
2. Check pod logs: `kubectl logs -n moodle -l app.kubernetes.io/name=moodle`
3. Verify database connectivity: `kubectl exec -n database moodle-db-cluster-0 -- psql -U moodleuser -d moodledb -c "SELECT 1;"`
4. Check Redis connectivity: Test from within a Moodle pod using the correct password

## Access Information

After successful deployment:
- **URL**: `http://<NODE_IP>:31203/login/index.php`
- **Admin Username**: `admin`
- **Admin Password**: `admin123`
- **Database**: PostgreSQL with direct connection
- **Cache**: Redis (Dragonfly)