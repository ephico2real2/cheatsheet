You're right. Using a single Secret resource with multiple keys is a more efficient approach for Kubernetes. Let's create a single secrets YAML file to store both MySQL passwords and modify the deployment to reference them correctly.

Here's the single secrets YAML file:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: insights-mysql-secrets
type: Opaque
data:
  root-password: # Base64 encoded root password goes here
  user-password: # Base64 encoded user password goes here
```

And here's the corrected MySQL deployment YAML that references this single secret:

```yaml
matchLabels:
  app: insights
  tier: mysql
strategy:
  type: Recreate
template:
  metadata:
    creationTimestamp: null
    labels:
      app: insights
      app-group: insights
      tier: mysql
  spec:
    containers:
    - command:
      - /bin/sh
      - /usr/local/bin/run.sh
      env:
      - name: MYSQL_ROOT_PASSWORD
        valueFrom:
          secretKeyRef:
            key: root-password
            name: insights-mysql-secrets
      - name: MYSQL_DATABASE
        value: insights
      - name: MYSQL_USER
        value: insights
      - name: MYSQL_PASSWORD
        valueFrom:
          secretKeyRef:
            key: user-password
            name: insights-mysql-secrets
      image: jamf/insights-mysql:8.4
      imagePullPolicy: IfNotPresent
      name: mysql-insights
      ports:
      - containerPort: 3306
        name: mysql
        protocol: TCP
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
      # Add your volume mounts here
```

This approach:
1. Uses a single Secret resource named `insights-mysql-secrets`
2. Contains two keys: `root-password` and `user-password`
3. Both MySQL environment variables reference the same secret resource but different keys

To use this:
1. Generate base64 encoded values for your passwords:
   ```
   echo -n "your-root-password" | base64
   echo -n "your-user-password" | base64
   ```
2. Add the encoded values to the secrets YAML
3. Apply the secrets YAML first: `kubectl apply -f mysql-secrets.yaml`
4. Then apply your MySQL deployment YAML: `kubectl apply -f mysql-deployment.yaml`

This approach consolidates all database credentials into a single Secret resource, making it easier to manage.


Based on both images you've shared, I'll now create the complete YAML file for the MySQL deployment, including the volume mounts and other configuration details. I'll also include the script to generate the associated secrets.

Here's the complete MySQL deployment YAML:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-insights
  labels:
    app: insights
    tier: mysql
spec:
  selector:
    matchLabels:
      app: insights
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: insights
        app-group: insights
        tier: mysql
    spec:
      containers:
      - command:
        - /bin/sh
        - /usr/local/bin/run.sh
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              key: root-password
              name: insights-mysql-secrets
        - name: MYSQL_DATABASE
          value: insights
        - name: MYSQL_USER
          value: insights
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              key: user-password
              name: insights-mysql-secrets
        image: jamf/insights-mysql:8.4
        imagePullPolicy: IfNotPresent
        name: mysql-insights
        ports:
        - containerPort: 3306
          name: mysql
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /bitnami/mysql/data
          name: insights-mysql
      # These lines are commented out in your YAML
      # imagePullSecrets:
      # - name: jamf
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - name: insights-mysql
        persistentVolumeClaim:
          claimName: insights-mysql-data
  status: {}
```

Here's the script to generate the Secret YAML from a text file:

```bash
#!/bin/bash

# Set the name of your secret
SECRET_NAME="insights-mysql-secrets"

# Start the YAML file
cat > ${SECRET_NAME}.yaml << EOF
apiVersion: v1
kind: Secret
metadata:
  name: ${SECRET_NAME}
type: Opaque
data:
EOF

# Read the text file line by line
while IFS='=' read -r key value; do
  # Skip empty lines or lines starting with #
  [[ -z $key || $key == \#* ]] && continue
  
  # Base64 encode the value and add it to the YAML
  echo "  $key: $(echo -n "$value" | base64)" >> ${SECRET_NAME}.yaml
done < insights-mysql-secrets.txt

echo "Secret YAML created: ${SECRET_NAME}.yaml"
```

And here's the PersistentVolumeClaim YAML that would be needed to support your deployment:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: insights-mysql-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi  # Adjust size as needed
```

To use this setup:

1. Create a text file named `insights-mysql-secrets.txt` with:
```
root-password=your-root-password-here
user-password=your-user-password-here
```

2. Run the script to generate the Secret YAML
3. Apply the resources in this order:
   - PersistentVolumeClaim
   - Secret
   - Deployment

This will create a properly configured MySQL deployment with persistent storage and secure password management.

