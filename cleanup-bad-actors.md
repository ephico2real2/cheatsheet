Sure, here are the high-level steps for running this script as a Kubernetes CronJob:

1. **Write and test the script locally**:
    - Develop and fine-tune your script locally to ensure it works as expected.
    - This would include running it in dry-run mode and setting the various environment variables to different values.

2. **Create a ConfigMap for the script**:
    - Create a ConfigMap that contains the script. This ConfigMap will be mounted as a volume in the CronJob's pod, allowing the script to be executed.
    - You can create the ConfigMap using `oc create configmap cleanup-script --from-file=cleanup.sh=/path/to/your/script.sh`.

3. **Create a ConfigMap for the environment variables**:
    - Create another ConfigMap that contains the environment variables for the script.
    - You can use `oc create configmap cleanup-script-env --from-literal=LOG_FILE=/path/to/log --from-literal=DRY_RUN=true` and so on for each environment variable.

4. **Create a ServiceAccount**:
    - The ServiceAccount will provide identity for the CronJob's pod.
    - You can create a ServiceAccount using `oc create serviceaccount cleanup`.

5. **Create a ClusterRole**:
    - This ClusterRole will provide the necessary permissions for the script to interact with the Kubernetes API.
    - Use `oc create clusterrole cleanup --verb=watch,delete,patch,update,get,list --resource=pods,cronjobs,deployments,deploymentconfigs`.

6. **Create a ClusterRoleBinding**:
    - The ClusterRoleBinding will bind the ClusterRole to the ServiceAccount, granting the ServiceAccount the necessary permissions.
    - Use `oc create clusterrolebinding cleanup --clusterrole=cleanup --serviceaccount=default:cleanup`.

7. **Create the CronJob**:
    - The CronJob will run your script on a schedule.
    - You can define the CronJob in a YAML file, and include the ConfigMaps and ServiceAccount in the pod spec.
    - The `spec.jobTemplate.spec.template.spec.containers` section would have something like: 
      ```yaml
      volumeMounts:
      - name: cleanup-script
        mountPath: /scripts
      envFrom:
      - configMapRef:
          name: cleanup-script-env
      ```
    - The `spec.jobTemplate.spec.template.spec.volumes` section would have something like:
      ```yaml
      volumes:
      - name: cleanup-script
        configMap:
          name: cleanup-script
      ```
    - The `spec.jobTemplate.spec.template.spec.serviceAccountName` would be `cleanup`.

8. **Apply the CronJob**:
    - Use `oc apply -f /path/to/your/cronjob.yaml` to create the CronJob in your cluster.

9. **Monitor the CronJob**:
    - You can use `oc get cronjobs.batch` and `oc get pods` to check the status of the CronJob and its pods.
    - The logs for each pod can be viewed with `oc logs pod/<pod-name>`.

Remember to replace `oc` with `kubectl` if you are not using OpenShift but a plain Kubernetes setup. Please also note that the actual paths, names, and values will depend on your specific setup and requirements. Always test everything in a non-production environment first.
