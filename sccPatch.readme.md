#!/bin/bash

# Check for at least two arguments
if [ "$#" -lt 2 ]; then
  echo "Usage: $0 <ClusterRoleBinding_Name> <Subject_To_Remove1> [<Subject_To_Remove2> ...]"
  exit 1
fi

# ClusterRoleBinding name is the first argument
CRB_NAME="$1"

# Subjects to remove are the remaining arguments
SUBJECTS_TO_REMOVE=("${@:2}")

# Get the current ClusterRoleBinding in YAML format and back it up
BACKUP_FILENAME="${CRB_NAME}-backup-$(date +%Y%m%d%H%M%S).yaml"
oc get clusterrolebinding "$CRB_NAME" -o yaml > "$BACKUP_FILENAME"
if [ $? -ne 0 ]; then
    echo "Failed to get ClusterRoleBinding or backup. Exiting."
    exit 2
fi

# Inform the user of the backup
echo "Backup of ClusterRoleBinding saved to $BACKUP_FILENAME"

# Get the current ClusterRoleBinding in JSON format for processing
CRB_JSON=$(oc get clusterrolebinding "$CRB_NAME" -o json)
if [ $? -ne 0 ]; then
    echo "Failed to get ClusterRoleBinding in JSON format. Exiting."
    exit 3
fi

# Initialize the patch JSON array
PATCH="[]"

# Iterate over the subjects to construct the patch for removal
for subject_to_remove in "${SUBJECTS_TO_REMOVE[@]}"; do
  # Filter out the subjects that are of the specified kind and match the name pattern
  PATCH=$(echo "${CRB_JSON}" | jq --arg NAME "$subject_to_remove" --argjson PATCH "$PATCH" '.subjects | to_entries | map(select(.value.kind == "Group" and (.value.name | split(":")[1] == $NAME) | not)) | map(.value) | add | $PATCH + map({"op": "remove", "path": "/subjects/-"})')
done

# Create a temporary file for the patch
PATCH_FILE=$(mktemp)
if [ ! -f "$PATCH_FILE" ]; then
    echo "Failed to create a temporary file for the patch. Exiting."
    exit 5
fi

# Write the patch to the temporary file
echo "$PATCH" > "$PATCH_FILE"

# Apply the patch using the temporary file
if [ "$PATCH" != "[]" ]; then
  oc patch clusterrolebinding "$CRB_NAME" --type='json' --patch-file="$PATCH_FILE"
  PATCH_RESULT=$?
  rm -f "$PATCH_FILE"
  if [ $PATCH_RESULT -eq 0 ]; then
    echo "ClusterRoleBinding $CRB_NAME modified."
  else
    echo "Failed to patch ClusterRoleBinding. Exiting."
    exit 4
  fi
else
  echo "No matching subjects of kind 'Group' found to remove."
fi
