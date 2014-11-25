# parent-pom
A maven parent pom to use for all Main Street Hub projects.  This parent pom
is unique in that it stores build artifacts in Amazon S3 using the aws-maven
extension instead of to a traditional maven repo like Sonatype or Artifactory.

# Releasing

## Initial setup
In order to release this pom to maven central you'll need to do some setup
the very first time.

### Create an oss.sonatype.org account
We'll be releasing to maven central via oss.sonatype.org.  In order to do
this you'll need an account.  You can create one [here](ossrh).

### Create a GPG signing key
One of the requirements for releasing binaries to oss.sonatype.org is that
they are signed with a GPG key.  This ensures that it's more difficult for
a hacker to change binaries should maven central or sonatype ever be
compromised.  Instructions for how to create a GPG key and to publish it
can be found [here](gpg).

### Create a $HOME/.m2/settings.xml file.
Now that you have a oss.sonatype.org account, you'll need to tell maven
your username and password (so that it can upload files).  We'll do this
using a `settings.xml` file in your `$HOME/.m2` directory.  Its contents
should look something like this:

```xml
<settings xmlns="http://maven.apache.org/Settings/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/Settings/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>oss.sonatype.org</id>
      <username>YOUR USERNAME HERE</username>
      <password>YOUR PASSWORD HERE</password>
    </server>
  </servers>
</settings>
```

Don't put these credentials or this file anywhere other than your laptop!!!

## Creating a new release

To release this pom to maven central we're going to follow a slightly
non-standard process that is also somewhat manual.  We need to do this right
now because currently the `pom.xml` file doesn't contain information about
where to deploy it -- it instead contains information for it's children about
how to publish their artifacts to S3.

The first thing we're going to do is to use the maven release:prepare to
manage version numbers and create tags in the source repo.  Then we'll
manually checkout the tag that was created and setup the pom.xml file to
have the correct name.  Next, we'll sign the file with our GPG key and
upload it to oss.sonatype.org.  Finally, we'll login to oss.sonatype.org and
release the file so that it can be published to maven central.

```bash
# Run maven release:prepare to update version numbers, create a scm tag,
# and prepare the next version of the pom.
mvn -B release:prepare

# Set the version number that was just created in the previous command
export VERSION=<VERSION NUMBER HERE>

# Check out the tag for the version we just created
git checkout v${VERSION}

# At this point git will helpfully remind you that you are now in a
# "detatched HEAD" state.  This is normal and completely expected.

# Setup a symbolic link to the pom with the filename we want to upload
# to oss.sonatype.org
ln -s pom.xml parent-pom-${VERSION}.xml

# Sign the pom with our GPG key and publish it to oss.sonatype.org
mvn gpg:sign-and-deploy-file                                          \
  -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ \
  -DrepositoryId=oss.sonatype.org                                     \
  -DpomFile=parent-pom-${VERSION}.xml                                 \
  -Dfile=parent-pom-${VERSION}.xml
```

Now at this point your pom should have been uploaded to oss.sonatype.org
and is waiting to be released.  Log in to the Sonatype console, find
the pom's bundle in the staging repository, close it and then release it.


[ossrh]: http://central.sonatype.org/pages/ossrh-guide.html
[gpg]: http://central.sonatype.org/pages/working-with-pgp-signatures.html