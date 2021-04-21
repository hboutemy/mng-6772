[MNG-6772](https://issues.apache.org/jira/browse/MNG-6772) dependencyManagement import repositories
==========

# Testing

```
$ \rm -rf local && mvn -s settings-override.xml validate

[INFO] Scanning for projects...
Downloading from central: file:///home/herve/projets/maven/misc/mng-6772-override-in-project//repo/org/apache/maven/its/mng6772/bom-a/0.1/bom-a-0.1.pom
Downloaded from central: file:///home/herve/projets/maven/misc/mng-6772-override-in-project//repo/org/apache/maven/its/mng6772/bom-a/0.1/bom-a-0.1.pom (1.6 kB at 71 kB/s)
Downloading from repo-bom-a: http://localhost/repo-bom-a/defined-in/bom-a/org/apache/maven/its/mng6772/bom-b/0.1/bom-b-0.1.pom
Downloading from central: http://localhost/central/defined-in/bom-a/org/apache/maven/its/mng6772/bom-b/0.1/bom-b-0.1.pom
[ERROR] [ERROR] Some problems were encountered while processing the POMs:
[ERROR] Non-resolvable import POM: Could not transfer artifact org.apache.maven.its.mng6772:bom-b:pom:0.1 from/to repo-bom-a (http://localhost/repo-bom-a/defined-in/bom-a): transfer failed for http://localhost/repo-bom-a/defined-in/bom-a/org/apache/maven/its/mng6772/bom-b/0.1/bom-b-0.1.pom @ org.apache.maven.its.mng6772:bom-a:0.1, /tmp/mng-6772-override-in-project/local/org/apache/maven/its/mng6772/bom-a/0.1/bom-a-0.1.pom, line 32, column 25
 @ 
[ERROR] The build could not read 1 project -> [Help 1]
[ERROR]   
[ERROR]   The project org.apache.maven.its.mng6772:test:0.1 (/home/herve/projets/maven/misc/mng-6772-override-in-project/pom.xml) has 1 error
[ERROR]     Non-resolvable import POM: Could not transfer artifact org.apache.maven.its.mng6772:bom-b:pom:0.1 from/to repo-bom-a (http://localhost/repo-bom-a/defined-in/bom-a): transfer failed for http://localhost/repo-bom-a/defined-in/bom-a/org/apache/maven/its/mng6772/bom-b/0.1/bom-b-0.1.pom @ org.apache.maven.its.mng6772:bom-a:0.1, /tmp/mng-6772-override-in-project/local/org/apache/maven/its/mng6772/bom-a/0.1/bom-a-0.1.pom, line 32, column 25: Connect to localhost:80 [localhost/127.0.0.1] failed: Connexion refusée (Connection refused) -> [Help 2]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/ProjectBuildingException
[ERROR] [Help 2] http://cwiki.apache.org/confluence/display/MAVEN/UnresolvableModelException
```

# Explanation

The project's [`pom.xml`](pom.xml) overrides `central` repository to point to [`repo/`](repo/) local directory.

That reactor-POM defined repository permits to download [`bom-a:0.1` dependencyManagement import](repo/org/apache/maven/its/mng6772/bom-a/0.1/bom-a-0.1.pom).

That bom-a tries to download [`bom-b:0.1` dependencyManagement import](repo/org/apache/maven/its/mng6772/bom-b/0.1/bom-b-0.1.pom), which fails: `bom-a` has overridden `central` definition.

# Issue

If `bom-a` was a "normal" dependency, overriding `central` would have beed ignored: only reactor's build POM can override repository urls (see [repository order](https://maven.apache.org/guides/mini/guide-multiple-repositories.html#repository-order)).

But `bom-a` is a dependencyManagement import, that is not loaded as part of dependencies resolution (when required by a plugin), but loaded during [effective model building](https://maven.apache.org/ref/3.8.1/maven-model-builder/): the code to resolve such POM is copied from code to resolve parent POM. It seems it applies reactor's build POM resolution rules, when it should probably use dependency resolution rules...
