'''yaml

kubectl get clusterrolebindings -o=json | jq -r '
  .items[]
  | select(
      .subjects // [] | .[]
      | select(.kind == "ServiceAccount" and .name == "YOUR_SERVICE_ACCOUNT" and .namespace == "YOUR_NAMESPACE")
    )
  | .roleRef.name
'

'''
