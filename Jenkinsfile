#!groovy
node {
  stage ('Initialize') {
    echo 'Initialize...'
    checkout scm
    String mvn = sh (script: 'find /var/jenkins_home/tools -name mvn',
                     returnStdout: true).trim()
    if (mvn.length() <= 0) {
      error("mvn is not installed")
    }

    // evosuite
    String evosuiteJar = sh (script: 'find /var/jenkins_home/tools -name evosuite-1.0.3.jar',
                             returnStdout: true).trim()
    if (evosuiteJar.length() <= 0) {
      // install evosuite
      sh 'mkdir -p /var/jenkins_home/tools/evosuite'
      sh 'wget -O /var/jenkins_home/tools/evosuite/evosuite-1.0.3.jar https://github.com/EvoSuite/evosuite/releases/download/v1.0.3/evosuite-1.0.3.jar'
      evosuiteJar = sh (script: 'find /var/jenkins_home/tools -name evosuite-1.0.3.jar',
                        returnStdout: true).trim()
    }

    // evosuite runtime
    String evosuiteRuntimeJar = sh (script: 'find /var/jenkins_home/tools -name evosuite-standalone-runtime-1.0.3.jar',
                                    returnStdout: true).trim()
    if (evosuiteRuntimeJar.length() <= 0) {
      // install evosuite runtime
      sh 'mkdir -p /var/jenkins_home/tools/evosuite'
      sh 'wget -O /var/jenkins_home/tools/evosuite/evosuite-standalone-runtime-1.0.3.jar https://github.com/EvoSuite/evosuite/releases/download/v1.0.3/evosuite-standalone-runtime-1.0.3.jar'
      evosuiteRuntimeJar = sh (script: '/var/jenkins_home/tools -name evosuite-standalone-runtime-1.0.3.jar',
                               returnStdout: true).trim()
    }

    def evosuite = "java -jar /var/jenkins_home/tools/evosuite/evosuite-1.0.3.jar"
    testingEnv = ["mvn=${mvn}",
                  "evosuite=${evosuite}",
                  "evosuiteRuntimeJar=${evosuiteRuntimeJar}"]
  }

  stage ('Build (prev)') {
    echo 'Build (prev)...'
    withEnv(testingEnv) {
      sh "git checkout HEAD~1"
      sh "${mvn} clean compile dependency:copy-dependencies"

      def deps = sh (script: 'ls -1 target/dependency',
                     returnStdout: true).trim().split("\n")
      def cp = "target/classes:target/evosuite-classes:${evosuiteRuntimeJar}:evosuite-tests"
      for (int i = 0; i < deps.size(); i++) {
        cp += ":target/dependency/" + deps[i]
      }
      testingEnv << "classpath=${cp}"
    }
  }

  stage ('TestGen') {
    echo 'TestGen...'
    def changeClasses = ["src.main.java.org.joda.time.Days", "src.main.java.org.joda.time.DateTime"]
    def es_cls_dir = "target/evosuite-classes"
    for (int i = 0; i < changeClasses.size(); i++) {
      def cc = changeClasses.get(i)
      withEnv(testingEnv) {
        sh "${evosuite} -class ${cc} -projectCP target/classes -Dsearch_budget=1"
        sh "mkdir -p ${es_cls_dir}"
        sh "javac -cp ${classpath} -d ${es_cls_dir} evosuite-tests/tutorial/*.java"
      }
    }
  }

  stage ('Build (cur)') {
    echo 'Build (cur)...'
  }

  stage ('TestExe') {
    echo 'TestExe...'
  }
}
