pipeline {
    agent { 
        node {
        label params.Node
        }
    }
    parameters{
        choice(name: "Node", choices: ["master", "develop"], description: "Node to be used for the building and deploying")
    }
    environment{
        KUBESPRAY_DIR= "kubespray"
        PATHKEY= "/root/.ssh/id_rsa"
        MASTERNODE="172.168.8.1"
        N1="172.168.8.2"
        N2="172.168.8.3"
        //docker or containerd
        CONTAINER_MANAGER = "docker"
    }

    stages {        
        stage('Prepare Tools') {
            steps {
                
                echo '--------------Preparing tools...--------------'
                sh "apt-get install python3 python3-pip -y"
                sh "git clone https://github.com/kubernetes-sigs/kubespray.git"
                sh "pip install -r ${KUBESPRAY_DIR}/requirements-2.12.txt"
            
            }
        }
        stage('Build') {
            steps {
                echo '--------------Build...--------------'
                sh "cp -rfp ${KUBESPRAY_DIR}/inventory/sample ${KUBESPRAY_DIR}/inventory/mycluster"
                sh "CONFIG_FILE=kubespray/inventory/mycluster/hosts.yaml python3 kubespray/contrib/inventory_builder/inventory.py ${MASTERNODE} ${N1} ${N2}"
                sh "curl https://raw.githubusercontent.com/Michal944/Jobs/master/hosts.yaml -o ${KUBESPRAY_DIR}/inventory/mycluster/hosts.yaml"
                sh "cat ${KUBESPRAY_DIR}/inventory/mycluster/hosts.yaml"
                sh "sed -i -e \'/container_manager:/s/containerd/${CONTAINER_MANAGER}/\' kubespray/inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml"
            }
        }
        stage('Authorized keys') {
            steps {
                echo "--------------Authorized keys--------------"
                sh "apt-get update && sudo apt-get install openssh-client -y"
                sh "apt-get install sshpass -y"
                sh "ssh-keygen -t rsa -f ${PATHKEY} -q -P \'\' "
                withCredentials([usernamePassword(credentialsId: 'k8snodes', usernameVariable: 'USER', passwordVariable: 'PASSWORD')]){                        
                    sh "sshpass -p ${PASSWORD} ssh-copy-id -f -o StrictHostKeyChecking=no -i ${PATHKEY} ${USER}@${MASTERNODE}"
                    sh "sshpass -p ${PASSWORD} ssh-copy-id -f -o StrictHostKeyChecking=no -i ${PATHKEY} ${USER}@${N1}"
                    sh "sshpass -p ${PASSWORD} ssh-copy-id -f -o StrictHostKeyChecking=no -i ${PATHKEY} ${USER}@${N2}"                    
                }
            }
            
            
       
        }
        stage('Deploy') {
            steps {
                echo '--------------Deploying....--------------'
                sh "ansible-playbook -i kubespray/inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml"
            }
        }
    }
    post {
        always {
            cleanWs()
            sh " rm -f ${PATHKEY} ${PATHKEY}.pub"
        }
    }
}