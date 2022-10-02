The current official version of JUnit 4 is junit-4.13.2.jar; released on Feb.13, 2021.
Unfortunately for Daikon users, this release version contains Java 5 bytecodes.
When a program that uses JUnit4 is processed by DynComp it produces lots of warnings
about the use of old bytes codes. More importantly, the Java 5 bytecodes generated for
try finally statements confuse the BCEL bytecode verifier and it reports a fatal error.

This version has been changed to have -target 1.8.  It also has a few type annotations for
the Checker Framework.

As Junit4 is in maintenance mode, it is unlikely there will ever be a need
to update to a newer version of the upstream library.


To build this project
---------------------

Change `pom.xml` to use the latest Checker Framework version.

```
mvn -B -Drat.skip=true clean package
```

There will be pluggable type-checking errors, because only the signatures, not
the bodies, of methods are annotated. However, all tests should pass:

Tests run: 1108, Failures: 0, Errors: 0, Skipped: 5


This will create the file:
`target/junit-4.13.2-Daikon.jar`

Test it in a branch of Daikon:
 * copy the target/junit-VERSION.jar file to the `daikon/java/lib` directory
 * remove the old junit-OLDVERSION.jar file
 * in Daikon branch run: make compile daikon.jar dyncomp-jdk
 * run:  make MPARG=-j1 -C tests clean diffs
   If there are any errors, then fix the bugs in junit and/or Daikon.
 * push your branch, and ensure that the the Azure Pipelines tests pass
 * merge your branch into master

Upload junit to Maven Central (instructions appear below).


To make changes between upstream releases
-----------------------------------------

If you need to release a new version of junit between upstream releases (for
example, because of a bug fix or because of added annotations), use the
version number policy explained at section "Version numbers for annotated
libraries" in
https://rawgit.com/typetools/checker-framework/master/docs/developer/developer-manual.html#annotated-library-version-numbers
.


To upload to Maven Central
--------------------------

This must be done on a CSE machine, which has access to the necessary passwords.

# Set the version number:
#  * in file cfMavenCentral.xml
#  * in file pom.xml (if different from upstream)
#  * environment variable PACKAGE below

# JAVA_HOME must be a JDK 8 JDK.

PACKAGE=junit-4.13.2-Daikon && \
mvn clean verify && \
mvn javadoc:javadoc && (cd target/site/apidocs && jar -cf ${PACKAGE}-javadoc.jar org)

## This does not seem to work for me:
# -Dhomedir=/projects/swlab1/checker-framework/hosting-info

mvn gpg:sign-and-deploy-file -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ -DrepositoryId=sonatype-nexus-staging -DpomFile=cfMavenCentral.xml -Dgpg.publicKeyring=/projects/swlab1/checker-framework/hosting-info/pubring.gpg -Dgpg.secretKeyring=/projects/swlab1/checker-framework/hosting-info/secring.gpg -Dgpg.keyname=ADF4D638 -Dgpg.passphrase="`cat /projects/swlab1/checker-framework/hosting-info/release-private.password`" -Dfile=target/${PACKAGE}.jar \
&& \
mvn gpg:sign-and-deploy-file -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ -DrepositoryId=sonatype-nexus-staging -DpomFile=cfMavenCentral.xml -Dgpg.publicKeyring=/projects/swlab1/checker-framework/hosting-info/pubring.gpg -Dgpg.secretKeyring=/projects/swlab1/checker-framework/hosting-info/secring.gpg -Dgpg.keyname=ADF4D638 -Dgpg.passphrase="`cat /projects/swlab1/checker-framework/hosting-info/release-private.password`" -Dfile=target/${PACKAGE}-sources.jar -Dclassifier=sources \
&& \
mvn gpg:sign-and-deploy-file -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ -DrepositoryId=sonatype-nexus-staging -DpomFile=cfMavenCentral.xml -Dgpg.publicKeyring=/projects/swlab1/checker-framework/hosting-info/pubring.gpg -Dgpg.secretKeyring=/projects/swlab1/checker-framework/hosting-info/secring.gpg -Dgpg.keyname=ADF4D638 -Dgpg.passphrase="`cat /projects/swlab1/checker-framework/hosting-info/release-private.password`" -Dfile=target/site/apidocs/${PACKAGE}-javadoc.jar -Dclassifier=javadoc

# Complete the release at https://oss.sonatype.org/#stagingRepositories :
#  * Search for a repository named orgcheckerframework-NNNN (NNNN are digits)
#  * Click on it
#  * Click "close" at the top.
#  * Click "refresh" at the top until the bottom pane has "Repositery closed"
#  * Click "release" at the top (make sure the "automatically drop" box is checked)


To use a locally-built version of the Checker Framework
-------------------------------------------------------
Go to your checker-framework enlistment and (after building) run:

./gradlew publishToMavenLocal

Then run:

./gradlew version

Now, go back to the project that uses the Checker Framework and
run mvn with your normal arguments but add:

-DcheckerFrameworkVersion=<result of the version command above>

For example:

-DcheckerFrameworkVersion=3.21.4-SNAPSHOT
