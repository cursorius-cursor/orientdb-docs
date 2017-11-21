stage 'Generate docs for branch ${env.BRANCH_NAME}'
node("master") {
    
    properties([[$class: 'BuildDiscarderProperty', 
                 strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '', 
                            artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10']]])
    sh "rm -rf ./*"

    checkout scm
    echo "building docs for branch  ${env.BRANCH_NAME}"
    
    def containerName = env.JOB_NAME.replaceAll(/\//, "_") + 
            "_build_${currentBuild.number}"
			
    def appNameLabel = "docker_ci";
    def taskLabel = env.JOB_NAME.replaceAll(/\//, "_")
    
    docker.image("orientdb/jenkins-slave-gitbook:20160511").inside("--label collectd_docker_app=${appNameLabel} --label collectd_docker_task=${taskLabel}" + 
                                                                   " --name ${containerName} --memory=2g ${env.VOLUMES}") {
        sh "rm -rf _book/*"
        sh "gitbook install --gitbook 3.2.2 . "
        sh "gitbook build --gitbook 3.2.2 . "
        sh "gitbook pdf --gitbook 3.2.2 . _book/OrientDB-Manual.pdf"
    }

    if (!env.BRANCH_NAME.startsWith("PR-")) {
        echo "sync generated content to OrientDB site"
        docker.image("orientdb/jenkins-slave-rsync:20160503").inside("--label collectd_docker_app=${appNameLabel} --label collectd_docker_task=${taskLabel}" +
                                                                     " --name ${containerName} --memory=2g ${env.VOLUMES}") {
            sh "rsync -ravh --stats _book/  -e ${env.RSYNC_DOC}/${env.BRANCH_NAME}/"
        }
    } else {
        echo "it's a PR, no sync required"
    }

}


