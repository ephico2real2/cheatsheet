#!/bin/bash
 
### Pattern =>  EXCLUDE_PATTERN="^openshift.*|^kube.*|^logscale.*"  (If you intend to exclude namespaces that specifically start with those prefixes, you can modify the pattern accordingly:)
### EXCLUDE_PATTERN="^test-.*|.*-dev$" (Exclude namespaces that either start with "test-" or end with "-dev":)


# Configurations
ROLEBINDING_NAME="your-rolebinding-name"
TEMPLATE_NAME="your-template-name"
TEMPLATE_PARAMETERS="--param=KEY=VALUE"  # If NAMESPACE is a parameter, specify it here.
EXCLUDE_PATTERN="exclude-pattern-*"   
LOG_FILE="script_log.txt"

# Function to log messages with timestamps
log() {
    echo "$(date +'%Y-%m-%d %H:%M:%S') - $1" | tee -a $LOG_FILE
}

# Function to process template and apply it
apply_template() {
    local namespace=$1
    # Processing the template with the namespace as a parameter
    oc process $TEMPLATE_NAME $TEMPLATE_PARAMETERS --param=NAMESPACE=$namespace --local=true | oc apply -f -
    if [ $? -ne 0 ]; then
        log "ERROR: Failed to apply template in namespace $namespace"
    fi
}

# Main Script Execution
NAMESPACES=$(oc get namespace -o=jsonpath='{.items[*].metadata.name}' | tr ' ' '\n' | grep -vE "$EXCLUDE_PATTERN")

for NAMESPACE in $NAMESPACES; do
    if ! oc get rolebinding $ROLEBINDING_NAME -n $NAMESPACE &>/dev/null; then
        log "RoleBinding $ROLEBINDING_NAME does not exist in namespace $NAMESPACE. Processing the template..."
        apply_template $NAMESPACE
    else
        log "RoleBinding $ROLEBINDING_NAME exists in namespace $NAMESPACE. Skipping template processing..."
    fi
done
