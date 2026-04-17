```md
# Phase 5: Analysis

## Q5.1: Implementing `pes checkout <branch>`

### Core Logic

**1. Reference Update**
- Update the `.pes/HEAD` file to point to the new branch reference  
- Example:
```

ref: refs/heads/main → ref: refs/heads/feature

```

**2. Working Directory Sync**
- Remove current files in the working directory  
- Replace them with files from the target branch's latest commit (tree object)

---

### Complexity

This operation is non-trivial due to handling dirty files:

- Dirty files = uncommitted changes in the working directory  
- Overwriting them without checks leads to permanent data loss  

Therefore, safety checks are essential before switching branches  

---

## Q5.2: Detecting "Dirty Working Directory" Conflicts

A dirty state can be detected efficiently using metadata without re-hashing all files.

### 1. Working Directory vs Index

- Compare:
- mtime (modification time)
- file size  
- Stored in: Index  

If different → Unstaged changes exist  

---

### 2. Index vs HEAD

- Compare:
- File hash in Index  
- File hash in HEAD commit (Object Store)  

If different → Staged but uncommitted changes exist  

---

### 3. Conflict Detection

The directory is considered dirty if:
- Either comparison shows differences  

Conflict condition:
- Dirty files also differ in the target branch  

Result:
- Checkout should be blocked or warned  

---

## Q5.3: Detached HEAD State

### Definition

A Detached HEAD occurs when:
```

.pes/HEAD contains a raw commit hash

```

Instead of:
```

ref: refs/heads/<branch>

```

---

### Committing in Detached HEAD

- Commits work normally  
- However:
  - No branch pointer moves forward  
  - Commits are not linked to any branch  

---

### Recovery

If the user switches away:
- These commits become orphaned  

To recover:
1. Find the commit hash (via terminal history or reflog)  
2. Create a new branch pointing to it:
```

pes branch <new-branch> <commit-hash>

```
```

