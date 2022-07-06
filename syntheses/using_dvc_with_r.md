# Using DVC (with R)

## Introduction

Official documentation: <https://dvc.org/doc>

About installation: <https://dvc.org/doc/install>

The repository corresponding to this note can be found at <https://github.com/nglvnt/dvc-practice>.

## Initialize DVC

DVC builds on Git, hence a Git repository is needed before we can initialize DVC. Let us create one quickly!

```console
$ mkdir dvc-practice
$ cd dvc-practice
$ git init
Initialized empty Git repository in /home/levente/projects/dvc-practice/.git/
$ git checkout -b main
Switched to a new branch 'main'
$ printf "# Repository for practicing and experimenting with DVC\n" > README.md
$ git add README.md
$ git commit -m "Initialize repository"
[main (root-commit) db8512d] Initialize repository
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

Now run the `dvc init` command to initialize DVC.

```console
$ dvc init
Initialized DVC repository.

You can now commit the changes to git.

What's next?
------------
- Check out the documentation: <https://dvc.org/doc>
- Get help and share ideas: <https://dvc.org/chat>
- Star us on GitHub: <https://github.com/iterative/dvc>
```

As the output writes, we have successfully initialized the DVC repository. This means that directories and files have been created, moreover, some of them have been automatically added to the Git staging area, let us commit them now!

```console
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
  new file:   .dvc/.gitignore
  new file:   .dvc/config
  new file:   .dvcignore

$ git commit -m "Initialize DVC repository"
[main e2bdd8f] Initialize DVC repository
 3 files changed, 6 insertions(+)
 create mode 100644 .dvc/.gitignore
 create mode 100644 .dvc/config
 create mode 100644 .dvcignore
```

We are now all set up!

### Deep dive on initialization

A detailed description of the created files can be found [here](https://dvc.org/doc/user-guide/project-structure/internal-files), but we can also get a glimpse of them using `tree` (and ignore the .git directory).

Detailed reference for dvc init: <https://dvc.org/doc/command-reference/init>

```console
$ tree -a -I ".git"
.
├── .dvc
│   ├── config
│   ├── .gitignore
│   └── tmp
│       ├── links
│       │   └── cache.db
│       └── md5s
│           └── cache.db
├── .dvcignore
└── README.md

4 directories, 6 files
```

## Data versioning

### Adding new data

To start use DVC, we will create an R script file that loads and saves the data in the *data* subdirectory as a csv file.

```console
$ mkdir data
```

The data is the **diamonds** dataset from the **ggplot2** package: the script loads it from the package and saves it as a csv file, imitating a - usually much more complicated - data loading script.

```r
if (!require("ggplot2", character.only = TRUE, quietly = TRUE)) {
  install.packages("ggplot2")
}

data(diamonds, package = "ggplot2")

write.csv(x = diamonds, file = "data/diamonds.csv", row.names = FALSE)
```

After running it, we get the data file and `git status` tells us about the two new untracked files.

```console
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
  data/
  get_diamonds_data.R

nothing added to commit but untracked files present (use "git add" to track)
```

Instead of adding and committing the data file to Git, we will use the `dvc add` command to start tracking it with DVC.

```console
$ dvc add data/diamonds.csv
100% Adding...|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████|1/1 [00:00, 15.73file/s]
                                                                                                                                                                                                           
To track the changes with git, run:

    git add data/.gitignore data/diamonds.csv.dvc

To enable auto staging, run:

  dvc config core.autostage true
```

The command has created two files in the *data* directory: diamonds.csv.dvc and .gitignore. What is the purpose of these files? The .gitignore file lists the data file itself, instructing Git to not track it.

```console
$ cat data/.gitignore
/diamonds.csv
```

The .dvc file contains information on the data file. `dvc add` moved the data to the project's DVC cache, and linked it back to the workspace. The hash value of the diamonds.csv file determines the path in the DVC cache.

```console
$ cat data/diamonds.csv.dvc
outs:
- md5: 4d3d1d4bbad5e0806dbaec425cf90196
  size: 2772143
  path: diamonds.csv
```

```console
$ tree .dvc/cache
.dvc/cache
└── 4d
    └── 3d1d4bbad5e0806dbaec425cf90196

1 directory, 1 file
```

```console
$ head -n5 .dvc/cache/4d/3d1d4bbad5e0806dbaec425cf90196
"carat","cut","color","clarity","depth","table","price","x","y","z"
0.23,"Ideal","E","SI2",61.5,55,326,3.95,3.98,2.43
0.21,"Premium","E","SI1",59.8,61,326,3.89,3.84,2.31
0.23,"Good","E","VS1",56.9,65,327,4.05,4.07,2.31
0.29,"Premium","I","VS2",62.4,58,334,4.2,4.23,2.63
```

The newly created file have **not** been staged automatically, we should stage them. DVC can auto-stage the files by setting `dvc config core.autostage true`.

```console
$ git add get_diamonds_data.R data/diamonds.csv.dvc data/.gitignore
$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
  new file:   data/.gitignore
  new file:   data/diamonds.csv.dvc
  new file:   get_diamonds_data.R
