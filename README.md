# cheatsheet

`` One-liner command to export HPA into CSV ``


```bash

echo "NAMESPACE,HPA,MIN,MAX" > hpa_list.csv && oc get hpa --all-namespaces -o \
custom-columns="NAMESPACE:.metadata.namespace,HPA:.metadata.name,MIN:.spec.minReplicas,MAX:.spec.maxReplicas" \
--no-headers | awk 'BEGIN {OFS=","} {print $1,$2,$3,$4}' >> hpa_list.csv


```

```

The oc command used to get the list of Horizontal Pod Autoscalers (HPAs), along with their minimum and maximum replica counts in all namespaces, and output the information in CSV format:

echo "NAMESPACE,HPA,MIN,MAX" > adding headers

oc get hpa --all-namespaces: List all HPAs in all namespaces using the oc command. The oc command is the OpenShift CLI tool, which is compatible with OpenShift 4.x and can be used as a drop-in replacement for kubectl.

-o custom-columns="NAMESPACE:.metadata.namespace,HPA:.metadata.name,MIN:.spec.minReplicas,MAX:.spec.maxReplicas": Use the custom columns output option to specify the columns we want to display. In this case, we're selecting the namespace, HPA name, minimum replicas, and maximum replicas.

--no-headers: Remove the header row from the output. This makes it easier to process the output in the following steps.

| awk 'BEGIN {OFS=","} {print $1,$2,$3,$4}': Pipe the output to the awk command, which processes the output and formats it as CSV. The BEGIN {OFS=","} block sets the output field separator (OFS) to a comma, and {print $1,$2,$3,$4} prints the four columns separated by commas.

> hpa_list.csv: Redirect the output to a file named "hpa_list.csv". This saves the CSV-formatted output to a file.

After running this command, you'll have a file named "hpa_list.csv" with the desired information about HPAs, their namespaces, and their minimum and maximum replica counts in CSV format.


```


`` one liner to get pvc name ``

```

oc get deployments -A -o json | jq -r '.items[] | select(.spec.template.spec.volumes[]? | .persistentVolumeClaim.claimName=="<PVC_NAME>") | .metadata.namespace + "/" + .metadata.name + " -> " + "<PVC_NAME>"'

oc get deploymentconfigs -A -o json | jq -r '.items[] | select(.spec.template.spec.volumes[]? | .persistentVolumeClaim.claimName=="<PVC_NAME>") | .metadata.namespace + "/" + .metadata.name + " -> " + "<PVC_NAME>"'


```


```bash

#!/bin/bash

PVC_NAME="$1"

if [ -z "$PVC_NAME" ]; then
  echo "Usage: $0 <PVC_NAME>"
  exit 1
fi

echo "Searching for resources using PVC: $PVC_NAME"
echo "---------------------------------------------"

DEPLOYMENTS=$(oc get deployments -A -o json | jq -r --arg PVC_NAME "$PVC_NAME" '.items[] | select(.spec.template.spec.volumes[]? | .persistentVolumeClaim.claimName==$PVC_NAME) | "Namespace: " + .metadata.namespace + "\nDeployment: " + .metadata.name + "\n"')
DEPLOYMENTCONFIGS=$(oc get deploymentconfigs -A -o json | jq -r --arg PVC_NAME "$PVC_NAME" '.items[] | select(.spec.template.spec.volumes[]? | .persistentVolumeClaim.claimName==$PVC_NAME) | "Namespace: " + .metadata.namespace + "\nDeploymentConfig: " + .metadata.name + "\n"')

if [ -n "$DEPLOYMENTS" ]; then
  echo "Deployments:"
  echo "-------------"
  echo -e "$DEPLOYMENTS"
fi

if [ -n "$DEPLOYMENTCONFIGS" ]; then
  echo "DeploymentConfigs:"
  echo "-------------------"
  echo -e "$DEPLOYMENTCONFIGS"
fi

if [ -z "$DEPLOYMENTS" ] && [ -z "$DEPLOYMENTCONFIGS" ]; then
  echo "No Deployments or DeploymentConfigs found using the specified PVC."
fi


```

`` chmod +x find_pvc.sh ``

`` ./find_pvc.sh <PVC_NAME> ``


`` on liner to get events ``


```bash
cat << EOF> get_all_events.sh
#!/bin/bash

# Get a list of all namespaces
namespaces=$(oc get ns -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}')

# Output file
output_file="events_count.csv"

# Write the header to the CSV file
echo "NAMESPACES,EVENTS COUNT" > $output_file

# Loop through namespaces and count the events
for ns in $namespaces
do
    count=$(oc get events -n $ns --no-headers | wc -l)
    echo "$ns,$count" >> $output_file
done
EOF


```
