The script will show the individual CPU counts, the total CPU cores, and the count of worker nodes that have both worker and app labels:

```bash
#!/bin/bash
echo "===== Worker Node CPU Resources (worker + app nodes only) ====="
total_cpu=0
node_count=0

# Get nodes with both worker and app roles
worker_app_nodes=$(oc get nodes -l node-role.kubernetes.io/worker,node-role.kubernetes.io/app -o name)

# Iterate through each worker node
for node in $worker_app_nodes; do
  node_name=$(echo $node | cut -d '/' -f 2)
  cpu_count=$(oc get node $node_name -o jsonpath='{.status.capacity.cpu}')
  echo "Node: $node_name - CPU cores: $cpu_count"
  total_cpu=$((total_cpu + cpu_count))
  node_count=$((node_count + 1))
done

echo "============================="
echo "Total worker nodes (with worker+app labels): $node_count"
echo "Total CPU cores across worker+app nodes: $total_cpu"
```

Alternatively, you can use these one-liners:

```bash
# Get count of worker+app nodes
oc get nodes -l node-role.kubernetes.io/worker,node-role.kubernetes.io/app --no-headers | wc -l

# Get CPU per node and total in one command
oc get nodes -l node-role.kubernetes.io/worker,node-role.kubernetes.io/app -o custom-columns=NAME:.metadata.name,CPU:.status.capacity.cpu --no-headers | tee >(awk '{sum+=$2; count+=1} END {print "Total nodes: " count; print "Total CPU cores: " sum}')
```
