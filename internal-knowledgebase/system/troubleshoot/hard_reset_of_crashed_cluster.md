---
tool_name: confd_client
doc_type: troubleshoot
category: system
title: "Hard Reset ‼️not for Wooga!!"
summary: "========== **‼️IMPORTANT‼️: DO NOT APPLY THIS TO WOOGA**"
---
# Hard Reset ‼️not for Wooga!!

==========
**‼️IMPORTANT‼️: DO NOT APPLY THIS TO WOOGA**

When we get a monitoring alert like "Customer cluster error monitor - Cluster UUID: ...."
Pay attention to the "Cluster UUID" in the alert.

## Identify the scenario

1. Login to AWS: https://github.com/exasol/internal-knowledgebase/blob/main/SaaS/saas-how-to-login.md
2. Enter COS container:

    ```bash
    sudo -u root -H sh -c "ssh -i /home/ubuntu/.ccc/play/local/*/main/*/root/root/.ssh/id_rsa root@localhost -p 20002"
    ```

    ![image](images/hard_reset/1_enter_cos_container.png)

3. Execute these commands to print the Scenario:

    ```bash
    # Ask the user for the crashed cluster ID
    read -p "Enter the crashed cluster ID: " crashed_cluster_id

    # Obtain the formatted section list
    sections=$(dwad_client list | grep -P "^Name|^State|^Connection state|^Master System" | sed '/^Name/i --------')

    # Identify the Master System ID dynamically
    master_system_id=$(echo "$sections" | grep "Master System" | tail -1 | awk -F': ' '{print $2}')

    # Extract all sections and format them for later processing
    cluster_sections=$(echo "$sections" | awk -v RS='--------' '{ if ($0 != "") print "--------" $0 }')

    # Check if the Master System has state not running or connection not up
    master_state_running=$(echo "$sections" | grep -A2 "Name: $master_system_id" | grep -q "State: running"; echo $?)
    master_connection_up=$(echo "$sections" | grep -A2 "Name: $master_system_id" | grep -q "Connection state: up"; echo $?)
    master_state_crashed=$(( $master_state_running -ne 0 || $master_connection_up -ne 0 ))

    # Define a function to print clusters and their statuses
    print_clusters () {
        echo "$cluster_sections" | awk -v master_id="$master_system_id" -v crashed_id="$crashed_cluster_id" -v master_crashed="$master_state_crashed" '
        $0 ~ /Name: / {
            name_line = $0
            name = substr(name_line, index(name_line, ": ") + 2)
            gsub(/^ +| +$/, "", name)  # Trim leading and trailing spaces
            if (name == master_id) {
                if (master_crashed || name == crashed_id) {
                    print "[main] " name " [crashed]"
                } else {
                    print "[main] " name
                }
            } else {
                if (name == crashed_id) {
                    print "[worker] " name " [crashed]"
                } else {
                    print "[worker] " name
                }
            }
        }'
    }

    # Check if the crashed system is the Master System
    if [ "$crashed_cluster_id" == "$master_system_id" ]; then
        if [ $(echo "$sections" | grep -c "^Name:") -eq 1 ]; then
            echo "Scenario 1"
            print_clusters
        else
            echo "Scenario 2"
            print_clusters
        fi
    else
        if [ $master_state_crashed -ne 0 ]; then
            echo "Scenario 2"
            print_clusters
        else
            echo "Scenario 3"
            print_clusters
        fi
    fi
    ```
   ![image](images/hard_reset/2_script_output.png)

## ▶ Scenario 1: MAIN cluster crashed (DB has no WORKERs)
   1. STOP the MAIN cluster using `confd_client infra_db_stop db_name: <main-cluster-uuid>`
   2. STOP (Shutdown) N10 EC2 instance
   3. START N10 EC2 instance - Wait for checks to be completed
   4. START the MAIN cluster using `confd_client infra_db_start db_name: <main-cluster-uuid>`

## ▶ Scenario 2: MAIN cluster crashed (DB has WORKERs)

   1. STOP the WORKER cluster(s) and wait until all EC2 instances of the WORKER cluster are in Stopped state

        ```
        confd_client infra_db_stop db_name: <worker-cluster-uuid>
        ```

   2. STOP the MAIN cluster and wait until all EC2 instances of the MAIN cluster are in Stopped state
      ```
      confd_client infra_db_stop db_name: <main-cluster-uuid>
      ```
      ![image](images/hard_reset/3_scenario_2.png)

   4. Exit COS container and find instance_id and regiond of N10

      ```bash
      c4 ps -o json -e 'map(select(.Instances.Configs["10"].ground.instance_id != null) | {instance_id: .Instances.Configs["10"].ground.instance_id, region: .Instances.Configs["10"].ground.region})'
      ```
      ![image](images/hard_reset/4_exit_cos_container.png)

   6. STOP (Shutdown) N10 EC2 instance
      ![image](images/hard_reset/5_stop_n10.png)

   8. START N10 EC2 instance and wait for checks to be completed

   9. START all the clusters that were stopped, beginning with the MAIN cluster.
      Start the cluster(s) and wait for the MAIN cluster to stabilize

      ```
      confd_client infra_db_start db_name: <cluster-uuid>
      ```

## ▶ Scenario 3: WORKER cluster crashed

   1. STOP the WORKER cluster using `confd_client infra_db_stop db_name: <worker-cluster-uuid>`
   2. START the WORKER cluster using `confd_client infra_db_start db_name: <worker-cluster-uuid>`
