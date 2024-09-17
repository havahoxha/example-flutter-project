pipeline {
    agent any
    
    environment {
        FLUTTER_VERSION = '3.24.3' // Specify the Flutter version you want to install
    }

    stages {
        stage('Install Dependencies') {
            steps {
                sh '''
                # Install apt dependencies
                apt-get update
                apt-get install -y wget xz-utils clang cmake ninja-build pkg-config libgtk-3-dev

                # Install Flutter SDK
                wget -q https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_${FLUTTER_VERSION}-stable.tar.xz
                tar xf flutter_linux_${FLUTTER_VERSION}-stable.tar.xz
                export PATH="$PATH:$WORKSPACE/flutter/bin"
                flutter doctor
                '''
            }
        }

        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/havahoxha/example-flutter-project.git'
            }
        }

        stage('Increase Patch Version') {
            steps {
                script {
                    def mainDartPath = 'lib/main.dart'
                    echo "Reading ${mainDartPath}"
                    def content = readFile(mainDartPath)
                    
                    // Find APP_VERSION in main.dart
                    echo "Finding APP_VERSION in ${mainDartPath}"
                    def versionPattern = /APP_VERSION\s*=\s*['"](\d+)\.(\d+)\.(\d+)['"]/
                    def matcher = content =~ versionPattern

                    if (matcher) {
                        echo "Found APP_VERSION in ${mainDartPath}"
                        def major = matcher[0][1].toInteger()
                        def minor = matcher[0][2].toInteger()
                        def patch = matcher[0][3].toInteger() + 1

                        def newVersion = "${major}.${minor}.${patch}"

                        // Replace the APP_VERSION line with updated patch version
                        echo "Updating APP_VERSION to ${newVersion}"

                        def updatedContent = content.replaceFirst(versionPattern, "APP_VERSION = '${newVersion}'")
                        echo "Updated content of ${mainDartPath}"

                        // Write updated content back to the file
                        writeFile(file: mainDartPath, text: updatedContent)
                        echo "Applying content to ${mainDartPath}"

                        echo "Updated APP_VERSION to ${newVersion}"
                    } else {
                        error "APP_VERSION not found in ${mainDartPath}"
                    }
                }
            }
        }

        stage('Build') {
            steps {
                sh '''
                export PATH="$PATH:$WORKSPACE/flutter/bin"
                flutter pub get
                flutter build linux
                '''
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: 'build/app/outputs/flutter-linux/*', allowEmptyArchive: true
        }
        success {
            echo 'Build completed successfully!'
        }
        failure {
            echo 'Build failed.'
        }
    }
}
