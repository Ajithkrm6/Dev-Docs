```
# Step 1: Clone git-filter-repo
```

cd C:\Users\Ajith
git clone https://github.com/newren/git-filter-repo.git

# Step 3: Test if git-filter-repo is working

git filter-repo --help

# OR (if not in PATH)

python C:/Users/Ajith/git-filter-repo/git-filter-repo.py --help

# Step 4: Remove the folder from history

cd path/to/your/nextjs-project
python C:/Users/Ajith/git-filter-repo/git-filter-repo.py --path public/videos --invert-paths

# Step 5: Force push the changes

git push origin --force --all

````
Let's go through a few troubleshooting steps to make sure the file is fully removed from your Git history and then push the changes.

Step 1: Double-check that the file is removed from the history
Run this command to check if the file is still in the history:

```bash
git rev-list --all --objects | grep "public/videos/BiriyaniFinalCut.mp4"
````

If the file appears in the output, it means it's still in the history. If it doesn't appear, you may just need to clear the cache and try pushing again.

Step 2: Run git-filter-repo again with an extra option
To ensure complete removal, run git-filter-repo with the --force option. This will force git-filter-repo to rewrite the history without skipping any commits.

```bash
python C:/Users/Ajith/git-filter-repo/git-filter-repo.py --path public/videos --invert-paths --force
```

Step 3: Clean up the Git cache
Run the following command to remove any lingering references to the deleted files:

```bash
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

This will clear any cached references to the file.

Step 4: Verify that the file is removed from history
Run the following command again to verify that the large file no longer exists in your history:

```bash
git rev-list --all --objects | grep "public/videos/BiriyaniFinalCut.mp4"
```

If the file is gone, proceed to the next step. If it still shows up, try repeating Step 2, as sometimes it may take a couple of attempts to fully remove it.

Step 5: Force push the cleaned history to GitHub
Now, force-push the changes to update your GitHub repository with the cleaned history:

```bash
git push origin --force --all
```

Step 6: Verify on GitHub
Once the push completes, go to your GitHub repository and verify that the large file is no longer present. GitHub should no longer reject the push if the file has been removed from the history.
