# Agent Instructions

## Architecture
### Dependency Management
Where there are dependencies initialized, be sure to use an initializer which creates a new dependency with a series of sane
defaults. Where there are requirements have them as normal, typed arguments but where there are "optional" arguments, allow
overriding them with the "variadic argument" style used in Uber, or popularized by Dave Cheany.

### Layout
Where an application starts to become more complex (for example, includes clients, a datastore, keeps state or otherwise 
non-trivial code), prefer to organize the application in the [hexagonal architecture] design popularized by alister cockburn.

[hexagonal architecture]: https://alistair.cockburn.us/hexagonal-architecture

## CI
As a general rule, try and use GitHub actions for CI related tasks, as this repository is stored and used primarily in GitHub.

## CLI
### Documentation

Where an application has a CLI, it should be easily possible to either view directly or generate a comprehensive `man`
page with clear examples of how to use this application.

Additionally, the application commands and subcommands must clearly describe what each command does via the `--help`
flag, ideally with examples.

### Interface

Where there is a CLI, there should be top level commands that categorize and then commands that take an imperative
action on the category, similar to how the Kubernetes CLI is structured. For example, a project that sends
announcements might have the structure:

```bash
./announce slack post --channel "#foo"
```

## Configuration
### Consumption

Where we have configuration parameters, they should be accessible via:

* Command Line Flag
* Environment Variable (with a prefix)
* File

The library [spf13/viper] does this especially well. A concrete example might be for an API key for Slack ("abcde"), and the application
might be called "Announce". The configuration should be suppliable via environment variabl

```bash
ANNOUNCE_SLACK_API_KEY="abcde" ./announce
```

Command line flag:

```bash
# Use dot notation to match what might exist in a file.
./announce --slack.api.key="abcde"
```

And File:

```yaml
# Prefer child objects rather than using a underscore to specify configuration values.
slack:
  api:
    key: abcde
```

[spf13/viper]: https://github.com/spf13/viper

### Path

Where applications have configuration, they should store that configuration in a path that follows the XDG standards. The
Golang library from [ardg/xdg] is a particularly excellent implementation finding or writing to those files.

[ardg/xdg]: https://github.com/ardg/xdg

## Git
### Staging Changes

If possible, stage changes by reviewing specific changes applied in files rather than staging files directly. In
practice, use:

```bash
git add --patch ./path/to/file
```

Rather than just `git add ./path to file`.

### Amending Commits

Where there are multiple commits required for work, you can either:

* **Create unique, well crafted commit messages for each line**; this is the ideal as it will give context to your 
  decisions at multiple stages of this work. 
* **Squash all commits together and summarize the work in one commit**; this is also fine as it will create a well
  structured git history.

Just don't create multiple "repeated commits" on a single branch. When you amend the commit, you'll need to
force push it with:

```
git push --force-with-lease
```

### Commit Styles

In general, commits should follow the styleguide set out in "[What constitutes a 'Good Commit']".

#### Title

For the "title" of a commit it should be maximally 72 characters long, be descriptive and be
expressed as though the change is being applied. The commit should be useful to view with commands such as:

```
$ git log --pretty=oneline
```

Some examples include:

* Deploy changes automatically on updates to the main branch
* Update the port (8083 → 9093) used for Prometheus connections
* Introduce the widget to handle image creation

[What constitutes a 'Good Commit']: https://www.andrewhowden.com/p/anatomy-of-a-good-commit-message

#### Body

The body should include primarily the justification for the changes, rather than a description of the changes themselves.
This allows understanding why this change was made by either humans or large language models when the change is reviewed
later with `git blame` or `git log --patch`.

It should be written as a paragraph, also breaking at 72 characters long.

#### Coauthor

Where you create commits, be sure to mark yourself as a co-author of the changes, if not the primary author. The syntax for
doing so is:

```

Co-authored-by: NAME <NAME@EXAMPLE.COM>
Co-authored-by: ANOTHER-NAME <ANOTHER-NAME@EXAMPLE.COM>"
```

Complete with the line break before the "Co-authored-by" key/value pair. The format follows RFC 5322 for defining the
display name / email pair.

## Task Runner

For common tasks within the project (for example, running tests, builds etc) then please use [Taskfile] as the task 
runner of choice. There are usually a few tasks I prefer to create

1. `build`: Runs the compilation of the binary. 
2. `test`: Runs all unit tests for the binary
3. `lint`: Runs any lints that exist for the code in the binary
4. `validate`: A combination of test, lint and other pre-checks that are useful to catch issues prior to commit

[Taskfile]: https://taskfile.dev/

## Languages
### Go
#### General Advice
For writing Go code, ensure that you follow out the best practices described by the [effective go] documentation, 
and prefer convententions from the standard library wherever possible.

Where I want to release an application to GitHub, prefer to use the [open source goreleaser] to run these releases.
Do a cross compilation build, and upload binaries suitable for the common platforms.

[effective go]: https://go.dev/doc/effective_go
[the open source goreleaser]: https://goreleaser.com/

#### Error Handling

For packages that are reused in the application (e.g. anything in `internal`), it is better to "wrap" errors that 
are returned upstream so that unit tests, and if necessary, business logic, can examine them. For example, 
instead of code that does:

```go
func DoSomething() (string, error) {

	// Assume this error was returned from a client
	return "", fmt.Errorf("upstream error")
}
```

Define a new error value at the top of the package, and use that to represent this error. For example, 

```go
// Err* are common errors
var (
	ErrUpstreamFailure = errors.New("dependency failed")
)

func DoSomething() (string, error) {

	// Assume this error was returned from a client
	return "", fmt.Errorf("%w: %s", ErrUpstreamFailure, fmt.Errorf("upstream error"))
}
```
### Markdown

For Markdown documents, please break the documents at 120 characters.

## Tests

For just about all changes, make sure you develop via "test driven design". This means:

1. Write tests for the current behavior of the application
2. Modify those tests so that they verify the new desired behavior of the application
3. Modify the logic of the application based on your request, so it validates against those tests.
4. Adjust either the application or the tests until the tests pass
5. Publish the change.

## Dependency Injection

Where possible and following the guidance of the library, use [Google Wire] as compile time dependency injection 
for the Go program.

[Google Wire]: https://github.com/google/wire

## Tools

Make sure you have any tools required to build, compile or otherwise maintain the application by reviewing the
file called README.md for instructions on the installation of any tools.

Common tools include things like:

* git
* [task]

[task]: https://taskfile.dev/docs/installation
