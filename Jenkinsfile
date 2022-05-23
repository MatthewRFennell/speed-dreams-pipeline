pipeline {
	agent any

	stages {
		stage('Checkout') {
			steps {
				retry (5) {
					svn 'svn://svn.code.sf.net/p/speed-dreams/code/trunk'
				}
				dir ("${env.WORKSPACE_TMP}/tests") {
					git changelog: false, poll: false, url: 'https://github.com/MatthewRFennell/speed-dreams-tests.git'
				}
				dir ("${env.WORKSPACE_TMP}/appimagebuilder") {
					git changelog: false, poll: false, url: 'https://github.com/MatthewRFennell/speed-dreams-appimage-builer'
				}
				sh "svn upgrade"
				sh "svn revert -R ."
				sh "svn cleanup . --remove-unversioned --remove-ignored ."
				sh "patch -p0 < ${env.WORKSPACE_TMP}/tests/testing.patch"
			}
		}

		stage('Build with OPTION_CLIENT_SERVER=OFF') {
			steps {
				dir ("build") {
					sh "cmake -D OPTION_OFFICIAL_ONLY=ON -D OPTION_CLIENT_SERVER=OFF -D OPTION_AUTOMATED_TESTING=ON -D CMAKE_BUILD_TYPE=Debug -D CMAKE_INSTALL_PREFIX=. -D CMAKE_PREFIX_PATH=. .."
					sh "cmake -D OPTION_OFFICIAL_ONLY=ON -D OPTION_CLIENT_SERVER=OFF -D OPTION_AUTOMATED_TESTING=ON -D CMAKE_BUILD_TYPE=Debug -D CMAKE_INSTALL_PREFIX=. .."
					sh "make"
				}
			}
		}

		stage('Test with OPTION_CLIENT_SERVER=OFF') {
			steps {
				dir ("build") {
					sh 'ctest'
				}
			}
		}

		stage('Build with OPTION_CLIENT_SERVER=ON') {
			steps {
				dir ("build") {
					sh "cmake -D OPTION_OFFICIAL_ONLY=ON -D OPTION_CLIENT_SERVER=ON -D OPTION_AUTOMATED_TESTING=ON -D CMAKE_BUILD_TYPE=Debug -D CMAKE_INSTALL_PREFIX=. -D CMAKE_PREFIX_PATH=. .."
					sh "cmake -D OPTION_OFFICIAL_ONLY=ON -D OPTION_CLIENT_SERVER=ON -D OPTION_AUTOMATED_TESTING=ON -D CMAKE_BUILD_TYPE=Debug -D CMAKE_INSTALL_PREFIX=. .."
					sh "make"
				}
			}
		}

		stage('Test with OPTION_CLIENT_SERVER=ON') {
			steps {
				dir ("build") {
					sh 'ctest'
				}
			}
		}

		stage('Generate AppImage') {
			steps {
				sh "appimage-builder --recipe ${env.WORKSPACE_TMP}/appimagebuilder/AppImageBuilder.yml"
			}
		}

		stage('Publish AppImage') {
			steps {
				archiveArtifacts artifacts: 'Speed Dreams-jenkins-latest-trunk-x86_64.AppImage', followSymlinks: false, onlyIfSuccessful: true
			}
		}
	}
}
