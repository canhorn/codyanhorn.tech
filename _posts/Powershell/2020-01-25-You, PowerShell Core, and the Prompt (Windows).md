---
layout: post
author: Cody Merritt Anhorn
title: You, PowerShell Core, and the Prompt (Windows)
excerpt_separator: <!--more-->
---

We will go over how to create a new prompt display for PowerShell Core (Windows).

<!--more-->

## How To 

First you will need to navigate to your PowerShell directory, by default mine was located in the Documents folder of my Users profile, ~\Documents\PowerShell.

Create a new file called Microsoft.PowerShell_profile.ps1, this is the filename used for the default profile file that will automatically get pulled in when PowerShell Core gets loaded.

To override/change the prompt you just need to create a function prompt, with a return of what you want your shiny new prompt to be. Below is the my personal prompt code snippet, and here is an example image.

![An image showing off example of prompt override.](/image/Posts/Powershell/2020-01-25/Prompt-override-example.png)

## Examples

***My Personal Prompt, requires git:***

~~~ powershell
# Override the prompt
function prompt {
    $base = "PS "
    $path = "$($executionContext.SessionState.Path.CurrentLocation)"
    $userPrompt = "$('>' * ($nestedPromptLevel + 1)) "

    Write-Host "`n$base" -NoNewline

    $symbolicRefHead = git symbolic-ref HEAD

    if ($NULL -ne $symbolicRefHead) {
        Write-Host $path -NoNewline -ForegroundColor "green"
        Write-BranchName
    }
    else {
        # we're not in a repo so don't bother displaying branch name/sha
        Write-Host $path -ForegroundColor "green"
    }

    return $userPrompt
}


# Git Prompt Display
function Write-BranchName () {
    try {
        $branch = git rev-parse --abbrev-ref HEAD

        if ($branch -eq "HEAD") {
            # we're probably in detached HEAD state, so print the SHA
            $branch = git rev-parse --short HEAD
            Write-Host " ($branch)" -ForegroundColor "red"
        }
        else {
            # we're on an actual branch, so print it
            Write-Host " ($branch)" -ForegroundColor "blue"
        }
    }
    catch {
        # we'll end up here if we're in a newly initiated git repo
        Write-Host " (no branches yet)" -ForegroundColor "yellow"
    }
}

~~~

***The Default Prompt:***

~~~ powershell
function prompt {
    "PS $($ExecutionContext.SessionState.Path.CurrentLocation)$('>' * ($nestedPromptLevel + 1)) "
}
~~~

***Reference Links:***

***--*** https://stackoverflow.com/questions/1287718/how-can-i-display-my-current-git-branch-name-in-my-powershell-prompt