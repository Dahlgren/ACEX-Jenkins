pipeline {
  agent {
    label 'mikero'
  }

  environment {
    PYTHONUNBUFFERED = '1'
  }

  stages {
    stage('Checkout') {
      steps {
        dir('x/cba') {
          git url: 'https://github.com/CBATeam/CBA_A3.git', branch: 'master', changelog: false, poll: false
        }

        dir('z/ace') {
          git url: 'https://github.com/acemod/ACE3.git', branch: 'master', changelog: false, poll: false
        }

        dir('z/acex') {
          script {
            def acexGit = git url: 'https://github.com/acemod/ACEX.git', branch: 'master', changelog: true, poll: true
            env.ACEX_COMMIT = acexGit.GIT_COMMIT
          }

          // Set bad exit code on error
          powershell '((Get-Content -path tools/make.py -Raw) -replace \'sys.exit\\(0\\)\', \'sys.exit(len(failedBuilds))\') | Set-Content -Path tools/make.py'
        }
      }
    }

    stage('Python') {
      steps {
        bat 'curl https://www.python.org/ftp/python/3.7.4/python-3.7.4-embed-win32.zip --output python3.zip'
        powershell 'Expand-Archive -Path python3.zip -DestinationPath python3'
      }
    }

    stage('Arma Data') {
      steps {
        // Link Arma 3 Data
        bat 'mklink /j a3 %A3_DATA%\\a3'
      }
    }

    stage('Build') {
      steps {
        // Set correct excludes for ACE in pboproject
        bat 'regedit /S %WORKSPACE%\\pboproject_acex.reg'

        // Mount P: drive
        bat 'subst P: .'

        // Build ACE with CI exit status and external files check enabled
        bat 'python3\\python.exe z/acex/tools/make.py ci'

        // Move built mod to root of workspace
        bat 'move z/acex/release/@acex @acex'
      }
    }

    stage('Steam Workshop') {
      steps {
        publishSteamWorkshop '2010872605', '@acex', "https://github.com/acemod/ACEX/commit/${env.ACEX_COMMIT}"
      }
    }
  }

  post { 
    always {
      // Archive built mod on success
      archiveArtifacts allowEmptyArchive: true, artifacts: '@acex/**/*'

      // Archive pboproject log files
      archiveArtifacts allowEmptyArchive: true, artifacts: 'temp/*.log'

      // Cleanup workspace to avoid wasting disk space
      deleteDir()

      // Dismount P: drive
      bat 'subst P: /D'
    }
  }
}

void publishSteamWorkshop(String id, String mod, String changeNote) {
  bat "\"C:\\Program Files (x86)\\Steam\\SteamApps\\common\\Arma 3 Tools\\Publisher\\PublisherCmd.exe\" update /changeNote:$changeNote /id:$id /path:$mod"
}
