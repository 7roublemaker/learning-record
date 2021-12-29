# 安装新版本vscode

# 下载CodeQL in GitHub

[https://github.com/github/codeql-cli-binaries/releases/](https://github.com/github/codeql-cli-binaries/releases/download/v2.5.0/codeql.zip)

# 配置环境变量

# 官方模板

`git clone git@github.com:github/codeql.git codeql-repo`

# 验证codeql可用性

`codeql resolve languages`

`codeql resolve qlpacks`

# 创建数据库

`codeql database create <database> --language=<language-identifier>`

## 官网给的示例：

```
$ # Clone the project
  $ git clone https://github.com/m-y-mo/struts_9805

$ # Create a CodeQL database
  $ codeql database create ./struts_db -s ./struts_9805 \
    -j 0 -l java --command "mvn -B -DskipTests \
    -DskipAssembly"
```

# codeql参数解析

- <database>: a path to the new database to be created. This directory will be created when you execute the command—you cannot specify an existing directory.

- --language: the identifier for the language to create a database for. When used with --db-cluster, the option accepts a comma-separated list, or can be specified more than once. CodeQL supports creating databases for the following languages:

|Language|Identifier|
|-|-|
|C/C++|cpp|
|C#|csharp|
|Go|go|
|Java|java|
|JavaScript/TypeScript|javascript|
|Python|python|
|Ruby|ruby|

- --source-root: the root folder for the primary source files used in database creation. By default, the command assumes that the current directory is the source root—use this option to specify a different location.
- --db-cluster: use for multi-language codebases when you want to create databases for more than one language.
- --command: used when you create a database for one or more compiled languages, omit if the only languages requested are Python and JavaScript. This specifies the build commands needed to invoke the compiler. Commands are run from the current folder, or --source-root if specified. If you don’t include a --command, CodeQL will attempt to detect the build system automatically, using a built-in autobuilder.
- --no-run-unnecessary-builds: used with --db-cluster to suppress the build command for languages where the CodeQL CLI does not need to monitor the build (for example, Python and JavaScript/TypeScript).

# codeql创建各种语言数据库的示例
##Creating databases for non-compiled languages
### JavaScript and TypeScript

`codeql database create --language=javascript --source-root <folder-to-extract> \`
`<output-folder>/javascript-database`

### Python

`codeql database create --language=python <output-folder>/python-database`

### Ruby

`codeql database create --language=ruby --source-root <folder-to-extract> \`
`<output-folder>/ruby-database`

## Creating databases for compiled languages
### Detecting the build system
自动识别编译的语言，叫做啥autobuilder的一个功能，不需要使用`--command`进行编译

`codeql database create --language=java <output-folder>/java-database`

### C/C++ project built using make:

`codeql database create cpp-database --language=cpp --command=make`

### C# project built using dotnet build (.NET Core 3.0 or later):

`codeql database create csharp-database --language=csharp \`
`--command='dotnet build /t:rebuild'`

### On Linux and macOS (but not Windows), you need to disable shared compilation when building C# projects with .NET Core 2 or earlier, so expand the command to:

`codeql database create csharp-database --language=csharp \`
`--command='dotnet build /p:UseSharedCompilation=false /t:rebuild'`

### Go project built using the COEQL_EXTRACTOR_GO_BUILD_TRACING=on environment variable:

`CODEQL_EXTRACTOR_GO_BUILD_TRACING=on codeql database create go-database --language=go`

### Go project built using a custom build script:

`codeql database create go-database --language=go --command='./scripts/build.sh'`

### Java project built using Gradle:

`codeql database create java-database --language=java --command='gradle clean test'`

### Java project built using Maven:

`codeql database create java-database --language=java --command='mvn clean install'`

### Java project built using Ant:

`codeql database create java-database --language=java --command='ant -f build.xml'`

### Project built using Bazel:

```
# Navigate to the Bazel workspace.

# Before building, remove cached objects
# and stop all running Bazel server processes.
bazel clean --expunge

# Build using the following Bazel flags, to help CodeQL detect the build:
# `--spawn_strategy=local`: build locally, instead of using a distributed build
# `--nouse_action_cache`: turn off build caching, which might prevent recompilation of source code
# `--noremote_accept_cached`, `--noremote_upload_local_results`: avoid using a remote cache
codeql database create new-database --language=<language> \
  --command='bazel build --spawn_strategy=local --nouse_action_cache --noremote_accept_cached --noremote_upload_local_results //path/to/package:target'

# After building, stop all running Bazel server processes.
# This ensures future build commands start in a clean Bazel server process
# without CodeQL attached.
bazel shutdown
```

### Project built using a custom build script:

`codeql database create new-database --language=<language> --command='./scripts/build.sh'`