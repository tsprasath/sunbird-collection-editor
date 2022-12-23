node() {
    try {
        String ANSI_GREEN = "\u001B[32m"
        String ANSI_NORMAL = "\u001B[0m"
        String ANSI_BOLD = "\u001B[1m"
        String ANSI_RED = "\u001B[31m"
        String ANSI_YELLOW = "\u001B[33m"

        ansiColor('xterm') {
            stage('Checkout') {
                cleanWs()
                    sh "git clone https://github.com/tsprasath/sunbird-collection-editor collection-editor"
                    sh "cd collection-editor && git checkout origin/release-5.1.0 -b release-5.1.0"
                    //commit_hash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    branch_name = 'release-5.1.0'
                    artifact_version = branch_name 
                    //println(ANSI_BOLD + ANSI_YELLOW + "github_release_tag not specified, using the latest commit hash: " + commit_hash + ANSI_NORMAL)
                    sh "git clone https://github.com/project-sunbird/sunbird-content-plugins.git plugins"
                    sh "cd plugins && git checkout origin/release-5.2.0 -b release-5.2.0"
                    echo "artifact_version: " + artifact_version
                }  

                stage('Build') {
                    sh """
                        export version_number=${branch_name}
                        rm -rf collection-editor
                        node -v
                        npm -v                        
                        npm install
                        cd app
                        bower cache clean
                        bower install --force
                        ls -la
                        cd ..
                        #gulp build
                        gulp packageCorePlugins
                        npm run collection-plugins
                        npm run build
                        npm run test
                        #cp collection-editor.zip collection-editor.zip:${artifact_version}
                    """
                }
                stage('ArchiveArtifacts') {
                    sh """
                        mkdir collection-editor-artifacts
                        cp collection-editor.zip collection-editor-artifacts
                        zip -j  collection-editor-artifacts.zip:${artifact_version}  collection-editor-artifacts/*                      
                    """
                    archiveArtifacts "collection-editor-artifacts.zip:${artifact_version}"
                    sh """echo {\\"artifact_name\\" : \\"collection-editor-artifacts.zip\\", \\"artifact_version\\" : \\"${artifact_version}\\", \\"node_name\\" : \\"${env.NODE_NAME}\\"} > metadata.json"""
                    archiveArtifacts artifacts: 'metadata.json', onlyIfSuccessful: true
                    currentBuild.description = "${artifact_version}"
                }
        }
    }
    catch (err) {
        currentBuild.result = "FAILURE"
        throw err
    }

}