```

We can now commit the changes.

```console
$ git commit -m "Add diamonds data"
[main 59d0e12] Add diamonds data
 3 files changed, 12 insertions(+)
 create mode 100644 data/.gitignore
 create mode 100644 data/diamonds.csv.dvc
 create mode 100644 get_diamonds_data.R
```

### Changing the data

Suppose that the data to be analyzed has changed, e.g. observations and/or variables were added/removed or values itself have changed.

Let us keep only the diamonds with weight of at least 1 carat.

```r
if (!require("ggplot2", character.only = TRUE, quietly = TRUE)) {
  install.packages("ggplot2")
}

data(diamonds, package = "ggplot2")

## we only would like to analyze diamonds of weight at least 1 carat
diamonds <- diamonds[diamonds$carat >= 1, ]

write.csv(x = diamonds, file = "data/diamonds.csv", row.names = FALSE)
```

`git status` tells us that the data script has changed. However, the data file is not in the "changes not staged" part. But `dvc status` tells that the data file has also changed!

```console
$ git status
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
  modified:   get_diamonds_data.R

no changes added to commit (use "git add" and/or "git commit -a")
```

```console
$ dvc status
data/diamonds.csv.dvc:                                                
  changed outs:
    modified:           data/diamonds.csv
```

We first use `dvc add`, the diamonds.csv.dvc file has now changed also. Then git add and commit

```console
$ dvc add data/diamonds.csv
100% Adding...|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████|1/1 [00:00, 29.42file/s]
                                                                                                                                                                                                           
To track the changes with git, run:

    git add data/diamonds.csv.dvc

To enable auto staging, run:

  dvc config core.autostage true

$ git add get_diamonds_data.R data/diamonds.csv.dvc
$ git commit -m "Keep only the diamonds of weight at least 1 carat"
[main 0498c0a] Keep only the diamonds of weight at least 1 carat
 2 files changed, 5 insertions(+), 2 deletions(-)
```

What happened? `git diff` shows that the md5 hash and the file size have been updated in the .dvc file.

```console
$ git diff HEAD^ HEAD data/diamonds.csv.dvc 
diff --git a/data/diamonds.csv.dvc b/data/diamonds.csv.dvc
index 2c70926..c511ebe 100644
--- a/data/diamonds.csv.dvc
+++ b/data/diamonds.csv.dvc
@@ -1,4 +1,4 @@
 outs:
-- md5: 4d3d1d4bbad5e0806dbaec425cf90196
-  size: 2772143
+- md5: 3b72460f4050794ad9cb812cdb8b168a
+  size: 986265
   path: diamonds.csv
```

There is a new file in cache with the new hash and the data/diamonds.csv now links to it.

```console
$ tree .dvc/cache
.dvc/cache
├── 3b
│   └── 72460f4050794ad9cb812cdb8b168a
└── 4d
    └── 3d1d4bbad5e0806dbaec425cf90196

2 directories, 2 files
```

### Adding a new branch

Before going on, we create a new branch based on our initial version of the script and leave out the columns related to the dimension of the diamonds:

```console
$ git checkout -b data_without_dimensions
Switched to a new branch 'data_without_dimensions'
```

```r
if (!require("ggplot2", character.only = TRUE, quietly = TRUE)) {
  install.packages("ggplot2")
}

data(diamonds, package = "ggplot2")

## we only would like to analyze diamonds of weight at least 1 carat
diamonds <- diamonds[diamonds$carat >= 1, ]

# leave out the columns related to the dimension of the diamonds:
# depth, table, x, y, z
diamonds <- diamonds[c("carat", "cut", "color", "clarity", "price")]

write.csv(x = diamonds, file = "data/diamonds.csv", row.names = FALSE)
```

```console
$ dvc add data/diamonds.csv
100% Adding...|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████|1/1 [00:00, 21.80file/s]
                                                                                                                                                                                                           
To track the changes with git, run:

    git add data/diamonds.csv.dvc

To enable auto staging, run:

  dvc config core.autostage true

$ git add get_diamonds_data.R data/diamonds.csv.dvc
$ git commit -m "Leave dimension-related columns out of the data"
[data_without_dimensions af648be] Leave dimension-related columns out of the data
 2 files changed, 6 insertions(+), 2 deletions(-)
