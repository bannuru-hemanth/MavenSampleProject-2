@Library('SomeRandomLib')
import com.mycompany.jenkins.Release
import com.mycompany.jenkins.Utilities

def utils = new Utilities(steps, env, currentBuild)
def release = new Release(steps, env, currentBuild)

echo "Executing build on branch ${env.BRANCH_NAME}"

//def gitRepo = 'asm_test'

node {
	try {
		
		stage(name: "Initializing ${env.BRANCH_NAME}") {
			deleteDir();
			checkout scm;
		}

		stage('Build/Test') {
			utils.mvn "clean install"
		}

		stage('QA Javadoc') {
			//utils.mvn "javadoc:javadoc"
		}

		stage('QA Sonar') {
			withSonarQubeEnv('My SonarQube Server') {
                //utils.sonar()
				bat 'mvn clean package sonar:sonar' //Prefer to use utils.sonar()
            }
		}

		stage("Quality Gate") {
            timeout(time: 1, unit: 'HOURS') {
                def qg = waitForQualityGate()
                if (qg.status != 'OK') {
                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
                else {
                    echo "SUCCES! SonarQube status is: ${qg.status}"
                }
            }
        }

	} catch (error) {
		stage('Mail') {
			echo "Something went wrong, see ${env.BUILD_URL}"
/* 			def to = emailextrecipients([
							[$class: 'CulpritsRecipientProvider'],
							[$class: 'DevelopersRecipientProvider'],
							[$class: 'RequesterRecipientProvider']
			])
			utils.mailTo "Build has failed!!!", "See ${env.BUILD_URL}", "${to}" */
		}
		throw error;
	}
}

if (env.BRANCH_NAME == 'develop') {
	node {
		stage("Prepare dependency management") {
			utils.mvn "clean install -DskipTests"
		}

		stage("QA Dependency Updates") {
			dependencyUpdates utils
		}

		stage("QA Dependency Security") {
			dependencySecurity utils
		}
	}
}

if (env.BRANCH_NAME == 'master') {
	//BRANCH_NAME zou release moeten zijn?
	def releaseNumber
	def releaseTag
	def developmentVersion

	node {
		stage("Prepare for release") {
			echo "Testing this feature"
		}

		stage("Create release") {
			echo "Creating release"
			utils.mvn 'clean install'
		}

		stage("Deploy to nexus") {
			//bat 'mvn clean deploy'
			//releaseNumber = release.get_version_from_pom "pom.xml"
			//release.updatePomWithVersion(releaseNumber)

			//release.deploy(releaseNumber, "snapshots")
			//release.deploy(releaseNumber, "SampleNexusProject")
		}
	}
}