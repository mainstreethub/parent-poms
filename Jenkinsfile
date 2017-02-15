node("java:8"){
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

  // Write the gpg.conf to the container
  def gpg = "keyserver hkp://keys.gnupg.net"
  writeFile(file: "/home/jenkins/.gnupg/gpg.conf",
      text: gpg)


  withCredentials([file(credentialsId: 'pubring.gpg', variable: 'TOKEN')]) {
    sh "cp ${TOKEN} /home/jenkins/.gnupg/pubring.gpg"
  }

  withCredentials([file(credentialsId: 'secring.gpg', variable: 'TOKEN')]) {
    sh "cp ${TOKEN} /home/jenkins/.gnupg/secring.gpg"
  }

  withCredentials([file(credentialsId: 'trustdb.gpg', variable: 'TOKEN')]) {
    sh "cp ${TOKEN} /home/jenkins/.gnupg/trustdb.gpg"
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

  stage("Compile") {
    sh "${mvn} -f dropwizard-parent-pom/pom.xml -Popen-source -Dresume=false -DdryRun=true -Dmaven.javadoc.skip=true -Darguments=\"-Popen-source -DskipTests=true -DskipITs=true -Dmaven.javadoc.skip=true\" release:clean release:prepare"
  }

  stage("Test") {
    sh "${mvn} -Popen-source -Dresume=false -Dmaven.javadoc.skip=true -Darguments=\"-Popen-source -DskipTests=true -DskipITs=true -Dmaven.javadoc.skip=true\" release:clean release:prepare release:perform"
  }

}