```

### Switching between different versions

We are now on the data_without_dimensions branch and let us see how our data looks.

```console
$ git branch
* data_without_dimensions
  main
$ head -n5 data/diamonds.csv
"carat","cut","color","clarity","price"
1.17,"Very Good","J","I1",2774
1.01,"Premium","F","I1",2781
1.01,"Fair","E","I1",2788
1.01,"Premium","H","SI2",2788
```

Now we switch to the main branch. Running git status tells us that everything is OK, but dvc status indicates that the data is out-of-sync.

```console
$ git checkout main
Switched to branch 'main'
$ git status
On branch main
nothing to commit, working tree clean
$ dvc status
data/diamonds.csv.dvc:                                                
  changed outs:
    modified:           data/diamonds.csv
```

What happens here? The diamonds.csv file is not under version control by Git, hence when we changed back to the main branch, it did not change. But the diamonds.csv.dvc file has, and that is how DVC can tell that the data file is out of sync.

We can get the data back with dvc checkout, and we can check that it has indeed changed.

```console
$ dvc checkout
M       data/diamonds.csv
$ head -n5 data/diamonds.csv
"carat","cut","color","clarity","depth","table","price","x","y","z"
1.17,"Very Good","J","I1",60.2,61,2774,6.83,6.9,4.13
1.01,"Premium","F","I1",61.8,60,2781,6.39,6.36,3.94
1.01,"Fair","E","I1",64.5,58,2788,6.29,6.21,4.03
1.01,"Premium","H","SI2",62.7,59,2788,6.31,6.22,3.93
```

We can also use the checkout commands to get an earlier version of the data back. First, we check out the .dvc file and then we sync the data.

```console
$ git checkout 59d0e12bc6e35db496ac8aa3acfea075aa07bd2f data/diamonds.csv.dvc
$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
  modified:   data/diamonds.csv.dvc

$ dvc status
data/diamonds.csv.dvc:                                                
  changed outs:
    modified:           data/diamonds.csv
$ dvc checkout
M       data/diamonds.csv
```

We get back the diamonds of weight less than 1 carat.

```console
$ head -n5 data/diamonds.csv
"carat","cut","color","clarity","depth","table","price","x","y","z"
0.23,"Ideal","E","SI2",61.5,55,326,3.95,3.98,2.43
0.21,"Premium","E","SI1",59.8,61,326,3.89,3.84,2.31
0.23,"Good","E","VS1",56.9,65,327,4.05,4.07,2.31
0.29,"Premium","I","VS2",62.4,58,334,4.2,4.23,2.63
```

Let us clean up the changes.

```console
$ git restore --staged .
$ git restore .
$ dvc checkout
M       data/diamonds.csv 
```

### Trackable objects

We have seen that DVC does not track the changes of the content of the data files, but keeps track of the different versions of the files. Hence not just data can be tracked, but anything else: models, plots, generated html files, etc. Let us see an example with an image.

```console
$ mkdir outputs
```

Let us create a script that loads the diamonds.csv file and outputs a scatterplot of price vs. carat.

```r
diamonds <- read.csv(file = "data/diamonds.csv")

png(filename = "outputs/scatterplot_price_vs_carat.png")

plot(x = diamonds$carat
     , y = diamonds$price
     , main = "Scatterplot of price vs. carat"
     , xlab = "Carat"
     , ylab = "Price"
     ) 

dev.off()
```

![scatterplot price vs carat](/images/using_dvc_with_r_scatterplot_price_vs_carat.png)

Now we can add the previously created png file to DVC.

```console
$ dvc add outputs/scatterplot_price_vs_carat.png
100% Adding...|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████|1/1 [00:00, 24.70file/s]
                                                                                                                                                                                                           
To track the changes with git, run:

    git add outputs/scatterplot_price_vs_carat.png.dvc outputs/.gitignore

To enable auto staging, run:

  dvc config core.autostage true
$ git add create_scatterplot_price_vs_carat.R outputs/scatterplot_price_vs_carat.png.dvc outputs/.gitignore
$ git commit -m "Create price vs. carat scatterplot"
[main 157b279] Create price vs. carat scatterplot
 3 files changed, 17 insertions(+)
 create mode 100644 create_scatterplot_price_vs_carat.R
 create mode 100644 outputs/.gitignore
 create mode 100644 outputs/scatterplot_price_vs_carat.png.dvc
```

And now any change in the plot can be tracked as well.

### Getting the list of tracked objects

The following command prints out all the objects (in all subdirectories) tracked by DVC:

```console
$ dvc list --dvc-only --recursive .
data/diamonds.csv
outputs/scatterplot_price_vs_carat.png
```
