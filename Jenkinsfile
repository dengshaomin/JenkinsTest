pipeline {
    agent any
    parameters {
        choice choices: ['Test', 'Release'], description: '包类型', name: 'PackageType'
    }

    stages {
        stage('commit log') {
            steps {
                script {
                    def changeLogSets = currentBuild.changeSets
                    for (int i = 0; i < changeLogSets.size(); i++) {
                        def entries = changeLogSets[i].items
                        for (int j = 0; j < entries.length; j++) {
                            def entry = entries[j]
                            echo "${entry.commitId} by ${entry.author} on ${changeTime(entry.timestamp)}: ${entry.msg}"
//                            def files = new ArrayList(entry.affectedFiles)
//                            for (int k = 0; k < files.size(); k++) {
//                                def file = files[k]
//                                echo "  ${file.editType.name} ${file.path}"
//                            }
                        }
                    }
                }
            }
        }
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
def changeTime(long time){
    return new Date(time).format("yyyy-MM-dd HH:mm:ss")
}