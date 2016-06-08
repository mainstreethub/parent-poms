# parent-poms
This repo contains our *private* maven parent poms.  Our public parent pom is
contained in [mainstreethub/parent-pom][parent-pom].

## ecs-parent-pom
This pom adds the ability to easily build a Docker image from the primary
artifact of a maven build.  This pom encapsulates all of the info necessary to
build as well as publish the Docker image into Main Street Hub's private Docker
registry (hosted in ECR).

**Note:** it is assumed that the primary artifact of the maven build is a single
fat jar that contains all of it's dependencies as well as a manifest specifying
a main class.

[parent-pom]: http://github.com/mainstreethub/parent-pom
