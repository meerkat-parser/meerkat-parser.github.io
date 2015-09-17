---
---

{% include_relative header.md %}
{% include_relative navigation.md %}

<div markdown="1">
## Download and Install

### From source
Currently the Meerkat library is under active development, and the only way to use it is to build it from source:

1. Install [SBT](http://www.scala-sbt.org/) (You can find the instructions how to get SBT running [here](http://www.scala-sbt.org/release/tutorial/)).
2. Clone the repository: `git clone git@github.com:Anastassija/Meerkat.git`.
3. Go to the Meerkat directory and run `sbt publishLocal`. This command will build the project, package it and puts the binary files in your local repository. 

### Dependent projects
If you want to write a parser using the Meerkat library, add the following dependencies to your SBT file:

```sbt
"org.meerkat" % "meerkat_2.11" % "0.1.0",
"org.bitbucket.inkytonik.dsinfo" %% "dsinfo" % "0.4.0"
```

Note that the dependency to the Meerkat library is resolved via your local repository,
so make sure to build the Meerkat library using `sbt publishLocal`. Also, set the Scala version 
to 2.11 in your SBT file: `scalaVersion := "2.11.6"`	
</div>

{% include_relative footer.md %}