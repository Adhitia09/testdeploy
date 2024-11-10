pipeline {
    agent any

    environment {
        // pullImageDC = "quayuser-djbc-dc-pull-secret"
        // pullImageDRC = "quayuser-djbc-drc-pull-secret"

        extRegistryQuayDC = "quay-registry.apps.proddc.customs.go.id"
        extRegistryQuayDRC = "quay-registry.apps.proddrc.customs.go.id"
        
        projectName = "test"
        repoUrl = "github.com/Adhitia09/testdeploy.git"
        branchName = "main"
    }

    stages {
        stage ('Git Clone') {
            steps {
                sh "git config --global http.sslVerify false"
                sh "rm -rf source"
                withCredentials([usernamePassword(credentialsId: 'customs-gitlab-credential', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh "git clone https://${USERNAME}:${PASSWORD}@${repoUrl} source"
                }
            }
        }
        
        stage ('App Build') {
            steps {
                dir("source") {
                    sh "git fetch"
                    sh "git switch ${branchName}"
                }
            }
        }

        stage ('push') {
            steps {
                dir("source") {
                    script {
                        withCredentials([usernamePassword(credentialsId: 'docker-credentials', variable: 'token')]){
                            tokenDocker = token
                            sh "docker login docker.io --token=${tokenDocker}"
                        }
                        sh "sleep 60"
                        withCredentials([usernamePassword(credentialsId: 'quay-dc-credential', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                            sh "skopeo copy --remove-signatures  --src-creds=jenkins:${tokenLocal} --src-tls-verify=false docker://${intRegistryDev}/${projectName}/${appName}:latest docker://${extRegistryQuayDC}/djbc/${projectName}_${appName}-dev:latest --dest-creds \${USERNAME}:\${PASSWORD} --dest-tls-verify=false"
                        }   
                    }
                }
            }
        }
        
        // stage('Deploy to Dev') {
        //     parallel {
        //         stage('App Deploy to DEV DC OCP') {
        //             steps {
        //                 dir("source") {
        //                     sh "cp plato-deployment.yaml kubernetes-dc.yaml"
        //                     sh "oc apply -f quayuser-djbc-dc-pull-secret.yaml -n ${projectName}"
        //                     sh "sed 's,\\\$REGISTRY/\\\$HARBOR_NAMESPACE/\\\$APP_NAME:\\\$BUILD_NUMBER,${extRegistryQuayDC}/djbc/plato:latest,g' kubernetes-dc.yaml > kubernetes-dc-quay.yaml"
        //                     sh "sed 's,\\\$IMAGEPULLSECRET,${pullImageDC},g' kubernetes-dc-quay.yaml > kubernetes-ocp-quay-dc.yaml"
        //                     sh "oc apply -f kubernetes-ocp-quay-dc.yaml -n ${projectName}"
        //                     sh "oc set triggers deployment/plato-deployment -c plato-deployment -n ${projectName} || true"
        //                     sh "oc rollout restart deployment/plato-deployment -n ${projectName}"
        //                 }
        //             }
        //         }

        //         stage('App Deploy to DEV DRC OCP') {
        //             steps {
        //                 dir("source") {
        //                     withCredentials([file(credentialsId: 'drc-dev-ocp', variable: 'KUBE_CONFIG_DEV_DRC')]) {
        //                         sh "cp plato-deployment.yaml kubernetes-drc.yaml"
        //                         sh "oc --kubeconfig=\$KUBE_CONFIG_DEV_DRC apply -f quayuser-djbc-drc-pull-secret.yaml -n ${projectName}"
        //                         sh "sed 's,\\\$REGISTRY/\\\$HARBOR_NAMESPACE/\\\$APP_NAME:\\\$BUILD_NUMBER,${extRegistryQuayDRC}/djbc/plato:latest,g' kubernetes-drc.yaml > kubernetes-drc-quay.yaml"
        //                         sh "sed 's,\\\$IMAGEPULLSECRET,${pullImageDRC},g' kubernetes-drc-quay.yaml > kubernetes-ocp-quay-drc.yaml"
        //                         sh "oc --kubeconfig=\$KUBE_CONFIG_DEV_DRC apply -f kubernetes-ocp-quay-drc.yaml -n ${projectName}"
        //                         sh "oc --kubeconfig=\$KUBE_CONFIG_DEV_DRC set triggers deployment/plato-deployment -c plato-deployment -n ${projectName} || true"
        //                         sh "oc --kubeconfig=\$KUBE_CONFIG_DEV_DRC rollout restart deployment/plato-deployment -n ${projectName}"
        //                     }
        //                 }
        //             }
        //         }
        //     }
        // }
    }
}
