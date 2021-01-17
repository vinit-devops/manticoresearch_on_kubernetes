# manticoresearch_on_kubernetes
This a semi-automated set up for manticore cluster set up on docker container-runtime  via kubernetess with the help of ansible.
Technologies needed for this set up-
            i) Kubernetes
            ii) Ansible
            iii) Git.
            
In this set up we are going to create two docker containers running manticoresearch latest service, both of which forming a cluster in which data is replicated from master to slave. Even if all the nodes goes down all the data will be safe within /docker-data, including your logs and mysql tables data.

* post_pod_creation and pre_pod_creation consists of playbooks that needs to be run on ansible servers.
* kubernetes-yml-files consists of defination files for various pods and services.


SEQUENCE OF USAGE ::    pre_pod_creation -> kubernetes-yml-files -> post_pod_creation .

Read all the steps below get a working manticore cluster in just 5 mins.

PRE-REQUISITES:
            1. Ansible server should be keyless with kubernetes worker nodes
            2. Worker nodes should show a ready status on kubernetes master.
            3. Git should be installed on kubernetes master and Ansible server.


STEPS FOR CLUSTER SET UP OF MANTICORESEARCH:

            i) Go to ansible server and take a pull of the repo
            
                git clone https://github.com/vinit-devops/manticoresearch_on_kubernetes.git
            
            ii) cd to manticoresearch_on_kubernetes/pre-pod-creation/ 
            
                edit hosts entry in pre-pod-creation.yml with the nodes or ip of kubernetes worker nodes ips on which service needs to be deployed.
                Run ansible playbook with below command
                ansible-playbook pre-pod-creation.yml             
                #ensure here that the manticore.conf exists on path manticoresearch_on_kubernetes/pre-pod-creation/new_files .
                        
            iii) Go to kubernetes master node and label nodes with commands
                        kubectl label nodes <node1-name> ip=qd1
                        kubectl label nodes <node2-name> ip=qd2
                Note: node1 and node2 are worker nodes which are defined in host entry of pre-pod-creation.yml, node-name can also be obtained via below command
                kubectl get nodes
                To ensure the labels are created successfully , run below command
                 kubectl get nodes --show-labels
                 
             iv) Take a pull of the the repo again.
                Next, cd to manticoresearch_on_kubernete/kubernetes-yml-files and spin up the pods and service  with below commands
                kubectl apply -f master_deployment_definition.yml
                kubectl apply -f slave_deployment_definition.yml
                kubectl apply -f manticoresearch-service.yaml
                         
            
             iv) Go to ansible server and cd to manticoresearch_on_kubernete/post_pod_creation/ and 
                 edit the post_pod_creation.yml with at hosts with ip of server on which manticore-dc pod is created and h1 and h2 with mantciore-dc node ip and manticore-dc2 
                 node ip respectively,
                 in order to see which pod is created on which node, run below command
                 kubectl get pods -o wide
                 
              v) At last run below command to run ansible playbook to create cluster
                        ansible-playbook post_pod_creation.yml
               at last step of playbook running if you see the status synced , the BINGO! mantcioer cluster is created successfully
               
               
               To check cluster status you can run below commands on worker nodes
                                    mysql -h0 -P 30306 -e "show status " | grep node_state


FOR ANY QUERY OR SUGGESTIONS: vinitk805582@gmail.com

