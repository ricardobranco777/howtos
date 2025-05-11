
Import a project from another git repository into a directory:

```
branch=master
project=wol
directory=wol
url=https://github.com/$USER/$project

# Do everything on a new branch
git checkout -b $project
git remote add $project $url
git fetch wol $branch
git merge -s ours --no-commit --allow-unrelated-histories wol/$branch

# No need to create the directory
git read-tree --prefix=$directory/ -u $project/$branch
git commit -m "Import $project"

# Optional:
git rebase -i $branch

git checkout $branch
git merge $project
git push

git remote rm wol
```
