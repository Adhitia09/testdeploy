node('maven') {

    def appName="plato-deployment"
    def projectName="pentaho"
    def repoUrl="gitlab.customs.go.id/sena.darodata/pentaho-bcare.git"
    def branchName="master"
    
    def pullImageDC="quayuser-djbc-dc-pull-secret"
    def pullImageDRC="quayuser-djbc-drc-pull-secret"

    def extRegistryQuayDC="quay-registry.apps.proddc.customs.go.id"
    def extRegistryQuayDRC="quay-registry.apps.proddrc.customs.go.id"

    stage ('Git Clone') {
        sh "git config --global http.sslVerify false"
        withCredentials([usernamePassword(credentialsId: 'customs-gitlab-credential', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            sh "git clone https://\${USERNAME}:\${PASSWORD}@${repoUrl} source "
        }
    }
    
    stage ('App Build') {
        dir("source") {
            sh "git fetch"
            sh "git switch ${branchName}" 
        }
    }

    stage ('App Push') {
        dir("source") {
		withCredentials([usernamePassword(credentialsId: 'quay-dc-credential', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){          
		        sh "skopeo copy docker://senaturana/plato:latest docker://${extRegistryQuayDC}/djbc/plato:latest --dest-creds \${USERNAME}:\${PASSWORD} --dest-tls-verify=false"        
	        }
	        withCredentials([usernamePassword(credentialsId: 'quay-drc-credential', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){          
		        sh "skopeo copy docker://senaturana/plato:latest docker://${extRegistryQuayDRC}/djbc/plato:latest --dest-creds \${USERNAME}:\${PASSWORD} --dest-tls-verify=false"        
	        }
        }
    }

    stage('Deploy to Dev') {
        parallel (
            "App Deploy to DEV DC OCP": {
                dir("source") {
                    sh "cp plato-deployment.yaml kubernetes-dc.yaml"
                    sh "oc  apply -f quayuser-djbc-dc-pull-secret.yaml -n ${projectName}"
                    sh "sed 's,\\\$REGISTRY/\\\$HARBOR_NAMESPACE/\\\$APP_NAME:\\\$BUILD_NUMBER,${extRegistryQuayDC}/djbc/plato:latest,g' kubernetes-dc.yaml > kubernetes-dc-quay.yaml"
                    sh "sed 's,\\\$IMAGEPULLSECRET,${pullImageDC},g' kubernetes-dc-quay.yaml > kubernetes-ocp-quay-dc.yaml"
                    sh "oc apply -f kubernetes-ocp-quay-dc.yaml -n ${projectName}"
                    sh "oc set triggers deployment/${appName} -c ${appName} -n ${projectName} || true "
                    sh "oc rollout restart deployment/${appName} -n ${projectName}"
                }
            },

            "App Deploy to DEV DRC OCP": {
                dir("source") {
                    withCredentials([file(credentialsId: 'drc-dev-ocp', variable: 'KUBE_CONFIG_DEV_DRC')]) {
                        sh "cp plato-deployment.yaml kubernetes-drc.yaml"
                        sh "oc --kubeconfig=\$KUBE_CONFIG_DEV_DRC apply -f quayuser-djbc-drc-pull-secret.yaml -n ${projectName}"
                        sh "sed 's,\\\$REGISTRY/\\\$HARBOR_NAMESPACE/\\\$APP_NAME:\\\$BUILD_NUMBER,${extRegistryQuayDRC}/djbc/plato:latest,g' kubernetes-drc.yaml > kubernetes-drc-quay.yaml"
                        sh "sed 's,\\\$IMAGEPULLSECRET,${pullImageDRC},g' kubernetes-drc-quay.yaml > kubernetes-ocp-quay-drc.yaml"
                        sh "oc --kubeconfig=\$KUBE_CONFIG_DEV_DRC apply -f kubernetes-ocp-quay-drc.yaml -n ${projectName}"
                        sh "oc --kubeconfig=\$KUBE_CONFIG_DEV_DRC set triggers deployment/${appName} -c ${appName} -n ${projectName} || true "
                        sh "oc --kubeconfig=\$KUBE_CONFIG_DEV_DRC rollout restart deployment/${appName} -n ${projectName}"
                    }
                }
            }
        )
    }
}
