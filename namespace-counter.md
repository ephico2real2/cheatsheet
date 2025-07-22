## working 

```bash
#!/bin/bash

# OpenShift Multi-Cluster Namespace Counter - Shell Script Version
# Compatible with Linux systems
# Requires: oc CLI tool

# set -e removed to allow proper error handling

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Emojis (fallback to text if not supported)
if locale charmap 2>/dev/null | grep -qi utf; then
    ROCKET="🚀"
    CHECK="✅"
    CROSS="❌"
    CLOCK="⏰"
    INFO="ℹ️"
    GEAR="⚙️"
else
    ROCKET="[*]"
    CHECK="[OK]"
    CROSS="[ERROR]"
    CLOCK="[TIME]"
    INFO="[INFO]"
    GEAR="[SETUP]"
fi

# Function to print colored output
print_status() {
    echo -e "${GREEN}${CHECK}${NC} $1"
}

print_error() {
    echo -e "${RED}${CROSS}${NC} $1"
}

print_info() {
    echo -e "${BLUE}${INFO}${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}${GEAR}${NC} $1"
}

# Function to check if oc CLI is installed
check_oc_cli() {
    if ! command -v oc &> /dev/null; then
        print_error "OpenShift CLI (oc) not found!"
        echo "Please install the OpenShift CLI from:"
        echo "https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/getting-started-cli.html"
        echo ""
        print_warning "Running in DEMO MODE with sample data..."
        echo ""
        export DEMO_MODE="true"
        return 1
    fi
    
    local oc_version=$(oc version --client 2>/dev/null | head -1 || echo "Unknown version")
    print_status "OpenShift CLI found: $oc_version"
    export DEMO_MODE="false"
    return 0
}

# Function to check login status
check_login() {
    local user=$(oc whoami 2>/dev/null || echo "")
    local server=$(oc whoami --show-server 2>/dev/null || echo "")
    
    if [[ -n "$user" && -n "$server" ]]; then
        print_status "Currently logged in as: $user"
        print_info "Server: $server"
        return 0
    else
        return 1
    fi
}

# Function to login to cluster
login_to_cluster() {
    local cluster_url="$1"
    local username="$2"
    
    if [[ -z "$cluster_url" ]]; then
        echo -n "Enter OpenShift cluster URL (e.g., https://api.cluster.com:6443): "
        read cluster_url
    fi
    
    # Validate URL format
    if [[ ! "$cluster_url" =~ ^https?:// ]]; then
        cluster_url="https://$cluster_url"
    fi
    
    if [[ ! "$cluster_url" =~ :6443$ ]] && [[ ! "$cluster_url" =~ :[0-9]+$ ]]; then
        cluster_url="$cluster_url:6443"
    fi
    
    print_warning "Logging into cluster: $cluster_url"
    
    if [[ -n "$username" ]]; then
        # Username/password authentication
        echo -n "Password for $username: "
        read -s password
        echo
        
        print_info "Attempting login with username/password..."
        if oc login "$cluster_url" -u "$username" -p "$password" &>/dev/null; then
            print_status "Login successful!"
            local logged_user=$(oc whoami 2>/dev/null)
            print_info "Logged in as: $logged_user"
            return 0
        else
            print_error "Login failed!"
            return 1
        fi
    else
        # Token authentication
        echo -n "Enter your authentication token: "
        read -s token
        echo
        
        print_info "Attempting login with token..."
        if oc login "$cluster_url" --token="$token" &>/dev/null; then
            print_status "Login successful!"
            local logged_user=$(oc whoami 2>/dev/null)
            print_info "Logged in as: $logged_user"
            return 0
        else
            print_error "Login failed!"
            return 1
        fi
    fi
}

# Function to get detailed namespace information with classifications
get_namespace_info() {
    local show_details="$1"
    
    # Get namespaces in JSON format
    local namespaces_result=$(oc get namespaces -o json 2>/dev/null)
    if [[ $? -ne 0 || -z "$namespaces_result" ]]; then
        echo "❌ Failed to get namespaces - insufficient permissions or connection issue"
        return 1
    fi
    
    # Parse namespace data using oc command with jsonpath
    local total_count=$(echo "$namespaces_result" | oc get -f - --no-headers 2>/dev/null | wc -l)
    
    # Get namespace names
    local namespace_names=$(echo "$namespaces_result" | oc get -f - -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}' 2>/dev/null)
    
    # Initialize counters
    local openshift_count=0
    local system_count=0
    local customer_count=0
    
    # Lists for detailed breakdown
    local openshift_namespaces=()
    local system_namespaces=()
    local customer_namespaces=()
    
    # Categorize namespaces
    while IFS= read -r namespace; do
        if [[ -n "$namespace" ]]; then
            if [[ "$namespace" =~ ^openshift- ]]; then
                ((openshift_count++))
                openshift_namespaces+=("$namespace")
            elif [[ "$namespace" == "kube-system" || "$namespace" == "kube-public" || "$namespace" == "kube-node-lease" || "$namespace" == "default" ]]; then
                ((system_count++))
                system_namespaces+=("$namespace")
            else
                ((customer_count++))
                customer_namespaces+=("$namespace")
            fi
        fi
    done <<< "$namespace_names"
    
    local control_plane_count=$((openshift_count + system_count))
    
    # Display results
    print_status "Found $total_count namespaces"
    echo ""
    print_info "Namespace Classification:"
    echo "  📦 Total Namespaces: $total_count"
    echo "  🎛️  Control Plane: $control_plane_count ($openshift_count OpenShift + $system_count System)"
    echo "  🏢 Customer Namespaces: $customer_count"
    
    # Calculate and show percentages
    if [[ $total_count -gt 0 ]]; then
        local control_plane_pct=$((control_plane_count * 100 / total_count))
        local customer_pct=$((customer_count * 100 / total_count))
        echo ""
        echo "  📈 Distribution:"
        echo "     🎛️  Control Plane: ${control_plane_pct}%"
        echo "     🏢 Customer: ${customer_pct}%"
    fi
    
    # Show detailed breakdown if requested
    if [[ "$show_details" == "true" ]]; then
        echo ""
        print_info "Detailed Namespace Breakdown:"
        
        if [[ ${#openshift_namespaces[@]} -gt 0 ]]; then
            echo ""
            echo "🎛️  OpenShift Platform Namespaces (${#openshift_namespaces[@]}):"
            for i in "${!openshift_namespaces[@]}"; do
                if [[ $i -lt 10 ]]; then  # Show first 10
                    echo "     $((i+1)). ${openshift_namespaces[i]}"
                elif [[ $i -eq 10 ]]; then
                    echo "     ... and $((${#openshift_namespaces[@]} - 10)) more"
                    break
                fi
            done
        fi
        
        if [[ ${#system_namespaces[@]} -gt 0 ]]; then
            echo ""
            echo "🔧 System Namespaces (${#system_namespaces[@]}):"
            for namespace in "${system_namespaces[@]}"; do
                echo "     • $namespace"
            done
        fi
        
        if [[ ${#customer_namespaces[@]} -gt 0 ]]; then
            echo ""
            echo "🏢 Customer/Application Namespaces (${#customer_namespaces[@]}):"
            for i in "${!customer_namespaces[@]}"; do
                if [[ $i -lt 10 ]]; then  # Show first 10
                    echo "     $((i+1)). ${customer_namespaces[i]}"
                elif [[ $i -eq 10 ]]; then
                    echo "     ... and $((${#customer_namespaces[@]} - 10)) more"
                    break
                fi
            done
        fi
    fi
    
    # Export data for use in other functions
    export TOTAL_NAMESPACES="$total_count"
    export CONTROL_PLANE_NAMESPACES="$control_plane_count"
    export CUSTOMER_NAMESPACES="$customer_count"
    export OPENSHIFT_NAMESPACES="$openshift_count"
    export SYSTEM_NAMESPACES="$system_count"
}

# Function to get simple namespace count (for backward compatibility)
get_namespace_count() {
    if [[ -n "$TOTAL_NAMESPACES" ]]; then
        echo "$TOTAL_NAMESPACES"
    else
        local namespace_count=$(oc get namespaces --no-headers 2>/dev/null | wc -l)
        if [[ $? -eq 0 ]]; then
            echo "$namespace_count"
        else
            echo "0"
        fi
    fi
}

# Function to get node information
get_node_info() {
    local show_details="$1"
    
    if [[ "$show_details" == "true" ]]; then
        print_info "Getting detailed node information..."
        
        # Get all nodes with their roles
        local all_nodes=$(oc get nodes --no-headers -o custom-columns="NAME:.metadata.name,ROLES:.metadata.labels" 2>/dev/null)
        
        if [[ -z "$all_nodes" ]]; then
            print_warning "Could not retrieve node information - may lack permissions"
            return
        fi
        
        # Initialize arrays to store node names by type
        local master_list=()
        local infra_list=()
        local storage_list=()
        local worker_list=()
        local other_list=()
        
        # Process each node and categorize
        while IFS= read -r node_line; do
            if [[ -n "$node_line" ]]; then
                local node_name=$(echo "$node_line" | awk '{print $1}')
                local node_roles=$(echo "$node_line" | cut -d' ' -f2-)
                
                # Categorize nodes based on roles (priority: master > infra > storage > worker)
                if [[ "$node_roles" =~ node-role.kubernetes.io/master ]] || [[ "$node_roles" =~ node-role.kubernetes.io/control-plane ]]; then
                    master_list+=("$node_name")
                elif [[ "$node_roles" =~ node-role.kubernetes.io/infra ]]; then
                    infra_list+=("$node_name")
                elif [[ "$node_roles" =~ node-role.kubernetes.io/storage ]] || [[ "$node_roles" =~ cluster.ocs.openshift.io/openshift-storage ]]; then
                    storage_list+=("$node_name")
                elif [[ "$node_roles" =~ node-role.kubernetes.io/worker ]] || [[ "$node_roles" =~ "<none>" ]] || [[ -z "$(echo "$node_roles" | grep -E 'node-role.kubernetes.io/(master|control-plane|infra|storage)')" ]]; then
                    worker_list+=("$node_name")
                else
                    other_list+=("$node_name")
                fi
            fi
        done <<< "$all_nodes"
        
        # Calculate totals
        local total_nodes=$((${#master_list[@]} + ${#infra_list[@]} + ${#storage_list[@]} + ${#worker_list[@]} + ${#other_list[@]}))
        
        echo "Node Summary:"
        echo "  Total Nodes: $total_nodes"
        echo "  Master/Control Plane: ${#master_list[@]}"
        echo "  Infrastructure: ${#infra_list[@]}"
        echo "  Storage: ${#storage_list[@]}"
        echo "  Worker: ${#worker_list[@]}"
        if [[ ${#other_list[@]} -gt 0 ]]; then
            echo "  Other: ${#other_list[@]}"
        fi
        echo ""
        
        # Show nodes grouped by type
        print_info "Nodes by Category:"
        
        if [[ ${#master_list[@]} -gt 0 ]]; then
            echo "Master/Control Plane Nodes (${#master_list[@]}):"
            for node in "${master_list[@]}"; do
                echo "  - $node"
            done
            echo ""
        fi
        
        if [[ ${#infra_list[@]} -gt 0 ]]; then
            echo "Infrastructure Nodes (${#infra_list[@]}):"
            for node in "${infra_list[@]}"; do
                echo "  - $node"
            done
            echo ""
        fi
        
        if [[ ${#storage_list[@]} -gt 0 ]]; then
            echo "Storage Nodes (${#storage_list[@]}):"
            for node in "${storage_list[@]}"; do
                echo "  - $node"
            done
            echo ""
        fi
        
        if [[ ${#worker_list[@]} -gt 0 ]]; then
            echo "Worker Nodes (${#worker_list[@]}):"
            for node in "${worker_list[@]}"; do
                echo "  - $node"
            done
            echo ""
        fi
        
        if [[ ${#other_list[@]} -gt 0 ]]; then
            echo "Other Nodes (${#other_list[@]}):"
            for node in "${other_list[@]}"; do
                echo "  - $node"
            done
            echo ""
        fi
        
    else
        local total_nodes=$(oc get nodes --no-headers 2>/dev/null | wc -l)
        echo "Total Nodes: $total_nodes"
    fi
}

# Function to save results to file
save_results() {
    local server=$(oc whoami --show-server 2>/dev/null)
    local user=$(oc whoami 2>/dev/null)
    local namespace_count=$(get_namespace_count)
    local timestamp=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
    local filename="namespace-count-$(date +%Y%m%d-%H%M%S).json"
    
    # Get categorized node information for JSON
    local all_nodes=$(oc get nodes --no-headers -o custom-columns="NAME:.metadata.name,ROLES:.metadata.labels" 2>/dev/null)
    local master_nodes_json="[]"
    local infra_nodes_json="[]"
    local storage_nodes_json="[]"
    local worker_nodes_json="[]"
    local other_nodes_json="[]"
    
    if [[ -n "$all_nodes" ]]; then
        local master_list=()
        local infra_list=()
        local storage_list=()
        local worker_list=()
        local other_list=()
        
        # Process each node and categorize
        while IFS= read -r node_line; do
            if [[ -n "$node_line" ]]; then
                local node_name=$(echo "$node_line" | awk '{print $1}')
                local node_roles=$(echo "$node_line" | cut -d' ' -f2-)
                
                # Categorize nodes based on roles (priority: master > infra > storage > worker)
                if [[ "$node_roles" =~ node-role.kubernetes.io/master ]] || [[ "$node_roles" =~ node-role.kubernetes.io/control-plane ]]; then
                    master_list+=("$node_name")
                elif [[ "$node_roles" =~ node-role.kubernetes.io/infra ]]; then
                    infra_list+=("$node_name")
                elif [[ "$node_roles" =~ node-role.kubernetes.io/storage ]] || [[ "$node_roles" =~ cluster.ocs.openshift.io/openshift-storage ]]; then
                    storage_list+=("$node_name")
                elif [[ "$node_roles" =~ node-role.kubernetes.io/worker ]] || [[ "$node_roles" =~ "<none>" ]] || [[ -z "$(echo "$node_roles" | grep -E 'node-role.kubernetes.io/(master|control-plane|infra|storage)')" ]]; then
                    worker_list+=("$node_name")
                else
                    other_list+=("$node_name")
                fi
            fi
        done <<< "$all_nodes"
        
        # Convert arrays to JSON format (safer approach)
        if [[ ${#master_list[@]} -gt 0 ]]; then
            master_nodes_json="["
            for i in "${!master_list[@]}"; do
                if [[ $i -gt 0 ]]; then master_nodes_json+=","; fi
                master_nodes_json+="\"${master_list[i]}\""
            done
            master_nodes_json+="]"
        fi
        if [[ ${#infra_list[@]} -gt 0 ]]; then
            infra_nodes_json="["
            for i in "${!infra_list[@]}"; do
                if [[ $i -gt 0 ]]; then infra_nodes_json+=","; fi
                infra_nodes_json+="\"${infra_list[i]}\""
            done
            infra_nodes_json+="]"
        fi
        if [[ ${#storage_list[@]} -gt 0 ]]; then
            storage_nodes_json="["
            for i in "${!storage_list[@]}"; do
                if [[ $i -gt 0 ]]; then storage_nodes_json+=","; fi
                storage_nodes_json+="\"${storage_list[i]}\""
            done
            storage_nodes_json+="]"
        fi
        if [[ ${#worker_list[@]} -gt 0 ]]; then
            worker_nodes_json="["
            for i in "${!worker_list[@]}"; do
                if [[ $i -gt 0 ]]; then worker_nodes_json+=","; fi
                worker_nodes_json+="\"${worker_list[i]}\""
            done
            worker_nodes_json+="]"
        fi
        if [[ ${#other_list[@]} -gt 0 ]]; then
            other_nodes_json="["
            for i in "${!other_list[@]}"; do
                if [[ $i -gt 0 ]]; then other_nodes_json+=","; fi
                other_nodes_json+="\"${other_list[i]}\""
            done
            other_nodes_json+="]"
        fi
    fi
    
    cat > "$filename" << EOF
{
  "timestamp": "$timestamp",
  "cluster": {
    "server": "$server",
    "user": "$user"
  },
  "results": {
    "namespace_count": ${TOTAL_NAMESPACES:-$namespace_count},
    "control_plane_namespaces": ${CONTROL_PLANE_NAMESPACES:-0},
    "customer_namespaces": ${CUSTOMER_NAMESPACES:-0},
    "openshift_namespaces": ${OPENSHIFT_NAMESPACES:-0},
    "system_namespaces": ${SYSTEM_NAMESPACES:-0},
    "total_nodes": $(oc get nodes --no-headers 2>/dev/null | wc -l),
    "nodes_by_type": {
      "master": $master_nodes_json,
      "infrastructure": $infra_nodes_json,
      "storage": $storage_nodes_json,
      "worker": $worker_nodes_json,
      "other": $other_nodes_json
    }
  }
}
EOF
    
    print_status "Results saved to: $filename"
}

# Function to write HTML node group
write_html_node_group() {
    local filename="$1"
    local group_title="$2"
    local emoji="$3"
    local -n node_array=$4  # nameref to array
    
    if [[ ${#node_array[@]} -gt 0 ]]; then
        {
            echo "                <div class=\"node-group\">"
            echo "                    <div class=\"node-group-header\">"
            echo "                        <span class=\"emoji\">$emoji</span>"
            echo "                        $group_title (${#node_array[@]})"
            echo "                    </div>"
            echo "                    <ul class=\"node-list\">"
            for node in "${node_array[@]}"; do
                echo "                        <li>$node</li>"
            done
            echo "                    </ul>"
            echo "                </div>"
        } >> "$filename"
    fi
}

# Function to generate HTML report
generate_html_report() {
    local server=$(oc whoami --show-server 2>/dev/null)
    local user=$(oc whoami 2>/dev/null)
    local namespace_count=$(get_namespace_count)
    local timestamp=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
    local filename="namespace-report-$(date +%Y%m%d-%H%M%S).html"
    
    # Get categorized node information for HTML
    local all_nodes=$(oc get nodes --no-headers -o custom-columns="NAME:.metadata.name,ROLES:.metadata.labels" 2>/dev/null)
    local master_list=()
    local infra_list=()
    local storage_list=()
    local worker_list=()
    local other_list=()
    
    if [[ -n "$all_nodes" ]]; then
        # Process each node and categorize
        while IFS= read -r node_line; do
            if [[ -n "$node_line" ]]; then
                local node_name=$(echo "$node_line" | awk '{print $1}')
                local node_roles=$(echo "$node_line" | cut -d' ' -f2-)
                
                # Categorize nodes based on roles (priority: master > infra > storage > worker)
                if [[ "$node_roles" =~ node-role.kubernetes.io/master ]] || [[ "$node_roles" =~ node-role.kubernetes.io/control-plane ]]; then
                    master_list+=("$node_name")
                elif [[ "$node_roles" =~ node-role.kubernetes.io/infra ]]; then
                    infra_list+=("$node_name")
                elif [[ "$node_roles" =~ node-role.kubernetes.io/storage ]] || [[ "$node_roles" =~ cluster.ocs.openshift.io/openshift-storage ]]; then
                    storage_list+=("$node_name")
                elif [[ "$node_roles" =~ node-role.kubernetes.io/worker ]] || [[ "$node_roles" =~ "<none>" ]] || [[ -z "$(echo "$node_roles" | grep -E 'node-role.kubernetes.io/(master|control-plane|infra|storage)')" ]]; then
                    worker_list+=("$node_name")
                else
                    other_list+=("$node_name")
                fi
            fi
        done <<< "$all_nodes"
    fi
    
    local total_nodes=$((${#master_list[@]} + ${#infra_list[@]} + ${#storage_list[@]} + ${#worker_list[@]} + ${#other_list[@]}))
    
    # Calculate percentages
    local master_pct=0
    local infra_pct=0
    local storage_pct=0
    local worker_pct=0
    local other_pct=0
    
    if [[ $total_nodes -gt 0 ]]; then
        master_pct=$(( ${#master_list[@]} * 100 / total_nodes ))
        infra_pct=$(( ${#infra_list[@]} * 100 / total_nodes ))
        storage_pct=$(( ${#storage_list[@]} * 100 / total_nodes ))
        worker_pct=$(( ${#worker_list[@]} * 100 / total_nodes ))
        other_pct=$(( ${#other_list[@]} * 100 / total_nodes ))
    fi
    
    # Write HTML file using direct echo instead of HERE documents
    
    # Write HTML header and styles
    {
        echo '<!DOCTYPE html>'
        echo '<html lang="en">'
        echo '<head>'
        echo '    <meta charset="UTF-8">'
        echo '    <meta name="viewport" content="width=device-width, initial-scale=1.0">'
        echo '    <title>OpenShift Cluster Report</title>'
        echo '    <style>'
        echo '        body { font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif; margin: 0; padding: 20px; background-color: #f5f5f5; color: #333; }'
        echo '        .container { max-width: 1200px; margin: 0 auto; background: white; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); overflow: hidden; }'
        echo '        .header { background: linear-gradient(135deg, #ee0000, #cc0000); color: white; padding: 30px; text-align: center; }'
        echo '        .header h1 { margin: 0; font-size: 2.5em; font-weight: 300; }'
        echo '        .header p { margin: 10px 0 0 0; opacity: 0.9; font-size: 1.1em; }'
        echo '        .content { padding: 30px; }'
        echo '        .info-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 20px; margin-bottom: 30px; }'
        echo '        .info-card { background: #f8f9fa; padding: 20px; border-radius: 6px; border-left: 4px solid #ee0000; }'
        echo '        .info-card h3 { margin: 0 0 10px 0; color: #ee0000; font-size: 1.1em; text-transform: uppercase; letter-spacing: 0.5px; }'
        echo '        .info-card p { margin: 0; font-size: 1.3em; font-weight: bold; color: #333; }'
        echo '        .section { margin-bottom: 30px; }'
        echo '        .section h2 { color: #ee0000; border-bottom: 2px solid #ee0000; padding-bottom: 10px; margin-bottom: 20px; }'
        echo '        .node-group { background: #f8f9fa; margin: 15px 0; border-radius: 6px; overflow: hidden; }'
        echo '        .node-group-header { background: #e9ecef; padding: 15px 20px; font-weight: bold; color: #495057; display: flex; align-items: center; }'
        echo '        .node-group-header .emoji { margin-right: 10px; font-size: 1.2em; }'
        echo '        .node-list { padding: 0; margin: 0; list-style: none; }'
        echo '        .node-list li { padding: 12px 20px; border-bottom: 1px solid #dee2e6; display: flex; align-items: center; }'
        echo '        .node-list li:last-child { border-bottom: none; }'
        echo '        .node-list li:hover { background: #f1f3f4; }'
        echo '        .node-list li::before { content: "🖥️"; margin-right: 10px; }'
        echo '        .summary-table { width: 100%; border-collapse: collapse; margin-top: 20px; }'
        echo '        .summary-table th, .summary-table td { padding: 12px; text-align: left; border-bottom: 1px solid #dee2e6; }'
        echo '        .summary-table th { background: #f8f9fa; font-weight: bold; color: #495057; }'
        echo '        .summary-table tr:hover { background: #f1f3f4; }'
        echo '        .timestamp { text-align: center; color: #6c757d; font-size: 0.9em; margin-top: 30px; padding-top: 20px; border-top: 1px solid #dee2e6; }'
        echo '        @media (max-width: 768px) { .container { margin: 0; border-radius: 0; } .info-grid { grid-template-columns: 1fr; } .header h1 { font-size: 2em; } .content { padding: 20px; } }'
        echo '    </style>'
        echo '</head>'
        echo '<body>'
        echo '    <div class="container">'
        echo '        <div class="header">'
        echo '            <h1>🚀 OpenShift Cluster Report</h1>'
        echo '            <p>Namespace and Node Analysis</p>'
        echo '        </div>'
        echo '        <div class="content">'
        echo '            <div class="info-grid">'
        echo '                <div class="info-card">'
        echo '                    <h3>Cluster Server</h3>'
        echo "                    <p>$server</p>"
        echo '                </div>'
        echo '                <div class="info-card">'
        echo '                    <h3>Current User</h3>'
        echo "                    <p>$user</p>"
        echo '                </div>'
        echo '                <div class="info-card">'
        echo '                    <h3>Total Namespaces</h3>'
        echo "                    <p>${TOTAL_NAMESPACES:-$namespace_count}</p>"
        echo '                </div>'
        echo '                <div class="info-card">'
        echo '                    <h3>Control Plane</h3>'
        echo "                    <p>${CONTROL_PLANE_NAMESPACES:-0}</p>"
        echo '                </div>'
        echo '                <div class="info-card">'
        echo '                    <h3>Customer Namespaces</h3>'
        echo "                    <p>${CUSTOMER_NAMESPACES:-0}</p>"
        echo '                </div>'
        echo '                <div class="info-card">'
        echo '                    <h3>Total Nodes</h3>'
        echo "                    <p>$total_nodes</p>"
        echo '                </div>'
        echo '            </div>'
        echo '            <div class="section">'
        echo '                <h2>📊 Node Summary</h2>'
        echo '                <table class="summary-table">'
        echo '                    <thead>'
        echo '                        <tr>'
        echo '                            <th>Node Type</th>'
        echo '                            <th>Count</th>'
        echo '                            <th>Percentage</th>'
        echo '                        </tr>'
        echo '                    </thead>'
        echo '                    <tbody>'
        echo '                        <tr>'
        echo '                            <td>🏛️ Master/Control Plane</td>'
        echo "                            <td>${#master_list[@]}</td>"
        echo "                            <td>${master_pct}%</td>"
        echo '                        </tr>'
        echo '                        <tr>'
        echo '                            <td>🏗️ Infrastructure</td>'
        echo "                            <td>${#infra_list[@]}</td>"
        echo "                            <td>${infra_pct}%</td>"
        echo '                        </tr>'
        echo '                        <tr>'
        echo '                            <td>💾 Storage</td>'
        echo "                            <td>${#storage_list[@]}</td>"
        echo "                            <td>${storage_pct}%</td>"
        echo '                        </tr>'
        echo '                        <tr>'
        echo '                            <td>🔧 Worker</td>'
        echo "                            <td>${#worker_list[@]}</td>"
        echo "                            <td>${worker_pct}%</td>"
        echo '                        </tr>'
        if [[ ${#other_list[@]} -gt 0 ]]; then
            echo '                        <tr>'
            echo '                            <td>❓ Other</td>'
            echo "                            <td>${#other_list[@]}</td>"
            echo "                            <td>${other_pct}%</td>"
            echo '                        </tr>'
        fi
        echo '                    </tbody>'
        echo '                </table>'
        echo '            </div>'
        echo '            <div class="section">'
        echo '                <h2>🖥️ Nodes by Category</h2>'
    } > "$filename"
    
    # Use the helper function to write node groups
    write_html_node_group "$filename" "Master/Control Plane Nodes" "🏛️" master_list
    write_html_node_group "$filename" "Infrastructure Nodes" "🏗️" infra_list  
    write_html_node_group "$filename" "Storage Nodes" "💾" storage_list
    write_html_node_group "$filename" "Worker Nodes" "🔧" worker_list
    write_html_node_group "$filename" "Other Nodes" "❓" other_list
    
    # Write HTML footer
    {
        echo '            </div>'
        echo '            <div class="timestamp">'
        echo "                <p>Report generated on: $timestamp</p>"
        echo '                <p>Generated by: OpenShift Namespace Counter (Bash Version)</p>'
        echo '            </div>'
        echo '        </div>'
        echo '    </div>'
        echo '</body>'
        echo '</html>'
    } >> "$filename"
    
    print_status "HTML report saved to: $filename"
}

# Function to display usage
show_usage() {
    echo "Usage: $0 [OPTIONS]"
    echo ""
    echo "OpenShift Multi-Cluster Namespace Counter - Shell Script Version"
    echo ""
    echo "Options:"
    echo "  --url URL         OpenShift cluster API URL"
    echo "  -u, --username    Username for authentication"
    echo "  --details, -d     Show detailed node information"
    echo "  --save, -s        Save results to JSON file"
    echo "  --format FORMAT   Output format: table, json, html (default: table)"
    echo "  --quiet, -q       Suppress non-essential output"
    echo "  --help, -h        Show this help message"
    echo ""
    echo "Examples:"
    echo "  $0                                    # Use current login"
    echo "  $0 --url https://api.cluster.com:6443 -u myuser"
    echo "  $0 --details --save"
    echo "  $0 --details --format html"
    echo "  $0 --url https://api.cluster.com:6443 --details --format html"
}

# Main function
main() {
    local cluster_url=""
    local username=""
    local show_details=false
    local save_output=false
    local output_format="table"
    local quiet=false
    
    # Parse command line arguments
    while [[ $# -gt 0 ]]; do
        case $1 in
            --url)
                cluster_url="$2"
                shift 2
                ;;
            -u|--username)
                username="$2"
                shift 2
                ;;
            --details|-d)
                show_details=true
                shift
                ;;
            --save|-s)
                save_output=true
                shift
                ;;
            --format)
                output_format="$2"
                if [[ "$output_format" != "table" && "$output_format" != "json" && "$output_format" != "html" ]]; then
                    print_error "Invalid format: $output_format. Supported formats: table, json, html"
                    exit 1
                fi
                shift 2
                ;;
            --quiet|-q)
                quiet=true
                shift
                ;;
            --help|-h)
                show_usage
                exit 0
                ;;
            *)
                print_error "Unknown option: $1"
                show_usage
                exit 1
                ;;
        esac
    done
    
    if [[ "$quiet" == "false" ]]; then
        echo -e "${BLUE}${ROCKET}${NC} OpenShift Multi-Cluster Namespace Counter"
        echo -e "${BLUE}${CLOCK}${NC} Started at: $(date)"
        echo ""
    fi
    
    # Check if oc CLI is installed
    if ! check_oc_cli; then
        # oc CLI not found, exit early
        exit 1
    fi
    
    # Check current login status
    if check_login; then
        if [[ -n "$cluster_url" ]]; then
            print_warning "New cluster URL provided, attempting login..."
            if ! login_to_cluster "$cluster_url" "$username"; then
                print_error "Failed to login to new cluster"
                exit 1
            fi
        else
            if [[ "$quiet" == "false" ]]; then
                echo -n "Continue with current cluster? (y/n): "
                read -r continue_choice
                if [[ "$continue_choice" != "y" && "$continue_choice" != "Y" ]]; then
                    if ! login_to_cluster "$cluster_url" "$username"; then
                        print_error "Failed to login to cluster"
                        exit 1
                    fi
                fi
            fi
        fi
    else
        print_warning "Not currently logged into any cluster"
        if ! login_to_cluster "$cluster_url" "$username"; then
            print_error "Failed to login to cluster"
            exit 1
        fi
    fi
    
    echo ""
    print_info "Analyzing namespaces..."
    
    # Get detailed namespace information with classifications
    get_namespace_info "$show_details"
    
    # Show node information if requested
    if [[ "$show_details" == "true" ]]; then
        echo ""
        get_node_info true
    else
        echo ""
        get_node_info false
    fi
    
    # Handle output format
    case "$output_format" in
        "json")
            echo ""
            save_results
            ;;
        "html")
            echo ""
            generate_html_report
            ;;
        "table")
            # Default table output already shown above
            # Save results if explicitly requested
            if [[ "$save_output" == "true" ]]; then
                echo ""
                save_results
            fi
            ;;
    esac
    
    if [[ "$quiet" == "false" ]]; then
        echo ""
        print_status "Namespace counting completed!"
        echo -e "${BLUE}${CLOCK}${NC} Finished at: $(date)"
    fi
}

# Run main function with all arguments
main "$@"


```
