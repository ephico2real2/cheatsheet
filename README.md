# cheatsheet

`` export HPA into CSV ``


```bash

oc get hpa --all-namespaces -o custom-columns="NAMESPACE:.metadata.namespace,HPA:.metadata.name,MIN:.spec.minReplicas,MAX:.spec.maxReplicas" --no-headers | awk 'BEGIN {OFS=","} {print $1,$2,$3,$4}' > hpa_list.csv


```


```

The oc command used to get the list of Horizontal Pod Autoscalers (HPAs), along with their minimum and maximum replica counts in all namespaces, and output the information in CSV format:

oc get hpa --all-namespaces: List all HPAs in all namespaces using the oc command. The oc command is the OpenShift CLI tool, which is compatible with OpenShift 4.x and can be used as a drop-in replacement for kubectl.

-o custom-columns="NAMESPACE:.metadata.namespace,HPA:.metadata.name,MIN:.spec.minReplicas,MAX:.spec.maxReplicas": Use the custom columns output option to specify the columns we want to display. In this case, we're selecting the namespace, HPA name, minimum replicas, and maximum replicas.

--no-headers: Remove the header row from the output. This makes it easier to process the output in the following steps.

| awk 'BEGIN {OFS=","} {print $1,$2,$3,$4}': Pipe the output to the awk command, which processes the output and formats it as CSV. The BEGIN {OFS=","} block sets the output field separator (OFS) to a comma, and {print $1,$2,$3,$4} prints the four columns separated by commas.

> hpa_list.csv: Redirect the output to a file named "hpa_list.csv". This saves the CSV-formatted output to a file.

After running this command, you'll have a file named "hpa_list.csv" with the desired information about HPAs, their namespaces, and their minimum and maximum replica counts in CSV format.


```
