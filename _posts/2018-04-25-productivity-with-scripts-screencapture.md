---
layout: post
title:  "Increasing Productivity with Scripts: Utilizing Mac's screencapture Utility"
date:   2018-04-25 12:00:00 -0400
categories: automation scripts
author: Scott Davidson
---

I am a firm believer that there is almost always room for improvement in process, especially when it comes to automating tasks. As an Automation Engineer here at Forward Financing, I am always trying to ensure optimal productivity for my team. However, I am also constantly looking for improvements that I can make within my personal process to help me save time throughout the day.

A great way to accomplish this is by first identifying some cumbersome task, and then writing a script for it. Currently, I have many useful scripts on both my work and home computers to accomplish various things. These scripts may not be useful to everyone, but they help me accomplish tasks in a fraction of the time, and end up saving me minutes or hours in my week. Recently, I encountered an arduous daily process in my workflow that was taking me too much time and wrote a script to help me solve this problem.

**Note: The specific example showcased here assumes that you are on macOS**

# Identifying the Problem

Since I am constantly either QAing or communicating with engineers about features, I like to provide as much context as possible. I found that visual aides are very helpful at providing additional context surrounding an issue.

I primarily use a MacBook. Luckily, macOS provides users the ability to take screenshots. There are a few different ways to do this [outlined here by Apple support](https://support.apple.com/en-us/HT201361). Since I usually only want a portion of the screen, I prefer the shortcut: `command` + `shift` + `4` to select a capture area with my mouse.

I would take a screenshot, find it in my filesystem, open it, edit it, change the name, and then attach it to a message after finding it again in my filesystem. This required a lot of manual steps, and took a decent amount of time. I looked into ways of changing the default name and location of screenshots, but I wanted something a little more custom. After some digging, I found out that macOS has the `screencapture` utility that can be used in `Terminal`. (In your terminal, type `man screencapture` for more information). As soon as I discovered this, I immediately wanted to write a script so I could shave time off this arduous process.

# Investigation Time

After learning of the `screencapture` utility, I began to explore its docs and try it out. I first identified what I wanted to accomplish with my script, and then looked for corresponding flags or variations on the command that would support the behavior I wanted.

I knew I wanted to specify a name and extension for my file on the fly. The following is from `man screencapture`.
```
SYNOPSIS
     screencapture [-SWCTMPcimswxto] file
```
I learned `<file>` is the path to destination of where the screenshot will be saved. I also knew that I wanted to select only a portion of the screen with my mouse when taking the screenshot. After exploring the documentation for this utility more I found the following flag.
```
-s      Only allow mouse selection mode.
```
So I came to the conclusion that I would be invoking the `screencapture` utility from my script as follows.
```
screencapture -s <path_to_file>
```
I felt comfortable enough with the utility to proceed after identifying how it would be used. All that was left was a matter of writing the bash script and making the file and extension names dynamic.

I also decided that it would be very helpful to have the `<path_to_file>` copied to my clipboard. This would make the file easy to open in preview on the command line after the script ran with the command: `open <path_to_file>`. Having the path to the clipboard would also make it easier to locate in `Finder` for Mac (You can enter absolute path in `Finder` with `command` + `shift` + `G`).
Luckily, Mac has the `pbcopy` command that allows you to paste to your system `Clipboard` via the command line. You can read more about `pbcopy` by typing `man pbcopy` in your terminal.

# Writing The Script

I will not go a ton into bash scripting in this tutorial. There are plenty of other great tutorials and resources for that online. However, I will provide a brief description of my thought process and detailed comments within the attached snippet for the script.

After looking into the `screencapture` utility, I learned that it requires one argument (`<path_to_file>`). This means that my script would at minimum require one argument as well. I didn't want to type out an absolute path or always invoke the script from my screenshot directory, so I decided I would only pass the file's base name to my script, and keep the path to my desired directory in a variable. I decided to concatenate the base file name argument with my desired directory within my script and pass this resulting absolute path to the `screencapture` command.

I decided that I may also want to specify the extension of the file. I'm usually fine with `png` as an extension. To limit keystrokes, I decided that this would be the default extension for my screenshots. I also decided to make the extension a second optional argument to my script.
```sh
#!/bin/bash

# assign variable SCREENSHOT_DIR value of shell ENV value SCREENSHOT_DIR
# if env SCREENSHOT_DIR is not defined, use $HOME as default directory
SCREENSHOT_DIR=${SCREENSHOT_DIR:-"$HOME"}

# use /bin/test to determine if length of string is zero
#   -z will return true if length of arg is zero
# In this case, if the first argument to this script ($1) is not defined,
#   throw error and exit script
if [[ -z "$1" ]]; then
  echo "ERROR: Please provide filename as first argument."
  exit 0
fi

# assign variable EXT value of second argument passed to script
# If no second argument is passed to script, assign 'png' as default extension
EXT=${2:-"png"}

# define timestamp that will be appended to screencapture's file path
# helpful to ensure screenshots with same name not overwritten
# using date utility (man date) for more info
timestamp=$(date +"%H_%M_%S_%p")

# Combination of
#   1. $SCREENSHOT_DIR defined above / in ENV variable
#   2. $1, the first argument passed to the script
#   3. $timestamp defined above
#   4. $EXT defined above / in second argument passed to script
SCREENSHOT_PATH="$SCREENSHOT_DIR/$1-$timestamp.$EXT"

# Add helpful output to STDOUT, notifying us of screenshot's destination
echo "screenshot will be saved to: $SCREENSHOT_PATH"

# invoke screencapture, specifying mouse capture and absolute path of destination
screencapture -s $SCREENSHOT_PATH
# At this point, mouse changes to cursor and use must select screenshot area.

# This command will copy the path to our screenshot to our clipboard
# Ready to paste with `command` + `V` shortcut
printf "$SCREENSHOT_PATH" | pbcopy
echo "This path to this screenshot has been copied to your clipboard."
```

At this point you can also create a file and paste in the above code, for the sake of this tutorial I'll say mine is at `~/screenshot.sh`.

This script may appear complicated at first sight, but it only took under 10 minutes to write!

# Calling our Script

As seen above in the script's comments, it's using an ENV var, `SCREENSHOT_DIR`. If this var is not exported to your shell, the default value in the script will be used.
Right now my default value is `"$HOME"`. You can either change that default value in your script, or define/export `SCREENSHOT_DIR` in the shell from which you run the script.

As mentioned in the last section, I decided to let this script take 2 arguments.
Assuming I saved my script to `~/screenshot.sh`, I can invoke it as follows.
```
bash ~/screenshot.sh <file_name> [<optional_extension>]
```
`<file_name>` - **This is required** By default, the file name is going to be `$SCREENSHOT_DIR/<file_name>.png`
`<optional_extension>` - (optional) If you don't want the file extension to be `png` you can define it with this arg. This option could be `jpg` for a `.jpg` extension.

All you have to do is:
1. Run script
2. Select area to capture with your mouse
3. The path will be copied to your clipboard! If you need to open the file, you can just type `open`, then paste the path in to your terminal!

# Setting up a bash alias
The point of this script in the first place was to save time. For that reason you probably don't want to type `bash <path_to_screenshot_script> <file_name> [<optional_extension>]` every single time.
Luckily, it's very easy to set up a bash alias for this script!
If you are unfamiliar with `aliases` in bash, there is an excellent overview here http://tldp.org/LDP/abs/html/aliases.html.

I chose to alias the execution of this script to `snap`. So instead of starting this command with `bash <path_to_screenshot_script>`, I can just type `snap`.
Note that you can make the following alias name whatever you want though...

Add the following to your `~/.bashrc`, `~/.bash_profile`, or anywhere that gets sourced when starting a new shell. (I suggest eventually setting up a file for all `aliases` like a `~/.bash_aliases` file. There are plenty of great tutorials for that online also).

For the purpose of this tutorial we'll put our `alias` in `~/.bash_profile` since this should be getting sourced on your Mac.

```sh
alias snap="bash ~/scripts/screencapture.sh"
```
As long as the above gets sourced when you open your new shell (or run `. ~/.bash_profile` to do in your current terminal), you will be able to use `snap` as an alias for our script!

# Running Script With Our New Alias
Now that our `alias` is set up, in your terminal, type:
```
snap <file_name> [<optional_path>]
```
Even though we have an alias, the arguments the script accepts remain the same. Just like we saw when running the script without an alias, you will everything will work exactly as it did before.

Just to recap:
1. Run script
2. Select area to capture with your mouse
3. The path will be copied to your clipboard! If you need to open the file, you can just type `open`, then paste the path in to your terminal!

# Conclusion
Hopefully after reading this, you will see how easy it is to make little tweaks in your workflow. As I mentioned before, the above script took me under 10 minutes to write. Now my cumbersome process of taking a screenshot is done right from my terminal in about a minute. I chose to write a bash script, but the above can be accomplished with pretty much any language you choose. I have used `Ruby`, `Perl`, and `Python` in the past to write scripts too!

Next time you find yourself wanting to complain about a task you are repeatedly performing, I suggest taking a step back and thinking about if there is a way to automate it. Even if it can't be fully automated, I find that there is usually some way to make it less painful with the use of scripts. I'm always surprised how a script that takes minutes to write can ultimately save me many hours. As engineers, we have tools at our disposal to make our lives easier, it's just a matter of identifying and implementing them.

Happy scripting!
