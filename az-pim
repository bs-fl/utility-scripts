#!/bin/bash

# Example role list commands
#   az rest --method get --url 'https://management.azure.com/subscriptions/SUB-1-GUID/providers/Microsoft.Authorization/roleDefinitions?api-version=2018-07-01'
#   az rest --method get --url 'https://management.azure.com/subscriptions/SUB-2-GUID/providers/Microsoft.Authorization/roleDefinitions?api-version=2018-07-01'

# Validate Azure CLI is installed
which az >&/dev/null
AZ_INSTALLED=${?}
(( AZ_INSTALLED )) && { printf "Azure CLI may not be installed.\nPlease ensure that it is installed and try again...\n"; exit 1; };

# If a custom justification message is passed in, use that in place of the default
[[ -z "${1}" ]] && PIM_JUSTIFICATION="Cloud resource management" || PIM_JUSTIFICATION="${1}"

# Troubleshooting-related variables
AZ_ERROR=0
AZ_LOG=~/tmp/az_pmi_output.log
: > ${AZ_LOG}

# The Object ID of the user
#   This can be obtained by going to the Azure portal and accessing:
#       Azure Active Directory > Users > Search for your name
MY_PRINCIPAL_ID=''
[[ -z "${MY_PRINCIPAL_ID}" ]] && { printf 'Please update this script with your principal ID...\nDetails can be found in script comments\n\n'; exit 1; };

# Subscription IDs
#   These names must match the names in "AZ_SUBS" with "_id" appeneded to the end, e.g.
#   Subscription_Name_1_id='11111111-1111-1111-1111-111111111111'

# Role scopes by subscription
#   These names must match the names in "AZ_SUBS", e.g.
#   Subscription_Name_1=(
#       'subscriptions/11111111-1111-1111-1111-111111111111'
#       'subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/resource-group-01'
#   )

# Human-readable subscription names
AZ_SUBS=(
    # 'Subscription_Name_1'
)

# Iterate over the array of subscriptions
for _sub in "${AZ_SUBS[@]}"; do
    id_varname="${_sub}_id"     # Variable name for the current subscription's ID
    sub_id="${!id_varname}"     # Get the ID for the current subscription

    scope_list_varname="${_sub}[@]"             # Variable name for the list of scopes for the current subscription
    scope_list=("${!scope_list_varname}")       # Get the list of scopes for the current subscription

    for _scope in "${scope_list[@]}"; do
        request_name="$(uuidgen)"   # Generate a GUID to use as the name of the PIM request

        # Create the URL to send the PIM request to
        az_url="https://management.azure.com/${_scope}/providers/Microsoft.Authorization/roleAssignmentScheduleRequests/${request_name}?api-version=2020-10-01"

        # The JSON data that provides the who/what/when for the PIM request
        #   This request is specific to the "Contributor" role (b24988ac-6180-42a0-ab88-20f7382dd24c)
        # 
        #   The "StartDateTime" is set to "null" so that the activation will start ASAP
        #     If the activation should happen some time in the future, this can be changed to match the following pattern: 2020-09-09T21:31:00.00Z
        # 
        #   The "Duration" can be changed to reflect how long the activation should last
        #     "PT8H" is for an 8 hour duration
        #     "PT30M" would be for a 30 minute duration
        az_json="
        {
            \"Properties\": {
                \"RoleDefinitionId\": \"/subscriptions/${sub_id}/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c\",
                \"PrincipalId\": \"${MY_PRINCIPAL_ID}\",
                \"RequestType\": \"SelfActivate\",
                \"Justification\": \"${PIM_JUSTIFICATION}\",
                \"ScheduleInfo\": {
                    \"StartDateTime\": null,
                    \"Expiration\": {
                        \"Type\": \"AfterDuration\",
                        \"EndDateTime\": null,
                        \"Duration\": \"PT8H\"
                    }
                }
            }
        }"

        printf "\nSending PIM request for:  ${_sub}"
        printf "\nRequest name:             ${request_name}\n"
        # Send the request using the above JSON data and URL
        az rest --method put --uri "${az_url}" --body "${az_json}" >>${AZ_LOG} 2>&1
        az_result=${?}
        printf "\n\n" >> ${AZ_LOG}  # Separate requests in output log
        (( az_result )) && { printf "\nError sending request...\nPlease check ${AZ_LOG} for details\n\n"; AZ_ERROR=1; } || { printf "Request sent successfully!\n"; };
    done
done
echo

# Cleanup log file if there were no errors
(( AZ_ERROR )) || rm ${AZ_LOG}