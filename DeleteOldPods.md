

oc get pods --all-namespaces -o=jsonpath='{range .items[?(@.status.phase!="Succeeded")]}{.metadata.name}{"\t"}{.metadata.namespace}{"\t"}{.metadata.creationTimestamp}{"\n"}{end}' | while read -r pod namespace date; do if [[ $(date -d "$date" +%s) -lt $(date -d 'now - 4 hours' +%s) ]]; then oc -n "$namespace" delete pod "$pod"; fi; done
