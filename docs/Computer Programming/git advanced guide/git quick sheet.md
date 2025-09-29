# Git Quick Sheet

{% embed url="https://www.atlassian.com/git/tutorials" %}

## Git Clone

## Git Credential config

```
// Some code
$ git config --global credential.helper store
(the default is ~/.git-credentials)
```

## Git Branch

// Some code&#x20;

git branch -a&#x20;

git checkout --track origin/branch-name

#### delete a branch

```
// Some code
git branch -d <branchname>
git branch -D <branchname>
```

#### show all branch

```
// Some code
git branch -a -vv
git branch -r   ## remote 

```

## Git Remote

<pre><code>// Some code
git remote add my-own-remote https://github.com/your-username/your-repository.git
<strong>
</strong><strong>// If you want to configure Git to fetch only branches that start with "active," you can modify the fetch configuration accordingly. Here's an example:
</strong>git config remote.my-own-remote.fetch "+refs/heads/active*:refs/remotes/my-own-remote/*"



</code></pre>



</code></pre>
````

##
