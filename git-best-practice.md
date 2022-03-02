# Using Git and GitHub at Elbel lab
All research projects should be created in the lab GitHub account (instead of your personal account). A helpful [guide](https://swcarpentry.github.io/git-novice/) to get started. You can download the git client [here](https://git-scm.com/downloads).

## Create a Git project on GitHub
- If you already have a repo for a project somewhere else (e.g. your personal account), you could import the existing repo into our team account. Click on *New* --> *Import a repository*.
- Repository naming convention:
   - Specific to your project. Do not simply label it with the grant name, such as tacobell, or soda tax 
   - Do not leave space in the name
- Each repo should have:
  -  A readme file. In the readme.md file, give a brief description of the research project, and what each script in the repo does (to help others read your code)
  -  A .gitignore
- Repo features
  - Make repos private
  - You should be the sole owner of the repos you create, unless you are collaborating with other analysts
  - Add team members to your repos with Read or Triage access. Click on a repo, go to *Settings* tab --> (on the left hand side) *Collaborators and teams* --> *Add teams* --> type in *analytical-team* --> choose the appropriate level of access
- Create a link to the analytical repo in the [*project-overview*](https://github.com/Brian-Elbel-s-Research-Projects/project-overview) page
- When you hand off or take over a project
   - Make sure your successor has admin access to the repo
   - Complete the readme.md so the information is up to date
   - Use ```git pull``` or ```git clone``` to copy existing scripts to your local drive

## Best practice
A couple of things to note when you incorporate Git into your workflow.
- For the repo
  - Keep only code on the repo (add other folders and files to .gitignore)
  - No PHI on GitHub, i.e. no hard coding such as ```data$name[data$ssn=='000-11-2222'] <- 'jane'``` You could pull in reference data like this from an external .csv file.
- Daily workflow
  - Write a meaningful commit message, i.e. "end of day Jan 1, 2022" is not a great commit message
  - You don't have to make a commit everyday, unless someone else's coding depends on yours

## Learning markdown
To make a readme.md, you'll have to learn at least some basic markdown, get started [here](https://www.markdownguide.org/getting-started/) and get a cheatsheet [here](https://www.markdownguide.org/cheat-sheet/).
