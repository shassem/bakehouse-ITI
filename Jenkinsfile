pipeline {
    agent {label 'agent1'} 
    parameters {
        choice(name:'BRANCH', choices: ['release','dev','test','prod'])
    }

    stages {

        stage('Ci') {
            steps {
                script {
                    if (params.BRANCH == "release") {
                            
                        withCredentials([usernamePassword(credentialsId: 'Dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) { 
                        
                        sh """
                            
                            docker build . -t shassem/bakehouse:$BUILD_NUMBER
                            docker login -u ${USERNAME} -p ${PASSWORD}
                            docker push shassem/bakehouse:$BUILD_NUMBER
                            echo ${BUILD_NUMBER} > ../bnumber.txt
                        """
                        
                        }    
                    }                
                }
            }
        }        
        
        stage('Cd') {
            steps {
                script{    
                    if (params.BRANCH == 'dev' || params.BRANCH == 'test' || params.BRANCH == 'prod' ) {
                        withCredentials([file(credentialsId: 'configtxt', variable: 'kubeconf')]){
                    
                            sh """
                                    if [ -f ../bnumber.txt ]; then
                                        export BUILD_NUMBER=\$(cat ../bnumber.txt)
                                    else
                                        export BUILD_NUMBER=0
                                    fi
                                    mv Deployment/deploy.yaml Deployment/deploy.yaml.tmp
                                    cat Deployment/deploy.yaml.tmp | envsubst > Deployment/deploy.yaml
                                    rm -f Deployment/deploy.yaml.tmp
                                    kubectl apply --kubeconfig=${kubeconf} -f Deployment    
                            
                            """
                        }
                    }               
                }    
                 
                 
            }     

            
        }         

    }
}