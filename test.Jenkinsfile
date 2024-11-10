
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
            sh "skopeo copy docker://adhitia09/app-nginx:1.0 docker://${extRegistryQuayDRC}/djbc/app-nginx:1.0 --dest-creds \${USERNAME}:\${PASSWORD} --dest-tls-verify=false"
            withCredentials([usernamePassword(credentialsId: 'quay-drc-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                
            }
        }
    }
 

}

