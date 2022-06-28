# Using DVC (with R)

## Introduction

Official documentation: <https://dvc.org/doc>

About installation: <https://dvc.org/doc/install>

## Initialize DVC

The repository corresponding to this note can be found at <https://github.com/nglvnt/dvc-practice>.

Inside a Git repository let's run the **dvc init** command.

```bash
dvc init
```

Initialization means that a lot of files have been created and automatically staged.

What files are created? <https://dvc.org/doc/user-guide/project-structure/internal-files>

Detailed reference for dvc init: <https://dvc.org/doc/command-reference/init>  

```bash
git status
```

Now we should commit these files.

```bash
git commit -m "Initialize DVC repository"
```

![dvc init](/images/using_dvc_with_r_dvc_init.png)

## Data versioning

### Adding new data

Here we will create an R script that loads and saves the data in the data subdirectory

```bash
mkdir data
```

The data is the diamonds dataset from the ggplot2 package: the script loads it from the package and saves it as a csv file, this imitates the usually much more complicated data loading scripts.

```r
.libPaths("~/R/x86_64-pc-linux-gnu-library/4.2")

if (!require("ggplot2", character.only = TRUE, quietly = TRUE)) {
  install.packages("ggplot2")
}

data(diamonds, package = "ggplot2")

write.csv(x = diamonds, file = "data/diamonds.csv", row.names = FALSE)
```

After running the script, we get our data file. git status tells us that we have new untracked files. We will stage the data-creating script:

```bash
git add get_diamonds_data.R
```

However, instead of staging the data file with Git, we will track it in DVC, hence we use the dvc add command.

```bash
dvc add data/diamonds.csv
```

Creates a diamonds.csv.dvc file and a gitignore file.

The newly created file were NOT staged automatically, we should stage them. DVC can auto-stage the files with `dvc config core.autostage true`.

```bash
git add data/diamonds.csv.dvc data/.gitignore
```

We can now commit.

```bash
git commit -m "Add diamonds data"
```

![dvc add](/images/using_dvc_with_r_dvc_add.png)

What happens under the hood? dvc add moved the data to the project's DVC cache, and linked it back to the workspace. The hash value of the diamonds.csv file determines the path in the DVC cache. Moreover, if one checks the dvc file created, the hash value can be found there, too.

The content of the dvc file:

```bash
cat data/diamonds.csv.dvc
```

The contents of the DVC cache and the file in the cache

```bash
tree .dvc/cache
head -n5 .dvc/cache/4d/3d1d4bbad5e0806dbaec425cf90196
```

![dvc add content](/images/using_dvc_with_r_dvc_add_content.png)

### Changing the data

Suppose that the data to be analyzed has changed, e.g. observations and/or variables were added/removed or values have changed.

We update our script and keep only the diamonds with weight of at least 1 carat.

```r
.libPaths("~/R/x86_64-pc-linux-gnu-library/4.2")

if (!require("ggplot2", character.only = TRUE, quietly = TRUE)) {
  install.packages("ggplot2")
}

data(diamonds, package = "ggplot2")

## we only would like to analyze diamonds of weight at least 1 carat
diamonds <- diamonds[diamonds$carat >= 1, ]

write.csv(x = diamonds, file = "data/diamonds.csv", row.names = FALSE)
```

Git status tells us that the data script has changed. However, the data file is not in the "changes not staged" part. But DVC status tells that the data file has also changed!

```bash
git status
```

```bash
dvc status
```

We first use dvc add, the diamonds.csv.dvc file has now changed also. Then git add and commit

```bash
dvc add data/diamonds.csv
git add get_diamonds_data.R data/diamonds.csv.dvc
git commit -m "Keep only the diamonds of weight at least 1 carat"
```

![dvc add after change](/images/using_dvc_with_r_dvc_add_after_change.png)

What happened?

There is a new file in cache and the content of the .dvc file has changed:

```bash
cat data/diamonds.csv.dvc
tree .dvc/cache
```

![dvc add after change content](/images/using_dvc_with_r_dvc_add_after_change_content.png)

### Adding a new branch

Before going on, we will create a new branch based on our initial version of the script and leave out the columns related to the dimension of the diamonds:

```bash
git checkout -b data_without_dimensions
```

```r
.libPaths("~/R/x86_64-pc-linux-gnu-library/4.2")

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

```bash
dvc add data/diamonds.csv
git add get_diamonds_data.R data/diamonds.csv.dvc
git commit -m "Leave dimension-related columns out of the data"
```

### Switching between different versions

We are now on the data_without_dimensions branch. Let us go back to the main branch and see how can we get back the data. First we check that we are on the right branch and how our data looks.

```bash
git branch
head data/diamonds.csv
```

Now we switch to the main branch. Running git status tells us that everything is OK, but dvc status indicates that the data is out-of-sync.

```bash
git checkout main
git status
dvc status
```

What happens here? The diamonds.csv file is not under version control by Git, hence when we changed back to the main branch, it did not change. But the diamonds.csv.dvc file has, and that is how DVC can tell that the data file is out of sync.

We can get the data back with dvc checkout, and we can check that it has indeed changed.

```bash
dvc checkout
head data/diamonds.csv
```

![dvc checkout](/images/using_dvc_with_r_dvc_checkout.png)

We can also use the checkout commands to get an earlier version of the data back. First, we check out the .dvc file and then we sync the data.

```bash
head data/diamonds.csv
```

```bash
git checkout 7118fd5 data/diamonds.csv.dvc
git status
dvc status
dvc checkout
```

```bash
head data/diamonds.csv
```

![dvc checkout past](/images/using_dvc_with_r_dvc_checkout_past.png)

Let us clean up the changes.

```bash
git restore --staged .
git restore .
dvc checkout
```

### Trackable objects

We have seen that DVC does not track the changes of the content of the data files, but keeps track of the different versions of the files. Hence not just data can be tracked, but anything else, for example, plots: either in an object or the image file.

```bash
mkdir outputs
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

Now we can add the previously created png file to DVC.

```bash
git add create_scatterplot_price_vs_carat.R
dvc add outputs/scatterplot_price_vs_carat.png
git add outputs/scatterplot_price_vs_carat.png.dvc outputs/.gitignore
git commit -m "Create price vs. carat scatterplot"
```

![scatterplot price vs carat](/images/using_dvc_with_r_scatterplot_price_vs_carat.png)

And now any change in the plot can be tracked as well.

### Getting the list of tracked objects

The following command prints out all the objects (in all subdirectories) tracked by DVC:

```bash
dvc list --dvc-only --recursive .
```
