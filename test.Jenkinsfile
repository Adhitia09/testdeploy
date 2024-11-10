
node('maven') {

    def appName="test"
    def projectName="test"
    def repoUrl="github.com/Adhitia09/testdeploy.git"
    def branchName="main"

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
                sh "skopeo copy --remove-signatures --src-creds=jenkins:${tokenLocal} --src-tls-verify=false docker://senaturana/plato:latest docker://${extRegistryQuayDC}/djbc/${projectName}_${appName}-prod:latest --dest-creds \${USERNAME}:\${PASSWORD} --dest-tls-verify=false"
            }
        }
    }
 

}

