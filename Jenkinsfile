node("java:8") {
  def mvn = tool("maven") + "/bin/mvn -B"

  checkout scm

  stage("Compile") {
    sh "${mvn} -f dropwizard-parent-pom/pom.xml clean compile test-compile"
  }

  stage("Test") {
    sh "${mvn} -f dropwizard-parent-pom/pom.xml verify"
  }

  stage("Package") {
    sh "${mvn} -f dropwizard-parent-pom/pom.xml -Dskip.docker.image.build=false -Dmaven.test.skip=true clean package"
  }
}

stage("Release") {
  input 'Release to Sonatype?'

  node("java:8") {
    def git = tool("git")
    def mvn = tool("maven") + "/bin/mvn -B"

    checkout scm

    sh "${git} config user.email engineering+jenkins2@mainstreethub.com"
    sh "${git} config user.name jenkins"
    withCredentials([[$class: "UsernamePasswordMultiBinding",
                      credentialsId: "github-http",
                      usernameVariable: "GIT_USERNAME",
                      passwordVariable: "GIT_PASSWORD"]]) {
      def username = URLEncoder.encode("${env.GIT_USERNAME}", "UTF-8")
      def password = URLEncoder.encode("${env.GIT_PASSWORD}", "UTF-8")

      writeFile(file: "${env.HOME}/.git-credentials",
          text: "https://${username}:${password}@github.com")
      sh "${git} config credential.helper store --file=${env.HOME}/.git-credentials"
    }

    sh "mkdir ${env.HOME}/.gnupg"
    sh "chmod 0700 ${env.HOME}/.gnupg"

    // Write the gpg.conf to the container
    writeFile(file: "${env.HOME}/.gnupg/gpg.conf",
        text: "keyserver hkp://keys.gnupg.net")

    withCredentials([file(credentialsId: 'pubring.gpg', variable: 'TOKEN')]) {
      sh "cp ${TOKEN} ${env.HOME}/.gnupg/pubring.gpg"
      sh "chmod 0600 ${env.HOME}/.gnupg/pubring.gpg"
    }

    withCredentials([file(credentialsId: 'secring.gpg', variable: 'TOKEN')]) {
      sh "cp ${TOKEN} ${env.HOME}/.gnupg/secring.gpg"
      sh "chmod 0600 ${env.HOME}/.gnupg/secring.gpg"
    }

    withCredentials([file(credentialsId: 'trustdb.gpg', variable: 'TOKEN')]) {
      sh "cp ${TOKEN} ${env.HOME}/.gnupg/trustdb.gpg"
      sh "chmod 0600 ${env.HOME}/.gnupg/trustdb.gpg"
    }

    withCredentials([[$class: "UsernamePasswordMultiBinding",
                      credentialsId: "oss.sonatype.org",
                      usernameVariable: "USERNAME",
                      passwordVariable: "PASSWORD"]]) {
      def username = URLEncoder.encode("${env.USERNAME}", "UTF-8")
      def password = URLEncoder.encode("${env.PASSWORD}", "UTF-8")

      def text = "<settings xmlns=\"http://maven.apache.org/Settings/1.0.0\"\n" +
          "          xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n" +
          "          xsi:schemaLocation=\"http://maven.apache.org/Settings/1.0.0\n" +
          "                      http://maven.apache.org/xsd/settings-1.0.0.xsd\">\n" +
          "  <servers>\n" +
          "    <server>\n" +
          "      <id>oss.sonatype.org</id>\n" +
          "      <username>${username}</username>\n" +
          "      <password>${password}</password>\n" +
          "    </server>\n" +
          "  </servers>\n" +
          "</settings>"

      writeFile(file: ".m2/settings.xml",
          text: text)
    }

    sh "${mvn} -f dropwizard-parent-pom/pom.xml -Popen-source -Dresume=false -Dmaven.javadoc.skip=true -Darguments='-Popen-source -DskipTests=true -DskipITs=true -Dmaven.javadoc.skip=true' release:clean release:prepare release:perform"
  }
}
