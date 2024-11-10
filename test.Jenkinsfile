
node('maven') {

    def appName="test"
    def projectName="test"
    def repoUrl="github.com/Adhitia09/testdeploy.git"
    def branchName="main"
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
            def tag = sh(returnStdout: true, script: "git rev-parse --short=8 HEAD").trim();
            echo "Commit ID: ${tag}"
        }
    }


     stage ('App Push') {
        dir("source") {
            def tokenLocal = sh(script: 'oc whoami -t', returnStdout: true).trim()
            withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                sh "set +x"
                sh "skopeo login --username=adhitia09 --password=Gumelar09!! docker.io"
                sh "skopeo copy --remove-signatures --src-creds=jenkins:${tokenLocal} --src-tls-verify=false docker://senaturana/plato:latest docker://${extRegistryQuayDRC}/djbc/plato:latest --dest-creds \${USERNAME}:\${PASSWORD} --dest-tls-verify=false"
            }
        }
    }
 

}

