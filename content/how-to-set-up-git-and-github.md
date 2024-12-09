---
title: 'How to set up Git and Github'
date: 2024-11-18T10:58:08+10:00
draft: false
---
# How to set up Git and Github

Git is a version control system created by Linus Torvalds, also known as the creator of Linux. He named Git after himself as he sees himself as an "egotistical bastard", condensed into the word "git".

Git just allows you to track your system progress so that if something breaks you can always revert back to a previous save. Think of git like a save file in a video game.

In the project folder, type the following in the command line:
```bash
git init
```
Initialises your project by creating a hidden git folder as well as a [.gitignore file.](obsidian://open?vault=docs&file=Ignoring%20files%20in%20Git)

```bash
git status
```
You can see the status of your committed/uncommitted files with git status. Doesn't 'do anything' but gives you some information about the process.

```bash
git add .
```
Creates a snapshot of the current state of the files in your folder to be saved. You can add single files or folders by writing their names instead.

```bash
git commit -m "commit my commit message"
```
Saves all the snapshotted files added. the "-m" allows you to write a short message explaining what the changes do. Common culture is to write in the imperative, ex "create nextjs app" rather than "i created a new nextjs app".

```bash
git branch -M main
```
Sets the name of your main branch to "main", as is common practice for your main project branch. Branches are used so that you can make separate changes to a project without touching the

### Now, for adding github:
On your github account, create a new repository for your project.
Run the following, switching out the link for your project link.
```bash
git remote add origin https://github.com/username/new-project.git
```
Connects your local git to your remote github repository. 

```bash
git push -U origin main
```
Pushes your most recent commit to the github project with "git push" and sets the default remote branch(also called upstream branch) to the main branch.

After this you only need to push with  `git push`

