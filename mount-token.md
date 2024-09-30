```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      serviceAccountName: my-service-account  # Specify your service account here
      containers:
      - name: my-container
        image: my-image:tag
        command: ["/bin/sh"]
        args:
        - -c
        - |
          TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
          export SERVICE_ACCOUNT_TOKEN=$TOKEN
          exec your_actual_command
        volumeMounts:
        - name: service-account-token
          mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          readOnly: true
      volumes:
      - name: service-account-token
        projected:
          sources:
          - serviceAccountToken:
              expirationSeconds: 3600
              audience: api  # This is optional and can be customized


```
