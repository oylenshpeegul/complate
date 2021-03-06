# complate

## Introduction and use case

`complate` (a word fusion of "commit" and "template") is a project that allows the user to generate strings in a guided way. The original use-case of this was the standardization of GIT commit messages.\
Many projects and teams are standardizing their commit messages in a certain way. This is somewhat error prone and people just tend to mess things up. Spaces, spelling errors or linebreaks are common issues that lead to inconsistency. It can also have more effects than just consistency in the format. If you use [github-tag-action by anothrnick](https://github.com/anothrNick/github-tag-action) in GitHub Workflows (like this project does), the commit message can have direct influence on your version number that is generated on build.

## Idea

The idea for the concrete use case of standardizing GIT commit messages is to have a configuration file inside the repository which is read by the program. You are then able to select a template that you would like to use for your message. The configuration file declares the template (in handlebars syntax) as well as variables and how to replace them.

## Usage

In order to use `complate`, the recommended way is to place the program including the configuration files and templates into the repository itself. Consider the following structure:
```
Repository root
├── .git
├── .complate
│   ├── complate
│   ├── config.yml
│   └── templates
│       └── template-a.tpl
├── src
│   └── *
└── docs
    └── *
```

`complate` writes the generated message to the stdout pipe.\
Expecting the recommended folder structure, you should be able to simply run `./.complate/complate | git commit -F -` in order to create a new standardized commit.

## General overview

The template itself can be declared as string inside the configuration file or as a reference to a file that contains the template. The template string can contain variables in handlebars syntax ( `{{ variable}}` ). All distinct variables must have a representation in the according section that then also defines on how to find the value for this variable.

## Technical documentation

### CLI

|Name|Short|Long|Description|
|-- |-- |-- |-- |
|Help|-h|--help|Calls the help that displays all the available arguments and commands.|
|Config file path|-c|--config|The path to the configuration file that shall be used. This path can be relative or absolute. The default path is `./complate/config.yml`.|
|Shell trust||--shell-trust|Enables the shell value provider for replacing template placeholders. Due to the potential security risk with this option, it is disabled by default. Possible values for this option are `none` (default), `prompt` and `ultimate`|

### Configuration file

#### Example

Please find an example for the configuration file here:
```
version: 0.4
templates:
    default:
        content:
            inline: |-
                {{ summary }} | {{ version }}
                Components: [{{ components }}]
                Author: {{ author.name }} | {{ author.account }}
                
                Files:
                {{ git.staged.files }}
        values:
            summary:
                prompt: "Enter the summary"
            author.name:
                static: "Reddington, Raymond"
            author.account:
                shell: "whoami"
            git.staged.files:
                shell: "git diff --name-status --cached"
            version:
                select:
                    text: Select the version level that shall be incremented
                    options:
                        - "#patch"
                        - "#minor"
                        - "#major"
            components:
                check:
                    text: Select the components that are affected
                    options:
                        - Security
                        - ValueProvider
                        - CLI
                        - Misc

```
This project also uses complate templates that can be found in `./complate/config.yml`.

#### Value providers

|Type|Description|Fields|
|-- |-- |-- |
|static|Replaces the value by a static string.|-|
|prompt|Asks the user for input when executing the templating process.|text: string|
|shell|Invokes a certain shell command and renders STDOUT as replacing string into the variable. Due to the fact that this option can run arbitrary shell commands, it is disabled by default. Pass the `--shell-trust` flag to the CLI in order to activate this feature.|-|
|select|Gives the user the option to select _one_ item from the provided array of items.|text: string, options: []string|
|check|Gives the user the option to select _n_ items from the provided array of items.|text: string, options: []string|
