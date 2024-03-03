
In practice, this means that you need to look at the documentation of your build system and test framework and make sure that all folders you want cached are placed under the Codefresh volume. This is a typical pattern with Java applications.

For Maven use mvn -Dmaven.repo.local=/codefresh/volume/m2_repository package as shown in the example.
For Gradle use gradle -g /codefresh/volume/.gradle -Dmaven.repo.local=/codefresh/volume/m2 as explained in the example.
For SBT use -Dsbt.ivy.home=/codefresh/volume/ivy_cache.
For Pip use pip install -r requirements.txt --cache-dir=/codefresh/volume/pip-cache as shown in the example
For Golang pass an environment variable GOPATH=/codefresh/volume/go to the freestyle step that is running go commands
For Rust pass an environment variable CARGO_HOME=/codefresh/volume/cargo to the freestyle step that is running rust/cargo commands
