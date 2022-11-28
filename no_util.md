# Summary 
For this challenge we are given a linux box with no access to busy box, meaning we have no access to ls and find, as well as many other commands. Meaning we have to use base bash commands to find the flag within the box.

# Main Discovery
By using \*, bash expands that to be the files within the current directory, so by using `ehco *` we can view the files in the current directory.

# Finding flag with script
With this new knowledge we can make a recursive file checker to find the flag, that way we don't have to dig around the box aimlessly.

```bash
list() { 
for VAR in "$1/"*;
do
  if [ -e flag.txt ]; then
    echo Found;
    pwd;
    return;
  else 
        if [ -d "$VAR" ] && [ "$VAR" != * ];
        echo "$VAR"
        then list "$VAR"
        fi 
  fi
done;
};

```

Here the function checks the directory and any subdirectoreis of our input. This script then returns the location of the flag and then we can submit it for points.