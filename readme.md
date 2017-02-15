# parent-poms
This repo contains our maven parent poms.

## parent-pom
A maven parent pom to use for all Main Street Hub projects.  This parent pom
is unique in that it stores build artifacts in Amazon S3 using the aws-maven
extension instead of to a traditional maven repo like Sonatype or Artifactory.

## ecs-parent-pom
This pom adds the ability to easily build a Docker image from the primary
artifact of a maven build.  This pom encapsulates all of the info necessary to
build as well as publish the Docker image into Main Street Hub's private Docker
registry (hosted in ECR).

**Note:** it is assumed that the primary artifact of the maven build is a single
fat jar that contains all of it's dependencies as well as a manifest specifying
a main class.

### Releases

#### 1.1.1
* Updated label `app` to `application`
* Updated label `app_version` to `version`

#### 1.1.0

* Updated base image from docker.io/java to docker.io/openjdk.
* Made base image configurable for image build.
* Added labels to Docker image for application (app) and application version (app_version)
* Upgraded docker-maven-plugin to version 0.18.1

## dropwizard-parent-pom
This pom adds the ability to easily build a Dropwizard application as a Docker
image.  The image is automatically configured to expose ports 8080 and 8081 to
the container host so that the web and admin ports can both be used.  In
addition, the arguments to the application command are set to
`server classpath://config.yaml`.  If necessary this can be overridden in a
child pom.

### Releases

#### 1.0.5-1 (Not released)

* Upgraded to Dropwizard 1.0.5
* Upgraded dropwizard-metrics-datadog-ecs to 0.4.0
* Added environment library, version 0.3.0
* Added time library, version 0.2.0
* Added dropwizard-config-testing library, version 0.1.0

#### 1.0.2-4

* Upgraded parent pom to ecs-parent-pom version 1.1.1.
* Added dropwizard-json-log-bundle 1.0.2-0
* Rename log-format Docker label to archetype

#### 1.0.2-3

* Upgraded parent pom to ecs-parent-pom version 1.1.0.

## dropwizard-scala-parent-pom
This pom wraps the dropwizard-parent-pom for development in Scala

### Releases

#### 1.0.3-1
* Moved non-essential dependencies out

#### 1.0.2-1
* Initial release

## datateam-dropwizard-scala-parent-pom

### Releases

#### 1.0.0-1
* Initial release
